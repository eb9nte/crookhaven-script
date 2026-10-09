-- SM4KOVO4KA R3AL1SE
-- ULTIMATE EDITION v12.0 + RTX, QuickStop, Dynamic InfoHUD, Keybind List & Particles
-- Features: Aimbot, ESP 2D, Anti-Aim, Air Strafe, QuickStop, Click TP, Target TP, 5 Trails, 5 Particles, RTX v2 Lighting, Forcefield Chams, Keybind List, Custom InfoHUD Animation

-- ===== SERVICES =====
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local VirtualInputManager = game:GetService("VirtualInputManager")
local HttpService = game:GetService("HttpService")

-- ===== CONFIGURATION =====
local Config = {
    Title = "SM4KOVO4KA R3AL1SE",
    Subtitle = "v12.0",
    MenuColor = Color3.fromRGB(15, 15, 15),
    AccentColor = Color3.fromRGB(31, 133, 255),
    TextColor = Color3.fromRGB(255, 255, 255),
    HudColor = Color3.fromRGB(31, 133, 255),
    ToggleKey = Enum.KeyCode.P,
    RainbowSpeed = 0.005,
    MenuMusicId = "rbxassetid://138118304933431",
    MenuMusicVolume = 0.2,
    BaseWidth = 670,
    BaseHeight = 560
}

-- ===== MODULES STATE =====
local Modules = {
    Aimbot = {
        Enabled = false,
        Smoothness = 0.3,
        FOV = 120,
        ShowFOV = false,
        TeamCheck = true,
        VisibleCheck = true,
        HitPart = "Head"
    },
    ESP = {
        Enabled = false,
        Boxes = true,
        Names = true,
        Health = true,
        Distance = true
    },
    Movement = {
        Speed = 16,
        JumpPower = 50,
        Noclip = false,
        SpeedGlitch = false,
        SpeedGlitchMultiplier = 2.0,
        SpeedGlitchBoost = 30,
        Strafe = false,
        StrafeSpeed = 50,
        TurboStrafe = false,
        TurboStrafeSpeed = 250,
        StrafeHeight = 2,
        AirStrafe = false,
        AirStrafeControl = 50,
        QuickStop = false,
        QuickStopFriction = 0.8
    },
    AntiAim = {
        Enabled = false,
        Mode = "Static", -- Static, Spinbot, Jitter
        YawOffset = 180,
        SpinSpeed = 20,
        JitterRange = 90
    },
    Combat = {
        AutoClick = false,
        ClickSpeed = 10,
        AutoClickR = false,
        ClickRInterval = 100
    },
    Fly = {
        Enabled = false,
        Speed = 50,
        TPose = false
    },
    Visuals = {
        Fullbright = false,
        FOV = 70,
        Skybox = "Red Sky 1",
        AntiFling = false,
        Trails = false,
        TrailType = "Basic", -- Basic, Rainbow, Pulse, Electric, Fade
        TrailColor = Color3.fromRGB(31, 133, 255),
        TrailWidth = 0.5,
        TrailLength = 2.0,
        Particles = false,
        ParticleType = "Sparkles", -- Sparkles, Fire, Smoke, Stars, Hearts
        ParticleColor = Color3.fromRGB(31, 133, 255),
        ParticleSize = 1.0,
        TargetHUD = false,
        Fog = {
            Enabled = false,
            Color = Color3.fromRGB(200, 0, 0),
            Start = 0,
            End = 100
        },
        RTX = {
            Shadows = false,
            ColorCorrection = false,
            Bloom = false,
            SunRays = false,
            DOF = false
        }
    },
    Chams = {
        Enabled = false,
        Color = Color3.fromRGB(31, 133, 255),
        Transparency = 0.5,
        Material = "ForceField",
        Mode = "Fill"
    },
    Teleport = {
        ClickTP = false,
        SelectedTarget = nil
    }
}

-- ===== KEYBINDS SYSTEM =====
local Keybinds = {
    AimbotToggle = nil,
    NoclipToggle = nil,
    FlyToggle = nil,
    TpUpAction = nil,
    AntiFlingToggle = nil,
    TrailsToggle = nil,
    ClickTPKey = Enum.KeyCode.Z,
    TargetSelectKey = Enum.KeyCode.L,
    TargetTPKey = Enum.KeyCode.K,
    QuickStopKey = Enum.KeyCode.X
}

-- ===== SKYBOX DATABASE =====
local Skyboxes = {
    ["Default"] = "",
    ["Red Sky 1"] = "rbxassetid://166576485",
    ["Scary Red Sky"] = "rbxassetid://136055162054954",
    ["Blood Red"] = "rbxassetid://448745249",
    ["Red Sunset"] = "rbxassetid://130979345",
    ["Red Nebula"] = "rbxassetid://159454299",
    ["Red Galaxy"] = "rbxassetid://5752460"
}

-- ===== GLOBAL VARIABLES =====
local ScreenGui, MainFrame, UIScale
local isMenuVisible = true -- Открывается сразу при инжекте!
local currentTab = "AIMBOT"
local Tabs = {}
local chamsInstances = {}
local flyConnection, autoClickRThread, targetHUDConnection, rainbowConnection
local currentTarget = nil
local fovCircle = nil
local rainbowEnabled = false
local savedAccentColor = Config.AccentColor
local menuSound = nil
local espDrawings = {}

-- InfoHUD & RTX Effect variables
local infoTitleLabel, fpsLabel, pingLabel
local lastFPSTime = tick()
local frameCount = 0
local rtxColorCorr, rtxBloom, rtxSunRays, rtxDOF
local keybindListFrame

-- UI Components Storage
local UIComponents = {
    strokes = {},
    scrollbars = {},
    toggleFills = {},
    sliderFills = {},
    titleLabels = {}
}

-- ===== FORWARD DECLARATIONS =====
local CreateTargetHUD, ToggleMenu, UpdateTrails, UpdateParticles, UpdateChams, UpdateKeybindListUI

-- ===== UTILITY FUNCTIONS =====
local function Notify(msg)
    print("[SM4KOVO4KA] " .. msg)
end

local function GetCharacter(player)
    return player and player.Character
end

local function GetHumanoid(player)
    local char = GetCharacter(player)
    return char and char:FindFirstChildOfClass("Humanoid")
end

local function GetRootPart(player)
    local char = GetCharacter(player)
    return char and char:FindFirstChild("HumanoidRootPart")
end

-- ===== LIVE COLOR UPDATE =====
local function UpdateAccentColor(newColor)
    Config.AccentColor = newColor
    for _, stroke in ipairs(UIComponents.strokes) do
        if stroke and stroke.Parent then stroke.Color = newColor end
    end
    for _, scroll in ipairs(UIComponents.scrollbars) do
        if scroll and scroll.Parent then scroll.ScrollBarImageColor3 = newColor end
    end
    for _, label in ipairs(UIComponents.titleLabels) do
        if label and label.Parent then label.TextColor3 = newColor end
    end
    for _, func in ipairs(UIComponents.toggleFills) do func(newColor) end
    for _, func in ipairs(UIComponents.sliderFills) do func(newColor) end
    if fovCircle then
        fovCircle.Color = newColor
    end
end

local function UpdateHudColor(newColor)
    Config.HudColor = newColor
    if Modules.Visuals.TargetHUD and CreateTargetHUD then
        CreateTargetHUD()
    end
end

local function UpdateMenuColor(newColor)
    Config.MenuColor = newColor
    if MainFrame then MainFrame.BackgroundColor3 = newColor end
end

-- ===== FIXED RGB ACCENT =====
local function StartRainbow()
    if rainbowConnection then rainbowConnection:Disconnect() end
    if not rainbowEnabled then return end

    local hue = 0
    rainbowConnection = RunService.RenderStepped:Connect(function(dt)
        hue = (hue + dt * Config.RainbowSpeed * 10) % 1
        local color = Color3.fromHSV(hue, 1, 1)
        UpdateAccentColor(color)
    end)
end

local function StopRainbow()
    if rainbowConnection then
        rainbowConnection:Disconnect()
        rainbowConnection = nil
    end
    UpdateAccentColor(savedAccentColor)
end

local function ToggleRainbow(state)
    rainbowEnabled = state
    if state then
        savedAccentColor = Config.AccentColor
        StartRainbow()
    else
        StopRainbow()
    end
end

-- ===== SKYBOX =====
local function ChangeSkybox(skyboxName)
    local skyboxId = Skyboxes[skyboxName]
    if skyboxId and skyboxId ~= "" then
        local sky = Instance.new("Sky")
        sky.SkyboxBk = skyboxId
        sky.SkyboxDn = skyboxId
        sky.SkyboxFt = skyboxId
        sky.SkyboxLf = skyboxId
        sky.SkyboxRt = skyboxId
        sky.SkyboxUp = skyboxId
        sky.Parent = Lighting
        for _, child in pairs(Lighting:GetChildren()) do
            if child:IsA("Sky") and child ~= sky then
                child:Destroy()
            end
        end
    else
        for _, child in pairs(Lighting:GetChildren()) do
            if child:IsA("Sky") then
                child:Destroy()
            end
        end
    end
    Modules.Visuals.Skybox = skyboxName
end

-- ===== ANTI FLING SYSTEM =====
local function processAntiFlingPlayer(player)
    if not Modules.Visuals.AntiFling then return end
    if player == LocalPlayer then return end
    
    task.spawn(function()
        if not player.Character then player.CharacterAdded:Wait() end
        local char = player.Character
        if not char then return end
        
        for _, child in pairs(char:GetChildren()) do
            if child:IsA("BasePart") then
                child.CanCollide = false
                if child.Name == "Torso" or child.Name == "UpperTorso" or child.Name == "LowerTorso" then
                    child.Massless = true
                end
                local vector = Vector3.new()
                child.Velocity = vector
                child.RotVelocity = vector
            end
        end
    end)
end

local function ApplyAntiFling()
    if not Modules.Visuals.AntiFling then return end
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            processAntiFlingPlayer(player)
        end
    end
end

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if Modules.Visuals.AntiFling then
            processAntiFlingPlayer(player)
        end
    end)
end)

-- ===== AIMBOT =====
local function IsVisible(targetPart)
    if not Modules.Aimbot.VisibleCheck then return true end
    local camera = Workspace.CurrentCamera
    local origin = camera.CFrame.Position
    local direction = (targetPart.Position - origin)
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = RaycastFilterType.Exclude
    local ignoreList = {camera}
    if LocalPlayer.Character then table.insert(ignoreList, LocalPlayer.Character) end
    raycastParams.FilterDescendantsInstances = ignoreList
    
    local result = Workspace:Raycast(origin, direction, raycastParams)
    return result == nil or result.Instance:IsDescendantOf(targetPart.Parent)
