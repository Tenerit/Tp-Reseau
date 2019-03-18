<!--(2)PIUoCH-->
## Sommaire

* [I. Mise en place du lab](#i-mise-en-place-du-lab)
   * [1. Création des VMs et adressage IP](#1-création-des-vms-et-adressage-ip)
   * [2. Routage statique](#2-routage-statique)
   * [3. Visualisation du routage avec Wireshark](#3-visualisation-du-routage-avec-wireshark)
* [II. NAT et services d'infra](#ii-nat-et-services-dinfra)
   * [1. Mise en place du NAT](#1-mise-en-place-du-nat)
   * [2. Serveur DHCP](#2-dhcp-server)
   * [3. Serveur NTP](#3-ntp-server)
   * [4. Serveur Web](#4-web-server)
  
  


# I. Mise en place du lab

## 1. Création des VMs et adressage IP

   Clones du patron de VM 
    * client1.net1.b2
        * SANS carte NAT
    * server1.net2.b2
        * SANS carte NAT
    * router1.net12.b2
        * AVEC carte NAT
    * router2.net12.b2
        * SANS carte NAT


### Tableau d'adressage IP

Machine | `net1` | `net2` | `net12`
--- | --- | --- | ---
PC | `10.2.1.1/24` | `10.2.2.1/24` | `10.2.12.1/29`
`client1.net1.b2` | `10.2.1.10/24` | X | X
`server1.net2.b2` | X | `10.2.2.10/24` | X
`router1.net12.b2` | `10.2.1.254/24` | X | `10.2.12.2/29`
`router2.net12.b2` | X | `10.2.2.254/24` | `10.2.12.3/29`

### Shéma joli
```
        router1.net12.b2                   router2.net12.b2
            +-----+                             +-----+
            |     |10.2.12.2/29     10.2.12.3/29|     |
            |     +-----------------------------+     |
            +-----+                             +-----+
   10.2.1.254/24|                                   |10.2.2.254/24
                |                                   |
                |                                   |
                |                                   |
                |                                   |
                |                                   |
    10.2.1.10/24|                                   |10.2.2.10/24
            +-----+                             +-----+
            |     |                             |     |
            |     |                             |     |
            +-----+                             +-----+
        client1.net1.b2                   server1.net2.b2
```

* 3 réseaux host only 
    * net1 : ```10.2.1.0/24```
    
    * net2 : ```10.2.2.0/24```

    * net12 : ```10.2.12.0/29```

### Checklist IP VMs

Exemple pour client1

- Definition ip statique
    ```
    NAME=enp0s8
    DEVICE=enp0s8

    BOOTPROTO=static
    ONBOOT=yes

    IPADDR=10.2.1.10
    NETMASK=255.255.255.0
    ```

- Definition du hostname 
    ```
    sudo hostname client1
    ```

- Remplissage de /etc/hosts
    ```
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    10.2.2.10   server1 server1.net2.b2
    10.2.1.254  router1 router1.net1.b2
    10.2.2.254  router2 router2.net2.b2
    ```



## 2. Routage statique

    Activation de l'IPv4 forwarding Sur routeur1 et routeur 2
	
	sudo sysctl -w net.ipv4.conf.all.forwarding=1


	On ajoute les routes statiques
	
	sur routeur 1
	sudo nano /etc/sysconfig/network-scripts/route-enp0s9
	10.2.2.0/24 via 10.2.12.3 dev enp0s9

	sur routeur 2
	sudo nano /etc/sysconfig/network-scripts/route-enp0s9
	10.2.1.0/24 via 10.2.12.2 dev enp0s9


	sur client1
	sudo nano /etc/sysconfig/network-scripts/route-enp0s8
	10.2.2.0/24 via 10.2.1.254 dev enp0s8


	sur serveur1
	sudo nano /etc/sysconfig/network-scripts/route-enp0s8
	10.2.1.0/24 via 10.2.2.254 dev enp0s8

	on lance 4 ping du serveur1 depuis client1 pour tester
	```
	ping -c 4 server1
	PING server1 (10.2.2.10) 56(84) bytes of data.
	64 bytes from server1 (10.2.2.10): icmp_seq=1 ttl=62 time=2.21 ms
	64 bytes from server1 (10.2.2.10): icmp_seq=2 ttl=62 time=2.64 ms
	64 bytes from server1 (10.2.2.10): icmp_seq=3 ttl=62 time=2.43 ms
	64 bytes from server1 (10.2.2.10): icmp_seq=4 ttl=62 time=2.95 ms
	^C

	```

## 3. Visualisation du routage avec Wireshark

   Capture de net12
   sudo tcpdump -i enp0s8 -w ping-enp0s8-r2.pcap 

   [Voir ping-net12.pcap](/Tp%202/pcap/ping-net12.pcap)

   ![alt text](/Tp%202/screens/Scr-Net12.png)
   
   Capture de net2
   sudo tcpdump -i enp0s3 -w ping-enp0s3-r2.pcap

   [Voir ping-net2.pcap](/Tp%202/pcap/ping-net2.pcap)

   ![alt text](/Tp%202/screens/Scr-Net2.png)


On peut voir que les deux capture on une	Adresse Mac differente




# II. NAT et services d'infra

## 1. Mise en place du NAT

   - Routeur1

   ```
   TYPE=Ethernet
	BOOTPROTO=dhcp
	NAME=enp0s3
	DEVICE=enp0s3
	ONBOOT=yes
	ZONE=public
   ```
   ping 8.8.8.8
   PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
	64 bytes from 8.8.8.8: icmp_seq=1 ttl=63 time=15.4 ms
	64 bytes from 8.8.8.8: icmp_seq=2 ttl=63 time=22.1 ms
	64 bytes from 8.8.8.8: icmp_seq=3 ttl=63 time=18.5 ms
	^C
	
	- mise en place des zones:
	echo 'ZONE=public' | sudo tee --append/etc/sysconfig/network-scripts/ifcfg-enp0s8
	echo 'ZONE=internal' | sudo tee --append/etc/sysconfig/network-scripts/ifcfg-enp0s9

	- Configuration du firewall

	sudo firewall-cmd --add-interface=enp0s8 --zone=public --permanent

	sudo firewall-cmd --add-interface=enp0s3 --zone=public --permanent

	sudo firewall-cmd --add-interface=enp0s9 --zone=internal --permanent

	sudo firewall-cmd --add-masquerade --zone=public --permanent

	sudo firewall-cmd --reload

	- mise en place des routes de default

	sur routeur 2:
	sudo ip route add default via 10.2.12.2 dev enp0s9

	sur client1: 
	sudo ip route add default via 10.2.1.254 dev enp0s3

## 2. Serveur DHCP

On va reconfigurer le nom de la machine (/etc/hostname)
echo "dhcp-server.net1.b2" | sudo tee /etc/hostname
sudo nano /etc/dhcp/dhcpd.conf

Installation :

sudo yum install -y dhcp
/////
modification du fichier /ect/dhcp/dhcpd.conf

```
# dhcpd.conf

# option definitions common to all supported networks
option domain-name "net1.tp2";

default-lease-time 600;
max-lease-time 7200;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
log-facility local7;

subnet 10.2.1.0 netmask 255.255.255.0 {
  range 10.2.1.50 10.2.1.70;
  option domain-name "net1.tp2";
  option routers 10.2.1.254;
  option broadcast-address 10.2.1.255;
  option domain-name-servers 8.8.8.8;
}

```
On lance le dhcp
`sudo systemctl start dhcpd`
Teste
Création de la nouvelle vm client2 avec carte host-only net1 comme serveur-dhcp

Configuration du fichier Modification dans `/etc/sysconfig/network-scripts/ifcfg-enp0s8`
    NAME=enp0s8
    DEVICE=enp0s8
    BOOTPROTO=dhcp
    ONBOOT=yes

	ifup/down enp0s8

	sudo dhpclient

	On verifie 
	```
	ip a | grep "inet "
	inet 127.0.0.1/8 scope host lo
	inet 10.2.1.50/24 brd 10.2.1.255 scope global noprefixroute dynamic enp0s8

	ip r s
	default via 10.2.1.254 dev enp0s8 proto static metric 100
	```
## 3. Serveur NTP
-Installation de chrony sur toutes les machines qui utilisent ntp :
`sudo yum install -y chrony`

- Ajout du port 123 en UDP sur toute les machines :
`sudo firewall-cmd --add-port=123/udp --permanent`

- Reload du firewall :
`sudo firewall-cmd --reload`

-Modification du fichier `/etc/chrony.conf` Sur router1 :
```
        server 0.fr.pool.ntp.org
        server 1.fr.pool.ntp.org
        server 2.fr.pool.ntp.org
        server 3.fr.pool.ntp.org
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
allow 10.2.1.0/24
local stratum 10
keyfile /etc/chrony.keys
logdir /var/log/chrony
log measurements statistics tracking
```

* Lancement de chrony :
`sudo systemctl start chronyd`

-Verifation :
```
chronyc tracking
Reference ID    : 33FFC594 (ovh.sqlserver.express)
Stratum         : 3
Ref time (UTC)  : Sun Mar 10 15:37:02 2019
System time     : 0.000114980 seconds fast of NTP time
Last offset     : +0.000336725 seconds
RMS offset      : 0.000482424 seconds
Frequency       : 21.904 ppm fast
Residual freq   : +0.002 ppm
Skew            : 0.046 ppm
Root delay      : 0.046410799 seconds
Root dispersion : 0.003142896 seconds
Update interval : 1032.0 seconds
Leap status     : Normal
```


- Modification du fichier `/etc/chrony.confsur` sur les autres machine pour se synchro sur router1 :
```
server router1.net12.tp2 prefer
initstepslew 20 router1.net12.tp2
allow 10.2.1.254
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
local stratum 10
keyfile /etc/chrony.keys
logdir /var/log/chrony
log measurements statistics tracking
```

 - Lancement de  chrony :
`sudo systemctl start chronyd`

- Vérification : 
```
chronyc tracking
Reference ID    : 7F7F0101 ()
Stratum         : 10
Ref time (UTC)  : Sun Mar 10 15:48:59 2019
System time     : 0.000000000 seconds fast of NTP time
Last offset     : +0.000000000 seconds
RMS offset      : 0.000000000 seconds
Frequency       : 0.000 ppm slow
Residual freq   : +0.000 ppm
Skew            : 0.000 ppm
Root delay      : 0.000000000 seconds
Root dispersion : 0.000000000 seconds
Update interval : 0.0 seconds
Leap status     : Normal
```


	

## 4. Serveur Web

- Installation des paquet pour le serveur web

```
sudo yum install -y epel-release
sudo yum install -y nginx
```

- Ouverture du port 80 en tcp
```
sudo firewall-cmd --add-port=80/tcp --permanent
sucess
```
 - Lancement du serveur web

```
sudo systemctl start nginx

Vérification que le server web soit bien lancé :
```
sudo ss -altnp4
State       Recv-Q Send-Q Local Address:Port               Peer Address:Port              
LISTEN      0      128      *:80                   *:*                   users:(("nginx",pid=2423,fd=6),("nginx",pid=2422,fd=6))
LISTEN      0      128      *:22                   *:*                   users:(("sshd",pid=940,fd=3))
LISTEN      0      100    127.0.0.1:25                   *:*                   users:(("master",pid=1173,fd=13))

- Verification


```
`curl 10.2.2.10`

  <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
    <head>
        <title>Test Page for the Nginx HTTP Server on Fedora</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
        <style type="text/css">
            /*<![CDATA[*/
            body {
                background-color: #fff;
                color: #000;
                font-size: 0.9em;
                font-family: sans-serif,helvetica;
                margin: 0;
                padding: 0;
            }
            :link {
                color: #c00;
            }
            :visited {
                color: #c00;
            }
            a:hover {
                color: #f50;
            }
            h1 {
                text-align: center;
                margin: 0;
                padding: 0.6em 2em 0.4em;
                background-color: #294172;
                color: #fff;
                font-weight: normal;
                font-size: 1.75em;
                border-bottom: 2px solid #000;
            }
            h1 strong {
                font-weight: bold;
                font-size: 1.5em;
            }
            h2 {
                text-align: center;
                background-color: #3C6EB4;
                font-size: 1.1em;
                font-weight: bold;
                color: #fff;
                margin: 0;
                padding: 0.5em;
                border-bottom: 2px solid #294172;
            }
            hr {
                display: none;
            }
            .content {
                padding: 1em 5em;
            }
            .alert {
                border: 2px solid #000;
            }

            img {
                border: 2px solid #fff;
                padding: 2px;
                margin: 2px;
            }
            a:hover img {
                border: 2px solid #294172;
            }
            .logos {
                margin: 1em;
                text-align: center;
            }
            /*]]>*/
        </style>
    </head>

    <body>
        <h1>Welcome t <strong>nginx</strong> on Fedora!</h1>
	<h1> it works</h1>
        <div class="content">
            <p>This page is used to test the proper operation of the
            <strong>nginx</strong> HTTP server after it has been
            installed. If you can read this page, it means that the
            web server installed at this site is working
            properly.</p>

            <div class="alert">
                <h2>Website Administrator</h2>
                <div class="content">
                    <p>This is the default <tt>index.html</tt> page that
                    is distributed with <strong>nginx</strong> on
                    Fedora.  It is located in
                    <tt>/usr/share/nginx/html</tt>.</p>

                    <p>You should now put your content in a location of
                    your choice and edit the <tt>root</tt> configuration
                    directive in the <strong>nginx</strong>
                    configuration file
                    <tt>/etc/nginx/nginx.conf</tt>.</p>

                </div>
            </div>

            <div class="logos">
                <a href="http://nginx.net/"><img
                    src="nginx-logo.png" 
                    alt="[ Powered by nginx ]"
                    width="121" height="32" /></a>

                <a href="http://fedoraproject.org/"><img 
                    src="poweredby.png" 
                    alt="[ Powered by Fedora ]" 
                    width="88" height="31" /></a>
            </div>
        </div>
    </body>
</html>

```
  
  
  
  
  
  
  
  
  
  
  <!-- Made by Ours Curieux -->