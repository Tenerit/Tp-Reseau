# Sommaire

* [I. Manipulation de switches et de VLAN](#i-manipulation-de-switches-et-de-vlan)
  * [1. Mise en place du lab](#1-mise-en-place-du-lab)
  * [2. Configuration des VLANs](#2-configuration-des-vlans)
* [II. Manipulation simple de routeurs](#ii-manipulation-simple-de-routeurs)
  * [1. Mise en place du lab](#1-mise-en-place-du-lab-1)
  * [2. Configuration du routage statique](#2-configuration-du-routage-statique)
* [III. Mise en place d'OSPF](#iii-mise-en-place-dospf)
  * [1. Mise en place du lab](#1-mise-en-place-du-lab-2)
  * [2. Configuration de OSPF](#2-configuration-de-ospf)
* [IV. Lab Final](#iv-lab-final)

# I. Manipulation de switches et de VLAN

## 1. Mise en place du lab

Tableau d'adressage

Hosts | `10.1.1.0/24`
--- | ---
`client1.lab1.tp3` | `10.1.1.1/24`
`client2.lab1.tp3` | `10.1.1.2/24`
`client3.lab1.tp3` | `10.1.1.3/24`

#### > Topologie
```
client1           SW1                  SW 2
+----+         +-------+            +-------+
|    +---------+       +------------+       |
+----+         +---+---+            +---+---+
                   |                    |
                   |                    |
                   |                    |
                   |                    |
                +--+-+               +--+-+
                |    |               |    |
                +----+               +----+
               client2               client3
```


Définition d'un ip static dans le réseau host-only ```10.1.2.0/24```
sur client 1 et 2

```
nano /etc/sysconfig/network-scripts/ifcfg-enp0s8
	TYPE=Ethernet
    BOOTPROTO=static
    NAME=enp0s8
    DEVICE=enp0s8
    ONBOOT=yes
    IPADDR=10.1.2.1
    NETMASK=255.255.255.0
```

* On ping les machines pour voir si tout est bon :

   ```
        [tener@client1 ~]$ ping 10.1.1.2
        PING 10.1.1.2 (10.1.1.2) 56(84) bytes of data.
        64 bytes from 10.1.1.2: icmp_seq=1 ttl=64 time=3.24 ms
        64 bytes from 10.1.1.2: icmp_seq=2 ttl=64 time=2.22 ms
        64 bytes from 10.1.1.2: icmp_seq=3 ttl=64 time=1.26 ms
        ^C
        --- 10.1.1.2 ping statistics ---
        3 packets transmitted, 3 received, 0% packet loss, time 2002ms
        rtt min/avg/max/mdev = 1.231/2.486/4.892/1.702 ms
        ```

ne pas oublier de redemarer les interphace avec if up et down 

Changement de nom de domaine sur chaque Vm : 
sudo hostname le nom de la vm(ex:pour client1=>client1)

## 2. Configuration des VLANs


```
client1           SW1                  SW 2
+----+  VLAN10 +-------+    TRUNK   +-------+
|    +---------+       +------------+       |
+----+         +-------+            +-------+
                   |VLAN20              |VLAN10
                   |                    |
                   |                    |
                   |                    |
                +--+-+               +--+-+
                |    |               |    |
                +----+               +----+
               client2               client3


```

* Configuration du Switch1 :

* Vlan 10

```
SW1#conf t
SW1(config)#vlan 10
SW1(config-vlan)#name VLAN10
SW1(config-vlan)#exit

SW1(config)#interface Ethernet 0/0
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 10
```
* Vlan 20

```
SW1#conf t
SW1(config)#vlan 20
SW1(config-vlan)#name VLAN20
SW1(config-vlan)#exit

SW1(config)#interface Ethernet 0/1
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 20
```

* Configuration Switch2 : 
```
SW2#conf t
SW2(config)#vlan 10
SW2(config-vlan)#name VLAN10
SW2(config-vlan)#exit

SW2(config)#interface Ethernet 0/0
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 10
```

* Ajout des TRUNK : 

connection de Switch 1 à Switch2 depuis Switch1

```
SW1#conf t
SW1(config-if)#interface Ethernet 0/2
SW1(config-if)#switchport trunk encapsulation dot1q
SW1(config-if)#switchport mode trunk
```
connection de Switch 1 à Switch2 depuis Switch2

SW2#conf t
SW2(config-if)#interface Ethernet 0/1
SW2(config-if)#switchport trunk encapsulation dot1q
SW2(config-if)#switchport mode trunk

# II. Manipulation simple de routeur
#### > Topologie

# 1. Mise en place du lab
```
                           10.2.12.0/30

                  router1                router2
client1          +------+               +------+
+----+.10        |      |.1           .2|      |.254     .10+----+
|    +-----------+      +---------------+      +------------+    |
+----+           +------+               +------+            +----+
                     |.254                                  server1
                     |
                     |
                     |
   10.2.1.0/24       |                         10.2.2.0/24
                     |.11
                  +----+
                  |    |
                  +----+
                  client2

```

#### > Réseau(x)

Nom | Adresse
--- | ---
`lab2-net1` | `10.2.1.0/24`
`lab2-net2` | `10.2.2.0/24`
`lab2-net12` | `10.2.12.0/30`

#### > Tableau d'adressage

Hosts | `lab2-net1` |  `lab2-net2` |  `lab2-net12` 
--- | --- | --- | ---
`client1.lab2.tp3` | `10.2.1.10/24` | x | x
`client2.lab2.tp3` | `10.2.1.11/24` | x | x
`server1.lab2.tp3` | x | `10.2.2.10/24` | x
`router1.lab2.tp3` | `10.2.1.254/24` | x | `10.2.12.1/30`
`router2.lab2.tp3` | x | `10.2.2.254/24` | `10.2.12.2/30`

## 2. Configuration du routage statique

* Ajout des routes


* Sur server1
```
sudo ip route add 10.2.1.0/24 via 10.2.2.254 dev enp0s3
```

* Sur client2

```
sudo ip route add 10.2.2.0/24 via 10.2.2.254 dev enp0s3
```

Router1 :

```
sudo ip route 10.2.2.0 255.255.255.0 10.2.12.2
```
Router2 :

```
sudo ip route 10.2.2.0 255.255.255.0 10.2.12.1
```

#### > Vérification
	
	*Client2->Server1 :

	```
    ping 10.2.2.10
        
    PING 10.2.2.10 (10.2.2.10) 56(84) bytes of data.
    64 bytes from 10.2.2.10: icmp_seq=1 ttl=255 time=1.98 ms
    64 bytes from 10.2.2.10: icmp_seq=2 ttl=255 time=1.81 ms
    ^C
	```

## III. Mise en place d'OSPF

# 1. Mise en place du lab

#### > Tableau d'adressage

Hosts | `10.3.100.0/30` | `10.3.100.4/30` | `10.3.100.8/30` | `10.3.100.12/30` | `10.3.100.16/30` | `10.3.100.20/30` | `10.3.101.0/24` | `10.3.102.0/24`
--- | --- | --- | --- | --- | --- | --- | --- | --- 
`client1.lab3.tp3` | x | x | x | x | x | x | `10.3.101.10/24` | x 
`server1.lab3.tp3` | x | x | x | x | x | x | x | `10.3.102.10/24` 
`router1.lab3.tp3` | `10.3.100.1/30` | x | x | x | x | `10.3.100.22/30` | x | `10.3.102.254/24` 
`router2.lab3.tp3` | `10.3.100.2/30` | `10.3.100.4/30` | x | x | x | x | x | x 
`router3.lab3.tp3` | x | `10.3.100.5/30` | `10.3.100.9/30` | x | x | x | x | x 
`router4.lab3.tp3` | x | x | `10.3.100.10/30` | `10.3.100.13/30` | x | x | `10.3.101.254/24` | x 
`router5.lab3.tp3` | x | x | x | `10.3.100.14/30` | `10.3.100.17/30` | x | x | x 
`router6.lab3.tp3` | x | x | x | x | `10.3.100.18/30` | `10.3.100.21/30` | x | x 

# 2. Configuration de OSPF
* Pour chaque routeur:

	* Activation de l'ospf  : 

	```
	(config)# router ospf 1
	```

	* Attribution d'une router-id :
	```
	# router-id 1.1.1.1
	```

    * Partage de tous les réseaux sur lequelle le routeur est connecté :
        ```
           network 10.3.100.0 0.0.0.3 area 0
           network 10.3.102.0 0.0.0.255 area 2
        ```

## IV. Lab Final

[Shema](/Tp%203/screens/Shema.png)


##conv des vm
```
sudo /etc/sysconfig/network-scripts/ifcfg-enp0s8

NAME=enp0s8
DEVICE=enp0s8

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.33.10.1
NETMASK=255.255.255.0
```
et son nom de domaine :

```
echo 'client1.lab3.tp4' | sudo tee /etc/hostname
reboot
```

## configuration de base des routeurs:

```
# conf t
(config)# interface FastEthernet <NUMERO>
(config-if)# ip address <IP> <MASK>
(config-if)# no shut
(config-if)# exit
(config)# exit
# show ip int br
```

##Mise en place de l'ospf :

* Activation de OSPF sur chaque router(R1,R2;R3) :
    ```
    (config)# router ospf 1
    ```
* Définition d'un **router-id** pour chaque routeur :

        
        R1(config-router)# router-id 1.1.1.1

        R2(config-router)# router-id 2.2.2.2

        R3(config-router)# router-id 3.3.3.3
        

* Partage de réseaux : 

        * Exemple sur router1 : 
        
            
            R1(config-router)#network 10.3.101.0 0.0.0.255 area 0
			R1(config-router)#network 10.3.102.0 0.0.0.255 area 0
            R1(config-router)#network 10.3.103.0 0.0.0.255 area 0
			R1(config-router)#network 10.3.104.0 0.0.0.255 area 0
            

* Ajout de gateway pour que les clients et le server se ping :

    * Sur client3 && server1 :

        
		GATEWAY=10.3.103.22
        GATEWAY=10.3.104.21
        

    * Sur client1 && client2 : 

        ```
        GATEWAY=10.3.101.22
        GATEWAY=10.3.102.22
        ```

* Test de ping entre les machines :

        * client1 vers server1 : 

           
            [tener@client1 ~]$ ping 10.3.104.20
            PING 10.3.102.10(10.3.102.20) 56(84) bytes of data.
            64 bytes from 10.3.102.20: icmp_seq=1 ttl=255 time=2.62 ms
            64 bytes from 10.3.102.20: icmp_seq=2 ttl=255 time=10.2 ms
            64 bytes from 10.3.102.20: icmp_seq=3 ttl=255 time=7.15 ms
            64 bytes from 10.3.102.20: icmp_seq=4 ttl=255 time=11.6 ms
            64 bytes from 10.3.102.20: icmp_seq=5 ttl=255 time=8.4 ms
            ^C
            
			cl1 10.3.101.20
			cl2 10.3.102.20
			cl3 10.3.103.20
			serv 10.3.104.20


        * server1 vers client1 : 
  
            [tener@server1 ~]$ ping 10.3.101.20
            PING 10.3.101.10(10.3.101.20) 56(84) bytes of data.
            64 bytes from 10.3.101.20: icmp_seq=1 ttl=255 time=3.4 ms
            64 bytes from 10.3.101.20: icmp_seq=2 ttl=255 time=5.00 ms
            64 bytes from 10.3.101.20: icmp_seq=3 ttl=255 time=2.69 ms
            ^C
           
- Configuration de base des routeurs:

``` 
# conf t
(config)# interface FastEthernet <NUMERO>
(config-if)# ip address <IP> <MASK>
(config-if)# no shut
(config-if)# exit
(config)# exit
# show ip int br
```

## Configuration du Nat

	* Configuration IP sur router1 : 

        ```
        R1#conf t
        R1(config)#interface FastEthernet0/0          
        R1(config-if)#ip address dhcp
        R1(config-if)#no shut
        ```

    * Vérification : 

        ```
        R1#show ip int br
        Interface                  IP-Address      OK? Method Status                Protocol
        => FastEthernet0/0         192.168.122.251 YES DHCP   up                    up
        FastEthernet1/0            10.3.1.9        YES NVRAM  up                    up
        FastEthernet2/0            10.3.1.1        YES NVRAM  up                    up
        FastEthernet3/0            unassigned      YES NVRAM  administratively down down
        NVI0                       unassigned      NO  unset  up                    up
        ```
		FastEthernet0/0 possede une Ip et est configurer en DHCP




## Configuration des switchs 

*Configuration du switch 1 : 

	* Acces entre le switch1 et client1 / client2 / client3 :

        ```
        Switch1(config)#vlan 10
        Switch1(config-vlan)#name client1
        Switch1(config-vlan)#exit

        Switch1(config)#vlan 20
        Switch1(config-vlan)#name client2
        Switch1(config-vlan)#exit

		Switch1(config)#vlan 20
        Switch1(config-vlan)#name client3
        Switch1(config-vlan)#exit

        Switch1(config)#interface Ethernet0/1
        Switch1(config-if)#switchport mode access
        Switch1(config-if)#switchport access vlan 1
        Switch1(config-if)#exit

		Switch1(config)#interface Ethernet0/2
        Switch1(config-if)#switchport mode access
        Switch1(config-if)#switchport access vlan 20
        Switch1(config-if)#exit

        Switch1(config)#interface Ethernet0/3
        Switch1(config-if)#switchport mode access
        Switch1(config-if)#switchport access vlan 20
        Switch1(config-if)#exit
        ```
	* trunk entre **router2** et **switch1** : 

        ```
        Switch1(config)#interface Ethernet0/0
        Switch1(config-if)#switchport trunk encapsulation dot1q
        Switch1(config-if)#switchport mode trunk
        Switch1(config-if)#exit
        ```
* On fait la meme pour le switch2 :

    * Acces entre switch2 et server1 :

    ```
    Switch2(config)#interface Ethernet0/1
    Switch2(config-if)#switchport mode access
    Switch2(config-if)#switchport access vlan 10
    Switch2(config-if)#exit

    Switch2(config)#interface Ethernet0/0
    Switch2(config-if)#switchport trunk encapsulation dot1q
    Switch2(config-if)#switchport mode trunk
	Switch2(config-if)#exit
    ```

	* Ok Maintenant on passe a l'interVlan :


    * Sur router3 :

        D'abord nous supprimons l'**IP** que nous avons définie dans le réseau des Vms

        ```
        R3#conf t
        R3(config)#interface FastEthernet1/0
        R3(config-if)#no ip address
        R3(config-if)#exit
        ```

    * On crée une sous interface sur **router2** et **router3** pour chaque **vlan** :

        * Sur router3 :

            ```
			//Définition des Ips
            R3(config)#interface FastEthernet1/0.10
            R3(config-subif)#encap dot1Q 10 
            R3(config-subif)#ip add 10.3.101.22 255.255.255.0 
            R3(config-subif)#no shut
            R3(config-subif)#exit

			R3(config)#interface FastEthernet1/0.20
            R3(config-subif)#encap dot1Q 20
            R3(config-subif)#ip add 10.3.102.22 255.255.255.0
			R3(config-subif)#ip add 10.3.103.22 255.255.255.0
            R3(config-subif)#no shut
            R3(config-subif)#exit
            ```

        * Sur router2 : 

            ```
            R2(config)#interface FastEthernet2/0.30
            R2(config-subif)#encap dot1Q 30 
            R2(config-subif)#ip add 10.3.104.21 255.255.255.0
            R2(config-subif)#no shut
            R2(config-subif)#exit
            ```



## 5. Installation du service d'infra

* Installation de **nginx** sur **server1** :

    ```
    sudo yum install nginx -y
    ```

* On ouvre le port 80 en **tcp** :

    ```
    sudo firewall-cmd --add-port=80/tcp --permanent
    sudo firewall-cmd --reload
    ```

* On lance le server

	```
    sudo systemclt start nginx
    ```

	Curl du site de server1 depuis client1 :

	```
	curl 10.3.11.1
	```


