#JOIN FOR FREE NOTFIY AND SOURCE 10M-10B
https://discord.gg/H6JpjjwXzx
-- M.T.S Hub semi TP
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ProximityPromptService = game:GetService("ProximityPromptService")
local CoreGui = game:GetService("CoreGui")
local TextChatService = game:GetService("TextChatService")

local player = Players.LocalPlayer

local pos1 = Vector3.new(-352.98, -7, 74.30)
local pos2 = Vector3.new(-352.98, -6.49, 45.76)
local standing1 = Vector3.new(-336.36, -4.59, 99.51)
local standing2 = Vector3.new(-334.81, -4.59, 18.90)

local Trigger_A = Vector3.new(-352.885, -7.300, 76.068)
local Target_A = Vector3.new(-348.345, -6.835, 10.607)
local Trigger_B = pos2
local Target_B = Vector3.new(-347.41278076171875, -7.3000030517578125, 103.7739486694336)

local spot1_sequence = {
    CFrame.new(-370.810913, -7.00000334, 41.2687263, 0.99984771, 1.22364419e-09, 0.0174523517, -6.54859778e-10, 1, -3.2596418e-08, -0.0174523517, 3.25800258e-08, 0.99984771),
    CFrame.new(-336.355286, -5.10107088, 17.2327671, -0.999883354, -2.76150569e-08, 0.0152716246, -2.88224964e-08, 1, -7.88441525e-08, -0.0152716246, -7.9275118e-08, -0.999883354)
}

local spot2_sequence = {
    CFrame.new(-354.782867, -7.00000334, 92.8209305, -0.999997616, -1.11891862e-09, -0.00218066527, -1.11958298e-09, 1, 3.03415071e-10, 0.00218066527, 3.05855785e-10, -0.999997616),
    CFrame.new(-336.942902, -5.10106993, 99.3276443, 0.999914348, -3.63984611e-08, 0.0130875716, 3.67094941e-08, 1, -2.35254749e-08, -0.0130875716, 2.40038975e-08, 0.999914348)
}

local autoWalkEnabled = false
local isAutoWalking = false
local walkDebounce = false
local activeAutoWalkMode = nil
local SPEED_BOOST = 28
local speedEnabled = false
local speedConnection = nil

local ThemeBlue = Color3.fromRGB(30, 120, 220)
local GlowBlue = Color3.fromRGB(80, 170, 255)
local DarkBg = Color3.fromRGB(5, 8, 18)
local MidBg = Color3.fromRGB(8, 15, 35)

if CoreGui:FindFirstChild("MTSHubGui") then
    CoreGui["MTSHubGui"]:Destroy()
end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MTSHubGui"
screenGui.ResetOnSpawn = false
screenGui.Parent = CoreGui

local function createESPBox(position, labelText)
    local espFolder = Instance.new("Folder")
    espFolder.Name = "ESPBox_" .. labelText
    espFolder.Parent = workspace
    local box = Instance.new("Part")
    box.Size = Vector3.new(5, 0.5, 5)
    box.Position = position
    box.Anchored = true
    box.CanCollide = false
    box.Transparency = 0.5
    box.Material = Enum.Material.Neon
    box.Color = GlowBlue
    box.Parent = espFolder
    local selectionBox = Instance.new("SelectionBox")
    selectionBox.Adornee = box
    selectionBox.LineThickness = 0.05
    selectionBox.Color3 = GlowBlue
    selectionBox.Parent = box
    local billboard = Instance.new("BillboardGui")
    billboard.Adornee = box
    billboard.Size = UDim2.new(0, 150, 0, 40)
    billboard.StudsOffset = Vector3.new(0, 2, 0)
    billboard.AlwaysOnTop = true
    billboard.Parent = box
    local textLabel = Instance.new("TextLabel")
    textLabel.Size = UDim2.new(1, 0, 1, 0)
    textLabel.BackgroundTransparency = 1
    textLabel.Text = labelText
    textLabel.TextColor3 = GlowBlue
    textLabel.TextSize = 18
    textLabel.Font = Enum.Font.GothamBold
    textLabel.TextStrokeTransparency = 0.3
    textLabel.TextStrokeColor3 = ThemeBlue
    textLabel.Parent = billboard
    return espFolder
end

createESPBox(pos1, "Teleport Here")
createESPBox(pos2, "Teleport Here")
createESPBox(standing1, "Standing 1")
createESPBox(standing2, "Standing 2")
local cf1 = CFrame.new(-349.325867, -7.00000238, 95.0031433, -0.999048233, -8.29406233e-09, -0.0436184891, -1.03892832e-08, 1, 4.78084594e-08, 0.0436184891, 4.82161227e-08, -0.999048233)
createESPBox(cf1.Position, "Auto tp Left")
local cf2 = CFrame.new(-349.560211, -7.00000238, 27.0543289, -0.999961913, 5.50995267e-08, -0.00872585084, 5.48100907e-08, 1, 3.34090586e-08, 0.00872585084, 3.29295204e-08, -0.999961913)
createESPBox(cf2.Position, "Auto tp Right")

-- Main Frame (bigger size)
local FULL_HEIGHT = 400
local MINI_HEIGHT = 130

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 210, 0, FULL_HEIGHT)
mainFrame.Position = UDim2.new(1, -225, 0.5, -(FULL_HEIGHT/2))
mainFrame.AnchorPoint = Vector2.new(0, 0)
mainFrame.BackgroundColor3 = DarkBg
mainFrame.BackgroundTransparency = 0.05
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.ClipsDescendants = true
mainFrame.Parent = screenGui
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 10)

