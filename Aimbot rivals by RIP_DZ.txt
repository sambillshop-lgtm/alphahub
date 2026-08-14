-- aimbot rivals for mobile working and undetected deobf by @5jr5 rip_dz in discord.gg/ZeMBpepp5
local aimbotActive = false
local currentCamera = workspace.CurrentCamera
local localPlayer = game:GetService("Players").LocalPlayer
local tweenService = game:GetService("TweenService")
local userInputService = game:GetService("UserInputService")
local runService = game:GetService("RunService")

-- FOV par défaut
local fovRadius = 80

-- GUI principale
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.ResetOnSpawn = false
screenGui.Name = "AimbotGUI"

-- Bouton flottant
local toggleButton = Instance.new("ImageButton", screenGui)
toggleButton.Size = UDim2.new(0, 50, 0, 50)
toggleButton.Position = UDim2.new(0, 10, 0, 10)
toggleButton.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
toggleButton.Image = "rbxassetid://132533655213092"
toggleButton.ImageColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.ScaleType = Enum.ScaleType.Fit
toggleButton.ZIndex = 10
Instance.new("UICorner", toggleButton).CornerRadius = UDim.new(1, 0)

-- Animation du bouton
local tweenInfo = TweenInfo.new(0.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
toggleButton.MouseEnter:Connect(function()
    local tween = tweenService:Create(toggleButton, tweenInfo, {Size = UDim2.new(0, 55, 0, 55)})
    tween:Play()
end)

toggleButton.MouseLeave:Connect(function()
    local tween = tweenService:Create(toggleButton, tweenInfo, {Size = UDim2.new(0, 50, 0, 50)})
    tween:Play()
end)

-- Fenêtre principale
local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 180, 0, 260)
mainFrame.Position = UDim2.new(0, 70, 0, 10)
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
mainFrame.Active = true
mainFrame.Draggable = true
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 10)

-- Gestion de l'affichage
local isVisible = true
toggleButton.MouseButton1Click:Connect(function()
    isVisible = not isVisible
    mainFrame.Visible = isVisible
end)

-- Titre
local titleLabel = Instance.new("TextLabel", mainFrame)
titleLabel.Size = UDim2.new(1, 0, 0, 30)
titleLabel.Text = "Aimbot Mobile"
titleLabel.BackgroundTransparency = 1
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextScaled = true

-- Bouton fermeture
local closeButton = Instance.new("TextButton", mainFrame)
closeButton.Size = UDim2.new(0, 30, 0, 30)
closeButton.Position = UDim2.new(1, -35, 0, 223)
closeButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
closeButton.Text = "X"
closeButton.TextColor3 = Color3.new(1, 1, 1)
closeButton.Font = Enum.Font.GothamBold
closeButton.TextScaled = true
Instance.new("UICorner", closeButton).CornerRadius = UDim.new(1, 0)

-- Cercle FOV (Drawing)
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 1.5
fovCircle.Color = Color3.fromRGB(255, 255, 255)
fovCircle.Filled = false
fovCircle.Visible = false

-- Bouton activer/désactiver Aimbot
local aimbotToggle = Instance.new("TextButton", mainFrame)
aimbotToggle.Size = UDim2.new(1, -20, 0, 30)
aimbotToggle.Position = UDim2.new(0, 10, 0, 35)
aimbotToggle.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
aimbotToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
aimbotToggle.Text = "Ativar Aimbot: OFF"
aimbotToggle.Font = Enum.Font.Gotham
aimbotToggle.TextScaled = true
Instance.new("UICorner", aimbotToggle).CornerRadius = UDim.new(0, 6)

aimbotToggle.MouseButton1Click:Connect(function()
    aimbotActive = not aimbotActive
    aimbotToggle.Text = "Ativar Aimbot: " .. (aimbotActive and "ON" or "OFF")
    fovCircle.Visible = aimbotActive
end)

