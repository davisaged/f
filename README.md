local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CollectionService = game:GetService("CollectionService")

local LocalPlayer = Players.LocalPlayer
if not LocalPlayer then repeat task.wait() LocalPlayer = Players.LocalPlayer until LocalPlayer end
local playerGui = LocalPlayer:WaitForChild("PlayerGui")
local camera = workspace.CurrentCamera

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

local existing = playerGui:FindFirstChild("SinglePlayerMenu_GUI")
if existing then existing:Destroy() end

local screenGui = new("ScreenGui", {Name = "SinglePlayerMenu_GUI", Parent = playerGui, ResetOnSpawn = false})

local FRAME_W, FRAME_H = 720, 420
local theme = {
	bg = Color3.fromRGB(28,28,28),
	side = Color3.fromRGB(22,22,22),
	panel = Color3.fromRGB(40,40,40),
	accent = Color3.fromRGB(0,175,255),
	text = Color3.fromRGB(230,230,230),
	subtext = Color3.fromRGB(200,200,200),
}

local mainFrame = new("Frame", {
	Parent = screenGui,
	Name = "Main",
	AnchorPoint = Vector2.new(0,0),
	Position = UDim2.fromOffset(0,0),
	Size = UDim2.new(0, FRAME_W, 0, FRAME_H),
	BackgroundColor3 = theme.bg,
	BorderSizePixel = 0,
	Visible = false,
})
new("UICorner", {Parent = mainFrame, CornerRadius = UDim.new(0,10)})
new("UIStroke", {Parent = mainFrame, Color = Color3.fromRGB(0,0,0), Transparency = 0.6, Thickness = 2})

local titleBar = new("TextButton", {
	Parent = mainFrame,
	Name = "TitleBar",
	Size = UDim2.new(1,0,0,40),
	Position = UDim2.new(0,0,0,0),
	BackgroundColor3 = Color3.fromRGB(24,24,24),
	Text = "",
	AutoButtonColor = false,
})
new("UICorner", {Parent = titleBar, CornerRadius = UDim.new(0,10)})
new("TextLabel", {Parent = titleBar, Text = "MENU", Size = UDim2.new(1,0,1,0), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.GothamBold, TextSize = 18})

local closeBtn = new("TextButton", {Parent = titleBar, Text = "X", Size = UDim2.new(0,36,0,24), Position = UDim2.new(1,-44,0,8), BackgroundColor3 = Color3.fromRGB(60,60,60), TextColor3 = Color3.new(1,1,1)})
new("UICorner", {Parent = closeBtn, CornerRadius = UDim.new(0,6)})
closeBtn.MouseButton1Click:Connect(function() mainFrame.Visible = false end)

-- Sidebar: add inner container so tabs don't touch the very left edge
local sidebar = new("Frame", {Parent = mainFrame, Size = UDim2.new(0,160,0,FRAME_H-40), Position = UDim2.new(0,0,0,40), BackgroundColor3 = theme.side})
new("UICorner", {Parent = sidebar, CornerRadius = UDim.new(0,8)})
local inner = new("Frame", {Parent = sidebar, Position = UDim2.new(0,8,0,8), Size = UDim2.new(1,-16,1,-16), BackgroundTransparency = 1})
local sidebarLayout = new("UIListLayout", {Parent = inner, Padding = UDim.new(0,8), FillDirection = Enum.FillDirection.Vertical, SortOrder = Enum.SortOrder.LayoutOrder})
sidebarLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
sidebarLayout.VerticalAlignment = Enum.VerticalAlignment.Top

local content = new("Frame", {Parent = mainFrame, Position = UDim2.new(0,180,0,40), Size = UDim2.new(1,-200,0,FRAME_H-50), BackgroundColor3 = theme.panel})
new("UICorner", {Parent = content, CornerRadius = UDim.new(0,8)})

