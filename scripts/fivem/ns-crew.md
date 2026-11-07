# ns-crew

Crew system for FiveM: players create crews, invite members, climb a crew leaderboard and fight other crews in round-based **Crew vs Crew (CvC)** wars.

Cross-framework: **ESX / QBCore / Qbox**. The framework is detected at runtime (`qbx_core` → `qb-core` → `es_extended`); the script body never calls a framework directly.

## Features

**Crews**
- Create / delete a crew (name, 1–3 letter tag, banner). Deleting asks for the crew name typed exactly.
- Join policy per crew: **Private** (invite only), **Applications** (players apply, officers approve) or **Open** (join from the leaderboard).
- Invites with a 15 s popup, answered with **Y / N** or the on-screen buttons.
- Custom roles with nine switchable permissions (invite, kick, banner, Discord link, post announcements, delete announcements, CvC, roles, join policy). Default roles: leader, manager, member.
- Kick, leave, transfer leadership.
- Announcements: chat message plus a top-of-screen alert to every online member, with history.
- Crew map: live positions of crew members.
- Kill / death tracking per member and per crew, K/D computed on the fly.
- Leaderboard with podium, sortable by kills, deaths, K/D and wins.
- Crew mates get nametags and map blips (only in the same routing bucket). Friendly fire between crew mates is off by default.
- Personal display settings (nametags, distance, colour, blips) in the gear menu, saved per player.

**Crew vs Crew**
- A leader (or a role with the CvC permission) declares war on another crew: mode, best-of, team size, map and up to four weapon category bans.
- The invite waits in the other crew's **CvC › Invites** tab. Unanswered invites are declined automatically after 10 minutes.
- When accepted, both crews see a **Join war** bar in the crew menu for 5 minutes. A side that nobody joins cancels the war.
- Matches run in their own routing bucket. Players are restored (position, weapons) when they leave or the war ends.
- **Modes** — every mode is round-based with no respawn; wiping out the other crew always wins the round:
  - **Classic** — elimination only.
  - **Capture the flag** — bring the enemy flag to your base. A dropped flag returns home when its owners touch it, or after 15 s.
  - **Hold the zone** — hold the zone alone for a total of 30 s. Both crews inside pauses the timer. At time-out the crew that held longer wins.
- Before each round players pick one primary, one pistol and their attachments. Every weapon comes with maximum ammo, and a slot left alone gets a random allowed weapon.
- Dead players spectate their alive teammates until the round ends (**LMB / RMB** to switch).
- Forfeit rules for players who leave mid-war (`Config.CvC.punishFirstRoundQuit`).
- Head-to-head history and recent matches per crew.

## Requirements

| Resource | Required |
|---|---|
| `oxmysql` | Yes |
| `es_extended` / `qb-core` / `qbx_core` | One of them |
| `ox_lib` | Only on Qbox |
| OneSync | Yes (routing buckets and server-side positions for CvC) |

## Installation

1. Put the folder in `resources/` as **`ns-crew`**. The NUI calls `https://ns-crew/...`, so the folder name must not change.
2. Run `crew.sql` on your database. It creates seven tables: `ns_crew`, `ns_crew_roles`, `ns_crew_join_requests`, `ns_crew_ex_members`, `ns_crew_announcements`, `ns_crew_war_history`, `ns_crew_activity`. Updating from an older version? Nothing to run: on start the resource renames the old `crew*` tables to the `ns_` names.
3. Add to `server.cfg`, after `oxmysql` and your framework:
   ```
   ensure ns-crew
   ```
4. Before going live, set `Config.EnableDevCommands = false` in `config.lua`.

## Commands

| Command | Who | What |
|---|---|---|
| `/crew` | Everyone | Open the crew menu (name set by `Config.CrewCommand`) |
| `/crewnotify <message>` | Roles with the announcement permission | Announce to your crew |
| `/allcrewnotify <message>` | Admin / mod | System message to every crew |
| `/cvcleave` | Players in a war | Leave the current war (same rules as the Leave button) |
| `/cvcmaps` | Admin / mod | Open the CvC map editor |

**Dev commands** — registered only while `Config.EnableDevCommands = true`. The server ones are admin/mod only.

| Command | Side | What |
|---|---|---|
| `/crewaddfake [count]` | Server | Add fake members to your crew |
| `/crewclearfake` | Server | Remove fake members and requests |
| `/crewfakerequest [count]` | Server | Add fake join requests |
| `/crewfakestat <kills> [deaths]` | Server | Change your own stats |
| `/cvcsolo [mode] [map]` | Server | One-player test war against an empty crew (no stats); leave with `/cvcleave` |
| `/cvcmap <step>` | Client | Record a CvC map in-game (see below) |
| `/crewinvite`, `/cvcintro`, `/crewbanner`, `/cvcflagtest` | Client | UI and flag previews |
| `/cvcpos`, `/cvctp`, `/cvcdebug` | Client | Positions, teleport and debug output |

