# Backlog

Unsorted, not yet promoted to a story. Each item says how it was found.

- **`Tick()`'s header comment could say more.** `signatures.h` says `Tick()` is
  "called from the emulation loop (also while paused); scans when the interval
  has passed". That is true: `paused_tick()` in `dosbox.cpp` does call it. But
  while paused it scans nothing, it only takes a pending screenshot (memory is
  frozen, so a scan would find nothing). A clause saying so would stop the next
  reader wondering. *Found 2026-10-08 reading the code; first read as a
  mismatch, then corrected after finding `paused_tick()`.*
- **The built-in API page is out of date.**
  `resources/webserver/index.html` documents nine endpoints and omits
  `capture/screenshot`, `dosbox/pause`, `dosbox/resume` and
  `signatures/reload`. [reference/http-api.md](reference/http-api.md) is the only
  description of those four. Adding them to the page is the right fix, and
  upstream's docs rules apply (match the existing markup).
- **`seq` is not unique across launches.** `hits.jsonl` is opened in append
  mode but `seq` restarts at 0 on each launch, so two sessions interleave
  ambiguously. A launch id on every line fixes it (see STORY-002).
- **A bad array entry drops the rest of its file.** `Reload()` keeps the
  entries before the one that failed and skips the ones after it, with a
  single warning. Probably better: skip only the bad entry, or reject the
  whole file. Behaviour change, so decide before touching it.
- **Pattern hits stop at 64 per scan without saying so.** `MaxHitsPerScan`
  silently truncates; a log line saying "N more hits not logged" would stop
  anyone trusting an undercount.
- **Bulk lines never pause or take screenshots.** Deliberate (a save loading
  would otherwise pause the game), but it is not in the help text. Add a
  sentence to the `webserver_signature_dir` help.
- **Field names of `cpu/state` and `dos/internals`** are undocumented
  outside the code. Write them into [reference/http-api.md](reference/http-api.md).
- **`mdl` has not been run on these docs.** It isn't installed on the machine
  they were written on. Run `scripts/linting/verify-markdown.sh`.
