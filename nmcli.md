# NMCLI - REHL 10

***

## Interface CLI du NetworkManager

A savoir:
- Les fichiers de configuration des connexions se situent dans `/etc/NetworkManager/system-connections/`
- La touche `tab` est une aide précieuse avec nmcli
- La plupart des options passées à nmcli peuvent être abrégées

Quelques abréviations utilisées dans ce lab:
- `con` &rarr; `connection`
- `sh` &rarr; `show`
- `dev` &rarr; `device`
- `eth` &rarr; `ethernet`

### Contexte

Une vm rehl 10 avec 2 interfaces réseaux:
- ens18: déjà présente et utilisée pour la connexion ssh
- ens19: non-paramétrée dédiée pour le lab

une vm connectée au même sous-réseau que l'interface ens19 avec l'adresse 10.10.1.10/24 et un serveur dhcp également connecté au même sous-réseau avec l'adresse 10.10.1.254/24

### Objéctifs

- Configurer ens19 pour obtenir sa configuration via un serveur DHCP
- Configurer ens19 avec une adresse statique du sous-réseau 10.10.1.0/24
- Configurer un bridge réseau avec ens19

***

## Informations sur l'état des connexions réseau

1. **Vérifier les permissions accordées à notre compte utilisateur (ici root) sur les gestion de NetworkManager**
```bash
nmcli general permissions
```
```
PERMISSION                                                        VALUE 
org.freedesktop.NetworkManager.checkpoint-rollback                oui   
org.freedesktop.NetworkManager.enable-disable-connectivity-check  oui   
org.freedesktop.NetworkManager.enable-disable-network             oui   
org.freedesktop.NetworkManager.enable-disable-statistics          oui   
org.freedesktop.NetworkManager.enable-disable-wifi                oui   
org.freedesktop.NetworkManager.enable-disable-wimax               oui   
org.freedesktop.NetworkManager.enable-disable-wwan                oui   
org.freedesktop.NetworkManager.network-control                    oui   
org.freedesktop.NetworkManager.reload                             oui   
org.freedesktop.NetworkManager.settings.modify.global-dns         oui   
org.freedesktop.NetworkManager.settings.modify.hostname           oui   
org.freedesktop.NetworkManager.settings.modify.own                oui   
org.freedesktop.NetworkManager.settings.modify.system             oui   
org.freedesktop.NetworkManager.sleep-wake                         oui   
org.freedesktop.NetworkManager.wifi.scan                          oui   
org.freedesktop.NetworkManager.wifi.share.open                    oui   
org.freedesktop.NetworkManager.wifi.share.protected               oui
```
2. **Vérifier les périphériques connus par le NetworkManager et leur état**
```bash
nmcli device status
```
```
DEVICE  TYPE      STATE                  CONNECTION 
ens18   ethernet  connecté               ens18      
lo      loopback  connecté (en externe)  lo         
ens19   ethernet  déconnecté             --
```
3. **Afficher toutes les connexions déjà configurées**
```bash
nmcli connection show
```
```
NAME   UUID                                  TYPE      DEVICE 
ens18  3458d2e7-e19c-3140-af6f-601e617901be  ethernet  ens18  
lo     37c49650-6e22-4d0c-87b7-f02d6e6b0cde  loopback  lo
```
4. **Afficher des informations détaillées sur une interface**
```bash
nmcli device show ens18
```
```
GENERAL.DEVICE:                         ens18
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         BC:24:11:EC:DB:B1
GENERAL.MTU:                            1500
GENERAL.STATE:                          100 (connecté)
GENERAL.CONNECTION:                     ens18
GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/ActiveConnection/2
WIRED-PROPERTIES.CARRIER:               marche
IP4.ADDRESS[1]:                         192.168.10.52/24
IP4.GATEWAY:                            192.168.10.254
IP4.ROUTE[1]:                           dst = 192.168.10.0/24, nh = 0.0.0.0, mt = 100
IP4.ROUTE[2]:                           dst = 0.0.0.0/0, nh = 192.168.10.254, mt = 100
IP4.ROUTE[3]:                           dst = 0.0.0.0/0, nh = 192.168.10.254, mt = 100
IP4.DNS[1]:                             192.168.10.254
IP4.DNS[2]:                             192.168.10.28
IP4.SEARCHES[1]:                        home.lab
IP6.ADDRESS[1]:                         fe80::be24:11ff:feec:dbb1/64
IP6.GATEWAY:                            --
IP6.ROUTE[1]:                           dst = fe80::/64, nh = ::, mt = 1024
```
---

