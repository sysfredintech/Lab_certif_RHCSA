# RHEL10 - Gestion du swap

***

**Contexte:**

- RHEL10 dans une vm avec une partition swap par défaut sur un volume logique
- Un disque vierge supplémentaire ajouté à la vm (/dev/sdd)

**Objectifs:**

- Ajouter un fichier de swap de façon temporaire ou persistante
- Ajouter une partition de swap temporaire ou persistante
- Définir une priorité à un espace swap
- Créer un espace swap avec LVM
- Configuration du swappiness

---

## Prise en compte de la configuration actuelle

Lister les fichiers et partitions déjà utilisés pour le swap
```bash
swapon -s
```
```
Filename				Type		Size		Used		Priority
/dev/dm-1                               partition	839676		0		-2
```
`/dev/dm-1` est la partition de swap crée à l'installation du système

```bash
lsblk | grep -i swap
```
```
  └─rhel-swap    253:1    0   820M  0 lvm  [SWAP]
```
Il s'agit bien d'une partition de swap crée sur un LV nommé `rhel-swap` de 820Mo

---

## Créer un fichier de swap d'une taille définie et dédié à cet usage

1. Créer le fichier qui sera utilisé pour le swap

```bash
dd if=/dev/zero of=/swaplab1 bs=1M count=256
```
```
256+0 enregistrements lus
256+0 enregistrements écrits
268435456 octets (268 MB, 256 MiB) copiés, 0,0329612 s, 8,1 GB/s
```
_Il est possible de créer ce fichier à l'aide de la commande suivante également_
```bash
fallocate -l 256M /swaplab1
```

2. Lui appliquer les bonnes permissions

```bash
chmod 600 /swaplab1
```

**On a créé un fichier de 256Mo nommé swaplab1 et on lui a définie les permissions exigées par swapon**

3. Formater ce fichier au format swap avec pour label _swaplab1_ (le label est facultatif)

```bash
mkswap /swaplab1 -L swaplab1
```
```
Setting up swapspace version 1, size = 256 MiB (268431360 bytes)
LABEL=swaplab1, UUID=a12193c7-4cea-44c9-8428-af14ab8e05d3
```

4. Activer le fichier comme swap supplémentaire sur le système

```bash
swapon /swaplab1
```

5. Vérification

```bash
swapon -s
```
```
Filename				Type		Size		Used		Priority
/dev/dm-1                               partition	839676		0		-2
/swaplab1                               file		262140		0		-3
```

6. Pour activer cet espace swap au démarrage du système

```bash
vim /etc/fstab
```
Ajouter la ligne suivante
```
/swaplab1 swap swap defaults 0 0
```

---

## Ajouter une partition de swap au système

1. Identifier le disque disponible pour créer une partition de swap

```bash
lsblk
```
```
NAME             MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                8:0    0     8G  0 disk 
├─sda1             8:1    0     1M  0 part 
├─sda2             8:2    0     1G  0 part /boot
└─sda3             8:3    0     7G  0 part 
  ├─rhel-root    253:0    0   6,2G  0 lvm  /
  └─rhel-swap    253:1    0   820M  0 lvm  [SWAP]
sdb                8:16   0     4G  0 disk 
└─sdb1             8:17   0     4G  0 part 
  └─Labo1-lvlab1 253:2    0   2,6G  0 lvm  
sdc                8:32   0     4G  0 disk 
└─sdc1             8:33   0     4G  0 part 
  └─Labo1-lvlab2 253:3    0   2,6G  0 lvm  
sdd                8:48   0     4G  0 disk 
sr0               11:0    1 816,4M  0 rom
```
Ici on utilise sdd `sdd                8:48   0     4G  0 disk`

2. Créer une partition dédiée

```bash
fdisk /dev/sdd
```
```
Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): 
```
`n` &rarr; `p` &rarr; `1` &rarr; `Enter` &rarr; `+256M` &rarr; `t` &rarr; `82` &rarr; `w`

On a créé une partition primaire de 256Mo de type Linux Swap

Vérification
```bash
lsblk | grep sdd
```
```
sdd                8:48   0     4G  0 disk 
└─sdd1             8:49   0   256M  0 part
```

3. Donner un formatage swap à cette partition (le label est facultatif)

```bash
mkswap /dev/sdd1 -L swaplab
```
```
Setting up swapspace version 1, size = 256 MiB (268431360 bytes)
LABEL=swaplab, UUID=5e033386-f145-4bcb-9590-062d9faf5fd8
```

4. Activer cette partition comme swap supplémentaire

```bash
swapon /dev/sdd1
```

5. Vérification

```bash
swapon -s
```
```
Filename				Type		Size		Used		Priority
/dev/dm-1                               partition	839676		0		-2
/swaplab1                               file		262140		0		-3
/dev/sdd1                               partition	262140		0		-4
```
---

