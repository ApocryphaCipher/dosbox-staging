# STORY-007: Protobuf on the existing endpoints

**Parent:** [EPIC-001](../epics/EPIC-001-protobuf-output.md)
**Status:** Not started, stretch. Depends on [STORY-002](STORY-002-signature-hits-as-protobuf.md).
**Size:** small

## The idea

`GET /api/v1/memory/...` already negotiates its format: raw bytes by default,
JSON with Base64 for `Accept: application/json`
(see [http-api.md](../reference/http-api.md)). Extend the same trick to
`Accept: application/x-protobuf` on the read endpoints, as a tidy way to show
that one URL can serve several formats.

## What changes

For `dosbox/info`, `cpu/state` and the memory read, answer with a protobuf
message when the request asks for it:

```proto
message CpuState { uint32 eax = 1; uint32 ebx = 2; /* ... */ uint32 eip = 9; /* ... */ }
message MemoryRead { uint32 addr = 1; bytes data = 2; }
```

JSON stays the default, and the memory read keeps its raw default.

## Acceptance

- A request without the new `Accept` value behaves exactly as before.
- `Accept: application/x-protobuf` returns a message that a Python client
  decodes with the generated code.
- `MemoryRead.data` is the raw bytes, not Base64 text: about a quarter smaller
  than the JSON form (Base64 turns 3 bytes into 4 characters).
- Reference page updated.

## Notes

- The `CpuState` field list must come from the real register set in `cpu.cpp`,
  not from memory. Write it into [http-api.md](../reference/http-api.md) at the
  same time (the backlog notes it is undocumented today).
- Not worth doing unless something consumes it. gama's `checkpoint` is the
  candidate: it fetches `cpu/state` and stores it as `.cpu.json`.