## Configuration basique avec nmcli

### Configurer l'interface ens19 pour utiliser le dhcp

1. Informations sur l'état actuelle de l'interface
```bash
nmcli dev sh ens19
```
```
GENERAL.DEVICE:                         ens19
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         BC:24:11:D7:BE:21
GENERAL.MTU:                            1500
GENERAL.STATE:                          30 (déconnecté)
GENERAL.CONNECTION:                     --
GENERAL.CON-PATH:                       --
WIRED-PROPERTIES.CARRIER:               marche
IP4.GATEWAY:                            --
IP6.GATEWAY:                            --
```
2. Définir les paramètres pour ajouter une connexion DHCP à l'interface ens19
```bash
nmcli con add type eth con-name ethlab ifname ens19 ipv4.method auto
```
```
Connexion « ethlab » (4ea93b3f-38a9-400f-800b-6f2d8901014f) ajoutée avec succès.
```
3. Activer la connexion ethlab
```bash
nmcli con up ethlab
```
```
Connexion activée (chemin D-Bus actif : /org/freedesktop/NetworkManager/ActiveConnection/11)
```

**Vérification:**

```bash
nmcli dev sh ens19
```
```
GENERAL.DEVICE:                         ens19
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         BC:24:11:D7:BE:21
GENERAL.MTU:                            1500
GENERAL.STATE:                          100 (connecté)
GENERAL.CONNECTION:                     ethlab
GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/ActiveConnection/5
WIRED-PROPERTIES.CARRIER:               marche
IP4.ADDRESS[1]:                         10.10.1.20/24
IP4.GATEWAY:                            10.10.1.254
IP4.ROUTE[1]:                           dst = 10.10.1.0/24, nh = 0.0.0.0, mt = 101
IP4.ROUTE[2]:                           dst = 0.0.0.0/0, nh = 10.10.1.254, mt = 101
IP4.DNS[1]:                             9.9.9.9
IP4.DOMAIN[1]:                          home.lab
IP6.ADDRESS[1]:                         fe80::dd4a:7ed4:95db:6c2e/64
IP6.GATEWAY:                            --
IP6.ROUTE[1]:                           dst = fe80::/64, nh = ::, mt = 1024
```
```bash
nmcli con sh ethlab | grep IP4
```
```
IP4.ADDRESS[1]:                         10.10.1.20/24
IP4.GATEWAY:                            10.10.1.254
IP4.ROUTE[1]:                           dst = 10.10.1.0/24, nh = 0.0.0.0, mt = 101
IP4.ROUTE[2]:                           dst = 0.0.0.0/0, nh = 10.10.1.254, mt = 101
IP4.DNS[1]:                             9.9.9.9
IP4.DOMAIN[1]:                          home.lab
```
```bash
nmcli con sh --active
```
```
NAME    UUID                                  TYPE      DEVICE 
ens18   3458d2e7-e19c-3140-af6f-601e617901be  ethernet  ens18  
ethlab  4ea93b3f-38a9-400f-800b-6f2d8901014f  ethernet  ens19  
lo      9b5ca389-5836-4b2d-8187-5af4e752b6df  loopback  lo
```
```bash
ping -c 3 10.10.1.10
```
```
PING 10.10.1.10 (10.10.1.10) 56(84) octets de données.
64 octets de 10.10.1.10 : icmp_seq=1 ttl=64 temps=0.138 ms
64 octets de 10.10.1.10 : icmp_seq=2 ttl=64 temps=0.236 ms
64 octets de 10.10.1.10 : icmp_seq=3 ttl=64 temps=0.180 ms

--- statistiques ping 10.10.1.10 ---
3 paquets transmis, 3 reçus, 0% packet loss, time 2061ms
rtt min/avg/max/mdev = 0.138/0.184/0.236/0.040 ms
```

**Notre machine a bien une connexion _ethlab_ active de type ethernet sur l'interface _ens19_ avec une adresse IP _10.10.1.20/24_ obtenue via un serveur DHCP et est capable de pinguer un serveur du même sous-réseau**

### Modification de la configuration existante pour paramétrer une connexion avec IP statique

