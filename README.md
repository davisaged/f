-- SinglePlayerMenu.final.local.lua
-- LocalScript completo: menu com abas (Visual, Mira, Player, World, Weapon),
-- ESP visual para modelos com tag "TestTarget", FOV visual ajustável,
-- atalho Insert ou M para abrir/fechar, e botão de abertura no canto.
-- COLOQUE ESTE ARQUIVO EM StarterPlayerScripts COMO LocalScript e rode Play/Play Solo.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local CollectionService = game:GetService("CollectionService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
if not LocalPlayer then
	repeat wait() LocalPlayer = Players.LocalPlayer until LocalPlayer
end
local playerGui = LocalPlayer:WaitForChild("PlayerGui")

-- util
local function new(className, props)
	local obj = Instance.new(className)
	if props then
		for k,v in pairs(props) do
			if k ~= "Parent" then
				pcall(function() obj[k] = v end)
			end
		end
		if props.Parent then
			pcall(function() obj.Parent = props.Parent end)
		end
	end
	return obj
end

local function getRoot(model)
	if not model then return nil end
	return model:FindFirstChild("HumanoidRootPart") or model:FindFirstChild("Torso") or model:FindFirstChild("UpperTorso")
end

-- BUILD GUI
local screenGui = new("ScreenGui", {Name = "SinglePlayerMenu_GUI", Parent = playerGui, ResetOnSpawn = false})

local mainFrame = new("Frame", {
	Parent = screenGui,
	Name = "Main",
	AnchorPoint = Vector2.new(0.5,0.5),
	Position = UDim2.new(0.5,0.5,0.5,0),
	Size = UDim2.new(0, 680, 0, 380),
	BackgroundColor3 = Color3.fromRGB(28,28,28),
	BorderSizePixel = 0,
	Visible = false,
})
new("UICorner", {Parent = mainFrame, CornerRadius = UDim.new(0,10)})

local sidebar = new("Frame", {Parent = mainFrame, Name = "Sidebar", Size = UDim2.new(0,150,1,0), BackgroundColor3 = Color3.fromRGB(22,22,22)})
new("UICorner", {Parent = sidebar, CornerRadius = UDim.new(0,10)})

local content = new("Frame", {Parent = mainFrame, Name = "Content", Position = UDim2.new(0,160,0,10), Size = UDim2.new(1,-170,1,-20), BackgroundColor3 = Color3.fromRGB(40,40,40)})
new("UICorner", {Parent = content, CornerRadius = UDim.new(0,8)})

local function makeTab(name, icon)
	local btn = new("TextButton", {
		Parent = sidebar,
		Text = icon .. "  " .. name,
		Size = UDim2.new(1,-12,0,44),
		BackgroundColor3 = Color3.fromRGB(28,28,28),
		TextColor3 = Color3.fromRGB(230,230,230),
		Font = Enum.Font.SourceSansBold,
		TextSize = 18,
		TextXAlignment = Enum.TextXAlignment.Left,
	})
	new("UICorner", {Parent = btn, CornerRadius = UDim.new(0,6)})
	return btn
end

local tabsDef = {
	{key="Visual", icon="👁️"},
	{key="Mira", icon="🎯"},
	{key="Player", icon="🧍"},
	{key="World", icon="🌐"},
	{key="Weapon", icon="🔫"},
}

local pages = {}
for _, t in ipairs(tabsDef) do
	local btn = makeTab(t.key, t.icon)
	local page = new("Frame", {Parent = content, Name = t.key .. "Page", Size = UDim2.new(1,0,1,0), BackgroundTransparency = 1, Visible = false})
	pages[t.key] = page
	btn.MouseButton1Click:Connect(function()
		for k,p in pairs(pages) do p.Visible = (k == t.key) end
	end)
end
pages.Visual.Visible = true

-- small ui creators
local function makeLabel(parent, text, y)
	local lbl = new("TextLabel", {Parent = parent, Text = text, Size = UDim2.new(1,0,0,22), BackgroundTransparency = 1, TextColor3 = Color3.fromRGB(220,220,220), Font = Enum.Font.SourceSans, TextSize = 16, TextXAlignment = Enum.TextXAlignment.Left})
	if y then lbl.Position = UDim2.new(0,0,0,y) end
	return lbl
end

local function makeToggle(parent, text, default, y)
	local frame = new("Frame", {Parent = parent, Size = UDim2.new(1,0,0,30), BackgroundTransparency = 1})
	if y then frame.Position = UDim2.new(0,6,0,y) end
	local label = new("TextLabel", {Parent = frame, Text = text, Size = UDim2.new(0.7,0,1,0), BackgroundTransparency = 1, TextColor3 = Color3.fromRGB(220,220,220), Font = Enum.Font.SourceSans, TextSize = 16, TextXAlignment = Enum.TextXAlignment.Left})
	local btn = new("TextButton", {Parent = frame, Text = default and "ON" or "OFF", Size = UDim2.new(0.28,-6,0.7,0), Position = UDim2.new(0.72,6,0.15,0), BackgroundColor3 = default and Color3.fromRGB(70,160,70) or Color3.fromRGB(160,70,70), TextColor3 = Color3.new(1,1,1), Font = Enum.Font.SourceSansBold})
	new("UICorner", {Parent = btn, CornerRadius = UDim.new(0,6)})
	local state = default
	btn.MouseButton1Click:Connect(function()
		state = not state
		btn.Text = state and "ON" or "OFF"
		btn.BackgroundColor3 = state and Color3.fromRGB(70,160,70) or Color3.fromRGB(160,70,70)
		if frame.OnChange then frame.OnChange(state) end
	end)
	frame.Set = function(_, v) if state ~= v then btn:MouseButton1Click() end end
	frame.Get = function() return state end
	return frame
end

-- Visual page controls
local visual = pages.Visual
makeLabel(visual, "Visual", 6)
local espBoxToggle = makeToggle(visual, "ESP BOX", true, 34)
local espHealthToggle = makeToggle(visual, "ESP HEALTH", true, 34+36)
local espNameToggle = makeToggle(visual, "ESP NAME", true, 34+36*2)
local espWeaponToggle = makeToggle(visual, "ESP WEAPON", false, 34+36*3)
local espDistanceToggle = makeToggle(visual, "ESP DISTANCIA", true, 34+36*4)
local espHighlightToggle = makeToggle(visual, "ESP HIGHLIGHT", false, 34+36*5)

-- Mira page controls
local mira = pages.Mira
makeLabel(mira, "Mira (VISUAL SOMENTE)", 6)
makeLabel(mira, "FOV Size:", 34)
local fovValue = new("TextLabel", {Parent = mira, Text = "150", Size = UDim2.new(0,40,0,22), Position = UDim2.new(0.78,0,0,34), BackgroundTransparency = 1, TextColor3 = Color3.fromRGB(220,220,220)})
local sliderFrame = new("Frame", {Parent = mira, Size = UDim2.new(0.78,0,0,20), Position = UDim2.new(0,8,0,60), BackgroundColor3 = Color3.fromRGB(60,60,60)})
new("UICorner", {Parent = sliderFrame, CornerRadius = UDim.new(0,6)})
local knob = new("Frame", {Parent = sliderFrame, Size = UDim2.new(0.3,0,1,0), BackgroundColor3 = Color3.fromRGB(200,200,200)})
new("UICorner", {Parent = knob, CornerRadius = UDim.new(0,6)})

local fovSize = 150
local function updateKnob()
	local pct = math.clamp((fovSize - 50) / (400 - 50), 0, 1)
	knob.Size = UDim2.new(pct,0,1,0)
	fovValue.Text = tostring(math.floor(fovSize))
end
updateKnob()

local dragging = false
sliderFrame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true end
end)
sliderFrame.InputEnded:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

