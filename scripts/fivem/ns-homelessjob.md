# ns-homelessjob

A homeless job that doubles as roleplay: put down a ready-made setup,
stand by your cup, and let pedestrians walk over and pay you. Write a real cardboard
sign that other players can read from across the street, level up into bigger
camps, and let other players stop and give you money.

Cross-framework: **ESX / QBCore / Qbox / vRP** — the framework is detected at
runtime, nothing to configure.

## How it plays

1. **Pick a setup.** **F7** (or `/hj`) opens the street panel. Three tabs
   down its left side: **Corner** (what you are doing right now), **Sign**
   (your cardboard) and **Setups** (what the next level asks for and what it
   unlocks). Every setup is a ready-made layout — eight of them, from a
   flattened box and a paper cup at level 1, through a cardboard shelter, a
   shopping cart home, a curbside couch, a tarp tent and a stove camp, up to
   a whole shantytown with a fire and the radio on at level 10.
2. **Put it down anywhere.** The whole layout appears as a see-through ghost
   with the gizmo on its main prop: drag the arrows to move it, press **R**
   and drag the ring to turn it to face the street, then press **Enter** to
   drop it (**Backspace** cancels and takes you back to the list). Your
   character then walks over and puts it together, kneeling and working on
   the ground for four seconds (`Config.Placement.Build`; **Backspace**
   stops). Packing up later (**Pack up** or `/hjpack`) is the same work the
   other way round: walk over, kneel, three seconds (`Config.Placement.Pack`).
   Two
   beggars cannot set up right next to each other, and server owners can
   block areas (police stations, hospitals).
3. **Stand by your cup.** There is no sitting: you work standing next to your
   setup (within `Config.Visits.WorkRadius`). Life on the street is not
   clean: while your setup is down, flies swarm around you like around the
   rubbish bags (`Config.Flies`). Pedestrians walk over at
   intervals, bend down and drop money in your cup — it goes straight to your
   pocket. Walk more than 15 m away and you get a warning; not back within
   10 seconds, and the setup is packed away (`Config.Placement`).
4. **Write a sign (optional).** Right after you set up, the game's help box
   says *Press F7 to write on the cardboard*, and **F7** opens the **Sign**
   tab: the board preview on top and five tools under it — **Write** (text
   pieces, up to four: type each one, drag it anywhere on the board, turn it
   with its round handle — **Shift** snaps to 15° — and pick its size and
   colour), **Draw** (click and drag on the board with a marker:
   nine colours, thin or thick, undo, clear), **Stickers** (add an arrow,
   heart, dollar… and drag it where you want it, small / medium / large, up
   to four), **Board** (cardboard, pizza box or plywood) and **Ready** (six
   ready-made signs such as *Anything helps* or *Will work for food*: one
   click fills the board, then change anything you like). Then **Write on
   the cardboard**, and your character writes it with a pencil. Text and
   drawing appear on the board only once you finish writing, exactly where
   you drew them, for everyone around. The writing faces the people reading
   it; the back of the board is plain cardboard.
5. **Hold it up or lean it.** The **Corner** tab shows what this session
   paid (with a tally of who gave), today's cap, your sign and your all-time
   numbers. **Hold the sign up** (or `/hjhold`) to hold it yourself. Whenever
   you put it down (**E** while holding it, the same button, or `/hjhold`),
   the gizmo opens on it: drag it where it should lean and press **Enter**
   (**Backspace** keeps the old spot). **Move sign** does the same any time;
   the spot is kept until you pack up. A written sign leaning against your
   setup brings people by more often; held up, they come even faster. While
   you work, the card at the bottom of the screen shows your level, what this
   session paid, what your sign says and how much more often people stop
   because of it. The panel stays open while you hold the sign up, put it
   down (it goes back to where it leaned) or throw it away.
6. **Level up.** Ten levels. Everyone who pays you — pedestrian or player —
   every dollar they give and every minute you spend working count. Each level pays a little more per pedestrian and
   brings them by a little more often (up to +40% pay and 27% faster at level
   10), and most levels unlock a new setup. The **Setups** tab shows what the
   next level asks for and what it adds.
7. **Or just carry a sign.** Not everyone wants a camp: **Just carry a
   sign** in the menu gives you a blank cardboard with no setup. Write,
   draw and put stickers on it the same way, hold it up and walk around with
   it -- everyone around sees it. Pedestrians never stop for a sign alone and
   the minutes do not count, but other players can give you money while you
   hold it up. Set up a spot any time and the cardboard comes along.
   **Throw it away** gets rid of it.
8. **Other players can give too.** Anyone who walks up to you, to your cup, or
   to you holding up a cardboard gets the game's own help box (**Press E to
   give money**) and picks an amount — real roleplay, real money. Everyone
   around sees the hand-over: the donor turns to you and holds out a note, and
   you take it when your hands are free; into a cup they bend down and drop
   it.
