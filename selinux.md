# RHEL10 - SELinux

---

SELinux fournit un système de sécurité granulaire qui implémente le contrôle d'accès obligatoire (MAC) au niveau du noyau Linux.


<ins>SELinux peut être activé ou désactivé totalement, si activé, deux modes possibles:</ins>
1. Enforcing = strict &rarr; applique les restrictions
>Le mode "Enforcing" est le mode de fonctionnement par défaut et recommandé. En mode "Enforcing", SELinux fonctionne normalement, appliquant la politique de sécurité chargée sur l'ensemble du système.
2. Permissive = permissif &rarr; Journalise uniquement
>En mode permissif, le système agit comme si SELinux appliquait la politique de sécurité chargée, notamment en étiquetant les objets et en émettant des entrées de refus d'accès dans les journaux, mais il ne refuse en fait aucune opération. Bien qu'il ne soit pas recommandé pour les systèmes de production, le mode permissif peut être utile pour le développement et le débogage de la politique SELinux.

<ins>Si SELinux est désactivé:</ins>

- Disabled = Désactivé &rarr; Dangereux pour l'intégrité du système
>Le mode désactivé est fortement déconseillé ; non seulement le système évite d'appliquer la politique SELinux, mais il évite également d'étiqueter les objets persistants tels que les fichiers, ce qui rend difficile l'activation de SELinux à l'avenir.

---

## Commandes essentielles pour prendre connaissance de l'état du système et le modifier si nécessaire

```bash
sestatus
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Memory protection checking:     actual (secure)
Max kernel policy version:      33
```
```bash
getenforce
```
Renvoi `Enforcing`

Ici: 
- `SELinux status:                 enabled`
- `Enforcing`

<ins>SELinux est activé et fonctionne en mode enforcing</ins>

**Pour passer en mode permissive de manière temporaire**

```bash
setenforce 0
```
- `setenforce` accèpete comme options `0` pour permissive ou `1` pour enforcing
- `getenforce` affiche l'état de la politique SELinux appliquée sur le système
```bash
getenforce
```
Renvoi: `Permissive`

<ins>Ce paramétrage sera réinitialisé au redémarrage du système pour repasser en mode Enforcing qui est le mode par défaut</ins>

**Pour changer de mode de façon permanente**

```bash
vim /etc/selinux/config
```
_Editer la ligne suivante_
```bash
SELINUX=enforcing
```
- `enforcing` = strict
- `permissive` = permissif
- `disabled` = désactivé

### Précaution importante lors du passage du mode permissive au mode enforcing

- En mode _disabled_ ou _permissive_ l'étiquetage SELinux ne sera pas correct
- Lors du retour en mode _enforcing_ des problèmes d'accès risquent de survenir

**Pour prévenir ce problème:**

1. Forcer le re-étiquetage complet au redémarrage du système
```bash
touch /.autorelabel
systemctl reboot
```
2. Ré-étiquetage manuel des éléments modifiés
```bash
# recursivement pour un dossier
restorecon -Rv /chemin/vers/modifications/
# ou pour un fichier
restorecon -v /chemin/vers/fichier_modifié
# ensuite on repasse en mode enforcing
setenforce 1
```

---

## Installation des paquets nécessaires pour définire la politique SELinux de façon granulaire

```bash
dnf install policycoreutils policycoreutils-gui libselinux-utils policycoreutils-python-utils setools-console checkpolicy setroubleshoot-server
```

---

## Les logs SELinux

1. Stockés dans `/var/log/audit/audit.log`
2. Gérés par le service `auditd`

- Consulter les logs avec informations détaillées

```bash
sealert -l "*" # | grep 'motif'
```

- Consulter les logs avec informations minimales

```bash
ausearch -m AVC --format text
```

---

## Ajouter un port d'écoute pour le service ssh

- Lister les ports valides pour le service ssh

```bash
semanage port -l | grep ssh
ssh_port_t                     tcp      22
```
_Seul le port 22 est admis_

- Ajouter le port tcp 2222 au type ssh_port_t 
```bash
semanage port -a -t ssh_port_t -p tcp 2222
```

- Vérification
```bash
semanage port -l | grep ssh
ssh_port_t                     tcp      2222, 22
```
_On peut maintenant définir le port 2222 comme port d'écoute dans /etc/ssh/sshd_config_

---

## Exercice pratique pour un partage de fichier samba en définissant la politique SELinux à l'aide d'un booléen

### Contexte