-- Animated Blue Border Glow
local borderStroke = Instance.new("UIStroke")
borderStroke.Thickness = 2.5
borderStroke.Color = GlowBlue
borderStroke.Transparency = 0
borderStroke.Parent = mainFrame

task.spawn(function()
    while true do
        TweenService:Create(borderStroke, TweenInfo.new(1.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {Transparency = 0.6, Thickness = 1.5}):Play()
        task.wait(1.5)
        TweenService:Create(borderStroke, TweenInfo.new(1.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {Transparency = 0, Thickness = 2.5}):Play()
        task.wait(1.5)
    end
end)

local bgGradient = Instance.new("UIGradient")
bgGradient.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(5, 10, 25)),
    ColorSequenceKeypoint.new(0.5, Color3.fromRGB(8, 18, 45)),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(3, 8, 20))
})
bgGradient.Rotation = 135
bgGradient.Parent = mainFrame

-- Stars
for i = 1, 8 do
    local star = Instance.new("Frame")
    star.Size = UDim2.new(0, math.random(2, 4), 0, math.random(2, 4))
    star.Position = UDim2.new(math.random(5, 95)/100, 0, math.random(5, 95)/100, 0)
    star.BackgroundColor3 = Color3.fromRGB(200, 230, 255)
    star.BackgroundTransparency = math.random(0, 5)/10
    star.BorderSizePixel = 0
    star.Parent = mainFrame
    Instance.new("UICorner", star).CornerRadius = UDim.new(1, 0)
    task.spawn(function()
        while true do
            TweenService:Create(star, TweenInfo.new(math.random(10, 25)/10, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {BackgroundTransparency = math.random(3, 9)/10}):Play()
            task.wait(math.random(10, 25)/10)
        end
    end)
end

-- Header
local headerFrame = Instance.new("Frame")
headerFrame.Size = UDim2.new(1, 0, 0, 36)
headerFrame.BackgroundTransparency = 1
headerFrame.Parent = mainFrame

-- Moon decoration
local moonDecor = Instance.new("Frame")
moonDecor.Size = UDim2.new(0, 18, 0, 18)
moonDecor.Position = UDim2.new(0, 8, 0.5, -9)
moonDecor.BackgroundColor3 = ThemeBlue
moonDecor.BackgroundTransparency = 0.3
moonDecor.BorderSizePixel = 0
moonDecor.Parent = headerFrame
Instance.new("UICorner", moonDecor).CornerRadius = UDim.new(1, 0)
local moonInner = Instance.new("Frame")
moonInner.Size = UDim2.new(0, 12, 0, 12)
moonInner.Position = UDim2.new(0, 5, 0, 3)
moonInner.BackgroundColor3 = DarkBg
moonInner.BorderSizePixel = 0
moonInner.Parent = moonDecor
Instance.new("UICorner", moonInner).CornerRadius = UDim.new(1, 0)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -70, 1, 0)
title.Position = UDim2.new(0, 30, 0, 0)
title.BackgroundTransparency = 1
title.Text = "M.T.S Hub semi TP"
title.TextColor3 = GlowBlue
title.TextSize = 12
title.Font = Enum.Font.GothamBlack
title.TextStrokeTransparency = 0.2
title.TextStrokeColor3 = ThemeBlue
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = headerFrame

task.spawn(function()
    while true do
        TweenService:Create(title, TweenInfo.new(2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {TextColor3 = Color3.fromRGB(120, 200, 255)}):Play()
        task.wait(2)
        TweenService:Create(title, TweenInfo.new(2, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {TextColor3 = GlowBlue}):Play()
        task.wait(2)
    end
end)

-- Minimize Button
local minBtn = Instance.new("TextButton")
minBtn.Size = UDim2.new(0, 26, 0, 26)
minBtn.Position = UDim2.new(1, -32, 0.5, -13)
minBtn.BackgroundColor3 = MidBg
minBtn.Text = "â"
minBtn.TextColor3 = GlowBlue
minBtn.TextSize = 18
minBtn.Font = Enum.Font.GothamBold
minBtn.Parent = headerFrame
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0, 6)
local minStroke = Instance.new("UIStroke", minBtn)
minStroke.Color = ThemeBlue
minStroke.Thickness = 1

-- Underline
local underline = Instance.new("Frame")
underline.Size = UDim2.new(0.85, 0, 0, 1)
underline.Position = UDim2.new(0.075, 0, 0, 36)
underline.BackgroundColor3 = GlowBlue
underline.BorderSizePixel = 0
underline.Parent = mainFrame
local underlineGlow = Instance.new("UIGradient")
underlineGlow.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 50, 150)),
    ColorSequenceKeypoint.new(0.5, GlowBlue),
    ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 50, 150))
})
underlineGlow.Parent = underline

local subtitle = Instance.new("TextLabel")
subtitle.Size = UDim2.new(1, 0, 0, 14)
subtitle.Position = UDim2.new(0, 0, 0, 38)
subtitle.BackgroundTransparency = 1
subtitle.Text = "â¦ M.T.S â¦"
subtitle.TextColor3 = ThemeBlue
subtitle.TextSize = 9
subtitle.Font = Enum.Font.GothamMedium
subtitle.TextStrokeTransparency = 0.5
subtitle.Parent = mainFrame

-- Content frame (all toggles/buttons go here, hidden when minimized)
local contentFrame = Instance.new("Frame")
contentFrame.Size = UDim2.new(1, 0, 1, -55)
contentFrame.Position = UDim2.new(0, 0, 0, 55)
contentFrame.BackgroundTransparency = 1
contentFrame.Parent = mainFrame

local semiTPEnabled = false