## Désactiver un espace swap

1. Lister les swaps utilisés

```bash
swapon -s
```
```
Filename				Type		Size		Used		Priority
/dev/dm-1                               partition	839676		0		-2
/swaplab1                               file		262140		0		-3
/dev/sdd1                               partition	262140		0		-4
```

2. Désactiver le swap ciblé

```bash
swapoff /dev/sdd1
```
```bash
swapon -s
```
```
Filename				Type		Size		Used		Priority
/dev/dm-1                               partition	839676		0		-2
/swaplab1                               file		262140		0		-3
```
On a désactivé le swap `/dev/sdd1`

---

## Définir un swap avec une priorité

1. Activer /dev/sdd1 comme swap avec une priorité plus élevée

```bash
swapon /dev/sdd1 -p 50
```
`-p 50` indique une valeur arbitraire entre 0 et 32767
> priority is a value between 0 and 32767. Higher numbers indicate higher priority

2. Vérification

```bash
swapon -s
```
```
Filename				Type		Size		Used		Priority
/dev/dm-1                               partition	839676		0		-2
/swaplab1                               file		262140		0		-3
/dev/sdd1                               partition	262140		0		50
```
`/dev/sdd1                               partition	262140		0		50` &larr; ce swap à une priorité supérieure aux autres

---

## Configuration persistante d'un espace swap avec fstab

1. Identifier l'UUID d'une partition de swap créé précédemment

```bash
blkid | grep swap
```
```
/dev/mapper/rhel-swap: UUID="512ad036-a4f3-40cb-ba3e-2133be9b2589" TYPE="swap"
/dev/sdd1: UUID="5e033386-f145-4bcb-9590-062d9faf5fd8" TYPE="swap" PARTUUID="c598ece6-01"
```
Ici `/dev/sdd1: UUID="5e033386-f145-4bcb-9590-062d9faf5fd8" TYPE="swap" PARTUUID="c598ece6-01"`

2. Éditer le fichier /etc/fstab et ajouter la partition swap identifié

```bash
vim /etc/fstab
```
```bash
# /etc/fstab
# Created by anaconda on Tue Oct  7 11:48:24 2025
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=468c8cc3-6a09-4743-82a0-3129ac7aaf43 /                       xfs     defaults        0 0
UUID=b6c7b31f-85a0-4e9b-9c9b-c46405c01497 /boot                   xfs     defaults        0 0
UUID=512ad036-a4f3-40cb-ba3e-2133be9b2589 none                    swap    defaults        0 0

# Custom swap
UUID=5e033386-f145-4bcb-9590-062d9faf5fd8 none                    swap    defaults        0 0
```
3. Vérifications

```bash
swapoff /dev/sdd1
```
```bash
swapon -av
```
```
swapon: /dev/mapper/rhel-swap: already active -- ignored
swapon: /dev/sdd1: found signature [pagesize=4096, signature=swap]
swapon: /dev/sdd1: pagesize=4096, swapsize=268435456, devsize=268435456
swapon /dev/sdd1
```
```bash
swapon -s
```
```
Filename				Type		Size		Used		Priority
/dev/dm-1                               partition	839676		0		-2
/swaplab1                               file		262140		0		-3
/dev/sdd1                               partition	262140		0		-4
```
Le swap `/dev/sdd1` est bien activé

4. Pour définir une priorité à un swap via fstab

Modifier la ligne ajoutée pour le nouvel espace de swap dans /etc/fstab
```bash
UUID=5e033386-f145-4bcb-9590-062d9faf5fd8 none                    swap    defaults,pri=50         0 0
```
L'option `pri=50` définit une priorité plus élevée (cette valeur est arbitraire) `priority is a value between 0 and 32767. Higher numbers indicate higher priority`

5. Vérifications

```bash
swapoff /dev/sdd1
```
```bash
swapon -av
```
```
swapon: /dev/mapper/rhel-swap: already active -- ignored
swapon: /dev/sdd1: found signature [pagesize=4096, signature=swap]
swapon: /dev/sdd1: pagesize=4096, swapsize=268435456, devsize=268435456
swapon /dev/sdd1
```
```bash
swapon -s
```
```
Filename				Type		Size		Used		Priority
/dev/dm-1                               partition	839676		0		-2
/swaplab1                               file		262140		0		-3
/dev/sdd1                               partition	262140		0		50
```
---

## Créer un volume logique dédié au swap

1. Supprimer l'espace swap précédemment ajouté

```bash
swapoff /dev/sdd1
```
Supprimer la ligne suivante dans /etc/fstab:
```
UUID=5e033386-f145-4bcb-9590-062d9faf5fd8 none                    swap    defaults,pri=50         0 0
```
2. Supprimer la partition /dev/sdd1

