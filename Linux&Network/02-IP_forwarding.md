

# 02-IP forwarding

## Forwarding IP
Il est nécessaire d'activer le forwarding IP pour qu'un serveur Linux soit capable de capable de retransmettre (forward) des paquets qu'il reçoit mais ne lui sont pas destinés. Sinon ces paquets sont tout simplement ignorés.

- Vérifier le statut: `cat /proc/sys/net/ipv4/ip_forward` ou `sysctl net.ipv4.ip_forward`
- Activer le forwarding de manière non persistante:  `echo 1 > /proc/sys/net/ipv4/ip_forward` ou `sysctl -w net.ipv4.ip_forward=1`
- Activer le forwarding de manière persistante:
	- Ajouter la ligne `net.ipv4.ip_forward = 1` dans le fichier */etc/sysctl.conf*
	- Appliquer les changements: `sysctl -p`

**Note:** Le principe est le même pour l'IPv6 (net.ipv6.ip_forward).