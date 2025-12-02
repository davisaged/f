local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local CollectionService = game:GetService("CollectionService")
local UserInputService = game:GetService("UserInputService")

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
		if props and props.Parent then
			pcall(function() obj.Parent = props.Parent end)
		end
	end
	return obj
end

-- cleanup previous
local existing = playerGui:FindFirstChild("SinglePlayerMenu_GUI")
if existing then existing:Destroy() end

local screenGui = new("ScreenGui", {Name = "SinglePlayerMenu_GUI", Parent = playerGui, ResetOnSpawn = false})
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local FRAME_W, FRAME_H = 720, 420
local theme = {
	bg = Color3.fromRGB(28,28,28),
	side = Color3.fromRGB(22,22,22),
	panel = Color3.fromRGB(40,40,40),
	accent = Color3.fromRGB(0,175,255),
	text = Color3.fromRGB(230,230,230),
	subtext = Color3.fromRGB(200,200,200),
}

-- build main menu (keeps your previous design)
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
mainFrame.ZIndex = 50
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
titleBar.ZIndex = 51
new("UICorner", {Parent = titleBar, CornerRadius = UDim.new(0,10)})
new("TextLabel", {Parent = titleBar, Text = "MENU", Size = UDim2.new(1,0,1,0), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.GothamBold, TextSize = 18})

local closeBtn = new("TextButton", {Parent = titleBar, Text = "X", Size = UDim2.new(0,36,0,24), Position = UDim2.new(1,-44,0,8), BackgroundColor3 = Color3.fromRGB(60,60,60), TextColor3 = Color3.new(1,1,1)})
new("UICorner", {Parent = closeBtn, CornerRadius = UDim.new(0,6)})
closeBtn.MouseButton1Click:Connect(function() mainFrame.Visible = false end)

local sidebar = new("Frame", {Parent = mainFrame, Size = UDim2.new(0,160,0,FRAME_H-40), Position = UDim2.new(0,0,0,40), BackgroundColor3 = theme.side})
new("UICorner", {Parent = sidebar, CornerRadius = UDim.new(0,8)})
local inner = new("Frame", {Parent = sidebar, Position = UDim2.new(0,12,0,12), Size = UDim2.new(1,-24,1,-24), BackgroundTransparency = 1})
local sidebarLayout = new("UIListLayout", {Parent = inner, Padding = UDim.new(0,8), FillDirection = Enum.FillDirection.Vertical, SortOrder = Enum.SortOrder.LayoutOrder})
sidebarLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
sidebarLayout.VerticalAlignment = Enum.VerticalAlignment.Top

local content = new("Frame", {Parent = mainFrame, Position = UDim2.new(0,180,0,40), Size = UDim2.new(1,-200,0,FRAME_H-50), BackgroundColor3 = theme.panel})
new("UICorner", {Parent = content, CornerRadius = UDim.new(0,8)})
content.ZIndex = 50

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
	local lbl = new("TextLabel", {Parent = frame, Text = text, Size = UDim2.new(0.72,0,1,0), Position = UDim2.new(0,6,0,0), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left})
	local btn = new("TextButton", {Parent = frame, Text = default and "ON" or "OFF", Size = UDim2.new(0.26,-6,0.7,0), Position = UDim2.new(0.74,6,0.15,0), BackgroundColor3 = default and theme.accent or Color3.fromRGB(80,80,80), TextColor3 = Color3.new(1,1,1)})
	new("UICorner", {Parent = btn, CornerRadius = UDim.new(0,6)})
	local state = default
	btn.MouseButton1Click:Connect(function()
		state = not state
		btn.Text = state and "ON" or "OFF"
		btn.BackgroundColor3 = state and theme.accent or Color3.fromRGB(80,80,80)
		if callback then callback(state) end
	end)
	if callback then callback(default) end
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

-- VISUAL page controls
do
	local p = pages.Visual.frame
	makeLabel(p, "Visual settings", 6)
	-- note: toggles below are independent; you can enable name/health/distance without enabling box
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

