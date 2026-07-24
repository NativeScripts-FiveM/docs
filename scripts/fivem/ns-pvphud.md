# ns-pvphud

A focused **PvP combat HUD** for FiveM — health, armor, the active weapon with
its ammo, sprint stamina and a kill-streak counter. No killfeed, no scoreboard,
no RP clutter. Just the combat state a player reads mid-fight, in one clean
overlay that every player can restyle and rearrange to taste.

Standalone-first and framework-aware: runs fully standalone or alongside
**ESX / QBCore / Qbox** — the framework is detected at runtime, nothing to set.

## Features

- **Health & armor** with a low-health warning tint
- **Weapon** — a real per-weapon silhouette (100+ weapons) plus a clip / reserve
  ammo readout; melee and throwables drop the ammo count automatically
- **Sprint stamina** bar (off by default — players enable it; server can disable
  it entirely on no-stamina servers)
- **Kill-streak counter** with a countdown bar that resets after a configurable
  window of no kills
- **Native health/armor bars removed** via a bundled `stream/minimap.gfx` so the
  game's own bars don't double up with the HUD (the minimap itself stays)

### Every player customises their own HUD (in-game, no config edits)

Open with **F7** (or `/pvphud`). Choices are saved per player and restored on
reconnect.

- **9 visual themes**, each with its own structure — bars, rings, pips, numbers,
  standing bars, framed modules — not just recolors
- **Colors** — health, armor, weapon/ammo, stamina and the kill counter, each a
  one-click preset, a custom picker, or "follow the theme"
- **Icons** — swap the health / armor / kill-counter glyphs
- **Sizes** — separate HUD-size and icon-size sliders
- **Free drag-and-drop layout** — place every element anywhere, with snap-to-edge
  and align-to-other-element guides so nothing lands crooked
- **Element toggles** — turn any part of the HUD on or off
- **Minimap** on/off

### Under the hood

- Only pushes a UI update when a value actually changes, and backs off polling
  while the HUD is hidden → idle resmon stays near `0.00ms`
- Hidden until the character is fully loaded — never flashes over the loading
  screen, character selector or spawn picker
- CDN-free NUI (React + Vite, fonts bundled locally)
- No custom net events; the HUD is fully client-side

## Requirements

- Nothing mandatory. Works standalone; ESX / QBCore / Qbox are auto-detected if
  present.
- Node.js + npm are only needed if you build the UI from source.

## Installation

1. Drop `ns-pvphud` into your `resources` folder.
2. Add `ensure ns-pvphud` to your `server.cfg`.
3. (Optional) Tune `config.lua` — starting theme, accent color, kill-streak
   window, which elements ship enabled.

The UI is pre-built in `html/`. If you edit the source in `ui/`, rebuild with
`cd ui && npm install && npm run build`.

> **Minimap note:** `stream/minimap.gfx` replaces the game's minimap to remove
> the default health/armor bars. Only one resource can replace it — if another
> HUD on your server ships its own `minimap.gfx` (qb-hud, esx_hud, most
> square-minimap packs), remove one of them.

## Configuration

Everything visual is a per-player choice in the menu. `config.lua` only holds
server-owner defaults and behaviour:

| Key | Purpose |
|---|---|
| `Config.Theme` | Starting theme for new players |
| `Config.Scale` | Starting HUD size (players fine-tune it, 0.70 – 1.50) |
| `Config.AccentRgb` | Accent color as `"R, G, B"` (the panel + default HUD accent) |
| `Config.Elements` | Which parts ship enabled (health / armor / weapon / stamina / kills) |
| `Config.Visibility` | `always`, or `armed` (only while holding a weapon) |
| `Config.KillResetSeconds` | Kill-streak window before it resets |
| `Config.KillPlayersOnly` | Count only player kills, or any ped |
| `Config.LowHealth` | Health warning-tint threshold (0-100) |
| `Config.StaminaEnabled` | Turn stamina off server-wide (hides every stamina control) |
| `Config.StaminaInverted` | Flip if the stamina bar reads backwards on your build |
| `Config.MinimapDefault` | Minimap on/off out of the box |
| `Config.HideWhileNoControl` | Hide the HUD while the player has no control (selectors, cutscenes) |
| `Config.SettingsKey` / `Config.SettingsCommand` | Menu keybind (`F7`) and command (`/pvphud`) |
| `Config.Debug` | Dev logging + `/killtest` / `/killreset` — **set `false` before release** |

Docs: https://fivem.nativescripts.com/docs/
