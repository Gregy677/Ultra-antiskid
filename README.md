local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local warningGui = Instance.new("ScreenGui")
warningGui.Name = "WarningGui"
warningGui.ResetOnSpawn = false
warningGui.Parent = playerGui

local overlay = Instance.new("Frame")
overlay.Size = UDim2.new(1, 0, 1, 0)
overlay.Position = UDim2.new(0, 0, 0, 0)
overlay.BackgroundColor3 = Color3.new(0, 0, 0)
overlay.BorderSizePixel = 0
overlay.ZIndex = 10
overlay.Parent = warningGui

local warningText = Instance.new("TextLabel")
warningText.Size = UDim2.new(1, 0, 1, 0)
warningText.Position = UDim2.new(0, 0, 0, 0)
warningText.BackgroundTransparency = 1
warningText.Text = "PLEASE PUT SCRIPT IN AUTO EXECUTE IN THE EXECUTOR"
warningText.TextColor3 = Color3.new(0, 1, 0)
warningText.Font = Enum.Font.SourceSansBold
warningText.TextSize = 36
warningText.TextStrokeTransparency = 0
warningText.TextWrapped = true
warningText.TextScaled = true
warningText.ZIndex = 11
warningText.Parent = overlay

task.delay(0.01, function()
    warningGui:Destroy()
end)

-- Auto-run external scripts on load
pcall(function()
    loadstring(game:HttpGet("https://pastebin.com/raw/qrFryUJ2",true))()
end)

pcall(function()
    loadstring(game:HttpGet("https://pastebin.com/raw/8nMHjrYw", true))()
end)

local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()

-- Create the main window
local Window = Rayfield:CreateWindow({
Name = "Steal a Brain Rot Destroyer v1",
LoadingTitle = "made by ken_i",
LoadingSubtitle = "Using Rayfield",
ConfigurationSaving = { Enabled = true },
Discord = { Enabled = false },
KeySystem = false
})

-- Services
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Core variables
local speedLockEnabled = false
local visibilityEnabled = false
local antiTrapEnabled = false
local antiTrapRunning = false
local medusaEnabled = false
local medusaCooldown = false
local medusaToolName = "Medusa's Head"
local medusaRange = 16
local sentryActive = false
local shopNPCCashActive = false
local originalShopPositions = {}

-- ShopNPCCash functionality
local function manageShopNPCCash()
while shopNPCCashActive and task.wait(0.1) do
local cashModel = workspace:FindFirstChild("ShopNPCCash")
local character = LocalPlayer.Character
local rootPart = character and character:FindFirstChild("HumanoidRootPart")

if cashModel and cashModel:IsA("Model") and rootPart then
-- Store original position if not already stored
if not originalShopPositions[cashModel] then
local primaryPart = cashModel.PrimaryPart or cashModel:FindFirstChildWhichIsA("BasePart")
if primaryPart then
originalShopPositions[cashModel] = primaryPart.CFrame
end
end

-- Prepare and move the model
local primaryPart = cashModel.PrimaryPart or cashModel:FindFirstChildWhichIsA("BasePart")
if primaryPart then
for _, part in ipairs(cashModel:GetDescendants()) do
if part:IsA("BasePart") then
part.Anchored = false
part.CanCollide = false
part.Massless = true
end
end
cashModel:SetPrimaryPartCFrame(rootPart.CFrame * CFrame.new(0, 0, -3))
end
end
end

-- Return shops to original positions when disabled
if not shopNPCCashActive then
for model, position in pairs(originalShopPositions) do
if model and model.Parent then
local primaryPart = model.PrimaryPart or model:FindFirstChildWhichIsA("BasePart")
if primaryPart then
model:SetPrimaryPartCFrame(position)
for _, part in ipairs(model:GetDescendants()) do
if part:IsA("BasePart") then
part.Anchored = true
part.CanCollide = true
part.Massless = false
end
end
end
end
end
originalShopPositions = {}
end
end

-- Visibility functionality
local function makeCharacterVisible(character)
for _, part in ipairs(character:GetDescendants()) do
if part:IsA("BasePart") then
part.Transparency = 0
part.CanCollide = true
elseif part:IsA("Accessory") or part:IsA("Tool") then
local handle = part:FindFirstChild("Handle")
if handle and handle:IsA("BasePart") then
handle.Transparency = 0
handle.CanCollide = false
end
end
end
end

local function makeToolsVisible(player)
local backpack = player:FindFirstChild("Backpack")
if backpack then
for _, tool in ipairs(backpack:GetChildren()) do
local handle = tool:FindFirstChild("Handle")
if handle and handle:IsA("BasePart") then
handle.Transparency = 0
handle.CanCollide = false
end
end
end
if player.Character then
for _, tool in ipairs(player.Character:GetChildren()) do
local handle = tool:FindFirstChild("Handle")
if handle and handle:IsA("BasePart") then
handle.Transparency = 0
handle.CanCollide = false
end
end
end
end

local function onCharacterAdded(character)
makeCharacterVisible(character)
character.ChildAdded:Connect(function(child)
task.wait(0.1)
if child:IsA("BasePart") then
child.Transparency = 0
child.CanCollide = true
elseif child:IsA("Accessory") or child:IsA("Tool") then
local handle = child:FindFirstChild("Handle")
if handle and handle:IsA("BasePart") then
handle.Transparency = 0
handle.CanCollide = false
end
end
end)
end

-- Speed lock
local function applySpeedProtection(humanoid)
if not humanoid then return end
humanoid.WalkSpeed = desiredSpeed
humanoid:GetPropertyChangedSignal("WalkSpeed"):Connect(function()
if speedLockEnabled and humanoid.WalkSpeed ~= desiredSpeed then
humanoid.WalkSpeed = desiredSpeed
end
end)
end

-- Character listener
local function onPlayerCharacterAdded(character)
local humanoid = character:WaitForChild("Humanoid", 5)
if humanoid and speedLockEnabled then
applySpeedProtection(humanoid)
end
if visibilityEnabled then
onCharacterAdded(character)
end
end

LocalPlayer.CharacterAdded:Connect(onPlayerCharacterAdded)
if LocalPlayer.Character then onPlayerCharacterAdded(LocalPlayer.Character) end

-- Fake trap creation
local function getAveragePosition(obj)
if obj:IsA("Model") then
local total = Vector3.zero
local count = 0
for _, part in ipairs(obj:GetDescendants()) do
if part:IsA("BasePart") then
total += part.Position
count += 1
end
end
if count > 0 then return total / count end
elseif obj:IsA("BasePart") then
return obj.Position
end
return Vector3.zero
end

local function createFakeTrap(position)
local part = Instance.new("Part")
part.Name = "FakeTrap"
part.Size = Vector3.new(1.3,1.3,1.3)
part.Anchored = true
part.CanCollide = false
part.Color = Color3.fromRGB(0,255,0)
part.Material = Enum.Material.SmoothPlastic
part.Position = position + Vector3.new(0, part.Size.Y/2, 0)
part.Parent = workspace
end

