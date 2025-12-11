# RHEL 10 - Permissions et ACLs

---

## Les permissions standards

### Concepts

1. Catégories d'utilisateurs

- Utilisateur propriétaire &rarr; `u`
- Groupe propriétaire &rarr; `g`
- Tout autre &rarr; `o`

2. Niveau de permission

- Lecture &rarr; `r` &rarr; `4`
- Ecriture &rarr; `w` &rarr; `2`
- Exécution &rarr; `x` &rarr; `1`

3. Type de fichier

- Fichier standard &rarr; `-`
- Dossier &rarr; `d`
- Lien symbolique &rarr; `l`
- Périphérique d'entrée &rarr; `c`
- Périphérique de type bloc &rarr; `b`
- Fichier pipe (fifo) &rarr; `p`
- Fichier socket &rarr; `s`

4. Représentation dans le terminal

|Type de fichier|Permissions `u`|Permissions `g`|Permissions `o`|Valeur octale|Exemple|
|:-------------:|:-------------:|:-------------:|:-------------:|:-----------:|:-----:|
|`-`            |`rw-`          |`r--`          |`r--`          |644          |`/etc/hosts`|
|`d`            |`rwx`          |`r-x`          |`r-x`          |755          |`/var/www`|
|`l`            |`rwx`          |`rwx`          |`rwx`          |777          |`/dev/cdrom`|

### Pratique

1. Afficher les permissions appliquées sur un fichier
```bash
ls -l fichier_lab.txt
```
`-rw-r--r--. 1 root root 0 30 oct.  09:15 fichier_lab.txt`

2. Modifier les permissions appliquées sur un fichier
```bash
chmod 664 fichier_lab.txt
ls -l fichier_lab.txt
```
`-rw-rw-r--. 1 root root 0 30 oct.  09:15 fichier_lab.txt`
_On a ajouté les droits en écriture pour le groupe propriétaire_
```bash
chmod o-r fichier_lab.txt
ls -l fichier_lab.txt
```
`-rw-rw----. 1 root root 0 30 oct.  09:15 fichier_lab.txt`
_On a retiré les droits en lecture pour tout autre utilisateur_

```bash
chmod +x script_lab.sh
ls -l script_lab.sh
```
`-rwxr-xr-x. 1 root root 0 30 oct.  09:19 script_lab.sh`
_On a ajouté les droits en exécution pour tous les utilisateurs_

3. Modifier l'utilisateur propriétaire ou le groupe propriétaire
```bash
chown labuser1: fichier_lab.txt
ls -l fichier_lab.txt
```
`-rw-rw----. 1 labuser1 labuser1 0 30 oct.  09:15 fichier_lab.txt`
_On a défini l'utilisateur labuser1 comme propriétaire du fichier_

```bash
chown :postgres fichier_lab.txt
ls -l fichier_lab.txt
```
Ou
```bash
chgrp postgres fichier_lab.txt
```
`-rw-rw----. 1 labuser1 postgres 0 30 oct.  09:15 fichier_lab.txt`
_On a défini le groupe postgres comme propriétaire du fichier_

```bash
chown root:root fichier_lab.txt
ls -l fichier_lab.txt
```
`-rw-rw----. 1 root root 0 30 oct.  09:15 fichier_lab.txt`
_On a défini l'utilisateur et le groupe root comme propriétaire_

---

## Les permissions spéciales

### Concepts

- SUID &rarr; set user ID &rarr; `4000` &rarr; Le fichier s'exécute avec les droits du propriétaire du fichier
- SGID &rarr; set group ID &rarr; `2000` &rarr; Pour les dossiers: les nouveaux fichiers héritent des permissions du groupe parent
- Sticky Bit &rarr; `1000` &rarr; Pour les dossier: restriction sur la suppression des fichiers

### Pratique

1. Obtenir des informations sur les permissions spéciales

