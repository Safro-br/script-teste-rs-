-- [[ Rscripts Risk Notice ]]
-- This script is not verified by rscripts.net. Deal with caution.
--
-- Stay safe:
--   • Never log in on unofficial Roblox sites or lookalike domains.
--   • Real Roblox links use roblox.com (check the .com ending).
--   • Treat fake Roblox login / "claim reward" pages as phishing.
-- [[ End Rscripts Risk Notice ]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local TweenService = game:GetService("TweenService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local Settings = {
	Aimbot = false,
	Triggerbot = false,
	ESP = false,
	WallCheck = false,
	NoRecoil = false,
	Smoothness = 0.12,
	LockFOV = 100,
	TriggerFOV = 16,
	TriggerDelay = 0.07,
	Prediction = 0.12,
	TargetPart = "Head",
	ESPMaxDistance = 1500,
}

local Enabled = false
local LockedTarget = nil
local LastTrigger = 0
local HoldingClick = false
local ESPObjects = {}
local MenuOpen = true

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "date"
ScreenGui.ResetOnSpawn = false
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.Parent = game:GetService("CoreGui")

local Main = Instance.new("Frame")
Main.Name = "Main"
Main.Size = UDim2.new(0, 260, 0, 380)
Main.Position = UDim2.new(0, 40, 0.5, -190)
Main.BackgroundColor3 = Color3.fromRGB(18, 18, 22)
Main.BorderSizePixel = 0
Main.Active = true
Main.Draggable = true
Main.Parent = ScreenGui

local UICorner = Instance.new("UICorner")
UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = Main

local UIStroke = Instance.new("UIStroke")
UIStroke.Color = Color3.fromRGB(45, 45, 55)
UIStroke.Thickness = 1
UIStroke.Parent = Main

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 36)
Title.BackgroundColor3 = Color3.fromRGB(25, 25, 32)
Title.BorderSizePixel = 0
Title.Text = "date"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Parent = Main

local TitleCorner = Instance.new("UICorner")
TitleCorner.CornerRadius = UDim.new(0, 8)
TitleCorner.Parent = Title

local TitleFix = Instance.new("Frame")
TitleFix.Size = UDim2.new(1, 0, 0, 12)
TitleFix.Position = UDim2.new(0, 0, 1, -12)
TitleFix.BackgroundColor3 = Color3.fromRGB(25, 25, 32)
TitleFix.BorderSizePixel = 0
TitleFix.Parent = Title

local Container = Instance.new("Frame")
Container.Size = UDim2.new(1, -20, 1, -50)
Container.Position = UDim2.new(0, 10, 0, 44)
Container.BackgroundTransparency = 1
Container.Parent = Main

local Layout = Instance.new("UIListLayout")
Layout.Padding = UDim.new(0, 8)
Layout.Parent = Container

local OpenPos = UDim2.new(0, 40, 0.5, -190)
local ClosedPos = UDim2.new(0, -280, 0.5, -190)

local function setMenuVisible(visible)
	MenuOpen = visible
	local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)

	if visible then
		Main.Visible = true
		Main.BackgroundTransparency = 0
		UIStroke.Transparency = 0
		Title.BackgroundTransparency = 0
		Title.TextTransparency = 0
		TitleFix.BackgroundTransparency = 0

		for _, child in Container:GetChildren() do
			if child:IsA("Frame") then
				child.BackgroundTransparency = 0
				for _, desc in child:GetDescendants() do
					if desc:IsA("TextLabel") then
						desc.TextTransparency = 0
					elseif desc:IsA("TextButton") then
						desc.BackgroundTransparency = 0
					elseif desc:IsA("Frame") then
						desc.BackgroundTransparency = 0
					end
				end
			elseif child:IsA("TextLabel") then
				child.BackgroundTransparency = 0
				child.TextTransparency = 0
			end
		end

		TweenService:Create(Main, tweenInfo, {Position = OpenPos}):Play()
	else
		local t = TweenService:Create(Main, tweenInfo, {Position = ClosedPos})
		t:Play()
		t.Completed:Connect(function()
			if not MenuOpen then
				Main.Visible = false
			end
		end)
	end
end

