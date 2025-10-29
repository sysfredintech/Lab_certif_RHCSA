# RHEL 10 - GRUB2 - Le chargeur de démarrage

---

**Manipulations basiques pour le paramétrage du chargeur de démarrage via la cli**

---

## Architecture du processus de démarrage

1. UEFI/BIOS de la machine = hardware
2. GRUB2 = chargeur d'amorçage
3. Kernel = le noyau Linux
4. initramfs = montage du système de fichier racine dans la ram
5. systemd = gestionnaire de processus
6. target = runlevel ou niveau de démarrage

---

## Les fichiers et dossiers clés

- `/boot/grub2/grub.cfg` &rarr; généré automatiquement, il n'est pas prévu de l'éditer manuellement - BIOS
- `/boot/efi/EFI/redhat/grub.cfg` &rarr; généré automatiquement, il n'est pas prévu de l'éditer manuellement - UEFI
- `/boot/grub2/grubenv` &rarr; fichier d'environnement de GRUB
- `/etc/default/grub` &rarr; défini les paramètres du menu GRUB
- `/etc/grub.d/` &rarr; contient les scripts de génération de GRUB
- `/boot/loader/entries/` &rarr; contient les entrées de démarrage

---

## La commande Grubby

**Grubby est un utilitaire de manipulation des fichiers du chargeur d'amorçage**

### Récupération des informations

- Afficher les informations concernant l'entrée par défaut
```bash
grubby --info=DEFAULT
```
```
index=0
kernel="/boot/vmlinuz-6.12.0-55.40.1.el10_0.x86_64"
args="ro crashkernel=2G-64G:256M,64G-:512M resume=UUID=512ad036-a4f3-40cb-ba3e-2133be9b2589 rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap $tuned_params rhgb quiet"
root="/dev/mapper/rhel-root"
initrd="/boot/initramfs-6.12.0-55.40.1.el10_0.x86_64.img $tuned_initrd"
title="Red Hat Enterprise Linux (6.12.0-55.40.1.el10_0.x86_64) 10.0 (Coughlan)"
id="650f1822bcd14a1e8a46f4f6b4186d71-6.12.0-55.40.1.el10_0.x86_64"
```
- Afficher les informations sur toutes les entrées
```bash
grubby --info=ALL
```
```
index=0
kernel="/boot/vmlinuz-6.12.0-55.40.1.el10_0.x86_64"
args="ro crashkernel=2G-64G:256M,64G-:512M resume=UUID=512ad036-a4f3-40cb-ba3e-2133be9b2589 rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap $tuned_params rhgb quiet"
root="/dev/mapper/rhel-root"
initrd="/boot/initramfs-6.12.0-55.40.1.el10_0.x86_64.img $tuned_initrd"
title="Red Hat Enterprise Linux (6.12.0-55.40.1.el10_0.x86_64) 10.0 (Coughlan)"
id="650f1822bcd14a1e8a46f4f6b4186d71-6.12.0-55.40.1.el10_0.x86_64"
index=1
kernel="/boot/vmlinuz-6.12.0-55.38.1.el10_0.x86_64"
args="ro crashkernel=2G-64G:256M,64G-:512M resume=UUID=512ad036-a4f3-40cb-ba3e-2133be9b2589 rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap $tuned_params rhgb quiet"
root="/dev/mapper/rhel-root"
initrd="/boot/initramfs-6.12.0-55.38.1.el10_0.x86_64.img $tuned_initrd"
title="Red Hat Enterprise Linux (6.12.0-55.38.1.el10_0.x86_64) 10.0 (Coughlan)"
id="650f1822bcd14a1e8a46f4f6b4186d71-6.12.0-55.38.1.el10_0.x86_64"
index=2
kernel="/boot/vmlinuz-0-rescue-650f1822bcd14a1e8a46f4f6b4186d71"
args="ro crashkernel=2G-64G:256M,64G-:512M resume=UUID=512ad036-a4f3-40cb-ba3e-2133be9b2589 rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet"
root="/dev/mapper/rhel-root"
initrd="/boot/initramfs-0-rescue-650f1822bcd14a1e8a46f4f6b4186d71.img"
title="Red Hat Enterprise Linux (0-rescue-650f1822bcd14a1e8a46f4f6b4186d71) 10.0 (Coughlan)"
id="650f1822bcd14a1e8a46f4f6b4186d71-0-rescue"
```

- Contenu d'une entrée

