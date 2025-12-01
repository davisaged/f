-- DebugMenu.local.lua
-- Versão de depuração do menu: abre automaticamente ao iniciar (debugAutoOpen = true),
-- imprime logs no Output e tem botão para abrir/fechar. Cole ESTE arquivo como LocalScript
-- em StarterPlayerScripts e rode Play/Play Solo. Se não aparecer, cole as mensagens do Output.
-- Este script é apenas UI (sem aimbot). Ajuste debugAutoOpen = false depois.

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local CollectionService = game:GetService("CollectionService")

local LocalPlayer = Players.LocalPlayer
if not LocalPlayer then
	repeat task.wait() LocalPlayer = Players.LocalPlayer until LocalPlayer
end

local playerGui = LocalPlayer:WaitForChild("PlayerGui")
local debugAutoOpen = true -- abre automaticamente para teste; depois coloque false

local function new(className, props)
	local ok, obj = pcall(function() return Instance.new(className) end)
	if not ok then return nil end
	if props and obj then
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

-- start logging
print("[DebugMenu] iniciando...")

-- Remove qualquer GUI antiga com mesmo nome (evitar duplicação)
local existing = playerGui:FindFirstChild("SinglePlayerMenu_GUI")
if existing then existing:Destroy(); print("[DebugMenu] GUI antiga removida") end

-- Build GUI
local screenGui = new("ScreenGui", {Name = "SinglePlayerMenu_GUI", Parent = playerGui, ResetOnSpawn = false})
if not screenGui then
	warn("[DebugMenu] não foi possível criar ScreenGui")
	return
end

local mainFrame = new("Frame", {
	Parent = screenGui,
	Name = "Main",
	AnchorPoint = Vector2.new(0.5,0.5),
	Position = UDim2.new(0.5,0.5),
	Size = UDim2.new(0,600,0,360),
	BackgroundColor3 = Color3.fromRGB(28,28,28),
	BorderSizePixel = 0,
	Visible = false,
})
new("UICorner", {Parent = mainFrame, CornerRadius = UDim.new(0,10)})

-- Header
local title = new("TextLabel", {Parent = mainFrame, Size = UDim2.new(1,0,0,36), BackgroundTransparency = 1, Text = "MENU DEBUG", Font = Enum.Font.SourceSansBold, TextSize = 20, TextColor3 = Color3.new(1,1,1)})
-- Sidebar + content
local sidebar = new("Frame", {Parent = mainFrame, Size = UDim2.new(0,140,1,0), BackgroundColor3 = Color3.fromRGB(22,22,22)})
new("UICorner", {Parent = sidebar, CornerRadius = UDim.new(0,8)})
local content = new("Frame", {Parent = mainFrame, Position = UDim2.new(0,150,0,10), Size = UDim2.new(1,-160,1,-20), BackgroundColor3 = Color3.fromRGB(40,40,40)})
new("UICorner", {Parent = content, CornerRadius = UDim.new(0,8)})

-- Create tabs quickly
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
for i,t in ipairs(tabs) do
	local b = makeTab(t.key, t.icon)
	local p = new("Frame", {Parent = content, Name = t.key.."Page", Size = UDim2.new(1,0,1,0), BackgroundTransparency = 1, Visible = false})
	pages[t.key] = p
	b.MouseButton1Click:Connect(function()
		for k,v in pairs(pages) do v.Visible = (k == t.key) end
	end)
end
pages.Visual.Visible = true

-- Simple controls for Visual page to confirm interactivity
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
makeLabel(visualPage, "Visual (debug)", 6)
makeToggle(visualPage, "ESP BOX", true, 34)
makeToggle(visualPage, "ESP NAME", true, 34+36)

-- Add open button (top-left)
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

-- Auto-open for debug
if debugAutoOpen then
	mainFrame.Visible = true
	print("[DebugMenu] aberto automaticamente (debugAutoOpen = true)")
else
	print("[DebugMenu] carregado. Use Insert/M ou botão Abrir Menu.")
end

-- Quick verification: print presence of GUI in PlayerGui
print("[DebugMenu] ScreenGui parent:", screenGui.Parent and screenGui.Parent.Name or "NIL")
print("[DebugMenu] mainFrame visible:", mainFrame.Visible)

-- Small on-screen hint that auto-removes
local hint = new("Hint", {Parent = playerGui, Text = "DebugMenu carregado. Abra com Insert/M ou botão."})
task.delay(3, function() pcall(function() hint:Destroy() end) end)

-- If ainda não aparecer, cole aqui a saída do Output (tudo que começa com [DebugMenu]) para eu revisar.
