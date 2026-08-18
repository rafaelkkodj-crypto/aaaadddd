--========================================================
-- NEON TRAINING SYSTEM
-- Roblox Studio
-- R6 / R15
-- ESP + SKELETON + ALL CHECK + AIM ASSIST
--========================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local Camera = workspace.CurrentCamera

--========================================================
-- CONFIGURAÇÃO
--========================================================

local Settings = {
	ESP = false,
	ESPDistance = 3000,

	Skeleton = false,
	SkeletonDistance = 3000,

	AllCheck = false,
	AllCheckDistance = 3000,

	AimAssist = false,
	AimDistance = 200,
	AimFOV = 120,
	AimSmooth = 0.18,

	Noclip = false,
}

local PINK = Color3.fromRGB(255, 0, 180)
local PURPLE = Color3.fromRGB(130, 0, 220)
local DARK = Color3.fromRGB(15, 8, 20)
local DARK2 = Color3.fromRGB(30, 12, 38)
local WHITE = Color3.new(1,1,1)
local GREEN = Color3.fromRGB(50,255,120)
local RED = Color3.fromRGB(255,60,70)
local YELLOW = Color3.fromRGB(255,220,50)

--========================================================
-- FUNÇÕES DE PLAYER
--========================================================

local function CharacterOf(player)
	return player and player.Character
end

local function HumanoidOf(player)
	local c = CharacterOf(player)
	return c and c:FindFirstChildOfClass("Humanoid")
end

local function RootOf(player)
	local c = CharacterOf(player)
	if not c then return nil end

	return c:FindFirstChild("HumanoidRootPart")
		or c:FindFirstChild("Torso")
end

local function HeadOf(player)
	local c = CharacterOf(player)
	return c and c:FindFirstChild("Head")
end

local function Alive(player)
	local hum = HumanoidOf(player)
	return hum and hum.Health > 0
end

local function DistanceFromMe(player)
	local a = RootOf(LocalPlayer)
	local b = RootOf(player)

	if not a or not b then
		return math.huge
	end

	return (a.Position - b.Position).Magnitude
end

--========================================================
-- GUI PRINCIPAL
--========================================================

local Gui = Instance.new("ScreenGui")
Gui.Name = "NeonTrainingGui"
Gui.ResetOnSpawn = false
Gui.Parent = PlayerGui

--========================================================
-- FOV
--========================================================

local FOV = Instance.new("Frame")
FOV.Name = "VisibleFOV"
FOV.AnchorPoint = Vector2.new(0.5,0.5)
FOV.Position = UDim2.fromScale(0.5,0.5)
FOV.Size = UDim2.fromOffset(Settings.AimFOV * 2, Settings.AimFOV * 2)
FOV.BackgroundTransparency = 1
FOV.Visible = false
FOV.ZIndex = 2
FOV.Parent = Gui

local FOVCorner = Instance.new("UICorner")
FOVCorner.CornerRadius = UDim.new(1,0)
FOVCorner.Parent = FOV

local FOVStroke = Instance.new("UIStroke")
FOVStroke.Color = PINK
FOVStroke.Thickness = 2
FOVStroke.Parent = FOV

--========================================================
-- PAINEL
--========================================================

local Main = Instance.new("Frame")
Main.Size = UDim2.fromOffset(420,570)
Main.Position = UDim2.new(.5,-210,.5,-285)
Main.BackgroundColor3 = DARK
Main.BorderSizePixel = 0
Main.Parent = Gui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0,15)
MainCorner.Parent = Main

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = PINK
MainStroke.Thickness = 2
MainStroke.Parent = Main

local Header = Instance.new("Frame")
Header.Size = UDim2.new(1,0,0,70)
Header.BackgroundColor3 = DARK2
Header.BorderSizePixel = 0
Header.Parent = Main

local HeaderCorner = Instance.new("UICorner")
HeaderCorner.CornerRadius = UDim.new(0,15)
HeaderCorner.Parent = Header

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,-20,0,30)
Title.Position = UDim2.fromOffset(12,8)
Title.BackgroundTransparency = 1
Title.Text = "NEON TRAINING"
Title.TextColor3 = PINK
Title.Font = Enum.Font.GothamBold
Title.TextSize = 21
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Header

local Sub = Instance.new("TextLabel")
Sub.Size = UDim2.new(1,-20,0,20)
Sub.Position = UDim2.fromOffset(12,40)
Sub.BackgroundTransparency = 1
Sub.Text = "PLAYER • R6 / R15"
Sub.TextColor3 = Color3.fromRGB(190,150,200)
Sub.Font = Enum.Font.Gotham
Sub.TextSize = 11
Sub.TextXAlignment = Enum.TextXAlignment.Left
Sub.Parent = Header