local function createToggle(name, default, callback)
	local holder = Instance.new("Frame")
	holder.Size = UDim2.new(1, 0, 0, 32)
	holder.BackgroundColor3 = Color3.fromRGB(28, 28, 36)
	holder.BorderSizePixel = 0
	holder.Parent = Container

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 6)
	corner.Parent = holder

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1, -50, 1, 0)
	label.Position = UDim2.new(0, 12, 0, 0)
	label.BackgroundTransparency = 1
	label.Text = name
	label.Font = Enum.Font.Gotham
	label.TextSize = 14
	label.TextColor3 = Color3.fromRGB(230, 230, 230)
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = holder

	local button = Instance.new("TextButton")
	button.Size = UDim2.new(0, 36, 0, 20)
	button.Position = UDim2.new(1, -44, 0.5, -10)
	button.BackgroundColor3 = default and Color3.fromRGB(80, 180, 100) or Color3.fromRGB(50, 50, 60)
	button.BorderSizePixel = 0
	button.Text = ""
	button.Parent = holder

	local btnCorner = Instance.new("UICorner")
	btnCorner.CornerRadius = UDim.new(1, 0)
	btnCorner.Parent = button

	local circle = Instance.new("Frame")
	circle.Size = UDim2.new(0, 16, 0, 16)
	circle.Position = default and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8)
	circle.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
	circle.BorderSizePixel = 0
	circle.Parent = button

	local circleCorner = Instance.new("UICorner")
	circleCorner.CornerRadius = UDim.new(1, 0)
	circleCorner.Parent = circle

	local state = default
	button.MouseButton1Click:Connect(function()
		state = not state
		local tweenInfo = TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
		if state then
			TweenService:Create(button, tweenInfo, {BackgroundColor3 = Color3.fromRGB(80, 180, 100)}):Play()
			TweenService:Create(circle, tweenInfo, {Position = UDim2.new(1, -18, 0.5, -8)}):Play()
		else
			TweenService:Create(button, tweenInfo, {BackgroundColor3 = Color3.fromRGB(50, 50, 60)}):Play()
			TweenService:Create(circle, tweenInfo, {Position = UDim2.new(0, 2, 0.5, -8)}):Play()
		end
		callback(state)
	end)
end

local stopClick
local StatusLabel

createToggle("Aimbot", Settings.Aimbot, function(v)
	Settings.Aimbot = v
	if not v then
		Enabled = false
		LockedTarget = nil
		stopClick()
		StatusLabel.Text = "Status: Idle"
		StatusLabel.TextColor3 = Color3.fromRGB(180, 180, 190)
	end
end)

createToggle("Triggerbot", Settings.Triggerbot, function(v)
	Settings.Triggerbot = v
end)

createToggle("ESP", Settings.ESP, function(v)
	Settings.ESP = v
end)

createToggle("Wall Check", Settings.WallCheck, function(v)
	Settings.WallCheck = v
end)

createToggle("No Recoil", Settings.NoRecoil, function(v)
	Settings.NoRecoil = v
end)

local KeybindLabel = Instance.new("TextLabel")
KeybindLabel.Size = UDim2.new(1, 0, 0, 28)
KeybindLabel.BackgroundColor3 = Color3.fromRGB(28, 28, 36)
KeybindLabel.BorderSizePixel = 0
KeybindLabel.Text = "Lock Key: E  |  Menu: RCtrl"
KeybindLabel.Font = Enum.Font.Gotham
KeybindLabel.TextSize = 12
KeybindLabel.TextColor3 = Color3.fromRGB(180, 180, 190)
KeybindLabel.Parent = Container

local KeybindCorner = Instance.new("UICorner")
KeybindCorner.CornerRadius = UDim.new(0, 6)
KeybindCorner.Parent = KeybindLabel

StatusLabel = Instance.new("TextLabel")
StatusLabel.Size = UDim2.new(1, 0, 0, 28)
StatusLabel.BackgroundColor3 = Color3.fromRGB(28, 28, 36)
StatusLabel.BorderSizePixel = 0
StatusLabel.Text = "Status: Idle"
StatusLabel.Font = Enum.Font.Gotham
StatusLabel.TextSize = 13
StatusLabel.TextColor3 = Color3.fromRGB(180, 180, 190)
StatusLabel.Parent = Container

local StatusCorner = Instance.new("UICorner")
StatusCorner.CornerRadius = UDim.new(0, 6)
StatusCorner.Parent = StatusLabel

local function getTargetPart(character)
	if Settings.TargetPart == "Head" and character:FindFirstChild("Head") then
		return character.Head
	elseif Settings.TargetPart == "Torso" and character:FindFirstChild("Torso") then
		return character.Torso
	elseif character:FindFirstChild("HumanoidRootPart") then
		return character.HumanoidRootPart
	end
	return nil
end

