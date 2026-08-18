--========================================================
-- NEON PLAYER TRAINING
-- ROBLOX STUDIO - PLAYER R6/R15
-- UM ÚNICO LOCALSCRIPT
--========================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

--========================================================
-- CONFIG
--========================================================

local Config = {
	ESP = false,
	ESPDistance = 3000,

	Skeleton = false,
	SkeletonDistance = 3000,

	AllCheck = false,
	AllCheckDistance = 3000,

	Aim = false,
	AimDistance = 200,
	AimFOV = 120,
	AimSmooth = 18,

	Noclip = false,
}

local Colors = {
	Pink = Color3.fromRGB(255, 20, 190),
	Pink2 = Color3.fromRGB(255, 70, 210),
	Purple = Color3.fromRGB(145, 25, 230),

	Background = Color3.fromRGB(10, 7, 15),
	Panel = Color3.fromRGB(20, 11, 28),
	Panel2 = Color3.fromRGB(30, 15, 40),

	White = Color3.fromRGB(255,255,255),
	Gray = Color3.fromRGB(170,160,180),

	Green = Color3.fromRGB(50,255,125),
	Red = Color3.fromRGB(255,60,75),
	Yellow = Color3.fromRGB(255,215,50),
}

--========================================================
-- PLAYER FUNCTIONS
--========================================================

local function Character(player)
	return player and player.Character
end

local function Humanoid(player)
	local char = Character(player)
	return char and char:FindFirstChildOfClass("Humanoid")
end

local function Root(player)
	local char = Character(player)

	if not char then
		return nil
	end

	return char:FindFirstChild("HumanoidRootPart")
		or char:FindFirstChild("Torso")
end

local function Head(player)
	local char = Character(player)
	return char and char:FindFirstChild("Head")
end

local function IsAlive(player)
	local hum = Humanoid(player)
	return hum and hum.Health > 0
end

local function GetDistance(player)
	local myRoot = Root(LocalPlayer)
	local targetRoot = Root(player)

	if not myRoot or not targetRoot then
		return math.huge
	end

	return (myRoot.Position - targetRoot.Position).Magnitude
end

--========================================================
-- GUI
--========================================================

local Gui = Instance.new("ScreenGui")
Gui.Name = "NeonPlayerTraining"
Gui.ResetOnSpawn = false
Gui.IgnoreGuiInset = true
Gui.Parent = PlayerGui

--========================================================
-- FOV
--========================================================

local FOV = Instance.new("Frame")
FOV.Name = "AimFOV"
FOV.AnchorPoint = Vector2.new(0.5,0.5)
FOV.Position = UDim2.fromScale(0.5,0.5)
FOV.Size = UDim2.fromOffset(240,240)
FOV.BackgroundTransparency = 1
FOV.Visible = false
FOV.ZIndex = 3
FOV.Parent = Gui

local FOVCorner = Instance.new("UICorner")
FOVCorner.CornerRadius = UDim.new(1,0)
FOVCorner.Parent = FOV

local FOVStroke = Instance.new("UIStroke")
FOVStroke.Color = Colors.Pink
FOVStroke.Thickness = 2
FOVStroke.Transparency = 0.1
FOVStroke.Parent = FOV

--========================================================
-- MAIN PANEL
--========================================================

local Main = Instance.new("Frame")
Main.Name = "Main"
Main.Size = UDim2.fromOffset(440,620)
Main.Position = UDim2.new(0.5,-220,0.5,-310)
Main.BackgroundColor3 = Colors.Background
Main.BorderSizePixel = 0
Main.Parent = Gui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0,16)
MainCorner.Parent = Main

local MainStroke = Instance.new("UIStroke")
MainStroke.Color = Colors.Pink
MainStroke.Thickness = 2
MainStroke.Transparency = 0.15
MainStroke.Parent = Main

--========================================================
-- TOP BAR
--========================================================

local Top = Instance.new("Frame")
Top.Size = UDim2.new(1,0,0,78)
Top.BackgroundColor3 = Colors.Panel
Top.BorderSizePixel = 0
Top.Parent = Main

