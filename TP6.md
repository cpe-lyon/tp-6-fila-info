# Exercice 1. Adressage IP (rappels)  
| Sous-réseau    | @sous réseau | @Broadcast | @Premier Machine | @Dernier machine |
| -----------    | -----------  | -----------| -----------      |-----------       |
1 | 172.16.0.0 | 172.16.0.63 | 172.16.0.1 | 172.16.0.62 
2 | 172.16.0.64 | 172.16.0.127 | 172.16.0.65 | 172.16.0.126
3 | 172.16.0.128 | 172.16.0.191 | 172.16.0.129 | 172.16.0.190
4 | 172.16.0.192 | 172.16.0.255 | 172.16.0.193 | 172.16.0.254
5 | 172.16.1.0 | 172.16.1.1 | 172.16.1.62 | 172.16.1.63
6 | 172.16.1.64 | 172.16.1.127 | 172.16.1.65 | 172.16.1.126
7 | 172.16.1.128 | 172.16.1.159 | 172.16.1.129 | 172.16.1.158

# Exercice 2. Préparation de l’environnement 

2. L'interface "lo" correspond à loop back, la boucle locale.   
3. Il faut utiliser la commande : `sudo apt-get remove cloud-init`  

# Exercice 3. Installation du serveur DHCP  
1. On installe le paquet avec la commande `sudo apt install isc-dhcp-server`  
2.  Pour modifier l'adresse ip de l'interface réseau il faut modifier dans le fichier "/etc/netplan/50-cloud-init.yaml"

3. on fait une copie `sudo cp dhcpd.conf dhcpd.conf.bak`  
Les deux premières lignes correspondent au temps par défaut et au temps maximal du bail en seconde :  
```
default-lease-time 120;
max-lease-time 600;
```

4 - Il faut modifier le fichier "/etc/default/isc-dhcp-server", les lignes suivante:  
```
INTERFACESv4="ens224"
INTERFACESv6="ens224"
```

5. On enregistre `dhcpd -tpuis je le redémarre avec` et redémarre `systemctl restart isc-dhcp-server`. 

6. Pour désinstaller "cloud-init" on fait `sudo apt-get remove cloud-init`  

Pour renomer la machine `sudo hostnamectl set-hostname client.tpadmin.local` 

7.
- **DHCP REQUEST** :La demande d'adresse ip du client au serveur
- **DHC PACK** : La confirmation de la récepetion de la demande du serveur
- **DHCP DISCOVER** : Le serveur trouve une adresse disponible à donner au client
- **DHC POFFER** : La réponse du serveur au client, il lui envoie son adresse IP  

8. **/var/lib/dhcp/dhcpd.leases"** contient les demandes de bails des clients.
`dhcp-lease-list` liste les bails avec leursdates d'expirations.  

9. On utilise la requête ping:  
```
PING 192.168.100.100 (192.168.100.100) 56(84) bytes of data.
64 bytes from 192.168.100.100: icmp_seq=1 ttl=64 time=0.245 ms
64 bytes from 192.168.100.100: icmp_seq=2 ttl=64 time=0.265 ms
64 bytes from 192.168.100.100: icmp_seq=3 ttl=64 time=0.224 ms
64 bytes from 192.168.100.100: icmp_seq=4 ttl=64 time=0.251 ms
```
10. L'adresse de l'interface à bien été changée.

# Exerice 4.  Donner un accès à Internet au client

1. On modifie le fichier config : **/etc/sysctl.conf**, la ligne suivante:  
```
net.ipv4.ip_forward=1
```
On enregistre le changement avec `sudo sysctl -p /etc/sysctl.conf`

2. On aautorise l'adresse source avec : `sudo iptables --table nat --append POSTROUTING --out-interface ens192 -j MASQUERADE`  

```
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=53 time=9.20 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=53 time=8.40 ms
64 bytes from 1.1.1.1: icmp_seq=3 ttl=53 time=8.89 ms
```
# Exercice 5. Installation du serveur DNS

1. On installe binn9 avec `apt install bind9`.

2. On modifie la configuration de bind9 dans **/etc/bind/named.conf.options**, On ajoute:
```
forwarders{
1.1.1.1;
8.8.8.8;
};
```
Il faut redémarrer bind9 pour actualiser la modification.

3 - Le ping de www.google.fr depuis le client fonctionne : 
```
PING www.google.fr (216.58.209.227) 56(84) bytes of data.
64 bytes from par10s29-in-f3.1e100.net (216.58.209.227): icmp_seq=1 ttl=111 time=8.45 ms
64 bytes from par10s29-in-f3.1e100.net (216.58.209.227): icmp_seq=2 ttl=111 time=8.78 ms
64 bytes from par10s29-in-f3.1e100.net (216.58.209.227): icmp_seq=3 ttl=111 time=8.11 ms
64 bytes from par10s29-in-f3.1e100.net (216.58.209.227): icmp_seq=4 ttl=111 time=8.56 ms
```
4 - On utilise lynx pour naviguer sur une page web avec la commande : `lynx fr.wikipedia.fr`
