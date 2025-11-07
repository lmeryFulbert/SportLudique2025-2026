# Resilience du Service critique DNS

## Tolerance aux pannes

Dans une architecture réseau professionnelle, garantir la disponibilité des services critiques est un enjeu essentiel. Pour éviter qu’une panne n’entraîne une interruption complète, on s’appuie souvent sur le principe de redondance, en déployant des serveurs secondaires capables de prendre le relais en cas de défaillance du serveur principal.
Le système DNS illustre parfaitement ce mécanisme.

## Pourquoi un serveur secondaire

Un serveur DNS primaire (ou master) est celui qui détient la zone DNS en écriture : il héberge le fichier de zone original et permet les modifications.
Si ce serveur tombe en panne, sans solution de secours, plus aucune résolution de nom ne serait possible pour les domaines qu’il gère.

Pour assurer la tolérance aux pannes, on met en place un serveur secondaire (ou slave). Celui-ci :

- héberge une copie en lecture seule de la zone ;
- peut répondre aux requêtes des clients à la place du primaire ;
- augmente la disponibilité globale du service ;
- permet une eventuelle répartition de la charge (load balancing simple).

## Syncronisation avec le Transfert de Zone

Pour que le secondaire dispose d’une copie fidèle du fichier de zone, le DNS utilise un mécanisme automatique : le transfert de zone (zone transfer).

Il existe deux types principaux :

✅ AXFR (Full Zone Transfer)

Transfert complet de la zone :
Le secondaire récupère tout le contenu du fichier de zone depuis le primaire.
Cela arrive :
- lors de la première synchronisation ;
- ou si le fichier de zone change radicalement.

✅ IXFR (Incremental Zone Transfer)

Transfert incrémentiel :
Le secondaire télécharge uniquement les changements (deltas) depuis la dernière version.
Ce mécanisme réduit :
- la charge réseau,
- les temps de synchronisation.

### Rôle du Serial Number

Chaque fichier de zone contient un champ Serial dans l’enregistrement SOA.
Il indique la version de la zone.
Le secondaire compare :
- son Serial local
- le Serial du primaire

Si celui du primaire est supérieur, le secondaire déclenche un transfert IXFR (incrementiel) si possible, sinon AXFR (complet).

## Securisation du transfert de zone

Un point essentiel : ne jamais laisser le transfert de zone ouvert à tout le monde.
Sinon, n’importe qui pourrait télécharger l’intégralité de la zone (ce qui serait un risque de sécurité).

On sécurise en :

- limitant les adresses IP autorisées (ACLS côté primaire) ;
- utilisant des clés TSIG pour authentifier le transfert ;
- filtrant les ports côté firewall (TCP/53).

??? info "clé TSIG"
    Une `clé TSIG` est un mécanisme défini par la `RFC 2845` permettant d’authentifier de façon sécurisée certaines opérations DNS sensibles. Contrairement au transfert de zone, qui consiste à copier une zone du serveur maître vers un serveur esclave, mon utilisation n’avait rien à voir avec cela : <br/>dans mon cas, j’ai utilisé TSIG pour autoriser des mises à jour DNS dynamiques, telles que définies dans la `RFC 2136`. Ce mécanisme permet à un client ( Certbot ) d’ajouter automatiquement un enregistrement TXT dans la zone DNS lors du challenge DNS-01 décrit par la `RFC 8555 (protocole ACME)`.<br>
    Cela prouve à Let’s Encrypt que je suis bien propriétaire du domaine, et permet d’automatiser le renouvellement de notre certificat `wildcard`.
    <br>Cela depasse un peu le cadre du BTS SIO.

## Résilience globale

Grâce à ce mécanisme, même si le primaire devient :

- indisponible (panne matérielle, réseau, maintenance),ou surchargé,
- le secondaire continue de répondre. La zone reste accessible, même si elle n’est plus modifiable temporairement.

Une bonne pratique consiste à avoir au moins un secondaire, parfois plus (y compris géographiquement répartis).
C’est indispensable pour atteindre une haute disponibilité du DNS.

L’ANSSI recommande de disposer d’au moins deux serveurs DNS sur des infrastructures séparées afin d’assurer la haute disponibilité et la résilience du service. Concrètement, cela implique :

- Deux adresses IP publiques distinctes, idéalement fournies par des fournisseurs différents (ce qui correspond à notre configuration avec deux routeurs de deux fournisseurs).

- Deux infrastructures de virtualisation indépendantes, reposant sur des serveurs physiques distincts (dans notre cas : Nutanix et Proxmox), afin qu’une panne matérielle ou logicielle sur l’une n’affecte pas l’autre.

Cette architecture garantit que le service DNS reste accessible même en cas de défaillance d’un fournisseur, d’un routeur ou d’une infrastructure de virtualisation, répondant ainsi aux bonnes pratiques de sécurité et de disponibilité recommandées par l’ANSSI.

## Schema de l'infra DNS redondante

![](.../../../../../medias/cours/dns/redondance_dns.drawio.png)

### Contenu du fichier de zone en prenant en compte cette resilience

````bash
; La durée de vie des enregistrements est de 12 heures. Les serveurs DNS récursifs
; stockeront les informations sur cette zone dans leur cache pendant ce laps de temps
$TTL 43200 ; 12 heures

; l’adresse du contact technique est postmaster@<ville>.sportludique.fr
<ville>.sportludique.fr. IN SOA ns1.<ville>.sportludique.fr. postmaster.<ville>.sportludique.fr. (
2025110701 ; Serial
1D ; Refresh
1H ; Retry
1W ; Expire
3H ) ; Negative Cache TTL

; Déclaration des serveurs DNS
<ville>.sportludique.fr. IN NS ns1.<ville>.sportludique.fr.
<ville>.sportludique.fr. IN NS ns2.<ville>.sportludique.fr.

; Serveurs DNS primaire et secondaire (infrastructures séparées)
ns1.<ville>.sportludique.fr. IN A 183.44.x.y  ;IPPubFAI1
ns2.<ville>.sportludique.fr. IN A 221.87.x.y   ;IPPubFAI2
````

### Activer le transfert de zone vers un serveur identifié

#### Sur le serveur primaire (Master)

````bash
zone "<ville>.sportludique.fr" {
    type master;
    file "/etc/bind/db.<ville>.sportludique.fr";
    allow-transfer { a.b.c.d; };   // IP du serveur secondaire
    allow-query { any; };
};
````

#### Sur le serveur secondaire (Slave)

C’est ici qu'on actives réellement le transfert de zone : Il doit aller chercher la zone auprès du maître.

````bash
zone "<ville>.sportludique.fr" {
    type slave;
    masters { a.b.c.d; };           // IP du master
    file "/var/cache/bind/slave.<ville>.sportludique.fr";
};
````


