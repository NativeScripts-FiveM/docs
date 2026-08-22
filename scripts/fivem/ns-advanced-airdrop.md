# ns-advanced-airdrop

Synchronized **airdrops** for FiveM. A supply plane flies in, releases a crate under
parachutes, and the whole server races to it. Admins get a full map panel to schedule,
place and inspect drops.

**Framework-optional.** ESX, QBCore, Qbox and plain standalone servers are detected at
runtime through the bundled `utils/` layer — there is nothing to install alongside it.

---

## Features

- **Plane + parachute delivery** — a cargo plane crosses the sky, drops the crate, and it
  descends under two chutes before settling on the ground. The scene is fully local: no
  entity is networked, yet every player sees the same plane on the same path at the same
  moment, at zero network cost.
- **In-world crate screen** — a panel on the crate itself shows a padlock icon and the
  live countdown, readable up close. Falls back to floating 3D text automatically, and
  admins can pick either mode per drop.
- **Map blips** — the supply plane appears on everyone's map as it approaches, from
  anywhere on the map; the crate blip opens the moment the plane releases it.
- **Two loot modes** — *Direct* (first player to open takes everything) or *Stash*
  (a shared container everyone loots from, on inventories that support stashes).
- **Automatic cycle** — drops on a timer, with a minimum-player gate so nothing lands on
  an empty server. Admin drops always ignore that gate.
- **No two drops on one spot** — the spawn point is still drawn at random, but points an
  active drop is already sitting on are excluded, so concurrent drops never stack. Once
  every configured point is busy the next drop lands beside one instead of inside it.
- **Admin panel** — a full-screen map with live drop markers, a Cycle tab (start/stop,
  interval, minimum players, instant drops), an Active tab, a Custom Drop builder (click
  the map to place it, pick items and amounts, crate, effect, announcement, countdown
  length), and a History tab with who claimed what.
- **Edit a live drop** — the contents of a crate already in the world can be changed from
  the Active tab: add items, remove them, change amounts. Useful when a drop went out with
  the wrong loot and you would rather fix it than delete it and start again.
- **Visual pickers** — crates and particle effects are chosen from in-game image cards
  that ship with the resource, not from a dropdown of model names.
- **Effects** — ten verified particle presets (flare, fire, lightning, money rain, smoke,
  confetti and more), selectable per drop. A live in-world preview shows one before you
  commit to it.
- **Legendary drops** — a separate effect, blip and announcement for rare drops.
- **Discord logging** — optional webhook with a detailed embed per drop, claim and
  expiry: who sent it, the crate, effect, contents, location, and a clickable mention
  plus Steam profile link for the player involved.
- **9 languages** — English, German, Spanish, French, Italian, Dutch, Polish,
  Portuguese, Turkish. Players pick their own; the choice is stored locally.

---

## Requirements

Nothing is mandatory.

| Optional | What it adds |
|---|---|
| ESX / QBCore / Qbox | Item handout through the framework, admin group fallback |
| An inventory resource | Better item handling; **required for loot mode 2 (stash)** |

The inventory layer already knows ox_inventory, origen, tgiann, core, qs, codem, qb, ps,
lj, esx_inventoryhud and gfx inventories, and falls back to the framework core when none
of them is running.

---

## Installation

1. Drop the `ns-advanced-airdrop` folder into your resources.
2. Add it to your server config:

```cfg
ensure ns-advanced-airdrop
```

3. Give your admins the permission:

```cfg
add_ace group.admin ns.airdrop.admin allow
```

4. Open `config.lua` and **check the loot table against your inventory** (see below).

---

## Admin permission

Access is granted by the ACE object `ns.airdrop.admin`. If the ACE check fails, the
resource falls back to the framework group (`admin` / `superadmin` by default,
configurable in `Config.Admin.frameworkGroups`).

---

## Commands

| Command | Who | What |
|---|---|---|
| `/airdrop` | admin | Opens the management panel |
| `/airdrop force` | admin | Drops one immediately |
| `/airdrop skip` | admin | Removes every active drop |
| `/airdrop status` | admin | Prints active drops to chat |
| `/airdroplang` | everyone | Language picker |
| `/airdroplang <code>` | everyone | Sets the language directly (`en`, `de`, `tr`, …) |
| `/airdropscreen` | admin | Aligns the crate screen live — needed only if you add your own crate prop |

