local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = game.Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Criar ESP para jogadores
local function createESP(player)
    if player.Character then
        for _, part in pairs(player.Character:GetChildren()) do
            if part:IsA("BasePart") then
                local highlight = Instance.new("BoxHandleAdornment")
                highlight.Size = part.Size + Vector3.new(0.1, 0.1, 0.1)
                highlight.Adornee = part
                highlight.AlwaysOnTop = true
                highlight.ZIndex = 5

                -- Definir cor com base no time
                if player.Team == LocalPlayer.Team then
                    highlight.Color3 = Color3.fromRGB(0, 0, 255) -- Azul (Amigos)
                else
                    highlight.Color3 = Color3.fromRGB(255, 0, 0) -- Vermelho (Inimigos)
                end

                highlight.Parent = game.CoreGui
            end
        end
    end
end

-- Aplicar ESP a todos os jogadores
local function applyESP()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            createESP(player)
        end
    end
end

Players.PlayerAdded:Connect(applyESP)
applyESP()

-- Aimbot que gruda na cabeça do inimigo ao segurar botão direito do mouse
local function getClosestEnemy()
    local closestEnemy = nil
    local shortestDistance = math.huge

    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            local screenPosition, onScreen = Camera:WorldToViewportPoint(head.Position)

            if onScreen then
                local mousePos = UserInputService:GetMouseLocation()
                local distance = (Vector2.new(screenPosition.X, screenPosition.Y) - mousePos).magnitude

                if distance < shortestDistance then
                    shortestDistance = distance
                    closestEnemy = head
                end
            end
        end
    end
    return closestEnemy
end

UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        local target = getClosestEnemy()
        if target then
            RunService.RenderStepped:Connect(function()
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Position)
            end)
        end
    end
end)
