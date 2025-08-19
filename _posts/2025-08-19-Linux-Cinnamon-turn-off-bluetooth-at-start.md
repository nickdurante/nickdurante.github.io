---
title: "How to keep Bluetooth disabled at startup on Linux"
categories:
  - Linux
tags:
  - tech
comments: true
---

On some Linux distributions, Bluetooth adapters automatically power on at system startup even when you set
`AutoEnable=false` in `/etc/bluetooth/main.conf`. Desktop helpers like Blueberry or the default rfkill
state restore can override that setting.

A reliable way to ensure the adapter remains off is to override the Bluetooth systemd unit and force
a block after the service starts.

Create a systemd override file:

```bash
sudo mkdir -p /etc/systemd/system/bluetooth.service.d
echo -e "[Service]\nExecStartPost=/usr/bin/rfkill block bluetooth" | \
  sudo tee /etc/systemd/system/bluetooth.service.d/override.conf
```
Reload systemd and restart the service:

```bash
sudo systemctl daemon-reload
sudo systemctl restart bluetooth
```
Now the adapter will remain soft-blocked after every boot.
You can still enable it manually at any time with:

```bash
rfkill unblock bluetooth
```

This approach works across Fedora, Ubuntu, and other systemd-based distributions, and avoids desktop tools automatically turning the adapter back on.
