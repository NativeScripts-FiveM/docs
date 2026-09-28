# ns-inventory

A PvP inventory for FiveM. The server decides every move: the NUI only asks, and nothing a player's client says can create, copy or keep an item. On top of that it has the things PvP servers otherwise build around their inventory — a protected pocket, loadouts, fast trade, death lootbags, a combat lock, an admin panel with live settings, Discord logs and 8 languages.

Framework: **QBCore**. ns-inventory replaces qb-inventory and answers qb-inventory's exports itself, so qb scripts that give, take or check items keep working.

## Features

**For players**
- **Inventory + Protected.** The main inventory is lost on death (see [Death](#death-lootbags-and-the-combat-lock)); Protected is always kept.
- **Hotbar** with 7 slots (keys 1–7). Hover an item and press a number to bind it.
- **Loadouts.** Save the kit you carry, take it back from your stash in one click, and share it as a code (`K7Q-4ZP`). A code copies the list, never the items.
- **Fast trade.** Give to a nearby player or any server ID from one dialog, with a trades history.
- **Item count limit.** Right-click an item to set the most you want of it in the inventory. Taking from a stash or Protected then stops there, so nobody pulls too many weapons by accident. Lootbags, gives and admin gives are not limited.
- **Weapons.** The inventory is the only source of weapons. Attachments, tints and finishes come from a right-click menu and are remembered per weapon type. Default attachments and a launcher-spam cooldown are configurable.
- **Ammo** runs in one of two modes: fixed ammo (nothing to manage) or tracked ammo (rounds belong to the weapon, reloading uses ammo items).
- **Personal settings:** accent color, background, card size, fonts and styles, tier colors. They are saved per player.

**Death, lootbags and the combat lock**
- **On death** the main inventory drops into a lootbag at the death spot (or is cleared, or kept — your choice).
- **Combat lock.** Weapon fights lock items out of the main inventory for a few seconds, so nobody hides loot mid-fight. Leaving the server while locked or down counts as a death.

**Stashes**
- **Personal stashes** at any number of locations, each character with its own content. They open with a 3D label or a guard NPC.
- **Shared stashes** for jobs, gangs, identifiers or ACE permissions.
- **Temporary stashes** that other scripts create (crates, drops, events).

**Admin panel (`/invadmin`)**
- **Players:** open anyone's inventory, online or offline, and move, add or remove items. You can also give to many players at once, wipe a player, or compare two players.
- **Logs:** every trade, give, death, loot, delete and admin action, searchable. Security flags are highlighted.
- **Economy:** how many of each item exist on the server and how that changed over 24 h / 7 days, and who holds them.
- **Staff:** one superadmin adds admins and moderators and switches single permissions per person.
- **Live settings** (Items, Weapons & loadout, Stashes, Limits): change weights, tiers, limits and stash locations without a restart.

**For the server owner**
- **Discord logs** with separate channels for admin actions and security.
- **Dupe watch:** a player gaining an unusual amount of an item from others is flagged, with where it came from.
- **Migration** from qb / ps / lj / qs / codem-inventory, ox_inventory and gfx-inventory.
- **Season wipe** with a full backup first.
- **8 languages:** English, Turkish, German, French, Spanish, Portuguese (Brazil), Polish, Italian.

## Requirements

- **QBCore** (`qb-core`).
- **oxmysql**.
- **OneSync.** Deaths and the combat lock are read on the server.
- **qb-inventory and qb-weapons must not run.** ns-inventory takes their place.

No build step and no CDN: the interface is already built and its fonts are included.

## Installation

1. Stop and remove **qb-inventory** and **qb-weapons**: delete or comment out their `ensure` lines.
2. Put the `ns-inventory` folder in your `resources`.
3. In `server.cfg`, after oxmysql and qb-core:
   ```cfg
   ensure ns-inventory
   ```
4. Give yourself the superadmin role (one account: the server owner):
   ```cfg
   add_ace identifier.license:YOUR_LICENSE ns-inventory.superadmin allow
   ```
   Staff can then be added from the panel's **Staff** screen, or by group:
   ```cfg
   add_ace group.admin ns-inventory.admin allow
   add_ace group.mod   ns-inventory.moderator allow
   ```
5. *(optional)* Set up [Discord logs](#discord-logs) and the [language](#languages).
6. Start the server. The database tables are created on their own; there is nothing to import.

Items your players already had in another inventory come over by themselves the first time each character loads. See [Moving from another inventory](#moving-from-another-inventory).

## Controls

| Key | Action |
|---|---|
| `TAB` | Open / close the inventory (players can rebind it: Settings → Key Bindings → FiveM) |
| `1`–`7` | Use the hotbar slot; with the mouse over an item, bind it to that slot |
| Drag | Move a stack · `SHIFT` + drag moves one |
| Click | Move one to Protected (with a stash open: to the stash and back) · `SHIFT` + click a set amount · `CTRL` + click all |
| Right click | Actions: use / equip, attachments, move, fast trade, favorite, prioritize, item count limit, delete |
| Middle click | Fast trade |
| `E` | Open a stash or loot a bag when standing at it |
| `R` | Reload from the ammo you carry (tracked ammo only) |

## Admin panel

Open it with `/invadmin`. What each role can do:

| Role | How to get it | Can |
|---|---|---|
| **Superadmin** | `ns-inventory.superadmin` ACE — one account | Everything, and the only one who adds, changes or removes staff |
| **Admin** | Staff screen, or the `ns-inventory.admin` ACE | Logs, move / remove items, give items, wipes, live settings |
| **Moderator** | Staff screen, or the `ns-inventory.moderator` ACE | Logs, move / remove items. Gives only with `Config.Admin.ModeratorsGive` or a switch on the Staff screen |

> Being in `group.admin` alone is **not** enough — a group is not an ACE. The `add_ace` line above is required.

**Live settings.** Most of `config.lua` can also be changed in the panel (Items, Weapons & loadout, Stashes, Limits). A value saved there wins over `config.lua`. If you change `config.lua` and nothing happens, check the panel: fields that differ from the config are marked, and **Reset** goes back to `config.lua`.

Every panel action is written to the Logs screen and, if set up, to Discord.

## Commands

### In game

| Command | Who | What it does |
|---|---|---|
| `/invadmin` | staff | Opens the admin panel |
| `/giveitem <id> [container] <item> [count]` | staff with *Give items* | Gives an item |
| `/removeitem <id> [container] <item> [count\|all]` | staff with *Move and remove items* | Removes an item |

- `<id>`: a player's server ID, or `me` for yourself.
- `[container]`: `inventory` (default), `protected` or `stash`. `stash` is that character's personal stash, and only with `stash` can `<id>` also be a citizen ID, for a character who is offline.
- `[count]`: how many, 1 if left out. `/removeitem` also takes `all`.
- Gives ignore weight limits; a remove takes all of it or nothing. Every use is logged.

```text
/giveitem 12 weapon_pistol
/giveitem me stash bandage 10
/removeitem 12 protected weapon_pistol all
```

Both item commands also work from the server console (without the `/`).

### Server console

Type these in the server console (txAdmin's Live Console works too). The superadmin can also use them in game.

| Command | What it does | When to use it |
|---|---|---|
| `invsave` | Saves every online player and every changed stash right now | Before a restart. Any staff member can use it in game too |
| `invdiscord` | Prints the Discord setup it found and posts one test message per log category | After setting the webhooks. Admins can use it in game too |
| `invmigrate` | **Report only** — what would move from your old inventory, what is skipped and why. Changes nothing | Before `invmigrate run` |
| `invmigrate run` | Moves every character and old stash from your old inventory at once | Once, after installing — **server empty** |
| `invseason` | **Report only** — what a season wipe would empty and what it keeps. Changes nothing | Before `invseason run` |
| `invseason run <name>` | The season wipe. `<name>` labels the backup, e.g. `invseason run season2` | **Server empty** |

> **Run `invmigrate run` and `invseason run` with nobody on the server.** They read and write every character in the database in one go, which on a big server can freeze it for a few seconds. Kick everyone or close the server to players, run the command, wait for the result in the console, then open again. The reports read everything too, so they are best run on an empty server as well.

Every command name can be changed in `config.lua` if another script already uses it: `Config.Admin.Command`, `Config.Admin.ItemCommands`, `Config.Save.Command`, `Config.Discord.TestCommand`, `Config.Migration.Command` and `Config.SeasonWipe.Command`.

## Configuration

Everything is in `config.lua`, with a comment on every value. The main parts:

| Section | What it sets |
|---|---|
| `Config.OpenKey` | The inventory key |
| `Config.Containers`, `Config.Weights` | How much the inventory and Protected hold (grams); weights on or off |
| `Config.Death`, `Config.Lootbag` | What death does: `lootbag`, `clear` or `keep`; the bag's lifetime, model, marker |
| `Config.CombatLock` | Combat lock: length, what starts it, death on quitting while locked |
| `Config.Give` | Fast trade: distance, same routing bucket, cooldown |
| `Config.Stash` | Personal stash locations, shared stashes and who may open them, NPC or 3D text |
| `Config.Weapons`, `Config.DefaultComponents`, `Config.Customization` | Weapon rules, default attachments, the attachments menu |
| `Config.Ammo` | Fixed or tracked ammo, and which item reloads which ammo type |
| `Config.ItemWeights`, `Config.ItemTiers` | Weights and tiers (card colors) over your framework's items |
| `Config.Loadouts` | How many loadouts a player keeps; share codes |
| `Config.OpenLock` | Keep the inventory closed while dead |
| `Config.Discord`, `Config.Logs` | Discord logs and how long the audit log is kept |
| `Config.DupeWatch` | Dupe watch limits |
| `Config.SeasonWipe`, `Config.Migration` | Season wipe and migration |
| `Config.Admin` | Panel command, item commands, moderator gives, give notifications |
| `Config.UI`, `Config.Sounds` | Default accent color and fonts; sounds |

**Weapons from addon packs.** With `Config.Weapons.GameWeaponsOnly = true` (default), only GTA's own weapons are items. Set it to `false` to use addon weapon packs.

**Other weapon scripts.** With `Config.Weapons.StripUnowned = true` (default), a weapon the inventory did not give is taken away and logged. Turn it off, or add the weapon to `Config.Weapons.Allowed`, if another script (paintball, an event) hands out weapons.

## Death, lootbags and the combat lock

- **Death** (`Config.Death.Mode`)
  - `lootbag`: the main inventory drops into a bag at the death spot. Anyone in the same routing bucket can loot it: `E` takes everything that fits, and the inventory key opens it beside the inventory.
  - `clear`: the main inventory is emptied.
  - `keep`: nothing is lost.

  **When** (`Config.Death.DropOn`): `'down'` (default) — the moment the player goes down; a qb last stand counts, and a revive does not bring the items back. `'dead'` — only once the framework says dead (the last stand ran out, or killed again while down), so a player revived in time loses nothing.

  Protected is always kept. Deaths are confirmed by the server itself, so a client cannot hide one.
- **Combat lock** (`Config.CombatLock`): hitting or being hit by a weapon locks both players for a few seconds.
  - Items can still come **into** the main inventory, but nothing can leave it: no Protected, no stash, no give.
  - Leaving the server while locked, or while down, applies the death mode.
- **Open lock** (`Config.OpenLock`): while dead or in last stand the inventory stays closed, and it opens again once revived. Other scripts can lock it too (handcuffs, a minigame); see [For developers](#for-developers).

## Stashes

- **Personal stash:** every character has one. Set its locations in `Config.Stash.Personal.Locations` or on the panel's **Stashes** screen.
- **Shared stashes:** add them in `Config.Stash.Shared`, with `access` by job and grade, gang, identifiers or an ACE. With no rules, only admins can open it.
- **How players open them** (`Config.Stash.Interact`)
  - `E` inside the radius opens the inventory with the stash beside it.
  - A location shows either a 3D text or a guard NPC with a help text (`Opener = 'text' | 'npc'`), set per location.
- **Other scripts:** qb scripts that call `exports['qb-inventory']:OpenInventory(source, 'stash-id')` open a stash here. Your own scripts can create temporary stashes (see below).

## Discord logs

The webhook URLs go in **`server.cfg`**, never in `config.lua`, because that file is sent to every player:

```cfg
set ns_inventory:webhook          "https://discord.com/api/webhooks/..."   # default channel
set ns_inventory:webhook:admin    "https://discord.com/api/webhooks/..."   # admin actions + live settings
set ns_inventory:webhook:security "https://discord.com/api/webhooks/..."   # security flags (can ping a role)
```

A category without its own webhook uses the default one. `Config.Discord.Categories` turns categories on and off: security, admin, settings, trade, stash, death, lootbag, delete, loadout. It also sets each category's channel and color, and whether it pings `MentionRole`.

Run `invdiscord` in the console to check the setup: it prints what it found and posts one test message per category.

## Languages

Set `Config.Locale` to one of: `en`, `tr`, `de`, `fr`, `es`, `pt`, `pl`, `it`. Everything follows it: the inventory, the admin panel, notifications, logs, Discord and commands.

- **Machine-translated:** de, fr, es, pt, pl and it. Corrections are welcome.
- **Change a text:** edit `locales/<code>.lua`. A text missing there shows in English.
- **Add a language:** copy `locales/en.lua` to `locales/<code>.lua` and change `'en'` in its `Register` line. Translate the texts and keep every `%s` / `%d` in the same order and every `{token}`.

## Moving from another inventory

Supported: **qb-inventory, ps-inventory, lj-inventory, qs-inventory, codem-inventory, ox_inventory, gfx-inventory**. Their tables are only read, never changed.

- **Automatically:** the first time a character loads here, its old items come over. A gfx-inventory row also brings its protected items and its stash.
- **All at once**, offline characters and old stashes too, from the server console — with **nobody on the server**:
  - `invmigrate` shows a report of what would move, what is skipped and why. It changes nothing.
  - `invmigrate run` moves it. Every old row moves only once, so running it again adds nothing.
- **Old stashes** go to their owner's personal stash, or to the shared stash with the same id. Trunks and gloveboxes are not moved.
- **Items your server does not define** are left out and listed: add them first, then run it.
- **Weapons** keep their rounds only.

## Season wipe

A fresh start for everyone: every inventory, Protected, personal and shared stash and lootbag is emptied, except the items in `Config.SeasonWipe.Keep`.

Do it with **nobody on the server**: kick everyone or close it to players first.

- `invseason` shows a report of what would be wiped and what is kept. It changes nothing.
- `invseason run season2` wipes. Every row is first copied to `ns_inventory_wipe_backup` under that name.
- `Config.SeasonWipe.Date = '2026-11-01 18:00'` wipes once at that time, or at the next start if the server was off then. Pick a time the server is closed, e.g. your scheduled restart.
- `Config.SeasonWipe.Stashes = false` keeps stashes and wipes only what players carry.

## Dupe watch

Items a player gets from **others** are counted per item: gives, other stashes, lootbags, and other scripts' AddItem. More than `Config.DupeWatch.Limits` within `Window` raises a flagged Security log and a Discord message saying how much and from where.

- Moves between a player's own containers and admin gives never count.
- A server-wide check (`ServerHour`) catches a broken shop or job script feeding everyone.
- Nothing is blocked: it only tells you where to look.

## Item pictures

Put pictures in `images/items/`, named after the item (`phone.webp`) or after your framework's `image` field (`phone.png`). WebP is smallest; PNG, JPG and GIF work too. Weapons already have pictures. Items without one show a placeholder.

## For developers

**Server exports**

```lua
-- Players
exports['ns-inventory']:getInventoryItems(source)   -- main inventory
exports['ns-inventory']:getProtectedItems(source)   -- Protected
exports['ns-inventory']:getAllItems(source)         -- both
--   → { { name, label, count, metadata, weight, container } } or nil while the player loads
exports['ns-inventory']:takeInventory(source)       -- empties the main inventory, returns the items

-- Opening the inventory
exports['ns-inventory']:setInventoryBlocked(source, true, 'cuffed')   -- each script lifts only its own reason
exports['ns-inventory']:canOpenInventory(source)                      -- → canOpen, { reasons }

-- Temporary stashes (memory only): crates, drops, events
exports['ns-inventory']:createTempStash(id, {
    label = 'Supply crate', items = { { name = 'bandage', count = 5 } },
    coords = vector3(0.0, 0.0, 0.0), radius = 2.0, openNearby = true,
    removeWhenEmpty = true, lifetime = 300, takeOnly = true,
})
exports['ns-inventory']:openStash(source, id)
exports['ns-inventory']:getStashItems(id)
exports['ns-inventory']:addStashItem(id, name, count, metadata)
exports['ns-inventory']:removeStashItem(id, name, count)
exports['ns-inventory']:removeStash(id)
```

**Client exports:** `setInventoryBlocked(blocked, reason)` is a local lock for UI scripts (a phone, a progress bar), not a security check. The others are `canOpenInventory()` and `isInventoryOpen()`.

**Server events** (`AddEventHandler`, only the server can trigger them)

| Event | Arguments |
|---|---|
| `ns-inventory:server:onDowned` | `source`: dead or in last stand |
| `ns-inventory:server:onRevived` | `source`: up again |
| `ns-inventory:server:onDeath` | `source, { reason, coords, bucket, items, lootbag }`. `reason` is `died`, `down_quit` or `combat_quit` |
| `ns-inventory:server:stashRemoved` | `id, reason`. `reason` is `removed`, `empty` or `expired` |

**qb-inventory compatibility**
- **Works:** `exports['qb-inventory']` item functions for players: `AddItem`, `RemoveItem`, `HasItem`, `GetItemCount`, `GetItemByName`, `GetItemsByName`, `GetItemBySlot`, `CanAddItem`, `GetFreeWeight`, `ClearInventory`, `OpenInventory` (own inventory and stashes), `CloseInventory`. The `Player.Functions` item methods and `QBCore.Functions.CreateUseableItem` items work too.
- **Not supported yet:** shops (`OpenShop` / `CreateShop`), ground drops, trunks and gloveboxes, other players' inventories, and `SetInventory` / `SetItemData`. These answer "not supported" and are logged once per calling resource.

## Troubleshooting

| Problem | Fix |
|---|---|
| The admin panel says no permission | Add the ACE (see [Installation](#installation)); being in `group.admin` is not enough |
| A `config.lua` change does nothing | It was saved in the panel's live settings, and the panel wins. Reset it there |
| Addon weapons are missing | `Config.Weapons.GameWeaponsOnly = false` |
| Weapons from another script disappear | `Config.Weapons.StripUnowned = false`, or add them to `Config.Weapons.Allowed` |
| The inventory will not open | Dead or in last stand (`Config.OpenLock`), or another script locked it |
| Nothing is saved / "no database" in the console | Start oxmysql before ns-inventory |
| A qb shop does not open | Shops are not supported yet (see above) |

For extra console output and the test commands `/giverandom` and `/testlootbag`, set `Config.Debug = true`. Keep it off on a live server.
