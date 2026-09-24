# Off-Season Checkpoint Tracker — GitHub + Firebase Setup

This is the same hosting pattern as your other live tools (GitHub Pages for the page itself, Firebase Firestore for live shared data), applied to this specific file. Read `claude/github-live-hosting-guide.md` in your project for the general version of these steps; this doc has the exact specifics for this tool.

## What's in this folder
- `index.html` — the tool itself. Rename nothing; GitHub Pages needs this exact filename to serve it at your site's root.
- `manifest.webmanifest`, `icon-192.png`, `icon-512.png`, `icon-512-maskable.png`, `apple-touch-icon.png` — let coaches "Add to Home Screen" on their phones with your logo and a proper app-like launch. Upload these alongside `index.html`, same folder.
- `firestore.rules` — paste this into the Firebase console (not part of the GitHub repo's job — see Step 3 below).

## Step 1: GitHub Pages (Part 1 of the general guide)
1. Create a new **public** repository, e.g. `offseason-checkpoint-tracker`.
2. Upload all 6 files from this folder (drag-and-drop via "Add file → Upload files" — no command line needed), including `firestore.rules` — it's fine to keep as documentation in the repo since it contains no secrets, but the copy that actually matters is the one you paste into the Firebase console in Step 3.
3. Settings → Pages → Source: "Deploy from a branch" → Branch: `main`, folder `/ (root)` → Save.
4. Your live URL will be `https://YOUR-USERNAME.github.io/offseason-checkpoint-tracker/`.

At this point the page works in **local demo mode** — anyone with the link can click around, but nothing is saved or shared. That's expected until Step 2.

## Step 2: Firebase project + Firestore
1. console.firebase.google.com → Add project → name it (e.g. "redsox-offseason-tracker").
2. Firestore Database → Create database → pick a US region → start in test mode (you'll lock it down in Step 3).
3. Project settings → Your apps → click the web icon (`</>`) → register a web app → copy the `firebaseConfig` object it shows you.
4. Authentication → Sign-in method → enable **Google** as a sign-in provider.
5. Authentication → Settings → Authorized domains → add `YOUR-USERNAME.github.io` (GitHub Pages domains aren't authorized by default).

## Step 3: Wire the config and the allowlist into `index.html`
1. Open `index.html` in GitHub (pencil/edit icon), find the `FIREBASE CONFIG` section near the top of the `<script type="module">` block, and paste your real values over the `PASTE_YOUR_...` placeholders.
2. In the same edit, find `COACH_EMAIL_ALLOWLIST` just below it and replace every `REPLACE_ME_...@example.com` placeholder with each coach's real Google sign-in email. **These were placeholders — only your own email (sthomas@redsox.com) was on file; the others need the real addresses.**
3. Commit changes. GitHub Pages redeploys automatically within a minute or two.
4. In the Firebase console: Firestore Database → Data → Start collection → collection ID `config` → document ID `coaches` → add one field, `emails` (type: array), and list the same coach email addresses, all lowercase. This is the list that actually enforces access (see `firestore.rules`) — the array in the HTML file only controls the sign-in screen's wording.
5. Firestore Database → Rules → replace the default rules with the contents of `firestore.rules` from this folder → Publish.

## Step 4: Test it
1. Open your GitHub Pages URL. You should see the Red Sox sign-in screen instead of the tracker.
2. Sign in with an email that's on both allowlists (yours, `sthomas@redsox.com`, already is). You should land on the tracker with your email shown top-right and a "Live and shared" banner.
3. Check a box, then open the same URL in a private/incognito window and sign in as a different allowed coach — you should see the same check reflected within a second or two.
4. Sign in (or try to) with an email that's on neither list — you should see "isn't on the coach list yet" instead of the tracker.

## Ongoing
- **Adding/removing a coach:** update both the `COACH_EMAIL_ALLOWLIST` array in `index.html` (commit via GitHub) and the `emails` array in the `config/coaches` Firestore document. Both need to match, or a newly-added coach will get past the sign-in screen's friendly message but still be denied by the Firestore rules (or vice versa).
- **Editing the roster, coach assignments, or calendar dates:** once Firebase is connected, do this directly in the live tool (Players & Coaches tab, Calendar Settings tab) — not by re-editing the seed data in `index.html`. The seed data only ever runs once, against a brand-new empty Firestore project.
- **Checking a deploy went out:** GitHub repo → Actions tab → look for a green check on the latest "pages build and deployment" run.
