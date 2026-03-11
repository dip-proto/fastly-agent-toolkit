---
name: xvcl
description: "Write VCL using XVCL transpiler. Use when you need to write new VCL/XVCL code, implement caching logic, route requests, manipulate headers, or use XVCL metaprogramming features (constants, loops, functions). Compile with `uvx xvcl main.xvcl -o main.vcl`."
---

# Writing VCL with XVCL

XVCL is a VCL transpiler that adds metaprogramming features to VCL. Write `.xvcl` files, compile to `.vcl`, then run with Falco locally or deploy to Fastly.

XVCL documentation: https://dip-proto.github.io/xvcl/

## Install

```bash
# Using uv (recommended)
uv pip install xvcl

# Using pip
pip install xvcl
```

## Quick Start

```bash
# Compile XVCL to VCL (no install needed with uvx)
uvx xvcl main.xvcl -o main.vcl

# Lint the output
falco lint main.vcl

# Test locally
falco simulate main.vcl
```

## Minimal Working Example

```xvcl
#const ORIGIN_HOST = "api.example.com"

backend origin {
  .host = "{{ORIGIN_HOST}}";
  .port = "443";
  .ssl = true;
}

sub vcl_recv {
  #FASTLY recv
  set req.backend = origin;
  return (lookup);
}

sub vcl_deliver {
  #FASTLY deliver
  set resp.http.X-Served-By = "edge";
  return (deliver);
}
```

## XVCL Features

### Constants
```xvcl
#const NAME = value
#const NAME TYPE = value
#const TTL INTEGER = 3600
#const ORIGIN = "origin.example.com"
#const ENABLED BOOL = true
#const DOUBLE_TTL = TTL * 2
```

Type is inferred from the value when omitted. Use in templates: `"{{TTL}}"` or `{{ORIGIN}}`

### For Loops
```xvcl
#const REGIONS = ["us_east", "us_west", "eu_west"]
#for region in REGIONS
backend F_{{region}} {
  .host = "{{region}}.example.com";
}
#endfor
```

### Tuple Unpacking in Loops
```xvcl
#const BACKENDS = [("api", "api.example.com"), ("static", "static.example.com")]
#for name, host in BACKENDS
backend F_{{name}} {
  .host = "{{host}}";
  .port = "443";
  .ssl = true;
}
#endfor
```

### Conditionals
```xvcl
#const PRODUCTION = true
#if PRODUCTION
set req.http.X-Env = "prod";
#else
set req.http.X-Env = "dev";
#endif
```

### Functions with Return Values
```xvcl
#def normalize_path(path STRING) -> STRING
  declare local var.result STRING;
  set var.result = std.tolower(path);
  set var.result = regsuball(var.result, "//+", "/");
  return var.result;
#enddef

sub vcl_recv {
  #FASTLY recv
  declare local var.clean STRING;
  set var.clean = normalize_path(req.url.path);
}
```

### Inline Macros (Zero Overhead)
```xvcl
#inline cache_key(url, host)
digest.hash_md5(url + "|" + host)
#endinline

// Use in vcl_hash:
set req.hash += cache_key(req.url, req.http.Host);
```

### Conditionals with Ternary
```xvcl
#const ENV = "prod"
set req.http.X-Debug = "{{ENV if ENV != 'prod' else 'disabled'}}";
```

### Includes
```xvcl
#include "includes/backends.xvcl"
#include <stdlib/security.xvcl>
```

## VCL Basics

### Request Lifecycle

1. `vcl_recv` - Request received, choose backend, decide cache behavior
2. `vcl_hash` - Generate cache key
3. `vcl_hit` - Cache hit handling
4. `vcl_miss` - Cache miss, fetch from origin
5. `vcl_pass` - Bypass cache
6. `vcl_fetch` - Process origin response
7. `vcl_deliver` - Send response to client
8. `vcl_error` - Error handling
9. `vcl_log` - Logging

### Return Actions

