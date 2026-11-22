# Update pop-ups ("What's new") — standing practice

## Patrick's words (2026-11-22)
> "everytime I make a change I wish to communicate with the band members to do a little updates
> pop up that explains the change. Ask if what I'm doing requires it and at which stage the pop up
> should be implemented. and remember the asthetics and clarity rules we've established."

## The rule
For **every** change Patrick asks for, before building:
1. **Ask whether it needs an update pop-up** for the band. (Bug fixes and Patrick-only tools
   usually don't; anything a bandmate will see or should start using usually does.)
2. **Ask at which stage it goes in**: with the change itself, or held back until a group of
   related changes is finished (e.g. the board guide was held until the sprint was done).
3. Log the answer in `PLAN.md` under the change's decision entry.

## How it's built (what exists)
- `/* ── WHAT'S NEW` in `index.html`: `WN_STEPS` (the cards), `WN_VERSION` (bump it to show the
  cards again to everyone, once per device, `ng_whatsnew`). Home → "What's new in this update"
  reopens it. `wnOpen(set)` can show any other list of cards (e.g. `wnTabHelp` for the tab
  editor's first use, `ng_tabhelp`).
- Unseen cards should pile up rather than replace each other: if bandmates haven't seen the last
  update yet, add to its cards instead of throwing them away (ask Patrick if unsure).

## Aesthetics and clarity rules (agreed 2026-11-22)
Written for someone with ADHD: skimmable in seconds.
- **One card per thing.** Title says what it is in a few words ("Flip: lyrics ⇄ notes").
- **A picture first:** a small mock-up of the real screen in the app's colours, not a screenshot.
- **Numbers on the picture match the numbered steps below it.** Every step that's an action gets
  its number on the picture, including step 1. No unlabelled steps, no numbers out of order.
- Circle the thing to tap (`_wnHl`). When the steps are about **columns**, put the numbers in a
  row above their columns with a pointer down (`_wnNum`) rather than scattering them. When one
  thing is tapped twice, give both numbers one label with a stepped arrow (the flip card).
- Mock-up rows must line up like the real board (fixed column widths, `_wnRow`).
- **Steps are punchy and imperative:** "Tap **+ Note**", "do this, then this". Bold the thing to
  tap. 2–4 steps per card.
- Optional 💡 tip: one line, for the second way to do it.
- **Details hide behind a button** ("Show me how…") so the card stays short. Long how-tos (like the
  tab editor) also show when the feature is first used, with a **?** to reopen them.
- Light theme only, app fonts and member colours. Back / Next / Skip, pips for progress.
- Check every card in a screenshot before pushing (numbers match steps, nothing clipped).
