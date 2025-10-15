# TP - Debug de Pipeline CI/CD GitLab

## 🎯 Objectif

Identifier et corriger les erreurs dans des pipelines GitLab CI/CD cassés. Chaque scénario représente un problème réel rencontré en production.

**Durée estimée : 60 minutes**

---

## 🔧 Scénario 1 : Le Pipeline qui ne démarre pas (10 min)

### Contexte
Vous venez de récupérer ce pipeline d'un collègue, mais GitLab refuse de le lancer et affiche : **"CI configuration is invalid"**

```yaml
stages:
  - build
  - test
  - deploy

variables:
  DOCKER_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  
build-app:
  stage: build
  image: python:3.9
  script:
    - pip install -r requirements.txt
    - python -m py_compile app.py
  artifacts:
    paths:
      - app.py
    expire_in: 1 day

test-app
  stage: test
  image: python:3.9
  script:
    - pip install -r requirements.txt
    - pip install pytest
    - pytest tests/
  dependencies:
    - build-app

deploy-staging:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying to staging..."
    - ./deploy.sh staging
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - master
  when: manual
```

### 🕵️ Mission
1. **Trouvez les 3 erreurs de syntaxe** qui empêchent le pipeline de démarrer
2. **Corrigez le fichier** `.gitlab-ci.yml`
3. **Expliquez** chaque erreur trouvée

### 💡 Indices
- Vérifiez les deux-points (`:`)
- Regardez attentivement la structure YAML
- GitLab CI/CD est sensible à l'indentation

---

## 🐛 Scénario 2 : Le Job qui échoue mystérieusement (15 min)

### Contexte
Votre pipeline démarre mais le job `build-docker` échoue systématiquement avec l'erreur :
```
ERROR: Cannot connect to the Docker daemon. Is the docker daemon running?
```

```yaml
stages:
  - build
  - package

build-app:
  stage: build
  image: python:3.9
  script:
    - pip install -r requirements.txt
    - python setup.py build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week

build-docker:
  stage: package
  image: docker:latest
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  dependencies:
    - build-app
  only:
    - main
```

### 🕵️ Mission
1. **Identifiez le problème** qui empêche Docker de fonctionner
2. **Corrigez le job** pour qu'il puisse exécuter des commandes Docker
3. **Optimisez** : réorganisez les commandes dans le bon ordre
4. **Bonus** : Ajoutez une gestion d'erreur si le build échoue

### 💡 Indices
- Docker-in-Docker nécessite un service spécial
- L'ordre des opérations Docker est important (login avant push !)

---

## 🔄 Scénario 3 : Les dépendances circulaires (10 min)

### Contexte
Le pipeline se bloque indéfiniment et GitLab affiche : **"Jobs are waiting for dependent jobs to complete"**

```yaml
stages:
  - prepare
  - build
  - test
  - package

install-deps:
  stage: prepare
  image: python:3.9
  script:
    - pip install -r requirements.txt --target ./libs
  artifacts:
    paths:
      - libs/
  needs:
    - lint-code

build-app:
  stage: build
  image: python:3.9
  script:
    - export PYTHONPATH=$PYTHONPATH:./libs
    - python -m py_compile app.py
  needs:
    - install-deps
    - unit-tests

unit-tests:
  stage: test
  image: python:3.9
  script:
    - export PYTHONPATH=$PYTHONPATH:./libs
    - pytest tests/unit/
  needs:
    - install-deps
    - build-app

lint-code:
  stage: test
  image: python:3.9
  script:
    - pip install flake8
    - flake8 .
  needs:
    - build-app

package-app:
  stage: package
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t myapp .
  needs:
    - build-app
```

### 🕵️ Mission
1. **Dessinez le graphe des dépendances** entre les jobs
2. **Identifiez le cycle** qui bloque le pipeline
3. **Corrigez les dépendances** pour créer un flux logique
4. **Expliquez** l'ordre d'exécution optimal

### 💡 Indices
- Un job ne peut pas dépendre d'un job qui dépend de lui (directement ou indirectement)
- Les jobs d'un même stage ne peuvent pas se dépendre mutuellement avec `needs`

---

## 💾 Scénario 4 : Les artifacts fantômes (10 min)

### Contexte
Votre job `deploy` échoue avec : **"No files to upload"** ou ne trouve pas les fichiers buildés.

```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  image: node:16
  script:
    - npm install
    - npm run build
    - ls -la dist/
  artifacts:
    paths:
      - build/
    expire_in: 1 hour

test:
  stage: test
  image: node:16
  script:
    - npm install
    - npm test
  dependencies: []

deploy:
  stage: deploy
  image: alpine:latest
  script:
    - ls -la
    - echo "Fichiers disponibles:"
    - find . -type f
    - cp -r dist/* /var/www/html/
  dependencies:
    - build
  only:
    - main
```

### 🕵️ Mission
1. **Identifiez l'erreur** entre le chemin des artifacts et leur utilisation
2. **Corrigez** pour que les fichiers soient disponibles dans le job `deploy`
3. **Vérifiez** que les chemins sont cohérents
4. **Ajoutez** une vérification pour s'assurer que les fichiers existent avant le déploiement

### 💡 Indices
- Le chemin dans `artifacts.paths` doit correspondre au chemin créé par le build
- La commande `npm run build` crée souvent un dossier `dist/` ou `build/`

---

## 🔐 Scénario 5 : Les variables manquantes (15 min)

