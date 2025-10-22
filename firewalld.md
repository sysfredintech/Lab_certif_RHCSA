# RHEL 10 - Firewalld

---

**Firewalld est un service qui fournit un firewall dynamique et personnalisable avec une interface de type D-Bus**

---

**Toutes les opérations réalisées doivent être valider avec `firewall-cmd --reload` si l'option `--permanent` a été passée pour prendre effet immédiatement ou exécuter `firewall-cmd --runtime-to-permanent` pour définir la configuration actuelle comme persistante**

---

## Découverte de l'état du système

### Afficher les informations essentielles sur le service firewalld

```bash
systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-10-20 11:31:32 CEST; 13min ago
 Invocation: e55564e792a7441c8b4b3fc1cb827671
       Docs: man:firewalld(1)
    Process: 1034 ExecStartPost=/usr/bin/firewall-cmd --state (code=exited, status=0/SUCCESS)
   Main PID: 992 (firewalld)
      Tasks: 2 (limit: 10680)
     Memory: 47.5M (peak: 70M)
        CPU: 448ms
     CGroup: /system.slice/firewalld.service
             └─992 /usr/bin/python3 -sP /usr/sbin/firewalld --nofork --nopid

oct. 20 11:31:32 rhel-srv systemd[1]: Starting firewalld.service - firewalld - dynamic firewall daemon...
oct. 20 11:31:32 rhel-srv systemd[1]: Started firewalld.service - firewalld - dynamic firewall daemon.
```
```bash
firewall-cmd --state
running
```
_Le service est bien actif_

---

## La notion de zones

> Vous pouvez utiliser le service firewalld pour diviser les réseaux en différentes zones selon le niveau de confiance que vous accordez aux interfaces et au trafic au sein de ce réseau. Une connexion ne peut appartenir qu'à une seule zone, mais vous pouvez utiliser cette zone pour plusieurs connexions réseau.

**L'utilisation des zones permet de cloisoner les réseaux en fonction du niveau de confiance avec les interfaces. Chaque zone possède ses règles et comportements par défaut. Il est possible d'affecter des zones aux interfaces ou à des sources**

- Lister les zones présentes

```bash
firewall-cmd --get-zones
block dmz drop external home internal nm-shared public trusted work
```
```bash
firewall-cmd --get-default-zone 
public
```
_La zone utilisée par défaut est `public`_

- Lister les informations pour la zone par défaut _public_
```bash
firewall-cmd --list-all  --zone=public
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: br-ethlab ens18 ens19
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules:
```

---

## Les services prédéfinis

1. Lister les services existants et afficher les informations essentielles

```bash
firewall-cmd --get-services
```
_renvoi une énumération exhaustive de tous les services_

```bash
firewall-cmd --list-services
cockpit dhcpv6-client ssh
```
_Liste les services autorisés dans la zone courante (public)_

```bash
firewall-cmd --list-services --zone internal 
cockpit dhcpv6-client mdns samba-client ssh
```
_liste les services autorisés dans une zone_

```bash
firewall-cmd --info-service=http
http
  ports: 80/tcp
  protocols: 
  source-ports: 
  modules: 
  destination: 
  includes: 
  helpers: 
```
_Fourni des informations sur le service `http` en particulier_

2. Autoriser un service

- Pour autoriser le service http de manière temporaire dans la zone courante
```bash
firewall-cmd --add-service=http
success
```
```bash
firewall-cmd --list-services
cockpit dhcpv6-client http ssh
```
- Il faut ajouter `--permanent` pour que cette configuration soit persistente
```bash
firewall-cmd --add-service=http --permanent
success
firewall-cmd --reload 
success
```
```bash
firewall-cmd --list-services --permanent 
cockpit dhcpv6-client http ssh
```
- Pour autoriser un service dans une zone défini
```bash
firewall-cmd --add-service=http --permanent --zone work
success
firewall-cmd --reload 
success
```
```bash
firewall-cmd --list-services --zone=work
cockpit dhcpv6-client http ssh
```

---

## La gestion des ports

1. Lister les ports ouverts dans la zone courante

```bash
firewall-cmd --list-ports 

```
_Aucun port n'est ouvert actuellement_

2. Ouvrir et fermer des ports définis

