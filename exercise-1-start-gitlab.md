# Exercice 1 - Prise en main d'un dépôt Git sur Gitlab

## Objectifs

Cet exercice a pour objectifs : 
* de créer un premier dépôt sur Gitlab
* de découvrir l'utilisation de Git

## Pré-requis : 


- **Git** : Installé sur votre ordinateur : https://git-scm.com/book/fr/v2/D%C3%A9marrage-rapide-Installation-de-Git 
- **Éditeur de texte** : VS Code, Atom, Sublime Text, etc.
- **Compte GitLab** : Gratuit sur [gitlab.com](https://gitlab.com)
- **Terminal** : Git Bash (Windows), Terminal (Mac/Linux) 
* Avoir une clé SSH sur sa machine : https://docs.gitlab.com/ee/user/ssh.html#generate-an-ssh-key-pair

## 🎯 Défi 1 : Configuration de Git

**Objectif** : Configurer Git avec votre identité

### Instructions

1. Vérifiez que Git est installé :
```bash
git --version
```

2. Configurez votre nom d'utilisateur :
```bash
git config --global user.name "Votre Nom"
```

3. Configurez votre email :
```bash
git config --global user.email "votre.email@example.com"
```

4. Vérifiez votre configuration :
```bash
git config --list
```

### ✅ Vérification
Vous devriez voir votre nom et email dans la liste.

---

## 🎯 Défi 2 : Créer un dépôt local

**Objectif** : Initialiser un nouveau dépôt Git

### Instructions

1. Créez un nouveau dossier :
```bash
mkdir mon-projet-gitlab
cd mon-projet-gitlab
```

2. Initialisez Git :
```bash
git init
```

3. Créez un fichier README :
```bash
echo "# Mon Premier Projet GitLab" > README.md
```

4. Vérifiez le statut :
```bash
git status
```

### ✅ Vérification
Vous devriez voir README.md dans les fichiers non suivis (untracked).

---

## 🎯 Défi 3 : Commit vos changements

**Objectif** : Sauvegarder vos modifications

### Instructions

1. Ajoutez le fichier à l'index :
```bash
git add README.md
```

2. Créez votre premier commit :
```bash
git commit -m "Premier commit: ajout du README"
```

3. Vérifiez l'historique :
```bash
git log
```

4. Vérifiez le statut :
```bash
git status
```

### ✅ Vérification
Vous devriez voir "working tree clean" et votre commit dans l'historique.

---

## 🎯 Défi 4 : Créer un dépôt GitLab

**Objectif** : Créer un dépôt distant sur GitLab

### Instructions

1. Connectez-vous à [gitlab.com](https://gitlab.com)

2. Cliquez sur **"New project"** ou **"Nouveau projet"**

3. Choisissez **"Create blank project"**

4. Remplissez les informations :
   - Project name: `mon-projet-gitlab`
   - Visibility: Public ou Private
   - Décochez "Initialize repository with a README"

5. Cliquez sur **"Create project"**

6. Copiez l'URL de votre dépôt (HTTPS ou SSH)

### ✅ Vérification
Vous avez maintenant un dépôt vide sur GitLab avec son URL.

---

## 🎯 Défi 5 : Remote control (Dépôt distant)

**Objectif** : Connecter votre dépôt local à GitLab

### Instructions

1. Ajoutez le dépôt distant :
```bash
git remote add origin https://gitlab.com/VotreUsername/mon-projet-gitlab.git
```

OU avec SSH :
```bash
git remote add origin git@gitlab.com:VotreUsername/mon-projet-gitlab.git
```

2. Vérifiez le remote :
```bash
git remote -v
```

3. Renommez la branche principale (si nécessaire) :
```bash
git branch -M main
```

### ✅ Vérification
Vous devriez voir "origin" avec l'URL de votre dépôt GitLab.

---

## 🎯 Défi 6 : Push vers GitLab

**Objectif** : Envoyer vos commits vers GitLab

### Instructions

1. Poussez votre code vers GitLab :
```bash
git push -u origin main
```

2. Si c'est votre premier push avec HTTPS, entrez vos identifiants GitLab

3. Rafraîchissez la page de votre projet sur GitLab

### ✅ Vérification
Votre README.md devrait maintenant être visible sur GitLab !

---

## 🎯 Défi 7 : Forker un projet

**Objectif** : Créer une copie d'un projet existant

### Instructions

1. Visitez un projet public sur GitLab (ex: https://gitlab.com/gitlab-org/gitlab-foss)

2. Cliquez sur le bouton **"Fork"** en haut à droite

3. Sélectionnez votre namespace (votre compte utilisateur)

4. Attendez que le fork soit créé

5. Clonez VOTRE fork :
```bash
git clone https://gitlab.com/VotreUsername/nom-du-projet.git
cd nom-du-projet
```

### ✅ Vérification
Vous avez maintenant une copie du projet dans votre compte GitLab.

---

## 🎯 Défi 8 : Créer une branche

**Objectif** : Travailler sur une nouvelle fonctionnalité

### Instructions

1. Créez et basculez sur une nouvelle branche :
```bash
git checkout -b ajout-fonctionnalite
```

OU (méthode moderne) :
```bash
git switch -c ajout-fonctionnalite
```

2. Modifiez ou créez un fichier :
```bash
echo "Nouvelle fonctionnalité" > feature.txt
```

3. Commitez vos changements :
```bash
git add feature.txt
git commit -m "Ajout d'une nouvelle fonctionnalité"
```

4. Listez vos branches :
```bash
git branch
```

### ✅ Vérification
Vous devriez voir deux branches : main et ajout-fonctionnalite (avec * sur la branche active).

---

## 🎯 Défi 9 : Merge Request (Pull Request)

**Objectif** : Proposer vos modifications

### Instructions

1. Poussez votre branche vers GitLab :
```bash
git push origin ajout-fonctionnalite
```

2. Allez sur GitLab, vous verrez un message "Create merge request"

3. Cliquez sur **"Create merge request"**

4. Remplissez :
   - Title: Description courte
   - Description: Expliquez vos changements
   - Assignee: Vous-même ou un collaborateur
   - Labels: Optionnel

5. Cliquez sur **"Create merge request"**

### ✅ Vérification
Votre Merge Request est créée et visible dans l'onglet "Merge requests".

---

## 🎯 Défi 10 : Merge (Fusion)

**Objectif** : Fusionner votre branche

### Instructions

1. Sur GitLab, ouvrez votre Merge Request

2. Vérifiez les changements dans l'onglet "Changes"

3. Si tout est bon, cliquez sur **"Merge"**

4. Supprimez la branche source si proposé

5. Localement, revenez sur main et mettez à jour :
```bash
git checkout main
git pull origin main
```

6. Supprimez votre branche locale :
```bash
git branch -d ajout-fonctionnalite
```

### ✅ Vérification
Votre fonctionnalité est maintenant dans la branche main !

---

## 🎯 Défi 11 : Collaborer (Issues)

**Objectif** : Utiliser les Issues GitLab

### Instructions

1. Sur GitLab, allez dans **"Issues"** > **"New issue"**

2. Créez une issue :
   - Title: "Améliorer la documentation"
   - Description: Expliquez ce qui doit être fait
   - Assignez à vous-même
   - Ajoutez un label "documentation"

3. Commentez votre propre issue

4. Créez une branche à partir de l'issue (bouton "Create merge request")

5. Fermez l'issue en créant un commit :
```bash
git commit -m "Amélioration de la doc - Closes #1"
```

### ✅ Vérification
L'issue se ferme automatiquement quand votre commit est merged dans main.

---

## 🎯 Défi 12 : Contribuer à un projet

**Objectif** : Contribuer à un vrai projet

### Instructions

1. Demander à un autre stagiaire, l'adresse de son projet

2. Forkez le projet

3. Clonez votre fork :
```bash
git clone https://gitlab.com/VotreUsername/projet.git
cd projet
```

4. Créez une branche :
```bash
git checkout -b fix-issue-123
```

5. Faites vos modifications

6. Commitez et poussez :
```bash
git add .
git commit -m "Fix: correction du bug #123"
git push origin fix-issue-123
```

7. Créez une Merge Request vers le projet original

### ✅ Vérification
Vous avez contribué à un projet! 🎉


## 📚 Ressources supplémentaires

- [Documentation Git officielle](https://git-scm.com/doc)
- [Documentation GitLab](https://docs.gitlab.com/)
- [GitLab Learn](https://about.gitlab.com/learn/)
- [Markdown Guide](https://about.gitlab.com/handbook/markdown-guide/)

## 🏆 Félicitations !

Vous maîtrisez maintenant les bases de Git et GitLab ! Continuez à pratiquer en contribuant à des projets open source.


## 🔑 Commandes Git essentielles

```bash
# Configuration
git config --global user.name "Nom"
git config --global user.email "email"

# Initialisation
git init
git clone <url>

# Changements
git status
git add <fichier>
git commit -m "message"
git push
git pull

# Branches
git branch
git checkout -b <branche>
git merge <branche>
git branch -d <branche>

# Remote
git remote add origin <url>
git remote -v

# Historique
git log
git log --oneline
git diff
```