end

local function GetClosestTarget()
    local closest = nil
    local shortestDist = Modules.Aimbot.FOV
    local camera = Workspace.CurrentCamera
    local mousePos = UserInputService:GetMouseLocation()
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            if Modules.Aimbot.TeamCheck and player.Team and player.Team == LocalPlayer.Team then continue end
            local hum = player.Character:FindFirstChildOfClass("Humanoid")
            if not hum or hum.Health <= 0 then continue end
            
            local targetPart = player.Character:FindFirstChild(Modules.Aimbot.HitPart) or player.Character:FindFirstChild("HumanoidRootPart")
            if targetPart then
                local screenPos, onScreen = camera:WorldToViewportPoint(targetPart.Position)
                if onScreen then
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                    if dist < shortestDist then
                        if IsVisible(targetPart) then
                            closest = player
                            shortestDist = dist
                        end
                    end
                end
            end
        end
    end
    return closest
end

local function AimbotLoop()
    if not Modules.Aimbot.Enabled then 
        currentTarget = nil
        return 
    end
    local target = GetClosestTarget()
    if target and target.Character then
        currentTarget = target
        local part = target.Character:FindFirstChild(Modules.Aimbot.HitPart) or target.Character:FindFirstChild("HumanoidRootPart")
        if part then
            local camera = Workspace.CurrentCamera
            local targetPos = part.Position
            local currentCFrame = camera.CFrame
            local targetCFrame = CFrame.lookAt(currentCFrame.Position, targetPos)
            camera.CFrame = currentCFrame:Lerp(targetCFrame, math.clamp(1 - Modules.Aimbot.Smoothness, 0.05, 1))
        end
    else
        currentTarget = nil
    end
end

-- ===== DRAWING 2D ESP =====
local function RemoveESP(player)
    if espDrawings[player] then
        for _, obj in pairs(espDrawings[player]) do
            if obj.Remove then obj:Remove() end
        end
        espDrawings[player] = nil
    end
end

local function Update2DESP()
    if not Modules.ESP.Enabled then
        for player, _ in pairs(espDrawings) do
            RemoveESP(player)
        end
        return
    end

    local camera = Workspace.CurrentCamera
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            if not espDrawings[player] then
                espDrawings[player] = {
                    Box = Drawing.new("Square"),
                    BoxOutline = Drawing.new("Square"),
                    Name = Drawing.new("Text"),
                    HealthBar = Drawing.new("Square"),
                    HealthOutline = Drawing.new("Square"),
                    Distance = Drawing.new("Text")
                }
                
                local d = espDrawings[player]
                d.Box.Thickness = 1
                d.Box.Filled = false
                d.Box.Color = Color3.fromRGB(255, 255, 255)
                
                d.BoxOutline.Thickness = 3
                d.BoxOutline.Filled = false
                d.BoxOutline.Color = Color3.fromRGB(0, 0, 0)
                
                d.Name.Size = 14
                d.Name.Center = true
                d.Name.Outline = true
                d.Name.Color = Color3.fromRGB(255, 255, 255)
                
                d.HealthBar.Filled = true
                d.HealthBar.Color = Color3.fromRGB(0, 255, 0)
                
                d.HealthOutline.Filled = true
                d.HealthOutline.Color = Color3.fromRGB(0, 0, 0)
                
                d.Distance.Size = 12
                d.Distance.Center = true
                d.Distance.Outline = true
                d.Distance.Color = Color3.fromRGB(200, 200, 200)
            end

            local d = espDrawings[player]
            local char = player.Character
            local root = char and char:FindFirstChild("HumanoidRootPart")
            local hum = char and char:FindFirstChildOfClass("Humanoid")

            if char and root and hum and hum.Health > 0 then
                local rootPos, onScreen = camera:WorldToViewportPoint(root.Position)
                local head = char:FindFirstChild("Head")
                local headPos = head and camera:WorldToViewportPoint(head.Position + Vector3.new(0, 0.5, 0)) or camera:WorldToViewportPoint(root.Position + Vector3.new(0, 3, 0))
                local legPos = camera:WorldToViewportPoint(root.Position - Vector3.new(0, 3, 0))

                if onScreen then
                    local height = math.abs(headPos.Y - legPos.Y)
                    local width = height / 2
                    local boxPos = Vector2.new(rootPos.X - width / 2, headPos.Y)

                    -- Box
                    if Modules.ESP.Boxes then
                        d.BoxOutline.Size = Vector2.new(width, height)
                        d.BoxOutline.Position = boxPos
                        d.BoxOutline.Visible = true

                        d.Box.Size = Vector2.new(width, height)
                        d.Box.Position = boxPos
                        d.Box.Visible = true
                    else
                        d.Box.Visible = false
                        d.BoxOutline.Visible = false
                    end

                    -- Name
                    if Modules.ESP.Names then
                        d.Name.Text = player.Name
                        d.Name.Position = Vector2.new(rootPos.X, boxPos.Y - 16)
                        d.Name.Visible = true
                    else
                        d.Name.Visible = false
                    end

                    -- Health
                    if Modules.ESP.Health then
                        local healthPerc = math.clamp(hum.Health / hum.MaxHealth, 0, 1)
                        local barHeight = height * healthPerc
                        
                        d.HealthOutline.Size = Vector2.new(4, height + 2)
                        d.HealthOutline.Position = Vector2.new(boxPos.X - 6, boxPos.Y - 1)
                        d.HealthOutline.Visible = true

                        d.HealthBar.Size = Vector2.new(2, barHeight)
                        d.HealthBar.Position = Vector2.new(boxPos.X - 5, boxPos.Y + (height - barHeight))
                        d.HealthBar.Color = Color3.fromRGB(255 * (1 - healthPerc), 255 * healthPerc, 0)
                        d.HealthBar.Visible = true
                    else
                        d.HealthBar.Visible = false
                        d.HealthOutline.Visible = false
                    end

                    -- Distance
                    if Modules.ESP.Distance then
                        local dist = (camera.CFrame.Position - root.Position).Magnitude
                        d.Distance.Text = math.floor(dist) .. "m"
                        d.Distance.Position = Vector2.new(rootPos.X, boxPos.Y + height + 2)
                        d.Distance.Visible = true
                    else
                        d.Distance.Visible = false
                    end
                else
                    for _, obj in pairs(d) do obj.Visible = false end
                end
            else
                for _, obj in pairs(d) do obj.Visible = false end
            end
        end
    end
end

Players.PlayerRemoving:Connect(RemoveESP)

-- ===== ANTI-AIM =====
local currentSpinYaw = 0
local function UpdateAntiAim()
    if not Modules.AntiAim.Enabled then return end
    local char = LocalPlayer.Character
    local root = GetRootPart(LocalPlayer)
    if not char or not root then return end

    local mode = Modules.AntiAim.Mode
    if mode == "Static" then
        root.CFrame = CFrame.new(root.Position) * CFrame.Angles(0, math.rad(Modules.AntiAim.YawOffset), 0)
    elseif mode == "Spinbot" then
        currentSpinYaw = (currentSpinYaw + Modules.AntiAim.SpinSpeed) % 360
        root.CFrame = CFrame.new(root.Position) * CFrame.Angles(0, math.rad(currentSpinYaw), 0)
    elseif mode == "Jitter" then
        local jitter = math.random(-Modules.AntiAim.JitterRange, Modules.AntiAim.JitterRange)
        root.CFrame = CFrame.new(root.Position) * CFrame.Angles(0, math.rad(Modules.AntiAim.YawOffset + jitter), 0)
    end
end

-- ===== AIR STRAFE & QUICKSTOP =====
local function UpdateAirStrafeAndQuickstop()
    local char = LocalPlayer.Character
    local root = GetRootPart(LocalPlayer)
    local hum = GetHumanoid(LocalPlayer)
    if not char or not root or not hum then return end

    -- QuickStop logic
    if Modules.Movement.QuickStop then
        local w = UserInputService:IsKeyDown(Enum.KeyCode.W)
        local a = UserInputService:IsKeyDown(Enum.KeyCode.A)
        local s = UserInputService:IsKeyDown(Enum.KeyCode.S)
        local d = UserInputService:IsKeyDown(Enum.KeyCode.D)
        
        if not (w or a or s or d) then
            root.AssemblyLinearVelocity = Vector3.new(
                root.AssemblyLinearVelocity.X * (1 - Modules.Movement.QuickStopFriction),
                root.AssemblyLinearVelocity.Y,
                root.AssemblyLinearVelocity.Z * (1 - Modules.Movement.QuickStopFriction)
            )
        end
    end

    -- Air Strafe logic
    if Modules.Movement.AirStrafe then
        local state = hum:GetState()
        if state == Enum.HumanoidStateType.Freefall or state == Enum.HumanoidStateType.Jumping then
            local moveDir = Vector3.new()
            local camCF = Workspace.CurrentCamera.CFrame
            
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir += camCF.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir -= camCF.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir -= camCF.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir += camCF.RightVector end
            
            moveDir = Vector3.new(moveDir.X, 0, moveDir.Z)
            if moveDir.Magnitude > 0 then
                moveDir = moveDir.Unit
                local currentVel = root.AssemblyLinearVelocity
                local targetVel = moveDir * Modules.Movement.AirStrafeControl
                root.AssemblyLinearVelocity = Vector3.new(targetVel.X, currentVel.Y, targetVel.Z)
            end
        end
    end
end

-- ===== FOV CIRCLE =====
local function UpdateFOVCircle()
    if not Modules.Aimbot.ShowFOV then
        if fovCircle then
            fovCircle.Visible = false
        end
        return
    end

    if not fovCircle then
        fovCircle = Drawing.new("Circle")
        fovCircle.Thickness = 2
        fovCircle.NumSides = 64
        fovCircle.Filled = false
        fovCircle.Transparency = 1
    end

    fovCircle.Radius = Modules.Aimbot.FOV
    fovCircle.Color = Config.AccentColor
    fovCircle.Position = UserInputService:GetMouseLocation()
    fovCircle.Visible = true
end

-- ===== FLY IN T-POSE =====
local function StopFly()
    if flyConnection then
        flyConnection:Disconnect()
        flyConnection = nil
    end
    local char = LocalPlayer.Character
    if char then
        local root = char:FindFirstChild("HumanoidRootPart")
        if root then
            if root:FindFirstChild("FlyBV") then root.FlyBV:Destroy() end
            if root:FindFirstChild("FlyBG") then root.FlyBG:Destroy() end
        end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.PlatformStand = false end
        local animate = char:FindFirstChild("Animate")
        if animate then animate.Disabled = false end
    end
