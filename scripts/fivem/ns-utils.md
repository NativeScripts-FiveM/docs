# ns-utils

A **framework-agnostic `Utils.*` layer** for FiveM Lua resources.
ESX, QBCore, Qbox (qbx_core) and standalone — all behind one API.

> Copy it into each resource, forget framework branching, write against `Utils.*`.

---

## Contents

- [Why?](#why)
- [Installation](#installation)
- [fxmanifest integration](#fxmanifest-integration)
- [Auto-detection](#auto-detection)
- [API — shared.lua](#api--sharedlua)
- [API — client.lua](#api--clientlua)
- [API — server.lua](#api--serverlua)
- [Config overrides](#config-overrides)
- [Examples](#examples)
- [Don'ts](#donts)

---

## Why?

Instead of writing the same script three separate times for ESX, QB and QBX, support all three from **a single codebase**.

```lua
-- ❌ Old way
if FW == 'esx' then
    ESX.GetPlayerFromId(src).addMoney(100)
elseif FW == 'qb' then
    QBCore.Functions.GetPlayer(src).Functions.AddMoney('cash', 100)
elseif FW == 'qbx' then
    exports.qbx_core:AddMoney(src, 'cash', 100)
end

-- ✅ with ns-utils
Utils.AddMoney(src, 'cash', 100)
```

The framework name never appears in your script body. All branching stays inside `utils/`.

---

## Installation

1. Copy the `utils/` folder as-is into a new or existing resource:

```
your-resource/
├── fxmanifest.lua
├── config.lua
├── client/
├── server/
└── utils/             ← copy here
    ├── shared.lua
    ├── client.lua
    └── server.lua
```

2. Set up the load order in [fxmanifest.lua](#fxmanifest-integration).
3. In your script body, only ever call `Utils.X`.

---

## fxmanifest integration

`shared.lua` MUST load **first** — `Utils.Framework`, `Utils.Inventory` etc. are detected there.

```lua
fx_version 'cerulean'
game 'gta5'
lua54 'yes'

shared_scripts {
    'utils/shared.lua',     -- 1️⃣ first — sets up the Utils table
    'config.lua',           -- 2️⃣ then — Config becomes available
    -- 'utils/locale.lua',  -- (optional) put a locale fn like _U here if you have one
}

client_scripts {
    'utils/client.lua',     -- 3️⃣ Utils client extension
    'client/*.lua',         -- 4️⃣ your script code
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',  -- (optional) if you use oxmysql
    'utils/server.lua',        -- 5️⃣ Utils server extension
    'server/*.lua',            -- 6️⃣ your script code
}

-- NO framework is a hard dependency — they are detected at runtime.
-- If you use oxmysql:
dependency 'oxmysql'
```

`fxmanifest.example.lua` in the repo is a fuller copy of this layout.

---

## Auto-detection

When `shared.lua` initializes it prints this to the server console:

```
[your-resource][INFO] shared.lua ready (framework=qbx, inv=ox_inventory, target=ox_target, skin=illenium-appearance, sql=oxmysql)
```

Detection priority:

| Category | Priority (first match wins) |
|---|---|
| **Framework** | `qbx_core` → `qb-core` → `es_extended` → `standalone` |
| **Inventory** | `ox_inventory` → `qb-inventory` → `qs-inventory` → `codem-inventory` → `ps-inventory` → `esx_inventoryhud` → `gfx-inventory` |
| **Target** | `ox_target` → `qb-target` → `qtarget` |
| **Skin** | `illenium-appearance` → `fivem-appearance` → `qb-clothing` → `esx_skin` → `skinchanger` |
| **SQL** | `oxmysql` → `ghmattimysql` → `mysql-async` |

If none are present, that category stays `nil` — the related API calls silently no-op.

---

## API — shared.lua

Helpers that run on both client and server.

### Detection values

| Property | Type | Example |
|---|---|---|
| `Utils.Framework` | string | `'esx'`, `'qb'`, `'qbx'`, `'standalone'` |
| `Utils.Inventory` | string?| `'ox_inventory'` |
| `Utils.Target` | string?| `'ox_target'` |
| `Utils.Skin` | string?| `'illenium-appearance'` |
| `Utils.SQL` | string?| `'oxmysql'` |
| `Utils.ResourceName` | string | `GetCurrentResourceName()` |
| `Utils.IsServer` / `Utils.IsClient` | boolean | runtime context |

### Logger

```lua
Utils.Info('Bootstrap done: %d jobs loaded', count)
Utils.Warn('Missing config: %s', key)
Utils.Error('SQL error: %s', err)
Utils.Debug('Player data: %s', json.encode(pd))  -- only prints if Config.Debug = true
```

### Vector

```lua
Utils.Dist3D(a, b)               -- vector3 distance
Utils.Dist2D(a, b)               -- ignore z
Utils.LerpVector(from, to, t)    -- linear interpolation
```

### Table

```lua
Utils.DeepCopy(t)                 -- recursive cycle-safe copy
Utils.TableContains(t, value)     -- linear search
Utils.TableSize(t)                -- element count, hash tables included
Utils.TableIntersect(a, b)        -- intersection of two ipairs lists
Utils.PickRandom(t)               -- random ipairs element
Utils.SortedPairs(t, order?)      -- sorted iterator
```

### String

```lua
Utils.Split('a,b,c', ',')         -- {'a','b','c'}
Utils.RandomPlate(8)              -- 'ABCDEFGH'
```

### JSON

```lua
Utils.JSON.encode(obj)            -- FiveM built-in json wrapper
Utils.JSON.decode(str)
```

---

## API — client.lua

### Player

```lua
Utils.GetPlayerData()        -- PlayerData table per framework
Utils.GetIdentifier()        -- ESX: identifier / QB,QBX: citizenid
Utils.IsPlayerLoaded()       -- whether the player is fully loaded
Utils.WaitForCore()          -- wait until the framework Core is ready
```

### Lifecycle

```lua
Utils.OnPlayerLoaded(function()
    -- esx:playerLoaded / QBCore:Client:OnPlayerLoaded / qbx_core:client:playerLoaded
end)
```

### Notify

```lua
Utils.Notify('Job started', 'success')
Utils.Notify('Error!', 'error', 'Trucker')   -- 3rd arg: title (defaults to resource name)
```

Priority: `Config.Notify` override → `ox_lib` → framework default → native feed.

### Callback (client → server)

```lua
-- callback (cb-based)
Utils.TriggerCallback('get_balance', function(balance)
    print('Money:', balance)
end, 'bank')

-- await (sync-based)
local balance = Utils.AwaitCallback('get_balance', 'bank')
```

### NUI

```lua
Utils.SendReactMessage('action_name', { foo = 1 })
Utils.DebugPrint('foo', 'bar')   -- only when convar `<resource>-debugMode 1` is set
```

### World

```lua
Utils.GetClosestPed(coords?, maxDist?)        -- returns ped, distance
Utils.GetClosestVehicle(coords?, maxDist?)    -- returns vehicle, distance
Utils.GetClosestPlayer(coords?, maxDist?)     -- returns playerId, distance
```

### Loaders (with timeout)

```lua
Utils.LoadAnimDict('amb@world_human_smoking@male@male_a@base', 5000)
local hash = Utils.LoadModel('adder', 5000)
Utils.LoadPtfx('core', 3000)
Utils.PlayPedAnim(ped, dict, anim, flag, speed, duration, playbackRate)
```

### Entity

```lua
local ped = Utils.CreatePedOnCoord(vec4(x, y, z, heading), 'a_m_y_business_01')
local blip = Utils.CreateBlipOnCoords(vec3(x, y, z), 1, 2, 'Job Site', 0.8)
```

### Drawing

```lua
Utils.DrawText3D(x, y, z, 'Press E', 4, 0.4)
Utils.DrawText2D(0.5, 0.5, 'Center text')
```

### Target (ox_target / qb-target / qtarget)

```lua
Utils.AddTargetEntity(entity, {
    {
        name     = 'open_menu',
        label    = 'Open menu',
        icon     = 'fa-solid fa-list',
        event    = 'myresource:openMenu',
        action   = function(entity) print('clicked') end,
        distance = 2.5,
        canInteract = function(entity) return true end,
    },
})

Utils.AddTargetModel(`prop_atm_01`, { ... })
```

### Skin (illenium / fivem-appearance / qb-clothing / esx_skin / skinchanger)

```lua
Utils.SetSkin(PlayerPedId(), skinTable)
```

---

## API — server.lua

### Player lookup

```lua
local p = Utils.GetPlayer(src)                       -- framework player object
local p = Utils.GetPlayerByIdentifier(citizenid)     -- ESX: identifier / QB,QBX: citizenid
local id = Utils.GetIdentifier(src)
local name = Utils.GetName(src)
local list = Utils.GetPlayers()
local job = Utils.GetJob(src)  -- { name, label, grade, grade_name }
```

### Money

```lua
local cash = Utils.GetMoney(src, 'cash')
local bank = Utils.GetMoney(src, 'bank')

Utils.AddMoney(src, 'cash', 500, 'job_reward')
Utils.RemoveMoney(src, 'bank', 200)
Utils.HasMoney(src, 100, 'bank')
```

### Inventory (same signature for every system)

```lua
Utils.AddItem(src, 'water', 1, metadata?, slot?)
Utils.RemoveItem(src, 'water', 1, metadata?, slot?)
Utils.GetItemCount(src, 'water')
Utils.HasItem(src, 'water', 1)
local items = Utils.GetInventory(src)

Utils.RegisterUsableItem('water', function(src) ... end)
```

### Notify (server → client)

```lua
Utils.Notify(src, 'Money deposited to your account.', 'success')
Utils.Notify(src, 'You do not have this item.', 'error', 'Inventory')
```

### Callback (request handler)

```lua
Utils.RegisterCallback('get_balance', function(src, account)
    local p = Utils.GetPlayer(src)
    return p and Utils.GetMoney(src, account) or 0
end)
```

### Permissions

```lua
if Utils.HasPermission(src, 'admin') then ... end
```

### Commands

```lua
Utils.RegisterCommand('giveweapon', function(src, args)
    -- ...
end, true, {  -- restricted (admin) + chat suggestion
    help   = 'Give a weapon to a player',
    params = {
        { name = 'id',     help = 'Server ID' },
        { name = 'weapon', help = 'Weapon name' },
    },
})
```

### SQL

```lua
-- callback style (async)
Utils.ExecuteSql('SELECT * FROM users WHERE identifier = ?', { id }, function(rows)
    print(#rows)
end)

-- sync style (return)
local rows = Utils.ExecuteSql('SELECT * FROM users WHERE identifier = ?', { id })
```

> If you use `oxmysql` you can also call `MySQL.query.await(...)` directly — `@oxmysql/lib/MySQL.lua` is loaded in the fxmanifest.

### Lifecycle

```lua
Utils.OnPlayerLoaded(function(src)
    -- esx:playerLoaded / QBCore:Server:OnPlayerLoaded / qbx_core:server:onPlayerLoaded
end)

Utils.OnPlayerDropped(function(src, identifier)
    -- identifier captured at disconnect time, safe to use
end)
```

### Avatar (Discord / Steam)

```lua
local url = Utils.GetDiscordAvatar(src)   -- requires Config.DiscordBotToken
local url = Utils.GetSteamAvatar(src)     -- public Steam profile lookup
```

---

## Config overrides

If your `Config` table has the following keys, the template uses them automatically.

```lua
Config = {}

-- Enable debug logs
Config.Debug = true

-- Discord avatar API
Config.DiscordBotToken = 'xxxxxx'

-- Override Notify (for both client and server)
Config.Notify = function(targetOrMessage, message, ntype, title)
    -- client: function(message, ntype, title)
    -- server: function(source, message, ntype, title)
end
```

---

## Examples

### Simple job-accept callback

```lua
-- server/jobs.lua
Utils.RegisterCallback('accept_job', function(src, jobId)
    local p = Utils.GetPlayer(src)
    if not p then return false end

    local job = Jobs[jobId]
    if not job then
        Utils.Notify(src, 'This job is no longer available.', 'error')
        return false
    end

    Utils.Notify(src, ('Job accepted: %s'):format(job.label), 'success')
    return true
end)
```

```lua
-- client/job_state.lua
local ok = Utils.AwaitCallback('accept_job', selectedJobId)
if ok then
    spawnVehicleAndStart()
end
```

### Inventory + money payout

```lua
Utils.RegisterCommand('sell_water', function(src)
    if not Utils.HasItem(src, 'water', 1) then
        return Utils.Notify(src, 'You have no water.', 'error')
    end
    Utils.RemoveItem(src, 'water', 1)
    Utils.AddMoney(src, 'cash', 10, 'water_sale')
    Utils.Notify(src, '+$10 (water sold)', 'success')
end)
```

### Auto-save on lifecycle

```lua
Utils.OnPlayerDropped(function(src, identifier)
    if not identifier then return end
    local state = ActiveJobs[identifier]
    if state then
        MySQL.update.await(
            'UPDATE jobs SET state = ? WHERE citizenid = ?',
            { json.encode(state), identifier }
        )
        ActiveJobs[identifier] = nil
    end
end)
```

---

## Don'ts

- Don't scatter `if Utils.Framework == 'esx' then ...` across your script body — keep all branching inside `utils/`, otherwise the whole point of the layer is lost.
- Don't call `Core.X` / `QBCore.X` / `ESX.X` directly — only `Utils.X`.
- Don't load `shared.lua` **last** — the other files will hit `nil` errors looking up `Utils.*`.
- If you use a SQL resource other than `oxmysql`, `Utils.ExecuteSql` has fallbacks, but oxmysql is recommended.
- If ox_lib is present it's used automatically for notifications — override `Config.Notify` if you don't want that.

---

## License

MIT — use it, fork it, change it, do whatever.

---

**Maintainer:** [NativeScripts-FiveM](https://github.com/NativeScripts-FiveM)
