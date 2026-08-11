# ns-kits

Kit menu for FiveM. Players open the menu with the `/kit` chat command and claim from a list of kits — free ones on a timer, Discord-role-gated ones, and premium tiers.

Cross-framework: **ESX / QBCore / Qbox / vRP 1.x / standalone** — the framework, inventory and notification system are detected at runtime.

## Kits

### Free (no role required)

| ID | Cooldown | Contents |
|---|---|---|
| `starter` | **once per character** | Bandage ×3, water ×2, sandwich ×2, phone ×1, $500 |
| `daily` | 24 hours | Water ×3, sandwich ×3, bandage ×1, $250 |
| `weekly` | 7 days | Bandage ×5, first aid ×1, repair kit ×1, water ×2, $1,000 |
| `monthly` | 30 days | Bandage ×10, first aid ×3, repair kit ×2, body armor ×1, water ×4, $5,000 — **off by default** |
| `medic` | 6 hours | Bandage ×4, first aid ×1 |
| `combat` | 12 hours | Body armor ×1, bandage ×4, first aid ×1 — **no weapons**, a top-up between the weekly tiers |
| `repping` | 24 hours | Bandage ×3, water ×2, $750 — **requires the server keyword in the player's Discord status** (off by default, see below) |

`medic` and `combat` are deliberately not scaled-down versions of the tiers below. Each does one job, neither hands over a weapon, and both come back several times a day, so they stay useful without competing with anything anyone paid for.

### Discord-gated

| ID | Required role | Cooldown | Contents |
|---|---|---|---|
| `discord` | `member` | 7 days | Bandage ×3, water ×3, sandwich ×3, radio ×1, $2,000 |
| `streamer` | `streamer` | 7 days | Pistol (60), bandage ×5, first aid ×1, repair kit ×2, radio ×1, $2,500 |
| `veteran` | `veteran` | 7 days | Body armor ×1, bandage ×6, first aid ×2, repair kit ×2, water ×3, $3,000 — **off by default** |
| `booster` | `booster` | 7 days | Pistol (100), body armor ×1, bandage ×5, first aid ×2, repair kit ×2, water ×3, $5,000 |

### Premium tiers (Discord-gated)

| ID | Required role | Cooldown | Contents |
|---|---|---|---|
| `vip` | `vip` | 7 days | Pistol (100), body armor ×1, bandage ×5, first aid ×1, repair kit ×1, $3,000 |
| `silver` | `silver` | 7 days | Pistol (100), body armor ×1, bandage ×6, first aid ×1, repair kit ×1, lockpick ×1, $3,500 — **off by default** |
| `gold` | `gold` | 7 days | Pistol (100), micro SMG (60), body armor ×2, bandage ×8, first aid ×2, repair kit ×2, lockpick ×3, $4,000 + $1,000 bank |
| `platinum` | `platinum` | 7 days | Pistol (120), micro SMG (80), body armor ×2, bandage ×9, first aid ×2, repair kit ×2, advanced lockpick ×1, $5,000 + $1,500 bank — **off by default** |
| `premium` | `premium` | 7 days | Pistol (150), SMG (100), pump shotgun (40), body armor ×3, bandage ×10, first aid ×3, repair kit ×3, advanced lockpick ×2, $6,000 + $2,000 bank |
| `diamond` | `diamond` | 7 days | Pistol (200), SMG (150), pump shotgun (60), carbine rifle (120), body armor ×3, bandage ×10, first aid ×5, repair kit ×3, advanced lockpick ×2, radio ×1, $10,000 + $5,000 bank |
| `elite` | `elite` | 7 days | Pistol (250), SMG (200), pump shotgun (80), carbine (180), assault rifle (200), body armor ×3, bandage ×12, first aid ×6, repair kit ×4, advanced lockpick ×3, radio ×1, $15,000 + $8,000 bank — **off by default** |

Disable any kit you don't offer by setting `enabled = false` in `config.lua` — it is hidden in the menu and server-side claims are rejected.

## Requirements

