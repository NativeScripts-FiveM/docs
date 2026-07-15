# ns-leaderboard

Framework-agnostic **in-world PvP leaderboard** for FiveM (ESX / QBCore / Qbox / standalone).
Top players are spawned as **podium peds with crowns**, kills are **cross-validated** server-side,
and everything — anim shop, badges, avatars, platform placement — is driven from one clean NUI.

Yellow / dark cyber UI (Chakra Petch), rendered to an in-world DUI surface.

> Full docs: **https://fivem.nativescripts.com/docs/**

---

## Features

- **Podium peds + crowns** — the top-9 of each board spawn in-world as frozen NPCs wearing the real player's skin + chosen animation. Gold / silver / bronze crown on the top-3 (only on the podium ped — never attached to a live player). Name floats above each ped in the player's chosen colour (DrawText3D, no NUI overlay).
- **Multiple boards** — `kill`, `death`, `playedtime`, `kd`, and `point` (from cash). Each board is its own podium; enable/place only the ones you want.
- **Live podium recolour** — one admin command (`/nspodiumcolor <colour>`) retints the podium **and** the signs (TOP KILLS / DEATHS / TIME / POINTS) in real time for everyone. 7 built-ins (yellow, blue, green, red, orange, purple, pink); the choice is saved server-side and persists across relogs and restarts. One model, no restart — a runtime texture is swapped in.
- **Real-time, low-overhead updates** — a diff push (1s debounce) plus a surgical client update: when one player changes skin/anim or the ranking shifts, only the ped that actually changed re-spawns — not the whole podium, and not for everyone.
- **Distance streaming** — a podium spawns only for players near it (`Config.PlatformStreamDistance`, default 150 m) and despawns beyond, so far players carry no idle podium peds. Set `0` to always spawn.
- **In-game placement tool** — place / edit podiums live with a mouse gizmo; the layout is saved to `data/platforms.json` and hot-reloads for everyone. No config editing, no copy-paste, no restart (see **Platform setup**).
- **Cross-validated, anti-cheat kills** — attacker **and** victim must both report; a lone client cannot fabricate a kill. Self-kills are rejected, distance limits are weapon-aware (snipers / explosives get a long-range cap so real long-shots and RPG multi-kills always count).
- **Anim shop** — 15 built-in animations (free + paid), live TRY preview on your own ped, per-podium assignment, themed icons, scrollable grid. Anim buys can fall back cash → bank. Add default or custom (add-on) anims with one config line.
- **Badge system** — kill-total / weapon-category / kill-streak / headshot / playtime / K-D triggers. One active badge shows on the row, above the podium ped, and via an export.
- **Customisable columns** — every leaderboard column (crew, kill, death, kd, playtime, point) can be toggled in config.
- **Avatars** — Discord (bot token) → Steam → default fallback, with an in-game picker and per-identifier cache.
- **Clothing-script sync** — the podium ped updates within ~2s when a player changes outfit; auto-detects popular clothing/appearance scripts, with a config override.
- **HUD auto-hide** — the fullscreen panel hides the vanilla minimap/HUD and auto-detects common NUI huds so they don't draw over it.
- **Profile modal** — kills / deaths / K-D / headshots / playtime / accuracy / most-used weapons, owned badges, last-seen, all from built-in tracking.
- **10 languages** — `en, tr, fr, de, es, pt, it, pl, nl, ru` — pick one in config, add your own easily.
- **Built on `ns-utils`** — all framework branching is behind `Utils.*`; works on ESX, QBCore, Qbox, or standalone.

---

## Dependencies

- **oxmysql** — required (SQL backend).
- **ESX / QBCore / Qbox** — any one, auto-detected at runtime. Not required (runs standalone too).

Optional, auto-detected — no config needed:

- **Clothing / appearance** — illenium-appearance / fivem-appearance / rcore_clothing / tgiann / bl_appearance / qb-clothing / esx_skin / skinchanger (for the podium ped skin). Override in `Config.Clothing`.
- **Inventory** — ox_inventory / qb-inventory / … (via `Utils`).
- **ox_lib** — for notifications (native fallback if absent).
- **ns-crew** — crew tag on the row, if present.

---

## Installation

The resource is **self-contained** — the podium, crown, sign and animation stream assets all ship inside `stream/`. No extra downloads.

**Before you start:** make sure **oxmysql** is installed and started. A framework (ESX / QBCore / Qbox) is auto-detected but not required — it also runs standalone.

**1. Drop the resource** into your server, e.g. `resources/[ns]/ns-leaderboard/`. Keep the folder name **`ns-leaderboard`**.