end

local function StartFly()
    if not Modules.Fly.Enabled then
        StopFly()
        return
    end
    local char = LocalPlayer.Character
    if not char then return end
    local root = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not root or not hum then return end

    hum.PlatformStand = true

    if Modules.Fly.TPose then
        for _, track in pairs(hum:GetPlayingAnimationTracks()) do
            track:Stop()
        end
        local animate = char:FindFirstChild("Animate")
        if animate then animate.Disabled = true end
    end

    local bv = root:FindFirstChild("FlyBV") or Instance.new("BodyVelocity")
    bv.Name = "FlyBV"
    bv.MaxForce = Vector3.new(1e5, 1e5, 1e5)
    bv.Velocity = Vector3.zero
    bv.Parent = root

    local bg = root:FindFirstChild("FlyBG") or Instance.new("BodyGyro")
    bg.Name = "FlyBG"
    bg.MaxTorque = Vector3.new(1e5, 1e5, 1e5)
    bg.P = 10000
    bg.CFrame = root.CFrame
    bg.Parent = root

    if flyConnection then flyConnection:Disconnect() end
    flyConnection = RunService.Heartbeat:Connect(function()
        if not Modules.Fly.Enabled or not root or not root.Parent then
            StopFly()
            return
        end
        local cam = Workspace.CurrentCamera
        local moveDir = Vector3.new()
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir += cam.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir -= cam.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir -= cam.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir += cam.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDir += Vector3.new(0,1,0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then moveDir += Vector3.new(0,-1,0) end

        bv.Velocity = moveDir.Magnitude > 0 and (moveDir.Unit * Modules.Fly.Speed) or Vector3.zero
        bg.CFrame = cam.CFrame
    end)
end

-- ===== TELEPORTS (CLICK TP & TARGET TP) =====
local function PerformClickTP()
    local mouse = LocalPlayer:GetMouse()
    local root = GetRootPart(LocalPlayer)
    if mouse and root and mouse.Hit then
        root.CFrame = CFrame.new(mouse.Hit.Position + Vector3.new(0, 3, 0))
        Notify("Click TP performed!")
    end
end

local function SelectTargetUnderMouse()
    local mouse = LocalPlayer:GetMouse()
    if mouse and mouse.Target then
        local model = mouse.Target:FindFirstAncestorOfClass("Model")
        if model then
            local p = Players:GetPlayerFromCharacter(model)
            if p and p ~= LocalPlayer then
                Modules.Teleport.SelectedTarget = p
                Notify("Target Selected: " .. p.Name)
                return
            end
        end
    end
    Notify("No valid target under mouse cursor!")
end

local function PerformTargetTP()
    local target = Modules.Teleport.SelectedTarget
    local root = GetRootPart(LocalPlayer)
    if target and target.Character then
        local targetRoot = GetRootPart(target)
        if targetRoot and root then
            root.CFrame = targetRoot.CFrame + Vector3.new(0, 2, 2)
            Notify("Teleported to " .. target.Name)
            return
        end
    end
    Notify("No target selected or target invalid!")
end

local function TpUp()
    local root = GetRootPart(LocalPlayer)
    if root then
        root.CFrame = root.CFrame + Vector3.new(0, 200, 0)
        Notify("Teleported up 200 studs")
    end
end

-- ===== TARGET HUD =====
CreateTargetHUD = function()
    if targetHUDConnection then targetHUDConnection:Disconnect() end
    if not Modules.Visuals.TargetHUD then
        if ScreenGui and ScreenGui:FindFirstChild("TargetHUD") then
            ScreenGui.TargetHUD:Destroy()
        end
        return
    end

    local hud = ScreenGui and ScreenGui:FindFirstChild("TargetHUD")
    if not hud then
        hud = Instance.new("Frame")
        hud.Name = "TargetHUD"
        hud.Size = UDim2.new(0, 220, 0, 70)
        hud.Position = UDim2.new(0.5, -110, 0, 15)
        hud.BackgroundColor3 = Config.MenuColor
        hud.BorderSizePixel = 0
        hud.BackgroundTransparency = 0.1
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 4)
        corner.Parent = hud
        local stroke = Instance.new("UIStroke")
        stroke.Color = Config.HudColor
        stroke.Thickness = 1
        stroke.Parent = hud
        table.insert(UIComponents.strokes, stroke)

        local nameLabel = Instance.new("TextLabel")
        nameLabel.Name = "NameLabel"
        nameLabel.Size = UDim2.new(1, -10, 0, 22)
        nameLabel.Position = UDim2.new(0, 5, 0, 5)
        nameLabel.BackgroundTransparency = 1
        nameLabel.TextColor3 = Config.TextColor
        nameLabel.Text = "Target: None"
        nameLabel.TextXAlignment = Enum.TextXAlignment.Left
        nameLabel.Font = Enum.Font.GothamSemibold
        nameLabel.TextSize = 14
        nameLabel.Parent = hud

        local healthBar = Instance.new("Frame")
        healthBar.Name = "HealthBar"
        healthBar.Size = UDim2.new(1, -10, 0, 10)
        healthBar.Position = UDim2.new(0, 5, 0, 30)
        healthBar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
        healthBar.BorderSizePixel = 0
        local barCorner = Instance.new("UICorner")
        barCorner.CornerRadius = UDim.new(0, 2)
        barCorner.Parent = healthBar
        healthBar.Parent = hud

        local fill = Instance.new("Frame")
        fill.Name = "Fill"
        fill.Size = UDim2.new(1, 0, 1, 0)
        fill.BackgroundColor3 = Config.HudColor
        fill.BorderSizePixel = 0
        local fillCorner = Instance.new("UICorner")
        fillCorner.CornerRadius = UDim.new(0, 2)
        fillCorner.Parent = fill
        fill.Parent = healthBar

        local healthLabel = Instance.new("TextLabel")
        healthLabel.Name = "HealthLabel"
        healthLabel.Size = UDim2.new(1, -10, 0, 18)
        healthLabel.Position = UDim2.new(0, 5, 0, 45)
        healthLabel.BackgroundTransparency = 1
        healthLabel.TextColor3 = Config.TextColor
        healthLabel.Text = "Health: 100/100"
        healthLabel.TextXAlignment = Enum.TextXAlignment.Left
        healthLabel.Font = Enum.Font.Gotham
        healthLabel.TextSize = 12
        healthLabel.Parent = hud

        hud.Parent = ScreenGui
    end

    targetHUDConnection = RunService.Heartbeat:Connect(function()
        if not Modules.Visuals.TargetHUD then
            if hud then hud:Destroy() end
            targetHUDConnection:Disconnect()
            return
        end
        if hud and hud.Parent then
            local target = currentTarget or Modules.Teleport.SelectedTarget
            if target and target.Character and target.Character:FindFirstChildOfClass("Humanoid") then
                local hum = target.Character:FindFirstChildOfClass("Humanoid")
                local name = target.Name
                local health = math.max(0, hum.Health)
                local maxHealth = hum.MaxHealth
                local perc = math.clamp(health / maxHealth, 0, 1)
                hud.NameLabel.Text = "Target: " .. name
                hud.HealthBar.Fill.Size = UDim2.new(perc, 0, 1, 0)
                hud.HealthLabel.Text = string.format("Health: %.0f/%.0f", health, maxHealth)
                hud.Visible = true
            else
                hud.Visible = false
            end
        end
    end)
end

-- ===== CHAMS (FORCEFIELD & GENERAL DYNAMIC UPDATE) =====
UpdateChams = function()
    for player, highlight in pairs(chamsInstances) do
        if highlight and highlight.Parent then highlight:Destroy() end
    end
    table.clear(chamsInstances)

    if not Modules.Chams.Enabled then return end

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            if Modules.Chams.Material == "ForceField" then
                for _, part in pairs(player.Character:GetChildren()) do
                    if part:IsA("BasePart") then
                        local ff = part:FindFirstChild("RageFF") or Instance.new("SpecialMesh")
                        ff.Name = "RageFF"
                        ff.MeshType = Enum.MeshType.FileMesh
                        ff.TextureId = "rbxassetid://404924007" -- Forcefield Texture
                        ff.Parent = part
                    end
                end
            end

            local highlight = Instance.new("Highlight")
            highlight.Name = "RageChams"
            highlight.Adornee = player.Character
            highlight.FillColor = Modules.Chams.Color
            highlight.OutlineColor = Modules.Chams.Color
            highlight.FillTransparency = Modules.Chams.Transparency
            highlight.OutlineTransparency = Modules.Chams.Mode == "Outline" and 0 or 1
            highlight.Parent = player.Character
            chamsInstances[player] = highlight
        end
    end
end

-- ===== 5 TRAILS SYSTEM =====
UpdateTrails = function()
    local char = LocalPlayer.Character
    if not Modules.Visuals.Trails or not char then
        if char and char:FindFirstChild("HumanoidRootPart") then
            local t = char.HumanoidRootPart:FindFirstChild("RageTrail")
            if t then t:Destroy() end
            local a0 = char.HumanoidRootPart:FindFirstChild("TrailAtt0")
            if a0 then a0:Destroy() end
            local a1 = char.HumanoidRootPart:FindFirstChild("TrailAtt1")
            if a1 then a1:Destroy() end
        end
        return
    end
    
    local root = GetRootPart(LocalPlayer)
    if not root then return end
    
    local att0 = root:FindFirstChild("TrailAtt0") or Instance.new("Attachment")
    att0.Name = "TrailAtt0"
    att0.Position = Vector3.new(0, 1, 0)
    att0.Parent = root
    
    local att1 = root:FindFirstChild("TrailAtt1") or Instance.new("Attachment")
    att1.Name = "TrailAtt1"
    att1.Position = Vector3.new(0, -1, 0)
    att1.Parent = root

    local trail = root:FindFirstChild("RageTrail") or Instance.new("Trail")
    trail.Name = "RageTrail"
    trail.Attachment0 = att0
    trail.Attachment1 = att1
    trail.Lifetime = Modules.Visuals.TrailLength
    trail.WidthScale = NumberSequence.new(Modules.Visuals.TrailWidth)
    
    -- Trail Types (5 Types)
    local tType = Modules.Visuals.TrailType
    if tType == "Basic" then
        trail.Color = ColorSequence.new(Modules.Visuals.TrailColor)
        trail.Transparency = NumberSequence.new(0, 1)
        trail.Texture = ""
    elseif tType == "Rainbow" then
        trail.Color = ColorSequence.new({
            ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)),
            ColorSequenceKeypoint.new(0.2, Color3.fromRGB(255, 255, 0)),
            ColorSequenceKeypoint.new(0.4, Color3.fromRGB(0, 255, 0)),
            ColorSequenceKeypoint.new(0.6, Color3.fromRGB(0, 255, 255)),
            ColorSequenceKeypoint.new(0.8, Color3.fromRGB(0, 0, 255)),
            ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 0, 255))
        })
        trail.Transparency = NumberSequence.new(0, 1)
        trail.Texture = ""
    elseif tType == "Pulse" then
        trail.Color = ColorSequence.new(Modules.Visuals.TrailColor)
        trail.Transparency = NumberSequence.new({
            NumberSequenceKeypoint.new(0, 0),
            NumberSequenceKeypoint.new(0.5, 0.8),
            NumberSequenceKeypoint.new(1, 0)
        })
        trail.Texture = ""
    elseif tType == "Electric" then
        trail.Color = ColorSequence.new(Color3.fromRGB(0, 255, 255), Color3.fromRGB(255, 255, 255))
        trail.Transparency = NumberSequence.new(0.1, 1)
        trail.Texture = "rbxassetid://481232338"
    elseif tType == "Fade" then
        trail.Color = ColorSequence.new(Modules.Visuals.TrailColor, Color3.fromRGB(0,0,0))
        trail.Transparency = NumberSequence.new(0.2, 1)
        trail.Texture = ""
    end

    trail.Parent = root
