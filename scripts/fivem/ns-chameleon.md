# ns-chameleon

**Paintable hide-and-seek for FiveM.** Hiders paint their own bodies to match the wall they are about to stand against, strike a pose and stop breathing. Seekers hunt them with a stun shotgun before the clock runs out.

It is not a prop-hunt reskin. A hider is a blank white body and a brush: what makes you invisible is the paint you put on yourself and where you choose to put that body. A good hider is a decision, not a lucky prop.

Cross-framework — **ESX · QBCore · Qbox · vRP 1.x · standalone**. The framework is detected at runtime; there is nothing to tell it.

---

## Install

1. Drop `ns-chameleon` into your `resources` folder.
2. Add `ensure ns-chameleon` to `server.cfg`.
3. If you run an inventory, register the seeker's weapon as an item — see **Seeker weapon** below. Three lines, and the one setup step this resource has.

**No hard dependencies.** It declares none, so it cannot refuse to start because something is missing. `ox_lib` is used for nicer notifications if you happen to have it.

Docs: <https://fivem.nativescripts.com/docs>

---

## How a round plays

**Lobby.** `/cham` opens the room browser — join one or open your own. The host sets the arena, hide time, hunt time, rounds and how many seekers; everyone in the room watches those rows update live. You pick your side by taking an empty seat. Several rooms can play at once, and a room survives its match for the rematch.

A room opens with 8 hider seats and 2 seekers, and the host can take it to **16 hiders and 6 seekers** — 22 players in one arena. Seats appear as they are added and the hider list starts scrolling once it is as tall as the settings beside it, so a full room reads as easily as an empty one.

A clown NPC in Legion Square opens the same panel, for players who never read commands.

**Hide.** Hiders paint, pose and place themselves. Seekers spend this phase in a private copy of the arena — same map, nobody in it — so they can learn the ground and get their weapon in hand without watching anyone choose a hiding place. When the hunt starts they are dropped on the seeker spawn.

Seekers paint in that copy too, on their own body and their own atlas. It is warpaint rather than camouflage — nothing about a seeker is meant to be hidden — and it is the reason the hide phase is not dead time for them. The paint stays on for the hunt; the brush does not.

**Hunt.** Seekers sweep the arena. A find is decided by the stun shotgun's damage event, so falling, traffic or an unrelated gun never gives a hider away. Once per interval every living hider whistles at the same time, which is the round's answer to sitting in one corner all match.

**Results.** The board ranks hiders by how long they lasted and seekers by how many they found. Close it with ✕ or ESC, or let it time out. After the last round of a series the lobby panel opens again.

---

## Controls

| Key | What it does |
|---|---|
| `H` | **Paint.** Orbit camera and a free cursor on your own body. Pick a colour, size the brush, draw. `/paint <hex>` fills the whole body at once. Hiders paint for as long as they are alive; seekers only during the hide phase. |
| `G` | **Place.** Aim a ghost of your painted body at a surface — scroll spins it, the marker turns green when a body fits and red when it does not, LMB seats you. Backspace abandons the aim. Once seated, fine-tune with WASD or the arrow keys, scroll for up and down, Q/E to turn on the spot. Space breaks free. |
| `J` (hold) | **Pose wheel.** Hold, move to a slice, release to strike it. A pose is *carried*: the clip owns your whole body and WASD pushes you along underneath it, so you go looking for a hiding place already wearing the disguise. Release over the centre to drop it. |
| `F` | **Look around** without moving. Your body stays exactly where you seated it — and stays shootable — while the camera roams up to 25 m. Hiders only. |
| `Y` | **Names.** Squad nametags so seekers can split the map up. Seekers only; hiders never get tags. |
| `N` | **Whistle.** Every seeker in range hears it in 3D from where you stand, and you wave as you do it. The one control that deliberately gives you away — bait someone into the wrong room, or gamble. |
| `/chamleave` | Leave the room or the round you are in. |

All of these are rebindable in **FiveM → Settings → Key Bindings**, and the on-screen key rail shows the key the player actually bound.

> Changing a key default in `config.lua` only reaches players who have never run the resource. FiveM applies a mapping default once; after that the player's own binding wins, and they have to clear it in that settings screen.

The HUD sits on the world rather than in panels, so it never covers the room you are hiding in: living players and the clock at the top, remaining hiders bottom right, your live keys down the right edge, and context prompts at the bottom that appear only when the current stance has something to say.

---

## Game modes

The host picks one per room.

| Mode | Being found means |
|---|---|
| **Basic** | You are **out**, and drop into a chase camera on the seekers (A/D cycles). The squad stays the size the host set, so the hunt is as hard for the last hider as it was for the first. |
| **Infection** | You **switch sides** and join the hunt. The squad grows all round, so the game accelerates and ends itself. |

