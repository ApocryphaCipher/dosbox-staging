# STORY-005: Viewport that follows a value

**Parent:** [EPIC-001](../epics/EPIC-001-protobuf-output.md)
**Status:** Not started. Depends on [STORY-004](STORY-004-viewport-sampler-and-ring-buffer.md).
**Size:** medium

## The idea

A fixed window only helps if what you want to watch stays put. Often it
doesn't: a program keeps a number saying *which* record it is working on, and
the interesting bytes are in that record. Let the viewport read that number and
move with it.

(The word is **value**, not pointer, to keep it apart from the mouse pointer,
which is a separate tag on each frame. "Memory pointer" is the same idea.)

## What changes

A second anchor kind for the `webserver_viewport` setting
([STORY-004](STORY-004-viewport-sampler-and-ring-buffer.md)):

```text
webserver_viewport = value:0x5F10,2,0x20000,114,256,250
                     ^kind ^where ^width ^table ^stride ^size ^interval_ms
```

Each tick, read `width` bytes (1, 2 or 4, little-endian) at `where`, call the
number `n`, and sample the window at `table + stride * n`. When the program
changes `n`, the view moves.

A second form takes the number as an address directly (`stride` omitted).
Keep both only if both are wanted; start with the table form.

## Not valid is normal

`n` is garbage between screens, while loading, or when the program reuses that
memory for something else. So every sample has a **status**, carried on the
frame:

| Status | Meaning |
| --- | --- |
| `OK` | the window was read |
| `OUT_OF_RANGE` | `table + stride * n + size` is past the end of memory; **no frame** is made |
| `UNCHANGED` | the bytes are the same as last time; no frame (as in STORY-004) |

An out-of-range sample makes no frame and logs once per change, not once
per tick. The frame's `anchor` also records `n`, so a reader can tell *which*
record they are looking at, and notice when it changes.

## Acceptance

- With a program that holds a record index in a known place, changing the index
  moves the window to the new record on the next tick, and the frame says so.
- An index that points past the end of memory produces no frame and no crash.
- A gtest unit test covers the address arithmetic, including overflow of
  `stride * n` in 32 bits (use 64-bit maths).
- Reference page updated.
- Works for any game: the test uses a synthetic memory block, not a game.

## Notes

- Nothing here knows about any one game. A signature file's `note` is where
  to write down *why* an address is a record index.
- *Guess:* most of the value of this anchor is following a "currently
  selected" index. Whether a game keeps one is a per-game fact, found with the
  usual dump-and-compare work.
