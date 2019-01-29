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
PING 10.1.0.254 (10.1.0.254) 56(84) bytes of data.
64 bytes from 10.1.0.254: icmp_seq=1 ttl=64 time=0.340 ms
64 bytes from 10.1.0.254: icmp_seq=2 ttl=64 time=0.369 ms
64 bytes from 10.1.0.254: icmp_seq=3 ttl=64 time=0.341 ms
64 bytes from 10.1.0.254: icmp_seq=4 ttl=64 time=0.332 ms
64 bytes from 10.1.0.254: icmp_seq=5 ttl=64 time=0.357 ms
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
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE3MjM1NzE4MTYsLTE5NjY3MDg4NzksMj
EzNjgwOTI1Miw3MzA5OTgxMTZdfQ==
-->