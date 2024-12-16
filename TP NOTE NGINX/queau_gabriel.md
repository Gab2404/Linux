# TP NOTE NGINX

### Set up du serveur

> sudo apt-get update

### Création d'un user

Pour sécuriser dans un premier temps un maximum le serveur, mieux vaut il ne pas laisser l'accès root à tous le monde :

> adduser "gabriel"

> usermod -aG sudo gabriel

### Installation NGINX

Ici on va choisir d'installer le "package prebuilt" pour ce faire on va utiliser les commandes suivantes :

> sudo apt install curl gnupg2 ca-certificates lsb-release debian-archive-keyring

> curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null

> gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg

*( Avec la commande au dessus on verifie que tous est en régles et bien installé)* 

## Setup de NGINX

### Backup


Dans un premier temps on va configurer une backup au cas où que l'on fasse une mauvaise manipulation :

> cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup-original

> nginx -s reload

### Multiple Worker Processes

    Dans un premier temps on va optimiser l'utilisation des cœurs CPU disponibles. Cela améliorera la performance, la scalabilité, et la fiabilité du serveur :

On va mettre le ``worker_processes`` en ``auto``

> worker_processes auto;

Ensuite je vais limiter les connexion a __10__ :

 ```
 events {
        worker_connections 10;
        # multi_accept on;
}
```

### Desactivation du token

On va désactivé le token pour qu'il soit plus dur de déterminé la version de Nginx :

```
 http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        server_tokens off;
```

### Configuration du site

Dans cette étapes on va configurer le répertoire racine de notre site :

```
server {
    listen 80;
    server_name gab.com www.gab.com;

    root /var/www/gab.com;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

On crée ici le répertoir avec la commande suivante :

> sudo mkdir -p /var/www/gab.com

Ensuite on s'assure qu'on est donnée les bonnes permissions : 

> sudo chown -R www-data:www-data /var/www/gab.com && sudo chmod -R 755 /var/www/gab.com

#### Setup des IPv4 et IPv6 seulemnt

Ici on va faire en sorte qu'il soit capable d'écouter les Ipv6 et pas eulement les IPv4 et le port 80 :

```
listen [::]:80;
listen [::]:443 ssl;
```

#### Compression du contenut du site

___**Résumé :**___

**Pourquoi activer la compression ? :** Pour améliorer les performances en réduisant la taille des fichiers envoyés.

**Pourquoi ne pas compresser tout le contenu ? :** Cela expose ton site à des attaques comme CRIME et BREACH, particulièrement avec HTTPS.

**Solution :** Compresser uniquement les fichiers statiques (HTML, CSS, images), et configurer la compression de manière spécifique pour chaque site dans le fichier NGINX.

On modifie/rajoute donc les lignes suivantes : 

```
gzip          on;
gzip_types    text/html text/plain text/css image/*;
```

### Configuration du pare-feu 

> sudo apt update

> sudo apt install ufw

> sudo ufw default deny incoming

Ici on autorise tout les trafics sortant à allée vers internet :

> sudo ufw default allow outgoing

Ici on autorise le ssh à se connecter :

> sudo ufw allow "OpenSSH"

Enfin ici on met en place toutes les rélges précédentes :

> sudo ufw enable

On va autorisé aussi, seulement mon ip pour rentrer sur ma vm :

>  sudo ufw allow from 10.33.68.134 to any port 22

### Ajout de Fail2ban

Afin d'éviter le force brute on va faire en sorte que les ip qui emmettent trop de tentative soit ban temporairement grace à Fail2ban :

> sudo apt-get install fail2ban

> sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

```
#banaction = iptables-multiport
#banaction_allports = iptables-allports
banaction = ufw
```
```
[sshd]
enabled = true
port = ssh
logpath = %(sshd_log)s
backend = %(ssh_backend)s

```


```
[nginx-http-auth]
enabled = true
port = http, https
logpath = /var/log/nginx/error.log
```

### Setup SFTP

Cela va nous permettre de sécurisé le transfert des fichiers :

> sudo apt-get install mysecureshell

> sudo nano /etc/ssh/sftp_config

### Système pas clès

Pour finir on va cofigurer une connexion par clès que seulement moi possédera :

    On commence par générer la clès

> ssh-keygen -t rsa -b 4096

ensuite on recherche la clé :

> cat ~/.ssh/id_rsa.pub

On rajoute le contenu sur notre user ssh :

> mkdir -p ~/.ssh
> nano ~/.ssh/authorized_keys

Enfin on s'assure que les permission sont correcte

> chmod 600 ~/.ssh/authorized_keys

#### Configuration du ssh par clés

On va modifier le fichier avec les perms :

> sudo nano /etc/ssh/sshd_config

On rajoute les lignes suivantes :

> PasswordAuthentication no
> PubkeyAuthentication yes

Et il nous reste plus qu'à redemarrer le ssh :

> sudo systemctl restart ssh
