# Serveur de Log

## Objectifs

Apprendre à centraliser, analyser et exploiter les logs avec des outils modernes et ergonomiques, en évitant les solutions obsolètes comme `syslog-ng`.

## Introductiuon

Dans un environnement informatique, **les logs sont la mémoire technique** de votre infrastructure. Ils enregistrent chaque événement, chaque erreur, chaque tentative d’accès – et constituent donc une source d’information **critique** pour la sécurité, le dépannage et la conformité.

Un serveur de logs bien conçu répond aux **quatre piliers de la sécurité de l’information** :

- **Disponibilité** :
  Les logs doivent être **accessibles en temps réel** pour permettre une réaction rapide en cas d’incident (ex : panne, attaque).
  *Exemple* : Un serveur de logs centralisé évite de perdre des informations critiques si un équipement tombe en panne.

- **Intégrité** :
  Les logs doivent être **protégés contre toute altération** (volontaire ou accidentelle).
  *Exemple* : Un attaquant pourrait tenter de supprimer ses traces ; un serveur de logs sécurisé empêche cette manipulation.

- **Confidentialité** :
  Les logs contiennent souvent des **données sensibles** (adresses IP, noms d’utilisateurs, actions système).
  *Exemple* : Un accès non contrôlé aux logs peut exposer des informations personnelles (RGPD).

- **Preuve (Traçabilité)** :
  Les logs servent de **preuve légale** en cas d’audit, d’intrusion ou de litige.
  *Exemple* : En cas de violation de données, les logs permettent de retracer l’origine de l’incident et de prouver la conformité aux réglementations.

## Solution traditionnelles et limites

Historiquement, des outils comme `syslog-ng` ou `rsyslog` ont été utilisés pour centraliser les logs. Cependant, ils présentent plusieurs inconvénients majeurs :

- **Interface utilisateur absente ou obsolète** :
  La gestion des logs se fait souvent en ligne de commande, ce qui rend l’analyse **lente et complexe** pour les équipes opérationnelles.

- **Parsing et recherche limités** :
  Les logs sont stockés sous forme de texte brut, ce qui rend leur exploitation **peu efficace** pour des analyses avancées.

- **Scalabilité insuffisante** :
  Ces outils ne sont pas conçus pour gérer des **volumes massifs de logs** (ex : milliers de messages par seconde).

- **Manque de fonctionnalités modernes** :
  Pas d’alerting natif, pas de dashboards, pas d’intégration facile avec d’autres outils de monitoring ou de sécurité.

 Ces limites justifient le recours à des solutions modernes comme `Graylog`, `ELK` ou `Loki`.

## Présentation de Graylog

Graylog est une plateforme open source conçue pour simplifier la collecte, le stockage, l’analyse et la visualisation des logs, tout en offrant une interface utilisateur moderne et intuitive. Contrairement aux solutions traditionnelles comme syslog-ng, Graylog propose une expérience utilisateur complète, accessible même aux équipes non techniques, sans sacrifier la puissance ni la flexibilité.

### Fonctionnalités clés


- **Interface web ergonomique** :
Graylog offre un tableau de bord personnalisable, des outils de recherche avancée et des visualisations claires. Plus besoin de fouiller dans des fichiers texte ou de maîtriser des commandes complexes : tout est accessible via une interface graphique.
Exemple : Un administrateur peut filtrer les logs par niveau de sévérité, par source ou par période, en quelques clics.


- **Parsing et structuration des logs** :
Grâce à des outils intégré Graylog permet de transformer des logs bruts en données structurées et exploitables.

Exemple : Un log Apache brut 
````bash
(192.168.1.1 - - [10/Oct/2023:13:55:36 +0200] "GET /index.html HTTP/1.1" 200 2326)
````
peut être découpé en champs distincts : 

    - client_ip, 
    - timestamp, 
    - http_method, 
    - status_code, 
    - etc.


- **Alerting et notifications** :
Graylog permet de configurer des alertes en temps réel basées sur des conditions personnalisées (ex : plus de 10 erreurs 500 en 5 minutes). Les notifications peuvent être envoyées par email, Slack, ou via des webhooks.
Cas d’usage : Détecter une attaque par force brute en surveillant les tentatives de connexion échouées.

Bien que Graylog offre des fonctionnalités d’alerting avancées, il ne remplace pas un vrai SIEM (comme Wazuh, Splunk, ou QRadar) pour la détection d’intrusions sophistiquées


- **Scalabilité et performance** :
Graylog s’appuie sur Elasticsearch pour le stockage et la recherche, ce qui lui permet de gérer des volumes massifs de logs (millions de messages par jour). Il supporte également les déploiements en cluster pour une haute disponibilité.


- **Intégrations multiples**:
Graylog est compatible avec de nombreux protocoles (Syslog, GELF, Beats, Kafka) et s’intègre facilement avec des outils de monitoring (Prometheus, Grafana) ou de sécurité (SIEM).


