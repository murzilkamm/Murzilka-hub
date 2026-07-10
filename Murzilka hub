--[[
    MURZIK HUB v3.0
    DELTA ULTIMATE - ВСЕ ФУНКЦИИ В ОДНОМ СКРИПТЕ
    Разработчик: Murzik
--]]

local Workspace = game:GetService("Workspace")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local Stats = game:GetService("Stats")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Debris = game:GetService("Debris")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

-- ============================================
-- ===== СОХРАНЕНИЕ НАСТРОЕК =====
-- ============================================

local SaveKey = "MurzikHubSettings"
local function saveSettings()
    pcall(function()
        local data = HttpService:JSONEncode(Settings)
        setclipboard and setclipboard(data)
        if shared then shared[SaveKey] = Settings end
    end)
end

local function loadSettings()
    pcall(function()
        if shared and shared[SaveKey] then
            for k, v in pairs(shared[SaveKey]) do
                if Settings[k] ~= nil then Settings[k] = v end
            end
        end
    end)
end

-- ============================================
-- ===== ПАРАМЕТРЫ =====
-- ============================================

local Settings = {
    -- Основные
    Enabled = true,
    Speed = 4.0,
    Prediction = 0.6,
    Range = 100,
    Color = "Red",
    
    -- Флинг
    FlingPower = 80,
    FlingCooldown = 0.5,
    FlingMode = 1,
    FlingAOE = 30,
    
    -- Флай
    FlyEnabled = false,
    FlySpeed = 50,
    
    -- Движение
    WalkSpeed = 16,
    JumpPower = 50,
    
    -- UI
    ShowStats = true,
    UIBackgroundColor = Color3.new(0.05, 0.05, 0.1),
    UIBackgroundTransparency = 0.15,
    UIBackgroundGradient = false,
    UIBackgroundGradientColor = Color3.new(0.3, 0, 0.3),
    
    -- Эффекты
    TrailEnabled = true,
    ParticlesEnabled = true,
    SoundsEnabled = true,
    
    -- Байпас
    BypassEnabled = true
}

loadSettings()

local lastFlingTime = 0
local flyBodyVelocity = nil
local flyBodyGyro = nil

-- ============================================
-- ===== АНТИЧИТ-БАЙПАС =====
-- ============================================

local function initBypass()
    pcall(function()
        if ReplicatedStorage then
            for _, child in ipairs(game:GetDescendants()) do
                if child:IsA("RemoteEvent") then
                    local oldFire = child.FireServer
                    child.FireServer = function(self, ...)
                        local args = {...}
                        for i, arg in ipairs(args) do
                            if type(arg) == "table" and (arg.Velocity or arg.Position or arg.CFrame) then
                                args[i] = nil
                            end
                        end
                        return oldFire(self, unpack(args))
                    end
                end
                if child:IsA("RemoteFunction") then
                    local oldInvoke = child.InvokeServer
                    child.InvokeServer = function(self, ...)
                        local args = {...}
                        for i, arg in ipairs(args) do
                            if type(arg) == "table" and (arg.Velocity or arg.Position or arg.CFrame) then
                                args[i] = nil
                            end
                        end
                        return oldInvoke(self, unpack(args))
                    end
                end
            end
        end
        
        local oldWarn = warn
        warn = function(...)
            local msg = tostring(...)
            if string.find(msg, "anticheat") or string.find(msg, "exploit") or 
               string.find(msg, "velocity") or string.find(msg, "detected") or
               string.find(msg, "injection") then
                return nil
            end
            oldWarn(...)
        end
        
        local oldGetChildren = game.GetChildren
        game.GetChildren = function(self)
            local children = oldGetChildren(self)
            if self == Workspace then
                for i, child in ipairs(children) do
                    if child == ball then table.remove(children, i) break end
                end
            end
            return children
        end
        
        local oldFindFirstChild = game.FindFirstChild
        game.FindFirstChild = function(self, name, ...)
            if name == ball and self == Workspace then return nil end
            return oldFindFirstChild(self, name, ...)
        end
    end)
end

-- ============================================
-- ===== СОЗДАНИЕ ШАРА =====
-- ============================================

local ball = Instance.new("Part")
ball.Size = Vector3.new(4, 4, 4)
ball.Shape = Enum.PartType.Ball
ball.BrickColor = BrickColor.new("Bright red")
ball.Material = Enum.Material.SmoothPlastic
ball.Anchored = true
ball.CanCollide = false
ball.CanQuery = false
ball.Position = character.PrimaryPart.Position + Vector3.new(0, 5, 0)
ball.Parent = Workspace

local glow = Instance.new("PointLight")
glow.Parent = ball
glow.Color = Color3.new(1, 0, 0)
glow.Range = 25
glow.Brightness = 4

-- ============================================
-- ===== ЭФФЕКТЫ: ТРЕЙЛ =====
-- ============================================

local trailParts = {}
local function createTrail()
    if not Settings.TrailEnabled then return end
    
    local trail = Instance.new("Part")
    trail.Size = Vector3.new(1, 1, 1)
    trail.Shape = Enum.PartType.Ball
    trail.BrickColor = ball.BrickColor
    trail.Material = Enum.Material.Neon
    trail.Anchored = true
    trail.CanCollide = false
    trail.CanQuery = false
    trail.Position = ball.Position
    trail.Transparency = 0.8
    trail.Parent = Workspace
    
    table.insert(trailParts, trail)
    Debris:AddItem(trail, 0.5)
    
    spawn(function()
        for i = 1, 10 do
            task.wait(0.05)
            trail.Transparency = trail.Transparency + 0.08
            trail.Size = trail.Size - Vector3.new(0.1, 0.1, 0.1)
        end
        trail:Destroy()
    end)
end

-- ============================================
-- ===== ЭФФЕКТЫ: ПАРТИКЛЫ =====
-- ============================================