-- other pages kept as before (Mira/Player/World/Weapon)
do
	local p = pages.Mira.frame
	makeLabel(p, "Mira (visual only)", 6)
	makeSlider(p, "FOV Size (px)", 40, 500, 150, 36, function(v) screenGui:SetAttribute("FOV_SIZE", v) end)
	makeToggle(p, "Show FOV Circle", true, 76, function(v) screenGui:SetAttribute("SHOW_FOV", v) end)
end
do
	local p = pages.Player.frame
	makeLabel(p, "Player", 6)
	makeSlider(p, "WalkSpeed (client)", 8, 100, 16, 36, function(v) screenGui:SetAttribute("WALKSPEED", v) end)
	makeSlider(p, "JumpPower (client)", 20, 120, 50, 96, function(v) screenGui:SetAttribute("JUMPPOWER", v) end)
end
do
	local p = pages.World.frame
	makeLabel(p, "World options", 6)
end
do
	local p = pages.Weapon.frame
	makeLabel(p, "Weapon", 6)
end

-- FOV circle visuals (always visible)
local fovContainer = new("Frame", {Parent = screenGui, AnchorPoint = Vector2.new(0.5,0.5), Position = UDim2.fromScale(0.5,0.5), Size = UDim2.fromOffset(300,300), BackgroundTransparency = 1, Visible = true})
local fovCircle = new("Frame", {Parent = fovContainer, AnchorPoint = Vector2.new(0.5,0.5), Position = UDim2.fromScale(0.5,0.5), Size = UDim2.fromOffset(300,300), BackgroundTransparency = 1})
new("UICorner", {Parent = fovCircle, CornerRadius = UDim.new(1,0)})
local fovStroke = new("UIStroke", {Parent = fovCircle, Color = theme.accent, Thickness = 2})

-- ESP manager (players + tagged models)
local espStore = {}

local function getRoot(model)
	if not model then return nil end
	return model:FindFirstChild("HumanoidRootPart") or model:FindFirstChild("Torso") or model:FindFirstChild("UpperTorso") or model:FindFirstChild("Head")
end
local function getHead(model)
	if not model then return nil end
	return model:FindFirstChild("Head") or model:FindFirstChild("UpperTorso") or model:FindFirstChild("HumanoidRootPart")
end

local function createESP(model)
	if not model or not model:IsA("Model") then return end
	if model == LocalPlayer.Character then return end
	if espStore[model] then return end
	local root = getRoot(model)
	if not root then return end

	-- BillboardGui parented to model so it follows character; Adornee=root
	local bill = new("BillboardGui", {Parent = model, Adornee = root, Size = UDim2.fromOffset(140,180), AlwaysOnTop = true, Name = "SPM_ESP"})
	bill.ResetOnSpawn = false
	bill.StudsOffset = Vector3.new(0, 2, 0)

	local container = new("Frame", {Parent = bill, BackgroundTransparency = 1, Size = UDim2.new(1,0,1,0), Position = UDim2.new(0,0)})
	-- name centered above (use anchorcenter)
	local nameLabel = new("TextLabel", {Parent = container, Text = model.Name or "?", Size = UDim2.new(1,0,0,18), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.GothamBold, TextSize = 14})
	nameLabel.AnchorPoint = Vector2.new(0.5,1)
	nameLabel.Position = UDim2.new(0.5,0,0,-6)

	-- prepare frames (health on left, box in center area)
	local healthBg = new("Frame", {Parent = container, BackgroundTransparency = 0.6, BackgroundColor3 = Color3.fromRGB(30,30,30)})
	new("UICorner", {Parent = healthBg, CornerRadius = UDim.new(0,3)})
	local healthFill = new("Frame", {Parent = healthBg, BackgroundColor3 = Color3.fromRGB(0,200,0)})
	healthFill.AnchorPoint = Vector2.new(0,0)
	local percLabel = new("TextLabel", {Parent = container, Text = "100%", Size = UDim2.fromOffset(36,18), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 14})
	percLabel.AnchorPoint = Vector2.new(1,0) -- we'll position relative in render loop

	local boxFrame = new("Frame", {Parent = container, BackgroundTransparency = 0.6, BackgroundColor3 = Color3.fromRGB(0,0,0)})
	new("UICorner", {Parent = boxFrame, CornerRadius = UDim.new(0,6)})
	new("UIStroke", {Parent = boxFrame, Color = theme.text, Thickness = 1})

	local weaponLabel = new("TextLabel", {Parent = container, Text = "", Size = UDim2.new(1,0,0,18), BackgroundTransparency = 1, TextColor3 = theme.subtext, Font = Enum.Font.Gotham, TextSize = 14})
	weaponLabel.TextXAlignment = Enum.TextXAlignment.Center

	local distLabel = new("TextLabel", {Parent = container, Text = "", Size = UDim2.new(1,0,0,16), BackgroundTransparency = 1, TextColor3 = theme.subtext, Font = Enum.Font.Gotham, TextSize = 12})
	distLabel.TextXAlignment = Enum.TextXAlignment.Center

	espStore[model] = {
		bill = bill,
		container = container,
		name = nameLabel,
		box = boxFrame,
		healthBg = healthBg,
		healthFill = healthFill,
		perc = percLabel,
		weapon = weaponLabel,
		dist = distLabel
	}
