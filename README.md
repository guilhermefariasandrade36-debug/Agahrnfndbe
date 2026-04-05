local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

-- CORES
local Colors = {
    Main = Color3.fromRGB(20, 20, 20),
    Accent = Color3.fromRGB(0, 170, 255),
    Text = Color3.fromRGB(255, 255, 255),
    Off = Color3.fromRGB(40, 40, 40),
    On = Color3.fromRGB(0, 170, 255)
}

-- ESTADOS
local Speed, Jump, FullBright, Noclip, ESP = false, false, false, false, false
local OriginalSpeed, OriginalJump
local ESPCache = {}

-- GUI
local gui = Instance.new("ScreenGui", game:GetService("CoreGui"))
gui.ResetOnSpawn = false

-- HEADER
local header = Instance.new("Frame", gui)
header.Size = UDim2.new(0, 220, 0, 35)
header.Position = UDim2.new(0, 50, 0, 50)
header.BackgroundColor3 = Colors.Main
header.BorderSizePixel = 0
header.Active = true
Instance.new("UICorner", header).CornerRadius = UDim.new(0, 8)

-- TÍTULO
local title = Instance.new("TextLabel", header)
title.Size = UDim2.new(1, -40, 1, 0)
title.Position = UDim2.new(0, 12, 0, 0)
title.Text = "PREMIUM HUB"
title.TextColor3 = Colors.Text
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.TextXAlignment = Enum.TextXAlignment.Left
title.BackgroundTransparency = 1

-- BOTÃO MINIMIZAR
local toggleBtn = Instance.new("TextButton", header)
toggleBtn.Size = UDim2.new(0, 25, 0, 25)
toggleBtn.Position = UDim2.new(1, -30, 0, 5)
toggleBtn.Text = "−"
toggleBtn.TextColor3 = Colors.Text
toggleBtn.BackgroundColor3 = Colors.Off
toggleBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", toggleBtn)

-- CONTAINER
local container = Instance.new("Frame", header)
container.Size = UDim2.new(1, 0, 0, 260)
container.Position = UDim2.new(0, 0, 1, 5)
container.BackgroundColor3 = Colors.Main
container.BackgroundTransparency = 0.1
Instance.new("UICorner", container)

local layout = Instance.new("UIListLayout", container)
layout.Padding = UDim.new(0, 6)
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
Instance.new("UIPadding", container).PaddingTop = UDim.new(0, 10)

-- TOGGLE
local function createToggle(text, callback)
    local b = Instance.new("TextButton", container)
    b.Size = UDim2.new(0.9, 0, 0, 32)
    b.Text = text
    b.Font = Enum.Font.GothamMedium
    b.TextSize = 13
    b.TextColor3 = Colors.Text
    b.BackgroundColor3 = Colors.Off
    b.AutoButtonColor = false
    Instance.new("UICorner", b)

    local stroke = Instance.new("UIStroke", b)
    stroke.Color = Colors.Accent
    stroke.Thickness = 0

    local on = false
    b.MouseButton1Click:Connect(function()
        on = not on
        callback(on)
        TweenService:Create(b, TweenInfo.new(0.2), {
            BackgroundColor3 = on and Colors.On or Colors.Off,
            TextColor3 = on and Color3.new(0,0,0) or Colors.Text
        }):Play()
        TweenService:Create(stroke, TweenInfo.new(0.2), {Thickness = on and 1.5 or 0}):Play()
    end)
end

-- DRAG MOBILE
local dragging = false
local dragStart = nil
local startPos = nil

header.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch 
	or input.UserInputType == Enum.UserInputType.MouseButton1 then
		
		dragging = true
		dragStart = input.Position
		startPos = header.Position
	end
end)

header.InputChanged:Connect(function(input)
	if (input.UserInputType == Enum.UserInputType.Touch 
	or input.UserInputType == Enum.UserInputType.MouseMovement) and dragging then
		
		local delta = input.Position - dragStart
		
		header.Position = UDim2.new(
			startPos.X.Scale,
			startPos.X.Offset + delta.X,
			startPos.Y.Scale,
			startPos.Y.Offset + delta.Y
		)
	end
end)

header.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.Touch 
	or input.UserInputType == Enum.UserInputType.MouseButton1 then
		
		dragging = false
	end
end)

