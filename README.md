local Players = game:GetService("Players")
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

local FRAME_W, FRAME_H = 600, 360

local mainFrame = new("Frame", {
	Parent = screenGui,
	Name = "Main",
	AnchorPoint = Vector2.new(0,0),
	Position = UDim2.fromOffset(0, 0),
	Size = UDim2.new(0, FRAME_W, 0, FRAME_H),
	BackgroundColor3 = Color3.fromRGB(28,28,28),
	BorderSizePixel = 0,
	Visible = false,
})
new("UICorner", {Parent = mainFrame, CornerRadius = UDim.new(0,10)})

local titleBar = new("TextButton", {
	Parent = mainFrame,
	Name = "TitleBar",
	Size = UDim2.new(1,0,0,36),
	Position = UDim2.new(0,0,0,0),
	BackgroundColor3 = Color3.fromRGB(24,24,24),
	Text = "",
	AutoButtonColor = false,
})
new("UICorner", {Parent = titleBar, CornerRadius = UDim.new(0,8)})
new("TextLabel", {Parent = titleBar, Text = "MENU", Size = UDim2.new(1,0,1,0), BackgroundTransparency = 1, TextColor3 = Color3.new(1,1,1), Font = Enum.Font.SourceSansBold, TextSize = 16})

local sidebar = new("Frame", {Parent = mainFrame, Size = UDim2.new(0,140,1,0), Position = UDim2.new(0,0,0,36), BackgroundColor3 = Color3.fromRGB(22,22,22)})
new("UICorner", {Parent = sidebar, CornerRadius = UDim.new(0,8)})
local content = new("Frame", {Parent = mainFrame, Position = UDim2.new(0,150,0,46), Size = UDim2.new(1,-160,1,-56), BackgroundColor3 = Color3.fromRGB(40,40,40)})
new("UICorner", {Parent = content, CornerRadius = UDim.new(0,8)})

local function makeTab(name, icon)
	local btn = new("TextButton", {Parent = sidebar, Text = icon .. "  " .. name, Size = UDim2.new(1,-12,0,40), BackgroundColor3 = Color3.fromRGB(28,28,28), TextColor3 = Color3.fromRGB(230,230,230), Font = Enum.Font.SourceSansBold, TextSize = 16, TextXAlignment = Enum.TextXAlignment.Left})
	new("UICorner", {Parent = btn, CornerRadius = UDim.new(0,6)})
	return btn
end

local tabs = {
	{key="Visual", icon="👁️"},
	{key="Mira", icon="🎯"},
	{key="Player", icon="🧍"},
	{key="World", icon="🌐"},
	{key="Weapon", icon="🔫"},
}
local pages = {}
for _, t in ipairs(tabs) do
	local b = makeTab(t.key, t.icon)
	local p = new("Frame", {Parent = content, Name = t.key.."Page", Size = UDim2.new(1,0,1,0), BackgroundTransparency = 1, Visible = false})
	pages[t.key] = p
	b.MouseButton1Click:Connect(function()
		for k,v in pairs(pages) do v.Visible = (k == t.key) end
	end)
end
pages.Visual.Visible = true

local function makeLabel(parent, text, y)
	local l = new("TextLabel", {Parent = parent, Text = text, Size = UDim2.new(1,0,0,22), BackgroundTransparency = 1, TextColor3 = Color3.fromRGB(220,220,220), Font = Enum.Font.SourceSans, TextSize = 16})
	if y then l.Position = UDim2.new(0,0,0,y) end
	return l
end

makeLabel(pages.Visual, "Visual", 6)
makeLabel(pages.Player, "Player", 6)
makeLabel(pages.World, "World", 6)
makeLabel(pages.Weapon, "Weapon", 6)

local openBtn = new("TextButton", {Parent = screenGui, Text = "Abrir Menu", Size = UDim2.new(0,110,0,28), Position = UDim2.new(0,8,0,8), BackgroundColor3 = Color3.fromRGB(60,60,60), TextColor3 = Color3.new(1,1,1)})
new("UICorner", {Parent = openBtn, CornerRadius = UDim.new(0,6)})

local function centerWindow()
	local vp = camera and camera.ViewportSize or Vector2.new(1280,720)
	local x = math.floor((vp.X - FRAME_W) / 2)
	local y = math.floor((vp.Y - FRAME_H) / 2)
	mainFrame.Position = UDim2.fromOffset(x, y)
end

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

local dragging = false
local dragInput = nil
local dragStart = nil
local startPos = nil

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
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement and dragInput and input == dragInput then
		local delta = input.Position - dragStart
		local newX = startPos.X + delta.X
		local newY = startPos.Y + delta.Y
		local vp = camera and camera.ViewportSize or Vector2.new(1280,720)
		newX = math.clamp(newX, 0, math.max(0, vp.X - mainFrame.AbsoluteSize.X))
		newY = math.clamp(newY, 0, math.max(0, vp.Y - mainFrame.AbsoluteSize.Y))
		mainFrame.Position = UDim2.fromOffset(newX, newY)
	end
end)

centerWindow()
mainFrame.Visible = false