Les fichiers de configuration des entrées sont placés dans: `/boot/loader/entries/`
```bash
ls /boot/loader/entries/
```
```
650f1822bcd14a1e8a46f4f6b4186d71-0-rescue.conf
650f1822bcd14a1e8a46f4f6b4186d71-6.12.0-55.38.1.el10_0.x86_64.conf
650f1822bcd14a1e8a46f4f6b4186d71-6.12.0-55.40.1.el10_0.x86_64.conf
```

```bash
cat /boot/loader/entries/650f1822bcd14a1e8a46f4f6b4186d71-6.12.0-55.40.1.el10_0.x86_64.conf
```
```
title Red Hat Enterprise Linux (6.12.0-55.40.1.el10_0.x86_64) 10.0 (Coughlan)
version 6.12.0-55.40.1.el10_0.x86_64
linux /vmlinuz-6.12.0-55.40.1.el10_0.x86_64
initrd /initramfs-6.12.0-55.40.1.el10_0.x86_64.img $tuned_initrd
options root=/dev/mapper/rhel-root ro crashkernel=2G-64G:256M,64G-:512M resume=UUID=512ad036-a4f3-40cb-ba3e-2133be9b2589 rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap $tuned_params rhgb quiet
grub_users $grub_users
grub_arg --unrestricted
grub_class rhel
```
`version 6.12.0-55.40.1.el10_0.x86_64` &rarr; la version du noyau
`initrd /initramfs-6.12.0-55.40.1.el10_0.x86_64.img $tuned_initrd` &rarr; image du ramdisk initiale
`options root=/dev/mapper/rhel-root ro crashkernel=2G-64G:256M,64G-:512M` &rarr; paramètres passés en ligne de commande pour le noyau

- Afficher le noyau chargé par défaut
```bash
grubby --default-kernel
```
`/boot/vmlinuz-6.12.0-55.40.1.el10_0.x86_64`

- Afficher l'index du noyau par défaut
```bash
grubby --default-index
```
`0`

### Effectuer des modifications

- Ajouter un argument pour tous les noyaux

**La liste des arguments est consultable sur le site kernel.org https://www.kernel.org/doc/html/v4.14/admin-guide/kernel-parameters.html**
```bash
grubby --update-kernel=ALL --args="ignore_loglevel"
```
_On a ajouté un argument pour activer un haut niveau de déboggage du kernel_

- Supprimer un ou plusieurs arguments pour le noyau par défaut
```bash
grubby --update-kernel=DEFAULT --remove-args="rhgb quiet"
```
_On a supprimé les arguments limitant l'affichage des informations au démarrage_

**Cette modification ne changera pas le contenu de `/etc/default/grub`, il est possible d'harmoniser la configuration du système en applicant les arguments définis ci-dessus selon la méthode détaillé plus bas avec le champs `GRUB_CMDLINE_LINUX`**

- Vérification
```bash
grubby --info=ALL
```
```
index=0
kernel="/boot/vmlinuz-6.12.0-55.40.1.el10_0.x86_64"
args="ro crashkernel=2G-64G:256M,64G-:512M resume=UUID=512ad036-a4f3-40cb-ba3e-2133be9b2589 rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap $tuned_params ignore_loglevel"
root="/dev/mapper/rhel-root"
initrd="/boot/initramfs-6.12.0-55.40.1.el10_0.x86_64.img $tuned_initrd"
title="Red Hat Enterprise Linux (6.12.0-55.40.1.el10_0.x86_64) 10.0 (Coughlan)"
id="650f1822bcd14a1e8a46f4f6b4186d71-6.12.0-55.40.1.el10_0.x86_64"
index=1
kernel="/boot/vmlinuz-6.12.0-55.38.1.el10_0.x86_64"
args="ro crashkernel=2G-64G:256M,64G-:512M resume=UUID=512ad036-a4f3-40cb-ba3e-2133be9b2589 rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap $tuned_params rhgb quiet ignore_loglevel"
root="/dev/mapper/rhel-root"
initrd="/boot/initramfs-6.12.0-55.38.1.el10_0.x86_64.img $tuned_initrd"
title="Red Hat Enterprise Linux (6.12.0-55.38.1.el10_0.x86_64) 10.0 (Coughlan)"
id="650f1822bcd14a1e8a46f4f6b4186d71-6.12.0-55.38.1.el10_0.x86_64"
index=2
kernel="/boot/vmlinuz-0-rescue-650f1822bcd14a1e8a46f4f6b4186d71"
args="ro crashkernel=2G-64G:256M,64G-:512M resume=UUID=512ad036-a4f3-40cb-ba3e-2133be9b2589 rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap rhgb quiet ignore_loglevel"
root="/dev/mapper/rhel-root"
initrd="/boot/initramfs-0-rescue-650f1822bcd14a1e8a46f4f6b4186d71.img"
title="Red Hat Enterprise Linux (0-rescue-650f1822bcd14a1e8a46f4f6b4186d71) 10.0 (Coughlan)"
id="650f1822bcd14a1e8a46f4f6b4186d71-0-rescue"
```

