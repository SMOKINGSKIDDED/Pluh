-- Changelog: Fixed for newest da hood update
-- Built in Anti Aimviewer.
-- Automatics doesnt work cus of function.

-- // Services // --
local G                   = game
local Run_Service         = G:GetService("RunService")
local Players             = G:GetService("Players")
local UserInputService    = G:GetService("UserInputService")
local Local_Player        = Players.LocalPlayer
local Mouse               = Local_Player:GetMouse()
local Current_Camera      = G:GetService("Workspace").CurrentCamera
local Replicated_Storage  = G:GetService("ReplicatedStorage")
local StarterGui          = G:GetService("StarterGui")
local Workspace           = G:GetService("Workspace")

-- // Variables // --
local Target = nil
local V2 = Vector2.new
local Fov = Drawing.new("Circle")
local holdingMouseButton = false
local lastToolUse = 0
local HitPoint = Drawing.new("Circle")
local FovParts = {}  -- Store the parts for square and triangle FOV

-- // Game Load Check // --
if not game:IsLoaded() then
    game.Loaded:Wait()
end


-- // Clear FOV Parts // --
local function clearFovParts()
    for _, part in pairs(FovParts) do
        part:Remove()
    end
    FovParts = {}
end

-- // Update FOV Function // --
local function updateFov()
    local settings = getgenv().SharpSSilent.FovSettings
    clearFovParts()

    if settings.FovShape == "Square" then
        local halfSize = settings.FovRadius / 2
        local corners = {
            V2(Mouse.X - halfSize, Mouse.Y - halfSize),
            V2(Mouse.X + halfSize, Mouse.Y - halfSize),
            V2(Mouse.X + halfSize, Mouse.Y + halfSize),
            V2(Mouse.X - halfSize, Mouse.Y + halfSize)
        }
        for i = 1, 4 do
            local line = Drawing.new("Line")
            line.Visible = settings.FovVisible
            line.From = corners[i]
            line.To = corners[i % 4 + 1]
            line.Color = settings.FovColor
            line.Thickness = settings.FovThickness
            line.Transparency = settings.FovTransparency
            table.insert(FovParts, line)
        end
    elseif settings.FovShape == "Triangle" then
        local points = {
            V2(Mouse.X, Mouse.Y - settings.FovRadius),
            V2(Mouse.X + settings.FovRadius * math.sin(math.rad(60)), Mouse.Y + settings.FovRadius * math.cos(math.rad(60))),
            V2(Mouse.X - settings.FovRadius * math.sin(math.rad(60)), Mouse.Y + settings.FovRadius * math.cos(math.rad(60)))
        }
        for i = 1, 3 do
            local line = Drawing.new("Line")
            line.Visible = settings.FovVisible
            line.From = points[i]
            line.To = points[i % 3 + 1]
            line.Color = settings.FovColor
            line.Thickness = settings.FovThickness
            line.Transparency = settings.FovTransparency
            table.insert(FovParts, line)
        end
    else  -- Default to Circle
        Fov.Visible = settings.FovVisible
        Fov.Radius = settings.FovRadius
        Fov.Position = V2(Mouse.X, Mouse.Y + (G:GetService("GuiService"):GetGuiInset().Y))
        Fov.Color = settings.FovColor
        Fov.Thickness = settings.FovThickness
        Fov.Transparency = settings.FovTransparency
        Fov.Filled = settings.Filled
        if settings.Filled then
            Fov.Transparency = settings.FillTransparency
        end
    end
end

-- // Notification Function // --
local function sendNotification(title, text, icon)
    StarterGui:SetCore("SendNotification", {
        Title = title,
        Text = text,
        Icon = icon,
        Duration = 5
    })
end

-- // Knock Check // --
local function Death(Plr)
    if Plr.Character and Plr.Character:FindFirstChild("BodyEffects") then
        local bodyEffects = Plr.Character.BodyEffects
        local ko = bodyEffects:FindFirstChild("K.O") or bodyEffects:FindFirstChild("KO")
        return ko and ko.Value
    end
    return false