local function makeTabBtn(name, icon)
	local btn = new("TextButton", {
		Parent = inner,
		Text = "  "..icon.."  "..name,
		Size = UDim2.new(1,0,0,44),
		BackgroundColor3 = Color3.fromRGB(28,28,28),
		TextColor3 = theme.text,
		Font = Enum.Font.Gotham,
		TextSize = 16,
		TextXAlignment = Enum.TextXAlignment.Left,
		BorderSizePixel = 0,
	})
	new("UICorner", {Parent = btn, CornerRadius = UDim.new(0,6)})
	return btn
end

local tabsData = {
	{key="Visual", icon="👁️"},
	{key="Mira", icon="🎯"},
	{key="Player", icon="🧍"},
	{key="World", icon="🌐"},
	{key="Weapon", icon="🔫"},
}

local pages = {}
for i,t in ipairs(tabsData) do
	local btn = makeTabBtn(t.key, t.icon)
	local page = new("Frame", {Parent = content, Name = t.key.."Page", Size = UDim2.new(1,0,1,0), BackgroundTransparency = 1, Visible = false})
	pages[t.key] = {frame = page, btn = btn}
	btn.MouseButton1Click:Connect(function()
		for k,v in pairs(pages) do
			v.frame.Visible = false
			v.btn.BackgroundColor3 = Color3.fromRGB(28,28,28)
		end
		page.Visible = true
		btn.BackgroundColor3 = theme.accent
	end)
end
pages.Visual.frame.Visible = true
pages.Visual.btn.BackgroundColor3 = theme.accent

local function makeLabel(parent, text, posY)
	local lbl = new("TextLabel", {Parent = parent, Text = text, Size = UDim2.new(1,-12,0,22), Position = UDim2.new(0,6,0,posY), BackgroundTransparency = 1, TextColor3 = theme.subtext, Font = Enum.Font.GothamSemibold, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left})
	return lbl
end

local function makeToggle(parent, text, default, posY, callback)
	local frame = new("Frame", {Parent = parent, Size = UDim2.new(1,-12,0,30), Position = UDim2.new(0,6,0,posY), BackgroundTransparency = 1})
	local lbl = new("TextLabel", {Parent = frame, Text = text, Size = UDim2.new(0.72,0,1,0), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left})
	local btn = new("TextButton", {Parent = frame, Text = default and "ON" or "OFF", Size = UDim2.new(0.26,-6,0.7,0), Position = UDim2.new(0.74,6,0.15,0), BackgroundColor3 = default and theme.accent or Color3.fromRGB(80,80,80), TextColor3 = Color3.new(1,1,1)})
	new("UICorner", {Parent = btn, CornerRadius = UDim.new(0,6)})
	local state = default
	btn.MouseButton1Click:Connect(function()
		state = not state
		btn.Text = state and "ON" or "OFF"
		btn.BackgroundColor3 = state and theme.accent or Color3.fromRGB(80,80,80)
		if callback then callback(state) end
	end)
	return frame
end

local function makeSlider(parent, text, min, max, default, posY, callback)
	local frame = new("Frame", {Parent = parent, Size = UDim2.new(1,-12,0,48), Position = UDim2.new(0,6,0,posY), BackgroundTransparency = 1})
	local label = new("TextLabel", {Parent = frame, Text = text.." : "..tostring(default), Size = UDim2.new(1,0,0,20), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left})
	local bar = new("Frame", {Parent = frame, Size = UDim2.new(1, -20, 0, 10), Position = UDim2.new(0,10,1,-20), BackgroundColor3 = Color3.fromRGB(70,70,70)})
	new("UICorner", {Parent = bar, CornerRadius = UDim.new(0,6)})
	local fill = new("Frame", {Parent = bar, Size = UDim2.new((default-min)/(max-min),0,1,0), BackgroundColor3 = theme.accent})
	new("UICorner", {Parent = fill, CornerRadius = UDim.new(0,6)})
	local dragging = false

	bar.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = true end
	end)
	UserInputService.InputEnded:Connect(function(input) if dragging and input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end end)
	UserInputService.InputChanged:Connect(function(input)
		if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
			local mx = input.Position.X
			local rel = math.clamp((mx - bar.AbsolutePosition.X) / math.max(1, bar.AbsoluteSize.X), 0, 1)
			local val = min + (max-min)*rel
			fill.Size = UDim2.new(rel,0,1,0)
			local display = math.floor(val)
			label.Text = text.." : "..tostring(display)
			if callback then callback(display) end
		end
	end)
	if callback then callback(default) end
	return frame
