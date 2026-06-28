# tasks, calendar, contacts

sadly, no (free?) tasks app can connect directly with Nextcloud.

luckily, there is a connector: DAVx5

## connection app

get DAVx5 app:
* https://www.davx5.com
* f-droid (free): https://f-droid.org/de/packages/at.bitfire.davdroid/
* google (6€): https://play.google.com/store/search?q=davx5&c=apps&hl=de

## tasks

overall straight-forward. only draw-back: you cannot create new lists in the app. you have to do that on Nextcloud web. Sharing a list as well.

### app

Tasks.org
* f-droid: https://f-droid.org/de/packages/org.tasks/
* google: https://play.google.com/store/apps/details?id=org.tasks&hl=de&pli=1

### setup

in DAVx5:

address: `<nextcloud url>/remote.php/dav/calendars`

## calendar

### app

get Etar. closest you get to google calendar

* google: https://play.google.com/store/search?q=etar&c=apps&hl=de
* f-droid: https://f-droid.org/de/packages/ws.xsoh.etar/


### setup

tbd

## contacts

tbd.

# migrate Nextcloud to Nextcloud AIO

(-1.) Update/upgrade your Nextcloud
(0.) if you have anything else but postgreSQL in your original nextcloud, migrate to postgreSQL database
1. save your `./data/` folder somewhere else
2. create sql dump
3. run AIO, wait till it works

```
# delete current database content
docker exec -it nextcloud-aio-database psql -U nextcloud -d nextcloud_database -c "DROP SCHEMA public CASCADE; CREATE SCHEMA public;"

# import previous database
DUMP=/home/achim/docker/nextcloud/nextcloud_dump.sql
cat ${DUMP} | docker exec -i nextcloud-aio-database psql -U nextcloud -d nextcloud_database

# fix db user from nextcloud to oc_nextcloud
docker exec -it nextcloud-aio-database psql -U nextcloud -d postgres -c "ALTER DATABASE nextcloud_database OWNER TO oc_nextcloud;"
docker exec -it nextcloud-aio-database psql -U nextcloud -d nextcloud_database -c "DO \$\$ DECLARE r RECORD; BEGIN FOR r IN (SELECT tablename FROM pg_tables WHERE schemaname = 'public') LOOP EXECUTE 'ALTER TABLE ' || quote_ident(r.tablename) || ' OWNER TO oc_nextcloud'; END LOOP; END \$\$;"

# check if all users have been migrated (this should show a number equal to your users)
docker exec -it nextcloud-aio-database psql -U nextcloud -d nextcloud_database -c "SELECT count(*) FROM oc_users;"

###### change db admin password

# obtain db password
DBPASS=$(sudo cat /var/lib/docker/volumes/nextcloud_aio_nextcloud/_data/config/config.php | grep dbpassword | cut -d ' ' -f 5 | sed "s/[',]//g") ; echo $DBPASS

# change oc_nextcloud password
docker exec -it nextcloud-aio-database psql -U nextcloud -d postgres -c "ALTER USER oc_nextcloud WITH PASSWORD '${DBPASS}';"

###### fix db Permissions

# 1. Sicherstellen, dass das Schema dem Nextcloud-User gehört
docker exec -it nextcloud-aio-database psql -U nextcloud -d nextcloud_database -c "ALTER SCHEMA public OWNER TO oc_nextcloud;"

# 2. Explizite Rechte für alle Tabellen und Sequenzen vergeben
docker exec -it nextcloud-aio-database psql -U nextcloud -d nextcloud_database -c "GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO oc_nextcloud;"
docker exec -it nextcloud-aio-database psql -U nextcloud -d nextcloud_database -c "GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO oc_nextcloud;"

# 3. Den Suchpfad zur Sicherheit noch einmal fest hämmern
docker exec -it nextcloud-aio-database psql -U nextcloud -d nextcloud_database -c "ALTER USER oc_nextcloud SET search_path TO public,oc_nextcloud;"

###### clean db environment

# set search path of new user
docker exec -it nextcloud-aio-database psql -U nextcloud -d nextcloud_database -c "ALTER USER oc_nextcloud SET search_path TO public;"

# get Redis pass and clear cache
PASS=$(docker exec nextcloud-aio-nextcloud grep "redis" -A 5 /var/www/html/config/config.php | grep "password" | cut -d"'" -f4)
docker exec -it nextcloud-aio-redis redis-cli -a "$PASS" FLUSHALL

###### Final steps
# upgrade instance
docker exec --user www-data -it nextcloud-aio-nextcloud php occ upgrade
```

