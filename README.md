# mt-registry

> Registry Composer privé de l'écosystème **martaf-labs**.
> Point d'entrée unique pour installer et gérer tous les packages `mt-*` dans vos projets Laravel 12+.

---

## Table des matières

- [Présentation](#présentation)
- [Packages disponibles](#packages-disponibles)
- [Installation (première fois)](#installation-première-fois)
- [Créer un nouveau projet](#créer-un-nouveau-projet)
- [Ajouter un module à un projet existant](#ajouter-un-module-à-un-projet-existant)
- [Fonctionnement interne](#fonctionnement-interne)
- [Administration du registry](#administration-du-registry)
- [Contribuer un nouveau package](#contribuer-un-nouveau-package)
- [FAQ](#faq)

---

## Présentation

`mt-registry` est le **registry Composer privé** de l'écosystème martaf-labs.

Il joue le même rôle que [Packagist](https://packagist.org) (le registry public de Composer), mais pour les packages privés `mt-*`. Grâce à lui, vous pouvez installer n'importe quel package de l'écosystème avec une simple commande `composer require`, exactement comme un package public.

```
mt-registry
│
├── martaf-labs/mt-skeleton     → Skeleton de démarrage de projet
├── martaf-labs/mt-foundation   → Socle commun obligatoire
├── martaf-labs/mt-views        → Composants Blade & layouts
├── martaf-labs/mt-icons        → Système d'icônes SVG
├── martaf-labs/mt-blog         → Module blog
├── martaf-labs/mt-school       → Module scolaire
└── martaf-labs/mt-docs         → Documentation en ligne
```

**URL du registry :** `https://martaf-labs.github.io/mt-registry`

---

## Packages disponibles

| Package | Version | Description |
|---------|---------|-------------|
| `martaf-labs/mt-skeleton` | `dev-main` | Skeleton de démarrage avec installateur interactif |
| `martaf-labs/mt-foundation` | `dev-main` | Socle obligatoire : helpers, traits, config globale |
| `martaf-labs/mt-views` | `dev-main` | Composants Blade réutilisables et layouts de base |
| `martaf-labs/mt-icons` | `dev-main` | Système d'icônes SVG intégré |
| `martaf-labs/mt-blog` | `dev-main` | Blog complet : articles, catégories, tags, auteurs |
| `martaf-labs/mt-school` | `dev-main` | Gestion scolaire : élèves, classes, notes, bulletins |
| `martaf-labs/mt-docs` | `dev-main` | Documentation en ligne au format Markdown |

---

## Installation (première fois)

> À effectuer **une seule fois** sur chaque machine de développement.

### Prérequis

- PHP **8.2+**
- Composer **2.x**
- Node.js **18+**
- Un compte GitHub membre de l'organisation **martaf-labs**

---

### Étape 1 — Générer un token GitHub

Rendez-vous sur GitHub :
**Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token**

Cochez la permission **`repo`** (accès en lecture aux repos privés), puis copiez le token généré.

> ⚠️ Le token ne s'affiche qu'une seule fois. Conservez-le en lieu sûr.

---

### Étape 2 — Enregistrer le token dans Composer

```bash
composer config --global github-oauth.github.com VOTRE_TOKEN_ICI
```

Cela enregistre le token dans `~/.composer/auth.json`.
Composer l'utilisera automatiquement pour accéder aux packages privés.

---

### Étape 3 — Déclarer mt-registry dans Composer

```bash
composer config --global repositories.martaf \
  '{"type":"composer","url":"https://martaf-labs.github.io/mt-registry"}'
```

Cela ajoute le registry dans `~/.composer/config.json`.
Composer consultera mt-registry en plus de Packagist pour résoudre les dépendances.

---

### Vérification

```bash
composer config --global --list | grep martaf
```

Vous devriez voir l'URL du registry apparaître dans la liste.

---

## Créer un nouveau projet

Une fois le setup effectué, créez un projet en une seule commande :

```bash
composer create-project martaf-labs/mt-skeleton nom-du-projet --stability=dev
cd nom-du-projet
```

L'installateur interactif se lance automatiquement :

```
Quel type de projet souhaitez-vous créer ?
  [0] Minimal (foundation uniquement)
  [1] Site Blog
  [2] Application Scolaire
  [3] Écosystème complet
  [4] Personnalisé
```

### Exemples concrets

```bash
# Application scolaire
composer create-project martaf-labs/mt-skeleton mon-ecole --stability=dev

# Site blog
composer create-project martaf-labs/mt-skeleton mon-blog --stability=dev

# Avec préset directement (sans menu interactif)
php artisan mt:install --preset=school
```

### Après la création du projet

```bash
# 1. Configurer l'environnement
cp .env.example .env
php artisan key:generate

# 2. Renseigner la base de données dans .env
# DB_CONNECTION=mysql
# DB_DATABASE=nom_de_la_base

# 3. Créer la base de données
mysql -u root -e "CREATE DATABASE nom_de_la_base CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# 4. Lancer les migrations
php artisan migrate

# 5. Installer les assets
npm install && npm run dev

# 6. Démarrer le serveur
php artisan serve
```

---

## Ajouter un module à un projet existant

```bash
# Via Composer
composer require martaf-labs/mt-blog

# Puis via la commande Artisan du skeleton
php artisan mt:module blog
```

La commande `mt:module` publie automatiquement les configs, migrations, vues et routes du module.

---

## Fonctionnement interne

```
Push sur n'importe quel package mt-*
              ↓
   notify-registry.yml se déclenche
   (GitHub Action dans le package)
              ↓
   Envoie un événement "satis-update"
   au repo martaf-labs/mt-registry
              ↓
   satis.yml se déclenche
   (GitHub Action dans mt-registry)
              ↓
   Satis reconstruit le registry
   (packages.json + archives zip)
              ↓
   Déploiement automatique sur GitHub Pages
              ↓
   ✓ Registry à jour en ~2 minutes
```

---

## Administration du registry

> Réservé aux membres de l'organisation martaf-labs.

### Reconstruire le registry manuellement

Dans l'onglet **Actions** du repo `martaf-labs/mt-registry` :
→ **Build & Deploy Registry** → **Run workflow**

### Ajouter un nouveau package

1. Créer le repo sur GitHub dans l'organisation `martaf-labs`
2. Ajouter son entrée dans `satis.json` :

```json
{
    "type": "vcs",
    "url": "https://github.com/martaf-labs/mt-nouveau-package"
}
```

3. Copier `.github/workflows/notify-registry.yml` dans le nouveau package
4. Ajouter le secret `SATIS_DISPATCH_TOKEN` dans le nouveau repo (ou utiliser l'Organization Secret)
5. Pousser sur `main` → le registry se reconstruit automatiquement

### Supprimer un package

1. Retirer son entrée de `satis.json`
2. Pousser sur `main`

### Secrets GitHub requis

**Dans le repo `martaf-labs/mt-registry` :**

| Secret | Description |
|--------|-------------|
| `SATIS_GITHUB_TOKEN` | Token avec accès `repo` à tous les packages privés martaf-labs |

**Dans chaque repo `mt-*` :**

| Secret | Description |
|--------|-------------|
| `SATIS_DISPATCH_TOKEN` | Token avec accès `repo` sur `martaf-labs/mt-registry` |

> 💡 **Recommandé :** Configurez `SATIS_DISPATCH_TOKEN` comme **Organization Secret** pour qu'il soit disponible automatiquement dans tous les repos `mt-*`.
>
> **GitHub → martaf-labs (org) → Settings → Secrets and variables → Actions → New organization secret**

---

## Contribuer un nouveau package

### Structure minimale requise

```
mt-nouveau-package/
├── composer.json
├── README.md
└── .github/
    └── workflows/
        └── notify-registry.yml   ← copié depuis ce repo
```

### `composer.json` minimal

```json
{
    "name": "martaf-labs/mt-nouveau-package",
    "description": "Description du package",
    "type": "library",
    "license": "MIT",
    "require": {
        "php": "^8.2",
        "martaf-labs/mt-foundation": "dev-main"
    },
    "extra": {
        "laravel": {
            "providers": [
                "MtNouveauPackage\\NouveauPackageServiceProvider"
            ]
        }
    }
}
```

---

## FAQ

**Le registry n'est pas à jour après mon push ?**
Le build prend ~2 minutes. Vérifiez l'onglet **Actions** du repo `mt-registry`.

**J'obtiens une erreur 401 lors d'un `composer install` ?**
Votre token GitHub est expiré. Régénérez-en un et relancez :
```bash
composer config --global github-oauth.github.com NOUVEAU_TOKEN
```

**Comment passer de `dev-main` à une version stable ?**
Créez un tag Git dans le package :
```bash
git tag v1.0.0 && git push origin v1.0.0
```
Puis mettez à jour la contrainte dans votre projet :
```bash
composer require martaf-labs/mt-foundation:^1.0
```

**Comment tester un package en cours de développement sans passer par le registry ?**
Utilisez un `path` repository dans votre `composer.json` local :
```json
{
    "repositories": [
        {
            "type": "path",
            "url": "../mt-mon-package",
            "options": { "symlink": true }
        }
    ]
}
```

---

## Structure du repo

```
mt-registry/
├── satis.json                              # Liste des packages du registry
├── README.md                               # Ce fichier
├── .github/
│   └── workflows/
│       ├── satis.yml                       # Build & deploy du registry
│       └── notify-registry.yml            # À copier dans chaque package mt-*
└── public/                                 # Généré automatiquement — ne pas modifier
    ├── index.html
    ├── packages.json
    └── dist/                               # Archives zip des packages
```

---

## Licence

Usage interne — [martaf-labs](https://github.com/martaf-labs)
