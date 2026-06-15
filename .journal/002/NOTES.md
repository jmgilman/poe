---
id: 002
title: PoE 2 official API reference doc
started: 2026-06-14
---

## 2026-06-14 17:36 — Kickoff
Goal for the session: not yet stated — developer ran `session-new` to open a
session; awaiting the actual request. Will refine this title and the INDEX row
once the goal is known.

Current state of the world (carried from session 001, complete):
- `poe2-mcp` is a rebranded Path of Exile 2 MCP server (module
  `github.com/jmgilman/poe`, env prefix `POE2_MCP_*`), built on
  `modelcontextprotocol/go-sdk`, orchestrated by Moon. Both stdio + http
  transports and the `random_int` demo tool exist as scaffolding.
- No real PoE functionality yet — `random_int` is still the only tool. The
  natural next phase is choosing a transport and building real PoE tooling
  (API auth, marketplace/pricing, build sites).
- Release automation + GitHub repo settings/rulesets are fully configured.
  `master` requires PR + green CI + signed squash merge. First real release
  still needs the package versioning reset decision
  (`.release-please-manifest.json` left at `0.1.3`).

Plan: wait for the developer's request, then update this NOTES log and the
INDEX title to match the real goal.

## 2026-06-14 17:37 — Goal set: PoE 2 official API reference doc
Developer's request: create a reference doc in this session journal folder
describing the **official Path of Exile 2 API**. Required coverage:
1. Where it's documented (ideally a full reference spec).
2. What it does and does not cover.
3. How OIDC works — specifically how a user authenticates to the API.
4. Other interesting notes relevant to consuming the API.

Approach: launched the `deep-research` workflow (run `wf_5da423a7-e87`, task
`wxib7au82`) with a developer-focused, MCP-consumer-oriented prompt covering
docs/spec, coverage + gaps, OAuth/OIDC flow, and consumption notes (rate limits,
User-Agent, ToS, developer onboarding). Prompt explicitly asks to flag where
info is PoE 1-specific vs confirmed for PoE 2. Awaiting completion, then will
synthesize the findings into a reference doc under `.journal/002/`.

## 2026-06-14 17:51 — Research done; reference doc written
Workflow completed (98 agents, ~2.08M subagent tokens, ~10 min): 16 sources →
68 claims → 25 adversarially verified (3-vote) → 22 confirmed / 3 refuted.
Confidence high on the core facts.

Wrote the deliverable: **`.journal/002/poe2-api-reference.md`** — a
developer reference for the official GGG web API as it applies to PoE2.

Key takeaways captured in the doc:
- **One shared API** for PoE1 + PoE2 (no separate PoE2 API); game selected via a
  `realm=poe2` parameter. Canonical docs at
  `pathofexile.com/developer/docs`, REST base `api.pathofexile.com`. **No
  OpenAPI/Swagger, no GraphQL.**
- **Auth = OAuth 2.1, explicitly NOT OIDC.** Authorize/token at
  `www.pathofexile.com/oauth/{authorize,token}`. Authorization Code + PKCE for
  `account:*` (user) scopes; Client Credentials for `service:*` (confidential
  clients only). Confidential tokens 28d/90d; public 10h/7d, PKCE S256,
  30s auth-code window, local redirect only.
- **PoE2 returns LIMITED data today.** Cross-version: profile, characters,
  item-filter, leagues, currency-exchange. **PoE1-only:** public stashes,
  account/guild stashes, PvP, account-leagues, league-accounts → **no official
  PoE2 trade/public-stash feed exists.** This is the biggest constraint for the
  product direction.
- **Access is gated:** email `oauth@grindinggear.com`; **low-effort/LLM-generated
  requests auto-rejected.** ⚠️ Direct tension for an MCP/AI product — the
  application must be human-written with a concrete, well-scoped vision.
- Required `User-Agent: OAuth {clientId}/{version} (contact: {contact})`. Rate
  limits are dynamic via `X-Rate-Limit-*` + `Retry-After` headers — **do NOT
  hard-code numbers** (three circulating numeric limits were refuted 0-3).

Open threads for the product (also listed in the doc §7): what the OAuth
application email must contain + approval bar; exactly which fields/scopes return
PoE2 data; whether any official PoE2 trade/market path exists or is forthcoming;
concrete rate-limit values for our hot endpoints.