- Changer le noyau par défaut

```bash
grubby --set-default=/boot/vmlinuz-6.12.0-55.38.1.el10_0.x86_64
```
```
The default is /boot/loader/entries/650f1822bcd14a1e8a46f4f6b4186d71-6.12.0-55.38.1.el10_0.x86_64.conf with index 1 and kernel /boot/vmlinuz-6.12.0-55.38.1.el10_0.x86_64
```
_Le kernel par défaut sera `6.12.0-55.38.1.el10_0.x86_64`_

- Vérification
```bash
grubby --default-index
```
`1`

---

## La configuration du menu GRUB

- La configuration se fait par l'édition du fichier /etc/default/grub

```bash
vim /etc/default/grub
```
```
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=2G-64G:256M,64G-:512M resume=UUID=512ad036-a4f3-40cb-ba3e-2133be9b2589 rd.lvm.lv=rhel/root rd.lvm.lv=rhel/swap ignore_loglevel"
GRUB_DISABLE_RECOVERY="true"
GRUB_ENABLE_BLSCFG=true
```

- Pour modifier le timer du GRUB
```
GRUB_TIMEOUT=10
```
_Le menu GRUB restera afficher 10 secondes sans intervention de l'utilisateur_

- Appliquer les changements

Identifier le type de système d'amorçage:
```bash
lsblk | grep boot
```
`├─sda2              8:2    0     1G  0 part /boot` &rarr; pas d'uefi

Si cette commande renvoi
`├─nvme0n1p1     259:1    0   600M  0 part /boot/efi` &rarr; uefi

```bash
grub2-mkconfig -o /boot/grub2/grub.cfg
```
```
Generating grub configuration file ...
File descriptor 3 (pipe:[15933]) leaked on vgs invocation. Parent PID 3260: grub2-probe
File descriptor 9 (pipe:[15955]) leaked on vgs invocation. Parent PID 3260: grub2-probe
File descriptor 3 (pipe:[15933]) leaked on vgs invocation. Parent PID 3260: grub2-probe
File descriptor 9 (pipe:[15955]) leaked on vgs invocation. Parent PID 3260: grub2-probe
File descriptor 3 (pipe:[15933]) leaked on vgs invocation. Parent PID 3296: grub2-probe
File descriptor 9 (pipe:[15955]) leaked on vgs invocation. Parent PID 3296: grub2-probe
File descriptor 3 (pipe:[15933]) leaked on vgs invocation. Parent PID 3296: grub2-probe
File descriptor 9 (pipe:[15955]) leaked on vgs invocation. Parent PID 3296: grub2-probe
Adding boot menu entry for UEFI Firmware Settings ...
```

**Dans le cas d'une installation sur UEFI, les changements doivent être appliqués dans `/boot/efi/EFI/redhat/grub.cfg`**

---

## Passer en mode de récupération d'urgence

- Redémarrer la machine
```bash
systemctl reboot
```

- Editer l'entrée grub temporairement

A l'affichage du menu GRUB, sélectionner l'entrée voulue et appuyer sur la touche `e` pour éditer les options et ajouter `single` ou `rd.break` à la fin de la ligne `linux` puis `ctrl`+`x` pour démarrer la machine

- Le système démarre sur le runlevel rescue.target avec un shell de secours

**Pour un démarrage qui contourne systemd, utiliser `init=/bin/bash` à la place de `single`**

### Cas pratique

- Redéfinir le mot de passe root

1. Redémarrer la machine puis éditer  l'entrée choisie depuis le menu GRUB avec la touche `e`
2. Ajouter `init=/bin/bash` à la fin de la ligne `linux` et démarrer le système avec `ctrl` + `x`
3. A l'invite de commande, monter la racine en _rw_ afin de pouvoir écrire sur le disque
```bash
mount -o remount,rw /
```
4. Changer le mot de passe du compte root avec la commande `passwd`
5. Créer un fichier vide pour que SELinux procède à un nouvel étiquetage
```bash
touch /.autorelabel
```
6. Redémarrer la machine avec `/sbin/reboot -f`
7. Dans le cas où l'authentification ne fonctionne pas

