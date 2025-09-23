# VLSM (Variable Length Subnet Mask)
## Introduction

Le **VLSM (Variable Length Subnet Mask)** est une technique d'adressage IP qui permet de subdiviser un réseau en sous-réseaux de tailles différentes, en fonction des besoins. Contrairement à la technique classique des sous réseaux - **FLSM (Fixed Length Subnet Mask)**, où tous les sous-réseaux ont la même taille, le VLSM permet une utilisation plus efficace de l'espace d'adressage.

### Avantages du VLSM

-    Efficacité de l'utilisation des adresses IP : Permet de minimiser le gaspillage des adresses IP en allouant des sous-réseaux de tailles adaptées aux besoins.
-    Flexibilité : Permet de créer des sous-réseaux de différentes tailles à partir du même espace d'adressage.
-    Scalabilité : Facilite l'extension du réseau en créant de nouveaux sous-réseaux selon les besoins.

!!! danger "Attention"
    **VLSM** est une technique d'optimisation pour l'adressage utile dans des plages d'adresse publiques. Dans un contexte d'adresse privées (**RFC1918**), cela a nettement moins d'interet.

??? info "RFC 1918"
    Les plages d'adresses privées définies dans cette RFC interdisent leur routage sur Internet. On a :</br>
    -    10.0.0.0 /8  </br>
    -    172.16.0.0 /12 </br>
    -    192.168.0.0 /16 </br>

### Principe de fonctionnement

-    Détermination des besoins en sous-réseaux : Identifiez le nombre de sous-réseaux et le nombre d'hôtes requis pour chaque sous-réseau.

-    Allouer des sous-réseaux à l'aide de masques de sous-réseau de longueurs variables :
-    Commencez par le plus grand sous-réseau (Il est très important de trier avant de commencer a travailler).
-    Choisissez un masque de sous-réseau qui satisfait le nombre d'hôtes requis.
-    Continuez à subdiviser les sous-réseaux restants pour répondre aux autres besoins en utilisant des masques de sous-réseau appropriés.

## Algorithme pour réussir un adressage VLSM 

??? info "Attention"
    Il est important de trier les réseaux du plus grand au plus petit !

1. **Lister les sous-réseaux**  
   Identifier tous les sous-réseaux nécessaires avec le nombre d’hôtes requis pour chacun.

2. **Trier les sous-réseaux par nombre d’hôtes décroissant**  
   - Traiter d’abord le plus grand sous-réseau (celui avec le plus d’hôtes).  
   - Cela permet d’allouer correctement les plages d’adresses sans conflit.

3. **Attribuer un masque à chaque sous-réseau**  
   - Pour chaque sous-réseau, calculer le nombre de bits nécessaires pour les hôtes :  
     ```
     bits_hotes = plus petit entier n tel que 2^n - 2 >= nombre d’hôtes requis

     le -2 = @ réseau et @ Diffusion
     ```  
   - Le masque de sous-réseau correspond à :  
     ```
     / (32 - bits_hotes)
     ```

4. **Allouer les adresses**  
   - Commencer à partir de l’adresse réseau initiale.  
   - Pour chaque sous-réseau, définir :  
     - l’adresse réseau,  
     - la plage d’adresses utilisables,  
     - l’adresse de broadcast.  

5. **Recommancer**
   - Passer à l’adresse immédiatement après la plage allouée pour le sous-réseau suivant.  
   - Continuer jusqu’à ce que tous les sous-réseaux soient alloués.


### Exemple de mise en oeuvre

Supposons que vous avez une adresse réseau 192.168.1.0/24 et que vous devez créer les sous-réseaux suivants en utilisant la technique VLSM :

-    Sous-réseau A : 50 hôtes
-    Sous-réseau B : 25 hôtes
-    Sous-réseau C : 10 hôtes

#### Étapes

-    Sous-réseau A : 50 hôtes → Nécessite 6 bits pour les hôtes (2^6 - 2 = 62 hôtes).
-    Masque de sous-réseau : /26 (255.255.255.192)
-    Sous-réseau : 192.168.1.0/26
-    Plage d'adresses : 192.168.1.1 à 192.168.1.62
----
-    Sous-réseau B : 25 hôtes → Nécessite 5 bits pour les hôtes (2^5 - 2 = 30 hôtes).
-    Masque de sous-réseau : /27 (255.255.255.224)
-    Sous-réseau : 192.168.1.64/27
-    Plage d'adresses : 192.168.1.65 à 192.168.1.94
----
-    Sous-réseau C : 10 hôtes → Nécessite 4 bits pour les hôtes (2^4 - 2 = 14 hôtes).
-    Masque de sous-réseau : /28 (255.255.255.240)
-    Sous-réseau : 192.168.1.96/28
-    Plage d'adresses : 192.168.1.97 à 192.168.1.110

## Remettre son travail dans un tableau

| Sous-réseau | Nombre d’hôtes | Bits hôtes | Masque | Adresse réseau     | Plage d’adresses       | Adresse broadcast |
|------------|----------------|-----------|--------|------------------|----------------------|-----------------|
| A          | 50             | 6         | /26    | 192.168.1.0      | 192.168.1.1 - 192.168.1.62  | 192.168.1.63   |
| B          | 25             | 5         | /27    | 192.168.1.64     | 192.168.1.65 - 192.168.1.94 | 192.168.1.95   |
| C          | 10             | 4         | /28    | 192.168.1.96     | 192.168.1.97 - 192.168.1.110| 192.168.1.111  |

## Conclusion

La moindre erreur de calcul entrainera une erreur complète de l'espace d'adressage ! Soyez vigilent !
