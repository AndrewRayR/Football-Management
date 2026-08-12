# Sideline

A personal, single-page site for keeping up with your school football team:
your Hudl film embedded (with a fallback link), an editable calendar for
games/practices, and a photo intake box that tries to OCR schedule photos
straight into calendar entries.

It's one HTML file (`index.html`) plus a couple of Firebase rules files.
No build step — just fill in the config below and push it to GitHub Pages.

## 1. About the Hudl embed

`https://app.hudl.com/watch/team/17063/analyze` is behind a login, and
almost every login-gated platform blocks being shown inside someone else's
page (it's a standard anti-clickjacking protection called
`X-Frame-Options`/CSP `frame-ancestors`). I couldn't confirm Hudl's exact
header from here, but it's very likely blocked. The page is built to try
the embed anyway and show an "Open Hudl in a new tab" link right below it
either way, so you're never stuck with a blank box.

## 2. Set up Firebase (new project — stays on the free Spark plan)

Everything here runs on Firebase's free **Spark** plan — no credit card, no
Blaze upgrade. That's why photos are **not** stored in Firebase Storage
(Storage requires upgrading to the paid Blaze plan just to turn it on,
even though light usage would stay within the free quota). Instead,
schedule photos are compressed in the browser and saved as regular
Firestore documents — same free database as the calendar events.

1. Go to [console.firebase.google.com](https://console.firebase.google.com) → **Add project** → name it whatever you want (e.g. "sideline").
2. **Build → Authentication → Get started → Sign-in method → Anonymous → Enable.** (This lets the app connect without you having to log in — it's just there to keep the database from being wide open to strangers.)
3. **Build → Firestore Database → Create database** → start in **production mode** → pick a region.
4. In Project settings (gear icon) → **Your apps** → **Add app → Web** → register it → copy the `firebaseConfig` object it gives you.
5. Paste that object into `firebaseConfig` near the top of the `<script type="module">` block in `index.html`.
6. In Firestore → **Rules** tab, paste in the contents of `firestore.rules` here, then **Publish**.

That's it — no Storage bucket, no billing setup needed anywhere in this project.

## 3. Set your PIN

1. Open `index.html` in a browser (locally is fine — file:// works for this part).
2. On the lock screen, open **"First time setting this up?"**, type your PIN, click **Generate hash**.
3. Copy the hash it shows you, and paste it as the value of `PIN_HASH` near the top of the script (replacing `"REPLACE_ME"`).

Worth knowing: this PIN is checked entirely in the browser. It'll keep out
anyone who stumbles on the link, but anyone who looks at "view source" and
knows what they're doing could bypass it. There's no real way around that
on a plain static GitHub Pages site (no server = nowhere to keep a real
secret). Combined with an unlisted URL, this is a reasonable "keep it
private-ish" layer for a personal tool — just don't put anything truly
sensitive behind it.

## 4. Deploy to GitHub Pages

1. Create a new GitHub repo (e.g. `sideline`).
2. Push `index.html` (with your real config + PIN hash filled in) to the repo.
3. Repo **Settings → Pages → Source: Deploy from a branch → main / (root)**.
4. Your site will be live at `https://<your-username>.github.io/sideline/`.

Don't make the repo name something obvious like "football-schedule" if
you'd rather keep the URL from being guessable — an unlisted, non-obvious
path is doing real work alongside the PIN here.

## 5. How the photo → calendar OCR works

- Every dropped photo is resized/compressed in the browser (so it fits
  Firestore's ~1MB document limit) and saved to your gallery **first**, no
  matter what happens next — so you never lose a photo even if OCR fails.
  If a photo is too large/detailed to compress down far enough, you'll get
  a message asking you to crop it tighter or use a lower-res photo instead.
- It then runs Tesseract.js (an OCR library, entirely in your browser) on
  the original photo and looks for date/time patterns line by line.
- Whatever it finds shows up as editable rows — check them over, fix
  anything wrong, uncheck anything bogus, then add them to the schedule.
- Handwritten schedules, tables with weird spacing, or low-quality photos
  will trip it up more. The raw OCR text is always shown too, so you can
  fall back to reading it yourself and adding entries manually with the
  **+ Add** button on the Schedule section.

## Customizing

- Section labels ("Film", "Schedule", "Photos") and colors are all in the
  `<style>` block at the top of `index.html`.
- The Hudl URL is used in two places — the `<iframe src>` and the fallback
  link `href`. Search for the URL to update both if it ever changes.
