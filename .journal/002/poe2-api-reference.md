# Path of Exile 2 — Official GGG Web API Reference

> Developer reference for the official Grinding Gear Games (GGG) web API as it
> applies to **Path of Exile 2**, written for building the `poe2-mcp` server.
>
> Compiled 2026-06-14 from a multi-source, adversarially verified research pass.
> Primary sources are GGG's own docs (`pathofexile.com/developer/docs`) and
> forum staff posts. Where a fact is PoE1-specific or unconfirmed for PoE2, it is
> flagged inline. **Always re-check the live docs before relying on specifics —
> the PoE2 surface is actively evolving.**

## TL;DR

- **There is ONE official API** shared by PoE1 and PoE2. PoE2 does **not** have a
  separate API. The game is selected per-request via a `realm` parameter
  (`poe2`).
- **Canonical docs:** <https://www.pathofexile.com/developer/docs> · REST base
  endpoint **`https://api.pathofexile.com`**.
- **No OpenAPI/Swagger, no GraphQL.** It is a plain REST/JSON API. The only
  OpenAPI file in the wild is community-made and outdated.
- **Auth is OAuth 2.1** (explicitly **not** OpenID Connect). Two grant types:
  Authorization Code + PKCE (user `account:*` scopes) and Client Credentials
  (service `service:*` scopes, confidential clients only).