9. **The police can clear you.** An on-duty officer (`Config.Police.Jobs`)
   next to a setup gets a second line under the give-money one: hold **G** for
   three seconds — the officer kneels down and works on the ground meanwhile
   (`Config.Police.Anims`) — and the setup is gone; its owner is told. "On duty" is read
   from QBCore / Qbox (`job.onduty`); on ESX an off-duty job name such as
   `offpolice` is simply not in the list; on ND, ox and vRP every officer of a
   listed job counts.
10. **No AFK farming.** No key or mouse input for five minutes and nobody
   stops by: no money, no minutes. Touch anything and it picks up again
   (`Config.Afk`). The check runs on the player's own game, so a macro or a
   modified client can get around it; `Config.Economy.DailyCap` still limits
   what one character takes in a day.

## Requirements

- **OneSync** — every distance check (placing, getting paid, donating) runs on
  the server's view of the player. Without OneSync the server cannot see
  anyone, and the script refuses to pay rather than trust the client.
- **One supported framework** for the money: ESX / QBCore / Qbox / vRP. Without
  one the script still starts and setups can be placed, but no cash can be
  paid or donated (a warning is printed on start).
- (optional) `oxmysql` — persists levels across restarts
- (optional) `ox_lib` — only for notifications, and only with
  `Config.NotifyStyle = 'framework'`; by default the script shows its own

## Installation

1. Drop `ns-homelessjob` into your server `resources` folder.
2. Add `ensure ns-homelessjob` to your `server.cfg`.
3. Start the server and run `/hjprobe` in-game once — it reports which prop
   models, animations and pedestrian models actually exist on your build, so you
   can fix `config.lua` from measurements instead of guesses.

The admin commands (`/hjedit`, `/hjclear`, `/hjlevel`) need
the `admin` ACE. On QBCore / Qbox that is not automatic for `group.admin`:
add `add_ace group.admin admin allow` to `server.cfg`. They are also
restricted commands, so on ESX give the group `command.<name>` (or `command`).

No SQL file to import: the `ns_homelessjob_progress` table is created automatically
when a database is available. Without one the script still runs; levels just do
not survive a restart.

## Commands

Everything is a command. **F7** also opens the panel: it is read as a game
control, not registered as a key binding, so `Config.MenuKey` changes it for
every player at once (`Enabled = false` turns the key off; `Control` is the
GTA control ID). The only key bindings
are the gizmo's seven (see **Gizmo keys** below), and they do nothing unless a
gizmo is on screen. The three player commands can be renamed in
`Config.Commands`.

| Command | Who | What |
|---|---|---|
| `/hj` or **F7** | everyone | Open the street panel |
| `/hjhold` | everyone | Hold the sign up / put it down |
| `/hjpack` | everyone | Pack your setup away |
| `/hjprobe` | everyone | Report which configured assets exist on this build |
| `/hjedit` | admin | Lay a setup out with the mouse gizmo (cup, props, cardboard), then save for everyone |
| `/hjclear [id]` | admin | Clear one setup, or all of them |
| `/hjlevel <id> <level>` | admin | Set a player to a level, up or down |

`<id>` is the player's server ID. The admin commands also work from the server
console, without the slash (`hjlevel 5 3`); in game they need the `admin` ACE
(see **Installation**).

### Changing a player's level

A level is not stored: it is worked out from three lifetime numbers per
character — `donations` (everyone who paid you), `earned` (what they paid) and
`minutes` (time spent working a setup).

- **Player online:** `/hjlevel <id> <level>` sets those three numbers to exactly
  what that level asks for in `Config.Levels`, so it raises or lowers the
  player. A level above the top one sets the top level; `/hjlevel <id> 1`
  starts the player over from zero.
- **Player offline:** with a database the numbers
  live in the `ns_homelessjob_progress` table, one row per character
  (`identifier` = `citizenid` on QBCore / Qbox, the character identifier on
  ESX). Set them to the `requires` of the level you want from
  `Config.Levels` — level 3 by default:

  ```sql
  UPDATE ns_homelessjob_progress
     SET donations = 60, earned = 400, minutes = 75
   WHERE identifier = 'ABC12345';
  ```

  Only edit the table while that player is offline: an online player's numbers
  are kept in memory and written back every 30 seconds, over your change.
  Without a database the numbers only live until the next restart.

## Configuration

`config.lua` holds only what a server owner sets, in the order you are likely
to touch it:

- **`Config.Commands`** / **`Config.MenuKey`** — the three player commands
  and the key that opens the street panel: `Enabled` (`false` = the command
  only) and `Control` (GTA control ID, default `168` = F7).
