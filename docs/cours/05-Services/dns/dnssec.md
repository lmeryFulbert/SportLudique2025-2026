# DNSSEC (Domain Name System Security Extensions)

## Introduction

Le DNS (Domain Name System) est le système qui traduit les noms de domaine lisibles par l’homme (ex. www.example.com) en adresses IP compréhensibles par les machines.
Le problème : le DNS classique n’est pas sécurisé. N’importe qui peut falsifier une réponse DNS, ce qui peut entraîner des attaques comme :

- `DNS Spoofing` : répondre avec une fausse IP pour un site légitime.
- `Man-in-the-middle` : intercepter et modifier les requêtes DNS.

`DNSSEC` a été créé pour résoudre ce problème : il apporte l’`authenticité` et l’`intégrité` des réponses DNS, mais pas la confidentialité.
Vous remarquez donc que ce sont les même apports que la `signature éléctronique`

## Principe de DNSSEC

1. Chaque zone DNS a une clé privée unique

2. Chaque enregistrement DNS (A, MX…) est signé avec cette clé privée.

3. La zone stocke la clé publique qui correspond à cette clé privée (enregistremnt DNSKEY)

4. Le résolveur DNS utilise cette clé publique pour vérifier que la réponse est authentique.

la zone signe les réponses et le client vérifie la signature.

## POrincipe du hash (intégrité) et de l'authentification (verifié avec clé publique de l'emetteur)

Hash :

La zone DNS prend chaque enregistrement et calcule un résumé unique appelé hash.

Ce hash est une empreinte numérique : même si un seul bit change, le hash change complètement.

Signature :

La zone utilise sa clé privée pour signer ce hash.

Le résultat est la signature (RRSIG) attachée à l’enregistrement.

Vérification côté client :

Le client récupère l’enregistrement et sa signature.

Il calcule lui-même le hash de l’enregistrement.

Il utilise la clé publique (contenu dans la zone interrogée via l'enregistrement DNSSEC) pour vérifier la signature et obtenir le hash original.

Si les deux hash correspondent --> la réponse est authentique.

Sinon --> la réponse a été modifiée ou falsifiée.


Pour en savoir plus : [Comment fonctionne le protocole DNSSEC ? – AmenSchool](https://www.amenschool.fr/comment-fonctionne-le-protocole-dnssec/)