**2. Import the database** (one time). Open your database with the tool you already use — **HeidiSQL**, **phpMyAdmin**, or your hosting panel's **"Databases → SQL / Import"** tab — then open the **`sql/install.sql`** file (it's inside this resource) and click **Run / Go / Execute**. That's all. The table updates itself on future versions, so you never touch SQL again.

**3. Add to `server.cfg`** — order matters (oxmysql, then your framework if any, then this):
```cfg
ensure oxmysql
# ensure es_extended   / qb-core / qbx_core   (only if you use one)
ensure ns-leaderboard
```

**4. Restart the server.** Open the leaderboard with **`/leaderboard`**.

**5. Place your podiums** (admin) — see **Platform setup** below. Type `/nsplatform-tool kill`, position it with the mouse, done. Repeat for `death`, `playedtime`, `point`.

**6. (Optional) Pick a colour:** `/nspodiumcolor blue` recolours the podium + signs for everyone and is saved.

That's it. Everything else (avatars, badges, anim shop, HUD hiding) works out of the box; tune it in `config.lua`.

> **Sample layout:** the resource ships with a demo podium placement so it works immediately. To start clean, run `/nsplatform-delete all`, then place your own with `/nsplatform-tool`.

> **Discord avatars (optional):** to show players' Discord avatars, add your bot token to `server.cfg` with `set` (never `setr`) — see **Discord avatar** below.

---

## Platform (podium) setup

Podium positions live in **`data/platforms.json`**, written by the in-game tool — you never edit coordinates by hand.

**Place a podium** (admin):
```
/nsplatform-tool kill
```
`<type>` is one of `kill · death · playedtime · kd · point`. You position **only the platform** with the mouse gizmo; the sign and all 9 ped slots auto-place from `Config.PlatformTemplate`. Modes:

| Mode | Command | What it does |
|---|---|---|
| template (default) | `/nsplatform-tool kill` | place the platform → sign + peds auto-follow |
| manual | `/nsplatform-tool kill manual` | place every piece by hand |
| ring | `/nsplatform-tool kill ring` | place the top ped, auto-ring the rest |
| **edit** | `/nsplatform-tool kill edit` | re-open the current placement; ←/→ pick a piece, **E** grab with the gizmo, **Enter** save all |

The result is **saved to JSON and live-reloads for every player instantly** — no restart, no pasting.

**Gizmo controls:** mouse = drag handles · **W** move · **R** rotate · **Q** world/local · **LAlt** snap to ground · **Enter** confirm · **Backspace** cancel.

**Manage placements:**
```
/nsplatform-refresh              -- reload data/platforms.json from disk (after a hand-edit)
/nsplatform-delete <type|all>    -- remove a podium (recoverable by re-placing)
```

A type only spawns when it is enabled in `Config.PlatformEnabled` **and** has a saved placement. Enable the boards you want there.

**Recolour the podium + signs** (admin):
```
/nspodiumcolor blue        -- yellow · blue · green · red · orange · purple · pink
```
Applies live to everyone, retints both the podium accent and all signs, and is saved (server KVP) so it survives relogs and restarts. The white play triangle on the TIME sign stays white by design. **Add a colour:** drop a solid-colour `html/assets/podium/<key>.png` and add a line to `Config.PodiumColors` — that's it.

---

## Configuration (`config.lua`)

| Section | Key | Description |
|---|---|---|
| General | `Config.Locale` | `en, tr, fr, de, es, pt, it, pl, nl, ru` (or your own) |
| General | `Config.Debug` | `true` enables the debug/test commands + AC accept-logs |
| General | `Config.OpenCommand` | leaderboard open command (default `leaderboard`) |
| General | `Config.SaveInterval` | DB write period (ms, default 15m) |
| General | `Config.PlayerPerPage` | rows per page (default 30) |
| Columns | `Config.Columns` | toggle each column: `crew, kill, death, kd, playtime, point` |
| Point | `Config.PointMoneyType` | wallet the POINT board reads (default `cash`) |
| Platform | `Config.PlatformEnabled[type]` | which boards are active |
| Platform | `Config.SignModels[type]` | sign prop per type (`false` = no sign) |
| Platform | `Config.PlatformLabelDistance` | how far (m) the ped name labels show (default 15) |
| Platform | `Config.PlatformStreamDistance` | spawn a podium only for players within this many m; `0`/`false` = always (default 150) |
| Podium colour | `Config.PodiumColors` | key → label map for `/nspodiumcolor` (add a colour = a PNG + a line) |
| Podium colour | `Config.PodiumDefaultColor` | colour used before an admin sets one (default `yellow`) |
| Anim | `Config.Animations` | the anim-shop catalogue (see below) |
| Anim | `Config.MoneyType` / `Config.MoneyFallback` | wallet charged for anim buys; fallback to the other wallet if short |
| Badge | `Config.Badges` | badge table (key → label + icon + colour + trigger) |
| Badge | `Config.WeaponCategoriesForBadges` | weapon hashes per category (weapon-kill triggers) |
| HUD | `Config.HideHudWhenOpen` | hide vanilla HUD/radar while the panel is open |
| HUD | `Config.HudCompat` | auto-hide third-party NUI huds (qb-hud, ps-hud, …) |
| Clothing | `Config.Clothing` | force a clothing script (all `nil` = auto-detect) |
| Anti-cheat | `Config.AntiCheat.*` | see below |
| Reset KDA | `Config.ResetKdaCost` | cost to reset your own K/D |

