# Waterline

Waterline is a self-built training app that turns a specific gym's equipment inventory into a structured weekly program, with set-level logging and progressive overload built into the interface.

**Live app:** https://khushi-2031.github.io/Waterline/

**Test credentials** (or create your own account via Sign Up):
- Username: `demo`
- Password: `demo1234`

---

## What it does

1. **Sign up / Log in / Log out** — each user has their own account and their own private training log, stored in their browser.
2. **Onboarding survey** (shown once, first login) — captures current weight, goal weight, ideal weight, primary goal, training days per week, injuries/limitations, up to 5 gym-equipment photos, and any additional comments. Editable anytime from the Profile tab.
3. **Core flow** — a 6-day training plan (Mon–Sat, Sunday off) built around the equipment available at one specific gym. Each day you check off sets as you complete them, log the weight used, track body weight, and see a weekly attendance strip and streak counter.
4. **Full CRUD on exercises** — beyond logging the built-in program, you can **create** your own custom exercises inside any block, **read** them alongside the program, **update** their name/sets/reps, and **delete** them — so the plan adapts to what you actually do in the gym.
5. **Progressive overload tracking** — the weight you log per exercise is stored per date, not just as a single overwritten value, so each weighted exercise shows a trend (e.g. "+2.5kg since Aug 24") and an expandable history of the last 10 logged dates/weights. Available on built-in weighted exercises and, optionally, on custom exercises too.

## Setup instructions

This is a static, single-file frontend app — no build step, no server, no external services required.

### Run it locally
```bash
git clone https://github.com/Khushi-2031/Waterline.git
cd Waterline
python3 -m http.server 8000
# open http://localhost:8000/index.html
```
Or simply double-click `index.html` to open it directly in a browser (some browsers restrict `localStorage` on `file://` URLs — a local server avoids that).

### Deploy your own copy
1. Fork or clone this repo.
2. In GitHub → Settings → Pages, set the source to the `main` branch, root folder.
3. Your app will be live at `https://<your-username>.github.io/Waterline/` within a minute or two.

Also deployable to Vercel by importing the GitHub repo directly (Framework Preset: "Other", no build command) — no config needed either way.

No environment variables, API keys, or backend services are required.

## Architecture & key decisions

- **Single HTML file** (`index.html`) — all markup, CSS, and JavaScript in one file, so it deploys as-is to GitHub Pages or Vercel with zero build tooling.
- **Frontend-only, no backend** — authentication and all user data live in the browser's `localStorage`, namespaced per username (`waterline:data:<username>`). This was a deliberate trade-off for a fast, zero-infrastructure deployment; the trade-off is that data does not sync across devices/browsers (a "Back up / Restore" JSON export-import feature is included to move data manually).
- **Auth model**: passwords are hashed client-side with SHA-256 (`crypto.subtle`) before being stored — this avoids plaintext storage, but is explicitly **not** production-grade security (no server-side salting/verification, and anyone with access to the browser's storage can read the account list). It's appropriate for a demo/prototype, not for real user data.
- **Gym equipment mapping**: since this is a static frontend with no AI/vision backend, gym photos are stored for reference (downscaled client-side via `<canvas>` before saving, to keep `localStorage` usage small) and equipment is manually catalogued in a checklist (Gear tab) rather than auto-detected from the photos.
- **CRUD entity**: the core data entity is the *exercise log entry*. Built-in program exercises support logging (set completion + optional dated weight-used history); user-added custom exercises support full create/read/update/delete, optionally with the same weight-history tracking.
- **Progressive overload**: weight used per exercise is stored per calendar date (not just the latest value), so the app can show a trend and a short history rather than silently overwriting a single field.

## Tools & stack used

- **Vanilla HTML/CSS/JavaScript** — no framework, no bundler.
- **Web Crypto API** (`crypto.subtle.digest`) for password hashing.
- **Canvas API** for client-side image downscaling of gym photos before storage.
- **localStorage** for all persistence (accounts, sessions, per-user training data).
- **GitHub Pages / Vercel** for static hosting/deployment.
- **GitHub CLI (`gh`) / git** for repository and deployment management.
- **Claude (Anthropic)** was used as an AI pair-programmer to design and implement the authentication flow, onboarding survey, CRUD refactor for custom exercises, and progressive-overload weight tracking on top of the original Waterline app.
