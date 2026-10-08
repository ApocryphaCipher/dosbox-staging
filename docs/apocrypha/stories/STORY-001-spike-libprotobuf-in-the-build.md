# STORY-001: Spike: is libprotobuf worth having in the build?

**Parent:** [EPIC-001](../epics/EPIC-001-protobuf-output.md)
**Status:** Not started
**Size:** small to medium (one build experiment and a write-up)

## The idea

Everything else in the epic depends on one question: can the fork emit
protobuf without a build cost that outweighs the fun? Answer it with
numbers before writing any real encoder.

## What to try

Three options, cheapest to try first. Stop at the first that is acceptable.

1. **libprotobuf from vcpkg.** Add `protobuf` to `vcpkg.json`, generate code
   with CMake's `protobuf_generate()`, and use `optimize_for = LITE_RUNTIME`
   in the `.proto` to get the smaller runtime. *Guess:* the lite runtime is
   enough, since the fork only ever serialises. Check it builds.
2. **Hand-encoded wire format.** No dependency. The wire format is a varint
   key per field, so a few messages need on the order of 100 lines in
   `src/webserver/`, with the `.proto` kept as the contract and a test that
   decodes the output with real protobuf code in another language.
3. **A tiny generator such as nanopb.** Probably not worth it; try only if 1
   and 2 are both rejected.

## What to measure

For each option tried, on macOS (the maintainer's machine) and, via CI or a
second machine, Linux and Windows:

- a **clean** vcpkg build time, before and after;
- the size of the `dosbox` binary, before and after;
- whether `protoc` is available as a vcpkg host tool on every platform;
- how many lines change **outside** `src/webserver/` (the rebase cost);
- that each commit compiles on its own (`scripts/tools/compile-commits.sh`).

## Acceptance

- A note in `docs/apocrypha/notes/` with the numbers and a **decision**:
  libprotobuf, hand-encoded, or drop the epic.
- The spike branch is thrown away or squashed; nothing half-done is merged.
- If the answer is "drop it", close the epic with the reason. That is a fine
  outcome.

## Notes

- The vcpkg baseline was last bumped on 2026-09-20 (`ceb8cc7b4`); a new port
  might need another bump, which touches more than the webserver.
- The fork already vendors `json` (nlohmann) and `httplib` under `src/libs/`.
  Vendoring a protobuf runtime there is a fourth option, not recommended: it
  is large.