local Scroll = Instance.new("ScrollingFrame")
Scroll.Size = UDim2.new(1,-20,1,-82)
Scroll.Position = UDim2.fromOffset(10,77)
Scroll.BackgroundTransparency = 1
Scroll.BorderSizePixel = 0
Scroll.ScrollBarThickness = 4
Scroll.ScrollBarImageColor3 = PINK
Scroll.CanvasSize = UDim2.fromOffset(0,900)
Scroll.Parent = Main

local Layout = Instance.new("UIListLayout")
Layout.Padding = UDim.new(0,8)
Layout.Parent = Scroll

--========================================================
-- GUI HELPERS
--========================================================

local function Section(text)

	local l = Instance.new("TextLabel")
	l.Size = UDim2.new(1,-10,0,28)
	l.BackgroundTransparency = 1
	l.Text = text
	l.TextColor3 = PINK
	l.Font = Enum.Font.GothamBold
	l.TextSize = 14
	l.TextXAlignment = Enum.TextXAlignment.Left
	l.Parent = Scroll

end

local function Toggle(text, initial, callback)

	local b = Instance.new("TextButton")
	b.Size = UDim2.new(1,-10,0,42)
	b.BackgroundColor3 = DARK2
	b.BorderSizePixel = 0
	b.Text = ""
	b.AutoButtonColor = false
	b.Parent = Scroll

	local c = Instance.new("UICorner")
	c.CornerRadius = UDim.new(0,8)
	c.Parent = b

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1,-80,1,0)
	label.Position = UDim2.fromOffset(12,0)
	label.BackgroundTransparency = 1
	label.Text = text
	label.TextColor3 = WHITE
	label.Font = Enum.Font.GothamMedium
	label.TextSize = 12
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = b

	local state = Instance.new("TextLabel")
	state.Size = UDim2.fromOffset(55,25)
	state.Position = UDim2.new(1,-67,.5,-12)
	state.BackgroundColor3 = Color3.fromRGB(55,25,60)
	state.TextColor3 = WHITE
	state.Font = Enum.Font.GothamBold
	state.TextSize = 10
	state.Parent = b

	local sc = Instance.new("UICorner")
	sc.CornerRadius = UDim.new(0,6)
	sc.Parent = state

	local value = initial

	local function update()

		state.Text = value and "ON" or "OFF"
		state.BackgroundColor3 = value and PINK or Color3.fromRGB(55,25,60)

		callback(value)

	end

	b.MouseButton1Click:Connect(function()
		value = not value
		update()
	end)

	update()

end

local function Slider(text,min,max,default,callback)

	local holder = Instance.new("Frame")
	holder.Size = UDim2.new(1,-10,0,65)
	holder.BackgroundColor3 = DARK2
	holder.BorderSizePixel = 0
	holder.Parent = Scroll

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0,8)
	corner.Parent = holder

	local title = Instance.new("TextLabel")
	title.Size = UDim2.new(1,-80,0,24)
	title.Position = UDim2.fromOffset(12,5)
	title.BackgroundTransparency = 1
	title.Text = text
	title.TextColor3 = WHITE
	title.Font = Enum.Font.GothamMedium
	title.TextSize = 12
	title.TextXAlignment = Enum.TextXAlignment.Left
	title.Parent = holder

	local valueLabel = Instance.new("TextLabel")
	valueLabel.Size = UDim2.fromOffset(65,24)
	valueLabel.Position = UDim2.new(1,-75,0,5)
	valueLabel.BackgroundTransparency = 1
	valueLabel.TextColor3 = PINK
	valueLabel.Font = Enum.Font.GothamBold
	valueLabel.TextSize = 12
	valueLabel.Parent = holder

	local bar = Instance.new("Frame")
	bar.Size = UDim2.new(1,-25,0,7)
	bar.Position = UDim2.fromOffset(12,43)
	bar.BackgroundColor3 = Color3.fromRGB(55,25,60)
	bar.BorderSizePixel = 0
	bar.Parent = holder

	local bc = Instance.new("UICorner")
	bc.CornerRadius = UDim.new(1,0)
	bc.Parent = bar

	local fill = Instance.new("Frame")
	fill.BackgroundColor3 = PINK
	fill.BorderSizePixel = 0
	fill.Parent = bar

	local fc = Instance.new("UICorner")
	fc.CornerRadius = UDim.new(1,0)
	fc.Parent = fill

	local dragging = false

	local function SetValue(v)

		v = math.clamp(math.floor(v+.5),min,max)

		local pct = (v-min)/(max-min)

		fill.Size = UDim2.new(pct,0,1,0)
		valueLabel.Text = tostring(v)

		callback(v)

	end

	local function MouseValue(x)

		local pct = math.clamp(
			(x-bar.AbsolutePosition.X) /
			bar.AbsoluteSize.X,
			0,
			1
		)

		SetValue(min+(max-min)*pct)

	end

	bar.InputBegan:Connect(function(input)

		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			MouseValue(input.Position.X)
		end

	end)

	UIS.InputChanged:Connect(function(input)

		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
			MouseValue(input.Position.X)
		end

	end)

	UIS.InputEnded:Connect(function(input)

		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = false
		end

	end)

	SetValue(default)