Either way the round ends when the hiders run out or the clock does. Seekers cannot hurt each other, and a player who is already out is taken off the map entirely — nobody can shoot a spectator.

---

## What you can configure

`config.lua` holds the settings: match lengths, arenas, player caps, keys, the lobby NPC, the whistle, the confetti, and the language.

`config.lua` and `locales/` ship **unencrypted**, because they are the parts you are meant to edit. Everything else — model names, paint pipeline constants, placement collision maths, the pose table — is the machinery the game mode is built out of rather than settings, and is not exposed: changing one of those numbers does not tune a round, it breaks it.

The one value that can genuinely differ per server is the routing bucket, and that is in `config.lua` for exactly that reason. Change it only if `1377` is already taken on your server.

**Turn `Config.Debug` off on a live server.** It ships off. On, it opens the dev commands to everyone.

### Arenas

Six ship, all walked in game: Legion Square, Pier Funfair, Tin Town, Plane Graveyard, The Ranch and Studio Backlot.

Adding your own is one entry in `Config.Maps` — fly to the spot, note the coordinates, and pick a radius. That radius is the single most important number on the entry: it is the visible dome, the walk limit *and* the limit on where a body may be placed, all at once. Walk the edge once before settling on it.

`image` is optional. Drop a screenshot in `nui/maps/` and name it on the entry — name and extension have to match exactly. Any arena without one falls back to a drawn plate, so screenshots can be added an arena at a time.

The host changes arena from the same plate two ways: the arrows step through the list, and clicking it opens the full picker.

### Language

Ships in eight: **English, Deutsch, Français, Español, Türkçe, Português (BR), Polski, Italiano**. Pick one with `Config.Locale` — it applies to the whole server, not per player.

Adding another is one file: copy `locales/en.lua`, translate the values, save it as `locales/<code>.lua`. The manifest globs the folder, so there is nothing to register, and any key you leave out falls back to the English line — a half-finished translation degrades line by line instead of showing blanks.

Three things must survive an edit, and all three fail quietly:

- **`%s`** — callers pass positional arguments. Keep the same count and the same order, or the line prints a literal `%s`.
- **`{tokens}`** — a few `ui.*` strings are filled in by the interface, not by Lua. A missing `{hider}` just never shows a name.
- **Apostrophes** — the strings are single-quoted, so a straight `'` ends the string early and **the whole language file fails to load**. Write `’` instead, or escape it as `\'`.

### Tattoos

A round swaps your model, and a new ped carries no tattoos — they are ped *decorations*, and the swap discards them.

They come back. Before the swap the resource reads the decorations off your ped with `GET_PED_DECORATIONS` and reapplies them once your appearance is restored, so it does not matter which tattoo script put them there — it never asks one. Nothing to configure.

If something else on your server wants to know a body has finished restoring, it can listen for `ns-chameleon:client:appearanceRestored`, which is fired with the ped handle.

### Confetti

Every seeker shot pops confetti, and where it pops says whether it found anybody: a big burst on the hider for a confirmed hit, a smaller one wherever the pellets actually landed for a miss. `Config.HitFx` and `Config.MissFx` set the effect, its size and the follow-up bursts that turn one puff into an explosion. Set either to `nil` to switch that half off.

> The names must be **real** particle names. A made-up one does not fail quietly — it can crash the client when the pause menu is opened. The shipped pair is a stock GTA effect.

GTA's own blood decals are cleared during a round on every client, because a red splatter lands on the exact surface the paint replaced and would mark a hider for the rest of the round.

---

## Seeker weapon

The seeker hunts with **`WEAPON_NS_SHOTGUN`**, a custom stungun-class shotgun shipped inside this resource (`weapons/*.meta` + `stream/w_sg_ns_shotgun.*`). There is no separate weapon resource to install. It is flagged `NonLethal`, so a hit stuns instead of killing, and its ammo type recharges on its own.

The gun itself is not a setting. A find is decided by the damage event's weapon hash, and a stungun-class weapon only produces that event at full damage — retune it and hits round to nothing, the engine drops the attribution, and nobody is ever found. So the weapon and its damage are fixed.

### ⚠️ If you run an inventory, register it as an item

`WEAPON_NS_SHOTGUN` is a custom weapon that lives inside this resource, so your inventory has never heard of it. Add it to your item list once and the gun is handed over the same legitimate way every other weapon on your server is.

**ox_inventory** — `ox_inventory/data/weapons.lua`, inside `Weapons = { … }`:

```lua
['WEAPON_NS_SHOTGUN'] = {
    label = 'NS Stun Shotgun',
    weight = 3400,
    durability = 0.1,
},
```

No `ammoname` on purpose: the weapon uses `AMMO_STUNGUN`, which recharges itself, so there is no ammo item to consume. This is exactly how ox ships `WEAPON_STUNGUN`.

