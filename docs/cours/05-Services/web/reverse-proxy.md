# 04 - Reverse Proxy

Un reverse proxy est un serveur qui agit comme un intermédiaire entre les clients (navigateurs) et les serveurs web internes. Il reçoit les requêtes, les redirige vers le bon serveur en fonction du nom de domaine ou du chemin demandé, puis renvoie la réponse au client.

Dans votre infrastructure, vous avez actuellement :

- un serveur web Apache hébergeant un site HTML statique, et vous allez progressivement y ajouter d’autres applications, 
- comme un WordPress (en PHP)
- ou une application développée par les étudiants SLAM.

La bonne pratique consiste à séparer chaque application sur un serveur distinct pour éviter les conflits entre environnements. 
Par exemple, un WordPress peut nécessiter PHP 8.1, alors qu’une autre application peut dépendre encore de PHP 7.4. Si toutes ces versions cohabitent sur le même serveur, les dépendances peuvent entrer en conflit.

Pour exécuter du PHP, deux approches existent :

- Apache avec le module ````libapache2-mod-php````, qui intègre directement PHP dans Apache (simple, mais monolithique),
- ````PHP-FPM (FastCGI Process Manager)````, un service séparé qui exécute le code PHP de manière indépendante. Cette seconde solution est plus performante et plus souple, car elle permet de gérer plusieurs versions de PHP sur le même système, chaque application ayant son propre moteur PHP.

Comme il n’y a qu’une seule adresse publique (et donc un seul port 80/443) disponible, il est impossible d’exposer directement plusieurs serveurs web sur Internet. Le ````reverse proxy```` règle ce problème : il agit comme un point d’entrée unique, recevant toutes les requêtes et les distribuant vers les bons serveurs internes selon le nom de domaine :

- www.monsite.fr → vers le serveur Apache du site HTML,
- blog.monsite.fr → vers le serveur WordPress,
- app.monsite.fr → vers l’application SLAM.

Nous allons utiliser Nginx comme reverse proxy principal, car il est plus performant et plus léger qu’Apache pour ce rôle :
- il gère mieux les connexions simultanées, consomme moins de ressources et sert les fichiers statiques plus rapidement.
- Apache peut également être configuré en reverse proxy, mais cela demande une adaptation différente (modules mod_proxy, mod_proxy_http, etc.) et n’est pas aussi optimisé pour les forts volumes de requêtes.

Plus tard, cette architecture évoluera naturellement vers l’utilisation de container ````Docker````, qui permettra d’isoler chaque application (et sa version de PHP) dans un conteneur dédié, tout en restant accessible via le même reverse proxy Nginx.

### Installation de Nginx (si ce n'est pas déjà fait) :

```bash
sudo apt-get update
sudo apt-get install nginx
```

### Configuration de Nginx :

1. Créer un fichier pour chaque site dans **/etc/nginx/sites-available** :

```bash
sudo nano /etc/nginx/sites-available/site1
```

2. Ajoutez une section pour chaque site, en remplaçant les valeurs par les vôtres :

```nginx

    server {
        listen 80;
        server_name site1.domaine.org;

        location / {
            proxy_pass http://site1.domaine.org;   //avec une resolution donnant l'IP interne par le resolver
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
```
```bash
sudo nano /etc/nginx/sites-available/site2
```
```nginx
    server {
        listen 80;
        server_name site2.domaine.org;

        location / {
            proxy_pass http://192.168.1.20:88; // avec un port d'écoute spécifique à un vhost
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
```
etc...
```nginx
    server {
        listen 80;
        server_name site3.domaine.org;

        location / {
            proxy_pass http://uneip:unport; 
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
```

Créer un lien symbolique pour chaque site référencé dans **sites-available** pour les activer dans **sites-enabled** 

```bash
# Create symbolic links for all sites in sites-available
sudo ln -s /etc/nginx/sites-available/site1 /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/site2 /etc/nginx/sites-enabled/
# Repeat the above command for each site configuration file

# Test Nginx configuration
sudo nginx -t

# If the configuration test is successful, reload Nginx
sudo systemctl reload nginx
```

Les directives `proxy_set_header` dans la configuration Nginx sont utilisées pour définir les en-têtes HTTP qui seront envoyés au serveur backend lorsqu'une requête est transmise par le reverse proxy.

Ces en-têtes sont généralement utilisés pour transférer des informations sur la requête ou le client d'origine au serveur backend. Voici une explication des directives que vous trouverez dans l'exemple fourni :

1. **`proxy_set_header Host $host;`**:
   - Cette directive définit l'en-tête `Host` qui est envoyé au serveur backend. Elle prend la valeur de `$host`, qui est la valeur de l'en-tête `Host` de la requête initiale du client. Cela est important pour que le serveur backend puisse distinguer entre différents domaines virtuels (vhosts) s'il en gère plusieurs.