end

local function removeESP(model)
	local d = espStore[model]
	if d then
		if d.bill and d.bill.Parent then d.bill:Destroy() end
		espStore[model] = nil
	end
end

local function refreshESP()
	-- players
	for _,pl in ipairs(Players:GetPlayers()) do
		local char = pl.Character
		if char and char.Parent and char ~= LocalPlayer.Character then
			if not espStore[char] then createESP(char) end
		end
	end
	-- tagged workspace models
	for _,m in ipairs(CollectionService:GetTagged("TestTarget")) do
		if not espStore[m] then createESP(m) end
	end
	-- cleanup
	for model,_ in pairs(espStore) do
		local keep = false
		for _,pl in ipairs(Players:GetPlayers()) do
			if pl.Character == model then keep = true; break end
		end
		if not keep and not CollectionService:HasTag(model, "TestTarget") then removeESP(model) end
	end
end

Players.PlayerAdded:Connect(function(pl) pl.CharacterAdded:Connect(function() task.wait(0.05); refreshESP() end) end)
Players.PlayerRemoving:Connect(function(pl) if pl.Character then removeESP(pl.Character) end end)
for _,pl in ipairs(Players:GetPlayers()) do pl.CharacterAdded:Connect(function() task.wait(0.05); refreshESP() end) end
CollectionService:GetInstanceAddedSignal("TestTarget"):Connect(function() task.wait(0.05); refreshESP() end)
CollectionService:GetInstanceRemovedSignal("TestTarget"):Connect(refreshESP)
refreshESP()

-- defaults
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
screenGui:SetAttribute("WALKSPEED", 16)
screenGui:SetAttribute("JUMPPOWER", 50)
screenGui:SetAttribute("SHOW_POS", false)

-- center + drag helpers (keeps previous behavior)
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
openBtn.ZIndex = 60
new("UICorner", {Parent = openBtn, CornerRadius = UDim.new(0,6)})
openBtn.MouseButton1Click:Connect(function()
	if not mainFrame.Visible then centerWindow() end
	mainFrame.Visible = not mainFrame.Visible
end)
UserInputService.InputBegan:Connect(function(input, processed)
	if processed then return end
	if input.UserInputType == Enum.UserInputType.Keyboard then
		if input.KeyCode == Enum.KeyCode.Insert or input.KeyCode == Enum.KeyCode.M then
			if not mainFrame.Visible then centerWindow() end
			mainFrame.Visible = not mainFrame.Visible
		end
	end
end)

