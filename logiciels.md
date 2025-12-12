# RHEL 10 - Gestion des logiciels

---

## Ecosystème RPM

- RPM est l'acronyme de "Red Hat Package Manager"

- RPM est un système de gestion de paquets permettant de distribuer, gérer et mettre à jour les logiciels.

- Un fichier `.rpm` est formaté comme suit:
`package_name-version-release-operating_system-CPU_architecture.rpm`

### La commande RPM

- Lister les paquets installés
```bash
rpm -qa
```

- Afficher les informations relatives à un paquet installé
```bash
rpm -qi # nom du paquet
```

- A quel paquet appartient un fichier
```bash
rpm -qf # chemin vers le fichier
```

- Vérifier l'intégrité des fichiers relatifs à un paquet
```bash
rpm -V # nom du paquet
rpm -Va # pour tous les paquets installés
```
_cf `man rpm` pour l'interpretation de la sortie_

- Afficher les dépendances requises par un paquet
```bash
rpm -qR # nom du paquet
```

- Afficher le paquet dont provient un fichier
```bash
rpm -q --whatprovides # nom du fichier
```

### La commande DNF

**Gestion du contenu des dépôts RPM à l'aide de l'outil de gestion de logiciels DNF**

**Le fichier de configuration principal de DNF est `/etc/dnf/dnf.conf`**

```bash
vim /etc/dnf/dnf.conf
```
```
[main]
gpgcheck=1
installonly_limit=3
clean_requirements_on_remove=True
best=True
skip_if_unavailable=False
```

#### Les commandes basiques

- `dnf search` &rarr; rechercher par nom
- `dnf info` &rarr; afficher des informations détaillées sur un paquet
- `dnf install` &rarr; installer un paquet
- `dnf update` &rarr; mettre à jour un ou tous les paquets
- `dnf remove` &rarr; désinstaller un paquet
- `dnf autoremove` &rarr; désinstaller les dépendances inutiles
- `dnf check` &rarr; vérifier les problèmes de dépendances
- `dnf reinstall` &rarr; Reinstaller un paquet
- `dnf provides` &rarr; Trouver quel(s) paquet(s) fournit un fichier ou une commande
- `dnf config-manager --dump` &rarr; Afficher la configuration globale

#### Les groupes de paquets

- Lister tous les groupes
```bash
dnf group list
```
_Affiche une liste exhaustive des groupes installés et disponibles_

- Afficher le contenu d'un groupe
```bash
dnf group info # nom du groupe
```

- Installer un groupe de paquets
```bash
dnf group install # nom du groupe
```
_Installe l'ensemble des paquets contenus dans le groupe_

- Désinstaller un groupe de paquets
```bash
dnf group remove # nom du groupe
```

#### Gestion des dépôts

- Lister les dépôts activés sur le système
```bash
dnf repolist enabled
```
```
id du dépôt                               nom du dépôt
codeready-builder-for-rhel-10-x86_64-rpms Red Hat CodeReady Linux Builder for RHEL 10 x86_64 (RPMs)
elrepo                                    ELRepo.org Community Enterprise Linux Repository - el10
epel                                      Extra Packages for Enterprise Linux 10 - x86_64
rhel-10-for-x86_64-appstream-rpms         Red Hat Enterprise Linux 10 for x86_64 - AppStream (RPMs)
rhel-10-for-x86_64-baseos-rpms            Red Hat Enterprise Linux 10 for x86_64 - BaseOS (RPMs)
```

- Afficher les informations relatives à un dépôt
```bash
dnf repoinfo rhel-10-for-x86_64-baseos-rpms
```
```
Mise à jour des référentiels de gestion des abonnements.
Dernière vérification de l’expiration des métadonnées effectuée il y a 0:13:59 le jeu. 11 déc. 2025 08:18:14.
Id du dépôt       : rhel-10-for-x86_64-baseos-rpms
Nom du dépôt      : Red Hat Enterprise Linux 10 for x86_64 - BaseOS (RPMs)
État du dépôt     : activé
Révision du dépôt : 1765203644
Dépôt mis à jour  : lun. 08 déc. 2025 15:20:43
Paquets du dépôt  : 3 345
Paquets dispo. : 3 345
Taille du dépôt   : 18 G
Baseurl du dépôt  : https://cdn.redhat.com/content/dist/rhel10/10/x86_64/baseos/os
Expirat° du dépôt : 86 400 secondes (dernier : jeu. 11 déc. 2025 08:18:14)
Nom de fichier du dépôt : /etc/yum.repos.d/redhat.repo
Total des paquets : 3 345
```

