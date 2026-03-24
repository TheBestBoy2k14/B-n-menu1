local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
    Name = "Grok Cheat - Rút Gọn",
    LoadingTitle = "Loading cho Quốc...",
    LoadingSubtitle = "by Grok",
    Icon = "rocket",
    Theme = "Default",
    ConfigurationSaving = { Enabled = true, FolderName = "GrokCheats", FileName = "Universal" },
    KeySystem = false
})

local Tab = Window:CreateTab("Cheats", 4483362458)

local player = game.Players.LocalPlayer
local FlySpeed = 50
local AimbotFOV = 120
local ESPObjects = {}

-- Fly
local function ToggleFly(state)
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local root = char.HumanoidRootPart
    if state then
        local bv = Instance.new("BodyVelocity")
        bv.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        bv.Parent = root
        game:GetService("RunService").RenderStepped:Connect(function()
            if not state then bv:Destroy() return end
            local cam = workspace.CurrentCamera
            local dir = Vector3.new()
            local uis = game:GetService("UserInputService")
            if uis:IsKeyDown(Enum.KeyCode.W) then dir += cam.CFrame.LookVector end
            if uis:IsKeyDown(Enum.KeyCode.S) then dir -= cam.CFrame.LookVector end
            if uis:IsKeyDown(Enum.KeyCode.A) then dir -= cam.CFrame.RightVector end
            if uis:IsKeyDown(Enum.KeyCode.D) then dir += cam.CFrame.RightVector end
            if uis:IsKeyDown(Enum.KeyCode.Space) then dir += Vector3.new(0,1,0) end
            if uis:IsKeyDown(Enum.KeyCode.LeftControl) then dir -= Vector3.new(0,1,0) end
            bv.Velocity = dir.Unit * FlySpeed
        end)
    end
end

-- Noclip
local function ToggleNoclip(state)
    if state then
        game:GetService("RunService").Stepped:Connect(function()
            for _,v in pairs(player.Character:GetDescendants()) do
                if v:IsA("BasePart") then v.CanCollide = false end
            end
        end)
    end
end

-- ESP đơn giản
local function ToggleESP(state)
    if state then
        for _,p in pairs(game.Players:GetPlayers()) do
            if p ~= player and p.Character and p.Character:FindFirstChild("Head") then
                local h = Instance.new("Highlight", p.Character)
                h.FillColor = Color3.new(1,0,0)
                h.OutlineColor = Color3.new(1,1,1)
                h.FillTransparency = 0.5
            end
        end
    else
        for _,p in pairs(game.Players:GetPlayers()) do
            if p.Character then
                for _,obj in pairs(p.Character:GetChildren()) do
                    if obj:IsA("Highlight") then obj:Destroy() end
                end
            end
        end
    end
end

-- Aimbot
local function ToggleAimbot(state)
    if state then
        game:GetService("RunService").RenderStepped:Connect(function()
            if not state then return end
            local closest, dist = nil, AimbotFOV
            local cam = workspace.CurrentCamera
            local center = cam.ViewportSize/2
            for _,p in pairs(game.Players:GetPlayers()) do
                if p ~= player and p.Character and p.Character:FindFirstChild("Head") then
                    local pos, onScreen = cam:WorldToViewportPoint(p.Character.Head.Position)
                    if onScreen then
                        local d = (Vector2.new(pos.X, pos.Y) - center).Magnitude
                        if d < dist then dist = d closest = p.Character.Head end
                    end
                end
            end
            if closest then
                cam.CFrame = CFrame.lookAt(cam.CFrame.Position, closest.Position)
            end
        end)
    end
end

-- Admin Commands
local function AddAdminCommands()
    local cmds = {
        ["kill"] = function(target)
            if target and target.Character and target.Character:FindFirstChild("Humanoid") then
                target.Character.Humanoid.Health = 0
            end
        end,
        ["tp"] = function(target)
            if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                player.Character.HumanoidRootPart.CFrame = target.Character.HumanoidRootPart.CFrame
            end
        end,
        ["bring"] = function(target)
            if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                target.Character.HumanoidRootPart.CFrame = player.Character.HumanoidRootPart.CFrame + Vector3.new(0,0,5)
            end
        end,
        ["god"] = function(target)
            if target and target.Character and target.Character:FindFirstChild("Humanoid") then
                target.Character.Humanoid.MaxHealth = 9e9
                target.Character.Humanoid.Health = 9e9
            end
        end,
        ["speed"] = function(target, num)
            if target and target.Character and target.Character:FindFirstChild("Humanoid") and tonumber(num) then
                target.Character.Humanoid.WalkSpeed = tonumber(num)
            end
        end
    }

    game.Players.LocalPlayer.Chatted:Connect(function(msg)
        local args = msg:split(" ")
        local cmd = args[1]:lower():sub(2) -- bỏ dấu /
        if cmds[cmd] then
            local targetName = args[2]
            local target = game.Players:FindFirstChild(targetName) or game.Players.LocalPlayer
            if cmd == "speed" then
                cmds[cmd](target, args[3])
            else
                cmds[cmd](target)
            end
        end
    end)
end

AddAdminCommands()

-- ==================== UI ====================
Tab:CreateSection("Movement")
Tab:CreateToggle({Name = "Fly", CurrentValue = false, Callback = ToggleFly})
Tab:CreateSlider({Name = "Fly Speed", Range = {10, 300}, Increment = 5, CurrentValue = 50, Callback = function(v) FlySpeed = v end})
Tab:CreateToggle({Name = "Noclip", CurrentValue = false, Callback = ToggleNoclip})
Tab:CreateSlider({Name = "WalkSpeed", Range = {16, 500}, Increment = 1, CurrentValue = 16, Callback = function(v)
    if player.Character and player.Character:FindFirstChild("Humanoid") then player.Character.Humanoid.WalkSpeed = v end
end})

Tab:CreateSection("Visual + Combat")
Tab:CreateToggle({Name = "ESP", CurrentValue = false, Callback = ToggleESP})
Tab:CreateToggle({Name = "Aimbot", CurrentValue = false, Callback = ToggleAimbot})
Tab:CreateSlider({Name = "Aimbot FOV", Range = {30, 300}, Increment = 10, CurrentValue = 120, Callback = function(v) AimbotFOV = v end})

Tab:CreateSection("Extras")
Tab:CreateButton({Name = "Infinite Jump (Toggle bằng phím Space)", Callback = function()
    game:GetService("UserInputService").JumpRequest:Connect(function()
        if player.Character and player.Character:FindFirstChild("Humanoid") then
            player.Character.Humanoid:ChangeState("Jumping")
        end
    end)
end})

Rayfield:Notify({Title = "✅ Script rút gọn loaded!", Content = "Dùng lệnh chat: /kill me, /tp playername, /bring playername, /god, /speed 100\nNhấn K mở menu", Duration = 8})