There is deliberately **no key binding**: an admin tool should not claim a key on every
client. `Config.Admin.command` registers an extra command name if you want one.

Two more admin-gated dev tools exist and are safe to ignore: `/airdropfx` previews a raw
particle by dict and name, `/airdropprop` previews a prop and prints its dimensions.

---

## Configuration

Everything lives in `config.lua`; the settings most servers change are at the top and the
developer options are grouped at the bottom. A few worth knowing about:

### Loot table

`Config.LootTable` drives both automatic drops (weighted random) and the admin item
picker (which the server also uses as a whitelist).

> **Item names must match your inventory.** The shipped list uses default QBCore names.
> On ESX, ox_inventory or a customised setup, rename them — an item that does not exist
> cannot be given, and the player sees a misleading "inventory full" message.
> You do not have to check by hand: every name is compared against your inventory on
> startup and mismatches are printed to the console.

### Editing a drop that is already out

The Active tab has an **Edit items** button on every drop. It opens the same item picker
the Custom Drop builder uses, pre-filled with what is currently in that crate.

The server re-checks the submitted list against `Config.LootTable` and the
`Config.CustomDropLimits`, exactly as it does for a custom drop — the panel is a client
and is not trusted.

- **Loot mode 1** — the contents are read when someone claims, so the change applies to
  whoever gets there. A drop that has already been claimed can no longer be edited.
- **Loot mode 2** — the items live in a stash, so the stash is emptied and refilled.
  Anything players had already taken out is theirs and is not clawed back.

Every edit is written to the console and, if a webhook is configured, posted to Discord
with the before and after contents.

### Minimum players

`Config.Cycle.MinPlayers` blocks **automatic** drops below a player count. The cycle keeps
running and retries next interval. Admin drops are never blocked. `0` disables the gate.

Admins can change it live from the Cycle tab, the same way the interval is changed. Like
the interval, a panel change lasts until the resource restarts — it is not written back to
`config.lua`. It cannot be set above your `sv_maxclients`, since such a limit could never
be met and the cycle would go quietly dead.

### Spawn points

A drop picks a random entry from `Config.Locations`, but only from the ones no active drop
is within 100 m of. A second drop therefore never lands on the first one's coordinate. With
nothing active the whole list is in play, so single drops are as varied as ever.

When every entry is occupied the next drop is placed 110–160 m from a random point rather
than being cancelled — far enough out that it still counts as a separate location.

Keep your own entries at least ~150 m apart. Points closer than that will not block each
other, but they will look like one drop location to players.

### Scene speed

How long the flypast and the descent take is set with two speeds, not with
durations — so you can picture the result instead of timing a drop by hand.

| Setting | Unit | Default | What it does |
|---|---|---|---|
| `Config.Scene.planeSpeedKmh` | km/h | `200` | Speed of the supply plane. 120 is a slow flypast, 300 is a real cruise. |
| `Config.Scene.fallSpeedMs` | m/s | `8` | How fast the crate descends. A real cargo parachute lands at 5–7 m/s; much above 12 and it reads as a dropped rock rather than a parachute. |
| `Config.Scene.altitude` | m | `220` | Release height. Raising it makes the fall longer without changing how fast the crate moves. |

The seconds the rest of the resource works in (`approachSec`, `fallSec`) are
worked out from these on startup — do not edit them directly, they are
overwritten. The server console prints the result when the resource starts:

```
[ns-advanced-airdrop] scene: plane 200 km/h -> 16 s approach, crate 8.0 m/s from 220 m -> 28 s fall (44 s in total).
```

The open countdown (`Config.Cycle.OpenDelayMs`) starts when the crate touches
the ground, so slowing the scene down does not eat into it.

### Crate screen

A drop shows its countdown on a panel attached to the crate. Admins can switch an
individual drop to floating 3D text in the Custom Drop tab; `Config.Screen.enabled = false`
turns the panel off server-wide and every drop falls back to floating text.

