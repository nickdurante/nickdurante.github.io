---
title: "Set up a RPi seedbox and media server"
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
I based my configuration on the [Transmission Arch wiki](https://wiki.archlinux.org/index.php/Transmission), for more configurations head there.

# Versions of software used
My Raspberry Pi 4 is runs Raspbian GNU/Linux 10 (buster)

| Software | Version|
|:------|------:|
| ```OS``` | ```Raspbian GNU/Linux 10 (buster)``` |
| ```nordvpn``` | ```3.8.6``` |
| ```transmission-daemon``` | ```2.94-2+deb10u1``` |
| ```minidlna``` | ```1.2.1+dfsg-1+b1``` |



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

Moreover you won't be able to stream content on your local network.

If you are not using NordVPN you can configure your iptables not to use the VPN on the local network, [see here](https://openvpn.net/community-resources/how-to/#redirect).


Edit ```.config/transmission-daemon/settings.json``` to your liking, it is important to enable RCP and add your IP to the RCP whitelist.
To allow access only from your subnet set RCP whitelist to something like:

```bash
"rpc-whitelist": "127.0.0.1,192.168.1.*"
```


# Manage privileges

Separating privileges and groups is an important security feature.
Create a new group to access your HDD download location and a user for transmission:

```bash
groupadd torrents
useradd --create-home --groups torrents transmission
```
Change the transmission service to run with dedicated user:

```bash
# cat /lib/systemd/system/transmission-daemon.service
[Unit]
Description=Transmission BitTorrent Daemon
After=network.target

[Service]
User=transmission
#User=debian-transmission
Type=notify
ExecStart=/usr/bin/transmission-daemon -f --log-error
ExecStop=/bin/kill -s STOP $MAINPID
ExecReload=/bin/kill -s HUP $MAINPID
[Install]
WantedBy=multi-user.target   

```

Then restart the Transmission daemon.
```bash
systemctl restart transmission-daemon.service
```

You now can reach the RCP interface on your local network with a browser (at http://RPi_IP:9091/transmission/web/) or an app such as [Tremotesf](https://f-droid.org/en/packages/org.equeim.tremotesf/) with the same IP.

On the interface or on the ```.config/transmission-daemon/settings.json``` set the download location of your torrents on your HDD, mine is mounted on ```/media/hdd/```.

```bash
"download-dir": "/media/hdd/torrents/complete",
"incomplete-dir": "/media/hdd/torrents/incomplete",   
```

# Stream torrents on your local network
The UNIX tool ```minidlna``` ([Arch wiki](https://wiki.archlinux.org/index.php/ReadyMedia)) allows you to broadcast on your local network (i.e. smart TV, tablets (VLC) and other PCs) the media content you downloaded.
Install minidlna and assign right permissions to the HDD folder (in my case mounted on ```/media/hdd/```) and start it.

```bash
apt install minidlna
usermod -aG torrents minidlna
cd /media/
chown -R pi:torrents hdd/
chmod -R g+rwx hdd/
```

Configure minidlna modifying ```/etc/minidlna.conf```.

```bash
user=minidlna
media_dir=/media/hdd/torrents/complete
merge_media_dirs=yes
inotify=yes
network_interface=eth0
friendly_name=your preferred name
```
Then start ```minidlna``` and you are ready to go:

```bash
systemctl enable --now minidlna
```

# Nice to have tools

* Using the tTorrent search ([F-Droid](https://f-droid.org/en/packages/hu.tagsoft.ttorrent.search/)) app you can search and add torrents to the seedbox through Tremotesf ([F-Droid](https://f-droid.org/en/packages/org.equeim.tremotesf/)).
* Port forwarding (not covered here) allows you to access the seedbox controls remotely (**set a strong authentication method!**)
