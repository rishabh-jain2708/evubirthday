# The Sorting Vote

A live A/B voting web app for a Harry Potter–themed 1st birthday party.
One self-contained `index.html` — no build step, no backend, no accounts.

It isn't a quiz — there are no right answers. Every question pits
**Anchit's** prediction (option A, Gryffindor crimson) against **Gunjan's**
(option B, Slytherin emerald), and the room picks a side. The closing screen
tallies the whole night. Rename `PARENT_A` / `PARENT_B` at the top of
`index.html` and every screen, including the scoreboard, follows.

- **Host** opens the *host link* on a laptop/TV and drives the pacing:
  Begin Voting → Reveal Results → Next Question, ×14, then an end screen.
- **Guests** open the *guest link* on their phones and tap A or B.
- Everything syncs live across every device via Firebase Realtime Database.

## The two links

| Who | Link |
|---|---|
| Guests | `https://<user>.github.io/<repo>/` |
| Host | `https://<user>.github.io/<repo>/?host=greathall` |

Only the URL with `?host=` shows the host controls; everyone else lands
straight in the guest view and can't start, reveal, or reset anything.
Change the key by editing `HOST_KEY` in `index.html`.

This prevents accidents, not attacks — the key sits in the page source, and
the database rules let any visitor write. That's the right trade-off for a
birthday party; don't reuse the pattern where it matters.

The host's lobby screen prints the guest link, so you can read it out or
turn it into a QR code without typing anything.

## Scan-to-join QR

The host screen shows a QR code of the guest link in the top-left corner, on
the lobby and on every question — so anyone arriving late can point a camera
at the big screen and be voting in seconds, without you reading a URL aloud.

It's generated in the page (qrcode-generator, MIT, inlined — no QR service is
contacted and nothing about your party leaves the browser), and it encodes
whatever URL the host page is actually served from, so it can't go stale.
It's hidden below 720px wide, since a phone has no use for it.

## Music

The host screen has a ♪ play/pause button, top-right. It plays
`music/theme.mp3` on a loop, on the **host device only** — that's the one
attached to speakers, and thirty phones playing the same track slightly out
of sync would be a mess.

The track lives at `music/theme.mp3`. Swap that file to change the music.
Note it's a copyrighted recording being served publicly by GitHub Pages —
fine in practice for a family party, worth knowing if the repo stays up.

Browsers block audio until someone interacts with the page. The host's click
*is* that interaction, so it just works — but the host has to click it; it
can't start on its own. Pausing keeps your place in the track; play resumes
from there.

## The final scoreboard

When the host finishes the last question, both screens show the night's
totals:

- the overall split — every vote across all 14 questions, as a percentage
- how many questions each parent won, plus ties, and the total votes cast
- the most one-sided question and the closest call
- a question-by-question breakdown with a crimson/emerald split bar each

Guests additionally see their own line — which parent they sided with, and
how often. That's computed from the votes held in their own tab, so it's
personal to each phone and nothing about individual voters is stored.

## The 20-second timer

Each question runs a `QUESTION_SECONDS` countdown (20 by default, at the top
of `index.html`). Both screens show the same clock — every device counts
against Firebase's server time, not its own — and at zero voting locks and
the host screen reveals the results automatically. The host can still hit
*Reveal Results* early.

The auto-reveal is written by the host's browser, so the host screen needs to
stay open for it to fire.

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

- **Restart** sits next to the host's main button on every question, plus the
  end screen. It takes two taps — the second confirms — and wipes every vote
  back to the lobby. Guests can vote again immediately; nobody has to reload.
- Guests can vote once per question, tracked in memory per browser tab
  (no login; a refresh lets someone vote again — fine for a party).
- Votes are rejected once the countdown hits zero.
- Votes use a Firebase transaction, so simultaneous taps can't overwrite
  each other.
- No localStorage/sessionStorage; all shared state lives in Firebase.
- Respects `prefers-reduced-motion` (vial fill animates instantly).
