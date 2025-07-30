--================================================
--  Umut Mod Menü Basic  –  Right‑Shift to Toggle UI
--================================================
local Players       = game:GetService("Players")
local RunService    = game:GetService("RunService")
local TweenService  = game:GetService("TweenService")
local UIS           = game:GetService("UserInputService")
local LocalPlayer   = Players.LocalPlayer
local Cam           = workspace.CurrentCamera

--================================
--  STATE
--================================
local State = {
    esp        = false,
    boxEsp     = false,
    speed      = false,
    aimLock    = false,
    autoKill   = false,
    xray       = false,   -- XRay eklendi
}

local SPEED_VALUE      = 50
local NORMAL_SPEED     = 16
local AUTO_KILL_RANGE  = 11
local antennas, tags, boxes = {}, {}, {}

--================================
--  GUI SETUP
--================================
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "MM2_MOD_MENU"
gui.ResetOnSpawn = false

local menuWidth, menuHeight = 380, 420

local menu = Instance.new("Frame", gui)
menu.Size = UDim2.new(0, menuWidth, 0, menuHeight)
menu.Position = UDim2.new(1, 10, 0.25, 0)  -- offscreen right initially
menu.BackgroundColor3 = Color3.fromRGB(18, 18, 18)
menu.BorderSizePixel = 0
menu.ClipsDescendants = true
menu.AnchorPoint = Vector2.new(1, 0)  -- so we can slide in nicely

local uicorner = Instance.new("UICorner", menu)
uicorner.CornerRadius = UDim.new(0, 12)

local uistroke = Instance.new("UIStroke", menu)
uistroke.Thickness = 1.5
uistroke.Color = Color3.fromRGB(30, 30, 30)
uistroke.Transparency = 0.4

-- Header
local header = Instance.new("TextLabel", menu)
header.Size = UDim2.new(1, 0, 0, 48)
header.BackgroundTransparency = 1
header.Text = "🔍  MM2 MOD MENU"
header.TextColor3 = Color3.fromRGB(255, 255, 255)
header.Font = Enum.Font.GothamBold
header.TextSize = 24
header.TextXAlignment = Enum.TextXAlignment.Left
header.Position = UDim2.new(0, 20, 0, 8)

local dragLine = Instance.new("Frame", menu)
dragLine.Size = UDim2.new(0, menuWidth, 0, 2)
dragLine.Position = UDim2.new(0, 0, 0, 48)
dragLine.BackgroundColor3 = Color3.fromRGB(55, 55, 55)
dragLine.BorderSizePixel = 0

-- Drag functionality
local dragging, dragStart, startPos
header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = menu.Position
    end
end)

UIS.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        menu.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- Container for buttons
local buttonsContainer = Instance.new("Frame", menu)
buttonsContainer.Size = UDim2.new(1, -40, 1, -70)
buttonsContainer.Position = UDim2.new(0, 20, 0, 60)
buttonsContainer.BackgroundTransparency = 1
buttonsContainer.ClipsDescendants = true

-- UIListLayout for buttons
local layout = Instance.new("UIListLayout", buttonsContainer)
layout.Padding = UDim.new(0, 14)
layout.FillDirection = Enum.FillDirection.Vertical
layout.SortOrder = Enum.SortOrder.LayoutOrder

