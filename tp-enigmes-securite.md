# 🔐 TP Énigmes - Sécurité CI/CD GitLab

## 🎯 Objectif

Résoudre des énigmes de sécurité dans un pipeline CI/CD en analysant du code, des configurations et des rapports. Durée : **45 minutes**

---

## 🕵️ Énigme 1 : Le Secret Caché (10 min)

### Contexte
Un développeur a commité ce fichier `config.py` dans le projet :

```python
import os

# Configuration de l'application
APP_NAME = "FlaskSecure"
DEBUG = False

# Base de données
DB_HOST = "localhost"
DB_USER = "admin"
DB_PASSWORD = "sup3rS3cr3t2024!"

# API Externe
API_KEY = "sk-proj-abcdef123456789"
STRIPE_SECRET = "sk_live_51HxYz..."

# Configuration AWS
AWS_ACCESS_KEY = os.getenv("AWS_KEY", "AKIAIOSFODNN7EXAMPLE")
AWS_SECRET = os.getenv("AWS_SECRET", "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY")
```

### Questions

1. **Combien de secrets critiques sont exposés dans ce fichier ?**
   - Indice : Les `os.getenv()` avec valeur par défaut sont aussi dangereux !

2. **Quel outil de notre pipeline détectera ces secrets ?**

3. **Corrigez ce fichier** en utilisant les bonnes pratiques :
   - Variables d'environnement sans valeur par défaut
   - Fichier `.env.example` pour la documentation
   - Mise à jour du `.gitignore`

---

## 🐛 Énigme 2 : La Vulnérabilité Invisible (10 min)

### Contexte
Votre pipeline génère ce rapport Safety :

```json
{
  "vulnerabilities": [
    {
      "package": "flask",
      "version": "0.12.0",
      "vulnerability": "Flask 0.12.0 has a XSS vulnerability",
      "severity": "HIGH",
      "cve": "CVE-2018-1000656"
    },
    {
      "package": "jinja2",
      "version": "2.10",
      "vulnerability": "Jinja2 2.10 sandbox escape",
      "severity": "CRITICAL",
      "cve": "CVE-2019-10906"
    },
    {
      "package": "werkzeug",
      "version": "0.14.0",
      "vulnerability": "Debug mode RCE",
      "severity": "MEDIUM",
      "cve": "CVE-2019-14806"
    }
  ]
}
```

### Questions

1. **Quelle vulnérabilité devrait bloquer le pipeline immédiatement ?**

2. **Complétez ce job pour qu'il bloque uniquement sur CRITICAL :**

```yaml
dependency-check:
  stage: security
  image: python:3.9
  script:
    - pip install safety
    - safety check --file requirements.txt --json --output report.json
    - # ⚠️ LIGNE À COMPLÉTER : bloquer si CRITICAL
  allow_failure: ???  # ⚠️ À CORRIGER
```

3. **Mettez à jour le `requirements.txt`** pour corriger ces vulnérabilités.

---

## 🔍 Énigme 3 : Le Code Dangereux (10 min)

### Contexte
Bandit a scanné ce code et génère des alertes :

```python
# app.py
from flask import Flask, request
import os
import pickle
import yaml

app = Flask(__name__)

@app.route('/upload', methods=['POST'])
def upload_file():
    file = request.files['file']
    # Alerte Bandit : B301 - pickle usage
    data = pickle.loads(file.read())
    return {"status": "ok", "data": data}

@app.route('/config', methods=['POST'])
def update_config():
    config_data = request.data
    # Alerte Bandit : B506 - yaml.load without Loader
    config = yaml.load(config_data)
    return {"config": config}

@app.route('/execute', methods=['POST'])
def execute_command():
    cmd = request.form.get('command')
    # Alerte Bandit : B605, B607 - shell injection
    os.system(cmd)
    return {"executed": cmd}

if __name__ == '__main__':
    # Alerte Bandit : B201 - Flask debug mode
    app.run(debug=True, host='0.0.0.0')
```

### Questions

1. **Identifiez les 4 vulnérabilités critiques** et expliquez pourquoi elles sont dangereuses.