end

--========================================================
-- PAINEL
--========================================================

Section("🎯 AIM ASSIST")

Toggle("Aim Assist de Treino",false,function(v)
	Settings.AimAssist = v
	FOV.Visible = v
end)

Slider("Distância do Aim",1,200,200,function(v)
	Settings.AimDistance = v
end)

Slider("FOV",1,500,120,function(v)

	Settings.AimFOV = v

	FOV.Size = UDim2.fromOffset(v*2,v*2)

end)

Slider("Suavidade",1,100,18,function(v)
	Settings.AimSmooth = v/100
end)

Section("👁 ESP")

Toggle("ESP — Nome / Vida / Distância",false,function(v)
	Settings.ESP = v
end)

Slider("Distância ESP",1,3000,3000,function(v)
	Settings.ESPDistance = v
end)

Section("🦴 SKELETON")

Toggle("Skeleton R6 / R15",false,function(v)
	Settings.Skeleton = v
end)

Slider("Distância Skeleton",1,3000,3000,function(v)
	Settings.SkeletonDistance = v
end)

Section("🔎 ALL CHECK")

Toggle("All Check — Visibilidade",false,function(v)
	Settings.AllCheck = v
end)

Slider("Distância All Check",1,3000,3000,function(v)
	Settings.AllCheckDistance = v
end)

Section("🌀 MOVEMENT")

Toggle("Noclip",false,function(v)
	Settings.Noclip = v
end)

--========================================================
-- ESP
--========================================================

local ESP = {}

local function RemoveESP(player)

	if ESP[player] then
		ESP[player]:Destroy()
		ESP[player] = nil
	end

end

local function UpdateESP(player)

	if player == LocalPlayer then return end

	local character = CharacterOf(player)
	local head = HeadOf(player)
	local humanoid = HumanoidOf(player)

	if not Settings.ESP or not character or not head or not humanoid or humanoid.Health <= 0 then

		if ESP[player] then
			ESP[player].Enabled = false
		end

		return
	end

	local distance = DistanceFromMe(player)

	if distance > Settings.ESPDistance then

		if ESP[player] then
			ESP[player].Enabled = false
		end

		return
	end

	local gui = ESP[player]

	if not gui or gui.Parent ~= Gui then

		RemoveESP(player)

		gui = Instance.new("BillboardGui")
		gui.Name = "ESP"
		gui.Size = UDim2.fromOffset(220,80)
		gui.AlwaysOnTop = true
		gui.Parent = Gui

		local name = Instance.new("TextLabel")
		name.Name = "Name"
		name.Size = UDim2.new(1,0,0,22)
		name.BackgroundTransparency = 1
		name.TextColor3 = PINK
		name.TextStrokeTransparency = 0
		name.TextStrokeColor3 = Color3.new(0,0,0)
		name.Font = Enum.Font.GothamBold
		name.TextSize = 13
		name.Parent = gui

		local dist = Instance.new("TextLabel")
		dist.Name = "Distance"
		dist.Size = UDim2.new(1,0,0,18)
		dist.Position = UDim2.fromOffset(0,22)
		dist.BackgroundTransparency = 1
		dist.TextColor3 = WHITE
		dist.TextStrokeTransparency = 0
		dist.Font = Enum.Font.Gotham
		dist.TextSize = 11
		dist.Parent = gui

		local back = Instance.new("Frame")
		back.Name = "HealthBack"
		back.Size = UDim2.new(.7,0,0,7)
		back.Position = UDim2.new(.15,0,0,45)
		back.BackgroundColor3 = Color3.fromRGB(50,20,55)
		back.BorderSizePixel = 0
		back.Parent = gui

		local bar = Instance.new("Frame")
		bar.Name = "Health"
		bar.Size = UDim2.new(1,0,1,0)
		bar.BackgroundColor3 = GREEN
		bar.BorderSizePixel = 0
		bar.Parent = back

		ESP[player] = gui

	end

	gui.Adornee = head
	gui.MaxDistance = Settings.ESPDistance+100
	gui.Enabled = true

	gui.Name.Text = player.DisplayName
	gui.Distance.Text = math.floor(distance).." studs"

	local health = math.clamp(
		humanoid.Health/math.max(humanoid.MaxHealth,1),
		0,
		1
	)

	gui.Health.Size = UDim2.new(health,0,1,0)

	if health <= .25 then
		gui.Health.BackgroundColor3 = RED
	elseif health <= .5 then
		gui.Health.BackgroundColor3 = YELLOW
	else
		gui.Health.BackgroundColor3 = GREEN
	end

