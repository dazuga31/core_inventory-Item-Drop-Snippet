# Installation Instructions for Inventory Snippet

Follow the steps below to correctly integrate the inventory snippet into your project.

### Step 1: Add the CreateDrop Function
Locate the beginning of `Core_inventory/main.lua` on the client-side and insert the following code:

```lua
-- Add the CreateDrop function here

local function CreateDrop(pedId, dropPosition, dropsName)
    -- Function content
end
```

### Step 2: Initialize `DropsProps`
Find the definition of `Drops = { }` (around line 42). Right after it, add:

```lua
DropsProps = { }
```

### Step 3: Add the Drop Loop
Find the comment `-- Drop Loop` just before the `Citizen.CreateThread` definition. After this thread creation, add the following code:

```lua
-- Add the Drop Loop code here

Citizen.CreateThread(function()
    -- Thread content
end)
```

### Step 4: Replace `dropItem` Callback
Find the `RegisterNUICallback` for `dropItem`. Replace it with the updated version:

```lua
RegisterNUICallback("dropItem", function(data)
    -- Updated dropItem callback code
end)
```

### Step 5: Replace `changeItemLocation` Callback
Locate the `RegisterNUICallback` for `changeItemLocation`. Replace it with the updated version:

```lua
RegisterNUICallback("changeItemLocation", function(data)
    -- Updated changeItemLocation callback code
end)
```

### Step 6: Modify `Config.lua`
Update the `Config.lua` file with the following settings:

```lua
------------------------
-- INVENTORY SETTINGS --
------------------------
OpenKey = 'tab', -- The key to open inventory
DropShowDistance = 5.0, -- Distance where marker will be displayed for drop and storage
DropProp = 'v_res_fa_cereal02', -- Default drop prop
NotifyHeightOffset = -0.7, -- Height offset for notifications

TextNotify = {
    Drop = "Предмет: {item}",
    Storage = "Сховище: {item}"
},

ItemProps = {
    -- Phones
    white_phone = { prop = 'prop_v_m_phone_o1s', rotation = {0.0, 0.0, 0.0} },
    yellow_phone = { prop = 'prop_prologue_phone', rotation = {90.0, 0.0, 0.0} },
    green_phone = { prop = 'sf_prop_sf_npc_phone_01a', rotation = {45.0, 0.0, 0.0} },
    newsmic = { prop = 'p_ing_microphonel_01', rotation = {0.0, 90.0, 0.0} },

    -- Ammo
    smg_ammo = { prop = 'v_ret_gc_ammo5', rotation = nil },
    shotgun_ammo = { prop = 'v_ret_gc_ammo3', rotation = nil },
    rifle_ammo = { prop = 'v_ret_gc_ammo4', rotation = nil },
    pistol_ammo = { prop = 'v_ret_gc_ammo5', rotation = nil },

    -- Engines
    toyota_tacoma_5_engine = { prop = 'prop_car_engine_01', rotation = nil },
    bison2_1_engine = { prop = 'imp_prop_impexp_engine_part_01a', rotation = nil }
}
```

### Notes:
- Ensure you save all changes before restarting your server.
- Double-check for syntax errors when copying and pasting the code.
- If you encounter issues, verify the locations and file structure to ensure compatibility with your current setup.

For further assistance, feel free to reach out to the support team or check the documentation.