- **Access is gated by manual approval.** You must email
  `oauth@grindinggear.com`. **Low-effort or LLM-generated requests are rejected
  on sight** — a real concern for an MCP-server pitch (see
  [Onboarding](#registration--onboarding)).
- **PoE2 returns limited game data today.** Several useful endpoints
  (public stashes/trade stream, account/guild stashes, PvP, account leagues)
  are **PoE1-only**. There is **no official PoE2 public-stash "river" / bulk
  item-listing feed** — the one thing every trade tool wants. The *only*
  official PoE2 economy feed is the **Currency Exchange API** (`service:cxapi`,
  aggregate currency data); live item prices come only from the **undocumented
  `trade2` site API**, which is against GGG's ToS §7i. See
  [§4 — where PoE2 market data comes from](#4-where-poe2-market-data-actually-comes-from).

---

## 1. Where it is documented

| Resource | URL |
|----------|-----|
| Docs home / Getting Started | <https://www.pathofexile.com/developer/docs> (anchor `…/docs/index#gettingstarted`) |
| Authorization (OAuth) | <https://www.pathofexile.com/developer/docs/authorization> |
| Endpoint reference | <https://www.pathofexile.com/developer/docs/reference> |
| Changelog | <https://www.pathofexile.com/developer/docs/changelog> |
| Staff forum confirmation | <https://www.pathofexile.com/forum/view-thread/3821465> (Tai_GGG, 2025-08-01) |
| Manage your apps/tokens | `https://www.pathofexile.com/my-account/applications` |
| Registration contact | `oauth@grindinggear.com` (**not** `support@…`) |

**Spec format:** REST/JSON over HTTPS. **No machine-readable OpenAPI/Swagger
spec is published by GGG**, and there is **no GraphQL** surface. The reference
page is the closest thing to a full spec (server endpoint stated verbatim:
`https://api.pathofexile.com`).

> ⚠️ The only OpenAPI artifact found in the wild
> (`Chuanhsing/poe-api/poe.yaml`) is **community-maintained, unofficial, and
> outdated** — its `realm` enum doesn't even include `poe2`. Do not treat it as
> authoritative.

---

## 2. PoE2 vs PoE1: one API, a `realm` switch

PoE1 and PoE2 share a single unified developer API. Endpoints select the game
via a **`realm`** parameter:

- Omitted → **PoE1 PC** (default).
- Values: `xbox`, `sony`, **`poe2`**.

Section headers in the reference are tagged with version markers. The changelog
confirms the `poe2` realm option was added to the **League** and **Character**
endpoints, and that Character/Item type definitions gained PoE2-specific
properties.

> GGG states verbatim: *"There are currently limited APIs that return PoE2 game
> information."* Treat PoE2 coverage as a moving target.

---

## 3. Coverage — what the API does and does not cover

### Endpoint map (verified against the reference page)

| Endpoint | Path | Realm support |
|----------|------|---------------|
| Account Profile | `GET /profile` | **Cross-version** |
| Account Characters | `/character` | **Cross-version** (PoE2 properties added) |
| Account Item Filters | `/item-filter` | **Cross-version** |
| Leagues | `/league` | **Cross-version** (PoE2 added) |
| Currency Exchange | `/currency-exchange` | **Cross-version** |
| Type / Extra Definitions | (schema/type defs) | **Cross-version** |
| Public Stashes | `/public-stash-tabs` | **PoE1 only** |
| Account Stashes | `/stash` | **PoE1 only** |
| Guild Stashes | `/guild/stash` | **PoE1 only** |
| Account Leagues | `/account/leagues` | **PoE1 only** |
| League Accounts | `/league-account` | **PoE1 only** |
| PvP Matches | `/pvp-match` | **PoE1 only** |
| Build Planner | (File Format) | **PoE2 only** — a file format, *not* a live endpoint |

### What IS covered (and works for PoE2)

- Authenticated **account profile**, **characters + inventories**, **item
  filters**, **league listings**, and **currency exchange** — these are
  cross-version and respond to `realm=poe2` (with PoE2-specific fields).

### What is NOT covered / gaps to plan around

- **No official PoE2 public-stash "river" / bulk item-listing feed.** The Public
  Stash Tab stream (the basis of every third-party PoE1 trade tool) is
  **PoE1-only**. The one official PoE2 economy feed is the narrow **Currency
  Exchange API** (currency aggregates only — see
  [§4](#4-where-poe2-market-data-actually-comes-from)).
- **No account/guild stash, PvP, or account-league data for PoE2.**
- **No published OpenAPI/Swagger, no GraphQL.**
- The **official developer API (`api.pathofexile.com`) is distinct from the
  trade-site search API** (`pathofexile.com/trade`). The trade-site endpoints
  are **not part of the documented developer reference**, are effectively
  unofficial, and carry their own undocumented limits. Don't conflate them.

### Public Stash Tab API (PoE1-only — context for what PoE2 lacks)

For reference, since it's what an MCP server would *want* for trade data:

- `GET https://api.pathofexile.com/public-stash-tabs[/<realm>]`
  (legacy: `http://www.pathofexile.com/api/public-stash-tabs`).
- Scope **`service:psapi`**; **guest/anonymous access has been removed** — now
  requires OAuth.
- A **forward-moving stream of current state**, **no historical data**.
- Pagination via a **`next_change_id`** cursor returned in the JSON, passed back
  as **`?id=`** (e.g. `?id=220-1652-744-1341-230`). An **empty `stashes` array
  means end of stream.** Updates surface with roughly a **~5-minute delay**.
- The data model is **PoE1-specific** (prophecies, divination cards,
  elder/shaper maps, etc.). **No PoE2 equivalent exists.**

---

## 4. Where PoE2 market data actually comes from

The biggest practical gap above (no PoE2 public-stash river) is why PoE2 economy
sites and overlays don't all work the same way. There are **three** data paths,
only one of which is officially sanctioned.

### A. Currency economy → official Currency Exchange API ✅ sanctioned

- `GET /currency-exchange[/<realm>][/<id>]`, scope **`service:cxapi`**, **supports
  `realm=poe2`**. Documented and OAuth-gated (Client Credentials).
- Returns **hourly aggregate digests** of currency-pair trade history (volumes,
  stock, exchange ratios), cursor-paginated via `next_change_id`. **Historical
  aggregates only — no live listings, no item-level data.**
- This is the in-game Currency Exchange / market system. **poe.ninja's PoE2
  "Economy" section is built on it** ("leverages the game's Currency Exchange
  system").

### B. Item-level prices → the undocumented `trade2` site API ⚠️ against ToS

- The PoE2 trade *website* (`pathofexile.com/trade2`) is backed by
  reverse-engineered HTTP endpoints **not in the developer docs**:
  - `POST https://www.pathofexile.com/api/trade2/search/{league}` → ordered list
    of result IDs + a query token.
  - `GET https://www.pathofexile.com/api/trade2/fetch/{ids}?query={queryId}` →
    listing details, **10 IDs per batch**.
  - plus `POST /api/trade2/exchange/{league}` (bulk) and `/api/trade2/data/static`.
- **Auth:** works largely unauthenticated, but a **POESESSID** session cookie
  raises rate limits and unlocks private leagues. **Cloudflare bot-protection**
  (CAPTCHAs / IP blocks, worst at league launch) is the real gatekeeper.
- **Rate limits:** the same dynamic `X-Rate-Limit-*` + `Retry-After` scheme,
  enforced and easy to trip.
- ⚠️ **This is against GGG's ToS.** The developer docs state plainly: *"It is
  against our Terms of Use (section 7i) to reverse-engineer endpoints outside of
  this documentation"* and *"Requests for access to any other internal website
  APIs … will be denied."* ToS §7 also forbids scraping (7f) and automation (7c).
- **poe.ninja uses this for PoE2 item/unique prices** — by its own admission:
  *"There is no River API for Path of Exile 2 yet, so prices are estimated from
  the official trade API."*

### C. Derived / aggregated community feeds

- Tools also consume each other: **poeprices.info** (ML price-prediction for rare
  items), poe.ninja deep-links / currency conversion, and tool-specific backends
  (e.g. Exiled Exchange 2 serves its own aggregated economy data from
  `api.exiledexchange2.dev`).

### How the clipboard overlays sidestep the problem (Exiled Exchange 2)

**Exiled Exchange 2** (`Kvan7/Exiled-Exchange-2`, a fork of Awakened PoE Trade) is
the leading PoE2 price-checker. It never needs a bulk river because it prices
**one item at a time, on a user keypress**:

1. User hits a hotkey over an in-game item → app reads the **OS clipboard** item
   text. It does **not** read game memory or inject into the client.
2. App drives the **`trade2` search + fetch** endpoints through an in-app Electron
   proxy with **`useSessionCookies: true`**, riding the **user's own logged-in
   pathofexile.com session** (POESESSID). It honors `X-Rate-Limit-*` headers via
   a proactive token bucket (default 1 req / 5 s per bucket).
3. For aggregates it also pulls its own backend + poeprices.info.

This "**one server action per keypress, no game-client interaction, user's own
session**" pattern is the community's tolerated-in-practice compliance norm —
**not** an official endorsement (GGG keeps no approved-app allowlist, and there is
documented precedent of a 2020 ban wave against a different overlay).

### What this means for `poe2-mcp`

- **Currency/economy data is cleanly available** via the official Currency
  Exchange API with a confidential client + `service:cxapi`. This is the safe
  foundation.
- **Item-level live pricing has no sanctioned server-side path.** Every tool that
  does it scrapes `trade2` (ToS §7i) while relying on a **user's own session +
  passing Cloudflare** — viable for a desktop overlay tied to a logged-in player,
  risky/uncertain for a headless server. An MCP server cannot legitimately serve
  bulk PoE2 item prices the way PoE1 tools serve stash-river data.
- Realistic options: (a) build on the sanctioned Currency Exchange API; (b) for
  item prices, act only within a single user's authenticated session + request
  budget, mirroring the overlay model; and/or (c) consume an existing aggregator
  rather than scraping `trade2` ourselves.

---

## 5. Authentication — OAuth 2.1 (not OIDC)

GGG implements **OAuth 2.1** (referencing the OAuth 2.1 Authorization Framework
draft RFC). It is **explicitly not OpenID Connect** — there are no OIDC mentions
in the docs and no `.well-known/openid-configuration` discovery document. So
this is **authorization for API access, not an identity/login (ID-token)
provider.**

### Endpoints (all on `www.pathofexile.com`)

| Purpose | URL |
|---------|-----|
| Authorization | `https://www.pathofexile.com/oauth/authorize` |
| Token | `https://www.pathofexile.com/oauth/token` |
| Revoke | `https://www.pathofexile.com/oauth/token/revoke` |
| Introspect | `https://www.pathofexile.com/oauth/token/introspect` |

### Client types

| | **Confidential** (standard) | **Public** |
|---|---|---|
| Who | *"the vast majority of applications"* | Native/desktop/CLI apps without a server secret |
| Grants | Authorization Code, **Client Credentials** | **Authorization Code + PKCE only** |
| `service:*` scopes | ✅ Allowed | ❌ **Not allowed** |
| Redirect URI | HTTPS, **registered domain** — **no IP / no localhost, even in dev** | **Local** URI, e.g. `http://127.0.0.1:8080/callback` |
| Access token life | **28 days** | **10 hours** |
| Refresh token life | **90 days** | **7 days** |

### Scopes

**Account scopes** — Authorization Code grant; act on a user's behalf; do **not**
require a confidential client:

| Scope | Grants access to |
|-------|------------------|
| `account:profile` | Basic profile info |
| `account:characters` | Characters and their inventories |
| `account:stashes` | Stashes and items *(PoE1 data only)* |
| `account:item_filter` | Managing item filters |
| `account:leagues` | Available / private leagues |
| `account:league_accounts` | Allocated atlas passives |
| `account:guild:stashes` | Guild stashes — **PoE1 only** |

**Service scopes** — Client Credentials grant; **require a confidential
client**:

| Scope | Grants access to |
|-------|------------------|
| `service:leagues` | League listings |
| `service:leagues:ladder` | League ladders |
| `service:pvp_matches` | PvP matches *(PoE1)* |
| `service:pvp_matches:ladder` | PvP ladders *(PoE1)* |
| `service:psapi` | Public Stash API *(PoE1)* |
| `service:cxapi` | Currency Exchange API |

### PKCE (required for public clients)

- `code_challenge_method` must be **`S256`**.
- `code_challenge` = **base64url( SHA256( code_verifier ) )**.
- `code_verifier` must carry **≥ 32 bytes of entropy** (RFC 7636 §4.1).
- **Authorization codes expire after 30 seconds** — exchange them immediately.
- Note: OAuth 2.1 makes PKCE mandatory generally.

### Which flow for an MCP server?

- To act on a **logged-in user's** account data (profile, characters, filters):
  **Authorization Code (+ PKCE)** with `account:*` scopes. A user-facing browser
  consent step is required.
- For **service-level** data (`service:*`, e.g. currency exchange): **Client
  Credentials** with a **confidential client** — no per-user consent, but you
  need a server-side secret and an HTTPS registered-domain redirect URI.

---

## 6. Registration / onboarding

- **No self-service signup.** *"Registration is currently handled by us directly
  at our discretion. You can make a request by emailing
  `oauth@grindinggear.com`."*
- ⚠️ **"We will immediately reject any low-effort or LLM-generated requests. We
  expect you to read and understand these docs, and to have a clear vision for
  what you want to do before requesting OAuth access."** → For `poe2-mcp`, the
  application email **must be human-written**, specific, and demonstrate a clear
  product vision. Pitching "an AI/MCP gateway" without a concrete, well-scoped
  use case risks auto-rejection.
- Approved applications (and their tokens) are reviewed/revoked at
  `https://www.pathofexile.com/my-account/applications`.

---

## 7. Other consumption notes

### Required User-Agent

Every request **must** set an identifiable User-Agent:

```
User-Agent: OAuth {clientId}/{version} (contact: {contact})
```

Example: `OAuth mypoeapp/1.0.0 (contact: mypoeapp@gmail.com) SomeOptionalThingHere`
(trailing content is allowed). This is marked **required** in the headers spec.

### Rate limiting — dynamic, read at runtime

Limits are **per-endpoint and dynamic**, communicated via response headers.
**Do not hard-code limits — parse the headers each response.**

| Header | Meaning |
|--------|---------|
| `X-Rate-Limit-Policy` | Policy name applying to this request |
| `X-Rate-Limit-Rules` | Comma-delimited active rules (e.g. `ip`, `account`, `client`) |
| `X-Rate-Limit-{rule}` | `max_hits : period_seconds : restriction_time` |
| `X-Rate-Limit-{rule}-State` | `current_hits : period : active_restriction_seconds` |
| `Retry-After` | Seconds to wait after being throttled |

> 🚫 **Refuted by verification — do NOT trust these numbers** that circulate in
> community libs/forums: "5 req/s general", "1 req/s public-stash", and
> trade-search per-IP buckets (5/12s, 15/62s, 30/302s). All three failed
> adversarial checks (0-3). Always read `X-Rate-Limit-*` + `Retry-After` live.

### Gotchas

- **Community libraries are stale.** The widely-cited `moepmoep12/poe-api-ts`
  was **archived Dec 2023** (last push Mar 2023) — predates PoE2 early access
  (Dec 2024) and has zero PoE2 awareness. Useful for the shared OAuth/header
  conventions, useless for PoE2-specific endpoints.
- **Community wikis can be wrong on URLs/versions** (one had a stale legacy URL
  and a wrong version attribution). Trust them for *behavior*, not specifics.
- **Token rotation:** confidential access tokens last 28 days / refresh 90 days;
  public 10 hours / 7 days. Build refresh handling accordingly.
- **30-second auth-code window** is tight — exchange synchronously.

---

## 8. Open questions for `poe2-mcp` (unresolved by research)

1. **What must the `oauth@grindinggear.com` application contain**, and what's the
   approval turnaround / acceptance bar for an MCP-server use case (given
   LLM-generated requests are auto-rejected)?
2. **Exactly which fields/properties does `realm=poe2` return** on the
   cross-version endpoints (profile, characters, leagues, currency-exchange)?
   Confirmed *that* they support PoE2; the exact PoE2 payload shape isn't
   enumerated in the docs.
3. **Will GGG ship a PoE2 public-stash / trade API?** As of this research there is
   none and none announced — so item-level pricing has no sanctioned path (see
   §4). Worth re-checking the changelog periodically.
4. **Concrete rate-limit values** for the endpoints we'd hit most
   (currency-exchange; and, if used, `trade2` search/fetch), and whether a
   confidential client gets higher limits than a user-session (`Account` rule)
   on `trade2`.

---

## 9. Sources

**Primary (GGG official):**
- <https://www.pathofexile.com/developer/docs> · `/index` · `/authorization` · `/reference` · `/changelog`
- <https://www.pathofexile.com/legal/terms-of-use-and-privacy-policy> (ToS §7 c/f/i — automation/scraping/reverse-engineering)
- <https://www.pathofexile.com/forum/view-thread/3821465> (staff: docs location + `oauth@` address)
- <https://www.pathofexile.com/forum/view-thread/3257587> (rate-limit headers, live attestation)
- <https://www.pathofexile.com/forum/view-thread/3655582> (a real `trade2` search request in the wild)

**Market-data sourcing (follow-up round):**
- poe.ninja: <https://poe.ninja/posts/poe2-unique-items> ("no River API … estimated from the official trade API") · <https://poe.ninja/posts/poe2-economy-and-rise-of-the-abyssal> (Currency Exchange system)
- Exiled Exchange 2: <https://github.com/Kvan7/Exiled-Exchange-2> (fork of <https://github.com/SnosMe/awakened-poe-trade>) — `trade2` paths, 10-ID fetch batches, rate-limit buckets, session proxy
- Awakened PoE Trade FAQ: <https://snosme.github.io/awakened-poe-trade/faq> (clipboard mechanism + tolerance norms)
- PoB-PoE2 trade system: <https://deepwiki.com/PathOfBuildingCommunity/PathOfBuilding-PoE2/5.1-trade-system>

**Secondary / community (corroborating behavior, treat as stale for PoE2):**
- <https://github.com/moepmoep12/poe-api-ts> (archived 2023; OAuth/header conventions)
- <https://pathofexile.fandom.com/wiki/Public_stash_tab_API> & <https://poewiki.net/wiki/Public_stash_tab_API>
- <https://github.com/Chuanhsing/poe-api/blob/master/poe.yaml> (unofficial OpenAPI, outdated)
- <https://pkg.go.dev/github.com/willroberts/poeapi> (PoE1-era Go client)

**Verification:** Round 1 — 16 sources → 68 claims → 25 adversarially verified
(22 confirmed / 3 refuted). Follow-up (§4 market-data) — 3 focused agents, all
findings source-backed against GGG docs/ToS and the Exiled-Exchange-2 source.
Confidence on the core docs/auth/coverage facts: **high**; PoE2-specific payload
details and `trade2` enforcement posture: **evolving / inferred**.