end

--========================================================
-- SKELETON
--========================================================

local Skeleton = {}

local R6 = {
	{"Head","Torso"},
	{"Torso","Left Arm"},
	{"Torso","Right Arm"},
	{"Torso","Left Leg"},
	{"Torso","Right Leg"},
}

local R15 = {
	{"Head","UpperTorso"},
	{"UpperTorso","LowerTorso"},

	{"UpperTorso","LeftUpperArm"},
	{"LeftUpperArm","LeftLowerArm"},
	{"LeftLowerArm","LeftHand"},

	{"UpperTorso","RightUpperArm"},
	{"RightUpperArm","RightLowerArm"},
	{"RightLowerArm","RightHand"},

	{"LowerTorso","LeftUpperLeg"},
	{"LeftUpperLeg","LeftLowerLeg"},
	{"LeftLowerLeg","LeftFoot"},

	{"LowerTorso","RightUpperLeg"},
	{"RightUpperLeg","RightLowerLeg"},
	{"RightLowerLeg","RightFoot"},
}

local function MakeBone(folder,a,b)

	if not a or not b then return end

	local att0 = Instance.new("Attachment")
	att0.Parent = a

	local att1 = Instance.new("Attachment")
	att1.Parent = b

	local beam = Instance.new("Beam")
	beam.Attachment0 = att0
	beam.Attachment1 = att1
	beam.Width0 = .045
	beam.Width1 = .045
	beam.Color = ColorSequence.new(PINK)
	beam.LightEmission = 1
	beam.FaceCamera = true
	beam.Parent = folder

end

local function RemoveSkeleton(player)

	if Skeleton[player] then
		Skeleton[player]:Destroy()
		Skeleton[player] = nil
	end

end

local function UpdateSkeleton(player)

	if player == LocalPlayer then return end

	if not Settings.Skeleton then
		RemoveSkeleton(player)
		return
	end

	if not Alive(player) then
		RemoveSkeleton(player)
		return
	end

	if DistanceFromMe(player) > Settings.SkeletonDistance then
		RemoveSkeleton(player)
		return
	end

	local character = CharacterOf(player)
	local humanoid = HumanoidOf(player)

	if not character or not humanoid then return end

	RemoveSkeleton(player)

	local folder = Instance.new("Folder")
	folder.Name = "Skeleton"
	folder.Parent = Gui

	local bones

	if humanoid.RigType == Enum.HumanoidRigType.R15 then
		bones = R15
	else
		bones = R6
	end

	for _,pair in ipairs(bones) do

		MakeBone(
			folder,
			character:FindFirstChild(pair[1]),
			character:FindFirstChild(pair[2])
		)

	end

	Skeleton[player] = folder

end

--========================================================
-- ALL CHECK
--========================================================

local Checks = {}

local function RemoveCheck(player)

	if Checks[player] then
		Checks[player]:Destroy()
		Checks[player] = nil
	end

end

local function Visible(player)

	local head = HeadOf(player)

	if not head then return false end

	local myCharacter = CharacterOf(LocalPlayer)

	local params = RaycastParams.new()
	params.FilterType = Enum.RaycastFilterType.Exclude
	params.FilterDescendantsInstances = {myCharacter}

	local result = workspace:Raycast(
		Camera.CFrame.Position,
		head.Position-Camera.CFrame.Position,
		params
	)

	if not result then
		return true
	end

	return result.Instance:IsDescendantOf(CharacterOf(player))

end

local function UpdateCheck(player)

	if player == LocalPlayer then return end

	if not Settings.AllCheck or not Alive(player) then
		RemoveCheck(player)
		return
	end

	if DistanceFromMe(player) > Settings.AllCheckDistance then
		RemoveCheck(player)
		return
	end

	local character = CharacterOf(player)

	if not character then return end

	local h = Checks[player]

	if not h then

		h = Instance.new("Highlight")
		h.Name = "AllCheck"
		h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
		h.FillTransparency = .85
		h.OutlineTransparency = 0
		h.Parent = character

		Checks[player] = h

	end

	h.Adornee = character

	if Visible(player) then
		h.FillColor = GREEN
		h.OutlineColor = GREEN
	else
		h.FillColor = RED
		h.OutlineColor = RED
	end

