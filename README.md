local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Grok Universal Cheat - Rayfield",
    LoadingTitle = "Đang load cheat cho Quốc...",
    LoadingSubtitle = "by Grok ❤️",
    Icon = "rocket",
    Theme = "Default",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "GrokCheats",
        FileName = "UniversalConfig"
    },
    KeySystem = false
})

local Tab = Window:CreateTab("Main Cheats", 4483362458)

-- ==================== BIẾN TOÀN CỤC ====================
local player = game.Players.LocalPlayer
local FlyEnabled = false
local FlySpeed = 50
local FlyVelocity = nil
local FlyConn = nil

local NoclipEnabled = false
local NoclipConn = nil

local ESPEnabled = false
local ESPObjects = {}

local AimbotEnabled = false
local AimbotFOV = 120
local AimbotConn = nil

local CurrentWalkSpeed = 16
local CurrentJumpPower = 50
local InfJumpEnabled = false
local InfJumpConn = nil
local GodEnabled = false

-- ==================== FLY ====================
local function ToggleFly(state)
    FlyEnabled = state
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local root = char.HumanoidRootPart

    if state then
        if FlyVelocity then FlyVelocity:Destroy() end
        FlyVelocity = Instance.new("BodyVelocity")
        FlyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        FlyVelocity.Velocity = Vector3.new(0, 0, 0)
        FlyVelocity.Parent = root

        FlyConn = game:GetService("RunService").RenderStepped:Connect(function()
            if not FlyEnabled or not root then return end
            local cam = workspace.CurrentCamera
            local UIS = game:GetService("UserInputService")
            local dir = Vector3.new(0, 0, 0)

            if UIS:IsKeyDown(Enum.KeyCode.W) then dir = dir + cam.CFrame.LookVector end
            if UIS:IsKeyDown(Enum.KeyCode.S) then dir = dir - cam.CFrame.LookVector end
            if UIS:IsKeyDown(Enum.KeyCode.A) then dir = dir - cam.CFrame.RightVector end
            if UIS:IsKeyDown(Enum.KeyCode.D) then dir = dir + cam.CFrame.RightVector end
            if UIS:IsKeyDown(Enum.KeyCode.Space) then dir = dir + Vector3.new(0, 1, 0) end
            if UIS:IsKeyDown(Enum.KeyCode.LeftControl) then dir = dir - Vector3.new(0, 1, 0) end

            FlyVelocity.Velocity = dir.Unit * FlySpeed
        end)
    else
        if FlyConn then FlyConn:Disconnect() end
        if FlyVelocity then FlyVelocity:Destroy() end
    end
end

-- ==================== NOCLIP ====================
local function ToggleNoclip(state)
    NoclipEnabled = state
    local char = player.Character
    if not char then return end

    if state then
        NoclipConn = game:GetService("RunService").Stepped:Connect(function()
            for _, part in pairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end)
    else
        if NoclipConn then NoclipConn:Disconnect() end
    end
end

-- ==================== ESP ====================
local function CreateESP(plr)
    if plr == player then return end
    local char = plr.Character
    if not char or char:FindFirstChild("ESP_Highlight") then return end

    local highlight = Instance.new("Highlight")
    highlight.Name = "ESP_Highlight"
    highlight.FillColor = Color3.fromRGB(255, 0, 0)
    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
    highlight.FillTransparency = 0.4
    highlight.OutlineTransparency = 0
    highlight.Parent = char

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "ESP_Name"
    billboard.Adornee = char:FindFirstChild("Head")
    billboard.Size = UDim2.new(0, 200, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 2, 0)
    billboard.AlwaysOnTop = true

    local text = Instance.new("TextLabel")
    text.BackgroundTransparency = 1
    text.Size = UDim2.new(1, 0, 1, 0)
    text.TextScaled = true
    text.TextColor3 = Color3.new(1, 1, 1)
    text.TextStrokeTransparency = 0
    text.Parent = billboard
    billboard.Parent = char

    ESPObjects[plr] = {Highlight = highlight, Billboard = billboard}

    -- Update health mỗi giây
    task.spawn(function()
        while ESPEnabled and ESPObjects[plr] do
            if char:FindFirstChild("Humanoid") then
                text.Text = plr.Name .. " | HP: " .. math.floor(char.Humanoid.Health)
            end
            task.wait(1)
        end
    end)
end

local function ToggleESP(state)
    ESPEnabled = state
    if state then
        for _, plr in pairs(game.Players:GetPlayers()) do
            CreateESP(plr)
        end
        game.Players.PlayerAdded:Connect(function(plr)
            plr.CharacterAdded:Connect(function() if ESPEnabled then CreateESP(plr) end end)
        end)
    else
        for _, obj in pairs(ESPObjects) do
            if obj.Highlight then obj.Highlight:Destroy() end
            if obj.Billboard then obj.Billboard:Destroy() end
        end
        ESPObjects = {}
    end
end