local function ResetToWork()
    local flags = {
        {"GameNetPVHeaderRotationalVelocityZeroCutoffExponent", "-5000"},
        {"LargeReplicatorWrite5", "true"}, {"LargeReplicatorEnabled9", "true"},
        {"AngularVelociryLimit", "360"}, {"TimestepArbiterVelocityCriteriaThresholdTwoDt", "2147483646"},
        {"S2PhysicsSenderRate", "15000"}, {"DisableDPIScale", "true"},
        {"MaxDataPacketPerSend", "2147483647"}, {"ServerMaxBandwith", "52"},
        {"PhysicsSenderMaxBandwidthBps", "20000"}, {"MaxTimestepMultiplierBuoyancy", "2147483647"},
        {"SimOwnedNOUCountThresholdMillionth", "2147483647"}, {"MaxMissedWorldStepsRemembered", "-2147483648"},
        {"CheckPVDifferencesForInterpolationMinVelThresholdStudsPerSecHundredth", "1"},
        {"StreamJobNOUVolumeLengthCap", "2147483647"}, {"DebugSendDistInSteps", "-2147483648"},
        {"MaxTimestepMultiplierAcceleration", "2147483647"}, {"LargeReplicatorRead5", "true"},
        {"SimExplicitlyCappedTimestepMultiplier", "2147483646"}, {"GameNetDontSendRedundantNumTimes", "1"},
        {"CheckPVLinearVelocityIntegrateVsDeltaPositionThresholdPercent", "1"},
        {"CheckPVCachedRotVelThresholdPercent", "10"}, {"LargeReplicatorSerializeRead3", "true"},
        {"ReplicationFocusNouExtentsSizeCutoffForPauseStuds", "2147483647"},
        {"NextGenReplicatorEnabledWrite4", "true"},
        {"CheckPVDifferencesForInterpolationMinRotVelThresholdRadsPerSecHundredth", "1"},
        {"GameNetDontSendRedundantDeltaPositionMillionth", "1"},
        {"InterpolationFrameVelocityThresholdMillionth", "5"}, {"StreamJobNOUVolumeCap", "2147483647"},
        {"InterpolationFrameRotVelocityThresholdMillionth", "5"}, {"WorldStepMax", "30"},
        {"TimestepArbiterHumanoidLinearVelThreshold", "1"},
        {"InterpolationFramePositionThresholdMillionth", "5"},
        {"TimestepArbiterHumanoidTurningVelThreshold", "1"},
        {"MaxTimestepMultiplierContstraint", "2147483647"},
        {"GameNetPVHeaderLinearVelocityZeroCutoffExponent", "-5000"},
        {"CheckPVCachedVelThresholdPercent", "10"}, {"TimestepArbiterOmegaThou", "1073741823"},
        {"MaxAcceptableUpdateDelay", "1"}, {"LargeReplicatorSerializeWrite4", "true"},
    }
    for _, data in ipairs(flags) do
        pcall(function() if setfflag then setfflag(data[1], data[2]) end end)
    end
    local char = player.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum:ChangeState(Enum.HumanoidStateType.Dead) end
        char:ClearAllChildren()
        local f = Instance.new("Model", workspace)
        player.Character = f task.wait()
        player.Character = char f:Destroy()
    end
end

local function executeTP(sequence)
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    local backpack = player:FindFirstChild("Backpack")
    if root and hum and backpack then
        local carpet = backpack:FindFirstChild("Flying Carpet")
        if carpet then hum:EquipTool(carpet) task.wait(0.1) end
        root.CFrame = sequence[1] task.wait(0.1) root.CFrame = sequence[2]
    end
end

-- Toggle creator (parented to contentFrame)
local function createToggle(text, yPos, callback)
    local container = Instance.new("Frame")
    container.Size = UDim2.new(0.9, 0, 0, 26)
    container.Position = UDim2.new(0.05, 0, 0, yPos)
    container.BackgroundTransparency = 1
    container.Parent = contentFrame

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -45, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Color3.fromRGB(180, 220, 255)
    label.TextSize = 9
    label.Font = Enum.Font.GothamMedium
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = container

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 38, 0, 20)
    btn.Position = UDim2.new(1, -38, 0.5, -10)
    btn.BackgroundColor3 = Color3.fromRGB(15, 25, 55)
    btn.Text = ""
    btn.Parent = container
    Instance.new("UICorner", btn).CornerRadius = UDim.new(1, 0)
    local btnStroke = Instance.new("UIStroke", btn)
    btnStroke.Color = ThemeBlue
    btnStroke.Thickness = 1

    local dot = Instance.new("Frame")
    dot.Size = UDim2.new(0, 14, 0, 14)
    dot.Position = UDim2.new(0, 2, 0.5, -7)
    dot.BackgroundColor3 = Color3.fromRGB(150, 200, 255)
    dot.Parent = btn
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

    local active = false
    btn.MouseButton1Click:Connect(function()
        active = not active
        TweenService:Create(dot, TweenInfo.new(0.15), {
            Position = active and UDim2.new(1, -16, 0.5, -7) or UDim2.new(0, 2, 0.5, -7),
            BackgroundColor3 = active and Color3.fromRGB(200, 235, 255) or Color3.fromRGB(150, 200, 255)
        }):Play()
        TweenService:Create(btn, TweenInfo.new(0.15), {
            BackgroundColor3 = active and ThemeBlue or Color3.fromRGB(15, 25, 55)
        }):Play()
        callback(active)
    end)
end

createToggle("Half Tp", 0, function(state) semiTPEnabled = state end)
createToggle("Auto Potion", 30, function(state) _G.AutoPotion = state end)

