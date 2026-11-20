# Doodle Board / Song Canvas — spec

Status: **vision captured, architecture not yet agreed. Nothing built from this yet.**
Decisions and open questions are logged in [PLAN.md](PLAN.md). Update both as things get decided.

> The original evening of dictation (around May 2026, probably a claude.ai chat) was never saved
> to the repo and can't be found. This spec is rebuilt from Patrick's restatement on 2026-11-20
> (quoted near the bottom) plus what the May build did.

## The vision (Patrick, 2026-11-20)
A place where the band can **be creative and build songs as seamlessly as possible**, a one-stop
canvas for putting a song together and getting ideas across. Everyone in the band is on a different
wavelength ("we can't even agree on what a bar is"), so the core is a **really clear visual
description of the song's structure**, used as a **spine** that lyrics, tabs and comments attach to.

## Principles
1. **The structure is the spine.** It's edited "on another level": a separate edit-structure mode.
   Outside that mode you pin things to it.
2. **Restructuring never deletes attachments.** Deleting a dot must not delete a lyric, tab or
   comment. Inserting or moving dots must not shift attachments onto the wrong dot.
   (The current build breaks this: pins store `barIdx`, an index, so inserting a bar early on
   moves every later pin. See PLAN.md.)
3. **Minimal by default.** Band-member lanes and layers are collapsible, and toggles live in the
   bottom bar, so the default view isn't overwhelming.
4. **Smart assumptions only where they help.** For example, guessing which lyric lines go with which
   section, but never so wrong that you have to fight it.
5. **Get the architecture crystal clear before building.** Ask first, and explain the implications.

## Features wanted
- **Pin to dots.** (Pinning to the *gaps* between dots: parked for now, maybe later.)
- **Layers:** lyrics, comments, tabs (and existing snippets/notes). They can be toggled.
- **Lyrics ↔ structure:** assign lyrics to the structure **line by line**, with smart defaults to
  get started. Also from the Lyrics side: assign a row to part of a section seamlessly, so it stays
  connected both ways. Link *existing* lyrics, don't retype them.
- **Who's doing what:** a lane per band member under the spine, showing their own rough
  structure as chunks that **don't have to line up with the song sections** (e.g. a bass line that
  carries through several sections, or changes halfway through one).
- **Moments:** "at this point, do this" callouts on a member's lane. Could be a comment or a tab,
  as well as the rough chunks.
- **Dot indicators:** a dot with an inner dot or a ring = it has something attached. Comments come
  out as callouts.
- **Tap a dot to expand it a bit** (show what's attached and the actions).
- **Add tabs or comments from here**, and link existing lyrics.
- **Multiple views.** A **vertical** spine (top to bottom) suits phones and lines up with lyrics.
  Horizontal is the current view.

## Layout (agreed 2026-11-20, details in PLAN.md)
```
 energy | dots | member lanes (5, thin, colour-coded) | callouts: lyrics by default; comments/tabs when toggled
   ▌    ◉ V1   ┃ ┊                   "first lyric line"
   █    ◎      ┃ ┊ ───               💬 comment callout (lyrics fade right back underneath)
 (energy = hype 1–5 set at chosen dots, shown as bar thickness)
```
- Vertical by default; horizontal toggle comes later and must still make sense.
- A selected lane gets thicker and shows labels, tabs and comments.
- Lyrics: one dot per row. The smart default spreads each paragraph evenly over its section's dots.
- Floating (unlinked) items: a small fix-links icon only. More obvious in Lyrics edit mode.
- Board comments are separate from track comments, but can link one and play it (range, or 2 s
  before to 10 s after a point).
- Lives in the song page's Structure tab. Song tabs: Recordings, Lyrics, Tabs, Structure (Notes removed).

## What exists today (built May 2026, `/* DOODLE BOARD` in index.html)
- `song.doodle = { bars: ['V','V','c',…], locked, customTiles, pins }` in songs.json.
  Lowercase = half bar. Sections are derived from runs of the same letter (`deriveSections`).
- Edit mode: key tiles I V C B O X S plus 3 custom tiles. Hold and slide up for a half bar. Cursor, ⌫.
- Locked mode: tap a dot → +Note / +Tab / +New Tab / +Snip / +Record Snip. Pins sit above the
  spine with curved lines and can be re-anchored. Pins = `{ id, type, barIdx, … }`.
- Horizontal only, pinch-zoom/pan. A read-only copy shows on the song page's Structure tab.

## Raw words (2026-11-20), kept so nothing gets lost
> So the idea of this section was to have a place that we can be creative and build songs as
> seamlessly as possible. we are all on different wavelengths, like we can't even agree on what a bar
> is, so I wanted a really clear visual description of what the song structure is and use that as
> spine to attach lyrics, tabs, comments to. I think we said attaching to a dot would make sense,
> especially for lyrics but we need to make sure deleting a dot doesn't delete a lyric for example.
> a restructure of the song shouldn't remove attachments. the architecture of this needs to be
> crystal before we dive in. I would like the structure as the spine, something you edit on another
> level, and when not in edit structure you can pin things to it. pin to the dots, pin to the gaps
> perhaps (though maybe not gaps for now - put a pin in that). ideally in lyrics you would be able
> to assign them to the structure, line by line, maybe with a bit of smart assumptions to get you
> going, as long as the assumptions aren't so bad they inhibit the process. then we can have a
> comment layer too, and a tab layer. also I want it to have a who's doing what line bar for each
> member. so you can crudely see when an individual member is doing a thing. so like the structure
> of the song is broken into chunks a bassist might have their own distinct chunks that overlay and
> is different to say the guitarist. like a bass line might carry through multiple sections, or
> change halfway, like that would be great to capture. so each person would have a rough structure,
> but it would also be neat to have a moments call out, maybe just comments, maybe add a tab, but a
> "at this point do this" kinda thing in addition to the basic structure. I would want the band mate
> things collapsible so the default ui is minimal and not overwhelming. maybe show dots with a dot
> inside or a ring round it to indicate it has a comment and the comments come out as callouts.
> toggles for that sort of thing at the bottom bar maybe. I envision multiple views, but also want a
> click of a dot to expand a bit. I want the ability to add tabs or comments from here, and link
> existing lyrics. also in lyrics to be able to assign a row to a bit of the section in a seamless
> way and have it be connected.

Earlier (same day): it'd make sense to have an option where the structure runs **vertically**, as
that fits a phone screen and the lyrics would line up better.
