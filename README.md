# Installation Instructions for Inventory Snippet

Follow the steps below to correctly integrate the inventory snippet into your project.

### Step 1: Add the CreateDrop Function
Locate the beginning of `Core_inventory/main.lua` on the client-side and insert the following code:

```lua
-- Add the CreateDrop function here

local function CreateDrop(pedId, dropPosition, dropsName, itemName, itemID)
    local x, y, z = table.unpack(GetOffsetFromEntityInWorldCoords(pedId, 0.0, 0.5, 0.05))

    -- Вибір пропа на основі itemID
    local propName = Config.ItemProps[itemID] and Config.ItemProps[itemID].prop or Config.DropProp
    local prop = CreateObject(GetHashKey(propName), x, y, z, true, true, true)

    -- Застосування ротації, якщо це необхідно
    if Config.ItemProps[itemID] and Config.ItemProps[itemID].rotation then
        local rotation = Config.ItemProps[itemID].rotation
        SetEntityRotation(prop, rotation[1], rotation[2], rotation[3], 2, true)
    end

    PlaceObjectOnGroundProperly(prop)
    table.insert(DropsProps, { coords = dropPosition, props = prop, itemNumber = 1, name = dropsName, item = itemName or "Невідомий предмет", itemID = itemID })
    print("Drop created successfully with item:", itemName, "and prop:", propName)
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

local function Draw3DText(x, y, z, text)
    local onScreen, _x, _y = World3dToScreen2d(x, y, z)
    local px, py, pz = table.unpack(GetGameplayCamCoords())

    if onScreen then
        SetTextScale(0.35, 0.35)
        SetTextFont(4)
        SetTextProportional(1)
        SetTextColour(255, 255, 255, 215)
        SetTextEntry("STRING")
        SetTextCentre(1)
        AddTextComponentString(text)
        DrawText(_x, _y)
    end
end




Citizen.CreateThread(function()
    while true do
        local threadSleep = 1000
        if DropsProps ~= nil and #DropsProps > 0 then
            local entryToDelete = {}
            for i = 1, #DropsProps, 1 do
                if Drops[DropsProps[i].name] == nil then
                    SetEntityAsMissionEntity(DropsProps[i].props)
                    DeleteEntity(DropsProps[i].props)
                    table.insert(entryToDelete, { name = DropsProps[i].name, index = i })
                end
            end
            for index, value in ipairs(entryToDelete) do
                table.remove(DropsProps, value.index)
            end
        end

        if playerLoaded then
            local ped = PlayerPedId()
            local coords = GetEntityCoords(ped)

            if DropsProps ~= nil and #DropsProps > 0 then
                for k, v in pairs(DropsProps) do
                    if #(v.coords - coords) < Config.DropShowDistance then
                        threadSleep = 0
                        -- Використання Draw3DText замість FloatingHelpNotification
                        local itemLabel = v.item or "Невідомий предмет"
                        local itemText = Config.TextNotify.Drop:gsub("{item}", itemLabel)
                        Draw3DText(v.coords[1], v.coords[2], v.coords[3] + Config.NotifyHeightOffset, itemText)
                    end
                end
            else
                for k, v in pairs(Config.Storage) do
                    if #(v.coords - coords) < Config.DropShowDistance then
                        threadSleep = 0
                        local storageText = Config.TextNotify.Storage:gsub("{item}", v.name or "Невідоме сховище")
                        Draw3DText(v.coords[1], v.coords[2], v.coords[3] + Config.NotifyHeightOffset, storageText)
                    end
                end
            end
        end

        Citizen.Wait(threadSleep)
    end
end)

```

### Step 4: Replace `dropItem` Callback
Find the `RegisterNUICallback` for `dropItem`. Replace it with the updated version:

```lua
RegisterNUICallback("dropItem", function(data)
    -- Extract item ID
    local itemId = data['item']:match("^(.-)-")

    -- Retrieve item label
    local itemLabel = nil
    if QBCore and QBCore.Shared and QBCore.Shared.Items[itemId] then
        itemLabel = QBCore.Shared.Items[itemId].label or "Невідомий предмет"
    else
        itemLabel = "Невідомий предмет"
    end

    local dropPosition = GetOffsetFromEntityInWorldCoords(PlayerPedId(), 0.0, 0.5, 0.05)
    TriggerServerEvent('core_inventory:server:createDrop', data['item'], dropPosition)
    Citizen.Wait(200)

    local propsExist = DropPropsExist(dropPosition)
    local dropsName = nil

    if not propsExist then
        dropsName = FindDropByCoords(dropPosition)
        if dropsName == nil then
            print('Error when try to retrieve drop name')
            return
        end
    end

    if not propsExist then
        PickAndThorwObjectAnimation(PlayerPedId())
        -- Передаємо itemId разом з іншими даними
        CreateDrop(PlayerPedId(), dropPosition, dropsName, itemLabel, itemId)
        Wait(1000)
        ClearPedTasks(PlayerPedId())
    else
        UpdateDropsPropsInfo(dropsName, dropPosition, 1)
        PickAndThorwObjectAnimation(PlayerPedId())
        Wait(1000)
        ClearPedTasks(PlayerPedId())
    end
end)
```

### Step 5: Replace `changeItemLocation` Callback
Locate the `RegisterNUICallback` for `changeItemLocation`. Replace it with the updated version:

```lua
RegisterNUICallback("changeItemLocation", function(data)
  local item, inventory, slot, fromInv, itemData = data['item'], data['inventory'], data['slot'], data['fromInv'], data['itemData']
  TriggerServerEvent('core_inventory:server:changeItemLocation', item, inventory, slot, fromInv, itemData)

  if string.find(fromInv, 'drop-') then
    local props = nil
    for i = 1, #DropsProps, 1 do
      if DropsProps[i].name == fromInv then
        props = { data = DropsProps[i], index = i}
        break
    end
  end

  if props ~= nil then
    if props.data.itemNumber > 1 then
      DropsProps[i].itemNumber = DropsProps[i].itemNumber - 1
    else
      PickAndThorwObjectAnimation(PlayerPedId())
      Wait(250)
      DeleteEntity(props.data.props)
      table.remove(DropsProps, props.index)
      Wait(1000)
      ClearPedTasks(PlayerPedId())
     end
    else
      PickAndThorwObjectAnimation(PlayerPedId())                
      Wait(1000)
      ClearPedTasks(PlayerPedId())
    end
  end
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
[![Join our Discord](https://i.imgur.com/yourImageLink.png)](https://discord.gg/yourDiscordInviteLink)




