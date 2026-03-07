# LXC with NFS

## set up NFS in QNAP

I only have QNAP, so ...

* http://QNAP
* (Enable NFS, first of all)
* User
* Shared Folder
* click Action "Change permissions for shared folder"
* go to your respective share
* in the top drop down menu, select "nfs host access"
* check "Access permission"
  * Host: *
  * Permission: read/write
  * Squash: no root squash

## Configure Proxmox

* Mount NFS in DataCenter / Storage
  * ID = name 
  * Server = IP 
  * export = share name
  * content: snippets
* shut down LXC
* edit LXC config file in proxmox host like this:
```
mp0: /mnt/pve/<nmae>,mp=<location in LXC>,mountoptions=noatime,replicate=0
```
* boot LXC
* look inside <location in LXC>
* try `touch <file>` and `rm <file>`

Although you cannot mount a subfolder in NFS, you can probably mount the subfolder *inside* the LXC, so eg. `mp0: /mnt/pve/<nmae>/<some subfolder>,mp=<location in LXC>,mountoptions=noatime,replicate=0`

## useful resources

* https://forum.proxmox.com/threads/tutorial-mounting-nfs-share-to-an-unprivileged-lxc.138506/
* https://www.simplehomelab.com/udms-part-9-bind-mount-remote-storage-in-proxmox-lxc-containers/ (ignore the stuff with IDs, didnt matter for me in PVE 9)


# PBS Backup Trace

