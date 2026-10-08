# Signature files

A **signature** tells the fork what to notice in emulated memory. Put
them in `*.json` files in the directory named by `webserver_signature_dir`
(see [settings.md](settings.md)). Each file holds one signature object or an
array of them. The fork reads every `*.json` file in the directory and
re-reads them on `POST /api/v1/signatures/reload`.

Source: `parse()` and `Reload()` in
[`src/webserver/signatures.cpp`](../../../src/webserver/signatures.cpp).

**A bad file is skipped, not fatal.** Files load in file-name order. If a file
is not valid JSON, or a signature in it fails to parse (a missing `name`, a
pattern with no fixed byte, a watch with `size` 0 or over 256), DOSBox logs
`SIGNATURES: Skipping '<path>': <reason>` and moves on. In an array file,
the entries **before** the bad one stay loaded and the ones after it are
dropped. If a signature seems silent, read the log for that line, and check
the `Loaded N signatures` message to see how many made it.

Numbers may be JSON numbers or strings; strings may be hex (`"0x35F80"`).

## Common fields

| Field | Required | Meaning |
| --- | --- | --- |
| `name` | yes | the name that appears in each hit |
| `kind` | no | `"pattern"` (the default) or `"watch"` |
| `window` | no | bytes of memory to include around a hit. Default 32, at most 1024 |
| `actions` | no | list of `"screenshot"` and/or `"pause"`, done when the signature hits |
| `note` | no | free text for people; ignored by the fork |

## `pattern`: bytes or text that appear

| Field | Meaning |
| --- | --- |
| `hex` | bytes as hex separated by spaces or commas, with `??` as a wildcard, e.g. `"4D 5A ?? ??"`. Needs at least one fixed byte |
| `text` | instead of `hex`: ASCII text to find |
| `start`, `end` | optional address range to search; default is all of memory |

A pattern hits when a match appears at an address where it was **not** there
on the previous scan. At most **64 hits per signature per scan** are logged.

## `watch`: an address or table that changes

| Field | Meaning |
| --- | --- |
| `address` | the first record's address |
| `size` | bytes per record, 1 to 256 |
| `stride` | bytes between records (default 0) |
| `count` | number of records (default 1), 1 to 4096 |

A watch hits for each record whose bytes changed since the last scan. A
table of 8 or more records where **more than half** change in one scan
logs a single **bulk** line instead (the program reloaded or reused that
memory); see [hit-log.md](hit-log.md#bulk-lines).

## Baseline

The first scan after loading (or reloading) is silent: it only records what
is there. Only changes after that are reported.

## Examples

The Master of Magic signatures live in
[gama's `signatures/mom/`](https://github.com/ApocryphaCipher/gama/tree/main/signatures/mom).
A minimal watch:

```json
{
  "name": "surveyor-text",
  "kind": "watch",
  "address": "0x35F80",
  "size": 256,
  "window": 16
}
```

A pattern that pauses the game and takes a screenshot:

```json
{ "name": "save-header", "kind": "pattern", "hex": "53 41 56 45 ?? ??",
  "actions": ["pause", "screenshot"] }
```

(That second one is an illustration, not a signature anyone uses.)

## Behaviour to know

- Addresses are **linear** (physical) addresses, not `segment:offset`.
- A signature may only read memory; none of the actions change it.
- `screenshot` and `pause` are requested when a hit happens, **except for
  bulk lines**, which never trigger them.
- `pause` stops the emulator; resume it with
  `POST /api/v1/dosbox/resume`. With both actions, the screenshot is taken
  once the pause has taken hold, so it shows the moment of the hit.
