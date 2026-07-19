# ns-chat

Fast, keyboard-first **PvP chat** for FiveM: channels, squad/crew groups,
private messages, mentions, server-side anti-spam and moderation. Drop-in
replacement for the stock `chat` resource.

Cross-framework: **ESX / QBCore / Qbox / standalone**, detected at runtime.

## Design

Built for reading mid-firefight. At rest there is no panel and no chrome, only
the message lines themselves, kept legible over bright or dark footage by weight
and a hard text shadow rather than a box drawn over the game. A surface appears
only while you are typing. Nothing blurs or filters the frame underneath, and
only opacity and transform animate, so the chat stays cheap while the game runs.

**Keyboard-first, mouse on demand.** `T` opens the input, `Esc` closes it, `Tab`
cycles channels / command suggestions. The cursor stays off so it never fights
the camera in a fight; **middle-click** while the input is open to bring it up
(to click a tab or a suggestion) and again to hide it. A one-line hint under the
input says so. The mouse key is rebindable in FiveM's key settings.

## Features

- **Channels** — `GLOBAL`, `SQUAD`, `CREW`, each its own colour and its own view:
  a channel only shows its own messages (announcements, DMs and system lines are
  visible everywhere). Cycle with `Tab` while typing. Squad/crew route only to
  your group members (server-side). Players can **block** a channel in Settings
  to stop seeing or posting in it.
- **Name colour** — players set their own colour with `/chatcolor`; it shows on
  their name, id and crew tag. Saved per player.
- **Crew tag** — a player's crew id shows as a badge next to their name.
- **Mentions** — `@name` or `@id` (e.g. `@5`) highlights and pings the target.
- **Private messages** — `/w` or `/dm <id> <message>`, `/r` to reply.
- **Player ignore** — `/ignore <id>` hides a player's messages for you. Keyed to
  a stable identifier and stored server-side, so it survives reconnects and never
  lands on whoever inherits the id later.
- **Auto announcements** — configurable server messages on a timer.
- **Anti-spam** — staged: a warning first, then an auto-mute if it continues.
- **Banned words** — server-side filter; staff edit the list in-game with
  `/filteradd` / `/filterremove`, persisted (no restart).
- **Moderation** — `/mute` / `/unmute` / `/announce` / `/clearall`,
  click-to-moderate on a name, and a staff-only channel (ACE-gated).
- **Message reactions** — 👍💀🔥 tapbacks on a message (bring up the cursor with
  middle-click). Only players who saw a message can react; the emoji set is
  configurable and server-validated (`Config.Reactions`).
- **Staff tags** — `ADMIN` / `MOD` badges mark staff in chat. Ranks match from
  **ACE**, a **Discord role**, a static **identifier list**, or a runtime
  `/setstaff <id> <rank>` grant — mix freely, no ACE required (`Config.StaffTag`).
  A staffer can hide their own tag from Settings → Privacy to blend in.
- **Rank / badge tags** — pull a leaderboard rank, VIP or clan badge from YOUR
  resource (`Config.Tags.resolve`) and show it before the player's name.
- **Chat lock** — staff freeze public chat during events with `/chatlock`
  (commands and staff still work).
- **Profile avatars** — optional Discord / Steam avatar next to a player's name
  (server-only credentials — see setup below).
- **Show/hide** — `L` cycles visibility (always visible / on new message /
  hidden); `T` to type. Only shows while actually in-game. Max 100 chars.
- **Full command autocomplete** — suggests every registered command (the game's
  own and other resources' `/commands`), not just ns-chat's (`Config.SuggestGameCommands`).
- **Drop-in compatible** — listens to the stock `chat:addMessage` /
  `chat:addSuggestion` / `chat:clear`, and fires the `chatMessage` server event,
  so resources built on the default chat keep working. Prebuilt UI is bundled in
  `html/` (React + Vite source in `ui/`); no CDN, fonts bundled locally.

## Commands

Switch channels with `Tab` or the tab bar — there is no per-channel send command.

| Command | Description |
|---|---|
| `/w` · `/dm <id> <msg>` | Private message a player |
| `/r <msg>` | Reply to the last player who messaged you |
| `/ignore <id>` · `/unignore <id>` | Mute / unmute a player locally |
| `/chatcolor <preset\|#hex>` | Set your name colour |
| `/chatsettings` | Open the settings panel |
| `/mute <id> [sec] [reason]` · `/unmute <id>` | Admin: mute a player (ACE-gated) |
| `/announce <msg>` · `/clearall` | Admin: broadcast / clear chat |
| `/filteradd <word>` · `/filterremove <word>` · `/filterwords` | Admin: manage the banned-word filter in-game |

Admins also moderate by clicking a player's name in chat (bring up the cursor
with **middle-click** first). Moderation commands and the staff channel appear
in autocomplete only for staff.

