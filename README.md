
-- BLUE RED CONTROL - SCRIPT COMPLETO
-- Fly + Fly Speed 1-1000 + WalkSpeed 1-1000
-- Noclip + Anti Void + Teleporte + Instant Proximity
-- Painel arrastavel + RightShift + botao BR
-- Reconexao automatica apos respawn/reinicio da partida

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local PPS = game:GetService("ProximityPromptService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local oldGui = playerGui:FindFirstChild("BlueRedControl")
if oldGui then oldGui:Destroy() end

-- ESTADO DOS RECURSOS
local character, humanoid, root
local bindingCharacter
local generation = 0

local flyEnabled = false
local speedEnabled = false
local noclipEnabled = false
local voidEnabled = false
local promptEnabled = false
local panelHidden = false

local flySpeed = 60
local walkSpeed = 16
local originalWalkSpeed = 16
local originalAutoRotate = true

local savedCFrame
local safePosition

local attachment, velocity, alignOrientation
local flyConnection, noclipConnection

local collisionOriginals = {}
local promptOriginals = {}

local flyButton, speedButton
local noclipButton, voidButton, promptButton

-- VALORES APLICADOS DIRETAMENTE, SEM LIMITE DE 80 OU 200
local function getFlySpeed()
    return math.clamp(flySpeed, 1, 1000)
end

local function getWalkSpeed()
    return math.clamp(walkSpeed, 1, 1000)
end

local function valid()
    return character and character.Parent
        and humanoid and humanoid.Parent
        and humanoid.Health > 0
        and root and root.Parent
end

local function updateSpeedButton()
    if speedButton then
        speedButton.Text = speedEnabled
            and ("WalkSpeed: ON (" .. getWalkSpeed() .. ")")
            or "WalkSpeed: OFF"
    end
end

local function applyWalkSpeed()
    if not humanoid or not humanoid.Parent then return end

    if flyEnabled then
        humanoid.WalkSpeed = 0
    elseif speedEnabled then
        humanoid.WalkSpeed = getWalkSpeed()
    else
        humanoid.WalkSpeed = originalWalkSpeed
    end

    updateSpeedButton()
end

-- FLY: LIMPEZA
local function clearFlyObjects()
    if flyConnection then
        flyConnection:Disconnect()
        flyConnection = nil
    end

    if velocity then
        velocity:Destroy()
        velocity = nil
    end

    if alignOrientation then
        alignOrientation:Destroy()
        alignOrientation = nil
    end

    if attachment then
        attachment:Destroy()
        attachment = nil
    end

    if humanoid and humanoid.Parent then
        humanoid.PlatformStand = false
        humanoid.AutoRotate = originalAutoRotate

        pcall(function()
            humanoid:ChangeState(Enum.HumanoidStateType.GettingUp)
        end)
    end

    if root and root.Parent then
        root.AssemblyAngularVelocity = Vector3.zero
        root.AssemblyLinearVelocity = Vector3.zero
    end

    applyWalkSpeed()
end

local function stopFly()
    flyEnabled = false
    clearFlyObjects()

    if flyButton then
        flyButton.Text = "Fly: OFF [X]"
    end
end

-- Verifica o estado real das teclas a cada atualizacao.
-- Assim, nao precisa soltar e apertar W novamente.
local function isKeyDown(keyCode)
    if UIS:GetFocusedTextBox() then
        return false
    end

    local ok, down = pcall(function()
        return UIS:IsKeyDown(keyCode)
    end)

    return ok and down
end

local function startFly()
    if not flyEnabled or not valid() or flyConnection then
        return
    end

    originalAutoRotate = humanoid.AutoRotate

    humanoid.WalkSpeed = 0
    humanoid.AutoRotate = false
    humanoid.PlatformStand = false

    root.AssemblyLinearVelocity = Vector3.zero
    root.AssemblyAngularVelocity = Vector3.zero

    attachment = Instance.new("Attachment")
    attachment.Name = "BR_FlyAttachment"
    attachment.Parent = root

    velocity = Instance.new("LinearVelocity")
    velocity.Name = "BR_FlyVelocity"
    velocity.Attachment0 = attachment
    velocity.RelativeTo = Enum.ActuatorRelativeTo.World
    velocity.VelocityConstraintMode =
        Enum.VelocityConstraintMode.Vector
    velocity.VectorVelocity = Vector3.zero
    velocity.ForceLimitsEnabled = false
    velocity.Parent = root

    alignOrientation = Instance.new("AlignOrientation")
    alignOrientation.Name = "BR_FlyOrientation"
    alignOrientation.Mode =
        Enum.OrientationAlignmentMode.OneAttachment
    alignOrientation.Attachment0 = attachment
    alignOrientation.RigidityEnabled = false
    alignOrientation.Responsiveness = 25
    alignOrientation.MaxTorque = 100000000
    alignOrientation.Parent = root

    pcall(function()
        humanoid:ChangeState(Enum.HumanoidStateType.Physics)
    end)

    flyConnection = RunService.Heartbeat:Connect(function()
        if not flyEnabled or not valid()
            or not velocity or not velocity.Parent
            or not attachment or not attachment.Parent then

            -- Se os objetos forem removidos durante um reset,
            -- limpa os recursos para permitir uma nova conexao.
            clearFlyObjects()
            return
        end

        local camera = workspace.CurrentCamera
        if not camera then return end

        local direction = Vector3.zero
        local look = camera.CFrame.LookVector
        local right = camera.CFrame.RightVector

        -- Lê o teclado diretamente, inclusive se W ja estava pressionado.
        if isKeyDown(Enum.KeyCode.W) then direction += look end
        if isKeyDown(Enum.KeyCode.S) then direction -= look end
        if isKeyDown(Enum.KeyCode.D) then direction += right end
        if isKeyDown(Enum.KeyCode.A) then direction -= right end

        -- Subir e descer verticalmente também funciona.
        if isKeyDown(Enum.KeyCode.Space) then
            direction += Vector3.yAxis
        end
        if isKeyDown(Enum.KeyCode.LeftControl) then
            direction -= Vector3.yAxis
        end

        if direction.Magnitude > 0 then
            direction = direction.Unit
        end

        -- O valor escolhido na barra e aplicado sem limite artificial.
        velocity.VectorVelocity = direction * getFlySpeed()

        -- Mantém o corpo ereto sem impedir a câmera de olhar para baixo.
        local flatLook = Vector3.new(look.X, 0, look.Z)

        if flatLook.Magnitude > 0.01 and alignOrientation then
            alignOrientation.CFrame =
                CFrame.lookAt(Vector3.zero, flatLook.Unit)
        end
    end)

    if flyButton then
        flyButton.Text = "Fly: ON [X]"
    end
end

local function toggleFly()
    if flyEnabled then
        stopFly()
    else
        flyEnabled = true
        if flyButton then flyButton.Text = "Fly: ON [X]" end
        applyWalkSpeed()
        startFly()
    end
end

-- NOCLIP
local function restoreCollisions()
    for part, original in pairs(collisionOriginals) do
        if part and part.Parent then
            part.CanCollide = original
        end
    end

    table.clear(collisionOriginals)
end

local function clearNoclipObjects()
    if noclipConnection then
        noclipConnection:Disconnect()
        noclipConnection = nil
    end

    restoreCollisions()
end

local function stopNoclip()
    noclipEnabled = false
    clearNoclipObjects()

    if noclipButton then
        noclipButton.Text = "Noclip: OFF"
    end
end

local function startNoclip()
    if not noclipEnabled or not valid() or noclipConnection then
        return
    end

    noclipConnection = RunService.Stepped:Connect(function()
        if not valid() then return end

        for _, part in ipairs(character:GetDescendants()) do
            if part:IsA("BasePart") then
                if collisionOriginals[part] == nil then
                    collisionOriginals[part] = part.CanCollide
                end
                part.CanCollide = false
            end
        end
    end)

    if noclipButton then
        noclipButton.Text = "Noclip: ON"
    end
end

local function toggleNoclip()
    if noclipEnabled then
        stopNoclip()
    else
        noclipEnabled = true
        startNoclip()
    end
end

-- RECONEXAO DO PERSONAGEM APOS RESET
local function bindCharacter(char)
    generation += 1
    local myGeneration = generation
    bindingCharacter = char

    -- Desconecta objetos ligados ao corpo anterior,
    -- mas preserva os estados escolhidos no painel.
    clearFlyObjects()
    clearNoclipObjects()

    character = nil
    humanoid = nil
    root = nil

    task.spawn(function()
        local newHumanoid = char:WaitForChild("Humanoid", 15)
        local newRoot = char:WaitForChild("HumanoidRootPart", 15)

        if not newHumanoid or not newRoot then return end
        if myGeneration ~= generation then return end
        if player.Character ~= char then return end

        character = char
        humanoid = newHumanoid
        root = newRoot
        bindingCharacter = nil

        originalWalkSpeed = humanoid.WalkSpeed
        safePosition = root.Position

        applyWalkSpeed()

        if flyEnabled then
            startFly()
        end

        if noclipEnabled then
            startNoclip()
        end
    end)
end

player.CharacterRemoving:Connect(function(char)
    if character ~= char and bindingCharacter ~= char then
        return
    end

    generation += 1

    clearFlyObjects()
    clearNoclipObjects()

    character = nil
    humanoid = nil
    root = nil
    bindingCharacter = nil

    -- flyEnabled, speedEnabled e noclipEnabled continuam ativos.
end)

player.CharacterAdded:Connect(bindCharacter)

-- INTERFACE
local gui = Instance.new("ScreenGui")
gui.Name = "BlueRedControl"
gui.ResetOnSpawn = false
gui.DisplayOrder = 20
gui.Parent = playerGui

local panel = Instance.new("ScrollingFrame")
panel.Name = "Panel"
panel.Size = UDim2.fromOffset(320, 475)
panel.Position = UDim2.new(0, 25, 0.5, -237)
panel.BackgroundColor3 = Color3.fromRGB(23, 26, 36)
panel.BorderSizePixel = 0
panel.Active = true
panel.ScrollBarThickness = 5
panel.CanvasSize = UDim2.fromOffset(0, 450)
panel.Parent = gui
Instance.new("UICorner", panel).CornerRadius = UDim.new(0, 12)

local outline = Instance.new("UIStroke")
outline.Color = Color3.fromRGB(220, 45, 65)
outline.Thickness = 1.5
outline.Parent = panel

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -10, 0, 40)
title.BackgroundTransparency = 1
title.Text = "BLUE RED CONTROL"
title.TextColor3 = Color3.fromRGB(255, 75, 90)
title.Font = Enum.Font.GothamBold
title.TextSize = 17
title.Parent = panel

