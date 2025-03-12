

# 05-systemd

https://www.unixmaniax.fr/wiki/index.php?title=Systemd
# Vocabulaire
- daemon
- service
- runlevel
- target
# Introduction
Remplaçant de sys V init (5 en chiffre romain) pour la gestion des services et du démarrage sous Linux.
Utilisé notamment par:
- Ubuntu/Debian
- Arch Linux
- Fedora/RedHat/...
## Avantages
- Parallélisme: contrairement à ses prédécesseurs, systemd permet de lancer les services au parallèle (gain de temps au démarrage). Il existe un système de dépendances entre les services (ex: ssdh ne sera pas lancé avant que la configuration réseau ne soit terminée).
## Critiques
- Remplacement de services système par des implémentations de systemd (ex: systemd-resolved)
# A quoi ça sert
# Son fonctionnement
## systemd units
- Service: un service système
- Target: permet de grouper plusieurs unités (remplace les runlevels de system V)
- socket
- busname
### Units directories
- _/etc/sysctl.conf_: Fichier de configuration principal
- _/usr/lib/systemd/system_ ou  _/lib/systemd/system_: configuration de base
- _`/run/systemd/system/`_
- _/etc/systemd/system/_: modifications personnelles
## Commands
- `systemctl`
- `systemctl -t help`: Lister tous les types d'unités
-`systemctl list-units`: Lister toutes les unités existantes sur le système
# Côté historique
Sources
- https://www.section.io/engineering-education/understanding-systemd/
- https://opensource.com/article/20/5/systemd-startup
- https://www.linode.com/docs/guides/what-is-systemd/
