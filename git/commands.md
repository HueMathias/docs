# 🌳 Commandes Git essentielles

> Référence pratique pour la gestion des versions, des branches et des dépôts Git.  

---

## 🧱 Configuration initiale

Configurer ton identité globale (utilisée dans tous les dépôts) :

```bash
git config --global user.name "nom utilisateur"
git config --global user.email "email@exemple.com"
```

Lister la configuration actuelle :

```bash
git config --list
```

## 📁 Initialiser et cloner un dépôt

Créer un nouveau dépôt local :
```bash
git init
```

Cloner un dépôt existant :
```bash
git clone <chemin.git>
```

Cloner uniquement une branche spécifique :
```bash
git clone -b <branche> <chemin.git>
```

## 💾 Suivi et sauvegarde des fichiers

Vérifier l'état du dépot :
```bash
git status
```

Ajouter des fichiers à la zone de staging :
```bash
git add <fichier>
```

ou tout le dossier courant :
```bash
git add .
```

Retirer un fichier du suivi :
```bash
git rm --cached <fichier>
```

## 🧩 Commits

Créer un commit :
```bash
git commit -m "Message"
```

Modifier le dernier commit (sans en créer un nouveau) :
```bash
git commit --amend
```

## 🌿 Branches

Lister les branches :
```bash
git branch
```

Créer une nouvelle branche :
```bash
git branch <nom_branche>
```

Basculer sur une branche :
```bash
git checkout <nom_branche>
```

Créer et basculer en une seule commande :
```bash
git checkout -b <nom_branche>
```

Supprimer une branche locale :
```bash
git branch -d <nom_branche>
```

## 🔄 Fusion & Réintégration

Fusionner une branche dans la branche courante :
```bash
git merge <nom_branche>
```

## 🚀 Synchronisation avec un dépôt distant

Lister les dépôts distants :
```bash
git remote -v
```

Ajouter un dépôt distant :
```bash
git remote add origin <chemin.git>
```

Envoyer les changements :
```bash
git push origin main
```

Récupérer les mises à jour :
```bash
git pull
```

Récupérer sans fusionner automatiquement :
```bash
git fetch
```

## 🧰 Revert, Reset & Stash

Annuler un commit (sans supprimer l’historique) :
```bash
git revert <commit_id>
```

Revenir à un commit précédent (Attention ! efface les changements locaux) :
```bash
git reset --hard <commit_id>
```

Mettre temporairement de côté des modifications non commitées :
```bash
git stash
```

Restaurer ce qui a été mis en stash :
```bash
git stash pop
```

## 🧭 Tagging (versions)

Créer un tag pour une version spécifique :
```bash
git tag -a <nom_tag> -m "Description"
```

Lister les tags :
```bash
git tag
```

Pousser les tags sur le dépôt distant :
```bash
git push origin --tags
```

Pousser un tag spécifique sur le dépôt distant :
```bash
git push origin <nom_tag>
```