local TopCorner = Instance.new("UICorner")
TopCorner.CornerRadius = UDim.new(0,16)
TopCorner.Parent = Top

local Accent = Instance.new("Frame")
Accent.Size = UDim2.new(1,0,0,3)
Accent.BackgroundColor3 = Colors.Pink
Accent.BorderSizePixel = 0
Accent.Parent = Top

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1,-30,0,30)
Title.Position = UDim2.fromOffset(15,10)
Title.BackgroundTransparency = 1
Title.Text = "NEON PLAYER"
Title.TextColor3 = Colors.Pink
Title.Font = Enum.Font.GothamBold
Title.TextSize = 22
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = Top

local Subtitle = Instance.new("TextLabel")
Subtitle.Size = UDim2.new(1,-30,0,22)
Subtitle.Position = UDim2.fromOffset(15,42)
Subtitle.BackgroundTransparency = 1
Subtitle.Text = "PLAYER TRAINING  •  R6 / R15"
Subtitle.TextColor3 = Colors.Gray
Subtitle.Font = Enum.Font.Gotham
Subtitle.TextSize = 11
Subtitle.TextXAlignment = Enum.TextXAlignment.Left
Subtitle.Parent = Top

--========================================================
-- SCROLL
--========================================================

local Scroll = Instance.new("ScrollingFrame")
Scroll.Name = "Controls"
Scroll.Size = UDim2.new(1,-20,1,-90)
Scroll.Position = UDim2.fromOffset(10,85)
Scroll.BackgroundTransparency = 1
Scroll.BorderSizePixel = 0
Scroll.ScrollBarThickness = 4
Scroll.ScrollBarImageColor3 = Colors.Pink
Scroll.CanvasSize = UDim2.fromOffset(0,1000)
Scroll.Parent = Main

local Layout = Instance.new("UIListLayout")
Layout.Padding = UDim.new(0,9)
Layout.SortOrder = Enum.SortOrder.LayoutOrder
Layout.Parent = Scroll

--========================================================
-- SECTION
--========================================================

local function Section(icon,text)

	local holder = Instance.new("Frame")
	holder.Size = UDim2.new(1,-8,0,30)
	holder.BackgroundTransparency = 1
	holder.Parent = Scroll

	local line = Instance.new("Frame")
	line.Size = UDim2.fromOffset(4,20)
	line.Position = UDim2.fromOffset(2,5)
	line.BackgroundColor3 = Colors.Pink
	line.BorderSizePixel = 0
	line.Parent = holder

	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1,-25,1,0)
	label.Position = UDim2.fromOffset(14,0)
	label.BackgroundTransparency = 1
	label.Text = icon.."  "..text
	label.TextColor3 = Colors.Pink2
	label.Font = Enum.Font.GothamBold
	label.TextSize = 14
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.Parent = holder

end

--========================================================
-- TOGGLE
--========================================================