Commands are listed in **`COMMANDS.txt`** (test commands only register when `Config.Debug = true`).

---

## Anti-cheat & kill validation

A kill is only counted when the **attacker's** and the **victim's** client both report it and the two match (5s window). A lone client can't fabricate a kill, and **self-kills are rejected**. On top of that:

| Key | Default | Purpose |
|---|---|---|
| `MaxAttackDistance` | `400.0` | max attacker→victim distance for normal weapons (m) |
| `MaxAttackDistanceSniper` | `1500.0` | larger cap for snipers + explosives/launchers |
| `MaxKillsPerMinute` | `60` | per-attacker rate limit (anti-farm) |
| `MaxCombatReportsPerMinute` | `40` | per-reporter combat-batch rate limit |
| `MaxAttackersPerReport` | `8` | attackers credited per report (anti mass-inflate) |
| `RoutingBucketCheck` | `true` | require attacker + victim in the same OneSync bucket |

The distance cap is **weapon-aware**: snipers and explosives (RPG, grenade/homing launchers, railgun, …) use the larger sniper cap, so long-range shots and RPG multi-kills are never dropped. Distance is a sanity check (coords are client-reported); the real protection is the cross-validation + self-kill guard + rate limit — keep the caps generous so legit PvP counts.

**Tuning:** a dropped kill is silent to players, so each rejection logs the measured value next to the threshold it broke. Set `Config.Debug = true` to also log every **accepted** kill's distance/weapon — that distribution tells you where your caps belong.

---

## Anim shop

`Config.Animations` is the catalogue. **Adding one animation = one line** — the card, icon, buy, TRY preview and assignment are all automatic:

```lua
['mykey'] = { label = 'My Anim', dict = '<animDict>', anim = '<animName>', price = 0 },
```

- `price = 0` → free; anything else → buyable.
- **Custom (add-on) anims** work exactly like default ones — just stream the `animDict`'s `.ycd` from a resource `ensure`d **before** ns-leaderboard; no code change.
- Optional `icon = '…'` overrides the auto-picked themed line icon.

Players buy from **settings > ANIM SHOP**, TRY previews on their own ped, and ASSIGN it to any podium type.

---

## Badges

Add a record to `Config.Badges`:
```lua
['my_badge'] = {
    label = 'Speed Demon', icon = 'hunter.png', color = '#FACC15',
    trigger = { type = 'kill_total', threshold = 250 },
},
```
Triggers: `kill_total`, `weapon_kill` (`category`), `kill_streak`, `headshot`, `playtime` (seconds), `kd` (`threshold`, `minKills`). Icons are files under `html/assets/badges/` (svg/png/webp/jpg/gif) or an emoji. It unlocks automatically on the next qualifying kill / tick. Players equip one active badge in **settings > PROFILE > ACTIVE BADGE** (a "None" tile clears it).

---

## Languages

Set `Config.Locale` to any built-in code (`en, tr, fr, de, es, pt, it, pl, nl, ru`). To add one:

1. Copy `locales/en.lua` to `locales/<code>.lua`, translate the values, set `Locales.<code>`.
2. Add `'locales/<code>.lua'` to the `shared_scripts` block in `fxmanifest.lua`.
3. `Config.Locale = '<code>'`.

Missing keys fall back to English automatically, so partial translations are safe.

---

## HUD auto-hide

The panel is a fullscreen in-world DUI framed by a scripted camera, so 2D HUDs would draw over it.

- `Config.HideHudWhenOpen = true` hides the **vanilla** HUD/radar while the panel is open.
- `Config.HudCompat` auto-detects running NUI huds (qb-hud, ps-hud, esx, ox …) and toggles them.
- For any custom hud, listen to the client event:
  ```lua
  AddEventHandler('ns-leaderboard:panelState', function(open) --[[ open = true/false ]] end)
  ```

Stock qb-hud exposes no external hide event — add a 4-line drop-in (see `COMMANDS.txt`) if you run it.

---

## API (exports)

> All `source` arguments accept a **server ID (number)** or an **identifier (string)**.

