# Virtual Network Setup Guide for Proxmox

This guide will walk you through the process of setting up a Linux bridge and configuring network settings using Proxmox WebUI.

## Step 1: Create a Linux Bridge and Assign IP using Proxmox WebUI

Create a Linux bridge with the name `vmbr1` and assign it the IP address `10.0.0.1/24`.

## Step 2: Edit /etc/network/interfaces

Edit the `/etc/network/interfaces` file and add the following lines under your virtual bridge `vmbr1`. Make sure to adjust the values according to your specific configuration:

```bash
post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
post-up   iptables -t nat -A POSTROUTING -s '10.0.0.0/24' -o vmbr0 -j MASQUERADE
post-down iptables -t nat -D POSTROUTING -s '10.0.0.0/24' -o vmbr0 -j MASQUERADE
post-up   iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone 1
```

The entire interface configuration should look like this:

```bash
auto lo
iface lo inet loopback

iface enp30s0 inet manual

auto vmbr0
iface vmbr0 inet static
        address 192.168.29.5/24
        gateway 192.168.29.1
        bridge-ports enp30s0
        bridge-stp off
        bridge-fd 0
auto vmbr1
iface vmbr1 inet static
        address 10.0.0.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
        post-up   echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up   iptables -t nat -A POSTROUTING -s '10.0.0.0/24' -o vmbr0 -j MASQUERADE
        post-down iptables -t nat -D POSTROUTING -s '10.0.0.0/24' -o vmbr0 -j MASQUERADE
        post-up   iptables -t raw -I PREROUTING -i fwbr+ -j CT --zone 1
        post-down iptables -t raw -D PREROUTING -i fwbr+ -j CT --zone 1
```
## Step 3: Enable IP Forwarding
Edit the file /etc/sysctl.conf and add or uncomment the following line to enable IP forwarding:

```bash
net.ipv4.ip_forward=1
```

Then, run the following command to apply the changes:

```bash
sysctl -p
Step 4: Save the IPTables Configuration
Save the IPTables configuration by running the following commands:
```

```bash
iptables -t nat -A POSTROUTING -o vmbr0 -j MASQUERADE
iptables-save | tee /etc/iptables/rules.v4
```

## Step 5: Reboot
Reboot your system to apply the changes.
That's it! Your Linux bridge setup is now complete.