-- Arrastar painel
do
    local dragging = false
    local startInput
    local startPosition

    title.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            startInput = input.Position
            startPosition = panel.Position
        end
    end)

    UIS.InputChanged:Connect(function(input)
        if dragging and (
            input.UserInputType == Enum.UserInputType.MouseMovement
            or input.UserInputType == Enum.UserInputType.Touch
        ) then
            local delta = input.Position - startInput

            panel.Position = UDim2.new(
                startPosition.X.Scale,
                startPosition.X.Offset + delta.X,
                startPosition.Y.Scale,
                startPosition.Y.Offset + delta.Y
            )
        end
    end)

    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
end

local function makeButton(text, y, callback)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(1, -24, 0, 34)
    b.Position = UDim2.fromOffset(12, y)
    b.BackgroundColor3 = Color3.fromRGB(45, 49, 63)
    b.TextColor3 = Color3.new(1, 1, 1)
    b.Font = Enum.Font.GothamSemibold
    b.TextSize = 12
    b.Text = text
    b.Parent = panel

    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 6)
    b.Activated:Connect(callback)

    return b
end

local function makeSlider(labelText, y, initialValue, callback)
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, -24, 0, 22)
    label.Position = UDim2.fromOffset(12, y)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(1, 1, 1)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Font = Enum.Font.GothamSemibold
    label.TextSize = 12
    label.Parent = panel

    local bar = Instance.new("Frame")
    bar.Size = UDim2.new(1, -30, 0, 10)
    bar.Position = UDim2.fromOffset(15, y + 29)
    bar.BackgroundColor3 = Color3.fromRGB(65, 68, 82)
    bar.BorderSizePixel = 0
    bar.Active = true
    bar.Parent = panel
    Instance.new("UICorner", bar).CornerRadius = UDim.new(1, 0)

    local fill = Instance.new("Frame")
    fill.BackgroundColor3 = Color3.fromRGB(230, 50, 70)
    fill.BorderSizePixel = 0
    fill.Parent = bar
    Instance.new("UICorner", fill).CornerRadius = UDim.new(1, 0)

    local knob = Instance.new("TextButton")
    knob.Size = UDim2.fromOffset(18, 18)
    knob.AnchorPoint = Vector2.new(0.5, 0.5)
    knob.BackgroundColor3 = Color3.new(1, 1, 1)
    knob.Text = ""
    knob.Parent = bar
    Instance.new("UICorner", knob).CornerRadius = UDim.new(1, 0)

    local value = initialValue
    local dragging = false

    local function refresh()
        local ratio = (value - 1) / 999

        fill.Size = UDim2.new(ratio, 0, 1, 0)
        knob.Position = UDim2.new(ratio, 0, 0.5, 0)
        label.Text = labelText .. ": " .. value .. " / 1000"

        callback(value)
    end

    local function updateFromX(x)
        local width = math.max(bar.AbsoluteSize.X, 1)
        local ratio = math.clamp(
            (x - bar.AbsolutePosition.X) / width, 0, 1
        )

        value = math.clamp(math.floor(ratio * 999 + 1.5), 1, 1000)
        refresh()
    end

    bar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            updateFromX(input.Position.X)
        end
    end)

    knob.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
        end
    end)

    UIS.InputChanged:Connect(function(input)
        if dragging and (
            input.UserInputType == Enum.UserInputType.MouseMovement
            or input.UserInputType == Enum.UserInputType.Touch
        ) then
            updateFromX(input.Position.X)
        end
    end)

    UIS.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)

    refresh()