local function Toggle(icon,text,default,callback)

	local Button = Instance.new("TextButton")
	Button.Size = UDim2.new(1,-8,0,46)
	Button.BackgroundColor3 = Colors.Panel
	Button.BorderSizePixel = 0
	Button.Text = ""
	Button.AutoButtonColor = false
	Button.Parent = Scroll

	local Corner = Instance.new("UICorner")
	Corner.CornerRadius = UDim.new(0,9)
	Corner.Parent = Button

	local Stroke = Instance.new("UIStroke")
	Stroke.Color = Colors.Purple
	Stroke.Transparency = 0.65
	Stroke.Parent = Button

	local Icon = Instance.new("TextLabel")
	Icon.Size = UDim2.fromOffset(35,46)
	Icon.Position = UDim2.fromOffset(5,0)
	Icon.BackgroundTransparency = 1
	Icon.Text = icon
	Icon.TextSize = 18
	Icon.Parent = Button

	local Label = Instance.new("TextLabel")
	Label.Size = UDim2.new(1,-120,1,0)
	Label.Position = UDim2.fromOffset(42,0)
	Label.BackgroundTransparency = 1
	Label.Text = text
	Label.TextColor3 = Colors.White
	Label.Font = Enum.Font.GothamMedium
	Label.TextSize = 12
	Label.TextXAlignment = Enum.TextXAlignment.Left
	Label.Parent = Button

	local Status = Instance.new("TextLabel")
	Status.Size = UDim2.fromOffset(58,27)
	Status.Position = UDim2.new(1,-70,0.5,-13)
	Status.BackgroundColor3 = Color3.fromRGB(55,30,65)
	Status.TextColor3 = Colors.Gray
	Status.Font = Enum.Font.GothamBold
	Status.TextSize = 10
	Status.Parent = Button

	local StatusCorner = Instance.new("UICorner")
	StatusCorner.CornerRadius = UDim.new(0,7)
	StatusCorner.Parent = Status

	local value = default

	local function Update()

		if value then
			Status.Text = "ON"
			Status.BackgroundColor3 = Colors.Pink
			Status.TextColor3 = Colors.White
			Stroke.Color = Colors.Pink
			Stroke.Transparency = 0.25
		else
			Status.Text = "OFF"
			Status.BackgroundColor3 = Color3.fromRGB(55,30,65)
			Status.TextColor3 = Colors.Gray
			Stroke.Color = Colors.Purple
			Stroke.Transparency = 0.65
		end

		callback(value)
	end

	Button.MouseButton1Click:Connect(function()
		value = not value
		Update()
	end)

	Update()

	return Button
end

--========================================================
-- SLIDER
--========================================================

local function Slider(text,min,max,default,callback)

	local Holder = Instance.new("Frame")
	Holder.Size = UDim2.new(1,-8,0,70)
	Holder.BackgroundColor3 = Colors.Panel
	Holder.BorderSizePixel = 0
	Holder.Parent = Scroll

	local Corner = Instance.new("UICorner")
	Corner.CornerRadius = UDim.new(0,9)
	Corner.Parent = Holder

	local Label = Instance.new("TextLabel")
	Label.Size = UDim2.new(1,-100,0,25)
	Label.Position = UDim2.fromOffset(14,5)
	Label.BackgroundTransparency = 1
	Label.Text = text
	Label.TextColor3 = Colors.White
	Label.Font = Enum.Font.GothamMedium
	Label.TextSize = 12
	Label.TextXAlignment = Enum.TextXAlignment.Left
	Label.Parent = Holder

	local Value = Instance.new("TextLabel")
	Value.Size = UDim2.fromOffset(70,25)
	Value.Position = UDim2.new(1,-84,0,5)
	Value.BackgroundTransparency = 1
	Value.TextColor3 = Colors.Pink
	Value.Font = Enum.Font.GothamBold
	Value.TextSize = 12
	Value.Parent = Holder

	local Bar = Instance.new("Frame")
	Bar.Size = UDim2.new(1,-28,0,7)
	Bar.Position = UDim2.fromOffset(14,46)
	Bar.BackgroundColor3 = Color3.fromRGB(55,28,65)
	Bar.BorderSizePixel = 0
	Bar.Parent = Holder

	local BarCorner = Instance.new("UICorner")
	BarCorner.CornerRadius = UDim.new(1,0)
	BarCorner.Parent = Bar

	local Fill = Instance.new("Frame")
	Fill.BackgroundColor3 = Colors.Pink
	Fill.BorderSizePixel = 0
	Fill.Parent = Bar

	local FillCorner = Instance.new("UICorner")
	FillCorner.CornerRadius = UDim.new(1,0)
	FillCorner.Parent = Fill

	local dragging = false

	local function SetValue(v)

		v = math.clamp(
			math.floor(v+0.5),
			min,
			max
		)

		local percent =
			(v-min)/(max-min)

		Fill.Size =
			UDim2.new(
				percent,
				0,
				1,
				0
			)

		Value.Text = tostring(v)

		callback(v)
	end

	local function MouseValue(x)

		local percent = math.clamp(
			(x-Bar.AbsolutePosition.X) /
			Bar.AbsoluteSize.X,
			0,
			1
		)

		SetValue(
			min+(max-min)*percent
		)

	end

	Bar.InputBegan:Connect(function(input)

		if input.UserInputType ==
			Enum.UserInputType.MouseButton1 then

			dragging = true
			MouseValue(input.Position.X)

		end

	end)

	UserInputService.InputChanged:Connect(function(input)

		if dragging and
			input.UserInputType ==
			Enum.UserInputType.MouseMovement then

			MouseValue(input.Position.X)

		end

	end)

	UserInputService.InputEnded:Connect(function(input)

		if input.UserInputType ==
			Enum.UserInputType.MouseButton1 then

			dragging = false

		end

	end)

	SetValue(default)