- One supported framework: **ESX / QBCore / Qbox / vRP 1.x** (standalone works, but there is no inventory to grant items into)
- Any inventory, paid or free — **ox_inventory, origen_inventory, tgiann-inventory, core_inventory, qs-inventory, codem-inventory, qb-inventory, ps-inventory, lj-inventory, esx_inventoryhud, gfx-inventory**, or **the framework's own** with no inventory resource at all, which is what a lot of PvP servers run. Detected at runtime; the first one running wins.

  **gfx-inventory** is wired from the vendor's documented exports, including its capacity check (`HasInventoryGotSpaceForItem`), so a full inventory refuses the claim before anything is handed over and no cooldown is burned. It is also one of the few non-ox inventories that carries per-item metadata, and ns-kits passes it through. Items go to the `inventory` type, not `protected` or `stash`.
- **A SQL driver** — any of **oxmysql**, **ghmattimysql** or **mysql-async**. There is no `dependency` line naming one, so none of the three is refused; ns-kits waits up to 30s for whichever is running and names all three in the console if none appears.
- *(optional)* **ox_lib** — used for notifications when present
- *(optional)* A Discord bot — only for the role-gated kits

No build step. No CDN. The menu is plain HTML/CSS/JS and renders on offline servers.

## Installation

1. Drop `ns-kits` into your server `resources` folder.
2. Add `ensure ns-kits` to `server.cfg`, after your framework and inventory.
3. **Verify the item keys** — see below. This is the step people skip.
4. *(optional)* Set up Discord role gating — see below.

The `ns_kits_claims` table is created automatically on first start.

## Item keys — read this before going live

The kits ship with **qb-core item names** (`bandage`, `water_bottle`, `sandwich`, `ifaks`, `repairkit`, `armor`, `radio`, `phone`, `lockpick`, `advancedlockpick`). If your server names things differently, edit `items[*].name` in `config.lua`.

**ns-kits checks this for you.** On startup it resolves every configured item against your server's real item list and prints the result:

```
[ns-kits] item check: 10/10 resolved (qb-core/shared/items.lua)
```

On a server with different names, the ones it cannot find are named explicitly:

```
[ns-kits] item check: 8/10 resolved (items table)
[ns-kits] 2 item(s) could not be resolved and will be SKIPPED when a kit is claimed:
[ns-kits]   • "armor"  used by kit(s): booster, gold
```

Unresolved items are simply left out — the rest of the kit (and the money) is still handed over. Nothing is substituted: an item you did not ask for never ends up in a kit. The check reads whichever of these it finds: `ox_inventory`'s item table, `qb-core` / `qbx_core`'s `shared/items.lua`, or the `items` table on ESX-style setups. If none is readable, names are used exactly as configured.

Where to find your real names:

- **ox_inventory:** `ox_inventory/data/items.lua`
- **QBCore:** `qb-core/shared/items.lua`
- **Qbox:** `qbx_core/shared/items.lua`
- **ESX:** `SELECT name FROM items;`

Watch each item's stack/weight limit — if a count exceeds it, players get "can't carry" even with an empty inventory.

**A kit is all-or-nothing.** If any item will not fit, the ones that already went
in are removed again, no weapons or money are handed over, and **no cooldown is
stored** — the player frees up space and retries, having lost nothing. This
matters because the up-front capacity check only works on `ox_inventory` and
recent `qb-inventory`; everywhere else the limit is discovered item by item, and
without the rollback a player would keep the light items and burn the cooldown.
(Weapons are the exception: there is no reliable cross-framework way to take one
back, so a weapon that will not fit still leaves a partial claim.)

Weapons are configured separately in the `weapons` list (`WEAPON_PISTOL` etc.) and handed over in whatever way your backend stores weapons — ox_inventory takes an uppercase item plus ammo metadata, ESX has its own `addWeapon`, and the qb family uses a lowercase item.

**On plain ESX** — no inventory resource, just `es_extended` — three things are worth knowing, because ESX reports none of them:

- `addInventoryItem` does **nothing** for an item that is not registered in your `items` table, and returns nothing either way. ns-kits reads the count back before and after, so an unregistered item is reported as a failure instead of a silent "claimed".
- `removeInventoryItem` only acts when the result would be ≥ 0, so a rollback can remove nothing. That is checked the same way — a rollback that did not happen is never reported as one.
- `addWeapon` is wrapped in `if not hasWeapon`, so giving a weapon to a player who already owns it did nothing at all, not even the ammo. ns-kits tops the ammo up instead, which on a PvP server is the usual case.

