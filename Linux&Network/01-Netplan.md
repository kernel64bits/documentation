

# 01-Netplan

## Systemd-networkd

Sur ubuntu 22, la configuration réseau est effectuée par le service systemd-networkd. Les fichiers sont généralement stockés dans */etc/systemd/network/*, ils sont répartis en 3 types:
- ` *.link`: permet de renommer des interfaces (alternative à udev)
- ` *.netdev`: Créé des interfaces réseau virtuelles
- `*.network`: Configuration réseau générale

### Commandes
- `networkctl list`: permet de lister les interfaces
- `networkctl status`
- `systemctl restart systems-networkd`

### Ressources:
- [Documentation systemd-networkd](https://wiki.archlinux.org/title/systemd-networkd) 

## Netplan
Netplan est un outil visant à faciliter la configuration réseau sur Linux. Neptlan se base sur des fichiers de configuratgion YAML générer des fichiers de conf système-networkd dans /run/systemd/network/ au démarrage de l’OS. Netplan est aussi compatible avec **Network Manager**, un concurrent de **systemd-networkd**.

Quelques fonctionnalités de netplan (list non-exhaustive):
- Assigner une IP à une interface
- Configurer un VLAN sur un interface
- Créer des règles de routage
- Renommer une interface

## Commandes
- `netplan generate`
- `netplan try`: Applique la configuration et attend une confirmation de l'utilisateur. Un rollback est effectué en cas d'absence de confirmation de l'utilisateur ou si le réseau est cassé.
- `netplan apply`: Applique la nouvelle configuration réseau

**Note:** `netplan try` et `netplan apply` se contentent d'appliquer la nouvelle configuration réseau mais ne supprime pas toutes les configurations antérieures.

### Ressources
- [https://netplan.io/examples](https://netplan.io/examples)
- https://netplan.io/reference
