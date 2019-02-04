# TP 4 - Spéléologie réseau : descente dans les couches
# I. Mise en place du lab
## 1/2 Création des réseaux et création des VMs
Configuration VM Client:
```
[root@vm1 ~]# ip route
10.1.0.0/24 dev enp0s3 proto kernel scope link src 10.1.0.10 metric 100
```
Configuration VM Serveur:
```
[root@vm2 ~]# ip route
10.2.0.0/24 dev enp0s3 proto kernel scope link src 10.2.0.10 metric 100
```
Configuration VM Routeur:
```
[root@vm3 ~]# ip route
10.1.0.0/24 dev enp0s3 proto kernel scope link src 10.1.0.254 metric 100
10.2.0.0/24 dev enp0s8 proto kernel scope link src 10.2.0.254 metric 101
```

**Nom de domaine:**
VM Client:
```
[root@vm1 ~]# hostname
vm1.tp4
```
VM Serveur:
```
[root@vm2 ~]# hostname
vm2.tp4
```
VM Routeur:
```
[root@vm3 ~]# hostname
vm3.tp4
```

**Ping:**

Client > Routeur:
```
[root@vm1 ~]# ping vm3.tp4
PING vm3.tp4 (10.1.0.254) 56(84) bytes of data.
64 bytes from vm3.tp4 (10.1.0.254): icmp_seq=1 ttl=64 time=0.346 ms
64 bytes from vm3.tp4 (10.1.0.254): icmp_seq=2 ttl=64 time=0.408 ms
64 bytes from vm3.tp4 (10.1.0.254): icmp_seq=3 ttl=64 time=0.360 ms
64 bytes from vm3.tp4 (10.1.0.254): icmp_seq=4 ttl=64 time=0.361 ms
```
Serveur > Routeur:
```
[root@vm2 ~]# ping vm3.tp4
PING vm3.tp4 (10.2.0.254) 56(84) bytes of data.
64 bytes from vm3.tp4 (10.2.0.254): icmp_seq=1 ttl=64 time=0.353 ms
64 bytes from vm3.tp4 (10.2.0.254): icmp_seq=2 ttl=64 time=0.356 ms
64 bytes from vm3.tp4 (10.2.0.254): icmp_seq=3 ttl=64 time=0.354 ms
64 bytes from vm3.tp4 (10.2.0.254): icmp_seq=4 ttl=64 time=0.351 ms
```
## 3. Mise en place du routage statique
**Sur  `router1`:** 
* activer l'IPv4 Forwarding:
```
[root@vm3 ~]# sudo sysctl -w net.ipv4.conf.all.forwarding=1
net.ipv4.conf.all.forwarding = 1
```
* désactiver le firewal:
```
[root@vm3 ~]# sudo systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
```
* ip route show:
```
[root@vm3 ~]# ip route show
10.1.0.0/24 dev enp0s3 proto kernel scope link src 10.1.0.254 metric 100
10.2.0.0/24 dev enp0s8 proto kernel scope link src 10.2.0.254 metric 101
```

**Sur  `client1`:**
* ip route show:
 ```
[root@vm1 network-scripts]# ip route show
10.1.0.0/24 dev enp0s3 proto kernel scope link src 10.1.0.10 metric 100
10.2.0.0/24 via 10.1.0.254 dev enp0s3 proto static metric 100
```

**Sur  `serveur1`:**
* ip route show:
 ```
[root@vm2 ~]# ip route show
10.1.0.0/24 via 10.2.0.254 dev enp0s3 proto static metric 100
10.2.0.0/24 dev enp0s3 proto kernel scope link src 10.2.0.10 metric 100
```

**Test:**
* `client1`  doit pouvoir ping  `server1`:
```
[root@vm1 network-scripts]# ping vm2.tp4
PING vm2.tp4 (10.2.0.10) 56(84) bytes of data.
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=1 ttl=63 time=0.567 ms
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=2 ttl=63 time=0.689 ms
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=3 ttl=63 time=0.689 ms
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=4 ttl=63 time=0.651 ms
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=5 ttl=63 time=0.691 ms
```
* `server1`  doit pouvoir ping  `client1`:
```
[root@vm2 ~]# ping vm1.tp4
PING vm1.tp4 (10.1.0.10) 56(84) bytes of data.
64 bytes from vm1.tp4 (10.1.0.10): icmp_seq=1 ttl=63 time=0.537 ms
64 bytes from vm1.tp4 (10.1.0.10): icmp_seq=2 ttl=63 time=0.730 ms
64 bytes from vm1.tp4 (10.1.0.10): icmp_seq=3 ttl=63 time=0.694 ms
64 bytes from vm1.tp4 (10.1.0.10): icmp_seq=4 ttl=63 time=0.568 ms
64 bytes from vm1.tp4 (10.1.0.10): icmp_seq=5 ttl=63 time=0.714 ms
```
**Traceroute:**
* Client > Serveur:
```
[root@vm1 network-scripts]# traceroute vm2.tp4
traceroute to vm2.tp4 (10.2.0.10), 30 hops max, 60 byte packets
 1  vm3.tp4 (10.1.0.254)  0.364 ms  0.273 ms  0.207 ms
 2  vm3.tp4 (10.1.0.254)  0.147 ms !X  0.241 ms !X  0.247 ms !X
```
* Serveur > Client:
```
[root@vm2 ~]# traceroute vm1.tp4
traceroute to vm1.tp4 (10.1.0.10), 30 hops max, 60 byte packets
 1  vm3.tp4 (10.2.0.254)  0.285 ms  0.184 ms  0.221 ms
 2  vm3.tp4 (10.2.0.254)  0.140 ms !X  0.174 ms !X  0.198 ms !X
```
# II. Spéléologie réseau
## 1. ARP
### **A. Manip 1**