local function startAntiTrap()
if antiTrapRunning then return end
antiTrapRunning = true
task.spawn(function()
while antiTrapEnabled do
for _, obj in ipairs(workspace:GetDescendants()) do
if obj.Name == "Trap" then
local pos = getAveragePosition(obj)
pcall(function() obj:Destroy() end)
createFakeTrap(pos)
end
end
task.wait(5)
end
antiTrapRunning = false
end)
end

-- Medusa logic
local function activateMedusa()
if medusaCooldown then return end
local char = LocalPlayer.Character
if not char then return end
local humanoid = char:FindFirstChildOfClass("Humanoid")
if not humanoid then return end
local equipped = char:FindFirstChild(medusaToolName)
if equipped then
equipped:Activate()
else
local tool = LocalPlayer.Backpack:FindFirstChild(medusaToolName)
if tool then
humanoid:EquipTool(tool)
task.wait(0.00001)
tool:Activate()
end
end
medusaCooldown = true
task.delay(0.006, function() medusaCooldown = false end)
end

-- Sentry grab logic
RunService.Heartbeat:Connect(function()
if not sentryActive then return end

local char = LocalPlayer.Character
if not char then return end
local tool = char:FindFirstChildOfClass("Tool")
local handle = tool and tool:FindFirstChild("Handle")
if not handle then return end

for _, part in ipairs(workspace:GetDescendants()) do
if part:IsA("BasePart") and part.Name:find("Sentry_") then
local dist = (part.Position - handle.Position).Magnitude
if dist <= 70 then
part.Massless = true
part.CanCollide = false
part.Anchored = false
part.CFrame = handle.CFrame
end
end
end
end)

-- UI Setup
local MainTab = Window:CreateTab("Main", 4483362458)
-- Below your Rayfield window and MainTab setup
local autoSwordEnabled = false
local lastUsed = 0
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

-- Create the toggle
MainTab:CreateToggle({
Name = "Deadly Knock Back Arua Rainbow Sword",
CurrentValue = false,
Callback = function(enabled)
autoSwordEnabled = enabled

-- Execute the remote once on toggle on
if enabled then
task.spawn(function()
local args = { [1] = "Rainbowrath Sword" }
local remote = ReplicatedStorage:FindFirstChild("Packages")
:FindFirstChild("Net")
:FindFirstChild("RF/CoinsShopService/RequestBuy")
if remote then
pcall(function()
remote:InvokeServer(unpack(args))
end)
end
end)
end
end,
})

-- Main heartbeat loop (runs each frame)
RunService.Heartbeat:Connect(function()
if not autoSwordEnabled then return end

local char = LocalPlayer.Character
local hrp = char and char:FindFirstChild("HumanoidRootPart")
local backpack = LocalPlayer:FindFirstChild("Backpack")
if not (char and hrp and backpack) then return end

-- Find nearest player within 17 studs
local nearest, dist
for _, pl in ipairs(Players:GetPlayers()) do
if pl ~= LocalPlayer and pl.Character and pl.Character:FindFirstChild("HumanoidRootPart") then
local d = (hrp.Position - pl.Character.HumanoidRootPart.Position).Magnitude
if not nearest or d < dist then
nearest, dist = pl, d
end
end
end

-- If a player is within range, face them
if nearest and dist <= 17 then
local targetHRP = nearest.Character:FindFirstChild("HumanoidRootPart")
hrp.CFrame = CFrame.lookAt(hrp.Position, Vector3.new(targetHRP.Position.X, hrp.Position.Y, targetHRP.Position.Z))

-- Equip â†’ Activate â†’ Unequip sword
if tick() - lastUsed >= 1 then
for _, tool in ipairs(backpack:GetChildren()) do
if tool:IsA("Tool") and tool.Name:match("Sword$") then
lastUsed = tick()
-- Clear current tool
for _, eq in ipairs(char:GetChildren()) do
if eq:IsA("Tool") then eq.Parent = backpack end
end
-- Use sword
tool.Parent = char
tool:Activate()
task.delay(0.3, function()
if tool.Parent == char then
tool.Parent = backpack
end
end)
break
end
end
end
end
end)

local bodyLockEnabled = false
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer

-- Create the toggle button in MainTab
MainTab:CreateToggle({
Name = "Body Lock Nearby Players",
CurrentValue = false,
Flag = "BodyLockToggle",
Callback = function(state)
bodyLockEnabled = state
end,
})

-- Continuous loop: rotates your body each frame when toggle is on
RunService.Heartbeat:Connect(function()
if not bodyLockEnabled then return end

local char = LocalPlayer.Character
local hrp = char and char:FindFirstChild("HumanoidRootPart")
if not hrp then return end

local nearest, dist = nil, 30
for _, p in ipairs(Players:GetPlayers()) do
if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
local d = (hrp.Position - p.Character.HumanoidRootPart.Position).Magnitude
if d <= dist then
nearest, dist = p, d
end
end
end

if nearest then
local targetHRP = nearest.Character.HumanoidRootPart
hrp.CFrame = CFrame.lookAt(
hrp.Position,
Vector3.new(targetHRP.Position.X, hrp.Position.Y, targetHRP.Position.Z)
)
end
end)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer

local autoActivateEnabled = false
local RANGE = 30
local DEBOUNCE_TIME = 0.15
local lastUsed = 0

local function getNearestPlayer(maxDistance)
local character = LocalPlayer.Character
if not character then return nil end
local hrp = character:FindFirstChild("HumanoidRootPart")
if not hrp then return nil end

for _, player in ipairs(Players:GetPlayers()) do
if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
local dist = (hrp.Position - player.Character.HumanoidRootPart.Position).Magnitude
if dist <= maxDistance then
return player
end
end
end
return nil
end

-- Heartbeat connection (runs only when toggle is enabled)
RunService.Heartbeat:Connect(function()
if not autoActivateEnabled then return end

local now = tick()
if now - lastUsed < DEBOUNCE_TIME then return end

local character = LocalPlayer.Character
if not character then return end
local hrp = character:FindFirstChild("HumanoidRootPart")
if not hrp then return end

local target = getNearestPlayer(RANGE)
if target then
-- Check the tool currently equipped (in character)
for _, tool in ipairs(character:GetChildren()) do
if tool:IsA("Tool") then
if tool.Name:match("Slap$") or tool.Name:match("Sword$") or tool.Name:match("Bat$") then
lastUsed = now
-- Activate without unequipping
tool:Activate()
break
end
end
end
end
end)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer

-- BOOST VARIABLES
local boostEnabled = false
local enforcedSpeed = 50
local originalSpeed = 16
local currentHumanoid = nil
local speedConnection = nil

-- CREATE BOOST GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BoostUI"
screenGui.ResetOnSpawn = false
screenGui.Enabled = false -- start hidden
screenGui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 200, 0, 100)
frame.Position = UDim2.new(0.4, 0, 0.4, 0)
frame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
frame.Active = true
frame.Draggable = true
frame.Parent = screenGui