-- Label FOV
local fovLabel = Instance.new("TextLabel", mainFrame)
fovLabel.Size = UDim2.new(1, -20, 0, 30)
fovLabel.Position = UDim2.new(0, 10, 0, 70)
fovLabel.BackgroundTransparency = 1
fovLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
fovLabel.Text = "FOV: 80"
fovLabel.Font = Enum.Font.Gotham
fovLabel.TextScaled = true

-- Slider FOV
local fovSlider = Instance.new("TextButton", mainFrame)
fovSlider.Size = UDim2.new(1, -20, 0, 15)
fovSlider.Position = UDim2.new(0, 10, 0, 105)
fovSlider.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
fovSlider.Text = ""
fovSlider.AutoButtonColor = false

local sliderHandle = Instance.new("Frame", fovSlider)
sliderHandle.Size = UDim2.new(0, 10, 1, 0)
sliderHandle.Position = UDim2.new(0.5, -5, 0, 0)
sliderHandle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
Instance.new("UICorner", sliderHandle).CornerRadius = UDim.new(1, 0)

-- Gestion du slider
userInputService.InputChanged:Connect(function(input)
    if aimbotActive and input.UserInputType == Enum.UserInputType.Touch then
        local newPosition = math.clamp(input.Position.X - fovSlider.AbsolutePosition.X, 0, fovSlider.AbsoluteSize.X)
        sliderHandle.Position = UDim2.new(0, newPosition - 5, 0, 0)
        fovRadius = math.floor(newPosition / fovSlider.AbsoluteSize.X * 300)
        fovLabel.Text = "FOV: " .. fovRadius
    end
end)

-- Menu déroulant pour la cible
local dropdownFrame = Instance.new("Frame", mainFrame)
dropdownFrame.Size = UDim2.new(0, 160, 0, 60)
dropdownFrame.Position = UDim2.new(0, 10, 0, 130)
dropdownFrame.BackgroundTransparency = 1

local dropdownLabel = Instance.new("TextLabel", dropdownFrame)
dropdownLabel.Text = "Partie ciblée"
dropdownLabel.Size = UDim2.new(1, 0, 0, 16)
dropdownLabel.TextSize = 13
dropdownLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
dropdownLabel.BackgroundTransparency = 1
dropdownLabel.Font = Enum.Font.Gotham

local dropdownButton = Instance.new("TextButton", dropdownFrame)
dropdownButton.Size = UDim2.new(1, 0, 0, 25)
dropdownButton.Position = UDim2.new(0, 0, 0, 16)
dropdownButton.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
dropdownButton.BorderSizePixel = 0
dropdownButton.Text = "Head"
dropdownButton.TextSize = 14
dropdownButton.TextColor3 = Color3.fromRGB(255, 255, 255)
dropdownButton.Font = Enum.Font.GothamBold

local dropdownList = Instance.new("Frame", dropdownFrame)
dropdownList.Visible = false
dropdownList.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
dropdownList.BorderSizePixel = 0
dropdownList.Position = UDim2.new(0, 0, 1, 0)
dropdownList.Size = UDim2.new(1, 0, 0, 100)
dropdownList.ClipsDescendants = true

local targetParts = {"Head", "Torso", "HumanoidRootPart", "LeftLeg", "RightLeg"}
local selectedTarget = "Head"

for i, partName in ipairs(targetParts) do
    local option = Instance.new("TextButton", dropdownList)
    option.Text = partName
    option.Size = UDim2.new(1, 0, 0, 20)
    option.Position = UDim2.new(0, 0, 0, (i - 1) * 20)
    option.BackgroundTransparency = 1
    option.TextSize = 14
    option.TextColor3 = Color3.fromRGB(255, 255, 255)
    option.Font = Enum.Font.Gotham
    
    option.MouseButton1Click:Connect(function()
        dropdownButton.Text = partName
        selectedTarget = partName
        dropdownList.Visible = false
    end)
