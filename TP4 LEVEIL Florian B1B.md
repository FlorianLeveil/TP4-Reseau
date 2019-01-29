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

```
VM Serveur:
```

```
VM Routeur:
```

```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxMTkyNzE5MjIsLTE5NjY3MDg4NzksMj
EzNjgwOTI1Miw3MzA5OTgxMTZdfQ==
-->