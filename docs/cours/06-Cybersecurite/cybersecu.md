# Cybersécurité

Ce chapitre présente les **principales attaques informatiques**, classées selon les **couches du modèle OSI**, de la plus basse (physique) à la plus haute (applicative).

Cette liste est **non exhaustive** :  

- Les techniques d’attaque évoluent constamment  
- De nouvelles vulnérabilités apparaissent régulièrement  

L’objectif est de :

- Comprendre les **mécanismes généraux des attaques**
- Associer chaque attaque à des **contre-mesures adaptées**
- Donner une **culture cybersécurité** commune aux étudiants

---
## 1. Couche 1 – Physique

### Physical Access Attack

**Traduction :** Attaque par accès physique

**Principe :**

Une attaque par accès physique consiste pour un individu malveillant à accéder physiquement à un équipement informatique afin de :

- Voler des données
- Installer un dispositif malveillant
- Contourner les mécanismes de sécurité logiques
- Obtenir un accès réseau non autorisé

!!! important "A retenir"
    Contrairement aux attaques réseau, ici l’attaquant est physiquement présent dans les locaux.

Beaucoup d’infrastructures sont très sécurisées logiquement (ACL, firewall, VLAN, MFA…),
mais deviennent vulnérables si quelqu’un peut :

- Brancher un PC sur une prise réseau
- Démarrer un serveur sur une clé USB
- Accéder directement aux disques durs
- Se connecter sur un port console

!!! important "A retenir"
    “Si un attaquant a un accès physique, il a presque gagné.”

**Exemples :**

- Un individu branche son ordinateur portable sur une prise RJ45 libre.
- Vol de disque dur
- Branchement d’un périphérique USB malveillant
- accès au port console d'un switch ( line console et mode enable non protégé par mot de passe)
- Laisser une session ouverte sur une station sans surveillance.

**Contre-mesures :**

- Contrôle d’accès physique de la salle des serveurs :
    - Baies fermées
    - Vidéo surveillance
    - Registre des entrées / sorties

- Sécurisation des ports physique
    - Ne jamais utiliser le vlan 1 (par défaut)
    - Créer un VLAN HoneyPot (ex: VLAN 999)
    - Sécuriser les ports console si les switchs sont accessibles physiquement
    - Mettre les ports inutilisés dans ce VLAN
    - Désactiver les ports inutilisés
    - Mettre en place un Vlan "Invité ou Guest"
    - Mettre une authentification 802.1X (voir chapitre dédié) pour accéder à un port
    - Activer le Port Security (Cisco) en limiter les Macs autorisées sur un port

Exemple n'autorisant qu'une seule Mac sur l'interface fa0/1
```bash
interface fa0/1
switchport mode access
switchport port-security
switchport port-security maximum 1
switchport port-security violation shutdown
```

- Sécurisation des stations de travail
    - Désactivation du boot USB dans le BIOS
    - Mot de passe BIOS
    - BitLocker activé (Chiffrement des données)
    - GPO pour bloquer les périphériques USB
    - Désactivation de l’automount


---

## 2. Couche 2 – Liaison de données

### ARP Spoofing / ARP Poisoning
**Traduction :** Usurpation / empoisonnement ARP

**Principe :**
L’attaquant se fait passer pour la passerelle afin d’intercepter le trafic réseau local.

**Conséquences :**

- Man‑In‑The‑Middle local
- Interception d’identifiants (confidentialité)
- Modification du trafic (intégrité)
- Déni de service local (Disponibilité)

**Contre-mesures :**

- Switchs manageable (éviter les switchs grand public du commerce)
- DHCP Snooping (permet au switch de bloquer les serveurs DHCP non autorisés afin d’empêcher une attaque de type **DHCP Spoofing** (faux serveur DHCP).)
- VLAN : Segmentation du réseau
- VLAN isolés / Private VLAN
    - Empêche la communication directe entre postes d’un même VLAN
- Port Security:  Limitation du nombre d’adresses MAC par port
- Filtrage ARP statique (réseaux très restreints): vlan de niveau 2

!!! important "A retenir"
      L’ARP spoofing est une attaque locale :
      la meilleure défense est la segmentation et l’isolation réseau, pas uniquement le chiffrement.

#### DHCP Snooping

**Traduction :** Surveillance / filtrage DHCP

**Principe :**  
Le DHCP Snooping permet au switch de bloquer les serveurs DHCP non autorisés  
afin d’empêcher une attaque de type **DHCP Spoofing** (faux serveur DHCP).

Le switch distingue :
- Ports **trusted** (serveur DHCP légitime)
- Ports **untrusted** (postes clients)

Seuls les ports "trusted" peuvent envoyer des réponses DHCP.

**Fonctionnement :**

  1. Le switch intercepte les messages DHCP.
  2. Il autorise uniquement les réponses venant d’un port marqué "trusted".
  3. Il construit une base de données IP ↔ MAC ↔ Port (binding table).

Exemple de configuration