end

dropdownButton.MouseButton1Click:Connect(function()
    dropdownList.Visible = not dropdownList.Visible
end)

-- Fonction principale de l'aimbot
local function getClosestPlayer()
    local center = Vector2.new(currentCamera.ViewportSize.X / 2, currentCamera.ViewportSize.Y / 2)
    local closestDistance = fovRadius
    local closestPlayer = nil
    
    for _, player in pairs(game:GetService("Players"):GetPlayers()) do
        if player ~= localPlayer and player.Character and player.Character:FindFirstChild(selectedTarget) then
            local targetPart = player.Character[selectedTarget]
            local screenPos, onScreen = currentCamera:WorldToViewportPoint(targetPart.Position)
            
            if onScreen then
                local distance = (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude
                if distance < closestDistance then
                    closestDistance = distance
                    closestPlayer = player
                end
            end
        end
    end
    
    return closestPlayer
end

-- Render de l'aimbot
runService:BindToRenderStep("AimbotRender", Enum.RenderPriority.Camera.Value + 1, function()
    local center = Vector2.new(currentCamera.ViewportSize.X / 2, currentCamera.ViewportSize.Y / 2)
    fovCircle.Position = center
    fovCircle.Radius = fovRadius
    fovCircle.Visible = aimbotActive
    
    if aimbotActive then
        local targetPlayer = getClosestPlayer()
        if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild(selectedTarget) then
            currentCamera.CFrame = CFrame.new(currentCamera.CFrame.Position, targetPlayer.Character[selectedTarget].Position)
        end
    end
end)

-- Message d'activation
local notificationFrame = Instance.new("Frame", screenGui)
notificationFrame.Size = UDim2.new(0, 310, 0, 60)
notificationFrame.Position = UDim2.new(0.5, -155, 1, 100)
notificationFrame.AnchorPoint = Vector2.new(0.5, 1)
notificationFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
notificationFrame.BackgroundTransparency = 0.1
notificationFrame.BorderSizePixel = 0
Instance.new("UICorner", notificationFrame).CornerRadius = UDim.new(0, 10)

local iconImage = Instance.new("ImageLabel", notificationFrame)
iconImage.Size = UDim2.new(0, 40, 0, 40)
iconImage.Position = UDim2.new(0, 5, 0.5, -20)
iconImage.BackgroundTransparency = 1
iconImage.Image = "rbxassetid://77474537431792"
iconImage.ZIndex = 1

local notificationText = Instance.new("TextLabel", notificationFrame)
notificationText.Size = UDim2.new(1, -60, 1, 0)
notificationText.Position = UDim2.new(0, 55, 0, 0)
notificationText.BackgroundTransparency = 1
notificationText.Text = "Script Activé\nBy ZecadaDiv"
notificationText.TextColor3 = Color3.fromRGB(255, 255, 255)
notificationText.TextSize = 18
notificationText.Font = Enum.Font.GothamBold
notificationText.TextXAlignment = Enum.TextXAlignment.Left
notificationText.TextYAlignment = Enum.TextYAlignment.Center
notificationText.TextWrapped = true

local notificationSound = Instance.new("Sound", screenGui)
notificationSound.SoundId = "rbxassetid://6026984224"
notificationSound.Volume = 1
notificationSound:Play()

local notificationTween = tweenService:Create(notificationFrame, TweenInfo.new(0.5, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {
    Position = UDim2.new(0.8, -10, 1, -99)
})
notificationTween:Play()

task.delay(3, function()
    local closeTween = tweenService:Create(notificationFrame, TweenInfo.new(0.5, Enum.EasingStyle.Quint, Enum.EasingDirection.In), {
        Position = UDim2.new(0.8, -10, 1, 100)
    })
    closeTween:Play()
    closeTween.Completed:Connect(function()
        notificationFrame:Destroy()
    end)
end)