```bash
fdisk /dev/sdd
```
`d` &rarr; `w`

3. Créer un VG supplémentaire avec /dev/sdd comme PV

```bash
vgcreate vgswap /dev/sdd
```
```
  Physical volume "/dev/sdd" successfully created.
  Volume group "vgswap" successfully created
```

4. Créer un LV dédié au swap sur le VG précédemment créé avec la taille souhaitée (ici 256Mo)

```bash
lvcreate vgswap -n swaplab2 -L 256M
```
```
  Logical volume "swaplab2" created.
```

5. Formater le LV swaplab2 comme nouvel espace swap

```bash
lvdisplay vgswap/swaplab2 | grep -i path
```
```
  LV Path                /dev/vgswap/swaplab2
```
```bash
mkswap /dev/vgswap/swaplab2
```
```
Setting up swapspace version 1, size = 256 MiB (268431360 bytes)
no label, UUID=9a939711-d74c-418e-8607-646b14fd7b34
```

6. Pour une configuration persistante, éditer /etc/fstab pour y ajouter l'espace swap nouvellement créé en ajoutant cette ligne:

```bash
/dev/vgswap/swaplab2                      swap                    swap    defaults        0 0
```

7. Tester le montage de la nouvelle partition de swap

```bash
swapon -av
```
```
swapon: /dev/mapper/rhel-swap: already active -- ignored
swapon: /dev/mapper/vgswap-swaplab2: found signature [pagesize=4096, signature=swap]
swapon: /dev/mapper/vgswap-swaplab2: pagesize=4096, swapsize=268435456, devsize=268435456
swapon /dev/mapper/vgswap-swaplab2
```

8. Vérifications

```bash
swapon -s
```
```
Filename				Type		Size		Used		Priority
/dev/dm-1                               partition	839676		0		-2
/dev/dm-4                               partition	262140		0		-3
```
```bash
lsblk | grep -i swap
```
```
  └─rhel-swap     253:1    0   820M  0 lvm  [SWAP]
└─vgswap-swaplab2 253:4    0   256M  0 lvm  [SWAP]
```

---

## Commandes de debugging et de surveillance

1. Au niveau des logs du kernel

```bash
dmesg | grep -i swap | tail -5
```
```
[    2.776404] systemd[1]: Activating swap dev-disk-by\x2duuid-512ad036\x2da4f3\x2d40cb\x2dba3e\x2d2133be9b2589.swap - /dev/disk/by-uuid/512ad036-a4f3-40cb-ba3e-2133be9b2589...
[    2.795855] Adding 839676k swap on /dev/mapper/rhel-swap.  Priority:-2 extents:1 across:839676k 
[    2.836042] systemd[1]: Activated swap dev-disk-by\x2duuid-512ad036\x2da4f3\x2d40cb\x2dba3e\x2d2133be9b2589.swap - /dev/disk/by-uuid/512ad036-a4f3-40cb-ba3e-2133be9b2589.
[ 2600.482309] Adding 262140k swap on /dev/sdd1.  Priority:50 extents:1 across:262140k SS
[ 4342.437058] Adding 262140k swap on /dev/mapper/vgswap-swaplab2.  Priority:-3 extents:1 across:262140k SS

```

2. Vérification d'un fichier de swap 

```bash
file /swaplab1
```
```
/swaplab1: Linux swap file, 4k page size, little endian, version 1, size 65535 pages, 0 bad pages, LABEL=swaplab, UUID=027980ea-139a-4af3-9555-2e5b82c43ab5
```

3. Surveiller l'utilisation du swap

```bash
cat /proc/meminfo | grep -i swap
```
```
SwapCached:            0 kB
SwapTotal:       1363956 kB
SwapFree:        1363956 kB
Zswap:                 0 kB
Zswapped:              0 kB
```
```bash
free -m
```
```
               total        used        free      shared  buff/cache   available
Mem:            1705         504         834           4         554        1201
Swap:           1331           0        1331
```

---

## Configuration du swappiness

- Détermine à partir de quel niveau d'utilisation de la RAM le système commence à swapper
```bash
cat /proc/sys/vm/swappiness 
```
`30`

Valeur par défaut du système = 30

- Pour modifier cette valeur temporairement

```bash
sysctl vm.swappiness=60
```
`vm.swappiness = 60`

- Vérification
```bash
cat /proc/sys/vm/swappiness
```
`60`

- Pour modifier cette valeur de manière persistante
```bash
vim /etc/sysctl.conf
```
Ajouter cette ligne dans le fichier (ou modifier la valeur si déjà présente)
```
vm.swappiness=60
```
```bash
sysctl -p
```
`vm.swappiness = 60`

---

_Sources principales:_

- https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/10/html/managing_storage_devices/getting-started-with-swap
- `man swapon`