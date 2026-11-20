
--==================================================
-- BUILD 06 | UNIFIED SYSTEM
-- COMMAND PANEL + PLAYER IDENTIFICATION
-- LocalScript: StarterPlayerScripts
--==================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TextChatService = game:GetService("TextChatService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local CONFIG = {
    MaxDistance = 500,
    UpdateInterval = 0.15,
    Color = Color3.fromRGB(220, 35, 45),
    Background = Color3.fromRGB(18, 20, 25),
    MessageDelay = 0.5,
}

local Enabled = false
local PanelOpen = false
local Tracked = {}
local Timer = 0
local Sending = false

--==================================================
-- LIMPEZA
--==================================================

local old = PlayerGui:FindFirstChild("BUILD06_UNIFIED")
if old then old:Destroy() end

--==================================================
-- GUI
--==================================================

local GUI = Instance.new("ScreenGui")
GUI.Name = "BUILD06_UNIFIED"
GUI.ResetOnSpawn = false
GUI.DisplayOrder = 100
GUI.Parent = PlayerGui

local function Corner(object, radius)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, radius)
    c.Parent = object
    return c
end

local function Outline(object, color, thickness)
    local s = Instance.new("UIStroke")
    s.Color = color
    s.Thickness = thickness or 1
    s.Parent = object
    return s
end

local function MakeLabel(parent, text, size, position, textSize)
    local label = Instance.new("TextLabel")
    label.BackgroundTransparency = 1
    label.Text = text
    label.Size = size
    label.Position = position
    label.TextColor3 = Color3.fromRGB(235, 235, 240)
    label.Font = Enum.Font.GothamBold
    label.TextSize = textSize or 14
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = parent
    return label
end

--==================================================
-- BOTAO FLUTUANTE V
--==================================================

local Toggle = Instance.new("TextButton")
Toggle.Name = "V_Toggle"
Toggle.Size = UDim2.fromOffset(58, 58)
Toggle.Position = UDim2.new(1, -78, 0.65, 0)
Toggle.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
Toggle.BackgroundTransparency = 0.2
Toggle.Text = "V"
Toggle.TextColor3 = CONFIG.Color
Toggle.TextSize = 30
Toggle.Font = Enum.Font.GothamBold
Toggle.AutoButtonColor = false
Toggle.Parent = GUI

Corner(Toggle, 100)
local toggleStroke = Outline(Toggle, CONFIG.Color, 1.5)

--==================================================
-- PAINEL
--==================================================

local Panel = Instance.new("Frame")
Panel.Name = "CommandPanel"
Panel.Size = UDim2.fromOffset(290, 390)
Panel.Position = UDim2.new(0.5, -145, 0.5, -195)
Panel.BackgroundColor3 = CONFIG.Background
Panel.BackgroundTransparency = 0.05
Panel.Visible = false
Panel.Parent = GUI

Corner(Panel, 12)
Outline(Panel, CONFIG.Color, 1)

local Header = Instance.new("Frame")
Header.Size = UDim2.new(1, 0, 0, 48)
Header.BackgroundColor3 = Color3.fromRGB(27, 29, 35)
Header.Parent = Panel
Corner(Header, 12)

MakeLabel(
    Header,
    "BUILD 06  //  CONTROL",
    UDim2.new(1, -55, 1, 0),
    UDim2.fromOffset(14, 0),
    14
)

local Close = Instance.new("TextButton")
Close.Size = UDim2.fromOffset(40, 40)
Close.Position = UDim2.new(1, -44, 0, 4)
Close.BackgroundTransparency = 1
Close.Text = "×"
Close.TextColor3 = CONFIG.Color
Close.TextSize = 26
Close.Font = Enum.Font.GothamBold
Close.Parent = Header

local Status = MakeLabel(
    Panel,
    "STATUS: ONLINE",
    UDim2.new(1, -24, 0, 22),
    UDim2.fromOffset(12, 55),
    11
)
Status.TextColor3 = Color3.fromRGB(100, 210, 130)

MakeLabel(
    Panel,
    "COMMAND CENTER",
    UDim2.new(1, -24, 0, 22),
    UDim2.fromOffset(12, 83),
    12
)

--==================================================
-- LISTA DE COMANDOS
--==================================================

local CommandScroll = Instance.new("ScrollingFrame")
CommandScroll.Size = UDim2.new(1, -20, 0, 165)
CommandScroll.Position = UDim2.fromOffset(10, 110)
CommandScroll.BackgroundTransparency = 1
CommandScroll.BorderSizePixel = 0
CommandScroll.ScrollBarThickness = 3
CommandScroll.CanvasSize = UDim2.fromOffset(0, 0)
CommandScroll.Parent = Panel

