# ns-dailyrewards

Daily login rewards for FiveM. Players open a weekly calendar with `/daily`, claim one reward a day and build a streak. Day 5 rolls a mystery box, day 7 is the big reward, then a new week starts.

Works with **ESX, QBCore, Qbox, ND_Core, ox_core, vRP (1.x, vRPex, vRP 2, Creative) and standalone servers**, and with 28 inventories. The framework, inventory and SQL driver are detected on start — nothing to set up.

**Free and open source** under the MIT license. Use it, change it, share it.

## Features

- **A week of rewards** — a reward every day and a big one on day 7, then the week starts over. Longer calendars (14, 28, 30 days) work too; the window shows one week at a time.
- **Streaks with a grace day** — miss one day and the streak survives. Or restart on any missed day, or never lose progress.
- **Every reward type** — cash, bank, items, weapons with ammo, owned vehicles, mystery boxes, and hooks that let any resource hand out anything (VIP, XP, job grades).
- **Every day is a surprise** — each day is its own closed gift box (six designs, plus a black-and-gold one for the big reward) until it is claimed; then the box shakes, opens, and the reward appears with its own picture. Prefer to show the rewards up front? One setting.
- **Mystery boxes** — weighted prizes picked on the server, opened with a case-opening reel that shows every prize with its chance.
- **A reward window that looks the part** — a centred window with your server's scene on top, one gold claim button, and the week as tall cards with the big reward in gold. Item pictures come straight from your inventory.
- **A command or an NPC** — players type `/daily`, or walk up to an NPC with a map icon. You choose; both work too. Players are told when a reward is waiting: on join and at the new day.
- **Safe by design** — the server decides which day is claimed. A claim is saved before anything is handed over, two claims cannot race, and a full inventory never uses up the day.
- **Discord logs** — claims, admin actions and errors, each to its own webhook.
- **One file of texts** — every line players see is in `locales/en.lua`, ready to translate.

## Requirements

- A SQL driver: **oxmysql** (recommended), **ghmattimysql** or **mysql-async**
- *(optional)* **ox_lib** — the window can take its accent colour
- *(optional)* **ox_target** or **qb-target** — for the NPC. Without one, players press E next to it

| Framework | Notes |
|---|---|
| ESX · QBCore · Qbox | Items through your inventory; on plain ESX / QBCore through the core itself |
| ND_Core · ox_core | Items through ox_inventory. On ox_core, cash is the ox_inventory `money` item and bank is the character's account |
| vRP 1.x · vRPex | Its own inventory, or a supported one on top. Weapons are given as `wbody|WEAPON_X` with their rounds as `wammo|WEAPON_X`; an item vRP does not define is refused |
| vRP 2 | ns-dailyrewards loads a small bridge into vRP (`utils/vrp2.lua`) — vRP 2 offers other resources nothing else |
| Creative (Passport API and v4/v5 bases) | Cash is the base's wallet item (`dollar` / `dollars`, read from its config). Use your base's exact item names: these bases accept any name, so a wrong one gives nothing and cannot be detected |
| Standalone | The window, streaks, mystery boxes and hooks work. Items need an inventory that runs without a framework, such as ns-inventory. Weapons go to the weapon wheel and last until the player dies or leaves. Money needs a framework |

<details>
<summary>Supported inventories (28, detected automatically)</summary>