local function isVisible(part)
	if not Settings.WallCheck then return true end
	local origin = Camera.CFrame.Position
	local direction = part.Position - origin
	local params = RaycastParams.new()
	params.FilterType = Enum.RaycastFilterType.Exclude
	params.FilterDescendantsInstances = {LocalPlayer.Character, Camera}
	local result = workspace:Raycast(origin, direction.Unit * direction.Magnitude, params)
	if not result then return true end
	return result.Instance and result.Instance:IsDescendantOf(part.Parent)
end

local function getHoveredEnemy()
	local mousePos = UserInputService:GetMouseLocation()
	local closestPart = nil
	local closestDist = Settings.LockFOV

	for _, player in Players:GetPlayers() do
		if player == LocalPlayer then continue end

		local char = player.Character
		if not char then continue end
		local humanoid = char:FindFirstChildOfClass("Humanoid")
		if not humanoid or humanoid.Health <= 0 then continue end

		local part = getTargetPart(char)
		if not part then continue end

		local screenPos, onScreen = Camera:WorldToViewportPoint(part.Position)
		if not onScreen then continue end

		local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
		if dist < closestDist and isVisible(part) then
			closestDist = dist
			closestPart = part
		end
	end
	return closestPart
end

local function startClick()
	if HoldingClick then return end
	HoldingClick = true
	local pos = UserInputService:GetMouseLocation()
	VirtualInputManager:SendMouseButtonEvent(pos.X, pos.Y, 0, true, game, 1)
end

stopClick = function()
	if not HoldingClick then
		return
	end

	HoldingClick = false

	local pos = UserInputService:GetMouseLocation()
	VirtualInputManager:SendMouseButtonEvent(pos.X, pos.Y, 0, false, game, 1)
end

local function disableAimbot()
	Enabled = false
	LockedTarget = nil
	stopClick()
	StatusLabel.Text = "Status: Idle"
	StatusLabel.TextColor3 = Color3.fromRGB(180, 180, 190)
end

local function createESP(player)
	if player == LocalPlayer then return end

	local highlight = Instance.new("Highlight")
	highlight.Name = player.Name
	highlight.FillTransparency = 0.75
	highlight.OutlineTransparency = 0
	highlight.FillColor = Color3.fromRGB(255, 60, 60)
	highlight.OutlineColor = Color3.fromRGB(255, 60, 60)
	highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	highlight.Parent = ScreenGui

	local billboard = Instance.new("BillboardGui")
	billboard.Name = player.Name .. "_Billboard"
	billboard.Size = UDim2.new(0, 200, 0, 50)
	billboard.StudsOffset = Vector3.new(0, 3.2, 0)
	billboard.AlwaysOnTop = true
	billboard.Parent = ScreenGui

	local nameLabel = Instance.new("TextLabel")
	nameLabel.Size = UDim2.new(1, 0, 0, 18)
	nameLabel.BackgroundTransparency = 1
	nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
	nameLabel.TextStrokeTransparency = 0.4
	nameLabel.Font = Enum.Font.GothamBold
	nameLabel.TextSize = 14
	nameLabel.Text = player.Name
	nameLabel.Parent = billboard

	local distLabel = Instance.new("TextLabel")
	distLabel.Size = UDim2.new(1, 0, 0, 16)
	distLabel.Position = UDim2.new(0, 0, 0, 16)
	distLabel.BackgroundTransparency = 1
	distLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
	distLabel.TextStrokeTransparency = 0.5
	distLabel.Font = Enum.Font.Gotham
	distLabel.TextSize = 12
	distLabel.Parent = billboard

	local healthBarBG = Instance.new("Frame")
	healthBarBG.Size = UDim2.new(0.7, 0, 0, 4)
	healthBarBG.Position = UDim2.new(0.15, 0, 0, 34)
	healthBarBG.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
	healthBarBG.BorderSizePixel = 0
	healthBarBG.Parent = billboard

	local healthBar = Instance.new("Frame")
	healthBar.Size = UDim2.new(1, 0, 1, 0)
	healthBar.BackgroundColor3 = Color3.fromRGB(0, 255, 100)
	healthBar.BorderSizePixel = 0
	healthBar.Parent = healthBarBG

	return {
		Highlight = highlight,
		Billboard = billboard,
		NameLabel = nameLabel,
		DistLabel = distLabel,
		HealthBar = healthBar
	}
end

