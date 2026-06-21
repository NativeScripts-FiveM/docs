# ns-poster

Designable, syncable **in-world posters** for FiveM. Players open a modern NUI
editor to compose a poster (text + images, presets, custom backgrounds,
categories), then place it as an interactive prop anywhere in the world. The
**actual design is rendered onto the prop face in the world** via DUI — not just
a generic prop. Nearby players inspect it (ox_target or a key prompt), **like**
it, **report** bad ones, and owners can **renew** them before they auto-expire.
Admins get a moderation queue with one-click teleport.

Framework-agnostic via **ns-utils** (ESX / QBCore / Qbox auto-detect, copied
into the resource — no shared dependency).

> **Discord:** https://discord.gg/UyyngemnF8 — support, bug reports, feedback.

---

## Highlights

- **Real in-world rendering** — the poster you design is drawn onto the prop in
  the world through a DUI runtime texture (per-poster, distance-streamed).
- **Modern NUI editor** — drag / resize / rotate text and image elements on a
  600×800 (3:4) canvas, undo/redo, fonts, image filters, presets.
- **Categories** — Notice / Wanted / Event / Missing Person / For Sale / Custom,
  each with its own blip colour and a filter in the `/posters` list.
- **Economy** — placing (and renewing) can cost money, with per-prop and
  per-category overrides and an optional refund on removal.
- **Permissions** — restrict who can place (by job), restrict categories to
  jobs, allow police-style takedowns, and gate placement to zones.
- **Image upload** — optional; players upload a local image which the server
  proxies to imgur (or a custom endpoint) and inserts the URL.
- **Likes / reports / TTL renew**, admin moderation queue.
- **Dual Discord webhooks** — a private admin audit log and a public member feed,
  both embedding a **rendered image of the actual poster** (uploaded straight to
  Discord — no external host needed).
- **MySQL or JSON** persistence.

---

## Features

### Designer (NUI)
- 9 built-in presets (modern Los Santos themes) — **Custom (blank)**,
  **Wanted (LSPD)**, **Club Night**, **For Sale**, **Magazine Cover**,
  **uwu Cafe**, **Now Hiring**, **Lost Pet**, **For Rent**.
- Drag / resize / rotate text and image elements on a 600×800 (3:4) canvas.
- 8 fonts shipped: Oswald, Bebas Neue, Anton (display), Inter (sans),
  IM Fell English, Playfair Display (serif), Special Elite (typewriter),
  Pacifico (script).
- 8 image filters: None, Sepia, Grunge, Noir, B&W, Vintage, Faded, Cold.
- Image sources: built-in asset library, whitelisted-domain URLs, and (optional)
  local upload.
- Custom backgrounds: library, https URL, or solid colour.
- Category selector + live cost preview on the Place button.
- Undo/redo via design history; save your own composition as a personal preset.

### In-world rendering (DUI)
- Each placed poster's design is drawn **flat onto the prop's face** as a
  world-space quad (`DrawSpritePoly`) sampling a per-poster DUI runtime texture —
  truly flat and angle-independent, readable on **both faces** of the board.
- Renders identically on a cold join and a warm restart (no model-reload or
  texture-streaming dependency).
- Distance-streamed: only posters within `Config.Dui.RenderDistance` render, and
  at most `Config.Dui.MaxConcurrent` at once (nearest first). The rest show the
  plain board.
- A custom flat board prop (`ns_poster`) is streamed with the resource and used
  by default; any flat-fronted prop works (see **Props & the design** below).
- Set `Config.Dui.Poly.Enabled = false` to fall back to a camera-facing billboard
  sprite, or `Config.Dui.Enabled = false` to show the bare prop only.

