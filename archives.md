# RHEL 10 - Archivage et compression

---

## Contexte pour le lab

- Création d'un dossier "archive" contenant 10 fichiers de 2Mo et d'un dossier vide
```bash
mkdir dossierlab && touch dossierlab/fichierlab{1..10}
for file in {1..10}; do dd if=/dev/zero of=dossierlab/fichierlab$file bs=1M count=2; done
mkdir dossierx
``` 

## Archives simples

- Création d'une archive
```bash
tar -cf archive.tar dossierlab/
```

- Examiner le contenu d'une archive
```bash
tar -tf archive.tar
```

- Extraire une archive dans un dossier
```bash
tar -xvf archive.tar -C dossierx/
```

## compression d'une archive

- Avec gzip (rapide, compression standard)

compresser
```bash
gzip archive.tar # -k pour conserver l'originale
```
Décompresser
```bash
gunzip archive.tar.gz # -k pour conserver l'originale
```

- Avec bzip2 (lent, meilleure compression)

compresser
```bash
bzip2 archive.tar # -k pour conserver l'originale
```
Décompresser
```bash
bunzip2 archive.tar.bz2 # -k pour conserver l'originale
```

## Commandes combinées

- Créer une archive compressée avec gzip
```bash
tar -czf archive.tar.gz dossierlab/
```

- Créer une archive compressée avec bzip2
```bash
tar -cjf archive.tar.bz2 dossierlab/
```

- Créer une archive compressée avec xz
```bash
tar -cJf archive.tar.xz dossierlab/
```

- Observation des performances de compression
```bash
ls -la | grep archive
```
```
-rw-r--r--.  1 root root  20981760 11 déc.  14:28 archive.tar
-rw-r--r--.  1 root root       262 11 déc.  14:28 archive.tar.bz2
-rw-r--r--.  1 root root     21085 11 déc.  14:28 archive.tar.gz
-rw-r--r--.  1 root root      3364 11 déc.  14:28 archive.tar.xz
```

- Décompression d'une archive compressée dans un dossier (quelque soit la méthode de compression)
```bash
tar -xf archive.tar.xz -C dossierx
```

## Manipulations avancées

- Ajouter un fichier à une archive non-compressée
```bash
touch dossierlab/fichierlab11 && dd if=/dev/zero of=dossierlab/fichierlab11 bs=1M count=2
tar -rf archive.tar dossierlab/fichierlab11
```

- Extraire un fichier d'une archive en tronquant la structure des dossiers
```bash
tar -xf archive.tar.xz dossierlab/fichierlab1 --strip-components=1
```

- Archiver et restaurer en conservant les permissions et le contexte SELinux
```bash
tar --selinux -cpf htmlZ.tar /var/www/html/
```
```bash
tar --selinux -xpf htmlZ.tar -C /
```

---

_Sources principales:_

- https://www.redhat.com/en/blog/taming-tar-command
- `man tar`