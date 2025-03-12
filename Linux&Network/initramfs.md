

# initramfs

# initramfs
## Introduction
> **initramfs:** type d'images de systèmes de fichiers permettant l'utilisation de mécanismes de mise en place de [répertoire racine](https://fr.wikipedia.org/wiki/R%C3%A9pertoire_racine "Répertoire racine") temporaire minimal chargé dans la [mémoire vive](https://fr.wikipedia.org/wiki/M%C3%A9moire_vive "Mémoire vive") au démarrage du [noyau Linux](https://fr.wikipedia.org/wiki/Noyau_Linux "Noyau Linux"). Ces mécanismes sont utilisés pour préparer le système d’exploitation avant la mise en place du vrai [système de fichiers](https://fr.wikipedia.org/wiki/Syst%C3%A8me_de_fichiers "Système de fichiers") racine contenant le reste du système d’exploitation, en utilisant un [point de montage](https://fr.wikipedia.org/wiki/Point_de_montage "Point de montage").

Pour plus d'informations: https://wiki.gentoo.org/wiki/Initramfs/Guide/en
Les initramfs sont utilisés quand le kernel ne dispose pas de tous les outils pour mener à bien le démarrage du serveur.

## Cas d'OVH
Les serveurs bare metal nécessitent un initramfs depuis la version 2 du kernel. Cette version a été développée après une mise à jour du template d'installation OVH qui a entraîné un passage de la version du superblock (raid logiciel) 0.9 à 1.* qui n'est pas reconnue nativement par le kernel (https://raid.wiki.kernel.org/index.php/RAID_superblock_formats).

```bash
ubuntu@server:~$ lsblk
NAME    MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda       8:0    0  12.8T  0 disk  /btrfs
sdb       8:16   0  12.8T  0 disk
sdc       8:32   0  12.8T  0 disk
sdd       8:48   0  12.8T  0 disk
sde       8:64   0 447.1G  0 disk
├─sde1    8:65   0   511M  0 part
├─sde2    8:66   0   512M  0 part
│ └─md2   9:2    0 511.4M  0 raid1 /boot
├─sde3    8:67   0    20G  0 part
│ └─md3   9:3    0    20G  0 raid1 /
├─sde4    8:68   0    10G  0 part
│ └─md4   9:4    0    10G  0 raid1 /var
├─sde5    8:69   0   9.8G  0 part
│ └─md5   9:5    0   9.8G  0 raid1 /tmp
├─sde6    8:70   0 406.3G  0 part
│ └─md6   9:6    0 406.2G  0 raid1 /home
├─sde7    8:71   0    10M  0 part  [SWAP]
└─sde8    8:72   0     2M  0 part
sdf       8:80   0 447.1G  0 disk
├─sdf1    8:81   0   511M  0 part  /boot/efi
├─sdf2    8:82   0   512M  0 part
│ └─md2   9:2    0 511.4M  0 raid1 /boot
├─sdf3    8:83   0    20G  0 part
│ └─md3   9:3    0    20G  0 raid1 /
├─sdf4    8:84   0    10G  0 part
│ └─md4   9:4    0    10G  0 raid1 /var
├─sdf5    8:85   0   9.8G  0 part
│ └─md5   9:5    0   9.8G  0 raid1 /tmp
├─sdf6    8:86   0 406.3G  0 part
│ └─md6   9:6    0 406.2G  0 raid1 /home
└─sdf7    8:87   0    10M  0 part  [SWAP]
```
Après analyse les partitions sde1 et sdf1 sont toutes les 2 des partitions EFI avec le même contenu mais sans raid (ou en tout cas il n'est pas utilisé): seule une des 2 partition est montée au démarrage parce que l'UEFI ne supporte pas le raid logiciel.

Bien qu'OVH n'autorise que du raid 1 pour /boot, GRUB est bien capable de gérer d'autres types de raid à condition de charger le module mdraid09 ou mdraid1x (en fonction de la version de superblock).
## Générer un initramfs
Il existe plusieurs outils: 
- [update-initramfs](https://manpages.ubuntu.com/manpages/focal/en/man8/update-initramfs.8.html): facile à utiliser mais très peu d'options de customisation. C'est juste un script bash qui appelle `mkinitramfs`
- [mkinitramfs](https://manpages.ubuntu.com/manpages/focal/en/man8/mkinitramfs.8.html): Pour un usage plus avancé. Utilise un ensemble de fichiers de configuration stockés dans _/etc/initramfs-tools/_
- [dracut](https://manpages.ubuntu.com/manpages/xenial/man8/dracut.8.html): Outil très avancé également