9. stop all containers via AIO admin center (port 8080)
10. restart containers
11. login as admin, disable email, enable email; didn't work for me once, so I did this; had to enter credentials again, but worked instantly

Restart mail app
```
# disable
docker exec --user www-data -it nextcloud-aio-nextcloud php occ app:disable mail
# uninstall
docker exec -it nextcloud-aio-nextcloud rm -rf /var/www/html/custom_apps/mail
# install
docker exec -it --user www-data  nextcloud-aio-nextcloud php occ app:install mail
```

## Migrate your files

```
# copy files
sourceDir="./data-backup/<user>/files/"
targetDir="/mnt/nextcloud/<user>/files"

sudo cp -rv $sourceDir $targetDir

# set permissions
echo "this must be reviewed. 755 is for folders, 644 is for files"
#sudo chown -R 33:33 /mnt/nextcloud/<user>/files/
#sudo chmod -R 755 /mnt/nextcloud/<user>/files/

# clean Nextcloud file cache
docker exec --user www-data -it nextcloud-aio-nextcloud php occ files:cleanup
# update the Nextcloud index
docker exec --user www-data -it nextcloud-aio-nextcloud php occ files:scan --all
# clean Nextcloud file cache --> not sure if before or after
docker exec --user www-data -it nextcloud-aio-nextcloud php occ files:cleanup
```

# put files on network share

NFS is suggested

## create nfs share

* Create share (here: QNAP 4.x)
* open nfs settings (via users -> network shares -> search for the nextcloud share -> open "edit privileges for network share" -> new window, select privilege type "NFS host access")
* set options
  * RW
  * all_squash (can also try "no_root_squash", but did not work for me)
  * anon GID: administrators
  * anon UID: you may choose the nextcloud user on your storage, for example
<img width="825" height="139" alt="image" src="https://github.com/user-attachments/assets/71a3cc1d-1d5b-4178-9d7d-2f10d743b36b" />


## in nextcloud host

```
# stop all containers
docker stop nextcloud-aio-apache ; docker stop nextcloud-aio-nextcloud ; docker stop nextcloud-aio-database ; docker stop nextcloud-aio-redis ; docker stop nextcloud-aio-imaginary ; docker stop nextcloud-aio-talk ; docker stop nextcloud-aio-collabora ; docker stop nextcloud-aio-whiteboard ; docker stop nextcloud-aio-notify-push ; docker stop nextcloud-aio-mastercontainer ; echo "all containers stopped"

# mount share
sudo mount -t nfs <network share ip>:/nextcloud /mnt/nextcloud -o rw,relatime,vers=4,rsize=131072,wsize=131072,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys

# move files
# see hint above (copied from above, might be subject to duplicate data -> quality degradation) regarding how to copy files, e.g.:
sudo cp -rv $sourceDir $targetDir
sudo chown -R 33:33 /mnt/nextcloud/<user>/files/
sudo chmod -R 755 /mnt/nextcloud/<user>/files/
docker exec --user www-data -it nextcloud-aio-nextcloud php occ files:cleanup
docker exec --user www-data -it nextcloud-aio-nextcloud php occ files:scan --all
docker exec --user www-data -it nextcloud-aio-nextcloud php occ files:cleanup

# start master container
docker compose up -d

# go to https://<host>:8080 and start the rest of the containers
```

# create talk bot for integration with Kuma

## in CLI
`sudo docker exec -it nextcloud-aio-nextcloud php occ talk:bot:create "Uptime Kuma Bot"`
note ID and secret

## in Nextcloud
1. create new conversation, supply a name, start conversation (no need to invite somebody)
2. open sandwich menu in conversation, click "conversation settings"
3. go down to "bots", select "Uptime Kuma Bot"
4. in the conversation, look at the URL: https://nextcloud.example.com/call/abc1efgh -> abc1efgh is your conversation token

## in Kuma
1. add alert, insert conversation token, add secret from 2., test

## back in nextclodu
look in conversation in Nextcloud, be happy