local function createParticles(position, color, count)
    if not Settings.ParticlesEnabled then return end
    
    for i = 1, count or 15 do
        local particle = Instance.new("Part")
        particle.Size = Vector3.new(math.random(1, 3), math.random(1, 3), math.random(1, 3))
        particle.Shape = Enum.PartType.Ball
        particle.BrickColor = BrickColor.new(color or "Bright yellow")
        particle.Material = Enum.Material.Neon
        particle.Anchored = true
        particle.CanCollide = false
        particle.CanQuery = false
        particle.Position = position + Vector3.new(math.random(-3, 3), math.random(-3, 3), math.random(-3, 3))
        particle.Parent = Workspace
        
        local velocity = Vector3.new(math.random(-20, 20), math.random(10, 30), math.random(-20, 20))
        Debris:AddItem(particle, 1)
        
        spawn(function()
            for j = 1, 20 do
                task.wait(0.03)
                particle.Position = particle.Position + velocity * 0.03
                particle.Transparency = j / 20
                particle.Size = particle.Size - Vector3.new(0.05, 0.05, 0.05)
                velocity = velocity + Vector3.new(0, -0.5, 0)
            end
            particle:Destroy()
        end)
    end
end

-- ============================================
-- ===== ЭФФЕКТЫ: ЗВУКИ =====
-- ============================================

local function playSound(soundId)
    if not Settings.SoundsEnabled then return end
    
    pcall(function()
        local sound = Instance.new("Sound")
        sound.SoundId = "rbxassetid://" .. soundId
        sound.Volume = 0.5
        sound.Parent = workspace
        sound.PlayOnRemove = true
        sound:Play()
        Debris:AddItem(sound, 2)
    end)
end

-- ============================================
-- ===== РЕЖИМЫ ФЛИНГА =====
-- ============================================

local flingModes = {"ОДИН", "ВВЕРХ", "AOE"}

local function flingPlayer(targetChar)
    if not targetChar or not targetChar.PrimaryPart then return end
    local now = tick()
    if now - lastFlingTime < Settings.FlingCooldown then return end
    lastFlingTime = now
    
    local rootPart = targetChar.PrimaryPart
    local direction = Vector3.new(0, 0, 0)
    
    if Settings.FlingMode == 1 then
        direction = Vector3.new(math.random(-2, 2), 1.5, math.random(-2, 2)).Unit
    elseif Settings.FlingMode == 2 then
        direction = Vector3.new(0, 2, 0).Unit
    elseif Settings.FlingMode == 3 then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= player then
                local char = plr.Character
                if char and char.PrimaryPart then
                    local dist = (char.PrimaryPart.Position - rootPart.Position).Magnitude
                    if dist < Settings.FlingAOE and char ~= targetChar then
                        flingPlayer(char)
                    end
                end
            end
        end
        direction = Vector3.new(math.random(-2, 2), 1.5, math.random(-2, 2)).Unit
    end
    
    local velocity = direction * Settings.FlingPower
    
    spawn(function()
        for i = 1, 3 do
            task.wait(0.02)
            rootPart.Velocity = rootPart.Velocity:Lerp(velocity, 0.3)
        end
    end)
    
    for _, part in ipairs(targetChar:GetChildren()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
            part.Velocity = velocity * 0.3 + Vector3.new(math.random(-10, 10), math.random(5, 20), math.random(-10, 10))
        end
    end
    
    createParticles(rootPart.Position, "Bright yellow", 20)
    playSound("9120385868")
end

-- ============================================
-- ===== ФУНКЦИИ ФЛАЯ =====
-- ============================================

local function toggleFly()
    Settings.FlyEnabled = not Settings.FlyEnabled
    
    if Settings.FlyEnabled then
        local humanoid = character:FindFirstChild("Humanoid")
        if humanoid then humanoid.PlatformStand = true end
        
        flyBodyVelocity = Instance.new("BodyVelocity")
        flyBodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)
        flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
        flyBodyVelocity.Parent = character.PrimaryPart
        
        flyBodyGyro = Instance.new("BodyGyro")
        flyBodyGyro.MaxTorque = Vector3.new(4000, 4000, 4000)
        flyBodyGyro.Parent = character.PrimaryPart
        
        player:Chat("✈️ Полёт включён")
        createParticles(character.PrimaryPart.Position, "Bright cyan", 10)
    else
        if flyBodyVelocity then flyBodyVelocity:Destroy() flyBodyVelocity = nil end
        if flyBodyGyro then flyBodyGyro:Destroy() flyBodyGyro = nil end
        
        local humanoid = character:FindFirstChild("Humanoid")
        if humanoid then humanoid.PlatformStand = false end
        
        player:Chat("✈️ Полёт выключен")
        createParticles(character.PrimaryPart.Position, "Bright red", 10)
    end
end

-- ============================================
-- ===== ПРИМЕНЕНИЕ НАСТРОЕК =====
-- ============================================

local function applyMovementSettings()
    local humanoid = character:FindFirstChild("Humanoid")
    if humanoid then
        humanoid.WalkSpeed = Settings.WalkSpeed
        humanoid.JumpPower = Settings.JumpPower
    end
end

applyMovementSettings()

player.CharacterAdded:Connect(function(newChar)
    character = newChar
    task.wait(0.5)
    applyMovementSettings()
    if Settings.FlyEnabled then
        Settings.FlyEnabled = false
        if flyBodyVelocity then flyBodyVelocity:Destroy() flyBodyVelocity = nil end
        if flyBodyGyro then flyBodyGyro:Destroy() flyBodyGyro = nil end
    end
end)

-- ============================================
-- ===== ПОИСК БЛИЖАЙШЕЙ ЦЕЛИ =====
-- ============================================

