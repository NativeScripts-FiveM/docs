# ns-truckerjob

An advanced trucker job for FiveM (ESX, QBCore, Qbox).

## Features

- **Job board** on a live map: a personal offer list for each player, plus shared "opportunity" jobs with limited slots.
- **Pay tags** that show every reason the pay goes up or down.
- **Rented or own truck.** Rent one per job, or buy a truck at a dealer, keep it in a truck park, repair, upgrade and repaint it.
- **Levels and skills.** XP, a skill tree and qualifications that unlock cargo.
- **Employers and rivals.** Each driver has a relation score with each employer.
- **Cargo rules.** General, fragile, valuable, chemical, cold and heavy cargo. Damage, lateness and spoiling cut the pay.
- **Deck loads.** Open trailers carry visible vehicles or props.
- **Convoys.** Players form a convoy, take one job together and split the pay.
- **Companies.** Shared vault, ledger, roles (owner, manager, driver), company trucks, member map with faces.
- **NPC drivers.** Companies hire NPC drivers who work in the background, and send them to training courses.
- **Dealer sales and inbox.** Timed sales, follow dealers and models, get notified.
- **Admin panel** with a world editor for every point on the map.
- **8 languages:** `en`, `tr`, `de`, `fr`, `es`, `pt`, `ru`, `pl`. Each player picks their own. `de` to `pl` are machine translations.

## Requirements

