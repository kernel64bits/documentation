

# vrf

ip link add vrf400 type vrf table 400
ip link set dev vrf400 up
ip link set bridge400 master vrf400

https://docs.kernel.org/networking/vrf.html
https://www.kernel.org/doc/html/v5.9/networking/vrf.html

Plus de détails sur comment l'utiliser
https://interpip.es/linux/creating-a-vrf-and-running-services-inside-it-on-linux/

