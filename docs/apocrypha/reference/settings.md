# Settings

All are in the fork's web server config section, read **at start only**
(`OnlyAtStart`). Set them in the config file or on the command line:

```bash
dosbox --set webserver_enabled=true --set webserver_signature_dir=$HOME/signatures
```

Source: `init_config_settings()` in
[`src/webserver/webserver.cpp`](../../../src/webserver/webserver.cpp).

| Setting | Type | Default | What it does |
| --- | --- | --- | --- |
| `webserver_enabled` | bool | `false` | Starts the HTTP API. |
| `webserver_bind_address` | string | `127.0.0.1` | Must be a loopback address (`127.0.0.1`, `::1`, `localhost`). Anything else is refused and the server does not start. |
| `webserver_port` | int, 1 to 65535 | `8086` | TCP port. |
| `webserver_allow_writes` | bool | `false` | Allows requests that change emulator state: memory writes, allocate, free and shutdown. When off they answer `403`. |
| `webserver_file_log` | string | empty (off) | A file to log every DOS file call to, as JSON Lines. See [file-call-log.md](file-call-log.md). |
| `webserver_signature_dir` | string | empty (off) | A directory of signature `*.json` files to watch memory for. Hits are appended to `hits.jsonl` in it. See [signature-files.md](signature-files.md). |
| `webserver_signature_interval` | int ms, 50 to 60000 | `500` | How often memory is scanned for signatures. |

## Notes

- `webserver_file_log` and `webserver_signature_dir` **work without**
  `webserver_enabled`. The loggers do not need the HTTP server.
- With `webserver_signature_dir` set, `hits.jsonl` is opened in **append**
  mode, so a restart adds to the file. Hit `seq` numbers restart at 0 each
  launch, so `seq` alone is not unique across launches.
- A scan does nothing while the emulator is paused (see
  [hit-log.md](hit-log.md#while-paused)).
- The built-in help text for each setting is the authoritative wording; if
  it and this page disagree, the code wins and this page is wrong.
