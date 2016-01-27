
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


## Installation


## Exemples simples, puis cool !



# Et avec des containers ?



# csysdig



# Aller plus loin ?




# Conclusion, notes finales, questions