end

-- ===== 5 PARTICLES SYSTEM =====
UpdateParticles = function()
    local root = GetRootPart(LocalPlayer)
    if not root then return end

    local existingPE = root:FindFirstChild("RageParticles")
    if existingPE then existingPE:Destroy() end

    if not Modules.Visuals.Particles then return end

    local pe = Instance.new("ParticleEmitter")
    pe.Name = "RageParticles"
    pe.Color = ColorSequence.new(Modules.Visuals.ParticleColor)
    pe.Size = NumberSequence.new(Modules.Visuals.ParticleSize)
    pe.Rate = 20
    pe.Speed = NumberRange.new(2, 5)

    local pType = Modules.Visuals.ParticleType
    if pType == "Sparkles" then
        pe.Texture = "rbxassetid://282200237"
    elseif pType == "Fire" then
        pe.Texture = "rbxassetid://258129206"
    elseif pType == "Smoke" then
        pe.Texture = "rbxassetid://258116742"
    elseif pType == "Stars" then
        pe.Texture = "rbxassetid://278783103"
    elseif pType == "Hearts" then
        pe.Texture = "rbxassetid://258117726"
    end

    pe.Parent = root
end

-- ===== RTX V2 LIGHTING SHADERS SYSTEM =====
local function ToggleRTXShadows(enabled)
    Lighting.GlobalShadows = enabled
    if enabled then
        Lighting.Ambient = Color3.fromRGB(50, 50, 50)
        Lighting.OutdoorAmbient = Color3.fromRGB(100, 100, 100)
    else
        Lighting.Ambient = Color3.fromRGB(128, 128, 128)
        Lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
    end
end

local function ToggleRTXColorCorrection(enabled)
    if enabled then
        if not rtxColorCorr then
            rtxColorCorr = Instance.new("ColorCorrectionEffect")
            rtxColorCorr.Brightness = 0.05
            rtxColorCorr.Contrast = 0.3
            rtxColorCorr.Saturation = 0.5
            rtxColorCorr.TintColor = Color3.fromRGB(235, 235, 235)
            rtxColorCorr.Name = "RTXv2ColorCorrection"
            rtxColorCorr.Parent = Lighting
        end
    else
        if rtxColorCorr then
            rtxColorCorr:Destroy()
            rtxColorCorr = nil
        end
    end
end

local function ToggleRTXBloom(enabled)
    if enabled then
        if not rtxBloom then
            rtxBloom = Instance.new("BloomEffect")
            rtxBloom.Intensity = 1.5
            rtxBloom.Size = 50
            rtxBloom.Threshold = 0.7
            rtxBloom.Name = "RTXv2Bloom"
            rtxBloom.Parent = Lighting
        end
    else
        if rtxBloom then
            rtxBloom:Destroy()
            rtxBloom = nil
        end
    end
end

local function ToggleRTXSunRays(enabled)
    if enabled then
        if not rtxSunRays then
            rtxSunRays = Instance.new("SunRaysEffect")
            rtxSunRays.Intensity = 0.2
            rtxSunRays.Spread = 0.3
            rtxSunRays.Name = "RTXv2SunRays"
            rtxSunRays.Parent = Lighting
        end
    else
        if rtxSunRays then
            rtxSunRays:Destroy()
            rtxSunRays = nil
        end
    end
end

local function ToggleRTXDOF(enabled)
    if enabled then
        if not rtxDOF then
            rtxDOF = Instance.new("DepthOfFieldEffect")
            rtxDOF.FocusDistance = 30
            rtxDOF.InFocusRadius = 15
            rtxDOF.NearIntensity = 0.3
            rtxDOF.FarIntensity = 0.5
            rtxDOF.Name = "RTXv2DOF"
            rtxDOF.Parent = Lighting
        end
    else
        if rtxDOF then
            rtxDOF:Destroy()
            rtxDOF = nil
        end
    end
end

-- ===== AUTO CLICK R =====
local function UpdateAutoClickR()
    if autoClickRThread then
        autoClickRThread:Cancel()
        autoClickRThread = nil
    end
    if not Modules.Combat.AutoClickR then return end

    autoClickRThread = task.spawn(function()
        while Modules.Combat.AutoClickR do
            pcall(function()
                VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.R, false, game)
                task.wait(0.02)
                VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.R, false, game)
            end)
            task.wait(Modules.Combat.ClickRInterval / 1000)
        end
    end)
end

-- ===== FOG =====
local function UpdateFog()
    if Modules.Visuals.Fog.Enabled then
        Lighting.FogColor = Modules.Visuals.Fog.Color
        Lighting.FogStart = Modules.Visuals.Fog.Start
        Lighting.FogEnd = Modules.Visuals.Fog.End
    else
        Lighting.FogColor = Color3.new(1, 1, 1)
        Lighting.FogStart = 0
        Lighting.FogEnd = 100000
    end
end

-- ===== WORKSPACE FILE CONFIG SYSTEM =====
local FolderName = "SM4KOVO4KA_Configs"

local function ColorToTable(c)
    return {R = c.R, G = c.G, B = c.B}
end

local function TableToColor(t)
    return Color3.new(t.R or 1, t.G or 1, t.B or 1)
end

local function EnsureFolderExists()
    if makefolder and isfolder then
        if not isfolder(FolderName) then
            makefolder(FolderName)
        end
    end
end

local function SaveConfigToFile(filename)
    EnsureFolderExists()
    local savedKeybinds = {}
    for name, code in pairs(Keybinds) do
        savedKeybinds[name] = code and code.Name or nil
    end
    local saveData = {
        Modules = Modules,
        Keybinds = savedKeybinds,
        Config = {
            MenuColor = ColorToTable(Config.MenuColor),
            AccentColor = ColorToTable(Config.AccentColor),
            TextColor = ColorToTable(Config.TextColor),
            HudColor = ColorToTable(Config.HudColor)
        }
    }
    local json = HttpService:JSONEncode(saveData)
    if writefile then
        writefile(FolderName .. "/" .. filename .. ".json", json)
        Notify("Config successfully saved to Workspace: " .. filename)
    else
        Notify("Executor does not support writefile!")
    end
end

local function LoadConfigFromFile(filename)
    EnsureFolderExists()
    local filePath = FolderName .. "/" .. filename .. ".json"
    if readfile and isfile and isfile(filePath) then
        local json = readfile(filePath)
        local data = HttpService:JSONDecode(json)
        if data then
            if data.Modules then Modules = data.Modules end
            if data.Keybinds then
                for name, codeName in pairs(data.Keybinds) do
                    Keybinds[name] = codeName and Enum.KeyCode[codeName] or nil
                end
            end
            if data.Config then
                if data.Config.MenuColor then Config.MenuColor = TableToColor(data.Config.MenuColor) end
                if data.Config.AccentColor then Config.AccentColor = TableToColor(data.Config.AccentColor) end
                if data.Config.TextColor then Config.TextColor = TableToColor(data.Config.TextColor) end
                if data.Config.HudColor then Config.HudColor = TableToColor(data.Config.HudColor) end
            end
            UpdateChams()
            UpdateTrails()
            UpdateParticles()
            CreateTargetHUD()
            UpdateFog()
            ApplyAntiFling()
            ToggleRTXShadows(Modules.Visuals.RTX.Shadows)
            ToggleRTXColorCorrection(Modules.Visuals.RTX.ColorCorrection)
            ToggleRTXBloom(Modules.Visuals.RTX.Bloom)
            ToggleRTXSunRays(Modules.Visuals.RTX.SunRays)
            ToggleRTXDOF(Modules.Visuals.RTX.DOF)
            if Modules.Fly.Enabled then StartFly() else StopFly() end
            UpdateAccentColor(Config.AccentColor)
            UpdateMenuColor(Config.MenuColor)
            UpdateHudColor(Config.HudColor)
            UpdateKeybindListUI()
            Notify("Config loaded: " .. filename)
        end
    else
        Notify("Config file not found: " .. filename)
    end
end

local function GetConfigsList()
    EnsureFolderExists()
    local list = {}
    if listfiles then
        local files = listfiles(FolderName)
        for _, file in ipairs(files) do
            local name = file:match("([^/^\\]+)%.json$")
            if name then
                table.insert(list, name)
            end
        end
    end
    return list
end

