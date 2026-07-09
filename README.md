# Devinez le prénom

Jeu à partager avec famille et amis pour deviner le prénom d'un nouveau-né.
Les prénoms s'affrontent en duels 1v1 (tournoi à élimination directe) jusqu'au
dernier restant, puis le participant publie sa prédiction — visible par tous
les autres sur le site.

## Comment ça marche

- **Site statique** : un seul fichier `index.html`, aucun serveur à gérer.
- **Page de création** : ouvrir le site sans paramètre → saisir le titre et
  la liste de prénoms → un lien de jeu est généré.
- **Page de jeu** : quiconque ouvre le lien joue le tournoi.
- **Administration** (mode en ligne) : le navigateur qui a créé le jeu est
  reconnu automatiquement (authentification anonyme Firebase) et dispose d'un
  bouton « Modifier la liste » sur la page du jeu — titre, email et prénoms
  sont modifiables à tout moment, sans changer le lien partagé ni perdre les
  prédictions déjà publiées.
- **Prédictions partagées** : à la fin, le participant publie sa prédiction
  (son nom, le prénom choisi, un mot pour les futurs parents). Tout le monde
  peut consulter la liste des prédictions et le décompte des voix par prénom.
- **Une prédiction par navigateur** : rejouer puis republier remplace sa
  prédiction précédente au lieu d'en ajouter une nouvelle.

## Deux modes

| | Mode en ligne (recommandé) | Mode email (fallback) |
|---|---|---|
| Prédictions visibles par tous | ✅ | ❌ |
| Liste modifiable après partage | ✅ (même lien) | ❌ (nouveau lien) |
| Lien de partage | Court (`#j=…`) | Long (config encodée) |
| Configuration | Projet Firebase gratuit (~5 min) | Aucune |
| Réception des réponses | Sur le site | Email pré-rempli vers le créateur |

Le mode est automatique : si `FIREBASE_CONFIG` est renseigné dans `index.html`,
le mode en ligne est actif ; sinon le jeu retombe sur l'envoi par email.

## Activer les prédictions partagées (Firebase, gratuit)

1. Aller sur https://console.firebase.google.com → **Créer un projet**
   (nom libre, Google Analytics inutile).
2. Dans le projet : **Build → Firestore Database → Créer une base de données**
   → mode **production**, région `europe-west` de préférence.
3. Onglet **Règles** de Firestore : remplacer le contenu par celui du fichier
   [`firestore.rules`](firestore.rules) de ce dépôt, puis **Publier**.
   (Ces règles n'autorisent que des prédictions valides ; chaque navigateur a
   une seule prédiction par jeu, qu'il peut remplacer en rejouant — jamais supprimer.)
4. **Build → Authentication → Get started** → onglet **Sign-in method** →
   activer **Anonyme**. (C'est ce qui permet de reconnaître le créateur d'un
   jeu pour qu'il puisse modifier sa liste — aucun compte à créer pour personne.)
5. Paramètres du projet (roue dentée) → **Vos applications → Web (`</>`)** →
   enregistrer l'app (pas besoin de hosting) → copier l'objet `firebaseConfig`.
6. Dans `index.html`, remplacer `const FIREBASE_CONFIG = null;` par
   `const FIREBASE_CONFIG = { ...votre config... };`
7. Redéployer le fichier. C'est tout !

> La `apiKey` Firebase n'est pas un secret : elle identifie le projet, la
> sécurité est assurée par les règles Firestore de l'étape 3.

## Tester en local

Ouvrir simplement `index.html` dans un navigateur (double-clic suffit).

## Héberger gratuitement

**Option 1 — Netlify Drop (le plus simple, 30 secondes)**
1. Aller sur https://app.netlify.com/drop
2. Glisser-déposer le dossier du projet
3. C'est en ligne — l'URL est personnalisable dans les réglages du site.
   Pour mettre à jour : « Deploys » → re-glisser le dossier.

**Option 2 — GitHub Pages**
1. Créer un dépôt GitHub et pousser `index.html`
2. Settings → Pages → Source : branche `main`
3. Le site est servi sur `https://<user>.github.io/<repo>/`

**Option 3 — Vercel / Cloudflare Pages** : importer le dossier, aucun build nécessaire.

## Modifier les prénoms après coup

- **Mode en ligne** : ouvrir le lien du jeu depuis le navigateur qui l'a créé →
  bouton « Modifier la liste » sur la page d'accueil du jeu. Le lien partagé
  reste le même et les prédictions sont conservées. Attention : les droits
  d'administration sont liés au navigateur (stockage local) — utiliser un autre
  appareil ou effacer les données du site les fait perdre.
- **Mode email** (lien long `#g=…`) : la liste vit dans l'URL, il faut donc
  générer un nouveau lien via la page de création. Un nouveau lien = un nouveau
  jeu.
