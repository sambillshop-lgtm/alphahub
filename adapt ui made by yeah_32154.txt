--made by yeah_32154
--made by yeah_32154
--made by yeah_32154
--made by yeah_32154
--made by yeah_32154
--made by yeah_32154
--made by yeah_32154

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local LP = Players.LocalPlayer

local existing = LP:FindFirstChild("PlayerGui") and LP.PlayerGui:FindFirstChild("AdaptUI")
if existing then existing:Destroy() end

local gui = Instance.new("ScreenGui")
gui.Name = "AdaptUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = LP:WaitForChild("PlayerGui")

-- ============ Helpers ============
local function new(class, props, children)
    local o = Instance.new(class)
    for k,v in pairs(props or {}) do o[k] = v end
    for _,c in ipairs(children or {}) do c.Parent = o end
    return o
end

local function corner(r) return new("UICorner",{CornerRadius=UDim.new(0,r)}) end
local function stroke(c,t,trans)
    return new("UIStroke",{Color=c or Color3.fromRGB(60,60,60),Thickness=t or 1,Transparency=trans or 0.3,ApplyStrokeMode=Enum.ApplyStrokeMode.Border})
end

-- ============ Main Frame ============
local main = new("Frame",{
    Name="Main",
    Parent=gui,
    Size=UDim2.fromOffset(330,540),
    Position=UDim2.new(0.5,-165,0.5,-270),
    BackgroundColor3=Color3.fromRGB(0,0,0),
    BackgroundTransparency=0,
    BorderSizePixel=0,
    Active=true,
    ClipsDescendants=true,
},{corner(14), stroke(Color3.fromRGB(25,25,25),1,0.2)})

-- ============ Background Image ============
new("ImageLabel",{
    Parent=main,
    Size=UDim2.fromScale(1,1),
    BackgroundTransparency=1,
    Image="rbxassetid://76428063854127",
    ScaleType=Enum.ScaleType.Crop,
    ImageTransparency=0,
    ZIndex=0,
})

-- Drag
do
    local dragging, dragStart, startPos
    main.InputBegan:Connect(function(input)
        if input.UserInputType==Enum.UserInputType.MouseButton1 or input.UserInputType==Enum.UserInputType.Touch then
            dragging=true; dragStart=input.Position; startPos=main.Position
            input.Changed:Connect(function() if input.UserInputState==Enum.UserInputState.End then dragging=false end end)
        end
    end)
    UIS.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType==Enum.UserInputType.MouseMovement or input.UserInputType==Enum.UserInputType.Touch) then
            local d = input.Position - dragStart
            main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+d.X, startPos.Y.Scale, startPos.Y.Offset+d.Y)
        end
    end)
end

-- ============ Header ============
local header = new("Frame",{
    Parent=main,
    Size=UDim2.new(1,0,0,270),
    BackgroundTransparency=1,
    BorderSizePixel=0,
    ClipsDescendants=true,
})

new("Frame",{
    Parent=header,
    Size=UDim2.new(1,-32,0,1),
    Position=UDim2.new(0,16,1,-1),
    BackgroundColor3=Color3.fromRGB(28,28,28),
    BorderSizePixel=0,
})

-- Thin divider under ADAPT title
new("Frame",{
    Parent=header,
    Size=UDim2.new(1,-32,0,1),
    Position=UDim2.new(0,16,0,108),
    BackgroundColor3=Color3.fromRGB(255,255,255),
    BackgroundTransparency=0,
    BorderSizePixel=0,
},{
    new("UIGradient",{
        Transparency=NumberSequence.new({
            NumberSequenceKeypoint.new(0,   1),
            NumberSequenceKeypoint.new(0.18,0.25),
            NumberSequenceKeypoint.new(0.5, 0.25),
            NumberSequenceKeypoint.new(0.82,0.25),
            NumberSequenceKeypoint.new(1,   1),
        }),
        Rotation=0,
    }),
})

