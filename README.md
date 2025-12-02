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

-- Simple theme / sizes
local FRAME_W, FRAME_H = 720, 420
local theme = {
	accent = Color3.fromRGB(0,175,255),
	text = Color3.fromRGB(230,230,230),
	sub = Color3.fromRGB(200,200,200),
}

-- Minimal menu (keeps previous structure but not required for ESP/FOV)
local mainFrame = new("Frame", {Parent = screenGui, Name = "Main", Size = UDim2.new(0,FRAME_W,0,FRAME_H), Position = UDim2.fromOffset(0,0), BackgroundColor3 = Color3.fromRGB(28,28,28), Visible = false})
new("UICorner", {Parent = mainFrame, CornerRadius = UDim.new(0,10)})
local titleBar = new("TextButton", {Parent = mainFrame, Size = UDim2.new(1,0,0,40), BackgroundColor3 = Color3.fromRGB(24,24,24), Text = "", AutoButtonColor = false})
new("TextLabel", {Parent = titleBar, Text = "MENU", Size = UDim2.new(1,0,1,0), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.GothamBold, TextSize = 18})

-- FOV circle (screen-centered). Visible independent of menu.
local fovContainer = new("Frame", {Parent = screenGui, Name = "FOVContainer", AnchorPoint = Vector2.new(0.5,0.5), Position = UDim2.fromScale(0.5,0.5), Size = UDim2.fromOffset(300,300), BackgroundTransparency = 1})
local fovCircle = new("Frame", {Parent = fovContainer, AnchorPoint = Vector2.new(0.5,0.5), Position = UDim2.fromScale(0.5,0.5), Size = UDim2.fromOffset(300,300), BackgroundTransparency = 1})
local fovStroke = new("UIStroke", {Parent = fovCircle, Color = theme.accent, Thickness = 2})
local fovCorner = new("UICorner", {Parent = fovCircle, CornerRadius = UDim.new(1,0)})

-- Ensure circle outline is circular even if scaled
fovCircle.BorderSizePixel = 0

-- ESP storage
local espStore = {} -- model -> {gui, frame, name, hp, dist}

local function getRoot(model)
	if not model then return nil end
	return model:FindFirstChild("HumanoidRootPart") or model:FindFirstChild("Torso") or model:FindFirstChild("UpperTorso") or model:FindFirstChild("Head")
end

local function createESPForModel(model)
	if not model or not model:IsA("Model") then return end
	if model == LocalPlayer.Character then return end
	if espStore[model] then return end
	local root = getRoot(model)
	if not root then return end

	local bill = new("BillboardGui", {Parent = model, Adornee = root, Size = UDim2.new(0,180,0,64), AlwaysOnTop = true, Name = "SPM_ESP"})
	local frame = new("Frame", {Parent = bill, Size = UDim2.new(1,0,1,0), BackgroundTransparency = 0.45, BackgroundColor3 = Color3.fromRGB(0,0,0)})
	new("UICorner", {Parent = frame, CornerRadius = UDim.new(0,6)})
	local nameLabel = new("TextLabel", {Parent = frame, Text = model.Name, Size = UDim2.new(1,0,0,20), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.GothamBold, TextSize = 14})
	local hpLabel = new("TextLabel", {Parent = frame, Text = "HP: ?", Position = UDim2.new(0,0,0,20), Size = UDim2.new(1,0,0,18), BackgroundTransparency = 1, TextColor3 = theme.sub, Font = Enum.Font.Gotham, TextSize = 13})
	local distLabel = new("TextLabel", {Parent = frame, Text = "Dist: ?", Position = UDim2.new(0,0,0,38), Size = UDim2.new(1,0,0,18), BackgroundTransparency = 1, TextColor3 = theme.sub, Font = Enum.Font.Gotham, TextSize = 13})
	espStore[model] = {gui = bill, frame = frame, name = nameLabel, hp = hpLabel, dist = distLabel}
end

local function removeESPForModel(model)
	local d = espStore[model]
	if d then
		if d.gui and d.gui.Parent then d.gui:Destroy() end
		espStore[model] = nil
	end
end

local function ensureESPForPlayersAndTagged()
	-- Create ESP for every player character (except local) if present
	for _, pl in ipairs(Players:GetPlayers()) do
		local char = pl.Character
		if char and char.Parent then
			if char ~= LocalPlayer.Character then
				if not espStore[char] then createESPForModel(char) end
			end
		end
	end
	-- Create ESP for tagged models in workspace
	for _, m in ipairs(CollectionService:GetTagged("TestTarget")) do
		if not espStore[m] then createESPForModel(m) end
	end
	-- Clean up entries that are no longer valid
	for model,_ in pairs(espStore) do
		local keep = false
		-- keep if is a player character present
		for _,pl in ipairs(Players:GetPlayers()) do
			if pl.Character == model then keep = true; break end
		end
		-- keep if still tagged
		if not keep and CollectionService:HasTag(model, "TestTarget") then keep = true end
		if not keep then removeESPForModel(model) end
	end
end

-- Listen for players & characters
Players.PlayerAdded:Connect(function(pl)
	pl.CharacterAdded:Connect(function(char) task.wait(0.05); ensureESPForPlayersAndTagged() end)
end)
Players.PlayerRemoving:Connect(function(pl)
	if pl.Character then removeESPForModel(pl.Character) end
end)
for _,pl in ipairs(Players:GetPlayers()) do
	pl.CharacterAdded:Connect(function(char) task.wait(0.05); ensureESPForPlayersAndTagged() end)
