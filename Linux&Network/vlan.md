

# vlan

# VLAN

## Native VLAN vs default VLAN

In _**Default VLAN**_ all switch ports turn into a member of the default VLAN just after the initial bootup of the switch. All these ports engage in the default VLAN which makes all of them a part of the same broadcast domain.  Because of this, any device connected to any port can communicate with other devices on other switch ports. For Cisco and some other vendors, VLAN1 is the default VLAN.

A _**native VLAN**_ is a special VLAN whose traffic traverses on the 802.1Q trunk without any VLAN tag. A native VLAN is defined in 802.1Q (it supports untagged traffic while inter-switch link doesn’t support untagged traffic.) trunk port standard which supports traffic coming from several VLANs as well as the traffic that doesn’t come from a VLAN. The native VLAN is per trunk per switch configuration. The 802.1Q trunk port assigns untagged traffic on a native VLAN.  That is, the native VLAN detects and identifies traffic coming from each end of a trunk link. By default, the native VLAN is VLAN 1, but it can be changed to any number such as VLAN 10, VLAN 20, VLAN 99, etc. The native VLAN is also useful when we deal with VoIP.

https://www.geeksforgeeks.org/difference-between-default-vlan-and-native-vlan/