**qb-core / qbx / qs-inventory** — two files, because the qb family splits the weapon from the item. In `qb-core/shared/weapons.lua`:

```lua
[`weapon_ns_shotgun`] = { name = 'weapon_ns_shotgun', label = 'NS Stun Shotgun', weapontype = 'Shotgun', ammotype = 'AMMO_STUNGUN', damagereason = 'Tagged' },
```

and in `qb-core/shared/items.lua`:

```lua
weapon_ns_shotgun = { name = 'weapon_ns_shotgun', label = 'NS Stun Shotgun', weight = 1000, type = 'weapon', ammotype = nil, image = 'weapon_ns_shotgun.png', unique = true, useable = false, description = 'Tags hiders without hurting them' },
```

The **lowercase** key here against ox's uppercase one is the qb convention, not a typo — the resource picks the right case for your backend on its own.

**ESX without ox_inventory** — `es_extended/shared/config/weapons.lua`, inside `Config.Weapons`:

```lua
{ name = 'WEAPON_NS_SHOTGUN', label = 'NS Stun Shotgun', tints = Config.DefaultWeaponTints, components = {} },
```

**Standalone servers** have nothing to register. The resource detects that and arms the ped directly.

### If you skip it

Nothing breaks and no round is lost. The first refusal prints one console line naming the item and the fix, then the server arms the ped instead for the rest of its runtime — seekers stay armed. Set `Config.SeekerWeaponMode = 'ped'` if you would rather choose that deliberately and never see the line.

### Who arms the seeker

`Config.SeekerWeaponMode` decides it, and `auto` — the default — asks which inventory is running and picks for you.

| Mode | What happens | For |
|---|---|---|
| **`item`** | The server hands over the item and stops. Your inventory equips it and owns the ped's weapon state. | Any server running an inventory |
| **`ped`** | The weapon goes straight to the ped, with a watchdog that re-issues it if it vanishes. | Standalone, or where the ped is the source of truth |
| **`both`** | Item *and* ped. | Only if your inventory ignores weapons it did not issue rather than removing them |

The mode is resolved once on the server and sent to the client, so the two sides can never disagree about who owns the ped's weapons.

### Your players' own weapons

**A round takes your weapons away, and it is unavoidable.** A round swaps your model, and swapping a model destroys the ped and builds a new one — anything it was holding goes with it. Every hide-and-seek resource that changes your appearance behaves this way.

On an **inventory server** they come back the way they always do: they are items, your inventory still has them, and it re-equips them after the round.

On a **standalone server** there is no inventory to remember, so the resource records the weapons and ammo counts before the swap and hands them back afterwards. **Attachments and tints are not restored** — a suppressor or a skin will be missing, because enumerating components per weapon means guessing, and a wrong guess is worse than a clean gun.

---

## Troubleshooting

**Seekers keep losing the gun.** The console line names which layer failed. The watchdog counts *consecutive* misses: one or two are ordinary and handled invisibly, but five in a row mean something is removing the weapon deliberately, so it backs off and reports rather than re-issuing a weapon every second — which is indistinguishable from the exploit an anticheat looks for.

**A key does nothing.** Two key bindings on the same key both fire and neither can suppress the other. Check whether an inventory or another resource binds the same key, and move one of them.

**The whistle sounds flat.** Replace `nui/sounds/whistle.mp3` with your own, but keep it **mono**, about a second long, trimmed tight at both ends and free of reverb. A stereo file carries its own left/right image and fights the 3D panner, which makes the direction unreadable — and locating that sound by ear is the mechanic.

---

## Dev commands

All gated behind `Config.Debug`, which ships off.

**`/chamsolo [hider|seeker]`** — start a round on your own, skipping the two-player minimum, to walk the whole loop: paint, place, pose, freecam, HUD and results.

**`/chamfake results|browser|lobby|close [n]`** — put a screen on the page with invented data, for looking at a layout without a match. The fabrications are deliberately awkward — overflowing names, one-character names, five-digit scores — because three tidy rows prove nothing.

**`/chamalign`** — from inside the painter, asks whether the statue the painter is showing matches the pose the body is actually in, and draws both origins so a wrong one reads as displaced or turned at a glance.

**`/chamwhistle [metres] [bearing]`** — play a whistle from a given direction, to hear the spatial audio without a second player.

**`/chamremote [seconds]`** — run it on the screen that is *watching* somebody. It picks the nearest other player and measures their body from your machine: which of our clips they are wearing, whether they are pinned in place, which way their skeleton faces against their entity, and how far each bone drifts over the window. A hider on a wall should read zero everywhere. It exists because a body can only look wrong from a camera its owner is not sitting behind.

**`/chamwho`** — name every ped and object within three metres, with model, visibility and alpha. The answer to "there is something else standing where I am standing", which is what a stray statue or an unhidden body looks like from the outside.
