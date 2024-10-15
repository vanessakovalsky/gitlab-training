# Exercice 4 - Développer notre pipeline

## Objectifs
Cet exercice a pour objectifs : 
* de mettre en place un pipeline sur Python
* d'enrichir ce pipeline en créant une image docker et en déployant notre application dans Kubernetes


## Mise en place du pipeline Python
* Forker le projet qui est ici sur votre propre utilisateur de gitlab : [https://gitlab.com/vanessakovalsky1/cicd-flask](https://gitlab.com/vanessakovalsky1/cicd-flask)
* Executer un pipeline pour vérifier qu'il fonctionne.
* Que fait le pipeline proposé ?


## Etendre le pipeline en utilisant docker 
* En vous inspriant de ce qui est présenté ici  https://tsi-ccdoc.readthedocs.io/en/master/ResOps/2019/gitlab/05_add-further-steps.html,
* rajouter les étapes nécessaires au pipelines pour builder et pousser une image docker sur le hub docker :

## Ajouter des étapes aux pipelines
* Ajouter des étape pour vérifier le respect des bonnes pratiques de développement sur le code python avec Pylint, myPy et Flake8 et l'execution des tests unitaires de manières séparée
* Pour chacune de ces étapes, vous utiliserez l'image docker construite et stockée à l'étape précédente
* Pour chacune des étapes vous trouverez ci-dessous les commandes à éxecuter
* * Pour l'execution des tests avec pytest --> artefact à récupérer dans le dossier htmlcov (paths)
  ```
  pip3 install pytest pytest-cov
  pytest tests --cov --cov-report term --cov-report html
  ```
* * Pour l'execution de l'analyse statique avec Flake8 
  ```
  pip3 install flake8
  flake8 --max-line-length=120 web/*.py
  ```
* * Pour l'execution de l'analyse statique avec myPy 
  ```
  pip3 install mypy
  python3 -m mypy web/*.py
  ```
* * Pour l'execution de l'analyse avec pylint 
  ```
  pip3 install pylint
  pylint -d C0301 web/*.py
  ```
* * Puis nous ajoutons une étapes qui permet de publier les rapport avec les commandes suivantes, le contenu du dossier public est à extraire en artefacts
  ```
  mkdir .public
  cp -r htmlcov/* .public
  mv .public public
  ```

## Pour aller plus loin : 

* un exemple complet d'un pipeline pour une app avec Flask, Docker et un cluster Kubernetes dans le cloud AWS : https://aws.plainenglish.io/ci-cd-with-gitlab-kubernetes-eks-and-helm-a3667b0fb8ed
  Déployer dans un cluster Kubernetes avec un Chart Helm : https://www.padok.fr/en/blog/kubernetes-gitlab-pipeline 