### Server
```lua
-- Profile
exports['ns-leaderboard']:GetProfile(source)                 -- full profile table (read-only)
-- Settings
exports['ns-leaderboard']:SetNameColor(source, '#EF4444')    -- nil = clear
exports['ns-leaderboard']:GetNameColor(source)
exports['ns-leaderboard']:SetDiscordName(source, 'user')     -- nil = clear
exports['ns-leaderboard']:GetDiscordName(source)
exports['ns-leaderboard']:SetAvatar(source, url)             -- URL/preset/badge path, '' = reset
exports['ns-leaderboard']:GetAvatar(source)
-- Badges
exports['ns-leaderboard']:GetUnlockedBadges(source)          -- { [key] = timestamp|true }
exports['ns-leaderboard']:GetActiveBadge(source)             -- key | nil
exports['ns-leaderboard']:UnlockBadge(source, key)
exports['ns-leaderboard']:LockBadge(source, key)
exports['ns-leaderboard']:SetActiveBadge(source, key)        -- nil = remove
-- Podium / ped
exports['ns-leaderboard']:GetPlatformLeaders(ptype)          -- top-N slot list
exports['ns-leaderboard']:GetPlayerLDIndex(source)           -- all of the player's rankings
exports['ns-leaderboard']:NotifySkinChanged(source)          -- re-spawn the ped instantly
exports['ns-leaderboard']:NotifyAnimChanged(source, ptype)   -- ptype nil → all types
```

### Client
```lua
-- After a client script changes its own character:
exports['ns-leaderboard']:NotifyLocalSkinChanged()
exports['ns-leaderboard']:NotifyLocalAnimChanged(ptype)
```

Typical use: one export call when leaving a clothing/anim menu → the podium ped updates in the world within ~1-2s (1s diff debounce). The skin also updates automatically via periodic + appearance-watch paths even without the export.

---

## Combat tracking (built-in stats)

Every profile stat comes from ns-leaderboard's own tracking — no external feed:

| Stat | Source |
|---|---|
| kills / deaths / K-D | cross-validated kill validator |
| headshots | head-bone hit from the damage event |
| playtime | session time |
| shots_fired | own clip ammo-delta (attacker-side) |
| hits / damage_dealt | damage event, **victim-reported** and clamped server-side |
| accuracy | `hits / shots_fired` |
| most kills with / deaths by | weapon-hash counters |

Hits, damage and headshots are read on the **victim's** client and credited to the attacker with per-report clamps + an attacker cap, so nobody can self-report or mass-inflate. By default they count **only against players** (PvP). `Config.Profile.Tracking.CountNpc = true` lets you test solo but makes NPC hits self-reported (a modified client could inflate them) — leave it off in production.

### Discord avatar (optional)

Players can pull their Discord avatar from settings > PROFILE. Off by default. The DISCORD button only appears when the server has a bot token **and** the player has a `discord:` identifier — both checked server-side, so the token is never exposed to clients.

```cfg
set ns_leaderboard_discord_token "your-bot-token"
```
> Use `set`, **never `setr`** — `setr` would replicate the token to every client. It's kept out of `config.lua` for the same reason (config is a `shared_script`). The token is re-read on use (no restart needed); if unset, the chain falls back to Steam → `Config.DefaultAvatar`.

---

## Architecture

| File | Role |
|---|---|
| `server/storage.lua` | `LeaderBoard[identifier]` cache + SQL CRUD + periodic save + column auto-migrate |
| `server/platforms.lua` | sort + `Platform[type]` cache + debounced diff push |
| `server/platform_store.lua` | `data/platforms.json` read/write, layout callback, save/refresh/delete |
| `server/kill_validator.lua` | cross-validation + self-kill guard + weapon-aware distance + rate limit |
| `server/profile.lua` | victim-reported combat tracking (clamped/capped) |
| `server/badges.lua` | badge unlock evaluator + active badge + exports |
| `server/economy.lua` | anim shop (buy / assign / list) |
| `server/points.lua` | POINT board (cash snapshot) |
| `server/avatars.lua` | Discord → Steam → default avatar chain |
| `client/platform.lua` | podium / sign / ped / crown spawn + surgical re-spawn |
| `client/podium_color.lua` + `server/podium_color.lua` | live podium/sign recolour (runtime texture swap) + KVP persistence + broadcast |
| `client/platform_draw.lua` | rank + name + badge above the ped via DrawText3D |
| `client/backdrop.lua` | in-world DUI surface for the panel |
| `client/object_spawn_tool.lua` + `client/gizmo.lua` | in-game placement tool + mouse gizmo |
| `client/appearance_watch.lua` | auto-detect outfit change → refresh ped |
| `client/hud_compat.lua` | third-party HUD auto-hide |

---

## License

MIT — NativeScripts-FiveM