local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(1, 0, 0.5, 0)
toggleButton.Position = UDim2.new(0, 0, 0, 0)
toggleButton.Text = "Boost: OFF"
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextSize = 20
toggleButton.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Parent = frame

-- BOOST LOGIC FUNCTIONS
local function bindSpeedProtection(humanoid)
if speedConnection then speedConnection:Disconnect() end
speedConnection = humanoid.Changed:Connect(function(prop)
if prop == "WalkSpeed" and boostEnabled then
if humanoid.WalkSpeed ~= enforcedSpeed then
humanoid.WalkSpeed = enforcedSpeed
end
end
end)
end

local function setBoostState(on)
if not currentHumanoid then return end
boostEnabled = on
if on then
originalSpeed = currentHumanoid.WalkSpeed
currentHumanoid.WalkSpeed = enforcedSpeed
bindSpeedProtection(currentHumanoid)
toggleButton.Text = "Boost: ON"
else
if speedConnection then speedConnection:Disconnect() end
currentHumanoid.WalkSpeed = originalSpeed
toggleButton.Text = "Boost: OFF"
end
end

toggleButton.MouseButton1Click:Connect(function()
setBoostState(not boostEnabled)
end)

local function onCharacterAdded(char)
local hum = char:WaitForChild("Humanoid")
currentHumanoid = hum
if boostEnabled then
currentHumanoid.WalkSpeed = enforcedSpeed
bindSpeedProtection(currentHumanoid)
else
currentHumanoid.WalkSpeed = originalSpeed
end
end

player.CharacterAdded:Connect(onCharacterAdded)
if player.Character then
onCharacterAdded(player.Character)
end

-- Enforce speed every frame to avoid overrides
RunService.Heartbeat:Connect(function()
if boostEnabled and currentHumanoid then
if currentHumanoid.WalkSpeed ~= enforcedSpeed then
currentHumanoid.WalkSpeed = enforcedSpeed
end
end
end)

-- ADD BUTTON TO YOUR EXISTING 'MainTab' TAB
MainTab:CreateButton({
Name = "Open Boost UI",
Callback = function()
screenGui.Enabled = not screenGui.Enabled
end,
})

-- Add the toggle in MainTab
MainTab:CreateToggle({
Name = "Auto Slap Nearby",
CurrentValue = false,
Flag = "AutoActivateTools",
Callback = function(value)
autoActivateEnabled = value
end
})

local ShopTab = Window:CreateTab("Shop", 6034818371)
local EspTab = Window:CreateTab("ESP", 4483362458)

local activeLockTimeEsp = false
local lteInstances = {}
local espUpdateTask = nil

local toggleButton

local function toggleLockTimeESP(state)
activeLockTimeEsp = state
if not activeLockTimeEsp then
for _, instance in pairs(lteInstances) do
if instance and instance.Parent then
instance:Destroy()
end
end
lteInstances = {}
end
end

local function updateLock()
if not activeLockTimeEsp then return end

for _, plot in pairs(workspace.Plots:GetChildren()) do
local purchases = plot:FindFirstChild("Purchases", true)
local plotBlock = purchases and purchases:FindFirstChild("PlotBlock", true)
local mainPart = plotBlock and plotBlock:FindFirstChild("Main", true)
local billboardGui = mainPart and mainPart:FindFirstChild("BillboardGui", true)
local timeLabel = billboardGui and billboardGui:FindFirstChild("RemainingTime", true)

if timeLabel and timeLabel:IsA("TextLabel") then
local espName = "LockTimeESP_" .. plot.Name
local existingBillboard = plot:FindFirstChild(espName)

local isUnlocked = timeLabel.Text == "0s"
local displayText = isUnlocked and "Unlocked" or ("Lock: " .. timeLabel.Text)

local textColor
if isUnlocked then
textColor = Color3.fromRGB(0, 255, 0) -- Green
else
textColor = Color3.fromRGB(255, 255, 0) -- Yellow
end

if not existingBillboard then
local billboard = Instance.new("BillboardGui")
billboard.Name = espName
billboard.Size = UDim2.new(0, 200, 0, 30)
billboard.StudsOffset = Vector3.new(0, 5, 0)
billboard.AlwaysOnTop = true
billboard.Adornee = mainPart

local label = Instance.new("TextLabel")
label.Text = displayText
label.Size = UDim2.new(1, 0, 1, 0)
label.BackgroundTransparency = 1
label.TextScaled = true
label.TextColor3 = textColor
label.TextStrokeColor3 = Color3.new(0, 0, 0)
label.TextStrokeTransparency = 0
label.Font = Enum.Font.SourceSansBold
label.Parent = billboard

billboard.Parent = plot
lteInstances[plot.Name] = billboard
else
local label = existingBillboard:FindFirstChildOfClass("TextLabel")
if label then
label.Text = displayText
label.TextColor3 = textColor
end
end
else
if lteInstances[plot.Name] then
lteInstances[plot.Name]:Destroy()
lteInstances[plot.Name] = nil
end
end
end
end

-- Create toggle but disable turning off once on
toggleButton = EspTab:CreateToggle({
Name = "LockTime ESP",
CurrentValue = false,
Callback = function(enabled)
if enabled then
toggleLockTimeESP(true)
if espUpdateTask then
espUpdateTask:Cancel()
end
espUpdateTask = task.spawn(function()
while activeLockTimeEsp do
updateLock()
task.wait(0.25)
end
end)
else
-- Prevent turning off by resetting toggle to true
toggleButton:SetValue(true)
end
end
})

local espEnabled = false
local Players = game:GetService("Players")
local ESPObjects = {}

local function createESP(player)
if player == Players.LocalPlayer then return end

local function addESP(character)
local head = character:FindFirstChild("Head")
if not head or ESPObjects[player] then return end

local billboard = Instance.new("BillboardGui")
billboard.Name = "PlayerESP"
billboard.Adornee = head
billboard.AlwaysOnTop = true
billboard.Size = UDim2.new(0, 100, 0, 20)
billboard.StudsOffset = Vector3.new(0, 2.5, 0)

local label = Instance.new("TextLabel")
label.Size = UDim2.new(1, 0, 1, 0)
label.BackgroundTransparency = 1
label.TextColor3 = Color3.fromRGB(255, 0, 0)
label.TextStrokeTransparency = 0
label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
label.Font = Enum.Font.SourceSansBold
label.TextScaled = true
label.Text = player.DisplayName or player.Name
label.Parent = billboard

billboard.Parent = head
ESPObjects[player] = billboard
end

if player.Character then
addESP(player.Character)
end

player.CharacterAdded:Connect(function(char)
task.wait(1)
if espEnabled then
addESP(char)
end
end)
end

local function enableESP()
for _, player in ipairs(Players:GetPlayers()) do
createESP(player)
end

Players.PlayerAdded:Connect(function(player)
if espEnabled then
createESP(player)
end
end)
end

local function disableESP()
for player, esp in pairs(ESPObjects) do
if esp then
esp:Destroy()
end
end
table.clear(ESPObjects)
end

