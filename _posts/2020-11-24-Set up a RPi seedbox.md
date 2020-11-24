---
title: "Set up a RPi Seedbox"
categories:
  - Development
tags:
  - development
  - linux
  - tech
comments: true

---


Raspberry Pis are cheap and extremely useful for computing small tasks.
You can set up one of those with an HDD as a seedbox to download **legal** torrents such as Linux distributions and videos from the Internet Archive (**wink wink**).

A VPN for downloading torrents is always recommended, in my case I use NordVPN.

I thought it might be useful to write this down.
I based my configuration on the Transmission ([Arch wiki](https://wiki.archlinux.org/index.php/Transmission)), for more configurations head there.


# Install software
We need to install the missing software:

```bash
wget -O /tmp/nordvpn.deb https://repo.nordvpn.com/deb/nordvpn/debian/pool/main/nordvpn-release_1.0.0_all.deb
apt install /tmp/nordvpn.deb
apt update
apt install -y nordvpn transmission-cli transmission-daemon
```

# Configure software

```bash
(as root)
systemctl enable --now nordvpnd
(as pi)
nordvpn login
nordvpn whitelist add subnet 192.168.1.0/24
nordvpn set killswitch on
nordvpn set autoconnect on [countrycode] p2p
nordvpn connect --group p2p
```
It is extremely important to set the whitelist on your local subnet, in my case ```192.168.1.0/24```, yours can be different.
If you don't do so, you'll lose connectivity and it will become quite difficult to login with SSH on your Raspberry.

*If you are not using NordVPN you can configure your iptables to accept connections on your SSH port using the default interface and not tun0.*

Edit the transmission service to run as a standard user
```bash
# cat /lib/systemd/system/transmission-daemon.service
[Unit]
Description=Transmission BitTorrent Daemon
After=network.target

[Service]
User=pi
#User=debian-transmission
Type=notify
ExecStart=/usr/bin/transmission-daemon -f --log-error
ExecStop=/bin/kill -s STOP $MAINPID
ExecReload=/bin/kill -s HUP $MAINPID
[Install]
WantedBy=multi-user.target   

```

Start the Transmission daemon

```bash
systemctl daemon-reload
systemctl enable --now transmission-daemon.service
```

edit ```.config/transmission-daemon/settings.json``` to your liking, it is important to enable RCP and add your IP to the RCP whitelist.

Then restart the Transmission daemon.
```bash
systemctl restart transmission-daemon.service
```

You now can reach the RCP interface on your local network with a browser (at http://<RPi IP>:9091/transmission/web/) or an app such as Tremotesf  ([F-Droid](https://f-droid.org/en/packages/org.equeim.tremotesf/)) with the same IP.

On the interface or on the ```.config/transmission-daemon/settings.json``` set the download location of your torrents, an HDD is recommended.

# Nice to have tools

* Using the tTorrent search ([F-Droid](https://f-droid.org/en/packages/hu.tagsoft.ttorrent.search/)) app you can search and add torrents to the seedbox through Tremotesf ([F-Droid](https://f-droid.org/en/packages/org.equeim.tremotesf/)).
* Port forwarding (not covered here) allows you to access the seedbox controls remotely (**set a strong authentication method!**)
* The unix tool ```minidlna``` ([Arch wiki](https://wiki.archlinux.org/index.php/ReadyMedia)) allows you to broadcast on your local network (i.e. smart TV, tablets (VLC) and other PCs) the media content you downloaded.