-- ===== DYNAMIC ANIMATED INFOHUD =====
local function CreateInfoHUD()
    local hudFrame = Instance.new("Frame")
    hudFrame.Name = "InfoHUD"
    hudFrame.Size = UDim2.new(0, 220, 0, 50)
    hudFrame.Position = UDim2.new(0, 15, 0, 15)
    hudFrame.BackgroundColor3 = Config.MenuColor
    hudFrame.BackgroundTransparency = 0.3
    hudFrame.BorderSizePixel = 0
    hudFrame.Parent = ScreenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = hudFrame

    local stroke = Instance.new("UIStroke")
    stroke.Color = Config.AccentColor
    stroke.Thickness = 1
    stroke.Parent = hudFrame
    table.insert(UIComponents.strokes, stroke)

    local infoContainer = Instance.new("Frame")
    infoContainer.Size = UDim2.new(1, -10, 1, -10)
    infoContainer.Position = UDim2.new(0, 5, 0, 5)
    infoContainer.BackgroundTransparency = 1
    infoContainer.Parent = hudFrame

    infoTitleLabel = Instance.new("TextLabel")
    infoTitleLabel.Size = UDim2.new(1, 0, 0.5, 0)
    infoTitleLabel.BackgroundTransparency = 1
    infoTitleLabel.Text = "SM4KOVO4KA"
    infoTitleLabel.TextColor3 = Config.AccentColor
    infoTitleLabel.TextSize = 15
    infoTitleLabel.Font = Enum.Font.GothamBlack
    infoTitleLabel.TextXAlignment = Enum.TextXAlignment.Left
    infoTitleLabel.Parent = infoContainer
    table.insert(UIComponents.titleLabels, infoTitleLabel)

    fpsLabel = Instance.new("TextLabel")
    fpsLabel.Size = UDim2.new(0.5, 0, 0.5, 0)
    fpsLabel.Position = UDim2.new(0, 0, 0.5, 0)
    fpsLabel.BackgroundTransparency = 1
    fpsLabel.Text = "FPS: 60"
    fpsLabel.TextColor3 = Config.TextColor
    fpsLabel.TextSize = 12
    fpsLabel.Font = Enum.Font.GothamBold
    fpsLabel.TextXAlignment = Enum.TextXAlignment.Left
    fpsLabel.Parent = infoContainer

    pingLabel = Instance.new("TextLabel")
    pingLabel.Size = UDim2.new(0.5, 0, 0.5, 0)
    pingLabel.Position = UDim2.new(0.5, 0, 0.5, 0)
    pingLabel.BackgroundTransparency = 1
    pingLabel.Text = "Ping: 0ms"
    pingLabel.TextColor3 = Config.TextColor
    pingLabel.TextSize = 12
    pingLabel.Font = Enum.Font.GothamBold
    pingLabel.TextXAlignment = Enum.TextXAlignment.Left
    pingLabel.Parent = infoContainer

    -- Text Change Animation Cycle (Every 5 seconds, typing effect)
    local titles = {"SM4KOVO4KA", "RAGEEHUUBB", "DEEPSEEKSLOOP<3", "СТРЕЙФЫ ХАБ"}
    task.spawn(function()
        local idx = 1
        while task.wait(5) do
            idx = idx % #titles + 1
            local targetText = titles[idx]
            for i = 1, #targetText do
                infoTitleLabel.Text = string.sub(targetText, 1, i)
                task.wait(0.05)
            end
        end
    end)
end

-- ===== KEYBIND LIST UI =====
local function CreateKeybindListUI()
    keybindListFrame = Instance.new("Frame")
    keybindListFrame.Name = "KeybindList"
    keybindListFrame.Size = UDim2.new(0, 180, 0, 150)
    keybindListFrame.Position = UDim2.new(1, -195, 0.5, -75)
    keybindListFrame.BackgroundColor3 = Config.MenuColor
    keybindListFrame.BackgroundTransparency = 0.2
    keybindListFrame.BorderSizePixel = 0
    keybindListFrame.Parent = ScreenGui

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = keybindListFrame

    local stroke = Instance.new("UIStroke")
    stroke.Color = Config.AccentColor
    stroke.Thickness = 1
    stroke.Parent = keybindListFrame
    table.insert(UIComponents.strokes, stroke)

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 25)
    title.BackgroundTransparency = 1
    title.Text = "Active Keybinds"
    title.TextColor3 = Config.AccentColor
    title.Font = Enum.Font.GothamBold
    title.TextSize = 13
    title.Parent = keybindListFrame

    local list = Instance.new("UIListLayout")
    list.Padding = UDim.new(0, 2)
    list.Parent = keybindListFrame
    
    local pad = Instance.new("UIPadding")
    pad.PaddingTop = UDim.new(0, 28)
    pad.PaddingLeft = UDim.new(0, 8)
    pad.PaddingRight = UDim.new(0, 8)
    pad.Parent = keybindListFrame
end

UpdateKeybindListUI = function()
    if not keybindListFrame then return end
    for _, child in pairs(keybindListFrame:GetChildren()) do
        if child:IsA("TextLabel") and child.Name ~= "Title" then child:Destroy() end
    end

    for name, key in pairs(Keybinds) do
        if key then
            local lbl = Instance.new("TextLabel")
            lbl.Size = UDim2.new(1, 0, 0, 16)
            lbl.BackgroundTransparency = 1
            lbl.Text = name .. " : [" .. key.Name .. "]"
            lbl.TextColor3 = Config.TextColor
            lbl.TextSize = 11
            lbl.Font = Enum.Font.Gotham
            lbl.TextXAlignment = Enum.TextXAlignment.Left
            lbl.Parent = keybindListFrame
        end
    end
end

-- ===== GUI CREATION HELPER FUNCTIONS =====
local function MakeDraggable(frame, target)
    local dragging, dragInput, dragStart, startPos
    frame.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = target.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)
    frame.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input == dragInput then
            local delta = input.Position - dragStart
            target.Position = UDim2.new(
                startPos.X.Scale, startPos.X.Offset + delta.X,
                startPos.Y.Scale, startPos.Y.Offset + delta.Y
            )
        end
    end)
end

local function CreateSectionTitle(text, parent)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 30)
    frame.BackgroundTransparency = 1
    frame.Parent = parent

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 1, 0)
    title.BackgroundTransparency = 1
    title.Text = text
    title.TextColor3 = Config.AccentColor
    title.TextSize = 16
    title.Font = Enum.Font.GothamBold
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = frame

    local line = Instance.new("Frame")
    line.Size = UDim2.new(1, 0, 0, 1)
    line.Position = UDim2.new(0, 0, 1, -1)
    line.BackgroundColor3 = Config.AccentColor
    line.BorderSizePixel = 0
    line.Parent = frame
    table.insert(UIComponents.strokes, line)
end

local function CreateButton(text, parent, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 35)
    btn.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    btn.TextColor3 = Config.TextColor
    btn.Text = text
    btn.TextSize = 14
    btn.Font = Enum.Font.Gotham
    btn.Parent = parent
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 4)
    corner.Parent = btn
    btn.MouseButton1Click:Connect(callback)
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Config.AccentColor}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(25,25,25)}):Play()
    end)
    return btn
end

local function CreateToggle(text, default, parent, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 30)
    frame.BackgroundTransparency = 1
    frame.Parent = parent

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Config.TextColor
    label.TextSize = 14
    label.Font = Enum.Font.Gotham
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local toggle = Instance.new("Frame")
    toggle.Size = UDim2.new(0, 44, 0, 22)
    toggle.Position = UDim2.new(1, -48, 0.5, -11)
    toggle.BackgroundColor3 = default and Config.AccentColor or Color3.fromRGB(40, 40, 40)
    toggle.Parent = frame
    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(1, 0)
    toggleCorner.Parent = toggle

    local circle = Instance.new("Frame")
    circle.Size = UDim2.new(0, 18, 0, 18)
    circle.Position = UDim2.new(0, default and 24 or 2, 0.5, -9)
    circle.BackgroundColor3 = Color3.new(1,1,1)
    circle.Parent = toggle
    local circleCorner = Instance.new("UICorner")
    circleCorner.CornerRadius = UDim.new(1,0)
    circleCorner.Parent = circle

    local state = default
    local function toggleState()
        state = not state
        TweenService:Create(toggle, TweenInfo.new(0.2), {BackgroundColor3 = state and Config.AccentColor or Color3.fromRGB(40,40,40)}):Play()
        TweenService:Create(circle, TweenInfo.new(0.2), {Position = UDim2.new(0, state and 24 or 2, 0.5, -9)}):Play()
        if callback then callback(state) end
    end
    toggle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            toggleState()
        end
    end)

    local function updateColor(newColor)
        if state then toggle.BackgroundColor3 = newColor end
    end
    table.insert(UIComponents.toggleFills, updateColor)

    return {Set = function(s) state = s; toggle.BackgroundColor3 = s and Config.AccentColor or Color3.fromRGB(40,40,40); circle.Position = UDim2.new(0, s and 24 or 2, 0.5, -9) end}
end

local function CreateSlider(text, min, max, default, parent, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 40)
    frame.BackgroundTransparency = 1
    frame.Parent = parent

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 0, 20)
    label.BackgroundTransparency = 1
    label.Text = text .. ": " .. string.format("%.1f", default)
    label.TextColor3 = Config.TextColor
    label.TextSize = 14
    label.Font = Enum.Font.Gotham
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local bar = Instance.new("Frame")
    bar.Size = UDim2.new(1, 0, 0, 4)
    bar.Position = UDim2.new(0, 0, 0, 24)
    bar.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    bar.Parent = frame
    local barCorner = Instance.new("UICorner")
    barCorner.CornerRadius = UDim.new(1,0)
    barCorner.Parent = bar

    local fill = Instance.new("Frame")
    fill.Size = UDim2.new((default-min)/(max-min), 0, 1, 0)
    fill.BackgroundColor3 = Config.AccentColor
    fill.Parent = bar
    local fillCorner = Instance.new("UICorner")
    fillCorner.CornerRadius = UDim.new(1,0)
    fillCorner.Parent = fill

    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 16, 0, 16)
    button.Position = UDim2.new((default-min)/(max-min), -8, 0.5, -8)
    button.BackgroundColor3 = Color3.new(1,1,1)
    button.Text = ""
    button.Parent = bar
    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(1,0)
    btnCorner.Parent = button

    local dragging = false
    local value = default
    local function update(val)
        value = math.clamp(val, min, max)
        label.Text = text .. ": " .. string.format("%.1f", value)
        local ratio = (value - min) / (max - min)
        fill.Size = UDim2.new(ratio, 0, 1, 0)
        button.Position = UDim2.new(ratio, -8, 0.5, -8)
        if callback then callback(value) end
    end

    button.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local mouse = LocalPlayer:GetMouse()
            local relX = math.clamp(mouse.X - bar.AbsolutePosition.X, 0, bar.AbsoluteSize.X)
            local newVal = min + (relX / bar.AbsoluteSize.X) * (max - min)
            update(newVal)
        end
    end)

    local function updateColor(newColor) fill.BackgroundColor3 = newColor end
    table.insert(UIComponents.sliderFills, updateColor)

    return {Set = update}
end