1. Désactiver la connexion ethlab
```bash
nmcli con down ethlab
```
```
Connexion « ethlab » désactivée (chemin D-Bus actif : /org/freedesktop/NetworkManager/ActiveConnection/11)
```
**Vérification:**
```bash
nmcli con sh --active
```
```
NAME   UUID                                  TYPE      DEVICE 
ens18  3458d2e7-e19c-3140-af6f-601e617901be  ethernet  ens18  
lo     9b5ca389-5836-4b2d-8187-5af4e752b6df  loopback  lo
```
**ethlab n'est plus dans la liste des connexions actives**

2. Modification des paramètres pour la connexion ethlab
```bash
nmcli con mod ethlab ipv4.addresses 10.10.1.5/24 ipv4.gateway 10.10.1.254 ipv4.dns "9.9.9.9 8.8.8.8" ipv4.method manual
```
**l'option `ipv4.method manual` est primordiale pour ne plus recevoir de configuration depuis le serveur dhcp**

3. Activer la nouvelle configuration
```bash
nmcli con up ethlab
```
```
Connexion activée (chemin D-Bus actif : /org/freedesktop/NetworkManager/ActiveConnection/12)
```

**Vérification:**

```bash
nmcli con sh --active
```
```
NAME    UUID                                  TYPE      DEVICE 
ens18   3458d2e7-e19c-3140-af6f-601e617901be  ethernet  ens18  
ethlab  4ea93b3f-38a9-400f-800b-6f2d8901014f  ethernet  ens19  
lo      9b5ca389-5836-4b2d-8187-5af4e752b6df  loopback  lo
```
```bash
nmcli con sh ethlab | grep IP4
```
```
IP4.ADDRESS[1]:                         10.10.1.5/24
IP4.GATEWAY:                            10.10.1.254
IP4.ROUTE[1]:                           dst = 0.0.0.0/0, nh = 10.10.1.254, mt = 101
IP4.ROUTE[2]:                           dst = 10.10.1.0/24, nh = 0.0.0.0, mt = 101
IP4.DNS[1]:                             9.9.9.9
IP4.DNS[2]:                             8.8.8.8
```
```bash
 ping -c3 10.10.1.10
```
```
PING 10.10.1.10 (10.10.1.10) 56(84) octets de données.
64 octets de 10.10.1.10 : icmp_seq=1 ttl=64 temps=0.222 ms
64 octets de 10.10.1.10 : icmp_seq=2 ttl=64 temps=0.236 ms
64 octets de 10.10.1.10 : icmp_seq=3 ttl=64 temps=0.230 ms

--- statistiques ping 10.10.1.10 ---
3 paquets transmis, 3 reçus, 0% packet loss, time 2078ms
rtt min/avg/max/mdev = 0.222/0.229/0.236/0.005 ms
```
**Notre machine a bien une connexion _ethlab_ active de type ethernet sur l'interface _ens19_ avec une adresse IP _10.10.1.5/24_ et est capable de pinguer un serveur du même sous-réseau**

---

## Création d'un bridge réseau avec l'interface ens19

1. Désactiver la connexion ethlab

```bash
nmcli con down ethlab
```
```
Connexion « ethlab » désactivée (chemin D-Bus actif : /org/freedesktop/NetworkManager/ActiveConnection/6)
```

2. Création d'une connexion de type bridge qui contrôle l'interface virtuelle nommée br-ethlab avec une configuration ip statique

```bash
nmcli con add type bridge con-name bridge-ethlab ifname br-ethlab ipv4.addresses 10.10.1.5/24 ipv4.gateway 10.10.1.254 ipv4.dns "9.9.9.9 8.8.8.8" ipv4.method manual
```
```
Connexion « bridge-ethlab » (443a3b26-bcfd-4472-a20e-a6ffabffc8d0) ajoutée avec succès.
```

3. Ajouter l'interface ens19 à l'interface virtuelle br-ethlab en tant qu'esclave

```bash
nmcli con add type bridge-slave con-name bridge-slave-ens19 ifname ens19 master br-ethlab 
```
```
Connexion « bridge-slave-ens19 » (a1b7721d-7096-44fa-9ba1-baa59536c7c2) ajoutée avec succès.
```

4. Activer les connexions crées

```bash
nmcli con up bridge-ethlab 
```
```
Connexion activée (controller waiting for ports) (Chemin D-Bus actif : /org/freedesktop/NetworkManager/ActiveConnection/10)
nmcli connection up bridge-slave-ens19
Connexion activée (chemin D-Bus actif : /org/freedesktop/NetworkManager/ActiveConnection/12)
```

