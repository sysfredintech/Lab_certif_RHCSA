# RELH 10 - Les Quotas

---

## Concept

**Limiter l'utilisation de l'espace de stockage avec les quotas**
> Vous devez activer les quotas de disque sur votre système avant de pouvoir les attribuer. Vous pouvez attribuer des quotas de disque par utilisateur, par groupe ou par projet. Toutefois, si une limite souple est définie, vous pouvez dépasser ces quotas pendant une période configurable, appelée période de grâce.

- La limite soft est une limite d'avertissement au-delà de laquelle l'utilisateur pourra continuer à écrire sur le disque jusqu'à la limite hard et durant la période de grâce
- Les quotas définis pour un groupe s'appliquent de manière collective à l'ensemble des utilisateurs de ce groupe (quota partagé)
- Les quotas définis en inodes agissent sur le nombre de fichiers

---

## Pratique

### Mise en place d'un contexte pour le lab

- 1 disque /dev/sdb de 4Go dédié au lab
- 1 partition /dev/sdb1 de 2Go de type ext4 montée sur /mnt/datalab1
- 1 espace de 2Go non utilisé sur le disque /dev/sdb
- 2 utilisateurs _labuser1_ et _labuser2_ membres du groupe _grpquotas_
- 1 utilisateur _labuser3_ non membre du groupe _grpquotas_

```bash
lsblk | grep sdb
```
```
sdb             8:16   0     4G  0 disk 
└─sdb1          8:17   0     2G  0 part /mnt/datalab1
```
```bash
mount | grep sdb
```
`/dev/sdb1 on /mnt/datalab1 type ext4 (rw,relatime,seclabel)`

### Objectifs

1. Implémenter les quotas sur _/dev/sdb1_ et définir le  groupe _grpquotas_ comme propriétaire avec un SGID
2. Créer une partition de type xfs de 2Go avec prise en charge des quotas sur _/dev/sdb_ et la montée sur _/mnt/datalab2_ et donner les permissions complètes à _labuser3_
3. Assigner un quota au groupe _grpquotas_ sur _/mnt/datalab1_: soft = 800Mo - hard = 1Go
4. Assigner un quota à l'utilisateur _labuser3_ sur _/mnt/datalab2_: soft = 100Mo - hard = 150Mo
5. Créer un projet nommé _projetlab_ avec l'ID _50_, lui associer le groupe _projetlab_ dont l'utilisateur _labuser4_ et _labuser5_ sont membres et lui assigner un quota de 50 inodes _soft_ et 60 _hard_, 1Go de limite _soft_ et 1.5Go de limite _hard_  sur _/mnt/datalab2/projetlab_
6. Définir une période de grâce de 3 jours en blocs pour _/mnt/datalab1_ et 5 jours en inodes pour _/mnt/datalab2_

### Mise en pratique

**Le paquet _quota_ sera installé ou non en fonction des options d'installation du système d'exploitation**

- Vérification et installation éventuelle du paquet _quota_
```bash
rpm -qa | grep quota
```
Si besoin
```bash
dnf install quota
```

1. Implémenter les quotas sur _/dev/sdb1_ et définir le  groupe _grpquotas_ comme propriétaire avec un SGID
- Démonter la partition
```bash
umount /mnt/datalab1
```
- Activer les quotas sur la partition

Modifier le système de fichier
```bash
tune2fs -O quota /dev/sdb1
tune2fs -Q usrquota,grpquota /dev/sdb1
```
Modifier le montage de _/dev/sdb1_
```bash
vim /etc/fstab
```
`/dev/sdb1                                 /mnt/datalab1           ext4    defaults,usrquota,grpquota      0 0`

Remonter la partition
```bash
systemctl daemon-reload
mount -av
```
Si le message `You just mounted a file system that supports labels which does not contain labels, onto an SELinux box...` apparaît:
```bash
restorecon -Rv /mnt/datalab1/
```
- Définir le groupe _grpquotas_ comme propriétaire du dossier avec un SGID
```bash
chgrp grpquotas /mnt/datalab1
chmod 2775 /mnt/datalab1
```

- Initialiser les quotas

Pour un système de fichier ext4
```bash
quotaon /mnt/datalab1/
```
- Vérification
```bash
quotaon -p /mnt/datalab1/
```
```
group quota on /mnt/datalab1 (/dev/sdb1) is on
user quota on /mnt/datalab1 (/dev/sdb1) is on
project quota on /mnt/datalab1 (/dev/sdb1) is off
```
2. Créer une partition de type xfs de 2Go avec prise en charge des quotas sur _/dev/sdb_ et la montée sur _/mnt/datalab2_ et donner les permissions complètes à _labuser3_

