# RHEL 10 - La commande `find`

---

**La commande `find` est une commande Linux très utile, notamment lorsqu'on est confronté à un grand nombre de fichiers et de dossiers. Elle permet de trouver des éléments, par leur nom, leur type, leur taille, leur date et bien d'autres paramètres.**

---

## Structure basique de la commande

```bash
find /chemin/où/chercher test action
```

### Critères basiques

- `-name` &rarr; par nom
- `-iname` &rarr; par nom (ignorer la casse)
- `-type` &rarr; par type
- `-size` &rarr; par taille
- `-user` &rarr; par utilisateur propriétaire
- `-group` &rarr; par groupe propriétaire
- `-perm` &rarr; par permissions
- `-empty` &rarr; sans contenu
- `-executable` &rarr; fichiers exécutables ou dossier traversables

#### Exemples simples

- Chercher tous les fichiers ayant l'extension `.conf` dans le dossier `/etc`
```bash
find /etc -name "*.conf"
```

- Chercher tous les fichiers cachés dans le dossier `/home/labuser1`
```bash
find /home/labuser1/ -type f -name ".*"
```

- Chercher tous les fichiers de plus de 50Ko dans le dossier `/var/log`
```bash
find /var/log -type f -size +50k
```

- Chercher tous les liens symboliques appartenant à `root` dans le dossier `/dev/disk`
```bash
find /dev/disk -user root -type l
```

- Chercher tous les dossiers ayant pour permissions exactes `755` dans le dossier `/var/www`
```bash
find /var/www -type d -perm 755
```

### Utilisations des opérateurs et combinaisons

Opérateur logique "ou" &rarr; `-o`
- Chercher tous les dossiers nommés exactement `yum` ou contenant `yum` et finissant par `.d`
```bash
find /etc -type d -name "yum" -o -name "*yum*.d"
```

Opérateur logique "et" &rarr; `-a`
- Chercher tous les fichiers ayant pour extension `.log` et tous les dossiers appartenant à l'utilisateur `root` non-vides dans le dossier `/var/log`
```bash
find /var/log \( -type f -name "*.log" -o -type d -user root \) -a -not -empty
```

Opérateur logique "non" &rarr; `\!` ou `-not`
- Chercher tous les fichiers ne finissant pas par `.log` dans le dossier `/var/log`
```bash
find /var/log \! -name "*.log" -a -type f
```

### Récursivité

- `-maxdepth` &rarr; stoppe le traitement à partir du niveau _n_
- `-mindepth` &rarr; ne commence à traiter qu'à partir du niveau _n_

**Toujours placer ces options en premier**

#### Exemples

- Chercher tous les fichiers dans le dossier `/var/log` sans descendre dans l'arborescence
```bash
find /var/log -maxdepth 1 -type f
```

- Chercher tous les fichiers ayant pour extension `.conf` dans les sous-dossiers du dossier `/etc` en excluant le dossier lui-même
```bash
find /etc -mindepth 2 -type f -name "*.conf"
```

### Utiliser les critères de temps

- `-mtime` &rarr; modifié _+/-n_ jours
- `-atime` &rarr; accédé _+/-n_ jours
- `-ctime` &rarr; créé _+/-n_ jours
- `-mmin` &rarr; modifié _+/-n_ minutes
- `-amin` &rarr; accédé _+/-n_ minutes
- `-cmin` &rarr; créé _+/-n_ minutes

#### Exemples

- Chercher les fichiers modifiés depuis moins d'une heure dans le dossier `/var/log`
```bash
find /var/log -type f -mmin -60
```

- Chercher les fichiers créés depuis moins d'un jour dans le dossier `/etc/`
```bash
find /etc -type f -ctime -1
```

- Chercher les dossiers accédés depuis plus d'une heure dans `/tmp`
```bash
find /tmp -type d -amin +60
```

### Utiliser les critères de permissions

- `-perm xxx` &rarr; permissions exactes
- `-perm -xxx` &rarr; au minimum cette permission
- `-perm /xxx` &rarr; au moins une permission

#### Exemple

- Chercher les dossiers ayant exactement les permissions `755` dans le dossier `/home/labuser1`
```bash
find /home/labuser1/ -type d -perm 755
```

- Chercher les fichiers dont tous les utilisateurs ont au minimum les droits d'exécution dans le dossier `/usr/bin`
```bash
find /usr/bin/ -type f -perm -111
```

- Chercher les dossiers où le groupe propriétaire ou les autres ont au minimum le droit en lecture dans `/var/www`
```bash
find /var/www -type d -perm /044
```

- Chercher tous les fichiers à la racine du système ayant un SUID activé
```bash
find / -type f -perm /4000
```

- Chercher tous les dossiers à la racine du système ayant un SGID activé
```bash
find / -type d -perm /2000
```

### Utiliser des actions

- Chercher et supprimer tous les fichiers ayant pour extension `.jpg` dans le dossier `/home/labuser1`
```bash
find /home/labuser1/ -type f -name "*.jpg" -delete
```

- Chercher et copier dans `backup/` tous les fichiers non-vides du dossier `/var/log` sans récursivité
```bash
find /var/log -maxdepth 1 -type f -not -empty -exec cp {} backup/ \;
```
`{}` &rarr; le résultat de la recherche et `\;` &rarr; marque la fin de la commande

- Chercher tous les fichiers non-vides du dossier `/var/log` avec une récursivité de 2 niveaux et écrire le résultat dans `sortie.txt` 
```bash
find /var/log -maxdepth 2 -type f -not -empty -fprint sortie.txt
```
_Le fichier `sortie.txt` sera écrasé s'il existait_

---

_Sources principales:_

- https://www.redhat.com/en/blog/linux-find-command
- `man find`