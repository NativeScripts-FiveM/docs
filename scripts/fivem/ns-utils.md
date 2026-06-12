# ns-utils

FiveM Lua resource'lar için **framework-agnostic `Utils.*` katmanı**.
ESX, QBCore, Qbox (qbx_core) ve standalone — hepsi tek API ile.

> Her resource'a kopyala, framework branş'ını unut, `Utils.*` üzerinden yaz.

---

## İçindekiler

- [Neden?](#neden)
- [Kurulum](#kurulum)
- [fxmanifest entegrasyonu](#fxmanifest-entegrasyonu)
- [Otomatik tespit](#otomatik-tespit)
- [API — shared.lua](#api--sharedlua)
- [API — client.lua](#api--clientlua)
- [API — server.lua](#api--serverlua)
- [Config override'ları](#config-overrideları)
- [Örnekler](#örnekler)
- [Yapılmazlar](#yapılmazlar)

---

## Neden?

Aynı scripti ESX, QB ve QBX için ayrı ayrı yazmak yerine **tek bir kod tabanı** ile üçünü de desteklemek.

```lua
-- ❌ Eski yaklaşım
if FW == 'esx' then
    ESX.GetPlayerFromId(src).addMoney(100)
elseif FW == 'qb' then
    QBCore.Functions.GetPlayer(src).Functions.AddMoney('cash', 100)
elseif FW == 'qbx' then
    exports.qbx_core:AddMoney(src, 'cash', 100)
end

-- ✅ template-utils ile
Utils.AddMoney(src, 'cash', 100)
```

Script gövdesinde framework adı geçmez. Tüm branş `utils/` içinde kalır.

---

## Kurulum

1. `utils/` klasörünü olduğu gibi yeni veya mevcut resource'a kopyala:

```
your-resource/
├── fxmanifest.lua
├── config.lua
├── client/
├── server/
└── utils/             ← buraya kopyala
    ├── shared.lua
    ├── client.lua
    └── server.lua
```

2. [fxmanifest.lua](#fxmanifest-entegrasyonu) yükleme sırasını ayarla.
3. Script gövdesinde sadece `Utils.X` çağır.

---

## fxmanifest entegrasyonu

`shared.lua` **ilk** yüklenmek zorunda — `Utils.Framework`, `Utils.Inventory` vb. tespit burada yapılır.

```lua
fx_version 'cerulean'
game 'gta5'
lua54 'yes'

shared_scripts {
    'utils/shared.lua',     -- 1️⃣ önce — Utils tablosunu kurar
    'config.lua',           -- 2️⃣ sonra — Config kullanılabilir
    -- 'utils/locale.lua',  -- (opsiyonel) _U gibi locale fonksiyonu varsa burada
}

client_scripts {
    'utils/client.lua',     -- 3️⃣ Utils client extension
    'client/*.lua',         -- 4️⃣ script kodu
}

server_scripts {
    '@oxmysql/lib/MySQL.lua',  -- (opsiyonel) oxmysql kullanılacaksa
    'utils/server.lua',        -- 5️⃣ Utils server extension
    'server/*.lua',            -- 6️⃣ script kodu
}

-- Framework HİÇBİRİ zorunlu dependency değil — runtime'da tespit ediliyor.
-- oxmysql kullanıyorsan:
dependency 'oxmysql'
```

`fxmanifest.example.lua` repo'da bu yapının daha geniş bir kopyası.

---

## Otomatik tespit

`shared.lua` başlatıldığında server console'a şunu basar:

```
[your-resource][INFO] shared.lua hazır (framework=qbx, inv=ox_inventory, target=ox_target, skin=illenium-appearance, sql=oxmysql)
```

Tespit önceliği:

| Kategori | Öncelik (önden arkaya) |
|---|---|
| **Framework** | `qbx_core` → `qb-core` → `es_extended` → `standalone` |
| **Inventory** | `ox_inventory` → `qb-inventory` → `qs-inventory` → `codem-inventory` → `ps-inventory` → `esx_inventoryhud` → `gfx-inventory` |
| **Target** | `ox_target` → `qb-target` → `qtarget` |
| **Skin** | `illenium-appearance` → `fivem-appearance` → `qb-clothing` → `esx_skin` → `skinchanger` |
| **SQL** | `oxmysql` → `ghmattimysql` → `mysql-async` |

Hiçbiri yoksa o kategori `nil` kalır — ilgili API çağrıları sessizce no-op döner.

---

## API — shared.lua

Hem client hem server'da çalışan helper'lar.

### Tespit değerleri

| Property | Tip | Örnek |
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
Utils.Info('Bootstrap tamam: %d job yüklendi', count)
Utils.Warn('Config eksik: %s', key)
Utils.Error('SQL hatası: %s', err)
Utils.Debug('Player data: %s', json.encode(pd))  -- yalnızca Config.Debug = true ise basar
```

### Vector

```lua
Utils.Dist3D(a, b)               -- vector3 mesafe
Utils.Dist2D(a, b)               -- z'yi yoksay
Utils.LerpVector(from, to, t)    -- lineer interpolation
```

### Table

```lua
Utils.DeepCopy(t)                 -- recursive cycle-safe kopya
Utils.TableContains(t, value)     -- linear search
Utils.TableSize(t)                -- hash table dahil eleman sayısı
Utils.TableIntersect(a, b)        -- iki ipairs liste kesişimi
Utils.PickRandom(t)               -- random ipairs eleman
Utils.SortedPairs(t, order?)      -- sorted iterator
```

### String

```lua
Utils.Split('a,b,c', ',')         -- {'a','b','c'}
Utils.RandomPlate(8)              -- 'ABCDEFGH'
```

### JSON

```lua
Utils.JSON.encode(obj)            -- FiveM yerleşik json wrapper
Utils.JSON.decode(str)
```

---

## API — client.lua

### Player

```lua
Utils.GetPlayerData()        -- framework'e göre PlayerData tablosu
Utils.GetIdentifier()        -- ESX: identifier / QB,QBX: citizenid
Utils.IsPlayerLoaded()       -- player tam yüklü mü
Utils.WaitForCore()          -- framework Core hazır olana kadar bekle
```

### Lifecycle

```lua
Utils.OnPlayerLoaded(function()
    -- esx:playerLoaded / QBCore:Client:OnPlayerLoaded / qbx_core:client:playerLoaded
end)
```

### Notify

```lua
Utils.Notify('Görev başladı', 'success')
Utils.Notify('Hata!', 'error', 'Trucker')   -- 3. arg: title (varsayılan resource adı)
```

Öncelik: `Config.Notify` override → `ox_lib` → framework default → native feed.

### Callback (client → server)

```lua
-- callback (cb tabanlı)
Utils.TriggerCallback('get_balance', function(balance)
    print('Para:', balance)
end, 'bank')

-- await (sync tabanlı)
local balance = Utils.AwaitCallback('get_balance', 'bank')
```

### NUI

```lua
Utils.SendReactMessage('action_name', { foo = 1 })
Utils.DebugPrint('foo', 'bar')   -- yalnızca convar `<resource>-debugMode 1` ise
```

### World

```lua
Utils.GetClosestPed(coords?, maxDist?)        -- returns ped, distance
Utils.GetClosestVehicle(coords?, maxDist?)    -- returns vehicle, distance
Utils.GetClosestPlayer(coords?, maxDist?)     -- returns playerId, distance
```

### Loaders (timeout'lu)

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
        label    = 'Menüyü Aç',
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

### Inventory (her sistem aynı imza)

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
Utils.Notify(src, 'Para hesabına yatırıldı.', 'success')
Utils.Notify(src, 'Bu eşyaya sahip değilsin.', 'error', 'Inventory')
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
    help   = 'Oyuncuya silah ver',
    params = {
        { name = 'id',     help = 'Server ID' },
        { name = 'weapon', help = 'Silah adı' },
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

> `oxmysql` kullanıyorsan `MySQL.query.await(...)` doğrudan da kullanılabilir — fxmanifest'te `@oxmysql/lib/MySQL.lua` yüklü.

### Lifecycle

```lua
Utils.OnPlayerLoaded(function(src)
    -- esx:playerLoaded / QBCore:Server:OnPlayerLoaded / qbx_core:server:onPlayerLoaded
end)

Utils.OnPlayerDropped(function(src, identifier)
    -- identifier disconnect anında alındı, kullanmak güvenli
end)
```

### Avatar (Discord / Steam)

```lua
local url = Utils.GetDiscordAvatar(src)   -- Config.DiscordBotToken zorunlu
local url = Utils.GetSteamAvatar(src)     -- public Steam profile lookup
```

---

## Config override'ları

`Config` tablonda aşağıdaki anahtarlar varsa template otomatik kullanır.

```lua
Config = {}

-- Debug log'larını aç
Config.Debug = true

-- Discord avatar API
Config.DiscordBotToken = 'xxxxxx'

-- Notify'ı override et (hem client hem server için)
Config.Notify = function(targetOrMessage, message, ntype, title)
    -- client: function(message, ntype, title)
    -- server: function(source, message, ntype, title)
end
```

---

## Örnekler

### Basit job kabul callback

```lua
-- server/jobs.lua
Utils.RegisterCallback('accept_job', function(src, jobId)
    local p = Utils.GetPlayer(src)
    if not p then return false end

    local job = Jobs[jobId]
    if not job then
        Utils.Notify(src, 'Bu iş artık mevcut değil.', 'error')
        return false
    end

    Utils.Notify(src, ('İş kabul edildi: %s'):format(job.label), 'success')
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

### Inventory + para ödemesi

```lua
Utils.RegisterCommand('sell_water', function(src)
    if not Utils.HasItem(src, 'water', 1) then
        return Utils.Notify(src, 'Suyun yok.', 'error')
    end
    Utils.RemoveItem(src, 'water', 1)
    Utils.AddMoney(src, 'cash', 10, 'water_sale')
    Utils.Notify(src, '+$10 (su satıldı)', 'success')
end)
```

### Lifecycle ile auto-save

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

## Yapılmazlar

- Script gövdesinde `if Utils.Framework == 'esx' then ...` dağıtma — tüm branş `utils/` içinde kalsın, dışarı çıkarsa template'in amacı kalmaz.
- `Core.X` / `QBCore.X` / `ESX.X` doğrudan çağırma — sadece `Utils.X`.
- `shared.lua`'yı **son** yükleme — diğer dosyalar `Utils.*` aramada `nil` hatası alır.
- `oxmysql` haricinde bir SQL kullanıyorsan `Utils.ExecuteSql` fallback'ı var ama oxmysql öneriliyor.
- ox_lib varsa otomatik notify'da kullanılır — istemiyorsan `Config.Notify` override et.

---

## Lisans

MIT — kullan, fork'la, değiştir, ne yaparsan yap.

---

**Maintainer:** [NativeScripts-FiveM](https://github.com/NativeScripts-FiveM)