## Adding a CvC map

### With the map editor

1. Run `/cvcmaps`, then pick a map or press **New map**. A top-down camera opens: `W` `A` `S` `D` move it and the mouse wheel zooms.
2. Click the ground to place **Team A spawn**, then **Team B spawn**.
3. Type a name and press **Save map**. The map is playable at once, with no restart.

Two spawns are all a map needs: the teams face each other, the center goes to the middle of the spawns, the lobby to the center, and the play area grows to fit every point. To change a point, click its row on the panel and click the ground. Flags and the zone show up when their mode is on.

- A click lands on the top surface, so a point under a roof must be added in `Config.CvC.maps`.
- Maps are stored in `cvc_maps.json` in the resource and override `Config.CvC.maps` entries with the same key. **Reset** on an edited map brings the config version back.
- A map that a war or an invite is using cannot be changed.

### In the config

Maps live in `Config.CvC.maps`. Record one in-game with `/cvcmap` (dev commands on):

1. Stand on each spot and run: `/cvcmap lobby`, `/cvcmap center`, `/cvcmap attacker`, `/cvcmap defender`.
2. Optional: `/cvcmap flagA`, `/cvcmap flagD` (CTF bases), `/cvcmap zone` (zone centre), `/cvcmap zoneradius 10`.
3. `/cvcmap name docks B Elysian Docks` — map key, card letter, display name.
4. `/cvcmap print` — copy the block from F8 into `Config.CvC.maps`.

The play area radius defaults to the farthest point + 20 m. The lobby must sit inside it: players outside the radius are sent back to the lobby. Without `flags` the spawn points are the CTF bases; without `zone` the map centre is used.

## Configuration

Everything is in `config.lua`:

