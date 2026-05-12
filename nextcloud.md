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
4. delete current database content -> `docker exec -it nextcloud-aio-database psql -U nextcloud -d nextcloud_database -c "DROP SCHEMA public CASCADE; CREATE SCHEMA public;"`
5. import previous database -> `$ cat /home/achim/docker/nextcloud/nextcloud_dump.sql | docker exec -i nextcloud-aio-database psql -U nextcloud -d nextcloud_database`
6. fix db user from nextcloud to oc_nextcloud

```
docker exec -it nextcloud-aio-database psql -U nextcloud -d postgres -c "ALTER DATABASE nextcloud_database OWNER TO oc_nextcloud;"
docker exec -it nextcloud-aio-database psql -U nextcloud -d nextcloud_database -c "DO \$\$ DECLARE r RECORD; BEGIN FOR r IN (SELECT tablename FROM pg_tables WHERE schemaname = 'public') LOOP EXECUTE 'ALTER TABLE ' || quote_ident(r.tablename) || ' OWNER TO oc_nextcloud'; END LOOP; END \$\$;"

# check if all users have been migrated (this should show a number equal to your users)
docker exec -it nextcloud-aio-database psql -U nextcloud -d nextcloud_database -c "SELECT count(*) FROM oc_users;"
```

7. change db admin password:
```
# obtain db password
DBPASS=$(sudo cat /var/lib/docker/volumes/nextcloud_aio_nextcloud/_data/config/config.php | grep dbpassword | cut -d ' ' -f 5 | sed "s/[',]//g") ; echo $DBPASS

# change oc_nextcloud password
docker exec -it nextcloud-aio-database psql -U nextcloud -d postgres -c "ALTER USER oc_nextcloud WITH PASSWORD '${DBPASS}';"
```

8. clean db environment

```
# set search path of new user
docker exec -it nextcloud-aio-database psql -U nextcloud -d nextcloud_database -c "ALTER USER oc_nextcloud SET search_path TO public;"

# upgrade instance
docker exec --user www-data -it nextcloud-aio-nextcloud php occ upgrade

# get Redis pass and clear cache
PASS=$(docker exec nextcloud-aio-nextcloud grep "redis" -A 5 /var/www/html/config/config.php | grep "password" | cut -d"'" -f4)
docker exec -it nextcloud-aio-redis redis-cli -a "$PASS" FLUSHALL

```

8. Set trusted proxy -> `docker exec --user www-data -it nextcloud-aio-nextcloud php occ config:system:set trusted_proxies 0 --value="192.168.0.51"` 
9. stop all containers via AIO admin center (port 8080)
10. restart containers
11. login as admin, disable email, enable email; didn't work for me once, so I did this; had to enter credentials again, but worked instantly
```
docker exec -it nextcloud-aio-nextcloud rm -rf /var/www/html/custom_apps/mail
docker exec --user www-data -it nextcloud-aio-nextcloud php occ app:install mail
```

Restart mail app
```
docker exec --user www-data -it nextcloud-aio-nextcloud php occ app:disable mail
docker exec --user www-data -it nextcloud-aio-nextcloud php occ app:enable mail
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