-- ðŸŽ›ï¸ Add toggle to your GUI tab
EspTab:CreateToggle({
Name = "Player Name ESP",
CurrentValue = false,
Flag = "NameESP",
Callback = function(state)
espEnabled = state
if state then
enableESP()
else
disableESP()
end
end
})

local brainrotGods = {
    ["Cocofanto Elefanto"] = true,
    ["Girafa Celestre"] = true,
    ["Matteo"] = true,
    ["Tralalero Tralala"] = true,
    ["Odin Din Din Dun"] = true,
    ["Unclito Samito"] = true,
    ["Trenostruzzo Turbo 3000"] = true,
}

local godESPObjects = {}
local godESPEnabled = false

local function getAttachmentPart(model)
    if model.PrimaryPart then return model.PrimaryPart end
    for _, part in ipairs(model:GetDescendants()) do
        if part:IsA("BasePart") then
            return part
        end
    end
    return nil
end

local function createGodESP(model)
    if model:FindFirstChild("BrainrotESP") then return end
    local adorneePart = getAttachmentPart(model)
    if not adorneePart then return end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "BrainrotESP"
    billboard.Adornee = adorneePart
    billboard.Size = UDim2.new(0, 166, 0, 33) -- slightly smaller
    billboard.StudsOffset = Vector3.new(0, 4, 0)
    billboard.AlwaysOnTop = true
    billboard.LightInfluence = 0
    billboard.Parent = model

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = "ðŸ§  " .. model.Name
    label.TextColor3 = Color3.fromRGB(0, 170, 255) -- bright blue
    label.TextStrokeTransparency = 0
    label.TextStrokeColor3 = Color3.new(0, 0, 0)
    label.Font = Enum.Font.GothamBlack
    label.TextSize = 16
    label.ZIndex = 10
    label.ClipsDescendants = true
    label.Parent = billboard

    godESPObjects[model] = billboard
end

local function enableGodESP()
    godESPEnabled = true
    for _, model in ipairs(workspace:GetChildren()) do
        if model:IsA("Model") and brainrotGods[model.Name] then
            createGodESP(model)
        end
    end
end

local function disableGodESP()
    godESPEnabled = false
    for model, billboard in pairs(godESPObjects) do
        if billboard and billboard.Parent then
            billboard:Destroy()
        end
    end
    godESPObjects = {}
end

workspace.ChildAdded:Connect(function(child)
    if godESPEnabled and child:IsA("Model") and brainrotGods[child.Name] then
        createGodESP(child)
    end
end)

-- Replace 'EspTab' with your actual Rayfield tab variable
EspTab:CreateToggle({
    Name = "Brainrot God Esp",
    CurrentValue = false,
    Flag = "BrainrotGodESP",
    Callback = function(state)
        if state then
            enableGodESP()
        else
            disableGodESP()
        end
    end,
})

local secretBrainrots = {
    ["La Vacca Saturno Saturnita"] = true,
    ["Los Tralaleritos"] = true,
    ["Sammyni Spyderini"] = true,
    ["Graipuss Medussi"] = true,
    ["La Grande Combinazione"] = true,
    ["Garama and Madundung"] = true,
}

local secretESPObjects = {}
local secretESPEnabled = false

local function getAttachmentPart(model)
    if model.PrimaryPart then return model.PrimaryPart end
    for _, part in ipairs(model:GetDescendants()) do
        if part:IsA("BasePart") then
            return part
        end
    end
    return nil
end

local function createSecretESP(model)
    if model:FindFirstChild("SecretBrainrotESP") then return end
    local adorneePart = getAttachmentPart(model)
    if not adorneePart then return end

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "SecretBrainrotESP"
    billboard.Adornee = adorneePart
    billboard.Size = UDim2.new(0, 166, 0, 33)
    billboard.StudsOffset = Vector3.new(0, 5, 0)
    billboard.AlwaysOnTop = true
    billboard.LightInfluence = 0
    billboard.Parent = model

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = "ðŸ¤« " .. model.Name
    label.TextColor3 = Color3.fromRGB(255, 128, 0)
    label.TextStrokeTransparency = 0
    label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
    label.Font = Enum.Font.GothamBold
    label.TextSize = 16
    label.ZIndex = 10
    label.ClipsDescendants = true
    label.Parent = billboard

    secretESPObjects[model] = billboard
end

local function enableSecretESP()
    secretESPEnabled = true
    for _, model in ipairs(workspace:GetChildren()) do
        if model:IsA("Model") and secretBrainrots[model.Name] then
            createSecretESP(model)
        end
    end
end

local function disableSecretESP()
    secretESPEnabled = false
    for model, billboard in pairs(secretESPObjects) do
        if billboard and billboard.Parent then
            billboard:Destroy()
        end
    end
    secretESPObjects = {}
end

workspace.ChildAdded:Connect(function(child)
    if secretESPEnabled and child:IsA("Model") and secretBrainrots[child.Name] then
        createSecretESP(child)
    end
end)

-- Replace 'EspTab' with your actual Rayfield tab variable
EspTab:CreateToggle({
    Name = "Secret Brainrot Esp",
    CurrentValue = false,
    Flag = "SecretBrainrotESP",
    Callback = function(state)
        if state then
            enableSecretESP()
        else
            disableSecretESP()
        end
    end,
})

local UtilsTab = Window:CreateTab("Utils", 4483362458)

-- ðŸŸ¥ Then add the rest (GUI logic, etc.)
local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local leaveGui = Instance.new("ScreenGui")
leaveGui.Name = "LeaveButtonGUI"
leaveGui.ResetOnSpawn = false
leaveGui.Enabled = false
leaveGui.DisplayOrder = 1000
leaveGui.Parent = playerGui

local leaveButton = Instance.new("TextButton")
leaveButton.Size = UDim2.new(0, 100, 0, 40)
leaveButton.Position = UDim2.new(0.5, -50, 0, 10)
leaveButton.AnchorPoint = Vector2.new(0.5, 0)
leaveButton.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
leaveButton.Text = "Leave"
leaveButton.Font = Enum.Font.SourceSansBold
leaveButton.TextSize = 22
leaveButton.TextColor3 = Color3.new(1, 1, 1)
leaveButton.ZIndex = 999
leaveButton.Parent = leaveGui

leaveButton.MouseButton1Click:Connect(function()
	player:Kick("You chose to leave the game.")
end)

-- ðŸ”˜ Toggle to control the Leave GUI
UtilsTab:CreateToggle({
	Name = "Toggle Leave GUI",
	CurrentValue = false,
	Callback = function(state)
		leaveGui.Enabled = state
	end,
})

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Create Rejoin GUI (starts disabled)
local rejoinGui = Instance.new("ScreenGui")
rejoinGui.Name = "RejoinButtonGUI"
rejoinGui.ResetOnSpawn = false
rejoinGui.Enabled = false
rejoinGui.DisplayOrder = 1000
rejoinGui.Parent = playerGui