local CommandLayout = Instance.new("UIListLayout")
CommandLayout.Padding = UDim.new(0, 6)
CommandLayout.Parent = CommandScroll

local Commands = {
    {name = "TIRO NA CARA", command = "tiro na cara", variants = true},
    {name = "RENDER", command = "render", variants = true},
    {name = "FACA", command = "facada", variants = true},
    {name = "TIRO NO PNEU", command = "tiro no pneu", variants = true},
    {name = "PD", command = "pd", variants = false},
}

--==================================================
-- ENVIO DE MENSAGENS
--==================================================

local function GetChatChannel()
    local channels = TextChatService:FindFirstChild("TextChannels")

    if not channels then
        return nil
    end

    return channels:FindFirstChild("RBXGeneral")
        or channels:FindFirstChild("RBXTeam")
end

local function SendCommand(command, repetitions)
    if Sending then
        Status.Text = "STATUS: WAIT"
        return
    end

    repetitions = repetitions or 1
    Sending = true

    task.spawn(function()
        local channel = GetChatChannel()

        if not channel then
            Status.Text = "STATUS: CHANNEL UNAVAILABLE"
            Sending = false
            return
        end

        for i = 1, repetitions do
            local success, err = pcall(function()
                channel:SendAsync(command)
            end)

            if not success then
                Status.Text = "STATUS: CHAT ERROR"
                warn("[BUILD06] Chat error:", err)
                Sending = false
                return
            end

            if i < repetitions then
                task.wait(CONFIG.MessageDelay)
            end
        end

        Status.Text = "STATUS: SENT " .. repetitions .. "X"
        Sending = false
    end)
end

local function CreateCommandButton(text, command, repetitions)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, -5, 0, 35)
    button.BackgroundColor3 = Color3.fromRGB(36, 39, 47)
    button.TextColor3 = Color3.fromRGB(235, 235, 240)
    button.Text = text
    button.TextSize = 11
    button.Font = Enum.Font.GothamBold
    button.AutoButtonColor = true
    button.Parent = CommandScroll

    Corner(button, 6)

    button.Activated:Connect(function()
        SendCommand(command, repetitions)
    end)

    return button
end

for _, item in ipairs(Commands) do
    if item.variants then
        CreateCommandButton(
            item.name .. " | 1X",
            item.command,
            1
        )

        CreateCommandButton(
            item.name .. " | 3X",
            item.command,
            3
        )
    else
        CreateCommandButton(
            item.name,
            item.command,
            1
        )
    end
end

local function UpdateCommandCanvas()
    CommandScroll.CanvasSize = UDim2.fromOffset(
        0,
        CommandLayout.AbsoluteContentSize.Y + 5
    )
end

CommandLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(
    UpdateCommandCanvas
)

--==================================================
-- PLAYER IDENTIFICATION
--==================================================

MakeLabel(
    Panel,
    "PLAYER IDENTIFICATION",
    UDim2.new(1, -24, 0, 22),
    UDim2.fromOffset(12, 285),
    12
)

local ScanStatus = MakeLabel(
    Panel,
    "ESP: DISABLED",
    UDim2.new(1, -24, 0, 20),
    UDim2.fromOffset(12, 310),
    11
)

local ScanButton = Instance.new("TextButton")
ScanButton.Size = UDim2.new(1, -24, 0, 34)
ScanButton.Position = UDim2.fromOffset(12, 340)
ScanButton.BackgroundColor3 = Color3.fromRGB(45, 35, 40)
ScanButton.Text = "TOGGLE PLAYER SCAN"
ScanButton.TextColor3 = CONFIG.Color
ScanButton.TextSize = 11
ScanButton.Font = Enum.Font.GothamBold
ScanButton.Parent = Panel

Corner(ScanButton, 6)

--==================================================
-- CRIAR IDENTIFICACAO
--==================================================

local function RemovePlayer(player)
    local data = Tracked[player]
    if not data then return end

    if data.Highlight then
        data.Highlight:Destroy()
    end

    if data.Billboard then
        data.Billboard:Destroy()
    end

    Tracked[player] = nil
end