- Création de la partition
```bash
umount /mnt/datalab1
fdisk /dev/sdb
```
`n` &rarr; `2` &rarr; `enter` &rarr; `enter` &rarr; `t` &rarr; `20` &rarr; `w`
```bash
mkfs.xfs /dev/sdb2
```
- Montage de la partition
```
mkdir /mnt/datalab2
vim /etc/fstab
```
`/dev/sdb2                                 /mnt/datalab2           xfs     defaults,usrquota,grpquota,prjquota   0 0`
```bash
mount -av
restorecon /mnt/datalab2
setfacl -m u:labuser3:rwx /mnt/datalab2
```

- Vérification
```bash
xfs_quota -x -c "report -h" /mnt/datalab2
```
```
User quota on /mnt/datalab2 (/dev/sdb2)
                        Blocks              
User ID      Used   Soft   Hard Warn/Grace   
---------- --------------------------------- 
root            0      0      0  00 [------]

Group quota on /mnt/datalab2 (/dev/sdb2)
                        Blocks              
Group ID     Used   Soft   Hard Warn/Grace   
---------- --------------------------------- 
root            0      0      0  00 [------]

Project quota on /mnt/datalab2 (/dev/sdb2)
                        Blocks              
Project ID   Used   Soft   Hard Warn/Grace   
---------- --------------------------------- 
#0              0      0      0  00 [------]
```

3. Assigner un quota au groupe _grpquotas_ sur _/mnt/datalab1_: soft = 800Mo - hard = 1Go

```bash
edquota -g grpquotas
```
```
Quotas disque pour group grpquotas (gid 1012) :
 Système de fichiers           blocs       souple     stricte   inodes    souple   stricte
  /dev/sdb1                         0          0          0          0        0        0
  /dev/sdb2                         0          0          0          0        0        0
```
- `Système de fichiers` = nom du périphérique concerné par les quotas appliqués
- `blocs` = blocs déjà utilisés par l'utilisateur (ou le groupe)
- `souple` = limite soft en Ko
- `stricte` = limite hard en Ko
- `inodes` = inodes déjà utilisés par l'utilisateur (ou le groupe)
- `souple` = limite soft en inodes
- `stricte` = limite hard en inodes

```
Quotas disque pour group grpquotas (gid 1012) :
 Système de fichiers           blocs       souple     stricte   inodes    souple   stricte
  /dev/sdb1                         4     819200    1048576          1        0        0
  /dev/sdb2                         0          0          0          0        0        0
```

- Vérification
```bash
repquota -g /mnt/datalab1
```
```
*** Rapport pour les quotas group sur le périphérique /dev/sdb1
Période de sursis bloc : 7days ; période de sursis inode : 7days
                        Block limits                File limits
Groupe        utilisé souple stricte sursis utilisé souple stricte sursis
----------------------------------------------------------------------
root      --      16       0       0              1     0     0       
grpquotas --       4  819200 1048576              1     0     0  
```
- Test d'occupation d'espace disque
```bash
su - labuser1
dd if=/dev/urandom of=/mnt/datalab1/bigfile bs=1M count=900 status=progress
exit
```
```bash
repquota -g /mnt/datalab1
```
```
*** Rapport pour les quotas group sur le périphérique /dev/sdb1
Période de sursis bloc : 7days ; période de sursis inode : 7days
                        Block limits                File limits
Groupe        utilisé souple stricte sursis utilisé souple stricte sursis
----------------------------------------------------------------------
root      --      16       0       0              1     0     0       
grpquotas +-  921608  819200 1048576  7days       2     0     0
```
La limite _soft_ est dépassée pour le groupe _grpquotas_
```bash
su - labuser2
dd if=/dev/urandom of=/mnt/datalab1/bigfile2 bs=1M count=200 status=progress
```
```
1M count=200 status=progress
dd: erreur d'écriture dans '/mnt/datalab1/bigfile2': Débordement du quota d'espace disque
124+0 enregistrements lus
123+0 enregistrements écrits
130015232 octets (130 MB, 124 MiB) copiés, 0,313079 s, 415 MB/s
```
Le groupe _grpquotas_ a atteint la limite _hard_ et un message d'erreur est renvoyé

4. Assigner un quota à l'utilisateur _labuser3_ sur _/mnt/datalab2_: soft = 100Mo - hard = 150Mo

