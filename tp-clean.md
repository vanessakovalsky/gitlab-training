# TP GitLab CI : Automatisation du nettoyage de dépôts et d'espaces de stockage

## Objectifs pédagogiques

À l'issue de ce TP, vous serez capable de :
- Créer des pipelines GitLab CI/CD pour automatiser des tâches de maintenance
- Utiliser les schedules pour exécuter des jobs périodiquement
- Manipuler l'API GitLab pour gérer les dépôts
- Nettoyer automatiquement les branches, tags et artefacts
- Libérer de l'espace de stockage sur les runners et les registres

## Prérequis

- Compte GitLab (gitlab.com ou instance privée)
- Connaissances de base en Git
- Notions de YAML
- Connaissances de base en scripting (Bash/Python)

## Durée estimée

4 heures

---

## Partie 1 : Mise en place de l'environnement (30 min)

### Exercice 1.1 : Création du projet de test

1. Créez un nouveau projet GitLab nommé `cleanup-automation`
2. Clonez le projet localement
3. Créez plusieurs branches de test :
```bash
git checkout -b feature/test-1
git commit --allow-empty -m "Test commit 1"
git push origin feature/test-1

git checkout -b feature/test-2
git commit --allow-empty -m "Test commit 2"
git push origin feature/test-2

git checkout -b hotfix/old-fix
git commit --allow-empty -m "Old hotfix"
git push origin hotfix/old-fix
```

4. Créez des tags de test :
```bash
git checkout main
git tag v1.0.0
git tag v1.1.0
git tag v2.0.0
git push --tags
```

---

## Partie 2 : Nettoyage des branches obsolètes (45 min)

### Exercice 2.1 : Pipeline de base

Créez un fichier `.gitlab-ci.yml` à la racine de votre projet :

```yaml
stages:
  - analyze
  - cleanup

variables:
  GIT_DEPTH: 0

list_branches:
  stage: analyze
  image: alpine/git
  script:
    - git branch -r
    - echo "Total remote branches:"
    - git branch -r | wc -l
  only:
    - schedules
```

**Questions :**
- Que fait `GIT_DEPTH: 0` ?
- Pourquoi utiliser `only: schedules` ?

### Exercice 2.2 : Identification des branches obsolètes

Ajoutez ce job qui identifie les branches non modifiées depuis 30 jours :

```yaml
find_stale_branches:
  stage: analyze
  image: alpine/git
  script:
    - apk add --no-cache bash coreutils
    - |
      echo "Branches not updated in 30 days:"
      for branch in $(git branch -r | grep -v HEAD | sed 's/origin\///'); do
        last_commit_date=$(git log -1 --format=%ci origin/$branch)
        last_commit_timestamp=$(date -d "$last_commit_date" +%s)
        current_timestamp=$(date +%s)
        days_old=$(( ($current_timestamp - $last_commit_timestamp) / 86400 ))
        
        if [ $days_old -gt 30 ]; then
          echo "  - $branch (${days_old} days old)"
        fi
      done
  artifacts:
    paths:
      - stale_branches.txt
    expire_in: 1 day
  only:
    - schedules
```

**Tâche :** Modifiez le script pour sauvegarder la liste des branches dans `stale_branches.txt`

### Exercice 2.3 : Suppression automatique via l'API

Ajoutez ce job pour supprimer les branches obsolètes :

```yaml
cleanup_branches:
  stage: cleanup
  image: curlimages/curl:latest
  script:
    - |
      # Protection contre la suppression de branches critiques
      PROTECTED_BRANCHES="main|master|develop|staging|production"
      
      # Simulation : changez DRY_RUN=false pour réellement supprimer
      DRY_RUN=true
      
      for branch in $(cat stale_branches.txt); do
        if echo "$branch" | grep -qE "$PROTECTED_BRANCHES"; then
          echo "Skipping protected branch: $branch"
          continue
        fi
        
        if [ "$DRY_RUN" = true ]; then
          echo "Would delete: $branch"
        else
          curl --request DELETE \
               --header "PRIVATE-TOKEN: $CI_GITLAB_TOKEN" \
               "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/repository/branches/${branch}"
          echo "Deleted: $branch"
        fi
      done
  dependencies:
    - find_stale_branches
  when: manual
  only:
    - schedules
```

