## Correction exercice release et déploiement


```
stages:
  - build
  - test
  - package
  - deploy
  - release

variables:
  IMAGE_NAME: $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_TAG

build:
  stage: build
  image: python:3.9
  script:
    - pip install -r requirements.txt
    - python -m py_compile app.py
  artifacts:
    paths:
      - app.py
    expire_in: 1 week

test:
  stage: test
  image: python:3.9
  script:
    - pip install -r requirements.txt
    - pip install pytest pytest-cov
    - pytest --cov=app tests/
  coverage: '/TOTAL.*\s+(\d+%)$/'

package:
  stage: package
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $IMAGE_NAME .
    - docker push $IMAGE_NAME
  only:
    - main
    - develop
    - tags

package-release:
  stage: package
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
  only:
    - tags

deploy-dev:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Déploiement en développement"
    - echo "Image utilisée: $IMAGE_NAME"
  environment:
    name: development
    url: https://dev.example.com
  only:
    - develop

deploy-staging:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Déploiement en staging"
    - echo "Image utilisée: $IMAGE_NAME"
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - main
  when: manual

deploy-production:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Déploiement en production"
    - echo "Image utilisée: $IMAGE_TAG"
  environment:
    name: production
    url: https://production.example.com
  only:
    - tags
  when: manual

create-release:
  stage: release
  image: registry.gitlab.com/gitlab-org/release-cli:latest
  script:
    - echo "Création de la release $CI_COMMIT_TAG"
  release:
    tag_name: '$CI_COMMIT_TAG'
    name: 'Release $CI_COMMIT_TAG'
    description: 'Release créée automatiquement pour la version $CI_COMMIT_TAG'
    assets:
      links:
        - name: 'Image Docker'
          url: '$CI_REGISTRY_IMAGE:$CI_COMMIT_TAG'
          link_type: 'image'
  only:
    - tags
```
