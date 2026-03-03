# 1. Couche 1 – Physique

## Physical Access Attack

**Traduction :** Attaque par accès physique

### Principe :

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

### Exemples :

- Un individu branche son ordinateur portable sur une prise RJ45 libre.
- Vol de disque dur
- Branchement d’un périphérique USB malveillant
- accès au port console d'un switch ( line console et mode enable non protégé par mot de passe)
- Laisser une session ouverte sur une station sans surveillance.

### Contre-mesures :

#### Contrôle d’accès physique de la salle des serveurs :

- Baies fermées à clé et dans un local sécurisé
- Vidéo surveillance
- Registre des entrées / sorties

#### Sécurisation des ports physique

 - Ne jamais utiliser le vlan 1 (par défaut)
 - Créer un VLAN HoneyPot (ex: VLAN 999) et mettre les ports inutilisés dans ce VLAN
 - Sécuriser les ports console si les switchs sont accessibles physiquement
 - Désactiver les ports inutilisés (shutdow)
 - Mettre en place un Vlan "Invité ou Guest"
 - Mettre une authentification 802.1X (voir chapitre dédié) pour accéder à un port
 - Activer le Port Security (Cisco) en limiter les Macs autorisées sur un port

#### Exemple de configuration Port Security (Cisco)

```bash
#Exemple n'autorisant qu'une seule Mac sur l'interface fa0/1
interface fa0/1
switchport mode access
switchport port-security
switchport port-security maximum 1
switchport port-security violation shutdown
```

#### Sécurisation des stations de travail
- Désactivation du boot USB dans le BIOS
- Définir un Mot de passe pour accéder au BIOS
- BitLocker activé (Chiffrement des données)
- GPO pour bloquer le montage des volumes via périphériques USB
- Désactivation de l’auto-mount (Linux) pour les periphériques externes USB