local function getClosestTarget()
    local closest, minDist = nil, math.huge
    local origin = ball.Position
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= player then
            local char = plr.Character
            if char and char.PrimaryPart then
                local dist = (char.PrimaryPart.Position - origin).Magnitude
                if dist < minDist and dist < Settings.Range then
                    minDist = dist
                    closest = char
                end
            end
        end
    end
    return closest
end

-- ============================================
-- ===== ТЕЛЕПОРТ ИГРОКА К СЕБЕ =====
-- ============================================

local function teleportPlayerToMe(targetPlayer)
    if not targetPlayer then return end
    
    local targetChar = targetPlayer.Character
    if not targetChar or not targetChar.PrimaryPart then
        player:Chat("Игрок не найден")
        return
    end
    
    if not character or not character.PrimaryPart then
        player:Chat("Вы не в игре")
        return
    end
    
    local myPosition = character.PrimaryPart.Position + Vector3.new(0, 2, 0)
    targetChar.PrimaryPart.CFrame = CFrame.new(myPosition)
    
    createParticles(myPosition, "Bright magenta", 25)
    createParticles(character.PrimaryPart.Position, "Bright cyan", 15)
    playSound("9120385868")
    player:Chat("Телепорт " .. targetPlayer.Name .. " к себе")
end

-- ============================================
-- ===== ТЕЛЕПОРТ ВСЕХ К СЕБЕ =====
-- ============================================

local function teleportAllToMe()
    if not character or not character.PrimaryPart then
        player:Chat("Вы не в игре")
        return
    end
    
    local myPosition = character.PrimaryPart.Position + Vector3.new(0, 2, 0)
    local count = 0
    
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= player then
            local targetChar = plr.Character
            if targetChar and targetChar.PrimaryPart then
                targetChar.PrimaryPart.CFrame = CFrame.new(myPosition + Vector3.new(math.random(-2, 2), 0, math.random(-2, 2)))
                createParticles(targetChar.PrimaryPart.Position, "Bright magenta", 10)
                count = count + 1
                task.wait(0.05)
            end
        end
    end
    
    createParticles(character.PrimaryPart.Position, "Bright cyan", 30)
    playSound("9120385868")
    player:Chat("Телепортировано " .. count .. " игроков к себе")
end

-- ============================================
-- ===== ЧАТ-КОМАНДЫ =====
-- ============================================

local function handleChatCommand(msg)
    msg = string.lower(msg)
    
    if msg == "/fly" then
        toggleFly()
        return true
    elseif msg == "/fling" then
        local target = getClosestTarget()
        if target then
            flingPlayer(target)
            player:Chat("Флинг!")
        end
        return true
    elseif msg == "/tpall" then
        teleportAllToMe()
        return true
    elseif msg == "/reset" then
        Settings.WalkSpeed = 16
        Settings.JumpPower = 50
        Settings.FlySpeed = 50
        Settings.FlingPower = 80
        Settings.Speed = 4.0
        Settings.Prediction = 0.6
        applyMovementSettings()
        player:Chat("Все настройки сброшены")
        return true
    elseif string.find(msg, "/tp ") then
        local targetName = string.sub(msg, 5)
        for _, plr in ipairs(Players:GetPlayers()) do
            if string.lower(plr.Name) == string.lower(targetName) then
                local targetChar = plr.Character
                if targetChar and targetChar.PrimaryPart and character and character.PrimaryPart then
                    character.PrimaryPart.CFrame = targetChar.PrimaryPart.CFrame + Vector3.new(0, 2, 0)
                    createParticles(character.PrimaryPart.Position, "Bright cyan", 15)
                    player:Chat("Телепорт к " .. plr.Name)
                end
                return true
            end
        end
        player:Chat("Игрок не найден: " .. targetName)
        return true
    end
    
    return false
end

player.Chatted:Connect(handleChatCommand)

-- ============================================
-- ===== ПИНГ / ФПС / ИГРОКИ =====
-- ============================================

local statsGui = Instance.new("ScreenGui")
statsGui.Parent = player.PlayerGui
statsGui.Name = "MurzikStats"
statsGui.ResetOnSpawn = false

local statsFrame = Instance.new("Frame")
statsFrame.Size = UDim2.new(0, 150, 0, 75)
statsFrame.Position = UDim2.new(1, -160, 0, 10)
statsFrame.BackgroundColor3 = Color3.new(0, 0, 0)
statsFrame.BackgroundTransparency = 0.7
statsFrame.BorderSizePixel = 1
statsFrame.BorderColor3 = Color3.new(0.8, 0.2, 0.8)
statsFrame.Parent = statsGui

local statsCorner = Instance.new("UICorner")
statsCorner.CornerRadius = UDim.new(0, 8)
statsCorner.Parent = statsFrame

-- Заголовок Murzik Hub в статистике
local statsTitle = Instance.new("TextLabel")
statsTitle.Size = UDim2.new(1, 0, 0, 16)
statsTitle.Position = UDim2.new(0, 0, 0, 0)
statsTitle.BackgroundTransparency = 1
statsTitle.Text = "🐱 Murzik Hub"
statsTitle.TextColor3 = Color3.new(0.8, 0.2, 0.8)
statsTitle.TextSize = 12
statsTitle.Font = Enum.Font.GothamBold
statsTitle.TextXAlignment = Enum.TextXAlignment.Center
statsTitle.Parent = statsFrame

local pingLabel = Instance.new("TextLabel")
pingLabel.Size = UDim2.new(1, 0, 0, 19)
pingLabel.Position = UDim2.new(0, 0, 0, 17)
pingLabel.BackgroundTransparency = 1
pingLabel.Text = "📶 0 ms"
pingLabel.TextColor3 = Color3.new(0.3, 1, 0.3)
pingLabel.TextSize = 12
pingLabel.Font = Enum.Font.GothamBold
pingLabel.TextXAlignment = Enum.TextXAlignment.Center
pingLabel.Parent = statsFrame

