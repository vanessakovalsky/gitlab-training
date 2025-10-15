# TP - Intégration de la sécurité dans le pipeline CI/CD

## Objectifs

L'objectif de cet exercice est d'intégrer plusieurs outils d'analyse de sécurité dans votre pipeline GitLab CI/CD pour votre application Python Flask. Vous allez mettre en place :

1. L'analyse de vulnérabilités des dépendances Python
2. L'analyse statique du code (SAST)
3. L'analyse de sécurité des conteneurs Docker
4. L'analyse de secrets exposés
5. La génération de rapports de sécurité

## Prérequis

- Un projet GitLab avec une application Python Flask
- Un fichier `.gitlab-ci.yml` fonctionnel
- Un `requirements.txt` avec vos dépendances Python
- Un `Dockerfile` pour containeriser l'application

## Partie 1 : Analyse des dépendances Python avec Safety

### Objectif
Détecter les vulnérabilités connues dans vos dépendances Python.

### Tâches

1. Ajoutez un nouveau stage `security` dans votre pipeline
2. Créez un job `dependency-scanning` qui :
   - Utilise l'image Python
   - Installe l'outil `safety`
   - Scanne le fichier `requirements.txt`
   - Génère un rapport des vulnérabilités

### Questions
- Que fait la commande `safety check --json` ?
- Comment configurer le job pour qu'il ne bloque pas le pipeline même en cas de vulnérabilités ?

## Partie 2 : Analyse statique avec Bandit

### Objectif
Identifier les problèmes de sécurité potentiels dans votre code Python.

### Tâches

1. Créez un job `sast-python` qui :
   - Installe l'outil `bandit`
   - Analyse tous les fichiers Python du projet
   - Génère un rapport au format JSON
   - Sauvegarde le rapport comme artifact

### Questions
- Quels types de vulnérabilités Bandit peut-il détecter ?
- Comment exclure certains fichiers ou répertoires de l'analyse ?

## Partie 3 : Scan de sécurité des conteneurs avec Trivy

### Objectif
Analyser les vulnérabilités présentes dans votre image Docker.

### Tâches

1. Créez un job `container-scanning` qui :
   - S'exécute après le job de build Docker
   - Utilise l'image `aquasec/trivy:latest`
   - Scanne l'image Docker construite précédemment
   - Génère un rapport des vulnérabilités
   - Configure des seuils de sévérité (HIGH, CRITICAL)

### Questions
- Quelle est la différence entre scanner une image et scanner les dépendances ?
- Comment ignorer des CVE spécifiques si vous savez qu'ils ne s'appliquent pas à votre cas ?

## Partie 4 : Détection de secrets avec Gitleaks

### Objectif
S'assurer qu'aucun secret (clés API, mots de passe, tokens) n'est présent dans le code.

### Tâches

1. Créez un job `secret-detection` qui :
   - Utilise l'image `zricethezav/gitleaks:latest`
   - Scanne l'historique Git et le code source
   - Bloque le pipeline en cas de secret détecté
   - Génère un rapport JSON

### Questions
- Pourquoi est-il important de scanner l'historique Git et pas seulement le commit actuel ?
- Comment gérer les faux positifs ?

## Partie 5 : Analyse de dépendances avec OWASP Dependency-Check

### Objectif
Utiliser OWASP Dependency-Check pour une analyse complète des dépendances.

### Tâches

1. Créez un job `owasp-dependency-check` qui :
   - Utilise l'image `owasp/dependency-check:latest`
   - Scanne le projet complet
   - Génère un rapport HTML
   - Configure le niveau de sévérité minimum pour échouer

### Questions
- Quelle est la différence entre Safety et OWASP Dependency-Check ?
- Comment mettre à jour la base de données de vulnérabilités ?

## Partie 6 : Centralisation des rapports de sécurité

### Objectif
Créer un job qui agrège tous les rapports de sécurité.

### Tâches

1. Créez un job `security-report` qui :
   - S'exécute après tous les jobs de sécurité
   - Collecte tous les artifacts de sécurité
   - Génère un rapport consolidé
   - Publie le rapport sur GitLab Pages (optionnel)

2. Configurez les artifacts pour qu'ils soient disponibles :
   - Dans l'interface GitLab
   - Avec une durée de rétention appropriée
   - Avec des rapports au format compatible GitLab Security Dashboard

### Questions
- Comment exploiter les rapports au format GitLab Security pour avoir une vue dans l'interface ?
- Quels sont les avantages d'avoir un rapport consolidé ?

## Partie 7 : Configuration des politiques de sécurité

### Objectif
Définir des règles pour le pipeline de sécurité.

### Tâches

1. Configurez votre pipeline pour :
   - Bloquer le merge si des vulnérabilités CRITICAL sont détectées
   - Permettre le merge avec des warnings pour les vulnérabilités LOW et MEDIUM
   - Exiger une revue manuelle pour les vulnérabilités HIGH

2. Ajoutez des badges de sécurité dans votre README.md

### Questions
- Comment gérer le compromis entre sécurité et vélocité de développement ?
- Quand faut-il accepter une exception de sécurité ?

## Bonus : Intégration continue de la sécurité

### Tâches avancées

1. Configurez un scan de sécurité régulier (scheduled pipeline) même sans nouveau commit
2. Intégrez des notifications Slack/Email en cas de nouvelle vulnérabilité
3. Créez des issues GitLab automatiquement pour les vulnérabilités détectées
4. Mettez en place un système de suppression des vulnérabilités (auto-update des dépendances)

## Livrables attendus

- Un fichier `.gitlab-ci.yml` complet avec tous les jobs de sécurité
- Des artifacts de rapports pour chaque outil
- Un fichier `.trivyignore` pour gérer les exceptions Trivy
- Un fichier `.gitleaks.toml` pour configurer Gitleaks
- Une documentation expliquant :
  - Comment interpréter chaque rapport
  - La procédure à suivre en cas de vulnérabilité détectée
  - Les seuils de sécurité définis

## Critères de réussite

- ✅ Tous les jobs de sécurité s'exécutent correctement
- ✅ Les rapports sont générés et accessibles
- ✅ Le pipeline se comporte correctement selon la sévérité des vulnérabilités
- ✅ Aucun secret n'est détecté dans le code
- ✅ Les vulnérabilités CRITICAL bloquent le pipeline
- ✅ Les rapports sont au format compatible avec GitLab Security Dashboard

## Ressources utiles

- [GitLab Security Scanning](https://docs.gitlab.com/ee/user/application_security/)
- [Safety](https://pyup.io/safety/)
- [Bandit](https://bandit.readthedocs.io/)
- [Trivy](https://aquasecurity.github.io/trivy/)
- [Gitleaks](https://github.com/gitleaks/gitleaks)
- [OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/)

## Temps estimé

- Parties 1-4 : 2-3 heures
- Parties 5-7 : 2-3 heures
- Bonus : 2-4 heures

---

**Note :** Cet exercice vous prépare aux bonnes pratiques DevSecOps et à l'intégration de la sécurité dans le cycle de développement (Shift Left Security).
