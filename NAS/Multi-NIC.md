
# Extra peer to peer link. 

Normal home network setup is everything plug into a switch on LAN. which have DHCP, mDNS, etc.

To improve speed, could add a highspeed p2p connection (like fiber) between NAS and desktop, no router or switch to cut cost.

## Design for network.

Since the two link's topology is different, can't use link aggration on layer 2. 

There is the option for layer 3, generally called equal cost multipath. However, most layer 3 protocal will not do live packet swtiching as it will cause packet re-ordering, which is bad for TCP. So this style will result in OS. So most likely the setup will be make the cost lower for high speed link, so OS prefer the high speed link more.

The option at layer 4 is multi path TCP. This allows tcp to actually use two network path at the same time.


## Some sysctl params 

These params help the setup for mptcp or ECMP

Prevent responding to arp for an address on wrong interface
`net.ipv4.conf.all.arp_ignore=1`

Use the best ip address on the ARP request. Prevent lan address advertised onto fiber network
`net.ipv4.conf.all.arp_announce=2`

Decides the hash keys used for Equal-Cost Multi-Path (ECMP) selection. 0 is only using ip address, 1 is includ tcp udp port number into it.
This help spread the connections.
`net.ipv4.fib_multipath_hash_policy=1`


This cause skipping dead next hop faster. in the case of the high speed connection failed.
`net.ipv4.fib_multipath_use_neigh=1` 


Turn on mptcp 
`net.mptcp.enabled=1` 