# DLC : "L'ultime défi du SysAdmin"

### Étape 1 : Analyse avancée et suppression des traces suspectes

1. **Rechercher des utilisateurs récemment ajoutés :**
     ```bash
     sudo grep "new user" /var/log/secure
     ```
```zsh
[root@vbox ~]# grep -a "new user" /var/log/secure
Nov 24 18:11:10 vbox useradd[1450]: new user: name=attacker, UID=1000, GID=1000, home=/home/attacker, shell=/bin/bash, from=/dev/pts/0
Nov 25 19:19:01 vbox useradd[5121]: new user: name=attacker, UID=1000, GID=1000, home=/home/attacker, shell=/bin/bash, from=/dev/pts/0
```
2. **Trouver les fichiers récemment modifiés dans des répertoires critiques :**
 
     ```bash
     sudo find /etc /usr/local/bin /var -type f -mtime -7
     ```
```zsh
[root@vbox ~]# sudo find /etc /usr/local/bin /var -type f -mtime -7
```
3. **Lister les services suspects activés :**
  
     ```bash
     sudo systemctl list-unit-files --state=enabled
     ```
```zsh
[root@vbox ~]# sudo systemctl list-unit-files --state=enabled
```
4. **Supprimer une tâche cron suspecte :**
  
     ```bash
     sudo crontab -u attacker -r
     ```
```zsh
[root@vbox ~]# sudo crontab -u attacker -r
```
---

## Étape 2 : Configuration avancée de LVM

1. **Créer un snapshot du volume logique** :
   
```zsh
[root@vbox ~]# lvcreate -L 200MB -s -n sec_data_snap /dev/vg_secure/secure_data 
  Logical volume "sec_data_snap" created.
```
2. **Tester le snapshot** :

```zsh
[root@vbox ~]# lvconvert --mergesnapshot /dev/vg_secure/sec_data_snap 
  Merging of volume vg_secure/sec_data_snap started.
  vg_secure/secure_data: Merged: 100.00%
[root@vbox ~]# mount /mnt/secure_data/
```
3. **Simuler une restauration** :
  
```zsh
[root@vbox ~]# lvconvert --mergesnapshot /dev/vg_secure/sec_data_snap 
  Merging of volume vg_secure/sec_data_snap started.
  vg_secure/secure_data: Merged: 100.00%
[root@vbox ~]# mount /mnt/secure_data/
```
---

## Étape 3 : Renforcement du pare-feu avec des règles dynamiques

1. **Bloquer les attaques par force brute** :
   
[root@vbox ~]# yum install epel-release -y
[root@vbox ~]# yum install fail2ban -y

[root@vbox ~]# systemctl start fail2ban
systemctl enable fail2ban

[root@vbox ~]# sudo cp /etc/fail2ban/fail2ban.conf /etc/fail2ban/fail2ban.local
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