- De façon temporaire
```bash
firewall-cmd --add-port=8080/tcp
success
```
```bash
firewall-cmd --list-ports 
8080/tcp
```
- De façon permanente
```bash
firewall-cmd --add-port=8080/tcp --permanent 
success
firewall-cmd --reload 
success
```
```bash
firewall-cmd --list-ports --permanent 
8080/tcp
```
- Pour une plage de ports
```bash
firewall-cmd --add-port=8081-8088/tcp --permanent 
success
firewall-cmd --reload 
success
firewall-cmd --list-ports --permanent 
8080/tcp 8081-8088/tcp
```
- Fermer un port
```bash
firewall-cmd --remove-port=8080/tcp --permanent
success
firewall-cmd --reload 
success
firewall-cmd --list-ports --permanent 
8081-8088/tcp
```

---

## Configurer les zones

### Les interfaces

**Une interface ne peut appartenir qu'à une seule zone à la fois**

- Lister les interfaces et leur zone
```bash
firewall-cmd --get-active-zones
public (default)
  interfaces: br-ethlab ens19 ens18
```

- Changer la zone d'une interface
```bash
firewall-cmd --change-interface=ens19 --zone=work
success
```
```bash
firewall-cmd --get-active-zones
public (default)
  interfaces: br-ethlab ens18
work
  interfaces: ens19
```

- Retirer une interface d'une zone
```bash
firewall-cmd --remove-interface=ens19 --zone=work
success
```
```bash
firewall-cmd --get-active-zones
public (default)
  interfaces: br-ethlab ens18
```

- Ajouter une interface à une zone de façon permanente
```bash
firewall-cmd --add-interface=ens19 --zone=dmz --permanent
The interface is under control of NetworkManager, setting zone to 'dmz'.
success
firewall-cmd --reload 
success
```
```bash
firewall-cmd --get-active-zones
dmz
  interfaces: ens19
public (default)
  interfaces: br-ethlab ens18
```

### Les sources

- Ajouter une adresse IP à la zone _trusted_
```bash
firewall-cmd --zone=trusted --add-source=192.168.10.181
success
```
- Ajouter un sous-réseau à la zone work
```bash
firewall-cmd --zone=work --add-source=192.168.10.0/24
success
```
- Application de la configuration
```bash
firewall-cmd --runtime-to-permanent 
success
```

- Vérification
```bash
firewall-cmd  --list-sources --zone trusted
192.168.10.181
firewall-cmd  --list-sources --zone work
192.168.10.0/24
```

- Supprimer une source
```bash
firewall-cmd --remove-source=192.168.10.181 --zone=trusted 
success
firewall-cmd --runtime-to-permanent 
success
```

---

##  Configuration de mises à jour dynamiques par fichier de listes avec IPsets

- Lister les types IP set utilisables
```bash
firewall-cmd --get-ipset-types
hash:ip hash:ip,mark hash:ip,port hash:ip,port,ip hash:ip,port,net hash:mac hash:net hash:net,iface hash:net,net hash:net,port hash:net,port,net
```

- Créer une IP set avec un nom explicite
```bash
firewall-cmd --new-ipset=allowlist --type=hash:ip --permanent
success
```

- Incrémenter la liste _allowlist_ avec des adresses IP
```bash
firewall-cmd --ipset=allowlist --add-entry=192.168.10.55 --permanent
success
```

- Créer une règle basée sur la liste _allowlist_ pour la zone _trusted_
```bash
firewall-cmd --add-source=ipset:allowlist --zone=trusted --permanent
success
firewall-cmd --reload
success
```

- Afficher les listes IP set
```bash
firewall-cmd --get-ipsets
allowlist
```

- Afficher le contenu d'une liste IP set
**Les listes sont stockées dans `/etc/firewalld/ipsets/`**
```bash
cat /etc/firewalld/ipsets/allowlist.xml
<?xml version="1.0" encoding="utf-8"?>
<ipset type="hash:ip">
  <entry>192.168.10.55</entry>
</ipset>
```
Ou
```bash
firewall-cmd --info-ipset=allowlist
allowlist
  type: hash:ip
  options: 
  entries: 192.168.10.55
```

- Ajouter dynamiquement des adresses IP dans la liste _allowlist_
```bash
firewall-cmd --ipset=allowlist --add-entry=192.168.10.45 --permanent
success
firewall-cmd --reload
success
```