## Squad & crew

ns-chat is a chat, not a gang or party system — squad/crew membership is owned
by whatever your server already runs. There is **no built-in join command**;
membership is read through a `resolve` setting in `config.lua` (`Config.Groups`)
or pushed in from your own resource via the `setSquad` / `setCrew` exports.

| `resolve` | Membership comes from |
|---|---|
| `'auto'` | The **framework gang** (QBCore / Qbox) or **ox_core** gang automatically. |
| `'gang'` | Framework gang name (qb / qbx `PlayerData.gang`). |
| `'ox_core'` | ox_core gang (a group of type `gang`). |
| `'job'` | Framework job name (esx / qb / qbx). |
| `'statebag'` | A party statebag — `Player(src).state[stateKey]` (set `stateKey`, default `party`). Most PvP party scripts store the party here. |
| `function(src)` | **Your own source** — return the player's group id from any resource. |
| `nil` / `false` | No resolver: membership comes only from the `setSquad` / `setCrew` exports below. |

Verified against `qb-core` / `qbx_core` / `ox_core` / `es_extended` source, and
the framework "empty" values (`none`, `unemployed`) count as no group.

Defaults: **crew** is `'auto'` (your gang system), **squad** has no resolver (wire
it to your party/lobby script). Point either at a custom resource or a party
statebag like this:

```lua
-- config.lua — a custom gang resource:
Config.Groups.crew.resolve = function(src)
    return exports['my_gangs']:GetPlayerGang(src)   -- return a group id, or nil
end

-- …or a party script that uses a statebag under a different key:
Config.Groups.squad.resolve  = 'statebag'
Config.Groups.squad.stateKey = 'lobbyId'
```

Everyone a source maps to the same id is one squad/crew. You can also push
membership from another resource via the exports below.

**Crew tag.** The badge before a name can show your crew script's real tag
(e.g. `LSMC`), not just the group id. `Config.CrewTag.resolve`:

- `'auto'` (default) — auto-detects a running known crew/gang script and pulls
  its tag; falls back to the built-in crew group id. Detected out of the box:
  **gfx-crew**, **crews** (LikeManTV), **rcore_gangs**, **av_gangs**,
  **vanish_gangs**. More can be added on request.
- `'gfx-crew'` — target one specific known script.
- a function — any other script:
  ```lua
  Config.CrewTag.resolve = function(src)
      return exports['my_crew']:GetPlayerTag(src)   -- 'LSMC' | { text='LSMC', color='#ff9d47' } | nil
  end
  ```
- `nil` — the built-in crew group id.

## Requirements

- One supported framework: ESX / QBCore / Qbox (or standalone)
- (optional) ox_lib — used for notifications if present

## Installation

1. Drop `ns-chat` into your server `resources` folder.
2. In `server.cfg`:
   - **Disable the stock `chat` resource** — comment out or remove the
     `ensure chat` line. (`ns-chat` satisfies `provide 'chat'`, so resources
     that depend on chat keep working.)
   - **Start ns-chat** (after your framework):
     ```cfg
     ensure ns-chat
     ```
3. Restart the server.
4. Configure channels, commands and anti-spam in `config.lua`.

ns-chat turns off FiveM's built-in system chat (the bottom-right `[ALL]` input
box) itself, from the client, the same way the stock chat does — so you do not
need `set resources_useSystemChat false`. If you prefer the convar as well it
does no harm, but it is read only at server boot.

## Settings panel

Players open it with `/chatsettings` or the gear in the composer. Everything is
per-player and saved automatically: font family, message size, **chat size**
(overall scale), accent colour, crew-tag colour, background opacity, fade time,
corner/position (or free drag-to-move), timestamps, player IDs, profile-avatar
source, **channel bar** (top tabs vs. next to the input), default channel,
blocked channels, mention highlight, autocomplete, mention/DM sounds, unread
badges, DM blocking and the ignore list.

### Profile avatars — Discord & Steam setup (optional)

Players can show their own Discord or Steam avatar next to their name and in the
settings header. This needs API credentials. They are **server-only** convars —
**never put them in `config.lua`**, which ships to every client. Keep the token
secret; if it ever leaks, reset it.

Enable the feature in `config.lua` under `Config.Avatars` (on by default), then
add whichever credentials you want below.

#### Discord avatar — how to get the bot token

