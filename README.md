# Waterline

Waterline is a self-built training app that turns a specific gym's equipment inventory into a structured weekly program, with set-level logging and progressive overload built into the interface.

**Live app:** https://khushi-2031.github.io/Waterline/

**Test credentials:**
- Email/password: create your own account via Sign Up, or
- **Google**: click "Continue with Google"
- **Phone**: use the Firebase test number configured in the project (e.g. `+1 650 555 3434`, code `123456`) — see setup below

---

## What it does

1. **Sign up / Log in / Log out** — via email+password, Google Sign-In, or phone number + SMS one-time code (all through Firebase Authentication). Each user has their own account and their own private training log, synced through Firestore.
2. **Onboarding survey** (shown once, first login) — captures current weight, goal weight, ideal weight, primary goal, training days per week, injuries/limitations, up to 5 gym-equipment photos, and any additional comments. Editable anytime from the Profile tab.
3. **Core flow** — a 6-day training plan (Mon–Sat, Sunday off) built around the equipment available at one specific gym. Each day you check off sets as you complete them, log the weight used (with dated progressive-overload history), track body weight, and see a weekly attendance strip and streak counter.
4. **Full CRUD on exercises** — beyond logging the built-in program, you can **create** your own custom exercises inside any block, **read** them alongside the program, **update** their name/sets/reps, and **delete** them — so the plan adapts to what you actually do in the gym.

## Setup instructions

This is a static single-file frontend (`index.html`) backed by **Firebase** (Authentication + Firestore) for accounts and data.

### 1. Create your own Firebase project
1. Go to [Firebase Console](https://console.firebase.google.com) → **Add project**.
2. **Authentication → Sign-in method**: enable **Email/Password**, **Google**, and **Phone**.
3. **Authentication → Settings → Authorized domains**: add your GitHub Pages domain (e.g. `<you>.github.io`) and any other domain you deploy to (e.g. a Vercel URL).
4. **Firestore Database → Create database** (production mode), then under **Rules**, publish:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{userId} {
         allow read, write: if request.auth != null && request.auth.uid == userId;
       }
     }
   }
   ```
5. **Project Settings → Your apps → Web app**: register an app and copy the `firebaseConfig` object.
6. (Optional, for demoing phone login without real SMS) **Authentication → Sign-in method → Phone → Phone numbers for testing**: add a test number and code.

### 2. Wire the config into the app
Open `index.html` and replace the `FIREBASE_CONFIG` object near the top of `<head>` with the config values from step 1.5. (Already done in this repo's deployed copy.)

### 3. Run it locally
```bash
git clone https://github.com/Khushi-2031/Waterline.git
cd Waterline
python3 -m http.server 8000
# open http://localhost:8000/index.html
```

### 4. Deploy
Push to GitHub, then in **Settings → Pages**, set the source to the `main` branch, root folder. Live within a minute or two at `https://<your-username>.github.io/Waterline/`. (Also deployable to Vercel by importing the GitHub repo directly — no build step needed.)

## Architecture & key decisions

- **Single HTML file** (`index.html`) — all markup, CSS, and JavaScript in one file; deploys as-is to GitHub Pages or Vercel with zero build tooling.
- **Firebase Authentication** — three sign-in methods: email/password, Google OAuth (`signInWithPopup`), and phone number with SMS one-time codes (`signInWithPhoneNumber` + reCAPTCHA). All three converge on the same `onAuthStateChanged` listener, which is the single source of truth for whether the app shows the login screen, the onboarding survey, or the dashboard.
- **Firestore as the database** — replaced the earlier `localStorage`-only prototype. Each user's training data (history, weight log, survey, custom exercises) lives in one document at `users/{uid}`, so it syncs across devices and browsers instead of being trapped in one browser's local storage. Security rules restrict every document to being read/written only by the user who owns it.
- **Gym equipment mapping**: no AI/vision backend is wired in, so gym photos are stored for reference (downscaled client-side via `<canvas>` before saving) and equipment is manually catalogued in a checklist (Gear tab) rather than auto-detected from the photos.
- **CRUD entity**: the core data entity is the *exercise log entry*. Built-in program exercises support logging (set completion + optional dated weight-used history); user-added custom exercises support full create/read/update/delete, optionally with the same weight-history tracking.
- **Progressive overload**: weight used per exercise is stored per calendar date (not just the latest value), so the app can show a trend (e.g. "+2.5kg since Aug 24") and a short history, rather than overwriting a single field.

## Tools & stack used

- **Vanilla HTML/CSS/JavaScript** — no framework, no bundler.
- **Firebase Authentication** (compat SDK, `firebase-auth-compat.js`) — Google OAuth, phone/SMS OTP, email/password.
- **Firebase Firestore** (compat SDK, `firebase-firestore-compat.js`) — per-user document storage.
- **Canvas API** for client-side image downscaling of gym photos before storage.
- **GitHub Pages / Vercel** for static hosting/deployment.
- **git / GitHub CLI (`gh`)** for repository and deployment management.
- **Claude (Anthropic)** was used as an AI pair-programmer throughout — designing the authentication flow, the onboarding survey, the CRUD refactor for custom exercises, progressive-overload weight tracking, and the migration from a localStorage prototype to Firebase Authentication + Firestore.

## Known limitations

- Gym equipment tagging from photos is manual, not automatic (no AI vision backend in this deployment).
- Phone authentication requires a reCAPTCHA challenge (Firebase's requirement to prevent SMS abuse) — this is expected, not a bug.
- The Firestore security rules here are intentionally simple (owner-only access); a production app would likely add field-level validation rules too.
