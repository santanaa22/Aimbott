local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local Teams = game:GetService("Teams")
local UserInputService = game:GetService("UserInputService")

-- Funções de ativação
local ENABLE_ESP = false
local ENABLE_HITBOX = false
local ENABLE_AIMBOT = false

-- HUD Setup
local ScreenGui = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
ScreenGui.ResetOnSpawn = false
local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0, 220, 0, 200)
Frame.Position = UDim2.new(0, 30, 0, 100)
Frame.BackgroundColor3 = Color3.fromRGB(40,40,40)
Frame.Visible = true

local function makeButton(name, pos, callback)
    local btn = Instance.new("TextButton", Frame)
    btn.Size = UDim2.new(0, 200, 0, 40)
    btn.Position = UDim2.new(0, 10, 0, pos)
    btn.Text = name
    btn.BackgroundColor3 = Color3.fromRGB(60,60,60)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.MouseButton1Click:Connect(callback)
    return btn
end

local EspBtn = makeButton("ESP: OFF", 10, function()
    ENABLE_ESP = not ENABLE_ESP
    EspBtn.Text = "ESP: " .. (ENABLE_ESP and "ON" or "OFF")
end)
local HitboxBtn = makeButton("Hitbox: OFF", 60, function()
    ENABLE_HITBOX = not ENABLE_HITBOX
    HitboxBtn.Text = "Hitbox: " .. (ENABLE_HITBOX and "ON" or "OFF")
end)
local AimbotBtn = makeButton("Aimbot: OFF", 110, function()
    ENABLE_AIMBOT = not ENABLE_AIMBOT
    AimbotBtn.Text = "Aimbot: " .. (ENABLE_AIMBOT and "ON" or "OFF")
end)
local ToggleHudBtn = makeButton("Fechar HUD", 160, function()
    Frame.Visible = not Frame.Visible
    ToggleHudBtn.Text = Frame.Visible and "Fechar HUD" or "Abrir HUD"
end)

-- ESP
local espBoxes = {}
local function clearESP()
    for _,v in pairs(espBoxes) do v:Destroy() end
    espBoxes = {}
end

RunService.RenderStepped:Connect(function()
    clearESP()
    if ENABLE_ESP then
        for _,plr in pairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                -- Team check
                if not plr.Team or not LocalPlayer.Team or plr.Team ~= LocalPlayer.Team then
                    local box = Instance.new("BoxHandleAdornment")
                    box.Adornee = plr.Character.HumanoidRootPart
                    box.Size = Vector3.new(4,7,2)
                    box.Color3 = Color3.fromRGB(255,0,0)
                    box.Transparency = 0.7
                    box.AlwaysOnTop = true
                    box.Parent = workspace
                    table.insert(espBoxes, box)
                end
            end
        end
    end
end)

-- Hitbox
local function updateHitboxes()
    for _,plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            if ENABLE_HITBOX then
                plr.Character.HumanoidRootPart.Size = Vector3.new(7,7,7)
                plr.Character.HumanoidRootPart.Transparency = 0.5
            else
                plr.Character.HumanoidRootPart.Size = Vector3.new(2,2,1)
                plr.Character.HumanoidRootPart.Transparency = 1
            end
        end
    end
end
Players.PlayerAdded:Connect(function(p) p.CharacterAdded:Connect(updateHitboxes) end)
RunService.RenderStepped:Connect(updateHitboxes)

-- Simples Aimbot (mira na cabeça do inimigo mais próximo ao mouse)
local function getClosestEnemy()
    local mouse = LocalPlayer:GetMouse()
    local closest, dist = nil, math.huge
    for _,plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
            if not plr.Team or not LocalPlayer.Team or plr.Team ~= LocalPlayer.Team then
                local pos = workspace.CurrentCamera:WorldToViewportPoint(plr.Character.Head.Position)
                local mag = (Vector2.new(pos.X,pos.Y) - Vector2.new(mouse.X,mouse.Y)).Magnitude
                if mag < dist and mag < 300 then
                    dist = mag
                    closest = plr.Character.Head
                end
            end
        end
    end
    return closest
end

UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 and ENABLE_AIMBOT then
        local target = getClosestEnemy()
        if target then
            workspace.CurrentCamera.CFrame = CFrame.new(workspace.CurrentCamera.CFrame.Position, target.Position)
        end
    end
end)

-- HUD open/close by clicking no botão
-- (já implementado no ToggleHudBtn)

-- Ativação inicial dos botões
EspBtn.Text = "ESP: OFF"
HitboxBtn.Text = "Hitbox: OFF"
AimbotBtn.Text = "Aimbot: OFF"
ToggleHudBtn.Text = "Fechar HUD"
