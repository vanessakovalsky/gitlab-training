# Exercice 3 - Mise en place d'un premier pipeline 

## Objectifs

Cet exercice a pour objectifs :
* de créer un premier pipeline et d'observer son fonctionnement
* d'utiliser la CI/CD pour déployer un site statique


## Création du fichier de pipeline et des étapes

* Revenir sur le projet sur Gitlab.com
* Créer un fichier à la racine du projet avec le nom : .gitlab-ci.yml 
* Sélectionner .gitlab-ci.yml pour le type de template et appliquer le template Bash. Cela va pré-remplir le fichier.
* Pour créer un fichier minimal : 
    * Supprimer toutes les lignes avant build 1 (donc les lignes 1 à 15)
    * Supprimer toutes les lignes après `echo "For example run a test suite"` dans la section test1
* Ajouter les étapes `build` et `test` en collant les lignes ci-dessous tout en haut du fichier. 
```yaml
    stages:
      - build 
      - test
```

* Pour visualiser et lancer le pipeline, cliquer dans le menu sur `CI/CD pipelines`
* Cliquer sur le pipeline de votre projet pour voir le statut de votre pipeline.
* Cliquer sur chaque étape et chercher quel runner a executer l'étape.

## Publication d'un site statique

* Un site statique peut être déployer via un générateur de site ou du code html directement par un pipeline Gitlab.
* Pour cela on choisit un générateur et on demande à gitlab-ci de générer nos pages.
* Exemple ici avec Pelican un générateur en Python : https://gitlab.com/pages/pelican
* Forker le projet : Pour forker un projet sur Gitlab, afficher la page d'accueil du projet (pas la page du dépôt) et cliquer sur Fork ou "Créer une bifurcation"
* Aller dans Build > Pipelines et exécuter une première fois le pipeline pour créer le déploiement à partir des fichiers d'origine.
* Ensuite, récupérer l'URL de votre propre page dans le menu Déploiement > Pages
* Si vous faite une modification du fichier dans content/posts le pipeline se déclenchera de nouveau et votre modification sera publiée sur la même url
* Vous avez automatisé la publication d'un site statique à partir du Pipeline et du générateur de site

## Pour aller plus loin

* En fonction de vos besoins / projets, choisir un exemple sur cette page : https://docs.gitlab.com/ee/ci/examples/
* Mettre en oeuvre le pipeline correspondant