**Lancer l'utilitaire _xfs_quota_ avec l'option `-x` pour l'utilisation du mode expert**
```bash
xfs_quota -x /mnt/datalab2
```
Ou
```bash
xfs_quota -x /dev/sdb2
```
_On bascule dans le prompt xfs_quota `xfs_quota>`_
- Activer l'application des quotas sur le système de fichier courant
```bash
enable
```
- Afficher les informations de quotas d'un système de fichier
```bash
report
```
```
User quota on /mnt/datalab2 (/dev/sdb2)
                               Blocks                     
User ID          Used       Soft       Hard    Warn/Grace     
---------- -------------------------------------------------- 
root                0          0          0     00 [--------]

Group quota on /mnt/datalab2 (/dev/sdb2)
                               Blocks                     
Group ID         Used       Soft       Hard    Warn/Grace     
---------- -------------------------------------------------- 
root                0          0          0     00 [--------]

Project quota on /mnt/datalab2 (/dev/sdb2)
                               Blocks                     
Project ID       Used       Soft       Hard    Warn/Grace     
---------- -------------------------------------------------- 
#0                  0          0          0     00 [--------]
```
```bash
limit bsoft=100m bhard=150m labuser3
report -u
```
```
User quota on /mnt/datalab2 (/dev/sdb2)
                               Blocks                     
User ID          Used       Soft       Hard    Warn/Grace     
---------- -------------------------------------------------- 
root                0          0          0     00 [--------]
labuser3            0     102400     153600     00 [--------]
```

- Test d'utilisation de l'espace disque
```bash
su - labuser3
dd if=/dev/urandom of=/mnt/datalab2/fichier_90m bs=1M count=90
exit
```
```bash
xfs_quota -x -c "report -u /dev/sdb2"
```
```
User quota on /mnt/datalab2 (/dev/sdb2)
                               Blocks                     
User ID          Used       Soft       Hard    Warn/Grace     
---------- -------------------------------------------------- 
root                0          0          0     00 [--------]
labuser3        92160     102400     153600     00 [--------]
```
```bash
su - labuser3
dd if=/dev/urandom of=/mnt/datalab2/fichier_100m bs=1M count=100
exit
```
```
dd: erreur d'écriture dans '/mnt/datalab2/fichier_100m': Débordement du quota d'espace disque
61+0 enregistrements lus
60+0 enregistrements écrits
62914560 octets (63 MB, 60 MiB) copiés, 0,132047 s, 476 MB/s
```
_L'utilisateur a dépassé sa limite de quota et un message d'erreur lui est renvoyé_

- Vérification
```bash
xfs_quota -x -c "report -u /dev/sdb2"
```
```
User quota on /mnt/datalab2 (/dev/sdb2)
                               Blocks                     
User ID          Used       Soft       Hard    Warn/Grace     
---------- -------------------------------------------------- 
root                0          0          0     00 [--------]
labuser3       153600     102400     153600     00  [6 days]
```

5. Créer un projet nommé _projetlab_ avec l'ID _50_, lui associer le groupe _projetlab_ dont l'utilisateur _labuser4_ et _labuser5_ sont membres et lui assigner un quota de 50 inodes _soft_ et 60 _hard_, 1Go de limite _soft_ et 1.5Go de limite _hard_  sur _/mnt/datalab2/projetlab_

- Mise en place du contexte
```bash
mkdir /mnt/datalab2/projetlab
groupadd projetlab
usermod -aG projetlab labuser4
usermod -aG projetlab labuser5
chown labuser4:projetlab /mnt/datalab2/projetlab/
chmod 2775 /mnt/datalab2/projetlab/
```

- Créer le projet
```bash
echo "50:/mnt/datalab2/projetlab/" >> /etc/projects
echo "projetlab:50" >> /etc/projid
```

- Initialiser le dossier _projetlab_ avec quota
```bash
xfs_quota -x -c "project -s projetlab" /mnt/datalab2
```
```
Setting up project projetlab (path /mnt/datalab2/projetlab/)...
Processed 1 (/etc/projects and cmdline) paths for project projetlab with recursion depth infinite (-1).
```

- Définir les limites d'occupation d'espace disque
```bash
xfs_quota -x -c "limit -p bsoft=1g bhard=1536m projetlab" /dev/sdb2
```

- Définir les limites d'inodes
```bash
xfs_quota -x /dev/sdb2
```
```bash
limit -p isoft=50 ihard=60 projetlab
```

