# Exercice  - Mise en place d'un premier runner 

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
* Dans la partie `Specific runners`, aller sur `New runner` et suivre les étapes demandé par Gitlab

* Entrer si vous le souhaitez une description et un tag
* Lorsqu'on vous demande l'`executor` entrer `docker`
* Pour le choix de l'image utiliser : `alpine:latest`


## Utilisez son runner dans un pipeline

* Editer un fichier de pipeline d'un précédent projet
* Au niveau d'un job ajouter le mot clé `tags` avec le tag choisi exemple :
```
  tags:
    - tagquiexistepas
```
* Exécuter votre pipeline et vérifier qu'il s'est bien exécuter sur votre runner