end

-- // Grab Check // --
local function Grabbed(Plr)
    return Plr.Character and Plr.Character:FindFirstChild("GRABBING_CONSTRAINT") ~= nil
end

-- // Check if Part in Fov and Visible // --
local function isPartInFovAndVisible(part)
    local screenPoint, onScreen = Current_Camera:WorldToScreenPoint(part.Position)
    local distance = (V2(screenPoint.X, screenPoint.Y) - V2(Mouse.X, Mouse.Y)).Magnitude
    return onScreen and distance <= getgenv().SharpSSilent.FovSettings.FovRadius
end

-- // Check if Part Visible // --
local function isPartVisible(part)
    if not getgenv().SharpSSilent.WallCheck then 
        return true
    end
    local origin = Current_Camera.CFrame.Position
    local direction = (part.Position - origin).Unit * (part.Position - origin).Magnitude
    local ray = Ray.new(origin, direction)
    local hit = Workspace:FindPartOnRayWithIgnoreList(ray, {Local_Player.Character, part.Parent})
    return hit == part or not hit
end

-- // Get Closest Hit Point on Part // --
local function GetClosestHitPoint(character)
    local closestPart = nil
    local closestPoint = nil
    local shortestDistance = math.huge

    for _, part in pairs(character:GetChildren()) do
        if part:IsA("BasePart") and isPartInFovAndVisible(part) and isPartVisible(part) then
            local screenPoint, onScreen = Current_Camera:WorldToScreenPoint(part.Position)
            local distance = (V2(screenPoint.X, screenPoint.Y) - V2(Mouse.X, Mouse.Y)).Magnitude

            if distance < shortestDistance then
                closestPart = part
                closestPoint = part.Position
                shortestDistance = distance
            end
        end
    end

    return closestPart, closestPoint
end

-- // Get Velocity Function // --
local OldPredictionY = getgenv().SharpSSilent.Prediction
local function GetVelocity(player, part)
    if player and player.Character then
        local velocity = player.Character[part].Velocity
        if velocity.Y < -30 and getgenv().SharpSSilent.Resolver then
            getgenv().SharpSSilent.Prediction = 0
            return velocity
        elseif velocity.Magnitude > 50 and getgenv().SharpSSilent.Resolver then
            return player.Character:FindFirstChild("Humanoid").MoveDirection * 16
        else
            getgenv().SharpSSilent.Prediction = OldPredictionY
            return velocity
        end
    end
    return Vector3.new(0, 0, 0)
end

-- // Get Closest Player // --
local function GetClosestPlr()
    local closestTarget = nil
    local maxDistance = math.huge

    for _, player in pairs(Players:GetPlayers()) do
        if player.Character and player ~= Local_Player and not Death(player) then  -- KO check using Death function
            local closestPart, closestPoint = GetClosestHitPoint(player.Character)
            if closestPart and closestPoint then
                local screenPoint = Current_Camera:WorldToScreenPoint(closestPoint)
                local distance = (V2(screenPoint.X, screenPoint.Y) - V2(Mouse.X, Mouse.Y)).Magnitude
                if distance < maxDistance then
                    maxDistance = distance
                    closestTarget = player
                end
            end
        end
    end

    return closestTarget
end

-- // Toggle Feature // --
local function toggleFeature()
    getgenv().SharpSSilent.Enabled = not getgenv().SharpSSilent.Enabled
    local status = getgenv().SharpSSilent.Enabled and "Sharp Enabled" or "Sharp Disabled"
    sendNotification("Sharp [S] Silent", status, "rbxassetid://17561420493")
    if not getgenv().SharpSSilent.Enabled then
        Fov.Visible = false
        HitPoint.Visible = false
        clearFovParts()
    end
end

-- // Convert Keybind to KeyCode // --
local function getKeyCodeFromString(key)
    return Enum.KeyCode[key]
end

-- // Keybind Listener // --
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == getKeyCodeFromString(getgenv().SharpSSilent.Keybind) then
        toggleFeature()
    elseif input.UserInputType == Enum.UserInputType.MouseButton1 then
        holdingMouseButton = true
    end