-- Speed Boost with TextBox
local speedContainer = Instance.new("Frame")
speedContainer.Size = UDim2.new(0.9, 0, 0, 26)
speedContainer.Position = UDim2.new(0.05, 0, 0, 60)
speedContainer.BackgroundTransparency = 1
speedContainer.Parent = contentFrame

local speedLabel = Instance.new("TextLabel")
speedLabel.Size = UDim2.new(0.5, 0, 1, 0)
speedLabel.BackgroundTransparency = 1
speedLabel.Text = "Speed Boost"
speedLabel.TextColor3 = Color3.fromRGB(180, 220, 255)
speedLabel.TextSize = 9
speedLabel.Font = Enum.Font.GothamMedium
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.Parent = speedContainer

local speedBox = Instance.new("TextBox")
speedBox.Size = UDim2.new(0, 35, 0, 20)
speedBox.Position = UDim2.new(1, -78, 0.5, -10)
speedBox.BackgroundColor3 = Color3.fromRGB(10, 18, 45)
speedBox.Text = tostring(SPEED_BOOST)
speedBox.TextColor3 = GlowBlue
speedBox.Font = Enum.Font.GothamBold
speedBox.TextSize = 10
speedBox.Parent = speedContainer
Instance.new("UICorner", speedBox).CornerRadius = UDim.new(0, 4)
local speedBoxStroke = Instance.new("UIStroke", speedBox)
speedBoxStroke.Color = ThemeBlue
speedBoxStroke.Thickness = 1

speedBox.FocusLost:Connect(function()
    local val = tonumber(speedBox.Text)
    if val then SPEED_BOOST = val
    else speedBox.Text = tostring(SPEED_BOOST) end
end)

local speedBtn = Instance.new("TextButton")
speedBtn.Size = UDim2.new(0, 38, 0, 20)
speedBtn.Position = UDim2.new(1, -38, 0.5, -10)
speedBtn.BackgroundColor3 = Color3.fromRGB(15, 25, 55)
speedBtn.Text = ""
speedBtn.Parent = speedContainer
Instance.new("UICorner", speedBtn).CornerRadius = UDim.new(1, 0)
local speedBtnStroke = Instance.new("UIStroke", speedBtn)
speedBtnStroke.Color = ThemeBlue
speedBtnStroke.Thickness = 1

local speedDot = Instance.new("Frame")
speedDot.Size = UDim2.new(0, 14, 0, 14)
speedDot.Position = UDim2.new(0, 2, 0.5, -7)
speedDot.BackgroundColor3 = Color3.fromRGB(150, 200, 255)
speedDot.Parent = speedBtn
Instance.new("UICorner", speedDot).CornerRadius = UDim.new(1, 0)

local function updateSpeedLogic(state)
    speedEnabled = state
    if state then
        if not speedConnection then
            speedConnection = RunService.Heartbeat:Connect(function()
                local char = player.Character
                if not char then return end
                local hum = char:FindFirstChild("Humanoid")
                local hrp = char:FindFirstChild("HumanoidRootPart")
                if hum and hrp and hum.MoveDirection.Magnitude > 0 then
                    if isAutoWalking then return end
                    local dir = hum.MoveDirection.Unit
                    hrp.AssemblyLinearVelocity = Vector3.new(dir.X * SPEED_BOOST, hrp.AssemblyLinearVelocity.Y, dir.Z * SPEED_BOOST)
                end
            end)
        end
    else
        if speedConnection then speedConnection:Disconnect() speedConnection = nil end
    end
end

speedBtn.MouseButton1Click:Connect(function()
    speedEnabled = not speedEnabled
    TweenService:Create(speedDot, TweenInfo.new(0.15), {
        Position = speedEnabled and UDim2.new(1, -16, 0.5, -7) or UDim2.new(0, 2, 0.5, -7),
        BackgroundColor3 = speedEnabled and Color3.fromRGB(200, 235, 255) or Color3.fromRGB(150, 200, 255)
    }):Play()
    TweenService:Create(speedBtn, TweenInfo.new(0.15), {
        BackgroundColor3 = speedEnabled and ThemeBlue or Color3.fromRGB(15, 25, 55)
    }):Play()
    updateSpeedLogic(speedEnabled)
end)

createToggle("Auto Walk Base", 90, function(state)
    autoWalkEnabled = state
    isAutoWalking = false
    walkDebounce = false
    activeAutoWalkMode = nil
end)

task.spawn(function() task.wait(1) ResetToWork() end)

-- SPAM AP Button
local spamButton = Instance.new("TextButton")
spamButton.Size = UDim2.new(0.9, 0, 0, 26)
spamButton.Position = UDim2.new(0.05, 0, 0, 122)
spamButton.Text = "â¦ SPAM AP NEAREST â¦"
spamButton.Font = Enum.Font.GothamBold
spamButton.TextSize = 9
spamButton.TextColor3 = Color3.fromRGB(200, 235, 255)
spamButton.BackgroundColor3 = ThemeBlue
spamButton.Parent = contentFrame
Instance.new("UICorner", spamButton).CornerRadius = UDim.new(0, 4)
local spamStroke = Instance.new("UIStroke", spamButton)
spamStroke.Color = GlowBlue
spamStroke.Thickness = 1.5

task.spawn(function()
    while true do
        TweenService:Create(spamStroke, TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {Thickness = 2.5}):Play()
        task.wait(1)
        TweenService:Create(spamStroke, TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {Thickness = 1}):Play()
        task.wait(1)
    end
end)