local function CreateDropdown(text, options, parent, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 40)
    frame.BackgroundTransparency = 1
    frame.Parent = parent

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.5, -5, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = text .. ":"
    label.TextColor3 = Config.TextColor
    label.TextSize = 14
    label.Font = Enum.Font.Gotham
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local dropdown = Instance.new("TextButton")
    dropdown.Size = UDim2.new(0.5, -5, 0, 28)
    dropdown.Position = UDim2.new(0.5, 0, 0.5, -14)
    dropdown.BackgroundColor3 = Color3.fromRGB(25,25,25)
    dropdown.TextColor3 = Config.TextColor
    dropdown.Text = options[1] or "None"
    dropdown.TextSize = 14
    dropdown.Font = Enum.Font.Gotham
    dropdown.Parent = frame
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 4)
    corner.Parent = dropdown

    dropdown.MouseButton1Click:Connect(function()
        local menu = Instance.new("Frame")
        menu.Size = UDim2.new(0, 150, 0, #options * 28)
        menu.Position = UDim2.new(0.5, 0, 1, 5)
        menu.BackgroundColor3 = Config.MenuColor
        menu.BorderSizePixel = 0
        menu.ZIndex = 10
        menu.Parent = dropdown
        local menuCorner = Instance.new("UICorner")
        menuCorner.CornerRadius = UDim.new(0, 4)
        menuCorner.Parent = menu
        local menuStroke = Instance.new("UIStroke")
        menuStroke.Color = Config.AccentColor
        menuStroke.Thickness = 1
        menuStroke.Parent = menu
        table.insert(UIComponents.strokes, menuStroke)

        local list = Instance.new("UIListLayout")
        list.Parent = menu

        for _, opt in ipairs(options) do
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1, 0, 0, 28)
            btn.BackgroundColor3 = Color3.fromRGB(30,30,30)
            btn.TextColor3 = Config.TextColor
            btn.Text = opt
            btn.TextSize = 14
            btn.Font = Enum.Font.Gotham
            btn.ZIndex = 11
            btn.Parent = menu
            btn.MouseButton1Click:Connect(function()
                dropdown.Text = opt
                menu:Destroy()
                if callback then callback(opt) end
            end)
        end
    end)
end

local function CreateColorPicker(text, defaultColor, parent, callback)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 35)
    frame.BackgroundTransparency = 1
    frame.Parent = parent

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.6, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Config.TextColor
    label.TextSize = 14
    label.Font = Enum.Font.Gotham
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local preview = Instance.new("TextButton")
    preview.Size = UDim2.new(0, 60, 0, 24)
    preview.Position = UDim2.new(1, -64, 0.5, -12)
    preview.BackgroundColor3 = defaultColor
    preview.Text = ""
    preview.Parent = frame
    local pCorner = Instance.new("UICorner")
    pCorner.CornerRadius = UDim.new(0, 4)
    pCorner.Parent = preview

    local currentHue, currentSat, currentVal = defaultColor:ToHSV()

    preview.MouseButton1Click:Connect(function()
        local pickerFrame = Instance.new("Frame")
        pickerFrame.Size = UDim2.new(0, 180, 0, 120)
        pickerFrame.Position = UDim2.new(1, -185, 1, 5)
        pickerFrame.BackgroundColor3 = Config.MenuColor
        pickerFrame.BorderSizePixel = 0
        pickerFrame.ZIndex = 15
        pickerFrame.Parent = preview

        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 4)
        corner.Parent = pickerFrame
        local stroke = Instance.new("UIStroke")
        stroke.Color = Config.AccentColor
        stroke.Thickness = 1
        stroke.Parent = pickerFrame

        local list = Instance.new("UIListLayout")
        list.Padding = UDim.new(0, 5)
        list.Parent = pickerFrame
        local pad = Instance.new("UIPadding")
        pad.PaddingTop = UDim.new(0, 5)
        pad.PaddingLeft = UDim.new(0, 5)
        pad.PaddingRight = UDim.new(0, 5)
        pad.Parent = pickerFrame

        CreateSlider("Hue", 0, 1, currentHue, pickerFrame, function(v)
            currentHue = v
            local c = Color3.fromHSV(currentHue, currentSat, currentVal)
            preview.BackgroundColor3 = c
            if callback then callback(c) end
        end)
        CreateSlider("Sat", 0, 1, currentSat, pickerFrame, function(v)
            currentSat = v
            local c = Color3.fromHSV(currentHue, currentSat, currentVal)
            preview.BackgroundColor3 = c
            if callback then callback(c) end
        end)
        CreateSlider("Val", 0, 1, currentVal, pickerFrame, function(v)
            currentVal = v
            local c = Color3.fromHSV(currentHue, currentSat, currentVal)
            preview.BackgroundColor3 = c
            if callback then callback(c) end
        end)
    end)
end

local function CreateKeybindPicker(actionName, parent)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 35)
    frame.BackgroundTransparency = 1
    frame.Parent = parent

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.6, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = actionName
    label.TextColor3 = Config.TextColor
    label.TextSize = 14
    label.Font = Enum.Font.Gotham
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 100, 0, 26)
    btn.Position = UDim2.new(1, -104, 0.5, -13)
    btn.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    btn.TextColor3 = Config.TextColor
    btn.Text = Keybinds[actionName] and Keybinds[actionName].Name or "[ None ]"
    btn.TextSize = 13
    btn.Font = Enum.Font.GothamBold
    btn.Parent = frame
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 4)
    corner.Parent = btn

    local listening = false
    btn.MouseButton1Click:Connect(function()
        listening = true
        btn.Text = "Press key..."
        local conn
        conn = UserInputService.InputBegan:Connect(function(input)
            if listening and input.UserInputType == Enum.UserInputType.Keyboard then
                if input.KeyCode == Enum.KeyCode.Escape then
                    Keybinds[actionName] = nil
                    btn.Text = "[ None ]"
                else
                    Keybinds[actionName] = input.KeyCode
                    btn.Text = input.KeyCode.Name
                end
                listening = false
                UpdateKeybindListUI()
                conn:Disconnect()
            end
        end)
    end)
end