end

--========================================================
-- AIM ASSIST
--========================================================

local function GetTarget()

	local best = nil
	local bestDistance = math.huge

	local center = Vector2.new(
		Camera.ViewportSize.X/2,
		Camera.ViewportSize.Y/2
	)

	for _,player in ipairs(Players:GetPlayers()) do

		if player ~= LocalPlayer and Alive(player) then

			local distance = DistanceFromMe(player)

			if distance <= Settings.AimDistance then

				local head = HeadOf(player)

				if head then

					local screen,visible =
						Camera:WorldToViewportPoint(head.Position)

					if visible and screen.Z > 0 then

						local point = Vector2.new(
							screen.X,
							screen.Y
						)

						local delta =
							(point-center).Magnitude

						if delta <= Settings.AimFOV then

							if not Settings.AllCheck or Visible(player) then

								if delta < bestDistance then
									bestDistance = delta
									best = player
								end

							end

						end

					end

				end

			end

		end

	end

	return best

end

--========================================================
-- AIM LOOP
--========================================================

RunService:BindToRenderStep(
	"TrainingAimAssist",
	Enum.RenderPriority.Camera.Value+1,
	function()

		if not Settings.AimAssist then return end

		local target = GetTarget()

		if not target then return end

		local head = HeadOf(target)

		if not head then return end

		local cameraPosition = Camera.CFrame.Position

		local targetCF = CFrame.lookAt(
			cameraPosition,
			head.Position
		)

		Camera.CFrame = Camera.CFrame:Lerp(
			targetCF,
			Settings.AimSmooth
		)

	end
)

--========================================================
-- NOCLIP
--========================================================

RunService.Stepped:Connect(function()

	if not Settings.Noclip then return end

	local character = CharacterOf(LocalPlayer)

	if not character then return end

	for _,object in ipairs(character:GetDescendants()) do

		if object:IsA("BasePart") then
			object.CanCollide = false
		end

	end

end)

--========================================================
-- ATUALIZAÇÃO
--========================================================

local skeletonTimer = 0

RunService.RenderStepped:Connect(function(dt)

	for _,player in ipairs(Players:GetPlayers()) do

		if player ~= LocalPlayer then

			UpdateESP(player)
			UpdateCheck(player)

		end

	end

	skeletonTimer += dt

	if skeletonTimer >= .12 then

		skeletonTimer = 0

		for _,player in ipairs(Players:GetPlayers()) do

			if player ~= LocalPlayer then
				UpdateSkeleton(player)
			end

		end

	end

	local viewport = Camera.ViewportSize

	FOV.Position = UDim2.fromOffset(
		viewport.X/2,
		viewport.Y/2
	)

end)

--========================================================
-- RESPAWN / PLAYER
--========================================================

local function SetupPlayer(player)

	if player == LocalPlayer then return end

	player.CharacterRemoving:Connect(function()

		RemoveESP(player)
		RemoveSkeleton(player)
		RemoveCheck(player)

	end)

	player.CharacterAdded:Connect(function()

		RemoveESP(player)
		RemoveSkeleton(player)
		RemoveCheck(player)

	end)

end

for _,player in ipairs(Players:GetPlayers()) do
	SetupPlayer(player)
end

Players.PlayerAdded:Connect(SetupPlayer)

Players.PlayerRemoving:Connect(function(player)

	RemoveESP(player)
	RemoveSkeleton(player)
	RemoveCheck(player)

end)

--========================================================
-- ARRASTAR PAINEL
--========================================================

local dragging = false
local dragStart
local startPos

Header.InputBegan:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1 then

		dragging = true
		dragStart = input.Position
		startPos = Main.Position

	end

end)

Header.InputEnded:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = false
	end

end)

UIS.InputChanged:Connect(function(input)

	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then

		local delta = input.Position-dragStart

		Main.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset+delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset+delta.Y
		)

	end

end)

--========================================================
-- RIGHT SHIFT
--========================================================

UIS.InputBegan:Connect(function(input,processed)

	if processed then return end

	if input.KeyCode == Enum.KeyCode.RightShift then
		Main.Visible = not Main.Visible
	end

end)

print("NEON TRAINING SYSTEM carregado.")
print("ESP R6/R15: OK")
print("SKELETON R6/R15: OK")
print("ALL CHECK: OK")
print("AIM ASSIST: OK")
