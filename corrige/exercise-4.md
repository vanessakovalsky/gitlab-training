# Exercice 4 - Correction

* Commencer par forker le projet sur votre propre espace pour pouvoir faire les modifications

## Build et push du docker file

* Modifier la première ligne du Dockerfile et remplacer la par : 
```
FROM debian:latest
```
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
  LATEST_IMAGE: $REGISTRY_USER/$APPLICATION:latest
  RELEASE_IMAGE: $REGISTRY_USER/$APPLICATION:$CI_BUILD_REF_NAME
  DOCKER_DRIVER: overlay

before_script:
  - echo "Starting"
  - export DOCKER_IMAGE=$RELEASE_IMAGE
  - if [ "$CI_BUILD_REF_NAME" == "master" ]; then export DOCKER_IMAGE=$LATEST_IMAGE; fi
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
* Dans les variables remplacer la valeur de REGISTRY_USER par votre username sur le docker hub
* Dans Settings > CI/CD > Variables, ajouter : 
CI_REGISTRY_PASSWORD
et la valeur du mot de passe de votre compte sur docker hub

* Vous pouvez maintenant lancer un pipeline cela devrait fonctionner et l'image se déployer sur le hub docker