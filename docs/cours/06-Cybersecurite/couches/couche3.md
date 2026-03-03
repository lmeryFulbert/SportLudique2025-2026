# 3. Couche 3 – Réseau

## IP Spoofing

**Traduction :** Usurpation d’adresse IP

### Principe :
L’attaquant falsifie l’adresse IP source des paquets.

### Contre-mesures :

- Filtrage anti-spoofing
- Firewall
- Segmentation réseau

## Attaques DDoS - type Flood

**Traduction :** Attaque par saturation

- ICMP Flood / Ping Flood : inonder une cible de echo requests pour saturer sa bande passante ou ses ressources.

- Smurf Attack : envoyer des requêtes ICMP à une adresse broadcast avec une source spoofée => toutes les machines répondent à la victime, la surchargeant.

### Contre-mesures :

- Rate limiting
- Filtrage ICMP

## Attaques Protocole de routage Dynamique

Un attaquant peut générer de faux messages de routage (protocoles de routage comme RIP/OSPF dans des réseaux internes) pour faire passer le trafic par un mauvais chemin, intercepter ou perturber les communications.