**Sur  `client1:`**
  * Afficher la table ARP:
```
[root@vm1 network-scripts]# ip neigh show
10.1.0.1 dev enp0s3 lladdr 0a:00:27:00:00:16 REACHABLE
```
 (C'est l'adresse de Broadcast)
 
**Sur  `server1`:**
 * Afficher la table ARP
```
[root@vm2 ~]# ip neigh show
10.2.0.1 dev enp0s3 lladdr 0a:00:27:00:00:0d REACHABLE
```
 (C'est l'adresse de Broadcast)

**Sur  `client1`**:
 * ping  `server1`:
```
[root@vm1 network-scripts]# ping vm2.tp4
PING vm2.tp4 (10.2.0.10) 56(84) bytes of data.
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=1 ttl=63 time=1.08 ms
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=2 ttl=63 time=0.718 ms
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=3 ttl=63 time=0.732 ms
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=4 ttl=63 time=0.677 ms
```
 * afficher la table ARP
```
[root@vm1 network-scripts]# ip neigh show
10.1.0.254 dev enp0s3 lladdr 08:00:27:88:74:c4 DELAY
10.1.0.1 dev enp0s3 lladdr 0a:00:27:00:00:16 DELAY
```
 Ça enregistre l'ip et la mac, pendant 60 secondes environ. (En l’occurrence c'est l'ip/mac du Routeur)
 
**Sur  `server1`:**
* afficher la table ARP
```
[root@vm2 ~]# ip neigh show
10.2.0.254 dev enp0s3 lladdr 08:00:27:41:1a:1b STALE
10.2.0.1 dev enp0s3 lladdr 0a:00:27:00:00:0d REACHABLE
```
 Ça enregistre l'ip et la mac, pendant 60 secondes environ par ce que la le client à fait un ping. (En l’occurrence c'est l'ip/mac du Routeur)

### **B. Manip 2**

**Sur  `router1`:**
* afficher la table ARP
```
[root@vm3 ~]# ip neigh show
10.1.0.1 dev enp0s3 lladdr 0a:00:27:00:00:16 DELAY
```
C'est l'adresse de broadcast

 **Sur  `client1`:**
   * ping  `server1`
```
[root@vm1 network-scripts]# ping vm2.tp4
PING vm2.tp4 (10.2.0.10) 56(84) bytes of data.
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=1 ttl=63 time=1.18 ms
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=2 ttl=63 time=0.732 ms
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=3 ttl=63 time=0.739 ms
```

**Sur  `router1`:**
  * afficher la table ARP
```
[root@vm3 ~]# ip neigh show
10.1.0.10 dev enp0s3 lladdr 08:00:27:06:2e:31 STALE
10.2.0.10 dev enp0s8 lladdr 08:00:27:94:ae:9d STALE
10.1.0.1 dev enp0s3 lladdr 0a:00:27:00:00:16 REACHABLE
```
Deux adresses ce sont ajouté c'est normal, la premier c'est l'ip du client qui a envoyé un ping, et la deuxième c'est l'ip du serveur qui à renvoyé un pong.

### **C. Manip 3**

**Sur l'hôte (votre PC)**
 * Afficher la table ARP:
```
PS C:\Users\Florian> arp -a

Interface : 192.168.102.2 --- 0xc
  Adresse Internet      Adresse physique      Type
  192.168.102.255       ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique

Interface : 10.2.0.1 --- 0xd
  Adresse Internet      Adresse physique      Type
  10.2.0.10             08-00-27-94-ae-9d     dynamique
  10.2.0.255            ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 10.1.0.1 --- 0x16
  Adresse Internet      Adresse physique      Type
  10.1.0.10             08-00-27-06-2e-31     dynamique
  10.1.0.254            08-00-27-88-74-c4     dynamique
  10.1.0.255            ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 10.33.2.105 --- 0x17
  Adresse Internet      Adresse physique      Type
  10.33.0.14            7c-76-35-51-ad-8d     dynamique
  10.33.0.68            90-cd-b6-64-bc-e7     dynamique
  10.33.2.41            08-d4-0c-c7-20-04     dynamique
  10.33.3.253           00-12-00-40-4c-bf     dynamique
  10.33.3.254           94-0c-6d-84-50-c8     dynamique
  10.33.3.255           ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique
```
  * Afficher de nouveau la table ARP:
