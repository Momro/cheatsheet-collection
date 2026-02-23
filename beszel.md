# host

tbd.

# client

go to your beszel host web site, click on add system on top right corner

```
sudo su
apt install curl

curl -sL https://get.beszel.dev -o /tmp/install-agent.sh && chmod +x /tmp/install-agent.sh && /tmp/install-agent.sh -p 45876 -k "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMlut7fsy/cBfqY2bLr8No44af3kOGu43LPC5jv+q/YV" -t "8144d0e9-5b78-44dd-8f46-2e390cdda4b1" -url "https://monitoring.pfister.tech"
