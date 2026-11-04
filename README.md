--=============================================================
-- VISUAL ASSIST HUD - Compatível com Delta (mobile) e Xeno (PC)
-- Base: Aimbot original + HUD moderno com botão flutuante
--=============================================================

local Players        = game:GetService("Players")
local RunService     = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Workspace      = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local Mouse       = LocalPlayer:GetMouse()
local Camera      = Workspace.CurrentCamera

--=============================================================
-- CONFIGURAÇÕES (estado dos toggles)
--=============================================================
local Config = {
    WallCheck       = false,
    TeamCheck       = false,
    FriendCheck     = false,
    TargetIndicator = false,
    FOVCircle       = false,
    AimbotActive    = false,  -- ativado pelo botão direito (PC) ou toggle
    FOVRadius       = 150,
}

-- Lista de amigos (adicione UserIds aqui)
local FriendsList = {}  -- ex: [123456789] = true

--=============================================================
-- INTERFACE — ScreenGui principal
--=============================================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "VisualAssistHUD"
ScreenGui.ResetOnSpawn = false
ScreenGui.IgnoreGuiInset = true
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = (gethui and gethui()) or LocalPlayer:WaitForChild("PlayerGui")

--=============================================================
-- BOTÃO FLUTUANTE (arrastável, no centro da tela)
--=============================================================
local FloatingBtn = Instance.new("TextButton")
FloatingBtn.Name = "FloatingBtn"
FloatingBtn.Size = UDim2.new(0, 52, 0, 52)
FloatingBtn.Position = UDim2.new(0.5, -26, 0.5, -26) -- centro
FloatingBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
FloatingBtn.BackgroundTransparency = 0
FloatingBtn.Text = "⦿"
FloatingBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
FloatingBtn.TextSize = 24
FloatingBtn.Font = Enum.Font.GothamBold
FloatingBtn.AutoButtonColor = false
FloatingBtn.BorderSizePixel = 0
FloatingBtn.ZIndex = 9999
FloatingBtn.Parent = ScreenGui

local BtnCorner = Instance.new("UICorner")
BtnCorner.CornerRadius = UDim.new(1, 0)
BtnCorner.Parent = FloatingBtn

local BtnStroke = Instance.new("UIStroke")
BtnStroke.Color = Color3.fromRGB(0, 220, 255)
BtnStroke.Thickness = 2
BtnStroke.Transparency = 0.4
BtnStroke.Parent = FloatingBtn

--=============================================================
-- HUD PANEL (começa oculto)
--=============================================================
local HUD = Instance.new("Frame")
HUD.Name = "HUDPanel"
HUD.Size = UDim2.new(0, 280, 0, 460)
HUD.Position = UDim2.new(0.5, -140, 0.5, -230)
HUD.BackgroundColor3 = Color3.fromRGB(15, 20, 30)
HUD.BackgroundTransparency = 0.1
HUD.BorderSizePixel = 0
HUD.Visible = false
HUD.ZIndex = 9998
HUD.Parent = ScreenGui

local HUDCorner = Instance.new("UICorner")
HUDCorner.CornerRadius = UDim.new(0, 14)
HUDCorner.Parent = HUD

local HUDStroke = Instance.new("UIStroke")
HUDStroke.Color = Color3.fromRGB(0, 200, 255)
HUDStroke.Thickness = 1
HUDStroke.Transparency = 0.6
HUDStroke.Parent = HUD

--=============================================================
-- HEADER
--=============================================================
local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 48)
Header.BackgroundColor3 = Color3.fromRGB(0, 120, 200)
Header.BackgroundTransparency = 0.3
Header.BorderSizePixel = 0
Header.ZIndex = 9999
Header.Parent = HUD

local HeaderCorner = Instance.new("UICorner")
HeaderCorner.CornerRadius = UDim.new(0, 14)
HeaderCorner.Parent = Header

local HeaderTitle = Instance.new("TextLabel")
HeaderTitle.Size = UDim2.new(1, -20, 1, 0)
HeaderTitle.Position = UDim2.new(0, 15, 0, 0)
HeaderTitle.BackgroundTransparency = 1
HeaderTitle.Text = "⚡ VISUAL ASSIST"
HeaderTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
HeaderTitle.TextSize = 15
HeaderTitle.Font = Enum.Font.GothamBold
HeaderTitle.TextXAlignment = Enum.TextXAlignment.Left
HeaderTitle.ZIndex = 9999
HeaderTitle.Parent = Header

