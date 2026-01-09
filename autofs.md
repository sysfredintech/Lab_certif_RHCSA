# RHEL 10 - Autofs

---

> Le service autofs permet de monter et de démonter automatiquement (à la demande) les systèmes de fichiers, ce qui permet d'économiser des ressources système. Il peut être utilisé pour monter des systèmes de fichiers tels que NFS, AFS, SMBFS, CIFS et les systèmes de fichiers locaux.

---

## Installation du paquet nécessaire et activation du service

- Vérifier si `autofs` est installé sur le système
```bash
rpm -qa autofs
```
`autofs-5.1.9-13.el10.x86_64`

- Si absent, l'installer
```bash
dnf install autofs
```

- Activation et démarrage du service
```bash
systemctl enable --now autofs
```

---

## Structure des fichiers de configuration

- Afficher les fichiers de configuration d'autofs
```bash
ls -l /etc | grep "auto*"
```
```
-rw-r--r--.  1 root root  16240 12 mai    2025 autofs.conf # fichier de configuration du service autofs
-rw-------.  1 root root    232 12 mai    2025 autofs_ldap_auth.conf # pour autoriser une authentification sur un serveur ldap
-rw-r--r--.  1 root root   1342 19 déc.  11:01 auto.master # fichier principal
drwxr-xr-x.  2 root root      6 12 mai    2025 auto.master.d # dossier pour inclure des configurations supplémentaires
-rw-r--r--.  1 root root    519 12 mai    2025 auto.misc # fichier d'exemple
-rwxr-xr-x.  1 root root    901 12 mai    2025 auto.net # fichier d'exemple
-rwxr-xr-x.  1 root root   2087 12 mai    2025 auto.smb # fichier d'exemple
```

---

## Fonctionnement d'autofs

- Un fichier de configuration principal &rarr; `auto.master` qui répertorie les points de montage gérés par autofs, ainsi que leurs fichiers de configuration ou sources réseau correspondants, appelés maps.
- Les fichiers de maps &rarr; `auto.nomdumontage` définissent les propriétés des points de montage à la demande. Autofs crée les dossiers s'ils n'existent pas. Si les dossiers existaient avant le montage, ils ne seront pas supprimés au démontage. Si un délai d'expiration est spécifié, le dossier est automatiquement démonté s'il n'est pas utilisé pendant ce délai.

---

## Mise en pratique

Contexte du lab:
- Une partition ext4 `sdd1` disponible
- Un serveur de fichier NFS configuré avec un partage `/srv/nfs        192.168.10.0/24(rw,sync,no_subtree_check)` ayant pour adresse `192.168.10.246`
- Un serveur de fichier samba configuré avec un partage `partage` accessible en lecture/écriture pour l'utilisateur `labuser1` dont le mot de passe est `redhat` ayant pour adresse `192.168.10.246`

### Montage d'une partition locale

- Configuration du `auto.master`
```bash
vim /etc/auto.master
```
On ajoute la ligne suivante:
```
/mnt/data    /etc/auto.datalab  --timeout=120
```

- Création du fichier `auto.datalab`
```bash
vim /etc/auto.datalab
```
On l'édite comme suit:
```
data_local      -fstype=ext4,rw :/dev/sdd1
```
`data_local` &rarr; le nom du dossier qui sera créé
`-fstype=ext4,rw` &rarr; le type de système de fichier et les options
`:/dev/sdd1` &rarr; le chemin vers la partition à monter

- Redémarrer le service autofs
```bash
systemctl restart autofs.service
```

- Tester l'accès au dossier et le montage automatique
```bash
ls /mnt/data/data_local
df | grep data_local
```
`/dev/sdd1                 2023272      24    1902112   1% /mnt/data/data_local`

### Montage d'un partage NFS

- Configuration du `auto.master`
```bash
vim /etc/auto.master
```
On ajoute la ligne suivante:
```
/mnt/shares     /etc/auto.nfs   --timeout=60
```

- Edition du fichier `auto.nfs`
```bash
vim /etc/auto.nfs
```
On ajoute cette ligne:
```
data_nfs    -fstype=nfs,rw  192.168.10.246:/srv/nfs
```
`data_nfs` &rarr; le nom du dossier qui sera créé
`-fstype=nfs,rw` &rarr; le type de système de fichier et les options
`192.168.10.246:/srv/nfs` &rarr; le chemin vers le partage

- Redémarrer le service autofs
```bash
systemctl restart autofs.service
```

- Tester l'accès au dossier et le montage automatique
```
ls /mnt/shares/data_nfs
df | grep data_nfs
```
`192.168.10.246:/srv/nfs     7669888 1416704    5842048  20% /mnt/shares/data_nfs`


### Montage d'un partage Samba

- Installation des paquets nécessaires
```bash
dnf install cifs-utils
```

- Configuration du `auto.master`
```bash
vim /etc/auto.master
```
On ajoute la ligne suivante:
```
/mnt/samba      /etc/auto.cifs  --timeout=60
```

- Création du fichier `auto.cifs`
```bash
vim /etc/auto.cifs
```
On ajoute cette ligne:
```
data_smb        -fstype=cifs,credentials=/etc/auto.creds,iocharset=utf8,uid=1000,gid=1000,file_mode=0775,dir_mode=0775  ://192.168.10.246/partage
```
`data_smb` &rarr; le nom du dossier qui sera créé
`-fstype=cifs,credentials=/etc/auto.creds,iocharset=utf8,uid=1000,gid=1000,file_mode=0775,dir_mode=0775` &rarr; le type de système de fichier et les options
`://192.168.10.246/partage` &rarr; le chemin vers le partage


- Création du fichier contenant les identifiants pour l'utilisateur samba
```bash
vim /etc/auto.creds
```
On l'édite comme suit:
```
username=labuser1
password=redhat
```
On lui attribue les bonnes permissions:
```bash
chmod 600 /etc/auto.creds
```

- Redémarrer le service autofs
```bash
systemctl restart autofs.service
```

- Tester l'accès au dossier et le montage automatique
```
ls /mnt/samba/data_smb
mount | grep data_smb
```
`//192.168.10.246/partage on /mnt/samba/data_smb type cifs (rw,relatime,vers=3.1.1,cache=strict,upcall_target=app,username=labuser1,uid=1000,forceuid,gid=1000,forcegid,addr=192.168.10.246,file_mode=0775,dir_mode=0775,iocharset=utf8,soft,nounix,serverino,mapposix,reparse=nfs,nativesocket,symlink=native,rsize=4194304,wsize=4194304,bsize=1048576,retrans=1,echo_interval=60,actimeo=1,closetimeo=1)`

---

_Sources principales:_
- https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_file_systems/mounting-file-systems-on-demand
- `man 5 autofs`