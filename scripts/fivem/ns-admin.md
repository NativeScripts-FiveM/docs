# ns-admin

A standalone FiveM **admin menu** — moderation, developer tools, a ticket system, and
**live player spectate over WebRTC** (a feature called *Watch Screen*), all in one in-game
panel. Open the menu, search the player list, act on any player, and watch their screen
peer-to-peer (ultra-low latency, media never touches the server) — all framework-optional.

**Standalone**: no framework dependency, no `screenshot-basic`, no external services. Drop it
in and grant your admins an ACE. Optional framework features (money / job / inventory) light up
automatically on ESX / QBCore / Qbox / vRP 1.x via the bundled `utils/` layer.

---

## Features

- **Live spectate (WebRTC)** — watch a player's screen P2P. Media flows peer-to-peer; the
  server only brokers signaling (SDP/ICE). Capture runs only while someone is watching.
- **Multi-window grid** — watch up to `Config.MaxCells` players at once, security-camera style,
  with a live stats overlay per cell (HP / armor / coords / weapon / speed / ping / FPS).
- **Admin panel** — `F6` (rebindable) opens a searchable player list with a per-player
  inspector (identifiers, framework job, money, warns, live entity info).
- **Moderation** — kick, ban (file-based, survives reconnect), warn (file-based), freeze, heal,
  revive, goto / bring. Offline ban manager (list / unban / add).
- **Ticket system** — players open a report with `/report` (a form: pick a category + describe the
  issue); admins work them in a dedicated **ticket dashboard** (open / claimed / resolved tabs,
  draggable, jump-to-id, claim / resolve / spectate / teleport, copy identifiers). Tickets persist
  to file, categories are config-editable, resolved ones auto-expire, and per-player rate-limit +
  open-cap stop flooding. Optional rich Discord embeds per ticket.
- **Developer tools** — noclip, free camera, Dev Laser (aim-to-inspect entity info + statebags,
  shown on a modern NUI card), clone / grab / spawn entities with an in-world placement editor,
  vehicle spawner (with vehicle-keys integration), become-ped, teleport, weapon picker, weather /
  time control. A hold-`Left Alt` **two-ring radial** gives quick access to tools + world actions.
- **ESP overlays** — native name tags with the game's health/armor bar above every player, and
  OneSync-aware map blips (vehicle icon when driving, dot on foot).
- **Roles & permissions** — tiered access (`admin` / `mod` by default) resolved from ACEs, with
  per-permission gating. Assign roles **live** from the panel (no restart) — grants persist.
- **Discord audit log** — moderation & management actions are posted to a Discord webhook as a
  categorized embed (emoji + colour per action, who/whom/details, with identity links). Cosmetic
  world changes (weather / time) are not logged. Set the webhook convar to enable it.

---

## How spectate works

```
TARGET player NUI                 SERVER (FxServer)            ADMIN (watcher) NUI
  WebGL magic-hook canvas           signaling broker             RTCPeerConnection
  → canvas.captureStream            (relays SDP/ICE only)        → <video>
  → RTCPeerConnection.addTrack ──────  events  ───────────────▶  live screen
```

The server only brokers signaling (SDP offers/answers + ICE candidates) between paired peers.
**Media flows peer-to-peer and never passes through the server or the Lua VM.** Each watcher of
the same player costs that player one extra video encode, so keep grid sizes sane.

---

## Install

### 1. Add the resource
Drop this folder into your server's resources (e.g. `resources/[ns]/ns-admin`), then add it to
`server.cfg`:

```cfg
ensure ns-admin
```

### 2. Give yourself (and your staff) access — **required**
The permission gate is **always on**: until you grant a role, the panel opens for **nobody — not
even you**. Access is **never** inherited from your framework's own admin perm — you grant it
explicitly. First, add these two lines once:

```cfg
add_ace group.admin ns.admin allow   # whoever is in group.admin → full admin
add_ace group.mod   ns.mod   allow   # whoever is in group.mod   → limited "mod"
```

These target a **group**, not a person — they say *"anyone in `group.admin` is an ns-admin admin."*
So now you just need each staff member to be **in** that group.

#### How to make a specific person admin/mod

**Method A — put them in the group** (most frameworks already do this for their admins):

```cfg
add_principal identifier.fivem:18927536 group.admin   # <- this person becomes admin
add_principal identifier.fivem:99999999 group.mod     # <- this person becomes mod
```

