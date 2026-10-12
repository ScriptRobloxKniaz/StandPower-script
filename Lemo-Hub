--[[
    ═══════════════════════════════════════════════════════════
    LEMO-HUB v10 | UNIVERSAL MOBILE
    Enhanced Trigger (Single/Burst/Spam + Humanize) 
    Anti-Detect • No Recoil/Spread • Anti-Fling • Owner RGB
    ═══════════════════════════════════════════════════════════
]]

local Players    = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInput  = game:GetService("UserInputService")
local TweenSvc   = game:GetService("TweenService")
local StarterGui = game:GetService("StarterGui")
local VIM        = game:GetService("VirtualInputManager")
local Lighting   = game:GetService("Lighting")

local LP     = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local VALID_KEY    = "Lemov1"
local DISCORD_LINK = "https://discord.gg/h7SemBSHn"

local T = {
    Bg = Color3.fromRGB(8, 5, 7), Bg2 = Color3.fromRGB(18, 8, 10),
    Card = Color3.fromRGB(22, 11, 13), CardHover = Color3.fromRGB(32, 15, 17),
    CardGrad = Color3.fromRGB(28, 14, 16),
    Border = Color3.fromRGB(72, 22, 28),
    Blood = Color3.fromRGB(180, 20, 30), Blood2 = Color3.fromRGB(120, 10, 20),
    Crimson = Color3.fromRGB(230, 40, 50), Crimson2 = Color3.fromRGB(255, 70, 80),
    Gold = Color3.fromRGB(220, 175, 90), Gold2 = Color3.fromRGB(160, 120, 55),
    Text = Color3.fromRGB(245, 235, 235), TextDim = Color3.fromRGB(160, 130, 130),
    TextMuted = Color3.fromRGB(90, 70, 70), ToggleOff = Color3.fromRGB(52, 26, 28),
    Green = Color3.fromRGB(120, 210, 120), Red = Color3.fromRGB(230, 90, 90),
    Skin = Color3.fromRGB(215, 200, 80), Pants = Color3.fromRGB(60, 80, 140)
}

_G.LEMO = {
    Aimbot = false, AimFOV = 120, AimSmooth = 0.35,
    AimPart = "Head", AimTeam = true, AimWall = true, AimPredict = true,
    AimSticky = true, AimPrecision = true,
    AimDeadzone = 2.5, AimMaxSpeed = 0.9,
    Silent = false, SilentFOV = 100, SilentChance = 100,
    Trigger = false, TriggerFOV = 60, TriggerDelay = 0.03, TriggerTeam = true,
    TriggerMode = "Single", TriggerBurst = 3, TriggerBurstDelay = 0.05,
    TriggerHumanize = true, TriggerMinDelay = 0.02, TriggerMaxDelay = 0.12,
    TriggerMaxDist = 100, TriggerHealthMin = 1, TriggerWallCheck = false,
    TriggerIndicator = true, TriggerAimRequired = false,
    NoRecoil = false, NoRecoilStrength = 1.0,
    NoSpread = false, AntiFling = false,
    AntiDetect = true, InitDelay = true,
    ESP = false, ESPBox = true, ESPName = true, ESPHealth = true,
    ESPDist = true, ESPTracer = false, Skeleton = true, ESPTeam = true,
    ESPMaxDist = 1500, ESPHealthBar = true,
    FOVCircle = true,
    Loops = {}, Cache = {}, SkelCache = {},
    CurrentTarget = nil, MenuOpen = false, Verified = false,
    BackgroundAssetId = "74964129254229"
}

-- ═══════════════════════════════════════
-- ANTI-DETECT INIT
-- ═══════════════════════════════════════
local function environmentCheck()
    if getgenv and getgenv().LEMO_LOADED then return false end
    if getgenv then getgenv().LEMO_LOADED = true end
    return true
end

local function initDelay()
    if _G.LEMO.InitDelay then
        task.wait(math.random(1, 3) + math.random())
    end
end

if not environmentCheck() then return end
initDelay()

local function randomizedName(prefix)
    return (prefix or "svc_") .. tostring(math.random(100000, 999999))
end

local function jitter(base)
    return base * (0.8 + math.random() * 0.4)
end

