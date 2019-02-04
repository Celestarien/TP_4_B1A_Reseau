# B1 Réseau 2018 - TP4

## TP 4 - Spéléologie réseau : descente dans les couches

### I. Mise en place du lab

#### 1. Création des réseaux

* Création du "réseau 1" :
    * Sur VirtualBox allez dans "Host Network Manager"
    * Cliquez sur "Créer"
    * Dans l'onglet "Adapter", cochez "Configure Adapter Manually"
    * Remplissez la ligne "Adresse IPv4" avec ```« 10.1.0.0 »```
    * Puis dans allez dans l'onglet "Serveur DHCP" et décochez "Activer le serveur""
    * Et enfin cliquez sur "Apply"

* Création du "réseau 2" :
    * Sur VirtualBox allez dans "Host Network Manager"
    * Cliquez sur "Créer"
    * Dans l'onglet "Adapter", cochez "Configure Adapter Manually"
    * Remplissez la ligne "Adresse IPv4" avec ```« 10.2.0.0 »```
    * Puis dans allez dans l'onglet "Serveur DHCP" et décochez "Activer le serveur""
    * Et enfin cliquez sur "Apply"

#### 2. Création des VMs

La **VM cliente (client1)** sera la machine avec l'IP ```« 10.1.0.10 »``` dans ```net1```  
La **VM serveur (server1)** sera la machine avec l'IP ```« 10.2.0.10 »``` dans ```net2```  
La **VM routeur (router1)** sera la machine avec l'IP ```« 10.1.0.254 »``` dans ```net1``` et l'IP ```« 10.2.0.254 »``` dans ```net2```  

#### 3. Mise en place du routage statique

* Sur ```router1``` réalisez les commandes suivantes:
    * ```« sudo sysctl -w net.ipv4.conf.all.forwarding=1 »```
    * ```« sudo systemctl stop firewalld »``` si l'on souhaite que cela soit temporaire
    * ou ```« sudo systemctl disable firewalld »``` si l'on souhaite que cela soit permanent
    * ```« ip route show »```

* Sur ```client1``` réalisez les commandes suivantes :
    * ```« ip route add <Network_adress> via <Local_IP> dev <Local_interface_net1> »```
    *  ```« ip route add <Network_adress> via <Local_IP> dev <Local_interface_net2> »```

* Sur ```server1``` réalisez les commandes suivantes :
    * ```« ip route add <Network_adress> via <Local_IP> dev <Local_interface_net1> »```
    *  ```« ip route add <Network_adress> via <Local_IP> dev <Local_interface_net2> »```  

Effectuez la commande ```« traceroute »``` pour vérifier.

### II. Spéléologie réseau

#### 1. ARP
#### A. Manip 1

* Vider la table ARP avec la commande : ```« sudo ip neigh flush all »```

* Sur le client1 :
    * ```« ip neigh show »```
    * La ligne qui s'affiche est le cache ARP

* Sur le server1 :
    * ```« ip neigh show »```
    * La ligne qui s'affiche est le cache ARP

* Sur le client1 :
    * ```« ping <adress_server1> »```
    * ```« ip neigh show »```

* Sur le server1 :
    * ```« ip neigh show »```

#### B. Manip 2

* Vider la table ARP avec la commande : ```« sudo ip neigh flush all »```

* Sur le router1 :
    * ```« ip neigh show »```
    * La ligne qui s'affiche est le cache ARP

* Sur le client1 :
    * ```« ping <adress_server1> »```

* Sur le router1 :
    * ```« ip neigh show »```
    * La ligne qui s'affiche est le cache ARP

#### C. Manip 3
* Vider la table ARP avec la commande : ```« sudo ip neigh flush all »```
* Sur l'hôte (votre PC) :
    * ```« ip neigh show »```
    * ```« sudo ip neigh flush all »```
    * ```« ip neigh show »```
    * Attendre un peu
    * ```« ip neigh show »```
    * La passerelle change

#### D. Manip 4
* Vider la table ARP avec la commande : ```« sudo ip neigh flush all »```
* Sur client1 :
    * ```« ip neigh show »```
    * Allez dans les paramètres réseau de la machine et activer la carte NAT
    * ```« curl google.com »```
    * ```« ip neigh show »```

#### 2. Wireshark
#### A. Interception d'ARP et ping

1) Sur le router1 :
    * Éxécuter la commande ```« sudo tcpdump -i enp0s9 -w ping.pcap »```

2) Sur le client1 :
    * Vider la table ARP avec la commande : ```« sudo ip neigh flush all »```
    * Utilisez la commande ```« ping -c 4 <adress_server1> »```

3) Sur le router1 :
    * Appuyez sur ```CTRL + C```
    * Utilisez ```« ls »``` et regardez si il y a bien le fichier ```ping.pcap```

4) Sur l'hôte :
    * Ouvrir le fichier ```ping.pcap``` avec wireshark
    * Une fois cela fait vous pouvais désormais voir le ```protocole ARP, les ings aller, les pings retour```

#### B. Interception d'une communication netcat

1) Sur le router1 :
    * Ouvrir ```Wireshark```
    * Capturez le trafic, vous pouvez utilisez un ```filtre``` pour y voir plus clair

2) Sur le client1 :
    * Connectez ```client1``` au serveur ```netcat``` de ```server1```
    * Lorsque vous enregistrerai vous capture nommez la ```netcat_ok.pcap```

3) Sur le server1 :
    * Ouvrir le port firewall

4) Sur toutes les machines :
    * Vider la table ARP avec la commande : ```« sudo ip neigh flush all »```

###### Vous pouvez refaire les manipulations précédentes mais sans ouvrir le firewall du ```server1```

#### C. Interception d'un trafic HTTP

#### Installation et configue du serveur Web

#### Créer le serveur Web :  

Sur le server1 :

* Éxécutez les commandes suivantes :
    * ```« sudo ifup enp0s3 »```
    * ```« sudo yum install -y nginx »```
    * ```« sudo firewall-cmd --add-port=80/tcp »```
    * ```« sudo firewall-cmd --reload »```
    * ```« sudo systemctl start nginx »```

* Pour éteindre le serveur Web :
    * ```« sudo ifdown enp0s3 »```

#### Tester si le serveur fonctionne :

* Sur le server1 :
    * ```« sudo systemctl status nginx » ```
    * ```« curl localhost:80 »```

* Sur le router1 :
    * ```« curl vers l'IP de server1 »```

* Sur l'hôte :
    * Allez sur votre navigateur internet et entrez dans la barre de recherche ```« http://<IP_server1>:80 »```

#### Interception du trafic :

1) Sur le server1 :
    * ```« sudo systemctl status nginx » ```
    * ```« curl localhost:80 »```
    * Vider la table ARP avec la commande : ```« sudo ip neigh flush all »```

2) Sur le client1 :
    * ```« sudo ip neigh flush all »```

3) Sur router1 :
    * ```« sudo ip neigh flush all »```
    * Lancez une capture que vous nommerez ```http.pcap```

4) Sur le client1 :
    * Connectez vous au serveur web avec le client web

5) Sur l'hôte :
    * Vous pouvez voir votre capture et voir que le trafic HTTP circule dans les trames TCP