Projet final Sécu
# Sommaire
* [I. Mon project](#i-Mon-project)
* [II. Maquette](#ii-Maquette)
* [III. Point sécu](#iii-Point-sécu)
* [IV. Les Materiaux](#iv-Les-Materiaux)


# I. Mon project
##Pourquoi j'ai choisi la sécu ?
J'ai choisi de parler de securiter comme projet final car c'est c'est un sujet qui m'interresse vachement en ce moments.

Nous avons tous simplement choisis ce sujet pour mettre en pratique les notions que l'on a pu voir
à travers les différents TPs sur une infra réelle.
# II. Maquette
Cette architecture a été conçu pour des petits entreprises (3 à 6 personnes)

![Shema](/Tp%204/screens/Maquette.PNG)

On peut remarquer que notre architecture comporte:

2 serveurs

6 Pc Clients

5 switchs

4 routeurs (dont 2 avec firewall)

1 firewall frontal de type pfsense 

1 Nat

# III. Point sécu
explication des points secu

Firewall

pour le firewall j'ai choisi d'installer pfSense car ça mise en place et claire et rapide (moins de 30 minutes) grace au faite qu'il est Open-source et donc une communauté asser ouverts reduissant fortement
les problemes que l'on peut faire voir recontrer grace au nombre faramineux de tuto pour setup pfSense mais aussi grace à l'interphace web qui est super détailler simple d'utilisation

SPOF


DMZ

DOUBLE CABLE

