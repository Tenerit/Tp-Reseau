Projet final S�cu
# Sommaire
* [I. Mon project](#i-Mon-project)
* [II. Maquette](#ii-Maquette)
* [III. Point s�cu](#iii-Point-s�cu)
* [IV. Les Materiaux](#iv-Les-Materiaux)


# I. Mon project
## Pourquoi j'ai choisi la s�cu ?
J'ai choisi de parler de securiter comme projet final car c'est un sujet qui m'int�resse vachement en ce moment.

Nous avons tous simplement choisis ce sujet pour mettre en pratique les notions que l'on a pu voir
� travers les diff�rents TPs sur une infra r�elle.

# II. Maquette
Cette architecture a �t� con�u pour des petits entreprises (3 � 6 personnes)

![Shema](/Tp%204/screens/Maquette.PNG)

On peut remarquer que notre architecture comporte:

2 serveurs

6 Pc Clients

5 switchs

4 routeurs (dont 2 avec firewall)

1 firewall frontal de type pfsense 

1 Nat

# III. Point s�cu
Explication des points secu

Firewall

pour le firewall j'ai choisi d'installer pfSense car �a mise en place est clair et rapide (moins de 30 minutes) gr�ce au fait qu'il est open-source et donc une communaut� assez ouverte reduissant fortement
les probl�mes que l'on peut rencontrer gr�ce au nombre faramineux de tuto pour pfSense mais aussi gr�ce � l'interphace web qui est pour moi super d�tailler et simple d'utilisation

SPOF(single point of failure)
le spof(single point of failure) est un point d'un syst�me informatique dont le reste du syst�me est d�pendant et
dont une panne entra�ne l'arr�t complet du syst�me.

Lors de la conception de notre maquette j'ai fait tres attention a avetier les spof en doublant les �l�ments sensible � devenir des futurs spof
(routeur, switch et firewall).

VLANS
les VLANs(Virtual Local Area Network) perment de cr�er un ensemble logique isol� am�liorant la s�curit� deplus de s�parer des d�partements o� des groupes de travail gr�ce au vlan on Optimise la bande passante.
 Les VLANs am�nent aussi un plus � la s�curiter des machines car les attaque utilisant le broadcast(ARP cache poisoning, DHCP spoofing, attaque smurf, MAC table overflow�) seront contenues au sein du vlan et ne s�taleront pas 


DMZ
Les services susceptibles d'�tre acc�d�s depuis Internet seront situ�s en DMZ tels qu'un serveur web, un dns, etc,
ce qui permeteras qu�en cas de compromission d�un des services dans la DMZ, le pirate n�aura acc�s qu�aux machines de la DMZ.
Pour securiser encore plus l'architecture j'ai utiliser deux firewall pour  cr�er la DMZ.
Le premier laisse passer uniquement le trafic vers la DMZ. Et le second n�autorise que le trafic entre la DMZ et le r�seau interne.

Authentification forte sur les �quipements:

J'ai fait tr�s attention � ce que chaque pc ne possede qu'un seul compte avec mots de passe et que n'existe aucun pc possedant le meme compte.
Mais aussi que l�acc�s � certaines ressources juger sensible pour l'entreprise, ne soient accessible que par un certain utilisateur ayant le bon compte et l'�quipement physique ad�quat.
(ex: seul le pc Client 6 a le droit de naviguer librement sur le web sans restrision sur les pages ou il navigue ou bien les telechargement qu'il peut faire)

