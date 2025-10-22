# linux-vpn-router
This is a linux route table example

This may also be considered policy routing.  It's sole purpose was as an experiment from what I was doing around 2000
The idea is multiple exit gateways.
A simple discription might be I want to watch different streaming services in different countries. So we get a VPS in each country, add openvpn and set an iptables rule.
Locally we have 802.1Q tagged vlans with a raspberry pi or similar acting as the router, finally some wifi access points with a SSID per country


* The [common](common) folder will contain elements which are common to both the gateways and the central policy router
* The [gateway](gateway) folder will contain the config for the remote gateways (only one example) some of this will be generic
* The [centralRouter](centralRouter) folder will contain the config for the linux box (raspberry pi) at the centre

In terms of IP ranges I'm using RFC1918 IP's from the `172.16.0.0/12` and `192.168.0.0/16` ranges. As this is a small example `/27`'s with 32 IP's (generally considered 30 usable) is more than sufficent. A `/27` has a Netmask of `255.255.255.224`

Assuming multiple links the table might look as follows. Note that I'm trying to align the ranges to make the explination simple
|Branch|VPN Range|Client Range|First IP|Last IP|
|-|-|-|-|-|
|A|172.16.12.0/27|192.168.12.0/27|1|30|
|B|172.16.12.32/27|192.168.12.32/27|33|62|
|C|172.16.12.64/27|192.168.12.64/27|65|94|
|D|172.16.12.96/27|192.168.12.96/27|97|126|
|A|172.16.12.128/27|192.168.12.128/27|129|158|
|B|172.16.12.160/27|192.168.12.160/27|161|190|
|C|172.16.12.192/27|192.168.12.192/27|193|222|
|D|172.16.12.224/27|192.168.12.224/27|225|254|