### Placement
- Uses [object_gizmo](https://github.com/DemiAutomatic/object_gizmo) for full 3D
  position + rotation control while placing.
- Ghost prop is local-only; the canonical networked record is spawned by the
  server on confirm — no duplicates.
- Cancel returns the player to the designer with the in-progress design intact.

### World + sync
- Distance-based streaming — props spawn at `Config.StreamDistance` (100m),
  despawn beyond `Config.UnstreamDistance` (120m), hard cap
  `Config.MaxActivePerClient` (30).
- Category-coloured map blip per poster — toggle with `Config.UseBlips`.
- `/posters` opens a list with category filter + GPS waypoint to any poster.

### Interaction
- `Config.Interaction = 'target'` → **ox_target / qb-target** hover menu.
- `Config.Interaction = 'key'` → proximity `[E]` prompt (DrawText3D).
- Target mode auto-falls back to the key prompt if no target resource is found.

### Economy
- `Config.Economy.Enabled` gates everything. Placing costs `PlaceCost`,
  renewing costs `RenewCost`, both via `Utils.RemoveMoney` (`cash` or `bank`).
- Per-prop (`PropCosts`) and per-category (`CategoryCosts`) overrides; optional
  `RefundOnRemove` when the owner removes their own poster.

### Permissions
- `PlaceJobs` — restrict who may place at all (empty = everyone).
- `CategoryJobs` — restrict specific categories to specific jobs.
- `TakedownJobs` — jobs that can remove anyone's poster (e.g. police); admins
  always can.
- `AllowedZones` / `BannedZones` — gate placement to areas.

### Social
- **Likes** — one per player per poster, rate-limited; owners optionally blocked
  from liking their own (`Config.AllowSelfLike`).
- **Reports** — players flag content with a reason; per-identifier cooldown plus
  a per-poster duplicate guard until an admin resolves it. Owners can be allowed
  to report their own poster with `Config.AllowSelfReport` (off by default).
- Every poster has a short **4-digit ID** shown to admins, searchable in the
  `/posters` list and referenced in reports and webhook logs.

### Expiration (TTL)
- Posters auto-expire after `Config.PosterTtl` (default 7 days; `0` = never).
- Owners renew from the inspect screen (`RenewExtension` from *now*).
- Server sweeps expired posters every `Config.ExpirySweepTick`.

### Admin moderation
- `/posterreports` opens the report queue (gated by ACE `Config.AdminAce` or the
  framework admin group). One-click **teleport**, dismiss / resolve / remove.
- Filter by Open / Resolved / Dismissed / All, **re-open** a report you closed
  by mistake, and **click any thumbnail to enlarge** the poster.
- Optional Discord webhook log on admin teleport.

### Discord webhooks (server-only)
Two independent channels under `Config.Discord` — set either URL to `''` to
disable it:

- **`AdminWebhook`** — full audit: place / renew / remove / report /
  admin-teleport. Each entry includes the owner (and, on reports, the reporter)
  with identifier, phone, **Discord mention** and **Steam profile link** when
  available, plus coordinates and the 4-digit poster ID.
- **`MemberWebhook`** — public feed: new poster / renewed / removed, showing only
  the content (category, author name, phone, poster ID) — no identifiers/coords.
- Both embed a **rendered PNG of the actual poster**, uploaded straight to the
  webhook as a file attachment via the bundled `server/discord.js` (Node
  `https` + `Buffer`) — **no imgur/external host required**. Set
  `Config.Discord.AttachPreview = false` for text-only embeds, or
  `UploadMode = 'host'` to push the image to `Config.Upload` (imgur/custom) instead.
- Per-event toggles: `LogPlace` / `LogRenew` / `LogRemove` / `LogReport` /
  `LogAdminTeleport` (admin) and `MemberOnPlace` / `MemberOnRenew` /
  `MemberOnRemove` (member).

> **Steam link** only appears when the server provides a `steam:` identifier —
> add `set steam_webApiKey "<key>"` to `server.cfg` (free key from
> steamcommunity.com/dev/apikey). **Discord mention** needs the player to have
> linked Discord to their FiveM account.

### Persistence
- **oxmysql** (default, `Config.UseMySQL = true`) or **JSON files**
  (`server/data/`). Tables auto-create on boot; `sql/install.sql` is provided for
  manual/production installs.

---

## Requirements

| Resource | Required | Purpose |
|---|---|---|
| [object_gizmo](https://github.com/DemiAutomatic/object_gizmo) | ✅ | 3D placement gizmo |
| `oxmysql` | ✅ (MySQL mode) | Persistence when `Config.UseMySQL = true` |
| `ox_target` / `qb-target` | optional | Only if `Config.Interaction = 'target'` |
| `ox_lib` | optional | Used automatically for notifications if present |
| Node.js + npm | optional | Only for rebuilding the NUI |

> **No framework is a hard dependency.** ESX / QBCore / Qbox are auto-detected at
> runtime by the bundled `utils/` (ns-utils). Standalone also boots, but money,
> jobs and identity features need a framework.

---

## Installation

1. Drop the `ns-poster` folder into your server's `resources/` directory.

2. Add to `server.cfg` **after** your framework + object_gizmo:
   ```
   ensure oxmysql
   ensure object_gizmo
   ensure ns-poster
   ```

3. (Optional) grant admin moderation access:
   ```
   add_ace group.admin ns-poster.admin allow
   ```

4. **MySQL mode (default)** — tables auto-create on first boot, or import
   [`sql/install.sql`](sql/install.sql) for an explicit/production install.

5. (Optional) populate `Config.AllowedImageDomains` to allow image URLs, and/or
   configure `Config.Upload` to allow local image upload.

6. Tune `Config.AllowedProps` + `Config.PropFaces` so the rendered design sits
   flush on each prop's front face (see **Props & the design quad** below).

---

## Commands & keys

| Action | How |
|---|---|
| Open designer | `/poster` |
| Open posters list (filter + GPS waypoint) | `/posters` |
| Open admin report queue | `/posterreports` (ACE/admin gated) |
| Inspect a poster | Hover with ox_target **or** walk up and press `E` |
| Placement gizmo | After pressing **Place**: left-click confirm, right-click cancel |

### Opening the designer: command and/or item

- `Config.Command` — the `/poster` command (toggle / rename it).
- `Config.Item` — open the designer by **using an inventory item**, and optionally
  consume one item per placement (`Config.Item.Consume`). Set
  `Config.Command.Enabled = false` for item-only access.
- Adding the item to your inventory (ox_inventory / qb-inventory / ESX) is a
  one-time step — see the [`install/`](install/) folder.

---

## Props & the design

By default the design is drawn as a **flat world-space quad on the prop face**.
A custom flat 3:4 board prop, `ns_poster`, ships in `stream/` and is the default
(`Config.DefaultProp`). Any flat-fronted prop works — add it to
`Config.AllowedProps` and give it a `Config.PropFaces` entry.

Size per prop in `Config.PropFaces` (metres, also sets the aspect ratio):

```lua
Config.PropFaces = {
    ['ns_poster']                    = { w = 0.350, h = 0.450, up = 0.0 },
    ['sum_prop_ac_qub3d_poster_01a'] = { w = 0.45,  h = 0.9,   up = 0.3 },
}
Config.PropFaceDefault = { w = 1.0, h = 1.35, up = 0.0 }
```

The quad itself is tuned in `Config.Dui.Poly`:

```lua
Config.Dui.Poly = {
    Enabled     = true,    -- false → camera-facing billboard sprite instead
    Forward     = 0.020,   -- metres in front of the prop face (avoids z-fighting)
    Up          = 0.185,   -- lift the quad centre to sit on the board
    DoubleSided = true,    -- readable on both faces of the board
    FlipSide    = false,   -- which physical side the design faces
    FlipU       = true,    -- mirror horizontally so the text reads correctly
}
```

- `PropFaces[*].w/h` — poster face size; `up` — extra height on top of
  `Config.Dui.HeightOffset`.
- The billboard fallback (`Poly.Enabled = false`) instead uses
  `Config.Dui.SpriteScale` for apparent size.

Shipped `Config.AllowedProps`: `ns_poster` (the custom flat board, default),
`sum_prop_ac_qub3d_poster_01a` (base-game wall poster) and `prop_news_disp_02a`
(newspaper stand). Props not listed use `Config.PropFaceDefault`.

---

## Configuration

All settings live in [`config.lua`](config.lua). Key blocks:

- **Storage** — `UseMySQL`, `AutoSaveInterval`.
- **Streaming + limits** — distances, ticks, per-player caps.
- **Placement** — `AllowedProps`, `PropFaces`, anti-teleport clamp.
- **Dui** — `Enabled`, `RenderDistance`, `MaxConcurrent`, texture resolution.
- **Categories** — keys, labels, blip sprite + colour.
- **Economy** — costs, account, overrides, refund.
- **Interaction** — `target` / `key`, distance, key control.
- **Permissions** — `PlaceJobs`, `CategoryJobs`, `TakedownJobs`, zones.
- **Image whitelist + Upload** — allowed hosts, imgur/custom upload proxy.
- **Expiration / renew**, **Social** (likes, reports, self-report).
- **Discord** — `AdminWebhook` / `MemberWebhook`, `AttachPreview`, `UploadMode`,
  per-event toggles, username/avatar, and per-event embed `Colors`.
- **Messages**.

### Image upload setup (optional)

```lua
Config.Upload = {
    Enabled  = true,
    Provider = 'imgur',          -- 'imgur' | 'custom'
    ImgurClientId = 'your-id',   -- api.imgur.com/oauth2/addclient (no callback)
    -- custom: CustomEndpoint accepts JSON { image = '<base64>' } → { url = '...' }
    MaxBytes = 3 * 1024 * 1024,
}
```

Upload runs **server-side** (base64 → JSON), so it avoids NUI CORS and FXServer
multipart issues. No secrets ship in the NUI.

---

## Building the NUI

Pre-built output ships in `html/`. Only rebuild if you change anything under
`ui/`.

```
cd ui
npm install
npm run build
```

The build emits to `../html/` and regenerates the library manifest from files in
`library/`. The DUI render page is the same bundle loaded at
`html/index.html#render`.

### Asset library

Drop your own art into `library/backgrounds/` and `library/decorations/`
(`.svg`/`.png`/`.jpg`/`.webp`/`.gif`). The picker entries are auto-generated by
`ui/scripts/gen-library-manifest.mjs` on every build.

---

## Security model

- Server is authoritative — every place / remove / save-preset / like / report /
  renew / resolve-report / upload goes through `shared/schema.lua` validation,
  payload-size cap, image-URL whitelist, permission/zone checks, economy charge,
  and (for remove/renew) ownership/takedown checks.
- Placement coordinates are clamped to within `Config.PlacementMaxDistance` of
  the requesting player (anti-teleport).
- React renders all user text via `{value}` (never `dangerouslySetInnerHTML`).
- Image URLs must be `https://` and match `Config.AllowedImageDomains`.
- Per-identifier rate limits on place / like / report / renew / upload.
- All ACE / job / ownership checks run server-side.

---

## Exports

All exports take a real player `source`. `source = 0` (console) is rejected so a
buggy/malicious peer resource can't bypass ownership.

```lua
-- Placement & lifecycle
exports['ns-poster']:placePoster(source, { design = {...}, world = { coords, heading, model } })
exports['ns-poster']:removePoster(source, posterId)
exports['ns-poster']:renewPoster(source, posterId)
exports['ns-poster']:getAllPosters()           -- read-only snapshot

-- Social
exports['ns-poster']:likePoster(source, posterId)
exports['ns-poster']:reportPoster(source, posterId, reason)

-- Moderation (require admin)
exports['ns-poster']:listReports(source, filterStatus)         -- 'open' | 'resolved' | 'dismissed' | nil
exports['ns-poster']:resolveReport(source, reportId, action)   -- 'dismiss' | 'resolve' | 'remove' | 'reopen'
```
