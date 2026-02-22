```
# download the installer, use stable channel, do not submit data
# this will install like 600mb via `apt`
wget -O /tmp/netdata-kickstart.sh https://get.netdata.cloud/kickstart.sh && sh /tmp/netdata-kickstart.sh --stable-channel --disable-telemetry

cd /etc/netdata
# generate a UUID
cat /proc/sys/kernel/random/uuid
# edit / create the config
./edit-config stream.conf
```

The config is already separated into 3 parts:
1. stream -> for clients
2. "[API_KEY]" -> for the host
3. "[MACHINE_GUID]" -> you can adjust settings per connecting client

# client config

```
[stream]
    enabled = yes
    destination = 192.168.0.53:19999
    # The API_KEY of the ohst, to use as the sender
    # change to what you generated above
    # no quotation marks required
    api key = f7da0e10-a374-4ed9-8ec7-8d2a2537c2f6
```

# host config 

```
# keep the brackets around the UUID
[enter your uuid from before]
    type = api
    enabled = yes
    allow from = *
    default history = 3600
    default memory mode = dbengine
```

# on both

```
systemctl restart netdata
```

# test

go to `http://<host ip>:19999`
click the tiny little "skip this" on the bottom or somewhere to remove the large "connect to our cloud" screen
done
