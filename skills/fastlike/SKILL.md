---
name: fastlike
description: "Run Fastly Compute WASM applications locally with Fastlike (Go-based). Use when you need to start a local development server for WASM, configure backends, stores, and ACLs, or test Compute apps without deploying. Fastlike is a Go-based alternative to Viceroy."
---

# Fastlike

Fastlike is a Go implementation of the Fastly Compute ABI. It runs compiled WebAssembly programs locally, implementing the same host functions that Fastly's production Compute platform provides: backends, dictionaries, KV stores, caching, geolocation, and more.

**Fastlike documentation**: https://github.com/avidal/fastlike

## Install from Source

Requires Go 1.23+.

```bash
# Clone and build
git clone https://github.com/avidal/fastlike.git
cd fastlike
make build        # Creates bin/fastlike

# Or install to GOPATH/bin
make install

# Or install directly
go install fastlike.dev/cmd/fastlike@latest
```

## Quick Start

```bash
# Minimal: WASM + single backend
bin/fastlike -wasm app.wasm -backend localhost:8000
```

## Common Configurations

**With named backends:**
```bash
bin/fastlike -wasm app.wasm \
  -backend api=api.example.com:8080 \
  -backend cache=redis:6379 \
  -backend localhost:8000
```

**Development mode with hot-reload:**
```bash
bin/fastlike -wasm app.wasm -backend localhost:8000 -reload -v 2
```
Send `SIGHUP` to reload the WASM without restarting.

**Full configuration:**
```bash
bin/fastlike -wasm app.wasm \
  -bind 0.0.0.0:5000 \
  -backend localhost:8000 \
  -dictionary config=./config.json \
  -kv store=./data.json \
  -config-store settings=./settings.json \
  -secret-store secrets=./secrets.json \
  -acl blocklist=./acl.json \
  -logger output=./logs.txt \
  -geo ./geodata.json \
  -compliance-region us-eu \
  -v 2 \
  -reload
```

## Required Flags

| Flag                           | Description                            |
| ------------------------------ | -------------------------------------- |
| `-wasm PATH`                   | Path to WebAssembly program (required) |
| `-backend VALUE` or `-b VALUE` | Backend server (required, repeatable)  |

## Optional Flags

| Flag                            | Default          | Description                         |
| ------------------------------- | ---------------- | ----------------------------------- |
| `-bind ADDR`                    | `localhost:5000` | Server bind address                 |
| `-reload`                       | false            | Enable SIGHUP hot-reload            |
| `-v INT`                        | 0                | Verbosity (0-2)                     |
| `-dictionary NAME=FILE` or `-d` | -                | Load dictionary from JSON           |
| `-kv NAME[=FILE]`               | -                | KV store (empty or from JSON)       |
| `-config-store NAME=FILE`       | -                | Config store from JSON              |
| `-secret-store NAME=FILE`       | -                | Secret store from JSON              |
| `-acl NAME=FILE`                | -                | ACL from JSON                       |
| `-logger NAME[=FILE]`           | -                | Log endpoint (file or stdout)       |
| `-geo FILE`                     | -                | Geolocation JSON file               |
| `-compliance-region REGION`     | -                | Compliance region (none, us-eu, us) |

## Source Code as Reference

The fastlike source code is useful for understanding how Fastly's platform primitives work under the hood. The ABI functions map directly to the operations available in both Compute and VCL:

| Concept                        | Source Files                               | What You Learn                                                      |
| ------------------------------ | ------------------------------------------ | ------------------------------------------------------------------- |
| Request/response lifecycle     | `instance.go`, `xqd_req.go`, `xqd_resp.go` | How requests flow through Fastly, header manipulation, status codes |
| Backend subrequests            | `backend.go`, `xqd_backend.go`             | How origin fetches work, backend selection, health checks           |
| Dictionaries and config stores | `xqd_dictionary.go`, `xqd_config_store.go` | Edge dictionary lookups (same as VCL `table` lookups)               |
| KV store                       | `xqd_kv_store.go`                          | Key-value storage operations, async patterns                        |
| Caching                        | `xqd_cache.go`                             | Cache keys, TTLs, stale-while-revalidate, surrogate keys            |
| Geolocation                    | `xqd_geo.go`                               | IP-to-geo lookups (same data as VCL `client.geo.*`)                 |
| Body handling                  | `xqd_body.go`                              | Streaming bodies, body append, body read/write                      |
| Logging                        | `xqd_log.go`                               | Log endpoint writes (same as VCL log statements)                    |

The code uses C-style signatures (not idiomatic Go) to match Fastly's Rust reference implementation, making it easy to cross-reference with Fastly's documentation.

## References

| Topic    | File                                  | Use when...                                                                        |
| -------- | ------------------------------------- | ---------------------------------------------------------------------------------- |
| Backends | [backends.md](references/backends.md) | Setting up named backends, catch-all backends, microservices routing               |
| Config   | [config.md](references/config.md)     | Creating JSON config files for dictionaries, KV stores, secrets, ACLs, geolocation |
| Build    | [build.md](references/build.md)       | Building Fastlike from source, running linters, make targets                       |
| Test     | [test.md](references/test.md)         | Running Go tests, Fastly Compute ABI spec tests                                    |
| ABI      | [abi.md](references/abi.md)           | Fastly Compute ABI internals, implementing new ABI functions, handle system        |
