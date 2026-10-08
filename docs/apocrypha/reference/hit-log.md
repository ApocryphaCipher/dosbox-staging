# The hit log (`hits.jsonl`)

When `webserver_signature_dir` is set, the fork appends one JSON object per
line to `hits.jsonl` in that directory each time a
[signature](signature-files.md) hits. The file is flushed after every line, so
a reader can tail it. The same hit is also written to DOSBox's log as
`SIGNATURES: '<name>' at 0x<address>`.

Source: `log_hit()` and `scan()` in
[`src/webserver/signatures.cpp`](../../../src/webserver/signatures.cpp).

## Fields on every line

| Field | Type | Meaning |
| --- | --- | --- |
| `seq` | integer | counts hits from 0 in this launch. **Restarts at 0 each launch**, and the file is appended to, so it is not unique across launches |
| `t` | string | UTC time with milliseconds, `2026-09-24T15:41:33.125Z` |
| `signature` | string | the signature's `name` |
| `kind` | string | `pattern` or `watch` |
| `address` | integer | linear address of the hit (a watch: the record's address) |
| `window_start` | integer | linear address of the first byte of `window` |
| `window` | string | memory around the hit as upper-case hex bytes separated by spaces. Up to the signature's `window` size, centred on `address`, and shorter at the ends of memory |
| `mouse` | `[x, y]` | the DOS mouse driver's position (INT 33h, AX=3) when the hit happened. **Absent when no DOS mouse driver is active** |

## Extra fields on a `watch` hit

| Field | Meaning |
| --- | --- |
| `index` | which record of the table changed |
| `old`, `new` | the record's bytes before and after, as hex |
| `old_value`, `new_value` | the same, read as a little-endian number (only the first 4 bytes of a larger record) |

A `pattern` hit has no extra fields.

## Bulk lines

When a table of 8 or more records has more than half its records change in
one scan, the fork writes **one** line for the whole table instead of one per
record:

| Field | Meaning |
| --- | --- |
| `bulk` | always `true` |
| `changed` | how many records changed |
| `count` | the table's record count |

On a bulk line `address` is the table's first record. The program has reloaded
or reused that memory (a save loading, or a battle reusing the city table). A
bulk line never triggers the signature's `screenshot` or `pause` actions.

## While paused

`Signatures::Tick()` is called while the emulator is paused, but it scans
nothing then; it only takes a screenshot left pending by a hit that paused the
game. Memory is frozen while paused, so there is nothing new to find. After
`POST /api/v1/dosbox/resume`, scanning carries on from the last baseline.

## Example

The first line of a real log, from a watch on Master of Magic's unit count
(address 0x34782, a `u16`; see gama's `layout.py`). It has no `mouse` field, so
no DOS mouse driver was active. Keys come out in alphabetical order, not the
order of the tables above:

```json
{"address":214914,"index":0,"kind":"watch","new":"00 C7","new_value":50944,"old":"00 00","old_value":0,"seq":0,"signature":"unit-count","t":"2026-09-24T18:15:56.617Z","window":"46 FE 00 00 C7 46 F8 00 00 C7 06 2E 8A 00 00 B8","window_start":214906}
```

> *Guess:* a `new_value` of 50944 is too big for a unit count (the table holds
> 1,009 units), so it was probably read while that memory was being set up.
> The line is quoted for its format only. To settle it, look at the next hits
> for `unit-count` in the same log.

## Reading it

[`gama surveyor hits.jsonl`](https://github.com/ApocryphaCipher/gama) lists the
Surveyor's hover texts with their mouse positions from this file. The file holds
a game's own memory, so it is **evidence, not source**: keep it out of git and
file it in an Evi vault.
