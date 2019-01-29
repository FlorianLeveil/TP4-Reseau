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

```
   **expliquer le(s) changement(s**
<!--stackedit_data:
eyJoaXN0b3J5IjpbNDg1MjQ4NDE4LC0yMTY1MjA0NzAsLTMyNj
QzMDE2NSwtMTk2NjcwODg3OSwyMTM2ODA5MjUyLDczMDk5ODEx
Nl19
-->