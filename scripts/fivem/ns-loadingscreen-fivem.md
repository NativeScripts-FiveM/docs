# ns-loadingscreen-fivem

A modern, fully self-contained loading screen for **FiveM** (GTA V). No CDN
dependencies — fonts use a system stack and all icons are inline SVG, so it
loads instantly and works on locked-down clients.

## Features

- Animated gradient background out of the box (no images required)
- Optional rotating background images (drop your own screenshots)
- Built-in music player with play/pause + volume (optional)
- Server rules panel and rotating gameplay tips
- Discord & website buttons (open via `openUrl`)
- Real FiveM `loadProgress` wiring + a browser-preview fallback
- Single config file — no need to touch HTML/CSS/JS

## Installation

1. Copy the `ns-loadingscreen-fivem` folder into your server `resources`.
2. Add to your `server.cfg` **before** other resources:
   ```cfg
   ensure ns-loadingscreen-fivem
   ```
3. Edit `html/config.js` to set your branding, links, rules and tips.
4. Restart the server.

> Only one loading screen resource can be active at a time — make sure no other
> `loadscreen` resource is ensured.

## Configuration (`html/config.js`)

| Key | Description |
|---|---|
| `serverName` / `serverSubtext` | Centered logo text |
| `accentColor` | Hex accent used across the UI |
| `backgroundImages` | Array of image paths; empty = animated gradient only |
| `backgroundChangeInterval` | ms between image transitions |
| `musicFile` | Path to track, or `null` to hide the player |
| `musicStartVolume` | 0.0 – 1.0 |
| `discordText` / `discordLink` | Discord button |
| `websiteText` / `websiteLink` | Website button |
| `rulesTitle` / `rules[]` | Rules panel |
| `tips[]` / `tipsInterval` | Rotating tips |

### Adding images
Drop `.jpg`/`.png` files in `html/assets/images/`, then list them in
`backgroundImages`. Recommended resolution: 1920×1080 or higher.

### Adding music
Drop an `.mp3`/`.ogg` in `html/assets/audio/`, then point `musicFile` at it.
Note: browsers may block autoplay until interaction — the play button always works.

## Preview locally
Open `html/index.html` in a browser. With no FiveM host present, a mock loading
animation runs so you can see the layout and timing.

## Credits
Built by **Native Scripts**.