local function getNearestPlayer()
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return nil end
    local nearest, distance = nil, math.huge
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= player and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local mag = (char.HumanoidRootPart.Position - p.Character.HumanoidRootPart.Position).Magnitude
            if mag < distance then distance = mag nearest = p end
        end
    end
    return nearest
end

local function spamAPNearest()
    local target = getNearestPlayer()
    if not target then
        spamButton.Text = "NO TARGET"
        task.wait(1)
        spamButton.Text = "â¦ SPAM AP NEAREST â¦"
        return
    end
    spamButton.Text = "SPAMMING " .. target.Name:upper()
    if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
        local channel = TextChatService.TextChannels:WaitForChild("RBXGeneral")
        local commands = {";jumpscare ", ";morph ", "tiny ", ";inverse ", ";nightvision ", ";rocket ", ";balloon ", ";ragdoll "}
        for _, cmd in ipairs(commands) do
            pcall(function() channel:SendAsync(cmd .. target.Name) end)
            task.wait(0.1)
        end
        task.wait(1)
        pcall(function() channel:SendAsync(";jail " .. target.Name) end)
    end
    task.wait(0.5)
    spamButton.Text = "â¦ SPAM AP NEAREST â¦"
end

spamButton.MouseButton1Click:Connect(spamAPNearest)

-- Auto TP Left (always visible)
local autoTPLeftBtn = Instance.new("TextButton")
autoTPLeftBtn.Size = UDim2.new(0.9, 0, 0, 26)
autoTPLeftBtn.Position = UDim2.new(0.05, 0, 0, 154)
autoTPLeftBtn.Text = "â Auto tp Left"
autoTPLeftBtn.Font = Enum.Font.GothamBold
autoTPLeftBtn.TextSize = 10
autoTPLeftBtn.BackgroundColor3 = MidBg
autoTPLeftBtn.TextColor3 = GlowBlue
autoTPLeftBtn.Parent = contentFrame
Instance.new("UICorner", autoTPLeftBtn).CornerRadius = UDim.new(0, 4)
local leftStroke = Instance.new("UIStroke", autoTPLeftBtn)
leftStroke.Color = ThemeBlue
leftStroke.Thickness = 1

-- Auto TP Right (always visible)
local autoTPRightBtn = Instance.new("TextButton")
autoTPRightBtn.Size = UDim2.new(0.9, 0, 0, 26)
autoTPRightBtn.Position = UDim2.new(0.05, 0, 0, 184)
autoTPRightBtn.Text = "â Auto tp Right"
autoTPRightBtn.Font = Enum.Font.GothamBold
autoTPRightBtn.TextSize = 10
autoTPRightBtn.BackgroundColor3 = MidBg
autoTPRightBtn.TextColor3 = GlowBlue
autoTPRightBtn.Parent = contentFrame
Instance.new("UICorner", autoTPRightBtn).CornerRadius = UDim.new(0, 4)
local rightStroke = Instance.new("UIStroke", autoTPRightBtn)
rightStroke.Color = ThemeBlue
rightStroke.Thickness = 1

-- Progress Bar
local bar = Instance.new("Frame")
bar.Size = UDim2.new(0.9, 0, 0, 10)
bar.Position = UDim2.new(0.05, 0, 0, 216)
bar.BackgroundColor3 = Color3.fromRGB(10, 18, 40)
bar.Parent = contentFrame
Instance.new("UICorner", bar).CornerRadius = UDim.new(1, 0)
local barStroke = Instance.new("UIStroke", bar)
barStroke.Color = ThemeBlue
barStroke.Thickness = 1

local fill = Instance.new("Frame")
fill.BackgroundColor3 = ThemeBlue
fill.Size = UDim2.new(0, 0, 1, 0)
fill.Parent = bar
Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)
local fillGrad = Instance.new("UIGradient")
fillGrad.Color = ColorSequence.new({
    ColorSequenceKeypoint.new(0, ThemeBlue),
    ColorSequenceKeypoint.new(0.5, GlowBlue),
    ColorSequenceKeypoint.new(1, ThemeBlue)
})
fillGrad.Rotation = 90
fillGrad.Parent = fill

local percentLabel = Instance.new("TextLabel")
percentLabel.Size = UDim2.new(1, 0, 1, 0)
percentLabel.BackgroundTransparency = 1
percentLabel.Text = "0%"
percentLabel.TextColor3 = GlowBlue
percentLabel.TextSize = 7
percentLabel.Font = Enum.Font.GothamBold
percentLabel.TextXAlignment = Enum.TextXAlignment.Right
percentLabel.Parent = bar

-- Dropdown
local dropdownContainer = Instance.new("Frame")
dropdownContainer.Size = UDim2.new(0.9, 0, 0, 26)
dropdownContainer.Position = UDim2.new(0.05, 0, 0, 232)
dropdownContainer.BackgroundTransparency = 1
dropdownContainer.ClipsDescendants = true
dropdownContainer.Parent = contentFrame

local dropdownButton = Instance.new("TextButton")
dropdownButton.Size = UDim2.new(1, 0, 0, 26)
dropdownButton.BackgroundColor3 = MidBg
dropdownButton.Text = "â¦ TP TO SPOT â¼"
dropdownButton.TextColor3 = GlowBlue
dropdownButton.TextSize = 9
dropdownButton.Font = Enum.Font.GothamBold
dropdownButton.Parent = dropdownContainer
Instance.new("UICorner", dropdownButton).CornerRadius = UDim.new(0, 4)
local dropStroke = Instance.new("UIStroke", dropdownButton)
dropStroke.Color = ThemeBlue
dropStroke.Thickness = 1