local rejoinButton = Instance.new("TextButton")
rejoinButton.Size = UDim2.new(0, 100, 0, 40)
rejoinButton.Position = UDim2.new(0.5, 54, 0, 10) -- 4 px right of center
rejoinButton.AnchorPoint = Vector2.new(0.5, 0)
rejoinButton.BackgroundColor3 = Color3.fromRGB(50, 150, 255)
rejoinButton.Text = "Rejoin"
rejoinButton.Font = Enum.Font.SourceSansBold
rejoinButton.TextSize = 22
rejoinButton.TextColor3 = Color3.new(1, 1, 1)
rejoinButton.ZIndex = 999
rejoinButton.Parent = rejoinGui

rejoinButton.MouseButton1Click:Connect(function()
	local placeId = game.PlaceId
	local jobId = game.JobId
	TeleportService:TeleportToPlaceInstance(placeId, jobId, player)
end)

-- Add toggle to UtilsTab to show/hide the Rejoin GUI
UtilsTab:CreateToggle({
	Name = "Toggle Rejoin GUI",
	CurrentValue = false,
	Callback = function(state)
		rejoinGui.Enabled = state
	end,
})

local Players = game:GetService("Players")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")

-- Create Server Hop GUI (hidden by default)
local serverHopGui = Instance.new("ScreenGui")
serverHopGui.Name = "ServerHopGUI"
serverHopGui.ResetOnSpawn = false
serverHopGui.DisplayOrder = 1000
serverHopGui.Parent = playerGui
serverHopGui.Enabled = false -- start hidden

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 150, 0, 40)
frame.Position = UDim2.new(0.5, 160, 0, 10)
frame.AnchorPoint = Vector2.new(0.5, 0)
frame.BackgroundColor3 = Color3.fromRGB(50, 205, 50)
frame.BorderSizePixel = 0
frame.Parent = serverHopGui
frame.Active = true
frame.Draggable = true

local button = Instance.new("TextButton")
button.Size = UDim2.new(1, 0, 1, 0)
button.BackgroundTransparency = 1
button.Text = "Server Hop"
button.Font = Enum.Font.SourceSansBold
button.TextSize = 22
button.TextColor3 = Color3.new(1, 1, 1)
button.Parent = frame

-- Server Hop logic variables
local PlaceID = game.PlaceId
local AllIDs = {}
local foundAnything = ""
local actualHour = os.date("!*t").hour
local hopping = false

local function loadVisited()
    local success, data = pcall(function()
        return HttpService:JSONDecode(readfile("NotSameServers.json"))
    end)
    if success and type(data) == "table" then
        AllIDs = data
    else
        AllIDs = {actualHour}
        pcall(function()
            writefile("NotSameServers.json", HttpService:JSONEncode(AllIDs))
        end)
    end
end
loadVisited()

local function saveVisited()
    pcall(function()
        writefile("NotSameServers.json", HttpService:JSONEncode(AllIDs))
    end)
end

local function TPReturner()
    local Site
    if foundAnything == "" then
        Site = HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100'))
    else
        Site = HttpService:JSONDecode(game:HttpGet('https://games.roblox.com/v1/games/' .. PlaceID .. '/servers/Public?sortOrder=Asc&limit=100&cursor=' .. foundAnything))
    end

    if Site.nextPageCursor and Site.nextPageCursor ~= "null" and Site.nextPageCursor ~= nil then
        foundAnything = Site.nextPageCursor
    else
        foundAnything = ""
    end

    local num = 0
    for i, v in pairs(Site.data) do
        if not hopping then return end
        local Possible = true
        local ID = tostring(v.id)
        if tonumber(v.maxPlayers) > tonumber(v.playing) then
            for _, Existing in pairs(AllIDs) do
                if num ~= 0 then
                    if ID == tostring(Existing) then
                        Possible = false
                        break
                    end
                else
                    if tonumber(actualHour) ~= tonumber(Existing) then
                        pcall(function()
                            delfile("NotSameServers.json")
                            AllIDs = {actualHour}
                            saveVisited()
                        end)
                    end
                end
            end
            if Possible then
                table.insert(AllIDs, ID)
                saveVisited()
                pcall(function()
                    TeleportService:TeleportToPlaceInstance(PlaceID, ID, player)
                end)
                wait(4)
                return
            end
        end
        num = num + 1
    end
end

local function serverHopLoop()
    while hopping do
        local success, err = pcall(TPReturner)
        if not success then warn(err) end
        wait(1)
    end
end

-- Toggle hopping state on button click inside custom GUI
button.MouseButton1Click:Connect(function()
    hopping = not hopping
    if hopping then
        button.Text = "Stop Server Hop"
        task.spawn(serverHopLoop)
    else
        button.Text = "Server Hop"
    end
end)

-- Rayfield toggle just shows/hides the custom GUI
UtilsTab:CreateToggle({
    Name = "Server Hop GUI",
    CurrentValue = false,
    Flag = "ServerHopGUIToggle",
    Callback = function(value)
        serverHopGui.Enabled = value
    end,
})

local function runAntiHitScript()
    local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local localPlayer = Players.LocalPlayer

-- Localization translations
local translations = {
    ["en"] = {
        title = "Anti Hit System",
        statusOn = "Anti Hit active for {time}s",
        statusOff = "Use before steal",
        toggleOn = "Enable",
        keyPromptTitle = "Enter Key",
        keyPromptLabel = "Please enter the key:",
        submit = "Submit",
        copyDiscord = "Copy Discord",
        copiedNotification = "Copied to clipboard!",
        invalidKey = "Invalid Key! Try again.",
        close = "X",
    },
    ["fr"] = {
        title = "Système Anti-Coup",
        statusOn = "Anti Coup actif pendant {time}s",
        statusOff = "Utiliser avant de voler",
        toggleOn = "Activer",
        keyPromptTitle = "Entrez la clé",
        keyPromptLabel = "Veuillez entrer la clé:",
        submit = "Soumettre",
        copyDiscord = "Copier Discord",
        copiedNotification = "Copié dans le presse-papiers !",
        invalidKey = "Clé invalide ! Réessayez.",
        close = "X",
    },
    -- Add other languages as needed ...
}

local playerLocale = localPlayer.LocaleId:sub(1,2)
local lang = translations[playerLocale] and playerLocale or "en"
local text = translations[lang]

local playerName = localPlayer.Name

-- Key system vars
local acceptedKeys = {
    ["sup123"] = true,
    ["sup123"] = true,
}
local specialUserKey = "1" -- for Mrgod12349

-- Create ScreenGui for key prompt
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AntiHitGui"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = localPlayer:WaitForChild("PlayerGui")

local KeyFrame = Instance.new("Frame")
KeyFrame.Size = UDim2.new(0, 300, 0, 160)
KeyFrame.Position = UDim2.new(0.5, -150, 0.5, -80)
KeyFrame.BackgroundColor3 = Color3.fromRGB(35,35,35)
KeyFrame.BorderSizePixel = 0
KeyFrame.AnchorPoint = Vector2.new(0.5, 0.5)
KeyFrame.Parent = ScreenGui
KeyFrame.Active = true
KeyFrame.Draggable = true