end

-- VISUAL tab
do
	local p = pages.Visual.frame
	makeLabel(p, "Visual settings", 6)
	makeToggle(p, "ESP BOX", true, 36, function(v) screenGui:SetAttribute("ESP_BOX", v) end)
	makeToggle(p, "ESP NAME", true, 76, function(v) screenGui:SetAttribute("ESP_NAME", v) end)
	makeToggle(p, "ESP HEALTH", true, 116, function(v) screenGui:SetAttribute("ESP_HEALTH", v) end)
	makeToggle(p, "ESP DISTANCE", true, 156, function(v) screenGui:SetAttribute("ESP_DISTANCE", v) end)
	makeToggle(p, "ESP HIGHLIGHT", false, 196, function(v) screenGui:SetAttribute("ESP_HIGHLIGHT", v) end)
	makeLabel(p, "Marker color", 236)
	local colors = {Color3.fromRGB(255,255,255), theme.accent, Color3.fromRGB(255,100,100), Color3.fromRGB(100,200,100)}
	for i,c in ipairs(colors) do
		local b = new("TextButton", {Parent = p, Size = UDim2.new(0,32,0,24), Position = UDim2.new(0, 12 + (i-1)*40, 0, 268), BackgroundColor3 = c, Text = ""})
		new("UICorner", {Parent = b, CornerRadius = UDim.new(0,6)})
		b.MouseButton1Click:Connect(function() screenGui:SetAttribute("ESP_COLOR_R", c.R); screenGui:SetAttribute("ESP_COLOR_G", c.G); screenGui:SetAttribute("ESP_COLOR_B", c.B) end)
	end
end

-- MIRA tab
do
	local p = pages.Mira.frame
	makeLabel(p, "Mira (visual only)", 6)
	makeSlider(p, "FOV Size (px)", 40, 500, 150, 36, function(v) screenGui:SetAttribute("FOV_SIZE", v) end)
	makeLabel(p, "FOV Color", 96)
	local fc = {theme.accent, Color3.fromRGB(255,60,60), Color3.fromRGB(80,140,220), Color3.fromRGB(200,200,50)}
	for i,c in ipairs(fc) do
		local b = new("TextButton", {Parent = p, Size = UDim2.new(0,36,0,26), Position = UDim2.new(0, 12 + (i-1)*44, 0, 128), BackgroundColor3 = c, Text = ""})
		new("UICorner", {Parent = b, CornerRadius = UDim.new(0,6)})
		b.MouseButton1Click:Connect(function() screenGui:SetAttribute("FOV_COLOR_R", c.R); screenGui:SetAttribute("FOV_COLOR_G", c.G); screenGui:SetAttribute("FOV_COLOR_B", c.B) end)
	end
	makeToggle(p, "Show FOV Circle", true, 176, function(v) screenGui:SetAttribute("SHOW_FOV", v) end)
end

-- PLAYER tab
do
	local p = pages.Player.frame
	makeLabel(p, "Player", 6)
	local nameLabel = new("TextLabel", {Parent = p, Text = "Player: "..LocalPlayer.Name, Size = UDim2.new(1,-12,0,22), Position = UDim2.new(0,6,0,36), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.GothamBold, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left})
	local healthLabel = new("TextLabel", {Parent = p, Text = "Health: -", Size = UDim2.new(1,-12,0,22), Position = UDim2.new(0,6,0,64), BackgroundTransparency = 1, TextColor3 = theme.subtext, Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left})
	makeSlider(p, "WalkSpeed", 8, 100, 16, 96, function(v)
		screenGui:SetAttribute("WALKSPEED", v)
	end)
	makeSlider(p, "JumpPower", 20, 120, 50, 156, function(v)
		screenGui:SetAttribute("JUMPPOWER", v)
	end)
	makeToggle(p, "Show Position", false, 216, function(v) screenGui:SetAttribute("SHOW_POS", v) end)
	RunService.RenderStepped:Connect(function()
		local char = LocalPlayer.Character
		local hum = char and char:FindFirstChildOfClass("Humanoid")
		if hum then
			healthLabel.Text = "Health: "..math.floor(hum.Health)
		else
			healthLabel.Text = "Health: -"
		end
	end)