**Method B — grant a person directly** (no group needed):

```cfg
add_ace identifier.fivem:18927536 ns.admin allow      # only this person → admin
add_ace identifier.fivem:99999999 ns.mod   allow      # only this person → mod
```

**Finding someone's identifier:** with them connected, run `getplayeridentifiers <serverId>` in the
server console (or check txAdmin's players list). Use a stable one like `license:` or `fivem:`.

> **Already a framework admin?** On QBCore / ESX / Qbox your admins are usually already in
> `group.admin` (via an `add_principal …` line your framework added). So once the two `add_ace`
> lines above are present, those people are admins automatically — you don't re-enter anyone.

> **Prefer not to touch server.cfg for every staffer?** Add just your **owner/main admins** here,
> then let them assign everyone else **in-game** from the panel's Staff manager (no restart). You
> can also use a different group name, or reuse your framework's own check with the
> `NsAdminBridge.GetRole` hook in `bridge.lua`. See [Roles & permissions](#roles--permissions).

### 3. Discord logging — **optional**
Want admin actions / screenshots posted to Discord? Add the webhook(s) as **convars**. They live
in `server.cfg` (which is server-only), so the URL is never sent to players — **never** put a
webhook in `config.lua`:

```cfg
set ns_admin_log_webhook    "https://discord.com/api/webhooks/..."   # admin audit log
set ns_admin_webhook        "https://discord.com/api/webhooks/..."   # screenshots / clips
set ns_admin_ticket_webhook "https://discord.com/api/webhooks/..."   # tickets (new / claim / resolve)
```

Each webhook is an independent channel. `ns_admin_ticket_webhook` falls back to
`ns_admin_webhook` if you don't set it. Leave a webhook out to keep that channel off.

### 4. Restart
Restart the **server** (not just the resource) so the new `add_ace` / `set` lines load — then
press **F6** in-game.

---

> **TL;DR** — the minimum to get going is `ensure ns-admin` + `add_ace group.admin ns.admin allow`,
> then restart. Everything else is optional. No resource dependencies; framework features (money /
> job / inventory) auto-detect ESX / QBCore / Qbox / vRP. If you forget the ACE, the server console
> prints a reminder and the panel stays locked — by design, there is no "open to everyone" mode.

---

## Usage

| Action | Default |
|---|---|
| Open the admin panel | `F6` (rebindable in Settings ▸ Key Bindings, or `/watchpanel`) |
| Spectate one player by id | `/spectate <id>` |
| Stop spectating | `BACKSPACE`, or `/unwatch` |
| Radial quick menu (two rings) | hold `Left Alt` |
| Open a report ticket (any player) | `/report` (form), or `/report <message>` for a quick one |

Access is always gated by the roles in `Config.Roles` — a player needs a matching role ACE to
use anything; no role means no access.

---

## Configuration

Everything lives in `config.lua` — roles & permissions, which actions each tier may use, ban
durations, spawn catalogs, ESP, keybinds, and messages. A few important ones:

```lua
Config.Debug    = false   -- noisy console logging; keep false in production
Config.PanelKey = 'F6'    -- panel open key (rebindable)
Config.MaxCells = 9       -- max simultaneous spectate cells
```

> The permission gate is always on — there is no "allow everyone" switch. Define your tiers in
> `Config.Roles` and grant their ACEs in `server.cfg`.

> **Webhooks are server-only.** `config.lua` is a `shared_script`, so anything in it is downloaded
> by every client. **Never** put a Discord webhook there — set it as a server convar in `server.cfg`
> (`ns_admin_webhook`, `ns_admin_log_webhook`, `ns_admin_ticket_webhook`) instead, as shown in
> Install. Unset = Discord off.

### Reports / tickets

Everything about the ticket system lives in `Config.Reports`:

```lua
Config.Reports = {
    command           = 'report',   -- /report opens the form (or /report <message> for a quick one)
    cooldown          = 60,         -- seconds between tickets per player
    maxOpenPerPlayer  = 3,          -- max simultaneous open/claimed tickets per player (0 = off)
    keepResolvedHours = 72,         -- auto-delete resolved tickets this many hours after resolving (0 = keep)
    categories = {                  -- the form's category chips; first entry is the default
        'Player Report', 'Cheating / Exploit', 'Bug', 'Stuck / Unstuck', 'Question', 'Other',
    },
}
```

Edit / reorder `categories` freely — the player form and the admin dashboard use exactly this list.
Cooldown + open-cap are keyed by a stable identifier, so reconnecting can't reset them.

### Roles & permissions

Access is driven entirely by `Config.Roles`. It ships with two tiers (`admin`, `mod`) but you can
define **as many custom roles as you want**. Each role is:

```lua
{
    name  = 'mod',                         -- any label you choose
    aces  = { 'ns.mod', 'group.mod' },     -- holding ANY of these ACEs = this role
    perms = { 'watch', 'players', ... },   -- '*' for everything, or a list of permission keys
}
```

- A player's role is the **first** entry (top = highest priority) whose ACE they hold, so order
  your roles strongest-first.
- `perms = '*'` grants everything; a list grants only those keys. Anything not listed is denied.

**Add a custom role** — e.g. a "helper" who can only spectate, see players, and handle reports:

```lua
-- config.lua
Config.Roles = {
    { name = 'admin',  aces = { 'ns.admin',  'group.admin'  }, perms = '*' },
    { name = 'mod',    aces = { 'ns.mod',    'group.mod'    }, perms = { 'watch', 'players', 'reports', 'kick', 'warn', 'freeze' } },
    { name = 'helper', aces = { 'ns.helper', 'group.helper' }, perms = { 'watch', 'players', 'reports' } },
}
```

```cfg
# server.cfg — grant the ACE, then put your helpers in that group
add_ace group.helper ns.helper allow
add_principal identifier.license:XXXXXXXX group.helper
```

**Permission keys** (use these in `perms`):

| Key | Grants |
|---|---|
| `watch` | spectate players (live screen) |
| `players` | open the panel + player list + inspector |
| `reports` | claim / resolve player reports |
| `evidence` | capture screenshot/clip to Discord |
| `kick` `warn` `freeze` `heal` `revive` `goto` `bring` | the matching player actions |
| `ban` `banmanager` | ban a player / open the ban manager |
| `godmode` `invisible` `setModel` `giveWeapon` | dev tweaks on the target |
| `repairVehicle` `cleanVehicle` `deleteVehicle` | the target's vehicle |
| `money` `job` `inventory` `giveItem` | framework player management (view inventory / give an item) |
| `world` `announce` | weather/time / server announcement |
| `selftools` | the admin's **own** dev tools (noclip, free cam, Dev Laser, invisible, spawn, become-ped…) — also gates the radial wheel |
| `roles` | assign/remove roles live (Staff manager) — **keep admin-only**, it's privilege escalation |

**Assign roles live** — open the panel ▸ **Staff** (needs the `roles` permission), pick an online
player (or type an identifier for an offline one) and grant any defined role. It applies instantly
and persists across restarts/reconnects (stored in `staff.json`). A live grant uses the **higher**
of the grant and the player's `server.cfg` ACE role — so it can **promote** someone but can never
**demote** a server.cfg admin below their ACE role. (A granted admin therefore cannot strip or
downgrade the owner from the panel.)

**Removing access:**
- **Panel grants** (staff.json) — remove them in-game from the Staff manager (set the role to
  *none*). No restart.
- **server.cfg ACE admins** — can only be removed from `server.cfg`: delete that person's
  `add_principal …` (or `add_ace …`) line, or run `remove_principal` / `remove_ace` in the server
  console, then restart. The panel can't touch `server.cfg`, so ACE-based admins are tamper-proof
  in-game. Reserve ACE for owners / permanent staff; use panel grants for flexible or temporary ones.

**Reuse your framework's permissions instead** — implement `NsAdminBridge.GetRole(src)` in
`bridge.lua` to return `'admin'` / `'mod'` / your role name (or `nil`); it overrides ACE resolution
entirely, so you can map your existing staff system straight onto ns-admin's roles.

### Owner hooks (`bridge.lua`)

`bridge.lua` exposes optional `NsAdminBridge.*` hooks so you can wire ns-admin into your own
framework / scripts (custom notify, kick/ban, set job, vehicle keys, role resolution, etc.)
without editing the core. Each hook returns `true` to override the built-in behavior.

### NAT / TURN

Spectate uses a public Google STUN server by default, which covers the vast majority of players.
Players behind **symmetric NAT** can't connect P2P with STUN alone — to support them, host a TURN
relay (e.g. coturn) or use a TURN provider and add it to the `iceServers` list in
[`client/main.lua`](client/main.lua). Most servers never need this.

---

## License

MIT
