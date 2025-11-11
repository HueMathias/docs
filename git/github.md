# Documentation GitHub et GitHub Actions

## 📋 Table des matières

1. [Introduction à GitHub](#introduction-à-github)
2. [Concepts de base](#concepts-de-base)
3. [GitHub Actions - Vue d'ensemble](#github-actions---vue-densemble)
4. [Créer un workflow](#créer-un-workflow)
5. [Syntaxe des workflows](#syntaxe-des-workflows)
6. [Exemples .NET](#exemples-.net)
7. [Bonnes pratiques](#bonnes-pratiques)
8. [Ressources utiles](#ressources-utiles)

---

## Introduction à GitHub

GitHub est une plateforme de développement collaboratif basée sur Git qui permet de :
- Héberger et versionner du code source
- Collaborer avec d'autres développeurs
- Gérer des projets avec des issues et des pull requests
- Automatiser des tâches avec GitHub Actions

---

## Concepts de base

### Repository (Dépôt)
Espace de stockage pour votre projet contenant tous les fichiers et l'historique des modifications.

### Branch (Branche)
Version parallèle du code permettant de travailler sur des fonctionnalités sans affecter la branche principale (`main` ou `master`).

### Commit
Enregistrement d'un ensemble de modifications avec un message descriptif.

### Pull Request (PR)
Demande de fusion de modifications d'une branche vers une autre, permettant la revue de code.

### Issue
Ticket pour suivre des bugs, des améliorations ou des tâches.

---

## GitHub Actions - Vue d'ensemble

### Qu'est-ce que GitHub Actions ?

GitHub Actions est un système d'intégration et de déploiement continu (CI/CD) intégré à GitHub qui permet d'automatiser des workflows directement depuis votre repository.

### Concepts clés

**Workflow** : Processus automatisé configurable composé d'un ou plusieurs jobs.

**Event** : Déclencheur d'un workflow (push, pull request, schedule, etc.).

**Job** : Ensemble d'étapes (steps) exécutées sur le même runner.

**Step** : Tâche individuelle qui exécute des commandes ou des actions.

**Action** : Application réutilisable pour effectuer une tâche complexe.

**Runner** : Serveur qui exécute les workflows (hébergé par GitHub ou auto-hébergé).

---

## Créer un workflow

### Étape 1 : Créer le fichier de workflow

Créez le dossier `.github/workflows/` à la racine du répertoire et ajouter un fichier YAML :

```yaml
# .github/workflows/hello-world.yml
name: Hello World

on: [push]

jobs:
  hello-job:
    runs-on: ubuntu-latest
    steps:
      - name: Say hello
        run: echo "Hello, World!"
```

### Étape 2 : Commit et push

```bash
git add .github/workflows/hello-world.yml
git commit -m "Add hello world workflow"
git push
```

### Étape 3 : Vérifier l'exécution

Aller dans l'onglet **Actions** du répertoire sur GitHub pour voir le workflow en cours d'exécution.

---

## Syntaxe des workflows

### Structure de base

```yaml
name: Nom du workflow

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  nom-du-job:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout du code
      uses: actions/checkout@v4
      
    - name: Exécuter une commande
      run: echo "Ma commande"
```

### Déclencheurs (Events)

```yaml
# Push sur une branche spécifique
on:
  push:
    branches: [ main, develop ]

# Pull request
on:
  pull_request:
    types: [ opened, synchronize ]

# Planification (cron)
on:
  schedule:
    - cron: '0 0 * * *'  # Tous les jours à minuit

# Déclenchement manuel
on:
  workflow_dispatch:

# Plusieurs événements
on: [push, pull_request, workflow_dispatch]
```

### Variables d'environnement

```yaml
env:
  GLOBAL_VAR: "valeur globale"

jobs:
  mon-job:
    runs-on: ubuntu-latest
    env:
      JOB_VAR: "valeur au niveau du job"
    
    steps:
    - name: Utiliser les variables
      run: |
        echo "Variable globale: $GLOBAL_VAR"
        echo "Variable du job: $JOB_VAR"
      env:
        STEP_VAR: "valeur au niveau du step"
```

### Secrets

```yaml
steps:
  - name: Utiliser un secret
    run: echo "Token: ${{ secrets.MY_SECRET }}"
```

Les secrets se configurent dans : **Settings → Secrets and variables → Actions**

---

## Exemples .NET

### Build et tests automatisés

```yaml
name: .NET CI

on: [push, pull_request]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
    - name: Récupérer le code
      uses: actions/checkout@v4
    
    - name: Configurer .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'
        
    - name: Restaurer les dépendances
      run: dotnet restore
      
    - name: Compiler le projet
      run: dotnet build --no-restore --configuration Release
      
    - name: Exécuter les tests
      run: dotnet test --no-build --configuration Release --verbosity normal
```

### Build multi-versions .NET

```yaml
name: Build Multi-versions

on: [push, pull_request]

jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        dotnet-version: ['6.0.x', '7.0.x', '8.0.x']
    
    steps:
    - name: Récupérer le code
      uses: actions/checkout@v4
      
    - name: Configurer .NET ${{ matrix.dotnet-version }}
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: ${{ matrix.dotnet-version }}
        
    - name: Restaurer les dépendances
      run: dotnet restore
      
    - name: Compiler
      run: dotnet build --configuration Release
      
    - name: Tester
      run: dotnet test --no-build --configuration Release
```

### Publication NuGet

```yaml
name: Publier sur NuGet

on:
  push:
    tags:
      - 'v*'

jobs:
  publish:
    runs-on: ubuntu-latest
    
    steps:
    - name: Récupérer le code
      uses: actions/checkout@v4
    
    - name: Configurer .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'
        
    - name: Restaurer les dépendances
      run: dotnet restore
      
    - name: Compiler en Release
      run: dotnet build --configuration Release --no-restore
      
    - name: Créer le package NuGet
      run: dotnet pack --configuration Release --no-build --output ./nupkg
      
    - name: Publier sur NuGet
      run: dotnet nuget push ./nupkg/*.nupkg --api-key ${{ secrets.NUGET_API_KEY }} --source https://api.nuget.org/v3/index.json
```

### Déployer une application Web

```yaml
name: Déployer sur Azure

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Récupérer le code
      uses: actions/checkout@v4
    
    - name: Configurer .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'
        
    - name: Restaurer les dépendances
      run: dotnet restore
      
    - name: Compiler
      run: dotnet build --configuration Release --no-restore
      
    - name: Publier l'application
      run: dotnet publish --configuration Release --no-build --output ./publish
      
    - name: Déployer sur Azure Web App
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ secrets.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ./publish
```

### Analyse de code avec SonarCloud

```yaml
name: Analyse de code

on: [push, pull_request]

jobs:
  analyze:
    runs-on: ubuntu-latest
    
    steps:
    - name: Récupérer le code
      uses: actions/checkout@v4
      with:
        fetch-depth: 0
    
    - name: Configurer .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'
        
    - name: Installer SonarScanner
      run: dotnet tool install --global dotnet-sonarscanner
      
    - name: Démarrer l'analyse SonarCloud
      run: |
        dotnet sonarscanner begin /k:"${{ secrets.SONAR_PROJECT_KEY }}" \
          /o:"${{ secrets.SONAR_ORGANIZATION }}" \
          /d:sonar.login="${{ secrets.SONAR_TOKEN }}" \
          /d:sonar.host.url="https://sonarcloud.io"
      
    - name: Compiler le projet
      run: dotnet build --configuration Release
      
    - name: Terminer l'analyse
      run: dotnet sonarscanner end /d:sonar.login="${{ secrets.SONAR_TOKEN }}"
```

---

## Bonnes pratiques

### Sécurité

- Toujours utiliser des **secrets** pour les informations sensibles (tokens, mots de passe)
- Spécifier des versions précises pour les actions (`@v4` plutôt que `@main`)
- Limiter les permissions avec `permissions:` dans les workflows
- Ne jamais afficher de secrets dans les logs

### Performance

- Exécuter des jobs en parallèle quand c'est possible
- Limiter les déclencheurs aux branches nécessaires

### Organisation

- Donner des noms clairs et descriptifs aux workflows et jobs
- Documenter les workflows complexes avec des commentaires
- Séparer les workflows par fonction (tests, déploiement, release)
- Utiliser des actions réutilisables pour éviter la duplication

### Déboggage

- Utiliser `actions/upload-artifact@v4` pour conserver des fichiers de log
- Activer le mode debug en ajoutant le secret `ACTIONS_STEP_DEBUG: true`
- Consulter les logs détaillés dans l'onglet Actions

---

## Ressources utiles

### Documentation officielle
- [GitHub Actions Documentation](https://docs.github.com/actions)
- [Marketplace GitHub Actions](https://github.com/marketplace?type=actions)
- [GitHub Actions Cheat Sheet](https://github.github.io/actions-cheat-sheet/)

### Actions populaires
- `actions/checkout@v4` : Clone le répertoire
- `actions/cache@v4` : Met en cache les dépendances
- `actions/upload-artifact@v4` : Sauvegarde des fichiers
- `peaceiris/actions-gh-pages@v3` : Déploiement sur GitHub Pages