- Serveur samba actif et configuré avec un partage nommé _labshare_
```bash
[labshare]
        path = /srv/labshare
        browseable = yes
        writable = yes
        read only = no
        valid users = @labsamba
        guest ok = no
        create mask = 0664
        directory mask = 0775
```
- Un groupe _labsamba_ contenant 2 utilisateurs _labuser1_ et _labuser2_
```bash
cat /etc/group | grep labsamba
labsamba:x:1004:labuser1,labuser2
```
- Un dossier _/srv/labshare_ appartenant au groupe _labsamba_ avec les permissions _2775_ contenant 2 fichiers
```bash
ls -la /srv/labshare/
total 0
drwxrwsr-x. 2 root     labsamba 44 15 oct.  14:25 .
drwxr-xr-x. 3 root     root     22 15 oct.  14:20 ..
-rw-r--r--. 1 labuser1 labsamba  0 15 oct.  14:25 f_user1.txt
-rw-r--r--. 1 labuser2 labsamba  0 15 oct.  14:25 f_user2.txt
```
- Un fichier nommé _testsel1.txt_ dans /home/labuser1
```bash
ls -l /home/labuser1
total 0
-rw-r--r--. 1 labuser1 labuser1 0 15 oct.  15:00 testsel1.txt
```

### Test d'accès avec les paramètres par défaut de SELinux

```bash
su - labuser1
smbclient -U labuser1 //localhost/labshare
Password for [SAMBA\labuser1]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Oct 15 14:25:46 2025
  ..                                  D        0  Wed Oct 15 14:25:46 2025

		6430720 blocks of size 1024. 3279040 blocks available
smb: \> put testsel1.txt
NT_STATUS_ACCESS_DENIED opening remote file \testsel1.txt
smb: \> 
```

**L'accès en lecture et en écriture n'est pas permis dans le dossier partagé**

- Consulter des logs SELinux

```bash 
sealert -l "*"
```
```bash
SELinux interdit à /usr/sbin/smbd d'utiliser l'accès create sur le fichier testsel1.txt.

*****  Le greffon samba_share (78.9 de confiance) suggère   ******************

Si vous souhaitez autoriser smbd à accéder à create sur testsel1.txt file
Alors vous devez modifier l'étiquette sur « testsel1.txt »
Faire
# semanage fcontext -a -t samba_share_t 'testsel1.txt'
# restorecon  -v 'testsel1.txt'

*****  Le greffon catchall_boolean (12.8 de confiance) suggère   *************

Si vous souhaitez allow samba to share any file/directory read/write.
Alors vous devez en informer SELinux en activant le booléen « samba_export_all_rw ».

Faire
setsebool -P samba_export_all_rw 1
**Changement de la politique SELinux**
```

```bash
SELinux interdit à /usr/sbin/smbd d'utiliser l'accès getattr sur le fichier /srv/labshare/f_user1.txt.

*****  Le greffon samba_share (70.3 de confiance) suggère   ******************

Si vous souhaitez autoriser smbd à accéder à getattr sur f_user1.txt file
Alors vous devez modifier l'étiquette sur « /srv/labshare/f_user1.txt »
Faire
# semanage fcontext -a -t samba_share_t '/srv/labshare/f_user1.txt'
# restorecon  -v '/srv/labshare/f_user1.txt'

*****  Le greffon catchall_boolean (11.4 de confiance) suggère   *************

Si vous souhaitez allow samba to share any file/directory read only.
Alors vous devez en informer SELinux en activant le booléen « samba_export_all_ro ».

Faire
setsebool -P samba_export_all_ro 1

*****  Le greffon catchall_boolean (11.4 de confiance) suggère   *************

Si vous souhaitez allow samba to share any file/directory read/write.
Alors vous devez en informer SELinux en activant le booléen « samba_export_all_rw ».

Faire
setsebool -P samba_export_all_rw 1
```

```bash
ausearch -m AVC --format text
At 18:42:24 15/10/2025 system, acting as labuser1, unsuccessfully accessed-mac-policy-controlled-object using /usr/sbin/smbd
At 18:42:24 15/10/2025 system, acting as labuser1, unsuccessfully accessed-mac-policy-controlled-object using /usr/sbin/smbd
```

**Les logs indiquent que le contexte SELinux n'est pas défini pour autoriser l'accès et suggère l'activation du booléen _samba_export_all_rw_**

### Activation d'un booléen pour autoriser l'accès

- Pour lister tous les booléens et leur état

```bash
semanage boolean -l
```

- Informations sur le booléen concerné

```bash
semanage boolean -l | grep samba_export_all_rw
samba_export_all_rw            (fermé,fermé)  Allow samba to share any file/directory read/write.
```

**Etat actuel `fermé` - Etat par défaut `fermé`**

```bash
setsebool samba_export_all_rw 1
```

```bash
semanage boolean -l | grep samba_export_all_rw
samba_export_all_rw            (ouvert,fermé)  Allow samba to share any file/directory read/write.
```

