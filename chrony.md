# RHEL 10 - Chrony et timedatectl

---

## Le service chronyd

**Synchronisation depuis des sources externes avec l'utilisation du protocole NTP et compensation de dérive d'horloge**

- Vérifier que le service est actif
```bash
systemctl status chronyd
```
```
● chronyd.service - NTP client/server
     Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; preset: enabled)
     Active: active (running) since Thu 2025-12-11 10:18:03 CET; 24h ago
 Invocation: c3dc057c5725433db0214b72177d322f
       Docs: man:chronyd(8)
             man:chrony.conf(5)
   Main PID: 914 (chronyd)
      Tasks: 1 (limit: 10680)
     Memory: 3.4M (peak: 3.9M)
        CPU: 183ms
     CGroup: /system.slice/chronyd.service
             └─914 /usr/sbin/chronyd -F 2
```

- Le démarrer et l'activer si nécessaire
```bash
systemctl enable --now chronyd
```

- Configuration du service chronyd
```bash
vim /etc/chrony.conf
```
**Possibilité d'utiliser un ou plusieurs serveur(s) spécifique(s) ou pool(s) de serveurs**

Par défaut:
`pool 2.rhel.pool.ntp.org iburst`

Peut être remplacé par un serveur local:
`server ntpserver.domaine.local iburst` ou `server 192.168.1.123 iburst`

- Consulter les logs du service chronyd
```bash
journalctl -xu chronyd
```

---

## La commande chronyc

**L'utilitaire chronyc est utilisé pour effectuer des modifications de l'instance locale de chronyd soit en mode interactif soit avec des options dans la cli**

- Vérifier la synchronisation
```bash
chronyc tracking && chronyc sources -v
```
```
Reference ID    : 268F1310 (38.143.19.16)
Stratum         : 3
Ref time (UTC)  : Fri Dec 12 10:03:55 2025
System time     : 0.000092552 seconds slow of NTP time
Last offset     : -0.000022552 seconds
RMS offset      : 0.000203115 seconds
Frequency       : 14.800 ppm fast
Residual freq   : -0.003 ppm
Skew            : 0.110 ppm
Root delay      : 0.007643121 seconds
Root dispersion : 0.001115534 seconds
Update interval : 1035.2 seconds
Leap status     : Normal

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current best, '+' = combined, '-' = not combined,
| /             'x' = may be in error, '~' = too variable, '?' = unusable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^- vps-mrs1.orleans.ddnss.de     2  10   377   661   -481us[ -501us] +/-   16ms
^+ dns.freewebworld.fr           1  10   377   650   -195us[ -216us] +/- 8173us
^- carbon.home-dn.net            2  10   377   349  +1042us[+1042us] +/-   54ms
^* 38.143.19.16                  2  10   377   523  +5839ns[  -17us] +/- 4203us
```

- Forcer la correction de l'horloge système immédiatement
```bash
chronyc makestep
```
`200 OK` &rarr; commande executée avec succès (les codes 500 à 504 signifient des erreurs)

- Vérifier la connectivité aux serveurs NTP
```bash
chronyc sourcestats
```

---

## La commande timedatectl

**Cette commande gère la configuration utilisateur, le fuseau horaire et le format d'affichage dans un contexte systemd**

- Vérification de l'état actuel
```bash
timedatectl
```
```
               Local time: ven. 2025-12-12 11:30:34 CET
           Universal time: ven. 2025-12-12 10:30:34 UTC
                 RTC time: ven. 2025-12-12 10:30:34
                Time zone: Europe/Paris (CET, +0100)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```
`NTP service: active` &rarr; chronyd est actif

- Pour désactiver chronyd
```bash
timedatectl set-ntp no
timedatectl | grep NTP
```
`NTP service: inactive` &rarr; chronyd est inactif

Vérification:
```bash
systemctl status chronyd | grep Active
```
`Active: inactive (dead)`

- Pour le reactiver
```bash
timedatectl set-ntp yes
```

- Modification du fuseau horaire

Lister les fuseaux horaires:
```bash
timedatectl list-timezones
```
_Renvoi une liste exhaustive des fuseaux horaires_

Définir un fuseau horaire
```bash
timedatectl set-timezone America/New_York
```
Vérification
```bash
timedatectl | grep "Time zone"
```
`Time zone: America/New_York (EST, -0500)`

- Définir l'heure du système manuellement

Désactiver le service chronyd
```bash
timedatectl set-ntp no
```
Définir l'heure souhaitée
```bash
timedatectl set-time "12:00:00"
```
Vérification
```bash
timedatectl | grep "Local time"
```
`Local time: ven. 2025-12-12 12:01:09 CET`

- Revenir à une configuration de temps correcte
```bash
timedatectl set-ntp yes
chronyc makestep
```
`200 OK`

---

Sources principales

- https://docs.redhat.com/fr/documentation/red_hat_enterprise_linux/10/html/configuring_time_synchronization/using-chrony
- `man timedatectl`
- `man chronyc`