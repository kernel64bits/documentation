

# Bird

# Bird

## Installation
Attention à installer le package `bird2`

## Protocol
Plusieurs protocoles sont supportés pas bird.
## Babel
Permet d'éviter les boucles. Je pense que ce n'est pas nécessaire avec BGP et que ça doit être utile uniquement pour des protocoles comme RIP qui est aussi supporté par BGP.
### Device
Permet à Bird de récupérer la liste des interfaces réseau
### Kernel
Permet de synchroniser les tables de routage de bird avec celles du kernel. On peut spécifier une table de routage spécifique. Les routes crées automatiquement par le kernel (ie les routes vers les sous-réseaux des interfaces) ne peuvent pas être importées via ce protocole, il faut passer par Direct.
### Direct
Permet d'importer dans bird les routes vers les réseaux directement connectés au serveur. On peut spécifier les interfaces.
### BGP
Permet de faire du BGP avec d'autres serveurs.

https://bird.network.cz/?get_doc&f=bird.html&v=20

## Commandes

- `birdc show protocols`: Affiche l'état des connexions en cours
- `birdc show route`