# RHEL 10 - Gestion des Utilisateurs et Groupes Locaux

---

## Gestion des comptes utilisateurs

### La commande `useradd` pour créer des utilisateurs

- `-m` &rarr; créer le répertoire home

- `-d` &rarr; spécifier le chemin du répertoire home

- `-s` &rarr; spécifier le shell

- `-c` &rarr; commentaire

- `-u` &rarr; définir UID

- `-g` &rarr; groupe principal

- `-G` &rarr; groupes supplémentaires

- `-e` &rarr; date d'expiration (YYYY-MM-DD)

- `-f` &rarr; délai avant désactivation après expiration du mot de passe

#### Exemple

- Créer un utilisateur labuser ayant pour UID `1010`, appartenant au groupe `wheel` et le dossier home dans `/mnt/dist_home/labuser`
```bash
useradd -m -d /mnt/dist_home/labuser -u 1010 -G wheel labuser
```

### La commande `userdel` pour supprimer des utilisateurs

- Pour conserver le dossier utilisateur
```bash
userdel labuser
```

- Pour supprimer le dossier utilisateur
```bash
userdel -r labuser
```

- Pour supprimer le dossier utilisateur et mailbox
```bash
userdel -rf labuser
```

### La commande `usermod` pour modifier des utilisateurs

- `-aG` &rarr; ajouter l'utilisateur à des groupes supplémentaires
- `-d` &rarr; changer le dossier home
- `-l` &rarr; changer le nom de login
- `-u` &rarr; changer l'UID
- `-L` &rarr; verrouiller un compte
- `-U` &rarr; déverrouiller un compte
- `-e` &rarr; Définir une date d'expiration pour un compte (YYYY-MM-DD)

#### Exemples

- Ajouter l'utilisateur _labuser_ au groupe `wheel`
```bash
usermod -aG wheel labuser
```

- Définir l'expiration du compte _labuser_ le 06/01/2026
```bash
usermod -e 2026-01-06 labuser
```

### La commande `lslogins` pour afficher les informations sur les utilisateurs

- Afficher les informations sur les utilisateurs connus du système
```bash
lslogins
```

- Afficher les informations sur un utilisateur en particulier
```bash
lslogins # nom de l'utilisateur
```

#### Exemple

- Afficher la date d'expiration du compte _labuser_
```bash
lslogins -o PWD-EXPIR labuser
```
`Password expiration:                Jan06/00:00`


---

## Gestion des mots de passe des comptes utilisateurs

### La commande `passwd`

- Changer le mot de passe du compte utilisateur courant
```bash
passwd
```

- Changer le mot de passe d'un autre utilisateur (privilège root)
```bash
passwd # nom de l'utilisateur
```

- Forcer le changement du mot de passe à la prochaine connexion
```bash
passwd -e # nom de l'utilisateur
```

- verrouiller le compte utilisateur
```bash
passwd -l # nom de l'utilisateur
```

- Déverrouiller le compte utilisateur
```bash
passwd -u # nom de l'utilisateur
```
_Il faudra redéfinir un mot de passe_

- Afficher les paramètres de mot de passe d'un utilisateur
```bash
passwd -S # nom de l'utilisateur
```

### La commande `chage`

- `-l` &rarr; Afficher la politique de validité du mot de passe
- `-M` &rarr; Nombre de jours maximum entre les changements
- `-m` &rarr; Nombre de jours minimum entre les changements
- `-W` &rarr; Nombre de jours d'avertissement avant expiration
- `-E` &rarr; Date d'expiration du compte (YYYY-MM-DD)

#### Exemple

- Définir 90 jours maximum entre les changements, un avertissement à 14 jours avant le changement obligatoire et une date d'expiration du compte au 31 décembre 2026 pour l'utilisateur _labuser_
```bash
chage -M 90 -W 14 -E 2026-12-31 labuser
```

- Vérification
```bash
chage -l labuser
```
```
Last password change					: Jan 05, 2026
Password expires					: Apr 05, 2026
Password inactive					: never
Account expires						: Dec 31, 2026
Minimum number of days between password change		: 0
Maximum number of days between password change		: 90
Number of days of warning before password expires	: 14
```

### Le fichier `/etc/security/pwquality.conf` du module PAM `pam_pwquality` pour la gestion de la qualité des mots de passe

```bash
vim /etc/security/pwquality.conf
```

