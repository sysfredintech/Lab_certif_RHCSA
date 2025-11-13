# RHEL 10 - Conteneurs avec podman

---

## Concept

Cette technologie s'appuie sur:
- "cgroups" pour la gestion des ressources
- "Namespaces" pour l'isolation des processus

> Red Hat Enterprise Linux (RHEL) propose plusieurs outils en ligne de commande pour la gestion des images de conteneurs. Podman permet de gérer les pods et les images de conteneurs. Buildah permet de créer, mettre à jour et gérer ces images. Skopeo permet de copier et d'examiner des images dans des dépôts distants. Outre ces outils en ligne de commande, Podman Desktop, une interface graphique, permet également de gérer les conteneurs, les images, les pods et les registres.

> Comme ces outils sont compatibles avec l'Open Container Initiative (OCI), ils peuvent être utilisés pour gérer les mêmes conteneurs Linux que ceux produits et gérés par Docker et d'autres moteurs de conteneurs compatibles avec l'OCI.

> Les principaux avantages des outils Podman, Podman Desktop, Skopeo et Buildah sont les suivants :
> - Fonctionnement en mode sans privilèges root : les conteneurs sans privilèges root sont beaucoup plus sécurisés, car ils s’exécutent sans privilèges supplémentaires.
> - Aucun démon requis : ces outils consomment beaucoup moins de ressources en veille, car si aucun conteneur n’est en cours d’exécution, Podman ne l’est pas non plus. Docker, en revanche, dispose d’un démon toujours actif.
> - Intégration native avec systemd : Podman permet de créer des fichiers d’unité systemd et d’exécuter des conteneurs en tant que services système.

### Type d'images de conteneurs

- Red Hat Enterprise Linux Base Images (RHEL base images)

> Prises en charge par Red Hat pour une utilisation avec des applications conteneurisées, elles contiennent les mêmes paquets logiciels sécurisés, testés et certifiés que ceux de Red Hat Enterprise Linux.

- Red Hat Universal Base Images (UBI images)

> Les images Red Hat Universal Base sont construites à partir d'un sous-ensemble du contenu standard de Red Hat Enterprise Linux.

> Fournis un ensemble de dépôts DNF associés qui incluent des paquets RPM et des mises à jour qui vous permettent d’ajouter des dépendances d’application et de reconstruire des images de conteneur UBI.

### Les registres de conteneurs

> Un registre de conteneurs est un référentiel, ou un ensemble de référentiels, permettant de stocker des images de conteneurs et des artefacts d'applications conteneurisées.

