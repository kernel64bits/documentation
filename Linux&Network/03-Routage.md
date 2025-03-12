

# 03-Routage

## IP route (modifier les règles de routage)

### Scope
Chaque route est associée à un scope:
- global: valide partout (valeur par défaut)
- site: valable uniquement sur le site (IPv6 only, à déterminer)
- link: valide uniquement sur le lien (utilisé pour le broadcast sur un LAN)
- host: valide uniquement sur la machine (ex: 127.0.0.1)
https://serverfault.com/questions/63014/ip-address-scope-parameter

## IP rule (tables de routage multiple)

Dans une table de routage, les règles ne prennent en compte que l’IP de destination pour prendre une décision.
Si l'on souhaite modifier le routage en fonction d'autres paramètres (IP source, protocole, interface réseau, ...) il est possible de créer plusieurs tables de routage (avec des règles différentes) et de créer des règles pour choisir quelle table de routage utiliser en fonction du paquet. Les tables de routage sont numérotées (de 0 à 255, par défaut il n'en existe que quelques unes mais on peut en créer d'autres). Il est également possible de donner un nom à une table de routage. 

- lister les règles (et leur poid, et les tables de routage associées): `ip rule`
- Afficher le contenu de la table de routage 220: `ip route show table 220`
```bash
ubuntu@server:~$ ip rule
0: from all lookup local
32766: from all lookup main
32767: from all lookup default
```
Chaque règle a une valeur de priorité.

Les règles peuvent être configurées sur netplan via routing-policy: [https://itecnotes.com/server/netplan-configure-two-interfaces-on-same-network/](https://itecnotes.com/server/netplan-configure-two-interfaces-on-same-network/)

Ressources:
- https://man7.org/linux/man-pages/man8/ip-rule.8.html