--=============================================================
-- FUNÇÃO: criar linha de toggle
--=============================================================
local function createToggle(labelText, yPos, configKey, callback)
    local Row = Instance.new("TextButton")
    Row.Size = UDim2.new(1, -20, 0, 42)
    Row.Position = UDim2.new(0, 10, 0, yPos)
    Row.BackgroundColor3 = Color3.fromRGB(25, 32, 45)
    Row.BackgroundTransparency = 0.4
    Row.BorderSizePixel = 0
    Row.Text = ""
    Row.AutoButtonColor = false
    Row.ZIndex = 9999
    Row.Parent = HUD

    local RowCorner = Instance.new("UICorner")
    RowCorner.CornerRadius = UDim.new(0, 8)
    RowCorner.Parent = Row

    local NameLabel = Instance.new("TextLabel")
    NameLabel.Size = UDim2.new(0.65, 0, 1, 0)
    NameLabel.Position = UDim2.new(0, 12, 0, 0)
    NameLabel.BackgroundTransparency = 1
    NameLabel.Text = labelText
    NameLabel.TextColor3 = Color3.fromRGB(220, 230, 245)
    NameLabel.TextSize = 13
    NameLabel.Font = Enum.Font.GothamMedium
    NameLabel.TextXAlignment = Enum.TextXAlignment.Left
    NameLabel.ZIndex = 9999
    NameLabel.Parent = Row

    -- Switch
    local SwitchBg = Instance.new("Frame")
    SwitchBg.Size = UDim2.new(0, 40, 0, 22)
    SwitchBg.Position = UDim2.new(1, -52, 0.5, -11)
    SwitchBg.BackgroundColor3 = Color3.fromRGB(50, 60, 75)
    SwitchBg.BorderSizePixel = 0
    SwitchBg.ZIndex = 9999
    SwitchBg.Parent = Row

    local SwitchCorner = Instance.new("UICorner")
    SwitchCorner.CornerRadius = UDim.new(1, 0)
    SwitchCorner.Parent = SwitchBg

    local Knob = Instance.new("Frame")
    Knob.Size = UDim2.new(0, 16, 0, 16)
    Knob.Position = UDim2.new(0, 3, 0.5, -8)
    Knob.BackgroundColor3 = Color3.fromRGB(200, 210, 220)
    Knob.BorderSizePixel = 0
    Knob.ZIndex = 10000
    Knob.Parent = SwitchBg

    local KnobCorner = Instance.new("UICorner")
    KnobCorner.CornerRadius = UDim.new(1, 0)
    KnobCorner.Parent = Knob

    local isOn = Config[configKey]

    local function updateVisual()
        if isOn then
            SwitchBg.BackgroundColor3 = Color3.fromRGB(0, 200, 255)
            Knob.Position = UDim2.new(1, -19, 0.5, -8)
            Knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        else
            SwitchBg.BackgroundColor3 = Color3.fromRGB(50, 60, 75)
            Knob.Position = UDim2.new(0, 3, 0.5, -8)
            Knob.BackgroundColor3 = Color3.fromRGB(200, 210, 220)
        end
    end
    updateVisual()

    Row.MouseButton1Click:Connect(function()
        isOn = not isOn
        Config[configKey] = isOn
        updateVisual()
        if callback then callback(isOn) end
    end)
end

--=============================================================
-- CRIAR OS TOGGLES
--=============================================================
createToggle("Wall Check",       58,  "WallCheck")
createToggle("Team Check",       104, "TeamCheck")
createToggle("Friend Check",     150, "FriendCheck")
createToggle("Target Indicator", 196, "TargetIndicator")
createToggle("FOV Circle",       242, "FOVCircle", function(on)
    FOVVisual.Visible = on
end)

--=============================================================
-- FOV CIRCLE VISUAL (Frame + UICorner no centro da tela)
--=============================================================
local FOVVisual = Instance.new("Frame")
FOVVisual.Name = "FOVCircle"
FOVVisual.AnchorPoint = Vector2.new(0.5, 0.5)
FOVVisual.Position = UDim2.new(0.5, 0, 0.5, 0)
FOVVisual.Size = UDim2.new(0, Config.FOVRadius * 2, 0, Config.FOVRadius * 2)
FOVVisual.BackgroundTransparency = 1
FOVVisual.BorderSizePixel = 0
FOVVisual.Visible = false
FOVVisual.ZIndex = 50
FOVVisual.Parent = ScreenGui

local FOVCorner = Instance.new("UICorner")
FOVCorner.CornerRadius = UDim.new(1, 0)
FOVCorner.Parent = FOVVisual

local FOVStroke = Instance.new("UIStroke")
FOVStroke.Color = Color3.fromRGB(0, 212, 255)
FOVStroke.Thickness = 1.5
FOVStroke.Transparency = 0.4
FOVStroke.Parent = FOVVisual