### Contexte
Le job échoue avec des erreurs mystérieuses liées aux credentials et à la connexion.

```yaml
stages:
  - deploy

deploy-production:
  stage: deploy
  image: alpine:latest
  script:
    - apk add --no-cache curl
    - echo "Deploying to $DEPLOY_SERVER"
    - curl -X POST https://$DEPLOY_SERVER/api/deploy \
        -H "Authorization: Bearer $API_TOKEN" \
        -d "version=$CI_COMMIT_SHA"
    - ssh $SSH_USER@$DEPLOY_SERVER "docker pull $DOCKER_IMAGE"
    - ssh $SSH_USER@$DEPLOY_SERVER "docker run -d $DOCKER_IMAGE"
  environment:
    name: production
    url: https://$PRODUCTION_URL
  only:
    - tags
```

**Logs d'erreur :**
```
Deploying to
curl: (6) Could not resolve host: 
ssh: Could not resolve hostname : Name or service not known
```

### 🕵️ Mission
1. **Listez toutes les variables** utilisées mais non définies
2. **Ajoutez la section `variables`** avec des valeurs de démonstration
3. **Identifiez** quelles variables devraient être des **secrets GitLab CI/CD**
4. **Documentez** comment configurer ces secrets dans GitLab
5. **Bonus** : Ajoutez des vérifications pour s'assurer que les variables requises existent

### 💡 Indices
- Les variables sensibles (tokens, passwords) ne doivent JAMAIS être dans le `.gitlab-ci.yml`
- Utilisez `Settings > CI/CD > Variables` dans GitLab
- Vous pouvez vérifier l'existence d'une variable avec `${VAR:?Variable VAR is required}`

---

## ⚡ Scénario 6 : Le pipeline trop lent (Bonus, 10 min)

### Contexte
Votre pipeline fonctionne mais prend 25 minutes alors qu'il pourrait être beaucoup plus rapide !

```yaml
stages:
  - test

unit-tests:
  stage: test
  image: python:3.9
  script:
    - pip install -r requirements.txt
    - pytest tests/unit/ -v

integration-tests:
  stage: test
  image: python:3.9
  script:
    - pip install -r requirements.txt
    - pytest tests/integration/ -v

e2e-tests:
  stage: test
  image: python:3.9
  script:
    - pip install -r requirements.txt
    - pytest tests/e2e/ -v

security-tests:
  stage: test
  image: python:3.9
  script:
    - pip install -r requirements.txt
    - pip install bandit safety
    - bandit -r .
    - safety check

lint:
  stage: test
  image: python:3.9
  script:
    - pip install -r requirements.txt
    - pip install flake8 black
    - flake8 .
    - black --check .
```

### 🕵️ Mission
1. **Identifiez les optimisations** possibles (au moins 4)
2. **Réorganisez le pipeline** pour gagner du temps
3. **Ajoutez du cache** pour les dépendances
4. **Créez un stage `prepare`** pour mutualiser les installations
5. **Calculez** le temps gagné estimé

### 💡 Pistes d'optimisation
- Cache des dépendances pip
- Parallélisation des jobs
- Artifacts pour partager les installations
- Images Docker pré-configurées
- Éliminer les installations redondantes

---

## 📊 Grille d'évaluation

| Scénario | Difficulté | Points | Compétences testées |
|----------|-----------|--------|---------------------|
| Scénario 1 | ⭐ Facile | 10 pts | Syntaxe YAML, structure de base |
| Scénario 2 | ⭐⭐ Moyen | 15 pts | Services Docker, ordre des opérations |
| Scénario 3 | ⭐⭐⭐ Difficile | 20 pts | Dépendances, graphe d'exécution |
| Scénario 4 | ⭐⭐ Moyen | 15 pts | Artifacts, chemins de fichiers |
| Scénario 5 | ⭐⭐ Moyen | 20 pts | Variables, secrets, sécurité |
| Scénario 6 | ⭐⭐⭐ Difficile | 20 pts | Optimisation, performance |
| **Total** | | **100 pts** | |

---

## 📝 Livrables attendus

Pour chaque scénario, fournissez :
1. ✅ **Le fichier `.gitlab-ci.yml` corrigé**
2. ✅ **Une explication des erreurs trouvées**
3. ✅ **Les tests de validation** (commandes pour vérifier que ça fonctionne)
4. ✅ **Les leçons apprises** (qu'avez-vous compris ?)

---

## 🎓 Méthodologie de debug

Pour chaque scénario, suivez cette méthode :

1. **Lire les logs** attentivement (cherchez "ERROR", "FATAL", "failed")
2. **Vérifier la syntaxe** YAML (indentation, deux-points, tirets)
3. **Valider les dépendances** entre jobs (graphe logique)
4. **Contrôler les chemins** de fichiers et artifacts
5. **Vérifier les variables** et leur disponibilité
6. **Tester localement** si possible avec `gitlab-runner exec`

---

## 🔗 Ressources utiles

- [GitLab CI/CD YAML syntax reference](https://docs.gitlab.com/ee/ci/yaml/)
- [Validateur YAML en ligne](https://www.yamllint.com/)
- [GitLab CI/CD troubleshooting](https://docs.gitlab.com/ee/ci/troubleshooting.html)
- [Docker-in-Docker](https://docs.gitlab.com/ee/ci/docker/using_docker_build.html)

---

**Bon debug ! 🐞🔧**
