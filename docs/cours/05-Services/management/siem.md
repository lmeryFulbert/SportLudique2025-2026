# SIEM

## Introduction

Dans un paysage numérique où les cybermenaces évoluent en permanence, les organisations doivent être capables de détecter, analyser et répondre rapidement aux incidents de sécurité. C’est là qu’intervient le `SIEM (Security Information and Event Management)`, une solution indispensable pour toute infrastructure soucieuse de sa sécurité.

## role d'un SIEM

Un SIEM est une plateforme qui agrège, corrèle et analyse les événements de sécurité provenants de diverses sources (serveurs, réseaux, applications, équipements de sécurité) pour :

Détecter les menaces en temps réel (ex : attaques par force brute, mouvements latéraux, exfiltration de données).
Centraliser les logs de sécurité et les rendre exploitables via des tableaux de bord et des alertes.
Automatiser les réponses aux incidents (ex : blocage d’une IP, isolation d’un endpoint compromis).
Faciliter la conformité en générant des rapports pour les normes (RGPD, ISO 27001, PCI DSS, etc.).

## Présentation de Wazuh

Wazuh est une solution open source de détection d’intrusions (IDS), de monitoring de sécurité et de réponse aux incidents (SIEM), conçue pour offrir une protection proactive contre les cybermenaces. Contrairement à des outils de log management comme `Graylog`, Wazuh se concentre sur la détection d’activités malveillantes, la corrélation d’événements de sécurité, et l’application automatique de contre-mesures (ex : blocage d’IP, isolation de machines compromises).
Wazuh combine plusieurs fonctionnalités clés :

- Détection d’intrusions : Grâce à des règles de corrélation avancées (basées sur des signatures d’attaques connues et des comportements anormaux), Wazuh identifie les tentatives d’intrusion, les mouvements latéraux, ou les exfiltrations de données.

- Monitoring d’intégrité des fichiers (FIM) : Surveillance en temps réel des modifications critiques sur les systèmes (ex : altération de fichiers système, création de backdoors).
Conformité et audit : Génération de rapports pour les normes PCI DSS, RGPD, ISO 27001, etc., avec des templates prédéfinis.

- Réponse automatique : Intégration avec des outils comme OSSEC (moteur de détection) pour bloquer des IP, arrêter des processus suspects, ou déclencher des scripts de remédiation.

## Emplacement dans un SOC

Un SIEM comme Wazuh est le cœur analytique d’un SOC (`Security Operation Center`). Il permet de :

- Détecter les menaces en temps réel :
Identifier des attaques comme les brute-force, les mouvements latéraux, ou les exfiltrations de données grâce à des règles de corrélation avancées.

- Centraliser et normaliser les logs de sécurité :
Agrégation des logs provenants des firewalls, IDS/IPS, serveurs, endpoints, et applications, puis les rendre exploitables via des tableaux de bord et des alertes priorisées.

- Automatiser les réponses aux incidents :
Déclencher des actions comme le blocage d’une IP, l’isolation d’un endpoint compromis, ou l’envoi d’une alerte aux analystes du SOC.

- Faciliter la conformité :
Générer des rapports automatisés pour répondre aux exigences réglementaires (RGPD, ISO 27001, PCI DSS, etc.).

Le SIEM doit être intégré au SOC, car c’est là que les analystes surveillent, analysent et répondent aux incidents de sécurité. Cependant, son emplacement réseau est crucial pour garantir sécurité, disponibilité et performance.

Placer le SIEM dans un VLAN de management dédié, strictement contrôlé (ACL, chiffrement, accès restreint), dans l'idéal un vlan dédié au SOC serait pertinent.
Isoler physiquement ou logiquement le SIEM des réseaux de production et publics.
Utiliser des liens sécurisés (TLS, VPN) pour la collecte des logs depuis les autres VLANs.

### Exemple de deploiement

Exemple de déploiement avec Wazuh et Graylog

- Graylog (VLAN de management) :
Centralise tous les logs (applicatifs, réseau, systèmes) et les rend exploitables via des dashboards.
- Wazuh (SIEM, VLAN de management) :
Consomme les logs de Graylog, applique des règles de corrélation, et déclenche des alertes/réponses automatiques.
- SOC :
Les analystes utilisent Graylog pour l’analyse des logs et Wazuh pour la détection des menaces et la réponse aux incidents.

| Élément               | Recommandation                                                                 |
|-----------------------|-------------------------------------------------------------------------------|
| **Emplacement**       | VLAN de management **isolé et sécurisé**, accessible uniquement depuis le SOC. |
| **Accès**             | Restreint aux analystes SOC, avec MFA.                               |
| **Chiffrement**       | TLS pour tous les flux, chiffrement des logs sensibles.                      |
| **Intégration**       | SIEM connecté à Graylog (logs) et aux outils de réponse (firewalls, EDR).    |
| **Redondance**        | Cluster SIEM + sauvegardes des logs dans un VLAN dédié.                      |
| **Segmentation**      | ACL strictes pour limiter les flux entrants/sortants du VLAN de management.  |
| **Sauvegardes**       | Logs sauvegardés dans un emplacement isolé et chiffré.                       |
| **Haute disponibilité** | Déploiement en cluster pour éviter les points de défaillance uniques.        |

## Rappel architecture souhaitée

![architecture SOC](../../../medias/cours/graylog/graylog.drawio.png)