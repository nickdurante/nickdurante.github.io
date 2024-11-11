---
title: "Fixing AdGuard on GL.iNet Router with VPN DNS Configuration"
categories:
  - Development
  - Privacy
tags:
  - development
  - tech
comments: true

gallery:
  - url: https://raw.githubusercontent.com/nickdurante/nickdurante.github.io/a1394d82d3fe7a18e0e6789dc19f61581c9aa3b9/assets/images/port_forward.jpg
    image_path: https://raw.githubusercontent.com/nickdurante/nickdurante.github.io/a1394d82d3fe7a18e0e6789dc19f61581c9aa3b9/assets/images/port_forward.jpg
    alt: "Port forward"
    title: "Port forward"
---
# Fixing AdGuard on GL.iNet Router with VPN DNS Configuration

If you're using AdGuardHome with a GL.iNet router on a VPN setup, follow these steps to configure AdGuard to work smoothly with VPN-based DNS.

## Show clients with their identifier instead of "localhost"
Set the following port forwarding rule in OpenWrt:

{% include gallery caption="Port forward" %}

## Step-by-Step Instructions

### 1. Configure DNS in VPN Server and Client Configuration

1. **SSH into the Router**
   - Use SSH to access the router (GL.iNet) by connecting to `192.168.8.1`. If `ssh` doesn’t connect directly, use `dbclient`:
     ```bash
     dbclient root@192.168.8.1
     ```

2. **Edit the OpenVPN Server Configuration**
   - Open the OpenVPN server configuration file to add the DNS option for the VPN:
     ```bash
     vi /etc/openvpn/ovpn/server.ovpn
     ```
   - Add the following line to push the VPN DNS to clients:
     ```bash
     push "dhcp-option DNS 10.8.0.1"
     ```
   - Here, `10.8.0.1` represents the VPN network IP address, which serves as the DNS.

3. **Edit the Client Configuration**
   - In the VPN client configuration (`client.ovpn`), ensure this DNS setting is also specified:
     ```bash
     dhcp-option DNS 10.8.0.1
     ```

### 2. Modify AdGuard for Standard Login Access

To access AdGuard with a standard login (rather than GL.iNet’s default setup), update the AdGuardHome configuration.

1. **Edit AdGuardHome Startup Script**
   - Open the AdGuardHome startup script:
     ```bash
     vim /etc/init.d/AdGuardHome
     ```
   - Remove `--glinet` from the command line in this file:
     ```bash
     procd_set_param command /usr/bin/AdGuardHome -c /etc/AdGuardHome/config.yaml -w /etc/AdGuardHome -l syslog
     ```

2. **Set Up a Username and Password**
   - To configure a login for AdGuard, use the following command to create a username and password:
     ```bash
     htpasswd -B -C 10 -n -b <USERNAME> <PASSWORD>
     ```
   - This command generates a hash that will look like:
     ```plaintext
     <USERNAME>:<HASH>
     ```

3. **Edit the AdGuard Configuration File**
   - Open the AdGuard configuration file:
     ```bash
     vim /etc/AdGuardHome/config.yml
     ```
   - Add the user credentials under the `users` section:
     ```yaml
     users:
       - name: nick
         password: <HASH>
     ```

By following these steps, you can ensure that AdGuard operates effectively on a GL.iNet router using VPN DNS and provides secure login access. This setup allows AdGuard to handle DNS queries via the VPN network and secures access with a username and password.