local dropdownList = Instance.new("Frame")
dropdownList.Size = UDim2.new(1, 0, 0, 56)
dropdownList.Position = UDim2.new(0, 0, 0, 28)
dropdownList.BackgroundTransparency = 1
dropdownList.Parent = dropdownContainer

local function createListButton(text, pos, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 26)
    btn.Position = pos
    btn.BackgroundColor3 = Color3.fromRGB(8, 15, 35)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(180, 220, 255)
    btn.TextSize = 9
    btn.Font = Enum.Font.GothamMedium
    btn.Parent = dropdownList
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
    local listStroke = Instance.new("UIStroke", btn)
    listStroke.Color = ThemeBlue
    listStroke.Thickness = 1
    btn.MouseButton1Click:Connect(function()
        local char = player.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        local backpack = player:FindFirstChild("Backpack")
        if hum and backpack then
            local carpet = backpack:FindFirstChild("Flying Carpet")
            if carpet then hum:EquipTool(carpet) task.wait(0.1) end
        end
        callback()
    end)
end

createListButton("TP TO SPOT 1", UDim2.new(0, 0, 0, 0), function() executeTP(spot1_sequence) end)
createListButton("TP TO SPOT 2", UDim2.new(0, 0, 0, 28), function() executeTP(spot2_sequence) end)

local dropdownOpen = false
dropdownButton.MouseButton1Click:Connect(function()
    dropdownOpen = not dropdownOpen
    local targetSize = dropdownOpen and UDim2.new(0.9, 0, 0, 86) or UDim2.new(0.9, 0, 0, 26)
    dropdownButton.Text = dropdownOpen and "â¦ TP TO SPOT â²" or "â¦ TP TO SPOT â¼"
    TweenService:Create(dropdownContainer, TweenInfo.new(0.2), {Size = targetSize}):Play()
    -- also expand main frame when dropdown opens
    if dropdownOpen then
        TweenService:Create(mainFrame, TweenInfo.new(0.2), {Size = UDim2.new(0, 210, 0, FULL_HEIGHT + 60)}):Play()
    else
        TweenService:Create(mainFrame, TweenInfo.new(0.2), {Size = UDim2.new(0, 210, 0, FULL_HEIGHT)}):Play()
    end
end)

-- Discord
local discordText = Instance.new("TextLabel")
discordText.Size = UDim2.new(1, 0, 0, 12)
discordText.Position = UDim2.new(0, 0, 1, -14)
discordText.BackgroundTransparency = 1
discordText.Text = "â¦ M.T.S Hub â¦"
discordText.TextColor3 = ThemeBlue
discordText.TextSize = 7
discordText.Font = Enum.Font.GothamBold
discordText.Parent = mainFrame

-- =====================
-- MINIMIZE LOGIC
-- Minimized: show only Auto TP Left, Auto TP Right, Spam AP
-- =====================
local isMinimized = false

-- Mini buttons (shown only when minimized)
local miniFrame = Instance.new("Frame")
miniFrame.Size = UDim2.new(1, 0, 1, -55)
miniFrame.Position = UDim2.new(0, 0, 0, 55)
miniFrame.BackgroundTransparency = 1
miniFrame.Visible = false
miniFrame.Parent = mainFrame

local function createMiniBtn(text, yPos, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.9, 0, 0, 26)
    btn.Position = UDim2.new(0.05, 0, 0, yPos)
    btn.BackgroundColor3 = MidBg
    btn.Text = text
    btn.TextColor3 = GlowBlue
    btn.TextSize = 10
    btn.Font = Enum.Font.GothamBold
    btn.Parent = miniFrame
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
    local s = Instance.new("UIStroke", btn)
    s.Color = ThemeBlue
    s.Thickness = 1
    btn.MouseButton1Click:Connect(callback)
    return btn
end

local miniSpamBtn = createMiniBtn("â¦ SPAM AP NEAREST â¦", 0, spamAPNearest)
local miniLeftBtn = createMiniBtn("â Auto tp Left", 32, function() end)
local miniRightBtn = createMiniBtn("â Auto tp Right", 64, function() end)

-- Pulsing for mini spam button
task.spawn(function()
    local s = Instance.new("UIStroke", miniSpamBtn)
    s.Color = GlowBlue
    s.Thickness = 1.5
    while true do
        TweenService:Create(s, TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {Thickness = 2.5}):Play()
        task.wait(1)
        TweenService:Create(s, TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {Thickness = 1}):Play()
        task.wait(1)
    end
end)

minBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    if isMinimized then
        contentFrame.Visible = false
        miniFrame.Visible = true
        TweenService:Create(mainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, 210, 0, MINI_HEIGHT)
        }):Play()
        minBtn.Text = "+"
    else
        miniFrame.Visible = false
        contentFrame.Visible = true
        TweenService:Create(mainFrame, TweenInfo.new(0.3, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), {
            Size = UDim2.new(0, 210, 0, FULL_HEIGHT)
        }):Play()
        minBtn.Text = "â"
    end
end)

-- Dragging
local dragging, dragStart, startPos
mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true dragStart = input.Position startPos = mainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- Proximity Prompt
local currentEquipTask = nil
local isHolding = false

ProximityPromptService.PromptButtonHoldBegan:Connect(function(prompt, plr)
    if plr ~= player or not semiTPEnabled then return end
    isHolding = true
    if currentEquipTask then task.cancel(currentEquipTask) end
    currentEquipTask = task.spawn(function()
        task.wait(1)
        if isHolding and semiTPEnabled then
            local backpack = player:WaitForChild("Backpack", 2)
            if backpack then
                local carpet = backpack:FindFirstChild("Flying Carpet")
                if carpet and player.Character and player.Character:FindFirstChild("Humanoid") then
                    player.Character.Humanoid:EquipTool(carpet)
                end
            end
        end
    end)
end)

