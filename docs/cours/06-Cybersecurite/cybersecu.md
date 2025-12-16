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
Accès non autorisé à un équipement (serveur, switch, poste client).

**Exemples :**
- Vol de disque dur
- Branchement d’un périphérique USB malveillant

**Contre-mesures :**
- Contrôle d’accès physique
- Baies fermées
- Désactivation du boot USB

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
- Switchs managés
  - Dynamic ARP Inspection (DAI)
  - DHCP Snooping
- VLAN : Segmentation du réseau
- VLAN isolés / Private VLAN
    - Empêche la communication directe entre postes d’un même VLAN
 - Port Security
   - Limitation du nombre d’adresses MAC par port
- Filtrage ARP statique (réseaux très restreints)

L’ARP spoofing est une attaque locale :
la meilleure défense est la segmentation et l’isolation réseau, pas uniquement le chiffrement.

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

### Conséquences
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
