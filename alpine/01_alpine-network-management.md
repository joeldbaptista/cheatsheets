# Alpine Linux Networking Cheatsheet

Alpine uses **BusyBox** (`ifconfig`, `ip` via `iproute2` if installed), **OpenRC** for service management, and its own `/etc/network/interfaces` format (Debian-style) managed through the `ifupdown-ng` (or classic `ifupdown`) package.

---

## 1. Core Packages

```sh
apk add iproute2        # modern ip command (recommended over ifconfig)
apk add ifupdown-ng      # ifup/ifdown + /etc/network/interfaces support
apk add wpa_supplicant   # WiFi
apk add dhcpcd           # or udhcpc (busybox built-in) for DHCP
apk add openresolv       # resolvconf management (optional)
```

Alpine's minimal install only ships **BusyBox applets** (`ifconfig`, `route`, `udhcpc`). Install `iproute2` if you want the full `ip` command suite.

---

## 2. Quick Interface Commands

### Show interfaces
```sh
ip link show          # or: ifconfig -a
ip addr show          # or: ifconfig
```

### Bring an interface up / down
```sh
ip link set eth0 up
ip link set eth0 down

# BusyBox equivalent
ifconfig eth0 up
ifconfig eth0 down
```

### Assign a static IP manually (non-persistent)
```sh
ip addr add 192.168.1.50/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.1.1
```

### Remove an address
```sh
ip addr del 192.168.1.50/24 dev eth0
```

### DHCP on demand (one-shot)
```sh
udhcpc -i eth0          # BusyBox client, minimal install
# or
dhcpcd eth0              # if dhcpcd installed
```

---

## 3. Persistent Config: `/etc/network/interfaces`

This is the file that survives reboots, read by `ifup`/`ifdown` (ifupdown-ng).

```sh
# /etc/network/interfaces

auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

# Static example
auto eth1
iface eth1 inet static
    address 192.168.1.50/24
    gateway 192.168.1.1
    dns-nameservers 1.1.1.1 8.8.8.8
```

Apply changes:
```sh
ifup eth0
ifdown eth0
ifup -a          # bring up all "auto" interfaces
```

`auto <iface>` = start automatically at boot / on `ifup -a`.

---

## 4. OpenRC Service Management

```sh
rc-service networking start
rc-service networking stop
rc-service networking restart
rc-update add networking boot     # enable on boot
rc-update del networking boot     # disable
rc-status                          # see running services
```

---

## 5. DNS

Edit directly (no systemd-resolved on Alpine):
```sh
# /etc/resolv.conf
nameserver 1.1.1.1
nameserver 8.8.8.8
```

If using `dhcpcd`/`udhcpc`, this file is usually overwritten automatically from DHCP-provided DNS. To pin static DNS alongside DHCP IP, use `dns-nameservers` in `/etc/network/interfaces` (ifupdown-ng honors it) or lock the file with `chattr +i /etc/resolv.conf`.

---

## 6. Routing

```sh
ip route show                              # list routes
ip route add default via 192.168.1.1       # set default gateway
ip route del default                       # remove default gateway
ip route add 10.0.0.0/24 via 192.168.1.254 # add a specific route

# BusyBox
route add default gw 192.168.1.1
route del default
```

---

## 7. WiFi (wpa_supplicant)

```sh
apk add wpa_supplicant

# Generate config
wpa_passphrase "MySSID" "MyPassword" > /etc/wpa_supplicant/wpa_supplicant.conf

# Bring up
wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
udhcpc -i wlan0
```

Persistent, in `/etc/network/interfaces`:
```sh
auto wlan0
iface wlan0 inet dhcp
    wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf
```

---

## 8. VLANs

```sh
apk add vlan
modprobe 8021q

ip link add link eth0 name eth0.10 type vlan id 10
ip link set eth0.10 up
ip addr add 192.168.10.5/24 dev eth0.10
```

Persistent:
```sh
auto eth0.10
iface eth0.10 inet static
    address 192.168.10.5/24
    vlan-raw-device eth0
```

---

## 9. Bridges (useful for LXC/QEMU on Alpine)

```sh
apk add bridge-utils   # legacy tool; ip link also works

ip link add br0 type bridge
ip link set eth0 master br0
ip link set br0 up
ip link set eth0 up
ip addr add 192.168.1.1/24 dev br0
```

Persistent:
```sh
auto br0
iface br0 inet static
    address 192.168.1.1/24
    bridge-ports eth0
```

---

## 10. Firewall Quick Reference (iptables / nftables)

```sh
apk add iptables       # or nftables

rc-service iptables save     # persist current rules
rc-update add iptables boot

# minimal example: allow SSH, drop rest on INPUT
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -P INPUT DROP
```

---

## 11. Diagnostics

```sh
ping -c4 1.1.1.1
ip neigh show           # ARP table (like `arp -a`)
ss -tulpn                # listening sockets (iproute2)
netstat -tulpn           # if net-tools installed
traceroute 8.8.8.8       # apk add traceroute
tcpdump -i eth0          # apk add tcpdump
```

---

## 12. Hostname

```sh
hostname                  # show current
echo "myhost" > /etc/hostname
hostname -F /etc/hostname # apply without reboot

# /etc/hosts
127.0.0.1   localhost
127.0.1.1   myhost
```

---

## Common Gotchas

- Alpine minimal images often don't have `iproute2` — you get BusyBox `ifconfig`/`route` only, which lack VLAN/bridge/advanced routing support.
- `ifupdown-ng` (current default) has slightly different syntax/behavior than classic Debian `ifupdown` — check `man interfaces.d` on newer Alpine.
- `/etc/resolv.conf` gets clobbered by DHCP clients unless you take explicit steps to pin it.
- Service is named `networking`, not `network` — a common typo when running `rc-service`/`rc-update`.