## Discord role setup

The bot is only needed for role-gated kits. Role gating has no on/off switch — it turns on the moment a bot token and guild ID are set in `server/sv_config.lua`. If you don't want gated kits, leave those empty and set `enabled = false` on the gated kits; they stay locked and nothing else changes.

1. Create a bot at the Discord Developer Portal, and under **Bot → Privileged Gateway Intents** enable **SERVER MEMBERS INTENT**.
2. Invite the bot to your guild (OAuth2 → URL Generator → scope `bot`).
3. Put the token and guild ID in `server/sv_config.lua`. It reads convars by default, so the recommended setup keeps the secret out of the resource entirely:
   ```
   set ns_kits_discord_token "YOUR_BOT_TOKEN"
   set ns_kits_discord_guild "YOUR_GUILD_ID"
   ```
   Put those in a `server.cfg` that is not committed anywhere public.
4. Copy each role ID (Discord → Developer Mode → right-click role → Copy Role ID) into `ConfigServer.Discord.Roles` in **`server/sv_config.lua`**. That's it — a role with an ID gates immediately, and any kit pointing at an empty key stays locked with a startup warning naming it.

> Everything that identifies your server — bot token, guild ID, role IDs, the status keyword, the unlock link — lives in `server/sv_config.lua`, which the server loads alone. `config.lua` is a `shared_script`: every value in it is downloaded by every player who connects, so it holds behaviour and wording only. Nothing in it names your Discord.

Every gated kit is resolved the same way, paid tiers included: `gold`, `diamond`, `elite` and the rest are role IDs in `ConfigServer.Discord.Roles` and nothing else. Add a key there and any kit can gate on it.

**Role changes take effect immediately on claim, and within seconds in the menu.** Role lookups are cached per player for 10 seconds, and that cache only feeds the *menu*. Claiming re-asks Discord at the moment items change hands, so a role you took away cannot pay out even once more, and a role you just granted works the instant the player can press the button.

The cache exists to stop one Discord API call per menu open, which a room full of players would turn into a 429 — and a rate-limited check fails every gated kit at once. Ten seconds means a role you just granted shows up as soon as the player reopens the menu. A player's entry is also dropped when they disconnect, and the cache lives in the resource, so restarting ns-kits clears it for everyone.

## Status reward — "put our name in your Discord status"

A kit with `requireStatus = true` unlocks when the player's **Discord custom status** contains `ConfigServer.Discord.StatusKeyword` (case-insensitive). The shipped `repping` kit is an example — it's disabled by default.

**Setup is two steps, and uses the same bot token as the role checks:**

1. Developer Portal → your app → Bot → Privileged Gateway Intents → enable **PRESENCE INTENT**.
2. In `server/sv_config.lua`:
   ```lua
   ConfigServer.Discord.StatusKeyword = 'nativescripts.com'
   ```
   …then set `enabled = true` on the `repping` kit (or add `requireStatus = true` to your own). Status rewards turn on automatically once the keyword is set — there is no separate toggle.

That's it. **No second bot, no extra token, no Node.js install, no separate process to keep running.** A custom status isn't exposed by Discord's REST API — it only arrives over the Gateway — so this resource keeps its own Gateway connection in `server/gateway.js`, running inside the FXServer JS runtime with zero npm dependencies.

Notes:

- The status is read from a live cache, so a player's status change applies **immediately** — no waiting for a refresh.
- The player must be **online** and share a guild with the bot. An offline user has no presence, so no status.
- `requireStatus` and `requireRole` can be combined; both must pass.
- If the intent is off, the server console prints a red line at startup telling you exactly that (Discord close code `4014`), and status kits stay locked instead of failing silently.

## Discord logs

Every claim, refusal, admin action and failure can be posted to Discord as a rich
embed. It is off until you paste a webhook URL — nothing is sent, and no HTTP
request is made, while it is unconfigured.

**Setup:** Discord → the channel you want → **Edit Channel → Integrations →
Webhooks → New Webhook → Copy Webhook URL**, then in `server/sv_config.lua`:

```lua
ConfigServer.Logs.Default = 'https://discord.com/api/webhooks/...'
```

Better: leave that line reading the convar it ships with and put the URL in a
`server.cfg` that is not in your public repo:

```cfg
set ns_kits_log_webhook "https://discord.com/api/webhooks/..."
```

Then run `/kitlogtest` in-game as an admin — it posts one test embed per
configured webhook and tells you which categories have nowhere to go.

### What gets logged

| Event | Colour | Contains |
|---|---|---|
| `claim` | green | Player identity, kit, **exactly what was handed over** (items, weapons, money), and when it can be claimed again |
| `partial` | orange | The same, plus what could **not** be delivered — and a note that the cooldown was recorded anyway |
| `failed` | red | Nothing was handed over (usually a full inventory), so no cooldown was recorded and the player can retry |
| `denied` | yellow | A gate refused it: cooldown still running, one-time kit already claimed, missing role, status keyword missing |
| `spam` | grey | The claim rate limiter refused it — **off by default**, it fires on ordinary button mashing |
| `admin` | blurple | `/resetkit` (who, target, rows removed), `/kitdebug`, `/kitlogtest`, and any non-admin who tried them |
| `error` | dark red | Bad bot token, Discord rate limits, a failed item rollback, no SQL driver, a claim that never completed, gateway stopped |

The layout matches ns-advanced-airdrop, so both scripts read the same way in a
staff channel: the title says what happened, emoji sit on **field names only**,
and everything about a player is folded into one field — in-game name, server id,
a clickable `@mention`, a Steam profile link and the character identifier.

Contents are a plain line of real item keys — `5x bandage, 2x water_bottle,
WEAPON_PISTOL (60 ammo), $500 cash`. That is deliberately undecorated: it is the
line you read to answer "what did they get" and then paste straight back into
`config.lua`, and a prettified name would have to be translated by hand each time.

Player names are escaped before they reach Discord. A player calling themselves
`[Free nitro](https://phish.tld)` would otherwise plant a clickable link in your
staff channel.

Timestamps use Discord's own `<t:…:R>` format, so "ready in 6 days" renders in
each reader's timezone rather than the server's.

There is **no "resource started" log**. A restart is something you already know
about, and one embed per restart is noise in a channel that exists to show player
activity — only a *broken* start (no SQL driver, gateway down) is logged.

### Routing and options

All of it is in `ConfigServer.Logs` in `server/sv_config.lua`:

| Key | Purpose |
|---|---|
| `Enabled` | Master switch — `false` means no HTTP call is ever made |
| `Default` | The webhook everything falls back to |
| `Webhooks` | Optional per-category URLs: `claim`, `denied`, `admin`, `error` |
| `Events` | Per-event on/off — turn `denied` off if the channel is too busy, `spam` on if you are hunting an exploiter |
| `ShowIdentifiers` | The full license/steam/fivem block in every player embed — **off by default**, these are ban-list keys. The character identifier and a Steam profile link are always shown |
| `MentionRoleOnError` | Role ID to `@mention` when an error is logged |
| `BotName` / `Avatar` / `Footer` | Embed presentation |
| `ServerName` | Appended to the footer. Empty by default and **not** taken from `sv_hostname` — a hostname is usually an advertising banner, and repeating it under every embed hurts readability. Set it only if several servers post into one channel |

Splitting claims and errors into two channels is worth the extra minute: claims
are a feed you skim, errors are something you want to actually notice.

### Notes

- **Never put a webhook URL in `config.lua`.** That file is a `shared_script`, so
  it is downloaded by every player who connects — and anyone holding the URL can
  post into your staff channel until you delete the webhook.
- Logging never delays a claim. Embeds are queued and sent in batches of up to 10,
  so a slow or dead webhook costs the player nothing.
- Discord rate limits (429) are honoured with its own `retry_after`, and the
  batch is re-queued rather than dropped. A webhook Discord rejects as invalid
  (401/403/404) is disabled with one console line instead of retrying forever.
- Repeated errors are collapsed: an identical failure inside 5 minutes is counted
  rather than re-posted, and the next embed says how many times it happened.
- The "Delivered" field lists what the player **received**, not what the kit
  contains. A kit that only partly fit shows only the part that landed.

## Admin commands

