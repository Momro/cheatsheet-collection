# host

tbd.

# client

* go to your beszel host web site, go to settings, then on the left side "Tokens and fingerfprints". set universal token to temporary.

* then click on add system on top right corner, add system, insert name/IP and copy then command.
* now you can add severl systems without manually getting a code each time.

login to new client

```
sudo su
apt install curl

TOKEN="some-uuid-you-got-from-system"
URL="https://monitoring.exmple.com"
FINGERPRINT="ssh-ed25519 <insert fingerprint of host here>"

curl -sL https://get.beszel.dev -o /tmp/install-agent.sh && chmod +x /tmp/install-agent.sh && /tmp/install-agent.sh -p 45876 -k "$FINGERPRINT" -t "$TOKEN" -url "$URL"
```

# uninstall

curl -sL https://get.beszel.dev -o /tmp/install-agent.sh && chmod +x /tmp/install-agent.sh && /tmp/install-agent.sh -u
