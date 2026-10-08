# The DOS file-call log

When `webserver_file_log` names a file, the fork writes one JSON object per
line for every DOS file call a program makes. It is for **mapping a file onto
memory**: a read or write line says which file bytes went to or from which
RAM address. It works without the web server.

Source:
[`src/webserver/file_log.cpp`](../../../src/webserver/file_log.cpp) and
[`file_log.h`](../../../src/webserver/file_log.h). The hook is a few lines in
the INT 21h handler in `src/dos/dos.cpp`.

## Calls logged

`create`, `open`, `open_ext`, `close`, `read`, `write` and `seek`. Other INT 21h
functions are not logged.

## Fields

| Field | On | Meaning |
| --- | --- | --- |
| `seq` | all | counts calls from 0 in this launch |
| `t` | all | UTC time with milliseconds, like the [hit log](hit-log.md) |
| `op` | all | the call, as listed above |
| `file` | all | the file name the program used |
| `ok` | all | whether DOS reported success |
| `attributes` | `create` | CX, the file attributes |
| `mode` | `open`, `open_ext` | AL, the access mode |
| `handle` | `open`, `create` (on success), `close`, `read`, `write`, `seek` | the DOS file handle |
| `buffer` | `read`, `write` | the **physical RAM address** of the buffer (DS:DX) |
| `requested` | `read`, `write` | bytes asked for (CX) |
| `position` | `read`, `write`, `seek` | the file offset: before a read or write (left out for devices such as the console), after a successful seek |
| `done` | `read`, `write` | bytes actually transferred (AX) |
| `whence` | `seek` | AL: 0 from start, 1 from current, 2 from end |
| `error` | failed calls | the DOS error code (AX) |

Which of these appear on which operation is set by the `switch` in
`file_log.cpp`; if a field seems missing, read it there.

## What it is for

`buffer` plus `position` is the link between a save file and RAM. Save and
load a game with the log on, and
[`gama filemap`](https://github.com/ApocryphaCipher/gama) turns the lines into
a table of "this block of the file is at this RAM address", joining blocks
written in several calls. That is how the Master of Magic save blocks were
placed in memory without guessing.

Like the hit log, it is evidence: file it in an Evi vault, don't commit it.
