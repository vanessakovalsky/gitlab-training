# Exercice 4 - Développer notre pipeline

## Objectifs
Cet exercice a pour objectifs : 
* de mettre en place un pipeline sur Python
* d'enrichir ce pipeline en créant une image docker et en déployant notre application dans Kubernetes


## Mise en place du pipeline Python
* Forker le projet qui est ici sur votre propre utilisateur de gitlab : https://gitlab.com/huseinzol05/cicd-flask/-/tree/master 
* Configurer le runner pour le faire pointer sur celui que vous avez installer à l'exercice 3
* Executer un pipeline pour vérifier qu'il fonctionne.
* Que fait le pipeline proposé ?


## Etendre le pipeline
* En vous inspriant de ce qui est présenté ici, rajouter les étapes nécessaires au pipelines pour builder et pousser une image docker sur le hub docker : https://tsi-ccdoc.readthedocs.io/en/master/ResOps/2019/gitlab/05_add-further-steps.html 
* Ajotuer une etape pour vérifier le respect des bonnes pratiques de développement sur le code python avec Pylint, myPy et Flake8 et l'execution des tests unitaires de manières séparée : https://www.patricksoftwareblog.com/setting-up-gitlab-ci-for-a-python-application/
* Déployer dans un cluster Kubernetes avec un Chart Helm : https://www.padok.fr/en/blog/kubernetes-gitlab-pipeline 

## Pour aller plus loin : 

* un exemple complet d'un pipeline pour une app avec Flask, Docker et un cluster Kubernetes dans le cloud AWS : https://aws.plainenglish.io/ci-cd-with-gitlab-kubernetes-eks-and-helm-a3667b0fb8ed 