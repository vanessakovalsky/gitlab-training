# Gérer les relases les registry et les environnement



## Créer un paquet logiciel et le déployer dans le registre gitlab

* Ajouter une étape à votre pipeline qui va créer un paquet logiciel (contenant le code python sous forme d'archive zip)
* Et stocker ce paquet dans le registre de paquet de gitlab

## Création d'une release

* Ajouter dans votre pipeline une étape qui va créer une release et son tag lors de la publication ou de la fusion sur votre branche par défaut
* Exécuter votre pipeline et vérifier ce que contient votre release

## Créer et gérer les environnements

* Créer trois environnements : dev, test et prod
* Créer une variable d'environnement avec pour clé "environmentname" et pour valeur le nom de l'environnement dans votre projet
* Ajouter à votre pipeline trois étapes de déploiement correspondant aux trois environnements, et afficher la valeur de variable correspondant à l'environnement.