-- ═══════════════════════════════════════
-- CORE UTILS
-- ═══════════════════════════════════════
local function getChar()
    local c = LP.Character
    if not c or not c.Parent then return nil end
    local h = c:FindFirstChildOfClass("Humanoid")
    local r = c:FindFirstChild("HumanoidRootPart")
    if not h or not r or h.Health <= 0 then return nil end
    return c, h, r
end

local function isAlive(plr)
    if not plr or not plr.Character then return false end
    local hum = plr.Character:FindFirstChildOfClass("Humanoid")
    local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
    return hum and hrp and hum.Health > 0
end

local function isTeam(plr)
    if not _G.LEMO.AimTeam then return false end
    if not LP.Team or not plr.Team then return false end
    return LP.Team == plr.Team
end

local function isVisible(toPart)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {LP.Character, Camera}
    local dir = toPart.Position - Camera.CFrame.Position
    local hit = workspace:Raycast(Camera.CFrame.Position, dir, params)
    return not hit or hit.Instance:IsDescendantOf(toPart.Parent)
end

local function getPart(plr, name)
    if not plr or not plr.Character then return nil end
    return plr.Character:FindFirstChild(name or "Head")
end

local function getTarget(fov, partName, teamCheck, wallCheck)
    local _, _, myHrp = getChar()
    if not myHrp then return nil, nil end
    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    local best, bestPart, bestD = nil, nil, math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr == LP then continue end
        if not isAlive(plr) then continue end
        if teamCheck and isTeam(plr) then continue end
        local part = getPart(plr, partName)
        if not part then continue end
        local sp, onScreen = Camera:WorldToViewportPoint(part.Position)
        if not onScreen then continue end
        local d = (Vector2.new(sp.X, sp.Y) - center).Magnitude
        if d > fov then continue end
        if wallCheck and not isVisible(part) then continue end
        if d < bestD then best, bestPart, bestD = plr, part, d end
    end
    return best, bestPart
end

-- ═══════════════════════════════════════
-- SMOOTH AIMBOT
-- ═══════════════════════════════════════
_G.LEMO.Loops.Aimbot = RunService.RenderStepped:Connect(function(dt)
    if not _G.LEMO.Aimbot then return end
    local target, part
    if _G.LEMO.AimSticky and _G.LEMO.CurrentTarget then
        local ct = _G.LEMO.CurrentTarget
        if isAlive(ct) then
            local tp = getPart(ct, _G.LEMO.AimPart)
            if tp then
                local sp, onScreen = Camera:WorldToViewportPoint(tp.Position)
                if onScreen then
                    local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
                    local d = (Vector2.new(sp.X, sp.Y) - center).Magnitude
                    if d <= _G.LEMO.AimFOV then target = ct; part = tp end
                end
            end
        end
    end
    if not target then
        target, part = getTarget(_G.LEMO.AimFOV, _G.LEMO.AimPart, true, true)
        _G.LEMO.CurrentTarget = target
    end
    if not target or not part then return end
    local aimPos = part.Position
    if _G.LEMO.AimPredict then
        local vel = part.AssemblyLinearVelocity or part.Velocity
        if vel then
            local ping = LP:GetNetworkPing()
            aimPos = aimPos + vel * math.clamp(ping * 1.5, 0.05, 0.4)
        end
    end
    if _G.LEMO.AimPrecision and _G.LEMO.AimPart == "Head" then
        aimPos = aimPos + Vector3.new(0, 0.15, 0)
    end
    local curCF = Camera.CFrame
    local dirToTarget = (aimPos - curCF.Position).Unit
    local currentDir = curCF.LookVector
    local dot = math.clamp(currentDir:Dot(dirToTarget), -1, 1)
    local angle = math.deg(math.acos(dot))
    if angle <= _G.LEMO.AimDeadzone then return end
    local targetCF = CFrame.new(curCF.Position, aimPos)
    local baseAlpha = 1 - math.exp(-_G.LEMO.AimSmooth * 8 * dt)
    local proximity = math.clamp(angle / 60, 0, 1)
    local easedAlpha = baseAlpha * (0.3 + proximity * 0.7)
    easedAlpha = math.min(easedAlpha, _G.LEMO.AimMaxSpeed)
    Camera.CFrame = curCF:Lerp(targetCF, easedAlpha)
end)