end

-- WORLD tab
do
	local p = pages.World.frame
	makeLabel(p, "World", 6)
	makeSlider(p, "Menu Background Transparency", 0, 100, 0, 36, function(v) mainFrame.BackgroundTransparency = v/100 end)
	makeSlider(p, "Accent Thickness (UIStroke)", 1, 6, 2, 96, function(v)
		local s = mainFrame:FindFirstChildOfClass("UIStroke")
		if s then s.Thickness = v end
	end)
	makeToggle(p, "Show Player Position on HUD", false, 156, function(v) screenGui:SetAttribute("SHOW_POS", v) end)
end

-- WEAPON tab
do
	local p = pages.Weapon.frame
	makeLabel(p, "Weapon", 6)
	local equippedLabel = new("TextLabel", {Parent = p, Text = "Equipped: None", Size = UDim2.new(1,-12,0,22), Position = UDim2.new(0,6,0,36), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left})
	local infoLabel = new("TextLabel", {Parent = p, Text = "Weapon info is client-side only.", Size = UDim2.new(1,-12,0,40), Position = UDim2.new(0,6,0,64), BackgroundTransparency = 1, TextColor3 = theme.subtext, Font = Enum.Font.Gotham, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left})
	RunService.RenderStepped:Connect(function()
		local char = LocalPlayer.Character
		local tool = char and char:FindFirstChildOfClass("Tool")
		if tool then
			equippedLabel.Text = "Equipped: "..tool.Name
		else
			local backpack = LocalPlayer:FindFirstChild("Backpack")
			local first = backpack and backpack:GetChildren()[1]
			equippedLabel.Text = "Equipped: "..(first and first.Name or "None")
		end
	end)
end

-- FOV circle visuals
local fovContainer = new("Frame", {Parent = screenGui, AnchorPoint = Vector2.new(0.5,0.5), Position = UDim2.fromScale(0.5,0.5), Size = UDim2.fromOffset(300,300), BackgroundTransparency = 1, Visible = false})
local fovCircle = new("Frame", {Parent = fovContainer, AnchorPoint = Vector2.new(0.5,0.5), Position = UDim2.fromScale(0.5,0.5), Size = UDim2.fromOffset(300,300), BackgroundTransparency = 1})
local fovStroke = new("UIStroke", {Parent = fovCircle, Color = theme.accent, Thickness = 2})

-- ESP manager
local espStore = {}
local function createESP(model)
	if not model or not model:IsA("Model") then return end
	if espStore[model] then return end
	local root = model:FindFirstChild("HumanoidRootPart") or model:FindFirstChild("Torso") or model:FindFirstChild("UpperTorso")
	if not root then return end
	local bill = new("BillboardGui", {Parent = model, Adornee = root, Size = UDim2.new(0,180,0,64), AlwaysOnTop = true})
	local frame = new("Frame", {Parent = bill, Size = UDim2.new(1,0,1,0), BackgroundTransparency = 0.45, BackgroundColor3 = Color3.fromRGB(0,0,0)})
	new("UICorner", {Parent = frame, CornerRadius = UDim.new(0,6)})
	local nameLabel = new("TextLabel", {Parent = frame, Text = model.Name, Size = UDim2.new(1,0,0,20), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.GothamBold, TextSize = 14})
	local hpLabel = new("TextLabel", {Parent = frame, Text = "HP: ?", Position = UDim2.new(0,0,0,20), Size = UDim2.new(1,0,0,18), BackgroundTransparency = 1, TextColor3 = theme.subtext, Font = Enum.Font.Gotham, TextSize = 13})
	local distLabel = new("TextLabel", {Parent = frame, Text = "Dist: ?", Position = UDim2.new(0,0,0,38), Size = UDim2.new(1,0,0,18), BackgroundTransparency = 1, TextColor3 = theme.subtext, Font = Enum.Font.Gotham, TextSize = 13})
	espStore[model] = {gui = bill, frame = frame, name = nameLabel, hp = hpLabel, dist = distLabel}
