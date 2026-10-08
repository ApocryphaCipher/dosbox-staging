# STORY-004: Viewport sampler and ring buffer, at a fixed address

**Parent:** [EPIC-001](../epics/EPIC-001-protobuf-output.md)
**Status:** Not started. Depends on [STORY-001](STORY-001-spike-libprotobuf-in-the-build.md).
**Size:** large (new behaviour, and the epic's one threading problem)

## The idea

A memory dump is 16 MB. Most questions only need a few hundred bytes. A
**viewport** is a small window of memory, sampled at an interval and published
as frames, so a client can watch just that window without pulling the whole
dump or writing a signature.

This story builds the machinery with the simplest anchor, a **fixed address**.
[STORY-005](STORY-005-viewport-follow-a-value.md) and
[STORY-006](STORY-006-viewport-follow-a-signature.md) add anchors that move.

## What changes

A setting, working name `webserver_viewport`:

```text
webserver_viewport = fixed:0x35F80,256,250
                     ^kind ^address ^size ^interval_ms
```

Each tick (next to `Signatures::Tick()`), when the interval has passed:

1. Copy `size` bytes from the address.
2. If they equal the previous frame's bytes, do nothing. **Only changes make
   frames.** This is what keeps it from flooding.
3. Otherwise store a frame in a **fixed-size ring buffer** (say the last 64).

A frame is a `Viewport` message. Sketch:

```proto
message Viewport {
  string launch_id = 1;
  uint64 seq = 2;                 // counts frames, not ticks
  google.protobuf.Timestamp time = 3;
  uint32 base = 4;                // linear address read
  bytes data = 5;                 // the window
  optional Mouse mouse = 6;       // DOS driver position, if active
  Anchor anchor = 7;              // FIXED here; others in 005 and 006
}
```

A new endpoint, `GET /api/v1/viewport?since=SEQ`, returns every frame still in
the buffer with `seq` greater than `SEQ`, as length-delimited `Viewport`
records with `Content-Type: application/x-protobuf`. A client that is slow
or absent just misses old frames.

## Threading: use the bridge, add no lock

The emulator has two kinds of thread:

- The **emulation thread** owns emulated memory. Only it may read memory, and
  it must never be slowed down.
- The **webserver thread** answers HTTP requests.

The fork already has the pattern for crossing between them
([`bridge.h`](../../../src/webserver/bridge.h)). A handler builds a `Command`
and calls `WaitForCompletion()`; the emulation loop runs its `Execute()` from
`Bridge::ProcessRequests()`, in the same place it calls `Signatures::Tick()`,
and the handler then formats the result on the webserver thread. The memory
read endpoint works exactly like that.

So the viewport follows the same shape:

- The **sampler** is a `Tick()`-style function called from the emulation loop.
  It copies, compares and pushes **raw bytes** into the ring buffer. No protobuf
  encoding, no I/O.
- The **endpoint** is a `ViewportCommand`. Its `Execute()` runs on the
  emulation thread and copies out the frames newer than `since`; the handler
  encodes them to protobuf on the webserver thread.
- Because only the emulation thread ever touches the ring buffer, **it needs no
  mutex**. This is the point of using the bridge. (An earlier sketch of this
  story had a mutex; that was wrong.)
- The buffer has a fixed size, so memory is bounded however slow a client is.

The costs of the bridge, to measure and accept:

- A request waits for the emulation loop. `WaitForCompletion()` times out
  after 250 ms by default and the handler then fails with "Failed to execute
  command: timeout". Copying 64 frames of 256 bytes is tiny, so this should
  not bite; a 4 KB window times 64 frames is 256 KB, which is still fine.
- `ProcessRequests()` also runs while the emulator is **paused**, so the
  endpoint keeps working then, which is useful (read the frames around a hit
  that paused the game). The *sampler*, like `Signatures::Tick()`, should do
  nothing while paused: memory is frozen, so there is nothing new to see.

## Acceptance

- With the setting off (the default), no sampler exists and no endpoint is
  registered or costs anything.
- With it on, a client polling with `since=` sees each changed window once,
  in order, and no frame for an unchanged window.
- Hold the emulator at a program that rewrites the window constantly, with a
  client that never reads: memory use stays flat and emulation speed is
  unaffected (measure frames per second before and after).
- A gtest unit test covers the ring buffer: wrap-around, `since` past the
  end, and `since` older than the oldest frame still held.
- Sampling is skipped while paused, and the endpoint still answers.
- The mouse position rides on each frame (`MOUSEDOS_GetPosition()`).
- Size is capped (suggest 4096 bytes) and the interval has a floor (50 ms, as
  signatures do).
- Reference pages are written: setting, endpoint, message.

## Notes

- Keep the buffer holding raw copies, not messages, so encoding never runs on
  the emulation thread.
- *Guess:* 64 frames of 256 bytes is a sensible default. Make both settings
  once someone has a reason to change them; not before.
- The sampler needs the pause state; `DOSBOX_IsPaused()` is what `Tick()` uses.
- An HTTP client that polls faster than the sampler interval just gets empty
  answers; say so in the reference page.