Reprendre les étapes de 1 à 3 puis
```bash
vim /etc/shadow
```
Supprimer le hash du mot de passe root situé entre les 2 premiers `:`, exemple: `root::20389:0:99999:7:::` puis sauvegarder et quitter avec `:wq!` pour forcer l'écriture

- Refaire les étape 5 et 6

La connexion avec le compte root se fasse sans mot de passe, il sera alors possible de le redéfinir avec `passwd` à la prochaine connexion

---

## Les commandes GRUB2

- Pour installer ou réinstaller GRUB2 sur une machine bios

Vérifier le périphérique avec la commande `lsblk`, ici c'est `/dev/sda`
```bash
grub2-install /dev/sda
```
```
Installation pour la plate-forme i386-pc.
Installation terminée, sans erreur.
```

- Pour installer ou réinstaller GRUB2 sur une machine UEFI

La procédure se fera de préférence avec le gestionnaire de paquets

```bash
dnf reinstall grub2-efi shim
```
Ou en mode de récupération d'urgence avec
```bash
grub2-install --target=x86_64-efi --efi-directory=/boot/efi
grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
```

- Pour lister les variables d'environnement
```bash
grub2-editenv list
```
```
saved_entry=650f1822bcd14a1e8a46f4f6b4186d71-6.12.0-55.38.1.el10_0.x86_64
menu_auto_hide=1
boot_success=0
boot_indeterminate=0
next_entry=
```

- Pour définir ou supprimer une variable d'environnement
```bash
grub2-editenv - unset menu_auto_hide
```
_On a supprimé la variable qui indique de masquer le menu au démarrage_

```bash
grub2-editenv - set menu_auto_hide=1
```
_On a redéfini cette variable_

- Pour définir l'entrée par défaut
```bash
grub2-set-default 0
```
_On à défini l'entrée par défaut à la valeur d'index `0`_
**L'effet de cette commande est identique à l'effet de `grubby --set-default=`**

- Pour déterminer l'entrée du prochain reboot (temporaire)
```bash
grub2-reboot 1
```
_On à défini l'entrée du prochain reboot à la valeur d'index `1`_

- Protéger GRUB2 avec un mot de passe pour l'utilisateur _root_

Pour sécuriser les modifications des entrées uniquement:
```bash
grub2-setpassword
```
```
Enter password: 
Confirm password:
```
_Un fichier `/boot/grub2/user.cfg` est créé_
>  Avec ce changement, pour modifier une entrée boot au moment de l'amorçage nécessite que vous spécifiez le nom d'utilisateur root et votre mot de passe.

Pour sécuriser les modifications d'entrées **et le démarrage**:
_Avoir défini un mot de passe avec `grub2-setpassword` au-préalable_
```bash
grub2-editenv - set grub_users="root"
```
_La protéction par mot de passe sera effective pour toutes les entrées_

> Warning
If you forget the GRUB password, you will not be able to boot the entries you have reconfigured.

- Pour supprimer la protection par mot de passe

```bash
grub2-editenv - unset grub_users
```
```bash
rm -r /boot/grub2/user.cfg
```
Ou sur un système UEFI
```bash
rm -r /boot/efi/EFI/redhat/user.cfg
```
**Cette procédure peut s'effectuer en mode rescue avec un live cd après avoir chrooté avec `chroot /mnt/sysroot`**

---

## Résumé des commandes essentielles et des bonnes pratiques

- Vérifier le firmware de la machine avant d'appliquer des changements
```bash
ls /sys/firmware/efi && echo "UEFI" || echo "BIOS"
```
ou
```bash
lsblk | grep boot
```

- Privilégier la commande `grubby` pour les changements permanents sur les entrées du chargeur de démarrage
- Ne pas éditer le fichier `/boot/grub2/grub.cfg` directement
- Toujours appliquer les changements avec la commande `grub2-mkconfig -o /boot/grub2/grub.cfg` ou `grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg` après modification de `/etc/default/grub` ou installation/réparation avec `grub2-install`
- Editer le menu GRUB2 au démarrage avec la touche `e` puis `ctrl`+`x` ou démarrer la machine avec le live cd pour utiliser le mode rescue
- Si mise en place d'un mot de passe pour le démarrage, il sera impossible de démarrer le système en cas de perte de ce dernier

---

_Sources:_
- https://docs.redhat.com/fr/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-working_with_the_grub_2_boot_loader
- https://www.gnu.org/software/grub/manual/grub/grub.html
- `man grubby`
- `man grub2-install`
- `man grub2-mkconfig`
- `man grub2-editenv`