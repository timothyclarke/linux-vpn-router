# Exits
This is a simple nat gateway and VPN server.

Software required
* `iproute2`
* `openvpn` server package if they are split
* `iptables` iptables are primary used for road-warrior type setups
* `iptables-persistent` to save and restore iptables

As a high level overview
* Set sysctl to forward network interfaces via `/etc/sysctl.d`
* vpn side of the networking in `/etc/openvpn`
  * Files in `/etc/openvpn` are copied from common. Ones under this part of the tree are placeholders
  * Files `/etc/openvpn/server` `*.conf` are openvpn configs
  * Files `/etc/openvpn/server` `Exit#-ccd` `*` Files in the ccd directory define routes back to the ip ranges of each site
    * The files here match the certificate name
      * eg `iroute 192.168.12.0 255.255.255.224` for ExitA -> Site-A / SSID-A.
    * This is not needed for a road warrior config which NAT's at the client end
* `/etc/iptables/rules.v4` Sets IP Masqurade (nat) rules
