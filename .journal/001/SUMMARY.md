---
id: 001
title: Rebrand template-mcp to poe2-mcp + release/repo configuration
date: 2026-06-14
status: complete
repos_touched: [poe]
related_sessions: []
---

## Goal
Strip the Meigma `template-mcp` starter branding from a freshly generated repo so
it identifies as its real product: a **Path of Exile 2 MCP server** (`poe2-mcp`) —
an AI gateway into the PoE marketplace. Then wire up release automation
credentials and apply the repository settings manifest.

## Outcome
Met. PR #1 (`chore: rebrand template-mcp to poe2-mcp`) merged to `master` as squash
commit `dffad47`. Release-app credentials are configured, and the GitHub repo is
fully configured from `.github/repository-settings.toml` (general settings,
security toggles, Pages, branch + tag protection rulesets). Final settings `plan`
reports "No supported changes are required."

## Key Decisions
- **Branding-only scope** — kept BOTH transports (stdio + http) and the
  `random_int` demo tool. Choosing a transport and building real PoE tools are
  deferred (functional decisions, not branding). User-confirmed.
- **NAME/BINARY = `poe2-mcp`** (env prefix `POE2_MCP_*`), **TITLE = "Path of Exile
  2 MCP Server"**, module `github.com/jmgilman/poe`. User-chosen.
- **Ordered string replacement** (most-qualified first) because `template-mcp`
  maps to two axes: REPO (`poe`) in module/URL/image contexts vs BINARY
  (`poe2-mcp`) elsewhere. A single global sub would have been wrong.
- **`is_template = false`** — poe is a real product, not a template repo. The
  rebrand's token search missed it (no `template-mcp`/`meigma` string).
- **Release app = `jmgilman-release-please`** (App ID `4055060`), NOT
  `meigma-release-please` (3342783). First attempt used the wrong app; corrected.
- **Tag-ruleset bypass via `type = "integration"` + App ID**, not `type = "app"`
  + slug: the app is private, so the configure script's `GET /apps/{slug}`
  resolver 404s. Integration actors reference the App ID directly.

## Changes
- Rebranded ~56 files: Go modules → `github.com/jmgilman/poe`, `cmd/poe2-mcp`,
  `internal/templateinfo` identity, CLI help/env strings (`POE2_MCP_*`), CI /
  release / packaging metadata (`ghcr.io/jmgilman/poe`, GoReleaser, release-please,
  Moon ×3, ghd.toml), docs site + content, `LICENSE-MIT` © Joshua Gilman, reset
  `CHANGELOG.md`, removed `DELETE_ME.md`.
- `MEIGMA_RELEASE_APP_*` → `RELEASE_APP_*` in `release-please.yml`.
- `.github/repository-settings.toml`: `is_template=false`, tag bypass →
  `{type="integration", actor_id=4055060}`.
- GitHub repo (not in git): pushed Actions var `RELEASE_APP_ID=4055060` + secret
  `RELEASE_APP_PRIVATE_KEY`; applied repo settings + rulesets (branch id 17669340,
  tag id 17669490).

## Open Threads
- No real PoE functionality yet — `random_int` is still the only tool. Next phase:
  choose a transport, build PoE API auth + marketplace/pricing tools, integrate
  Path of Building / build sites.
- `master` now requires PR + green CI + signed squash-merge; the two release
  dry-run checks are path-gated (skip on non-release changes) and count as
  satisfied when skipped.
- First real release still needs the package versioning reset decision
  (`.release-please-manifest.json` left at `0.1.3` from the template).

## References
- PR #1: https://github.com/jmgilman/poe/pull/1 (merge `dffad47`)
- Release app creds: 1Password `Homelab` vault, item `jmgilman-release-please`
  (`app_id`=4055060, `key.pem`).
- Rulesets: branch `17669340`, tag `17669490`.

## Lessons
- For **private** GitHub Apps, ruleset bypass actors must use
  `type="integration"` + App ID, not `type="app"` + slug (the public
  `/apps/{slug}` endpoint 404s for private apps). The app must also be **installed**
  on the repo or the ruleset POST 422s ("must be part of ruleset source or owner
  organization") — pushing its secrets ≠ installing it.
- The configure script applies Pages before rulesets and aborts on first error;
  GitHub Pages HTTPS enforcement transiently 404s ("certificate does not exist
  yet") right after site creation, but `*.github.io` reports `https_enforced=true`
  by default, so it self-resolves.
- `rg` skips dotfiles without `--hidden` (missed `.github/**`, `.goreleaser.yaml`).
  The Bash tool runs zsh (no `mapfile`).
