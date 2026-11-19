# ns-christmas

Decorate any part of the map with Christmas props and adjustable lights, without buying a map. Admins place props in game and every player sees them.

## Features

- 107 custom props plus 18 of the game's own festive props, in a searchable catalogue with favourites and recently placed
- Solid props: players and vehicles stop at huts, sleighs, carousels and statues
- Snow across the whole map with one switch, plus snow patches for single areas
- 10 animated props: a waving Santa, Santa rocking in his sleigh, a wobbling Santa toy, a waving snowman, two turning carousels, a toy train on a ring track, a swinging bell, a spinning star and ornament
- Six different Santas, from a classic statue to a cartoon Santa
- New Year set: a Happy New Year sign, light numbers 0-9 to spell the year, a midnight clock, a champagne toast, light fireworks and a fireworks box
- Any player can light a placed fireworks box with E: the fuse burns for 5 seconds, then a show goes up that only that player sees (30 second cooldown)
- Ten gift boxes in different colours, a potted tree and a tree with presents underneath
- 33 light props, among them a walk-through light tunnel, a light canopy, an 8 m light tree, icicle lights and light outlines of a reindeer, snowman, bell and gift
- Six hand-made scenes for well-known spots, designed on a scan of the real map so nothing stands in a wall, a road or a planter: Legion Square (town tree, light tunnel, carousel, Santa's corner), Del Perro Pier (New Year countdown), Vespucci Beach (snowy corner), Mirror Park (Christmas market by the lake), Sandy Shores (desert fair), Paleto Bay (snowy cove on the beach). Go takes you there; Place lays the scene out on its spot
- Five ready-made layouts, and your own: save an area, then place it anywhere
- Every layout shows up under the gizmo first, so you slide or turn the whole scene before it is saved
- Place a row: up to 20 copies of a prop in a line, with the gap you choose between them; the row follows the slope of the ground
- Light text: type your own words and they are placed in bulb letters (A-Z, Turkish letters, numbers and ! ? . , - '), with the same colours and blink modes as the other lights, as one group
- Select several placed props and move or delete them together
- Undo for every edit (last 20 per admin)
- Season dates: decorations and snow show only between two dates you set, and admins can still decorate early
- Light props with 11 colours, steady, blinking or rainbow, changeable after placing
- Mouse gizmo to move and rotate each prop, with 15 degree turning steps, snap to ground and snap to a wall
- Saved in SQL, restored on restart
- Props are local objects on each client, so they do not use networked entity slots

## Install

1. Put `ns-christmas` in your resources folder.
2. Add `ensure ns-christmas` to your server config, after your database and framework.
3. Add `add_ace group.admin ns-christmas.admin allow`, or rely on `Config.AdminGroup`.

The table `ns_christmas_props` is created on first start. Needs oxmysql, ghmattimysql or mysql-async.

## Use

Type `/xmas`. Pick a prop and press Place. Move it with the gizmo, then press Enter to save or Backspace to cancel. The star next to Place keeps a prop under Favourites; Recent lists the last props you placed. The Placed tab lists what is near you, nearest first, with a search box. You can walk while the menu is open; the camera stays still and typing in a text box stops the walking.

In the gizmo, turns snap to `Config.RotationStep` degrees; hold Shift to turn freely. Alt drops the prop to the ground, Shift + Alt puts its back against the nearest wall behind it.

The row button next to Place sets how many props to place in a line and the gap between them (the step is the prop's own width plus the gap, so wide props never overlap). They follow the first one under the gizmo.

Light text is at the top of the Layouts tab: type up to `Config.MaxTextLetters` characters, pick a colour and press Place. The text appears ahead of you facing you, under the gizmo; Enter saves it as one group you can move or remove later. Each letter is one prop and counts towards `Config.MaxProps`.

In the Placed tab, click a thumbnail (or Shift-click a row) to select several props, then Move together or Delete them.

The undo button in the header (or Ctrl+Z) takes back your last edit: a placed prop, a move, a light change, a delete or a whole layout. Each admin keeps their own last `Config.UndoSteps` edits until they leave. Remove all cannot be undone.

The Layouts tab lists Places and Sets. A set appears ahead of you under the gizmo; slide or turn it, then press Enter to save it or Backspace to drop it. A place opens on its own spot: press its Go button first to stand there, or Place it from wherever you are. Save area stores every prop within 30 m of you as a new layout.

Placed layouts appear under On the map in the Layouts tab: Move drags the whole set with the gizmo, the bin removes it. The World tab also has Remove all, which deletes every decoration on the map after a second click.

Set `Config.ShowFrom` and `Config.ShowUntil` (for example `'12-01'` and `'01-07'`, server clock) to show decorations and snow only in that season. Out of season nothing is drawn for players, but an admin with the editor open still sees everything. The World tab shows whether the season is on.

The World tab switches snow across the map for every player. The setting survives a restart. While snow is on, the common weather sync scripts are paused on each client so they do not fight it: qb-weathersync, Renewed-Weathersync (Qbox), cd_easytime and vSync. When snow goes off (or the season ends) they get their weather back. Any other weather script is overridden every 2 seconds; if it rewrites the weather faster, set its weather to `XMAS` too.

Some base-game props need a recent game build. The catalogue disables the ones a server's build does not have.

## Config

Everything a server owner may change is in `config.lua`:

| Setting | Default | What it does |
|---|---|---|
| `Command` | `xmas` | Opens the editor |
| `AdminAce` / `AdminGroup` | `ns-christmas.admin` / `admin` | Who may edit |
| `MaxProps` | 3000 | Decorations on the whole server |
| `MaxLayoutItems` | 150 | Props in one layout, selection or row |
| `MaxRow` | 20 | Props in one "place a row" |
| `MaxTextLetters` | 32 | Characters in one light text |
| `UndoSteps` | 20 | Edits each admin can take back |
| `ShowFrom` / `ShowUntil` | nil | Season dates, `'MM-DD'` |
| `RotationStep` | 15 | Gizmo turn step in degrees, 0 = free |
| `StreamDistance` | 180 | Metres; further decorations are not spawned |
| `MaxVisible` | 400 | Nearest decorations kept spawned |
| `MaxDynamicLights` | 6 | Real light sources per player, 0 = off |
| `LightsNightOnly` | true | Real light sources only at night |
| `BlinkMs` / `RainbowMs` | 700 / 1500 | Blink and rainbow speed |
| `SnowWeather` | `XMAS` | XMAS, SNOW, SNOWLIGHT or BLIZZARD |
| `FireworkDelay` / `FireworkCooldown` | 5 / 30 | Fuse seconds, seconds between shows |
| `Messages` | | Every text a player sees |

The light colours are in `shared/palette.lua`; they are rows of the tint texture, so they are not a setting. The scenes in the Places list are in `shared/places.lua` (not encrypted), so a server with a map mod can move one.

## Credits

Third-party models and their licences are listed in `CREDITS.md`, which ships with the resource.
