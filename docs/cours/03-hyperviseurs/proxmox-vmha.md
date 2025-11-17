# VMs et HA sous Promox

## HA dans un cluster
Dans un cluster Promox avec plusieurs noeuds, la haute disponibilité des VMs permet :

- la **migration automatique** des VMs sur un autre noeud en cas de défaillance.
- de gérer la **priorisation du noeud** d'hébergement
- de gérer des **affinités entre VMs** tel que le regroupement ou la séparation de groupes
- le **démarrage automatique** de VMs en cas de redémarrage du cluster

Il est donc important de configurer correctement la haute disponibilité sur les services critiques.


## HA sur un noeud unique (cas Sport Ludique)
Sur un hyperviseur d'hébergement autnome, la haute disponibilité peut à minima permettre un d"marrage automatique de VMs en cas de redémarrage de Promox.

Dans l'interface Web, la partie *"Ressources"* du menu *"HA"* de l'objet *"Centre de données"* permet de configurer la liste des machines virtuelles concernées par de la haute dispobilité.
Pour cela il suffit de cliquer sur "Ajouter" pour ajouter toute chaque VM concernée par de la haute disponibilité :

![Ajout de la HA sur une VM](../../medias/cours/proxmox/vmha.png)

Sélectionnez simplement la VM concernée ainsi qu'état désiré :

- Started : La VM sera toujours démarrée si possible et pourra être changée de noeud
- Stopped : La VM doit être éteinte, mais pourra être changée de noeud
- Disabled : La VM doit être éteinte, et ne sera jamais changée de noeud.
- Ignored : La VM sera ignorée par défaut, les actions devront êtres manuelles

Sur un hyperviseur autonome, l'état "stopped" équivaut à l'état "Disabled". La partie "Retour en arrière" et "Nombre max de déménagements" ne seront également pas prises en compte.