**Etat actuel `ouvert` - Etat par défaut `fermé`**

### Test avec le nouveau paramétrage SELinux

```bash
smbclient //localhost/labshare -U labuser1
Password for [SAMBA\labuser1]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Oct 15 14:25:46 2025
  ..                                  D        0  Wed Oct 15 14:25:46 2025
  f_user1.txt                         N        0  Wed Oct 15 14:25:39 2025
  f_user2.txt                         N        0  Wed Oct 15 14:25:46 2025

		6430720 blocks of size 1024. 3279276 blocks available
smb: \> put testsel1.txt
putting file testsel1.txt as \testsel1.txt (0,0 kb/s) (average 0,0 kb/s)
smb: \> ls
  .                                   D        0  Wed Oct 15 19:09:41 2025
  ..                                  D        0  Wed Oct 15 19:09:41 2025
  f_user1.txt                         N        0  Wed Oct 15 14:25:39 2025
  f_user2.txt                         N        0  Wed Oct 15 14:25:46 2025
  testsel1.txt                        A        0  Wed Oct 15 19:09:41 2025

		6430720 blocks of size 1024. 3279304 blocks available
```

**On a accès en lecture et en écriture sur le partage**
_Cette configuration sera réinitialisée au reboot du système_

- Pour un paramétrage persistent

```bash
setsebool -P samba_export_all_rw 1
```

```bash
semanage boolean -l | grep samba_export_all_rw
samba_export_all_rw            (ouvert,ouvert)  Allow samba to share any file/directory read/write.
```

**Etat actuel `ouvert` - Etat par défaut `ouvert`**

- Pour revenir à la configuration originale et interdire l'accès

```bash
setsebool -P samba_export_all_rw 0
```

```bash
semanage boolean -l | grep samba_export_all_rw
samba_export_all_rw            (fermé,fermé)  Allow samba to share any file/directory read/write.
```

**Etat actuel `fermé` - Etat par défaut `fermé`**

---

## Exercice pratique pour configurer SELinux afin d'autoriser temporairement l'accés à une base postgresql en utilisant chcon

- Pré-requis

1. PostgreSQL installé et démarré
2. Un dossier de test _/opt/db_lab_ appartenant à l'utilisateur _postgres_

```bash
dnf install postgresql-server postgresql-contrib -y
postgresql-setup --initdb --unit postgresql
 * Initializing database in '/var/lib/pgsql/data'
 * Initialized, logs are in /var/lib/pgsql/initdb_postgresql.log
systemctl start postgresql
mkdir /opt/db_lab
chown -R postgres:postgres /opt/db_lab
```

- Tester l'utilisation du dossier _/opt/db_lab_ pour créer une table

```bash
sudo -u postgres psql -c "CREATE TABLESPACE external LOCATION '/opt/db_lab';"
ERREUR:  n'a pas pu configurer les droits du répertoire « /opt/db_lab » : Permission non accordée
```

**Cette opération n'est pas permise**

- Consulter les logs SELinux

```bash
sealert -l "*"
SELinux interdit à /usr/bin/postgres d'utiliser l'accès setattr sur le dossier db_lab.
Si vous souhaitez autoriser postgres à accéder à setattr sur db_lab directory
Alors l'étiquette sur db_lab doit être modifiée
Faire
# semanage fcontext -a -t FILE_TYPE 'db_lab'
où FILE_TYPE est l'une des valeurs suivantes : faillog_t, postgresql_db_t, postgresql_log_t, postgresql_tmp_t, postgresql_var_run_t.
```

- Modifier l'étiquette sur le dossier _/opt/db_lab_

**Pour connaitre l'étiquette à définir, on peut se référer au dossier par défaut pour les bases postgresql situées dans _/var/lib/pgsql/data/_**

```bash
matchpathcon /var/lib/pgsql/data/
/var/lib/pgsql/data	system_u:object_r:postgresql_db_t:s0
```

```bash
chcon -R -t postgresql_db_t /opt/db_lab/
```

- Tester après modification de la politique

```bash
sudo -u postgres psql -c "CREATE TABLESPACE external LOCATION '/opt/db_lab';"
CREATE TABLESPACE
sudo -u postgres psql -c "CREATE DATABASE db_lab TABLESPACE external;"
CREATE DATABASE
```
**L'opération s'est déroulée avec succès**

### Cette configuration est temporaire

Elle sera perdue:
- Au redémarrage du système 
- Au prochain `restorecon`

---

## Personnalisation de la politique SELinux pour le serveur HTTP Apache dans une configuration non standard

- Conditions préalables

1. Le service httpd configuré pour écouter sur le port TCP 8088 et pour utiliser le répertoire _/var/labwww/_
2. Un dossier _/var/labwww/html_ contenant un _index.html_