- **`Config.NotifyStyle`** — `'ns'` (default): the script's own notifications,
  top right in the panel's look;
  `'framework'`: ox_lib, then your framework's notify, then the game's feed.
  `Config.NotifyTime` is how long one stays (ms; errors 1.5 s longer). A
  `Config.Notify` function of your own replaces both.
- **`Config.Access`** — optionally make it a job (without it the menu does
  not open) and / or require an item (without it you cannot set up or take a
  cardboard; `ConsumeItem` uses one up for every setup put down). Both are
  checked the moment you open the menu or press **Set up**, not after
  placing.
- **`Config.Levels`** — what each level requires (`donations`,
  `earned` and `minutes`; a level is reached once every number is met) and
  what it adds: `pay` (% more cash from each pedestrian) and `faster` (% less
  time between two). Add or remove levels freely.
- **`Config.Setups`** — the ready-made layouts. Each has a `level`, a
  `container` (the cup pedestrians walk up to), a `signSpot`, dressing
  `props`, how often people come (`visits`, seconds) and what one person
  pays (`amount`). Offsets are local to the spot you stand on: x = right,
  y = forward, z = up — easiest set in-game with `/hjedit` (below). Every
  model is a **candidate list**: the first one that exists on your build
  wins.
- **`Config.Placement`** — how far you can place, the minimum distance
  between two beggars, the wait after packing up, how far you may walk from
  your setup (`LeashDistance`, 15 m) and for how long (`AbandonSeconds`, 10 s)
  before it is packed away, and blocked zones (police stations, hospitals).
- **`Config.Visits`** — how much a written or held sign speeds people up, how
  close to your setup you must stand to be paid, and the pedestrian models.
- **`Config.Economy`** — the account money goes to and the daily cap on money
  from pedestrians.
- **`Config.Donate`** — player donations on or off, the amount limits, the
  presets in the picker and the cooldown between two gifts.
- **`Config.Sign`** — the longest text, drawing on or off, players moving
  their own sign on or off, and `Dui.BothSides` (default `false`: the back of
  the board is plain material, `html/img/back-<style>.png`).
- **`Config.Flies`** — the fly swarm around everyone with a setup, on or off.
- **`Config.Afk`** — minutes without input before nobody stops by, or off.
- **`Config.Police`** — which jobs can clear a setup, on duty only or not,
  the key (GTA control ID, default `47` = G), how long to hold it and how
  close to stand.
- **`Config.Locale`** — the language: `en`, `tr`, `de`, `fr`, `es`, `pt`,
  `pl` or `it`. Every text a player reads, and the setup and level names,
  live in `locales/<code>.lua` (shipped readable): edit a text there, or copy
  `en.lua` to a new code to add a language. A key a file lacks falls back to
  English. Keep every `%s` and `~INPUT_...~` token, and write `’` instead of
  a plain apostrophe inside a text.

### Fine tuning

Everything else — the animations, where the cardboard sits in each hand, the
pencil, the hand-over, marker colours and ink limits, sticker sizes, render
distances and the timings that keep payouts honest — lives in
`shared/defaults.lua` (shipped readable). You rarely need it. To change one
of those values, copy the key into `config.lua` under the same name; yours
wins and everything you leave out keeps its default. Lists (models, colours,
animations) are taken whole.

The ready-made signs of the **Ready** tab are `Config.Sign.Templates` in the
same file: where each text piece sits, how big, its colour, plus stickers and
marker strokes, on the same 0–255 board grid as a player's sign. The words come
from `templates` in `locales/<code>.lua`, one string per piece (`\n` starts a
new line); a `t = '...'` on a piece in the config is used instead. To use your
own, copy `Config.Sign.Templates` into `config.lua` and edit it; an empty list
hides the tab. A ready-made sign is checked like any other when it is written.

If a prop you add floats above the ground (a tent, a fire pit), its collision
reaches below the visible model and the game stands it on the collision. Add
the model to `Config.Placement.Sink` with how many metres to lower it.

### Tuning the layout in-game

Every model and animation sits a little differently, so the layout is tuned in
the game with the mouse, not guessed in a file:

1. Put the setup you want to tune down and run `/hjedit` next to it.
2. **Left / Right** picks a piece (a yellow arrow marks it): the cup, the
   cardboard (propped up, held up, being written on) and every prop. For the
   held and written cardboard your own character takes the pose.
3. **E** puts the gizmo on the picked piece. Drag it with the left mouse
   button: in move mode (**W**) the arrows move it, in rotate mode (**R**) the
   rings turn it, **Q** switches between world axes and the piece's own axes.
   **G** hides the cursor so you can look around, and brings it back.
   **Left Alt** drops a prop onto the ground. **Enter** keeps the move,
   **Backspace** puts that piece back. A cardboard in a hand rides on that
   hand in the hand's own axes, so it moves and turns with the animation:
   held up, it sits in the right hand like the sign of the community sign
   emote for the same animation; being written on, it sits in the left hand
   like the notepad of the notepad emote, so the pencil stays on it. Both
   keep every direction you give them here.