| Subroutine  | Common Returns             |
| ----------- | -------------------------- |
| vcl_recv    | `lookup`, `pass`, `error`  |
| vcl_fetch   | `deliver`, `pass`, `error` |
| vcl_deliver | `deliver`                  |
| vcl_error   | `deliver`                  |

### Variable Types

```vcl
declare local var.str STRING;
declare local var.num INTEGER;
declare local var.flag BOOL;
declare local var.time TIME;
declare local var.duration RTIME;
declare local var.decimal FLOAT;
```

### Key Variables

Request: `req.method`, `req.url`, `req.url.path`, `req.url.qs`, `req.http.*`
Response: `resp.status`, `resp.http.*`
Backend Response: `beresp.status`, `beresp.ttl`, `beresp.http.*`
Client: `client.ip`, `client.geo.country_code`

## References

Read the relevant reference file completely before implementing specific VCL features.

| Topic       | File                                        | Use when...                                      |
| ----------- | ------------------------------------------- | ------------------------------------------------ |
| VCL Syntax  | [vcl-syntax.md](references/vcl-syntax.md)   | Working with data types, operators, control flow |
| Subroutines | [subroutines.md](references/subroutines.md) | Understanding request lifecycle, custom subs     |
| Headers     | [headers.md](references/headers.md)         | Manipulating HTTP headers                        |
| Backends    | [backends.md](references/backends.md)       | Configuring origins, directors, health checks    |
| Caching     | [caching.md](references/caching.md)         | Setting TTL, grace periods, cache keys           |
| Strings     | [strings.md](references/strings.md)         | String manipulation functions                    |
| Crypto      | [crypto.md](references/crypto.md)           | Hashing, HMAC, base64 encoding                   |
| Tables/ACLs | [tables-acls.md](references/tables-acls.md) | Lookup tables, access control lists              |
| Testing VCL | [testing-vcl.md](references/testing-vcl.md) | Writing unit tests, assertions, test helpers     |

## Common Patterns

### Route by Path
```vcl
sub vcl_recv {
  #FASTLY recv
  if (req.url.path ~ "^/api/") {
    set req.backend = api_backend;
  } else if (req.url.path ~ "^/static/") {
    set req.backend = static_backend;
  } else {
    set req.backend = default_backend;
  }
  return (lookup);
}
```

### Add Security Headers
```vcl
sub vcl_deliver {
  #FASTLY deliver
  set resp.http.X-Content-Type-Options = "nosniff";
  set resp.http.X-Frame-Options = "DENY";
  set resp.http.Strict-Transport-Security = "max-age=31536000";
  return (deliver);
}
```

### Normalize Cache Key
```vcl
sub vcl_recv {
  #FASTLY recv
  set req.url = querystring.sort(req.url);
  set req.url = querystring.filter(req.url, "utm_");
}

sub vcl_hash {
  #FASTLY hash
  set req.hash += req.http.Host;
  set req.hash += req.url.path;
}
```

### Redirect HTTP to HTTPS
```vcl
sub vcl_recv {
  #FASTLY recv
  if (!req.is_ssl) {
    error 801 "Redirect to HTTPS";
  }
}

sub vcl_error {
  #FASTLY error
  if (obj.status == 801) {
    set obj.status = 301;
    set obj.http.Location = "https://" + req.http.Host + req.url;
    return (deliver);
  }
}
```

## XVCL Compilation

```bash
# Basic compilation (output defaults to input with .vcl extension)
uvx xvcl input.xvcl -o output.vcl

# With include paths
uvx xvcl main.xvcl -o main.vcl -I ./includes -I ./shared

# Debug mode (shows expansion)
uvx xvcl main.xvcl -o main.vcl --debug
uvx xvcl main.xvcl -o main.vcl -v  # alias for --debug

# Source maps (track code origin)
uvx xvcl main.xvcl -o main.vcl --source-maps
```

## Testing with Falco

```bash
# Lint VCL
falco lint main.vcl

# Start local simulator
falco simulate main.vcl

# Run unit tests
falco test main.vcl

# Format VCL
falco fmt main.vcl
```