2. **Corrigez chaque fonction** pour la rendre sécurisée :
   - Route `/upload` : Remplacez pickle
   - Route `/config` : Utilisez SafeLoader
   - Route `/execute` : Utilisez subprocess avec validation
   - `app.run()` : Configuration sécurisée

3. **Quel niveau de sévérité Bandit attribue-t-il à `os.system()` ?**

---

## 🐳 Énigme 4 : L'Image Compromise (15 min)

### Contexte
Trivy scanne votre Dockerfile et trouve ces problèmes :

```dockerfile
FROM python:3.7
USER root
RUN apt-get update && apt-get install -y curl vim git

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . /app
WORKDIR /app

# Secrets hardcodés
ENV DB_PASSWORD="MonMotDePasse123"
ENV API_SECRET="secret-key-1234567890"

EXPOSE 5000
CMD ["python", "app.py"]
```

**Rapport Trivy (extrait) :**
```
CRITICAL: CVE-2023-XXXX in libssl1.1
HIGH: CVE-2023-YYYY in python:3.7 base image
MEDIUM: Running as root user
LOW: Unnecessary packages installed
```

### Questions

1. **Proposez un Dockerfile corrigé** qui :
   - Utilise une image de base récente et non-vulnérable
   - N'expose pas de secrets
   - N'exécute pas en tant que root
   - N'installe que les packages nécessaires
   - Utilise un multi-stage build (bonus)

2. **Créez un fichier `.trivyignore`** pour ignorer une CVE spécifique (CVE-2023-YYYY) avec justification.

3. **Complétez ce job** pour qu'il scanne l'image et bloque uniquement sur CRITICAL :

```yaml
container-scan:
  stage: security
  image: aquasec/trivy:latest
  script:
    - trivy image --format json --output report.json ??? # Image à scanner
    - trivy image --exit-code ??? --severity ??? ??? # Bloquer sur CRITICAL
```

---

## 🎲 Énigme Bonus : Le Pipeline Mystère (facultatif)

### Contexte
Ce pipeline ne fonctionne pas comme prévu :

```yaml
stages:
  - security

secret-scan:
  stage: security
  image: zricethezav/gitleaks:latest
  script:
    - gitleaks detect --source . --report-format json
  allow_failure: true
  only:
    - main

sast-scan:
  stage: security
  image: python:3.9
  script:
    - pip install bandit
    - bandit -r . -f json -o report.json
  artifacts:
    paths:
      - report.json
  needs: [secret-scan]
  allow_failure: false
```

### Questions

1. **Trouvez 3 problèmes** dans cette configuration :
   - Indice 1 : Que se passe-t-il si `secret-scan` trouve un secret ?
   - Indice 2 : Regardez la relation entre les jobs
   - Indice 3 : Le rapport de gitleaks est-il sauvegardé ?

2. **Corrigez le pipeline** pour qu'il soit cohérent et fonctionnel.

---

## 📝 Grille de correction

| Énigme | Points | Critères |
|--------|--------|----------|
| Énigme 1 | 20 pts | Identification des secrets (10) + Correction (10) |
| Énigme 2 | 20 pts | Analyse vulnérabilités (10) + Configuration job (10) |
| Énigme 3 | 25 pts | Identification (10) + Correction code (15) |
| Énigme 4 | 30 pts | Dockerfile (15) + Trivy config (10) + .trivyignore (5) |
| Bonus | 5 pts | Analyse et correction pipeline |
| **Total** | **100 pts** | |

---

## 🎯 Solutions à remettre

1. Fichier `config.py` corrigé + `.env.example`
2. Job `dependency-check` complété
3. Fichier `app.py` corrigé avec explications
4. `Dockerfile` sécurisé + `.trivyignore`
5. (Bonus) Pipeline corrigé avec explications

---

## 💡 Ressources

- [OWASP Top 10](https://owasp.org/Top10/)
- [Bandit Documentation](https://bandit.readthedocs.io/)
- [Trivy Documentation](https://aquasecurity.github.io/trivy/)
- [Gitleaks Patterns](https://github.com/gitleaks/gitleaks)

**Bon courage, détective ! 🕵️‍♂️**
