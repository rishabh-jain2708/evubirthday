# The Sorting Vote

A live A/B voting web app for a Harry Potter–themed 1st birthday party.
One self-contained `index.html` — no build step, no backend, no accounts.

- **Host** opens the link on a laptop/TV and drives the pacing:
  Begin Voting → Reveal Results → Next Question, ×14, then an end screen.
- **Guests** open the *same* link on their phones and tap A or B.
- Everything syncs live across every device via Firebase Realtime Database.

## Setup (~3 minutes, free tier)

1. https://console.firebase.google.com → **Add project** (Analytics off).
2. **Build → Realtime Database → Create Database** (any region, test mode).
3. **Project settings → Your apps → Web (`</>`)** → register an app → copy the
   `firebaseConfig` object over the placeholder near the top of `index.html`.
   Make sure `databaseURL` is included.
4. **Realtime Database → Rules** → paste and Publish:

```json
{
  "rules": {
    "sortingvote": {
      "state": { ".read": true, ".write": true },
      "votes": {
        ".read": true,
        "$q": {
          "a": { ".read": true, ".write": true, ".validate": "newData.isNumber()" },
          "b": { ".read": true, ".write": true, ".validate": "newData.isNumber()" },
          "$other": { ".validate": false }
        }
      }
    }
  }
}
```

Open read/write, but scoped to this app's subtree only — appropriate for an
anonymous party app with no auth layer. Delete the project after the party.

## Deploy to GitHub Pages

```bash
git init && git add . && git commit -m "The Sorting Vote"
```

Push to a GitHub repo, then **Settings → Pages → Deploy from branch → `main` / root**.
Share the single `https://<user>.github.io/<repo>/` link with everyone.

## Before the party

Open the host view and check the lobby pill says **● live**. If it says
*not connected*, the config or the rules are wrong. If it says *demo mode*,
the `firebaseConfig` placeholders haven't been replaced — the app still runs,
but only on that one device.

## Personalising

Everything editable lives at the top of the `<script>`: `BABY_NAME`,
`PARTY_TAGLINE`, and the `QUESTIONS` array (14 entries of `{ q, a, b }`).
Add or remove questions freely — counts and the "of N" label follow the array.

## Notes

- Guests can vote once per question, tracked in memory per browser tab
  (no login; a refresh lets someone vote again — fine for a party).
- Votes use a Firebase transaction, so simultaneous taps can't overwrite
  each other.
- No localStorage/sessionStorage; all shared state lives in Firebase.
- Respects `prefers-reduced-motion` (vial fill animates instantly).