-- FUNÇÕES
createToggle("Speed Hack", function(v) Speed = v end)
createToggle("Infinite Jump", function(v) Jump = v end)
createToggle("Full Bright", function(v)
    Lighting.Brightness = v and 2 or 1
    Lighting.ClockTime = v and 14 or 12
end)
createToggle("Noclip", function(v) Noclip = v end)
createToggle("Visual ESP", function(v) ESP = v end)

-- PLAYER LOOP
RunService.RenderStepped:Connect(function()
    local char = player.Character
    if not char or not char:FindFirstChild("Humanoid") then return end
    local h = char.Humanoid
    
    if not OriginalSpeed then OriginalSpeed = h.WalkSpeed end
    if not OriginalJump then OriginalJump = h.JumpPower end

    h.WalkSpeed = Speed and 50 or OriginalSpeed

    if Jump then
        h.UseJumpPower = true
        h.JumpPower = 100
    else
        h.JumpPower = OriginalJump
    end

    if Noclip then
        for _, p in pairs(char:GetDescendants()) do
            if p:IsA("BasePart") then p.CanCollide = false end
        end
    end
end)

-- ===== ESP INSANO =====
function NewESP(plr)
    if plr == player then return end

    local box = Drawing.new("Square")
    local name = Drawing.new("Text")
    local dist = Drawing.new("Text")
    local line = Drawing.new("Line")

    box.Filled = false
    box.Thickness = 1

    name.Size = 13
    name.Center = true
    name.Outline = true

    dist.Size = 13
    dist.Center = true
    dist.Outline = true

    line.Thickness = 1

    ESPCache[plr] = {box, name, dist, line}
end

local function removeESP(plr)
    if ESPCache[plr] then
        for _, v in pairs(ESPCache[plr]) do
            v:Remove()
        end
        ESPCache[plr] = nil
    end
end

Players.PlayerAdded:Connect(NewESP)
Players.PlayerRemoving:Connect(removeESP)

for _, p in pairs(Players:GetPlayers()) do
    NewESP(p)
end

RunService.RenderStepped:Connect(function()
    if not ESP then
        for _, esp in pairs(ESPCache) do
            for _, v in pairs(esp) do v.Visible = false end
        end
        return
    end

    local myChar = player.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return end

    local screen = camera.ViewportSize

    for plr, esp in pairs(ESPCache) do
        local char = plr.Character
        local box, name, dist, line = unpack(esp)

        if char and char:FindFirstChild("HumanoidRootPart") and char:FindFirstChild("Head") then
            local root = char.HumanoidRootPart
            local head = char.Head

            local pos, vis = camera:WorldToViewportPoint(root.Position)

            if vis and pos.Z > 0 then
                local headPos = camera:WorldToViewportPoint(head.Position + Vector3.new(0,0.5,0))
                local legPos = camera:WorldToViewportPoint(root.Position - Vector3.new(0,3,0))

                local height = math.abs(headPos.Y - legPos.Y)
                local width = height * 0.6

                box.Size = Vector2.new(width, height)
                box.Position = Vector2.new(pos.X - width/2, pos.Y - height/2)
                box.Color = Colors.Accent
                box.Visible = true

                name.Text = plr.Name
                name.Position = Vector2.new(pos.X, pos.Y - height/2 - 15)
                name.Color = Color3.new(1,1,1)
                name.Visible = true

                local distance = (root.Position - myChar.HumanoidRootPart.Position).Magnitude
                dist.Text = math.floor(distance).."m"
                dist.Position = Vector2.new(pos.X, pos.Y + height/2 + 5)
                dist.Color = Color3.new(1,1,1)
                dist.Visible = true

                line.From = Vector2.new(screen.X/2, screen.Y)
                line.To = Vector2.new(pos.X, pos.Y)
                line.Color = Colors.Accent
                line.Visible = true
            else
                box.Visible = false
                name.Visible = false
                dist.Visible = false
                line.Visible = false
            end
        else
            box.Visible = false
            name.Visible = false
            dist.Visible = false
            line.Visible = false
        end
    end
end)

-- MINIMIZAR
local aberto = true
toggleBtn.MouseButton1Click:Connect(function()
    aberto = not aberto
    container.Visible = aberto
    toggleBtn.Text = aberto and "−" or "+"
end)# Agahrnfndbe