--=============================================================
-- INDICADOR DE ALVO (frame pequeno que segue o alvo)
--=============================================================
local TargetBox = Instance.new("Frame")
TargetBox.Size = UDim2.new(0, 50, 0, 70)
TargetBox.AnchorPoint = Vector2.new(0.5, 0.5)
TargetBox.BackgroundTransparency = 1
TargetBox.BorderSizePixel = 0
TargetBox.Visible = false
TargetBox.ZIndex = 60
TargetBox.Parent = ScreenGui

local TargetStroke = Instance.new("UIStroke")
TargetStroke.Color = Color3.fromRGB(255, 60, 60)
TargetStroke.Thickness = 1.5
TargetStroke.Parent = TargetBox

--=============================================================
-- ARRASTAR DO BOTÃO FLUTUANTE
--=============================================================
local dragging = false
local dragStart, startPos

FloatingBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = FloatingBtn.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
    or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        FloatingBtn.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

--=============================================================
-- CLIQUE NO BOTÃO FLUTUANTE: abre/fecha HUD
-- (só dispara se NÃO arrastou)
--=============================================================
local btnClicked = false
FloatingBtn.MouseButton1Click:Connect(function()
    if dragging then return end
    HUD.Visible = not HUD.Visible
    FloatingBtn.Text = HUD.Visible and "✕" or "⦿"
end)

--=============================================================
-- LÓGICA DO AIMBOT (base do seu script)
--=============================================================
function getClosestPlayer()
    local closestPlayer = nil
    local closestDistance = math.huge
    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return nil end

    for _, target in pairs(Players:GetPlayers()) do
        if target ~= LocalPlayer
        and target.Character
        and target.Character:FindFirstChild("HumanoidRootPart")
        and target.Character:FindFirstChildOfClass("Humanoid")
        and target.Character:FindFirstChildOfClass("Humanoid").Health > 0 then

            -- Team Check: pula aliados
            if Config.TeamCheck and target.Team == LocalPlayer.Team then
                continue
            end

            -- Friend Check: pula amigos
            if Config.FriendCheck and FriendsList[target.UserId] then
                continue
            end

            local hrp = target.Character.HumanoidRootPart
            local dist = (hrp.Position - myChar.HumanoidRootPart.Position).Magnitude

            if dist < closestDistance then
                closestDistance = dist
                closestPlayer = target
            end
        end
    end

    return closestPlayer
end

-- Verifica se há parede entre LocalPlayer e target
local function isVisible(target)
    if not Config.WallCheck then return true end
    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("Head") then return false end
    local targetHRP = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
    if not targetHRP then return false end

    local origin = Camera.CFrame.Position
    local direction = (targetHRP.Position - origin)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {myChar, target.Character, Camera}

    local result = Workspace:Raycast(origin, direction, params)
    return result == nil
end

function aimAt(target)
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, target.Character.HumanoidRootPart.Position)
    end
end

-- Botão direito do mouse (PC)
Mouse.Button2Down:Connect(function() Config.AimbotActive = true  end)
Mouse.Button2Up:Connect(function()   Config.AimbotActive = false end)

--=============================================================
-- LOOP PRINCIPAL
--=============================================================
RunService.RenderStepped:Connect(function()
    -- Aimbot
    if Config.AimbotActive then
        local closest = getClosestPlayer()
        if closest and isVisible(closest) then
            aimAt(closest)
        end
    end

    -- Target Indicator (mostra quadrado no alvo mais próximo)
    if Config.TargetIndicator then
        local closest = getClosestPlayer()
        if closest and closest.Character and closest.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = closest.Character.HumanoidRootPart
            local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
            if onScreen then
                TargetBox.Position = UDim2.new(0, screenPos.X, 0, screenPos.Y)
                TargetBox.Visible = true
                -- cor diferente se não estiver visível (wall check)
                if Config.WallCheck and not isVisible(closest) then
                    TargetStroke.Color = Color3.fromRGB(255, 200, 0) -- amarelo = atrás de parede
                else
                    TargetStroke.Color = Color3.fromRGB(255, 60, 60) -- vermelho = visível
                end
            else
                TargetBox.Visible = false
            end
        else
            TargetBox.Visible = false
        end
    else
        TargetBox.Visible = false
    end

    -- Atualiza FOV visual conforme slider (se quiser adicionar slider depois)
    if Config.FOVCircle then
        FOVVisual.Size = UDim2.new(0, Config.FOVRadius * 2, 0, Config.FOVRadius * 2)
    end
end)

print("[Visual Assist] HUD carregado — botão flutuante no centro da tela.")