**Questions :**
- Pourquoi utiliser `when: manual` ?
- Comment testeriez-vous ce pipeline en toute sécurité ?

---

## Partie 3 : Nettoyage des tags anciens (45 min)

### Exercice 3.1 : Politique de rétention des tags

Créez une politique pour ne garder que les 5 derniers tags :

```yaml
cleanup_old_tags:
  stage: cleanup
  image: alpine/git
  before_script:
    - apk add --no-cache curl jq
  script:
    - |
      # Récupérer tous les tags via l'API
      tags=$(curl --silent --header "PRIVATE-TOKEN: $CI_GITLAB_TOKEN" \
        "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/repository/tags" | jq -r '.[].name')
      
      # Compter les tags
      total_tags=$(echo "$tags" | wc -l)
      echo "Total tags: $total_tags"
      
      # Garder les 5 derniers, supprimer les autres
      KEEP_COUNT=5
      
      if [ $total_tags -gt $KEEP_COUNT ]; then
        tags_to_delete=$(echo "$tags" | tail -n +$(($KEEP_COUNT + 1)))
        
        for tag in $tags_to_delete; do
          echo "Deleting tag: $tag"
          curl --request DELETE \
               --header "PRIVATE-TOKEN: $CI_GITLAB_TOKEN" \
               "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/repository/tags/${tag}"
        done
      else
        echo "Only $total_tags tags found, nothing to delete"
      fi
  when: manual
  only:
    - schedules
```

**Tâche :** Modifiez le script pour garder uniquement les tags de production (format `v*.*.0`)

---

## Partie 4 : Nettoyage des artefacts CI/CD (45 min)

### Exercice 4.1 : Génération d'artefacts de test

Ajoutez un job qui génère des artefacts :

```yaml
generate_artifacts:
  stage: analyze
  image: alpine:latest
  script:
    - mkdir -p build/logs
    - dd if=/dev/zero of=build/large_file.bin bs=1M count=50
    - echo "Build completed at $(date)" > build/logs/build.log
  artifacts:
    paths:
      - build/
    expire_in: 1 week
  except:
    - schedules
```

### Exercice 4.2 : Nettoyage des anciens artefacts

```yaml
cleanup_artifacts:
  stage: cleanup
  image: alpine:latest
  before_script:
    - apk add --no-cache curl jq
  script:
    - |
      # Récupérer les jobs avec artefacts de plus de 7 jours
      jobs=$(curl --silent --header "PRIVATE-TOKEN: $CI_GITLAB_TOKEN" \
        "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/jobs?per_page=100")
      
      echo "$jobs" | jq -r '.[] | select(.artifacts_file != null) | "\(.id) \(.created_at)"' | \
      while read job_id created_at; do
        # Calculer l'âge en jours
        created_timestamp=$(date -d "$created_at" +%s)
        current_timestamp=$(date +%s)
        age_days=$(( ($current_timestamp - $created_timestamp) / 86400 ))
        
        if [ $age_days -gt 7 ]; then
          echo "Deleting artifacts from job $job_id (${age_days} days old)"
          curl --request DELETE \
               --header "PRIVATE-TOKEN: $CI_GITLAB_TOKEN" \
               "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/jobs/${job_id}/artifacts"
        fi
      done
  when: manual
  only:
    - schedules
```

**Question :** Quels sont les avantages et inconvénients de supprimer les artefacts automatiquement ?

---

## Partie 5 : Nettoyage du cache du runner (30 min)

### Exercice 5.1 : Gestion du cache

```yaml
clear_cache:
  stage: cleanup
  image: alpine:latest
  script:
    - echo "Clearing all caches for this project"
    - |
      curl --request DELETE \
           --header "PRIVATE-TOKEN: $CI_GITLAB_TOKEN" \
           "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/runner_caches"
  cache: {}
  when: manual
  only:
    - schedules
```

### Exercice 5.2 : Rapport d'utilisation du stockage

Créez un job qui génère un rapport sur l'utilisation du stockage :