```bash
ip dhcp snooping
ip dhcp snooping vlan 10,20,30  # id Vlans ou se trouvent les clients légitimes

interface g0/1    # le port sur lequel se trouve le serveur DHCP ou le relai DHCP (souvent un port d'interco 802.1Q)
ip dhcp snooping trust   # autorise a voir des réponses DHCP

interface g0/10   # on limite le nombre de requetes clients DHCP venant sur ce port
ip dhcp snooping limit rate 10 #Evite les attaques DHCP Starvation

```

!!! important "A retenir"
    Avec DHCP Snooping, seuls les ports “trusted” peuvent envoyer des réponses DHCP.
    En cas de relay, le port vers le routeur ou serveur doit être trusted.

---

### MAC Flooding
**Traduction :** Saturation de table MAC

**Principe :**
Inondation du switch avec de fausses adresses MAC pour le forcer à diffuser les trames.

**Contre-mesures :**

- Port Security
- Limitation du nombre de MAC par port


## Attaques DHCP

### DHCP Starvation
**Traduction :** Épuisement du serveur DHCP  

**Couche OSI :** 2 / 3  

### Principe
L’attaquant envoie un grand nombre de requêtes DHCP avec des **adresses MAC falsifiées** afin d’épuiser le pool d’adresses IP du serveur DHCP.

### Conséquences:

- Plus aucune adresse IP disponible
- Déni de service réseau (DoS)
- Nouveaux postes incapables de se connecter

### DHCP Spoofing (DHCP Rogue)
**Traduction :** Usurpation de serveur DHCP  

**Principe**
Après une attaque de starvation, l’attaquant met en place un **faux serveur DHCP** qui fournit :

- une fausse passerelle
- un faux DNS

Permet ensuite des attaques MITM ou DNS spoofing.

## Contre‑mesures

### Mesures réseau (essentielles)

- **Switchs managés**
  - **DHCP Snooping**
    - Ports *trusted* (serveur DHCP)
    - Ports *untrusted* (clients)
- **Rate limiting DHCP**
- **Port Security**
  - Limitation du nombre de MAC par port
- **VLAN / VLAN isolés**
  - Limiter la propagation de l’attaque

### Mesures serveur

- Pool DHCP dimensionné correctement
- Durée de bail adaptée
- Supervision du serveur DHCP

### Mesures complémentaires

- IP statiques pour équipements critiques
- IDS/IPS réseau
- Segmentation Wi‑Fi (invités / internes)

## Risque de confusion

| Attaque | Principe | Portée | Dangerosité |
|-------|----------|--------|-------------|
| DHCP Spoofing | Falsification de réponses DHCP | Ciblée | Moyenne |
| DHCP Rogue | Faux serveur DHCP complet | Réseau entier | Élevée |


Le DHCP est un service critique mais non sécurisé par défaut.  
Sans contrôle au niveau des switchs, il est vulnérable aux attaques locales.
L'isolation est donc indispensable pour limiter la surface d'attaque.

---

## 3. Couche 3 – Réseau

### IP Spoofing
**Traduction :** Usurpation d’adresse IP

**Principe :**
L’attaquant falsifie l’adresse IP source des paquets.

**Contre-mesures :**

- Filtrage anti-spoofing
- Firewall
- Segmentation réseau

### ICMP Flood
**Traduction :** Attaque par saturation ICMP

**Principe :**
Envoi massif de requêtes ICMP (ping) pour saturer la cible.

**Contre-mesures :**

- Rate limiting
- Filtrage ICMP

---
## 4. Couche 4 – Transport

### SYN Flood
**Traduction :** Attaque par saturation SYN

**Principe :**
L’attaquant envoie de nombreuses requêtes TCP SYN sans terminer la connexion (n'envoi pas le ACK en réponse au SYN du serveur).

**Conséquences :**
- Déni de service (DoS)

**Contre-mesures :**
- SYN cookies
- Firewall
- Load balancer

### UDP Flood
**Traduction :** Attaque par saturation UDP

**Principe :**
Envoi massif de paquets UDP vers la cible.

**Contre-mesures :**

- Filtrage
- Rate limiting

---
## 5. Couche 5 – Session

### Session Hijacking
**Traduction :** Détournement de session

**Principe :**
Vol ou réutilisation d’un identifiant de session valide.

**Contre-mesures :**
- HTTPS
- Cookies sécurisés (`HttpOnly`, `Secure`)
- Timeout de session

---

### SSL Stripping
**Traduction :** Suppression du chiffrement SSL

**Principe :**
Forcer l’utilisateur à rester en HTTP au lieu de HTTPS.

**Contre-mesures :**
- HTTPS
- HSTS

---

## 7. Couche 7 – Application

### SQL Injection (SQLi)
**Traduction :** Injection SQL

**Principe :**
Injection de requêtes SQL malveillantes via les entrées utilisateur.

**Contre-mesures :**
- Requêtes préparées
- Validation des entrées

### Cross-Site Scripting (XSS)
**Traduction :** Injection de script côté client

**Principe :**
Injection de code JavaScript malveillant dans une page web.

**Contre-mesures :**
- Encodage des sorties
- Content-Security-Policy (CSP)


### Cross-Site Request Forgery (CSRF)
**Traduction :** Falsification de requêtes inter-sites

**Principe :**
Forcer un utilisateur authentifié à exécuter une action à son insu.

**Contre-mesures :**
- Token CSRF
- Vérification de l’origine
