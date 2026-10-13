# STORY-008: An endpoint that says where each DOS program was loaded

**Status:** Not started. **Priority: nice to have** (found 2026-10-13, see below).
**Size:** small to medium

## The idea

A tool reading a DOS game's memory has to know *where* the game's data sits. Today it scans the 16 MiB for something it
recognises, because nothing in the API says where a program was loaded. Add a read-only endpoint, say `GET /api/v1/dos/memory-map`, that
returns DOS's memory-control-block (MCB) chain: for each block its segment, size in paragraphs, owner (the owning program's PSP segment) and the owner's name
when DOS has one. A client that knows the program's layout (its data segment is a fixed number of paragraphs above its load segment) can then compute the
address directly and use a scan only as a cross-check.

## Why it matters (found 2026-10-13, mompy)

The same game's tables sat **`0x520` bytes lower** in DOSBox Staging 0.83.0 than in this fork's 0.84-alpha. The cause is DOS `STACKS` support
(`src/dos/dos_stacks.cpp`, added upstream 2026-04-28; the default `stacks = auto` reserves 81 paragraphs plus one MCB header = 82 paragraphs = `0x520`), which
moves everything loaded after it, and a non-default `stacks` setting moves it again. So a program's load address depends on the DOSBox build and its settings,
and any tool that hard-codes addresses breaks. `mompy doctor` already finds the table by scanning; this endpoint would make that exact.

## What to do

1. Walk the MCB chain the way the DOS code already does (`DOS_MCB` in `src/dos/dos_memory.cpp`), starting from the first MCB the "List of Lists" gives
   (`/api/v1/dos/internals` already reports `listOfLists`).
2. Return JSON: `[{"segment": ..., "paragraphs": ..., "owner": ..., "name": "..."}]`, in chain order, plus the current PSP segment.
3. Read-only, so it is not behind `webserver_allow_writes`. Update `resources/webserver/index.html` and [reference/http-api.md](../reference/http-api.md).

## Definition of done

The endpoint lists the shell, the loaded game and the `STACKS` block on a real run; a client computes the game's data segment from it and it equals the segment
`cpu/state` reports for `SS` while the game runs. A small test of the chain walk if the codebase has a place for one. Offer it upstream as its own pull request (it needs none of this fork's other changes).
