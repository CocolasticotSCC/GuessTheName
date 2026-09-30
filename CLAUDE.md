# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

« Devinez le prénom » — a baby-name guessing game shared with family/friends. Names face off in 1v1 single-elimination duels until one remains; the player then publishes their prediction, visible to everyone. UI text, code comments, and commit messages are in **French**.

## Architecture

The entire app is **one file: `index.html`** (CSS + HTML screens + vanilla JS). No build step, no dependencies, no tests, no framework. To test locally, just open `index.html` in a browser.

Key structural facts:

- **Screens**: `<div class="screen">` blocks (`screen-admin`, `screen-intro`, `screen-loading`, `screen-game`, `screen-final`, `screen-results`), toggled by `show(id)`. Routing is hash-based via `initFromHash()` (also fired on `hashchange`): no hash → creation screen; `#j=<gameId>` → online game (new games use a 7-char id from `newGameId()`; older links carry 20-char Firestore auto-ids); `#g=<base64url>` → offline game with full config encoded in the URL.
- **Two modes**, decided by the `FIREBASE_CONFIG` constant at the top of the `<script>`:
  - **Online** (config present): games stored in Firestore `games` collection (short link, editable list), predictions in `guesses`. Firebase SDK (v10, `firebase-app` / `firebase-firestore-lite` / `firebase-auth`) is loaded lazily via dynamic `import()` from gstatic CDN.
  - **Offline fallback**: game config base64url-encoded in the link; predictions are copy-pasted by players. Every online feature must keep degrading gracefully to this mode.
- **Identity without accounts**:
  - Game creator = anonymous Firebase auth; `games.owner` uid grants the "Modifier la liste" button (admin rights are tied to that browser's local storage).
  - One prediction per browser: `localStorage` device id, and `guesses` doc id = `<gameId>_<deviceId>` so republishing overwrites (`setDoc`). localStorage keys `gtn-device`, `gtn-prono-<gameId>` and `gtn-who` are relied on by existing players — don't rename them.
- **Themes**: 4 themes (`dore` default, `poudre`, `celeste`, `nuit`) implemented as CSS variable overrides on `body[data-theme]`; stored in game config as `th`.
- **Security**: all user content is inserted via `textContent`/`createTextNode` (never `innerHTML`) — keep it that way. The Firebase `apiKey` in `index.html` is public by design; security relies entirely on `firestore.rules`.

## Firestore rules ↔ code coupling

`firestore.rules` validates the exact key sets written by `index.html` (`games`: `hasOnly(['t','n','e','l','th','vp','owner','ts'])` — `vp` = players may see predictions after finishing a run (absent = true); the gate is client-side only, `canSeeResults()`; `guesses`: `hasOnly(['g','who','name','top','msg','ts'])`, `top` = the player's last 4 names in order, used for the points ranking). `comments` (per-name comments, online only): `hasOnly(['g','name','who','msg','ts','hidden'])`; only the game owner may update, and only `hidden` (moderation — hidden comments are filtered client-side, still readable via the API). If you add/remove a field in `createGame()`/`saveEdit()`/`saveRun()`, you must update the rules **and deploy them**, or writes will be rejected. Note: `e` (email) is a leftover from the removed email feature (commit 171a76b) — it's optional in the rules only so that old game docs that still have it stay valid when edited. Rules changes are not live until deployed:

```bash
firebase deploy --only firestore:rules   # project: guessthename-ot-467d6 (.firebaserc)
```

(or paste into the Firebase console → Firestore → Règles). Deletes: never for `games`; for `guesses`/`comments` only by the game owner (the "Tout effacer" reset in the edit screen, `resetGameData()`). Also, `guesses` updates can't change their `g` (game id).

## Deployment

Static hosting on GitHub Pages at `https://guessthename.gastory.fr/` (custom domain via the `CNAME` file, DNS at OVH; old `cocolasticotscc.github.io/GuessTheName/` links redirect there), served from `main` — pushing to `main` deploys. No build. `og-image.png` is the link-preview image; its `og:image` URL in `index.html` is absolute and must be updated if the domain changes. (The README also documents Netlify Drop / Vercel.) `firebase.json` exists only for deploying Firestore rules (no Firebase Hosting).

## Backward compatibility

Shared links live in people's messages forever. Don't break parsing of existing `#j=` and `#g=` links, including old `#g=` configs without an `id` field (handled by `simpleHash`).
