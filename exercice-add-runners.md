# Exercice  - Mise en place d'un premier pipeline 

## Objectifs

Cet exercice a pour objectifs :
* d'apprendre à installer et configurer un runner

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
