# 04-Netfilter

# Netfilter
Netfilter est (une organisation et) un logiciel de filtrage des paquets embarqué dans le noyau Linux. Il peut être configuré par `iptables` ou son successeur `nftables`.

## Principe
Il existe 5 tables différentes, elles-même consitituées de différentes chaînes:
- **default** (table par défaut):
	- INPUT (paquets à destination des sockets locaux)
	- FORWARD (paquets qui sont routés)
	- OUTPUT (paquets générés localement)
- **nat** (utilisé à chaque fois que l'on rencontre un paquet nécessitant de créer une nouvelle connexion):
	- PREROUTING (pour altérer les paquets à leur entrée)
	- INPUT (pour altérer les paquets destinés à des sockets locaux)
	- OUTPUT (pour altérer des paquets générés localement avant de les router)
	- POSTROUTING (pour altérer les paquets à leur sortie)
- **mangle**
	- https://www.inetdoc.net/guides/iptables-tutorial/mangletable.html
- **raw**
- **security**

Dans chaque chaîne on peut définir des règles.

### Ressources:
- [Packet flow in Netfilter](https://i.stack.imgur.com/68Cvx.png)

# IPtables
Une règle iptable spécifie des critères pour un paquet (IP source/destination, port source/destination, protocole, interface entrée/sortie, ...) une cible (target).
Les règles iptables sont lues une par une pour chaque paquet. Dès que le paquet matche les critères d'une règle, la prochaine règle est spécifiée par la valeur de la target. Cette target peut être:
- une chaîne définie par l'utilisateur
- une target décrie dans les [iptables-extensions](https://man7.org/linux/man-pages/man8/iptables-extensions.8.html)
- une des valeurs suivantes: 
	- **ACCEPT** (on laisse passer le paquet)
	- **DROP** (le paquet est bloqué et détruit)
	- **RETURN** (on arrête de traverser la chaîne actuelle et on passe à la prochaine règle de la chaîne précédente)

## Paramètres iptable
- -S: Liste toutes les règles iptables avec exactement le même format que la commande utilisée pour les ajouter.
- -A (append): ajoute une règle à une table/chaîne
- -D (delete): supprime une règle
- -L (list): Affiche
- -t (table): spécifie le nom de la table. Sinon non spécifié c'est la table `default` qui est utilisée
- -j (jump): permet de spécifier la cible (target) de la règle
- -i (in): nom de l'interface par laquelle le paquet a été reçu (seulement pour paquets entrant dans  les chaînes **INPUT**, **FORWARD** et **PREROUTING**)
- -o (out): nom de l'interface vers laquelle le paquet est envoyé (seulement pour les paquets entrant dans les chaines **FORWARD**, **OUTPUT** et **POSTROUTING**)

**Note**: Pour l'IPv4 on utilise la commande `iptables`, pour l'IPv6 on utilise `ip6tables` qui s'utilise exactement de la même manière.

## Exemples
- Droper tout le traffic entrant: `iptables -A INPUT -j DROP` (le nom de la table n'est pas spécifié, on utilise donc `default`)
- Autoriser les pings: `iptables -A INPUT -p icmp -j ACCEPT`
- Autoriser le routage de tout les paquet entrant par l'interface eth1: `iptables -A FORWARD -i eth1 -j ACCEPT`
- Rediriger tous les paquets tcp à destination du port *80* arrivant sur l'interface *eth0* vers  *172.31.0.23:80*: `iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to 172.31.0.23:80`.  La chaîne **DNAT** permet de changer l'IP de destination. Permet par exemple de rendre une IP privée accessible depuis Internet
- Remplacer l'adresse/port source de tous les paquets tcp avec une IP source dans le range  *10.0.0.0/24*, et sortant par *eth0* par *10.0.1.15:1024*: `iptables -t nat -A POSTROUTING -p tcp -o eth0 -j SNAT --to-source 10.0.1.15:1024`. Il est également possible de définir des ranges pour les nouveaux IP/ports source (`--to-source 194.236.50.155-194.236.50.160:1024-32000`). Dans ce cas dans ce cas la nouvelle IP source sera déterminé aléatoirement mais restera la même pour tout le flux (plus de détails [ici](https://www.inetdoc.net/guides/iptables-tutorial/snattarget.html)).
- La même chose de précédemment avec du masquerade (ici la nouvelle adresse source n'est pas spécifiée, elle sera déterminée dynamiquement en fonction de l'IP portée par l'interface *eth0*. Cf prochain paragraphe): `iptables -t nat -A POSTROUTING -o eth0 -s 10.0.0.0/24 -j MASQUERADE`
- Limiter le traffic sur le port 80 à 100 requêtes par minute: `iptables -A INPUT -p tcp --dport 80 -m limit --limit 100/minute --limit-burst 200 -j ACCEPT`.  On commence avec un compteur de 200 paquets qu'on décrémente à chaque paquet qui passe dans la chaîne (et qui est autorisé), quand le compteur tombe à zéro la règle cesse d'être utilisée (les paquets seront alors probablement dropés par une règle par défaut plus bas dans la liste). Le compteur est incrémenté de 100 par heure de manière régulière (+1 toutes les 1/100 minutes)
- Créé une chaîne pour loguer et dropper les paquets avec le préfix *IP6T:APPENVLDROP*: `iptables -N APPENVLDROP; -A APPENVLDROP -j LOG --log-level debug --log-prefix "IP6T:APPENVLDROP; iptables -A APPENVLDROP -j DROP`
- [Plus d'exemples](https://www.tecmint.com/linux-iptables-firewall-rules-examples-commands/)

## SNAT vs DNAT vs MASQUERADE
Il existe 3 types de targets différentes pour faire du NAT:
- **DNAT** (Destination NAT): Permet de changer la destination d'un paquet (IP et/ou port). On peut s'en servir pour faire un forwarding de port. 
- **SNAT** (Source NAT): Permet de changer la source (IP et/ou port d'un paquet). On peut par exemple s'en servir pour donner un accès Internet depuis une IP privée.
- **MASQUERADE**: Type particulier de SNAT où la nouvelle adresse source (i.e celle du NAT, et pas celle du client) est inconnue au moment où la règle est créée. Il n'y a donc pas besoin de l'attribut `--to-source` puisque la nouvelle adresse source est déterminée dynamiquement. Masquerade va remplacer l'adresse source par l'adresse de l'interface de sortie. Dans le cas où l'adresse source est connue, il vaut mieux utiliser SNAT qui est plus rapide.

## Ressources
- https://www.inetdoc.net/guides/iptables-tutorial/traversingoftables.html
- https://www.netfilter.org/
- https://www.netfilter.org/documentation
- https://wiki.archlinux.org/title/nftables
- https://doc.ubuntu-fr.org/iptables
- https://man7.org/linux/man-pages/man8/iptables.8.html
- https://man7.org/linux/man-pages/man8/iptables-extensions.8.html

## nftables
nftables a été créé pour palier aux problèmes de performance et de scalabilité d'iptables.  nftable introduit une nouvelle synthaxe.

**Note**: 
- Il existe des [outils](https://wiki.nftables.org/wiki-nftables/index.php/Moving_from_iptables_to_nftables) pour convertir de la conf iptables en conf nftables.
- Il n'existe pas de chaînes par défaut dans nftables

### Ressources
- https://linuxhandbook.com/iptables-vs-nftables/
- https://wiki.archlinux.org/title/nftables