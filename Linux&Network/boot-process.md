

# boot process

# Linux boot process

## 1. Firmware de la carte-mère: BIOS ou UEFI

### BIOS (Legacy)

### UEFI
**UEFI**: Unified Extensible Firmware Interface.
C'est une nouvelle norme qui remplace les BIOS. Les améliorations sont:
- Support de la souris et interface graphique
- Support de tailles de disques plus importantes
- Secure boot: vérification de la chaîne de démarrage via des signatures cryptographiques des différents programmes (les clés publiques sont stockées dans les TPM).
## 2. Table de partition: MBR ou GPT

### MBR (Master Boot Recode) (Legacy)
Nom donné au premier secteur adressable d'un disque dur. Sa taille est de 512 octets. Le MBR contient la table des partitions (les quatre partitions primaires) du disque dur, ainsi que le code d'amorçage (Master Boot Code).

Au démarrage le BIOS lit le contenu du MBR et exécute le code d'amorçage dont l'objectif est de lancer le chargeur d'amorçage (bootloader).

Limitations:
- 2 To max par partition
- Le MBR ne peut stocker que 4 partitions (appelées partitions primaires). Pour dépasser cette limitation on peut étendre la table de partition dans une partition étendue (remplace une partition primaire) qui peut contenir jusqu'à 128 partitions logiques.

MBR est utilisé sur les anciennes installations OVH.
### GPT (GUID (Globally Unique Identifier Partition Table) Partition Table)
Nouveau standard de table de partitionnement qui remplace le MBR, c'est une spécification de l'UEFI.
Les améliorations sont:
- Plus de partitions supportées
- Tailles maximale des partitions plus élevée

Au démarrage, l'UEFI charge directement le bootloader (stocké dans /boot, /efi, ou /boot/efi) enregistré sur la forme d'un fichier .efi stocké dans une partition efi (chez OVH elle est montée dans _/boot/efi_).

Afficher la table de partition
- `lsblk`
- `fdisk -l`

GPT est utilisé sur les nouvelles installations OVH.
## 3. Boot loader
Ses objectifs sont:
- Localiser le kernel sur le disque
- Le charger en RAM
- L'exécuter avec les bonnes options
- Si un initramfs existe:
	- le localiser sur le disque
	- le charger en RAM

Le bootloader le plus répandu s'appelle GRUB (Grand Unified Boot Loader). C'est un des plus complets (mais aussi un des plus complexes), il permet notamment de choisir parmi plusieurs choix au démarrage (un choix c'est un kernel, une partition root, et d'autres options éventuelles).

**NB:** initramfs dispose d'une documentation dédiée
## 4. Kernel
Here, the Linux kernel follows a predefined procedure:
1. decompress itself in place
2. perform hardware checks
3. gain access to vital peripheral hardware
4. run the init process

Next, the init process continues the system startup by running init scripts for the parent process. Also, the init process inserts more kernel modules (like device drivers).
## 5. Init Process
Le processus init est le parent de tous les autres processus (pid 0). Son objectif est de gérer le démarrage et l'extinction des process. Notamment quels process lancer automatiquement au démarrage et dans quel ordre.
### Systemd
Ce n'est pas le seul mais c'est celui le plus répandu (et qui est utilisé par Ubuntu). Parmis ses tâches:
- probe all remaining hardware
- mount filesystems
- initiate and terminate services
- manage essential system processes like uer login
- run a desktop environment
## 6. Run levels
Le run level (ou niveau d'exécution) est un chiffre ou une lettre utilité par le processus init pour déterminer les fonctions activées
En général il y a 7 niveaux (0-6) et parfois un niveau "S". Chacun correspond à un ensemble d'applications à mettre en marche. En général, plus le run level est élevé, plus il y aura de fonctions actives. La gestion des run levels dépend beaucoup de la distribution utilisée. En général on a:
0. Arrêt
1. Mode mono-utilisateur ou maintenance 
2. Varie selon la distribution
3. Varie selon la distribution
4. Varie selon la distribution
5. Varie selon la distribution
6. Reboot 

On passe d'un run level à l'autre en utilisant la commande [init](https://fr.wikipedia.org/wiki/Init "Init") ou telinit (ou encore [shutdown](https://fr.wikipedia.org/wiki/Shutdown "Shutdown") pour les transitions vers 0 ou 6). La transition d'un niveau à l'autre va lancer des scripts d'arrêt et de démarrage de fonctions.
### Ubuntu
Sur ubuntu les run levels sont remplacés par des targets:
- poweroff.target (0)
- rescue.target (1): initie un shell de rescue
- multi-user.target (3): configure le système en environnement multi-utilisateur sans interface graphique
- graphical.target (5)
- reboot.target (6)
Au démarrage d'ubuntu, Sytemd active le run_level par defaut qui est graphical (même si aucune interface graphique n'est installée). 

```bash
# systemctl get-default
graphical.target
```

https://fr.wikipedia.org/wiki/Run_level
https://www.answertopia.com/ubuntu/managing-ubuntu-systemd-units/
https://likegeeks.com/linux-runlevels/
https://www.liquidweb.com/kb/linux-runlevels-explained/
https://www.freecodecamp.org/news/the-linux-booting-process-6-steps-described-in-detail/