- Pour renforcer la complexité du mot de passe du compte _root_
```
enforce_for_root
```
- Pour définir une longueur minimal acceptée
```
minlen = x
```
- Pour forcer l'utilisation d'au moins un caractère en majuscule
```
ucredit = -1
```
- Pour forcer l'utilisation d'exactement 3 chiffres
```
dcredit = 3
```
- Pour forcer l'utilisation d'au moins 2 types de caractères
```
minclass = 2
```
- Pour empêcher la réutilisation de 4 caractères de l'ancien mot de passe
```
difok = 4
```
- Pour empêcher l'utilisation du nom d'utilisateur dans le mot de passe
```
usercheck = 1
```

### Le fichier `/etc/login.defs` pour définir les paramètres par défaut pour la création des comptes utilisateurs et des groupes

```bash
vim /etc/login.defs
```
- Définir à 90 jours maximum l'obligation de changer de mot de passe
```
PASS_MAX_DAYS   90
```

- Définir la longueur minimal des mots de passe à 10 caractères
```
PASS_MIN_LEN    10
```

- Définir le nombre maximum d'essais en cas d'erreur de mot de passe à 5
```
LOGIN_RETRIES          5
```

- Modifier la méthode de chiffrement des mots de passe pour utiliser SHA512
```
ENCRYPT_METHOD SHA512
```

- Forcer la commande `useradd` à ne pas créer par défaut le dossier home de l'utilisateur
```
CREATE_HOME     no
```

---

## Gestion des groupes

### Les commande `groupadd`, `groupdel` et `groupmod`

- Créer un groupe
```bash
groupadd # nom du groupe
```

- Créer un groupe avec un GID spécifique
```bash
groupadd -g xxxx # nom du groupe
```

- Créer un groupe en y ajoutant des utilisateurs
```bash
groupadd -U "user1,user2" # nom du groupe
```

- Supprimer un groupe
```bash
groupdel # nom du groupe
```

- Changer le GID d'un groupe
```bash
groupmod -g xxxx # nom du groupe
```

---

## Gérer les membres des groupes

- Lister les groupes
```bash
getent group
```

- Lister les membres d'un groupe
```bash
getent group # nom du groupe
```

### La commande `gpasswd`

- Etablir une liste d'utilisateurs pour un groupe
```bash
gpasswd -M "user1,user2" # nom du groupe
```

- Supprimer un utilisateur d'un groupe
```bash
gpasswd -d # nom d'utilisateur # nom du groupe
```

- Définir un administrateur du groupe
```
gpasswd -A # nom d'utilisateur # nom du groupe
```

---

## Les fichiers importants

### `/etc/passwd`

Accessible en lecture à tous, contient une liste d'utilisateurs, chacun sur une ligne distincte. Chaque ligne contient une liste séparée par deux-points.
```
labuser:x:1001:1001:utilisateur test:/home/labuser:/bin/bash
```
- `labuser` &rarr; nom d'utilisateur
- `x` &rarr; mot de passe caché
- `1001:1001` &rarr; UID:GID
- `utilisateur test` &rarr; commentaire
- `/home/labuser` &rarr; chemin vers le dossier _home_
- `/bin/bash` &rarr; shell

### `/etc/group`

Le fichier /etc/group est accessible en lecture à tous et contient une liste de groupes, chacun sur une ligne distincte. Chaque ligne contient une liste de quatre champs, séparés par des deux-points.
```
labgrp:x:2500:labuser,labuser2
```

- `labgrp` &rarr; nom du groupe
- `x` &rarr; mot de passe caché
- `2500` &rarr; GID
- `labuser,labuser2` &rarr; membres du groupe

### `/etc/shadow` et `/etc/gshadow`

Ces deux fichiers contiennent les "hashs" des mots de passe utilisateurs et groupes

- `/etc/shadow`
```
labuser:$y$j9T$7pC.ttWWTnAQ.eEyZ0R.61$wYxE3ASBST8rhZt354ImsOoAAvnu0J9cVJu2AW7nbl4:20459:0:90:14::20459:
```

- `/etc/gshadow`
```
labgrp:$y$j9T$lxS6H2tLiumIxPVJ92qui0$FOcOqKj7sAEYLPFLtDFJH72lxODV8jUp74HB5us6vs5::labuser,labuser2
```

---

_Sources principales:_

- https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/configuring_basic_system_settings/managing-users-and-groups_configuring-basic-system-settings