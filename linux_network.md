Most content are learned from this blog post:

https://blog.packagecloud.io/monitoring-tuning-linux-networking-stack-receiving-data/

===
===

## A word on NIC to CPU
For most modern network card, when data come in from NIC, it use DMA to place the data on a ring buffer. When this buffer is filled, data is dropped.

Sections below are in incremental order of Receive stack from NIC to CPU.

## Check for network statistic

`ethtool -S <NIC>` can check on statistic of a NIC device. However there are no standard on what the output is. 

sysfs can give a lot of statistic but at higher level. Check each file for different count. 

`/sys/class/net/<eth_tool>/statistics/<file>`
 
It is still up to the NIC to decide which is which. Best check the driver source code to make sure of. 

`/proc/net/dev` is another file for getting such info at high level. It will give info on all NIC as a subset as info from above. But still each driver decide what is what.


## RRS, Multiqueue
Some NIC support Receive Side Scaling (RSS) or multiqueue. Which allow NIC to place packet on multiple queue (independent memory region) and multiple CPU and process it together. ethtool can be used to tune this.

### Check the RSS setting of a NIC

`ethtool -l <nic>` to check for RSS ability and setting

Having error on this command could mean RSS is not supported, RSS queue adjusting is not supported, or driver is not updated to support it. 
```
$ sudo ethtool -l eth0
Channel parameters for eth0:
Cannot get device channel parameters
: Operation not supported
```

### Change the setting of RSS of a NIC.

`ethtool -L` to set NIC RSS queue size.

Set combined NIC transmit and receive queues to 8: `sudo ethtool -L <NIC> combined 8`

If driver support setting individual RX channel, similar option could set the RX queue to 8 `$ sudo ethtool -L <NIC> rx 8`

**Changing these setting could take the interface down then up.**

## Change the size of RX queue

ethtool provide a generic way to adjust RX queue size. Increase the RX queue can help prevent data drop at NIC (which is still possible to be dropped in software). 

`ethtool -g <NIC>` to check the queue size.

`ethtool -G <NIC> rx <size>` to change the queue size

## Adjust process weight of RX queue

when NIC support flow indirection and register some ethtool functions, user can change the queue priority among multiple RSS queues.

## ntuple filtering for flow steering

Some NIC support "ntuple filtering". The name vary base on each brand's marketing term. Intel use "Intel Ethernet Flow Director". 

With this, User can specify a set of parameter to filter incoming network data in hardware and place it in a particular RX queue. This help in the way that it increase CPU cache hit by keeping data locally (to the same CPU)

This is a crucial component of Accelerated Receive Flow Steering (aRFS).

=====

## Data arrived at memory

At this point, data is placed int RAM via DMA (give queue is not filled), and the IRQ is raised.

=====

## Side note on IRQ and Soft IRQ

SoftIRQs is a mechanism in Linux's kernel to defer code execution from interrupt handler (IRQ) to outside. Any interrupt happened when handling a hardware interrupt is dropped. Thus it is important to keep the IRQ as short as possible and re-enable interrupt as fast as possible.

**The softirq handler will execute on the same CPU as driver's IRQ handler.**


## Interrupt coalescing

Prevent interrupts be raised by a device until specific amount of work or event are pending.

Could increase throughput at the cost of latency. 

Can use ethtool to set these, but anything not supported by driver or NIC will be silently ignored. 

"adaptive RX/TX IRQ coalescing" is usually a hardware feature where it dynamically adjust IRQ when packet rate is low or high, and keep latency low at low packet rate, while keep throughput hight at high packet rate. 

## Change IRQ affinities

For NIC supports RSS/multiqueue, setting specific CPU to handle interrupts from which NIC IRQ can be beneficial.

`irqbalance` daemon is there to automatically balance IRQs to CPUs. It may overwrite your setting. Might want to turn it off or use `--banirq` with `IRQBALANCE_BANNED_CPUS` to isolate some IRQs and CPUs.

check `/proc/irq/<IRQ_NUMBER>/smp_affinity` for IRQ number, and write a bit mask to this file to set the CPU. 

Example: Set the IRQ affinity for IRQ 8 to CPU 0
```
$ sudo bash -c 'echo 1 > /proc/irq/8/smp_affinity'
```

## Network data process loop `net_rx_action`

A softirq is pending when the data is ready to be processed. 

The processing loop `net_rx_action` dequeue every NAPI structure being DMA'd and operate on it. 

