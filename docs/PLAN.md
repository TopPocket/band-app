# Plan & decision log

Patrick dips in and out of this project. This file is the memory: **what he's said, what's been
decided, what's open, what's next.** Every session: read it first, and log new decisions and wishes
before building anything. Newest entries at the top of each section. Feature specs live in
`docs/*_SPEC.md`.

## Now
- **Doodle Board / Song Canvas redesign:** see [DOODLE_SPEC.md](DOODLE_SPEC.md). Design agreed
  2026-11-20. Build order:
  1. ✅ (2026-11-20, not yet pushed) Permanent dot IDs (`doodle.barIds`), pins point at `barId`,
     deleted dot → pin floats, old `barIdx` pins migrated on open.
  2. Board becomes the song page's Structure tab: vertical, energy | dots | lanes | lyrics, with
     Edit structure as the deeper level. Notes tab removed. Fix-links icon for floating items.
  3. Lyrics linking (even spread, per-line tweaks from both sides, broken links in Lyrics edit mode).
  4. Member lanes and moments. 5. Layers (comments/tabs callouts, lyrics fade, linked track
     comment ▶). 6. Versions, then the horizontal toggle.
- After pushing step 1: everyone should hard-refresh. An old app that edits a structure changes
  `bars` without `barIds`, and pins may then land on the wrong dot.

## Decisions
- 2026-11-20: **Versions:** people can save versions (snapshots) of the structure and lanes.
  Details open (Q18).
- 2026-11-20: **Anyone can edit any lane.** Each chunk records who made it.
- 2026-11-20: **Member colours:** I pick them (distinct from section colours).
- 2026-11-20: **Energy shape:** taper between set points (dots without a value don't count). Each
  set point holds **flat across its own dot** before tapering, so a single quiet dot reads as a
  square step, not a spike. Before the first point, hold the first value. After the last, hold the last.
- 2026-11-20: **Energy bar** to the *left* of the dots. Set hype points (1–5) at chosen dots. The
  bar's thickness shows how hard to go for it. (Open: smooth or stepped between points.)
- 2026-11-20: **One lane per member (all five), colour-coded.** There are no member colours in the app
  yet, so they must not clash with the section colours.
- 2026-11-20: **The Structure tab *is* the board.** "Edit structure" is the deeper level (the keyboard).
  The separate full-screen overlay goes away.
- 2026-11-20: Notes tab content (`noteText`, `noteImages`): Patrick doubts there's anything worth
  keeping. Plan: leave the data in songs.json untouched (don't delete), just stop showing it.
- 2026-11-20: **Default board view = structure dots | member lanes | lyrics.** When comments (or
  other layers) are switched on, the lyrics fade far into the background *underneath* the comments,
  barely readable.
- 2026-11-20: **Linked track-comment playback:** a ranged comment plays its range. A point (or very
  short) comment plays from 2 s before to 10 s after.
- 2026-11-20: **Member lanes stay very thin until selected.** A selected lane gets thicker and shows
  its labels plus its tabs and comments.
- 2026-11-20: **Lyrics smart default:** spread a paragraph's lines evenly over its section's dots,
  where that makes sense.
- 2026-11-20: **Dots on the left**, then member lanes, then callouts.
- 2026-11-20: **Song (track) page tabs become: Recordings, Lyrics, Tabs, Structure.** The
  Structure tab *is* the doodle board (embedded), with a way to go deeper into structure editing from
  there. **The Notes tab is removed.** (Open: what happens to existing `noteText`/`noteImages`.)
- 2026-11-20: Every dot gets a permanent ID and attachments point at that ID, not at a position.
  Old pins get migrated on first open. (Built.)
- 2026-11-20: Versions save the dots, lanes and energy (optional name, who, when). Attachments
  aren't copied: they point at dot IDs, so restoring brings back whatever still matches, and the rest
  floats. Restoring auto-saves the current board as a version first.
- 2026-11-20: **Deleted dot → its attachments float.** Keep them, flag them quietly. No toasts,
  banners or badges that clutter. Expect boards to be half-finished most of the time, so floating
  items are normal, not an error. UI: **just a small "fix links" icon somewhere** (shown only when
  something is floating) that flags it and maybe opens a fix mode. No count, no toasts. Exception:
  on the **Lyrics page in edit mode**, broken or floating lyric links should be more obvious.
- 2026-11-20: **Lyrics ↔ dots: one dot per lyric row.** If a lyric spans several dots, the user
  splits it across rows. Smart default: assign lyrics **per chunk** (paragraph → section), and the user
  fine-tunes the per-dot assignment. Patrick sees the Lyrics page tying to the dots nicely, so linking
  works from both sides.
- 2026-11-20: **Member lanes are thin bars/lines running alongside the dots, not dots.** Column
  order: spine dots | member bars | callouts (lyrics, tabs, comments…). A **moment** is a single
  callout line, like a pin.
- 2026-11-20: **Default view is vertical.** Build around vertical, then make sure a horizontal
  toggle still makes sense.
- 2026-11-20: **Board comments are separate from track comments**, but a board comment can **link a
  timestamped track comment** (e.g. "Rory, play this riff here"). The board comment gets a small ▶
  that plays that stretch of the recording without leaving the board. Exact mechanics TBD.
- 2026-11-20: Restructuring the spine must never delete or misplace attachments (lyrics, tabs,
  comments, pins).
- 2026-11-20: Pinning to gaps between dots is parked. Dots only for now.
- 2026-11-20: Member lanes and layers are collapsed or hidden by default. Toggles go in the bottom bar.
- 2026-11-20: Want a vertical spine view (phone-friendly, lines up with lyrics).
- 2026-10-24: Doodle board is core. It's the only structure tool (old `structures.json` only seeds it).

## Open questions (Doodle Board)
Asked 2026-11-20, awaiting answers:
1. ~~What happens to attachments when their dot is deleted?~~ Answered: they float (see Decisions).
1–8, 10–11: answered 2026-11-20 (see Decisions).
9. Lyric links are stored by line text (like lyric annotations). If a linked line is edited, try to
   re-find it, and if that fails it floats. Not objected to so far. Treat it as the plan.
12–14: answered 2026-11-20.
15–17: answered 2026-11-20. (Proposed chunk drawing: select lane → tap start dot → tap end dot → label.)
18. Answered 2026-11-20 (see Decisions).

## Known problems
- ~~Doodle pins store `barIdx` (position), so inserting a bar shifts every later pin.~~ Fixed by
  step 1 (2026-11-20).

## Parked / someday
- Pin to the gaps between dots.

## Lost context
- Patrick's original evening of dictation about the doodle board (around May 2026) was not saved
  in the repo or in any Claude Code session. Assume it was a claude.ai chat. If it turns up, fold
  it into DOODLE_SPEC.md.