end

flyButton = makeButton("Fly: OFF [X]", 45, toggleFly)

makeSlider("Fly Speed", 85, flySpeed, function(value)
    flySpeed = value
end)

speedButton = makeButton("WalkSpeed: OFF", 137, function()
    speedEnabled = not speedEnabled
    applyWalkSpeed()
end)

makeSlider("WalkSpeed", 177, walkSpeed, function(value)
    walkSpeed = value
    applyWalkSpeed()
end)

makeButton("Salvar posição [Z]", 228, function()
    if valid() then savedCFrame = root.CFrame end
end)

makeButton("Teleportar [Y]", 267, function()
    if valid() and savedCFrame then
        root.AssemblyLinearVelocity = Vector3.zero
        root.AssemblyAngularVelocity = Vector3.zero
        root.CFrame = savedCFrame
    end
end)

promptButton = makeButton("Instant Proximity: OFF", 306, function()
    promptEnabled = not promptEnabled

    if promptEnabled then
        promptButton.Text = "Instant Proximity: ON"

        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("ProximityPrompt") then
                if promptOriginals[obj] == nil then
                    promptOriginals[obj] = obj.HoldDuration
                end
                obj.HoldDuration = 0
            end
        end
    else
        promptButton.Text = "Instant Proximity: OFF"

        for prompt, original in pairs(promptOriginals) do
            if prompt and prompt.Parent then
                prompt.HoldDuration = original
            end
        end

        table.clear(promptOriginals)
    end
end)