end

CollectionService:GetInstanceAddedSignal("TestTarget"):Connect(function() task.wait(0.05); ensureESPForPlayersAndTagged() end)
CollectionService:GetInstanceRemovedSignal("TestTarget"):Connect(ensureESPForPlayersAndTagged)

-- default attributes (you can expose these in the menu)
screenGui:SetAttribute("ESP_BOX", true)
screenGui:SetAttribute("ESP_NAME", true)
screenGui:SetAttribute("ESP_HEALTH", true)
screenGui:SetAttribute("ESP_DISTANCE", true)
screenGui:SetAttribute("ESP_HIGHLIGHT", false)
screenGui:SetAttribute("ESP_COLOR_R", 1)
screenGui:SetAttribute("ESP_COLOR_G", 1)
screenGui:SetAttribute("ESP_COLOR_B", 1)
screenGui:SetAttribute("FOV_SIZE", 150)
screenGui:SetAttribute("FOV_COLOR_R", theme.accent.R)
screenGui:SetAttribute("FOV_COLOR_G", theme.accent.G)
screenGui:SetAttribute("FOV_COLOR_B", theme.accent.B)
screenGui:SetAttribute("SHOW_FOV", true)

-- Ensure ESP creation at start
ensureESPForPlayersAndTagged()

-- Render update
RunService.RenderStepped:Connect(function()
	-- update FOV visuals (always allowed to show regardless of menu open)
	local fovSize = screenGui:GetAttribute("FOV_SIZE") or 150
	local fr = screenGui:GetAttribute("FOV_COLOR_R") or theme.accent.R
	local fg = screenGui:GetAttribute("FOV_COLOR_G") or theme.accent.G
	local fb = screenGui:GetAttribute("FOV_COLOR_B") or theme.accent.B
	local showfov = screenGui:GetAttribute("SHOW_FOV")
	fovContainer.Size = UDim2.fromOffset(fovSize*2, fovSize*2)
	fovCircle.Size = UDim2.fromOffset(fovSize*2, fovSize*2)
	fovStroke.Color = Color3.new(fr,fg,fb)
	fovContainer.Visible = showfov -- visible independent of menu state

	-- ensure we have ESP for players/tagged models every frame (lightweight)
	ensureESPForPlayersAndTagged()

	-- update each ESP entry
	for model,data in pairs(espStore) do
		if not data.gui or not data.gui.Parent then espStore[model] = nil else
			data.gui.Enabled = screenGui:GetAttribute("ESP_BOX")
			data.name.Visible = screenGui:GetAttribute("ESP_NAME")
			data.hp.Visible = screenGui:GetAttribute("ESP_HEALTH")
			data.dist.Visible = screenGui:GetAttribute("ESP_DISTANCE")
			local color = Color3.new(screenGui:GetAttribute("ESP_COLOR_R") or 1, screenGui:GetAttribute("ESP_COLOR_G") or 1, screenGui:GetAttribute("ESP_COLOR_B") or 1)

			-- update labels
			local hum = model:FindFirstChildOfClass("Humanoid")
			if hum and hum.Health ~= nil then
				data.hp.Text = "HP: "..math.floor(hum.Health)
			else
				data.hp.Text = ""
			end

			local root = getRoot(model)
			if root and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
				local d = (root.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
				data.dist.Text = "Dist: "..math.floor(d).."m"
			else
				data.dist.Text = ""
			end

			-- highlight inside FOV (purely visual)
			local highlight = screenGui:GetAttribute("ESP_HIGHLIGHT")
			local hs = data.frame:FindFirstChild("HighlightStroke")
			if highlight and root then
				local sp, onScreen = camera:WorldToViewportPoint(root.Position)
				if onScreen then
					local cx, cy = camera.ViewportSize.X/2, camera.ViewportSize.Y/2
					local dx = sp.X - cx
					local dy = sp.Y - cy
					local dist = math.sqrt(dx*dx + dy*dy)
					if dist <= (screenGui:GetAttribute("FOV_SIZE") or 150) then
						if not hs then new("UIStroke", {Parent = data.frame, Name = "HighlightStroke", Color = color, Thickness = 2}) end
					else
						if hs then hs:Destroy() end
					end
				else
					if hs then hs:Destroy() end
				end
			else
				if hs then hs:Destroy() end
			end

			-- apply accent stroke color
			local stroke = data.frame:FindFirstChildOfClass("UIStroke")
			if not stroke then
				new("UIStroke", {Parent = data.frame, Color = color, Thickness = 1})
			else
				stroke.Color = color
			end
		end
	end
end)

-- Simple open button & keybind for menu (menu independent of FOV/ESP)
local openBtn = new("TextButton", {Parent = screenGui, Text = "Abrir Menu", Size = UDim2.new(0,120,0,30), Position = UDim2.new(0,12,0,12), BackgroundColor3 = Color3.fromRGB(60,60,60), TextColor3 = theme.text})
new("UICorner", {Parent = openBtn, CornerRadius = UDim.new(0,6)})
openBtn.MouseButton1Click:Connect(function() mainFrame.Visible = not mainFrame.Visible end)
UserInputService.InputBegan:Connect(function(input, processed)
	if processed then return end
	if input.UserInputType == Enum.UserInputType.Keyboard then
		if input.KeyCode == Enum.KeyCode.Insert or input.KeyCode == Enum.KeyCode.M then
			mainFrame.Visible = not mainFrame.Visible
		end
	end
end)