local function CreateTab(name, displayName, parent, contentParent)
    local btn = Instance.new("TextButton")
    btn.Name = name .. "Tab"
    btn.Size = UDim2.new(1, 0, 0, 38)
    btn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    btn.TextColor3 = Config.TextColor
    btn.Text = displayName
    btn.TextSize = 13
    btn.Font = Enum.Font.GothamBold
    btn.Parent = parent
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 4)
    corner.Parent = btn

    local content = Instance.new("ScrollingFrame")
    content.Name = name .. "Content"
    content.Size = UDim2.new(1, 0, 1, 0)
    content.BackgroundTransparency = 1
    content.Visible = false
    content.ScrollBarThickness = 4
    content.ScrollBarImageColor3 = Config.AccentColor
    content.Parent = contentParent
    table.insert(UIComponents.scrollbars, content)

    local list = Instance.new("UIListLayout")
    list.Padding = UDim.new(0, 10)
    list.SortOrder = Enum.SortOrder.LayoutOrder
    list.Parent = content
    local pad = Instance.new("UIPadding")
    pad.PaddingTop = UDim.new(0, 10)
    pad.PaddingLeft = UDim.new(0, 10)
    pad.PaddingRight = UDim.new(0, 10)
    pad.Parent = content

    local function updateCanvasSize()
        content.CanvasSize = UDim2.new(0, 0, 0, list.AbsoluteContentSize.Y + 20)
    end
    list:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(updateCanvasSize)

    Tabs[name] = content

    btn.MouseButton1Click:Connect(function()
        for _, tab in pairs(Tabs) do tab.Visible = false end
        content.Visible = true
        currentTab = name
        TweenService:Create(btn, TweenInfo.new(0.2), {BackgroundColor3 = Config.AccentColor}):Play()
        for _, otherBtn in pairs(parent:GetChildren()) do
            if otherBtn:IsA("TextButton") and otherBtn ~= btn then
                TweenService:Create(otherBtn, TweenInfo.new(0.2), {BackgroundColor3 = Color3.fromRGB(20,20,20)}):Play()
            end
        end
    end)

    if name == "AIMBOT" then
        CreateSectionTitle("Aimbot", content)
        CreateToggle("Enabled", Modules.Aimbot.Enabled, content, function(s) Modules.Aimbot.Enabled = s end)
        CreateSlider("Smoothness", 0, 0.95, Modules.Aimbot.Smoothness, content, function(v) Modules.Aimbot.Smoothness = v end)
        CreateSlider("FOV", 10, 500, Modules.Aimbot.FOV, content, function(v) Modules.Aimbot.FOV = v end)
        CreateToggle("Show FOV Circle", Modules.Aimbot.ShowFOV, content, function(s) Modules.Aimbot.ShowFOV = s end)
        CreateToggle("Team Check", Modules.Aimbot.TeamCheck, content, function(s) Modules.Aimbot.TeamCheck = s end)
        CreateToggle("Visible Check", Modules.Aimbot.VisibleCheck, content, function(s) Modules.Aimbot.VisibleCheck = s end)
        CreateDropdown("Hit Part", {"Head","HumanoidRootPart","Torso"}, content, function(s) Modules.Aimbot.HitPart = s end)
    elseif name == "ANTIAIM" then
        CreateSectionTitle("Anti-Aim Controls", content)
        CreateToggle("Enable Anti-Aim", Modules.AntiAim.Enabled, content, function(s) Modules.AntiAim.Enabled = s end)
        CreateDropdown("Mode", {"Static", "Spinbot", "Jitter"}, content, function(s) Modules.AntiAim.Mode = s end)
        CreateSlider("Yaw Offset", 0, 360, Modules.AntiAim.YawOffset, content, function(v) Modules.AntiAim.YawOffset = v end)
        CreateSlider("Spin Speed", 1, 100, Modules.AntiAim.SpinSpeed, content, function(v) Modules.AntiAim.SpinSpeed = v end)
        CreateSlider("Jitter Range", 1, 180, Modules.AntiAim.JitterRange, content, function(v) Modules.AntiAim.JitterRange = v end)
    elseif name == "ESP" then
        CreateSectionTitle("Drawing 2D ESP", content)
        CreateToggle("Enabled", Modules.ESP.Enabled, content, function(s) Modules.ESP.Enabled = s end)
        CreateToggle("2D Boxes", Modules.ESP.Boxes, content, function(s) Modules.ESP.Boxes = s end)
        CreateToggle("Player Names", Modules.ESP.Names, content, function(s) Modules.ESP.Names = s end)
        CreateToggle("Health Bar", Modules.ESP.Health, content, function(s) Modules.ESP.Health = s end)
        CreateToggle("Distance", Modules.ESP.Distance, content, function(s) Modules.ESP.Distance = s end)
    elseif name == "MOVEMENT" then
        CreateSectionTitle("Movement & QuickStop", content)
        CreateSlider("Speed", 16, 100, Modules.Movement.Speed, content, function(v) Modules.Movement.Speed = v end)
        CreateSlider("Jump Power", 50, 200, Modules.Movement.JumpPower, content, function(v) Modules.Movement.JumpPower = v end)
        CreateToggle("QuickStop (Anti-Drift)", Modules.Movement.QuickStop, content, function(s) Modules.Movement.QuickStop = s end)
        CreateSlider("QuickStop Power", 0.1, 0.95, Modules.Movement.QuickStopFriction, content, function(v) Modules.Movement.QuickStopFriction = v end)
        CreateToggle("Noclip", Modules.Movement.Noclip, content, function(s) Modules.Movement.Noclip = s end)
        CreateToggle("Air Strafe", Modules.Movement.AirStrafe, content, function(s) Modules.Movement.AirStrafe = s end)
        CreateSlider("Air Strafe Control", 10, 200, Modules.Movement.AirStrafeControl, content, function(v) Modules.Movement.AirStrafeControl = v end)
        CreateSectionTitle("HVH Strafe", content)
        CreateToggle("Strafe", Modules.Movement.Strafe, content, function(s) Modules.Movement.Strafe = s end)
        CreateSlider("Strafe Speed", 20, 2500, Modules.Movement.StrafeSpeed, content, function(v) Modules.Movement.StrafeSpeed = v end)
        CreateButton("TP Up HVH", content, TpUp)
    elseif name == "TELEPORT" then
        CreateSectionTitle("Teleport Controls", content)
        CreateButton("Click TP (Or Press Key)", content, PerformClickTP)
        CreateButton("Select Target Under Mouse", content, SelectTargetUnderMouse)
        CreateButton("TP to Selected Target", content, PerformTargetTP)
    elseif name == "FLY" then
        CreateSectionTitle("Fly Settings", content)
        CreateToggle("Enable Fly", Modules.Fly.Enabled, content, function(s) Modules.Fly.Enabled = s; if s then StartFly() else StopFly() end end)
        CreateToggle("T-Pose Mode", Modules.Fly.TPose, content, function(s) Modules.Fly.TPose = s; if Modules.Fly.Enabled then StartFly() end end)
        CreateSlider("Fly Speed", 10, 300, Modules.Fly.Speed, content, function(v) Modules.Fly.Speed = v end)
    elseif name == "COMBAT" then
        CreateSectionTitle("Auto Click", content)
        CreateToggle("Auto Click", Modules.Combat.AutoClick, content, function(s) Modules.Combat.AutoClick = s end)
        CreateSlider("Click Speed", 5, 20, Modules.Combat.ClickSpeed, content, function(v) Modules.Combat.ClickSpeed = v end)
        CreateToggle("Auto Click R", Modules.Combat.AutoClickR, content, function(s) Modules.Combat.AutoClickR = s; UpdateAutoClickR() end)
        CreateSlider("Click R Delay (ms)", 1, 500, Modules.Combat.ClickRInterval, content, function(v) Modules.Combat.ClickRInterval = v; UpdateAutoClickR() end)
    elseif name == "VISUALS" then
        CreateSectionTitle("Visuals & Protection", content)
        CreateToggle("Fullbright", Modules.Visuals.Fullbright, content, function(s) Modules.Visuals.Fullbright = s end)
        CreateSlider("Camera FOV", 70, 120, Modules.Visuals.FOV, content, function(v) Modules.Visuals.FOV = v end)
        CreateToggle("Anti-Fling", Modules.Visuals.AntiFling, content, function(s) Modules.Visuals.AntiFling = s; if s then ApplyAntiFling() end end)
        
        CreateSectionTitle("RTX v2 Shaders", content)
        CreateToggle("Realistic Shadows", Modules.Visuals.RTX.Shadows, content, function(s) Modules.Visuals.RTX.Shadows = s; ToggleRTXShadows(s) end)
        CreateToggle("Color Correction", Modules.Visuals.RTX.ColorCorrection, content, function(s) Modules.Visuals.RTX.ColorCorrection = s; ToggleRTXColorCorrection(s) end)
        CreateToggle("Bloom Effect", Modules.Visuals.RTX.Bloom, content, function(s) Modules.Visuals.RTX.Bloom = s; ToggleRTXBloom(s) end)
        CreateToggle("Sun Rays", Modules.Visuals.RTX.SunRays, content, function(s) Modules.Visuals.RTX.SunRays = s; ToggleRTXSunRays(s) end)
        CreateToggle("Depth of Field", Modules.Visuals.RTX.DOF, content, function(s) Modules.Visuals.RTX.DOF = s; ToggleRTXDOF(s) end)

        CreateSectionTitle("Fog Settings", content)
        CreateToggle("Enable Fog", Modules.Visuals.Fog.Enabled, content, function(s) Modules.Visuals.Fog.Enabled = s; UpdateFog() end)
        CreateColorPicker("Fog Color", Modules.Visuals.Fog.Color, content, function(c) Modules.Visuals.Fog.Color = c; UpdateFog() end)
        CreateSlider("Fog Start", 0, 500, Modules.Visuals.Fog.Start, content, function(v) Modules.Visuals.Fog.Start = v; UpdateFog() end)
        CreateSlider("Fog End", 10, 2000, Modules.Visuals.Fog.End, content, function(v) Modules.Visuals.Fog.End = v; UpdateFog() end)

        CreateSectionTitle("Trails System (5 Types)", content)
        CreateToggle("Enable Trails", Modules.Visuals.Trails, content, function(s) Modules.Visuals.Trails = s; UpdateTrails() end)
        CreateDropdown("Trail Type", {"Basic","Rainbow","Pulse","Electric","Fade"}, content, function(t) Modules.Visuals.TrailType = t; UpdateTrails() end)
        CreateColorPicker("Trail Color", Modules.Visuals.TrailColor, content, function(c) Modules.Visuals.TrailColor = c; UpdateTrails() end)
        CreateSlider("Trail Width", 0.1, 5, Modules.Visuals.TrailWidth, content, function(v) Modules.Visuals.TrailWidth = v; UpdateTrails() end)
        CreateSlider("Trail Length", 0.5, 10, Modules.Visuals.TrailLength, content, function(v) Modules.Visuals.TrailLength = v; UpdateTrails() end)
        
        CreateSectionTitle("Particles System (5 Types)", content)
        CreateToggle("Enable Particles", Modules.Visuals.Particles, content, function(s) Modules.Visuals.Particles = s; UpdateParticles() end)
        CreateDropdown("Particle Type", {"Sparkles","Fire","Smoke","Stars","Hearts"}, content, function(t) Modules.Visuals.ParticleType = t; UpdateParticles() end)
        CreateColorPicker("Particle Color", Modules.Visuals.ParticleColor, content, function(c) Modules.Visuals.ParticleColor = c; UpdateParticles() end)
        CreateSlider("Particle Size", 0.2, 5, Modules.Visuals.ParticleSize, content, function(v) Modules.Visuals.ParticleSize = v; UpdateParticles() end)

        CreateSectionTitle("Skybox", content)
        for sky, _ in pairs(Skyboxes) do
            CreateButton(sky, content, function() ChangeSkybox(sky) end)
        end
        CreateSectionTitle("HUD Controls", content)
        CreateToggle("Target HUD", Modules.Visuals.TargetHUD, content, function(s) Modules.Visuals.TargetHUD = s; CreateTargetHUD() end)
    elseif name == "CHAMS" then
        CreateSectionTitle("Chams & ForceField", content)
        CreateToggle("Enable Chams", Modules.Chams.Enabled, content, function(s) Modules.Chams.Enabled = s; UpdateChams() end)
        CreateColorPicker("Chams Color", Modules.Chams.Color, content, function(c) Modules.Chams.Color = c; UpdateChams() end)
        CreateSlider("Transparency", 0, 1, Modules.Chams.Transparency, content, function(v) Modules.Chams.Transparency = v; UpdateChams() end)
        CreateDropdown("Material", {"ForceField","Neon","SmoothPlastic","Glass"}, content, function(s) Modules.Chams.Material = s; UpdateChams() end)
        CreateDropdown("Mode", {"Fill","Outline"}, content, function(s) Modules.Chams.Mode = s; UpdateChams() end)
    elseif name == "KEYBINDS" then
        CreateSectionTitle("Keybind Manager", content)
        CreateKeybindPicker("AimbotToggle", content)
        CreateKeybindPicker("NoclipToggle", content)
        CreateKeybindPicker("FlyToggle", content)
        CreateKeybindPicker("ClickTPKey", content)
        CreateKeybindPicker("TargetSelectKey", content)
        CreateKeybindPicker("TargetTPKey", content)
        CreateKeybindPicker("QuickStopKey", content)
        CreateKeybindPicker("TpUpAction", content)
        CreateKeybindPicker("AntiFlingToggle", content)
        CreateKeybindPicker("TrailsToggle", content)
    elseif name == "CONFIG" then
        CreateSectionTitle("Workspace Config System", content)
        local nameBox = Instance.new("TextBox")
        nameBox.Size = UDim2.new(1, 0, 0, 30)
        nameBox.PlaceholderText = "Enter config name..."
        nameBox.Text = ""
        nameBox.BackgroundColor3 = Color3.fromRGB(30,30,30)
        nameBox.TextColor3 = Config.TextColor
        nameBox.Font = Enum.Font.Gotham
        nameBox.TextSize = 14
        nameBox.Parent = content
        local boxCorner = Instance.new("UICorner")
        boxCorner.CornerRadius = UDim.new(0, 4)
        boxCorner.Parent = nameBox

        CreateButton("Save File Config", content, function()
            local name = nameBox.Text ~= "" and nameBox.Text or "default"
            SaveConfigToFile(name)
        end)

        CreateButton("Load File Config", content, function()
            local name = nameBox.Text ~= "" and nameBox.Text or "default"
            LoadConfigFromFile(name)
        end)

        CreateSectionTitle("Saved Configs", content)
        local refreshBtn = CreateButton("Refresh File List", content, function() end)
        local configContainer = Instance.new("Frame")
        configContainer.Size = UDim2.new(1, 0, 0, 0)
        configContainer.BackgroundTransparency = 1
        configContainer.Parent = content
        local cList = Instance.new("UIListLayout")
        cList.Padding = UDim.new(0, 5)
        cList.Parent = configContainer

        local function RefreshFiles()
            for _, child in pairs(configContainer:GetChildren()) do
                if child:IsA("TextButton") then child:Destroy() end
            end
            local files = GetConfigsList()
            for _, fileName in ipairs(files) do
                CreateButton("Load: " .. fileName, configContainer, function()
                    LoadConfigFromFile(fileName)
                end)
            end
        end
        refreshBtn.MouseButton1Click:Connect(RefreshFiles)
        RefreshFiles()
    elseif name == "CUSTOMIZE" then
        CreateSectionTitle("Visual Style & Menu Customization", content)
        CreateToggle("Rainbow Accent", rainbowEnabled, content, function(state) ToggleRainbow(state) end)
        CreateColorPicker("Accent Color", Config.AccentColor, content, function(c)
            if not rainbowEnabled then
                savedAccentColor = c
                UpdateAccentColor(c)
            end
        end)
        CreateColorPicker("Menu Background Color", Config.MenuColor, content, function(c)
            UpdateMenuColor(c)
        end)
        CreateColorPicker("Target HUD Color", Config.HudColor, content, function(c)
            UpdateHudColor(c)
        end)
    end
