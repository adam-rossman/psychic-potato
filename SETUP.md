# Bella's Care Dashboard — Setup Guide

This is a single static HTML file (`index.html`) with everything bundled in —
React, styling, and logic. No build step. The only setup required is creating
a free Firebase project so everyone's checkmarks sync to each other.

## 1. Create a Firebase project (free)

1. Go to https://console.firebase.google.com and sign in with any Google account.
2. Click **Add project**. Name it something like `bella-care-dashboard`. You can
   skip Google Analytics (not needed).
3. Once created, click the **web icon (`</>`)** on the project overview page to
   register a new web app. Name it anything (e.g. "Bella Dashboard").
4. Firebase will show you a config object that looks like this:

   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "bella-care-dashboard.firebaseapp.com",
     projectId: "bella-care-dashboard",
     storageBucket: "bella-care-dashboard.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abc123"
   };
   ```

   Copy these six values.

5. In `index.html`, find the `firebaseConfig` object near the top of the
   `<script type="text/babel">` block and paste your values in, replacing the
   placeholders.

## 2. Turn on Firestore (the database)

1. In the Firebase console, go to **Build → Firestore Database** in the left
   sidebar.
2. Click **Create database**.
3. Choose a location close to your family (any US region is fine for most
   families).
4. Start in **production mode** (we'll set rules in the next step).

## 3. Set security rules

By default, production mode blocks all reads/writes. Since this app has no
login system (just a name people type in once), we'll open access to the
two things this app uses: today's live document, and the daily history log:

1. In Firestore, click the **Rules** tab.
2. Replace the contents with:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /bellaCare/state {
         allow read, write: if true;
       }
       match /dailyLogs/{date} {
         allow read, write: if true;
       }
     }
   }
   ```

3. Click **Publish**.

   **Note on privacy:** this makes the live document (`bellaCare/state`)
   and every saved daily history record (`dailyLogs/*`) readable and
   writable by anyone who has your deployed URL — there's no login. That's
   fine for a private family tool that isn't indexed or shared publicly,
   but don't post the link somewhere public. If
   you want real authentication later, Firebase supports that too, but it's
   more setup than this guide covers.

## 5. Deploy to GitHub Pages

1. Create a new GitHub repo (or use an existing one), e.g. `bella-care`.
2. Add `index.html`, `manifest.json`, `sw.js`, and the **`icons/` folder**
   (with all the `.png` files inside it) to the root of the repo — keep the
   same folder structure, since `index.html` and `manifest.json` reference
   `icons/...` as a relative path.
3. Go to the repo's **Settings → Pages**.
4. Under **Source**, choose **Deploy from a branch**, pick `main` (or
   `master`) and `/ (root)`, then **Save**.
5. After a minute or two, GitHub will give you a URL like:

   `https://yourusername.github.io/bella-care/`

6. Share that link with whoever is dog-sitting. On a phone, opening that
   link and using "Add to Home Screen" (iOS Safari share menu, or the
   install prompt on Android Chrome) installs Bella's photo as the app icon.

## Updating Bella's photo / app icon later

The `icons/` folder has everything pre-generated from one source photo:

| File | Used for |
|---|---|
| `icon-512.png`, `icon-192.png` | Standard PWA icon (most platforms) |
| `icon-maskable-512.png`, `icon-maskable-192.png` | Android adaptive icon — has padding built in since Android can crop icons into a circle/squircle |
| `apple-touch-icon.png` | iOS home screen icon |
| `favicon-32.png`, `favicon-16.png` | Browser tab icon |

To swap in a new photo, crop it to a square focused on Bella's face first
(square crop, face filling most of the frame), then regenerate each size
from that square — or just send Claude the new photo and ask it to redo the
icon set.

## Password protection

The dashboard has a simple shared-password lock screen. It asks once per
device — after someone enters the password correctly, that browser
remembers it (via `localStorage`) and won't ask again unless they clear
their browser data.

**Important: this is not real security.** It's client-side only, meaning
anyone who views the page source can read the password in plain text. The
underlying data in Firestore is also openly accessible to anyone with the
URL (see the security rules note in step 3 above). This lock screen is just
meant to keep the dashboard from being stumbled into by a stranger or
indexed by a search engine — not to protect sensitive information.

To change the password, open `index.html` and find this line near the top
of the script:

```js
const SHARED_PASSWORD = "bella2026";
```

Replace `"bella2026"` with whatever you want, then redeploy. Anyone who was
already unlocked on their device stays unlocked — they won't be asked
again unless they clear their browser data or you change the
`PASSWORD_STORAGE_KEY` value too (which forces everyone to re-enter it).

## How daily reset works

- The dashboard automatically clears all checkmarks and custom tasks at
  midnight (checked every minute the app is open), using each device's
  local time.
- Anyone can also tap **"Start new day now"** at the bottom to reset early —
  it asks for confirmation first since it affects everyone.
- The care card (feeding, treats, temperament, emergency contacts) does
  **not** reset — that information persists until someone edits it.
- Right before a reset happens (automatic or manual), the day's final
  schedule, custom tasks, and care card are saved to the `dailyLogs`
  collection in Firestore, dated by that day. This is what powers the
  history view described below.

## Viewing history

Use the **‹ ›** arrows next to the date at the top to step backward and
forward one day at a time, or tap the date itself to open a calendar and
jump to any specific day. Viewing a past day shows that day's final record
— what was checked off, who did it, and what the care card said that day
— but everything is **read-only**; nothing on a historical day can be
edited.

Tap **"Jump to today"** (shown whenever you're viewing a past day) to
return to the live, editable dashboard.

A day only has a history record once it has actually finished (reset
either automatically or manually) — today's in-progress record won't show
up in history until the day ends.

## Customizing the default schedule

If you want to change the default meal times, let-out windows, or task names,
edit the `makeDefaultSchedule()` function in `index.html`. Changes there only
affect what a *new* day starts with — they don't change Firebase data
retroactively.

## Troubleshooting

- **"Local only" status badge:** Firebase config still has placeholder values.
  Double check step 1.
- **"Offline" status badge:** Firestore rules may be blocking access, or the
  project ID is wrong. Check the browser console (F12) for the exact error.
- **Changes not showing on other phones:** make sure everyone is using the
  same GitHub Pages URL, not a locally saved copy of the file.
