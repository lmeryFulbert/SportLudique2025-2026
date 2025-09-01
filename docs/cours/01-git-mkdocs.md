# 01-Utilisation de git et de mkdocs

## Prérequis
Avant de commencer, assurez-vous d'avoir les éléments suivants installés sur votre machine :
- Git (`sudo apt-get install git`)
- Python et pip (`sudo apt-get install python3-pip`)
- MkDocs (`pip install mkdocs`)

## 1. Création d'un dépôt GitHub public

1. Rendez-vous sur [GitHub](https://github.com) et connectez-vous à votre compte.
2. Cliquez sur le bouton **New repository**.
3. Entrez un nom pour votre dépôt (ex: `mon-projet-mkdocs`).
4. Sélectionnez l'option **Public** pour rendre le dépôt accessible à tous.
5. Cochez l'option **Add a README file** pour initier votre dépôt avec un fichier README.
6. Cliquez sur **Create repository**.

## 2. Cloner le dépôt sur Linux Mint

1. Ouvrez un terminal.
2. Naviguez vers le répertoire où vous souhaitez cloner votre dépôt (ex: `cd ~/MesDocumentations`).
3. Copiez l'URL de votre dépôt depuis GitHub (bouton vert "Code" > HTTPS).
4. Tapez la commande suivante pour cloner le dépôt sur votre machine :

    ```bash
    git clone https://github.com/votre-nom-utilisateur/mon-projet-mkdocs.git
    ```

5. Naviguez dans le répertoire cloné :

    ```bash
    cd mon-projet-mkdocs
    ```

## 2.bis Configuration du compte GitHub pour le dépôt local

Avant de commencer à travailler avec Git et GitHub sur votre dépôt local, vous devez configurer votre compte GitHub pour que vos commits soient associés à votre profil.

### 1. Configurer votre nom d'utilisateur et votre adresse e-mail

1. **Ouvrez un terminal** sur votre machine.
2. **Définissez votre nom d'utilisateur** :

    ```bash
    git config --global user.name "Votre Nom d'utilisateur GitHub"
    ```

3. **Définissez votre adresse e-mail** (cette adresse doit être la même que celle utilisée sur GitHub) :

    ```bash
    git config --global user.email "votre-email@example.com"
    ```

### 2. Vérifier la configuration

1. Pour vérifier que votre configuration a bien été prise en compte, utilisez la commande suivante :

    ```bash
    git config --global --list
    ```

    Vous devriez voir votre nom d'utilisateur et votre adresse e-mail listés.

### 3. Configurer l'authentification par clé SSH 

Sous windows (Client graphique TortoiseGit), il est possible de s'authentifier avec un login/password.
github n'accepte plus les push avec la methode http de puis ?, sous linux et avec le terminal il faut forcement s'authentifier avec une clé SSH
Pour faciliter l'authentification et éviter de saisir vos identifiants à chaque push, vous pouvez configurer une clé SSH :

1. **Générer une clé SSH** :

    === "mkdocs"

    ```bash
    ssh-keygen -t ed25519 -C "votre-email@example.com"
    ```

    === "Docusaurus "

    ```bash
    ssh-keygen rsa -C "votre-email@example.com"
    ```

    Appuyez sur `Entrée` pour accepter l'emplacement par défaut du fichier. Vous pouvez aussi définir une phrase de passe pour sécuriser votre clé.

    ??? Warning "Attention"
    **Attention** ne perdez pas le mot de passe associé à votré clé privée.

2. **Ajouter la clé SSH à votre compte GitHub** :
   - Copiez la clé publique générée :

    ```bash
    cat ~/.ssh/id_ed25519.pub
    ```

   - Connectez-vous à GitHub, allez dans **Settings** > **SSH and GPG keys**, puis cliquez sur **New SSH key**. Collez la clé dans le champ prévu à cet effet et donnez-lui un nom.

3. **Tester la connexion** :

    ```bash
    ssh -T git@github.com
    ```

    Si tout est bien configuré, vous recevrez un message de confirmation de la connexion.

---

## 3. Initialisation de MkDocs

1. Dans votre terminal, assurez-vous d'être dans le répertoire de votre projet :

    ```bash
    cd ~/MesDocumentations/mon-projet-mkdocs
    ```

2. Créez une structure de base pour MkDocs :

    ```bash
    mkdocs new .
    ```

3. Cette commande crée un répertoire `docs/` et un fichier `mkdocs.yml`.

## 4. Arborescence de fichiers attendue pour un projet MkDocs

Une structure de fichiers de base pour un projet MkDocs devrait ressembler à ceci :

    ```plaintext
        mon-projet-mkdocs/
        ├── .git/
        ├── .vscode/
        │   └── settings.json
        ├── docs/
        │   ├── index.md
        │   └── autres_pages.md
        ├── mkdocs.yml
        └── README.md
    ```
### Details des repertoires

-    **.git/** : Répertoire caché contenant les fichiers de configuration et l'historique des versions Git.
-    **.vscode/** : Répertoire caché pour les configurations spécifiques de l'éditeur (comme Codium), avec des fichiers comme settings.json.
-    **docs/** : Contient les fichiers Markdown pour la documentation.
-    **mkdocs.yml** : Fichier de configuration pour MkDocs.
-    **README.md** : Fichier de présentation du projet.

## 5. Faire un commit local

1. Après avoir modifié ou ajouté des fichiers, vérifiez l'état de votre dépôt :

    ```bash
    git status
    ```

2. Ajoutez les fichiers modifiés ou ajoutés pour le commit :

    ```bash
    git add .
    ```

3. Créez un commit avec un message explicatif :

    ```bash
    git commit -m "Initialisation du projet avec MkDocs"
    ```

## 6. Envoyer les modifications sur GitHub (Push)

1. Pour envoyer vos modifications locales vers GitHub, utilisez la commande :

    ```bash
    git push origin main
    ```

    **Note** : Si votre branche principale s'appelle `master` au lieu de `main`, adaptez la commande.

## 7. Récupérer les modifications de GitHub (Pull)

1. Avant de commencer à travailler sur votre projet, récupérez les dernières modifications depuis GitHub pour éviter les conflits :

    ```bash
    git pull origin main
    ```

2. Si vous et d'autres collaborateurs avez fait des modifications en même temps, vous devrez peut-être résoudre des conflits. Git vous indiquera les fichiers en conflit et vous guidera pour les résoudre.

## 8. Bonnes pratiques pour éviter les conflits

- Tirez (avec la commande **pull**) toujours les dernières modifications **avant** de commencer à travailler.
- Commitez et poussez (commande **push**) vos changements régulièrement.
- Si vous travaillez sur une fonctionnalité ou une grosse modification, envisagez de créer une branche séparée (si vous voulez tester un autre thème par exemple)

## 9. Workflow de travail collaboratif

Afin d’assurer un développement propre et collaboratif, nous suivons un **workflow basé sur les branches et les Pull Requests**.

### 🔹 Règles principales
- Chaque collaborateur travaille sur **sa propre branche**.
- Toute modification doit être proposée via une **Pull Request (PR)**.
- Le propriétaire du projet (ou un responsable) effectue une **code review** avant toute intégration dans la branche principale (`main`).

### 🔹 Bonnes pratiques
1. Créer une nouvelle branche à partir de `main` :  
   ```bash
   git checkout main
   git pull origin main
   git checkout -b ma-branche
   ```
2. Travailler et committer régulièrement :
   ```bash
    git add .
    git commit -m "Ajout de la section sur le workflow collaboratif"
   ```
3. Pousser la branche sur le dépôt distant :
   ```bash
    git push origin ma-branche
   ```
4. Ouvrir une Pull Request sur GitHub pour proposer les changements.

### 🔹 Processus de revue

✅ Validation : si la proposition est correcte, la PR est validée et fusionnée dans main.

❌ Refus : si des corrections sont nécessaires (orthographe, clarté, cohérence…), la PR est rejetée avec des commentaires explicatifs.

🔄 Amélioration : l’auteur peut alors mettre à jour sa branche, corriger ses modifications puis soumettre une nouvelle PR.

## 10. Configuration du CI/CD avec GitHub Actions via l'interface GitHub

 ??? Warning "Définitions"
    **CI (Continuous Integration – Intégration Continue)**
L’intégration continue est une pratique qui consiste à fusionner régulièrement le code développé par chaque collaborateur dans la branche principale.
CD (**Continuous Delivery / Continuous Deployment** – Livraison ou Déploiement Continu)
**Continuous Delivery** : le code est automatiquement testé et préparé pour être mis en production. Le déploiement nécessite encore une validation manuelle.
**Continuous Deployment** : étape supplémentaire où chaque changement validé est déployé automatiquement en production sans intervention humaine.


   - Ouvrez votre dépôt sur GitHub.
   - Cliquez sur l'onglet **Actions** en haut de la page de votre dépôt.
   - GitHub vous proposera des templates pour différents workflows. Vous pouvez choisir un template existant, comme "Deploy static content to GitHub Pages", ou commencer à partir de zéro en cliquant sur **Set up a workflow yourself**.
   - Une fois dans l'éditeur, un fichier YAML nommé `main.yml` (ou un autre nom de votre choix) sera créé dans le répertoire `.github/workflows/`.
   - Éditez le fichier directement dans l'éditeur GitHub pour spécifier les étapes de votre pipeline (installation de MkDocs, génération de la documentation, déploiement).
   - Après avoir configuré le workflow, ajoutez un message de commit dans le champ prévu à cet effet sous l'éditeur.
   - Assurez-vous que l'option "Commit directly to the `main` branch" est sélectionnée, puis cliquez sur **Commit new file**.
   - Une fois le commit effectué, GitHub exécutera automatiquement le workflow si vous avez configuré un déclencheur comme `push` ou `pull_request`.
   - Vous pouvez suivre l'exécution de votre workflow dans l'onglet **Actions** pour vérifier qu'il se déroule correctement.
   - Après avoir modifié le workflow directement sur GitHub, il est important de synchroniser votre dépôt local avec le dépôt distant pour que toutes les modifications soient alignées.
   - Ouvrez votre terminal et naviguez dans votre répertoire de projet local, puis exécutez :

    ```bash
    git pull origin main
    ```
   - Cette commande téléchargera les dernières modifications du dépôt distant vers votre dépôt local.

## 10. Activer GitHub Pages avec la branche de déploiement

Une fois votre workflow CI/CD configuré pour déployer la documentation, vous pouvez activer GitHub Pages pour héberger votre site de documentation.

   - Dans votre dépôt GitHub, cliquez sur l'onglet **Settings** (Paramètres) en haut à droite.
   - Dans le menu latéral gauche, faites défiler vers le bas jusqu'à la section **Pages**.
   - Sous la section **Build and deployment**, trouvez l'option **Source**.
   - Dans le menu déroulant, sélectionnez la branche utilisée pour le déploiement. Il s'agit généralement d'une branche nommée `gh-deploy` ou d'une autre branche spécifiée dans votre workflow YAML.
   - Cliquez sur **Save** (Enregistrer) pour valider les paramètres.
   - GitHub Pages sera maintenant activé pour cette branche spécifique. Si le déploiement a été correctement configuré et exécuté, votre documentation sera disponible à l'adresse URL fournie par GitHub.
   - GitHub affichera l'URL où votre site est hébergé. Vous pouvez y accéder pour vérifier que votre documentation est correctement déployée.
---

Avec cette configuration, votre documentation sera automatiquement mise à jour et déployée sur GitHub Pages chaque fois que le workflow CI/CD est exécuté, en utilisant la branche de déploiement définie.