```bash
ls -l /usr/bin/sudo
```
`---s--x--x. 1 root root 297752  9 juil. 02:00 /usr/bin/sudo`
_Le fichier /usr/bin/sudo a une permission fixée à `4111` pour s'exécuter avec les droits de l'utilisateur propriétaire_

```bash
ls -ld /run/log/journal/
```
`drwxr-sr-x+ 3 root systemd-journal 60 30 oct.  08:07 /run/log/journal/`
_Le dossier /run/log/journal/ a une permission fixée à `2755` et les fichiers créés dans ce dossier hériteront de ses permissions_

```bash
ls -ld /tmp/
```
`drwxrwxrwt. 11 root root 4096 30 oct.  09:38 /tmp/`
_Le dossier /tmp/ a une permission fixée à `1777` qui détermine l'impossibilité de supprimer des fichiers dont on est pas le propriétaire_

2. Modifier les permissions spéciales

```bash
chmod 4755 script_lab.sh
ls -l script_lab.sh
```
`-rwsr-xr-x. 1 root root 0 30 oct.  09:19 script_lab.sh`
_Le fichier script_lab.sh sera exécuté avec les droits root pour tous les utilisateurs_

```bash
chmod g+s dossier_lab/
ls -ld dossier_lab/
```
`drwxr-sr-x. 2 root root 6 30 oct.  13:48 dossier_lab/`
_Le dossier dossier_lab/ est maintenant défini avec un SGID_

```bash
chmod 1777 sticky_dir/
ls -ld sticky_dir/
```
`drwxrwxrwt. 2 root root 6 30 oct.  13:54 sticky_dir/`
_Le dossier sticky_dir/ est maintenant défini avec un StickyBit_

**Si le marqueur de permission spéciale apparaît en majuscule: cela signifie une incohérence avec les permissions en exécution**

---

## Umask

### Concept

**User File Creation MASK - Le masque du mode de création de fichiers par l'utilisateur**
**Permet de définir des droits d'accès par défaut pour l'ensemble des fichiers et des dossiers créés**

- Ce masque se calcule par soustraction

Permissions fichier standard &rarr; `666` moins umask `022` &rarr; `644`
Permissions dossier standard &rarr;  `777` moins umask `022` &rarr; `755`

```bash
umask
```
`0022` &rarr; il s'agit du umask par défaut pour tous les utilisateurs

### Pratique

- Modifier son umask temporairement
```bash
su - labuser1
umask 077
touch testumask.txt
ls -l testumask.txt
```
`-rw-------. 1 labuser1 labuser1 0 30 oct.  14:44 testumask.txt`
_L'utilisateur labuser1 a modifié son umask jusqu'à sa prochaine déconnexion_

- Modifier son umask de manière persistente

```bash
su - labuser1
vim ~/.bashrc
```
Ajouter une ligne `umask` avec la valeur choisie
`umask 077` par exemple
```bash
touch testumask2.txt
ls -l testumask2.txt
```
`-rw-------. 1 labuser1 labuser1 0 30 oct.  14:52 testumask2.txt`
_L'utilisateur labuser1 a modifié son umask de manière persistente_

- Modifier l'umask pour tous les nouveaux utilisateurs créés
```bash
vim /etc/login.defs
```
Modifier la ligne `UMASK           022` par la définition voulue
`UMASK           027` par exemple

- Vérification
```bash
useradd umaskuser
su - umaskuser
touch fichier_test.txt
ls -l fichier_test.txt 
```
`-rw-r-----. 1 umaskuser umaskuser 0 30 oct.  15:07 fichier_test.txt`
_Le nouvel utilisateur umaskuser a un umask défini à `027`_

---

## Les ACLs

**Les ACL nous permettent d'appliquer un ensemble d'autorisations plus spécifique à un fichier ou un dossier sans (nécessairement) modifier le propriétaire et les autorisations de base**

**2 commandes essentielles:**
- `getfacl` &rarr; pour afficher les ACLs
- `setfacl` &rarr; pour définir les ACLs

### Créer un contexte pour le lab

