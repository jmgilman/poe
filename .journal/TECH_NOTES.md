# Technical Notes

- Use hexagonal architecture at all times. Keep business logic isolated from CLI, filesystem, network, storage, and other external adapters.
- Prefer functional testing before calling any feature complete. Unit tests are useful, but they do not prove the tool works the way the design intends.
- Take an agile approach to development. Avoid waterfall: underspecify when useful, prototype early, learn from the result, and refine from working behavior.

## Release & repo infrastructure

- The project is `poe2-mcp` (module `github.com/jmgilman/poe`, env prefix `POE2_MCP_*`), built on `modelcontextprotocol/go-sdk`, orchestrated by Moon. Both stdio + http transports and the `random_int` demo tool exist as scaffolding pending real PoE tooling.
- Release automation uses the **`jmgilman-release-please`** GitHub App (App ID `4055060`). Its credentials live in 1Password, `Homelab` vault, item `jmgilman-release-please` (`app_id`, `key.pem`). They are mirrored to repo `jmgilman/poe` as Actions var `RELEASE_APP_ID` + secret `RELEASE_APP_PRIVATE_KEY` (consumed by `.github/workflows/release-please.yml`). Pushing secrets does NOT install the app — it must be installed on the repo separately.
- Repo settings are managed declaratively in `.github/repository-settings.toml`, applied via `uv run .github/scripts/configure_github_repo.py {plan|apply} --repo jmgilman/poe`. The script is idempotent; always `plan` before `apply`. Private-app ruleset bypass actors must use `type = "integration"` + App ID (not `type = "app"` + slug — the public `/apps/{slug}` endpoint 404s for private apps).
- `master` is protected (rulesets `Default branch`/`Default tags`): PRs required, signed commits, linear history, squash-only merges. Integrate via `gh pr` + squash merge.