ProximityPromptService.PromptButtonHoldEnded:Connect(function(prompt, plr)
    if plr ~= player then return end
    isHolding = false
    if currentEquipTask then task.cancel(currentEquipTask) end
end)

ProximityPromptService.PromptTriggered:Connect(function(prompt, plr)
    if plr ~= player or not semiTPEnabled then return end
    local root = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
    if root then
        local backpack = player:FindFirstChild("Backpack")
        local hum = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
        if backpack and hum then
            local carpet = backpack:FindFirstChild("Flying Carpet")
            if carpet then hum:EquipTool(carpet) task.wait(0.1) end
        end
        local d1 = (root.Position - pos1).Magnitude
        local d2 = (root.Position - pos2).Magnitude
        root.CFrame = CFrame.new(d1 < d2 and pos1 or pos2)
        if _G.AutoPotion then
            local bp = player:FindFirstChild("Backpack")
            if bp then
                local potion = bp:FindFirstChild("Giant Potion")
                if potion and player.Character and player.Character:FindFirstChild("Humanoid") then
                    player.Character.Humanoid:EquipTool(potion)
                    task.wait(0.)
                    pcall(function() potion:Activate() end)
                end
            end
        end
    end
    isHolding = false
end)

-- Steal system
local combo_IsStealing = false
local combo_StealProgress = 0
local combo_ComboActive = false

task.spawn(function()
    while true do
        if combo_ComboActive then
            combo_IsStealing = true
            for i = 0, 100 do combo_StealProgress = i/100 task.wait(0.013) end
            combo_IsStealing = false combo_StealProgress = 0 task.wait(1)
        else task.wait(0.1) end
    end
end)

local CONFIG = {ANTI_STEAL_ACTIVE = false}
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local AnimalsData = require(ReplicatedStorage:WaitForChild("Datas"):WaitForChild("Animals"))
local allAnimalsCache = {}
local PromptMemoryCache = {}
local InternalStealCache = {}
local IsStealing = false
local StealProgress = 0
local CurrentStealTarget = nil
local AUTO_STEAL_PROX_RADIUS = 200

local function getHRP()
    local char = player.Character
    if not char then return nil end
    return char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso")
end

local function isMyBase(plotName)
    local plot = workspace.Plots:FindFirstChild(plotName)
    if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    return sign and sign:FindFirstChild("YourBase") and sign.YourBase.Enabled
end

local function scanSinglePlot(plot)
    if not plot or not plot:IsA("Model") or isMyBase(plot.Name) then return end
    local podiums = plot:FindFirstChild("AnimalPodiums")
    if not podiums then return end
    for _, podium in ipairs(podiums:GetChildren()) do
        if podium:IsA("Model") and podium:FindFirstChild("Base") then
            table.insert(allAnimalsCache, {
                plot = plot.Name, slot = podium.Name,
                worldPosition = podium:GetPivot().Position,
                uid = plot.Name .. "_" .. podium.Name,
            })
        end
    end
end

local function initializeScanner()
    task.wait(2)
    local plots = workspace:WaitForChild("Plots", 10)
    for _, plot in ipairs(plots:GetChildren()) do scanSinglePlot(plot) end
    plots.ChildAdded:Connect(scanSinglePlot)
    task.spawn(function()
        while task.wait(5) do
            table.clear(allAnimalsCache)
            for _, plot in ipairs(plots:GetChildren()) do scanSinglePlot(plot) end
        end
    end)
end

local function findPrompt(animal)
    local cached = PromptMemoryCache[animal.uid]
    if cached and cached.Parent then return cached end
    local plot = workspace.Plots:FindFirstChild(animal.plot)
    local podium = plot and plot.AnimalPodiums:FindFirstChild(animal.slot)
    local prompt = podium and podium.Base.Spawn.PromptAttachment:FindFirstChildOfClass("ProximityPrompt")
    if prompt then PromptMemoryCache[animal.uid] = prompt end
    return prompt
end

local function buildStealCallbacks(prompt)
    if InternalStealCache[prompt] then return end
    local data = {holdCallbacks = {}, triggerCallbacks = {}, ready = true}
    local ok1, conns1 = pcall(getconnections, prompt.PromptButtonHoldBegan)
    if ok1 then for _, c in ipairs(conns1) do table.insert(data.holdCallbacks, c.Function) end end
    local ok2, conns2 = pcall(getconnections, prompt.Triggered)
    if ok2 then for _, c in ipairs(conns2) do table.insert(data.triggerCallbacks, c.Function) end end
    InternalStealCache[prompt] = data
end

local function executeInternalStealAsync(prompt, animalData)
    local data = InternalStealCache[prompt]
    if not data or not data.ready or IsStealing then return end
    data.ready = false IsStealing = true StealProgress = 0 CurrentStealTarget = animalData
    local tpDone = false
    task.spawn(function()
        for _, fn in ipairs(data.holdCallbacks) do task.spawn(fn) end
        local startTime = tick()
        while tick() - startTime < 1.3 do
            StealProgress = (tick() - startTime) / 1.3
            if StealProgress >= 0.73 and not tpDone then
                tpDone = true
                local hrp = getHRP()
                if hrp then
                    local char = player.Character
                    local hum = char and char:FindFirstChildOfClass("Humanoid")
                    local backpack = player:FindFirstChild("Backpack")
                    if hum and backpack then
                        local carpet = backpack:FindFirstChild("Flying Carpet")
                        if carpet then hum:EquipTool(carpet) task.wait(0.1) end
                    end
                    hrp.CFrame = spot1_sequence[1] task.wait(0.1)
                    hrp.CFrame = spot1_sequence[2] task.wait(0.2)
                    local d1 = (hrp.Position - pos1).Magnitude
                    local d2 = (hrp.Position - pos2).Magnitude
                    hrp.CFrame = CFrame.new(d1 < d2 and pos1 or pos2)
                end
            end
            task.wait()
        end
        StealProgress = 1
        for _, fn in ipairs(data.triggerCallbacks) do task.spawn(fn) end
        task.wait(0.2)
        data.ready = true IsStealing = false StealProgress = 0
        CurrentStealTarget = nil CONFIG.ANTI_STEAL_ACTIVE = false
    end)
