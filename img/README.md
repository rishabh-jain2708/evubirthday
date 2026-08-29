# Parent portraits

Four files, two per parent:

- `anchit.jpg` / `gunjan.jpg` — square, the round portraits on the closing
  scoreboard (crimson ring for A, emerald for B)
- `anchit-card.jpg` / `gunjan-card.jpg` — the photo you tap to vote, shown
  behind each option on every question

All four are cropped from the Hogwarts shoot. To swap them, replace the files
(same names) or repoint `PARENT_A_PIC` / `PARENT_A_CARD` and their B
equivalents in `index.html`. The full-size originals aren't committed.

They're rendered as circles, so **square crops work best** — a head-and-
shoulders shot centred on the face. Anything from about 400×400 up is plenty;
keep each under ~500 KB so phones load them instantly.

Whoever wins the night gets a gold ring and a 👑 on their caption.

If a file is missing the scoreboard just renders without portraits — nothing
breaks, so it's safe to add them later. Change the paths with
`PARENT_A_PIC` / `PARENT_B_PIC` in `index.html`.