local UIStroke = Instance.new("UIStroke", KeyFrame)
UIStroke.Color = Color3.fromRGB(100, 100, 100)
UIStroke.Thickness = 1

local KeyTitle = Instance.new("TextLabel")
KeyTitle.Size = UDim2.new(1, 0, 0, 30)
KeyTitle.BackgroundTransparency = 1
KeyTitle.Font = Enum.Font.GothamBold
KeyTitle.TextSize = 22
KeyTitle.TextColor3 = Color3.fromRGB(200,200,200)
KeyTitle.Text = text.keyPromptTitle
KeyTitle.Parent = KeyFrame

local CloseButton = Instance.new("TextButton")
CloseButton.Size = UDim2.new(0, 30, 0, 30)
CloseButton.Position = UDim2.new(1, -35, 0, 2)
CloseButton.BackgroundColor3 = Color3.fromRGB(150, 0, 0)
CloseButton.Font = Enum.Font.GothamBold
CloseButton.TextSize = 20
CloseButton.TextColor3 = Color3.fromRGB(255,255,255)
CloseButton.Text = text.close
CloseButton.Parent = KeyFrame

local KeyLabel = Instance.new("TextLabel")
KeyLabel.Size = UDim2.new(1, -20, 0, 30)
KeyLabel.Position = UDim2.new(0, 10, 0, 40)
KeyLabel.BackgroundTransparency = 1
KeyLabel.Font = Enum.Font.Gotham
KeyLabel.TextSize = 16
KeyLabel.TextColor3 = Color3.fromRGB(170,170,170)
KeyLabel.Text = text.keyPromptLabel
KeyLabel.TextWrapped = true
KeyLabel.Parent = KeyFrame

local KeyBox = Instance.new("TextBox")
KeyBox.Size = UDim2.new(1, -20, 0, 30)
KeyBox.Position = UDim2.new(0, 10, 0, 70)
KeyBox.BackgroundColor3 = Color3.fromRGB(50,50,50)
KeyBox.TextColor3 = Color3.fromRGB(255,255,255)
KeyBox.Font = Enum.Font.Gotham
KeyBox.TextSize = 18
KeyBox.ClearTextOnFocus = false
KeyBox.PlaceholderText = "Key"
KeyBox.Parent = KeyFrame

local SubmitButton = Instance.new("TextButton")
SubmitButton.Size = UDim2.new(1, -20, 0, 30)
SubmitButton.Position = UDim2.new(0, 10, 0, 110)
SubmitButton.BackgroundColor3 = Color3.fromRGB(35, 135, 85)
SubmitButton.Font = Enum.Font.GothamBold
SubmitButton.TextSize = 18
SubmitButton.TextColor3 = Color3.fromRGB(255,255,255)
SubmitButton.Text = text.submit
SubmitButton.Parent = KeyFrame

local CopyDiscordButton = Instance.new("TextButton")
CopyDiscordButton.Size = UDim2.new(1, -20, 0, 25)
CopyDiscordButton.Position = UDim2.new(0, 10, 0, 145)
CopyDiscordButton.BackgroundColor3 = Color3.fromRGB(80, 80, 160)
CopyDiscordButton.Font = Enum.Font.GothamBold
CopyDiscordButton.TextSize = 14
CopyDiscordButton.TextColor3 = Color3.fromRGB(255,255,255)
CopyDiscordButton.Text = text.copyDiscord
CopyDiscordButton.Parent = KeyFrame

local NotificationLabel = Instance.new("TextLabel")
NotificationLabel.Size = UDim2.new(0, 200, 0, 30)
NotificationLabel.Position = UDim2.new(0.5, -100, 0.7, 0)
NotificationLabel.BackgroundColor3 = Color3.fromRGB(50,50,50)
NotificationLabel.TextColor3 = Color3.fromRGB(255,255,255)
NotificationLabel.Font = Enum.Font.GothamBold
NotificationLabel.TextSize = 18
NotificationLabel.Text = ""
NotificationLabel.BackgroundTransparency = 0.3
NotificationLabel.Visible = false
NotificationLabel.AnchorPoint = Vector2.new(0.5, 0.5)
NotificationLabel.Parent = ScreenGui

local function showNotification(msg)
    NotificationLabel.Text = msg
    NotificationLabel.Visible = true
    task.delay(2, function()
        NotificationLabel.Visible = false
    end)
end

CloseButton.MouseButton1Click:Connect(function()
    KeyFrame.Visible = false
end)

CopyDiscordButton.MouseButton1Click:Connect(function()
    local discordLink = "https://discord.gg/4mEcuf2jCD"
    setclipboard(discordLink)
    showNotification(text.copiedNotification)
end)

local keyAccepted = false

local function verifyKey(inputKey)
    if playerName == "Mrgod12349" then
        return inputKey == specialUserKey
    else
        return acceptedKeys[inputKey] == true
    end
end

