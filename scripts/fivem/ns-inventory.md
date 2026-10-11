# ns-inventory

A PvP inventory for FiveM. The server decides every move: the NUI only asks, and nothing a player's client says can create, copy or keep an item. On top of that it has the things PvP servers otherwise build around their inventory — a protected pocket, loadouts, fast trade, death lootbags, a combat lock, an admin panel with live settings, Discord logs and 8 languages.

Runs **on its own, with no framework** (standalone), or on **QBCore** when it finds one. On QBCore it replaces qb-inventory and answers qb-inventory's exports itself, so qb scripts that give, take or check items keep working.

## Features

**For players**
- **Inventory + Protected.** The main inventory is lost on death (see [Death](#death-lootbags-and-the-combat-lock)); Protected is always kept.
- **Hotbar** with 7 slots (keys 1–7). Hover an item and press a number to bind it.
- **Loadouts.** Save the kit you carry, take it back from your stash in one click, and share it as a code (`K7Q-4ZP`). A code copies the list, never the items.
- **Fast trade.** Give to a nearby player or any server ID from one dialog, with a trades history. Nobody is picked until you pick; two quick buttons fill in half or a quarter of the stack. The people you traded with since joining are one click away, under the server ID they have now (the list starts empty on every join). The nearby list refreshes once per `Config.Give.RefreshCooldown` seconds. Both players get a dated notification.
- **Item count limit.** Right-click an item to set the most you want of it in the inventory. Taking from a stash or Protected then stops there, so nobody pulls too many weapons by accident. Lootbags, gives and admin gives are not limited.
- **Weapons.** The inventory is the only source of weapons. Attachments, tints and finishes come from a right-click menu and are remembered per weapon type. Default attachments and a launcher-spam cooldown are configurable. An empty weapon stays in hand until it is reloaded. Grenades, sticky bombs, molotovs and other throwables come out with as many as the stack holds; each one thrown takes one item.
- **Ammo** runs in one of two modes: fixed ammo (nothing to manage) or tracked ammo (rounds belong to the weapon, reloading uses ammo items). One ammo item loads one magazine of the weapon in hand — an extended or drum clip loads more (`Config.Ammo.RoundsPerItem = 'clip'`, the game's own magazine sizes). Every gun and launcher can be item-fed (RPG, Homing Launcher, Grenade Launcher, Minigun, Railgun, Firework, Flare Gun…); the ammo items qb-core lacks are added at start (`Config.Ammo.AddItems`), and no weapon loads more than GTA itself keeps in it. Reload with **R** or by using an ammo item; ammo is taken from the inventory first, then from Protected (`Config.Ammo.FromProtected`). With tracked ammo a weapon card shows the rounds loaded in that weapon, and nothing when it is empty; two equal weapons holding different rounds are two stacks; turn tracking off and they stack again.
- **Usable items with a progress bar** (`Config.UseItems`): armour, medkits, bandages… set the time, health, armour, animation and cooldown per item. The item is taken only when the bar has run.
- **Personal settings:** language, accent color, background, card size, fonts and styles, tier colors, and swapping the left and right mouse buttons. They are saved per player; **Reset** puts them all back except the language. The language covers the inventory and the panel; messages from the server stay in `Config.Locale`.
- **Notifications** in GTA's feed style, bottom left, always on screen: inventory open or not, and whatever your HUD does with the minimap (`Config.Notifications = 'gta'` uses GTA's own feed instead).
- **Walk with the inventory open** (`Config.MoveWhileOpen`). Mouse look, attack and aim stay off, and typing in a search box never walks.

**Death, lootbags and the combat lock**
- **On death** the main inventory drops into a lootbag at the death spot (or is cleared, or kept — your choice). Players can put items into a bag too (`Config.Lootbag.Deposit`), and the bag's prop, marker and particle effect are set live in the panel.
- **Free loadout** (`Config.FreeLoadout`): a kit given back on every respawn after a death. Its items stay with that one life — they never drop, go into a stash or get given — so it cannot be farmed.
- **Combat lock.** Weapon fights lock items out of the main inventory for a few seconds, so nobody hides loot mid-fight. Leaving the server while locked or down counts as a death.
- **Deaths that cost nothing** where you say so: an arena, an event, a safe zone (`Config.Death.Keep`, or `setKeepOnDeath` from your own script).

**Stashes**
- **Personal stashes** at any number of locations, each character with its own content. They open with a 3D label or an NPC, with an animation picked per location.
- **Shared stashes** for jobs (with the lowest grade) and gangs.
- **Temporary stashes** that other scripts create (crates, drops, events).

**Admin panel (`/invadmin`)**
- **Players:** open anyone's inventory, online or offline, and move, add or remove items. You can also give to many players at once, wipe a player, compare two players, or give one character a bigger or smaller inventory and Protected (**Carry limit**; characters with their own limit are marked).
- **Logs:** every trade, give, death, loot, delete and admin action, searchable. Security flags are highlighted.
- **Economy:** how many of each item exist on the server and how that changed over 24 h / 7 days, and who holds them.
- **Staff:** one superadmin adds admins and moderators and switches single permissions per person.
- **Live settings** (Items, Weapons & loadout, Stashes, Lootbags, Limits): change weights, tiers, limits, stash locations and NPCs, starter items, the free loadout and the lootbag's look without a restart.

**For the server owner**
- **Starter items** a new character gets once, on its first login (`Config.StarterItems`, or later from your own script).
- **Discord logs** with separate channels for admin actions and security.
- **Migration** from qb / ps / lj / qs / codem-inventory, ox_inventory and gfx-inventory.
- **Season wipe** with a full backup first.
- **8 languages:** English, Turkish, German, French, Spanish, Portuguese (Brazil), Polish, Italian.

## Editions

ns-inventory comes in two editions with the same features:

- **Escrow:** the code is protected by Cfx.re's asset escrow. `config.lua`, `items.lua`, `locales/` and `utils/` stay readable and editable; that covers every setting, the standalone item list and every text.
- **Open source:** every file is readable, including the interface's source in `ui/` (React + Vite). To change the interface, edit `ui/src/` and build it again: `cd ui`, `npm install`, `npm run build`. The build goes to `html/`, which the resource loads.

## Requirements

- **No framework needed.** Without one it runs standalone; with **QBCore** it uses QBCore's players, items, jobs and gangs (`Config.Framework`, see [Standalone or a framework](#standalone-or-a-framework)).
- **oxmysql** (recommended) or **mysql-async**.
- **OneSync.** Deaths and the combat lock are read on the server.
- On QBCore: **qb-inventory and qb-weapons must not run.** ns-inventory takes their place.

No build step and no CDN: the interface is already built and its fonts are included.

## Installation

1. On QBCore only: stop and remove **qb-inventory** and **qb-weapons** (delete or comment out their `ensure` lines).
2. Put the `ns-inventory` folder in your `resources`.
3. In `server.cfg`, after oxmysql or mysql-async (and after qb-core on QBCore):
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
| `X` | Cancel an item being used (`Config.UseItems.CancelKey`) |

Players who prefer it swap the left and right button in **Settings › Swap mouse buttons**: a click opens the actions and a right click moves one. The middle button stays fast trade, and a left drag always moves the stack. The hint bar at the bottom of the inventory shows each button as a mouse icon with what it does.

## Admin panel

Open it with `/invadmin`. What each role can do:

| Role | How to get it | Can |
|---|---|---|
| **Superadmin** | `ns-inventory.superadmin` ACE — one account | Everything, and the only one who adds, changes or removes staff |
| **Admin** | Staff screen, or the `ns-inventory.admin` ACE | Logs, move / remove items, give items, wipes, live settings |
| **Moderator** | Staff screen, or the `ns-inventory.moderator` ACE | Logs, move / remove items. Gives only with `Config.Admin.ModeratorsGive` or a switch on the Staff screen |

> Being in `group.admin` alone is **not** enough — a group is not an ACE. The `add_ace` line above is required.

**Live settings.** Most of `config.lua` can also be changed in the panel (Items, Weapons & loadout, Stashes, Lootbags, Limits). A value saved there wins over `config.lua`. If you change `config.lua` and nothing happens, check the panel: fields that differ from the config are marked, and **Reset to config** goes back to `config.lua`. Only the superadmin can reset.

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
| `Config.Framework`, `Config.Standalone` | Standalone or QBCore; the identifier and weapons when standalone |
| `items.lua` | The item list when running standalone |
| `Config.OpenKey`, `Config.MoveWhileOpen` | The inventory key; walking while it is open |
| `Config.Notifications` | `'nui'` (always on screen) or `'gta'` (GTA's own feed) |
| `Config.Containers`, `Config.Weights` | How much the inventory and Protected hold (grams); weights on or off |
| `Config.Death`, `Config.Lootbag` | What death does: `lootbag`, `clear` or `keep`; your death script's events; deaths that cost nothing (`Keep`); the bag's lifetime, model, marker, effect, and whether players can put items in |
| `Config.StarterItems` | What a new character gets once, on its first login, where (inventory or Protected), and once per account or per character |
| `Config.FreeLoadout` | The kit a player gets back on every respawn after a death that took the inventory |
| `Config.CombatLock` | Combat lock: length, what starts it, death on quitting while locked |
| `Config.Give` | Fast trade: distance, same routing bucket, cooldown, the dated notification |
| `Config.Stash` | Personal stash locations, shared stashes and who may open them, NPC or 3D text |
| `Config.Weapons`, `Config.DefaultComponents`, `Config.Customization` | Weapon rules, default attachments, the attachments menu |
| `Config.Ammo` | Fixed or tracked ammo, and which item reloads which ammo type |
| `Config.UseItems` | Items ns-inventory uses itself: progress bar, health / armour, animation, cooldown |
| `Config.ItemWeights`, `Config.ItemTiers` | Weights and tiers (card colors) over your framework's items |
| `Config.Loadouts` | How many loadouts a player keeps; share codes |
| `Config.Discord`, `Config.Logs` | Discord logs and how long the audit log is kept |
| `Config.SeasonWipe`, `Config.Migration` | Season wipe and migration |
| `Config.Admin` | Panel command, item commands, moderator gives, give notifications |
| `Config.UI`, `Config.Sounds` | Default accent color and fonts; sounds |

**Weapons from addon packs.** With `Config.Weapons.GameWeaponsOnly = true` (default), only GTA's own weapons are items. Set it to `false` to use addon weapon packs.

**Other weapon scripts.** With `Config.Weapons.StripUnowned = true` (default), a weapon the inventory did not give is taken away and logged. Turn it off, or add the weapon to `Config.Weapons.Allowed`, if another script (paintball, an event) hands out weapons.

## Standalone or a framework

`Config.Framework = 'auto'` (default) uses QBCore when the server has `qb-core`, and runs standalone without it. Name one to choose yourself: `'qb'` or `'standalone'` (standalone works on a QBCore server too, ignoring QBCore).

**Standalone** (no framework):
- **Characters** are player accounts, by `Config.Standalone.Identifier` (`'license'` by default). The inventory loads as soon as a player joins.
- **Items** come from `items.lua`, plus every GTA weapon (`Config.Standalone.Weapons`) and the ammo items in `Config.Ammo.AddItems`. Each item: `label`, `weight` (grams) and an optional `image`. Pictures go in `images/items/`.
- **Usable items**: `Config.UseItems` (progress bar, health, armour) or your own resource:
  ```lua
  exports['ns-inventory']:registerUsableItem('bandage', function(source, item)
      -- item = { name, label, count, metadata, container }
      exports['ns-inventory']:removeItem(source, item.container, item.name, 1)
  end)
  ```
- **Deaths** are read on the server from the player's health; list your death script's events in `Config.Death.DeathEvents` / `ReviveEvents` if it has them.
- **Shared stashes** open by ACE or identifier (`Config.Stash.Shared`); there are no jobs or gangs.
- **Admin panel** search finds offline players by the name they last joined with.
- Coming from **gfx-inventory**: the old items come over by themselves on first join (Config.Migration).

`registerUsableItem` works on QBCore too, next to `QBCore.Functions.CreateUseableItem`.

## Death, lootbags and the combat lock

- **Death** (`Config.Death.Mode`)
  - `lootbag`: the main inventory drops into a bag at the death spot. Anyone in the same routing bucket can loot it: `E` takes everything that fits, and the inventory key opens it beside the inventory. The dead player waits `Config.Lootbag.OwnerDelay` seconds before looting their own bag (`false` = never), so respawning next to it does not hand the loot back.
    - **Putting items in** (`Config.Lootbag.Deposit`): opened with the inventory key, a bag takes items too, up to `MaxWeight` grams. Bags live in memory: what is in a bag when it expires or the server restarts is gone.
    - **Look:** the prop (`Model`) and the marker (`Marker`: type 0–43, color, size, height, distance, or off). The panel's **Lootbags** screen changes these, the effect and putting items in live, also for the bags already on the ground.
    - **Effect** (`Effect`): a particle effect on legendary loot only (a bag holding one of `Legendary`), on every bag, or off. It is one of `EffectPresets` — effects checked in game (money rain, red flare, fire, lightning, smoke column, confetti…). There is no free name field on purpose: a particle name that does not exist can crash a player's game when the pause menu opens. Add your own presets in `config.lua` and try them first with **Lootbags › Test in game**, which plays the effect in front of you (only you see it) and steps through the presets live. Bursts (`loop = false`) play again every `repeatMs`; no effect starts while the pause menu is open.
  - `clear`: the main inventory is emptied.
  - `keep`: nothing is lost.

  **When:** once the player is fully dead — a revive in time loses nothing. ns-inventory knows it by itself: on QBCore when the last stand ends (qb's dead state), without a framework the moment the player dies. Another death script? List its events:
  ```lua
  Config.Death.DeathEvents  = { 'hospital:server:SetDeathStatus' }   -- the player is fully dead
  Config.Death.ReviveEvents = { 'hospital:client:Revive' }           -- the player is up again
  ```
  Client and server events both work; an event called with `false` counts as the other one. The server still checks every death itself: a death event only counts once the server saw the player go down, a revive event only once the server sees the player up again, and a death with no event at all drops after 10 minutes. Until the death script decides, the inventory stays locked. Leaving the server while down counts as a death.

  Protected is always kept. Deaths are confirmed by the server itself, so a client cannot hide one.

  **Deaths that cost nothing** (`Config.Death.Keep`): return `true` to keep the items, e.g. in an arena or during an event. Scripts can switch it per player instead: `exports['ns-inventory']:setKeepOnDeath(source, true, 'arena')`, and `false` with the same reason when the player leaves the arena.
  ```lua
  Keep = function(source, reason)   -- reason: 'died', 'down_quit' or 'combat_quit'
      return GetPlayerRoutingBucket(source) ~= 0
  end,
  ```
- **Combat lock** (`Config.CombatLock`): hitting or being hit by a weapon locks both players for a few seconds.
  - Items can still come **into** the main inventory, but nothing can leave it: no Protected, no stash, no give.
  - Leaving the server while locked, or while down, applies the death mode.
- **Free loadout** (`Config.FreeLoadout`, or the panel's Weapons & loadout › Free loadout): the items a player gets back every time they respawn after a death that took the inventory (`lootbag` or `clear`). A death that costs nothing gives no new kit.
  - The kit's items stay with that one life: on death they are removed (from the inventory and Protected, never dropped into the bag), they never go into a stash (not even from the admin view), and they cannot be given. Free ammo only loads a free weapon.
  - Other scripts do not see them: framework item checks and removals (`HasItem`, `GetItemCount`, `RemoveItem`, and ns-inventory's own `hasItem` / `removeItem`) skip them, so a shop or a crafting script cannot turn a kit into money. The one way out is using one: a free item handed to another script's usable-item callback can be taken by that script right after.
  - Dying, stashing, selling or passing them to a friend never farms them.
  - A player who leaves before respawning gets the kit on the next login.
- **Open lock:** while dead or in last stand the inventory stays closed, and it opens again once revived. Other scripts can lock it too (handcuffs, a minigame); see [For developers](#for-developers).

## Usable items

Items listed in `Config.UseItems.Items` are used by ns-inventory itself; every other item still goes to your framework's usable-item callback (`QBCore.Functions.CreateUseableItem`).

```lua
Config.UseItems = {
    ProgressBar = 'nui',   -- 'nui' (built in), 'qb' (the progressbar resource) or false (none: used at once)
    CancelKey   = 'X',
    Items = {
        armor   = { duration = 4, armour = 100, anim = { dict = 'clothingtie', clip = 'try_tie_negative_a', flag = 49 } },
        bandage = { duration = 3, health = 15 },
        ifaks   = { duration = 5, health = 50, cooldown = 10, canMove = false },
    },
}
```

- `duration` seconds on the bar · `health` added (a full bar is 100) · `armour` added up to 100 · `remove = false` keeps the item · `cooldown` seconds before the same item again · `canMove` / `inVehicle` · `anim`.
- Your own effect: `event` (client event, gets the item name) and `serverEvent` (server event, gets `source` and the item name). `ns-inventory:server:itemUsed` (`source, itemName`) fires for every finished use.
- The server decides: the item is held while the bar runs (it cannot be moved, given or deleted), it is taken and the effect applied only after the bar's full time, and dying or the cancel key ends it with nothing used.

## Stashes

- **Personal stash:** every character has one. Set its locations in `Config.Stash.Personal.Locations`. Each location can have its own name (`label`, e.g. "Legion Square Stash") shown at the spot; without one players see the personal stash's name. Stash ids are made by the panel and never shown.
- **Shared stashes:** add them in `Config.Stash.Shared`, with `access` by job and grade or gang. Only those rules open it in the world — admins too (`Config.Stash.AdminsOpenShared = true` lets admins open every one); the panel's Stashes screen shows and edits every stash's contents. For special cases `config.lua` also takes identifiers and an ACE; the panel leaves those as they are.
- **In game:** on the panel's **Stashes** screen, stand on the spot and press **Add stash**. It is placed where you stand; pick **Personal** or **Shared**, and the screen shows that type's settings. **Save** puts it in place for everyone.
- **How players open them** (`Config.Stash.Interact`)
  - `E` inside the radius opens the inventory with the stash beside it.
  - A location shows either a 3D text or an NPC with a help text (`Opener = 'text' | 'npc'`), set per location.
  - An NPC's idle is set per location (`anim`): `'none'`, a scenario (`'WORLD_HUMAN_SMOKING'`) or an animation as `'dictionary clip'`. Without one it uses `Interact.Ped.Scenario`. The panel's Stashes screen has ready-made ones (guard, cop, arms crossed, phone, smoking…). **Test › Preview in game** (under the animations) spawns the NPC at the position entered above, turns the game camera to it with the panel out of the way, and steps through the animations live (`←` `→`); **Done** or `ESC` goes back to the panel with the one you picked.
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

## Item pictures

Put pictures in the `images/items/` folder, named after the item (`phone.webp`) or after your framework's `image` field (`phone.png`). WebP is smallest; PNG, JPG and GIF work too. Every GTA weapon already has a picture there; a picture of yours with the same name replaces it. Items without a picture show a placeholder.

On QBCore, bring your item pictures over from your old inventory (qb-inventory keeps them in `html/images/`). Copying them as they are works. To make them smaller, convert the folder to WebP (needs Node.js and ffmpeg):

```text
node tools/import-icons.js "C:/path/to/qb-inventory/html/images"
```

It writes `images/items/<name>.webp` and skips pictures that are already there (`--force` replaces them). A picture added while the server runs shows after a restart of ns-inventory.

## For developers

**Server exports**

```lua
-- Players
exports['ns-inventory']:getInventoryItems(source)   -- main inventory
exports['ns-inventory']:getProtectedItems(source)   -- Protected
exports['ns-inventory']:getAllItems(source)         -- both
--   → { { name, label, count, metadata, weight, container } } or nil while the player loads
exports['ns-inventory']:takeInventory(source)       -- empties the main inventory, returns the items

-- Items by container: 'inventory' or 'protected' ('all' = both where it is allowed)
exports['ns-inventory']:hasItem(source, 'all', 'bandage', 3)             -- → has, amount
exports['ns-inventory']:addItem(source, 'protected', 'bandage', 3)       -- → ok, reason (weight checked)
exports['ns-inventory']:removeItem(source, 'inventory', 'bandage', 1)    -- → ok, reason ('all' = inventory first)
exports['ns-inventory']:clearInventory(source, 'protected')              -- → the items it held ('all' = both); saved at once
exports['ns-inventory']:getWeight(source, 'inventory')                   -- → used, max (grams)
exports['ns-inventory']:canCarry(source, 'inventory', 'bandage', 10)     -- → ok, reason
exports['ns-inventory']:useItem(source, 'bandage')                       -- as if the player used it
exports['ns-inventory']:saveInventory(source)                            -- saves the player now
--   reasons: not_ready, invalid_container, invalid_item, invalid, not_enough_space, not_enough_items

-- Starter items (Config.StarterItems): once per character
exports['ns-inventory']:giveStarterItems(source)    -- → ok, reason: not_ready, disabled, already_given, unknown

-- Deaths that cost nothing while any reason holds. Reasons belong to your resource: they go when it
-- stops, on logout, and when ns-inventory restarts (set them again on its onResourceStart)
exports['ns-inventory']:setKeepOnDeath(source, true, 'arena')

-- Opening the inventory. While locked the server also refuses the hotbar, reload and lootbag keys;
-- useItem above still works.
exports['ns-inventory']:setInventoryBlocked(source, true, 'cuffed')   -- each script lifts only its own reason
exports['ns-inventory']:canOpenInventory(source)                      -- → canOpen, { reasons }

-- Game modes. allowWeapon lets one weapon your script gives natively stay in hand (it is not
-- stripped); setting it holsters the inventory's weapon first. Reasons belong to your resource,
-- like setKeepOnDeath. → ok, reason: not_ready, invalid_weapon
exports['ns-inventory']:allowWeapon(source, 'WEAPON_PISTOL', 'duel')  -- false instead of the name lifts it
exports['ns-inventory']:isCombatLocked(source)                        -- → locked, msLeft, down

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

**Carry limits** (grams; `target` = a server ID or a citizen ID)

```lua
exports['ns-inventory']:getCarryLimits(target)
--   → { inventory, protected, own = { inventory, protected }, bonus = { inventory, protected }, server = { … } }
exports['ns-inventory']:setCarryLimits(target, 120000, nil)          -- own limit, saved (nil = the server's)
exports['ns-inventory']:setCarryBonus(source, 'backpack', 20000)     -- +20 kg while online, not saved
exports['ns-inventory']:setCarryBonus(source, 'backpack', 0)         -- removes it
```

`setCarryLimits` is the same as **Players › Carry limit** on the admin panel. `setCarryBonus` is for things that come and go (a backpack, a perk, a heavy armour with a negative value); each script uses its own reason, and the bonuses add up.

**An arena or duel script**

```lua
local INV = exports['ns-inventory']

local leaving = {}   -- [src] = token of the pending exit (a new match cancels it)

local function Enter(src, weapon)
    -- A player in a fight (leaving now costs their items), down (also a down the server will still
    -- apply), or locked by another script (cuffed) never starts a match.
    local locked, _, down = INV:isCombatLocked(src)
    if locked or down or not INV:canOpenInventory(src) then return false end
    leaving[src] = nil
    if not INV:setKeepOnDeath(src, true, 'duel') then return false end
    INV:setInventoryBlocked(src, true, 'duel')
    if not INV:allowWeapon(src, weapon, 'duel') then
        INV:setInventoryBlocked(src, false, 'duel')
        INV:setKeepOnDeath(src, false, 'duel')
        return false
    end
    GiveWeaponToPed(GetPlayerPed(src), GetHashKey(weapon), 250, false, true)
    return true
end

-- lastDeathAt: GetGameTimer() of the player's last death in the match, or nil.
local function Leave(src, weapon, lastDeathAt)
    RemoveWeaponFromPed(GetPlayerPed(src), GetHashKey(weapon))
    INV:allowWeapon(src, false, 'duel')
    -- Keep and lock stay a little longer and go together (no weapon while a death is still free).
    -- On QBCore a down that qb never reported is applied ~10 s after the player is up again,
    -- so wait past that, and until the server reads the player alive (10 minutes at most).
    local token = {}
    leaving[src] = token
    CreateThread(function()
        local liftAt = math.max(GetGameTimer() + 5000, (lastDeathAt or 0) + 11000)
        local giveUp = GetGameTimer() + 600000
        while leaving[src] == token and GetPlayerName(src) and GetGameTimer() < giveUp
            and (GetGameTimer() < liftAt or GetEntityHealth(GetPlayerPed(src)) <= 100) do
            Wait(500)
        end
        -- Dropped (ns-inventory forgot them on logout) or in a new match: leave it alone.
        if leaving[src] ~= token or not GetPlayerName(src) then return end
        leaving[src] = nil
        INV:setKeepOnDeath(src, false, 'duel')
        INV:setInventoryBlocked(src, false, 'duel')
    end)
end
```

All three go when ns-inventory restarts: on its `onResourceStart`, set them again or end the running matches. Do not lift them when a player drops; ns-inventory forgets them on logout, after it applies the quit penalty.

**Client exports:** `setInventoryBlocked(blocked, reason)` is a local lock for UI scripts (a phone, a progress bar), not a security check. The others are `canOpenInventory()`, `isInventoryOpen()`, `openInventory()` (as if the key was pressed; every lock still applies) and `closeInventory()`.

**Server events** (`AddEventHandler`, only the server can trigger them)

| Event | Arguments |
|---|---|
| `ns-inventory:server:onDowned` | `source`: dead or in last stand |
| `ns-inventory:server:onRevived` | `source`: up again |
| `ns-inventory:server:onDeath` | `source, { reason, coords, bucket, items, lootbag, kept }`. `reason` is `died`, `down_quit` or `combat_quit`; `kept` is `true` when the death cost nothing |
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
| Weapons from another script disappear | `Config.Weapons.StripUnowned = false`, add them to `Config.Weapons.Allowed`, or let that script call `allowWeapon` for the player |
| The inventory will not open | Dead or in last stand (`Config.Death`), or another script locked it |
| Nothing is saved / "no database" in the console | Start oxmysql or mysql-async before ns-inventory |
| A qb shop does not open | Shops are not supported yet (see above) |

For extra console output and the test commands `/giverandom` and `/testlootbag`, set `Config.Debug = true`. Keep it off on a live server.