| Command | Does |
|---|---|
| `/resetkit` | Clears **all your own** cooldowns |
| `/resetkit <id>` | Clears **all** cooldowns for that player |
| `/kitdebug` | Prints a per-kit UNLOCKED/LOCKED diagnosis to the server console |
| `/kitlogtest` | Posts a test embed to every configured Discord log webhook |

All are gated on ACE. `ConfigServer.AdminAces` in `server/sv_config.lua` lists
which ones count (`admin` and `ns-kits.admin` by default), and both `<ace>` and
`group.<ace>` are accepted — being a **member** of `group.admin` is enough, which
a plain `IsPlayerAceAllowed(src, 'admin')` would miss. `/resetkit` also works from
the server console, where the player id is required.

The target must be online, and every reset is printed to the console with who ran
it. There is deliberately no "reset everyone" command; for that, one line of SQL:

```sql
DELETE FROM ns_kits_claims;                        -- everything
DELETE FROM ns_kits_claims WHERE kit_id = 'daily'; -- one kit, all players
```

### Troubleshooting

Run `/kitdebug` in-game as an admin. It prints to the **server console**: the detected framework/inventory, your `discord:` identifier, the configured role IDs, the raw roles Discord returned, and an UNLOCKED/LOCKED verdict per gated kit.

The most common causes of a permanently locked kit:

| Symptom | Cause |
|---|---|
| `discord:* identifier MISSING` | The player is not running the Discord desktop app, or your server doesn't allow the Discord identifier |
| `ERROR=bad_token` | The token is wrong, or the bot was never invited to the guild |
| Roles list is empty for a real member | **SERVER MEMBERS INTENT** is off |
| `ERROR=not_configured` | Token/guild ID are still empty in `sv_config.lua` |
| Status kit locked, gateway `stopped=true` | **PRESENCE INTENT** is off (Discord close code 4014) |
| Status kit locked, `no presence cached` | The player is offline, or not in your guild |

## Adding / removing kits

Open **`config.lua`** and edit the `Config.Kits = { ... }` table at the bottom. The block right above it has a copy-paste template and a field reference. After editing, run `restart ns-kits` — no build step.

Available `icon` values (SVG icons shipped in `html/kit-menu.js`):

```
Starter | Daily | Weekly | Clock | Calendar | Status | Discord | Streamer
Youtube | Kick | Steam | Booster | VIP | Gold | Premium | Diamond
```

`accent` is optional and you will rarely want it: leave it out and **each row is
trimmed with its own icon's colour** — Discord kits blurple, Gold gold, Diamond
cyan. Set it only to force a colour:

```
free | elite | community | brand | creator | patron | vip | premium | gold
```

The `contents` list is what the player sees in the menu — it is decorative, and is **not** derived from `items` / `weapons` / `money`. Keep it in sync by hand when you edit a kit.

Write as many lines as the kit really contains; the length never affects the menu. Contents are not shown in the row at all — every row is the same height whatever it holds, and clicking one opens its full list underneath. A twelve-line vault and a three-line starter kit sit identically in the list. Lines starting with `$` are picked out in gold inside that panel.

A **locked** kit opens the same panel, under a *What you're missing* heading. A role gate only does its job if the player can see what is behind it, so the contents stay legible rather than being greyed out — the row itself is dimmed and the padlock is on the button.

### Sending a locked kit somewhere

Set `ConfigServer.Discord.UnlockUrl` in `server/sv_config.lua` to your Discord invite or your store page, and every locked kit's button changes from a dead **Locked** to **Unlock**, which copies that link to the player's clipboard. One kit can override it with its own `unlockUrl` in `config.lua` — useful when the premium kits point at the store and the rest point at the Discord.

Leave it empty and locked kits keep the plain disabled button. That is deliberate: no button is better than one that leads nowhere.

### Reading order

Kits appear in the order you write them in `config.lua`, inside their group. Nothing is re-sorted by state — the list is in the same place every time it opens, so players learn where their kit is instead of hunting for it.

A kit that is counting down draws a ring around its icon that fills as its cooldown runs out, so "nearly back" is something you can see down the list without reading every countdown. When nothing at all is claimable, the footer says when the first one returns.

## Cooldowns

A cooldown belongs to the framework's own player identifier: qb / qbx / Qbox use a
per-character `citizenid`, so a kit is once-per-character there; ESX has no
per-character id, so cooldowns are per account and are shared by all of a player's
characters. Cooldowns are stored in `ns_kits_claims` keyed on that identifier.

