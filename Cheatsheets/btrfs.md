Afficher des infos
```
btrfs fi df /btrfs #Type de raid
btrfs filesystem show /dev/sda #Infos générales
df -h #Utilisation espace disque

sudo btrfs subvolume list /btrfs/ #Lister les sous-volumes btrfs
sudo btrfs subvolume show /btrfs/subvolume #informations sur un volume particulier

sudo btrfs fi usage /btrfs
sudo btrfs fi df /btrfs #Espace utilisé/libre/total, il indique l'espace total sans prendre en compte le type de raid
sudo btrfs fi du -s /btrfs/* #Espace disque pour chaque sous-volume
```
​
Créer un volume en raid 1 (écrase toutes les données)
```
umount /btrfs
mkfs.btrfs -L data -m raid1 -d raid1 /dev/sd{c,d} -f
mount /btrfs
```
​
Créer un nouveau sous-volume
```
btrfs subvolume create /btrfs/subvolume
```

Supprimer un sous-volume
```
sudo umount /data/vms_data/<subvolume_name>
sudo rmdir /data/vms_datas/<subvolume_name>
btrfs subvolume delete /btrfs/<subvolume_name>
```
​
Monter un sous-volume
```
mount -t btrfs -o subvol=subvolume /dev/sda /data/vms_datas/subvolume
```
​
Convertir un volume en raid1
```
sudo btrfs balance start -dconvert=raid1 -mconvert=raid1 /btrfs
```

Avancement de la conversion
```
sudo btrfs balance status /btrfs
```

Remplacer un disque défectueux

!!! Procédure en cours de rédaction. Backuper les données critiques.

1. Backuper les données critiques
2. Redémarrer le serveur en rescue et faire une demande de changement de disque
3. Après l'intervention, redémarrer le serveur en mode classique. Les conteneurs n'utilisant pas le btrfs peuvent être redémarrés
4. Identifier le nom de du disque survivant et le devid du disque à remplacer (celui du nouveau disque): `sudo btrfs filesystem show`
5. Monter le volume btrfs en mode dégradé: `sudo mount -o degraded /dev/sda /btrfs` (il faut parfois s'y reprendre à plusieurs fois: 10-15 ou bien attendre ?). On peut vérifier qu'il a bien été monté avec un `lsblk` ou un `mount -l`
6. Ajouter le nouveau disque à l'array: `sudo btrfs replace start <devid> /dev/<new_disk> /btrfs` (pareil ça ne marche pas toujours du 1er coup). L'ajout va se faire en tâche de fond.
7. Monitorer l'état de la reconstruction du raid (peut prendre du temps s'il y a bcp de données): ` sudo btrfs replace status /btrfs`
8. Une fois le raid reconstruit, remonter le raid en mode normal:
  1. `sudo umount /data/vms_datas/*`
  2. `sudo umount /btrfs`
  3. `sudo mount /btrfs`
  4. `sudo systemctl restart btrfs_mount.service`
9. Vérifier l'état du btrfs avec `sudo btrfs filesystem show`, lancer une conversion en raid 1 si besoin
10. Redémarrer les conteneurs restants

https://ohthehugemanatee.org/blog/2019/02/11/btrfs-out-of-space-emergency-response/
