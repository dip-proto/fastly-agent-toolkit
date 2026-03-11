---
name: falco
description: "Falco CLI for VCL development — linting, testing, simulating, formatting, and Terraform integration. Use when you need to lint, test, simulate, or format VCL code, use the VCL REPL, get VCL statistics, or validate VCL from Terraform plans."
---

# Falco CLI Overview

Falco is a Fastly VCL development tool for linting, testing, simulating, and formatting VCL code.

**Official VCL documentation**: https://www.fastly.com/documentation/guides/full-site-delivery/fastly-vcl/about-fastly-vcl/

**Falco documentation**: https://github.com/ysugimoto/falco

## Install

```bash
# Homebrew
brew install falco

# From source (requires Go 1.25+)
go install github.com/ysugimoto/falco/cmd/falco@latest

# Or clone and build
git clone https://github.com/ysugimoto/falco.git
cd falco
make darwin_arm64    # or darwin_amd64, linux_amd64, linux_arm64
```

## Commands

| Command           | Description                      |
| ----------------- | -------------------------------- |
| `falco [lint]`    | Lint VCL files (default command) |
| `falco test`      | Run VCL unit tests               |
| `falco simulate`  | Start local simulator server     |
| `falco fmt`       | Format VCL files                 |
| `falco stats`     | Show VCL code statistics         |
| `falco console`   | Interactive VCL REPL             |
| `falco terraform` | Lint VCL from Terraform plans    |
| `falco dap`       | Debug Adapter Protocol server    |

## Common flags (all commands)

| Flag                 | Description                      |
| -------------------- | -------------------------------- |
| `-I, --include_path` | Add include path for VCL imports |
| `-h, --help`         | Show help                        |
| `-V, --version`      | Show version                     |
| `-r, --remote`       | Fetch snippets from Fastly API   |
| `--refresh`          | Refresh remote snippet cache     |

## Quick reference

**Lint before deployment:**
```bash
falco -vv -I ./vcl ./vcl/main.vcl
```

**Run tests:**
```bash
falco test -I ./vcl ./vcl/main.vcl
```

**Development with watch mode:**
```bash
falco test -w -I ./vcl ./vcl/main.vcl
```

**Local testing server:**
```bash
falco simulate -p 8080 -I ./vcl ./vcl/main.vcl
```

**Format all VCL:**
```bash
falco fmt -w ./vcl/**/*.vcl
```

**Terraform integration:**
```bash
terraform show -json planned.out | falco terraform -vv
```

## Configuration

Create `.falco.yaml` in project root for persistent settings:

```yaml
include_paths:
  - ./vcl
  - ./includes

linter:
  verbose: "warning"
  rules:
    rule-name: ERROR  # or WARNING, INFO, IGNORE

testing:
  timeout: 10  # minutes (default: 10)
  filter: "*.test.vcl"

simulator:
  port: 3124

format:
  indent_width: 2
  line_width: 120
```

## Environment variables

| Variable            | Description               |
| ------------------- | ------------------------- |
| `FASTLY_SERVICE_ID` | Service ID for Fastly API |
| `FASTLY_API_KEY`    | API key for Fastly API    |

Required when using `-r, --remote` flag.

## References

| Topic          | File                                              | Use when...                                       |
| -------------- | ------------------------------------------------- | ------------------------------------------------- |
| Testing VCL    | [testing-vcl.md](references/testing-vcl.md)       | Running test suites, coverage, watch mode for TDD |
| Formatting VCL | [formatting-vcl.md](references/formatting-vcl.md) | Formatting VCL for consistent style               |
| Linting VCL    | [linting-vcl.md](references/linting-vcl.md)       | Checking VCL for errors before deployment         |
| Simulating VCL | [simulating-vcl.md](references/simulating-vcl.md) | Testing VCL against HTTP requests locally         |
| Terraform VCL  | [terraform-vcl.md](references/terraform-vcl.md)   | Validating VCL from Terraform plans               |
| VCL Console    | [vcl-console.md](references/vcl-console.md)       | Experimenting with VCL expressions interactively  |
| VCL Statistics | [vcl-statistics.md](references/vcl-statistics.md) | Analyzing VCL project size and complexity         |

## Source Code as Reference

The falco source code at `github.com/ysugimoto/falco` is a complete implementation of Fastly's VCL dialect and is useful for understanding how VCL works:

| Concept               | Source Path                        | What You Learn                                                                        |
| --------------------- | ---------------------------------- | ------------------------------------------------------------------------------------- |
| VCL syntax and tokens | `token/token.go`, `lexer/lexer.go` | Every VCL keyword, operator, and literal type                                         |
| VCL grammar and AST   | `parser/`, `ast/`                  | How VCL code is structured — declarations, statements, expressions                    |
| Request lifecycle     | `interpreter/context/`             | The 9 subroutine scopes (recv, hash, hit, miss, pass, fetch, error, deliver, log)     |
| Predefined variables  | `interpreter/variable/`            | Every `req.*`, `beresp.*`, `client.*`, etc. variable and which scopes can access them |
| Built-in functions    | `interpreter/function/`            | All 390+ built-in functions with type signatures and scope restrictions               |
| Type system           | `interpreter/value/`               | VCL types (STRING, INTEGER, FLOAT, BOOL, TIME, RTIME, IP, BACKEND) and coercion rules |
| Linter rules          | `linter/`                          | Common VCL mistakes and best practices                                                |
| VCL execution         | `interpreter/`                     | How VCL statements execute, variable scoping, backend selection                       |
