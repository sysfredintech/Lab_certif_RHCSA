# RHEL 10 - La gestion des processus et le monitoring

---

## La commande `ps`

> La commande ps permet d'afficher des informations sur l'exécution de processus. Elle produit une liste statique, ou instantanée, de ce qui est cours d'exécution au moment où la commande est exécutée. 

- Lister tous les processus en cours d'exécution avec le propriétaire associé et les consommation CPU et RAM
```bash
ps aux
```

Pour trier la sortie par consommation de ressources

- `--sort=%cpu` &rarr; par utilisation CPU décroissante
- `--sort=%mem` &rarr; par utilisation RAM décroissante

```bash
ps aux --sort=%cpu | head -5
```
_Affiche les 5 processus les plus consommateurs de CPU_

- Personnaliser la sortie de la commande

Pour afficher les colonnes concernant la priorité, l'id du processus, l'utilisateur et la commande associée en classant par priorité décroissante
```bash
ps -eo ni,pid,user,cmd --sort=ni
```
_La liste des code est disponible dans la page de manuel `man ps`_

Pour afficher tous les processus d'un utilisateur en particulier
```bash
ps -u apache
```
_Affiche la liste de tous les processus de l'utilisateur `apache`_

---

## La commande `pstree`

> `pstree` affiche les processus en cours d'exécution sous forme d'arborescence. La racine de l'arborescence se trouve sur: soit le pid, soit le processus init si le pid est omis. Si un nom d'utilisateur est spécifié, tous les arbres de processus dont la racine se trouve sur les processus appartenant à cet utilisateur sont affichés.

- Afficher une arborescence compacte des processus en cours d'exécution
```bash
pstree
```

- Afficher une arborescence des processus en cours d'exécution avec les `pid` et les utilisateurs associés
```bash
pstree -up
```

- Afficher une arborescence des processus appartenant un utilisateur spécifique
```bash
pstree apache
```
_Affiche l'arborescence de tous les processus de l'utilisateur `apache`_

---

## La commande `top`

> La commande top affiche une liste en temps réel des processus exécutés sur le système. Elle affiche également des informations supplémentaires sur le temps d'activité du système, l'utilisation actuelle du CPU et de la mémoire, ou le nombre total de processus en cours d'exécution, et vous permet d'effectuer des actions comme le tri de la liste ou l'arrêt d'un processus.

### normalements interactives

- `P` &rarr; Trie par utilisation CPU (défaut)
- `M` &rarr; Trie par utilisation RAM
- `u` &rarr; Affiche les processus d'un utilisateur
- `n` &rarr; Limite la sortie à _n_ lignes
- `k` &rarr; Stoppe un processus par son `pid`
- `h` &rarr; Affiche une aide
- `r` &rarr; Change la priorité d'un processus
- `q` &rarr; Quitte le programme

---

## La commande `free`

> La commande free fournit à la fois des informations sur la mémoire physique (Mem) et sur l'espace swap (Swap). Elle affiche le montant de mémoire total (total), ainsi que le montant de mémoire utilisée (used), la mémoire libre (free), la mémoire partagée (shared), les mémoires tampon et cache ajoutées ensemble (buff/cache), et ce qui reste de disponible (available).

- Afficher l'état de la mémoire du système en Mo
```bash
free -m
```

- Afficher l'état de la mémoire du système de façon facilement lisible
```bash
free -h
```

- Afficher l'état de la mémoire du système avec une ligne _total (mem + swap)_ supplémentaire de façon facilement lisible
```bash
free -th
```

---

## normalements `nice` et `renice`

Ces deux commandes permettent d'exécuter un programme avec une priorité d'ordonnancement modifiée ou de redéfinir la priorité d'un processus en cours d'exécution.

Plage de valeurs nice: de `-20` &rarr; priorité haute à `19` &rarr; priorité basse

- Lancer une commande avec une priorité haute
```bash
nice -n -10 ./script_important.sh
```

- Lancer une commande avec une priorité basse
```bash
nice -n 15 ./script_secondaire.sh
```

- Donner une priorité haute à un processus en cours d'exécution
```bash
renice -n -10 -p 1151
```
`1151 (process ID) old priority 0, new priority -10`

- Donner la priorité la plus basse à un processus en cours d'exécution
```bash
renice -n 19 -p 1151
```
`1151 (process ID) old priority -10, new priority 19`

- Donner une priorité basse à tous les processus de l'utilisateur _labuser1_
```bash
renice -n 15 -u labuser1
```
`1000 (user ID) old priority 0, new priority 15`

---

## Gestion des tâches avec `jobs`, `bg` et `fg`

- Lancer une tâche en arrière-plan
```bash
sleep 180 &
```

- Lister les tâches du shell
```bash
jobs -l
```
`[1]+  2531 En cours d'exécution   sleep 180 &`

- Ramener une tâche au premier-plan
```bash
fg %1
```
` sleep 180`

- Suspendre la tâche
`Ctrl` + `z`
`[1]+  Stoppé                 sleep 180`

- Reprendre la tâche en arrière-plan

```bash
bg %1
```
`[1]+ sleep 180 &`

---

## Les commandes `kill` et `killall`

> La commande `kill` envoie le signal spécifié aux processus ou groupes de processus spécifiés. Si aucun signal n'est spécifié, le signal TERM est envoyé. L'action par défaut pour ce signal est de mettre fin au processus.

- Lister les signaux disponibles
```bash
kill -l
```
Les plus utilisés:
- `SIGTERM (15)` &rarr; arrêter normalement
- `SIGKILL (9)` &rarr; forcer l'arrêt
- `SIGSTOP (19)` &rarr; suspendre
- `SIGCONT (18)` &rarr; reprendre après suspension

- Arrêter un processus en cours d'exécution
```bash
kill # PID
```

- Forcer l'arrêt d'un processus en cours d'exécution
```bash
kill -9 # PID
```
Ou
```bash
kill -SIGKILL # PID
```

> `killall` envoie un signal à tous les processus exécutant l'une des commandes spécifiées. Si aucun nom de signal n'est spécifié, SIGTERM est envoyé.

- Terminer tous les processus d'un utilisateur
```bash
killall -u # utilisateur
```
Pour forcer l'arrêt
```bash
killall -9 -u # utilisateur
```

---

_Sources principales:_

- https://docs.redhat.com/fr/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-system_monitoring_tools
- https://www.redhat.com/en/blog/linux-command-basics-7-commands-process-management
- https://www.redhat.com/fr/blog/jobs-bg-fg