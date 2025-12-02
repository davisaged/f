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

-- VISUAL tab basic controls
do
	local p = pages.Visual.frame
	makeLabel(p, "Visual settings", 6)
end

-- MIRA tab
do
	local p = pages.Mira.frame
	makeLabel(p, "Mira (visual only)", 6)
end

-- PLAYER tab
do
	local p = pages.Player.frame
	makeLabel(p, "Player options", 6)
end

-- WORLD tab
do
	local p = pages.World.frame
	makeLabel(p, "World options", 6)
end

-- WEAPON tab
do
	local p = pages.Weapon.frame
	makeLabel(p, "Weapon options", 6)
end

-- FOV visuals
local fovContainer = new("Frame", {Parent = screenGui, AnchorPoint = Vector2.new(0.5,0.5), Position = UDim2.fromScale(0.5,0.5), Size = UDim2.fromOffset(300,300), BackgroundTransparency = 1, Visible = false})
local fovCircle = new("Frame", {Parent = fovContainer, AnchorPoint = Vector2.new(0.5,0.5), Position = UDim2.fromScale(0.5,0.5), Size = UDim2.fromOffset(300,300), BackgroundTransparency = 1})
local fovStroke = new("UIStroke", {Parent = fovCircle, Color = theme.accent, Thickness = 2})

-- ESP manager: now handles Players' characters + tagged workspace models
local espStore = {}

local function getRoot(model)
	if not model then return nil end
	return model:FindFirstChild("HumanoidRootPart") or model:FindFirstChild("Torso") or model:FindFirstChild("UpperTorso") or model:FindFirstChild("Head")
end

local function createESP(model)
	if not model or not model:IsA("Model") then return end
	if model == LocalPlayer.Character then return end
	if espStore[model] then return end
	local root = getRoot(model)
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
	-- remove those no longer valid or untagged if not players
	for m,_ in pairs(espStore) do
		-- keep if it's a player character still present OR if it is tagged
		local isPlayerChar = false
		for _,pl in ipairs(Players:GetPlayers()) do
			if pl.Character == m then isPlayerChar = true; break end
		end
		if not isPlayerChar and not CollectionService:HasTag(m, "TestTarget") then
			removeESP(m)
		end
	end
	-- ensure player characters have ESP
	for _,pl in ipairs(Players:GetPlayers()) do
		local char = pl.Character
		if char and char.Parent then
			if char ~= LocalPlayer.Character then
				if not espStore[char] then createESP(char) end
			end
		end
	end
	-- ensure tagged models have ESP
	for _,m in ipairs(CollectionService:GetTagged("TestTarget")) do
		if not espStore[m] then createESP(m) end
	end
end

-- Player / character events
Players.PlayerAdded:Connect(function(pl)
	pl.CharacterAdded:Connect(function(char) task.wait(0.1); refreshESP() end)
end)
Players.PlayerRemoving:Connect(function(pl)
	if pl.Character then removeESP(pl.Character) end
end)
-- Existing players
for _,pl in ipairs(Players:GetPlayers()) do
	pl.CharacterAdded:Connect(function(char) task.wait(0.1); refreshESP() end)
end

-- Tagged model signals
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

-- update loop
RunService.RenderStepped:Connect(function()
	local fovSize = screenGui:GetAttribute("FOV_SIZE") or 150
	local fr = screenGui:GetAttribute("FOV_COLOR_R") or theme.accent.R
	local fg = screenGui:GetAttribute("FOV_COLOR_G") or theme.accent.G
	local fb = screenGui:GetAttribute("FOV_COLOR_B") or theme.accent.B
	local showfov = screenGui:GetAttribute("SHOW_FOV")
	fovContainer.Size = UDim2.fromOffset(fovSize*2, fovSize*2)
	fovCircle.Size = UDim2.fromOffset(fovSize*2, fovSize*2)
	fovStroke.Color = Color3.new(fr,fg,fb)
	fovContainer.Visible = showfov and mainFrame.Visible

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
			local root = getRoot(model)
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
end)

centerWindow()
mainFrame.Visible = false
