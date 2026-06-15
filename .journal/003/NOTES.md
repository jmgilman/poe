---
id: 003
title: Local MCP market design
started: 2026-06-14
---

## 2026-06-14 19:00 — Kickoff
Goal for the session: capture the Exiled Exchange 2 market/pricing process in `docs/drafts/market_design.md`, focusing on the hard-to-preserve design details for making a similar local browser-session-backed flow work through poe2-mcp.
Current state of the world: the repo is still mostly scaffolded (`random_int` remains the only real tool), journal notes already record that official PoE2 item-level market pricing has no sanctioned API path, and a temp clone of Exiled Exchange 2 was inspected at `/tmp/exiled-exchange-2.hymD2u` commit `006aa1cef60d25d6a24b9aeaaff6f34febc9d076`.
Plan: create an isolated implementation branch, draft the document from the verified EE2 flow, and keep the document focused on local-session mechanics, security boundaries, rate-limit behavior, and MCP-specific design implications.

## 2026-06-14 19:04 — Draft committed
Created Worktrunk branch/worktree `feat/market-design-doc` at `/Users/josh/code/poe/.wt/feat-market-design-doc`.
Added `docs/drafts/market_design.md` as a draft design note covering Exiled Exchange 2's trade-query/listing-summarizer model, `trade2` versus official API boundaries, local browser-profile authentication, query/fetch routing, rate-limit and complexity handling, result semantics, cookie/log safety, and a narrow MCP tool surface.
Validation: `git diff --check` passed, and `moon run docs:build --summary minimal` passed with only the upstream Material/MkDocs 2.0 warning. Implementation commit: `6d578fd` (`docs: draft local market design`).

## 2026-06-14 19:13 — Browser-profile addendum
Added an addendum that argues for `Go MCP server + chromedp/Rod + dedicated profile dir + visible login command + narrow typed trade tools` as the first local implementation path instead of copying Electron.
The addendum preserves the critical distinction that EE2's useful pattern is a persistent browser cookie jar plus typed market requests, not Electron itself. It also records the dedicated-profile boundary, visible login flow, explicit auth status probe, session clearing command, and the request-routing tradeoff between browser-context calls and a scoped HTTP client.
Validation: `git diff --check` passed. Implementation commit: `23187a7` (`docs: add browser profile addendum`).

## 2026-06-14 20:30 — Internal library design draft
Reworked `docs/drafts/market_design.md` into a proper initial design for an internal Go trade website session library. The scope is now explicitly limited to dedicated browser-session management, user-assisted login, and typed trade website queries/responses; item parsing, image work, pricing, and MCP tool details are listed as non-goals.
The design records the intended hexagonal shape: `internal/trade` as core API and ports, `internal/trade/chromium` for the browser/profile adapter, `internal/trade/tradeweb` for endpoint IO, and optional `internal/trade/local` assembly. It also captures constructor shape, small port interfaces, capability-based auth status, visible login behavior, dedicated profile handling, typed query/response strategy, dynamic rate limits, sanitized errors/logging, mockery-driven unit tests, `httptest` adapter tests, and an agile first implementation slice.
Validation: `git diff --check` passed, and `moon run docs:build --summary minimal` passed with only the upstream Material/MkDocs 2.0 warning. Implementation commit: `609c4a8` (`docs: draft trade session library design`).
