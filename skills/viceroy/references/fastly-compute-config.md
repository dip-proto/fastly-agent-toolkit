# Fastly Compute Configuration

Configure `fastly.toml` for local development with Viceroy.

## Basic Structure

```toml
name = "my-compute-app"
description = "My Fastly Compute application"
authors = ["Your Name <you@example.com>"]
language = "rust"

[local_server]
  # Configuration sections go here
```

## Backends

Define origin servers that your Compute service communicates with.

```toml
[local_server.backends]
  [local_server.backends.origin]
  url = "http://localhost:8000"

  [local_server.backends.api]
  url = "https://api.example.com"
  override_host = "api.example.com"
```

### Backend Options

| Field            | Required | Description                            |
| ---------------- | -------- | -------------------------------------- |
| `url`            | Yes      | Backend URL (http or https)            |
| `override_host`  | No       | Override the Host header               |
| `ca_certificate` | No       | Custom CA certificate (inline or file) |

### With TLS/mTLS

```toml
[local_server.backends.secure]
url = "https://secure.example.com"

# Inline certificate
ca_certificate = '''
-----BEGIN CERTIFICATE-----
MIIDqTCCApGgAwIBAgIU...
-----END CERTIFICATE-----
'''

# Or from file
# ca_certificate.file = "certs/ca.pem"
```

### Dynamic Backends

Dynamic backends created at runtime (using the Rust or JavaScript SDK) require a health check probe to be configured. Without a probe, dynamic backends will not function correctly. Configure probes via the SDK's `BackendBuilder` when creating dynamic backends programmatically.

## Dictionaries / Config Stores

Key-value configuration data. Both `dictionaries` and `config_stores` work identically.

### Inline TOML Format

```toml
[local_server.dictionaries.settings]
format = "inline-toml"
contents = { api_key = "secret123", timeout = "30" }
```

### JSON File Format

```toml
[local_server.dictionaries.settings]
file = "config/settings.json"
format = "json"
```

Where `config/settings.json`:
```json
{
  "api_key": "secret123",
  "timeout": "30"
}
```

## KV Stores / Object Stores

Persistent key-value storage. Aliases: `object_stores`, `object_store`, `kv_stores`.

```toml
[local_server.object_stores.my_store]
key_file = "data/store.json"

[local_server.object_stores.cache]
key_file = "data/cache.json"
```

The key file is a JSON object mapping keys to values:
```json
{
  "user:123": "John Doe",
  "config": "{\"theme\": \"dark\"}"
}
```

## Secret Stores

Sensitive configuration values.

```toml
[local_server.secret_stores.secrets]
file = "secrets/local.json"
format = "json"
```

Where `secrets/local.json`:
```json
{
  "api_secret": "supersecret",
  "db_password": "localdev"
}
```

## Geolocation

Mock geolocation data for IP addresses.

### Inline Format

```toml
[local_server.geolocation]
format = "inline-toml"

[local_server.geolocation.addresses."127.0.0.1"]
as_name = "Local ISP"
city = "San Francisco"
country_code = "US"
latitude = 37.7749
longitude = -122.4194
```

### JSON File Format

```toml
[local_server.geolocation]
file = "geo/locations.json"
format = "json"
```

### Available Geolocation Fields

- `as_name`, `as_number` - Autonomous system info
- `city`, `country_code`, `country_name`
- `continent_code`, `region`
- `latitude`, `longitude`
- `postal_code`, `metro_code`
- `utc_offset`, `gmt_offset`
- `conn_speed`, `conn_type`

## Device Detection

Mock device detection for User-Agent strings.

```toml
[local_server.device_detection]
format = "inline-toml"

[local_server.device_detection.user_agents."Mozilla/5.0 (iPhone)"]
device_type = "smartphone"
brand = "Apple"
model = "iPhone"
is_mobile = true
```

## Access Control Lists (ACLs)

IP-based access control.

```toml
[local_server.acls.allowed_ips]
file = "acls/allowed.json"
format = "json"
```

Where `acls/allowed.json`:
```json
{
  "entries": [
    {"ip": "192.168.1.0", "prefix": 24, "action": "allow"},
    {"ip": "10.0.0.0", "prefix": 8, "action": "block"}
  ]
}
```

## Complete Example

```toml
name = "my-edge-app"
description = "Edge computing application"
authors = ["Developer <dev@example.com>"]
language = "rust"

[local_server]
  [local_server.backends]
    [local_server.backends.origin]
    url = "http://localhost:3000"

    [local_server.backends.api]
    url = "https://api.example.com"
    override_host = "api.example.com"

  [local_server.dictionaries.config]
  format = "inline-toml"
  contents = { feature_flag = "true", version = "1.0" }

  [local_server.object_stores.cache]
  key_file = "data/cache.json"

  [local_server.secret_stores.secrets]
  file = "secrets/local.json"
  format = "json"

  [local_server.geolocation]
  format = "inline-toml"
  [local_server.geolocation.addresses."127.0.0.1"]
  country_code = "US"
  city = "San Francisco"
```

## Validation

Run Viceroy to validate your configuration:

```bash
viceroy -C fastly.toml bin/main.wasm
```

Errors will be reported with specific details about invalid fields or missing files.
