# 02- Resolveur DNS

Un résolveur DNS est un logiciel ou un service responsable de la résolution des noms de domaine en adresses IP. Les résolveurs sont généralement utilisés par les appareils clients pour interroger le système DNS et obtenir les adresses IP associées aux noms de domaine.

## Avec Bind

Dans le fichier `/etc/bind/named.conf.options`

```plaintext
    options {
            directory "/var/cache/bind";

            //Autorise les requetes recursives
            recursion yes;  

            //Liste des réseaux autorisés à interoger le resolver
            //Par defaut seul les équipements du meme réseau IP que le serveur peuvent l\'interroger.
            allow-query { 
                172.16.x.0/24;       //LAN
                127.0.0.1;          //LOCALHOST
                192.168.x.0/24;    //DMZ
            };

            //Desactivation de DNSSec
            dnssec-validation no;

            //Ecoute sur l\'ensemble des interfaces IPv4
            listen-on { any; };
    };
```

## Configuration des redirecteurs

Dans le fichier `/etc/bind/named.conf.local`

Les requetes à destination de la zone locale doivent être redirigées vers le serveur DNS de la DMZ en interogeant la vue interne.

```shell
zone "ville.sportludique.fr" {
    type forward;
    forwarders { 192.168.x.y };  // Remplace par l'IP du serveur DNS ayant autorité sur la zone (dans la DMZ)
};
```

Les requetes à destination de la zone de l'entreprise (sportludique.fr) doivent partir vers le serveur resolver de l'enseignant gérant cette zone.

```shell
zone "sportludique.fr" {
    type forward;
    forwarders { 121.183.90.205; };  // IP du serveur de l'enseignant ayant autorité sur la zone sportludique.fr
};
```

Toutes les autres requetes sont recursives et interrogent donc les serveurs racines (connu de Bind).
Cela permet d'éviter une potentielle censure en utilisant le serveur DNS d'un Opérateur (celui du prof :-) )
