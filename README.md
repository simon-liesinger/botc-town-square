# BotC Town Square

Static web app — a Blood on the Clocktower town-square / grimoire that runs entirely in the
browser. Tap **+** to add seats, tap a seat to assign a character, long-press a seat to add
notes. Optional Firebase sync shares just liveness + ghost-vote state across devices.

## Deploy to GitHub Pages

```
cd botc-grimoire-web
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/<you>/botc-town-square.git
git push -u origin main
```

Then on the GitHub repo: **Settings → Pages → Source: Deploy from a branch → main / root → Save**.
URL: `https://<you>.github.io/botc-town-square/`.

## Optional: enable cross-device sync

The app works offline (localStorage) out of the box. To sync **dead / ghost-vote** state
across multiple phones in the same room:

1. Go to <https://console.firebase.google.com>, **Add project** (any name).
2. In the project, **Build → Realtime Database → Create Database**.
   - Pick any region.
   - Start in **test mode** (or paste the rules below afterwards).
3. **Database → Rules** — paste:
   ```json
   {
     "rules": {
       "rooms": {
         "$code": {
           ".read":  "$code.matches(/^[A-Z0-9]{4,12}$/)",
           ".write": "$code.matches(/^[A-Z0-9]{4,12}$/)"
         }
       }
     }
   }
   ```
   This restricts reads/writes to `rooms/<CODE>` paths only.
4. **Project Settings → General → Your apps → Add app → Web** (the `</>` icon). Skip Hosting.
   Copy the `firebaseConfig` object it shows you.
5. Open `index.html` and paste the values into the `FIREBASE_CONFIG` block near the top of the
   `<script>` tag:
   ```js
   const FIREBASE_CONFIG = {
     apiKey:      "AIza...",
     authDomain:  "your-project.firebaseapp.com",
     databaseURL: "https://your-project-default-rtdb.firebaseio.com",
     projectId:   "your-project",
     appId:       "1:...:web:..."
   };
   ```
6. Commit and push. Once deployed, a **ROOM** field appears in the header.
   - Tap **New** to generate a writer code (10 characters). The viewer code
     (first 6 characters) appears below — share that with your players.
   - **Writer code** holders see the full UI (dead/ghost/name controls).
   - **Viewer code** holders are read-only (controls fade out, a READ-ONLY
     badge appears next to the ROOM input). They still receive live updates.
   - Tap **Off** to disable sync.

   Enforcement is app-level. Anyone who knows the writer code (or with
   Firebase API knowledge) can technically write either way. For a friendly
   game this is fine; if you need server-enforced write protection you'd
   need to layer on Firebase Anonymous Auth + custom rules.

### What syncs / what doesn't

| Field | Synced |
|---|---|
| Dead / alive flag per seat | yes |
| Ghost-vote available per seat | yes |
| Player name per seat | yes |
| Character assignments | no — each device picks its own |
| Script (TB / S&V / BMR / custom) | no |
| Seat count | no — by index; if devices disagree on count, the higher seats just won't sync |
| Tags / notes | no — local to each device |

Seat *index* is the sync key. If one device has 8 seats and another has 6, the 8-seat device
will see dead-state changes for seats 1-6 but not 7-8.

## Files

- `index.html` — the whole app (CSS + JS inlined; ~1500 lines)
- `manifest.json` — PWA manifest, makes it installable on iOS/Android
- `icon-192.png`, `icon-512.png` — app icons
- `README.md` — this file