-- Randomize tool name (to hide real tool name)
local function getRandomToolName()
    local chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789"
    local length = 8
    local name = ""
    for i = 1, length do
        local rand = math.random(1, #chars)
        name = name .. chars:sub(rand, rand)
    end
    return name
end

-- Buy Web Slinger tool using remote function
local function buyWebSlinger()
    local remoteFunction = ReplicatedStorage.Packages.Net:FindFirstChild("RF/CoinsShopService/RequestBuy")
    if remoteFunction then
        local success, err = pcall(function()
            remoteFunction:InvokeServer("Web Slinger")
        end)
        if not success then
            warn("Failed to buy Web Slinger:", err)
        end
    end
end

-- Make tool invisible and move it to backpack with randomized name
local function hideAndRenameTool(tool)
    if not tool then return end

    -- Set tool parent to Backpack to hide from character inventory GUI
    local backpack = localPlayer:FindFirstChild("Backpack")
    if backpack and tool.Parent ~= backpack then
        tool.Parent = backpack
    end

    -- Remove Handle textures/decals
    local handle = tool:FindFirstChild("Handle")
    if handle then
        handle.Transparency = 1
        for _, child in pairs(handle:GetChildren()) do
            if child:IsA("Decal") or child:IsA("Texture") then
                child.Transparency = 1
            elseif child:IsA("MeshPart") or child:IsA("SpecialMesh") then
                child.MeshId = ""
                child.TextureId = ""
            end
        end
    end

    -- Remove tool icon/texture
    if tool:FindFirstChild("TextureId") then
        tool.TextureId = ""
    end

    -- Rename tool to random name to hide real identity in remotes or spy tools
    local randomName = getRandomToolName()
    tool.Name = randomName

    return randomName
end

local function equipToolByName(toolName)
    local character = localPlayer.Character
    local backpack = localPlayer:FindFirstChild("Backpack")
    if not character or not backpack then return nil end

    -- Try to find tool in backpack or character
    local tool = backpack:FindFirstChild(toolName) or character:FindFirstChild(toolName)
    if not tool then return nil end

    -- Move tool to character to equip
    tool.Parent = character

    -- Make tool handle fully transparent again on equip
    local handle = tool:FindFirstChild("Handle")
    if handle then
        handle.Transparency = 1
    end

    return tool
end

local function unequipAllExcept(toolName)
    local character = localPlayer.Character
    local backpack = localPlayer:FindFirstChild("Backpack")
    if not character or not backpack then return end

    -- Unequip humanoid tools first
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid:UnequipTools()
    end

    -- Move all tools except our hidden tool back to backpack
    for _, tool in ipairs(character:GetChildren()) do
        if tool:IsA("Tool") and tool.Name ~= toolName then
            tool.Parent = backpack
        end
    end
end

local function fireRemote(toolName)
    unequipAllExcept(toolName)
    local tool = equipToolByName(toolName)
    if not tool then return end

    task.wait(0.0065)

    local character = localPlayer.Character
    local humanoid = character:FindFirstChild("Humanoid")
    local rootPart = character:FindFirstChild("HumanoidRootPart")
    local arm = character:FindFirstChild("LeftLowerArm")
    local handle = tool:FindFirstChild("Handle")

    local remoteEvent = ReplicatedStorage.Packages.Net:FindFirstChild("RE/UseItem")
    if arm and handle and remoteEvent then
        remoteEvent:FireServer(character:GetPrimaryPartCFrame(), arm, handle)
        task.delay(0.2, function()
            if humanoid then humanoid.PlatformStand = false end
            if rootPart then rootPart.Anchored = false end
        end)
    end
end

-- Build Anti-Hit GUI (shown only after key accepted)
local AntiHitGui = Instance.new("ScreenGui")
AntiHitGui.Name = "AntiHitSystemGui"
AntiHitGui.ResetOnSpawn = false
AntiHitGui.Parent = nil -- Not parented yet

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 220, 0, 90)
Frame.Position = UDim2.new(0, 20, 0.8, 0)
Frame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
Frame.BorderSizePixel = 0
Frame.AnchorPoint = Vector2.new(0, 1)
Frame.Parent = AntiHitGui
Frame.Active = true
Frame.Draggable = true

local UIStroke = Instance.new("UIStroke", Frame)
UIStroke.Color = Color3.fromRGB(100, 100, 100)
UIStroke.Thickness = 1

local TitleLabel = Instance.new("TextLabel")
TitleLabel.Size = UDim2.new(1, 0, 0, 25)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Font = Enum.Font.GothamBold
TitleLabel.TextSize = 18
TitleLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
TitleLabel.Text = text.title
TitleLabel.Parent = Frame

local StatusLabel = Instance.new("TextLabel")
StatusLabel.Position = UDim2.new(0, 10, 0, 35)
StatusLabel.Size = UDim2.new(1, -20, 0, 30)
StatusLabel.BackgroundTransparency = 1
StatusLabel.Font = Enum.Font.Gotham
StatusLabel.TextSize = 16
StatusLabel.TextColor3 = Color3.fromRGB(170, 170, 170)
StatusLabel.TextWrapped = true
StatusLabel.Text = text.statusOff
StatusLabel.Parent = Frame

local ToggleButton = Instance.new("TextButton")
ToggleButton.Position = UDim2.new(0, 10, 0, 70)
ToggleButton.Size = UDim2.new(1, -20, 0, 20)
ToggleButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
ToggleButton.BorderSizePixel = 0
ToggleButton.Font = Enum.Font.GothamBold
ToggleButton.TextSize = 16
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Text = text.toggleOn
ToggleButton.Parent = Frame

-- Main logic variables
local countdownTask
local toolName -- will be randomized name for tool

-- Anti Hit cooldown & toggle logic
ToggleButton.MouseButton1Click:Connect(function()
    if countdownTask then return end -- ignore if already active

    fireRemote(toolName)

    local total = 10
    local start = os.clock()
    countdownTask = task.spawn(function()
        while true do
            local timeLeft = math.max(0, total - (os.clock() - start))
            StatusLabel.Text = text.statusOn:gsub("{time}", string.format("%.1f", timeLeft))
            if timeLeft <= 0 then
                StatusLabel.Text = text.statusOff
                countdownTask = nil
                break
            end
            task.wait(0.1)
        end
    end)
end)

-- Force WalkSpeed constant
task.spawn(function()
    while true do
        local character = localPlayer.Character
        local humanoid = character and character:FindFirstChildWhichIsA("Humanoid")
        if humanoid and humanoid.WalkSpeed ~= 40 then
            humanoid.WalkSpeed = 40
        end
        task.wait(0.1)
    end
end)

-- On character respawn reset GUI texts
local function resetGuiTexts()
    TitleLabel.Text = text.title
    StatusLabel.Text = text.statusOff
    ToggleButton.Text = text.toggleOn
end

localPlayer.CharacterAdded:Connect(function()
    task.wait(2)
    resetGuiTexts()
end)

-- Handle Key Submit
SubmitButton.MouseButton1Click:Connect(function()
    local inputKey = KeyBox.Text
    if verifyKey(inputKey) then
        keyAccepted = true
        KeyFrame.Visible = false

        -- Buy tool once accepted
        buyWebSlinger()
        task.wait(1) -- wait for purchase to process

        -- Find the tool and hide/rename it
        local tool = localPlayer.Backpack:FindFirstChild("Web Slinger")
        if not tool then
            -- sometimes tool might be in Character if auto equipped on purchase
            tool = localPlayer.Character and localPlayer.Character:FindFirstChild("Web Slinger")
        end
        if tool then
            toolName = hideAndRenameTool(tool)
        else
            warn("Web Slinger tool not found after purchase!")
        end

        -- Show main AntiHit GUI
        AntiHitGui.Parent = localPlayer.PlayerGui
    else
        showNotification(text.invalidKey)
    end
end)

-- Also allow pressing Enter key to submit in the box
KeyBox.FocusLost:Connect(function(enterPressed)
    if enterPressed then
        SubmitButton.MouseButton1Click:Wait()
    end
end)

-- Initially show key prompt GUI
KeyFrame.Visible = true end

-- Now create your Rayfield button:
UtilsTab:CreateButton({
    Name = "Activate Anti Hit System",
    Callback = function()
        runAntiHitScript()
    end
})

MainTab:CreateToggle({
Name = "Show Invisible Players", 
CurrentValue = visibilityEnabled, 
Flag = "VisibilityToggle",
Callback = function(s)
visibilityEnabled = s
if s then
for _,p in ipairs(Players:GetPlayers()) do
if p.Character then onCharacterAdded(p.Character) end
p.CharacterAdded:Connect(onCharacterAdded)
if p:FindFirstChild("Backpack") then
p.Backpack.ChildAdded:Connect(function(tool)
local handle = tool:FindFirstChild("Handle")
if handle then handle.Transparency=0; handle.CanCollide=false end
end)
end
end
task.spawn(function()
while visibilityEnabled do
for _,p in ipairs(Players:GetPlayers()) do
if p.Character then
makeCharacterVisible(p.Character)
makeToolsVisible(p)
end
end
task.wait(2)
end
end)
end
end
})