end

local function removeESP(model)
	local d = espStore[model]
	if d then
		if d.gui and d.gui.Parent then d.gui:Destroy() end
		espStore[model] = nil
	end
end

local function refreshESP()
	for m,_ in pairs(espStore) do if not CollectionService:HasTag(m, "TestTarget") then removeESP(m) end end
	for _,m in ipairs(CollectionService:GetTagged("TestTarget")) do if not espStore[m] then createESP(m) end end
end

CollectionService:GetInstanceAddedSignal("TestTarget"):Connect(function() task.wait(0.05); refreshESP() end)
CollectionService:GetInstanceRemovedSignal("TestTarget"):Connect(refreshESP)
refreshESP()

-- attributes defaults
screenGui:SetAttribute("ESP_BOX", true)
screenGui:SetAttribute("ESP_NAME", true)
screenGui:SetAttribute("ESP_HEALTH", true)
screenGui:SetAttribute("ESP_DISTANCE", true)
screenGui:SetAttribute("ESP_HIGHLIGHT", false)
screenGui:SetAttribute("ESP_COLOR_R", theme.text.R)
screenGui:SetAttribute("ESP_COLOR_G", theme.text.G)
screenGui:SetAttribute("ESP_COLOR_B", theme.text.B)
screenGui:SetAttribute("FOV_SIZE", 150)
screenGui:SetAttribute("FOV_COLOR_R", theme.accent.R)
screenGui:SetAttribute("FOV_COLOR_G", theme.accent.G)
screenGui:SetAttribute("FOV_COLOR_B", theme.accent.B)
screenGui:SetAttribute("SHOW_FOV", true)
screenGui:SetAttribute("SHOW_POS", false)
screenGui:SetAttribute("WALKSPEED", 16)
screenGui:SetAttribute("JUMPPOWER", 50)

-- center/drag logic
local function centerWindow()
	local vp = camera and camera.ViewportSize or Vector2.new(1280,720)
	local x = math.floor((vp.X - FRAME_W) / 2)
	local y = math.floor((vp.Y - FRAME_H) / 2)
	mainFrame.Position = UDim2.fromOffset(x, y)
end

local dragging = false
local dragInput, dragStart, startPos
titleBar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragInput = input
		dragStart = input.Position
		startPos = Vector2.new(mainFrame.AbsolutePosition.X, mainFrame.AbsolutePosition.Y)
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
				dragInput = nil
			end
		end)
	end
end)
UserInputService.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement and input == dragInput then
		local delta = input.Position - dragStart
		local newX = startPos.X + delta.X
		local newY = startPos.Y + delta.Y
		local vp = camera and camera.ViewportSize or Vector2.new(1280,720)
		newX = math.clamp(newX, 0, math.max(0, vp.X - mainFrame.AbsoluteSize.X))
		newY = math.clamp(newY, 0, math.max(0, vp.Y - mainFrame.AbsoluteSize.Y))
		mainFrame.Position = UDim2.fromOffset(newX, newY)
	end
end)

local openBtn = new("TextButton", {Parent = screenGui, Text = "Abrir Menu", Size = UDim2.new(0,120,0,30), Position = UDim2.new(0,12,0,12), BackgroundColor3 = Color3.fromRGB(60,60,60), TextColor3 = theme.text})
new("UICorner", {Parent = openBtn, CornerRadius = UDim.new(0,6)})
openBtn.MouseButton1Click:Connect(function()
	mainFrame.Visible = not mainFrame.Visible
	if mainFrame.Visible then centerWindow() end
end)

UserInputService.InputBegan:Connect(function(input, processed)
	if processed then return end
	if input.UserInputType == Enum.UserInputType.Keyboard then
		if input.KeyCode == Enum.KeyCode.Insert or input.KeyCode == Enum.KeyCode.M then
			mainFrame.Visible = not mainFrame.Visible
			if mainFrame.Visible then centerWindow() end
		end
	end
end)

