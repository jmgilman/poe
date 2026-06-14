---
title: Overview
slug: /
description: An MCP server for Path of Exile 2 — an AI gateway into the PoE marketplace.
---

# Path of Exile 2 MCP Server

`poe2-mcp` is a [Model Context Protocol](https://modelcontextprotocol.io) (MCP)
server for [Path of Exile 2](https://www.pathofexile.com), built on the official
[`modelcontextprotocol/go-sdk`](https://github.com/modelcontextprotocol/go-sdk).
It is an AI gateway into the PoE marketplace — authenticating against the
official PoE API to help with pricing, market research, and gearing.

It currently ships a transport-agnostic server with one demo tool (`random_int`)
served over either the STDIO or Streamable HTTP transport, plus Moon tasks,
pinned CI, dependency automation, secure-by-default settings, and an exercised
release pipeline, while the marketplace tooling is built out.

## Documentation

This site follows the [Diátaxis](https://diataxis.fr/) structure:

- **[Getting started](getting-started.md)** — a tutorial: build and run the
  server over both transports.
- **[Add a tool](add-a-tool.md)** — a how-to: replace `random_int` or add your
  own tool alongside it.
- **[Configuration](configuration.md)** — reference for the CLI flags,
  `POE2_MCP_*` environment variables, and transports.
- **[Security](security.md)** — an explanation of the server's
  secure-by-default choices and how to harden a real deployment.

The Go API reference is published on
[pkg.go.dev](https://pkg.go.dev/github.com/jmgilman/poe).