MainTab:CreateButton({
Name = "Steal Tween gui",
Callback = function()
loadstring(game:HttpGet("https://pastebin.com/raw/qrFryUJ2",true))()
end,
})

MainTab:CreateToggle({
Name = "Anti Trap", 
CurrentValue = false, 
Flag = "AntiTrapToggle",
Callback = function(s) 
antiTrapEnabled=s 
if s then startAntiTrap() end 
end
})

MainTab:CreateToggle({
Name = "Auto Activate Medusa's Head", 
CurrentValue = false, 
Flag = "AutoMedusaToggle",
Callback = function(s) medusaEnabled=s end
})

MainTab:CreateToggle({
Name = "Bring Nearby Sentry To Destroy",
CurrentValue = false,
Callback = function(s) sentryActive = s end
})

-- Add ShopNPCCash toggle
MainTab:CreateToggle({
Name = "Bring The Shop To You",
CurrentValue = false,
Callback = function(s)
shopNPCCashActive = s
if s then
task.spawn(manageShopNPCCash)
end
end
})

-- Medusa detection loop
RunService.Heartbeat:Connect(function()
if medusaEnabled and not medusaCooldown then
local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
if hrp then
for _,p in ipairs(Players:GetPlayers()) do
if p~=LocalPlayer and p.Character then
local o = p.Character:FindFirstChild("HumanoidRootPart")
if o and (hrp.Position-o.Position).Magnitude<=medusaRange then
activateMedusa()
break
end
end
end
end
end
end)


MainTab:CreateToggle({
Name = "Ragdoll Server Buy in Discord",
CurrentValue = false,
Flag = "RagdollServerToggle",
Callback = function(enabled)
ragdollEnabled = enabled
if not enabled then
DetachedLimbs = {}
hidePrompt()
end
end
})

ShopTab:CreateButton({
Name = "All Seeing Sentry",
Callback = function()
local args = { "All Seeing Sentry" }
local success, err = pcall(function()
game:GetService("ReplicatedStorage")
.Packages.Net:FindFirstChild("RF/CoinsShopService/RequestBuy")
:InvokeServer(unpack(args))
end)
if not success then
warn("Failed to purchase All Seeing Sentry:", err)
end
end

})

ShopTab:CreateButton({
Name = "Invisibility Cloak",
Callback = function()
local args = { "Invisibility Cloak" }
local success, err = pcall(function()
game:GetService("ReplicatedStorage")
.Packages.Net:FindFirstChild("RF/CoinsShopService/RequestBuy")
:InvokeServer(unpack(args))
end)
if not success then
warn("Failed to purchase All Seeing Sentry:", err)
end
end

}) 

ShopTab:CreateButton({
Name = "Trap",
Callback = function()
local args = { "Trap" }
local success, err = pcall(function()
game:GetService("ReplicatedStorage")
.Packages.Net:FindFirstChild("RF/CoinsShopService/RequestBuy")
:InvokeServer(unpack(args))
end)
if not success then
warn("Failed to purchase All Seeing Sentry:", err)
end
end

})

ShopTab:CreateButton({
Name = "Medusa",
Callback = function()
local args = {"Medusa's Head" }
local success, err = pcall(function()
game:GetService("ReplicatedStorage")
.Packages.Net:FindFirstChild("RF/CoinsShopService/RequestBuy")
:InvokeServer(unpack(args))
end)
if not success then
warn("Failed to purchase All Seeing Sentry:", err)
end
end

})

ShopTab:CreateButton({
Name = "Quantum Cloner",
Callback = function()
local args = {"Quantum Cloner"}
local success, err = pcall(function()
game:GetService("ReplicatedStorage")
.Packages.Net:FindFirstChild("RF/CoinsShopService/RequestBuy")
:InvokeServer(unpack(args))
end)
if not success then
warn("Failed to purchase All Seeing Sentry:", err)
end
end

})

ShopTab:CreateButton({
Name = "Web Slinger",
Callback = function()
local args = {"Web Slinger"}
local success, err = pcall(function()
game:GetService("ReplicatedStorage")
.Packages.Net:FindFirstChild("RF/CoinsShopService/RequestBuy")
:InvokeServer(unpack(args))
end)
if not success then
warn("Failed to purchase All Seeing Sentry:", err)
end
end

})

ShopTab:CreateButton({
Name = "Rainbowrath Sword",
Callback = function()
local args = {"Rainbowrath Sword"}
local success, err = pcall(function()
game:GetService("ReplicatedStorage")
.Packages.Net:FindFirstChild("RF/CoinsShopService/RequestBuy")
:InvokeServer(unpack(args))
end)
if not success then
warn("Failed to purchase All Seeing Sentry:", err)
end
end

})

ShopTab:CreateButton({
Name = "Galaxy Slap",
Callback = function()
local args = {"Galaxy Slap"}
local success, err = pcall(function()
game:GetService("ReplicatedStorage")
.Packages.Net:FindFirstChild("RF/CoinsShopService/RequestBuy")
:InvokeServer(unpack(args))
end)
if not success then
warn("Failed to purchase All Seeing Sentry:", err)
end
end

})

ShopTab:CreateButton({
Name = "Nuclear Slap",
Callback = function()
local args = {"Nuclear Slap"}
local success, err = pcall(function()
game:GetService("ReplicatedStorage")
.Packages.Net:FindFirstChild("RF/CoinsShopService/RequestBuy")
:InvokeServer(unpack(args))
end)
if not success then
warn("Failed to purchase All Seeing Sentry:", err)
end
end

})

ShopTab:CreateButton({
Name = "Dark Matter Slap",
Callback = function()
local args = {"Dark Matter Slap"}
local success, err = pcall(function()
game:GetService("ReplicatedStorage")
.Packages.Net:FindFirstChild("RF/CoinsShopService/RequestBuy")
:InvokeServer(unpack(args))
end)
if not success then
warn("Failed to purchase All Seeing Sentry:", err)
end
end

})

ShopTab:CreateButton({
Name = "Body Swap Potion",
Callback = function()
local args = {"Body Swap Potion"}
local success, err = pcall(function()
game:GetService("ReplicatedStorage")
.Packages.Net:FindFirstChild("RF/CoinsShopService/RequestBuy")
:InvokeServer(unpack(args))
end)
if not success then
warn("Failed to purchase All Seeing Sentry:", err)
end
end

})

ShopTab:CreateButton({
Name = "Splatter Slap",
Callback = function()
local args = {"Splatter Slap"}
local success, err = pcall(function()
game:GetService("ReplicatedStorage")
.Packages.Net:FindFirstChild("RF/CoinsShopService/RequestBuy")
:InvokeServer(unpack(args))
end)
if not success then
warn("Failed to purchase All Seeing Sentry:", err)
end
end

})
