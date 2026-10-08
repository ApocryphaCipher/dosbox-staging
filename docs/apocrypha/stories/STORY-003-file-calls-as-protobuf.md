# STORY-003: DOS file calls as protobuf

**Parent:** [EPIC-001](../epics/EPIC-001-protobuf-output.md)
**Status:** Not started. Depends on [STORY-002](STORY-002-signature-hits-as-protobuf.md) (it reuses its setup).
**Size:** small to medium

## The idea

The file-call log ([reference/file-call-log.md](../reference/file-call-log.md))
is the second JSON Lines format the fork writes. `buffer` plus `position` is
what `gama filemap` uses to place a save file in memory. Give it the same
protobuf treatment as the hits, as a second, independent use of the machinery.

## What changes

A setting, working name `webserver_file_log_format` (`json` default,
`protobuf`, `both`), writing a stream of length-delimited `FileCall` records
to the path given by `webserver_file_log` (with a `.pb` suffix alongside the
`.jsonl` when `both`). Sketch:

```proto
message FileCall {
  string launch_id = 1;
  uint64 seq = 2;
  google.protobuf.Timestamp time = 3;
  enum Op { OP_UNSPECIFIED = 0; CREATE = 1; OPEN = 2; OPEN_EXT = 3;
            CLOSE = 4; READ = 5; WRITE = 6; SEEK = 7; }
  Op op = 4;
  string file = 5;
  bool ok = 6;
  optional uint32 handle = 7;
  optional uint32 buffer = 8;     // physical address of DS:DX
  optional uint32 requested = 9;
  optional uint32 position = 10;  // absent for devices
  optional uint32 done = 11;
  optional uint32 mode = 12;
  optional uint32 attributes = 13;
  optional uint32 whence = 14;
  optional uint32 error = 15;     // DOS error code, on failure
}
```

`optional` keeps "not applicable" apart from zero, which matters here: a
`position` of 0 is a real file offset, and the JSON gets this right only by
leaving the key out. This is the best demonstration in the epic of why protobuf
has `optional`.

## Acceptance

- With `json`, output is unchanged.
- With `protobuf`, [gama](https://github.com/ApocryphaCipher/gama)'s
  `filemap` produces the **same table** from the `.pb` as from the `.jsonl`
  of the same run, on a real Master of Magic save and load.
- A gtest unit test covers each operation, including a failed call and a
  device read with no `position`.
- The reference page and settings table are updated.

## Notes

- The two formats share `launch_id` and the timestamp type with
  [STORY-002](STORY-002-signature-hits-as-protobuf.md). Put the shared pieces in
  one `.proto` file.
- The `*_value` and hex-string conveniences of the hit log have no equivalent
  here, so this story is a plain field-for-field translation.