local function SetupPlayer(player)
    if player == LocalPlayer then return end

    RemovePlayer(player)

    local character = player.Character
    if not character then return end

    local head = character:FindFirstChild("Head")
    if not head then return end

    local highlight = Instance.new("Highlight")
    highlight.Name = "BUILD06_Outline"
    highlight.Adornee = character
    highlight.DepthMode = Enum.HighlightDepthMode.Occluded
    highlight.OutlineColor = CONFIG.Color
    highlight.OutlineTransparency = 0
    highlight.FillTransparency = 1
    highlight.Enabled = false
    highlight.Parent = character

    local billboard = Instance.new("BillboardGui")
    billboard.Name = "BUILD06_Identity"
    billboard.Adornee = head
    billboard.Size = UDim2.fromOffset(200, 35)
    billboard.StudsOffset = Vector3.new(0, 2.8, 0)
    billboard.AlwaysOnTop = false
    billboard.Enabled = false
    billboard.Parent = PlayerGui

    local label = Instance.new("TextLabel")
    label.Size = UDim2.fromScale(1, 1)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.TextStrokeTransparency = 0.2
    label.TextSize = 13
    label.Font = Enum.Font.GothamBold
    label.Text = player.DisplayName
    label.Parent = billboard

    Tracked[player] = {
        Highlight = highlight,
        Billboard = billboard,
        Label = label,
    }
end

--==================================================
-- LINHA DE VISAO
--==================================================

local function HasLineOfSight(character, targetPart)
    local camera = workspace.CurrentCamera

    if not camera or not targetPart then
        return false
    end

    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {
        LocalPlayer.Character
    }

    local origin = camera.CFrame.Position
    local direction = targetPart.Position - origin
    local result = workspace:Raycast(origin, direction, params)

    return result ~= nil
        and result.Instance:IsDescendantOf(character)
end

--==================================================
-- ATUALIZACAO DO ESP
--==================================================

local function UpdateESP()
    local localCharacter = LocalPlayer.Character
    local localRoot = localCharacter
        and localCharacter:FindFirstChild("HumanoidRootPart")

    for player, data in pairs(Tracked) do
        local character = player.Character
        local head = character and character:FindFirstChild("Head")
        local root = character and character:FindFirstChild("HumanoidRootPart")
        local humanoid = character and character:FindFirstChildOfClass("Humanoid")

        if not character or not head or not root
        or not humanoid or humanoid.Health <= 0
        or not localRoot then
            data.Highlight.Enabled = false
            data.Billboard.Enabled = false
            continue
        end

        local distance = (localRoot.Position - root.Position).Magnitude

        local visible = distance <= CONFIG.MaxDistance
            and HasLineOfSight(character, head)

        local active = Enabled and visible

        data.Highlight.Enabled = active
        data.Billboard.Enabled = active

        if active then
            data.Label.Text = string.format(
                "%s | %d STUDS",
                player.DisplayName,
                math.floor(distance + 0.5)
            )
        end
    end
end

--==================================================
-- CONTROLES
--==================================================

local function SetPanel(state)
    PanelOpen = state
    Panel.Visible = state
end

Toggle.Activated:Connect(function()
    SetPanel(not PanelOpen)
end)

Close.Activated:Connect(function()
    SetPanel(false)
end)

ScanButton.Activated:Connect(function()
    Enabled = not Enabled

    ScanStatus.Text = Enabled and "ESP: ENABLED" or "ESP: DISABLED"
    ScanButton.Text = Enabled
        and "DISABLE PLAYER SCAN"
        or "TOGGLE PLAYER SCAN"

    toggleStroke.Transparency = Enabled and 0 or 0.2

    if not Enabled then
        for _, data in pairs(Tracked) do
            data.Highlight.Enabled = false
            data.Billboard.Enabled = false
        end
    end
end)

--==================================================
-- ARRASTAR INTERFACES
--==================================================

local function MakeDraggable(object, handle)
    local dragging = false
    local dragStart
    local startPosition
    local dragInput

    handle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then

            dragging = true
            dragStart = input.Position
            startPosition = object.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    handle.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if dragging and input == dragInput then
            local delta = input.Position - dragStart

            object.Position = UDim2.new(
                startPosition.X.Scale,
                startPosition.X.Offset + delta.X,
                startPosition.Y.Scale,
                startPosition.Y.Offset + delta.Y
            )
        end
    end)
end

MakeDraggable(Toggle, Toggle)
MakeDraggable(Panel, Header)

--==================================================
-- REGISTRO DE JOGADORES
--==================================================

local function RegisterPlayer(player)
    if player == LocalPlayer then return end

    if player.Character then
        SetupPlayer(player)
    end

    player.CharacterAdded:Connect(function()
        task.wait(0.5)
        SetupPlayer(player)
    end)
end

for _, player in ipairs(Players:GetPlayers()) do
    RegisterPlayer(player)
end

Players.PlayerAdded:Connect(RegisterPlayer)
Players.PlayerRemoving:Connect(RemovePlayer)

--==================================================
-- LOOP PRINCIPAL
--==================================================

RunService.Heartbeat:Connect(function(deltaTime)
    Timer += deltaTime

    if Timer >= CONFIG.UpdateInterval then
        Timer = 0

        if Enabled then
            UpdateESP()
        end
    end
end)

print("[BUILD 06] UNIFIED SYSTEM ONLINE")
