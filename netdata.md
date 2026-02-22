# host config 

## install

```
# download the installer, use stable channel, do not submit data
# this will install like 600mb via `apt`
wget -O /tmp/netdata-kickstart.sh https://get.netdata.cloud/kickstart.sh && sh /tmp/netdata-kickstart.sh --stable-channel --disable-telemetry
```

## config

```
# generate a UUID
cat /proc/sys/kernel/random/uuid

# edit / create the config
cd /etc/netdata
./edit-config stream.conf
```

The config is already separated into 3 parts:
1. stream -> for clients
2. "[API_KEY]" -> for the host
3. "[MACHINE_GUID]" -> you can adjust settings per connecting client

```
# keep the brackets around the UUID
[enter your uuid from before]
    type = api
    enabled = yes
    allow from = *
    default history = 3600
    default memory mode = dbengine
```


```
cd /etc/netdata
# edit / create the config
./edit-config stream.conf
```


# client

## install

```
# download the installer, use stable channel, do not submit data
# this will install like 600mb via `apt`
wget -O /tmp/netdata-kickstart.sh https://get.netdata.cloud/kickstart.sh && sh /tmp/netdata-kickstart.sh --stable-channel --disable-telemetry
```

## config

```
cd /etc/netdata
# edit / create the config
./edit-config stream.conf
```

```
[stream]
    enabled = yes
    destination = 192.168.0.53:19999
    # The API_KEY of the ohst, to use as the sender
    # change to what you generated above
    # no quotation marks required
    api key = f7da0e10-a374-4ed9-8ec7-8d2a2537c2f6
```

# on both

```
systemctl restart netdata
```

# test

* go to `http://<host ip>:19999`
* click the tiny little "skip this" on the bottom or somewhere to remove the large "connect to our cloud" screen
* done

# host restriction

* create lxc
* install this proxy:

```
curl -fsSL https://deno.land/install.sh | sh
# Neustart + SSH neu rein
git clone https://github.com/unixfox/netdata-unlock-5-nodes
cd netdata-unlock-5-nodes
echo "NETDATA_BASE_URL=http://<netdata host>:19999" > .env
deno task run
```

* go back to `http://<netdata host>:19999`
* select 1-5 nodes
* hit save

* go back to `http://<netdata proxy>:8000`
* enjoy

see also [the project](https://github.com/unixfox/netdata-unlock-5-nodes) and 
this [issue](https://github.com/unixfox/netdata-unlock-5-nodes/issues/3)
and this [related issue](https://github.com/unixfox/netdata-unlock-5-nodes/issues/1)