- Créer une liste IP set pour les sous-réseaux nommé _allownetlist_
```bash
firewall-cmd --new-ipset=allownetlist --type=hash:net --permanent
success
```
```bash
firewall-cmd --ipset=allownetlist --add-entry=192.168.0.0/24 --permanent
success
firewall-cmd --reload
success
```

- Vérifications
```bash
firewall-cmd --info-ipset=allowlist
allowlist
  type: hash:ip
  options: 
  entries: 192.168.10.55 192.168.10.45
```
```bash
firewall-cmd --info-ipset=allownetlist
allownetlist
  type: hash:net
  options: 
  entries: 192.168.0.0/24
```

- Créer un fichier contenant une liste d'adresses IP pour alimenter une liste IP set
```bash
vim iplist

192.168.10.200
192.168.10.201
192.168.10.203
```
```bash
firewall-cmd --new-ipset=filelist --type=hash:ip --permanent
success
```
```bash
firewall-cmd --ipset=filelist --add-entries-from-file=iplist --permanent
success
```
- Verification
```bash
firewall-cmd --ipset=filelist --get-entries
192.168.10.200
192.168.10.201
192.168.10.203
```
- Supprimer le contenu précedement ajouté
```bash
firewall-cmd --ipset=filelist --remove-entries-from-file=iplist --permanent
success
firewall-cmd --reload
success
```

---

## Les rich rules (définition de règles fines)

**Ces règles ont une pripriéte plus élevée**

- Refuser une ip pour le service ssh
```bash
firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.10.181" service name="ssh" drop' --permanent
success
```

- Autoriser un sous-réseau pour le service http
```bash
firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.1.0/24" service name=http accept' --permanent
success
```

- Valider
```bash
firewall-cmd --reload
success
```

- Vérification
```bash
firewall-cmd --list-rich-rules 
rule family="ipv4" source address="192.168.1.0/24" service name="http" accept
rule family="ipv4" source address="192.168.10.181" service name="ssh" drop
```

- Redirection de port
```bash
firewall-cmd --add-rich-rule='rule family="ipv4" forward-port port="8888" protocol="tcp" to-port="80"' --permanent --zone=public
success
firewall-cmd --reload
success
```
_Redirige le port _8888_ vers le port 80 dans la zone public_

- Vérification
```bash
firewall-cmd --list-rich-rules --zone=public 
rule family="ipv4" forward-port port="8888" protocol="tcp" to-port="80"
rule family="ipv4" source address="192.168.1.0/24" service name="http" accept
rule family="ipv4" source address="192.168.10.181" service name="ssh" drop
```

- Supprimer la règle précedement créée
```bash
firewall-cmd --remove-rich-rule='rule family="ipv4" forward-port port="8888" protocol="tcp" to-port="80"'
firewall-cmd --reload
success
```

---

## Les règles direct

**L'option `--direct` permet d'utiliser la syntaxe iptables/ip6tables à travers firewalld, cela contourne le système par zones et permet d'accéder aux tables et chaînes bas niveau. Le risque de conflis est important avec les règles natives de firewalld**

- Bloquer le trafic sortant vers le port 53 (dns)
```bash
firewall-cmd --direct --add-rule ipv4 filter OUTPUT 0 -p udp --dport 53 -j DROP
success
```
`ipv4` &rarr; famille d'adressage ip
`filter OUTPUT` &rarr; trafic sortant
`0` &rarr; priorité la plus haute
`-p udp` &rarr; protocole choisi
`--dport 53` &rarr; port de destination choisi
`-j DROP` &rarr; décision

- Vérification
```bash
firewall-cmd --direct --get-all-rules 
ipv4 filter OUTPUT 0 -p udp --dport 53 -j DROP
```

- Supprimer la règle précédemment créé
```bash
firewall-cmd --direct --remove-rule ipv4 filter OUTPUT 0 -p udp --dport 53 -j DROP
success
```

---

## Sauvegarder et restaurer la configuration de firewalld

- Afficher l'ensemble de la configuration de firewalld
```bash
firewall-cmd --list-all-zones
```

- Exporter la configuration actuellement appliquée dans un fichier
```bash
firewall-cmd --list-all-zones > firewalld_config.old
```