local function updateESP()
	for _, player in Players:GetPlayers() do
		if player == LocalPlayer then continue end

		local char = player.Character
		local hrp = char and char:FindFirstChild("HumanoidRootPart")
		local humanoid = char and char:FindFirstChildOfClass("Humanoid")

		if not ESPObjects[player] then
			ESPObjects[player] = createESP(player)
		end

		local data = ESPObjects[player]
		local show = Settings.ESP and char and hrp and humanoid and humanoid.Health > 0

		if show then
			local dist = (Camera.CFrame.Position - hrp.Position).Magnitude
			if dist > Settings.ESPMaxDistance then
				show = false
			end
		end

		if show then
			data.Highlight.Adornee = char
			data.Highlight.Enabled = true
			data.Billboard.Adornee = hrp
			data.Billboard.Enabled = true

			local dist = math.floor((Camera.CFrame.Position - hrp.Position).Magnitude)
			data.DistLabel.Text = dist .. "m"
			data.NameLabel.Text = player.Name

			local healthPercent = math.clamp(humanoid.Health / humanoid.MaxHealth, 0, 1)
			data.HealthBar.Size = UDim2.new(healthPercent, 0, 1, 0)

			if healthPercent > 0.6 then
				data.HealthBar.BackgroundColor3 = Color3.fromRGB(0, 255, 100)
			elseif healthPercent > 0.3 then
				data.HealthBar.BackgroundColor3 = Color3.fromRGB(255, 200, 0)
			else
				data.HealthBar.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
			end

			if player.Team and player.Team.TeamColor then
				data.Highlight.FillColor = player.Team.TeamColor.Color
				data.Highlight.OutlineColor = player.Team.TeamColor.Color
			end
		else
			data.Highlight.Enabled = false
			data.Billboard.Enabled = false
		end
	end

	for player, data in pairs(ESPObjects) do
		if not player.Parent then
			data.Highlight:Destroy()
			data.Billboard:Destroy()
			ESPObjects[player] = nil
		end
	end
end

local function onCharacterAdded(character)
	local humanoid = character:WaitForChild("Humanoid", 5)
	if humanoid then
		humanoid.Died:Connect(function()
			disableAimbot()
		end)
	end
end

if LocalPlayer.Character then
	onCharacterAdded(LocalPlayer.Character)
end
LocalPlayer.CharacterAdded:Connect(onCharacterAdded)

UserInputService.InputBegan:Connect(function(input, gpe)
	if gpe then return end

	if input.KeyCode == Enum.KeyCode.RightControl then
		setMenuVisible(not MenuOpen)
		return
	end

	if input.KeyCode == Enum.KeyCode.E then
		if not Settings.Aimbot then return end

		if Enabled then
			disableAimbot()
		else
			local hovered = getHoveredEnemy()
			if hovered then
				Enabled = true
				LockedTarget = hovered
				StatusLabel.Text = "Status: Locked"
				StatusLabel.TextColor3 = Color3.fromRGB(80, 220, 120)
			end
		end
	end
end)

RunService:BindToRenderStep("dab", Enum.RenderPriority.Camera.Value + 1, function(dt)
	updateESP()

	if not Enabled or not LockedTarget or not Settings.Aimbot then
		stopClick()
		return
	end

	local char = LocalPlayer.Character
	if not char then
		disableAimbot()
		return
	end

	local localHum = char:FindFirstChildOfClass("Humanoid")
	if not localHum or localHum.Health <= 0 then
		disableAimbot()
		return
	end

	if not LockedTarget.Parent then
		disableAimbot()
		return
	end

	local humanoid = LockedTarget.Parent:FindFirstChildOfClass("Humanoid")
	if not humanoid or humanoid.Health <= 0 then
		disableAimbot()
		return
	end

	if Settings.WallCheck and not isVisible(LockedTarget) then
		stopClick()
		return
	end

	local aimPos = LockedTarget.Position
	if Settings.Prediction > 0 then
		local velocity = LockedTarget.AssemblyLinearVelocity
		if velocity then
			aimPos = aimPos + velocity * Settings.Prediction
		end
	end

	local targetCF = CFrame.new(Camera.CFrame.Position, aimPos)

	if Settings.NoRecoil then
		Camera.CFrame = targetCF
	else
		local alpha = math.clamp(Settings.Smoothness * (dt * 60), 0, 1)
		Camera.CFrame = Camera.CFrame:Lerp(targetCF, alpha)
	end

	if Settings.Triggerbot then
		local screenPos, onScreen = Camera:WorldToViewportPoint(LockedTarget.Position)
		if onScreen then
			local mousePos = UserInputService:GetMouseLocation()
			local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
			if dist <= Settings.TriggerFOV then
				if tick() - LastTrigger >= Settings.TriggerDelay then
					LastTrigger = tick()
					startClick()
					task.delay(0.025, stopClick)
				end
			else
				stopClick()
			end
		else
			stopClick()
		end
	end
end)