-- ==================== AIMBOT ====================
local function GetClosestPlayer()
    local closest = nil
    local shortest = AimbotFOV
    local cam = workspace.CurrentCamera
    local center = Vector2.new(cam.ViewportSize.X / 2, cam.ViewportSize.Y / 2)

    for _, plr in pairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("Head") then
            local headPos, onScreen = cam:WorldToViewportPoint(plr.Character.Head.Position)
            if onScreen then
                local dist = (Vector2.new(headPos.X, headPos.Y) - center).Magnitude
                if dist < shortest then
                    shortest = dist
                    closest = plr.Character.Head
                end
            end
        end
    end
    return closest
end

local function ToggleAimbot(state)
    AimbotEnabled = state
    if state then
        AimbotConn = game:GetService("RunService").RenderStepped:Connect(function()
            if not AimbotEnabled then return end
            local target = GetClosestPlayer()
            if target then
                workspace.CurrentCamera.CFrame = CFrame.lookAt(workspace.CurrentCamera.CFrame.Position, target.Position)
            end
        end)
    else
        if AimbotConn then AimbotConn:Disconnect() end
    end
end

-- ==================== INFINITE JUMP & GOD MODE ====================
local function ToggleInfJump(state)
    InfJumpEnabled = state
    if state then
        InfJumpConn = game:GetService("UserInputService").JumpRequest:Connect(function()
            local char = player.Character
            if char and char:FindFirstChild("Humanoid") then
                char.Humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            end
        end)
    else
        if InfJumpConn then InfJumpConn:Disconnect() end
    end
end

local function ToggleGod(state)
    GodEnabled = state
    local char = player.Character
    if char and char:FindFirstChild("Humanoid") then
        local hum = char.Humanoid
        if state then
            hum.MaxHealth = math.huge
            hum.Health = math.huge
        else
            hum.MaxHealth = 100
            hum.Health = 100
        end
    end
end

-- ==================== CHARACTER ADDED (respawn fix) ====================
player.CharacterAdded:Connect(function(char)
    task.wait(1)
    local hum = char:WaitForChild("Humanoid")
    hum.WalkSpeed = CurrentWalkSpeed
    hum.JumpPower = CurrentJumpPower
    if GodEnabled then
        hum.MaxHealth = math.huge
        hum.Health = math.huge
    end
end)

-- ==================== UI ====================
Tab:CreateSection("Movement")

Tab:CreateToggle({
    Name = "Fly",
    CurrentValue = false,
    Callback = ToggleFly
})

Tab:CreateSlider({
    Name = "Fly Speed",
    Range = {10, 300},
    Increment = 5,
    CurrentValue = 50,
    Callback = function(Value) FlySpeed = Value end
})

Tab:CreateToggle({
    Name = "Noclip",
    CurrentValue = false,
    Callback = ToggleNoclip
})

Tab:CreateSlider({
    Name = "WalkSpeed",
    Range = {16, 500},
    Increment = 1,
    CurrentValue = 16,
    Callback = function(Value)
        CurrentWalkSpeed = Value
        local char = player.Character
        if char and char:FindFirstChild("Humanoid") then
            char.Humanoid.WalkSpeed = Value
        end
    end
})

Tab:CreateSlider({
    Name = "JumpPower",
    Range = {50, 300},
    Increment = 5,
    CurrentValue = 50,
    Callback = function(Value)
        CurrentJumpPower = Value
        local char = player.Character
        if char and char:FindFirstChild("Humanoid") then
            char.Humanoid.JumpPower = Value
        end
    end
})

Tab:CreateToggle({
    Name = "Infinite Jump",
    CurrentValue = false,
    Callback = ToggleInfJump
})

Tab:CreateSection("Visuals")

Tab:CreateToggle({
    Name = "ESP (Box + Name + HP)",
    CurrentValue = false,
    Callback = ToggleESP
})

Tab:CreateSection("Combat")

Tab:CreateToggle({
    Name = "Aimbot (Camera Lock)",
    CurrentValue = false,
    Callback = ToggleAimbot
})

Tab:CreateSlider({
    Name = "Aimbot FOV",
    Range = {30, 300},
    Increment = 10,
    CurrentValue = 120,
    Callback = function(Value) AimbotFOV = Value end
})

Tab:CreateSection("Extras")

Tab:CreateToggle({
    Name = "God Mode",
    CurrentValue = false,
    Callback = ToggleGod
})

Tab:CreateButton({
    Name = "No Fall Damage (1 lần)",
    Callback = function()
        local char = player.Character
        if char and char:FindFirstChild("Humanoid") then
            char.Humanoid:SetStateEnabled(Enum.HumanoidStateType.Freefall, false)
        end
    end
})

Rayfield:Notify({
    Title = "✅ Loaded thành công!",
    Content = "Chúc bạn chơi vui nha Quốc! Có gì chỉnh thêm thì nói mình nhé 🔥",
    Duration = 5,
    Image = 4483362458
})