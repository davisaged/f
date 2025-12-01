local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
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

-- Sidebar occupies full height minus titleBar (40)
local sidebar = new("Frame", {Parent = mainFrame, Size = UDim2.new(0,160,0,FRAME_H-40), Position = UDim2.new(0,0,0,40), BackgroundColor3 = theme.side})
new("UICorner", {Parent = sidebar, CornerRadius = UDim.new(0,8)})
local sidebarLayout = new("UIListLayout", {Parent = sidebar, Padding = UDim.new(0,8), FillDirection = Enum.FillDirection.Vertical, SortOrder = Enum.SortOrder.LayoutOrder})
sidebarLayout.HorizontalAlignment = Enum.HorizontalAlignment.Left
sidebarLayout.VerticalAlignment = Enum.VerticalAlignment.Top

local content = new("Frame", {Parent = mainFrame, Position = UDim2.new(0,170,0,40), Size = UDim2.new(1,-180,0,FRAME_H-50), BackgroundColor3 = theme.panel})
new("UICorner", {Parent = content, CornerRadius = UDim.new(0,8)})

local function makeTabBtn(name, icon)
	local btn = new("TextButton", {
		Parent = sidebar,
		Text = icon .. "  " .. name,
		Size = UDim2.new(1,-20,0,44),
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

-- show default tab
pages.Visual.frame.Visible = true
pages.Visual.btn.BackgroundColor3 = theme.accent

local function makeLabel(parent, text, posY)
	local lbl = new("TextLabel", {Parent = parent, Text = text, Size = UDim2.new(1,-12,0,22), Position = UDim2.new(0,6,0,posY), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.GothamSemibold, TextSize = 14, TextXAlignment = Enum.TextXAlignment.Left})
	return lbl
end

-- Fill pages with placeholder controls (you can expand)
makeLabel(pages.Visual.frame, "Visual settings", 6)
makeLabel(pages.Mira.frame, "Mira (visual only)", 6)
makeLabel(pages.Player.frame, "Player options", 6)
makeLabel(pages.World.frame, "World options", 6)
makeLabel(pages.Weapon.frame, "Weapon options", 6)

-- Centering and dragging
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

centerWindow()
mainFrame.Visible = false
