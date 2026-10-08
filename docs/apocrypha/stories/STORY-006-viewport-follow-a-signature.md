# STORY-006: Viewport that follows a signature

**Parent:** [EPIC-001](../epics/EPIC-001-protobuf-output.md)
**Status:** Not started. Depends on [STORY-004](STORY-004-viewport-sampler-and-ring-buffer.md).
**Size:** medium

## The idea

A [signature](../reference/signature-files.md) already notices a moment: a
table record changes, or some bytes appear. When one hits, the viewport should
look **where it hit**, and keep looking there, so you see what happens to that
spot afterwards, not only the instant it changed.

## What changes

A third anchor kind:

```text
webserver_viewport = signature:surveyor-text,256,250
                     ^kind ^signature name ^size ^interval_ms
```

The viewport's base is the `address` of the most recent hit of the named
signature, centred by half of `size`. Until that signature has hit, there is
no base and no frame.

This needs one small addition to `signatures.h`, the only change outside the
viewport's own files: a way to ask for the last hit of a signature by name,
for example `std::optional<Hit> Signatures::LastHit(name)`, returning the
address, the hit's `seq` and when it happened. It is read on the emulation
thread, where both the signatures and the sampler live, so it needs no lock.

## Stale anchors

"Most recent hit" can be minutes old. Each frame carries the hit's `seq` and
age (milliseconds since the hit), so a reader can tell a fresh anchor from a
stale one, and can notice when it jumped to a new hit.

A signature that hits many times a second (a table watch during a bulk
reload) would drag the view around. Bulk lines don't count as a hit for this
purpose: they say the whole table changed, not where.

## Acceptance

- Naming a signature that exists: the first frame appears after its first
  hit; later hits move the window; frames carry the hit's `seq` and age.
- Naming a signature that doesn't exist logs once and makes no frames.
- After `POST /api/v1/signatures/reload`, a missing name is noticed and the
  anchor recovers when it comes back. (`Reload()` clears all hit state.)
- A gtest unit test covers the anchor logic with a fake `LastHit`.
- Reference page updated.

## Notes

- This lets a hit with `"actions": ["pause"]` and a viewport work together:
  the game pauses at the moment, and the API (which still answers while paused,
  see [http-api.md](../reference/http-api.md#how-a-request-runs)) serves the
  window frames from just before and at that moment.
- Alternatively the hit log's own `window` already holds the bytes around
  the hit. The viewport adds *what happens next*, not the instant.
