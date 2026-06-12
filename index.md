---
title: Native Scripts — FiveM
---

# Native Scripts — FiveM

A framework-agnostic, production-ready script collection for **FiveM (GTA5)** servers. The same codebase runs unchanged on **ESX, QBCore, Qbox (qbx_core), and standalone**.

> **🧰 Core:** FiveM scripts ship with the `ns-utils` layer — a self-contained `utils/` folder copied into each resource. It auto-detects the framework, inventory, target, skin and SQL systems at runtime. Your code only ever calls `Utils.X(...)`. No shared library dependency.

## Quick start

1. Grab any ns-* FiveM script from [nativescripts.com](https://nativescripts.com)
2. Drop it into `resources/`
3. Add it to `server.cfg`:

```cfg
ensure ns-loadingscreen
```

That's it — `ns-utils` is bundled inside each resource, so there's nothing extra to install. Optional: `ox_lib` (nicer notifications) and `oxmysql` (for scripts that use a database).

The `Utils.*` layer is explained in [How it works](#how-it-works) below.

## Scripts

- **[ns-loadingscreen](/scripts/fivem/ns-loadingscreen-fivem)** — Themed loading screen with rotating backgrounds, in-screen music player, server rules panel and rotating tips.

## How it works

Every FiveM script bundles `ns-utils`. There is **no shared `ns-lib` dependency** (that's the RedM side) — instead `utils/shared.lua`, `utils/client.lua` and `utils/server.lua` live inside each resource and detect the active framework on startup. You write against `Utils.*` and never branch on the framework name.

## Community

- **Store:** [nativescripts.com](https://nativescripts.com) — full catalogue, free + paid
- **Discord:** [discord.gg/UyyngemnF8](https://discord.gg/UyyngemnF8) — bug reports, feature requests, server invites
- **RedM docs:** [nativescripts-redm.github.io/docs](https://nativescripts-redm.github.io/docs/) — the RDR3 side of the catalogue