```bash
mkdir -p /var/labwww/html
touch /var/labwww/html/index.html
echo "<h1>Hello World</h1>" > /var/labwww/html/index.html
```
```bash
vim /etc/httpd/conf/httpd.conf
```
```bash
Listen 8088
#
DocumentRoot "/var/labwww/html"
#
<Directory "/var/labwww">
    AllowOverride None
    # Allow open access:
    Require all granted
</Directory>
# Further relax access to the default document root:
<Directory "/var/labwww/html">
```

- Démarrer le service httpd

```bash
systemctl start httpd
Job for httpd.service failed because the control process exited with error code.
See "systemctl status httpd.service" and "journalctl -xeu httpd.service" for details.
```

```bash
systemctl status httpd
oct. 15 10:16:11 rhel-srv httpd[5995]: (13)Permission denied: AH00072: make_sock: could not bind to address [::]:8088
oct. 15 10:16:11 rhel-srv httpd[5995]: (13)Permission denied: AH00072: make_sock: could not bind to address 0.0.0.0:8088
```

**L'utilisation du port 8088 n'est pas permise**

- Consulter les logs SELinux

```bash
sealert -l "*" | grep SELinux
SELinux interdit à /usr/sbin/httpd d'utiliser l'accès name_bind sur le tcp_socket port 8088.
```

- Lister les ports définis pour http par SELinux

```bash
semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
```
`http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000` &rarr; le port 8088 n'est pas listé pour le contexte http

- Ajouter le port 8088 au contexte http

```bash
semanage port -a -t http_port_t -p tcp 8088
```
```bash
semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      8088, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
```

- Démarrer le service httpd

```bash
systemctl start httpd
systemctl status httpd
oct. 15 10:28:15 rhel-srv httpd[6046]: Server configured, listening on: port 8088
```

- Tester l'accès

```bash
curl http://localhost:8088/index.html
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
</body></html>
```

**L'accès à cette ressource n'est pas permis**

- Consulter les logs SELinux

```bash
sealert -l "*" | grep SELinux
SELinux interdit à /usr/sbin/httpd d'utiliser l'accès getattr sur le fichier /var/labwww/html/index.html.
```

- Vérifier l'étiquette SELinux du fichier concerné

```bash
ls -Z /var/labwww/html/
unconfined_u:object_r:var_t:s0 index.html
```

- Vérifier le contexte de sécurité appliqué au processus httpd

```bash
ps -auxZ | grep httpd
system_u:system_r:httpd_t:s0    root        6046  0.0  0.6  25204 10936 ?        Ss   10:28   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    apache      6047  0.0  0.3  36080  5312 ?        S    10:28   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    apache      6048  0.0  0.4 1585336 7820 ?        Sl   10:28   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    apache      6049  0.0  0.4 1454200 8512 ?        Sl   10:28   0:00 /usr/sbin/httpd -DFOREGROUND
system_u:system_r:httpd_t:s0    apache      6051  0.0  0.4 1454200 7480 ?        Sl   10:28   0:00 /usr/sbin/httpd -DFOREGROUND
```

- Pour obtenir la liste des politiques SELinux relatives à http

```bash
seinfo -t | grep http
```

- Vérifier l'étiquette SELinux sur le dossier par défaut pour le contenu http

```bash
matchpathcon /var/www
/var/www	system_u:object_r:httpd_sys_content_t:s0
```

- Définir la bonne étiquette pour le dossier _/var/labwww_ 

```bash
semanage fcontext -a -t httpd_sys_content_t "/var/labwww(/.*)?"
```


**On a modifié le contexte pour le chemin _/var/labwww/_ et son contenu pour qu'ils soient traités comme du contenu web**

- Ce nouvel enregistrement doit être appliqué avec la commande suivante

```bash
restorecon -R -v /var/labwww
Relabeled /var/labwww from unconfined_u:object_r:var_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
Relabeled /var/labwww/html from unconfined_u:object_r:var_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
Relabeled /var/labwww/html/index.html from unconfined_u:object_r:var_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
```

- Vérifications

```bash
ls -dZ /var/labwww/html/
unconfined_u:object_r:httpd_sys_content_t:s0 /var/labwww/html/
```

```bash
curl http://localhost:8088/index.html
<h1>Hello World</h1>
```

**L'accès au contenu de _/var/labwww/_ sur le serveur http est permis**

### Cette configuration est permanente

---

## Bonnes pratiques

1. Vérification des logs avec `sealert` systématiquement
2. Favoriser l'utilisation des booléens lorsque cela est possible
3. Pour ne pas désactiver SELinux, passer en mode `permissive` lorsque le debuggage est nécessaire