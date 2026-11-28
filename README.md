# Betterleaks

Fast, configurable secrets scanning for repositories, filesystems, and cloud-hosted sources.

> **Upstream / attribution:** This repository is a copy of [betterleaks/betterleaks](https://github.com/betterleaks/betterleaks), the project created and maintained by Zachary Rice and contributors (also known for Gitleaks). Use the upstream repository for releases and issue reports. Licensed under MIT © 2026 Zachary Rice — see [LICENSE](LICENSE).

Betterleaks finds credentials and other sensitive strings in Git history, local files, stdin, and a growing set of hosted sources. Its rule engine uses [Expr](https://expr-lang.org) expressions for cheap prefilters and post-match filtering, so teams can keep false positives low without giving up coverage. The same engine can validate a candidate secret asynchronously with an HTTP request defined in the rule.

## Features

- **Expr-based rules** — `prefilter` expressions run before regex matching; `filter` expressions run on candidates with access to resource attributes and finding data. Existing CEL-shaped configs are still accepted, but new configs should use Expr.
- **Live validation** — a rule can call an HTTP endpoint and use the response to decide whether a candidate is a real, active secret.
- **Low-noise matching** — Aho–Corasick keyword filtering, optional BPE "token rarity" filtering, and entropy helpers reduce natural-language false positives.
- **Many sources** — Git repositories and history, local directories, stdin, GitHub, GitLab, Hugging Face, and S3/S3-compatible buckets.
- **Portable** — a single Go binary, with a container image also available.
- **Extensible config** — extend the bundled default rule set or maintain a fully independent config.

## Install

```sh
# Package managers (upstream packages)
brew install betterleaks
sudo dnf install betterleaks

# Container
docker pull ghcr.io/betterleaks/betterleaks:latest

# Go
go install github.com/betterleaks/betterleaks@latest

# From this checkout (Go 1.25+)
git clone https://github.com/anya12forger12-max/betterleaks.git
cd betterleaks
make build
```

## Quick start

```sh
# Git history, with verbose findings
betterleaks git /path/to/repo -v

# Current filesystem state
betterleaks dir . -v

# Piped input
cat some_file.txt | betterleaks stdin

# GitHub org or a single pull request
betterleaks github https://github.com/betterleaks
betterleaks github https://github.com/betterleaks/betterleaks/pull/113

# GitLab project and merge request
betterleaks gitlab https://gitlab.com/mygroup/myproject
betterleaks gitlab https://gitlab.com/mygroup/myproject/-/merge_requests/42

# Hugging Face and S3-compatible sources
betterleaks hf https://huggingface.co/datasets/myorg/mydataset
betterleaks s3 'https://<account-id>.r2.cloudflarestorage.com/*'
```

See [docs/scanning.md](docs/scanning.md) for more target-specific patterns and [docs/config.md](docs/config.md) for the configuration reference. The generated default rules live in [`config/betterleaks.toml`](config/betterleaks.toml); for production, keep your own config rather than extending the upstream default so rule changes stay reviewable.

## Project layout

| Path | What it contains |
| :--- | :--- |
| `cmd/` | CLI commands (`git`, `dir`, `stdin`, `github`, `gitlab`, `huggingface`/`hf`, `s3`, `protect`, …) |
| `sources/` | Source adapters that turn targets into scannable fragments |
| `detect/`, `regexp/` | Matching engine and rule regex helpers |
| `internal/exprruntime/` | Expr bindings used by filters and validation |
| `internal/ahocorasick/`, `internal/words/` | Keyword and token-rarity prefilters |
| `config/`, `.betterleaksignore` | Default rules and repository-local ignores |
| `docs/` | Scanning and configuration guides |

## Responsible use

Secrets scanning can expose credentials that belong to other people. Scan only systems you own or are authorized to assess, handle findings as sensitive data, and rotate any exposed secret before sharing reports. Validation rules make outbound requests — keep them narrowly scoped and never send candidate credentials to untrusted endpoints.

## License and credits

MIT License — Copyright (c) 2026 Zachary Rice. See [LICENSE](LICENSE). Upstream development is supported by [Aikido Security](https://www.aikido.dev). All credit for the design and implementation belongs to the authors and contributors of [betterleaks/betterleaks](https://github.com/betterleaks/betterleaks); this repository is a copy, not the canonical upstream.

## Quick start — 1 or 2 commands

```bash
./setup.sh          # install dependencies
./setup.sh run      # start the project (when a run target exists)
```

`setup.sh` detects this repository's build system and installs the right toolchain
dependencies. Use `./setup.sh test` to run the available checks. If a project has no
standard run target, the script prints the correct command from the sections above.
