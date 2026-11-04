--// Visual Assist HUD - Single Script for Delta & Xeno
--// Compatível com executores que suportam Drawing API e serviços básicos.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera

--// Configurações
local Config = {
    FOVCircle = true,
    FOVRadius = 150,
    TeamCheck = true,
    WallCheck = true,
    ESP = true,
}

--// Verifica se a Drawing API está disponível (comum no Xeno, limitada no Delta mobile)
local hasDrawing = pcall(function() return Drawing.new("Circle") end)

--// Variáveis de desenho
local FOVCircle, ESPObjects = nil, {}

--// Função para criar o FOV Circle
local function createFOVCircle()
    if not hasDrawing or not Config.FOVCircle then return end
    if FOVCircle then FOVCircle:Remove() end
    
    FOVCircle = Drawing.new("Circle")
    FOVCircle.Thickness = 1.5
    FOVCircle.Color = Color3.fromRGB(0, 212, 255)
    FOVCircle.Transparency = 0.7
    FOVCircle.Radius = Config.FOVRadius
    FOVCircle.Filled = false
    FOVCircle.Visible = true
end

--// Função para verificar se há parede entre o jogador e o alvo
local function isVisible(target)
    if not Config.WallCheck then return true end
    local origin = Camera.CFrame.Position
    local direction = (target.Position - origin)
    local rayParams = RaycastParams.new()
    rayParams.FilterType = Enum.RaycastFilterType.Exclude
    rayParams.FilterDescendantsInstances = {LocalPlayer.Character, target.Parent}
    
    local result = Workspace:Raycast(origin, direction, rayParams)
    return result == nil
end

--// Função para desenhar o ESP de um jogador
local function updateESP(player)
    if not hasDrawing or not Config.ESP or player == LocalPlayer then return end
    local char = player.Character
    if not char then return end
    
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if not hrp or not humanoid or humanoid.Health <= 0 then return end
    
    -- Team Check
    if Config.TeamCheck and player.Team == LocalPlayer.Team then return end
    
    -- Wall Check
    if Config.WallCheck and not isVisible(hrp) then return end
    
    local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
    
    if onScreen then
        -- Cria ou atualiza o objeto de desenho
        if not ESPObjects[player] then
            local box = Drawing.new("Square")
            box.Thickness = 1
            box.Color = Color3.fromRGB(255, 60, 60) -- Vermelho para inimigos
            box.Filled = false
            ESPObjects[player] = { box = box }
        end
        
        local box = ESPObjects[player].box
        box.Size = Vector2.new(40, 60) -- Tamanho fixo simplificado
        box.Position = Vector2.new(screenPos.X - 20, screenPos.Y - 30)
        box.Visible = true
    else
        if ESPObjects[player] and ESPObjects[player].box then
            ESPObjects[player].box.Visible = false
        end
    end
end

--// Loop de atualização
RunService.RenderStepped:Connect(function()
    if hasDrawing and Config.FOVCircle and FOVCircle then
        FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
        FOVCircle.Radius = Config.FOVRadius
    end
    
    if Config.ESP then
        for _, player in ipairs(Players:GetPlayers()) do
            updateESP(player)
        end
    end
end)

--// Inicialização
createFOVCircle()

--// Limpeza ao remover jogadores
Players.PlayerRemoving:Connect(function(player)
    if ESPObjects[player] and ESPObjects[player].box then
        ESPObjects[player].box:Remove()
        ESPObjects[player] = nil
    end
end)

print("[Visual Assist] Script carregado com sucesso!")
