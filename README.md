# The Sorting Vote

A live A/B voting web app for a Harry Potter–themed 1st birthday party.
One self-contained `index.html` — no build step, no backend, no accounts.

It isn't a quiz — there are no right answers. Every question is a "who?" —
*who spoils him, who says no, who wakes first when he cries* — and the two
options are simply the two parents' photographs. Guests tap a face; nothing
is written on the cards. The closing screen tallies the whole night.

Rename `PARENT_A` / `PARENT_B` (both scripts) at the top of `index.html` and
every screen follows, scoreboard included.

- **Host** opens the *host link* on a laptop/TV and drives the pacing:
  Begin Voting → Reveal Results → Next Question, through all 16, then an end screen.
- **Guests** open the *guest link* on their phones and tap one of the two
  photographs — Anchit or Gunjan — for each of the 16 questions.
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

Before you start, the host screen shows the QR **large and centred**, so the
whole room can scan it at once from wherever they're standing. The moment you
hit Begin Voting it shrinks to the top-left corner and stays there for every
question — out of the way, but still there for anyone arriving late. It
retires on the closing screen.

Tap that card and the QR fills the screen for anyone at the back of the
room; close it with the button, a click anywhere outside, or Escape.

It's generated in the page (qrcode-generator, MIT, inlined — no QR service is
contacted and nothing about your party leaves the browser), and it encodes
whatever URL the host page is actually served from, so it can't go stale.
It's hidden below 720px wide, since a phone has no use for it.

## Pacing

There's no timer. A question stays open for exactly as long as you leave it —
you press *Reveal Results* when the room has finished voting, then *Next
Question*. Guests can vote right up until you reveal.

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

## Language

Guests get an **English / हिंदी** toggle in the top-right of their own screen.
It swaps every guest-facing string — questions, options, timer, messages and
the whole scoreboard — on that phone only. Nobody else's screen changes, and
the host's big screen always stays English.

The choice lives in memory, so a refresh returns to English. That's
deliberate: nothing is written to anyone's phone.

To add a language, add a key (say `ta:`) to every entry in `STRINGS` and to
every question in `QUESTIONS`, then extend the toggle. Hindi styling relaxes
the small-caps and letter-spacing automatically — Devanagari reads badly with
wide tracking — via the `:root[lang="hi"]` rules.

## The final scoreboard

When the host finishes the last question, both screens show the night's
totals:

- a crown line naming the winner, and a verdict that gently insults the loser
- both parents' portraits either side of the split, the winner ringed in gold
- the overall split — every vote across all the questions, as a percentage
- **the loser's comeback plan**: the three questions the room was most
  emphatic about, each with a daft instruction for winning it back next year
  (edit the `fix` line on any question in `QUESTIONS` to change the joke)
- how many questions each parent won, plus ties, and the total votes cast
- the most one-sided question and the closest call
- a question-by-question breakdown with a crimson/emerald split bar each

Guests additionally see their own line — which parent they sided with, and
how often. That's computed from the votes held in their own tab, so it's
personal to each phone and nothing about individual voters is stored.

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
      "pub":   { ".read": true, ".write": true },
      "votes": {
        ".read": true,
        ".write": true,
        "$q": {
          "$vote": {
            ".validate": "newData.isString() && (newData.val() == 'a' || newData.val() == 'b')"
          }
        }
      }
    }
  }
}
```

Open read/write, scoped to this app's subtree only, with every vote validated
as the string `a` or `b` — appropriate for an anonymous party app with no auth
layer. Delete the project after the party.

## How many guests it holds

Unlimited, on the free plan.

Firebase's free tier caps *live* connections at 100, so only the **host** holds
one. Guests hold none: they poll a single small endpoint (`pub`, about 200
bytes) every 700 ms over plain HTTPS, which no quota limits. They don't even
download the Firebase SDK, so a guest's page loads faster than the host's.

Because REST has no transactions, votes are **appended** — one child per vote
with a unique key — so two people voting in the same instant can't overwrite
each other. The host, the single writer, counts them and publishes compact
tallies when it reveals a question; that's all a guest ever reads.

At 200 guests that's roughly 285 requests/second of ~200 bytes — about 100 MB
across a two-hour party, against a 10 GB monthly allowance. No card, no
billing, no ceiling worth worrying about.

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