-- FOV visual circle
local fovGui = new("Frame", {Parent = screenGui, Name = "FOVCircle", Size = UDim2.new(0, fovSize*2, 0, fovSize*2), Position = UDim2.new(0.5, -fovSize, 0.5, -fovSize), BackgroundTransparency = 0.8, BorderSizePixel = 0})
new("UICorner", {Parent = fovGui, CornerRadius = UDim.new(1,0)})
fovGui.Visible = false
local fovColor = Color3.fromRGB(200,200,50)
fovGui.BackgroundColor3 = fovColor
new("UIStroke", {Parent = fovGui, Color = Color3.fromRGB(255,255,255), Thickness = 1})

-- FOV color options
makeLabel(mira, "FOV Color:", 100)
local colors = {
	{ name="Amarelo", color=Color3.fromRGB(200,200,50) },
	{ name="Vermelho", color=Color3.fromRGB(200,60,60) },
	{ name="Azul", color=Color3.fromRGB(80,140,220) },
}
for i,c in ipairs(colors) do
	local b = new("TextButton", {Parent = mira, Text = c.name, Size = UDim2.new(0,80,0,26), Position = UDim2.new(0, 8 + (i-1)*90, 0, 126), BackgroundColor3 = c.color, TextColor3 = Color3.new(1,1,1)})
	new("UICorner", {Parent = b, CornerRadius = UDim.new(0,6)})
	b.MouseButton1Click:Connect(function()
		fovColor = c.color
		fovGui.BackgroundColor3 = fovColor
	end)
