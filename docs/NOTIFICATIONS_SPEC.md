# New comments inbox — spec

## Patrick's words (2026-11-22)
> "another thing I want to add is a new comments function, that when you log in flags all the new
> comments that have been made on tracks or structures, with special attention given to the ones
> for that user. I think the tracks should have a little mark to indicate there's unread comments
> or notes in them on the tracks page but also a home page thing. best place is probably around
> their circle in the top right. i want tapping that to bring down a drop down popup of all the new
> comments, with a "for you" and "all" filter at the top, like 2 buttons that open a different list
> within the pop up. clicking one takes them to it. also want a mark as read, and mark all as read.
> obviously clicking it from the notification or opening it in the structure or comments under a
> track also marks it as read. also a hold to pin it function, or hold to share."

## What it is
- On sign-in, flag every **new** comment: recording comments (on tracks) and notes on the
  structure board. Ones **for you** (addressed to you) get special attention.
- **Tracks** with unread comments or notes get a small mark on the tracks page.
- **Home / everywhere:** a mark around your avatar circle (top right). Tap it → a drop-down with
  two buttons at the top, **For you** and **All**, each showing its own list.
- Tapping an item takes you to it (the track at that comment, or the board at that note) and marks
  it read. **Mark as read** per item and **Mark all as read**.
- Opening a comment anywhere else (the structure, or comments under a track) also marks it read.
- **Hold an item:** pin it, or share it.

## Decided (2026-11-22)
- Hold an item → **Keep in my list** (stays after it's read, just for you), **Share**, Mark read/unread.
- Read state **follows you between devices** (`_meta/reads.json`, one entry per person).
- **For you** = addressed to you, or a reply under your comment.
- Update pop-up: a card in the guide ("New comments: tap your circle"), shipped with it.
- Built: see PLAN.md for the details and my calls (old stuff starts read, Sign out moved into the
  drop-down, red ring = for you).
