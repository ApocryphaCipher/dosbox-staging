# ApocryphaCipher fork: working docs

**Picking this up? Read [notes/2026-10-08-handoff.md](notes/2026-10-08-handoff.md) first.**

This is the **ApocryphaCipher** fork of DOSBox Staging
(`webserver-write-guard` branch). It adds a small, localhost-only web
server and two loggers that let outside tools look inside the running
machine. The goal is software forensics on 90s DOS games, namely
reconstructing long-lost, undocumented code.

The fork is one of four tools:

| Tool | Role |
| --- | --- |
| **this fork** | the instrumented machine: exposes memory, screenshots, registers, DOS file calls, and notices moments |
| [gama](https://github.com/ApocryphaCipher/gama) | decodes the fork's RAM dumps and logs into SQLite |
| [Evi](https://github.com/ApocryphaCipher/evi) | keeps the originals and links claims to evidence |
| Mirror | a Master of Magic save viewer and editor |

All of the fork's code is in [`src/webserver/`](../../src/webserver), plus a
few lines in `src/dosbox.cpp`, `src/dos/dos.cpp`, `src/capture/` and
`src/hardware/input/` (see the [handoff](notes/2026-10-08-handoff.md)).
Keeping it in one directory is deliberate: it keeps rebasing on
`upstream` cheap.

## Where things are

These are working docs, not part of the user manual. Upstream's own files
in [`docs/`](..) and the MkDocs site in [`website/`](../../website) are
untouched.

- [`reference/`](reference): what the fork does *now*, checked against the code
  - [settings.md](reference/settings.md): the `webserver_*` settings
  - [http-api.md](reference/http-api.md): every HTTP endpoint
  - [signature-files.md](reference/signature-files.md): the JSON that tells the fork what to watch
  - [hit-log.md](reference/hit-log.md): the `hits.jsonl` lines it writes
  - [file-call-log.md](reference/file-call-log.md): the DOS file-call log
- [`epics/`](epics): large bodies of work, one file each, status at the top
- [`stories/`](stories): units of work under an epic, indexed in [stories/README.md](stories/README.md)
- [`backlog.md`](backlog.md): unsorted ideas and known issues
- [`notes/`](notes): dated session notes; the newest handoff is the entry point

## Rules this fork keeps

- **Localhost only.** The server refuses non-loopback bind addresses.
- **Read-only by default.** Anything that changes emulator state
  (`memory` writes, allocate, free, shutdown) needs
  `webserver_allow_writes = true`.
- **No game data in the repo.** Memory dumps and hit logs hold a game's
  own code and data. Never commit them.
- **Stay in `src/webserver/`.** A change outside it needs a reason, because
  it is where rebase conflicts come from.
- **Follow the repo's own rules** in [`.claude/rules/`](../../.claude/rules):
  C++ style, commit prefixes (`docs:`, `build:`, and so on, no `feat:` or `fix:`),
  every commit must compile, and British spelling in docs. This fork's
  commits carry no `Co-Authored-By` line.

## Checking docs

`scripts/linting/verify-markdown.sh` runs `mdl` over every tracked `*.md`
file, which includes these. `mdl` is not installed on the machine these
were written on, so run the script before opening a pull request.