1. Open the **Discord Developer Portal**: <https://discord.com/developers/applications>
2. **New Application** → give it any name → **Create**.
3. In the left menu open **Bot** → **Add Bot** (if prompted) → **Reset Token** →
   **Copy**. That copied string is your bot token.
   - No gateway intents, permissions or server invite are required — the token is
     only used to read public avatars via `GET /users/{id}`.
4. Add it to `server.cfg` (server-only — put it above `ensure ns-chat`, and keep
   it out of any file you share or commit):
   ```cfg
   set sv_chat_discord_bot_token "PASTE_YOUR_BOT_TOKEN_HERE"
   ```
5. That's it. The player must have **Discord running** while in game so FiveM
   provides their `discord:` identifier (the server looks the avatar up from it).
   If a player has Discord closed, they just get a monogram placeholder.

> Use a **bot token**, not a webhook URL — webhooks cannot read user profiles.
> Already run another resource with a Discord bot? ns-chat also falls back to the
> `ns_leaderboard_discord_token` convar, so you can reuse an existing token.

#### Steam avatar — how to get the Web API key

1. Get a key at <https://steamcommunity.com/dev/apikey> (sign in, register a
   domain — any placeholder works — and copy the key).
2. Add it to `server.cfg`:
   ```cfg
   set sv_chat_steam_web_api_key "PASTE_YOUR_STEAM_KEY_HERE"
   ```
   - ns-chat also reads FiveM's standard `steam_webApiKey` convar as a fallback,
     so if you already set that you don't need a second line. Note `steam:`
     identifiers only exist at all when `steam_webApiKey` is set, so Steam avatars
     require it either way.

#### Notes

- A player only ever receives **their own** avatar URLs; nothing about anyone
  else is exposed to a client.
- Leave a credential unset and that platform simply shows a monogram placeholder.
- Resolved avatars are cached server-side for `Config.Avatars.cacheMinutes`.
- Each player chooses which avatar (Discord / Steam / off) shows on their
  messages, in **Settings → Appearance → My profile picture**.

## Staff tags

Mark mods/admins with a badge before their name. Each rank in `Config.StaffTag`
matches from any source — you don't need ACE:

```lua
Config.StaffTag.ranks = {
    { key = 'admin',  label = 'ADMIN',  color = '#ff5c8a', ace = 'admin' },
    { key = 'mod',    label = 'MOD',    color = '#5b9dff', discord = '1122334455' },   -- Discord role id
    { key = 'helper', label = 'HELPER', color = '#43d17f', identifiers = { 'license:ab…' } },
}
```

- **ACE** — `ace = 'admin'` (with the `group.<ace>` fallback).
- **Discord role** — `discord = '<roleId>'`. Set `Config.StaffTag.discordGuild`
  (your guild id) and the bot token convar `sv_chat_discord_bot_token` (same as
  avatars); the bot must be in that guild. Roles are cached per player.
- **Manual (static)** — `identifiers = { 'license:…', 'discord:…' }`.
- **Manual (runtime)** — `/setstaff <id> <rankKey>` / `/unstaff <id>` (admin).
  Saved to a plain JSON file in the resource (`data/staff-grants.json`) — easy to
  read, edit or back up. Stored under the player's best id (license → steam →
  discord → fivem) and matched against **any** of their identifiers, so
  hand-edited entries keyed by `license:…`, `steam:…`, `discord:…` all work.

Ranks are checked top-to-bottom; the first match wins. Staff can hide their tag
in **Settings → Privacy** (unless `allowHide = false`).

## Moderation

`/mute <id> <seconds> <reason>` mutes a player; the reason is optional. When
`Config.MuteAnnounce` is on, the mute is announced in chat for everyone with the
duration and reason, and the muted player is told why.

## Exports (server)

```lua
-- Send a message to one player, or -1 for everyone.
exports['ns-chat']:addMessage(target, {
    mode   = 'system',          -- say | whisper | system | announce
    author = 'Dispatch',
    text   = 'The next round starts in 60 seconds.',
})

-- Send only to the members of a player's squad or crew.
exports['ns-chat']:addGroupMessage('squad', source, {
    mode   = 'system',
    text   = 'Your squad captured the point.',
})

-- Drive squad / crew membership from a gang or lobby resource (pass nil to
-- remove). If you set a `resolve` for that group in config, that source wins.
exports['ns-chat']:setSquad(source, 'alpha')
exports['ns-chat']:setCrew(source, 'redline')
```

## Message modes

| mode | rendered as |
|---|---|
| `say` | `[id] Name: text`, coloured by the channel it was sent to |
| `whisper` | private message between two players |
| `system` | plain system line |
| `announce` | highlighted server announcement |