-- update loop: update FOV + ESP visuals; important: visibility of NAME/HEALTH/DIST independent of BOX
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
	fovContainer.Visible = showfov

	-- ensure ESP objects
	refreshESP()

	-- update ESP elements
	for model,data in pairs(espStore) do
		if not data.bill or not data.bill.Parent then espStore[model] = nil else
			-- independent toggles:
			local showBox = screenGui:GetAttribute("ESP_BOX")
			local showName = screenGui:GetAttribute("ESP_NAME")
			local showHealth = screenGui:GetAttribute("ESP_HEALTH")
			local showDist = screenGui:GetAttribute("ESP_DISTANCE")

			-- box visibility and other components independent
			data.box.Visible = showBox
			data.name.Visible = showName

			-- color
			local color = Color3.new(screenGui:GetAttribute("ESP_COLOR_R") or 1, screenGui:GetAttribute("ESP_COLOR_G") or 1, screenGui:GetAttribute("ESP_COLOR_B") or 1)
			local stroke = data.box:FindFirstChildOfClass("UIStroke")
			if stroke then stroke.Color = color end

			-- compute screen-projected size
			local head = getHead(model)
			local root = getRoot(model)
			if head and root then
				local topWorld = head.Position + Vector3.new(0, 0.3, 0)
				local bottomWorld = root.Position - Vector3.new(0, 0.8, 0)
				local topScreen, topOn = camera:WorldToViewportPoint(topWorld)
				local botScreen, botOn = camera:WorldToViewportPoint(bottomWorld)

				local pixelH = 120
				if topOn and botOn then
					pixelH = math.max(24, math.abs(topScreen.Y - botScreen.Y))
				end
				-- slightly wider so box better fits: increase ratio
				local pixelW = math.max(24, math.floor(pixelH * 0.50))
				local hbW = 12

				-- measure perc label width (fixed)
				local percW = 36
				local gap = 6

				local totalW = percW + gap + hbW + gap + pixelW + 40
				local totalH = pixelH + 44

				data.bill.Size = UDim2.fromOffset(totalW, totalH)

				-- positions: perc (left-most, anchored right so it sits just before health bar)
				data.perc.Size = UDim2.fromOffset(percW, 18)
				data.perc.AnchorPoint = Vector2.new(0,0)
				data.perc.Position = UDim2.fromOffset(6, 6)
				data.perc.Text = "0%"

				-- health background to the right of perc
				local healthX = 6 + percW + gap
				data.healthBg.Size = UDim2.fromOffset(hbW, pixelH)
				data.healthBg.Position = UDim2.fromOffset(healthX, 6)

				-- box to the right of health
				local boxX = healthX + hbW + gap
				data.box.Size = UDim2.fromOffset(pixelW, pixelH)
				data.box.Position = UDim2.fromOffset(boxX, 6)

				-- compute health %
				local hum = model:FindFirstChildOfClass("Humanoid")
				local healthPct = 0
				if hum and hum.Health and hum.MaxHealth and hum.MaxHealth > 0 then
					healthPct = math.clamp(hum.Health / hum.MaxHealth, 0, 1)
				end
				-- healthFill anchored top; height = pct * pixelH
				data.healthFill.Size = UDim2.new(1, 0, healthPct, 0)
				data.healthFill.Position = UDim2.new(0, 0, 0, 0)
				data.healthFill.BackgroundColor3 = Color3.fromHSV(healthPct * 0.33, 1, 0.9)

				-- perc text to the left of health bar or above depending space
				data.perc.Text = tostring(math.floor(healthPct * 100)) .. "%"

				-- name center above
				data.name.Position = UDim2.new(0.5, 0, 0, -6)
				data.name.AnchorPoint = Vector2.new(0.5, 1)

				-- weapon & dist below box
				local weaponText = ""
				if model and model:IsA("Model") then
					local tool = model:FindFirstChildOfClass("Tool")
					if tool then weaponText = tool.Name end
				end
				data.weapon.Text = weaponText
				data.weapon.Position = UDim2.fromOffset(boxX, 6 + pixelH + 6)
				data.dist.Text = ""
				if root and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
					local d = (root.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
					data.dist.Text = string.format("%dm", math.floor(d))
				end
				data.dist.Position = UDim2.fromOffset(boxX, 6 + pixelH + 24)

				-- toggles independent of box
				data.healthBg.Visible = showHealth
				data.perc.Visible = showHealth
				data.weapon.Visible = true
				data.dist.Visible = showDist
			end
		end
	end

	-- apply client-only walk/jump (unchanged)
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
end)

-- keep centered on resize
camera:GetPropertyChangedSignal("ViewportSize"):Connect(function() if mainFrame.Visible then centerWindow() end end)

-- init
centerWindow()
mainFrame.Visible = false
