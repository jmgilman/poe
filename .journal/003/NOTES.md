---
id: 003
title: Local MCP market design
started: 2026-06-14
---

## 2026-06-14 19:00 — Kickoff
Goal for the session: capture the Exiled Exchange 2 market/pricing process in `docs/drafts/market_design.md`, focusing on the hard-to-preserve design details for making a similar local browser-session-backed flow work through poe2-mcp.
Current state of the world: the repo is still mostly scaffolded (`random_int` remains the only real tool), journal notes already record that official PoE2 item-level market pricing has no sanctioned API path, and a temp clone of Exiled Exchange 2 was inspected at `/tmp/exiled-exchange-2.hymD2u` commit `006aa1cef60d25d6a24b9aeaaff6f34febc9d076`.
Plan: create an isolated implementation branch, draft the document from the verified EE2 flow, and keep the document focused on local-session mechanics, security boundaries, rate-limit behavior, and MCP-specific design implications.