end

-- ===== TOGGLE MENU =====
ToggleMenu = function()
    isMenuVisible = not isMenuVisible
    if MainFrame then
        MainFrame.Visible = isMenuVisible
    end
    
    if menuSound then
        if isMenuVisible then
            menuSound.TimePosition = 0
            menuSound:Play()
        else
            menuSound:Pause()
        end
    end
end

local function CreateGUI()
    if ScreenGui then ScreenGui:Destroy() end
    ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "SM4KOVO4KA"
    ScreenGui.Parent = CoreGui

    UIScale = Instance.new("UIScale")
    UIScale.Parent = ScreenGui

    MainFrame = Instance.new("Frame")
    MainFrame.Name = "Main"
    MainFrame.Size = UDim2.new(0, Config.BaseWidth, 0, Config.BaseHeight)
    MainFrame.Position = UDim2.new(0.5, -Config.BaseWidth/2, 0.5, -Config.BaseHeight/2)
    MainFrame.BackgroundColor3 = Config.MenuColor
    MainFrame.BorderSizePixel = 0
    MainFrame.ClipsDescendants = true
    MainFrame.Parent = ScreenGui
    MainFrame.Visible = true -- Меню открывается автоматически при инжекте!

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 6)
    corner.Parent = MainFrame
    local stroke = Instance.new("UIStroke")
    stroke.Color = Config.AccentColor
    stroke.Thickness = 1
    stroke.Parent = MainFrame
    table.insert(UIComponents.strokes, stroke)

    local titleBar = Instance.new("Frame")
    titleBar.Name = "TitleBar"
    titleBar.Size = UDim2.new(1, 0, 0, 40)
    titleBar.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    titleBar.BorderSizePixel = 0
    titleBar.Parent = MainFrame

    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(0, 250, 1, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = Config.Title .. " " .. Config.Subtitle
    titleLabel.TextColor3 = Config.AccentColor
    titleLabel.TextSize = 16
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.TextXAlignment = Enum.TextXAlignment.Left
    titleLabel.Parent = titleBar
    table.insert(UIComponents.titleLabels, titleLabel)

    local closeBtn = Instance.new("TextButton")
    closeBtn.Size = UDim2.new(0, 30, 0, 30)
    closeBtn.Position = UDim2.new(1, -35, 0, 5)
    closeBtn.BackgroundColor3 = Color3.fromRGB(200, 40, 40)
    closeBtn.TextColor3 = Color3.new(1,1,1)
    closeBtn.Text = "X"
    closeBtn.TextSize = 16
    closeBtn.Font = Enum.Font.GothamBold
    closeBtn.Parent = titleBar
    closeBtn.MouseButton1Click:Connect(function() ToggleMenu() end)

    MakeDraggable(titleBar, MainFrame)

    local tabContainer = Instance.new("Frame")
    tabContainer.Size = UDim2.new(0, 140, 1, -40)
    tabContainer.Position = UDim2.new(0, 0, 0, 40)
    tabContainer.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
    tabContainer.BorderSizePixel = 0
    tabContainer.Parent = MainFrame
    local tabList = Instance.new("UIListLayout")
    tabList.Padding = UDim.new(0, 5)
    tabList.Parent = tabContainer

    local contentFrame = Instance.new("Frame")
    contentFrame.Size = UDim2.new(1, -145, 1, -45)
    contentFrame.Position = UDim2.new(0, 145, 0, 45)
    contentFrame.BackgroundTransparency = 1
    contentFrame.Parent = MainFrame

    CreateTab("AIMBOT", "A AIMBOT", tabContainer, contentFrame)
    CreateTab("ANTIAIM", "AA ANTI-AIM", tabContainer, contentFrame)
    CreateTab("ESP", "E ESP 2D", tabContainer, contentFrame)
    CreateTab("MOVEMENT", "M MOVEMENT", tabContainer, contentFrame)
    CreateTab("TELEPORT", "T TELEPORT", tabContainer, contentFrame)
    CreateTab("FLY", "X FLY", tabContainer, contentFrame)
    CreateTab("COMBAT", "C COMBAT", tabContainer, contentFrame)
    CreateTab("VISUALS", "R RENDER", tabContainer, contentFrame)
    CreateTab("CHAMS", "P CHAMS", tabContainer, contentFrame)
    CreateTab("KEYBINDS", "K KEYBINDS", tabContainer, contentFrame)
    CreateTab("CONFIG", "F CONFIG", tabContainer, contentFrame)
    CreateTab("CUSTOMIZE", "S CUSTOMIZE", tabContainer, contentFrame)

    Tabs["AIMBOT"].Visible = true
    local firstTab = tabContainer:FindFirstChild("AIMBOTTab")
    if firstTab then firstTab.BackgroundColor3 = Config.AccentColor end

    menuSound = Instance.new("Sound")
    menuSound.SoundId = Config.MenuMusicId
    menuSound.Volume = Config.MenuMusicVolume
    menuSound.Looped = true
    menuSound.Parent = ScreenGui
    menuSound:Play()
end

-- ===== GAME LOOP =====
local function UpdateGame()
    AimbotLoop()
    UpdateFOVCircle()
    Update2DESP()
    UpdateAntiAim()
    UpdateAirStrafeAndQuickstop()
    UpdateChams()

    local char = LocalPlayer.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            pcall(function()
                hum.WalkSpeed = Modules.Movement.Speed
                hum.JumpPower = Modules.Movement.JumpPower
            end)
        end
        if Modules.Movement.Noclip then
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    pcall(function() part.CanCollide = false end)
                end
            end
        end
    end

    if Modules.Visuals.Fullbright then
        Lighting.Ambient = Color3.new(1,1,1)
        Lighting.Brightness = 2
        Lighting.GlobalShadows = false
    elseif not Modules.Visuals.RTX.Shadows then
        Lighting.Ambient = Color3.new(0,0,0)
        Lighting.Brightness = 1
        Lighting.GlobalShadows = true
    end

    if Workspace.CurrentCamera then
        Workspace.CurrentCamera.FieldOfView = Modules.Visuals.FOV
    end

    UpdateFog()

    if fpsLabel and pingLabel then
        frameCount = frameCount + 1
        local now = tick()
        if now - lastFPSTime >= 0.5 then
            local fps = math.floor(frameCount / (now - lastFPSTime) + 0.5)
            fpsLabel.Text = "FPS: " .. fps
            frameCount = 0
            lastFPSTime = now
        end
        local ping = LocalPlayer:GetNetworkPing() * 1000
        pingLabel.Text = "Ping: " .. math.floor(ping + 0.5) .. "ms"
    end
end

-- ===== INIT =====
local function Init()
    UserInputService.ModalEnabled = false
    CreateGUI()
    CreateInfoHUD()
    CreateKeybindListUI()

    UserInputService.InputBegan:Connect(function(input, gp)
        if not gp then
            if input.KeyCode == Config.ToggleKey then
                ToggleMenu()
            elseif Keybinds.ClickTPKey and input.KeyCode == Keybinds.ClickTPKey then
                PerformClickTP()
            elseif Keybinds.TargetSelectKey and input.KeyCode == Keybinds.TargetSelectKey then
                SelectTargetUnderMouse()
            elseif Keybinds.TargetTPKey and input.KeyCode == Keybinds.TargetTPKey then
                PerformTargetTP()
            elseif Keybinds.QuickStopKey and input.KeyCode == Keybinds.QuickStopKey then
                local root = GetRootPart(LocalPlayer)
                if root then
                    root.AssemblyLinearVelocity = Vector3.new(0, root.AssemblyLinearVelocity.Y, 0)
                    Notify("Emergency QuickStop!")
                end
            elseif Keybinds.AimbotToggle and input.KeyCode == Keybinds.AimbotToggle then
                Modules.Aimbot.Enabled = not Modules.Aimbot.Enabled
                Notify("Aimbot: " .. tostring(Modules.Aimbot.Enabled))
            elseif Keybinds.NoclipToggle and input.KeyCode == Keybinds.NoclipToggle then
                Modules.Movement.Noclip = not Modules.Movement.Noclip
                Notify("Noclip: " .. tostring(Modules.Movement.Noclip))
            elseif Keybinds.FlyToggle and input.KeyCode == Keybinds.FlyToggle then
                Modules.Fly.Enabled = not Modules.Fly.Enabled
                if Modules.Fly.Enabled then StartFly() else StopFly() end
                Notify("Fly: " .. tostring(Modules.Fly.Enabled))
            elseif Keybinds.TpUpAction and input.KeyCode == Keybinds.TpUpAction then
                TpUp()
            elseif Keybinds.AntiFlingToggle and input.KeyCode == Keybinds.AntiFlingToggle then
                Modules.Visuals.AntiFling = not Modules.Visuals.AntiFling
                if Modules.Visuals.AntiFling then ApplyAntiFling() end
                Notify("Anti-Fling: " .. tostring(Modules.Visuals.AntiFling))
            elseif Keybinds.TrailsToggle and input.KeyCode == Keybinds.TrailsToggle then
                Modules.Visuals.Trails = not Modules.Visuals.Trails
                UpdateTrails()
                Notify("Trails: " .. tostring(Modules.Visuals.Trails))
            end
        end
    end)

    LocalPlayer.CharacterAdded:Connect(function()
        if Modules.Fly.Enabled then
            Modules.Fly.Enabled = false
            StopFly()
        end
        task.wait(0.5)
        UpdateTrails()
        UpdateParticles()
    end)

    RunService.RenderStepped:Connect(UpdateGame)
    UpdateAutoClickR()
    CreateTargetHUD()
    UpdateKeybindListUI()
    ChangeSkybox("Red Sky 1")
    Notify("SM4KOVO4KA R3AL1SE v12.0 Loaded! Menu is automatically opened.")
end

local success, err = pcall(Init)
if not success then
    warn("[SM4KOVO4KA Error]: " .. tostring(err))
end