-- ============ Close Button (small circle) ============
local closeBtn = new("TextButton",{
    Parent=header,
    Size=UDim2.fromOffset(24,24),
    Position=UDim2.new(1,-38,0,12),
    BackgroundColor3=Color3.fromRGB(20,20,20),
    BackgroundTransparency=0,
    Text="X",
    Font=Enum.Font.GothamBold,
    TextSize=11,
    TextColor3=Color3.fromRGB(255,255,255),
    AutoButtonColor=false,
    ZIndex=5,
},{
    new("UICorner",{CornerRadius=UDim.new(0,12)}),
    new("UIStroke",{Color=Color3.fromRGB(50,50,50),Thickness=1,Transparency=0.2}),
})
closeBtn.MouseButton1Click:Connect(function() gui.Enabled=false end)

-- ============ Tab content container ============
local body = new("ScrollingFrame",{
    Parent=main,
    Position=UDim2.fromOffset(0,110),
    Size=UDim2.new(1,0,1,-170),
    BackgroundTransparency=1,
    BorderSizePixel=0,
    ScrollBarThickness=2,
    ScrollBarImageColor3=Color3.fromRGB(60,60,60),
    CanvasSize=UDim2.new(),
    AutomaticCanvasSize=Enum.AutomaticSize.Y,
},{
    new("UIPadding",{PaddingLeft=UDim.new(0,16),PaddingRight=UDim.new(0,16),PaddingTop=UDim.new(0,14),PaddingBottom=UDim.new(0,16)}),
    new("UIListLayout",{Padding=UDim.new(0,10),SortOrder=Enum.SortOrder.LayoutOrder}),
})

-- ============ Components ============
local function sectionLabel(text)
    local lbl = new("TextLabel",{
        Size=UDim2.new(1,0,0,20),
        BackgroundTransparency=1,
        Text=string.upper(text),
        Font=Enum.Font.GothamBold,
        TextSize=10,
        TextColor3=Color3.fromRGB(200,200,200),
        TextXAlignment=Enum.TextXAlignment.Left,
    })
    return lbl
end

local function row(height)
    return new("Frame",{
        Size=UDim2.new(1,0,0,height or 48),
        BackgroundColor3=Color3.fromRGB(10,10,10),
        BorderSizePixel=0,
    },{corner(10), stroke(Color3.fromRGB(55,55,55),1,0)})
end

local function makeSlider(name, min, max, default, callback)
    local r = row(46)
    new("TextLabel",{Parent=r,Size=UDim2.new(1,-80,1,0),Position=UDim2.fromOffset(14,0),BackgroundTransparency=1,
        Text=name,Font=Enum.Font.GothamMedium,TextSize=14,TextColor3=Color3.fromRGB(235,235,235),TextXAlignment=Enum.TextXAlignment.Left})
    local valBox = new("TextBox",{Parent=r,Size=UDim2.fromOffset(58,26),Position=UDim2.new(1,-66,0.5,-13),
        BackgroundColor3=Color3.fromRGB(18,18,18),Text=tostring(default),Font=Enum.Font.GothamBold,TextSize=13,
        TextColor3=Color3.fromRGB(235,235,235),TextXAlignment=Enum.TextXAlignment.Center,
        ClearTextOnFocus=true,BorderSizePixel=0},{corner(6)})
    valBox.FocusLost:Connect(function()
        local n = tonumber(valBox.Text)
        if n then
            n = math.clamp(n, min, max)
            if max-min<=10 then n=math.floor(n*10+0.5)/10 else n=math.floor(n+0.5) end
            valBox.Text = tostring(n)
            if callback then callback(n) end
        else
            valBox.Text = tostring(default)
        end
    end)
    return r
end

-- ============ Keybind System ============
local listeningBtn = nil
local keybindMap = {}
UIS.InputBegan:Connect(function(input, gameProcessed)
    if input.UserInputType ~= Enum.UserInputType.Keyboard then return end
    if input.KeyCode == Enum.KeyCode.Escape and listeningBtn then
        listeningBtn.btn.Text = listeningBtn.prev
        listeningBtn.btn.TextColor3 = Color3.fromRGB(220,220,220)
        listeningBtn = nil
        return
    end
    if listeningBtn then
        listeningBtn.onKey(input.KeyCode)
        listeningBtn = nil
        return
    end
    local cb = keybindMap[input.KeyCode]
    if cb then cb() end
end)