- Lister l'ensemble des dépôts utilisables
```bash
dnf repolist all
```
_Affiche une liste exhaustive_

- Activer un dépôt
```bash
dnf config-manager --set-enabled # nom du dépôt
```

- Désactiver un dépôt
```bash
dnf config-manager --set-disabled # nom du dépôt
```

- Les fichiers de configuration des dépôts
```bash
ls -l /etc/yum.repos.d/
```
```
-rw-r--r--. 1 root root  1983 10 déc.  18:59 elrepo.repo
-rw-r--r--. 1 root root  1612 17 avril  2025 epel.repo
-rw-r--r--. 1 root root  1714 17 avril  2025 epel-testing.repo
-rw-r--r--. 1 root root 68524  4 déc.  19:59 redhat.repo
```
```bash
cat /etc/yum.repos.d/epel.repo
```
```
[epel]
name=Extra Packages for Enterprise Linux $releasever - $basearch
# It is much more secure to use the metalink, but if you wish to use a local mirror
# place its address here.
#baseurl=https://download.example/pub/epel/$releasever${releasever_minor:+z}/Everything/$basearch/
metalink=https://mirrors.fedoraproject.org/metalink?repo=epel${releasever_minor:+-z}-$releasever&arch=$basearch
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-$releasever_major
gpgcheck=1
repo_gpgcheck=0
metadata_expire=24h
countme=1
enabled=1
```

#### Gestion des clés de signatures GPG

- Lister les clés déjà importées sur le système
```bash
rpm -qa gpg-pubkey*
```

- Vérifier les signatures de paquets
```bash
rpm --checksig # chemin vers fichier.rpm
```

- Importer une clé
```bash
rpm --import # chemin vers RPM-GPG-KEY
```

- Supprimer une clé
```bash
rpm -e # gpg-pubkey-xxxxxxxxx
```

---

## Ecosystème Flatpak

### Concept

> Flatpak fournit un environnement isolé (sandbox) pour la création, le déploiement, la distribution et l'installation d'applications.

> Les applications lancées avec Flatpak disposent d'un accès minimal au système hôte, ce qui protège l'installation contre les applications tierces. Flatpak garantit la stabilité des applications, quelles que soient les versions des bibliothèques installées sur le système hôte.

> Les applications Flatpak sont distribuées depuis des dépôts appelés « remotes ». Red Hat fournit un dépôt distant contenant les applications RHEL. D'autres dépôts distants tiers sont également disponibles. Red Hat n'assure pas le support des applications provenant de dépôts distants tiers.

---

## Installation et configuration

### Installation du paquet Flatpak

```bash
dnf install flatpak
```

### Ajout du dépôt distant RHEL

```bash
flatpak remote-add --if-not-exists rhel https://flatpaks.redhat.io/rhel.flatpakrepo
```

### Vérification

```bash
flatpak remotes
```
```
Name Options
rhel system,oci,no-gpg-verify
```

---

## La commande flatpak

- lister le contenu d'un dépôt distant configuré

```bash
flatpak remote-ls
```
```
Name                   Application ID                Version       Branch       Arch
Thunderbird            org.mozilla.Thunderbird                     stable       x86_64
Firefox                org.mozilla.firefox           140.5.0       stable       x86_64
Red Hat Platform       com.redhat.Platform           10            el10         x86_64
Red Hat Platform       com.redhat.Platform           8             el8          x86_64
Red Hat Platform       com.redhat.Platform           9             el9          x86_64
Red Hat SDK            com.redhat.Sdk                10            el10         x86_64
Red Hat SDK            com.redhat.Sdk                8             el8          x86_64
Red Hat SDK            com.redhat.Sdk                9             el9          x86_64
```

- Chercher une application ou un runtime

