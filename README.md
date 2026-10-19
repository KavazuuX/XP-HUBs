# XP-HUBs

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local aimbotAtivo = false
local distanciaMaxima = 150

local function encontrarAlvo()
    local alvo = nil
    local menorDistancia = distanciaMaxima

    for _, outro in ipairs(Players:GetPlayers()) do
        if outro ~= player and outro.Character then
            local humanoid = outro.Character:FindFirstChildOfClass("Humanoid")
            local cabeca = outro.Character:FindFirstChild("Head")

            if humanoid and humanoid.Health > 0 and cabeca then
                local distancia = (camera.CFrame.Position - cabeca.Position).Magnitude

                if distancia < menorDistancia then
                    menorDistancia = distancia
                    alvo = cabeca
                end
            end
        end
    end

    return alvo
end

UserInputService.InputBegan:Connect(function(input, processado)
    if processado then return end

    if input.KeyCode == Enum.KeyCode.E then
        aimbotAtivo = not aimbotAtivo
        print("Aimbot:", aimbotAtivo)
    end
end)

RunService:BindToRenderStep("AimbotCamera", Enum.RenderPriority.Camera.Value + 1, function()
    if not aimbotAtivo then return end

    local alvo = encontrarAlvo()

    if alvo then
        camera.CFrame = CFrame.lookAt(camera.CFrame.Position, alvo.Position)
    end
end)