- Vérification
```bash
xfs_quota -x -c "report -p -i -b -h" /dev/sdb2
```
```
Project quota on /mnt/datalab2 (/dev/sdb2)
                        Blocks                            Inodes              
Project ID   Used   Soft   Hard Warn/Grace     Used   Soft   Hard Warn/Grace  
---------- --------------------------------- --------------------------------- 
#0           150M      0      0  00 [------]      5      0      0  00 [------]
projetlab       0     1G   1,5G  00 [------]      1     50     60  00 [------]
```

- Test de limitations des quotas
```bash
su - labuser4
dd if=/dev/urandom of=/mnt/datalab2/projetlab/bigfile bs=1M count=800 status=progress
exit
```
```bash
xfs_quota -x -c "report -p -i -b -h" /dev/sdb2
```
```
Project quota on /mnt/datalab2 (/dev/sdb2)
                        Blocks                            Inodes              
Project ID   Used   Soft   Hard Warn/Grace     Used   Soft   Hard Warn/Grace  
---------- --------------------------------- --------------------------------- 
#0           150M      0      0  00 [------]      5      0      0  00 [------]
projetlab    800M     1G   1,5G  00 [------]      2     50     60  00 [------]
```
```bash
su - labuser5
dd if=/dev/urandom of=/mnt/datalab2/projetlab/bigfile2 bs=1M count=800 status=progress
exit
```
```
500170752 octets (500 MB, 477 MiB) copiés, 1 s, 499 MB/s
dd: erreur d'écriture dans '/mnt/datalab2/projetlab/bigfile2': Aucun espace disponible sur le périphérique
737+0 enregistrements lus
736+0 enregistrements écrits
771751936 octets (772 MB, 736 MiB) copiés, 1,59896 s, 483 MB/s
``` 
_La limite hard est dépassée pour le projet et l'utilisateur reçoit un message d'erreur_

```bash
xfs_quota -x -c "report -p -i -b -h" /dev/sdb2
```
```
Project quota on /mnt/datalab2 (/dev/sdb2)
                        Blocks                            Inodes              
Project ID   Used   Soft   Hard Warn/Grace     Used   Soft   Hard Warn/Grace  
---------- --------------------------------- --------------------------------- 
#0           150M      0      0  00 [------]      5      0      0  00 [------]
projetlab    1,5G     1G   1,5G  00 [6 days]      3     50     60  00 [------]
```

```bash
su - labuser4
rm -f /mnt/datalab2/projetlab/bigfile2
touch /mnt/datalab2/projetlab/fichier{1..40}
exit
```
```bash
su - labuser5
touch /mnt/datalab2/projetlab/fichier{41..100}
```
`touch: impossible de faire un touch '/mnt/datalab2/projetlab/fichier59': Aucun espace disponible sur le périphérique`
_La limite d'inodes est dépassée, l'utilisateur reçoit un message d'erreur_

```bash
xfs_quota -x -c "report -p -i -b -h" /dev/sdb2
```
```
Project quota on /mnt/datalab2 (/dev/sdb2)
                        Blocks                            Inodes              
Project ID   Used   Soft   Hard Warn/Grace     Used   Soft   Hard Warn/Grace  
---------- --------------------------------- --------------------------------- 
#0           150M      0      0  00 [------]      5      0      0  00 [------]
projetlab  800,0M     1G   1,5G  00 [------]     60     50     60  00 [6 days]
```

6. Définir une période de grâce de 3 jours en blocs sur _/mnt/datalab1_ et 5 jours en inodes sur _/mnt/datalab2_

> Si un quota donné possède des limites « soft », vous pouvez modifier la période de grâce (la période pendant laquelle la limite « soft » peut être dépassée)

**Cette commande fonctionne sur les quotas pour les inodes ou les blocs pour tous les utilisateurs et groupes**

```bash
edquota -t
```
```
Sursis avant l'application des limites souples pour users :
Unités de temps peuvent être : days (jours), hours (heures), minutes, ou seconds
  Système de fichiers  période de sursis bloc  période de sursis inode
  /dev/sdb1                     7days                  7days
  /dev/sdb2                     7days                  7days
```
```
  /dev/sdb1                     3days                  7days
  /dev/sdb2                     7days                  5days
```

---

_Sources principales:_

- https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_file_systems/limiting-storage-space-usage-on-ext4-with-quotas
- https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_file_systems/limiting-storage-space-usage-on-xfs-with-quotas

- `man quotaon` `man edquota` `man repquota` `man xfs_quota`