| Inventory | Items & weapons | Room check | Pictures in the window |
|---|---|---|---|
| ns-inventory | ✓ | ✓ | ✓ |
| ox_inventory (also on ND_Core, ox_core, Qbox) | ✓ | ✓ | ✓ (honours `inventory:imagepath`) |
| wasabi_inventory · nextgenfivem_ox_inventory | ✓ | ✓ | ✓ |
| f4-inventory · devix-inventory | ✓ | ✓ | ✓ |
| aty_inventory · oph3z-inventory | ✓ | ✓ | ✓ |
| cc_inventory · bp_inventory | ✓ | ✓ | ✓ |
| qs-inventory-pro | ✓ | ✓ | ✓ |
| qb-inventory 2.x / 1.x | ✓ | 2.x only | ✓ |
| qs-inventory | ✓ | ✓ | ✓ |
| jaksam_inventory | ✓ | ✓ | ✓ |
| tgiann-inventory | ✓ | ✓ | ✓ |
| ak47_inventory / ak47_qb_inventory | ✓ | ✓ | ✓ |
| origen_inventory | ✓ | ✓ | ✓ |
| codem-inventory (mInventory) | ✓ | — | ✓ |
| codem-inventoryv2 (Inventory AIO) | ✓ | ✓ | ✓ |
| core_inventory | ✓ | ✓ | ✓ |
| ps-inventory · lj-inventory | ✓ | — | ✓ |
| jpr-inventory · one_inventory | ✓ | — / ✓ | ✓ |
| gfx-inventory | ✓ | ✓ | ✓ |
| esx_inventoryhud · plain ESX | ✓ (weapons in the loadout) | ✓ | ✓ / built-in |
| QBCore without an inventory resource · vRP · Creative | ✓ | — | built-in |
| axr_inventory (vRP) | ✓ | — | built-in |

"Room check" means a full inventory is refused **before** anything is handed over. Without one, items are still all-or-nothing: what went in is taken back if the rest does not fit.

Another inventory? Add its picture folder to `shared/inventories.lua` and its item calls to `utils/server.lua` (both not encrypted).

</details>

## Installation

1. Put `ns-dailyrewards` in your `resources` folder.
2. Add `ensure ns-dailyrewards` to `server.cfg`, **after** your framework, inventory and SQL resource (and after ox_target / qb-target, for the NPC).
3. Give your admins the permission in `server.cfg`:
   ```cfg
   add_ace group.admin ns-dailyrewards.admin allow
   ```
4. Open `server/sv_config.lua` and set `ConfigServer.RollSalt` to any random text. It keeps mystery box results unpredictable.
5. Start the server and read the console. Every item name in the calendar is checked against your server:
   ```
   [ns-dailyrewards] item check: 12/14 resolved (ox_inventory)
   [ns-dailyrewards]   • "sandwich" — day 2, pool small
   ```
   Rename anything it lists in `config.lua`. ESX servers usually call `water_bottle` and `sandwich` `water` and `bread`.

The database table is created on first start.

**Framework not detected?** A renamed core resource can be named in `server.cfg`:
```cfg
setr ns_framework "vrp"              # esx | qb | qbx | nd | ox | vrp | standalone
setr ns_framework_resource "vrp"     # vRP only: the core's folder name
setr ns_vrp_wallet "dollars"         # Creative only: the wallet item, if not read from the base's config
```

## Configuration

| File | What is in it |
|---|---|
| `config.lua` | The settings below, the week and the mystery boxes |
| `server/sv_config.lua` | Discord webhooks, admin command and permission, mystery salt. Never sent to players. |
| `server/vehicles.lua` | Garage and plate format for vehicle rewards, or your own garage code |

### Settings

```lua
Config.Locale     = 'en'              -- the texts are in locales/en.lua

Config.OpenWith   = 'command'         -- 'command' = players type /daily · 'npc' = players talk to an NPC · 'both'
Config.Command    = 'daily'
Config.Npc = {
    model     = 'a_m_y_business_03',
    scenario  = 'WORLD_HUMAN_CLIPBOARD',  -- false = it only stands
    blip      = true,
    locations = {
        vector4(193.43, -936.21, 30.70, 324.5),   -- Legion Square
    },
}

Config.ResetHour  = 0                 -- the hour (0-23) a new day starts for everyone
Config.TimeZone   = 3                 -- UTC offset: 3 Istanbul · 1 Berlin · 0 London · -5 New York
Config.GraceDays  = 1                 -- missed days in a row a player can skip. 0 = one missed day restarts at day 1

Config.Logo       = 'img/logo.png'    -- replace html/img/logo.png with your logo. false = no logo
Config.Accent     = '#E8B04B'         -- the main colour
Config.Currency   = '$'
```

