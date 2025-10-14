# Exercice 4 - Correction

* Commencer par forker le projet sur votre propre espace pour pouvoir faire les modifications

## Build et push du docker file


* Cela permet à votre application de builder (y-compris en local)

* Remplacer le contenu du fichier .gitlab-ci.yml par le contenu suivant : 
```yaml
stages:
  - build
  - test

variables:
  DOCKER_TLS_CERTDIR: ""
  GIT_STRATEGY: clone
  REGISTRY_USER: vanessakovalsky
  APPLICATION: demo-python-flask
  IMAGE_NAME: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG


before_script:
  - echo "Starting"
  - export DOCKER_IMAGE=$IMAGE_NAME:IMAGE_TAG
  - echo "Build docker image $DOCKER_IMAGE"

compile:
  stage: build
  image: docker:latest
  services: 
    - docker:dind
  script:
    - echo $CI_REGISTRY_PASSWORD | docker login -u $REGISTRY_USER --password-stdin
    - echo Building $DOCKER_IMAGE
    - docker build -t $DOCKER_IMAGE .
    - echo Deploying $DOCKER_IMAGE
    - docker push $DOCKER_IMAGE

test: 
  stage: test
  image: docker:latest
  services: 
    - docker:dind
  script:
    - docker run $DOCKER_IMAGE
```

* Vous pouvez maintenant lancer un pipeline cela devrait fonctionner et l'image se déployer sur le registre gitlab

## Ajout des test et de l'analyse qualité

* Ajouter à votre pipeline :
```yaml
pytest:
  stage: test
  image: $LATEST_IMAGE
  script:
    - pip install pytest pytest-cov
    - pytest tests --cov --cov-report term --cov-report html
  artifacts:
    paths:
      - htmlcov

flake8:
  stage: staticAnalysis
  image: $LATEST_IMAGE
  script:
    - pip install flake8
    - flake8 --max-line-length=120 web/*.py

mypy:
  stage: staticAnalysis
  image: $LATEST_IMAGE
  script:
    - pip install mypy
    - python3 -m mypy web/*.py

pylint:
  stage: staticAnalysis
  image: $LATEST_IMAGE
  allow_failure: true
  script:
    - pip install pylint
    - pylint -d C0301 web/*.py

pages:
  stage: publish
  script:
    - mkdir .public
    - cp -r htmlcov/* .public
    - mv .public public
  artifacts:
    paths:
      - public
```
* N'oubliez pas de rajouter les stage correspondants au début du fichier
* L'image $LATEST_IMAGE correspond à la variable définit juste avant pour le build et le push de l'image docker

## Corrigé complet

* Le corrigé complet est disponible ici: https://gitlab.com/vanessakovalsky1/cicd-flask-corrige
