# STORY-009: An endpoint for the DOS mouse position

**Status:** Not started. **Priority: nice to have** (raised 2026-10-13 by mompy's STORY-010, `mompy where`).
**Size:** small

## The idea

`GET /api/v1/dos/mouse` returns what the DOS mouse driver reports for INT 33h AX=3: the cursor position (x 0 to 639, y 0 to 199 in the driver's coordinates) and the button state, or an
error when no DOS mouse driver is active. The fork already reads this value for its signature log (`src/hardware/input/mouse.h`, added in the commit "Record the DOS mouse position with every signature hit";
`reference/hit-log.md` documents the `mouse` field), so the endpoint is a thin wrapper around an accessor that exists.

## Why

A tool that wants to know where the player is pointing (mompy's `where`: the map tile under the cursor of a running game) has to read the mouse, and the web API offers no way to. Another route is to find the game's own copy in
its memory (mompy's research step), which would work on stock DOSBox; this endpoint is the fallback and the general answer for any DOS game.

## What to do

1. Add `MouseCommand::Get` beside the other read-only commands (`src/webserver/`), returning `{"x": ..., "y": ..., "buttons": ...}`; a 404-style JSON error when no driver is active.
2. Register it as a read-only route (not behind `webserver_allow_writes`); document it in `resources/webserver/index.html` and [reference/http-api.md](../reference/http-api.md).
3. Offer it upstream as its own pull request (it needs nothing else from this fork).

## Definition of done

Moving the mouse changes the answer; the position agrees with the `mouse` field of a signature hit at the same moment; no driver gives the error.
