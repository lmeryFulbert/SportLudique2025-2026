# Introduction sur l'infrastructure

## <span style="color:red;">Dossier technique de l’infrastructure du réseau *SportLudique*</span>

## Lycée Fulbert Chartres 2025-2026

**Auteur :** Ludovic MERY

## Introduction :

Les Actions Professionnelles (AP) seront développées autour du
contexte *SportLudique*. Cet environnement permettra à l’étudiant de
réaliser un ensemble de projets autour de différentes situations
professionnelles (les missions) et acquérir ainsi les compétences en
conformité avec le référentiel.

## Rappel du contexte :

*SportLudique* est une société basée sur 4 sites distants **Chartres**,
**Tours**, **Orléans** et **Bourges**. Elle est spécialisée dans la conception
et la production d’articles de sports non conventionnels.

L’infrastructure informatique et réseau est prise en charge par le
département **"Solutions d’Infrastructure Système et Réseau" (SISR)** qui se
charge de l’exploitation de l’ensemble des équipements du parc ainsi que
de la réalisation des projets informatiques.

La DSI délègue les taches techniques de réalisation, déploiement,
maintenance et support à son prestataire maître d’œuvre autour d’un
contrat de service dans lequel il assure la maîtrise d’ouvrage.

## Présentation Générale de l’infrastructure :

L’infrastructure est découpée en 3 parties comprenant
le siège ainsi que les différents sites distants et le réseau d’accès à
Internet. La DSI est responsable de l’administration de
l’infrastructure, de la fourniture de services et de la sécurisation des
données en déléguant ces tâches à son prestataire homologué qui est le
maître d’œuvre pour la réalisation des opérations d’administration du
système d’information de *SportLudique*.

Le service Informatique de SportLudique est chargé de la mise en œuvre et de la gestion de l’ensemble des projets relatifs au parc informatique de l’entreprise. Il agit en tant que prestataire à travers des contrats de service pour les différents sites distants.
Chaque site dispose de sa propre infrastructure et utilise un domaine de type :

````bash
   <nomville>.sportludique.fr.
````

Un Controlleur de domaine disposant d'un service d’annuaire Active Directory assure la gestion et
l’administration du réseau.

-  Les serveurs publics situés en DMZ doivent être accessibles sous la forme :

````bash 
<codeservice>.<ville>.sportludique.fr
````

??? important "Remarque"
    **<codeservice>** correspond au protocole ou à la convention associé.
    Exemple: <br/>
    www   -->  pour un serveur http/https <br/>
    ns --> pour un serveur DNS <br/>
    smtp --> Pour un serveur d'envoi de mail (messagerie) <br/>

Pour la partie LAN, il est nécessaire de définir une délégation DNS interne spécifique à chaque site.

Exemple d’approche déjà utilisée au sein de la section SIO du lycée Fulbert:  

````bash 
lan.sio.lyceefulbert.fr
````

Si vous adoptez  un schéma du type ````lan.<ville>.sportludique.fr```` les machines risquent d’être confondues avec les domaines Active Directory utilisés pour l’authentification.

## Bonnes pratiques DNS et recommandations

Sur le sujet des domaines internes Active Directory, l’ancienne pratique courante consistait à utiliser un TLD inventé (ex. : ville.local ou entreprise.local).

!!!! warning "Attention"
     Ce n’est plus recommandé car .local est réservé par RFC 6762 pour mDNS (Multicast DNS), ce qui entraîne des conflits avec macOS, Linux et certains services modernes.

Aujourd’hui, les bonnes pratiques Microsoft et celles de l’IETF recommandent :

-  d’utiliser un sous-domaine du domaine public existant (ex. : ad.ville.sportludique.fr ou corp.ville.sportludique.fr),

-  ou d’utiliser un préfixe réservé à l’interne clairement distinct de la zone publique (int.ville.sportludique.fr, ad.ville.sportludique.fr, lan.ville.sportludique.fr).

Microsoft précise bien que le FQDN interne doit être routable et enregistré dans un domaine possédé par l’entreprise afin d’éviter les problèmes de certificats, d’authentification Kerberos et d’intégration avec Azure/Office365.

### Problématique de nommage AD et NetBIOS et délégation DNS

L’infrastructure DNS de SportLudique est organisée de manière hiérarchique :  
- La zone principale `sportludique.fr` est gérée par votre enseignant.  
- Chaque ville dispose d’une **sous-zone déléguée** :  
  - `bourges.sportludique.fr`  
  - `chartres.sportludique.fr`  
  - `orleans.sportludique.fr`  
  - `tours.sportludique.fr`  

À l’intérieur de chaque sous-zone, les étudiants doivent créer leur domaine Active Directory.  

!!!! warning "Attention"
    Problème : le nom du domaine AD détermine automatiquement un **nom NetBIOS** (15 caractères maximum, dérivé du premier label).  
    Si tout le monde crée un domaine du type `lan.<ville>.sportludique.fr`, le NetBIOS sera toujours `LAN`, entraînant des confusions lors des ouvertures de session (`LAN\user`).

Pour éviter cela, nous imposons un schéma de nommage qui intègre un **préfixe unique par ville** directement dans le nom DNS du domaine.  
Ainsi, le nom NetBIOS reflète aussi la ville concernée, ce qui permet une identification rapide et sans ambiguïté.

### Proposition de nommage

- Chartres : `cha.chartres.sportludique.fr` → NetBIOS = `CHA`  
- Orléans : `orl.orleans.sportludique.fr` → NetBIOS = `ORL`  
- Tours : `trs.tours.sportludique.fr` → NetBIOS = `TRS`  
- Bourges : `brg.bourges.sportludique.fr` → NetBIOS = `BRG` 

````bash

sportludique.fr  (zone principale publique, gérée par vos profs)
│
├── chartres.sportludique.fr  (sous-zone déléguée)
│   └── cha.chartres.sportludique.fr  (domaine AD interne)
│       └─ NetBIOS = CHA
│
├── orleans.sportludique.fr  (sous-zone déléguée)
│   └── orl.orleans.sportludique.fr  (domaine AD interne)
│       └─ NetBIOS = ORL
│
└── tours.sportludique.fr  (sous-zone déléguée)
│    └── trs.tours.sportludique.fr  (domaine AD interne)
│        └─ NetBIOS = TRS
│
├── bourges.sportludique.fr  (sous-zone déléguée)
│   └── brg.bourges.sportludique.fr  (domaine AD interne)
│       └─ NetBIOS = BRG

````

## Gestion du DNS

Le nom de domaine *sportludique.fr* a été réservé auprès de l’AFNIC et
le bureau d’enregistrement (registar) gérant ce nom de domaine est
nordnet (en réalité il est géré par l’enseignant M MERY et n'est donc pas accessible sur le web de chez vous, il le sera via l'utilisation d'un VPN mis à disposition par votre enseignant).

### WHOIS

Le nom de domaine "**sportludique.fr**" est déjà déposé.

 Résultat de votre recherche

-   **Nom de domaine :** sportludique.fr

-   **État :** Actif

-   **DNSSEC :** inactif

-   **Bureau d'enregistrement
    > : [**NORDNET**](http://www.afnic.fr/fr/produits-et-services/services/whois/)

-   **Date de création : **29 mai 2020 17:37

-   **Date d'expiration : **29 mai 2030 17:37

-   **Serveurs de noms (DNS)**

    -   ***Serveur n° 1: **ns2.lerelaisinternet.com*

    -   ***Serveur n° 2: **ns1.lerelaisinternet.com*