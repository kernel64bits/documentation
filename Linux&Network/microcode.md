

# microcode

# Microcode
## Introduction
>CPU **microcode** is a form of [firmware](https://en.wikipedia.org/wiki/Firmware "wikipedia:Firmware") that controls the processor's internals. This document describes various ways to update a CPU's microcode in Gentoo.
>In modern [x86](https://en.wikipedia.org/wiki/X86 "wikipedia:X86") processors, the microcode often handles execution of complex and highly specialized instructions. Parts of the microcode also act as firmware for the processor's embedded controllers, and it is even used to fix or to mitigate **processor design/implementation errata/bugs**. Given the complexity of modern processors, a CPU may have over a hundred such errata[[1]](https://wiki.gentoo.org/wiki/Microcode#cite_note-1).
https://wiki.gentoo.org/wiki/Microcode
### Firmware vs driver
Les firmwares sont stockés directement dans le device (ex: CPU) alors que les drivers sont stockés dans l'OS (directement dans le kernel ou en module).
https://wiki.ubuntu.com/Kernel/Firmware.
## Mise à jour
Ici on s'intéresse uniquement aux microcodes de CPU, cela peut potentiellement s'appliquer à d'autres types de composants.

**NB:** Il n'est pas possible de modifier directement le microcode stocké dans la ROM CPU, mais les processeurs peuvent charger des patches (via le BIOS/UEFI ou l'OS) à n'importe quel moment. Ces mises à jour sont volatiles et doivent être appliquées après redémarrage.
https://community.amd.com/t5/general-discussions/demystifying-microcode-updates-for-intel-and-amd-processors/td-p/262562
### UEFI (anciennement BIOS)
Les UEFI contiennent des màj des microcodes des CPU. Au démarrage du serveur l'UEFI effectue la mise à jour automatiquement.

**Problème:** Pour ça il faut être capable de mettre à jour son BIOS et on n'a pas la main dessus -> intervention OVH nécessaire.
### iPXE
Le **boot PXE** pour _**P**reboot E**x**ecution **E**nvironment_ est un **protocole de démarrage réseau** qui permet à un ordinateur local de démarrer à partir de données situées sur un serveur distant.  Pour cela DHCP (pour la configuration IP automatique) et TFTP (pour le transfert du fichier) sont utilisés.
On peut par exemple s'en servir pour automatiser l'installation des serveurs dans un datacenter. Le client PXE est stocké dans une ROM localisée sur une carte réseau compatible.

iPXE est une implémentation d'un client PXE qui rajoute en plus de nouvelles fonctionnalités (plus de protocoles supportés, peut servir de bootloader pour le noyaux Linux, ...).

Les serveurs OVH bootent d'abord sur iPXE, avant d'éventuellement rebooter sur disque. iPXE est notamment utile lors d'un démarrage en rescue ou lors de l'installation d'un physique. OVH peut aussi s'en servir pour lancer un OS minimal au démarrage du serveur qui pourra patcher le microcode du CPU avant de rebooter sur GRUB.
### Par l'OS
Le nouveau microcode peut théoriquement être chargé à n'importe quel moment, notamment bien après la phase de démarrage (late-loading), mais cela peut engendrer des erreurs (notamment si une fonctionnalité désactivée par la mise à jour était précédemment utilisée).
#### Linux-firmware package
Le package linux-firmware contient des binaire avec, entre autre, les microcodes des CPUs.
>Linux firmware is a package distributed alongside the Linux kernel that contains firmware [binary blobs](https://en.wikipedia.org/wiki/binary_blob "wikipedia:binary blob") necessary for partial or full functionality of certain hardware devices. These binary blobs are usually proprietary because some hardware manufacturers do not release source code necessary to build the firmware itself.
https://wiki.gentoo.org/wiki/Linux_firmware

>In the context of [free and open-source software](https://en.wikipedia.org/wiki/Free_and_open-source_software "Free and open-source software"), [proprietary software](https://en.wikipedia.org/wiki/Proprietary_software "Proprietary software") only available as a [binary executable](https://en.wikipedia.org/wiki/Executable "Executable") is referred to as a **blob** or **binary blob**.
https://en.wikipedia.org/wiki/Binary_blob

```bash
ubuntu@server:~$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 22.04.2 LTS
Release:	22.04
Codename:	jammy

ubuntu@server:~$ sudo apt upgrade linux-firmware
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
linux-firmware is already the newest version (20220329.git681281e4-0ubuntu3.16).
```
Les blobs des firmwares sont stockés dans `/lib/firmwares`
##### Gestion de linux-firmware par Canonical
Le package n'est pas mis à jour très régulièrement par Canonical (la dernière mise à jour sur bionic date de mars 2022, soit plus d'un an avant la fin du support), mais les correctifs de CVE sont probablement pris en charge de la même manière que les autres paquets.

Etrangement Canonical semble avoir décidé de ne pas conserver tous les drivers. Ainsi la dernière version du package pour jammy fait ~220 Mo (août 2023) contre ~440 Mo pour celui de kernel.org.

Par contre on peut remarquer que le package `linux-firmware d'ubuntu` fait ~150 Mo contre 400 Mo pour celui de kernel.org -> il manque des choses, notamment dans les microcodes de certaines familles de CPU AMD

#### Chargement des blobs par Linux
Des blobs contenant les binaires des firmwares sont installés dans _/lib/firmware_ par l'intermédiaire d'un paquet (ex: linux-firmware). Au démarrage:
- Le driver demande un firmware particulier
- Le kernel envoie un évènement à udev pour lui demander le firmware
- Udev lancer un script qui pousse les données binaires du firmware dans un fichier spécial créé par le kernel
- Le kernel lit les données du firmware depuis le fichier spécial qu'il a créé et envoie les données au driver
- Le driver se charge ensuite de charger le firmware dans le périphérique

Le firmware est donc toujours chargé par le kernel au démarrage mais cela peut être fait à différentes étapes en fonction de là où il est récupéré (et chargé):
##### Directement dans le kernel
Les mises à jour du firmware peuvent être ajoutées directement dans le binaire du kernel.
https://wiki.gentoo.org/wiki/AMD_microcode
procédure à détailler
##### Dans le initramfs
- Des blobs contenant les binaires des firmwares sont installés dans _/lib/firmware_ par l'intermédiaire d'un paquet (ex: linux-firmware)
- L'installation trigger la mise à jour du fichier initramfs (ou bien c'est fait à la main) 
- Comme on a le paramètre `MODULES=most` dans _/etc/initramfs-tools/initramfs.conf_, mkinitramfs décide d'intégrer les firmwares nécessaires (à creuser parce que c'est pas très clair)
https://toroid.org/possible-missing-firmware
##### Depuis _/lib/firmware_
Options la plus simple mais pas possible pour le microdode d'un CPU.
>When microcode is available and the kernel is configured, it will update microcode automatically. In most modern configurations the root partition (where the /lib/firmware directory is located) will be mounted during the boot process. For this reason, to be able to update the microcode as soon as possible, it is also necessary to include the microcode firmware blobs either in the kernel image or the initrd/initramfs.
https://wiki.gentoo.org/wiki/AMD_microcode
Le dossier _/lib/firmware_ est chargé trop tard. Il faut nécessairement stocker les blobs dans lhttps://wiki.gentoo.org/wiki/AMD_microcodee kernel ou l'initramfs.
## Suivre les mises à jour des microcodes
La mise à jour du microcode d'un CPU est surtout utile en cas de vulnérabilité (ex: zenbleed). Il est difficile de trouver des communications constructeur à ce sujet. La source la plus intéressante que j'ai trouvée est le projet CPUMicrocodes (voir annexe) qui fournit les binaires et les numéros de versions de nombreux modèles de CPU.
## Mitigation en cas de faille
Plusieurs solutions:
- Installer une nouvelle version du microcode sur laquelle la faille a été corrigée
- Désactiver la fonctionnalité impactée par la faille (peut impacter les performances du CPU) via un [chicken bit](https://en.wiktionary.org/wiki/chicken_bit)
	- Dans la configuration kernel (nécessite une mise à jour du kernel et un reboot)
	- A chaud en éditant directement les registres du processeur via l'utilitaire msr-tools (non-persistant au reboot et potentiellement risqué)
## Conclusion
Ubuntu est normalement configuré pour mettre automatiquement à jour le microcode du firmware au démarrage à partir du package linux-firmware (à condition que l'OS reçoive toujours les mises à jour). ll est théoriquement possible de télécharger les binaires et de les inclure directement dans le kernel, mais c'est un travail manuel fastidieux que je n'ai pas essayé.

Les constructeurs communiquent peu sur les microcodes et leurs mises à jour (cf annexe), mais le plus important n'est pas d'avoir un microcode à jour. On veut surtout ne pas être vulnérable aux exploits CPU connus: pour cela on peut agir à la fois sur le microcode mais aussi le kernel. Il est donc important d'avoir une politique de mise à jour efficace aussi bien pour les paquets classiques que pour le kernel.
En parallèle il est important de pouvoir auditer régulièrement la configuration des serveurs bare metal afin de vérifier leur vulnérabilités aux différentes attaques connues. L'outil `spectre-meltdown-checker` (recommandé par OVH) semble à jour et efficace, propose un rapport complet et des actions pour corriger les failles (options kernel, ...) en cas de vulnérabilité.
# Annexes
- La doc de Gentoo très complète: https://wiki.gentoo.org/wiki/AMD_microcode
- La liste la plus exhaustive des dernières versions de microcodes disponibles: https://github.com/platomav/CPUMicrocodes
- Récupérer le cpuid et la révision du microcode (utile pour la source précédente): https://gist.github.com/liske/6743d6dc692dead93228848dc480281c
- Audit des principales failles CPU: https://github.com/speed47/spectre-meltdown-checker
-  https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/: pour télécharger la dernière version du package. Regarder l'onglet tree pour avoir le détails des fichiers.
- Changelog du package `linux-firmware` d'Ubuntu: https://launchpad.net/ubuntu/+source/linux-firmware
- Audit de la vulnérabilité aux différentes failles CPU connues: https://github.com/speed47/spectre-meltdown-checker