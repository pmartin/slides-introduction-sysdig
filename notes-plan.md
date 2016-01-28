
Dans les grandes lignes, les idées dont j'aimerais parler, quelques notes, ça peut évoluer vers une sorte de plan.

# Que fait mon système ???

Questione récurrente : que se passe-t-il sur mon système ? Que ce soit machine locale, serveur de prod, physique ou virtuelle ou container.

Nombreux outils pour répondre, en partie, à cette question (descriptions fort simplifiées) :

 * [`ps`](https://en.wikipedia.org/wiki/Ps_%28Unix%29) : liste les processus en cours
 * `top` / `htop` : affiche les processus en cours, triés par consommation CPU/RAM/autre ; éventuellement avec en plus quelques mini-graphiques
 * `nethogs` : liste les processus qui consomment du réseau et affiche combien pour chaque
 * `tcpdump` : liste ce qui passe sur le réseau
 * [`lsof`](https://en.wikipedia.org/wiki/Lsof) : liste les fichiers ouverts et les processus qui y accèdent -> vu que plein de choses sous UNIX sont représentées par des fichiers, c'est déjà fort pratique ;-)
 * [`netstat`](https://en.wikipedia.org/wiki/Netstat) : infos à propos du réseau (connexions ouvertes, tables de routage, ...)
 * `strace` : liste les appels systèmes effectués par un processus, au fur et à mesure de son exécution

Bon, ça permet d'obtenir **plein** d'informations, mais il faut utiliser 36 outils différents, qui ont tous des options différentes... Pas très marrant et pas évident !


# sysdig


## C'est quoi sysdig ?

Cf [http://www.sysdig.org/wiki/sysdig-overview/](http://www.sysdig.org/wiki/sysdig-overview/)

## Installation

Cf [Installation](http://www.sysdig.org/install/)

Sous Debian :

```bash
curl -s https://s3.amazonaws.com/download.draios.com/DRAIOS-GPG-KEY.public | sudo apt-key add -
sudo curl -s -o /etc/apt/sources.list.d/draios.list http://download.draios.com/stable/deb/draios.list
sudo apt-get update

sudo apt-get install -y linux-headers-$(uname -r)
sudo apt-get install -y sysdig
```

Attention : lancer la commande `sysdig` sans aucun paramètre pour contrôler un peu ce qu'elle va faire, c'est s'exposer à **beaucoup** de lignes qui vont défiler dans la console !


## Exemples simples, puis cool !

Cf [Bonne série d'exemples de commandes](http://www.sysdig.org/wiki/sysdig-examples/) et [Sysdig CLI examples](https://ma.ttias.be/sysdig-cli-examples/)


Dans les trucs à montrer :

 * On voit *des choses* qui *défilent* en live
 * On peut les filtrer
 * On a des chisels pour agréger ou autres
 * On peut enregistrer ce qu'il se passe vers des fichiers, puis analyser ce qui a été enregistré, en off-line


### Les filtres



```bash

```



```bash

```



```bash

```

Lister toutes les connexions réseau entrantes, qui sont servies par un processus autre que `nginx` :

```bash
 sysdig -p"%proc.name %fd.name" "evt.type=accept and proc.name!=nginx"
```


### Les chisels

Scripts qui analysent le flux d'événements pour réaliser des actions *utiles*.

Liste des chisels disponibles :

```bash
sysdig -cl
```

Plus d'informations sur un chisel

```bash
sysdig -i NOM_DU_CHISEL
```

Exécuter un chisel :

```bash
sysdig -c NOM_DU_CHISEL
```

Exemple : lister en temps réel les fichiers avec le plus de lectures/écritures (en volume d'octets) :

```bash
sysdig -c topfiles_bytes
```

Par défaut : n'affiche pas d'information liée à des containers.

Même chose, mais en incluant les informations liées à des containers ; on ajoute le flag `-pc`

```bash
sysdig -c topfiles_bytes -pc
```

=> On peut faire une passe sur les chisels existant : il y en a pas mal qui peuvent être utiles, selon les cas ;-)

Tracer les accès fichier lents (plus de 3ms), y compris dans des containers :

```bash
sysdig -pc -c fileslower 3
```

Afficher les containers en cours d'exécution (docker, coreos, lxc) :

```bash
sysdig -c lscontainers [desc]
```

Je parlais plus haut de `nethogs` pour voir les process qui consomment du réseau ; ça se fait aussi avec sysdig :

```bash
sysdig -c topprocs_net
```




### Chisels + Filtres

Bien sûr, les chisels peuvent être combinés avec des filtres, pour affiner ce sur quoi les chisels sont appliqués !

Par exemple, pour visionner les fichiers qui font le plus gros volume d'I/O, en se limitant à un répertoire donné :

```bash
# TODO n'a pas l'air de marcher
sysdig -pc -c topfiles_bytes "fd.name contains /www/var/cache/"
```

Ou pour les fichiers accédés uniquement par le processus php-fpm :

```bash
# TODO n'a pas l'air de marcher
sysdig -pc -c topfiles_bytes "proc.name=php-fpm"
```

Lister les fichiers qui représentent les plus d'I/O, uniquement pour le container `tea-ecommerce` :

```bash
sysdig -pc -c topfiles_bytes container.name=tea-ecommerce
```


# Et avec des containers ?

Depuis le système *physique*, on peut analyser ce qu'il se passe dans les containers : juste *ça marche* ;-)



# csysdig



Lister tous les processus (y compris ceux dans des containers) :

```bash
csysdig
```

Lister tous les processus (y compris ceux dans des containers), avec le contexte du container :

```bash
csysdig -pc
```

Lister tous les containers qui tournent en local, avec les ressources qu'ils consomment :

```bash
csysdig -vcontainers
```





```bash

```



```bash

```



```bash

```







# Aller plus loin ?

Ca se scripte ; en Lua notamment, cf [Chisels User Guide](http://www.sysdig.org/wiki/chisels-user-guide/)



# Conclusion, notes finales, questions

C'est cool \o/

Un seul outil et plus 36 outils différents avec des interfaces et options différentes.

Analyse *on-line* ou *off-line*. Fouillage en profondeur.