4. **Z / X** resize the picked cardboard.
5. **Enter** (while picking) saves for everyone; **Backspace** throws every
   change away.

### Gizmo keys

The gizmo is the game's own (`DRAW_GIZMO`), the same one the NS leaderboard
uses — a port of DemiAutomatic/object_gizmo (GPLv3; `client/gizmo.lua` and
`client/gizmo_dataview.lua` ship unencrypted). No extra resource is needed.

The game's gizmo only listens to key mappings, so the script registers seven,
under Settings > Key Bindings > FiveM ("ns-homelessjob gizmo: …"). They are set up
automatically, players never have to bind anything, and they do nothing while
no gizmo is on screen:

| Key | Does |
|---|---|
| left mouse | grab and drag a handle |
| **W** / **R** | move mode / rotate mode |
| **Q** | world axes ↔ the piece's own axes |
| **Enter** / **Backspace** | confirm / cancel |
| **Left Alt** | drop the piece onto the ground |

**G** (cursor on / off) is an ordinary game control.

Saved layouts go to `data/layout.json` inside the resource and are merged over
`config.lua` on every start. Delete that file to go back to the config values.

### Sign moderation

Sign text is filtered and logged server-side. Drawings cannot be
word-filtered: every stroke is checked for shape and size on the server, and
an admin removes an unwanted sign with `/hjclear <id>`. To mirror blocked signs to
Discord, set a convar (never put a webhook in `config.lua` — that file is a
shared script and reaches every client):

```
set ns_homelessjob_webhook "https://discord.com/api/webhooks/..."
```

## Exports

**Server**

```lua
exports['ns-homelessjob']:isBegging(source)       -- boolean
exports['ns-homelessjob']:getSetup(source)        -- { setup, coords, heading, pose, session } or nil
exports['ns-homelessjob']:getLevel(source)        -- current level number
exports['ns-homelessjob']:getProgress(source)     -- { donations, earned, minutes, level } or nil
exports['ns-homelessjob']:clearSetup(source)      -- tear a setup down
```

**Client**

```lua
exports['ns-homelessjob']:isBegging()             -- boolean
exports['ns-homelessjob']:getSetup()              -- { setup, coords, heading, pose } or nil
```

## How the money works

The server decides everything. It runs a countdown per setup that only ticks
while the owner is actually working (standing by the setup, holding the sign
up or not, alive and on foot — measured by the server), rolls the amount from the setup's range, and
hands the owner's client a one-shot token. Packing up does not reset that
countdown: a setup put down again within the hour picks it up where it
stopped, and there is a short wait before you can set up again at all. The client only stages the pedestrian walking over and
returns the token on arrival. A token is claimable once, by its owner, not
before the pedestrian had time to walk, and only while the owner is still at
the setup. The client never sends an amount.

Player donations move money from the donor's pocket to the beggar's pocket
after a server-side amount, distance (to the beggar or to their cup),
cooldown and balance check; a cardboard only takes money while it is held
up. Every gift counts towards the beggar's level in full: one more donation,
and the whole amount as earned. Note that this lets two players who pass cash
back and forth level each other up; `Config.Economy.DailyCap` only limits
money from pedestrians.

A setup belongs to the character that put it down: logging out to character
select tears it down, so the next character never works a setup it has not
unlocked.

## Servers with NPCs turned off

The pedestrians who walk over and pay are not taken from the street: the
script creates each one itself (`CreatePed`, flagged as a mission entity) and
lets it go again after the visit. What your server's setup means for them:

- **Ambient population off** — density multipliers at `0`,
  `onesync_population false`, population disabled on a routing bucket, or a
  "no NPCs" script that only lowers the density: **pedestrians still come.**
  These settings stop the game's own street NPCs, not script-made ones.
- **A script or anticheat that deletes peds** — one that loops over every ped
  and deletes everything that is not a player, or deletes entities it did not
  create itself: **the pedestrian is deleted on the way, that visit is
  cancelled and pays nothing.** The countdown carries on and the next
  pedestrian tries again, so on such a server a setup earns no money from
  pedestrians at all. Whitelist `ns-homelessjob` in that script (or have it
  skip mission entities). Player donations and the sign keep working, and
  levels then grow only from player donations and minutes.
- **Entity lockdown** — `sv_entityLockdown strict` or `relaxed`: clients may
  not create networked script entities, so the pedestrian is created on the
  beggar's own game only (`Config.Visits.Networked` falls back by itself).
  **Payments work as usual**; the players around just do not see that
  pedestrian walk over and drop the money.

To check after installing: put a setup down, stand by it and wait 15–30
seconds — the first pedestrian should walk up to your cup. If none ever
arrives, look for one of the cases above.

Docs: https://nativescripts.com/docs