-- ═══════════════════════════════════════
-- SILENT AIM
-- ═══════════════════════════════════════
local origNC
local function startSilent()
    if not hookmetamethod or not getrawmetatable then return false end
    local mt = getrawmetatable(game)
    origNC = mt.__namecall
    setreadonly(mt, false)
    local keys = {"attack","hit","damage","punch","swing","shoot","fire","cast",
                  "melee","strike","projectile","bullet","slash","combat","stab",
                  "arrow","shot","weapon"}
    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        if _G.LEMO.Silent and method == "FireServer" then
            local n = tostring(self.Name):lower()
            for _, k in ipairs(keys) do
                if n:find(k) then
                    if math.random(1, 100) <= _G.LEMO.SilentChance then
                        local _, part = getTarget(_G.LEMO.SilentFOV, _G.LEMO.AimPart, true, false)
                        if part then
                            local args = {...}
                            local aimPos = part.Position
                            if _G.LEMO.AimPredict then
                                local vel = part.AssemblyLinearVelocity or part.Velocity
                                if vel then
                                    local ping = LP:GetNetworkPing()
                                    aimPos = aimPos + vel * math.clamp(ping * 1.5, 0.05, 0.4)
                                end
                            end
                            for i, v in ipairs(args) do
                                if typeof(v) == "CFrame" then
                                    args[i] = CFrame.new(Camera.CFrame.Position, aimPos); break
                                elseif typeof(v) == "Vector3" then
                                    args[i] = aimPos
                                end
                            end
                            return origNC(self, unpack(args))
                        end
                    end
                    break
                end
            end
        end
        return origNC(self, ...)
    end)
    setreadonly(mt, true)
    return true
end

local function stopSilent()
    if origNC and getrawmetatable then
        local mt = getrawmetatable(game)
        setreadonly(mt, false); mt.__namecall = origNC; setreadonly(mt, true); origNC = nil
    end
end

-- ═══════════════════════════════════════
-- TRIGGER BOT v2 — ENHANCED
-- ═══════════════════════════════════════
local lastTriggerFire = 0
local triggerLockedOn = nil
local triggerBurstActive = false
local triggerFiredIndicator = 0

local function humanizeDelay()
    if not _G.LEMO.TriggerHumanize then
        return _G.LEMO.TriggerDelay
    end
    local minD = _G.LEMO.TriggerMinDelay
    local maxD = _G.LEMO.TriggerMaxDelay
    return minD + math.random() * (maxD - minD)
end

local function isCameraOnTarget(part)
    if not _G.LEMO.TriggerAimRequired then return true end
    local _, _, myHrp = getChar()
    if not myHrp then return false end
    local dirToTarget = (part.Position - Camera.CFrame.Position).Unit
    local cameraDir = Camera.CFrame.LookVector
    local dot = math.clamp(cameraDir:Dot(dirToTarget), -1, 1)
    local angle = math.deg(math.acos(dot))
    return angle <= 8
end

local function fireOnce()
    pcall(function()
        VIM:SendMouseButtonEvent(0, 0, 0, true, game, 0)
        task.wait(0.012)
        VIM:SendMouseButtonEvent(0, 0, 0, false, game, 0)
    end)
end

local function fireBurst()
    triggerBurstActive = true
    for i = 1, _G.LEMO.TriggerBurst do
        if not _G.LEMO.Trigger then break end
        fireOnce()
        if i < _G.LEMO.TriggerBurst then
            task.wait(_G.LEMO.TriggerBurstDelay)
        end
    end
    triggerBurstActive = false
end