local function makeToggle(name, default, keybind, callback, showDropdown)
    local state = default
    local expanded = false
    local boundKey = nil

    local container = new("Frame",{
        Size=UDim2.new(1,0,0,46),
        BackgroundTransparency=1,
        BorderSizePixel=0,
        AutomaticSize=Enum.AutomaticSize.Y,
        ClipsDescendants=false,
    })
    new("UIListLayout",{Parent=container,SortOrder=Enum.SortOrder.LayoutOrder,Padding=UDim.new(0,4)})

    -- main line
    local r = new("Frame",{
        Parent=container,
        Size=UDim2.new(1,0,0,46),
        BackgroundColor3=Color3.fromRGB(10,10,10),
        BorderSizePixel=0,
        LayoutOrder=1,
        ClipsDescendants=true,
    },{corner(10), stroke(Color3.fromRGB(55,55,55),1,0)})

    -- name tag
    new("TextLabel",{
        Parent=r,
        Size=UDim2.new(1,-125,1,0),
        Position=UDim2.fromOffset(14,0),
        BackgroundTransparency=1,
        Text=name,
        Font=Enum.Font.GothamBold,
        TextSize=13,
        TextColor3=Color3.fromRGB(235,235,235),
        TextXAlignment=Enum.TextXAlignment.Left,
    })

    -- Dropdown "v" button
    local dropBtn
    if showDropdown then
        dropBtn = new("TextButton",{
            Parent=r,
            Size=UDim2.fromOffset(22,22),
            Position=UDim2.new(1,-120,0.5,-11),
            BackgroundColor3=Color3.fromRGB(18,18,18),
            Text="v",
            Font=Enum.Font.GothamBold,
            TextSize=11,
            TextColor3=Color3.fromRGB(160,160,160),
            AutoButtonColor=false,
            BorderSizePixel=0,
        },{corner(5)})

        -- Separator
        new("Frame",{
            Parent=r,
            Size=UDim2.new(0,1,0.5,0),
            Position=UDim2.new(1,-96,0.25,0),
            BackgroundColor3=Color3.fromRGB(60,60,60),
            BorderSizePixel=0,
        })
    end

    -- Keybind button
    local initText = keybind and string.upper(keybind) or "-"
    local kbBtn = new("TextButton",{
        Parent=r,
        Size=UDim2.fromOffset(36,22),
        Position=UDim2.new(1,-92,0.5,-11),
        BackgroundColor3=Color3.fromRGB(18,18,18),
        Text=initText,
        Font=Enum.Font.GothamBold,
        TextSize=12,
        TextColor3=Color3.fromRGB(220,220,220),
        AutoButtonColor=false,
        BorderSizePixel=0,
    },{corner(5)})

    -- Separator
    new("Frame",{
        Parent=r,
        Size=UDim2.new(0,1,0.5,0),
        Position=UDim2.new(1,-53,0.25,0),
        BackgroundColor3=Color3.fromRGB(60,60,60),
        BorderSizePixel=0,
    })

    -- toggle switch
    local sw = new("TextButton",{
        Parent=r,
        Size=UDim2.fromOffset(38,20),
        Position=UDim2.new(1,-48,0.5,-10),
        BackgroundColor3=default and Color3.fromRGB(240,240,240) or Color3.fromRGB(55,55,55),
        Text="",AutoButtonColor=false,BorderSizePixel=0,
    },{corner(10)})
    local dot = new("Frame",{
        Parent=sw,
        Size=UDim2.fromOffset(16,16),
        Position=default and UDim2.new(1,-18,0.5,-8) or UDim2.new(0,2,0.5,-8),
        BackgroundColor3=default and Color3.fromRGB(20,20,20) or Color3.fromRGB(200,200,200),
        BorderSizePixel=0,
    },{corner(8)})

    local function updateToggle(newState)
        state = newState
        TweenService:Create(sw,TweenInfo.new(0.18),{BackgroundColor3=state and Color3.fromRGB(240,240,240) or Color3.fromRGB(55,55,55)}):Play()
        TweenService:Create(dot,TweenInfo.new(0.18),{
            Position=state and UDim2.new(1,-18,0.5,-8) or UDim2.new(0,2,0.5,-8),
            BackgroundColor3=state and Color3.fromRGB(20,20,20) or Color3.fromRGB(200,200,200),
        }):Play()
        if callback then callback(state) end
    end

    local toggleFn = function()
        updateToggle(not state)
    end

    sw.MouseButton1Click:Connect(function()
        updateToggle(not state)
    end)

    -- V1/V2 select
    local choicePanel = new("Frame",{
        Parent=container,
        Size=UDim2.new(1,0,0,28),
        BackgroundTransparency=1,
        BorderSizePixel=0,
        Visible=false,
        LayoutOrder=2,
    })

    local v1Btn = new("TextButton",{
        Parent=choicePanel,
        Size=UDim2.new(0.5,-3,1,0),
        Position=UDim2.fromOffset(0,0),
        BackgroundColor3=Color3.fromRGB(235,235,235),
        Text="V1",
        Font=Enum.Font.GothamBold,
        TextSize=13,
        TextColor3=Color3.fromRGB(15,15,15),
        AutoButtonColor=false,
        BorderSizePixel=0,
    },{corner(10)})

    local v2Btn = new("TextButton",{
        Parent=choicePanel,
        Position=UDim2.new(0.5,3,0,0),
        Size=UDim2.new(0.5,-3,1,0),
        BackgroundColor3=Color3.fromRGB(18,18,18),
        Text="V2",
        Font=Enum.Font.GothamBold,
        TextSize=13,
        TextColor3=Color3.fromRGB(120,120,120),
        AutoButtonColor=false,
        BorderSizePixel=0,
    },{corner(10), stroke(Color3.fromRGB(40,40,40),1,0)})

    -- Dropdown
    if dropBtn then
        dropBtn.MouseButton1Click:Connect(function()
            expanded = not expanded
            choicePanel.Visible = expanded
            dropBtn.Text = expanded and "^" or "v"
        end)
    end

    -- Keybind attach
    kbBtn.MouseButton1Click:Connect(function()
        local prevText = kbBtn.Text
        listeningBtn = {
            btn = kbBtn,
            prev = prevText,
            onKey = function(kc)
                if boundKey then keybindMap[boundKey] = nil end
                boundKey = kc
                keybindMap[kc] = toggleFn
                local nm = kc.Name
                kbBtn.Text = #nm <= 3 and nm:upper() or nm:sub(1,2):upper()
                kbBtn.TextColor3 = Color3.fromRGB(220,220,220)
            end
        }
        kbBtn.Text = ".."
        kbBtn.TextColor3 = Color3.fromRGB(255,215,50)
    end)

    if keybind then
        for _, kc in pairs(Enum.KeyCode:GetEnumItems()) do
            if kc.Name:lower() == keybind:lower() then
                boundKey = kc
                keybindMap[kc] = toggleFn
                break
            end
        end
    end

    -- V1 / V2
    v1Btn.MouseButton1Click:Connect(function()
        v1Btn.BackgroundColor3 = Color3.fromRGB(235,235,235)
        v1Btn.TextColor3 = Color3.fromRGB(15,15,15)
        v2Btn.BackgroundColor3 = Color3.fromRGB(18,18,18)
        v2Btn.TextColor3 = Color3.fromRGB(120,120,120)
    end)
    v2Btn.MouseButton1Click:Connect(function()
        v2Btn.BackgroundColor3 = Color3.fromRGB(235,235,235)
        v2Btn.TextColor3 = Color3.fromRGB(15,15,15)
        v1Btn.BackgroundColor3 = Color3.fromRGB(18,18,18)
        v1Btn.TextColor3 = Color3.fromRGB(120,120,120)
    end)

    return container
