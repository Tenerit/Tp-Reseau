# TP 1

Premier TP un peu tranquille pour se remettre dans le bain. Au programme : 
* des VMs (avec du SSH)
* du routage
* revue de IP, MAC, ports, ARP

## Sommaire
* [I. Exploration du réseau d'une machine CentOS](#i-exploration-du-réseau-dune-machine-centos)
    * [1. Mise en place](#1-mise-en-place)
        * [Configuration](#configuration)
    * [2. Basics](#2-basics)
        * [Routes](#routes)
        * [Table ARP](#table-arp)
        * [Capture Reseau](#capture-reseau)
* [II. Communication simple entre deux machines](#ii-communication-simple-entre-deux-machines)
    * [1. Mise en place](#1-mise-en-place-1)
    * [2. Basics](#2-basics-1)
        * [Ping && ARP](#ping-&&-arp)
        * [UDP](#udp)
        * [TCP](#tcp)
        * [Firewall](#firewall)
* [III. Routage statique simple](#iii-routage-statique-simple)


Machine | `net1` | `net2`
--- | --- | ---
PC | `10.1.1.1/24` | `10.1.2.1/30`
VM (`client1.tp1.b2`) | `10.1.1.2/24` | `10.1.2.2/30`


# I. Exploration du réseau d'une machine CentOS

## 1. Mise en place 

* Combien y a-t-il d'adresses disponibles dans un `/24` ?
   * Il y a 254 adresses disponibles

* Combien y a-t-il d'adresses disponibles dans un `/30` ?
   * Il y a 2 adresses disponibles
  
* Quelle est l'utilité d'un /30 ?
   * Un /30 n'a que 2 adresses disponibles il est donc facile de voir qui est sur ce réseau de manière claire

* NAT : 
    * `curl google.com` 
    * le code d'erreur 301 signifie une **redirection permanente**.
    
* NET1 : 
    * `ping 10.1.1.1`
    * ça fonctionne
    
* NET2 : 
    * `ping 10.1.2.1`
    * ça fonctionne

### Configuration

## 2. Basics

### Routes

* Afficher les routes :

    ```
    ip route show

    default via 10.0.2.2 dev enp0s3 proto static metric 100 
    10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
    10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.2 metric 100 
    10.1.2.0/30 dev enp0s9 proto kernel scope link src 10.1.2.2 metric 100 
    ```
    * La première route la route par défault qui va permettre l'accès au réseau `n'importe lequel`.

    * La route route est la route via  enp0s3 (NAT) qui va permettre l'accès à `10.0.2.0/24 `.

    * La troisième route est la route via la enp0s8 (net1) qui va permettre l'accès à `10.1.1.0/24 `.

    * La quatrième route est la route via enp0s9 (net2) qui va permettre l'accès à `10.1.2.0/30 `.
  
* Supprimer une route :

    `sudo ip route del 10.1.2.0/30`

    ```
    ping 10.1.2.1

    PING 10.1.2.1 (10.1.2.1) 56(84) bytes of data.
    From 10.1.1.2 icmp_seq=1 Destination Host Unreachable
    ```
    * Elle ne fonctionne plus, on ne peut plus utiliser le reseau concerné

* Remettre une route :

    `sudo ip route add 10.1.2.0/30 via 10.1.2.2 dev enp0s9`

    ```
    ip route show 

   default via 10.0.2.2 dev enp0s3 proto static metric 100 
    10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100 
    10.1.1.0/24 dev enp0s8 proto kernel scope link src 10.1.1.2 metric 100 
    10.1.2.0/30 dev enp0s9 proto kernel scope link src 10.1.2.2 metric 100 
    ```
    * On voit bien que la route `10.1.2.0/30` c'est bien rajouter.

### Table ARP

* Afficher les voisins que connaît notre machine = la table ARP :

    ```
    ip neigh show

    10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 REACHABLE
    ```

    * Cette ligne veut dire que j'ai déjà discuté avec la machine en `10.1.1.0` qui est sur le même réseau.

* Vider la table ARP :

    `sudo ip neigh flush all`

    * Quand l'on fait un `ip neigh show`il n'y a plus rien.

* Effectuer une requête simple vers l'hôte :

    `ping 10.1.1.2`

    ```
    ip neigh show

    10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 REACHABLE
    ```

### Capture réseau

* On capture 10 packets

    ```
    sudo tcpdump -i enp0s9 -w ping.pcap 
    tcpdump: listening on enp0s9, link-type EN10MB (Ethernet), capture size 262144 bytes
    ^C10 packets captured
    10 packets received by filter
    0 packets dropped by kernel
    ```



     [Voir ping.pcap](/Tp%201/pcap/ping.pcap)
     
     
     ![alt text](/Tp%201/screens/ping.PNG "Whireshark")

## Communication simple entre deux machines

## 1. Mise en place

Machine | `net1` | `net2`
--- | --- | ---
PC | `10.1.1.1` | `10.1.2.1`
`client1.tp1.b2` | `10.1.1.2` | `10.1.2.2`
`client2.tp1.b2` | `10.1.1.3` | X

* Nouveau clone de VM 

    * Configuration d'une nouvelle ip statique
    ```
    [tener@client2 ~]$ cat /etc/sysconfig/network-scripts/ifcfg-enp0s8
    IPADDR=10.1.1.3
    NETMASK=255.255.255.0
    ```
    ```
        ifdown enp0s8
        ifup enp0s8
    ```

    * Changement du nom de domaine : 
    ```
    sudo hostname client2.tp1.b2
    ```

    * Edition du fichier hosts de client2 :
    ```
    [tener@client2 ~]$ cat /etc/hosts
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    10.1.1.3    localhost client2.tp1.b2
    ```

## 2. Basics

### Ping && ARP

* Vider les deux tables ARP des deux machines:

    ```
    sudo ip neigh flush all
    ```

* Ping **client2** depuis **client1** :

    ```
    ping -c 4 client2 (10.1.1.3)
    ```
    * On observe les changements depuis la table ARP : 

        ```
        tener@localhost ~]$ ip neigh show
        10.1.1.1 dev enp0s8 lladdr 0a:00:27:00:00:06 REACHABLE
        10.1.1.3 dev enp0s8 lladdr 08:00:27:26:2e:dc REACHABLE
        ```
        
        
    [Voir ping-2.pcap](/Tp%201/pcap/ping-2.pcap)
    
    
    ![alt text](/Tp%201/screens/ping-2.PNG "Whireshark2")
    
### UDP

* Sur client1 :

    * Ouverture du port 8888 : 

        ```
        sudo firewall-cmd --add-port=8888/udp --permanent
        [sudo] password for tener:
        ****
        success
        ****
        sudo firewall-cmd --reload
        success
        ```

    * On lance netcat pour qu'il écoute sur le port UDP 8888 : 

        ```
        nc -u -l 8888

        ```

* Sur client2 : 

    ```
    nc -u 10.1.1.2 8888

    ```

    ![alt text](/Tp%201/img/udp.PNG "udp")

* Sur client1 (2nd shell ) : 

    ```
    ss -unp
    Recv-Q Send-Q Local Address:Port               Peer Address:Port              
    0      0      10.1.1.2:8888               10.1.1.3:43841              users:(("nc",pid=1512,fd=4))
    ```

* Sur client2 (2nd shell) : 

    ```
    ss -unp
    Recv-Q Send-Q Local Address:Port               Peer Address:Port              
    0      0      10.1.1.3:43841             10.1.1.2:8888                users:(("nc",pid=1494,fd=3))
    ```

* Sur le client1 (3eme shell) : 

    ```
    sudo tcpdump -i enp0s8 -w nc-udp.pcap
    tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
    ^C6 packets captured
    6 packets received by filter
    0 packets dropped by kernel
    ```

    [Voir nc-udp.pcap](/Tp%201/pcap/nc-udp.pcap)

    ![alt text](/Tp%201/img/nc-udp.PNG "nc-udp")

    * Il y a bien une transmission de données faites entre un client et un serveur par le protocole UDP.


### TCP 

* Sur client1 :

    * Ouverture du port 8888 : 

        ```
        sudo firewall-cmd --add-port=8888/tcp --permanent
        [sudo] password for tener:
        ****
        success
        ****
        sudo firewall-cmd --reload
        success
        ```

 * On lance netcat pour qu'il écoute sur le port TCP 8888 : 

        ```
        nc -l 8888

        ```

* Sur client2 : 

    ```
    nc 10.1.1.2 8888

    ```

* Sur client1 (2nd shell) : 

    ```
    ss -tnp
    State       Recv-Q Send-Q Local Address:Port               Peer Address:Port              
    ESTAB       0      0      10.1.1.2:8888               10.1.1.3:45716              users:(("nc",pid=1474,fd=5))
    ESTAB       0      0      10.1.1.2:22                 10.1.1.1:50020              
    ESTAB       0      0      10.1.1.2:22                 10.1.1.1:50071  
    ```

* Sur client2 (2nd shell) : 

    ```
    ss -tnp
    State       Recv-Q Send-Q Local Address:Port               Peer Address:Port              
    ESTAB       0      0      10.1.1.3:45716             10.1.1.2:8888                users:(("nc",pid=1190,fd=3))
    ESTAB       0      0      10.1.1.3:22                 10.1.1.1:50072              
    ESTAB       0      0      10.1.1.3:22                 10.1.1.1:50021 
    ```

* Sur client1 (3eme shell) : 

    ```
    sudo tcpdump -i enp0s8 -w nc-tcp.pcap 
    tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
    ^C15 packets captured
    15 packets received by filter
    0 packets dropped by kernel
    ```

    [Voir nc-tcp.pcap](/Tp%201/pcap/nc-tcp.pcap)

    ![alt text](/Tp%201/img/nc-tcp.PNG "nc-tcap")

    * Ici nous avons des requêtes TCP qui passe par un tunnel cette fois-ci et nous avons un 'accusé de réception' a contrario du protocole UDP

### Firewall

* Sur client1 :

    * Fermeture du port 8888 UDP : 

        ```
        sudo firewall-cmd --remove-port=8888/udp --permanent
        [sudo] password for tener:
        success
        ****
        ```
        On reload pour que les changengements soient pris en compte
        ```
        sudo firewall-cmd --reload
        success
        ```

    * On lance netcat sur le port TCP 8888 : 

        ```
        nc -u -l 8888

        ```

* Sur client2 : 

    ```
    nc -u 10.1.1.2 8888

    ```

* Sur client1 (2nd shell) : 

    ```
    sudo tcpdump -i enp0s8 -w firewall.pcap 
    tcpdump: listening on enp0s8, link-type EN10MB (Ethernet), capture size 262144 bytes
    ^C10 packets captured
    10 packets received by filter
    0 packets dropped by kernel
    ```
    
    [Voir firewall.pcap](/Tp%201/pcap/firewall.pcap)
    
    
    ![alt text](/Tp%201/img/firewall.PNG "firewall-tcap")
    
    * Le firewall a bien empecher la connexion

# III. Routage statique simple 

* Sur client1 : 

On transforme client1 en routeur 

`sudo sysctl -w net.ipv4.ip_forward=1
 net.ipv4.ip_forward = 1`

* Sur client2 : 

Ajouter d'une route statique sur net2 :

`sudo nano /etc/sysconfig/network-scripts/route-enp0s9`

```
10.1.2.0/30 via 10.1.1.2 dev enp0s9
```

<!--(1)QY1_-7n -->

ou en faisant un :

`sudo ip route add 10.1.2.0/30 via 10.1.2.2 dev enp0s9`

ensuite on teste avec un ping

`ping 10.1.2.2
PING 10.1.2.2 (10.1.2.2) 56(84) bytes of data.
64 bytes from 10.1.2.2: icmp_seq=1 ttl=64 time=0.922 ms
64 bytes from 10.1.2.2: icmp_seq=2 ttl=64 time=0.529 ms

`



<!-- Made by Ours Curieux -->