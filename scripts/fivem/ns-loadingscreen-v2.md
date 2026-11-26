# ns-loadingscreen-v2

A modern loading screen for **FiveM** (GTA V). Free, standalone and fully self-contained: fonts, icons and images ship
with the resource, nothing loads from a CDN.

## Features

- **Music playlist**: as many tracks as you like, with previous / play / next, a playlist popup, volume, mute,
  optional cover art and shuffle.
- **News board**: update, event, info and notice cards with a date line.
- **Rules tab**: your server rules, one tap away from the news.
- **Staff team list**: name, role chip in your own color, and an avatar image or initials.
- **Personal welcome**: greets every player by name ("Welcome back, Alex") through FiveM's loading-screen handover.
- **Live load stages**: the status line follows the real FiveM load phases (game data, map, world, session) next to
  the percentage.
- **Rotating background**: your own screenshots with a slow zoom, or a clean gradient when you add none.
- **Rotating tips**, **Discord / website buttons** and your **logo**.
- Scales to any resolution from a 1920x1080 layout.
- One config file: `html/config.js`. No need to touch HTML, CSS or JS.

## Installation

1. Put the `ns-loadingscreen-v2` folder in your server `resources`.
2. Add it to `server.cfg`:
   ```cfg
   ensure ns-loadingscreen-v2
   ```
3. Edit `html/config.js` (name, colors, links, news, rules, staff, playlist).
4. Restart the server.

> Only one loading screen can be active. Remove or stop any other resource that declares `loadscreen`.

## Configuration (`html/config.js`)

| Key | What it does |
|---|---|
| `serverName`, `serverTagline` | Name and the small line under it |
| `logo` | Path to a logo image, or `null` |
| `accentColor` | Hex color for the progress bar, active tab and play button |
| `welcomeText` | Welcome line; `{name}` becomes the player's name. `null` hides it |
| `backgroundImages`, `backgroundInterval`, `backgroundZoom` | Background slideshow |
| `links` | Top-right buttons: `{ label, url, icon }` with icon `discord`, `globe` or `link` |
| `playlist` | Tracks: `{ title, artist, file, cover }`. Empty list hides the player |
| `musicVolume`, `musicShuffle`, `musicAutoplay` | Player start settings |
| `news` | Cards: `{ tag, title, date, text }`; tag is `update`, `event`, `info` or `warning` |
| `rules` | List of rule lines. Empty list hides the Rules tab |
| `staff` | Rows: `{ name, role, color, avatar }`. Empty list hides the panel |
| `tips`, `tipsInterval` | Rotating tips above the progress bar |

### Music

No music is bundled. Put `.mp3` or `.ogg` files you have the rights to in `html/assets/audio/` and list them in
`playlist`. Some clients block autoplay until the first click; the play button always works.

### Images

- Backgrounds: `html/assets/images/` (1920x1080 or larger).
- Staff avatars: `html/assets/staff/` (square images).

## Preview in a browser

Open `html/index.html` in a browser. Without a FiveM client a simulated load runs, so you can check your config.

## Credits

Font: Poppins by Indian Type Foundry, SIL Open Font License 1.1.

## Support

Questions or issues: open a ticket on the Native Scripts Discord.