2. **`proxy_set_header X-Real-IP $remote_addr;`**:
   - Cette directive définit l'en-tête `X-Real-IP`, qui est souvent utilisé pour transmettre l'adresse IP réelle du client au serveur backend. Cela peut être utile lorsque Nginx est placé derrière un autre proxy.

3. **`proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;`**:
   - Cette directive définit l'en-tête `X-Forwarded-For`. Il s'agit d'un en-tête standard utilisé pour transmettre l'adresse IP du client d'origine au serveur backend. La valeur `$proxy_add_x_forwarded_for` ajoute l'adresse IP du client à la liste existante de adresses IP déjà présentes dans l'en-tête.

4. **`proxy_set_header X-Forwarded-Proto $scheme;`**:
   - Cette directive définit l'en-tête `X-Forwarded-Proto`. Cet en-tête indique au serveur backend le protocole utilisé par le client pour accéder à Nginx (HTTP ou HTTPS). La valeur `$scheme` prend la valeur du schéma de la requête (http ou https).

L'utilisation de ces en-têtes est importante lorsqu'un reverse proxy est utilisé, car elle permet de fournir au serveur backend des informations cruciales sur la requête initiale du client. Cela est particulièrement important dans les architectures où plusieurs serveurs interagissent les uns avec les autres, et où certaines informations de la requête d'origine peuvent être perdues lors du passage par le proxy.


3. Sauvegardez et fermez le fichier.

4. Testez la configuration Nginx :

```bash
sudo nginx -t
```

5. Redémarrez Nginx pour appliquer les modifications :

```bash
sudo systemctl restart nginx
```

Maintenant, Nginx agira en tant que reverse proxy et dirigera les requêtes en fonction des noms de domaine vers les serveurs appropriés.

Assurez-vous de remplacer les valeurs comme `192.168.1.20`, `192.168.1.30`, `unport` par les valeurs spécifiques à votre configuration. Vous pouvez également personnaliser les options de proxy en fonction de vos besoins.

## Gestion de TLS

La gestion du HTTPS avec Nginx implique la configuration d'un certificat SSL/TLS pour sécuriser les connexions entre les clients et le serveur Nginx. Voici comment vous pouvez ajouter la prise en charge du HTTPS à votre configuration existante :

2. **Modifiez la configuration Nginx :**
   - Ajoutez les directives SSL dans chaque bloc `server` pour activer le support HTTPS.

     ```nginx
     server {
         listen 80;
         server_name site1.domaine.org;
         return 301 https://$host$request_uri; // forcer la redirection en https
     }

     server {
         listen 443 ssl;
         server_name site1.domaine.org;

         ssl_certificate /etc/ssl/certificat.pem;
         ssl_certificate_key /etc/ssl/privkey.pem;

         # ... autres directives SSL (comme ssl_protocols, ssl_ciphers, etc.)
         
         location / {
             proxy_pass http://192.168.1.20;
             include /etc/nginx/proxy_params;
         }
     }
     ```

     - Répétez le même processus pour les autres serveurs virtuels (`site2.domaine.org` et `site3.domaine.org`).

3. **Redémarrez Nginx :**
   - Une fois que vous avez configuré les certificats et ajusté la configuration Nginx, redémarrez Nginx pour appliquer les changements.

     ```bash
     sudo systemctl restart nginx
     ```

Maintenant, votre serveur Nginx devrait être configuré pour gérer les connexions HTTPS. Assurez-vous de remplacer les chemins des certificats (`ssl_certificate` et `ssl_certificate_key`) avec les chemins réels de vos certificats.

N'oubliez pas de maintenir vos certificats SSL à jour. Vous pouvez configurer un renouvellement automatique avec Certbot ou en ajoutant une tâche planifiée (cron tab) .

Dans le scénario de configuration qci dessus, les certificats SSL/TLS sont généralement nécessaires uniquement sur le serveur Nginx, qui agit en tant que reverse proxy. Les serveurs d'origine (Apache, serveurs Docker, etc.) ne nécessitent pas de certificats SSL/TLS propres, car la communication entre le serveur Nginx et les serveurs d'origine peut souvent se faire en interne via des connexions non chiffrées.

Le trafic chiffré SSL/TLS entre les clients et Nginx s'arrête au niveau du serveur Nginx, et la communication entre Nginx et les serveurs d'origine peut être non chiffrée (HTTP) ou chiffrée séparément (par exemple, si les serveurs d'origine prennent en charge HTTPS et ont leurs propres certificats).

Assurez-vous que le trafic entre Nginx et les serveurs d'origine est sécurisé en fonction de vos exigences de sécurité. Si nécessaire, vous pouvez activer HTTPS entre Nginx et les serveurs d'origine, mais cela impliquera la configuration de certificats sur ces serveurs d'origine également. Dans de nombreux cas, cette couche de chiffrement supplémentaire n'est pas nécessaire si le réseau interne est considéré comme sécurisé.