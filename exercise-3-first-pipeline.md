# Exercice 3 - Mise en place d'un premier pipeline 

## Objectifs

Cet exercice a pour objectifs :
* d'apprendre à installer et configurer un runner
* de créer un premier pipeline et d'observer son fonctionnement
* d'utiliser la CI/CD pour déployer un site statique

## Configuration du runners

### Installation du runner 
* Nous allons installer un runner sur votre machine pour pouvoir l'utiliser.
* Sous Linux utiliser les commandes suivantes : 
```sh
## Debian/Ubuntu
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
sudo apt-get install gitlab-runner
## Red Hat / centos
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh" | sudo bash
sudo yum install gitlab-runner
```
* Pour vérifier que le runner fonctionne : 
```
sudo gitlab-runner status
```
* Si vous souhaitez installer un runner sur Windows : https://docs.gitlab.com/runner/install/windows.html#installation 
* Ou sur Mac Os : https://docs.gitlab.com/runner/install/osx.html#manual-installation-official

### Enregistrement du runners sur notre projet
* Sur l'interface de Gitlab, aller dans les paramètres du projet puis sur `CI/CD`
* Descendre jusqu'à la section `Runners` et cliquer sur `Expand` pour ouvrir cette section
* Dans la partie `Specific runners`, aller sur `Set up a specific runner manually`
* Copier l'URL en face de `Register the runner with this URL`
* En fonction de votre système utiliser les commandes suivantes : 
    * In a Linux terminal:
    ```
    sudo gitlab-runner register
    ```
    * In a macOS terminal:
    ```
    gitlab-runner register
    ```
    * In a normal (not elevated) Windows PowerShell window:
    ```
    cd C:\GitLab-Runner
    ./gitlab-runner.exe register
    ```
* Coller alors l'URL que vous avez copié à l'étape précédente
* Revenir sur la page de Gitlab et copier le `registration token` dans la même section que l'URL
* Dans le terminal, coller le registration token que vous venez de copier
* Entrer si vous le souhaitez une description et/ou un tags (ou laisser ceux par défaut en appuyant seulement sur `entrée`)
* Lorsqu'on vous demande l'`executor` entrer `shell`
* Confirmer que votre gitlab-runner fonctionne correctemen en utilisant les commande suivante (en fonction de votre OS)
    * In a Linux terminal:
    ```
    sudo gitlab-runner list
    ```
    * In a macOS terminal:
    ```
     gitlab-runner list
    ```
    * In a normal (not elevated) Windows PowerShell window:
    ```
      cd C:\GitLab-Runner
      ./gitlab-runner.exe list
    ```
* Si vous êtes sous windows suivre ces instructions supplémentaires : 
    * Ouvrir le fichier C:\GitLab-Runner\config.toml.
    * Changer la ligne:
    ```
    shell = "pwsh"
    ```
    par celle-ci : 
    ```
    shell = "powershell"
    ```
    * Enregistrer le fichier


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

* Pour visualiser et lancer le pipeline, cliqquer dans le menu sur `CI/CD pipelines`
* Cliquer sur le pipeline de votre projet pour voir le statut de votre pipeline.
* Cliquer sur chaque étape et chercher quel runner a executer l'étape.

## Publication d'un site statique

* Un site statique peut être déployer via un générateur de site ou du code html directement par un pipeline Gitlab.
* Pour cela on choisit un générateur et on demande à gitlab-ci de générer nos pages.
* Exemple ici avec Pelican un générateur en Python : https://gitlab.com/pages/pelican/-/tree/main 