local fpsLabel = Instance.new("TextLabel")
fpsLabel.Size = UDim2.new(1, 0, 0, 19)
fpsLabel.Position = UDim2.new(0, 0, 0, 36)
fpsLabel.BackgroundTransparency = 1
fpsLabel.Text = "⚡ 60 FPS"
fpsLabel.TextColor3 = Color3.new(0.3, 1, 0.3)
fpsLabel.TextSize = 12
fpsLabel.Font = Enum.Font.GothamBold
fpsLabel.TextXAlignment = Enum.TextXAlignment.Center
fpsLabel.Parent = statsFrame

local playersLabel = Instance.new("TextLabel")
playersLabel.Size = UDim2.new(1, 0, 0, 19)
playersLabel.Position = UDim2.new(0, 0, 0, 55)
playersLabel.BackgroundTransparency = 1
playersLabel.Text = "👤 0 игроков"
playersLabel.TextColor3 = Color3.new(0.5, 0.8, 1)
playersLabel.TextSize = 11
playersLabel.Font = Enum.Font.Gotham
playersLabel.TextXAlignment = Enum.TextXAlignment.Center
playersLabel.Parent = statsFrame

local frameCount = 0
local lastFpsUpdate = tick()

local function getPing()
    local ping = 0
    pcall(function()
        if Stats and Stats.Network and Stats.Network.ServerStatsItem then
            local item = Stats.Network:FindFirstChild("ServerStatsItem")
            if item and item.Ping then ping = item.Ping end
        end
    end)
    return ping
end

spawn(function()
    while task.wait(1) do
        if Settings.ShowStats then
            local ping = getPing()
            local pingColor = Color3.new(0.3, 1, 0.3)
            if ping > 150 then pingColor = Color3.new(1, 1, 0) end
            if ping > 300 then pingColor = Color3.new(1, 0.3, 0.3) end
            pingLabel.Text = "📶 " .. math.floor(ping) .. " ms"
            pingLabel.TextColor3 = pingColor
            
            local now = tick()
            local dt = now - lastFpsUpdate
            if dt > 0 then
                local fps = math.floor(frameCount / dt)
                local fpsColor = Color3.new(0.3, 1, 0.3)
                if fps < 30 then fpsColor = Color3.new(1, 0.3, 0.3)
                elseif fps < 50 then fpsColor = Color3.new(1, 1, 0) end
                fpsLabel.Text = "⚡ " .. math.min(fps, 999) .. " FPS"
                fpsLabel.TextColor3 = fpsColor
                frameCount = 0
                lastFpsUpdate = now
            end
            
            playersLabel.Text = "👤 " .. #Players:GetPlayers() .. " игроков"
        else
            statsFrame.Visible = false
        end
    end
end)

RunService.Heartbeat:Connect(function()
    frameCount = frameCount + 1
end)

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.F3 then
        Settings.ShowStats = not Settings.ShowStats
        statsFrame.Visible = Settings.ShowStats
    end
end)

-- ============================================
-- ===== СОЗДАНИЕ MURZIK HUB UI =====
-- ============================================

local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player.PlayerGui
screenGui.Name = "MurzikHub"
screenGui.ResetOnSpawn = false

-- Главный фон
local mainBg = Instance.new("Frame")
mainBg.Size = UDim2.new(0, 560, 0, 395)
mainBg.Position = UDim2.new(0, 0, 0, 0)
mainBg.BackgroundColor3 = Settings.UIBackgroundColor
mainBg.BackgroundTransparency = Settings.UIBackgroundTransparency
mainBg.BorderSizePixel = 0
mainBg.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 14)
mainCorner.Parent = mainBg

if Settings.UIBackgroundGradient then
    local mainGradient = Instance.new("UIGradient")
    mainGradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Settings.UIBackgroundColor),
        ColorSequenceKeypoint.new(1, Settings.UIBackgroundGradientColor)
    })
    mainGradient.Rotation = 45
    mainGradient.Parent = mainBg
end

local border = Instance.new("Frame")
border.Size = UDim2.new(1, 0, 1, 0)
border.BackgroundTransparency = 1
border.BorderSizePixel = 2
border.BorderColor3 = Color3.new(0.8, 0.2, 0.8)
border.Parent = mainBg
local borderCorner = Instance.new("UICorner")
borderCorner.CornerRadius = UDim.new(0, 14)
borderCorner.Parent = border

-- ===== ЗАГОЛОВОК MURZIK HUB =====
local titleFrame = Instance.new("Frame")
titleFrame.Size = UDim2.new(1, 0, 0, 30)
titleFrame.Position = UDim2.new(0, 0, 0, 0)
titleFrame.BackgroundColor3 = Color3.new(0.8, 0.2, 0.8)
titleFrame.BackgroundTransparency = 0.3
titleFrame.BorderSizePixel = 0
titleFrame.Parent = mainBg

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 10)
titleCorner.Parent = titleFrame

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 1, 0)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "🐱 MURZIK HUB v3.0"
titleLabel.TextColor3 = Color3.new(1, 1, 1)
titleLabel.TextSize = 18
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextXAlignment = Enum.TextXAlignment.Center
titleLabel.Parent = titleFrame

-- Подзаголовок
local subLabel = Instance.new("TextLabel")
subLabel.Size = UDim2.new(1, 0, 0, 16)
subLabel.Position = UDim2.new(0, 0, 0, 28)
subLabel.BackgroundTransparency = 1
subLabel.Text = "⚡ DELTA | Разработчик: Murzik"
subLabel.TextColor3 = Color3.new(0.6, 0.6, 0.8)
subLabel.TextSize = 11
subLabel.Font = Enum.Font.Gotham
subLabel.TextXAlignment = Enum.TextXAlignment.Center
subLabel.Parent = mainBg