## Notifications

ns-kits draws its own, in the menu's own style, and they appear whether the menu is open or closed — the notification layer sits outside the panel, so a claim result still shows after the menu has shut.

```lua
Config.Notify = {
    Provider = 'ns',            -- 'ns' (built-in) | 'framework' (via ns-utils)
    Position = 'top-right',     -- top-right | top-center | top-left | bottom-right | bottom-left
    Duration = 4500,            -- milliseconds on screen
}
```

Set `Provider = 'framework'` to hand every message to your framework's own notification instead (ESX / qb / ox / vRP, whichever ns-utils detects). Worth doing if your server has one house style you want everything to obey; the trade is that those four all look different, so the same claim looks like a different product on each — and a plain framework with no notify at all drops the message entirely.

At most four notifications are on screen at once. The newest always appears the same distance from the screen edge and older ones slide away from it, so there is one place to look.

## Configuration

Everything a server admin needs is in `config.lua`:

| Key | Purpose |
|---|---|
| `Config.UI` | Your server name and an optional logo image |
| `Config.Locale` | Language — `en`, `tr`, `fr` or `pt`, see [Languages](#languages) |
| `Config.Notify` | Notification style, corner and duration — see [Notifications](#notifications) |
| `Config.OpenCommand` | Chat command that opens the menu (default `kit`) |
| `Config.Debug` | Print debug logs to the server console |
| `Config.Kits` | The kit definitions |

Discord — bot token, guild ID, role IDs, the status keyword, the unlock link and
the [log webhooks](#discord-logs) — is entirely in `server/sv_config.lua`, which
the server loads alone. There is no Discord switch in `config.lua`: role and
status gating turn on automatically when the bot is configured, and stay off
(kits locked) when it isn't.

`Config.Messages` and the menu's own wording are **not** in `config.lua` — they
are folded in from `locales/<code>.lua` at load. Translate there.

## Languages

The menu's own wording — buttons, statuses, headings, errors — lives in
`locales/<code>.lua`. Kit names and descriptions are in `config.lua`, since those
are yours to write. A missing key falls back to English, so a partial translation
is safe to ship.

```lua
Config.Locale = 'en'    -- en, tr, fr, pt ship; copy en.lua to add your own
```

- **A partial translation is safe.** English is applied first and your file on
  top of it, per key, so a key you have not translated shows English rather than
  a blank.
- **A wrong code is safe.** An unknown `Config.Locale` falls back to `en` and
  prints one console line listing the codes that do exist.
- **No manifest edit** to add a language — `locales/*.lua` is globbed.
- Keep every `%s` and its order; they are substituted at runtime.

Kit names and descriptions are **not** in the locale files — they live with the
kit in `config.lua`, because they are yours to write rather than ours to
translate. Full notes in `locales/README.md`.

## Appearance

The menu ships **one layout and one theme**, so there is nothing to pick in
`config.lua`. The layout is the `stack` described below; the theme is `crimson`.

**The layout** — a vertical list of self-contained rows under `FREE` /
`COMMUNITY` / `PREMIUM` headings, in a 620px panel. One row per kit: icon, name,
cooldown badge, description, and its own button. There is no detail view, no
popup and no status column, because an enabled **Claim Kit** already means the
kit is ready, and when it is not the button itself says why (`Locked`, `Unlock`)
or when it comes back (`6D 23H`, counting down live). A kit's contents open
downward when the row is clicked, so every row is the same height whatever it
holds. The panel grows with the list and stops at 66% of the screen height.

Kits are filed under groups: set `group = 'free'`, `'community'` or `'premium'`
on a kit to place it, or leave it out and ns-kits files gated kits under
community and everything else under free. Inside a group they appear in the
order you wrote them.

**Theme** — one ships: **crimson**. Translucent near-black glass with red used as
light rather than as a surface — the action button, the active row and the earned
tiers. It lives in `html/themes/crimson.css` and sets nothing but colour,
borders, radii and type sizes, so a full reskin never touches layout.

**Type** — two faces, both SIL OFL 1.1, both shipped in `html/fonts/`: **Anton**
for the header wordmark and **Inter** for everything else. A NUI cannot reach a
CDN, so they are served from the resource — nothing is fetched over the network,
and each ships a latin + latin-ext subset so a translated `Config.Messages` keeps
its glyphs (`ğ ş ı İ ł ř`…).

The server name is set as a wordmark: Anton, uppercase, first word at full
strength and the rest stepped back in colour. Rebranding the header is one line —
`--font-display` in `html/styles.css`; see `html/fonts/README.md`.

Note for anyone changing the countdown: the bundled Inter subset carries no
`tnum` table, so `font-variant-numeric: tabular-nums` is a silent no-op here
(measured — digit `1` is 5.5px and digit `4` is 8.4px either way). The countdown
stays on one line and its column is sized for the widest label it can produce;
do not swap that for tabular figures expecting it to work.

**Kit artwork** — two ways to set it.

*Your own images.* Drop a file in `html/img/` and point the kit at it:

```lua
image = 'img/gold-crate.png'    -- 128×128 PNG or WebP; overrides `icon`
```

The manifest already streams `html/img/*`, so no edit is needed when you add a
file. A wrong path falls back to the named `icon` rather than leaving a hole.
The header logo works the same way — `Config.UI.logo = 'img/logo.png'` replaces
the server-name wordmark. Full notes in `html/img/README.md`.

*Built-in artwork.* `icon` picks one of these, no files involved:

| | |
|---|---|
| **Brand tiles** — the real mark on the real brand colour | `Discord` `Streamer` (Twitch) `Youtube` `Kick` `Steam` |
| **Discord features** | `Status` — the mark with a written status bubble, for the custom-status kit · `Booster` — the boost badge on Discord fuchsia |
| **Timed kits** — these carry their interval, because the interval is the one thing that separates them | `Daily` (alarm clock reading **24**, blue) `Weekly` (calendar reading **7**, amber) · `Clock` / `Calendar` are the same art without a number |
| **Tiers** | `VIP` (star, copper) `Silver` (two coins, grey) `Gold` (three coins, gold) `Platinum` (ingot, icy blue) `Premium` (crown, magenta) `Diamond` (gem, cyan) `Elite` (trophy, violet) |
| **Objects and tools** | `Starter` (gift box, green) `Medic` (first-aid case, red) `Mechanic` (wrench, slate) `Combat` (pistol, olive) |

Every one is built the same way: a rounded tile carrying a vertical colour
gradient, one white glyph, interior detail knocked out in the tile's own
gradient. That single rule is what makes them read as a set. Hues are spread so
no two neighbours in the list collide, and each colour is held at the highest
saturation that still lets the white glyph clear 3:1 against the tile's lightest
stop. The brand tiles use official mark geometry from
[simple-icons](https://github.com/simple-icons/simple-icons) (CC0 1.0) rather
than redrawn approximations — a logo that is nearly right looks worse than none.

To add one, append an entry to `ICONS` in `html/kit-menu.js`: `tile('#hex',
glyph)`, where the glyph is a white shape inside a `0 0 24 24` box and the word
`TILE` in any `fill` or `stroke` is substituted with the tile's gradient. Add a
matching entry to `ICON_ACCENT` just below if you want the row trimmed in the
same colour — that map holds a *lifted* version of each tile colour, because a
value dark enough to carry a white glyph is mud on dark glass.

To reskin, edit `html/themes/crimson.css` — it is only tokens, so nothing in
the layout has to change.

There is no `backdrop-filter` anywhere and there never should be: in a NUI the
game is not part of the page, so it cannot blur gameplay — and with a
transparent page behind it, CEF paints an opaque black rectangle over the screen.

## Database

```sql
TRUNCATE ns_kits_claims;                                      -- reset all cooldowns
DELETE FROM ns_kits_claims WHERE identifier = 'IDENTIFIER';   -- reset a single player
DELETE FROM ns_kits_claims WHERE kit_id = 'starter';          -- let everyone claim starter again
```

`identifier` is the framework identifier — `citizenid` on QBCore/Qbox, `identifier` on ESX.

## Testing

```
/kit        -- open the menu
/kitdebug   -- admin-only Discord/role diagnostic (server console)
```

Docs: https://fivem.nativescripts.com/docs/

## License

Proprietary / commercial — redistribution, resale, or republishing the source is prohibited.
