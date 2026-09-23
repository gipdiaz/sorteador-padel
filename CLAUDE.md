# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

"Sorteador de Parejas de Padel" is a single-page app that randomly pairs 4 drive players with 4 backhand (revés) players and draws 2 matchups for a padel "americano". The whole app lives in `index.html` (markup, a small `<style>` block, and inline JavaScript). The only other asset is `images/home.png`, used on the landing page. UI text and identifiers are in Spanish (Rioplatense).

## Running / Deploying

- No build, package manager, linter, or tests. Open `index.html` in a browser, or serve the folder statically.
- Google sign-in (`signInWithPopup`) only works on domains authorized in the Firebase console. Opening the file via `file://` won't allow login, so serve it from `localhost` (authorized by default) or test on the deployed site.
- Deployed via GitHub Pages at https://gipdiaz.github.io/sorteador-padel/ (served straight from `main`).

## Architecture

- **Libraries (CDN only):** Bootstrap 5.3.0 (CSS + bundle JS, used for the share modal), and the Firebase **compat** SDK v10.7.1 (`firebase-app/auth/database-compat`). Use the namespaced `firebase.*` API (`database.ref(...)`, `auth.onAuthStateChanged`), not the modular v9+ imports.
- **Firebase config** is the real `sorteador-padel` project config, inline at the top of the script. Web API keys are public by design.
- **Screens:** `#landingPage` (signed out) and `#mainContent` (signed in) are toggled by `auth.onAuthStateChanged`. Both branches call `verificarSorteoCompartido()`, which checks for a `?sorteo=<id>` query param. If one is present, `mostrarSorteoCompartido()` switches `#mainContent` into a read-only view: it hides inputs, the draw button, and the profile, and replaces `#shareContainer` with a "Crear mi propio sorteo" link. This makes shared links viewable without logging in.
- **Draw flow (`sortear()`):**
  1. Requires a signed-in user and all 8 fields filled.
  2. Builds a duplicate key from the *sorted* drive names + sorted revés names joined with `-`. If that key is already in `sorteosAnteriores` (loaded from `historialDia`, filtered to the last 24h), it shows the joke warning in every `.mensaje-repetido` element, increments the count, and **aborts without drawing**. Otherwise it records the key and continues.
  3. Saves names to `users/{uid}/players`, which feeds the `#jugadoresList` datalist autocomplete.
  4. Shuffles with `sort(() => Math.random() - 0.5)`, pairs `drive[i]` with `reves[i]`, then shuffles the pairs into 2 matchups. Pairs and matchups are stored as display strings (`"A y B"`, `"A y B vs C y D"`).
  5. Keeps the result in `ultimoSorteoId`. The name is misleading: it holds the draw data, not an id. `compartirSorteo()` pushes it to `sorteos/{pushKey}` and shows the link in `#shareModal`.
- Firebase keys can't contain `. # $ [ ] /`, so a player name with any of these breaks the `historialDia` write. Old `historialDia` entries are only filtered on read and never deleted.

### Firebase Realtime Database layout

```
users/{uid}/players                 # array of unique player names (autocomplete)
users/{uid}/historialDia/{key}      # { count, timestamp }, key = sorted drive + sorted revés names
sorteos/{pushId}                    # public shared draw:
                                    #   owner, ownerName, jugadoresDrive, jugadoresReves,
                                    #   parejas, enfrentamientos, created
```

`sorteos/*` must be publicly readable for shared links to work without auth. Security rules live in the Firebase console, not in this repo.