-- ===== КНОПКИ =====
local function createButton(text, position, color, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 92, 0, 35)
    frame.Position = position
    frame.BackgroundColor3 = color
    frame.BackgroundTransparency = 0.2
    frame.BorderSizePixel = 2
    frame.BorderColor3 = Color3.new(1, 1, 1)
    frame.Parent = mainBg
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = frame
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Color3.new(1, 1, 1)
    label.TextSize = 13
    label.Font = Enum.Font.GothamBold
    label.Parent = frame
    
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, 0, 1, 0)
    button.BackgroundTransparency = 1
    button.Text = ""
    button.Parent = frame
    
    button.MouseButton1Click:Connect(callback)
    return frame
end

local startY = 48
local rowHeight = 43

-- Ряд 1
local toggleBtn = createButton("ВКЛ", UDim2.new(0, 5, 0, startY), Color3.new(0, 0.8, 0), function()
    Settings.Enabled = not Settings.Enabled
    toggleBtn.BackgroundColor3 = Settings.Enabled and Color3.new(0, 0.8, 0) or Color3.new(0.8, 0, 0)
    toggleBtn:FindFirstChild("TextLabel").Text = Settings.Enabled and "ВКЛ" or "ВЫКЛ"
end)

local colors = {"Red", "Blue", "Green", "Yellow", "Purple", "Orange", "Pink"}
local colorIndex = 1
local colorMap = {
    Red = Color3.new(1, 0, 0), Blue = Color3.new(0, 0, 1),
    Green = Color3.new(0, 1, 0), Yellow = Color3.new(1, 1, 0),
    Purple = Color3.new(0.5, 0, 1), Orange = Color3.new(1, 0.5, 0),
    Pink = Color3.new(1, 0, 0.8)
}

createButton("ЦВЕТ", UDim2.new(0, 105, 0, startY), Color3.new(0.3, 0.3, 0.8), function()
    colorIndex = colorIndex % #colors + 1
    local newColor = colors[colorIndex]
    Settings.Color = newColor
    ball.BrickColor = BrickColor.new(newColor)
    glow.Color = colorMap[newColor]
    saveSettings()
end)

createButton("СКОР+", UDim2.new(0, 205, 0, startY), Color3.new(0.8, 0.5, 0), function()
    Settings.Speed = math.min(Settings.Speed + 0.5, 10)
end)

createButton("СКОР-", UDim2.new(0, 305, 0, startY), Color3.new(0.5, 0.3, 0.8), function()
    Settings.Speed = math.max(Settings.Speed - 0.5, 1)
end)

createButton("РАНГ+", UDim2.new(0, 405, 0, startY), Color3.new(0.2, 0.6, 0.8), function()
    Settings.Range = math.min(Settings.Range + 10, 300)
end)

createButton("РАНГ-", UDim2.new(0, 505, 0, startY), Color3.new(0.8, 0.4, 0.2), function()
    Settings.Range = math.max(Settings.Range - 10, 20)
end)

-- Ряд 2
local y2 = startY + rowHeight
createButton("УПР+", UDim2.new(0, 5, 0, y2), Color3.new(0.8, 0, 0.8), function()
    Settings.Prediction = math.min(Settings.Prediction + 0.1, 1)
end)

createButton("УПР-", UDim2.new(0, 105, 0, y2), Color3.new(0, 0.6, 0.6), function()
    Settings.Prediction = math.max(Settings.Prediction - 0.1, 0)
end)

createButton("ФЛИНГ", UDim2.new(0, 205, 0, y2), Color3.new(0.8, 0.2, 0.2), function()
    local target = getClosestTarget()
    if target then
        flingPlayer(target)
        player:Chat("Флинг!")
    end
end)

local modeColors = {Color3.new(0.2, 0.8, 0.2), Color3.new(0.8, 0.8, 0), Color3.new(0.8, 0.2, 0.8)}
local modeIndex = 1
local modeBtn = createButton("РЕЖИМ", UDim2.new(0, 305, 0, y2), modeColors[1], function()
    modeIndex = modeIndex % #flingModes + 1
    Settings.FlingMode = modeIndex
    modeBtn.BackgroundColor3 = modeColors[modeIndex]
    modeBtn:FindFirstChild("TextLabel").Text = flingModes[modeIndex]
    saveSettings()
end)

createButton("СИЛА+", UDim2.new(0, 405, 0, y2), Color3.new(0.8, 0.4, 0), function()
    Settings.FlingPower = math.min(Settings.FlingPower + 10, 200)
end)

createButton("СИЛА-", UDim2.new(0, 505, 0, y2), Color3.new(0.4, 0.6, 0.8), function()
    Settings.FlingPower = math.max(Settings.FlingPower - 10, 20)
end)

-- Ряд 3 (Флай)
local y3 = y2 + rowHeight
local flyBtn = createButton("ФЛАЙ", UDim2.new(0, 5, 0, y3), Color3.new(0.2, 0.6, 0.8), function()
    toggleFly()
    flyBtn.BackgroundColor3 = Settings.FlyEnabled and Color3.new(0.8, 0.2, 0.2) or Color3.new(0.2, 0.6, 0.8)
    flyBtn:FindFirstChild("TextLabel").Text = Settings.FlyEnabled and "ФЛАЙ✈️" or "ФЛАЙ"
    saveSettings()
end)

createButton("ФЛСП+", UDim2.new(0, 105, 0, y3), Color3.new(0.2, 0.8, 0.4), function()
    Settings.FlySpeed = math.min(Settings.FlySpeed + 10, 200)
end)

createButton("ФЛСП-", UDim2.new(0, 205, 0, y3), Color3.new(0.8, 0.4, 0.2), function()
    Settings.FlySpeed = math.max(Settings.FlySpeed - 10, 10)
end)