end

--========================================================
-- CONTROLES
--========================================================

Section("🎯","AIM ASSIST")

Toggle(
	"🎯",
	"Aim Assist",
	false,
	function(v)
		Config.Aim = v
		FOV.Visible = v
	end
)

Slider(
	"Aim Distance",
	1,
	200,
	200,
	function(v)
		Config.AimDistance = v
	end
)

Slider(
	"Aim FOV",
	1,
	500,
	120,
	function(v)

		Config.AimFOV = v

		FOV.Size =
			UDim2.fromOffset(
				v*2,
				v*2
			)

	end
)

Slider(
	"Aim Smooth",
	1,
	100,
	18,
	function(v)

		Config.AimSmooth =
			math.clamp(v/100,0.01,1)

	end
)

Section("👁","ESP")

Toggle(
	"👁",
	"ESP — Nome / Vida / Distância",
	false,
	function(v)
		Config.ESP = v
	end
)

Slider(
	"ESP Distance",
	1,
	3000,
	3000,
	function(v)
		Config.ESPDistance = v
	end
)

Section("🦴","SKELETON")

Toggle(
	"🦴",
	"Skeleton R6 / R15",
	false,
	function(v)
		Config.Skeleton = v
	end
)

Slider(
	"Skeleton Distance",
	1,
	3000,
	3000,
	function(v)
		Config.SkeletonDistance = v
	end
)

Section("🔎","ALL CHECK")

Toggle(
	"🔎",
	"All Check — Visibilidade",
	false,
	function(v)
		Config.AllCheck = v
	end
)

Slider(
	"All Check Distance",
	1,
	3000,
	3000,
	function(v)
		Config.AllCheckDistance = v
	end
)

Section("🌀","MOVEMENT")

Toggle(
	"🌀",
	"Noclip",
	false,
	function(v)
		Config.Noclip = v
	end
)

--========================================================
-- ESP SYSTEM
--========================================================

local ESPObjects = {}

local function RemoveESP(player)

	local object = ESPObjects[player]

	if object then
		object:Destroy()
		ESPObjects[player] = nil
	end

end

