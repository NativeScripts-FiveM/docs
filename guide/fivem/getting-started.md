# Overview

**Native Scripts (FiveM)** is a framework-agnostic script collection for FiveM (GTA5) servers. The goal: a single codebase that runs unchanged on ESX, QBCore, Qbox (qbx_core) and standalone.

## Core layer: `ns-utils`

Unlike the RedM side (which uses a shared `ns-lib` resource), FiveM scripts use **`ns-utils`** — a self-contained `utils/` folder **copied into each resource**. There is no shared dependency to install. On startup it auto-detects the framework, inventory, target, skin and SQL systems and exposes a single `Utils.*` API:

```lua
Utils.OnPlayerLoaded(function(src)
    local player = Utils.GetPlayer(src)   -- framework-agnostic
end)

if Utils.HasItem(src, 'water', 1) then
    Utils.RemoveItem(src, 'water', 1)
    Utils.AddMoney(src, 'cash', 10, 'water_sale')
    Utils.Notify(src, '+$10 (water sold)', 'success')
end
```

You never branch on the framework name — all of that lives inside `utils/`.

## Server setup

1. Get any ns-* FiveM script from the [nativescripts.com](https://nativescripts.com) catalog
2. Drop it into your `resources/` folder
3. Add it to `server.cfg`:
   ```cfg
   ensure ns-loadingscreen
   ```
4. Apply the SQL and item registrations from the script's README (if any)
5. Restart — the console prints the detected stack:
   ```
   [ns-loadingscreen][INFO] shared.lua ready (framework=qbx, inv=ox_inventory, target=ox_target, skin=illenium-appearance, sql=oxmysql)
   ```

### Optional dependencies

- **ox_lib** — if present, `Utils.Notify` routes through it for nicer notifications. Otherwise it falls back to the framework-native notify.
- **oxmysql** — required only for scripts that use a database (`Utils.ExecuteSql`).

## Detection priority

| Category | Priority (first match wins) |
|---|---|
| Framework | `qbx_core` → `qb-core` → `es_extended` → standalone |
| Inventory | `ox_inventory` → `qb-inventory` → `qs-inventory` → `codem-inventory` → `ps-inventory` → `esx_inventoryhud` → `gfx-inventory` |
| Target | `ox_target` → `qb-target` → `qtarget` |
| Skin | `illenium-appearance` → `fivem-appearance` → `qb-clothing` → `esx_skin` → `skinchanger` |
| SQL | `oxmysql` → `ghmattimysql` → `mysql-async` |

## Next

- [ns-utils →](/scripts/fivem/ns-utils) — the full `Utils.*` API reference
- [Scripts →](/scripts/fivem/ns-loadingscreen-fivem) — documentation for each FiveM script