end

-- placeholder pages
makeLabel(pages.Player, "Player options (placeholder)", 6)
makeLabel(pages.World, "World options (placeholder)", 6)
makeLabel(pages.Weapon, "Weapon options (placeholder)", 6)

-- ESP manager (tag "TestTarget")
local espStore = {}
local function createESPForModel(model)
	if not model or not model:IsA("Model") then return end
	if model == LocalPlayer.Character then return end
	local root = getRoot(model)
	if not root then return end
	if espStore[model] then return end

	local bill = new("BillboardGui", {Parent = model, Name = "TestESP", Adornee = root, Size = UDim2.new(0,160,0,70), AlwaysOnTop = true})
	local frame = new("Frame", {Parent = bill, Size = UDim2.new(1,1,1,1), BackgroundTransparency = 0.45, BackgroundColor3 = Color3.fromRGB(0,0,0)})
	new("UICorner", {Parent = frame, CornerRadius = UDim.new(0,6)})
	local nameLabel = new("TextLabel", {Parent = frame, Text = model.Name, Size = UDim2.new(1,0,0,20), BackgroundTransparency = 1, TextColor3 = Color3.new(1,1,1), Font = Enum.Font.SourceSansBold, TextSize = 14})
	local healthLabel = new("TextLabel", {Parent = frame, Text = "HP: ?", Position = UDim2.new(0,0,0,20), Size = UDim2.new(1,0,0,18), BackgroundTransparency = 1, TextColor3 = Color3.new(1,1,1), Font = Enum.Font.SourceSans, TextSize = 13})
	local distLabel = new("TextLabel", {Parent = frame, Text = "Dist: ?", Position = UDim2.new(0,0,0,38), Size = UDim2.new(1,0,0,18), BackgroundTransparency = 1, TextColor3 = Color3.new(1,1,1), Font = Enum.Font.SourceSans, TextSize = 13})
	espStore[model] = {gui = bill, frame = frame, name = nameLabel, hp = healthLabel, dist = distLabel}
end

local function removeESPForModel(model)
	local d = espStore[model]
	if d then
		if d.gui and d.gui.Parent then d.gui:Destroy() end
		espStore[model] = nil
	end
end

local function refreshTagged()
	for m,_ in pairs(espStore) do
		if not CollectionService:HasTag(m, "TestTarget") then removeESPForModel(m) end
	end
	for _, m in ipairs(CollectionService:GetTagged("TestTarget")) do
		if not espStore[m] then createESPForModel(m) end
	end
end