end)

UserInputService.InputEnded:Connect(function(input, gameProcessed)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        holdingMouseButton = false
    end
end)

-- // Main Loop // --
Run_Service.RenderStepped:Connect(function()
    if getgenv().SharpSSilent.Enabled then
        Target = GetClosestPlr()
        updateFov()
        if Target and Target.Character then
            local closestPart, closestPoint = GetClosestHitPoint(Target.Character)
            if closestPart and closestPoint then
                local hitPointSettings = getgenv().SharpSSilent.HitPoint
                if hitPointSettings.ShowHitPoint then
                    HitPoint.Visible = true
                    local screenPoint = Current_Camera:WorldToViewportPoint(closestPoint)
                    HitPoint.Position = V2(screenPoint.X, screenPoint.Y)
                    HitPoint.Color = hitPointSettings.HitPointColor
                    HitPoint.Radius = hitPointSettings.HitPointRadius
                    HitPoint.Thickness = hitPointSettings.HitPointThickness
                    HitPoint.Transparency = hitPointSettings.HitPointTransparency
                    HitPoint.Filled = true
                else
                    HitPoint.Visible = false
                end

                if holdingMouseButton then
                    local velocity = GetVelocity(Target, closestPart.Name)
                    Replicated_Storage.MainEvent:FireServer("UpdateMousePosI", closestPoint + velocity * getgenv().SharpSSilent.Prediction)
                end
            end
        else
            HitPoint.Visible = false
        end
    end
end)

-- // Hook Tool Activation // --
local function HookTool(tool)
    if tool:IsA("Tool") then
        tool.Activated:Connect(function()
            if Target and Target.Character and tick() - lastToolUse > 0.1 then  -- Debounce for 0.1 seconds
                lastToolUse = tick()
                local closestPart, closestPoint = GetClosestHitPoint(Target.Character)
                if closestPart and closestPoint then
                    local velocity = GetVelocity(Target, closestPart.Name)
                    Replicated_Storage.MainEvent:FireServer("UpdateMousePosI", closestPoint + velocity * getgenv().SharpSSilent.Prediction)
                end
            end
        end)
    end
end

local function onCharacterAdded(character)
    character.ChildAdded:Connect(HookTool)
    for _, tool in pairs(character:GetChildren()) do
        HookTool(tool)
    end
end

Local_Player.CharacterAdded:Connect(onCharacterAdded)
if Local_Player.Character then
    onCharacterAdded(Local_Player.Character)
end

-- // Initial Notification // --
sendNotification("dc.gg/sharpcc", "♣️ #1 Triggerbot", "rbxassetid://17561420493")
wait(3)
sendNotification("dc.gg/sharpcc", "♣️ Supports Solara", "rbxassetid://17561420493")
--[[ 

Go get to Skidding !! Source Made By @wxrpedd & @canyoulovemeback on Discord,
Everything is from scrap took us like 10 mins to make this so have fun.

--]]
--[[

 ________  ___  ___  ________  ________  ________   ________  ________     
|\   ____\|\  \|\  \|\   __  \|\   __  \|\   __  \ |\   ____\|\   ____\    
\ \  \___|\ \  \\\  \ \  \|\  \ \  \|\  \ \  \|\  \\ \  \___|\ \  \___|    
 \ \_____  \ \   __  \ \   __  \ \   _  _\ \   ____\\ \  \    \ \  \       
  \|____|\  \ \  \ \  \ \  \ \  \ \  \\  \\ \  \___|_\ \  \____\ \  \____  
    ____\_\  \ \__\ \__\ \__\ \__\ \__\\ _\\ \__\ |\__\ \_______\ \_______\
   |\_________\|__|\|__|\|__|\|__|\|__|\|__|\|__| \|__|\|_______|\|_______|
   \|_________|                                                            

                                                                    
discord.gg/sharpcc
supercoolboi34 @canyoulovemeback
Kai @wxrpedd
--]]