_G.LEMO.Loops.Trigger = RunService.RenderStepped:Connect(function()
    if not _G.LEMO.Trigger then
        triggerLockedOn = nil
        return
    end
    if triggerBurstActive then return end
    local now = tick()
    if now - lastTriggerFire < humanizeDelay() then return end

    local target, part = getTarget(
        _G.LEMO.TriggerFOV, _G.LEMO.AimPart,
        _G.LEMO.TriggerTeam, _G.LEMO.TriggerWallCheck
    )
    if not target or not part then triggerLockedOn = nil; return end

    local _, _, myHrp = getChar()
    if myHrp then
        local dist = (myHrp.Position - part.Position).Magnitude
        if dist > _G.LEMO.TriggerMaxDist then triggerLockedOn = nil; return end
    end

    local targetHum = target.Character and target.Character:FindFirstChildOfClass("Humanoid")
    if targetHum and targetHum.Health < _G.LEMO.TriggerHealthMin then
        triggerLockedOn = nil; return
    end

    if not isCameraOnTarget(part) then triggerLockedOn = nil; return end

    triggerLockedOn = target
    triggerFiredIndicator = now
    lastTriggerFire = now

    if _G.LEMO.TriggerMode == "Burst" then
        fireBurst()
    elseif _G.LEMO.TriggerMode == "Spam" then
        task.spawn(function()
            for i = 1, 3 do
                if not _G.LEMO.Trigger then break end
                fireOnce()
                task.wait(0.02)
            end
        end)
    else
        fireOnce()
    end
end)

-- Trigger visual indicator
local triggerInd = Drawing.new("Circle")
triggerInd.Thickness = 2
triggerInd.NumSides = 24
triggerInd.Filled = false
triggerInd.Color = Color3.fromRGB(255, 50, 50)
triggerInd.Transparency = 0
triggerInd.Radius = 12

_G.LEMO.Loops.TriggerIndicator = RunService.RenderStepped:Connect(function()
    if not _G.LEMO.Trigger or not _G.LEMO.TriggerIndicator then
        triggerInd.Visible = false; return
    end
    local _, _, myHrp = getChar()
    if not myHrp then triggerInd.Visible = false; return end
    if tick() - triggerFiredIndicator < 0.15 then
        triggerInd.Visible = true
        triggerInd.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
        triggerInd.Radius = 14
        triggerInd.Color = Color3.fromRGB(255, 40, 40)
    elseif triggerLockedOn then
        local part = getPart(triggerLockedOn, _G.LEMO.AimPart)
        if part then
            local vp, onScreen = Camera:WorldToViewportPoint(part.Position)
            if onScreen then
                triggerInd.Visible = true
                triggerInd.Position = Vector2.new(vp.X, vp.Y)
                triggerInd.Radius = 10
                triggerInd.Color = Color3.fromRGB(220, 175, 90)
            else triggerInd.Visible = false end
        else triggerInd.Visible = false end
    else
        triggerInd.Visible = false
    end
end)

-- ═══════════════════════════════════════
-- NO RECOIL
-- ═══════════════════════════════════════
local lastCameraCF = nil
_G.LEMO.Loops.NoRecoil = RunService.RenderStepped:Connect(function()
    if not _G.LEMO.NoRecoil then lastCameraCF = nil; return end
    local curCF = Camera.CFrame
    if lastCameraCF then
        local rotDelta = (curCF.LookVector - lastCameraCF.LookVector)
        if rotDelta.Magnitude > 0.001 then
            local strength = _G.LEMO.NoRecoilStrength
            local compensation = rotDelta * (strength * 0.5)
            local newLook = curCF.LookVector - compensation
            pcall(function()
                Camera.CFrame = CFrame.lookAt(curCF.Position, curCF.Position + newLook)
            end)
        end
    end
    lastCameraCF = Camera.CFrame
end)

-- ═══════════════════════════════════════
-- NO SPREAD
-- ═══════════════════════════════════════
local origSpreadNC = nil
local function startNoSpread()
    if not hookmetamethod or not getrawmetatable then return false end
    local mt = getrawmetatable(game)
    origSpreadNC = mt.__namecall
    setreadonly(mt, false)
    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        if _G.LEMO.NoSpread and method == "FireServer" then
            local n = tostring(self.Name):lower()
            if n:find("spread") or n:find("recoil") or n:find("accuracy") then
                return nil
            end
        end
        return origSpreadNC(self, ...)
    end)
    setreadonly(mt, true)
    return true
end
local function stopNoSpread()
    if origSpreadNC and getrawmetatable then
        local mt = getrawmetatable(game)
        setreadonly(mt, false); mt.__namecall = origSpreadNC; setreadonly(mt, true); origSpreadNC = nil
    end