This processing loop is **bounded by amount of work and execution time**. Both `budget` and elapsed time are check. The kernel prevent packet processing from lasting too long. The `budget` is the total amount spent on each of the NAPI structure.

In case of multiqueue NIC or multiple NIC, when their IRQ affinity are on the same core, their softirq handler will be on the same core, and the budget is shared among them. 

When not enough CPUs are available to distribute IRQs, increasing the `net_rx_action budget` can allow more packet processing at the cost of higher CPU usage (`sitime` / `si` in `top`). However, this could **reduce latency**. Regardless of budget, the CPU is still bounded by 2 jiffies time.

In `net/core/dev.c`, `time_squeeze` field measures the number of time `net_rx_action` didn't complete the work because budget or time limit is reached.

at the `out` label, `net_rps_action_and_irq_enable` is called. This wakes up remote CPUs to start process network data (adn affected by Receive Packet Steering). 

## Monitor the health of `net_rx_action`

Statistic is tracked as part of `struct softnet_data` with CPU, and can be accessed at `/proc/net/softnet_stat`. The documentation on this is poor, the filed and meaning are specific to kernel releases. Best place to look for is the kernel source code `/net/core/net-procfs.c` 

some important details from the website about /proc/net/softnet_stat: 

    * Each line of /proc/net/softnet_stat corresponds to a struct softnet_data structure, of which there is 1 per CPU.
    * The values are separated by a single space and are displayed in hexadecimal
    * The first value, sd->processed, is the number of network frames processed. This can be more than the total number of network frames received if you are using ethernet bonding. There are cases where the ethernet bonding driver will trigger network data to be re-processed, which would increment the sd->processed count more than once for the same packet.
    * The second value, sd->dropped, is the number of network frames dropped because there was no room on the processing queue. More on this later.
    * The third value, sd->time_squeeze, is (as we saw) the number of times the net_rx_action loop terminated because the budget was consumed or the time limit was reached, but more work could have been. Increasing the budget as explained earlier can help reduce this.
    * The next 5 values are always 0.
    * The ninth value, sd->cpu_collision, is a count of the number of times a collision occurred when trying to obtain a device lock when transmitting packets. This article is about receive, so this statistic will not be seen below.
    * The tenth value, sd->received_rps, is a count of the number of times this CPU has been woken up to process packets via an Inter-processor Interrupt
    * The last value, flow_limit_count, is a count of the number of times the flow limit has been reached. Flow limiting is an optional Receive Packet Steering feature that will be examined shortly.

These are as of Linux 3.13.0. Before using the data here, needs to check the content and meaning have not changed. 

## Changing the budget

change the budget to 600 (default in Linux 3.13.0 is 300)
`sudo sysctl -w net.core.netdev_budget=600`

## Generic Receive Offloading (GRO)

GRO is a software implementation of a hardware optimization: Large Receive Offloading (LRO)

It combines "similar enough" packet into one packet with huge payload to reduce protocol layer header processing. For example when receiving a large file.

*When using tcpdump, if a unrealistically large sized packet is received, GRO is likely to be enabled* 

For reference in following text, `netif_receive_skb` is called when GRO is done, and it will decide on handling data off to protocol layers (and RPS)

## Receive Packet Steering (RPS)

RSS: Receive side scaling, where NIC have multiple queues at hardware level and multiple CPU handles interrupts and process packet.

RPS is a software implementation of RSS, and can be used on any NIC, even on NIC with only one RX queue. But it can only effect packet harvested from DMA memory region.

The CPU time spent on handling IRQs and NAPI poll stays the same. But the packet process after this is split across CPU.

RPS generates a hash for incoming data to know which CPU should process each packet. The data is enqueued to per-CPU network rx backlog, and a Inter-Process-Interrupt (IPI) is used to notify the target CPU. `/proc/net/softnet_stat` count this in field `received_rps`

## Enable RPS 

A bit mask in each NIC device's each queue sets which cpu to use `/sys/class/net/<DEVICE_NAME>/queues/<QUEUE>/rps_cpus`. Writing a Hex number for CPU into this file will change it.

## Receive Flow Steering (RFS)

This is used in conjunction with RPS to increase data locality (cache hit). It direct packet for the same flow to the same CPU.

RFS need RPS enabled and configured to work. RFS keeps track of a global hash table of all flows. This hash table size can be increased by `sudo sysctl -w net.core.rps_sock_flow_entries=<value>`

Each Rx queue can be specify a number of flows. This can be set by writing the number to ` /sys/class/net/eth0/queues/rx-0/rps_flow_cnt` 