end

-- ============ Tabs ============
local tabs = {"Movement","Combat","Utility","Settings"}
local tabContents = {}

local function buildCombat(parent)
    sectionLabel("Steal Configuration").Parent = parent
    makeSlider("Radius",1,100,62).Parent = parent
    makeSlider("Duration",0.1,5,1.3).Parent = parent
    makeToggle("Auto Steal",false).Parent = parent
    sectionLabel("Bat Aimbot").Parent = parent
    makeToggle("Bat Aimbot",true,"e",nil,true).Parent = parent
    makeToggle("Desync Aimbot",true,"c").Parent = parent
    sectionLabel("Auto Path").Parent = parent
    makeToggle("Auto Left",true,"u").Parent = parent
    makeToggle("Auto Right",false,"i").Parent = parent
end

local function buildMovement(parent)
end

local function buildUtility(parent)
end

local function buildSettings(parent)
    sectionLabel("Settings").Parent = parent
    makeToggle("Auto Save Config",true).Parent = parent
    local r = row(46)
    r.Parent = parent
    new("TextLabel",{Parent=r,Size=UDim2.new(1,-28,1,0),Position=UDim2.fromOffset(14,0),BackgroundTransparency=1,
        Text="Version  1.0.0",Font=Enum.Font.GothamMedium,TextSize=14,
        TextColor3=Color3.fromRGB(180,180,180),TextXAlignment=Enum.TextXAlignment.Left})
