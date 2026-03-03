# 7. Couche 7 – Application

## SQL Injection (SQLi)

**Traduction :** Injection SQL

### Principe :
Injection de requêtes SQL malveillantes via les entrées utilisateur.

### Contre-mesures :

- Requêtes préparées
- Validation des entrées

## Cross-Site Scripting (XSS)

**Traduction :** Injection de script côté client

### Principe :
Injection de code JavaScript malveillant dans une page web.

### Contre-mesures :

- Encodage des sorties
- Content-Security-Policy (CSP)

## Cross-Site Request Forgery (CSRF)
**Traduction :** Falsification de requêtes inter-sites

### Principe :
Forcer un utilisateur authentifié à exécuter une action à son insu.

### Contre-mesures :

- Token CSRF
- Vérification de l’origine