What you can tune:

| Setting | What it does |
|---|---|
| `enabled` | Turn the crate panel on or off entirely |
| `distance` | Draw range in metres. It redraws every frame, so keep it short |
| `lockedColor` / `openColor` | Panel colour before and after the crate opens, `{ r, g, b, a }` |
| `iconLocked` / `iconOpen` | The icon shown before and after the crate opens. Any emoji, or short text |
| `iconSize` / `iconAspect` | Icon size, and its stretch if it looks wrong on your crate |
| `countdownScale` | Raise this if the timer is hard to read |

The icon is drawn by the browser layer rather than the game's fonts, so it is not limited
to characters the game can render. Paste any emoji straight between the quotes — 💰 ⭐ 🎁
all work, including multi-part ones like ☠️ or 👍🏽, and so does a short word like `'LOOT'`
(text longer than one character is scaled down to fit).

> Save `config.lua` as UTF-8. Every code editor does by default and Windows Notepad asks;
> if the encoding is wrong the icon falls back to a padlock rather than showing garbage.

The render-target internals — which prop carries the surface, and where the panel sits on
it — are not in the config. They are a matched set that only works for one specific prop,
so they live in `client/screen.lua`. Per-crate alignment is done in game with
`/airdropscreen`, which writes into `Config.Prop.options`.

> Only one crate can carry the screen at a time. That is an engine limit, not a setting:
> a named render target exists once per model, so the screen goes to the crate nearest
> the player and the others fall back to floating text automatically.
>
> If another resource already owns the render target, ns-advanced-airdrop stands down and
> logs a line instead of hijacking its screen.

### Adding your own crate

Add an entry to `Config.Prop.options` with an `id`, `label` and `model`. The crate's height
is measured at runtime, so it will already work — but the screen may not sit flush on an
unusual shape. Stand next to a landed drop, run `/airdropscreen`, nudge it with the on-screen
keys, press ENTER, and paste the printed `screen = { … }` block into your entry.

### Discord webhook

**Preferred:** put the URL in `server.cfg`, so it stays out of the resource folder and out
of any backup or zip you share.

```cfg
set ns_advanced_airdrop_webhook "https://discord.com/api/webhooks/..."
```

Otherwise fill in `ConfigServer.Webhook.url` in `config_server.lua`. The convar wins when
both are set. Which events are sent, the bot name and the embed colours are configured in
the same file.

> Never put the URL in `config.lua`. That file is a `shared_script`, so it is sent to every
> client — a webhook placed there is readable, and spammable, by any player.

---

## Custom inventories

If your inventory is not one of the supported ones, register an adapter from your own
resource:

```lua
exports['ns-advanced-airdrop']:RegisterInventoryAdapter('my_inventory', {
    add = function(src, item, count, metadata, slot)
        -- hand the item to the player, return true on success
    end,
})
```

---

## Languages

Nine are included: English, German, Spanish, French, Italian, Dutch, Polish, Portuguese
and Turkish. Each player picks their own from the panel dropdown or with `/airdroplang`,
and the choice is remembered on their machine. `Config.DefaultLang` sets the starting one.

**Editing what players read** — `locales/locales.lua` is left unencrypted on purpose.
Change any wording in there and restart; missing keys fall back to English, so a partial
edit is safe.

That file covers everything a player sees: notifications, the text above a crate, blip
names and command output. The admin panel has its own dictionary compiled into the UI
bundle, so its labels cannot be edited without the UI source — but the panel is an admin
tool, and all nine languages are already translated in full.

---

## Notes

- The plane, pilot, parachutes and crate are **local** entities. Nothing is networked, so
  the scene costs no server bandwidth and cannot desync between players — every client
  derives it from the same two values sent by the server.
- Loot is decided and validated entirely on the server. Distance, on-foot state, drop
  status and item whitelist are all re-checked there; the client is never trusted.
- Drop history is kept in memory and resets when the resource restarts.

---

## Documentation

Full docs: <https://fivem.nativescripts.com/docs/>