| | |
|---|---|
| Required | OneSync, one framework (`qbx_core`, `qb-core` or `es_extended`), one SQL driver (`oxmysql`, `ghmattimysql` or `mysql-async`) |
| Optional | `ox_lib` (notices), a key script and a fuel script (see [Keys and fuel](#keys-and-fuel)) |
| Not needed | inventory or target scripts |

- Start the framework and the SQL driver **before** this resource.
- Standalone is not supported.
- Only `oxmysql` was tested against a real database.
- All money uses the `bank` account.

## Install

1. Copy `ns-truckerjob` into your `resources` folder.
2. Add to `server.cfg`, in this order:

   ```
   ensure oxmysql
   ensure qbx_core        # or qb-core / es_extended
   ensure ns-truckerjob
   ```

3. Start the server. The tables are created automatically. When the console prints `Ready: v..., database schema v15`, the resource is ready.
4. Optional: check the map points in game (see below) and set the balance values.

`html/` is the built UI. `ui/` is its source and is not needed on the server.

## Setup

- **Map points.** All pickups, dealers and truck parks have default coordinates. Change them in the admin panel (**Locations → Edit in the world**, or `/truckeredit`), or stand on a spot and run `/truckercal`. Changes are saved in `data/calibration.json` and apply at once.
- **Road distances.** `data/distances.json` ships with measured road distances between the default locations. If you add or move a location, run `/truckerdistances` once. Without a measurement, the server uses the straight-line distance × `Config.Distance.fallbackFactor`.
- **Balance.** Pay, prices and costs in `config.lua` are starting values (marked `PLACEHOLDER`): tune them to your economy. Many can also be changed live in the admin panel.
- **Language.** `Config.Locale` (server and default) and `Config.Languages` (what players can pick).

## Database

- Migrations run by themselves. Current schema version: **15**.
- All tables start with `ns_truckerjob_`.
- The resource never drops or truncates a table.
- If a migration fails, the resource stays "not ready" and the console tells you why.
- Old tables from v1 of this script are only reported, never touched. If `ns_truckerjob_active_jobs` from v1 exists, rename or drop it by hand and restart.

## Commands

### Players

| Command | Default key | Action |
|---|---|---|
| `/trucker` | `F7` | Open or close the job tablet. |
| `/truckerconfirm` | `E` | Hitch the job trailer (at the pickup or after it came off), confirm the delivery (at the drop zone). |
| `/truckerslip` | `DELETE` | Close the delivery slip. |
| `/truckerinteract` | `E` | Use a dealer or a truck park. |

Convoy invites: `PAGEUP` accepts, `PAGEDOWN` declines. Players can rebind every key in the FiveM key settings.

### Admins

| Command | Action |
|---|---|
| `/truckeradmin` | Open the admin panel. |
| `/truckeredit` | Open the world editor (not in a vehicle). |
| `/truckercal <locationId> <field>` | Save your position as `trailerSpawn`, `dropZone` or `truckSpawn` of a location. |
| `/truckercal dealer <id> <field>` | The same for a dealer (`point`, `spawn`). |
| `/truckercal park <id> <field>` | The same for a truck park (`point`, `spawn1`, `spawn2`, ...). |
| `/truckertp <locationId> [field]` | Teleport to a saved point (`dealer` / `park` also work). |
| `/truckerdistances` | Measure the road distance of every location pair (max. 30 minutes). |
| `/truckersale <dealerId\|all>` | Start a dealer sale now (for testing). |

Command names are set in `config.lua`. Admin commands work only in game, not from the server console.

## Admin access

- `/truckeradmin` and `/truckeredit`: groups in `Config.Admin.groups` (default `admin`).
- `/truckercal`, `/truckertp`, `/truckerdistances`, `/truckersale`: always the group `admin`.

| Framework | Check |
|---|---|
| ESX | The player's group is the group name, or `superadmin`. |
| QBCore / Qbox | The ACE with the group name is allowed. |

QBCore / Qbox example in `server.cfg`:

```
add_ace group.admin admin allow
add_principal identifier.license:xxxxxxxxxxxxxxxx group.admin
```

The server checks the permission again on every admin request.

## Admin panel

| Tab | What you can do |
|---|---|
| Traits | Turn NPC traits on or off, change their values and weights. |
| Systems | Main switches: cargo damage, opportunities, rivals, NPC simulation, sales, skill reset. |
| Economy | Pay limits, tag rates, rental cut, deposit, fees, repair price, XP, NPC costs, timers, convoy size. |
| Employers and rivalries | Relation rates and rival pairs. |
| Player and company tools | Find players and companies: give XP, reset skills, set relations, add or remove traits, move vault money (with confirm and note), tune deck load positions with a live preview. |
| Live watch | Running jobs, NPCs on the road, open opportunities, recent NPC events. |
| History | Who changed what, when, old and new value. |
| Locations | Edit every pickup, dealer and truck park. **Edit in the world** opens a free camera with ghost markers and vehicles. |

- Panel values are saved in the database, override `config.lua` and apply at once. Each one has a "back to default" button.
- A job that is already running keeps its original offer.
- Only whitelisted keys inside their ranges are accepted.
- These need a `config.lua` edit and a restart: truck and dealer prices, cargo pay, trailer types, opportunity templates, the XP curve and skill tree, keys, command names, `Config.Admin`, new traits.

## Main config blocks

| Block | Sets |
|---|---|
| `Config.Locale`, `Config.Languages` | Server language, languages players can pick. |
| `Config.Admin` | Admin groups and tool limits. |
| `Config.Locations` | Pickups and drop-offs: coordinates, cargo made and accepted. |
| `Config.Cargo`, `Config.TrailerTypes` | Cargo pay and trailer models. |
| `Config.Deck`, `Config.DeckModels`, `Config.TrailerDecks` | Loads on open trailers. |
| `Config.Distance` | Distance fallback and measurement. |
| `Config.Trucks`, `Config.Dealers`, `Config.TruckParks` | Trucks, prices, dealers, parks. |
| `Config.Fleet`, `Config.Modify` | Truck limits, repair, upgrades. |
| `Config.Pay`, `Config.CargoDamage` | Pay tags, caps, deposit, rental cut, damage. |
| `Config.Skills`, `Config.Employers` | XP, skill tree, employers, rivals. |
| `Config.Offers`, `Config.Opportunities` | Personal offers and shared jobs. |
| `Config.Job` | Job flow: radii, cancel, cooldown, minimum delivery time. |
| `Config.Convoy` | Convoy size, timers, invite keys. |
| `Config.Company`, `Config.CrewMap` | Companies, ranking, member map. |
| `Config.Npc`, `Config.Traits`, `Config.Candidates`, `Config.Training` | NPC drivers, traits, hiring pool, training courses. |
| `Config.Sales`, `Config.Notifications` | Dealer sales and inbox. |
| `Config.Receipt`, `Config.Toast`, `Config.UI`, `Config.Tablet` | Delivery slip, notices, theme, tablet animation. |
| `Config.Integrations` | Key and fuel script choice. |

## Adding a deck load

1. **Model:** add a line to `Config.DeckModels`.
   - Vehicle: `mymodel = { name = 'My Car', length = 4.8, width = 2.1 },` (metres, seen from above; round up).
   - Prop: `prop_my_crate = { name = 'Crates', prop = true },` and add `deck.prop_my_crate` to the UI language files.
2. **Cargo:** put the model name in `deck = { models = { ... } }` of a cargo in `Config.Cargo`.
3. **Trailer:** the cargo's trailer type needs a model listed in `Config.TrailerDecks`.

The console warns about unknown names and loads that fit no trailer. Fine-tune the position in **Admin panel → Player and company tools → Deck loads**.

## Keys and fuel

When a truck is handed to a player, the resource gives keys and a full tank.

- **Keys** (first started one is used): `qbx_vehiclekeys`, `qb-vehiclekeys`, `wasabi_carlock`*, `qs-vehiclekeys`*, `mk_vehiclekeys`*. No key script: the truck is unlocked.
- **Fuel:** `ox_fuel`, `LegacyFuel`, `cdn-fuel`, `ps-fuel`, `qb-fuel`, `lj-fuel`*. No fuel script: the game's own tank is filled.
- Choose with `Config.Integrations = { keys = 'auto', fuel = 'auto' }`: `'auto'`, `'none'` or a script name.

\* Export names not verified. A wrong name only prints a warning.

## Connect your own scripts

Use the two bridge files. They are not escrowed.

- `bridge/vehicle_server.lua`: keys, owner, plate, fuel, spawn and remove hooks.
- `bridge/vehicle_client.lua`: client fuel and "player entered the truck".

Order for keys and fuel: **your bridge function** (return `true` to stop here) → the script in `Config.Integrations` → auto search → safe default.

### Server functions

| Function | Runs when | Return |
|---|---|---|
| `Bridge.GiveKeys(src, entity, plate, info)` | A truck is handed to a driver. | `true` = handled. |
| `Bridge.RemoveKeys(src, entity, plate, info)` | A truck is parked or removed (entity may be gone, use `plate`). | — |
| `Bridge.SetOwner(src, entity, plate, info)` | After keys and fuel. | — |
| `Bridge.SetFuel(src, entity, level, info)` | A truck is handed out (`level` 1-100). | `true` = handled. |
| `Bridge.MakePlate(default, info)` | A truck is bought. | A plate (1-8 chars, `A-Z 0-9` and spaces) or `nil`. |
| `Bridge.OnVehicleSpawned(src, entity, plate, info)` | First hand-over of a truck. | — |
| `Bridge.OnVehicleRemoved(src, entity, plate, info)` | A truck is removed. | — |

`info`: `kind` (`'rental'`, `'owned'`, `'company'`), `truckId`, `model`, `jobId`, `summoned`.

Functions run in their own thread. An error only prints a warning. `src` is `nil` when the driver is offline.

### Client functions

| Function | Action |
|---|---|
| `Bridge.SetFuel(vehicle, level)` | Fill the tank. Return `true` when done. |
| `Bridge.GetFuel(vehicle)` | Return the tank level (0-100) or `nil`. |
| `Bridge.OnEnterTruck(vehicle, plate)` | The player sat in one of these trucks. |

### Example

```lua
-- bridge/vehicle_server.lua
function Bridge.GiveKeys(src, entity, plate, info)
    exports.qbx_vehiclekeys:GiveKeys(src, entity, true)
    return true
end

-- bridge/vehicle_client.lua
function Bridge.SetFuel(vehicle, level)
    exports.LegacyFuel:SetFuel(vehicle, level)
    return true
end
```

### Exports (server)

| Export | Returns |
|---|---|
| `IsTruckerVehicle(entityOrPlate)` | `true` / `false` |
| `GetTruckInfo(plate)` | `info` + `plate`, `entity`, `netId`, `owner`, or `false` |
| `GetTruckOwner(plate)` | `{ type, id, driver }` or `false` (`type`: `'driver'`, `'company'`, `'rental'`) |

```lua
local info = exports['ns-truckerjob']:GetTruckInfo('NS 12345')
```

### Events (server, local only)

```lua
AddEventHandler('ns-truckerjob:vehicleSpawned', function(src, entity, plate, info) end)
AddEventHandler('ns-truckerjob:vehicleRemoved', function(src, entity, plate, info) end)
```

Use `AddEventHandler`, not `RegisterNetEvent`: clients must not trigger these.

## Known limits

- `PLACEHOLDER` values are starting values, not tuned to any economy. NPC drivers have no daily income cap.
- NPC simulation pauses while the server is empty (`Config.Npc.runWhenEmpty`).
- GTA has no refrigerated trailer: cold cargo uses closed box trailers.
- The company ranking refreshes once per `rankingCacheSeconds` (default 60).
- Names from `config.lua` (places, firms, trucks) are not translated.
- A company close and a truck take-out at the same moment can deadlock in the database. Money and trucks stay correct; the take-out fails and can be retried.
