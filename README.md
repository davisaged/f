-- DebugMenu.Draggable.local.lua
-- Versão do menu com suporte a arrastar (drag) pelo mouse.
-- Cole como LocalScript em StarterPlayerScripts e rode Play/Play Solo.
-- Pressione Insert ou M, ou clique no botão "Abrir Menu" para mostrar/ocultar.
-- Esta é apenas UI (sem automação de mira).

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local LocalPlayer = Players.LocalPlayer
if not LocalPlayer then
	repeat task.wait() LocalPlayer = Players.LocalPlayer until LocalPlayer
end
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

-- Remove GUI antiga se existir
local existing = playerGui:FindFirstChild("SinglePlayerMenu_GUI")
if existing then existing:Destroy() end

-- ScreenGui
local screenGui = new("ScreenGui", {Name = "SinglePlayerMenu_GUI", Parent = playerGui, ResetOnSpawn = false})

-- MAIN FRAME: usar Position via pixels (fromOffset) para facilitar arrastar
local frameWidth, frameHeight = 600, 360
local viewport = camera and camera.ViewportSize or Vector2.new(1280,720)
local initialX = math.floor((viewport.X - frameWidth) / 2)
local initialY = math.floor((viewport.Y - frameHeight) / 2)

local mainFrame = new("Frame", {
	Parent = screenGui,
	Name = "Main",
	AnchorPoint = Vector2.new(0,0),
	Position = UDim2.fromOffset(initialX, initialY),
	Size = UDim2.new(0, frameWidth, 0, frameHeight),
	BackgroundColor3 = Color3.fromRGB(28,28,28),
	BorderSizePixel = 0,
	Visible = false,
})
new("UICorner", {Parent = mainFrame, CornerRadius = UDim.new(0,10)})

-- TITLE BAR (área de arrastar)
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
local titleLabel = new("TextLabel", {Parent = titleBar, Text = "MENU DEBUG (arraste aqui)", Size = UDim2.new(1,0,1,0), BackgroundTransparency = 1, TextColor3 = Color3.new(1,1,1), Font = Enum.Font.SourceSansBold, TextSize = 16})
titleLabel.RichText = false

-- Sidebar + content
local sidebar = new("Frame", {Parent = mainFrame, Size = UDim2.new(0,140,1,0), Position = UDim2.new(0,0,0,36), BackgroundColor3 = Color3.fromRGB(22,22,22)})
new("UICorner", {Parent = sidebar, CornerRadius = UDim.new(0,8)})
local content = new("Frame", {Parent = mainFrame, Position = UDim2.new(0,150,0,46), Size = UDim2.new(1,-160,1,-56), BackgroundColor3 = Color3.fromRGB(40,40,40)})
new("UICorner", {Parent = content, CornerRadius = UDim.new(0,8)})

-- Tabs
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

-- Simple controls to show functionality
local function makeLabel(parent, text, y)
	local l = new("TextLabel", {Parent = parent, Text = text, Size = UDim2.new(1,0,0,22), BackgroundTransparency = 1, TextColor3 = Color3.fromRGB(220,220,220), Font = Enum.Font.SourceSans, TextSize = 16})
	if y then l.Position = UDim2.new(0,0,0,y) end
	return l
end
local function makeToggle(parent, text, default, y)
	local f = new("Frame", {Parent = parent, Size = UDim2.new(1,0,0,30)})
	if y then f.Position = UDim2.new(0,6,0,y) end
	new("TextLabel", {Parent = f, Text = text, Size = UDim2.new(0.7,0,1,0), BackgroundTransparency = 1, TextColor3 = Color3.fromRGB(220,220,220), Font = Enum.Font.SourceSans})
	local btn = new("TextButton", {Parent = f, Size = UDim2.new(0.28,-6,0.7,0), Position = UDim2.new(0.72,6,0.15,0), Text = default and "ON" or "OFF", BackgroundColor3 = default and Color3.fromRGB(70,160,70) or Color3.fromRGB(160,70,70), TextColor3 = Color3.new(1,1,1)})
	new("UICorner", {Parent = btn, CornerRadius = UDim.new(0,6)})
	local state = default
	btn.MouseButton1Click:Connect(function()
		state = not state
		btn.Text = state and "ON" or "OFF"
		btn.BackgroundColor3 = state and Color3.fromRGB(70,160,70) or Color3.fromRGB(160,70,70)
		print("[DebugMenu] Toggle", text, "->", state)
	end)
	return f
end

local visualPage = pages.Visual
makeLabel(visualPage, "Visual (arrastável)", 6)
makeToggle(visualPage, "ESP BOX", true, 34)
makeToggle(visualPage, "ESP NAME", true, 34+36)

-- Button to open/close menu (top-left)
local openBtn = new("TextButton", {Parent = screenGui, Text = "Abrir Menu", Size = UDim2.new(0,110,0,28), Position = UDim2.new(0,8,0,8), BackgroundColor3 = Color3.fromRGB(60,60,60), TextColor3 = Color3.new(1,1,1)})
new("UICorner", {Parent = openBtn, CornerRadius = UDim.new(0,6)})
openBtn.MouseButton1Click:Connect(function()
	mainFrame.Visible = not mainFrame.Visible
	print("[DebugMenu] botão AbrirMenu clicado; mainFrame.Visible =", mainFrame.Visible)
end)

-- Keybinds (Insert and M)
UserInputService.InputBegan:Connect(function(input, processed)
	if processed then return end
	if input.UserInputType == Enum.UserInputType.Keyboard then
		if input.KeyCode == Enum.KeyCode.Insert or input.KeyCode == Enum.KeyCode.M then
			mainFrame.Visible = not mainFrame.Visible
			print("[DebugMenu] tecla detectada:", input.KeyCode.Name, "mainFrame.Visible =", mainFrame.Visible)
		end
	end
end)

-- DRAGGING LOGIC (arrastar pela titleBar)
local dragging = false
local dragStart = nil
local frameStart = nil

local function clampPosition(x, y)
	local vp = camera and camera.ViewportSize or Vector2.new(1280,720)
	local fw, fh = mainFrame.AbsoluteSize.X, mainFrame.AbsoluteSize.Y
	local nx = math.clamp(x, 0, math.max(0, vp.X - fw))
	local ny = math.clamp(y, 0, math.max(0, vp.Y - fh))
	return nx, ny
end

titleBar.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 then
		dragging = true
		dragStart = input.Position
		frameStart = Vector2.new(mainFrame.AbsolutePosition.X, mainFrame.AbsolutePosition.Y)
		-- capture release on this input
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
		local delta = input.Position - dragStart
		local newX = frameStart.X + delta.X
		local newY = frameStart.Y + delta.Y
		newX, newY = clampPosition(newX, newY)
		mainFrame.Position = UDim2.fromOffset(newX, newY)
	end
end)

-- Auto-open on spawn for convenience (remove if not needed)
mainFrame.Visible = true
print("[DebugMenu] carregado com suporte a arrastar. Arraste o título para mover a janela.")
