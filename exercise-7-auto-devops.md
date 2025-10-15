# Exercice 7 - Découverte de l'Auto DevOps de Gitlab

## Objectifs
Cet exercice a pour objectifs :
* de créer un projet en utilisant Auto DevOps
* de déclencher le pipeline sur un commit

## Créer un nouveau projet Node JS Express avec Auto DevOps

1. Depuis le menu cliquer sur Project puis sur **Create New project**. 
2. Cliquer sur **Create from template**,puis cliquer sur **Use template** à côté de  **NodeJS Express**.
3. Dans le champs **Project name**, entrer `Auto DevOps-test`
4. Assurer vous que le  **Visiblity Level** soit bien sur **Private**, et cliquer sur **Create project**.
5. Auto DevOps est une alternative à l'écriture et l'utilisation de votre propre fichier `.gitlab-ci.yml`. Noter la bannière en haut de la fenêtre qui nous alerte que Auto DevOps est activé. GitLab a activé automatiquement cette fonctionnalité car il ne trouve pas de fichier `.gitlab-ci.yml` dans le dossier du dépôt.
6. Dans le menu de gauche, cliquer sur **CI/CD > Pipelines**. Noter que le pipeline n'a pas encore fonctionner.
7. Dans le menu de gauche, cliquer sur **Repository > Branches**. Cliquer sur **New branch**.
8. Dans le champ **Branch name**, entrer `new-feature` et cliquer sur **Create branch**.
9. Dans le menu de gauche, cliquer sur **CI/CD > Pipelines**. Vous verrez un pipeline avec le label **Auto DevOps**, qui est entrain de s'exécuter sur la branche que vous venez de créer.
10. Cliquer sur l'icone de status du  pipeline en cours d'exécution et noter les 6 étapes (representées par 6 colonnes dans le graphique de pipeline) que Auto DevOps a créé.

## Déclencher une exécution de pipeline en validant une modification

La manière la plus commune de déclencher l'exécution d'un pipeline est de le faire via un commit sur une branche du dépôt de votre projet. 

1. Dans le menu de gauche, cliquer sur **Repository > Files**.
2. En haut à gauche de l'écran, changer pour la branche **new-feature** en la selectionnnant dans la liste déroulante qui affiche actuellement **main**.
3. Dans la liste des fichiers du dépôts, cliquer sur le dossier `views` et sur le fichier `index.pug`.
4. Cliquer sur **Edit** et modifier la dernière ligne du fichier. 
*IMPORTANT: preserver l'indentation de 2 espace pour cette ligne et inclure le  `p` au début de la ligne.*
   
   ```
     p GitLab welcomes you to #{title}
   ```
   
5. Dans le champs **Commit message**, taper `Update welcome message in index.pug`
6. Laisser le champ **Target branch** définit sur `new-feature`. 
7. Cliquer sur **Commit changes**.
8. Cliquer sur **Create merge request**.
9. Assigner la requête de merge à vous-même.
10. Ajouter `Draft:` au début du texte dans le champ **Title** pour montrer que la requête de merge n'est pas encore prête à être fusionné.
11. Laisser les autres champs avec leur valeur par défaut et cliquer sur **Create merge request** en bas de la page. 
   
    Vous avez maintenant une requête de merge active pour fusionner la branche `new-feature` dans la branche `master`. La page qui vous montre les détails de la requête de merge, incluant le statut du dernier pipeline qui a été exécutée sur la branche `new-feature` (vous aurez peut être besoin de rafraichir la page pour voir le status du pipeline). GitLab exécutera un nouveeau pipeline à chaque fois que vous commiter sur la branche `new-feature`.

12. L'étape **Review** du pipeline Auto DevOps  deploie votre application NodeJS Express dans un environnement dédié à la branche. Vous pouvez voir le status de chaque pipeline en survolant l'icone circulaire du status de pipeline. Une fois le  pipeline terminé l'étape **Review**, voir l'application déployée en cliquant sur **View latest app** au milieu de la requête de merge. Vous devriez alors voir le texte que vous avez modifié.
13. Revenir à l'onglet de GitLab. Dans le menu de gauche, cliquer sur **Packages & Registries > Container Registry**. Vous devriez voir le conteneur Docker que le pipeline Auto DevOps a créé lorsqu'il a déployé l'application dans l'environnement.


## Ressources supplémentaires

* [Fichier gitlab-ci.yaml utilisé par AutoDevOps](https://gitlab.com/gitlab-org/gitlab/-/blob/master/lib/gitlab/ci/templates/Auto-DevOps.gitlab-ci.yml)
* [Personnaliser l'autodevops](https://docs.gitlab.com/topics/autodevops/customize/)
