# Exercice 1 - Prise en main d'un dépôt Git sur Gitlab

## Objectifs

Cet exercice a pour objectifs : 
* de créer un premier dépôt sur Gitlab
* de découvrir l'utilisation de Git

## Pré-requis : 

* Avoir une clé SSH sur sa machine : https://docs.gitlab.com/ee/user/ssh.html#generate-an-ssh-key-pair
* Avoir installé l'utilitaire git sur sa machine : https://git-scm.com/book/fr/v2/D%C3%A9marrage-rapide-Installation-de-Git 

## Créer un compte et un dépôt sur Gitlab

* Aller sur gitlab.com 
* Se créer un compte (vous pouvez utiliser un compte que vous posséder déjà ailleurs comme un compte google ou github par exemple)
* Aller dans les paramètres de son compte, puis ajouter une clé publique SSH, elle sera utile pour travailler sur les différents projets de notre compte : [https://docs.gitlab.com/ee/user/ssh.html#add-an-ssh-key-to-your-gitlab-account](https://docs.gitlab.com/ee/user/ssh.html#add-an-ssh-key-to-your-gitlab-account)
* Créer un projet sur son compte Gitlab : https://docs.gitlab.com/ee/user/project/working_with_projects.html#create-a-project 

## Authentifier son compte Github/Gitlab depuis un dépôt local :

* Soit vous passez par la génération d'une clé ssh sur la machine sur laquelle vous travailler, puis dans votre compte sur le dépôt git distant il faut déposer la clé SSH publiqué, et ensuite vous pouvez cloner votre projet en SSH et vous pouvez travailler dessus 
* Soit vous voulez rester en HTTPS (et donc travailler sans clé SSH) et dans ce cas là, il faut générer un jeton d'accès temporaire qui va remplacer votre mot de passe pour vous connecter et faire vos actions sur votre dépôt distant. Dans Github > Votre compte > Paramètres > Developper settings > Personnal access token ; Sur gitlab votre compte > Paramètre utilisateur > Access Token
* Pensez également à définir dans votre projet local les paramètre user.username et user.email (/!\ email et pas user.mail) 

## Récupération du projet en local et travail avec les commandes Git

* En local sur votre machine cloner le projet avec votre clé SSH (voir les instructions et commande sur la page de votre projet directement)
* Créer (ou modifier s'il existe selon les options que vous avez choisi à la création de votre projet) un fichier Readme
* Ecrire dans ce fichier pourquoi vous suivez cette formation
* Enregistrer votre travail en local et le tracer dans git  - avec un commit
* Envoyer votre travail sur le serveur en utilisant la commande push 
* Vérifier sur le site de gitlab que votre travail apparait bien.
* Editer sur le site de gitlab.com le même fichier et enregistrer le (cela va créer un commit directement)
* En local, récupérer le travail qui a été fait sur le serveur avec un pull.


## A l'assaut des commandes de Git : 

* Afin de connaitre les commandes disponibles sur Git, voici un outil à installer sur votre machine
https://github.com/jlord/git-it-electron/releases
* Il manque une bibliothèque dans le package
  * Sous linux, https://2h3ph3rd.medium.com/how-to-install-libgconf-2-4-on-ubuntu-23-10-fec6bda8d5f5
  * Sous windows : 
* Une fois installé, exécuter le et suivre les différents exercices qui vous expliqueront les différentes commandes de git qui existe ainsi que leur contexte d'utilisation.