local function CreateESP(player)

	local Billboard = Instance.new("BillboardGui")
	Billboard.Name = "PlayerESP"
	Billboard.Size = UDim2.fromOffset(220,90)
	Billboard.AlwaysOnTop = true
	Billboard.LightInfluence = 0
	Billboard.Parent = Gui

	local Name = Instance.new("TextLabel")
	Name.Name = "PlayerName"
	Name.Size = UDim2.new(1,0,0,23)
	Name.BackgroundTransparency = 1
	Name.TextColor3 = Colors.Pink
	Name.TextStrokeTransparency = 0
	Name.TextStrokeColor3 = Color3.new(0,0,0)
	Name.Font = Enum.Font.GothamBold
	Name.TextSize = 14
	Name.Parent = Billboard

	local Distance = Instance.new("TextLabel")
	Distance.Name = "Distance"
	Distance.Size = UDim2.new(1,0,0,18)
	Distance.Position = UDim2.fromOffset(0,23)
	Distance.BackgroundTransparency = 1
	Distance.TextColor3 = Colors.White
	Distance.TextStrokeTransparency = 0
	Distance.Font = Enum.Font.Gotham
	Distance.TextSize = 11
	Distance.Parent = Billboard

	local Back = Instance.new("Frame")
	Back.Size = UDim2.new(.70,0,0,7)
	Back.Position = UDim2.new(.15,0,0,47)
	Back.BackgroundColor3 = Color3.fromRGB(45,20,55)
	Back.BorderSizePixel = 0
	Back.Parent = Billboard

	local BackCorner = Instance.new("UICorner")
	BackCorner.CornerRadius = UDim.new(1,0)
	BackCorner.Parent = Back

	local Health = Instance.new("Frame")
	Health.Name = "Health"
	Health.Size = UDim2.new(1,0,1,0)
	Health.BackgroundColor3 = Colors.Green
	Health.BorderSizePixel = 0
	Health.Parent = Back

	local HealthCorner = Instance.new("UICorner")
	HealthCorner.CornerRadius = UDim.new(1,0)
	HealthCorner.Parent = Health

	local HealthText = Instance.new("TextLabel")
	HealthText.Size = UDim2.new(1,0,0,18)
	HealthText.Position = UDim2.fromOffset(0,56)
	HealthText.BackgroundTransparency = 1
	HealthText.TextColor3 = Colors.White
	HealthText.TextStrokeTransparency = 0
	HealthText.Font = Enum.Font.Gotham
	HealthText.TextSize = 10
	HealthText.Parent = Billboard

	ESPObjects[player] = Billboard

	return Billboard
end

local function UpdateESP(player)

	if player == LocalPlayer then
		return
	end

	if not Config.ESP then

		if ESPObjects[player] then
			ESPObjects[player].Enabled = false
		end

		return
	end

	if not IsAlive(player) then
		RemoveESP(player)
		return
	end

	local head = Head(player)

	if not head then
		return
	end

	local distance = GetDistance(player)

	if distance > Config.ESPDistance then

		if ESPObjects[player] then
			ESPObjects[player].Enabled = false
		end

		return
	end

	local Billboard = ESPObjects[player]

	if not Billboard then
		Billboard = CreateESP(player)
	end

	Billboard.Adornee = head
	Billboard.Enabled = true

	local humanoid = Humanoid(player)

	Billboard.PlayerName.Text =
		player.DisplayName ..
		"  @" ..
		player.Name

	Billboard.Distance.Text =
		math.floor(distance) ..
		" studs"

	local percent =
		math.clamp(
			humanoid.Health /
			math.max(humanoid.MaxHealth,1),
			0,
			1
		)

	Billboard.Health.Size =
		UDim2.new(
			percent,
			0,
			1,
			0
		)

	Billboard.HealthText.Text =
		math.floor(humanoid.Health) ..
		" / " ..
		math.floor(humanoid.MaxHealth)

	if percent <= .25 then
		Billboard.Health.BackgroundColor3 =
			Colors.Red
	elseif percent <= .5 then
		Billboard.Health.BackgroundColor3 =
			Colors.Yellow
	else
		Billboard.Health.BackgroundColor3 =
			Colors.Green
	end

end

--========================================================
-- SKELETON
--========================================================

local Skeletons = {}

local R6Bones = {
	{"Head","Torso"},
	{"Torso","Left Arm"},
	{"Torso","Right Arm"},
	{"Torso","Left Leg"},
	{"Torso","Right Leg"},
}

