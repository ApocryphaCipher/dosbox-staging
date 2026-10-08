# STORY-002: Signature hits as protobuf

**Parent:** [EPIC-001](../epics/EPIC-001-protobuf-output.md)
**Status:** Not started. Depends on [STORY-001](STORY-001-spike-libprotobuf-in-the-build.md).
**Size:** medium

## The idea

The hit log is the loop that already works: hover a tile in the Master of
Magic Surveyor, the game rewrites its text slots, a watch fires, and the line
carries the memory window and the DOS mouse position. This story writes the
same hit as a protobuf record, and nothing else. It is the smallest
change that exercises the whole idea.

## What changes

A new setting, working name `webserver_hit_format`:

| Value | Writes |
| --- | --- |
| `json` (default) | `hits.jsonl` only, exactly as today |
| `protobuf` | `hits.pb` only |
| `both` | both files, for comparing |

`hits.pb` is a stream of length-delimited `SignatureHit` records. Sketch:

```proto
syntax = "proto3";
package apocrypha.v1;

import "google/protobuf/timestamp.proto";

message SignatureHit {
  string launch_id = 1;      // fixes "seq restarts each launch"
  uint64 seq = 2;
  google.protobuf.Timestamp time = 3;
  string signature = 4;
  enum Kind { KIND_UNSPECIFIED = 0; PATTERN = 1; WATCH = 2; }
  Kind kind = 5;
  uint32 address = 6;        // linear
  uint32 window_start = 7;
  bytes window = 8;          // raw bytes, not hex text
  optional Mouse mouse = 9;  // absent when no DOS mouse driver
  oneof detail {
    WatchChange change = 10;
    BulkChange bulk = 11;
  }
}

message Mouse { uint32 x = 1; uint32 y = 2; }
message WatchChange { uint32 index = 1; bytes old = 2; bytes new = 3; }
message BulkChange { uint32 changed = 1; uint32 count = 2; }
```

The `old_value`/`new_value` fields of the JSON are left out on purpose: they
are `old`/`new` read as a little-endian number, which any reader can compute.
They are not unused: gama's `surveyor.hovers()` reads `new_value` for the
`map-plane` signature
([fork-logs-read-by-gama.md](https://github.com/ApocryphaCipher/gama/blob/main/docs/reference/fork-logs-read-by-gama.md)).
So the protobuf reader in gama has to compute it (gama's STORY-002 covers
that). If that turns out to be a nuisance, put the fields back; they cost
a few bytes.

## Acceptance

- With `json`, output is byte-for-byte what it is today.
- With `protobuf`, every JSON field has a protobuf equivalent, and a Python
  script using the generated code reads `hits.pb` and agrees with a
  `hits.jsonl` made in the same run (`both`): same count, same addresses,
  same windows.
- `protoc --decode=apocrypha.v1.SignatureHit` works on a record.
- A gtest unit test covers encoding of each variant: pattern, watch, bulk,
  with and without a mouse.
- The hit is flushed per record, as the JSON one is, so a reader can tail it.
- [reference/hit-log.md](../reference/hit-log.md) documents the new file and
  setting; [reference/settings.md](../reference/settings.md) gets the row.

## Notes

- `launch_id` is new information, not a translation of an existing field. It is
  the fix for the backlog item that `seq` is not unique across launches. A
  random 64-bit value, or the start time, would do.
- Measure and record the size of `hits.pb` against `hits.jsonl` from the same
  run. Hex text is three characters per byte, so expect `window`, `old` and
  `new` to shrink to about a third. Write down what is actually seen
  ([gama STORY-007](https://github.com/ApocryphaCipher/gama/blob/main/docs/stories/STORY-007-measure-json-against-protobuf.md)
  collects the numbers).