voidButton = makeButton("Anti Void: OFF", 345, function()
    voidEnabled = not voidEnabled
    voidButton.Text = voidEnabled
        and "Anti Void: ON"
        or "Anti Void: OFF"
end)

noclipButton = makeButton("Noclip: OFF", 384, toggleNoclip)

-- Botao BR
local reopen = Instance.new("TextButton")
reopen.Name = "BRReopen"
reopen.Size = UDim2.fromOffset(48, 48)
reopen.Position = UDim2.new(0, 12, 0.5, -24)
reopen.Text = "BR"
reopen.Font = Enum.Font.GothamBold
reopen.TextSize = 18
reopen.TextColor3 = Color3.new(1, 1, 1)
reopen.BackgroundColor3 = Color3.fromRGB(220, 45, 65)
reopen.Visible = false
reopen.Parent = gui
Instance.new("UICorner", reopen).CornerRadius = UDim.new(1, 0)

local function togglePanel()
    panelHidden = not panelHidden
    panel.Visible = not panelHidden
    reopen.Visible = panelHidden
end

reopen.Activated:Connect(togglePanel)

-- Atalhos do teclado
UIS.InputBegan:Connect(function(input, processed)
    if processed then return end

    local key = input.KeyCode

    if key == Enum.KeyCode.X then
        toggleFly()
    elseif key == Enum.KeyCode.Z then
        if valid() then savedCFrame = root.CFrame end
    elseif key == Enum.KeyCode.Y then
        if valid() and savedCFrame then
            root.AssemblyLinearVelocity = Vector3.zero
            root.AssemblyAngularVelocity = Vector3.zero
            root.CFrame = savedCFrame
        end
    elseif key == Enum.KeyCode.RightShift then
        togglePanel()
    end
end)

-- Novos prompts tambem respeitam a opcao ativa.
PPS.PromptShown:Connect(function(prompt)
    if promptEnabled and prompt:IsA("ProximityPrompt") then
        if promptOriginals[prompt] == nil then
            promptOriginals[prompt] = prompt.HoldDuration
        end
        prompt.HoldDuration = 0
    end
end)

-- Monitor de respawn e de valores de velocidade.
local elapsed = 0

RunService.Heartbeat:Connect(function(dt)
    local current = player.Character

    if current and current ~= character
        and current ~= bindingCharacter then
        bindCharacter(current)
    end

    if not valid() then return end

    elapsed += dt

    -- Verifica periodicamente sem criar novas conexoes a cada frame.
    if elapsed >= 0.3 then
        elapsed = 0

        if flyEnabled and not flyConnection then
            startFly()
        end

        if noclipEnabled and not noclipConnection then
            startNoclip()
        end

        if speedEnabled and not flyEnabled
            and humanoid.WalkSpeed ~= getWalkSpeed() then
            humanoid.WalkSpeed = getWalkSpeed()
        end
    end

    if root.Position.Y > -100 then
        safePosition = root.Position
    elseif voidEnabled and safePosition then
        root.AssemblyLinearVelocity = Vector3.zero
        root.AssemblyAngularVelocity = Vector3.zero
        root.CFrame = CFrame.new(safePosition + Vector3.new(0, 5, 0))
    end
end)

if player.Character then
    bindCharacter(player.Character)
end

print("BLUE RED CONTROL carregado.")
