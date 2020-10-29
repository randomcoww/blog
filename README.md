# Homelab hacks

## HA gateway hacks

There are many examples of how to set up two WAN gateways in high availability failover using VRRP or CARP. The *proper* way to do this requires three static public IPs: One each for your routers, and another floating IP which will failover between your routers to always reside on your active node. This is well know, and well supported, but not really feasible for most homelab envrionmens. Most of us have one DHCP allocated IP from our ISP to access the internet.

This setup attempts to create a *good enough* dual router failover setup between using only one dynamic public IP. It is not perfect. I'm aware issues like some VPN connections dropping on failover and becoming inaccessible unless I reconnect manually. Things like youtube and web browsing for the most part are not affected and work perfectly though.

Here are two main configurations we need to work out:
- Both gateways need to acquire the same dynamic IP from ISP.
- Only one gateway can be used at a time for any traffic that hits the internet.

### Both gateways need to acquire the same dynamic IP from ISP

Firstly, we want modem to split off and feed the WAN NIC of both routers. You can feed the line from your modem to a switch and connect it to both of your routers. A VLAN is convenient for this.

The second step is commit a cardinal sin of networking and duplicate the MAC address on the WAN interface of both routers. Does this break things? It actually seems to be okay as long as I only use one node to send traffic. I at least haven't seen issues on my home network, but would not be something I would do at work without consulting a real network engineer.

Now both routers can request DHCP and will get the same address. Doesn't this violate only sending traffic out of one node though? Well, yes but this is the only time as far as I can tell, and does not seem to cause any noticeable issues. Perhaps it is small and infrequent enough to not be noticeable in a home environment?



Here is a simple systemd-networkd configuration for this interface

```
[Link]
RequiredForOnline=false

[DHCP]
UseDNS=false
UseNTP=false
SendHostname=false
UseHostname=false
UseDomains=false
UseTimezone=false
RouteTable=${master_default_route_table}
```


## Only one gateway can be used at a time for any traffic that hits the internet

Now to only allow traffic out of one router at a given time, we can take advantage of Keepalived and tansition scripts.

## Multicast DNS hacks
