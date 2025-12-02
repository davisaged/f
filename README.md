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
		if props.Parent then
			pcall(function() obj.Parent = props.Parent end)
		end
	end
	return obj
end

local existing = playerGui:FindFirstChild("SinglePlayerMenu_GUI")
if existing then existing:Destroy() end

local screenGui = new("ScreenGui", {Name = "SinglePlayerMenu_GUI", Parent = playerGui, ResetOnSpawn = false})

-- theme (keep same style as before)
local theme = {
	accent = Color3.fromRGB(0,175,255),
	text = Color3.fromRGB(230,230,230),
	sub = Color3.fromRGB(200,200,200),
	bg = Color3.fromRGB(28,28,28),
}

-- FOV circle (keeps working)
local fovContainer = new("Frame", {Parent = screenGui, AnchorPoint = Vector2.new(0.5,0.5), Position = UDim2.fromScale(0.5,0.5), Size = UDim2.fromOffset(300,300), BackgroundTransparency = 1, Visible = true})
local fovCircle = new("Frame", {Parent = fovContainer, AnchorPoint = Vector2.new(0.5,0.5), Position = UDim2.fromScale(0.5,0.5), Size = UDim2.fromOffset(300,300), BackgroundTransparency = 1})
new("UICorner", {Parent = fovCircle, CornerRadius = UDim.new(1,0)})
local fovStroke = new("UIStroke", {Parent = fovCircle, Color = theme.accent, Thickness = 2})

-- ESP store
local espStore = {}

local function getRoot(model)
	if not model then return nil end
	return model:FindFirstChild("HumanoidRootPart") or model:FindFirstChild("Torso") or model:FindFirstChild("UpperTorso") or model:FindFirstChild("Head")
end

local function getHead(model)
	if not model then return nil end
	return model:FindFirstChild("Head") or model:FindFirstChild("UpperTorso") or model:FindFirstChild("HumanoidRootPart")
end

-- create ESP GUI for a model (player character or tagged model)
local function createESP(model)
	if not model or not model:IsA("Model") then return end
	if model == LocalPlayer.Character then return end
	if espStore[model] then return end

	local root = getRoot(model)
	if not root then return end

	-- BillboardGui sized dynamically
	local bill = new("BillboardGui", {Name = "SPM_ESP", Parent = playerGui, Adornee = root, Size = UDim2.fromOffset(120,160), AlwaysOnTop = true, Active = false})
	bill.StudsOffset = Vector3.new(0, 2, 0)
	bill.ResetOnSpawn = false

	-- container (transparent)
	local container = new("Frame", {Parent = bill, BackgroundTransparency = 1, Size = UDim2.new(1,0,1,0), Position = UDim2.new(0,0)})
	-- name label (above box)
	local nameLabel = new("TextLabel", {Parent = container, Text = model.Name or "?", Size = UDim2.new(1,0,0,18), Position = UDim2.new(0,0,-0.12,0), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.GothamBold, TextSize = 14, TextScaled = false})
	nameLabel.AnchorPoint = Vector2.new(0.5,1)
	nameLabel.Position = UDim2.new(0.5, 0, 0, -6)

	-- box frame (visual bounding box)
	local boxFrame = new("Frame", {Parent = container, BackgroundTransparency = 0.6, BackgroundColor3 = Color3.fromRGB(0,0,0), Size = UDim2.fromOffset(80,140), Position = UDim2.new(0,0,0,0)})
	new("UICorner", {Parent = boxFrame, CornerRadius = UDim.new(0,6)})
	new("UIStroke", {Parent = boxFrame, Color = theme.text, Thickness = 1})

	-- health bar (vertical) to the right of box
	local healthBg = new("Frame", {Parent = container, BackgroundTransparency = 0.6, BackgroundColor3 = Color3.fromRGB(30,30,30), Size = UDim2.fromOffset(10,140), Position = UDim2.new(0, boxFrame.Size.X.Offset + 6, 0, 0)})
	new("UICorner", {Parent = healthBg, CornerRadius = UDim.new(0,3)})
	-- fill (height will be updated)
	local healthFill = new("Frame", {Parent = healthBg, BackgroundColor3 = Color3.fromRGB(0,200,0), Size = UDim2.fromScale(1,1), Position = UDim2.new(0,0,0,0)})
	healthFill.AnchorPoint = Vector2.new(0,0)
	-- percentage label to the right of the bar
	local percLabel = new("TextLabel", {Parent = container, Text = "100%", Size = UDim2.fromOffset(36,18), Position = UDim2.new(0, boxFrame.Size.X.Offset + healthBg.Size.X.Offset + 10, 0, 0), BackgroundTransparency = 1, TextColor3 = theme.text, Font = Enum.Font.Gotham, TextSize = 14})
	percLabel.AnchorPoint = Vector2.new(0,0)

	-- weapon label (below the box)
	local weaponLabel = new("TextLabel", {Parent = container, Text = "", Size = UDim2.new(1,0,0,18), Position = UDim2.new(0,0,1,6), BackgroundTransparency = 1, TextColor3 = theme.sub, Font = Enum.Font.Gotham, TextSize = 14})
	weaponLabel.TextWrapped = true
	weaponLabel.TextXAlignment = Enum.TextXAlignment.Center

	-- distance label (below weapon)
	local distLabel = new("TextLabel", {Parent = container, Text = "", Size = UDim2.new(1,0,0,16), Position = UDim2.new(0,0,1,26), BackgroundTransparency = 1, TextColor3 = theme.sub, Font = Enum.Font.Gotham, TextSize = 12})
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

