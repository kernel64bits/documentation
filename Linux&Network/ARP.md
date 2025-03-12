

# ARP

**Address Resolution Protocol** (ARP) is a protocol or procedure that connects an ever-changing Internet Protocol (IP) address to a fixed physical machine address, also known as a media access control (MAC) address, in a local-area network (LAN).

Usually an ARP cache is used on host, with an expiration of the value and other settings of the protocol that can be changed via the kernel configuration. See https://stackoverflow.com/questions/37341598/arp-timeout-in-broadcast-message

Some of this settings can be found here https://man7.org/linux/man-pages/man7/arp.7.html.

They can be changed by editing the files inside the directory `/proc/sys/net/ipv4/neigh/<interface_name>`

You can display the state of the arp cache by using the command `ip neigh`.
For each IP you will also see the corresponding mac address, network interface, and state of the cache line.

The ARP process start by sending an ARP Request as a broadcast frame to the entire network. Due to the topology of our network, **all** ARP request are broadcasted on the same network.

This create a lot of congestion, and in some cases step of the FSM above timeout resulting on an error. This can happen when you try requesting a lot of MAC at the same time.