Example: increase the number of flows for RX queue 0 on eth0 to 2048.
```
$ sudo bash -c 'echo 2048 > /sys/class/net/eth0/queues/rx-0/rps_flow_cnt'
```

## Hardware accelerated Receive Flow Steering (aRFS)

This is RFS with the help of NIC and driver. When everything is configured correctly, aRFS will automatically move data to RX queue tie to the same CPU that will process that flow. Also, ntuple filter rule no longer need to be manually set. 

## Reached `netif_receive_skb` 

`netif_receive_skb` is called after GRO is done.

It will first check `sysctl` if user requested receive timestamping before packet hits the backlog queue. If yes, time stamp is done now and before packet is put into RPS. If no, time stamp happen when packet reach backlog (could also distribute CPU load at cost of some delay). This is controlled by `net.core.netdev_tstamp_prequeue`, default to 1

Without RPS: it will move to `__netif_receive_skb_core`

With RPS: `netif_receive_skb` will compute which CPU's backlog to queue this. Then call `enqueue_to_backlog`


## Inside `enqueue_to_backlog` (with RPS or RFS aRFS)

before putting data onto remote CPU's backlog: data is dropped if:
* The remote CPU's `input_pkt_queue` queue length is longer then `netdev_max_backlog`
* The flow rate is exceeded

The remote CPU's `softnet_data` structure will increment drop count when such dropping happens. (note: it's the CPU the data was going to be queued on will increment this count.)