-- ensure ESP for players and tagged models
local function refreshESP()
	-- players
	for _,pl in ipairs(Players:GetPlayers()) do
		local char = pl.Character
		if char and char.Parent and char ~= LocalPlayer.Character then
			if not espStore[char] then createESP(char) end
		end
	end
	-- tagged models in workspace
	for _,m in ipairs(CollectionService:GetTagged("TestTarget")) do
		if not espStore[m] then createESP(m) end
	end
	-- cleanup removed
	for model,_ in pairs(espStore) do
		local keep = false
		for _,pl in ipairs(Players:GetPlayers()) do
			if pl.Character == model then keep = true; break end
		end
		if not keep and not CollectionService:HasTag(model, "TestTarget") then removeESP(model) end
	end
end

-- hook up player/collection events
Players.PlayerAdded:Connect(function(pl)
	pl.CharacterAdded:Connect(function() task.wait(0.05); refreshESP() end)
end)
Players.PlayerRemoving:Connect(function(pl)
	if pl.Character then removeESP(pl.Character) end
end)
for _,pl in ipairs(Players:GetPlayers()) do
	pl.CharacterAdded:Connect(function() task.wait(0.05); refreshESP() end)
end
CollectionService:GetInstanceAddedSignal("TestTarget"):Connect(function() task.wait(0.05); refreshESP() end)
CollectionService:GetInstanceRemovedSignal("TestTarget"):Connect(refreshESP)
refreshESP()

-- default attributes (colors/toggles)
screenGui:SetAttribute("ESP_BOX", true)
screenGui:SetAttribute("ESP_NAME", true)
screenGui:SetAttribute("ESP_HEALTH", true)
screenGui:SetAttribute("ESP_DISTANCE", true)
screenGui:SetAttribute("ESP_COLOR_R", theme.text.R)
screenGui:SetAttribute("ESP_COLOR_G", theme.text.G)
screenGui:SetAttribute("ESP_COLOR_B", theme.text.B)
screenGui:SetAttribute("FOV_SIZE", 150)
screenGui:SetAttribute("FOV_COLOR_R", theme.accent.R)
screenGui:SetAttribute("FOV_COLOR_G", theme.accent.G)
screenGui:SetAttribute("FOV_COLOR_B", theme.accent.B)
screenGui:SetAttribute("SHOW_FOV", true)