end

local function getNearestAnimal()
    local hrp = getHRP()
    if not hrp then return nil end
    local nearest, dist = nil, math.huge
    for _, animal in ipairs(allAnimalsCache) do
        local d = (hrp.Position - animal.worldPosition).Magnitude
        if d < dist and d <= AUTO_STEAL_PROX_RADIUS then dist = d nearest = animal end
    end
    return nearest
end

-- Wire up Auto TP buttons
local function doAutoTPLeft()
    if IsStealing then return end
    CONFIG.ANTI_STEAL_ACTIVE = true
    activeAutoWalkMode = "Left"
    local animal = getNearestAnimal()
    if not animal then return end
    local prompt = findPrompt(animal)
    if not prompt then return end
    buildStealCallbacks(prompt)
    executeInternalStealAsync(prompt, animal)
end

local function doAutoTPRight()
    if IsStealing then return end
    CONFIG.ANTI_STEAL_ACTIVE = true
    activeAutoWalkMode = "Right"
    local animal = getNearestAnimal()
    if not animal then return end
    local prompt = findPrompt(animal)
    if not prompt then return end
    buildStealCallbacks(prompt)
    local data = InternalStealCache[prompt]
    if not data or not data.ready or IsStealing then return end
    data.ready = false IsStealing = true StealProgress = 0 CurrentStealTarget = animal
    local tpDone = false
    task.spawn(function()
        for _, fn in ipairs(data.holdCallbacks) do task.spawn(fn) end
        local startTime = tick()
        while tick() - startTime < 1.3 do
            StealProgress = (tick() - startTime) / 1.3
            if StealProgress >= 0.73 and not tpDone then
                tpDone = true
                local hrp = getHRP()
                if hrp then
                    local char = player.Character
                    local hum = char and char:FindFirstChildOfClass("Humanoid")
                    local backpack = player:FindFirstChild("Backpack")
                    if hum and backpack then
                        local carpet = backpack:FindFirstChild("Flying Carpet")
                        if carpet then hum:EquipTool(carpet) task.wait(0.1) end
                    end
                    hrp.CFrame = spot2_sequence[1] task.wait(0.1)
                    hrp.CFrame = spot2_sequence[2] task.wait(0.2)
                    local d1 = (hrp.Position - pos1).Magnitude
                    local d2 = (hrp.Position - pos2).Magnitude
                    hrp.CFrame = CFrame.new(d1 < d2 and pos1 or pos2)
                end
            end
            task.wait()
        end
        StealProgress = 1
        for _, fn in ipairs(data.triggerCallbacks) do task.spawn(fn) end
        task.wait(0.2)
        data.ready = true IsStealing = false StealProgress = 0
        CurrentStealTarget = nil CONFIG.ANTI_STEAL_ACTIVE = false
    end)
end

autoTPLeftBtn.MouseButton1Click:Connect(doAutoTPLeft)
autoTPRightBtn.MouseButton1Click:Connect(doAutoTPRight)
miniLeftBtn.MouseButton1Click:Connect(doAutoTPLeft)
miniRightBtn.MouseButton1Click:Connect(doAutoTPRight)

task.spawn(function()
    while true do
        fill.Size = UDim2.new(math.clamp(StealProgress, 0, 1), 0, 1, 0)
        percentLabel.Text = (math.floor(StealProgress * 100 + 0.5)) .. "%"
        task.wait(0.02)
    end
end)

initializeScanner()

-- AUTO WALK LOOP
RunService.Heartbeat:Connect(function()
    if not autoWalkEnabled or not activeAutoWalkMode then return end
    local char = player.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end
    local currentTrigger, currentTarget = nil, nil
    if activeAutoWalkMode == "Right" then
        currentTrigger = Trigger_A currentTarget = Target_A
    elseif activeAutoWalkMode == "Left" then
        currentTrigger = Trigger_B currentTarget = Target_B
    end
    if not currentTrigger or not currentTarget then return end
    local distToTrigger = (root.Position - currentTrigger).Magnitude
    local distToTarget = (root.Position - currentTarget).Magnitude
    if distToTrigger < 6 and not isAutoWalking and not walkDebounce then
        walkDebounce = true
        task.spawn(function()
            task.wait(0.2)
            if autoWalkEnabled then isAutoWalking = true else walkDebounce = false end
        end)
    end
    if distToTarget < 4 and isAutoWalking then
        isAutoWalking = false walkDebounce = false
        activeAutoWalkMode = nil
        root.AssemblyLinearVelocity = Vector3.zero
    end
    if isAutoWalking then
        local dir = (currentTarget - root.Position).Unit
        root.AssemblyLinearVelocity = Vector3.new(dir.X * SPEED_BOOST, root.AssemblyLinearVelocity.Y, dir.Z * SPEED_BOOST)
    end
end)
