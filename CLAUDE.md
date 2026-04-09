# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sublink Worker is a lightweight proxy subscription converter that transforms proxy protocol URIs (SS, VMess, VLESS, Hysteria2, Trojan, TUIC) into client configs for Sing-Box, Clash, Surge, and Xray/V2Ray. It runs on multiple runtimes: Cloudflare Workers, Vercel, Node.js, and Docker.

## Commands

### Development
```bash
npm run dev          # Cloudflare Workers local dev (wrangler dev)
npm run dev:node     # Node.js local dev (build + run on port 8787)
```

### Build
```bash
npm run build        # Vercel build (esbuild -> dist/vercel/)
npm run build:node   # Node.js build (esbuild -> dist/node-server.cjs)
```

### Test
```bash
npm test             # Run all tests (vitest with Cloudflare Workers pool)
npx vitest run test/worker.test.js          # Run a single test file
npx vitest run -t "test name pattern"       # Run tests matching a name
```

### Deploy
```bash
npm run deploy       # Deploy to Cloudflare Workers (runs setup-kv first)
```

## Architecture

### Multi-Runtime Design

The app uses a **runtime abstraction pattern** to support multiple deployment targets from a single codebase:

- **Entry points**: `src/worker.jsx` (Cloudflare), `src/platforms/node-server.js` (Node.js), `api/index.js` (Vercel)
- **Runtime factories** in `src/runtime/`: each creates a `{ kv, assetFetcher, logger, config }` object
  - `cloudflare.js` — uses Cloudflare KV binding (`SUBLINK_KV`) and Workers Assets
  - `node.js` — resolves KV from env vars (Redis > Upstash > Memory), serves static files from disk
  - `vercel.js` — same KV resolution as Node, bundles via `scripts/build-vercel.mjs`
- `src/runtime/runtimeConfig.js` — `normalizeRuntime()` fills defaults for any runtime

All entry points call `createApp(runtime)` from `src/app/createApp.jsx`, which returns a Hono app.

### Web Framework

Uses **Hono** with JSX (server-side rendered). Components in `src/components/` are Hono JSX, not React. The JSX pragma is `hono/jsx`.

### Config Builder Pipeline

`BaseConfigBuilder` (abstract) → `SingboxConfigBuilder`, `ClashConfigBuilder`, `SurgeConfigBuilder`

Flow: `build()` → `parseCustomItems()` (fetch/parse subscriptions) → `addCustomItems()` (convert proxies) → `addSelectors()` (create proxy groups + rules) → `formatConfig()`

- `src/builders/helpers/` — shared utilities for group building, proxy conversion, country grouping
- `src/config/` — base configs, rule sets (predefined: minimal/balanced/comprehensive), rule generators

### Protocol Parsers

`src/parsers/ProxyParser.js` dispatches by URI scheme to protocol-specific parsers in `src/parsers/protocols/`. Also handles HTTP subscription fetching (`src/parsers/subscription/`).

Parsed proxies use a **unified internal format** with a `tag` field as identifier, then each builder's `convertProxy()` transforms to the target client format.

### KV Storage Adapters

All adapters in `src/adapters/kv/` implement a common interface: `get(key)`, `put(key, value, options?)`, `delete(key)`.

Used by two services:
- `ShortLinkService` — short URL codes for subscription links
- `ConfigStorageService` — store custom base configs by ID

### API Routes

| Route | Description |
|-------|-------------|
| `/singbox` | Sing-Box JSON config |
| `/clash` | Clash YAML config |
| `/surge` | Surge config |
| `/xray` | Base64-encoded proxy list |
| `/subconverter` | Subconverter-compatible INI |
| `/shorten-v2` | Create short link |
| `/s/:code`, `/b/:code`, `/c/:code`, `/x/:code` | Short link redirects |
| `/config` | POST to store custom base config |
| `/resolve` | Resolve short link to original URL |

### i18n

`src/i18n/index.js` — translation dictionaries for zh-CN, en-US, fa, ru. `createTranslator(lang)` returns a `t()` function. Language resolved from `?lang=` query param or `Accept-Language` header.

## Testing

Tests use `vitest` with `@cloudflare/vitest-pool-workers` (runs in Cloudflare Workers runtime simulation). Config in `vitest.config.js` references `wrangler.toml`. Test files are in `test/` and typically import builders/parsers directly or test HTTP endpoints via the worker.

## Key Conventions

- Pure JavaScript (no TypeScript), ESM (`"type": "module"`)
- JSX files use `.jsx` extension with Hono JSX runtime
- Config query params use `config` for the subscription input (newline-separated URLs/URIs)
- `selectedRules` accepts predefined preset names (`minimal`, `balanced`, `comprehensive`) or JSON arrays
- The `wrangler.toml` KV binding is `SUBLINK_KV`
