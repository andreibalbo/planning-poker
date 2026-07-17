# Planning Poker (static + Firebase)

A single-file planning poker app. Hosts as a static site on GitHub Pages; team state
(who's in the room, votes, reveal/clear) syncs live through Firebase Realtime Database.
No server to run, no database to manage yourself — Firebase's free "Spark" plan is plenty.

## Features
- Enter a name to join; everyone sees the live participant list.
- Fibonacci deck (0, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89) plus `?`.
- Votes stay hidden — a `✓` shows who has voted without revealing the number.
- **Reveal** button flips all cards face-up; **Hide** flips them back.
- On reveal: average, range, votes-in count, and an agreement indicator.
- **New round / clear session** wipes everyone and resets for the next story.
- Your name is remembered on your device so you don't retype it.
- Multiple rooms via URL: `?room=team-alpha` (defaults to `default`).

---

## Setup (about 5 minutes)

### 1. Create a Firebase project
1. Go to https://console.firebase.google.com and click **Add project** (any name). You can skip Analytics.
2. In the left menu open **Build → Realtime Database → Create Database**.
   - Pick a location, then start in **test mode** for now (we'll lock it down in step 4).

### 2. Register a web app and copy the config
1. Project **Overview** → click the **`</>`** (web) icon → give it a nickname → **Register app**.
2. Firebase shows a `firebaseConfig = { ... }` snippet. Copy those values.

### 3. Paste the config into `index.html`
Near the bottom of `index.html`, replace the placeholder block:

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  databaseURL: "https://YOUR_PROJECT-default-rtdb.firebaseio.com",
  projectId: "YOUR_PROJECT",
  appId: "YOUR_APP_ID"
};
```

Make sure `databaseURL` is included (test-mode snippets sometimes omit it — grab it from the
Realtime Database page; it looks like `https://your-project-default-rtdb.firebaseio.com`).

### 4. Turn on Anonymous auth + set security rules
The app signs each visitor in anonymously so writes are authenticated.

1. **Build → Authentication → Get started → Sign-in method → Anonymous → Enable.**
2. **Build → Realtime Database → Rules**, paste this, and **Publish**:

```json
{
  "rules": {
    "rooms": {
      "$room": {
        ".read": "auth != null",
        ".write": "auth != null"
      }
    }
  }
}
```

This lets any signed-in visitor read/write rooms but blocks anonymous public traffic.
Good enough for an internal team tool. (For stricter control you can scope member writes
to `auth.uid`, but the clear-session button needs broader write, so keep it simple to start.)

### 5. Deploy to GitHub Pages
1. Create a repo (e.g. `planning-poker`) and add `index.html` to it.
2. **Settings → Pages → Build and deployment → Source: Deploy from a branch**, pick `main` / root, **Save**.
3. After a minute your site is live at `https://<your-username>.github.io/planning-poker/`.
4. Share that link (optionally with `?room=my-team`) with the team.

---

## Notes
- **Free tier limits:** the Spark plan covers a small team easily. Data is tiny (names + votes).
- **Privacy:** anyone with the link and room name can join. Use an unguessable `?room=...` if that matters.
- **Cost:** $0 — GitHub Pages and Firebase Spark are both free.
- Want to change the deck? Edit the `DECK` array at the top of the `<script>` in `index.html`
  (e.g. add `"☕"` for a coffee-break card, or switch to T-shirt sizes).