```bash
flatpak search firefox
```
```
Name         Description         Application ID           Version      Branch     Remotes
Firefox      Navigateur Web      org.mozilla.firefox      140.5.0      stable     rhel
```

- Installer une application ou un runtime

```bash
flatpak install org.mozilla.firefox
```
_Une confirmation concernant l'installation du runtime `runtime/com.redhat.Platform/x86_64/el10` nécessaire à l'application choisie est demandée_
```
Required runtime for org.mozilla.firefox/x86_64/stable (runtime/com.redhat.Platform/x86_64/el10) found in remote rhel
Do you want to install it? [Y/n]:
```
`Y`

_Puis une information relative aux permissions de l'application et un résumé des éléments qui vont être installés est affiché_
```
org.mozilla.firefox permissions:
    ipc                  network               pcsc                       pulseaudio
    wayland              x11                   devices                    file access [1]
    dbus access [2]      bus ownership [3]     system dbus access [4]

    [1] /run/.heim_org.h5l.kcm-socket, home:ro, xdg-download, xdg-run/gvfs, xdg-run/gvfsd,
        xdg-run/pipewire-0, ~/.cache/firefox:create, ~/.mozilla:create
    [2] org.a11y.Bus, org.freedesktop.FileManager1, org.freedesktop.Notifications,
        org.freedesktop.ScreenSaver, org.gnome.SessionManager, org.gtk.vfs.*
    [3] org.mozilla.firefox.*, org.mpris.MediaPlayer2.firefox.*
    [4] org.freedesktop.NetworkManager

        ID                         Branch        Op       Remote       Download
 1.     com.redhat.Platform        el10          i        rhel         < 406,9 Mo
 2.     org.mozilla.firefox        stable        i        rhel         < 145,9 Mo

Proceed with these changes to the system installation? [Y/n]:
```
`Y`

`Installation complete.`

- Lister les applications installés

```bash
flatpak list --app
```
```
Name                   Application ID            Version      Branch      Installation
Firefox                org.mozilla.firefox       140.5.0      stable      system
```

- Obtenir des informations sur une application ou un runtime installé

```bash
flatpak info org.mozilla.firefox
```
```
Firefox - Navigateur Web

          ID: org.mozilla.firefox
         Ref: app/org.mozilla.firefox/x86_64/stable
        Arch: x86_64
      Branch: stable
     Version: 140.5.0
     License: GPL-3.0+
      Origin: rhel
  Collection: 
Installation: system
   Installed: 331,0 Mo
     Runtime: com.redhat.Platform/x86_64/el10
         Sdk: com.redhat.Sdk/x86_64/el10

      Commit: 9e2e5d1df7aff4f2daabc164c3cbfd66756bf4baa658c24f717d4a4bc591cd4d
     Subject: Export org.mozilla.firefox
        Date: 2025-12-01 01:08:30 +0000
      Alt-id: dc481479e3e9a629120427187804d370e5df438bff797eae11ddc96fd0e32933
```

- Lancer une application

```bash
flatpak run org.mozilla.firefox
```

- Mettre à jour des applications installées

Pour toutes les applications
```bash
flatpak update
```

Pour une application ciblée
```bash
flatpak update org.mozilla.firefox
```

- Désinstaller une application

```bash
flatpak uninstall org.mozilla.firefox
```
```
        ID                         Branch       Op
 1.     org.mozilla.firefox        stable       r

Proceed with these changes to the system installation? [Y/n]:
```
`Y`

`Uninstall complete.`

- Afficher les runtimes installées

```bash
flatpak list --runtime
```

- Désinstaller les runtimes non-utilisés

```bash
flatpak uninstall --unused
```
```
        ID                         Branch       Op
 1.     com.redhat.Platform        el10         r

Proceed with these changes to the system installation? [Y/n]:
```
`Y`

---

_Source principale:_

- https://docs.redhat.com/fr/documentation/red_hat_enterprise_linux/10/html/managing_software_with_the_dnf_tool/index
- https://docs.redhat.com/fr/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/s1-rpm-using
- https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/administering_the_system_using_the_gnome_desktop_environment/assembly_installing-applications-using-flatpak_administering-the-system-using-the-gnome-desktop-environment