- **`OpenWith`** — how players open the window. See [Command or NPC](#command-or-npc).
- **`ResetHour` / `TimeZone`** — daylight saving is not applied: use the offset your players live in most of the year. Change them only at the moment a new day starts; a change in the middle of a day can give players one extra claim.
- **`GraceDays`** — with `1`, a player can miss one day and keep both the calendar and the streak. Miss more days in a row and the calendar restarts at day 1.
- **`Logo`** — the Native Scripts logo by default. Replace `html/img/logo.png` with your own (a PNG or WebP with a transparent background works best), point `Logo` at another file, a `nui://` path or an `https://` URL, or set `false` for none. Any shape fits: it is held to the title bar's height.
- **`Accent`** — the claim button and today's card. `nil` follows your ox_lib theme.

### Command or NPC

- **`'command'`** — players type `/daily` (the name is `Config.Command`).
- **`'npc'`** — an NPC stands at each entry of `locations`; there is no command. A claim is only accepted near an NPC (15 m), checked on the server.
- **`'both'`** — the command and the NPCs.

With **ox_target** or **qb-target** running, players open the window through the target; without one they press **E** next to the NPC. The NPC exists only for players within 60 m, and only on their own game. It cannot be hurt and does not move.

To place an NPC, stand where it should be, read your position (`x, y, z`) and the direction you face (heading) from your admin menu, and add a line `vector4(x, y, z, heading)`. The height does not need to be exact: the NPC is put on the ground. More lines = more NPCs, each with its own map icon.

### Optional settings

These are not in `config.lua`. Add a line to change one.

| Setting | Default | What it does |
|---|---|---|
| `Config.Surprise` | `true` | `false` shows each day's reward, its picture and, for a mystery box, the odds, instead of a closed box. |
| `Config.Banner` | `'img/scene.jpg'` | The picture at the top: a file in `html/img/`, a `nui://` path, an `https://` URL, or `false`. Use a wide screenshot of your server (about 1600×500); the left side is darkened for the text. |
| `Config.ItemImages` | `'auto'` | `'auto'` reads pictures from the running inventory. Or a pattern such as `'nui://my_inventory/images/%s.png'`, or `false`. An item without a picture shows a parcel. |
| `Config.Blur` | `true` | Blur the game behind the window. |
| `Config.Sounds` | `true` | Game sounds on open and claim. |
| `Config.RollingReset` | `false` | `true` opens the next day 24 hours after each player's own last claim. `ResetHour` and `TimeZone` are then not used. |
| `Config.AnnounceNewDay` | `true` | Tell everyone online that a new day started. Not used with `RollingReset`. |
| `Config.NeverRestart` | `false` | `true` never sends the calendar back to day 1. A missed day only sets the streak counter back to 0. |
| `Config.TrackBy` | `'character'` | `'license'` gives each Rockstar account one calendar for all its characters, so a player cannot claim once per character. |
| `Config.Debug` | `false` | Debug lines in the server and F8 console. |

### The week

One entry per day. After the last day the calendar starts again at day 1. The default week is filled in as an example — change, add or remove anything.

| Day | Default reward |
|---|---|
| 1 · Welcome Pack | $1,000 cash, 5 water bottles, 5 sandwiches |
| 2 · First Aid | 5 bandages, 3 painkillers, 1 first aid kit |
| 3 · Payday | $2,500 cash, $2,500 bank |
| 4 · Toolbox | 3 lockpicks, 2 repair kits, 1 cleaning kit |
| 5 | A Mystery Box and $500 cash |
| 6 · Gear Up | 2 armor, 1 radio, a knife |
| 7 · Weekly Jackpot | A Golden Box and $5,000 cash (the gold card) |

```lua
Config.Days = {
    -- Day 1
    {
        label = 'Welcome Pack',
        money = { cash = 1000 },
        items = {
            { name = 'water_bottle', count = 5 },
            { name = 'sandwich',     count = 5 },
        },
    },
    -- …
    -- Day 7: the big reward of the week
    {
        milestone = true,
        label     = 'Weekly Jackpot',
        mystery   = 'big',
        money     = { cash = 5000 },
    },
}
```

| Field | Example |
|---|---|
| `money` | `{ cash = 500, bank = 1000 }` |
| `items` | `{ { name = 'water_bottle', count = 3 } }` — `metadata = { … }` is passed to the inventory |
| `weapons` | `{ { name = 'WEAPON_PISTOL', ammo = 60 } }` |
| `vehicle` | `{ model = 'sultan', label = 'Karin Sultan' }` — add `image = 'img/sultan.png'` for a picture; ESX: `type = 'boat'` for non-cars |
| `mystery` | `'small'` — one prize from a box in `Config.Pools` |
| `hooks` | your own server event — see [Hooks](#hooks) |
| `milestone` | `true` — the big reward card (gold) |
| `label` | `'Payday'` — the day's name, shown once it is claimed (a surprise until then) |
| `image` | `'img/starter.png'` — the card's picture. A file in `html/img/`, a `nui://` path or an `https://` URL |

A day can mix fields: `{ money = { cash = 1500 }, items = { … } }` gives both.

**Want a month?** Add more days. 28 entries make four weeks, and the window pages through them week by week. A milestone without a `label` is then called "Week 1", "Week 2"… A car for finishing the month:

```lua
    { milestone = true, vehicle = { model = 'sultan', label = 'Karin Sultan' }, money = { cash = 5000 } },
```

**A reward that cannot be delivered:**
- Inventory full — nothing is handed over, the day is not used up, the player is told to make room.
- An item your server does not have, or a hook whose resource is stopped — the day still counts, the player is told what did not arrive, and the Errors webhook gets the details.

### Mystery boxes

```lua
Config.Pools = {
    small = {
        label = 'Mystery Box',
        entries = {
            { weight = 40, money = { cash = 600 } },
            { weight = 25, items = { { name = 'sandwich', count = 4 } } },
            { weight = 5,  money = { cash = 3000 } },
        },
    },
}
```

The server picks one entry by weight. Entries take the same reward fields as a day. The window shows each entry's chance, worked out from the weights. The default Golden Box has a commented car line: set the garage in `server/vehicles.lua`, then remove the `--` to make a car the rarest prize.

The result is fixed per claim: a player whose inventory was full gets the **same** prize when they retry, so emptying the inventory cannot re-roll it.

### Hooks

A hook fires a **server-side** event with the player, your `args` and the day number. Use it to give anything another resource controls.

```lua
-- config.lua
hooks = { { event = 'my-vip:dailyreward', args = { days = 3 }, resource = 'my-vip', label = 'VIP for 3 days', icon = 'star' } }

-- your own resource
AddEventHandler('my-vip:dailyreward', function(source, args, day)
    if GetInvokingResource() ~= 'ns-dailyrewards' then return end
    exports['my-vip']:AddVip(source, args.days)
end)
```

- Listen with `AddEventHandler`, **never** `RegisterNetEvent` — a net event can be fired by any client.
- With `resource` set, a hook whose resource is not running counts as not delivered.
- Without a `label`, the window calls it "Bonus".

### Vehicles

Set the garage and plate format at the top of `server/vehicles.lua`.

| Framework | How the vehicle is registered |
|---|---|
| ESX | A row in `owned_vehicles` (like esx_vehicleshop), stored in the garage. Uses the `parking` column on ESX Legacy when `GARAGE` is set. |
| QBCore | A row in `player_vehicles`, parked in `pillboxgarage` or `GARAGE`. |
| Qbox | `exports.qbx_vehicles:CreatePlayerVehicle`. **Set `GARAGE`** to one of your qbx_garages garages, or the car is created as "out". |
| ND_Core | A row in `nd_vehicles` (like ND's own `setVehicleOwned`), stored. Boats and aircraft: add `type = 'boat'`, `'heli'` or `'plane'` so ND's water and air garages list them. |
| ox_core | `exports.ox_core:CreateVehicle`, stored in `GARAGE` (or ox_core's `ox:defaultVehicleStore`, the impound by default). The model must be in ox_core's vehicle list. |
| vRP · Creative · custom garage | Write a `CustomGive` function in `server/vehicles.lua`. |

Plates follow `PLATE_FORMAT` (`'NS......'` — each `.` is a random letter or digit) and are checked for duplicates.

## Rules for your server

The rewards in this resource are **free**: a player earns each day by logging in. Keep it that way. Rockstar's Creator Platform License Agreement (section 3.1) forbids offering or selling loot boxes, random drawings or similar chance-based features for real money **or in-game currency**, and selling in-game currency for real money. In practice:

- Do not sell claims, day skips, extra mystery boxes or unlocks — on Tebex or for in-game money.
- Do not tie paid packages (VIP and the like) to extra boxes or claims through a hook.
- `/dailyadmin` is for staff and testing, not for sale.

Read the current agreement at fivem.net/terms. This note is not legal advice.

## Admin commands

Need the `ns-dailyrewards.admin` permission (or the server console).

| Command | Effect |
|---|---|
| `/dailyadmin unlock [id]` | They can claim again right now — skips the wait (handy for testing) |
| `/dailyadmin setday [id] <day>` | They can claim `<day>` now |
| `/dailyadmin reset [id]` | Wipe their progress — they start at day 1 |
| `/dailyadmin info [id]` | Show the player's day, streak, best streak and claim count |

In game, leave the id out to act on yourself: `/dailyadmin unlock`, `/dailyadmin setday 7`. The server console always needs an id.

There is no "reset everyone" command on purpose. If you need it: `DELETE FROM ns_dailyrewards;`

## Discord logs

Put the webhook URLs in `server/sv_config.lua`. Leave one empty to turn that channel off.

| Channel | What it gets |
|---|---|
| `Claims` | Every claim: player, day, what was delivered, streak, mystery prize |
| `Admin` | Every admin command |
| `Errors` | Failed deliveries, failed rollbacks, SQL timeouts |

## For developers

**Server exports**

```lua
-- progress = nil when the player is not loaded
exports['ns-dailyrewards']:getProgress(source, function(progress)
    -- progress.day, .canClaim, .streak, .bestStreak, .totalClaims, .cycles, .nextAt
end)

exports['ns-dailyrewards']:openMenu(source)
```

**Server event** — after every successful claim:

```lua
AddEventHandler('ns-dailyrewards:server:claimed', function(source, day, granted)
    -- granted = { items = {…}, weapons = {…}, money = {…}, vehicles = {…}, hooks = {…} }
end)
```

**Client exports**: `exports['ns-dailyrewards']:open()`, `:close()`, `:isOpen()`.

With `Config.OpenWith = 'npc'`, `openMenu` and `open` still open the window anywhere, but a claim far from an NPC is refused.

## Translations

The resource ships in English. Every line players see is in `locales/en.lua`.

- **One language:** translate the lines in `locales/en.lua`. Keep every `%s` and `%d`.
- **A language next to English:** copy `locales/en.lua` to `locales/<code>.lua`, change `Locales.en` to `Locales.<code>`, translate, and set `Config.Locale = '<code>'`. Missing lines fall back to English.

## License

The Lua, HTML, CSS and JavaScript source is released under the [MIT license](LICENSE). The bundled fonts keep their SIL Open Font License 1.1 (see `html/fonts/README.md`). The Native Scripts name and logo are not part of the license — replace `html/img/logo.png` with your own.