createButton("ХОД+", UDim2.new(0, 305, 0, y3), Color3.new(0.4, 0.8, 0.4), function()
    Settings.WalkSpeed = math.min(Settings.WalkSpeed + 2, 100)
    applyMovementSettings()
    saveSettings()
end)

createButton("ХОД-", UDim2.new(0, 405, 0, y3), Color3.new(0.8, 0.4, 0.4), function()
    Settings.WalkSpeed = math.max(Settings.WalkSpeed - 2, 8)
    applyMovementSettings()
    saveSettings()
end)

createButton("ПРЫЖ+", UDim2.new(0, 505, 0, y3), Color3.new(0.4, 0.4, 0.8), function()
    Settings.JumpPower = math.min(Settings.JumpPower + 10, 200)
    applyMovementSettings()
    saveSettings()
end)

-- Ряд 4
local y4 = y3 + rowHeight
createButton("ПРЫЖ-", UDim2.new(0, 5, 0, y4), Color3.new(0.8, 0.4, 0.8), function()
    Settings.JumpPower = math.max(Settings.JumpPower - 10, 20)
    applyMovementSettings()
    saveSettings()
end)

createButton("ВСЕХ", UDim2.new(0, 105, 0, y4), Color3.new(0.8, 0.6, 0), function()
    teleportAllToMe()
end)

createButton("ТРЕЙЛ", UDim2.new(0, 205, 0, y4), Color3.new(0, 0.6, 0.6), function()
    Settings.TrailEnabled = not Settings.TrailEnabled
    player:Chat("Трейл: " .. (Settings.TrailEnabled and "Вкл" or "Выкл"))
    saveSettings()
end)

createButton("ПАРТИК", UDim2.new(0, 305, 0, y4), Color3.new(0.6, 0, 0.6), function()
    Settings.ParticlesEnabled = not Settings.ParticlesEnabled
    player:Chat("Партиклы: " .. (Settings.ParticlesEnabled and "Вкл" or "Выкл"))
    saveSettings()
end)

createButton("ЗВУК", UDim2.new(0, 405, 0, y4), Color3.new(0.6, 0.6, 0), function()
    Settings.SoundsEnabled = not Settings.SoundsEnabled
    player:Chat("Звук: " .. (Settings.SoundsEnabled and "Вкл" or "Выкл"))
    saveSettings()
end)

createButton("СБРОС", UDim2.new(0, 505, 0, y4), Color3.new(0.5, 0, 0.5), function()
    Settings.WalkSpeed = 16
    Settings.JumpPower = 50
    Settings.FlySpeed = 50
    Settings.FlingPower = 80
    Settings.Speed = 4.0
    Settings.Prediction = 0.6
    applyMovementSettings()
    player:Chat("Сброс настроек")
    saveSettings()
end)

-- Ряд 5 (Сохранение/Загрузка)
local y5 = y4 + rowHeight
createButton("СОХР", UDim2.new(0, 5, 0, y5), Color3.new(0, 0.5, 0), function()
    saveSettings()
    player:Chat("Настройки сохранены")
end)

createButton("ЗАГР", UDim2.new(0, 105, 0, y5), Color3.new(0.5, 0.5, 0), function()
    loadSettings()
    applyMovementSettings()
    player:Chat("Настройки загружены")
end)

-- ===== СТАТУС-БАР =====
local statusFrame = Instance.new("Frame")
statusFrame.Size = UDim2.new(0, 410, 0, 24)
statusFrame.Position = UDim2.new(0, 5, 0, 360)
statusFrame.BackgroundColor3 = Color3.new(0, 0, 0)
statusFrame.BackgroundTransparency = 0.5
statusFrame.BorderSizePixel = 1
statusFrame.BorderColor3 = Color3.new(0.8, 0.2, 0.8)
statusFrame.Parent = mainBg

local statusCorner = Instance.new("UICorner")
statusCorner.CornerRadius = UDim.new(0, 6)
statusCorner.Parent = statusFrame

local statusLabel = Instance.new("TextLabel")
statusLabel.Size = UDim2.new(1, 0, 1, 0)
statusLabel.BackgroundTransparency = 1
statusLabel.Text = "🐱 Murzik Hub | ⚡Наводка ✓ | 🚶16 | 🦘50 | ✈️50 | Режим: ОДИН"
statusLabel.TextColor3 = Color3.new(1, 1, 1)
statusLabel.TextSize = 11
statusLabel.Font = Enum.Font.Gotham
statusLabel.Parent = statusFrame

spawn(function()
    while task.wait(0.5) do
        statusLabel.Text = "🐱 Murzik Hub | ⚡" .. (Settings.Enabled and "Наводка ✓" or "Наводка ✗") ..
            " | 🚶" .. math.floor(Settings.WalkSpeed) ..
            " | 🦘" .. math.floor(Settings.JumpPower) ..
            " | ✈️" .. math.floor(Settings.FlySpeed) ..
            (Settings.FlyEnabled and " 🟢" or " 🔴") ..
            " | Режим: " .. flingModes[Settings.FlingMode] ..
            " | 👤" .. #Players:GetPlayers()
    end
end)

-- ============================================
-- ===== ТЕЛЕПОРТ: СПИСОК ИГРОКОВ =====
-- ============================================

local teleportFrame = Instance.new("Frame")
teleportFrame.Size = UDim2.new(0, 185, 0, 280)
teleportFrame.Position = UDim2.new(0, 425, 0, 50)
teleportFrame.BackgroundColor3 = Settings.UIBackgroundColor
teleportFrame.BackgroundTransparency = Settings.UIBackgroundTransparency
teleportFrame.BorderSizePixel = 2
teleportFrame.BorderColor3 = Color3.new(0.8, 0.2, 0.8)
teleportFrame.Parent = mainBg

