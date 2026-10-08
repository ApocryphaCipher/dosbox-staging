# HTTP API

Enabled by `webserver_enabled`; see [settings.md](settings.md). It listens on
`127.0.0.1:8086` by default and answers JSON unless the table says
otherwise.

Sources: `setup_api_handlers()` in
[`src/webserver/webserver.cpp`](../../../src/webserver/webserver.cpp), the command
files beside it, and the page the server itself shows at `/`
([`resources/webserver/index.html`](../../../resources/webserver/index.html)).
That page documents only the original upstream-style endpoints. **It does not
list `capture/screenshot`, `dosbox/pause`, `dosbox/resume` or
`signatures/reload`**; this page is the only description of those four.

Numeric URL parameters are decimal, or hexadecimal with a `0x` prefix.

## How a request runs

Requests that touch the machine do not run on the HTTP thread. The handler
queues a `Command` on the **bridge** ([`bridge.h`](../../../src/webserver/bridge.h))
and waits; the emulation thread runs it from `Bridge::ProcessRequests()` once
per pass of its loop, and the handler then builds the response. This also
happens while the emulator is **paused**, so the API still answers then.

If the emulation thread doesn't get to the command within **250 ms**, the
request fails with `Failed to execute command: timeout`.

## Read-only endpoints

| Method and path | Returns |
| --- | --- |
| `GET /api/v1/dosbox/info` | JSON with `configHome`, `configWebserver` and `version` |
| `GET /api/v1/cpu/state` | the CPU registers |
| `GET /api/v1/dos/internals` | DOS internals |
| `GET /api/v1/memory/:offset/:len` | `len` bytes of emulated memory from a linear address |
| `GET /api/v1/memory/:segment/:offset/:len` | the same, from a segment. `segment` is a number or a register name (`CS`, `SS`, `DS`, `ES`, `FS`, `GS`) |
| `POST /api/v1/capture/screenshot` | takes a screenshot. `?type=raw`, `upscaled` or `rendered` (anything else is an error). Answers `{"path": ...}`, or the PNG itself with `?inline=1`. Waits up to 5 s for the file to appear |
| `POST /api/v1/dosbox/pause` | pauses the emulator |
| `POST /api/v1/dosbox/resume` | resumes it |
| `POST /api/v1/signatures/reload` | re-reads the signature files and takes a new baseline; says how many were loaded |

`len` must be 1 to 134,217,728 (128 MiB) per request. A range that runs past
the end of emulated memory is an **error**, not clamped.

By default a memory read returns the raw bytes as a download
(`Content-Disposition: attachment; filename="memory.bin"`). Send
`Accept: application/json` to get JSON instead: `registers`, and
`memory.addr` and `memory.data` (Base64).

A full 16 MB dump is `GET /api/v1/memory/0/16777216`, which is what
[gama](https://github.com/ApocryphaCipher/gama) does.

The pause, resume, screenshot and reload `POST`s do not change the emulated
machine's memory, so they are **not** behind the write guard.

> *Guess:* the JSON field names of `cpu/state` and `dos/internals` are not
> written down here. They are in `cpu.cpp` and `dos.cpp`; copy them in
> when someone needs them.

## Write-guarded endpoints

These answer `403` with a JSON error unless `webserver_allow_writes = true`.

| Method and path | Does |
| --- | --- |
| `PUT /api/v1/memory/:offset` | writes the body into memory at a linear address |
| `PUT /api/v1/memory/:segment/:offset` | the same, from a segment |
| `POST /api/v1/memory/allocate` | allocates DOS memory |
| `POST /api/v1/memory/free` | frees DOS memory |
| `POST /api/v1/dosbox/shutdown` | quits DOSBox |

A memory `PUT` takes raw bytes (`Content-Type: application/octet-stream`) or
JSON with a Base64 `data` field. A write past the end of memory is an error.

## Security

There is no authentication. The loopback-only bind is the access control, and
the server also rejects any request whose `Host` header isn't the bind
address (or `localhost`), which stops DNS-rebinding attacks from a web page
(`setup_host_validation()`).

The upstream page warns that the API has been tested mainly with
`core = normal` and 16-bit real-mode programs.
