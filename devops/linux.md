# 🐧 Commandes Linux essentielles

> Notes pratiques pour l’administration système, les bases de données, la compression, et la configuration web.

---

## 📂 Liens symboliques

Créer un lien symbolique nommé `fichier` qui pointe vers un fichier :

```bash
ln -s <chemin_fichier> <fichier>
```

## 📦 Compression & Archivage

Bzip2 → pour compresser des fichiers :
```bash
bzip2 <fichier>
```

## 🗄️ Sauvegardes MySQL / MariaDB

Exporter une base de données complète :
```bash
mysqldump -u <user> -p <nom_base> > <fichier.sql>
```

## 💾 Vérification de l’espace disque

Espace disque global :
```bash
df -h
```

Espace utilisé par dossier (ajouter * pour les sous-dossiers) :
```bash
du -sh
```

## 🔗 Connexions et processus

Voir le nombre de connexions MariaDB actives (par exemple pour dotnet) :
```bash
watch -n 1 "netstat -petulan | grep dotnet | wc -l"
```

## 🌐 Configuration Nginx

Emplacement des fichiers de configuration : `/etc/nginx/conf.d`

Vérifier la configuration avant redémarrage :
```bash
nginx -t
```

Recharger la configuration sans redémarrer le service :
```bash
nginx -s reload
```

## 📜 Logs système & applicatifs

Afficher les logs d’un service (depuis hier) :
```bash
journalctl -u <service> --since yesterday
```

Logs Nginx :
```bash
tail -f /var/log/nginx/<site>
```
Logs de synchronisation :
```bash
/var/log/synx
```

## 🔍 Recherche dans les fichiers

Options utiles :
- `-i` → insensible à la casse
- `-n` → afficher le numéro de ligne
- `-r` → recherche récursive

Exemple :
```bash
grep -inr "texte à chercher" /chemin/du/dossier/*
```

## 👤 Gestion des utilisateurs

Créer un utilisateur :
```bash
useradd -d <chemin> -m -s /bin/sh <utilisateur>
```

| Option | Signification | Description |
|---  |:-:  |---  |
| `-d <chemin>` | **Répertoire personnel (home directory)** | Définit le chemin du dossier personnel de l’utilisateur au lieu du répertoire par défaut `/home/<utilisateur>`. |
| `-m` | **Créer le répertoire s’il n’existe pas** | Si le dossier n’existe pas, il sera automatiquement créé. |
| `-s /bin/sh` | **Shell par défaut** | Définit quel interpréteur de commande (shell) sera utilisé par défaut. Ici, `/bin/sh` est choisi (shell standard et léger). |
| `<utilisateur>` | **Nom de l’utilisateur** | Le nom du compte utilisateur à créer. |


Définir un mot de passe :
```bash
passwd <utilisateur>
```

Ajouter l’utilisateur à un groupe :
```bash
usermod -a -G <groupe> <utilisateur>
```