-- update loop: recompute pixel sizes and update GUI elements
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

	-- refresh ESP entries existance
	refreshESP()

	-- update all ESP GUIs
	for model,data in pairs(espStore) do
		if not data.bill or not data.bill.Parent then espStore[model] = nil else
			local showBox = screenGui:GetAttribute("ESP_BOX")
			data.bill.Enabled = showBox

			-- color
			local color = Color3.new(screenGui:GetAttribute("ESP_COLOR_R") or 1, screenGui:GetAttribute("ESP_COLOR_G") or 1, screenGui:GetAttribute("ESP_COLOR_B") or 1)
			local stroke = data.box:FindFirstChildOfClass("UIStroke")
			if stroke then stroke.Color = color end

			-- positions in world: compute top (head) and bottom (root) projection for pixel height
			local head = getHead(model)
			local root = getRoot(model)
			if head and root then
				local topWorld = head.Position + Vector3.new(0, 0.3, 0)
				local bottomWorld = root.Position - Vector3.new(0, 0.8, 0)
				local topScreen, topOn = camera:WorldToViewportPoint(topWorld)
				local botScreen, botOn = camera:WorldToViewportPoint(bottomWorld)
				-- fallback size if not on screen
				local pixelH = 120
				if topOn and botOn then
					pixelH = math.max(24, math.abs(topScreen.Y - botScreen.Y))
				end
				-- width approx as fraction of height
				local pixelW = math.max(18, math.floor(pixelH * 0.45))

				-- health bar width
				local hbW = 10
				-- total billboard size to include bar and some padding and labels under
				local totalW = pixelW + 8 + hbW + 40
				local totalH = pixelH + 40 -- extra for weapon & distance labels

				data.bill.Size = UDim2.fromOffset(totalW, totalH)

				-- position box at left inside billboard
				data.box.Size = UDim2.fromOffset(pixelW, pixelH)
				data.box.Position = UDim2.fromOffset(6, 6)

				-- position health bar to the right of box
				data.healthBg.Size = UDim2.fromOffset(hbW, pixelH)
				data.healthBg.Position = UDim2.fromOffset(6 + pixelW + 6, 6)
				-- health fill: anchored at top, height = percent * pixelH
				local hum = model:FindFirstChildOfClass("Humanoid")
				local healthPct = 0
				if hum and hum.Health and hum.MaxHealth and hum.MaxHealth > 0 then
					healthPct = math.clamp(hum.Health / hum.MaxHealth, 0, 1)
				end
				-- fill height top->bottom (fill height = pct*pixelH). When health decreases, bar shrinks downward.
				data.healthFill.Size = UDim2.fromScale(1, math.max(0.01, healthPct))
				data.healthFill.Position = UDim2.new(0, 0, 0, 0)
				data.healthFill.BackgroundColor3 = Color3.fromHSV(healthPct * 0.33, 1, 0.9) -- green->red

				-- percentage label to the right of health bar
				data.perc.Position = UDim2.fromOffset(6 + pixelW + hbW + 12, 6)
				data.perc.Text = tostring(math.floor(healthPct * 100)) .. "%"

				-- name above centered to the box
				data.name.Text = model.Name or "?"
				data.name.Position = UDim2.fromOffset((totalW/2) - (data.name.Size.X.Offset/2), -18)
				-- weapon and distance below box (centered)
				local weaponText = ""
				local char = model
				if char and char:IsA("Model") then
					local tool = char:FindFirstChildOfClass("Tool")
					if tool then weaponText = tool.Name end
				end
				data.weapon.Text = weaponText
				data.weapon.Position = UDim2.fromOffset(6, 6 + pixelH + 4)
				-- distance
				local distText = ""
				if root and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
					local dt = (root.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
					distText = string.format("%dm", math.floor(dt))
				end
				data.dist.Text = distText
				data.dist.Position = UDim2.fromOffset(6, 6 + pixelH + 22)

				-- visibility of name/health/dist per toggles
				data.name.Visible = screenGui:GetAttribute("ESP_NAME")
				data.healthBg.Visible = screenGui:GetAttribute("ESP_HEALTH")
				data.perc.Visible = screenGui:GetAttribute("ESP_HEALTH")
				data.weapon.Visible = true -- user requested weapon always under box
				data.dist.Visible = screenGui:GetAttribute("ESP_DISTANCE")
			end
		end
	end
end)