5. Vérifications

```bash
nmcli con sh
```
```
NAME                UUID                                  TYPE      DEVICE    
ens18               3458d2e7-e19c-3140-af6f-601e617901be  ethernet  ens18     
bridge-ethlab       443a3b26-bcfd-4472-a20e-a6ffabffc8d0  bridge    br-ethlab 
bridge-slave-ens19  a1b7721d-7096-44fa-9ba1-baa59536c7c2  ethernet  ens19     
lo                  74d169bd-8dc0-4921-990f-bd9839c9be0c  loopback  lo        
ethlab              bd167fa7-275d-4e4d-b38b-73168f5473ee  ethernet  -- 
```
```bash
nmcli dev sh br-ethlab 
```
```
GENERAL.DEVICE:                         br-ethlab
GENERAL.TYPE:                           bridge
GENERAL.HWADDR:                         BC:24:11:D7:BE:21
GENERAL.MTU:                            1500
GENERAL.STATE:                          100 (connecté)
GENERAL.CONNECTION:                     bridge-ethlab
GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/ActiveConnection/10
IP4.ADDRESS[1]:                         10.10.1.5/24
IP4.GATEWAY:                            10.10.1.254
IP4.ROUTE[1]:                           dst = 0.0.0.0/0, nh = 10.10.1.254, mt = 425
IP4.ROUTE[2]:                           dst = 10.10.1.0/24, nh = 0.0.0.0, mt = 425
IP4.DNS[1]:                             9.9.9.9
IP4.DNS[2]:                             8.8.8.8
IP6.ADDRESS[1]:                         fe80::21b3:2774:1571:5f4e/64
IP6.GATEWAY:                            --
IP6.ROUTE[1]:                           dst = fe80::/64, nh = ::, mt = 1024
```
```bash
nmcli con show bridge-ethlab | grep IP4
```
```
IP4.ADDRESS[1]:                         10.10.1.5/24
IP4.GATEWAY:                            10.10.1.254
IP4.ROUTE[1]:                           dst = 0.0.0.0/0, nh = 10.10.1.254, mt = 425
IP4.ROUTE[2]:                           dst = 10.10.1.0/24, nh = 0.0.0.0, mt = 425
IP4.DNS[1]:                             9.9.9.9
IP4.DNS[2]:                             8.8.8.8
```
```bash
ping -c 3 -I br-ethlab 10.10.1.10
```
```
PING 10.10.1.10 (10.10.1.10) de 10.10.1.5 br-ethlab : 56(84) octets de données.
64 octets de 10.10.1.10 : icmp_seq=1 ttl=64 temps=0.207 ms
64 octets de 10.10.1.10 : icmp_seq=2 ttl=64 temps=0.210 ms
64 octets de 10.10.1.10 : icmp_seq=3 ttl=64 temps=0.374 ms

--- statistiques ping 10.10.1.10 ---
3 paquets transmis, 3 reçus, 0% packet loss, time 2040ms
rtt min/avg/max/mdev = 0.207/0.263/0.374/0.078 ms
```
```bash
bridge link show br-ethlab
```
```
3: ens19: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master br-ethlab state forwarding priority 32 cost 100
```

**Le bridge bridge-ethlab est fonctionnel, à ce stade il n'est pas utile de conserver la connexion inactive nommée ethlab hormis si l'on veut pouvoir restaurer l'ancienne configuration rapidement**

6. Suppréssion de la connexion ethlab (facultatif)
```bash
nmcli con delete ethlab
```
```
Connexion « ethlab » (bd167fa7-275d-4e4d-b38b-73168f5473ee) supprimée.
```
```bash
nmcli con sh
```
```
NAME                UUID                                  TYPE      DEVICE    
ens18               3458d2e7-e19c-3140-af6f-601e617901be  ethernet  ens18     
bridge-ethlab       443a3b26-bcfd-4472-a20e-a6ffabffc8d0  bridge    br-ethlab 
bridge-slave-ens19  a1b7721d-7096-44fa-9ba1-baa59536c7c2  ethernet  ens19     
lo                  74d169bd-8dc0-4921-990f-bd9839c9be0c  loopback  lo
```

---

_Source principale:_

- https://docs.redhat.com/fr/documentation/red_hat_enterprise_linux/10/html/configuring_and_managing_networking/configuring-an-ethernet-connection#configuring-an-ethernet-connection-by-using-nmcli

- man nmcli