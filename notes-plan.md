
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
 * `nmap` : pour lister les ports sur lesquels quelque chose écoute -> doit pouvoir être remplacé par une sonde sysdig ?
 * `iostat`
 * `vmstat`

Bon, ça permet d'obtenir **plein** d'informations, mais il faut utiliser 36 outils différents, qui ont tous des options différentes... Pas très marrant et pas évident !


Exemples :

```bash
ps -Alf
top
htop
nethogs wlan0
tcmdump
lsof /home/pmartin/projects/ecommerce

netstat
netstat --route
netstat --statistics

strace php -S localhost:8001
```


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


## Premiers pas

Attention : lancer la commande `sysdig` sans aucun paramètre pour contrôler un peu ce qu'elle va faire, c'est s'exposer à **beaucoup** de lignes qui vont défiler dans la console !

Mais on peut limiter à quelques événements, pour voir à quoi ça ressemble :

```bash
sysdig -n 100
```

Note : pour pas mal de trucs, il faut lancer `sysdig` en root : ça va chercher des informations bien bas-niveau, pas forcément accessibles à n'importe qui ;-)


Format de sortie par défaut :

```
By default, sysdig prints the information for each captured event on a single
 line with the following format:

 %evt.num %evt.outputtime %evt.cpu %proc.name (%thread.tid) %evt.dir %evt.type %evt.info

where:
 evt.num is the incremental event number
 evt.time is the event timestamp
 evt.cpu is the CPU number where the event was captured
 proc.name is the name of the process that generated the event
 thread.tid id the TID that generated the event, which corresponds to the
   PID for single thread processes
 evt.dir is the event direction, > for enter events and < for exit events
 evt.type is the name of the event, e.g. 'open' or 'read'
 evt.info is the list of event arguments.
```


## Analyse en live / replay

Par défaut, sortie directement sur la console. Mais on peut également enregistrer les traces vers un fichier, pour ensuite les analyser.

Enregistrer vers un fichier :

```bash
sysdig -w mon-fichier.scap
```

Analyser ce qui est dans le fichier :

```bash
sysdig -r mon-fichier.scap
```

On peut spécifier une taille max par fichier, faire du rotate de fichiers, compresser les fichiers, ...


## Et avec des containers ?

Depuis le système *physique*, on peut analyser ce qu'il se passe dans les containers : juste *ça marche* ;-)

Il faut, occasionnellement, utiliser l'option `-pc` pour avoir de la visibilité à l'intérieur des containers.

Et quand on a `-pc`, sortie par défaut :

```
Using -pc or -pcontainer, the default format will be changed to a container-friendly one:

%evt.num %evt.outputtime %evt.cpu %container.name (%container.id) %proc.name (%thread.tid:%thread.vtid) %evt.dir %evt.type %evt.info
```


## Exemples simples, puis cool !