**increasing the backlog will not have any noticeable effect if RPS is not used or driver not using `netif_rx` (which it shouldn't be, it should use `netif_receive_skb` instead)**

The flow limit is to ensure smaller flow is not starved by large flow (which could monopolize the CPU). by default, it is not turned on. 

`dropped` field in `/proc/net/softnet_stat` can check for this drop. 

changing `net.core.netdev_max_backlog ` in sysctl can adjust this backlog size. 

## Reached `__netif_receive_skb_core` (with/without RPS should both reach here)

`__netif_receive_skb_core` first deliver data to packet taps if any is installed. Example: `AF_PACKET` address family, typically used via `libpcap`. 

After this, the packet is delivered to protocol layer. `__netif_receive_skb_core ` iterate through a list of deliver functions registered for each protocol type (which themselves will register themselves before in the list and hash table)

## Entering Protocol layer (IP as example)

IP protocol plugs itself into `ptype_base` so data will be delivered to it. `__netif_receive_skb_core` will call `deliver_skb` which will call `func` (at `net/ipv4/af_inet.c`). In case of IP, `ip_recv` is called.

`ip_recv` pass data to any installed filters ans rules (netfilter, iptable, conntrack), then pass it to `ip_rcv_finish`. These filters and rules inside iptable/netfilter/conntrack are executed within the softirq. So complex rule will cause the latency to grow. It will try not to dive too deep into these filter first, and return execution to Protocol layer first.

if packet is not dropped, it will reach `ip_rcv_finish`. Some routing will happen to find the packet's destination. If destination is local system, `ip_local_deliver` will be called next.

## `ip_local_deliver` to continue

This is similar to `ip_recv`, netfilter will take a look at the data first, if not dropped, `ip_local_deliver_finish` is called.

`ip_local_deliver_finish` get the protocol from packet, and get the correct `net_protocol` structure and call `handler` in it. This hands it to higher protocol layer

## Monitor IP protocol layer statistics.

`/proc/net/snmp` can be read for info. The IP protocol layer is displayed first. The field names can be referenced in `include/uapi/linux/snmp.h`

`/proc/net/netstat` can be used for extended IP statistic, filter for `IpExt` will only show that. `cat /proc/net/netstat | grep IpExt`

A sub list of items that matter

    * InReceives: The total number of IP packets that reached ip_rcv before any data integrity checks.
    * InHdrErrors: Total number of IP packets with corrupted headers. The header was too short, too long, non-existent, had the wrong IP protocol version number, etc.
    * InAddrErrors: Total number of IP packets where the host was unreachable.
    * ForwDatagrams: Total number of IP packets that have been forwarded.
    * InUnknownProtos: Total number of IP packets with unknown or unsupported protocol specified in the header.
    * InDiscards: Total number of IP packets discarded due to memory allocation failure or checksum failure when packets are trimmed.
    * InDelivers: Total number of IP packets successfully delivered to higher protocol layers. Keep in mind that those protocol layers may drop data even if the IP layer does not.
    * InCsumErrors: Total number of IP Packets with checksum errors.


## A UDP packet (as example)

`udp_rcv` is the first entry point, calls into `__udp4_lib_rcv`. This will check UDP datagram info, checksums, etc. 

The receiving socket will be looked up (or if the early optimization in IP layer already put it in). If the socket is found, the packet will be queued to that socket. (the datagram is dropped if nothing is found)

## Reached the socket 

`sk_rcvqueues_full` check if receive queue for the socket is full. The socket backlog length is checked against `sk_rmem_alloc` to determine if sum is greater then `sk_rcvbuf` 

`sk_rcvbuf` for each socket is default to `net.core.rmem_default`. `sk_rcvbuf` can be increased to the limit set at `net.core.rmem_max`. Both max and default can be changed through `sysctl`. 

`sk_rcvbuf` is also set by calling `setsockopt` from application with `SO_RCVBUF`. This is still limited by `net.core.rmem_max`. However, `setsockopt` can use `SO_RCVBUFFORCE` to override this limit, but application need to have `CAP_NET_ADMIN` capability.

## Monitor UDP Protocol Layer 

`/proc/net/snmp` have line for udp similar to lines for IP. 
    
    * InDatagrams: Incremented when recvmsg was used by a userland program to read datagram. Also incremented when a UDP packet is encapsulated and sent back for processing.
    * NoPorts: Incremented when UDP packets arrive destined for a port where no program is listening.
    * InErrors: Incremented in several cases: no memory in the receive queue, when a bad checksum is seen, and if sk_add_backlog fails to add the datagram.
    * OutDatagrams: Incremented when a UDP packet is handed down without error to the IP protocol layer to be sent.
    * RcvbufErrors: Incremented when sock_queue_rcv_skb reports that no memory is available; this happens if sk->sk_rmem_alloc is greater than or equal to sk->sk_rcvbuf.
    * SndbufErrors: Incremented if the IP protocol layer reported an error when trying to send the packet and no error queue has been setup. Also incremented if no send queue space or kernel memory are available.
    * InCsumErrors: Incremented when a UDP checksum failure is detected. Note that in all cases I could find, InCsumErrors is incrememnted at the same time as InErrors. Thus, InErrors - InCsumErros should yield the count of memory related errors on the receive side.

`/proc/net/udp` is specific udp socket statistics.

The data here are more per-socket specific.

```
$ cat /proc/net/udp
  sl  local_address rem_address   st tx_queue rx_queue tr tm->when retrnsmt   uid  timeout inode ref pointer drops
  515: 00000000:B346 00000000:0000 07 00000000:00000000 00:00000000 00000000   104        0 7518 2 0000000000000000 0
  558: 00000000:0371 00000000:0000 07 00000000:00000000 00:00000000 00000000     0        0 7408 2 0000000000000000 0
  588: 0100007F:038F 00000000:0000 07 00000000:00000000 00:00000000 00000000     0        0 7511 2 0000000000000000 0
  769: 00000000:0044 00000000:0000 07 00000000:00000000 00:00000000 00000000     0        0 7673 2 0000000000000000 0
  812: 00000000:006F 00000000:0000 07 00000000:00000000 00:00000000 00000000     0        0 7407 2 0000000000000000 0
```

    * sl: Kernel hash slot for the socket
    * local_address: Hexadecimal local address of the socket and port number, separated by :.
    * rem_address: Hexadecimal remote address of the socket and port number, separated by :.
    * st: The state of the socket. Oddly enough, the UDP protocol layer seems to use some TCP socket states. In the example above, 7 is TCP_CLOSE.
    * tx_queue: The amount of memory allocated in the kernel for outgoing UDP datagrams.
    * rx_queue: The amount of memory allocated in the kernel for incoming UDP datagrams.
    * tr, tm->when, retrnsmt: These fields are unused by the UDP protocol layer.
    * uid: The effective user id of the user who created this socket.
    * timeout: Unused by the UDP protocol layer.
    * inode: The inode number corresponding to this socket. You can use this to help you determine which user process has this socket open. Check /proc/[pid]/fd, which will contain symlinks to socket[:inode].
    * ref: The current reference count for the socket.
    * pointer: The memory address in the kernel of the struct sock.
    * drops: The number of datagram drops associated with this socket. Note that this does not include any drops related to sending datagrams (on corked UDP sockets or otherwise); this is only incremented in receive paths as of the kernel version examined by this blog post.

The code which outputs this can be found in `net/ipv4/udp.c`