local teleportCorner = Instance.new("UICorner")
teleportCorner.CornerRadius = UDim.new(0, 10)
teleportCorner.Parent = teleportFrame

if Settings.UIBackgroundGradient then
    local tpGradient = Instance.new("UIGradient")
    tpGradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Settings.UIBackgroundColor),
        ColorSequenceKeypoint.new(1, Settings.UIBackgroundGradientColor)
    })
    tpGradient.Rotation = 45
    tpGradient.Parent = teleportFrame
end

local teleportTitle = Instance.new("TextLabel")
teleportTitle.Size = UDim2.new(1, 0, 0, 22)
teleportTitle.Position = UDim2.new(0, 0, 0, 0)
teleportTitle.BackgroundTransparency = 1
teleportTitle.Text = "📌 ТЕЛЕПОРТ"
teleportTitle.TextColor3 = Color3.new(1, 1, 1)
teleportTitle.TextSize = 13
teleportTitle.Font = Enum.Font.GothamBold
teleportTitle.Parent = teleportFrame

local teleportSub = Instance.new("TextLabel")
teleportSub.Size = UDim2.new(1, 0, 0, 16)
teleportSub.Position = UDim2.new(0, 0, 0, 22)
teleportSub.BackgroundTransparency = 1
teleportSub.Text = "🟢К игроку | 🔴К себе"
teleportSub.TextColor3 = Color3.new(0.7, 0.7, 0.7)
teleportSub.TextSize = 10
teleportSub.Font = Enum.Font.Gotham
teleportSub.Parent = teleportFrame

local refreshBtn = Instance.new("TextButton")
refreshBtn.Size = UDim2.new(0, 40, 0, 20)
refreshBtn.Position = UDim2.new(0, 135, 0, 2)
refreshBtn.BackgroundColor3 = Color3.new(0, 0.6, 0.8)
refreshBtn.Text = "🔄"
refreshBtn.TextColor3 = Color3.new(1, 1, 1)
refreshBtn.TextSize = 14
refreshBtn.Font = Enum.Font.GothamBold
refreshBtn.Parent = teleportFrame

local refreshCorner = Instance.new("UICorner")
refreshCorner.CornerRadius = UDim.new(0, 8)
refreshCorner.Parent = refreshBtn

local playerList = Instance.new("ScrollingFrame")
playerList.Size = UDim2.new(1, -10, 1, -45)
playerList.Position = UDim2.new(0, 5, 0, 40)
playerList.BackgroundTransparency = 1
playerList.ScrollBarThickness = 5
playerList.CanvasSize = UDim2.new(0, 0, 0, 0)
playerList.Parent = teleportFrame

local listLayout = Instance.new("UIListLayout")
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Padding = UDim.new(0, 4)
listLayout.Parent = playerList

local playerButtons = {}