end

-- ═══════════════════════════════════════
-- ANTI-FLING
-- ═══════════════════════════════════════
_G.LEMO.Loops.AntiFling = RunService.Heartbeat:Connect(function()
    if not _G.LEMO.AntiFling then return end
    local _, hum, hrp = getChar()
    if not hum or not hrp then return end
    local v = hrp.Velocity
    if v.Magnitude > 500 then
        pcall(function() hrp.Velocity = Vector3.new(0, v.Y, 0) end)
    end
    local state = hum:GetState()
    if state == Enum.HumanoidStateType.FallingDown or state == Enum.HumanoidStateType.Ragdoll then
        pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
    end
end)

-- ═══════════════════════════════════════
-- SKELETON BONES
-- ═══════════════════════════════════════
local R15_BONES = {
    {"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"},
    {"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"}, {"LeftLowerArm", "LeftHand"},
    {"UpperTorso", "RightUpperArm"}, {"RightUpperArm", "RightLowerArm"}, {"RightLowerArm", "RightHand"},
    {"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"}, {"LeftLowerLeg", "LeftFoot"},
    {"LowerTorso", "RightUpperLeg"}, {"RightUpperLeg", "RightLowerLeg"}, {"RightLowerLeg", "RightFoot"}
}
local R6_BONES = {
    {"Head", "Torso"}, {"Torso", "Left Arm"}, {"Torso", "Right Arm"},
    {"Torso", "Left Leg"}, {"Torso", "Right Leg"}
}

-- ═══════════════════════════════════════
-- ESP
-- ═══════════════════════════════════════
_G.LEMO.Loops.ESP = RunService.RenderStepped:Connect(function()
    if not _G.LEMO.ESP then return end
    local _, _, myHrp = getChar()
    if not myHrp then return end
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr == LP then continue end
        local function cleanup()
            if _G.LEMO.Cache[plr] then
                for _, d in pairs(_G.LEMO.Cache[plr]) do
                    if d and d.Remove then pcall(function() d:Remove() end) end
                end
                _G.LEMO.Cache[plr] = nil
            end
            if _G.LEMO.SkelCache[plr] then
                for _, d in pairs(_G.LEMO.SkelCache[plr]) do
                    if d and d.Remove then pcall(function() d:Remove() end) end
                end
                _G.LEMO.SkelCache[plr] = nil
            end
        end
        if _G.LEMO.ESPTeam and isTeam(plr) then cleanup(); continue end
        local char = plr.Character
        if not char or not char.Parent then cleanup(); continue end
        local hum = char:FindFirstChildOfClass("Humanoid")
        local hrp = char:FindFirstChild("HumanoidRootPart")
        local head = char:FindFirstChild("Head")
        if not hum or not hrp or not head or hum.Health <= 0 then cleanup(); continue end
        local d = (myHrp.Position - hrp.Position).Magnitude
        if d > _G.LEMO.ESPMaxDist then cleanup(); continue end
        if not _G.LEMO.Cache[plr] then
            local c = {}
            c.box = Drawing.new("Square"); c.box.Thickness = 1.5; c.box.Filled = false; c.box.Transparency = 1
            c.boxFill = Drawing.new("Square"); c.boxFill.Filled = true; c.boxFill.Transparency = 0.92
            c.name = Drawing.new("Text"); c.name.Size = 13; c.name.Center = true; c.name.Outline = true
            c.hp = Drawing.new("Text"); c.hp.Size = 11; c.hp.Center = true; c.hp.Outline = true
            c.hpBarBg = Drawing.new("Square"); c.hpBarBg.Filled = true; c.hpBarBg.Color = Color3.fromRGB(20, 5, 8)
            c.hpBar = Drawing.new("Square"); c.hpBar.Filled = true
            c.dist = Drawing.new("Text"); c.dist.Size = 11; c.dist.Center = true; c.dist.Outline = true
            c.tracer = Drawing.new("Line"); c.tracer.Thickness = 1.2
            _G.LEMO.Cache[plr] = c
        end
        local c = _G.LEMO.Cache[plr]
        local vp, onScreen = Camera:WorldToViewportPoint(hrp.Position)
        if not onScr
