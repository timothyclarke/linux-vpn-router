# Sites
This is for the policy router in on each site.

Software required
* `iproute2`
* `openvpn` client package if they are split
* `netplan` I put this on a raspberry pi and it was using NetworkManager with netplan. I disabled NetworkManager for `eth0` as it feeds into the rest
* `iptables` iptables are primary used for road-warrior type setups


As a high level overview
* Set sysctl to forward network interfaces via `/etc/sysctl.d`
* Set route tables in `/etc/iproute2/rt_tables` to define the route tables
* Ethernet side of the networking in `/etc/netplan`
* vpn side of the networking in `/etc/openvpn`
  * Files in `/etc/openvpn` are copied from common. Ones under this part of the tree are placeholders
  * Files `/etc/openvpn/client` `*.conf` are openvpn configs
  * Files `/etc/openvpn/client` `*.up` are scripts run when the VPN establisges. They essentially add a default route to the correct table. They are essentially
    `/usr/sbin/ip ro add default via $5 dev vpn-exitA table ExitA` The 5th arg that openvpn passes to the script is the peer ip

Once everything is in place
* `nmcli device set eth0 managed false` to disable NetworkManager management of eth0
* `systemctl restart systemd-sysctl` to set sysctl values
* `netplan try` confirm you still have access then `netplan apply`
* `systemctl enable openvpn-client@ExitA` to enable the vpn for ExitA (repeat for the rest)
* reboot and test. The reboot is simply to ensure everything starts correctly, eg eth0, ip forwarding etc


For a road warrior config we would need to add additional ip rules
`usr/sbin/ip rule add fwmark 10 lookup ExitA` (add 11 & ExitB etc)
`iptables -t mangle -A OUTPUT -s 192.168.12.0/27 -j MARK --set-mark 10`
`iptables -t nat -A POSTROUTING -o vpn-ExitA -m mark --mark 10 -j MASQUERADE`
