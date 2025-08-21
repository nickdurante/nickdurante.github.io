---
title: "Making Synchting run on GL.iNet Velica -or- The Bane of my existance"
categories:
  - Linux
  - GL.iNet
  - OpenWrt
tags:
  - tech
comments: true
---

This task has been way more difficult than expected.
First, I grabbed the executable as described [here](https://github.com/brglng/syncthing-openwrt?tab=readme-ov-file).

```bash
$ curl -k https://github.com/syncthing/syncthing/releases/download/v0.11.9/syncthing-linux-arm-v0.11.9.tar.gz
$ tar -zxf syncthing-linux-arm-v0.11.9.tar.gz 
$ cd syncthing-linux-arm-v0.11.9
$ cp syncthing /usr/bin/
```

Easy.

Following the instructions I was able to run the executable but the service could not be enabled. 
Maybe my version of GL.iNet + OpenWRT does not support services enabling? Who knows.

Looking for an alternative I've added the line to crontab (`crontab -e`):
``` bash
@reboot sleep 15 && /usr/bin/syncthing -home /root/.local/state/syncthing >> /root/log.txt 2>&1
```

To my surprise the crontab is erased at every boot (damn overlays).

Adding the line to `/etc/rc.local` started the service but with wrong configuration file and wrong permissions.
However, adding few seconds of sleep to allow volumes to mount (maybe) worked:
``` bash
(sleep 15; /usr/bin/syncthing -home /root/.local/state/syncthing > /root/log.txt 2>&1) &
```

Now the process was starting at every boot, but the folder permissions were wrong (why?).
To fix this here is the workaround:

1. Start with no folders other than the default one.
2. Add a local folder on the router, specifying:
  * The name of the folder as you whish
  * The id, using the same as the folder id in the cluster
  * The path, using the full path without tildes.
3. Set it to receive only
4. Now share that folder with the other nodes that have it in the cluster
5. The router receives the data from the cluster