-- Style for toggle buttons
local function createToggle(text, order, initialState, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 42)
    btn.BackgroundColor3 = Color3.fromRGB(32, 32, 32)
    btn.AutoButtonColor = false
    btn.Text = ""
    btn.LayoutOrder = order
    btn.ClipsDescendants = true
    btn.Name = text:gsub("%s", "") .. "Toggle"

    local uicorner = Instance.new("UICorner", btn)
    uicorner.CornerRadius = UDim.new(0, 8)

    local hover = Instance.new("Frame", btn)
    hover.Size = UDim2.new(1, 0, 1, 0)
    hover.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    hover.BackgroundTransparency = 1
    hover.ZIndex = 2
    local hoverCorner = Instance.new("UICorner", hover)
    hoverCorner.CornerRadius = UDim.new(0, 8)

    btn.MouseEnter:Connect(function()
        hover.BackgroundTransparency = 0.4
    end)
    btn.MouseLeave:Connect(function()
        hover.BackgroundTransparency = 1
    end)

    local label = Instance.new("TextLabel", btn)
    label.Size = UDim2.new(1, -60, 1, 0)
    label.Position = UDim2.new(0, 16, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = text
    label.TextColor3 = Color3.fromRGB(220, 220, 220)
    label.Font = Enum.Font.Gotham
    label.TextSize = 20
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.ZIndex = 3

    local toggleCircle = Instance.new("Frame", btn)
    toggleCircle.Size = UDim2.new(0, 34, 0, 24)
    toggleCircle.Position = UDim2.new(1, -46, 0.5, -12)
    toggleCircle.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
    toggleCircle.ClipsDescendants = true
    toggleCircle.ZIndex = 3

    local toggleUICorner = Instance.new("UICorner", toggleCircle)
    toggleUICorner.CornerRadius = UDim.new(1, 0)

    local toggleCircleInner = Instance.new("Frame", toggleCircle)
    toggleCircleInner.Size = UDim2.new(0, 18, 0, 18)
    toggleCircleInner.Position = UDim2.new(0, 3, 0.5, -9)
    toggleCircleInner.BackgroundColor3 = Color3.fromRGB(180, 180, 180)
    toggleCircleInner.ClipsDescendants = true
    toggleCircleInner.ZIndex = 4
    local toggleInnerCorner = Instance.new("UICorner", toggleCircleInner)
    toggleInnerCorner.CornerRadius = UDim.new(1, 0)

    local toggled = initialState

    local function updateToggle()
        if toggled then
            toggleCircle.BackgroundColor3 = Color3.fromRGB(0, 185, 85)
            toggleCircleInner.Position = UDim2.new(1, -21, 0.5, -9)
            toggleCircleInner.BackgroundColor3 = Color3.fromRGB(245, 245, 245)
        else
            toggleCircle.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
            toggleCircleInner.Position = UDim2.new(0, 3, 0.5, -9)
            toggleCircleInner.BackgroundColor3 = Color3.fromRGB(180, 180, 180)
        end
    end
    updateToggle()

    btn.MouseButton1Click:Connect(function()
        toggled = not toggled
        updateToggle()
        State[text:gsub("%s", ""):lower()] = toggled
        if callback then
            callback(toggled)
        end
    end)

    return btn, function(state)
        toggled = state
        updateToggle()
    end
end

--================================================
--  TOGGLE CALLBACKS & FUNCTIONALITY
--================================================

-- ESP & BoxESP stuff
local function clearEsp()
    for _, v in pairs(antennas) do if v then v:Destroy() end end
    for _, v in pairs(tags) do if v then v:Destroy() end end
    for _, v in pairs(boxes) do if v then v:Destroy() end end
    antennas, tags, boxes = {}, {}, {}
end

local function roleColor(player)
    local tool = player.Backpack:FindFirstChildOfClass("Tool") or (player.Character and player.Character:FindFirstChildOfClass("Tool"))
    if tool then
        local n = tool.Name:lower()
        if n:find("knife") then return Color3.fromRGB(255, 0, 0) end
        if n:find("gun") then return Color3.fromRGB(0, 153, 255) end
    end
    return Color3.fromRGB(0, 255, 0)
end

local function addEspFor(player)
    if player == LocalPlayer or not player.Character or not player.Character:FindFirstChild("Head") then return end
    local col = roleColor(player)

    local tagGui = Instance.new("BillboardGui", player.Character)
    tagGui.AlwaysOnTop = true
    tagGui.Size = UDim2.new(0, 120, 0, 20)
    tagGui.StudsOffset = Vector3.new(0, 2.5, 0)
    local tl = Instance.new("TextLabel", tagGui)
    tl.Size = UDim2.new(1, 0, 1, 0)
    tl.BackgroundTransparency = 1
    tl.Text = player.Name
    tl.Font = Enum.Font.GothamBold
    tl.TextScaled = true
    tl.TextColor3 = col
    tags[#tags + 1] = tagGui

    local ant = Instance.new("Part", workspace)
    ant.Anchored = true
    ant.CanCollide = false
    ant.Size = Vector3.new(0.1, 600, 0.1)
    ant.Material = Enum.Material.Neon
    ant.Transparency = 0.3
    ant.Color = col
    local hb = RunService.Heartbeat:Connect(function()
        if player.Character and player.Character:FindFirstChild("Head") then
            ant.Position = player.Character.Head.Position + Vector3.new(0, 300, 0)
        end
    end)
    ant:GetPropertyChangedSignal("Parent"):Connect(function() hb:Disconnect() end)
    antennas[#antennas + 1] = ant

    local boxGui = Instance.new("BillboardGui", player.Character)
    boxGui.Size = UDim2.new(0, 110, 0, 110)
    boxGui.AlwaysOnTop = true
    local fr = Instance.new("Frame", boxGui)
    fr.Size = UDim2.new(1, 0, 1, 0)
    fr.BackgroundTransparency = 0.75
    fr.BackgroundColor3 = Color3.new(1, 1, 1)
    fr.BorderSizePixel = 2
    fr.BorderColor3 = col
    boxes[#boxes + 1] = boxGui
end

local function refreshESP()
    clearEsp()
    for _, pl in ipairs(Players:GetPlayers()) do addEspFor(pl) end
end

local function setSpeed(on)
    if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        if on then
            LocalPlayer.Character.Humanoid.WalkSpeed = SPEED_VALUE
        else
            LocalPlayer.Character.Humanoid.WalkSpeed = NORMAL_SPEED
        end
    end
end

local function killerTool()
    return LocalPlayer.Backpack:FindFirstChildOfClass("Tool") or (LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Tool"))
end

local function isKiller()
    local t = killerTool()
    return t and t.Name:lower():find("knife")
end

local function autoKill()
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer and pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") then
            local dist = (LocalPlayer.Character.HumanoidRootPart.Position - pl.Character.HumanoidRootPart.Position).Magnitude
            if dist < AUTO_KILL_RANGE then
                LocalPlayer.Character.HumanoidRootPart.CFrame = pl.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 1)
                local t = killerTool()
                if t then t:Activate() end
                task.wait(0.15)
            end
        end
    end
end

local aimTarget
local function getClosestPlayerToMouse()
    local closest
    local closestDist = math.huge
    local mousePos = UIS:GetMouseLocation()
    for _, pl in ipairs(Players:GetPlayers()) do
        if pl ~= LocalPlayer and pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") then
            local screenPos, onScreen = Cam:WorldToViewportPoint(pl.Character.HumanoidRootPart.Position)
            if onScreen then
                local dist = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(mousePos.X, mousePos.Y)).Magnitude
                if dist < closestDist then
                    closest = pl
                    closestDist = dist
                end
            end
        end
    end
    return closest
end

-- Aimlock
local function aimAt(target)
    if target and target.Character and target.Character:FindFirstChild("Head") then
        Cam.CFrame = CFrame.new(Cam.CFrame.Position, target.Character.Head.Position)
    end
end

-- XRay toggle callback
local function setXRay(on)
    State.xray = on
    for _, part in pairs(workspace:GetDescendants()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" and part.Transparency < 1 then
            if on then
                if part.CanCollide then
                    part.Transparency = 0.6
                    part.Material = Enum.Material.Neon
                    part.CastShadow = false
                end
            else
                part.Transparency = 0
                part.Material = Enum.Material.SmoothPlastic
                part.CastShadow = true
            end
        end
    end
end

--================================================
--  CREATE TOGGLE BUTTONS
--================================================
local espToggle, setEspToggle         = createToggle("ESP", 1, false, function(state)
    State.esp = state
    if state then refreshESP() else clearEsp() end
end)

local boxEspToggle, setBoxEspToggle   = createToggle("Box ESP", 2, false, function(state)
    State.boxEsp = state
    -- Box ESP için detaylı kod buraya eklenecek
end)

local speedToggle, setSpeedToggle     = createToggle("Speed", 3, false, function(state)
    State.speed = state
    setSpeed(state)
end)

local aimLockToggle, setAimLockToggle = createToggle("AimLock", 4, false, function(state)
    State.aimLock = state
end)

local autoKillToggle, setAutoKillToggle = createToggle("Auto Kill", 5, false, function(state)
    State.autoKill = state
end)

local xrayToggle, setXrayToggle       = createToggle("XRay", 6, false, function(state)
    setXRay(state)
end)

-- Add buttons to container
espToggle.Parent        = buttonsContainer
boxEspToggle.Parent     = buttonsContainer
speedToggle.Parent      = buttonsContainer
aimLockToggle.Parent    = buttonsContainer
autoKillToggle.Parent   = buttonsContainer
xrayToggle.Parent       = buttonsContainer

--================================
--  TOGGLE HOTKEY
--================================
UIS.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == Enum.KeyCode.RightShift then
        if menu.Position.X.Offset > 0 then
            menu:TweenPosition(UDim2.new(1, -menuWidth - 10, 0.25, 0), "Out", "Quart", 0.4, true)
        else
            menu:TweenPosition(UDim2.new(1, 10, 0.25, 0), "Out", "Quart", 0.4, true)
        end
    end
end)

--================================
--  MAIN LOOP
--================================
RunService.Heartbeat:Connect(function()
    if State.esp then refreshESP() end

    if State.speed then setSpeed(true) else setSpeed(false) end

    if State.autoKill and isKiller() then
        autoKill()
    end

    if State.aimLock then
        aimTarget = getClosestPlayerToMouse()
        aimAt(aimTarget)
    end
end)