-- Apply client-side attributes (walkspeed/jumppower/camera fov/apply visual settings)
RunService.RenderStepped:Connect(function()
	-- FOV visuals
	local fovSize = screenGui:GetAttribute("FOV_SIZE") or 150
	local fr = screenGui:GetAttribute("FOV_COLOR_R") or theme.accent.R
	local fg = screenGui:GetAttribute("FOV_COLOR_G") or theme.accent.G
	local fb = screenGui:GetAttribute("FOV_COLOR_B") or theme.accent.B
	local showfov = screenGui:GetAttribute("SHOW_FOV")
	fovContainer.Size = UDim2.fromOffset(fovSize*2, fovSize*2)
	fovCircle.Size = UDim2.fromOffset(fovSize*2, fovSize*2)
	fovStroke.Color = Color3.new(fr,fg,fb)
	fovContainer.Visible = showfov and mainFrame.Visible

	-- ESP
	for model,data in pairs(espStore) do
		if not data.gui or not data.gui.Parent then espStore[model] = nil else
			local showBox = screenGui:GetAttribute("ESP_BOX")
			data.gui.Enabled = showBox
			data.name.Visible = screenGui:GetAttribute("ESP_NAME")
			data.hp.Visible = screenGui:GetAttribute("ESP_HEALTH")
			data.dist.Visible = screenGui:GetAttribute("ESP_DISTANCE")
			local color = Color3.new(screenGui:GetAttribute("ESP_COLOR_R") or 1, screenGui:GetAttribute("ESP_COLOR_G") or 1, screenGui:GetAttribute("ESP_COLOR_B") or 1)
			if screenGui:GetAttribute("ESP_HIGHLIGHT") then
				local adornPos = data.gui.Adornee.Position
				local screenPos, onScreen = camera:WorldToViewportPoint(adornPos)
				if onScreen then
					local cx, cy = camera.ViewportSize.X/2, camera.ViewportSize.Y/2
					local dx = screenPos.X - cx
					local dy = screenPos.Y - cy
					local dist = math.sqrt(dx*dx + dy*dy)
					if dist <= (screenGui:GetAttribute("FOV_SIZE") or 150) then
						if not data.frame:FindFirstChild("HighlightStroke") then
							new("UIStroke", {Parent = data.frame, Name = "HighlightStroke", Color = color, Thickness = 2})
						end
					else
						local s = data.frame:FindFirstChild("HighlightStroke")
						if s then s:Destroy() end
					end
				end
			else
				local s = data.frame:FindFirstChild("HighlightStroke")
				if s then s:Destroy() end
			end
			local hum = model:FindFirstChildOfClass("Humanoid")
			if hum and hum.Health ~= nil then data.hp.Text = "HP: "..math.floor(hum.Health) end
			local root = model:FindFirstChild("HumanoidRootPart") or model:FindFirstChild("Torso") or model:FindFirstChild("UpperTorso")
			if root and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
				local d = (root.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
				data.dist.Text = "Dist: "..math.floor(d).."m"
			end
			local stroke = data.frame:FindFirstChildOfClass("UIStroke")
			if not stroke then
				new("UIStroke", {Parent = data.frame, Color = color, Thickness = 1})
			else
				stroke.Color = color
			end
		end
	end

	-- apply walk/jump locally (client-side; server may override)
	local ws = screenGui:GetAttribute("WALKSPEED") or 16
	local jp = screenGui:GetAttribute("JUMPPOWER") or 50
	local char = LocalPlayer.Character
	if char then
		local hum = char:FindFirstChildOfClass("Humanoid")
		if hum then
			if hum.WalkSpeed ~= ws then pcall(function() hum.WalkSpeed = ws end) end
			if hum.JumpPower ~= jp then pcall(function() hum.JumpPower = jp end) end
		end
	end

	-- show pos HUD (optional)
	if screenGui:GetAttribute("SHOW_POS") and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
		local pos = LocalPlayer.Character.HumanoidRootPart.Position
		-- one simple way to show is to set the title text (non-intrusive)
		-- but avoid creating many UI objects each frame; using attribute + small label could be added
	end
end)

centerWindow()
mainFrame.Visible = false