local function updatePlayerList()
    for _, btn in ipairs(playerButtons) do btn:Destroy() end
    playerButtons = {}
    
    local players = Players:GetPlayers()
    playerList.CanvasSize = UDim2.new(0, 0, 0, #players * 50 + 10)
    
    for _, plr in ipairs(players) do
        if plr ~= player then
            local container = Instance.new("Frame")
            container.Size = UDim2.new(1, -10, 0, 47)
            container.BackgroundTransparency = 1
            container.Parent = playerList
            
            local nameLabel = Instance.new("TextLabel")
            nameLabel.Size = UDim2.new(1, 0, 0, 16)
            nameLabel.Position = UDim2.new(0, 0, 0, 0)
            nameLabel.BackgroundTransparency = 1
            nameLabel.Text = plr.Name
            nameLabel.TextColor3 = Color3.new(1, 1, 1)
            nameLabel.TextSize = 11
            nameLabel.Font = Enum.Font.GothamBold
            nameLabel.TextXAlignment = Enum.TextXAlignment.Center
            nameLabel.Parent = container
            
            local toPlayerBtn = Instance.new("TextButton")
            toPlayerBtn.Size = UDim2.new(0.48, 0, 0, 26)
            toPlayerBtn.Position = UDim2.new(0, 0, 0, 18)
            toPlayerBtn.BackgroundColor3 = Color3.new(0, 0.6, 0)
            toPlayerBtn.BackgroundTransparency = 0.2
            toPlayerBtn.Text = "🟢 К игроку"
            toPlayerBtn.TextColor3 = Color3.new(1, 1, 1)
            toPlayerBtn.TextSize = 10
            toPlayerBtn.Font = Enum.Font.Gotham
            toPlayerBtn.Parent = container
            
            local toPlayerCorner = Instance.new("UICorner")
            toPlayerCorner.CornerRadius = UDim.new(0, 6)
            toPlayerCorner.Parent = toPlayerBtn
            
            toPlayerBtn.MouseButton1Click:Connect(function()
                local targetChar = plr.Character
                if targetChar and targetChar.PrimaryPart and character and character.PrimaryPart then
                    character.PrimaryPart.CFrame = targetChar.PrimaryPart.CFrame + Vector3.new(0, 2, 0)
                    createParticles(character.PrimaryPart.Position, "Bright cyan", 15)
                    playSound("9120385868")
                end
            end)
            
            local toMeBtn = Instance.new("TextButton")
            toMeBtn.Size = UDim2.new(0.48, 0, 0, 26)
            toMeBtn.Position = UDim2.new(0.52, 0, 0, 18)
            toMeBtn.BackgroundColor3 = Color3.new(0.8, 0, 0)
            toMeBtn.BackgroundTransparency = 0.2
            toMeBtn.Text = "🔴 К себе"
            toMeBtn.TextColor3 = Color3.new(1, 1, 1)
            toMeBtn.TextSize = 10
            toMeBtn.Font = Enum.Font.Gotham
            toMeBtn.Parent = container
            
            local toMeCorner = Instance.new("UICorner")
            toMeCorner.CornerRadius = UDim.new(0, 6)
            toMeCorner.Parent = toMeBtn
            
            toMeBtn.MouseButton1Click:Connect(function()
                teleportPlayerToMe(plr)
            end)
            
            table.insert(playerButtons, container)
        end
    end
end

updatePlayerList()
refreshBtn.MouseButton1Click:Connect(updatePlayerList)
Players.PlayerAdded:Connect(updatePlayerList)
Players.PlayerRemoving:Connect(updatePlayerList)

-- ============================================
-- ===== ОСНОВНАЯ ЛОГИКА САМОНАВОДКИ =====
-- ============================================

spawn(function()
    while task.wait() do
        local dt = task.wait()
        
        if Settings.Enabled then
            local target = getClosestTarget()
            
            if target and target.PrimaryPart then
                local targetPos = target.PrimaryPart.Position
                local timeToHit = (targetPos - ball.Position).Magnitude / (Settings.Speed * 30)
                local predicted = targetPos + target.PrimaryPart.Velocity * timeToHit * Settings.Prediction
                
                ball.Position = ball.Position:Lerp(predicted, Settings.Speed * dt)
                
                if dt > 0.05 then createTrail() end
                
                if (ball.Position - targetPos).Magnitude < 4 then
                    flingPlayer(target)
                    ball.BrickColor = BrickColor.new("Bright yellow")
                    ball.Size = Vector3.new(6, 6, 6)
                    glow.Color = Color3.new(1, 1, 0)
                    glow.Brightness = 8
                    createParticles(targetPos, "Bright yellow", 30)
                    task.wait(0.2)
                    ball.BrickColor = BrickColor.new(Settings.Color)
                    ball.Size = Vector3.new(4, 4, 4)
                    glow.Brightness = 4
                end
            else
                if character and character.PrimaryPart then
                    local origin = character.PrimaryPart.Position + Vector3.new(0, 5, 0)
                    ball.Position = ball.Position:Lerp(origin, Settings.Speed * dt)
                end
            end
        end
        
        ball.Orientation = ball.Orientation + Vector3.new(0, dt * 60, 0)
    end
end)

-- ============================================
-- ===== УПРАВЛЕНИЕ ФЛАЕМ =====
-- ============================================

UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    
    if Settings.FlyEnabled and character and character.PrimaryPart then
        local key = input.KeyCode
        local direction = Vector3.new(0, 0, 0)
        local speed = Settings.FlySpeed
        
        if key == Enum.KeyCode.W then direction = direction + Vector3.new(0, 0, -speed) end
        if key == Enum.KeyCode.S then direction = direction + Vector3.new(0, 0, speed) end
        if key == Enum.KeyCode.A then direction = direction + Vector3.new(-speed, 0, 0) end
        if key == Enum.KeyCode.D then direction = direction + Vector3.new(speed, 0, 0) end
        if key == Enum.KeyCode.Space then direction = direction + Vector3.new(0, speed, 0) end
        if key == Enum.KeyCode.LeftControl then direction = direction + Vector3.new(0, -speed, 0) end
        
        if flyBodyVelocity and direction.Magnitude > 0 then
            local camera = workspace.CurrentCamera
            local camCFrame = camera.CFrame
            local forward = camCFrame.LookVector * direction.Z
            local right = camCFrame.RightVector * direction.X
            local up = Vector3.new(0, direction.Y, 0)
            
            flyBodyVelocity.Velocity = (forward + right + up)
            
            if flyBodyGyro then
                flyBodyGyro.CFrame = CFrame.lookAt(
                    character.PrimaryPart.Position,
                    character.PrimaryPart.Position + (forward + right + up).Unit * 10
                )
            end
        end
    end
end)

UserInputService.InputEnded:Connect(function(input, processed)
    if processed then return end
    if Settings.FlyEnabled and flyBodyVelocity then
        local key = input.KeyCode
        if key == Enum.KeyCode.W or key == Enum.KeyCode.S or 
           key == Enum.KeyCode.A or key == Enum.KeyCode.D or
           key == Enum.KeyCode.Space or key == Enum.KeyCode.LeftControl then
            flyBodyVelocity.Velocity = Vector3.new(0, 0, 0)
        end
    end
end)

-- ============================================
-- ===== СВАЙПЫ ДЛЯ ТЕЛЕФОНА =====
-- ============================================

local touchStart = nil

UserInputService.TouchStarted:Connect(function(input, processed)
    if processed then return end
    touchStart = input.Position
end)

UserInputService.TouchEnded:Connect(function(input, processed)
    if processed or not touchStart then return end
    local delta = input.Position - touchStart
    if delta.Magnitude > 50 then
        local angle = math.atan2(delta.Y, delta.X)
        if angle > -math.pi/4 and angle < math.pi/4 then
            Settings.Prediction = math.min(Settings.Prediction + 0.1, 1)
        elseif angle > math.pi/4 and angle < 3*math.pi/4 then
            Settings.Speed = math.max(Settings.Speed - 0.5, 1)
        elseif angle > -3*math.pi/4 and angle < -math.pi/4 then
            Settings.Speed = math.min(Settings.Speed + 0.5, 10)
        else
            Settings.Prediction = math.max(Settings.Prediction - 0.1, 0)
        end
    end
    touchStart = nil
end)

-- ============================================
-- ===== ЗАПУСК БАЙПАСА =====
-- ============================================

if Settings.BypassEnabled then
    initBypass()
end

print("🐱 MURZIK HUB v3.0 ЗАГРУЖЕН")
print("Чат-команды: /fly, /fling, /tpall, /reset, /tp [имя]")
print("F3 - скрыть статистику")
print("Разработчик: Murzik")
