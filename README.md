-- Arena de Tiro Script | CS:GO Style Hub
-- Full Features - Pentest Autorizado

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local rootpart = character:WaitForChild("HumanoidRootPart")
local camera = Workspace.CurrentCamera

-- Config Hub
getgenv().ArenaConfig = {
    Aimbot = false,
    TriggerBot = false,
    ESP = true,
    WallHack = true,
    BunnyHop = false,
    NoRecoil = true,
    Speed = 50,
    FOV = 120,
    AimSmooth = 0.1,
    KillAura = false,
    AutoFire = true
}

-- Aimbot System
local function getClosestEnemy()
    local closest, shortest = nil, getgenv().ArenaConfig.FOV
    for _, v in pairs(Players:GetPlayers()) do
        if v ~= player and v.Character and v.Character:FindFirstChild("Head") then
            local enemyHead = v.Character.Head
            local screenPos, onScreen = camera:WorldToScreenPoint(enemyHead.Position)
            local centerScreen = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
            local distance = (Vector2.new(screenPos.X, screenPos.Y) - centerScreen).Magnitude
            
            if onScreen and distance < shortest then
                closest = v
                shortest = distance
            end
        end
    end
    return closest
end

-- Smooth Aimbot
RunService.Heartbeat:Connect(function()
    if getgenv().ArenaConfig.Aimbot then
        local target = getClosestEnemy()
        if target and target.Character then
            local headPos = target.Character.Head.Position
            local targetCFrame = CFrame.lookAt(rootpart.Position, headPos)
            camera.CFrame = camera.CFrame:Lerp(targetCFrame, getgenv().ArenaConfig.AimSmooth)
        end
    end
end)

-- TriggerBot
local mouse = player:GetMouse()
mouse.Button1Down:Connect(function()
    if getgenv().ArenaConfig.TriggerBot then
        local target = getClosestEnemy()
        if target then
            mouse1click() -- Auto shoot
        end
    end
end)

-- BunnyHop
local bhoping = false
RunService.Stepped:Connect(function()
    if getgenv().ArenaConfig.BunnyHop and humanoid then
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
            humanoid.WalkSpeed = 60
        else
            humanoid.WalkSpeed = getgenv().ArenaConfig.Speed
        end
    end
end)

-- NoRecoil (Weapon Stabilizer)
local lastShot = tick()
RunService.Heartbeat:Connect(function()
    if getgenv().ArenaConfig.NoRecoil and tick() - lastShot > 0.1 then
        -- Simula recoil compensation
        local recoilComp = Vector3.new(math.random(-2,2), -1, 0)
        camera.CFrame = camera.CFrame * CFrame.new(recoilComp * 0.01)
    end
end)

-- ESP + WallHack
for _, v in pairs(Players:GetPlayers()) do
    if v ~= player then
        spawn(function()
            while v.Parent do
                if v.Character then
                    for _, part in pairs(v.Character:GetChildren()) do
                        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
                            part.Transparency = getgenv().ArenaConfig.WallHack and 0.3 or 1
                            part.CanCollide = false
                        end
                    end
                end
                wait(0.1)
            end
        end)
    end
end

-- KillAura
spawn(function()
    while true do
        if getgenv().ArenaConfig.KillAura then
            local target = getClosestEnemy()
            if target and target.Character then
                -- Headshot simulada
                local args = {
                    [1] = target.Character.Head,
                    [2] = 100, -- Damage
                    [3] = rootpart
                }
                pcall(function()
                    game.ReplicatedStorage.Remotes.Shoot:FireServer(unpack(args))
                    game.ReplicatedStorage.Events.Damage:FireServer(unpack(args))
                end)
            end
        end
        wait(0.05)
    end
end)

-- Controles (Teclas)
UserInputService.InputBegan:Connect(function(key, processed)
    if processed then return end
    
    if key.KeyCode == Enum.KeyCode.Insert then
        getgenv().ArenaConfig.Aimbot = not getgenv().ArenaConfig.Aimbot
        print("Aimbot: " .. (getgenv().ArenaConfig.Aimbot and "ON" or "OFF"))
        
    elseif key.KeyCode == Enum.KeyCode.Delete then
        getgenv().ArenaConfig.TriggerBot = not getgenv().ArenaConfig.TriggerBot
        print("TriggerBot: " .. (getgenv().ArenaConfig.TriggerBot and "ON" or "OFF"))
        
    elseif key.KeyCode == Enum.KeyCode.Home then
        getgenv().ArenaConfig.BunnyHop = not getgenv().ArenaConfig.BunnyHop
        print("BunnyHop: " .. (getgenv().ArenaConfig.BunnyHop and "ON" or "OFF"))
        
    elseif key.KeyCode == Enum.KeyCode.End then
        getgenv().ArenaConfig.KillAura = not getgenv().ArenaConfig.KillAura
        print("KillAura: " .. (getgenv().ArenaConfig.KillAura and "ON" or "OFF"))
    end
end)

-- Status HUD
local ScreenGui = Instance.new("ScreenGui")
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 250, 0, 150)
Frame.Position = UDim2.new(0, 10, 0, 10)
Frame.BackgroundColor3 = Color3.new(0, 0, 0)
Frame.BackgroundTransparency = 0.3
Frame.Parent = ScreenGui
ScreenGui.Parent = player.PlayerGui

local function updateHUD()
    Frame:ClearAllChildren()
    local y = 10
    for k, v in pairs(getgenv().ArenaConfig) do
        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, 0, 0, 25)
        label.Position = UDim2.new(0, 0, 0, y)
        label.Text = k .. ": " .. (type(v) == "boolean" and (v and "ON" or "OFF") or tostring(v))
        label.BackgroundTransparency = 1
        label.TextColor3 = v and Color3.new(0,1,0) or Color3.new(1,0,0)
        label.TextScaled = true
        label.Parent = Frame
        y = y + 30
    end
end

spawn(function()
    while true do
        updateHUD()
        wait(0.5)
    end
end)

-- Auto respawn
player.CharacterAdded:Connect(function(char)
    character = char
    humanoid = char:WaitForChild("Humanoid")
    rootpart = char:WaitForChild("HumanoidRootPart")
    humanoid.WalkSpeed = getgenv().ArenaConfig.Speed
end)

humanoid.WalkSpeed = getgenv().ArenaConfig.Speed

print("=== ARENA DE TIRO SCRIPT LOADED ===")
print("INSERT: Aimbot | DELETE: TriggerBot")
print("HOME: BunnyHop | END: KillAura")
print("ESP/Wallhack sempre ON | HUD no canto!")