```yaml
storage_report:
  stage: analyze
  image: alpine:latest
  before_script:
    - apk add --no-cache curl jq
  script:
    - |
      echo "=== Storage Usage Report ==="
      
      # Taille du dépôt
      repo_size=$(curl --silent --header "PRIVATE-TOKEN: $CI_GITLAB_TOKEN" \
        "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}" | jq -r '.statistics.repository_size')
      echo "Repository size: $(($repo_size / 1024 / 1024)) MB"
      
      # Taille des artefacts
      artifacts_size=$(curl --silent --header "PRIVATE-TOKEN: $CI_GITLAB_TOKEN" \
        "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}" | jq -r '.statistics.job_artifacts_size')
      echo "Artifacts size: $(($artifacts_size / 1024 / 1024)) MB"
      
      # Nombre de branches
      branches_count=$(curl --silent --header "PRIVATE-TOKEN: $CI_GITLAB_TOKEN" \
        "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/repository/branches" | jq '. | length')
      echo "Total branches: $branches_count"
      
      # Nombre de tags
      tags_count=$(curl --silent --header "PRIVATE-TOKEN: $CI_GITLAB_TOKEN" \
        "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/repository/tags" | jq '. | length')
      echo "Total tags: $tags_count"
  artifacts:
    reports:
      dotenv: storage_report.env
  only:
    - schedules
```

---

## Partie 6 : Configuration du scheduling (30 min)

### Exercice 6.1 : Création d'un schedule

1. Allez dans **CI/CD > Schedules**
2. Créez un nouveau schedule :
   - Description : "Weekly cleanup"
   - Interval pattern : `0 2 * * 0` (tous les dimanches à 2h)
   - Target branch : `main`
   - Variables : `CLEANUP_MODE=auto`

3. Créez un second schedule :
   - Description : "Daily storage report"
   - Interval pattern : `0 8 * * *` (tous les jours à 8h)
   - Target branch : `main`

### Exercice 6.2 : Adaptation du pipeline aux schedules

Modifiez votre `.gitlab-ci.yml` pour différencier les comportements :

```yaml
workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "schedule" && $CLEANUP_MODE == "auto"
      variables:
        DRY_RUN: "false"
    - if: $CI_PIPELINE_SOURCE == "schedule"
      variables:
        DRY_RUN: "true"
    - when: always
```

---

## Partie 7 : Projet final (45 min)

### Mission : Pipeline de nettoyage complet

Créez un pipeline complet qui :

1. **Analyse** (tous les jours) :
   - Génère un rapport sur l'utilisation du stockage
   - Liste les branches non utilisées depuis 60 jours
   - Liste les tags de plus de 6 mois
   - Compte les artefacts de plus de 30 jours

2. **Alerte** :
   - Envoie une notification si le stockage dépasse 1 GB
   - Crée une issue GitLab avec la liste des éléments à nettoyer

3. **Nettoyage** (hebdomadaire, mode manuel) :
   - Supprime les branches obsolètes (sauf protégées)
   - Supprime les artefacts de plus de 30 jours
   - Garde uniquement les 10 derniers tags
   - Vide le cache du runner

4. **Documentation** :
   - Génère un artifact avec un rapport de nettoyage
   - Journalise toutes les actions effectuées

### Template de départ

```yaml
stages:
  - analyze
  - alert
  - cleanup
  - report

variables:
  STORAGE_THRESHOLD_MB: 1024
  BRANCH_AGE_DAYS: 60
  TAG_KEEP_COUNT: 10
  ARTIFACT_AGE_DAYS: 30

# À compléter...
```

---

## Questions de réflexion

1. Quels sont les risques d'automatiser complètement le nettoyage sans validation humaine ?
2. Comment mettre en place une politique de rétention différente selon le type de branche (feature, hotfix, release) ?
3. Comment intégrer ce système dans une organisation avec plusieurs projets ?
4. Quelles métriques devriez-vous suivre pour évaluer l'efficacité de votre stratégie de nettoyage ?

---

## Ressources complémentaires

- [Documentation GitLab CI/CD](https://docs.gitlab.com/ee/ci/)
- [GitLab API Reference](https://docs.gitlab.com/ee/api/)
- [Pipeline Schedules](https://docs.gitlab.com/ee/ci/pipelines/schedules.html)
- [Best practices pour les artefacts](https://docs.gitlab.com/ee/ci/pipelines/job_artifacts.html)

## Solution complète

https://gitlab.com/formation15/cleanup-automation