```
Interface : 192.168.102.2 --- 0xc
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.2.0.1 --- 0xd
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.1.0.1 --- 0x16
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.33.2.105 --- 0x17
  Adresse Internet      Adresse physique      Type
  10.33.3.253           00-12-00-40-4c-bf     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
```
 * Afficher encore la table ARP:
```
PS C:\WINDOWS\system32> arp -a

Interface : 192.168.102.2 --- 0xc
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.2.0.1 --- 0xd
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.1.0.1 --- 0x16
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.33.2.105 --- 0x17
  Adresse Internet      Adresse physique      Type
  10.33.0.68            90-cd-b6-64-bc-e7     dynamique
  10.33.2.41            08-d4-0c-c7-20-04     dynamique
  10.33.3.253           00-12-00-40-4c-bf     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
```
  * Expliquer le(s) changement(s):
  Il y a eu de nouvelle connexions enregistrée quelque secondes. (Chaque ligne ce met à jour toute les  secondes environ)
  
### **D. Manip 4**

**Sur  `client1`:**

* afficher la table ARP
```
[root@vm1 network-scripts]# ip neigh show
10.1.0.1 dev enp0s3 lladdr 0a:00:27:00:00:16 REACHABLE
```
* activer la carte NAT
```
3: enp0s10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:20:37:a9 brd ff:ff:ff:ff:ff:ff
    inet 10.0.5.15/24 brd 10.0.5.255 scope global noprefixroute dynamic enp0s10
       valid_lft 86117sec preferred_lft 86117sec
    inet6 fe80::6baa:de39:c1ad:6865/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```
* joindre internet (`curl google.com`  par exemple)
```
[root@vm1 ~]# curl google.com
<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.com/">here</A>.
</BODY></HTML>
```
* afficher la table ARP
```
[root@vm1 ~]# ip neigh show
10.0.5.2 dev enp0s10 lladdr 52:54:00:12:35:02 STALE
10.1.0.1 dev enp0s3 lladdr 0a:00:27:00:00:16 REACHABLE
```

* expliquer le(s) changement(s):

Nous maintenant une ip/mac en plus qui appartient à la nouvelle carte réseau que nous avons activé.
## 2. Wireshark
## A. Interception d'ARP et  `ping`
1. sur  `router1`:
* lancer Wireshark pour enregistrer le trafic
```
[root@vm3 /]# sudo tcpdump -i enp0s3 -w ping.pcap
tcpdump: listening on enp0s3, link-type EN10MB (Ethernet), capture size 262144 bytes
```
2. sur  `client1`:
* Vider la table ARP
```
[root@vm1 ~]# sudo ip neigh flush all
```
* Envoyer 4 pings à  `server1`:
```
[root@vm1 ~]# ping -c 4 vm2.tp4
PING vm2.tp4 (10.2.0.10) 56(84) bytes of data.
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=1 ttl=63 time=1.15 ms
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=2 ttl=63 time=0.696 ms
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=3 ttl=63 time=0.687 ms
64 bytes from vm2.tp4 (10.2.0.10): icmp_seq=4 ttl=63 time=0.687 ms

--- vm2.tp4 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 0.687/0.806/1.155/0.202 ms
```
3. Sur `router1`:
* Vérification de la présence du fichier  `ping.pcap`  avec un  `ls`:
```
[root@vm3 /]# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  ping.pcap  proc  root  run  sbin  srv  sys  tmp  usr  var
```
4. sur l'hôte (votre PC) :

* Ouvrir le fichier  `ping.pcap`  dans Wireshark:

 <img src="./Capture_ping_wirechark.PNG">

## B. Interception d'une communication  `netcat`

1. s'occuper du firewall sur le serveur1:
```
[root@vm2 ~]# firewall-cmd --add-port=5454/tcp --permanent
success
```
2. Videz les tables ARP de tout le monde:
* Serveur1:
```

```
* Client1:
```

```
* Routeur:
```

```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyNjg1NTM5NzYsLTEyMTg4NDk3MjcsND
U5Nzg5NjE5LDIxNjY1NjMyMywtMTI3NzQ2NDUxNSw5NjA2ODYz
ODUsLTE4ODAxOTkzNDcsLTQ3OTg2MTIxLC0yMTY1MjA0NzAsLT
MyNjQzMDE2NSwtMTk2NjcwODg3OSwyMTM2ODA5MjUyLDczMDk5
ODExNl19
-->