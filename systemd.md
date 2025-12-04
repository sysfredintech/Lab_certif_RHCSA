# RHEL 10 - Systemd

---

## Définition

> Systemd est une suite de composants essentiels pour un système Linux. Il fournit un gestionnaire de système et de services qui s'exécute avec l'identifiant de processus 1 (PID 1) et lance le reste du système.

> Systemd offre des capacités de parallélisation avancées, utilise l'activation par socket et D-Bus pour le démarrage des services, permet le démarrage à la demande des démons, gère les processus à l'aide des groupes de contrôle Linux, maintient les points de montage et de montage automatique, et implémente une logique de contrôle de service transactionnelle et basée sur les dépendances. systemd prend en charge les scripts d'initialisation SysV et LSB et remplace sysvinit.

> Parmi ses autres composants figurent un démon de journalisation, des utilitaires pour la configuration système de base (nom d'hôte, date, paramètres régionaux), la gestion des utilisateurs connectés, des conteneurs et machines virtuelles en cours d'exécution, les comptes système, les répertoires et paramètres d'exécution, ainsi que des démons pour la configuration réseau simple, la synchronisation de l'heure réseau, le transfert des journaux et la résolution de noms.

---

## La commande systemctl

> systemctl peut être utilisé pour examiner et contrôler l'état du système « systemd » et du gestionnaire de services.

### Les options pour obtenir des informations sur l'état des services

Afficher les services actuellement en mémoire. Par défaut, seuls les services actifs, ceux ayant des tâches en attente ou ceux ayant échoué sont affichées ; ce comportement peut être modifié avec l’option `--all`
```bash
systemctl list-units
```

Afficher l'état d'activation au démarrage de toutes les unités systemd
```bash
systemctl list-unit-files
```

Lister les sockets actuellement en mémoire, triés par adresse d'écoute.
```bash
systemctl list-sockets
```

Vérifier qu'un service est actif
```bash
systemctl is-active # nom du service
```

Vérifier qu'un service est activé au démarrage du système
```bash
systemctl is-enabled # nom du service
```

Afficher la configuation d'un service
```bash
systemctl cat # nom du service
```

Afficher tous les services en échec
```bash
systemctl --failed
```

### Les options courantes de gestion des services

|option|effet|
|:----:|:---:|
|start|Démarrer|
|stop|Arrêter|
|restart|Redémarrer|
|reload|Recharger la configuration|
|status|Afficher le statut|
|enable|Activer au démarrage|
|disable|Désactiver au démarrage|
|daemon-reload|Recharger la configuration|
|mask|Masquer|
|unmask|Démasquer|

---

## Les niveaux d'execution (targets)

### Les principales targets disponibles

- emergency &rarr; Démarrage sur une console de récupération (urgence)
- rescue &rarr; Démarrage en mode _single-user_ minimal
- multi-user &rarr; Démarrage normal sans interface graphique
- graphical &rarr; Démarrage normal avec interface graphique

### Gestion des targets

Afficher la target par défaut utilisée par systemd
```bash
systemctl get-default
```

Afficher toutes les targets actuellement chargées
```bash
systemctl list-units --type target
```

Configurer systemd de manière à utiliser une unité de cible différente par défaut
```bash
systemctl set-default # nom.target
```

Passer à une target différente dans la session actuelle
```bash
systemctl isolate # nom.target
```

---

## Création d'un service avec systemd

Les services gérés par systemd doivent être placés dans `/etc/systemd/system/` et avoir pour extension `.service`

### Création d'un service simple de serveur web léger

Prérequis: un dossier contenant une page web html, ici:
```bash
mkdir /srv/labweb
echo "<h1>Hello World</h1>" > /srv/labweb/index.html
```

```bash
nano /etc/systemd/system/labweb.service
```
```
[Unit]
Description=Serveur web python
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/python -m http.server -d /srv/labweb

[Install]
WantedBy=multi-user.target
```

Explications:

- `[Unit]` &rarr; contient des options génériques 
- `Description` &rarr; Description significative de l'unité
- `After` &rarr;  L'unité est lancée uniquement après l'activation des unités spécifiées
- `[Service]` &rarr; directives spécifiques au type de service
- `Type` &rarr; Configure le type de démarrage
- `ExecStart` &rarr; Spécifie les commandes ou scripts à exécuter 
- `[Install]` &rarr; contient des informations sur l'installation de l'unité utilisée par les commandes `systemctl enable` et `disable`
- `WantedBy` &rarr; Une liste des unités qui dépendent de l'unité

Le processus principal du service exécute la commande `/usr/bin/python -m http.server -d /srv/labweb` qui sera lancé après la cible `network` dans la cible `multi-user`

### Test du bon fonctionnement du service

- Chargement de la configuration et démarrage du service

```bash
systemctl daemon-reload
systemctl start labweb.service
systemctl status labweb.service
```
```
● labweb.service - Serveur web python
     Loaded: loaded (/etc/systemd/system/labweb.service; disabled; preset: disabled)
     Active: active (running) since Thu 2025-12-04 17:03:12 CET; 6s ago
 Invocation: 6a9173caec7a47fb8fee36edfd5911c2
   Main PID: 3210 (python)
      Tasks: 1 (limit: 10680)
     Memory: 9.7M (peak: 9.9M)
        CPU: 42ms
     CGroup: /system.slice/labweb.service
             └─3210 /usr/bin/python -m http.server -d /srv/labweb

déc. 04 17:03:12 rhel-srv systemd[1]: Started labweb.service - Serveur web python
```
_Le service `Serveur web python` est bien au statut `active (running)`_

- Activation du service pour un lancement au démarrage du système

```bash
systemctl enable labweb.service
```
```
Created symlink '/etc/systemd/system/multi-user.target.wants/labweb.service' → '/etc/systemd/system/labweb.service'.
```
_Un lien symbolique a été créé dans le sous-dossier `multi-user.target.wants` afin que le service soit démarrer dans cette cible_

```bash
systemctl is-active labweb.service
```
`active`

---

## La journalisation

La journalisation de systemd est indépendante et gérée par un service nommé `systemd-journald`

### La commande journalctl

`journalctl` est l'unique commande pour consulter les journaux de systemd, elle peut s'utiliser avec de nombreuses options

|option|affichage|
|:----:|:---:|
|-n _x_|les _x_ dernières entrées|
|-f|en temps réel|
|-k|les journaux du kernel|
|-p _priorité_|filtre par priorité|
|_UID=_x_|filtre par utilisateur selon l'uid|
|_PID=_x_|filtre par processus selon le pid|
|--since _x_|après le moment défini|
|--until _x_|avant le moment défini|
|-u _x_|filtre par nom de service|
|-x|explications|
|-e|immédiatement la fin du journal|
|-b|informations sur le dernier démarrage|

#### Exemples

```bash
journalctl -xu sshd
```
_Affiche les journaux du service `sshd.service` avec des explications_
```
déc. 04 09:03:34 rhel-srv systemd[1]: Starting sshd.service - OpenSSH server daemon...
░░ Subject: L'unité (unit) sshd.service a commencé à démarrer
░░ Defined-By: systemd
░░ Support: https://access.redhat.com/support
░░ 
░░ L'unité (unit) sshd.service a commencé à démarrer.
déc. 04 09:03:34 rhel-srv (sshd)[1001]: sshd.service: Referenced but unset environment variable evaluates to an empty string: OPTIONS
déc. 04 09:03:34 rhel-srv sshd[1001]: Server listening on 0.0.0.0 port 22.
déc. 04 09:03:34 rhel-srv sshd[1001]: Server listening on :: port 22.
déc. 04 09:03:34 rhel-srv systemd[1]: Started sshd.service - OpenSSH server daemon.
░░ Subject: L'unité (unit) sshd.service a terminé son démarrage
░░ Defined-By: systemd
░░ Support: https://access.redhat.com/support
░░ 
░░ L'unité (unit) sshd.service a terminé son démarrage, avec le résultat done.
déc. 04 09:05:52 rhel-srv sshd-session[1739]: Accepted publickey for root from 192.168.10.31 port 46514 ssh2: ED25519 SHA256:YMiLkLhAkZrrZtV>
déc. 04 09:05:52 rhel-srv sshd-session[1739]: pam_unix(sshd:session): session opened for user root(uid=0) by root(uid=0)
déc. 04 09:16:09 rhel-srv sshd-session[1841]: Accepted publickey for root from 192.168.10.31 port 45884 ssh2: ED25519 SHA256:YMiLkLhAkZrrZtV>
déc. 04 09:16:09 rhel-srv sshd-session[1841]: pam_unix(sshd:session): session opened for user root(uid=0) by root(uid=0)
```

```bash
journalctl -b -p err
```
_Affiche uniquement les évenements de type `error` dans les journaux du dernier démarrage_
```
déc. 04 09:03:34 rhel-srv firewalld[857]: ERROR: INVALID_ZONE: labzone
```

```bash
journalctl -ke
```
_Affiche directement les derniers messages des journaux du kernel_
```
déc. 04 09:03:34 rhel-srv kernel: br-ethlab: port 1(ens19) entered blocking state
déc. 04 09:03:34 rhel-srv kernel: br-ethlab: port 1(ens19) entered disabled state
déc. 04 09:03:34 rhel-srv kernel: virtio_net virtio3 ens19: entered allmulticast mode
déc. 04 09:03:34 rhel-srv kernel: virtio_net virtio3 ens19: entered promiscuous mode
déc. 04 09:03:34 rhel-srv kernel: br-ethlab: port 1(ens19) entered blocking state
déc. 04 09:03:34 rhel-srv kernel: br-ethlab: port 1(ens19) entered listening state
déc. 04 09:03:41 rhel-srv kernel: block dm-0: the capability attribute has been deprecated.
déc. 04 09:04:27 rhel-srv kernel: br-ethlab: port 1(ens19) entered learning state
déc. 04 09:04:42 rhel-srv kernel: br-ethlab: port 1(ens19) entered forwarding state
déc. 04 09:04:42 rhel-srv kernel: br-ethlab: topology change detected, propagating
```

---

_Sources principales:_
- https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/using_systemd_unit_files_to_customize_and_optimize_your_system/managing-systemd
- https://systemd.io
- https://man7.org/linux/man-pages/man1/journalctl.1.html
- `man systemctl`
- `man journalctl`