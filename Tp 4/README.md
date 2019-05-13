Projet final Sécu
# Sommaire
* [I. Mon project](#i-Mon-project)
* [II. Maquette](#ii-Maquette)
* [III. Point sécu](#iii-Point-sécu)
* [IV. Les Materiaux](#iv-Les-Materiaux)


# I. Mon project
## Pourquoi j'ai choisi la sécu ?
J'ai choisi de parler de securiter comme projet final car c'est un sujet qui m'intéresse vachement en ce moment.

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
Explication des points secu

Firewall

pour le firewall j'ai choisi d'installer pfSense car ça mise en place est clair et rapide (moins de 30 minutes) grâce au fait qu'il est open-source et donc une communauté assez ouverte reduissant fortement
les problèmes que l'on peut rencontrer grâce au nombre faramineux de tuto pour pfSense mais aussi grâce à l'interphace web qui est pour moi super détailler et simple d'utilisation

SPOF(single point of failure)
le spof(single point of failure) est un point d'un système informatique dont le reste du système est dépendant et
dont une panne entraîne l'arrêt complet du système.

Lors de la conception de notre maquette j'ai fait tres attention a avetier les spof en doublant les éléments sensible à devenir des futurs spof
(routeur, switch et firewall).

VLANS
les VLANs(Virtual Local Area Network) perment de créer un ensemble logique isolé améliorant la sécurité deplus de séparer des départements où des groupes de travail grâce au vlan on Optimise la bande passante.
 Les VLANs amènent aussi un plus à la sécuriter des machines car les attaque utilisant le broadcast(ARP cache poisoning, DHCP spoofing, attaque smurf, MAC table overflow…) seront contenues au sein du vlan et ne sétaleront pas 


DMZ
Les services susceptibles d'être accédés depuis Internet seront situés en DMZ tels qu'un serveur web, un dns, etc,
ce qui permeteras qu’en cas de compromission d’un des services dans la DMZ, le pirate n’aura accès qu’aux machines de la DMZ.
Pour securiser encore plus l'architecture j'ai utiliser deux firewall pour  créer la DMZ.
Le premier laisse passer uniquement le trafic vers la DMZ. Et le second n’autorise que le trafic entre la DMZ et le réseau interne.

Authentification forte sur les équipements:

J'ai fait très attention à ce que chaque pc ne possede qu'un seul compte avec mots de passe et que n'existe aucun pc possedant le meme compte.
Mais aussi que l’accès à certaines ressources juger sensible pour l'entreprise, ne soient accessible que par un certain utilisateur ayant le bon compte et l'équipement physique adéquat.
(ex: seul le pc Client 6 a le droit de naviguer librement sur le web sans restrision sur les pages ou il navigue ou bien les telechargement qu'il peut faire)