| Section | What it controls |
|---|---|
| `Config.Locale` | `'en'` or `'tr'` (strings in `Config.Strings`) |
| `Config.DefaultDiscordInvite` | Discord button target for crews without their own link (`''` hides it) |
| `Config.FriendlyFireEnabled` | Crew mates can hurt each other |
| `Config.Permissions`, `Config.DefaultRoles` | Permission list and the roles every new crew starts with |
| `Config.Leaderboard` | Crews listed, crews per page, K/D shown green from `goodKD` |
| `Config.Announce` | Announcement spam guard: `maxActive` per crew, `expireHours`, per-player `cooldown` |
| `Config.Avatars` | Profile pictures: `'both'` (player picks), `'steam'`, `'discord'` (forced) or `false`. Discord needs `set ns_crew_discord_token "..."` in `server.cfg` |
| `Config.InviteKeys` | Accept / decline keys for crew and war invites: `{ control = <GTA control index>, label = "Y" }`. Defaults Y (246) / N (306) |
| `Config.Server` | Name/tag limits, member limit (`maxMember`, applies to every crew), default banner, join policy, save interval, banner validation |
| `Config.Client` | Nametag and blip defaults. `NametagColor` is a HUD colour index (1 white, 12 yellow, 6 red, 18 green, 9 blue, 21 purple, 15 orange, 24 pink). `DistantBlipPollSec` (default 2, 1–60) is how often blips of members out of streaming range move |
| `Config.AmbulanceCompat` | Ambulance jobs to reset when CvC revives a player |
| `Config.CvC` | Modes, timings, best-of and team sizes, weapon categories, loadout, maps |
| `Config.CvC.defaultPlayers` | Players per side preselected on the war setup screen (1 = 1v1) |
| `Config.CvC.pickTime` | Seconds players stand frozen at spawn picking one primary (classes that are not banned) and one pistol; the pick carries over, a slot left alone gets a random allowed weapon each round. `0` gives `defaultLoadout` instead |
| `Config.CvC.sounds` | Game sounds for a received crew or war invite and an accepted war invite (`false` = silent) |
| `Config.CvC.pistols` | Pistols on the pick screen (a class the war setup can ban like any other). Every picked weapon gets the game's maximum ammo. Players also pick attachments there; each weapon keeps its attachments for the next time (saved on the client) |
| `Config.CvC.blockInventory` | No inventory and no death drop while in a war: ns-inventory through its game-mode exports, ox_inventory / qb-inventory through the `invBusy` / `inv_busy` statebags. Other inventories hook the war events (see [Integration](#integration-inventory-and-loot)) |

## Exports

**Server**

```lua
exports['ns-crew']:GetAllCrew()                -- { [lowername] = crew, ... }
exports['ns-crew']:isInCrew(identifier)        -- crew name or nil
exports['ns-crew']:GetCrewData(name)           -- crew or nil
exports['ns-crew']:GetCrewAndRank(identifier)  -- { name, rank, banner } or nil
exports['ns-crew']:GetCrewTag(identifier)      -- tag or nil
exports['ns-crew']:GetCrewPlayers(identifier)  -- { src, ... } online members
exports['ns-crew']:getBanner(name)             -- banner path / URL
exports['ns-crew']:AreCrewMates(a, b)          -- boolean; a/b = server id or identifier
exports['ns-crew']:IsInWar(source)             -- boolean; skip these players in a lootbag / drop script
```

**Client**

```lua
exports['ns-crew']:openCrew(true)       -- open / close the menu
exports['ns-crew']:GetPlayerCrew()      -- local player's crew data
exports['ns-crew']:IsCrewMate(serverId) -- boolean
```

## Events

Server-side, **local only** (`TriggerEvent` from another server script — they are not net events, so clients cannot fire them):

```lua
TriggerEvent('ns-crew:server:updateNickname', identifier, newName)  -- character renamed
TriggerEvent('ns-crew:server:WipePlayer', identifier)               -- remove a player from crews
```

Client event fired on a player whose crew membership changes (create, join, kick, leave, delete):

```lua
RegisterNetEvent('ns-crew:client:crewUpdated', function() --[[ refresh your UI ]] end)
```

### Integration: inventory and loot

ns-inventory, ox_inventory, qs-inventory and qb-inventory with qb-weapons need no setup: ns-crew uses their own switches, so they leave war weapons alone and the player's items stay as they are. If any other inventory strips a picked weapon, ns-crew gives it back within 500 ms with the same rounds; add `ns-crew:server:warWeapons` below to stop the strip itself. Anticheats need their own exception: see [Integration: anticheat](#integration-anticheat).

For any other inventory, lootbag or death-drop script, ns-crew fires these events during a war; the script listens and blocks itself.

```lua
-- server: player entered (true) or left (false) a war.
-- false comes ~12 s after the war ends, after the last revive.
AddEventHandler('ns-crew:server:warState', function(src, inWar)
    -- e.g. block the inventory, keep items on death, no lootbag
end)

-- server: the weapons ns-crew gave this round (nil = none any more).
-- Do not strip these as "not in the inventory".
AddEventHandler('ns-crew:server:warWeapons', function(src, primary, sidearm) end)

-- client: same war state on the player's own client
AddEventHandler('ns-crew:client:warState', function(inWar) end)
```

You can also poll: `exports['ns-crew']:IsInWar(source)` (server) or `LocalPlayer.state.nsCrewWar` (client).

### Integration: anticheat

War weapons are ped weapons given with `GiveWeaponToPed`; they are not inventory items. An anticheat that removes weapons a player does not own, such as `qb-anticheat`, also removes war weapons. ns-crew gives them back within 500 ms, so the weapon disappears and returns during the round, and the anticheat can log or punish the player each time.

Make the anticheat skip players in a war:

```lua
-- client side of the anticheat, at the top of its weapon check
if LocalPlayer.state.nsCrewWar then return end

-- server side of the anticheat
if GetResourceState('ns-crew') == 'started' and exports['ns-crew']:IsInWar(source) then return end
```

Inside a loop, use `goto continue` or an `if` block in place of `return`, so the check still runs for other players and later ticks.

- `LocalPlayer.state.nsCrewWar` is set only while `Config.CvC.blockInventory` is on (the default). With it off, use the `ns-crew:client:warState` event.
- Add the same skip to teleport, revive and health checks: a war teleports and revives players every round.
- Check weapon blacklists too. `Config.BlacklistedWeapons` in `qb-smallresources` lists `WEAPON_RAILGUN` by default, which is in the `heavy` class. Remove it there, or remove it from `Config.CvC.weaponCategories`.
- For a closed anticheat, ask its author for an exception and give them the checks above.

## Security notes

- Kill stats take the killer from `source`, never from client data.
- Every crew action re-checks the player's crew and permission on the server.
- Invites expire on the server even if the client never answers.
- Banners are validated (extension whitelist, length limit, no markup characters).
- Crew deletion needs the leader check; typing the crew name only guards against mistakes.
- Announcements have a per-player cooldown and a per-crew cap; posts are serialised per crew so the cap holds when members post at once.
- The Discord bot token is read from a server convar, never from `config.lua` (which is sent to clients).
- CvC objectives (flags, zone) are decided on the server from server-side positions.
