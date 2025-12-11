# RHEL 10 - Scripting Bash

---

## Les expressions régulières en bash

### Meta-caractères basiques

- `.` &rarr; n'importe quel caractère unique
- `*` &rarr; zéro ou plusieurs répétitions
- `+` &rarr; une ou plusieurs répétitions
- `?` &rarr; zéro ou une répétition
- `^` &rarr; début de ligne
- `$` &rarr; fin de ligne
- `\` &rarr; caractère d'échappement

### Classes de caractères

- `[abc]` &rarr; un caractère parmi la liste
- `[A-Z]` &rarr; un caractère entre A et Z en majuscule
- `[a-Z]` &rarr; un caractère entre a et Z en minuscule ou majuscule
- `[0-9]` &rarr; un chiffre entre 0 et 9
- `[[:space:]]` &rarr; un espace

### Les quantificateurs

- `{n}` &rarr; exactement `n` occurrences
- `{n,}` &rarr; au moins `n` occurrences
- `{n,m}` &rarr; entre `n` et `m` occurrences

---

## Les redirections d'entrées et de sorties

### Les principaux opérateurs

- `>` &rarr; redirige la sortie
- `>>` &rarr; redirige la sortie en ajoutant le contenu à la cible
- `<` &rarr; redirige l'entrée
- `2>` &rarr; redirige l'erreur
- `&>` &rarr; redirige l'erreur et la sortie
- `|` &rarr; Pipe, redirige la sortie vers une autre commande

Exemple:

```bash
ls /var/log/*.log > listlog.txt
cat listlog.txt
```
```
/var/log/boot.log
/var/log/dnf.librepo.log
/var/log/dnf.log
/var/log/dnf.rpm.log
/var/log/hawkey.log
```
```bash
lab 2> erreur.txt
cat erreur.txt
```
```
-bash: lab : commande introuvable
```

---

## Les variables spéciales

- `$0` &rarr; nom du script
- `$1` &rarr; argument selon sa position
- `$#` &rarr; nombre d'arguments
- `$@` &rarr; liste de tous les arguments
- `$*` &rarr; chaîne de tous les arguments
- `$?` &rarr; code retour de la dernière commande

```bash
lab 2> erreur.txt
echo $?
```
`127`

---

### Les opérateurs logiques

- `&&` &rarr; "ET" s'arrête après la première commande qui échoue (code 1-255)
- `||` &rarr; "OU" s'arrête après la première commande qui réussit (code 0)

Exemples:

```bash
mkdir /tmp/labet && touch /tmp/labet/test
ls /tmp/labet/test
```
`/tmp/labet/test`
```bash
mkdir /temp/labou || mkdir /tmp/labou
```
`mkdir: impossible de créer le répertoire « /temp/labou »: Aucun fichier ou dossier de ce nom`
```bash
ls -d /tmp/labou
```
`/tmp/labou`
```bash
touch /tmp/lab/combo/test || mkdir -p /tmp/lab/combo && touch /tmp/lab/combo/test
```
`touch: impossible de faire un touch '/tmp/lab/combo/test': Aucun fichier ou dossier de ce nom`
```bash
ls /tmp/lab/combo/test
```
`/tmp/lab/combo/test`

---

## Les structures conditionnelles

### Syntaxe

- `if` &rarr; exécution si la condition est vraie
- `elif` &rarr; exécution sinon si cette condition est vraie
- `then` &rarr; action à effectuer
- `else` &rarr; exécution sinon
- `fi` &rarr; clôture du bloc conditionnel

### Structure

```bash
if [ condition ]; then <commande>; fi
```
```bash
if [[ condition ]]; then <commande>
elif [[ autre condition ]]; then <commande>
else <commande>
fi
```

---

## Les boucles

### La boucle "for"

```bash
for VARIABLE in <list>; do <commande> $VARIABLE; done
```

- Exemple:

```bash
cd /var/log
for fichier in $(ls *.log); do echo "$fichier"; done
```
```
boot.log
dnf.librepo.log
dnf.log
dnf.rpm.log
hawkey.log
```

### La boucle "while"

```bash
while [ condition ]; do <commande>; done
```

- Exemple:

```bash
ls *.log > list
while read -r ligne; do echo "$ligne"; done < list
```
```
boot.log
dnf.librepo.log
dnf.log
dnf.rpm.log
hawkey.log
```

---

## Création de scripts

**Un fichier de script bash doit toujours commencer par un "shebang" représenté par `#!/bin/bash` et doit être rendu exécutable `chmod +x` pour être utilisé**

### Créer une liste de fichiers de log en excluant les fichiers vides

```bash
vim log_non_vides.sh
```
```bash
#!/bin/bash
# déclaration d'une variable "fichier" contenant les éléments du dossier "/var/log"
for fichier in /var/log/*; do
   # condition pour filtrer uniquement les fichiers (excluant les dossiers)
   if [[ -f "$fichier" ]]; then
      # condition pour filtrer les fichiers de taille non nulle
      if [[ -s "$fichier" ]]; then
         # écriture du résultat dans le fichier "log_non_vides.log"
         echo "$fichier" >> ./log_non_vides.log
      fi
   fi
done
```

### Créer une énumération des fichiers contenant une chaîne de caractères spécifique

```bash
vim ports_ecoute.sh
```
```bash
#!/bin/bash
# Lister tous les fichiers .conf du dossier "/etc" et écrire le résultat dans un fichier "conf_list.txt"
find /etc -name "*.conf" > conf_list.txt
# Rechercher le motif "listen" dans tous ces fichiers et renvoyer les sorties vers "/dev/null"
while read -r conf_file; do
    if grep -i "listen" "$conf_file" &> /dev/null; then
        # Ecrire le résultat dans le fichier "listen.txt"
        echo "$conf_file contient 'listen'" >> listen.txt
    fi
done < conf_list.txt
# Supprimer le fichier "conf_list.txt" devenu inutile
rm -f conf_list.txt
```

#### Variante avec argument

```bash
vim recherche_conf.sh
```
```bash
#!/bin/bash
# Vérification que l'utilisateur a bien passé un argument au script
if [[ -z "$1" ]]; then
        echo "Vous devez entrer un argument"
        exit 1
fi
# Lister tous les fichiers .conf du dossier "/etc" et écrire le résultat dans un fichier "conf_list.txt"
find /etc -name "*.conf" > conf_list.txt
# Rechercher le motif défini par l'utilisateur dans tous ces fichiers et renvoyer les sorties vers "/dev/null"
while read -r conf_file; do
    if grep -i "$1" "$conf_file" &> /dev/null; then
        # Ecrire le résultat dans le fichier "resultats.txt"
        echo "$conf_file contient '$1'" >> resultats.txt
    fi
done < conf_list.txt
# Supprimer le fichier "conf_list.txt" devenu inutile
rm -f conf_list.txt
```

### Créer une fonction et insérer du texte dans un fichier dans un script interactif

```bash
#!/bin/bash
# Création d'une fonction nommée "newvhost"
newvhost ()
{
# Insérer du texte dans un fichier
cat <<EOF > /etc/httpd/conf.d/$VHOST.conf
Listen 80
<VirtualHost *:80>
    DocumentRoot "/www/$VHOST"
    ServerName www.$VHOST.com
</VirtualHost>
EOF
}
# Inviter l'utilisateur à entrer une valeur
read -p "Entrez le nom du nouveau virtual host: " VHOST
# Appeler la fonction "newvhost"
newvhost
```

---

_Sources principales:_
- https://www.oracle.com/ca-fr/technical-resources/articles/linux/saternos-scripting.html
- https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html
