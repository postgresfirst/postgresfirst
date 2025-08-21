# Contributing to PGF Postgres

Thanks for your interest! This repo packages **PGF Postgres** (meta-installer and per-version kits).

## Quick start (dev)
- Use a Linux host (or container) with `bash`, `make`, and `docker` (optional).
- Lint: `./scripts/dev/lint.sh`
- Build Debian: `cd pgf-postgres && debuild -us -uc`
- Build RPM: see `rpm/*.spec` in each kit.

## Branch & commit style
- Use feature branches: `feat/…`, `fix/…`, `docs/…`.
- Prefer **Conventional Commits** style: `feat:`, `fix:`, `docs:`, `chore:` …

## Pull requests
- Include a clear description of *what* and *why*.
- Add/adjust docs (README/INSTALL) if behavior changes.
- CI must pass (lint + build matrix).

## Code style
- Shell: POSIX/Bash; keep `set -euo pipefail` in scripts.
- Run `shellcheck` and `shfmt -w` before pushing.
- Keep the installer idempotent and safe to re-run.

## Testing
- Test on at least one DEB (Ubuntu/Debian) and one RPM (Rocky/Alma) distro, ideally via containers.
- Verify:
  - fresh install
  - re-run install (idempotence)
  - uninstall + purge
  - custom `--port`, `--allow`, `--use-distro-repo`

## Security
- Don’t commit secrets. Use GitHub Actions **Secrets** for signing keys or repo tokens.
- Report vulnerabilities via `SECURITY.md`.