Les registres RHEL:
- registry.redhat.io (nécessite une authentification)
- registry.access.redhat.com (ne nécessite pas d'authentification)
- registry.connect.redhat.com (contient des images du programme Red Hat Partner Connect)

---

## Préparation du système

### Installation des paquets nécessaires

**Un meta-package est disponible**
```bash
dnf install container-tools
```
**Il contient:**
> Latest versions of podman, buildah, skopeo, runc, conmon, CRIU, Udica, etc as
well as dependencies such as container-selinux built and tested together, and
updated.

### Connexion au registre registry.redhat.io

```bash
podman login registry.redhat.io
```
```
Username: 
Password: 
Login Succeeded!
```

---

## Opérations basiques

- Rechercher une image parmis les registres
```bash
podman search httpd
```
```
NAME                                                                DESCRIPTION
registry.redhat.io/rhscl/httpd-24-rhel7                             Platform for running Apache httpd 2.4 or bui...
registry.redhat.io/rhel8/httpd-24                                   Platform for running Apache httpd 2.4 or bui...
registry.redhat.io/ubi8/httpd-24                                    Platform for running Apache httpd 2.4 or bui...
registry.redhat.io/rhel9/httpd-24                                   Platform for running Apache httpd 2.4 or bui...
registry.redhat.io/ubi9/httpd-24                                    Platform for running Apache httpd 2.4 or bui...
registry.redhat.io/rhel10/httpd-24                                  Platform for running Apache httpd 2.4 or bui...
registry.redhat.io/ubi10/httpd-24                                   Platform for running Apache httpd 2.4 or bui...
registry.redhat.io/rhmap44/httpd                                    RHMAP Docker container that provides the RHM...
registry.redhat.io/rhmap45/httpd                                    RHMAP Container image that provides the RHMA...
registry.redhat.io/cloudforms46-beta/cfme-openshift-httpd           CloudForms 4.6 Httpd for OpenShift
registry.redhat.io/cloudforms46/cfme-openshift-httpd                CloudForms 4.6 Httpd for OpenShift
registry.redhat.io/rhmap46/httpd                                    RHMAP Container image that provides the RHMA...
registry.redhat.io/amq7/amq-online-1-console-httpd                  AMQ Online Console HTTPD
registry.redhat.io/openshift4/ose-egress-http-proxy                 OpenShift Container Platform Egress Http Pro...
registry.redhat.io/rhmap42/httpd                                    RHMAP Docker container that provides the RHM...
registry.redhat.io/rhmap43/httpd                                    RHMAP Docker container that provides the RHM...
registry.redhat.io/rhmap47/httpd                                    RHMAP container image that provides the RHMA...
registry.redhat.io/cloudforms47/cfme-openshift-httpd                CloudForms 4.7 Httpd for OpenShift
registry.redhat.io/openshift4/ose-egress-http-proxy-rhel9           OpenShift Container Platform Egress Http Pro...
registry.redhat.io/rhscl/varnish-4-rhel7                            Varnish 4 high-performance HTTP accelerator
registry.redhat.io/amq7-tech-preview/amq-online-1-iot-http-adapter  AMQ Online IoT HTTP Adapter
registry.redhat.io/rhscl/varnish-5-rhel7                            Platform for running Varnish or building Var...
registry.redhat.io/rhscl/varnish-6-rhel7                            Platform for running Varnish or building Var...
registry.redhat.io/openshift3/ose-egress-http-proxy                 OpenShift Container Platform Egress Http Pro...
registry.redhat.io/rhtas-tech-preview/client-server-rhel9           An HTTP server providing RHTAS CLIs
```

- Inspecter une image distante
```bash
skopeo inspect --config docker://registry.redhat.io/ubi10/httpd-24
```
_Affiche les informations détaillées de l'image au format json_

- Télécharger une image
```bash
podman pull registry.redhat.io/ubi10/httpd-24
```

- Lister les images disponibles sur le système
```bash
podman images
```
```
REPOSITORY                         TAG         IMAGE ID      CREATED     SIZE
registry.redhat.io/ubi10/httpd-24  latest      fc1d350bd526  9 days ago  287 MB
```

- Inspecter une image locale
```bash
podman image inspect fc1d350bd526
```
_Affiche les informations détaillées de l'image au format json_

- Lancer un conteneur depuis une image locale en mode interactif

```bash
podman run -ti registry.redhat.io/ubi10/httpd-24 /bin/bash
```
_Le prompt indique que nous avons ouvert un terminal dans le conteneur_
`bash-5.2$`
```bash
exit
```

- Lancer un conteneur depuis une image locale en arrière-plan
```bash
podman run -d registry.redhat.io/ubi10/httpd-24
```

- lister les conteneurs lancés en arrière plan
```bash
podman ps
```
```
CONTAINER ID  IMAGE                                     COMMAND               CREATED        STATUS        PORTS               NAMES
850326eb7e7d  registry.redhat.io/ubi10/httpd-24:latest  /usr/bin/run-http...  8 seconds ago  Up 9 seconds  8080/tcp, 8443/tcp  brave_ardinghelli
```

- Stopper un conteneur en cours d'exécution
```bash
podman stop 850326eb7e7d
```

- lister tous les conteneurs
```bash
podman ps -all
```
```
CONTAINER ID  IMAGE                                     COMMAND               CREATED             STATUS                     PORTS               NAMES
850326eb7e7d  registry.redhat.io/ubi10/httpd-24:latest  /usr/bin/run-http...  About a minute ago  Exited (0) 14 seconds ago  8080/tcp, 8443/tcp  brave_ardinghelli
```

- Supprimer un conteneur
```bash
podman rm 850326eb7e7d
```

- Supprimer une image stockée localement
```bash
podman rmi fc1d350bd526
```
```
Untagged: registry.redhat.io/ubi10/httpd-24:latest
Deleted: fc1d350bd526027d32eeeb0470bfc390917eab76338d5336f05079532e408ff8
```

---

## Utilisation avancée

**Podman étant _rootless_ nous allons utilisé un compte utilisateur nommé _poduser_ dédié pour le lab**

**Podman utilise systemd pour la gestion des cgroups en mode utilisateur. Pour que l'utilisateur _poduser_ puisse bénéficier d'une session systemd persistante, même après la déconnexion, nous allons activer le _lingering_ avec la commande `loginctl` comme suit:**

```bash
loginctl enable-linger poduser
```
Vérification:
```bash
loginctl list-users
```
```
 UID USER    LINGER STATE    
   0 root    no     active
1009 poduser yes    lingering

2 users listed.
```

**La connexion au registre doit être réalisée indépendamment pour chaque utilisateur**
```bash
su - poduser
podman login registry.redhat.io
```
```
Username: 
Password: 
Login Succeeded!
```

### Gestion des volumes

- Création d'un volume nommé _labvol_
```bash
podman volume create labvol
```
- Inspection du volume _labvol_
```bash
podman volume inspect labvol
```
```json
[
     {
          "Name": "labvol",
          "Driver": "local",
          "Mountpoint": "/home/poduser/.local/share/containers/storage/volumes/labvol/_data",
          "CreatedAt": "2025-11-06T08:58:30.318142876+01:00",
          "Labels": {},
          "Scope": "local",
          "Options": {},
          "MountCount": 0,
          "NeedsCopyUp": true,
          "NeedsChown": true,
          "LockNumber": 0
     }
]
```
- Utilisation du volume _labvol_ à la création d'un container

```bash
podman run -d --name labredis -v labvol:/var/lib/redis/data:Z -p 6379:6379 registry.redhat.io/rhel9/redis-6
```
- `d` &rarr; mode détaché
- `--name` &rarr; on donne le nom _labredis_ au conteneur
- `-p` &rarr; mappage du port 6379 de l'hôte vers 6379 du conteneur
- `-v` &rarr; mappage du volume _labvol_ de l'hôte vers le dossier _/var/lib/redis/data_ du conteneur (:Z pour la bonne intégration SELinux)

```bash
podman ps
```
```
CONTAINER ID  IMAGE                                      COMMAND               CREATED         STATUS         PORTS                                           NAMES
c097f66f2b0e  registry.redhat.io/rhel9/redis-6:latest    run-redis             3 seconds ago   Up 4 seconds   0.0.0.0:6379->6379/tcp                          labredis
```
```bash
ls -la /home/poduser/.local/share/containers/storage/volumes/labvol/_data
```
```
total 4
drwxrwx---. 2 1115112 poduser 22  6 nov.  11:42 .
drwx------. 3 poduser poduser 19  6 nov.  11:23 ..
-rw-r--r--. 1 1115112 1115109 77  6 nov.  11:42 dump.rdb
```
_Le conteneur utilise le volume labvol pour stocker ses données_

- Ecrire dans la base et tester la persistance des données
```bash
podman exec labredis redis-cli SET welcome "Hello World"
podman exec labredis redis-cli GET welcome
```
`Hello World`
```bash
podman stop labredis
podman start labredis
```
```bash
podman exec labredis redis-cli GET welcome
```
`Hello World`

### Monter un dossier de l'hôte dans le conteneur à sa création

- Préparation du contexte et lancement du conteneur
```bash
mkdir -p labwww/html/
echo "<h1>Bienvenu dans le lab podman</h1>" > labwww/html/index.html
podman run -d --name httpdlab -p 8080:8080 -p 8443:8443 -v ./labwww:/var/www:Z registry.redhat.io/rhel10/httpd-24
podman ps
```
```
CONTAINER ID  IMAGE                                      COMMAND               CREATED         STATUS         PORTS                                           NAMES
fc7ddb1a02cc  registry.redhat.io/rhel10/httpd-24:latest  /usr/bin/run-http...  10 seconds ago  Up 10 seconds  0.0.0.0:8080->8080/tcp, 0.0.0.0:8443->8443/tcp  httpdlab
```
```bash
curl http://localhost:8080
```
`<h1>Bienvenu dans le lab podman</h1>`

- Tester la persistence des données
```bash
podman stop httpdlab
podman start httpdlab
```
```bash
curl http://localhost:8080
```
`<h1>Bienvenu dans le lab podman</h1>`

### Gestion des réseaux

- Lister les réseaux
```bash
podman network ls
```
```
NETWORK ID    NAME        DRIVER
2f259bab93aa  podman      bridge
```

-  Créer un réseau
```bash
podman network create labnet
podman network ls
```
```
NETWORK ID    NAME        DRIVER
3406bfb16e31  labnet      bridge
2f259bab93aa  podman      bridge
```

- Inspecter un réseau
```bash
podman network inspect labnet
```
```json
[
     {
          "name": "labnet",
          "id": "3406bfb16e31f588f1178da3f31db6274b9efe453e8c6c0e4c69a41fade5c815",
          "driver": "bridge",
          "network_interface": "podman1",
          "created": "2025-11-07T08:29:36.891402017+01:00",
          "subnets": [
               {
                    "subnet": "10.89.0.0/24",
                    "gateway": "10.89.0.1"
               }
          ],
          "ipv6_enabled": false,
          "internal": false,
          "dns_enabled": true,
          "ipam_options": {
               "driver": "host-local"
          },
          "containers": {}
     }
]
```

- Lancer des conteneurs sur le réseau _labnet_
```bash
podman rm -f httpdlab
podman rm -f labredis
podman run -d --name httpdlab -v ./labwww:/var/www:Z -p 8080:8080 -p 8443:8443 --network labnet registry.redhat.io/rhel10/httpd-24
podman run -d --name labredis -v labvol:/var/lib/redis/data:Z -p 6379:6379 --network labnet registry.redhat.io/rhel9/redis-6
```

- Vérification
```bash
podman network inspect labnet
```
```json
[
     {
          "name": "labnet",
          "id": "3406bfb16e31f588f1178da3f31db6274b9efe453e8c6c0e4c69a41fade5c815",
          "driver": "bridge",
          "network_interface": "podman1",
          "created": "2025-11-07T08:29:36.891402017+01:00",
          "subnets": [
               {
                    "subnet": "10.89.0.0/24",
                    "gateway": "10.89.0.1"
               }
          ],
          "ipv6_enabled": false,
          "internal": false,
          "dns_enabled": true,
          "ipam_options": {
               "driver": "host-local"
          },
          "containers": {
               "593c5c9300be49a92f5943832e85e01ca01447fb71557887ec77d5231bc0a63b": {
                    "name": "labredis",
                    "interfaces": {
                         "eth0": {
                              "subnets": [
                                   {
                                        "ipnet": "10.89.0.3/24",
                                        "gateway": "10.89.0.1"
                                   }
                              ],
                              "mac_address": "a2:fe:c4:cd:13:44"
                         }
                    }
               },
               "72c10377afad7d87ab94203a6ec46d2ce4f30ed298ed9f30d475ca49466a0491": {
                    "name": "httpdlab",
                    "interfaces": {
                         "eth0": {
                              "subnets": [
                                   {
                                        "ipnet": "10.89.0.2/24",
                                        "gateway": "10.89.0.1"
                                   }
                              ],
                              "mac_address": "fa:f9:ed:88:c3:66"
                         }
                    }
               }
          }
     }
]
```
_Les 2 conteneurs sont bien connectés au réseau labnet_

- Tester la connectivité entre les 2 conteneurs
```bash
podman exec labredis sh -c "curl http://10.89.0.2:8080"
```
```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
<h1>Bienvenu dans le lab podman</h1> 0      0 --:--:-- --:--:-- --:--:--     0
100    37  100    37    0     0  37000      0 --:--:-- --:--:-- --:--:-- 37000
```
_Le conteneur labredis peut contacter httpdlab_


### Les journaux

- Afficher les logs d'un conteneur en cours d'exécution
```bash
podman logs labredis
```

- Afficher les logs en temps réel d'un conteneur en cours d'exécution
```bash
podman logs -f labredis
```

- Afficher les logs selon une période d'un conteneur en cours d'exécution
```bash
podman logs --since 30m httpdlab
```
`10.89.0.3 - - [07/Nov/2025:08:28:46 +0000] "GET / HTTP/1.1" 200 37 "-" "curl/7.76.1"`
_Affiche les logs de moins de 30mn_

- Afficher les logs d'un conteneur en cours d'exécution en limitant la sortie
```bash
podman logs --tail 5 httpdlab
```
_Affiche les 5 dernières lignes_

---

## Création d'une image

1. Préparation de l'environnement

```bash
su - poduser
podman login registry.redhat.io
mkdir httpd
cd httpd
echo '<h1>Bienvenue sur le serveur web!</h1>' > ./index.html
```

2. Choisir l'image de base à utiliser
```bash
podman search "ubi minimal" --limit 6
```
```
NAME                                        DESCRIPTION
registry.redhat.io/ubi7-minimal             Provides the latest release of the minimal R...
registry.redhat.io/ubi8/ubi-minimal         Provides the latest release of the Minimal R...
registry.redhat.io/ubi9/ubi-minimal         Provides the latest release of the Minimal R...
registry.redhat.io/ubi8-minimal             Provides the latest release of the minimal R...
registry.redhat.io/ubi7/ubi-minimal         Provides the latest release of the minimal R...
registry.redhat.io/ubi10/ubi-minimal        Provides the latest release of the Minimal R...
```
_Pour le lab, on choisi `registry.redhat.io/ubi10/ubi-minimal`_

3. Création du fichier _Containerfile_
```bash
vim Containerfile
```
```
FROM registry.redhat.io/ubi10/ubi-minimal:latest
MAINTAINER PodUser poduser@redhat.com
LABEL Serveur HTTPD basé sur UBI 10
COPY index.html /var/www/html/
RUN microdnf update -y && \
    microdnf install -y httpd && \
    microdnf clean all &&\
    chown -R apache:apache /var/www/html && \
    chmod -R 755 /var/www/html
EXPOSE 80
CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
```

- `FROM` &rarr; l'image existante à partir de laquelle on créer notre image
- `MAINTAINER` et `LABEL` &rarr; métadonnées de l'image
- `RUN` &rarr; commandes passées durant le processus de création de l'image
- `COPY` &rarr; copie du contenu du dossier courant dans l'image
- `EXPOSE` &rarr; exposition du port utilisé pour le service
- `CMD` &rarr; commande lancée après le processus de création

4. Lancer la création de l'image
```bash
podman build --tag httpdlab .
```
Vérification
```bash
podman images
```
```
REPOSITORY                               TAG         IMAGE ID      CREATED         SIZE
localhost/httpdlab                       latest      99c3224a8c25  26 seconds ago  159 MB
```

5. Lancer un conteneur à partir de l'image précédemment crée
```bash
podman run -d --name weblab -p 8080:80 httpdlab
```
Vérification
```bash
podman ps
```
```
CONTAINER ID  IMAGE                      COMMAND               CREATED        STATUS        PORTS                 NAMES
3a42e528fbca  localhost/httpdlab:latest  /usr/sbin/httpd -...  7 seconds ago  Up 7 seconds  0.0.0.0:8080->80/tcp  weblab
```
```bash
curl http://localhost:8080
```
`<h1>Bienvenue sur le serveur web!</h1>`

---

## Créer une nouvelle image à partir d'un conteneur

```bash
podman ps
```
```
CONTAINER ID  IMAGE                      COMMAND               CREATED      STATUS      PORTS                 NAMES
3a42e528fbca  localhost/httpdlab:latest  /usr/sbin/httpd -...  2 hours ago  Up 2 hours  0.0.0.0:8080->80/tcp  weblab
```
```bash
podman commit weblab weblabimg
```
```bash
podman images
```
```
REPOSITORY                               TAG         IMAGE ID      CREATED         SIZE
localhost/weblabimg                      latest      54fcf349f8f3  13 seconds ago  159 MB
localhost/httpdlab                       latest      9d3e715b4674  2 hours ago     159 MB
```
```bash
podman rm -f weblab
podman run -d -p 8080:80 weblabimg:latest
podman ps
```
```
CONTAINER ID  IMAGE                       COMMAND               CREATED         STATUS         PORTS                 NAMES
414047415c13  localhost/weblabimg:latest  /usr/sbin/httpd -...  42 seconds ago  Up 42 seconds  0.0.0.0:8080->80/tcp  naughty_bouman
```
```bash
curl http://localhost:8080
```
```
<h1>Bienvenue sur le serveur web!</h1>
```

---

## Monter le système de fichier racine d'un conteneur sur l'hôte

> Si un utilisateur sans privilèges souhaite monter un conteneur et l'utiliser, il doit exécuter `podman unshare`. L'exécution de `podman mount` échoue pour les utilisateurs sans privilèges, sauf s'ils se trouvent dans une session `podman unshare`.

```bash
podman unshare
podman mount naughty_bouman
```
`/home/poduser/.local/share/containers/storage/overlay/678c7db33bd380d10115ef054405e16a4408275f1b1aace922c629b7ff12509e/merged`
```bash
ls /home/poduser/.local/share/containers/storage/overlay/678c7db33bd380d10115ef054405e16a4408275f1b1aace922c629b7ff12509e/merged
```
`afs  boot  etc   lib    media  opt   root  sbin  sys  usr
bin  dev   home  lib64  mnt    proc  run   srv   tmp  var`

---

## Résumé  des commandes essentielles

- Gestion des conteneurs

|Commande|Action|
|:--------:|:-----------------:|
|podman run|Lancer un conteneur|
|podman ps|Lister les conteneurs actifs|
|podman ps a|Lister tous les conteneurs|
|podman stop|Stopper un conteneur|
|podman start|Démarrer un conteneur existant|
|podman restart|Redémarrer un conteneur|
|podman rm|Supprimer un conteneur|
|podman exec|Executer une commande dans un conteneur|
|podman inspect|Inspecter un conteneur|

- Gestion des volumes

|Commande|Action|
|:--------:|:-----------------:|
|podman volume create|Créer un volume|
|podman volume ls|Lister les volumes|
|podman volume inspect|Inspecter un volume|
|podman volume rm|Supprimer un volume|

- Gestion des réseaux

|Commande|Action|
|:--------:|:-----------------:|
|podman network create|Créer un réseau|
|podman network ls|Lister les réseau|
|podman network inspect|Inspecter un réseau|
|podman network rm|Supprimer un réseau|
|podman network connect|Connecter un conteneur à un réseau|
|podman network disconnect|Deconnecter un conteneur d'un réseau|

- Surveillance

|Commande|Action|
|:--------:|:-----------------:|
|podman logs|Afficher les logs d'un conteneur|
|podman stats|Surveillance en temps réel d'un conteneur|
|podman top|Afficher les processus d'un conteneur|
|podman system df|Utilisation de l'espace disque|

- Nettoyage

|Commande|Action|
|:--------:|:-----------------:|
|podman system prune|Supprimer les conteneurs et images inutilisés|
|podman system reset|Réinitialisation totale|

---

_Sources principales:_
- https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html-single/building_running_and_managing_containers/index
- https://www.redhat.com/en/blog/write-your-first-containerfile-podman
- `man podman`