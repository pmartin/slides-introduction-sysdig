
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

## Exemples simples, puis cool !

Cf [Bonne série d'exemples de commandes](http://www.sysdig.org/wiki/sysdig-examples/) et [Sysdig CLI examples](https://ma.ttias.be/sysdig-cli-examples/)


Dans les trucs à montrer :

 * On voit *des choses* qui *défilent* en live
 * On peut les filtrer
 * On a des chisels pour agréger ou autres
 * On peut enregistrer ce qu'il se passe vers des fichiers, puis analyser ce qui a été enregistré, en off-line


# Et avec des containers ?


Depuis le système *physique*, on peut analyser ce qu'il se passe dans les containers.



# csysdig



# Aller plus loin ?

Ca se scripte ; en Lua notamment, cf [Chisels User Guide](http://www.sysdig.org/wiki/chisels-user-guide/)



# Conclusion, notes finales, questions

C'est cool \o/

Un seul outil et plus 36 outils différents avec des interfaces et options différentes.

Analyse *on-line* ou *off-line*. Fouillage en profondeur.
