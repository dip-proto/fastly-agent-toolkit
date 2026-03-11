# Fastly Agent Skills

A collection of skills for AI coding agents to work with the Fastly platform and edge computing tools.

## Structure

```
skills/
├── fastly/       # Fastly platform (services, caching, VCL, WAF, TLS, DDoS, purging, APIs)
├── fastly-cli/   # Fastly CLI (services, compute, logging, WAF, TLS, stores, stats)
├── falco/        # VCL development (lint, test, simulate, format, REPL, Terraform)
├── fastlike/     # Local Compute — Go-based (backends, config, build, test)
├── viceroy/      # Local Compute — Viceroy/WASM (serve, config, test, adapt)
└── xvcl/         # XVCL VCL transpiler (syntax, subroutines, headers, caching)
```

Each skill has a `SKILL.md` entrypoint and a `references/` directory with detailed topic files.

## Usage

Copy the skills you need into your agent's skills directory. You probably don't need all of them. Pick what's relevant to your project.

### Claude Code

#### Plugin Marketplace

```bash
claude plugin marketplace add git@github.com:fastly/fastly-agent-toolkit.git
claude plugin list

# If this fails, add skills manually in the next section.
```

#### Manual

```bash
mkdir -p .claude/skills
cp -R ./skills/{falco,viceroy} .claude/skills/
```

For immediate, reliable setup in local environments, prefer the manual copy above first (it does not depend on the marketplace installation step).

### Codex

```bash
mkdir -p ~/.codex/skills
cp -R ./skills/{falco,viceroy} ~/.codex/skills/
```

### Qwen Code

Qwen Code requires the experimental skills feature. Enable it by adding to `.qwen/settings.json`:

```json
{
  "tools": {
    "experimental": {
      "skills": true
    }
  }
}
```

Then copy skills to the project directory:

```bash
mkdir -p .qwen/skills
cp -R ./skills/{falco,viceroy} .qwen/skills/
```

### Gemini CLI

```bash
gemini extensions link .
```

Swap `{falco,viceroy}` for whatever combination you need. For VCL work, `falco` and `xvcl` are the most useful. For Fastly Compute, grab `fastly-cli` and either `viceroy` or `fastlike`.

## Skill format

Each skill lives in its own directory as a `SKILL.md` file with YAML frontmatter following the [Agent Skills spec](https://agentskills.io/specification).