- Un dossier _/root/labacl/_ avec les permissions `770`
- Un fichier _/root/labacl/fichier_acl1_ avec les permissions `700`
- Un fichier _/root/labacl/fichier_acl2_ avec les permissions `700`
- Un utilisateur _labuser1_ membre du groupe _groupallow_
- Un utilisateur _labuser2_ membre du groupe _groupdeny_
```bash
ls -la /root/labacl/
```
```
total 4
drwxrwx---.  2 root root   46 30 oct.  16:13 .
dr-xr-x---. 11 root root 4096 30 oct.  15:54 ..
-rwx------.  1 root root    0 30 oct.  15:58 fichier_acl1
-rwx------.  1 root root    0 30 oct.  16:13 fichier_acl2
```

### Définition des ACLs à appliquer dans le lab

1. Permissions pour les groupes
- Donner les permissions en lecture, écriture et exécution sur le dossier _/root/labacl/_ au groupe _groupallow_
- Donner les permissions en lecture et exécution de manière récursive sur le dossier _/root/labacl/_ au groupe _groupdeny_
2. Permissions pour les utilisateurs
- Donner les permissions en lecture, écriture et exécution à l'utilisateur _labuser1_ sur le fichier _fichier_acl1_
- Donner les permissions en lecture uniquement à l'utilisateur _labuser1_ sur le fichier _fichier_acl2_
3. Vérifications
- Afficher les ACLs en place pour le dossier _/root/labacl/_
- Afficher les ACLs en place pour les fichiers _fichier_acl1_ et _fichier_acl2_

### Mise en pratique

1. Permissions pour les groupes
```bash
setfacl -m g:groupallow:rwx /root/labacl/
setfacl -R -m g:groupdeny:rx /root/labacl/
```

2. Permissions pour les utilisateurs
```bash
setfacl -m u:labuser1:rwx /root/labacl/fichier_acl1
setfacl -m u:labuser1:r /root/labacl/fichier_acl2
```

3. Vérifications
```bash
getfacl /root/labacl/
```
```
getfacl : suppression du premier « / » des noms de chemins absolus
# file: root/labacl/
# owner: root
# group: root
user::rwx
group::rwx
group:groupallow:rwx
group:groupdeny:r-x
mask::rwx
other::---
```
```bash
getfacl /root/labacl/fichier_acl1 /root/labacl/fichier_acl2
```
```
getfacl : suppression du premier « / » des noms de chemins absolus
# file: root/labacl/fichier_acl1
# owner: root
# group: root
user::rwx
user:labuser1:rwx
group::---
group:groupdeny:r-x
mask::rwx
other::---

# file: root/labacl/fichier_acl2
# owner: root
# group: root
user::rwx
user:labuser1:r--
group::---
group:groupdeny:r-x
mask::r-x
other::---
```

### Supprimer les ACLs en place

- Pour supprimer les ACLs d'un utilisateur sur un fichier
```bash
setfacl -x u:labuser1 /root/labacl/fichier_acl2
```

- Pour supprimer les ACLs d'un groupe sur un dossier
```bash
setfacl -x g:groupdeny /root/labacl/
```

- Pour supprimer toutes les ACLs définies sur un fichier
```bash
setfacl -b /root/labacl/fichier_acl1
```

- Vérification sur un dossier de manière récursive
```bash
getfacl -R /root/labacl/
```
```
getfacl : suppression du premier « / » des noms de chemins absolus
# file: root/labacl/
# owner: root
# group: root
user::rwx
group::rwx
group:groupallow:rwx
mask::rwx
other::---

# file: root/labacl//fichier_acl1
# owner: root
# group: root
user::rwx
group::---
other::---

# file: root/labacl//fichier_acl2
# owner: root
# group: root
user::rwx
group::---
group:groupdeny:r-x
mask::r-x
other::---
```

---

_Sources principales:_

- https://www.redhat.com/en/blog/suid-sgid-sticky-bit
- https://www.redhat.com/en/blog/linux-access-control-lists
- `man getfacl` `man setfacl`