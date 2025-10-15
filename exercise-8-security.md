# Exercice 6 - Intégration de la sécurité dans le pipeline CI/CD

## Objectifs

Intégrer des analyses de sécurité essentielles dans votre pipeline GitLab CI/CD pour votre application Python Flask en moins d'une heure.

## Prérequis

- Un projet GitLab avec une application Python Flask
- Un fichier `.gitlab-ci.yml` fonctionnel
- Un `Dockerfile` pour containeriser l'application

## Durée estimée : 60 minutes

## Partie 0 : préparation du dépôt

* Reprenez votre dépôt avec l'application Python.
* Ajouter à la racine un fichier nommé requirements.txt avec le contenu suivant :
```
flask
gunicorn
```
* Puis remplacer le contenu  du fichier Dockerfile avec les lignes suivantes :
```
FROM python:3
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt

ENV LC_ALL C.UTF-8
ENV LANG C.UTF-8

WORKDIR /app

COPY . /app
```

## Partie 1 : Scan des dépendances Python (15 min)

### Objectif
Détecter les vulnérabilités dans vos dépendances Python avec Safety.

### Tâches

1. Ajoutez un stage `security` à votre pipeline
2. Créez un job `dependency-check` qui :
   - Utilise l'image `python:3.9`
   - Installe `safety` avec `pip install safety`
   - Exécute `safety check --file requirements.txt --json --output safety-report.json`
   - Sauvegarde le rapport en artifact
   - Autorise l'échec du job (`allow_failure: true`)

<details>
  <summary>Solution</summary>
   **Code attendu :**
   
   ```yaml
   dependency-check:
     stage: security
     image: python:3.9
     script:
       - pip install safety
       - safety check --file requirements.txt --json --output safety-report.json || true
     artifacts:
       paths:
         - safety-report.json
       expire_in: 1 week
     allow_failure: true
   ```
</details>

## Partie 2 : Analyse statique du code avec Bandit (15 min)

### Objectif
Identifier les problèmes de sécurité dans votre code Python.

### Tâches

1. Créez un job `sast-scan` qui :
   - Utilise l'image `python:3.9`
   - Installe `bandit` avec `pip install bandit`
   - Scanne votre code avec `bandit -r . -f json -o bandit-report.json`
   - Sauvegarde le rapport en artifact
   - Autorise l'échec du job

<details>
  <summary>Solution</summary>
   **Code attendu :**
      
   ```yaml
   sast-scan:
     stage: security
     image: python:3.9
     script:
       - pip install bandit
       - bandit -r . -f json -o bandit-report.json || true
     artifacts:
       paths:
         - bandit-report.json
       expire_in: 1 week
     allow_failure: true
   ```
</details>

## Partie 3 : Scan de l'image Docker avec Trivy (20 min)

### Objectif
Analyser les vulnérabilités de votre image Docker.

### Tâches

1. Créez un job `container-scan` qui :
   - S'exécute après le build de votre image Docker
   - Utilise l'image `aquasec/trivy:latest`
   - Scanne l'image avec Trivy
   - Génère un rapport JSON
   - Bloque uniquement sur les vulnérabilités CRITICAL

<details>
  <summary>Solution</summary>
   **Code attendu :**

   ```yaml
   container-scan:
     stage: security
     image: aquasec/trivy:latest
     script:
       - trivy image --format json --output trivy-report.json $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
       - trivy image --exit-code 0 --severity HIGH $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
       - trivy image --exit-code 1 --severity CRITICAL $CI_REGISTRY_IMAGE:$CI_COMMIT_REF_SLUG
     artifacts:
       paths:
         - trivy-report.json
       expire_in: 1 week
     dependencies:
       - package
     allow_failure: false
   ```
</details>

**Note :** Ajustez le nom du job de build Docker selon votre configuration (ici `package`).

## Partie 4 : Détection de secrets avec Gitleaks (10 min)

### Objectif
S'assurer qu'aucun secret n'est exposé dans le code.

### Tâches

1. Créez un job `secret-scan` qui :
   - Utilise l'image `zricethezav/gitleaks:latest`
   - Scanne le dépôt avec `gitleaks detect`
   - Génère un rapport JSON
   - Bloque le pipeline si des secrets sont détectés

<details>
  <summary>Solution</summary>
   
   **Code attendu :**
   
   ```yaml
   secret-scan:
     stage: security
     image: zricethezav/gitleaks:latest
     script:
       - gitleaks detect --source . --report-format json --report-path gitleaks-report.json --verbose
     artifacts:
       paths:
         - gitleaks-report.json
       expire_in: 1 week
       when: always
     allow_failure: false
   ```
</details>

## Vérification et tests

1. **Committez et pushez** vos modifications
2. **Vérifiez dans GitLab CI/CD** que tous les jobs s'exécutent
3. **Consultez les artifacts** pour voir les rapports générés
4. **Testez les scénarios** :
   - Ajoutez une dépendance avec vulnérabilité connue (ex: `flask==0.12.0`)
   - Ajoutez un faux secret (ex: `API_KEY="sk-1234567890"`)
   - Observez le comportement du pipeline

---

## Questions de validation

1. Quel job bloque le pipeline en cas de problème critique ?
2. Pourquoi utilise-t-on `allow_failure: true` pour certains jobs ?
3. Comment consulter les rapports de sécurité générés ?
4. Que faire si Trivy détecte une vulnérabilité CRITICAL dans une image de base ?

---

## Livrables

- ✅ Un fichier `.gitlab-ci.yml` mis à jour avec les 4 jobs de sécurité
- ✅ Un pipeline qui s'exécute avec succès
- ✅ Des artifacts de rapports accessibles dans GitLab


## Critères de réussite

- 🎯 Les 4 jobs de sécurité sont présents et fonctionnels
- 🎯 Les rapports sont générés et téléchargeables
- 🎯 Le pipeline bloque sur les vulnérabilités CRITICAL et les secrets
- 🎯 Le pipeline autorise les warnings sur les autres vulnérabilités


## Ressources rapides

- [Safety](https://pyup.io/safety/) - Scan de dépendances Python
- [Bandit](https://bandit.readthedocs.io/) - SAST pour Python
- [Trivy](https://aquasecurity.github.io/trivy/) - Scan de conteneurs
- [Gitleaks](https://github.com/gitleaks/gitleaks) - Détection de secrets

