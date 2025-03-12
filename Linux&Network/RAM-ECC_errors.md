

# RAM&ECC errors

en cours de rédaction
# RAM and ECC errors
https://en.wikipedia.org/wiki/ECC_memory
> More than 8% of DIMM memory modules were affected by errors per year.

> in large-scale production sites, memory errors are one of the most-common hardware causes of machine crashes


Possibilité d'avoir des stats ?
> The [BIOS](https://en.wikipedia.org/wiki/BIOS "BIOS") in some computers, when matched with operating systems such as some versions of [Linux](https://en.wikipedia.org/wiki/Linux "Linux"), [BSD](https://en.wikipedia.org/wiki/BSD "BSD"), and [Windows](https://en.wikipedia.org/wiki/Windows "Windows") ([Windows 2000](https://en.wikipedia.org/wiki/Windows_2000 "Windows 2000") and later[[13]](https://en.wikipedia.org/wiki/ECC_memory#cite_note-15)), allows counting of detected and corrected memory errors, in part to help identify failing memory modules before the problem becomes catastrophic.


RAM tests

Cause des erreurs:
Interférences magnétiques ou radiations cosmiques

DRAM (Dynamic Random Access Memory) -> expliquer comment ça fonctionne, pourquoi c'est random.

Failles: https://en.wikipedia.org/wiki/Row_hammer

Les contrôles de parité permettent de détecter (mais pas résoudre) les erreurs, les ECC permettent également la correction
-> [Hamming codes](https://en.wikipedia.org/wiki/Hamming_code "Hamming code") il y a également des codes plus élaborés comme TRM


https://www.futureplus.com/blog/who-is-to-blame-for-ddr-memory-ecc-errors

## Accès aléatoire vs séquentiel (random vs sequential access)
Les périphériques de stockage sont constitués de bits. Mais le bit n'est pas le plus petit "groupe de données" que l'on peut écrire/lire (ie on ne peut pas demander à un HDD, SSD, RAM, ... d'écrire un seul bit). Il existe différents différentes subdivisions:
- cluster, bloc, secteurs pour les HDD
- pages et blocs pour les SSD
A chaque "groupe de données" est associée une adresse (ie un pointeur) qui va permettre de lire les données. Un des objectifs d'un système de fichier et de faire le lien entre le chemin d'un fichier et sa/ses localisations physiques (ie adresses) sur le disque.
- Un accès séquentiel consiste à accéder (lecture ou écriture) aux données dans l'ordre (on indique uniquement l'adresse de début et la quantité de données à lire ou écrire).
- Dans un accès aléatoire, les informations sont stockées sur des groupes de données réparties de manière aléatoire (ie non consécutives) sur le périphérique de stockage. Il est donc nécessaire de fournir l'ensemble des adresses de ces groupes de données. 

Sur un disque mécanique, l'accès à une zone de données nécessite des mouvements des plateaux et des têtes de lecture/écriture. En fonction de l'endroit où se situe la tête et de l'endroit auquel elle veut accéder le temps de déplacement sera différent.

Sur un disque dur mécanique, l'accès séquentiel est donc bien plus performant que l'accès aléatoire. Dans le premier cas on lit les données dans l'ordre, alors que dans l'autre il est nécessaire de faire des va-et-viens constants sur les différentes zones du disques. D'où l'intérêt du défragmentage.

**NB:** La vitesse de rotation d'un disque étant plus élevée aux bords qu'au centre, les données situées sur les bords seront d'accès plus rapide que celles stockées au centre.

### Qu'est-ce qu'une RAM
Une Random Access Memory ou Mémoire à Accès Aléatoire est un type de mémoire pour laquelle le temps de lecture/écriture ne dépend pas de la localisation physique des données. 

On différentie 2 types de RAM:
#### La RAM volatile
Les données sont effacées en cas d'arrêt de l'alimentation électrique. C'est le cas des fameuses barrettes de RAM. 
#### La RAM non volatile
Les données sont persistantes à un arrêt d'alimentation. C'est notamment le cas de la mémoire flash. Les clés USB, cartes mémoire, et SSD sont donc considérés comme de la RAM au sens de cette définition.

**NB:** Même si la mémoire flash est de type RAM, l'accès est plus lent pour les petits fichiers que pour les gros (à taille totale de données égale). Plus d'infos [ici](https://www.infobidouille.com/articles/26/03-secteurs-pages-blocs-comprendre-ssd).


Adressage sur une barrette de RAM:
