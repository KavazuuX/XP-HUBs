# XP-HUBs


local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

local Config = {
    Aimbot = false,
    ESP = false,
    TeamCheck = true,
    FOV = 150,
    Smooth = 0.15
}

local GUI = Instance.new("ScreenGui")
GUI.Name = "UniversalAimESP"
GUI.ResetOnSpawn = false
GUI.Parent = LocalPlayer:WaitForChild("PlayerGui")

local Frame = Instance.new("Frame")
Frame.Size = UDim2.fromOffset(220, 230)
Frame.Position = UDim2.new(0, 20, 0.5, -115)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
Frame.Parent = GUI

Instance.new("UICorner", Frame).CornerRadius =
    UDim.new(0, 10)

local function criarBotao(texto, y, callback)
    local botao = Instance.new("TextButton")
    botao.Size = UDim2.fromOffset(200, 35)
    botao.Position = UDim2.fromOffset(10, y)
    botao.BackgroundColor3 = Color3.fromRGB(55, 55, 75)
    botao.TextColor3 = Color3.new(1, 1, 1)
    botao.TextSize = 14
    botao.Text = texto
    botao.Parent = Frame

    Instance.new("UICorner", botao).CornerRadius =
        UDim.new(0, 6)

    botao.MouseButton1Click:Connect(function()
        callback(botao)
    end)
end

criarBotao("Aimbot: OFF", 15, function(b)
    Config.Aimbot = not Config.Aimbot
    b.Text = "Aimbot: " .. (Config.Aimbot and "ON" or "OFF")
end)

criarBotao("ESP: OFF", 55, function(b)
    Config.ESP = not Config.ESP
    b.Text = "ESP: " .. (Config.ESP and "ON" or "OFF")
end)

criarBotao("Team Check: ON", 95, function(b)
    Config.TeamCheck = not Config.TeamCheck
    b.Text = "Team Check: " ..
        (Config.TeamCheck and "ON" or "OFF")
end)

criarBotao("FOV: 150", 135, function(b)
    Config.FOV = Config.FOV + 50
    if Config.FOV > 350 then Config.FOV = 100 end
    b.Text = "FOV: " .. Config.FOV
end)

criarBotao("Smooth: 0.15", 175, function(b)
    Config.Smooth = Config.Smooth + 0.1
    if Config.Smooth > 0.95 then
        Config.Smooth = 0.05
    end
    b.Text = string.format("Smooth: %.2f", Config.Smooth)
end)

local function inimigo(p)
    if p == LocalPlayer then return false end
    if Config.TeamCheck and p.Team ~= nil
        and p.Team == LocalPlayer.Team then
        return false
    end
    return true
end

local function pegarAlvo()
    local alvo
    local menor = Config.FOV
    local centro = Camera.ViewportSize / 2

    for _, p in ipairs(Players:GetPlayers()) do
        if inimigo(p) and p.Character then
            local head = p.Character:FindFirstChild("Head")
            local hum = p.Character:FindFirstChildOfClass("Humanoid")

            if head and hum and hum.Health > 0 then
                local pos, visivel =
                    Camera:WorldToViewportPoint(head.Position)

                if visivel then
                    local dist =
                        (Vector2.new(pos.X, pos.Y) - centro).Magnitude

                    if dist < menor then
                        menor = dist
                        alvo = head
                    end
                end
            end
        end
    end

    return alvo
end

local ESPObjects = {}

local function criarESP(p)
    if p == LocalPlayer or ESPObjects[p] then return end

    local label = Instance.new("TextLabel")
    label.Size = UDim2.fromOffset(180, 35)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(255, 70, 70)
    label.TextStrokeTransparency = 0
    label.TextSize = 14
    label.Visible = false
    label.Parent = GUI

    ESPObjects[p] = label
end

for _, p in ipairs(Players:GetPlayers()) do
    criarESP(p)
end

Players.PlayerAdded:Connect(criarESP)

Players.PlayerRemoving:Connect(function(p)
    if ESPObjects[p] then
        ESPObjects[p]:Destroy()
        ESPObjects[p] = nil
    end
end)

RunService.RenderStepped:Connect(function()
    Camera = workspace.CurrentCamera
    if not Camera then return end

    if Config.Aimbot then
        local alvo = pegarAlvo()
        if alvo then
            local destino = CFrame.lookAt(
                Camera.CFrame.Position,
                alvo.Position
            )
            Camera.CFrame =
                Camera.CFrame:Lerp(destino, Config.Smooth)
        end
    end

    for p, label in pairs(ESPObjects) do
        label.Visible = false

        if Config.ESP and inimigo(p) and p.Character then
            local root =
                p.Character:FindFirstChild("HumanoidRootPart")
            local hum =
                p.Character:FindFirstChildOfClass("Humanoid")

            if root and hum and hum.Health > 0 then
                local pos, visivel =
                    Camera:WorldToViewportPoint(root.Position)

                if visivel then
                    local distancia = (
                        Camera.CFrame.Position - root.Position
                    ).Magnitude

                    label.Position = UDim2.fromOffset(
                        pos.X - 90,
                        pos.Y - 55
                    )
                    label.Text = string.format(
                        "%s | %.0f studs",
                        p.DisplayName,
                        distancia
                    )
                    label.Visible = true
                end
            end
        end
    end
end)