- Créer une archive tar du dossier _/etc/firewalld_
```bash
tar -cf firewalld_backup.tar /etc/firewalld/
tar: Suppression de « / » au début des noms des membres
```

- Réinitialiser la configuration d'origine &rarr; **détruit toute la configuration**
```bash
firewall-cmd --reset-to-defaults
success
firewall-cmd --reload 
success
```

- Vérification
```bash
firewall-cmd --list-all-zones
```
_On constate que l'ensemble des règles précédemment configurées ne sont plus en place_

- Restaurer l'ancienne configuration
```bash
tar -xf firewalld_backup.tar -C /
```
```bash
firewall-cmd --reload 
success
```

- Vérification
```bash
firewall-cmd --list-all --zone=public
public (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: br-ethlab ens18
  sources: 
  services: cockpit dhcpv6-client http ssh
  ports: 8081-8088/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" forward-port port="8888" protocol="tcp" to-port="80"
	rule family="ipv4" source address="192.168.1.0/24" service name="http" accept
	rule family="ipv4" source address="192.168.10.181" service name="ssh" drop
```
Les anciennes règles sont à nouveau en place

---

## Exercice de création d'une zone avec des règles à appliquer

**Objectif:**

- Créer une zone nommée _labzone_
- Définir une règle pour ajouter le service http dans cette zone
- Définir une régle pour autoriser l'accès au port 2222/tcp dans cette zone
- Définir une règle pour bloquer l'accès au port 2222/tcp au sous-réseau 192.168.20.0/24 dans cette zone
- Ajouter l'interface ens19 à cette zone
- Définir cette zone comme zone par défaut
- Rendre cette configuration persistente

1. Création de la nouvelle zone
```bash
firewall-cmd --new-zone=labzone --permanent
success
```

2. Définition d'une nouvelle règle autorisant http dans la zone _labzone_
```bash
firewall-cmd --add-service=http --zone=labzone --permanent
success
```

3. Définition d'une règle pour autoriser l'accès sur le port 2222/tcp dans la zone _labzone_
```bash
firewall-cmd --add-port=2222/tcp --zone=labzone --permanent
success
```

4. Définition d'une règle pour interdire le port 2222/tcp au sous-réseau _192.168.20.0/24_ dans la zone _labzone_
```bash
firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.20.0/24" port port="2222" protocol="tcp" drop' --zone=labzone --permanent
success
```

5. Ajouter l'interface _ens19_ dans la zone _labzone_
```bash
firewall-cmd --change-interface=ens19 --zone=labzone --permanent
The interface is under control of NetworkManager, setting zone to 'labzone'.
success
```

6. Appliquer la configuration et définir _labzone_ comme zone par défaut
```bash
firewall-cmd --reload 
success
```
```bash
firewall-cmd --set-default-zone=labzone
success
```
```bash
firewall-cmd --runtime-to-permanent 
success
```

7. Verification
```bash
firewall-cmd --list-all --zone=labzone
labzone (default, active)
  target: default
  ingress-priority: 0
  egress-priority: 0
  icmp-block-inversion: no
  interfaces: ens19
  sources: 
  services: http
  ports: 2222/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
	rule family="ipv4" source address="192.168.20.0/24" port port="2222" protocol="tcp" drop
```

---

## Résumé des commandes de vérifications de l'état et de la configuration de firewalld

- `systemctl status firewalld.service` &rarr; état du service firewalld
- `firewall-cmd --state` &rarr; vérifier si firewalld est actif
- `firewall-cmd --list-all-zones` &rarr; informations détaillées sur l'ensemble des zones
- `firewall-cmd --list-all` &rarr; informations détaillées sur la zone courante
- `firewall-cmd --list-services` &rarr; liste les services autorisés dans la zone courante
- `firewall-cmd --list-sources` &rarr; liste les sources autorisés dans la zone courante
- `firewall-cmd --list-interfaces` &rarr; liste les interfaces liées à la zone courante
- `firewall-cmd --get-active-zones` &rarr; affiche les sources et interfaces des zones pour lesquelles une configuration a été appliquée

---

_Sources principales:_
- https://docs.redhat.com/fr/documentation/red_hat_enterprise_linux/10/html/configuring_firewalls_and_packet_filters/using-and-configuring-firewalld
- `man firewall-cmd`