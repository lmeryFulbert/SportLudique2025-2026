# 03 - Serveur DNS D'autorité

## Exemple de configuration d'un Serveur (BIND 9)

Supposons que vous utilisez BIND comme serveur DNS principal pour le domaine `exemple.com` et que vous souhaitez déléguer la gestion du sous-domaine `sousdomaine.exemple.com` à un autre serveur DNS dont l'adresse IP est `203.0.113.100`.

### Configuration BIND (zone principale pour exemple.com) :

1. Dans le fichier de zone pour `exemple.com`, ajoutez les enregistrements NS pour le sous-domaine délégué comme ceci :

```plaintext
; Fichier de zone pour exemple.com (zone publique)
$TTL 3600

@      IN      SOA    ns1.exemple.com. admin.exemple.com. (
                  2023091301 ; Numéro de série
                  3600       ; Intervalle de rafraîchissement
                  1800       ; Intervalle de réessai
                  604800     ; Intervalle d'expiration
                  86400 )    ; Durée minimale de cache

;choisir une des 2 syntaxes avec le @ ou en recopiant le domaine (avec le point à la fin)
;@      IN      NS     ns1.exemple.com.
exemple.com.        IN      NS      ns1.exemple.com.

; Serveur DNS primaire
ns1     IN      A       203.0.113.40

; Enregistrements IPv4 des serveurs web
@      IN      A      203.0.113.10
; le @ remplace la zone DNS, tres utile pour acceder au site web sans le www, on en reparlera avec le Subject Alternative Name des certificats pour TLS
www    IN      A      203.0.113.10

; Enregistrements IPv4 du serveur de messagerie
mail   IN      A      203.0.113.30

; Délégation du sous-domaine à un autre serveur DNS
sousdomaine.exemple.com.  IN  NS  ns.sousdomaine.exemple.com.
```

!!! info "Information"
    Notez que dans cet exemple, nous utilisons l'enregistrement NS `ns.serveurdns.externe.com` pour déléguer la gestion du sous-domaine `sousdomaine.exemple.com` à un serveur DNS externe. Vous devez remplacer `ns.serveurdns.externe.com` par le nom de domaine complet du serveur DNS externe qui gérera le sous-domaine.

### Configuration sur le serveur DNS externe (ns.sousdomaine.exemple.com) :

1. Configurez le serveur DNS externe pour répondre aux requêtes pour le sous-domaine `sousdomaine.exemple.com`.

2. Créez les enregistrements DNS appropriés pour le sous-domaine `sousdomaine.exemple.com` sur le serveur DNS externe, par exemple :

```plaintext
; Configuration sur le serveur DNS externe
$TTL 3600

@      IN      SOA    ns.sousdomaine.exemple.com. admin.sousdomaine.exemple.com. (
                  2023091301 ; Numéro de série
                  3600       ; Intervalle de rafraîchissement
                  1800       ; Intervalle de réessai
                  604800     ; Intervalle d'expiration
                  86400 )    ; Durée minimale de cache

@      IN      NS     ns.sousdomaine.exemple.com.
sousdomaine IN      A      203.0.113.100  ; Adresse IP du serveur pour sousdomaine.exemple.com
```

3. Assurez-vous que le serveur DNS externe est configuré pour répondre aux requêtes DNS pour le sous-domaine `sousdomaine.exemple.com` et qu'il pointe correctement vers l'adresse IP `203.0.113.100`.

De cette manière, la gestion du sous-domaine `sousdomaine.exemple.com` est déléguée au serveur DNS externe `ns.sousdomaine.exemple.com.`, et ce dernier est responsable de la résolution DNS pour ce sous-domaine. Le serveur BIND gère toujours la zone principale pour `exemple.com`.

## Division des Zones (Privées et Publiques)

Les zones DNS peuvent être divisées en zones privées et publiques. Les zones publiques contiennent des informations accessibles au public, telles que les enregistrements DNS pour un site web. Les zones privées, en revanche, contiennent des informations spécifiques à un réseau privé, comme les enregistrements pour les serveurs internes. La division permet de contrôler l'accès aux informations DNS en fonction des besoins de sécurité.

En conclusion, le DNS est un système fondamental pour le fonctionnement d'Internet. Il repose sur une hiérarchie de serveurs, des résolveurs pour la résolution des requêtes DNS, et la délégation des zones pour gérer la répartition des responsabilités. La compréhension de ces concepts est essentielle pour la gestion efficace des noms de domaine et la gestion des ressources Internet.

### DNS Partagé (Split-Horizon DNS) :

Le DNS partagé, également appelé DNS split-horizon, implique d'avoir des vues différentes de l'espace de noms DNS pour les utilisateurs internes et externes. Cela est souvent utilisé pour fournir des réponses DNS différentes en fonction de la provenance de la requête DNS, c'est-à-dire si la requête provient de l'intérieur ou de l'extérieur du réseau local. Pour mettre en œuvre le DNS partagé avec BIND9, vous configurez généralement des vues distinctes dans le fichier de configuration BIND.

Voici un exemple simplifié :

```shell
view "interne" {
    match-clients { localhost; 172.16.x.0/24; };
    zone "exemple.com" {
        type master;
        file "/etc/bind/db.interne";
    };
};

view "externe" {
    match-clients { any; };
    zone "exemple.com" {
        type master;
        file "/etc/bind/db.externe";
    };
};
```

Dans cet exemple, les utilisateurs provenant du réseau local (172.16.x.0/24) verront une vue DNS définie par le fichier ````/etc/bind/db.interne````, tandis que les utilisateurs externes verront une vue définie par le fichier ````/etc/bind/db.externe````.


### Fichier /etc/bind/named.conf.local

Il faut choisir la bonne zone en fonction des IP sources via des ACL

```shell
    
    //zone externe
    view "outside" {
        match-clients { 
            !172.x.x.0/24;    //requète de provenant pas du LAN
            !192.168.x.0/24;    //requète de provenant pas de la DMZ
            any;                //Toutes les autres adresses
        };

        zone "ville.sportludique.fr." {
            type master;
            file "/etc/bind/db.ville.sp.fr.externe";
        };
    };

    view "inside" {

        match-clients { 
            172.x.0.0/24;    //requète provenant du LAN
            192.168.x.0/24;    //requète provenant de la DMZ
        };

        zone "ville.sportludique.fr." {
            type master;
            file "/etc/bind/db.ville.sp.fr.interne";
        };

    };

```

??? info "Attention"
    Toutes les lignes du fichier `named.conf.default-zones` doivent être mises `en commentaire` car ne elles ne sont associées à aucune vue.