end

local builders = {Movement=buildMovement,Combat=buildCombat,Utility=buildUtility,Settings=buildSettings}

local function clear() for _,c in ipairs(body:GetChildren()) do if not c:IsA("UIPadding") and not c:IsA("UIListLayout") then c:Destroy() end end end

-- ============ Tab Bar ============
local tabBar = new("Frame",{
    Parent=main,
    Position=UDim2.new(0,12,1,-56),
    Size=UDim2.new(1,-24,0,46),
    BackgroundTransparency=1,
    BorderSizePixel=0,
},{
    new("UIListLayout",{FillDirection=Enum.FillDirection.Horizontal,Padding=UDim.new(0,0),SortOrder=Enum.SortOrder.LayoutOrder,VerticalAlignment=Enum.VerticalAlignment.Center,HorizontalAlignment=Enum.HorizontalAlignment.Center}),
    new("UIPadding",{PaddingLeft=UDim.new(0,0),PaddingRight=UDim.new(0,0),PaddingTop=UDim.new(0,4),PaddingBottom=UDim.new(0,4)}),
})

local current
local tabBorders = {}
local tabButtons = {}
local function selectTab(name)
    current = name
    clear()
    builders[name](body)
    for n,btn in pairs(tabButtons) do
        local active = n==name
        local border = tabBorders[n]
        if active then
            if border then border.BackgroundTransparency = 0 end
            btn.TextColor3 = Color3.fromRGB(255,255,255)
            btn.Font = Enum.Font.GothamBold
        else
            if border then border.BackgroundTransparency = 0.65 end
            btn.TextColor3 = Color3.fromRGB(120,120,120)
            btn.Font = Enum.Font.GothamMedium
        end
    end
end

for i,name in ipairs(tabs) do
    local border = new("Frame",{
        Parent=tabBar,
        Size=UDim2.new(0.25,0,1,0),
        BackgroundColor3=Color3.fromRGB(255,255,255),
        BackgroundTransparency=0.65,
        BorderSizePixel=0,
        LayoutOrder=i,
    },{corner(10)})
    local b = new("TextButton",{
        Parent=border,
        Size=UDim2.new(1,-2,1,-2),
        Position=UDim2.fromOffset(1,1),
        BackgroundColor3=Color3.fromRGB(10,10,10),
        BackgroundTransparency=0,
        BorderSizePixel=0,
        AutoButtonColor=false,
        Text=name,
        Font=Enum.Font.GothamMedium,
        TextSize=12,
        TextColor3=Color3.fromRGB(120,120,120),
    },{corner(9)})
    tabBorders[name] = border
    tabButtons[name] = b
    b.MouseButton1Click:Connect(function() selectTab(name) end)
end

selectTab("Combat")

print("[ADAPT] UI loaded.")
print("made by yeah_32154")