Cf [Bonne série d'exemples de commandes](http://www.sysdig.org/wiki/sysdig-examples/) et [Sysdig CLI examples](https://ma.ttias.be/sysdig-cli-examples/)


Dans les trucs à montrer :

 * On voit *des choses* qui *défilent* en live
 * On peut les filtrer
 * On a des chisels pour agréger ou autres
 * On peut enregistrer ce qu'il se passe vers des fichiers, puis analyser ce qui a été enregistré, en off-line


### Les filtres

Observer *tout* ce qu'il se passe, c'est **très** verbeux et clairement pas une bonne idée ^^

=> On peut filtrer \o/

Par exemple, pour avoir toutes les informations liées à un (ou plusieurs) processus nommé(s) `php` :

```bash
sysdig proc.name=php
```

On peut lancer un serveur PHP en CLI une fois cette commande en cours :

```bash
cd examples
php -S localhost:8001
```

=> Si on fait des requêtes sur `http://localhost:8001/` ou sur `http://localhost:8001/phpinfo.php`, on voit tout ce que fait le processus \o/

On peut combiner des filtres, bien sûr, pour ne conserver que ce qui nous intéresse :

```bash
sysdig -r mon-fichier.scap "proc.name=php and evt.type=open and fd.name contains php and not fd.directory contains /usr/"
```

Pour ce qui est de combiner des filtres et de filtres un peu avancés :

 * Opérateurs : =, !=, <, <=, >, >=, contains
 * Combinaison : or, and, not
 * On peut utiliser des parenthèses


On peut voir tous les répertoires parcourus par l'utilisateur `root` :

```bash
sysdig "evt.type=chdir and user.name=root"
```

Voir passer toutes les ouvertures de fichiers au fur et à mesure qu'elles arrivent, en choisissant les informations affichées gràce à `-p` :

```bash
sysdig -p "%12user.name %6proc.pid %12proc.name %3fd.num %fd.typechar %fd.name" evt.type=open
```

Même chose, en limitant à une portion du chemin :

```bash
sysdig -pc -p "%12user.name %6proc.pid %12proc.name %3fd.num %fd.typechar %fd.name" "evt.type=open and fd.name contains /var/cache/"
```

Lister toutes les connexions réseau entrantes, qui sont servies par un processus autre que `nginx` :

```bash
 sysdig -p"%proc.name %fd.name" "evt.type=accept and proc.name!=nginx"
```

Et si on veut tous les fichiers auxquels nginx essaye d'accéder ; mais échoue (genre, les fichiers en 404 sur nos sites) :

```bash
sysdig "proc.name=nginx and evt.type=open and evt.failed=true"
```

Ou pour voir toutes les requêtes SQL de sélection jouées depuis des process PHP :

```bash
sysdig "proc.name contains php-fpm and evt.buffer contains SELECT"
```

Pour obtenir la liste des filtres existant :

```bash
sysdig -l
```

Et pour obtenir la liste des types d'événements existant (`>` indique un événement *entrant* et `<` un événement *sortant*) :

```bash
sysdig -L
```


### Les chisels

Scripts qui analysent le flux d'événements pour réaliser des actions *utiles*.

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

Liste des chisels disponibles :

```bash
sysdig -cl
```

Plus d'informations sur un chisel

```bash
sysdig -i NOM_DU_CHISEL
```


### Chisels + Filtres

Bien sûr, les chisels peuvent être combinés avec des filtres, pour affiner ce sur quoi les chisels sont appliqués !

Lister les fichiers qui représentent les plus d'I/O, uniquement pour le container `tea-ecommerce` :

```bash
sysdig -pc -c topfiles_bytes "container.name=tea-ecommerce"
```

Ou, pour visionner les fichiers qui font le plus gros volume d'I/O, en se limitant à un répertoire donné :

```bash
sysdig -pc -c topfiles_bytes "container.name=tea-ecommerce and fd.directory contains cache"
```

Ou pour les fichiers accédés uniquement par le processus php-fpm :

```bash
sysdig -pc -c topfiles_bytes "proc.name contains php-fpm"
```

Pour afficher une sorte de spectrogramme des fichiers accédés :

```bash
sysdig -c spectrogram "fd.type=file and container.name=tea-ecommerce"
```

Suivre les logs de plusieurs containers (surtout si on n'a rien fait pour les mettre *où il faut*), c'est un peu compliqué ? Bah non ;-)

```bash
sysdig -pc -cspy_logs
```

Bien sûr, ça se filtre :

```bash
sysdig -pc -cspy_logs "fd.name contains access.log or container.name=tea-db"
```

Et il y a plein de choses qu'on peut faire ; genre visionner les requêtes solr sur notre index ecommerce :

```bash
sysdig -pc -cspy_logs "evt.args contains IndexEcommerce_fr and evt.args contains path=/select"
```



```bash

```



```bash

```



# csysdig

Interface *graphique* qui permet de visualiser ce qu'il se passe en live -- pensez à un `htop` boosté de folie ^^

On peut réutiliser l'option `-r` pour analyser un fichier de trace existant, également ;-)


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

Lister tous les processus qui tournent dans des containers :

```bash
csysdig -pc "container.name!=host"
```

On peut aussi analyser une requêtes sur notre serveur web PHP de test :

```bash
csysdig -pc "proc.name=php"
```

=> C'est l'occasion de faire F5 et F6 pour la sortie, digger ce qu'il se passe, ...


```bash

```




# Aller plus loin ?

Ca se scripte ; en Lua notamment, cf [Chisels User Guide](http://www.sysdig.org/wiki/chisels-user-guide/)



# Conclusion, notes finales, questions

C'est cool \o/

Un seul outil et plus 36 outils différents avec des interfaces et options différentes.

Analyse *on-line* ou *off-line*. Fouillage en profondeur.
