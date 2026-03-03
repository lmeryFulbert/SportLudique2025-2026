# 4. Couche 4 – Transport

## SYN Flood
**Traduction :** Attaque par saturation SYN

### Principe :

L’attaquant envoie de nombreuses requêtes TCP SYN sans terminer la connexion (n'envoi pas le ACK en réponse au SYN du serveur).

### Conséquences :

- Déni de service (DoS)

### Contre-mesures :

- SYN cookies
- Firewall
- Load balancer

## UDP Flood

**Traduction :** Attaque par saturation UDP

### Principe :
Envoi massif de paquets UDP vers la cible.

### Contre-mesures :

- Filtrage
- Rate limiting