local R15Bones = {
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

local function RemoveSkeleton(player)

	local folder = Skeletons[player]

	if folder then

		for _,object in ipairs(folder:GetChildren()) do

			if object:IsA("Attachment") then
				object:Destroy()
			end

		end

		folder:Destroy()

		Skeletons[player] = nil
	end

end

local function MakeBone(folder,a,b)

	if not a or not b then
		return
	end

	local A = Instance.new("Attachment")
	A.Name = "SkeletonAttachment"
	A.Parent = a

	local B = Instance.new("Attachment")
	B.Name = "SkeletonAttachment"
	B.Parent = b

	local Beam = Instance.new("Beam")
	Beam.Attachment0 = A
	Beam.Attachment1 = B
	Beam.Width0 = .045
	Beam.Width1 = .045
	Beam.FaceCamera = true
	Beam.LightEmission = 1
	Beam.Color = ColorSequence.new(
		Colors.Pink
	)
	Beam.Parent = folder

end

local function UpdateSkeleton(player)

	if player == LocalPlayer then
		return
	end

	if not Config.Skeleton then
		RemoveSkeleton(player)
		return
	end

	if not IsAlive(player) then
		RemoveSkeleton(player)
		return
	end

	if GetDistance(player) >
		Config.SkeletonDistance then

		RemoveSkeleton(player)
		return
	end

	-- Só recria quando necessário.
	if Skeletons[player] then
		return
	end

	local character = Character(player)
	local humanoid = Humanoid(player)

	if not character or not humanoid then
		return
	end

	local folder = Instance.new("Folder")
	folder.Name = "Skeleton_"..player.Name
	folder.Parent = character

	local bones

	if humanoid.RigType ==
		Enum.HumanoidRigType.R15 then

		bones = R15Bones

	else

		bones = R6Bones

	end

	for _,pair in ipairs(bones) do

		local A = character:FindFirstChild(pair[1])
		local B = character:FindFirstChild(pair[2])

		MakeBone(folder,A,B)

	end

	Skeletons[player] = folder

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

local function IsVisible(player)

	local head = Head(player)

	if not head then
		return false
	end

	local character = Character(player)
	local myCharacter = Character(LocalPlayer)

	local camera = workspace.CurrentCamera

	if not camera then
		return false
	end

	local params = RaycastParams.new()

	params.FilterType =
		Enum.RaycastFilterType.Exclude

	params.FilterDescendantsInstances = {
		myCharacter
	}

	local direction =
		head.Position -
		camera.CFrame.Position

	local result =
		workspace:Raycast(
			camera.CFrame.Position,
			direction,
			params
		)

	if not result then
		return true
	end

	return result.Instance:IsDescendantOf(character)

end

local function UpdateCheck(player)

	if player == LocalPlayer then
		return
	end

	if not Config.AllCheck or
		not IsAlive(player) then

		RemoveCheck(player)
		return
	end

	if GetDistance(player) >
		Config.AllCheckDistance then

		RemoveCheck(player)
		return
	end

	local character = Character(player)

	if not character then
		return
	end

	local highlight = Checks[player]

	if not highlight then

		highlight = Instance.new("Highlight")
		highlight.Name = "AllCheck"
		highlight.DepthMode =
			Enum.HighlightDepthMode.AlwaysOnTop
		highlight.FillTransparency = .82
		highlight.OutlineTransparency = .05
		highlight.Parent = character

		Checks[player] = highlight

	end

	highlight.Adornee = character

	if IsVisible(player) then

		highlight.FillColor = Colors.Green
		highlight.OutlineColor = Colors.Green

	else

		highlight.FillColor = Colors.Red
		highlight.OutlineColor = Colors.Red

	end

end

--========================================================
-- AIM ASSIST
--========================================================

local function FindTarget()

	local camera = workspace.CurrentCamera

	if not camera then
		return nil
	end

	local center =
		Vector2.new(
			camera.ViewportSize.X/2,
			camera.ViewportSize.Y/2
		)

	local bestPlayer = nil
	local bestScreenDistance = math.huge

	for _,player in ipairs(
		Players:GetPlayers()
	) do

		if player ~= LocalPlayer
			and IsAlive(player) then

			local distance =
				GetDistance(player)

			if distance <=
				Config.AimDistance then

				local head = Head(player)

				if head then

					local screen,visible =
						camera:WorldToViewportPoint(
							head.Position
						)

					if visible and screen.Z > 0 then

						local point =
							Vector2.new(
								screen.X,
								screen.Y
							)

						local screenDistance =
							(point-center).Magnitude

						if screenDistance <=
							Config.AimFOV then

							local allowed = true

							if Config.AllCheck then
								allowed =
									IsVisible(player)
							end

							if allowed and
								screenDistance <
								bestScreenDistance then

								bestScreenDistance =
									screenDistance

								bestPlayer =
									player

							end

						end

					end

				end

			end

		end

	end

	return bestPlayer

end

RunService:BindToRenderStep(
	"NeonAimAssist",
	Enum.RenderPriority.Camera.Value+1,
	function()

		if not Config.Aim then
			return
		end

		local camera =
			workspace.CurrentCamera

		if not camera then
			return
		end

		local target =
			FindTarget()

		if not target then
			return
		end

		local head =
			Head(target)

		if not head then
			return
		end

		local targetCFrame =
			CFrame.lookAt(
				camera.CFrame.Position,
				head.Position
			)

		camera.CFrame =
			camera.CFrame:Lerp(
				targetCFrame,
				Config.AimSmooth
			)

	end
)

--========================================================
-- NOCLIP
--========================================================

RunService.Stepped:Connect(function()

	if not Config.Noclip then
		return
	end

	local character =
		Character(LocalPlayer)

	if not character then
		return
	end

	for _,object in ipairs(
		character:GetDescendants()
	) do

		if object:IsA("BasePart") then
			object.CanCollide = false
		end

	end

end)

--========================================================
-- UPDATE
--========================================================

local SkeletonTimer = 0

RunService.RenderStepped:Connect(function(dt)

	for _,player in ipairs(
		Players:GetPlayers()
	) do

		if player ~= LocalPlayer then

			UpdateESP(player)
			UpdateCheck(player)
			UpdateSkeleton(player)

		end

	end

	SkeletonTimer += dt

	-- Remove Skeleton quando sai do limite
	if SkeletonTimer >= .15 then

		SkeletonTimer = 0

		for player,folder in pairs(Skeletons) do

			if not Config.Skeleton
				or not IsAlive(player)
				or GetDistance(player) >
					Config.SkeletonDistance then

				RemoveSkeleton(player)

			end

		end

	end

	local camera =
		workspace.CurrentCamera

	if camera then

		FOV.Position =
			UDim2.fromOffset(
				camera.ViewportSize.X/2,
				camera.ViewportSize.Y/2
			)

	end

end)

--========================================================
-- PLAYER / RESPAWN
--========================================================

local function SetupPlayer(player)

	if player == LocalPlayer then
		return
	end

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

for _,player in ipairs(
	Players:GetPlayers()
) do

	SetupPlayer(player)

end

Players.PlayerAdded:Connect(
	SetupPlayer
)

Players.PlayerRemoving:Connect(function(player)

	RemoveESP(player)
	RemoveSkeleton(player)
	RemoveCheck(player)

end)

--========================================================
-- ARRASTAR PAINEL
--========================================================

local Dragging = false
local DragStart
local StartPosition

Top.InputBegan:Connect(function(input)

	if input.UserInputType ==
		Enum.UserInputType.MouseButton1 then

		Dragging = true
		DragStart = input.Position
		StartPosition = Main.Position

	end

end)

Top.InputEnded:Connect(function(input)

	if input.UserInputType ==
		Enum.UserInputType.MouseButton1 then

		Dragging = false

	end

end)

UserInputService.InputChanged:Connect(function(input)

	if not Dragging then
		return
	end

	if input.UserInputType ~=
		Enum.UserInputType.MouseMovement then
		return
	end

	local delta =
		input.Position - DragStart

	Main.Position =
		UDim2.new(
			StartPosition.X.Scale,
			StartPosition.X.Offset + delta.X,

			StartPosition.Y.Scale,
			StartPosition.Y.Offset + delta.Y
		)

end)

--========================================================
-- RIGHT SHIFT
--========================================================

UserInputService.InputBegan:Connect(
	function(input,processed)

		if processed then
			return
		end

		if input.KeyCode ==
			Enum.KeyCode.RightShift then

			Main.Visible =
				not Main.Visible

		end

	end
)

print("====================================")
print(" NEON PLAYER TRAINING")
print(" ESP: READY")
print(" SKELETON R6/R15: READY")
print(" ALL CHECK: READY")
print(" AIM ASSIST: READY")
print("====================================")