CollectionService:GetInstanceAddedSignal("TestTarget"):Connect(function() wait(0.05); refreshTagged() end)
CollectionService:GetInstanceRemovedSignal("TestTarget"):Connect(refreshTagged)
refreshTagged()

local mouse = LocalPlayer:GetMouse()

-- Main update
RunService.RenderStepped:Connect(function()
	-- slider dragging
	if dragging then
		local mx = mouse.X - sliderFrame.AbsolutePosition.X
		local pct = math.clamp(mx / math.max(1, sliderFrame.AbsoluteSize.X), 0, 1)
		fovSize = 50 + pct * (400 - 50)
		updateKnob()
	end

	-- update fov circle
	local size = math.floor(fovSize)
	fovGui.Size = UDim2.new(0, size*2, 0, size*2)
	fovGui.Position = UDim2.new(0.5, -size, 0.5, -size)

	-- update ESP info
	for model, data in pairs(espStore) do
		if not data.gui or not data.gui.Parent then
			espStore[model] = nil
		else
			data.gui.Enabled = espBoxToggle:Get()
			data.name.Visible = espNameToggle:Get()
			-- health
			local humanoid = model:FindFirstChildOfClass("Humanoid")
			if humanoid then
				data.hp.Text = "HP: " .. math.floor(humanoid.Health)
				data.hp.Visible = espHealthToggle:Get()
			else
				data.hp.Visible = false
			end
			-- distance
			local root = getRoot(model)
			if root and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
				local dist = (root.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
				data.dist.Text = string.format("Dist: %dm", math.floor(dist))
				data.dist.Visible = espDistanceToggle:Get()
			else
				data.dist.Visible = false
			end
		end
	end

	-- highlight inside FOV visually
	local cam = workspace.CurrentCamera
	for model, data in pairs(espStore) do
		if data and data.gui and data.gui.Adornee and data.frame then
			local adornPos = data.gui.Adornee.Position
			local screenPos, onScreen = cam:WorldToViewportPoint(adornPos)
			if onScreen then
				local cx = cam.ViewportSize.X / 2
				local cy = cam.ViewportSize.Y / 2
				local dx = screenPos.X - cx
				local dy = screenPos.Y - cy
				local screenDist = math.sqrt(dx*dx + dy*dy)
				local stroke = data.frame:FindFirstChild("FOVStroke")
				if screenDist <= fovSize and espHighlightToggle:Get() then
					if not stroke then
						new("UIStroke", {Parent = data.frame, Color = Color3.fromRGB(255,255,255), Thickness = 2, Name = "FOVStroke"})
					end
				else
					if stroke then stroke:Destroy() end
				end
			end
		end
	end
end)

-- toggle menu
local function toggleMenu()
	mainFrame.Visible = not mainFrame.Visible
	fovGui.Visible = mainFrame.Visible
end

-- keybinds: Insert and M
UserInputService.InputBegan:Connect(function(input, processed)
	if processed then return end
	if input.UserInputType == Enum.UserInputType.Keyboard then
		if input.KeyCode == Enum.KeyCode.Insert or input.KeyCode == Enum.KeyCode.M then
			toggleMenu()
		end
	end
end)

-- small open button (top-left) for convenience
local debugButton = new("TextButton", {Parent = screenGui, Text = "Abrir Menu", Size = UDim2.new(0,110,0,28), Position = UDim2.new(0,8,0,8), BackgroundColor3 = Color3.fromRGB(60,60,60), TextColor3 = Color3.new(1,1,1)})
new("UICorner", {Parent = debugButton, CornerRadius = UDim.new(0,6)})
debugButton.MouseButton1Click:Connect(toggleMenu)

-- cleanup holder on character remove
Players.LocalPlayer.CharacterRemoving:Connect(function(char)
	for model,_ in pairs(espStore) do
		if model == char then removeESPForModel(model) end
	end
end)

print("[SinglePlayerMenu] carregado. Use Insert ou M, ou botão no canto para abrir. Tag modelos com CollectionService:AddTag(model,'TestTarget') para ESP.")
