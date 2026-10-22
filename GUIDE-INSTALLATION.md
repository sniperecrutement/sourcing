# Snipe Sourcing — installation sur Netlify

Environ 20 minutes, une seule fois. Les recruteurs se connectent ensuite avec leur courriel @sniperecrutement.com et un NIP, sans compte Claude ni Google.

## Ce que contient le dossier

| Fichier | Rôle |
|---|---|
| `public/index.html` | L'application (même interface que la version Claude) |
| `netlify/functions/auth.mjs` | Connexion courriel + NIP, limitée à @sniperecrutement.com |
| `netlify/functions/ghl.mjs` | Relais vers GHL. La clé GHL reste sur le serveur, jamais dans le navigateur |
| `netlify/functions/db.mjs` | Accès, projets, refus de suggestions (stockés dans Netlify Blobs) |
| `netlify/functions/users.mjs` | Noms et photos des collègues pour « Gérer les accès » |
| `netlify.toml`, `package.json` | Réglages Netlify |

---

## Étape 1 — Clé GHL (Private Integration)

1. GHL → **Paramètres** → **Private Integrations** → **Create new integration**.
2. Nom : `Snipe Sourcing`.
3. Coche ces permissions (scopes) :
   - `contacts.readonly`, `contacts.write`
   - `opportunities.readonly`, `opportunities.write`
   - `locations/customFields.readonly`
   - `conversations.readonly`, `conversations/message.readonly`
   - `associations.readonly`, `associations/relation.readonly`, `associations/relation.write`
4. Copie la clé (elle commence par `pit-`). Garde-la pour l'étape 4. **Ne l'envoie à personne.**

## Étape 2 — Mettre le code sur GitHub (privé)

Netlify a besoin de GitHub pour installer les fonctions serveur. Le glisser-déposer simple ne les prend pas.

1. Crée un compte sur github.com si tu n'en as pas, avec info@sniperecrutement.com.
2. **New repository** → nom `snipe-sourcing` → coche **Private** → **Create repository**.
3. Clique **uploading an existing file**, puis glisse **le contenu** du dossier dézippé : les dossiers `public`, `netlify`, `src` et les fichiers `netlify.toml`, `package.json`, `GUIDE-INSTALLATION.md`.
4. **Commit changes**.

## Étape 3 — Créer le site Netlify relié à GitHub

1. Netlify → **Add new site** → **Import an existing project** → **GitHub** → autorise → choisis `snipe-sourcing`.
2. Ne touche à rien dans les réglages de build (le fichier `netlify.toml` s'en occupe) → **Deploy**.
3. Note l'adresse du site, ex. `https://snipe-sourcing-xyz.netlify.app`. Tu peux la renommer dans **Site configuration → Change site name**.
4. Supprime l'ancien site fait par glisser-déposer : **Site configuration → Delete site**.

## Étape 4 — Variables secrètes dans Netlify

Netlify → ton site → **Site configuration** → **Environment variables** → **Add a variable**, une par une :

| Nom | Valeur |
|---|---|
| `GHL_TOKEN` | la clé GHL de l'étape 1 (`pit-…`) |
| `GHL_LOCATION_ID` | `vYQWjTK5DcPsvZjuhyux` |
| `SESSION_SECRET` | une longue phrase aléatoire d'au moins 40 caractères (générateur de mot de passe) |
| `ADMIN_EMAILS` | `info@sniperecrutement.com` (ajoute d'autres admins séparés par une virgule) |
| `ALLOWED_DOMAIN` | `sniperecrutement.com` |

Ensuite : **Deploys** → **Trigger deploy** → **Deploy site**. Les variables ne s'appliquent qu'après un nouveau déploiement.

## Étape 5 — Premier lancement

1. Ouvre le site, tape **info@sniperecrutement.com**, laisse le NIP vide et clique **Se connecter**. L'app te demande de **choisir ton NIP** (6 à 12 chiffres). Fais-le tout de suite après le déploiement. Tu es Admin.
2. Clique **Synchroniser** pour charger les candidats. La première fois prend environ 1 minute.
3. **⋯ → Recharger mandats et opportunités.**
4. **⋯ → Charger toutes les notes** (une fois, environ 15 à 20 minutes ; tu peux travailler pendant ce temps).
5. **⋯ → Accès et permissions** : tape le courriel d'un recruteur (ex. `caroline@sniperecrutement.com`), puis **+ Recruteur**. Un **NIP temporaire** s'affiche : clique **Copier le message** et envoie-le-lui. À sa première connexion, il choisit son propre NIP.

**NIP oublié :** ⋯ → Accès et permissions → **Nouveau NIP** à côté de la personne. **Changer son NIP :** ⋯ → Changer mon NIP.

## Sécurité

- **Accès :** seuls les courriels @sniperecrutement.com que tu as ajoutés dans « Accès » peuvent entrer.
- **NIP :** jamais stocké en clair (haché avec scrypt). 5 essais ratés bloquent le compte 15 minutes. Les NIP trop simples (111111, 123456…) sont refusés.
- **Clé GHL :** elle n'est jamais envoyée au navigateur.
- **Opérations GHL :** seules celles dont l'app a besoin sont permises. Supprimer un contact, par exemple, est impossible.
- **Permissions des recruteurs :** elles sont vérifiées aussi sur le serveur (associer, modifier les notes, voir les discussions…).
- **Champs sensibles :** NAS, spécimen chèque, date de naissance, salaire, antécédents et permis ne sont jamais chargés.
- **Moteurs de recherche :** le site est marqué « noindex » et n'y apparaît pas.
- **Session :** elle dure 12 h, puis on se reconnecte.

## Mises à jour

Pour une nouvelle version : sur GitHub, remplace le fichier modifié (ex. `public/index.html` → **Add file → Upload files**). Netlify redéploie tout seul en environ 1 minute.

## Problèmes fréquents

- **« Trop d’essais »** : attends 15 minutes, ou génère un nouveau NIP à la personne.
- **« Erreur GHL : … 401 »** : la clé GHL est fausse ou il manque une permission (étape 1).
- **Un recruteur voit « Accès en attente »** : ajoute-le dans « Accès », puis il recharge la page.
