# EPIC-001: Protobuf output from the fork

**Status:** Proposed (2026-10-08). Not started.
**Requested by:** [Kevin](https://github.com/KevinAsbury), 2026-10-08, "for funsies":
a play project turning the Apocrypha tools into protobuf versions.
**Sibling epics:** [gama EPIC-001](https://github.com/ApocryphaCipher/gama/blob/main/docs/epics/EPIC-001-protobuf-pipeline.md),
[Evi EPIC-001](https://github.com/ApocryphaCipher/evi/blob/main/docs/epics/EPIC-001-protobuf-export.md)

## Goal

Let the fork emit its observations as **Protocol Buffers** as well as JSON,
and add one new thing that protobuf suits well: a **viewport**, a small
window of memory sampled at intervals and served as a stream of compact
frames.

This is a learning and tidiness project, not an optimisation. At this volume
JSON is fast enough. The real gains are:

- a written, checked **contract** for what the fork emits (today the hit and
  file-call formats live in the code and in [reference/](../reference));
- `window` and `old`/`new` as raw `bytes` instead of hex text (three characters
  per byte in JSON);
- a second language reading the same bytes (Python in gama, Elixir in Mirror).

## Decisions made

- **JSON stays the default.** Protobuf is opt-in by setting. Nothing that works
  today changes.
- **The fork writes protobuf itself**, rather than gama translating, if the
  [STORY-001](../stories/STORY-001-spike-libprotobuf-in-the-build.md) spike says
  the build cost is acceptable. If it isn't, the fallback is a hand-encoded wire
  format for the two or three messages (small, since protobuf's wire format is
  simple) with the `.proto` file kept as the contract.
- **Files are length-delimited streams**: each record is a varint length
  followed by that many bytes of message. HTTP bodies use
  `Content-Type: application/x-protobuf`.
- **Proposed names** (nothing is fixed until a story is done): package
  `apocrypha.v1`, messages `SignatureHit`, `FileCall`, `Viewport`.
- **The contract's home is undecided.** One `.proto` set shared by the fork,
  gama and Evi would live in its own repo (working name `apocrypha-proto`); see
  gama's STORY-001. Until it exists, the fork's stories carry the message
  sketches.

## Stories, in order

1. [STORY-001](../stories/STORY-001-spike-libprotobuf-in-the-build.md): spike,
   libprotobuf in the build (or not)
2. [STORY-002](../stories/STORY-002-signature-hits-as-protobuf.md): signature
   hits as protobuf
3. [STORY-003](../stories/STORY-003-file-calls-as-protobuf.md): file calls as
   protobuf
4. [STORY-004](../stories/STORY-004-viewport-sampler-and-ring-buffer.md):
   viewport sampler and ring buffer, fixed address
5. [STORY-005](../stories/STORY-005-viewport-follow-a-value.md): viewport that
   follows a value
6. [STORY-006](../stories/STORY-006-viewport-follow-a-signature.md): viewport
   that follows a signature
7. [STORY-007](../stories/STORY-007-protobuf-content-negotiation.md): protobuf
   on the existing endpoints (stretch)

STORY-002 is the smallest slice that proves the whole idea: the Surveyor loop
already produces mouse-tagged hits, so it only changes how they are encoded.
The viewport (004 to 006) is new behaviour and comes after.

## Out of scope

- Replacing JSON for anything a person reads.
- A gRPC server inside DOSBox. Streaming is gama's job (gama STORY-006).
- Writing to emulated memory over protobuf.
- Anything specific to one game. Anchors and signatures take addresses and
  numbers, never game names.

## Risks

- **Rebase cost.** A new dependency and generated code touch `vcpkg.json` and
  the CMake files, outside `src/webserver/`. Keep that to a few lines.
- **Build time.** libprotobuf and `protoc` add to a clean vcpkg build. The
  spike measures it.
- **Every commit must compile** on this repo (`scripts/tools/compile-commits.sh`),
  so the dependency and the code that uses it can't be split carelessly.
