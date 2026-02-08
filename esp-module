-- ESP.luau
local ESP = {}
ESP.__index = ESP

function ESP.new(config)
	local self = setmetatable({}, ESP)
	
	self.Player = config.Player
	self.GuiParent = config.GuiParent
	self.Services = config.Services
	self.Notification = config.Notification
	
	self.Players = self.Services.Players
	self.RunService = self.Services.RunService
	self.TweenService = self.Services.TweenService
	self.Camera = workspace.CurrentCamera
	
	-- ESP Settings
	self.settings = {
		ESP_ENABLED = false,
		SELF_ESP = false,
		HEALTH_ESP = true,
		DISTANCE_ESP = true,
		WEAPON_ESP = true,
		TEAM_CHECK = false,
		TEAM_HIDE = false,
		AVAILABLE_ESP = false,
		CHAMS_ENABLED = false,
		CHAMS_GLOW = false,
		CHAMS_OUTLINE = true,
		CHAMS_SOLID = false,
		TRACERS_ENABLED = false,
		SKELETON_ENABLED = false,
		CAN_HIT = false,
		CORNER_BOX = false,
		ASPECT_RATIO = 1,
		
		-- Colors
		COLOR_FRIENDLY = Color3.fromRGB(120, 195, 255),
		COLOR_ENEMY = Color3.fromRGB(255, 80, 80),
		COLOR_VISIBLE = Color3.fromRGB(120, 255, 120),
		COLOR_NOT_VISIBLE = Color3.fromRGB(255, 50, 50),
		COLOR_TEXT = Color3.fromRGB(255, 255, 255),
		COLOR_HEALTH_GREEN = Color3.fromRGB(128, 255, 128),
		COLOR_HEALTH_YELLOW = Color3.fromRGB(255, 255, 80),
		COLOR_HEALTH_RED = Color3.fromRGB(255, 80, 80),
		COLOR_BOX_OUTLINE = Color3.fromRGB(0, 0, 0),
		COLOR_TRACER = Color3.fromRGB(255, 255, 255),
		COLOR_CHAMS_ENEMY = Color3.fromRGB(255, 80, 80),
		COLOR_CHAMS_FRIENDLY = Color3.fromRGB(120, 195, 255),
		COLOR_CHAMS_CAN_HIT = Color3.fromRGB(120, 255, 120),
		COLOR_CHAMS_CANNOT_HIT = Color3.fromRGB(255, 50, 50),
		COLOR_SKELETON = Color3.fromRGB(255, 255, 255),
		COLOR_HEALTH_BG = Color3.fromRGB(10, 10, 10),
		COLOR_HEALTH_OUTLINE = Color3.fromRGB(0, 0, 0)
	}
	
	self.espObjects = {}
	self.espLoop = nil
	
	self.BASE_WIDTH = 150
	self.BASE_HEIGHT = 190
	self.BASE_DISTANCE = 15
	
	self.SKELETON_CONNECTIONS = {
		{"Head", "UpperTorso"},
		{"UpperTorso", "LowerTorso"},
		{"UpperTorso", "LeftUpperArm"},
		{"LeftUpperArm", "LeftLowerArm"},
		{"LeftLowerArm", "LeftHand"},
		{"UpperTorso", "RightUpperArm"},
		{"RightUpperArm", "RightLowerArm"},
		{"RightLowerArm", "RightHand"},
		{"LowerTorso", "LeftUpperLeg"},
		{"LeftUpperLeg", "LeftLowerLeg"},
		{"LeftLowerLeg", "LeftFoot"},
		{"LowerTorso", "RightUpperLeg"},
		{"RightUpperLeg", "RightLowerLeg"},
		{"RightLowerLeg", "RightFoot"}
	}
	
	self.SKELETON_CONNECTIONS_R6 = {
		{"Head", "Torso"},
		{"Torso", "Left Arm"},
		{"Torso", "Right Arm"},
		{"Torso", "Left Leg"},
		{"Torso", "Right Leg"}
	}
	
	self:SetupPlayerListeners()
	
	return self
end

function ESP:worldToScreen(position)
	local screenPoint, onScreen = self.Camera:WorldToViewportPoint(position)
	return Vector2.new(screenPoint.X, screenPoint.Y), onScreen, screenPoint.Z
end

function ESP:applyAspectRatio(screenY)
	if self.settings.ASPECT_RATIO ~= 1 then
		local viewportSize = self.Camera.ViewportSize
		local centerY = viewportSize.Y / 2
		return centerY + (screenY - centerY) * self.settings.ASPECT_RATIO
	end
	return screenY
end

function ESP:isPlayerVisible(targetPlayer)
	if not targetPlayer.Character then return false end
	local targetRoot = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
	local targetHead = targetPlayer.Character:FindFirstChild("Head")
	if not targetRoot or not targetHead then return false end

	local origin = self.Camera.CFrame.Position
	local checkPoints = {
		targetHead.Position,
		targetRoot.Position,
		targetRoot.Position + Vector3.new(0, -2, 0)
	}

	local raycastParams = RaycastParams.new()
	raycastParams.FilterType = Enum.RaycastFilterType.Exclude
	raycastParams.FilterDescendantsInstances = {self.Player.Character, self.Camera}
	raycastParams.IgnoreWater = true

	for _, checkPoint in ipairs(checkPoints) do
		local direction = (checkPoint - origin)
		local distance = direction.Magnitude
		if distance > 0 then
			direction = direction.Unit
			local raycastResult = workspace:Raycast(origin, direction * distance, raycastParams)
			if not raycastResult then
				return true
			end
			local hitCharacter = raycastResult.Instance:FindFirstAncestorWhichIsA("Model")
			if hitCharacter == targetPlayer.Character then
				return true
			end
		end
	end
	return false
end

function ESP:isSameTeam(targetPlayer)
	if not self.Player.Team or not targetPlayer.Team then return false end
	return self.Player.Team == targetPlayer.Team
end

function ESP:getTeamColor(targetPlayer, isVisible)
	if self.settings.TEAM_CHECK then
		if targetPlayer.Team and targetPlayer.TeamColor then
			return targetPlayer.TeamColor.Color
		end
		return self.settings.COLOR_TEXT
	end
	if self.settings.AVAILABLE_ESP then
		return isVisible and self.settings.COLOR_VISIBLE or self.settings.COLOR_NOT_VISIBLE
	end
	if targetPlayer.Team and self.Player.Team then
		if targetPlayer.Team == self.Player.Team then
			return self.settings.COLOR_FRIENDLY
		else
			return self.settings.COLOR_ENEMY
		end
	end
	return isVisible and self.settings.COLOR_VISIBLE or self.settings.COLOR_NOT_VISIBLE
end

function ESP:getEquippedWeapon(character)
	for _, child in ipairs(character:GetChildren()) do
		if child:IsA("Tool") then
			return child.Name
		end
	end
	return nil
end

function ESP:createCornerBox(parent)
	local boxFrame = Instance.new("Frame")
	boxFrame.Name = "BoxFrame"
	boxFrame.BackgroundTransparency = 1
	boxFrame.Size = UDim2.new(1, 0, 1, 0)
	boxFrame.Position = UDim2.new(0, 0, 0, 0)
	boxFrame.Parent = parent

	local corners = {}
	local cornerTypes = {
		{name = "tlh", isHorizontal = true},
		{name = "tlv", isHorizontal = false},
		{name = "trh", isHorizontal = true},
		{name = "trv", isHorizontal = false},
		{name = "blh", isHorizontal = true},
		{name = "blv", isHorizontal = false},
		{name = "brh", isHorizontal = true},
		{name = "brv", isHorizontal = false}
	}

	for _, corner in ipairs(cornerTypes) do
		local outline = Instance.new("Frame")
		outline.BackgroundColor3 = self.settings.COLOR_BOX_OUTLINE
		outline.BorderSizePixel = 0
		outline.ZIndex = 1
		outline.Parent = boxFrame

		local line = Instance.new("Frame")
		line.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
		line.BorderSizePixel = 0
		line.ZIndex = 2
		line.Parent = boxFrame

		table.insert(corners, {outline = outline, line = line, type = corner.name})
	end

	return boxFrame, corners
end

function ESP:updateCornerBox(corners, width, height)
	local cornerW = math.floor(width * 0.25)
	local cornerH = math.floor(height * 0.25)
	cornerW = math.max(cornerW, 8)
	cornerH = math.max(cornerH, 8)

	for _, corner in ipairs(corners) do
		if corner.type == "tlh" then
			corner.outline.Size = UDim2.new(0, cornerW + 2, 0, 3)
			corner.outline.Position = UDim2.new(0, -1, 0, -1)
			corner.line.Size = UDim2.new(0, cornerW, 0, 1)
			corner.line.Position = UDim2.new(0, 0, 0, 0)
		elseif corner.type == "tlv" then
			corner.outline.Size = UDim2.new(0, 3, 0, cornerH + 2)
			corner.outline.Position = UDim2.new(0, -1, 0, -1)
			corner.line.Size = UDim2.new(0, 1, 0, cornerH)
			corner.line.Position = UDim2.new(0, 0, 0, 0)
		elseif corner.type == "trh" then
			corner.outline.Size = UDim2.new(0, cornerW + 2, 0, 3)
			corner.outline.Position = UDim2.new(1, -cornerW - 1, 0, -1)
			corner.line.Size = UDim2.new(0, cornerW, 0, 1)
			corner.line.Position = UDim2.new(1, -cornerW, 0, 0)
		elseif corner.type == "trv" then
			corner.outline.Size = UDim2.new(0, 3, 0, cornerH + 2)
			corner.outline.Position = UDim2.new(1, -2, 0, -1)
			corner.line.Size = UDim2.new(0, 1, 0, cornerH)
			corner.line.Position = UDim2.new(1, -1, 0, 0)
		elseif corner.type == "blh" then
			corner.outline.Size = UDim2.new(0, cornerW + 2, 0, 3)
			corner.outline.Position = UDim2.new(0, -1, 1, -2)
			corner.line.Size = UDim2.new(0, cornerW, 0, 1)
			corner.line.Position = UDim2.new(0, 0, 1, -1)
		elseif corner.type == "blv" then
			corner.outline.Size = UDim2.new(0, 3, 0, cornerH + 2)
			corner.outline.Position = UDim2.new(0, -1, 1, -cornerH - 1)
			corner.line.Size = UDim2.new(0, 1, 0, cornerH)
			corner.line.Position = UDim2.new(0, 0, 1, -cornerH)
		elseif corner.type == "brh" then
			corner.outline.Size = UDim2.new(0, cornerW + 2, 0, 3)
			corner.outline.Position = UDim2.new(1, -cornerW - 1, 1, -2)
			corner.line.Size = UDim2.new(0, cornerW, 0, 1)
			corner.line.Position = UDim2.new(1, -cornerW, 1, -1)
		elseif corner.type == "brv" then
			corner.outline.Size = UDim2.new(0, 3, 0, cornerH + 2)
			corner.outline.Position = UDim2.new(1, -2, 1, -cornerH - 1)
			corner.line.Size = UDim2.new(0, 1, 0, cornerH)
			corner.line.Position = UDim2.new(1, -1, 1, -cornerH)
		end
	end
end

function ESP:create2DBox(parent)
	local boxFrame = Instance.new("Frame")
	boxFrame.Name = "BoxFrame"
	boxFrame.BackgroundTransparency = 1
	boxFrame.Size = UDim2.new(1, 0, 1, 0)
	boxFrame.Position = UDim2.new(0, 0, 0, 0)
	boxFrame.Parent = parent

	local lines = {}
	for i = 1, 4 do
		local outline = Instance.new("Frame")
		outline.Name = "Outline" .. i
		outline.BackgroundColor3 = self.settings.COLOR_BOX_OUTLINE
		outline.BorderSizePixel = 0
		outline.ZIndex = 1
		outline.Parent = boxFrame

		local line = Instance.new("Frame")
		line.Name = "Line" .. i
		line.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
		line.BorderSizePixel = 0
		line.ZIndex = 2
		line.Parent = boxFrame

		table.insert(lines, {outline = outline, line = line})
	end

	lines[1].outline.Size = UDim2.new(1, 2, 0, 3)
	lines[1].outline.Position = UDim2.new(0, -1, 0, -1)
	lines[1].line.Size = UDim2.new(1, 0, 0, 1)
	lines[1].line.Position = UDim2.new(0, 0, 0, 0)

	lines[2].outline.Size = UDim2.new(1, 2, 0, 3)
	lines[2].outline.Position = UDim2.new(0, -1, 1, -2)
	lines[2].line.Size = UDim2.new(1, 0, 0, 1)
	lines[2].line.Position = UDim2.new(0, 0, 1, -1)

	lines[3].outline.Size = UDim2.new(0, 3, 1, 2)
	lines[3].outline.Position = UDim2.new(0, -1, 0, -1)
	lines[3].line.Size = UDim2.new(0, 1, 1, 0)
	lines[3].line.Position = UDim2.new(0, 0, 0, 0)

	lines[4].outline.Size = UDim2.new(0, 3, 1, 2)
	lines[4].outline.Position = UDim2.new(1, -2, 0, -1)
	lines[4].line.Size = UDim2.new(0, 1, 1, 0)
	lines[4].line.Position = UDim2.new(1, -1, 0, 0)

	return boxFrame, lines
end

function ESP:shouldShowChams()
	return self.settings.CHAMS_ENABLED and (self.settings.CHAMS_SOLID or self.settings.CHAMS_OUTLINE or self.settings.CHAMS_GLOW)
end

function ESP:updateChamsAppearance(highlight, targetPlayer, isVisible)
	local fillColor = self.settings.COLOR_CHAMS_ENEMY
	local outlineColor = self.settings.COLOR_CHAMS_ENEMY

	if self.settings.CAN_HIT then
		fillColor = isVisible and self.settings.COLOR_CHAMS_CAN_HIT or self.settings.COLOR_CHAMS_CANNOT_HIT
		outlineColor = fillColor
	elseif self.settings.TEAM_CHECK and targetPlayer.Team and targetPlayer.TeamColor then
		fillColor = targetPlayer.TeamColor.Color
		outlineColor = targetPlayer.TeamColor.Color
	elseif self:isSameTeam(targetPlayer) then
		fillColor = self.settings.COLOR_CHAMS_FRIENDLY
		outlineColor = self.settings.COLOR_CHAMS_FRIENDLY
	end

	highlight.FillColor = fillColor
	highlight.OutlineColor = outlineColor

	if self.settings.CHAMS_SOLID then
		highlight.FillTransparency = 0.4
		highlight.OutlineTransparency = 0
	elseif self.settings.CHAMS_OUTLINE then
		highlight.FillTransparency = 1
		highlight.OutlineTransparency = 0
	elseif self.settings.CHAMS_GLOW then
		highlight.FillTransparency = 0.3
		highlight.OutlineTransparency = 1
	end
end

function ESP:createChams(character, targetPlayer)
	local oldHighlight = character:FindFirstChild("ESP_Chams")
	if oldHighlight then
		oldHighlight:Destroy()
	end

	if not self:shouldShowChams() then
		return nil
	end

	local highlight = Instance.new("Highlight")
	highlight.Name = "ESP_Chams"
	highlight.Adornee = character
	highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop

	local isVisible = self:isPlayerVisible(targetPlayer)
	self:updateChamsAppearance(highlight, targetPlayer, isVisible)

	highlight.Parent = character
	return highlight
end

function ESP:createTracerLine(parent)
	local tracerFrame = Instance.new("Frame")
	tracerFrame.Name = "TracerLine"
	tracerFrame.BackgroundColor3 = self.settings.COLOR_TRACER
	tracerFrame.BorderSizePixel = 0
	tracerFrame.AnchorPoint = Vector2.new(0.5, 0)
	tracerFrame.ZIndex = 1
	tracerFrame.Visible = false
	tracerFrame.Parent = parent

	local tracerOutline = Instance.new("Frame")
	tracerOutline.Name = "TracerOutline"
	tracerOutline.BackgroundColor3 = self.settings.COLOR_BOX_OUTLINE
	tracerOutline.BorderSizePixel = 0
	tracerOutline.AnchorPoint = Vector2.new(0.5, 0)
	tracerOutline.ZIndex = 0
	tracerOutline.Visible = false
	tracerOutline.Parent = parent

	return tracerFrame, tracerOutline
end

function ESP:createSkeletonLines(parent)
	local skeletonLines = {}
	for i = 1, 14 do
		local outline = Instance.new("Frame")
		outline.Name = "SkeletonOutline" .. i
		outline.BackgroundColor3 = self.settings.COLOR_BOX_OUTLINE
		outline.BorderSizePixel = 0
		outline.AnchorPoint = Vector2.new(0.5, 0)
		outline.ZIndex = 1
		outline.Visible = false
		outline.Parent = parent

		local line = Instance.new("Frame")
		line.Name = "SkeletonLine" .. i
		line.BackgroundColor3 = self.settings.COLOR_SKELETON
		line.BorderSizePixel = 0
		line.AnchorPoint = Vector2.new(0.5, 0)
		line.ZIndex = 2
		line.Visible = false
		line.Parent = parent

		table.insert(skeletonLines, {line = line, outline = outline})
	end
	return skeletonLines
end

function ESP:drawLine(line, x1, y1, x2, y2, color, thickness, outline)
	thickness = thickness or 1
	local dx = x2 - x1
	local dy = y2 - y1
	local length = math.sqrt(dx * dx + dy * dy)
	local angle = math.deg(math.atan2(dy, dx))
	local cx = (x1 + x2) / 2
	local cy = (y1 + y2) / 2

	line.Size = UDim2.new(0, length, 0, thickness)
	line.Position = UDim2.new(0, cx, 0, cy)
	line.AnchorPoint = Vector2.new(0.5, 0.5)
	line.Rotation = angle
	line.BackgroundColor3 = color
	line.Visible = true

	if outline then
		outline.Size = UDim2.new(0, length + 2, 0, thickness + 2)
		outline.Position = UDim2.new(0, cx, 0, cy)
		outline.AnchorPoint = Vector2.new(0.5, 0.5)
		outline.Rotation = angle
		outline.Visible = true
	end
end

function ESP:isR15(character)
	return character:FindFirstChild("UpperTorso") ~= nil
end

function ESP:updateSkeleton(espData, character, targetPlayer)
	if not self.settings.SKELETON_ENABLED then
		for _, lineData in ipairs(espData.SkeletonLines) do
			lineData.line.Visible = false
			lineData.outline.Visible = false
		end
		return
	end

	local connections = self:isR15(character) and self.SKELETON_CONNECTIONS or self.SKELETON_CONNECTIONS_R6
	local skeletonColor = self.settings.COLOR_SKELETON

	if self.settings.TEAM_CHECK and targetPlayer.Team and targetPlayer.TeamColor then
		skeletonColor = targetPlayer.TeamColor.Color
	elseif self:isSameTeam(targetPlayer) then
		skeletonColor = self.settings.COLOR_CHAMS_FRIENDLY
	else
		skeletonColor = self.settings.COLOR_CHAMS_ENEMY
	end

	for i, connection in ipairs(connections) do
		local part1 = character:FindFirstChild(connection[1])
		local part2 = character:FindFirstChild(connection[2])

		if part1 and part2 and espData.SkeletonLines[i] then
			local screenPos1, onScreen1 = self:worldToScreen(part1.Position)
			local screenPos2, onScreen2 = self:worldToScreen(part2.Position)

			if onScreen1 and onScreen2 then
				local y1 = self:applyAspectRatio(screenPos1.Y)
				local y2 = self:applyAspectRatio(screenPos2.Y)
				self:drawLine(espData.SkeletonLines[i].line, screenPos1.X, y1, screenPos2.X, y2, skeletonColor, 1, espData.SkeletonLines[i].outline)
			else
				espData.SkeletonLines[i].line.Visible = false
				espData.SkeletonLines[i].outline.Visible = false
			end
		elseif espData.SkeletonLines[i] then
			espData.SkeletonLines[i].line.Visible = false
			espData.SkeletonLines[i].outline.Visible = false
		end
	end

	for i = #connections + 1, #espData.SkeletonLines do
		if espData.SkeletonLines[i] then
			espData.SkeletonLines[i].line.Visible = false
			espData.SkeletonLines[i].outline.Visible = false
		end
	end
end

function ESP:removeESP(targetPlayer)
	if self.espObjects[targetPlayer] then
		if self.espObjects[targetPlayer].ScreenGui and self.espObjects[targetPlayer].ScreenGui.Parent then
			self.espObjects[targetPlayer].ScreenGui:Destroy()
		end
		if targetPlayer.Character then
			local oldHighlight = targetPlayer.Character:FindFirstChild("ESP_Chams")
			if oldHighlight then
				oldHighlight:Destroy()
			end
		end
		self.espObjects[targetPlayer] = nil
	end
end

function ESP:createESP(targetPlayer)
	self:removeESP(targetPlayer)

	local screenGui = Instance.new("ScreenGui")
	screenGui.Name = targetPlayer.Name .. "_ESP"
	screenGui.ResetOnSpawn = false
	screenGui.IgnoreGuiInset = true
	screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

	local container = Instance.new("Frame")
	container.Name = "Container"
	container.BackgroundTransparency = 1
	container.Size = UDim2.new(0, self.BASE_WIDTH, 0, self.BASE_HEIGHT)
	container.Position = UDim2.new(0.5, 0, 0.5, 0)
	container.AnchorPoint = Vector2.new(0.5, 0.5)
	container.Visible = false
	container.Parent = screenGui

	local cornerBoxFrame, cornerLines = self:createCornerBox(container)
	local fullBoxFrame, fullBoxLines = self:create2DBox(container)

	fullBoxFrame.Visible = not self.settings.CORNER_BOX
	cornerBoxFrame.Visible = self.settings.CORNER_BOX

	local nameLabel = Instance.new("TextLabel")
	nameLabel.Name = "NameLabel"
	nameLabel.BackgroundTransparency = 1
	nameLabel.Size = UDim2.new(2, 0, 0, 14)
	nameLabel.Position = UDim2.new(0.5, 0, 0, -16)
	nameLabel.AnchorPoint = Vector2.new(0.5, 1)
	nameLabel.Text = targetPlayer.Name
	nameLabel.TextColor3 = self.settings.COLOR_TEXT
	nameLabel.TextStrokeTransparency = 0
	nameLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
	nameLabel.Font = Enum.Font.SourceSans
	nameLabel.TextSize = 13
	nameLabel.TextXAlignment = Enum.TextXAlignment.Center
	nameLabel.ZIndex = 3
	nameLabel.Parent = container

	local weaponLabel = Instance.new("TextLabel")
	weaponLabel.Name = "WeaponLabel"
	weaponLabel.BackgroundTransparency = 1
	weaponLabel.Size = UDim2.new(2, 0, 0, 12)
	weaponLabel.Position = UDim2.new(0.5, 0, 1, 2)
	weaponLabel.AnchorPoint = Vector2.new(0.5, 0)
	weaponLabel.Text = ""
	weaponLabel.TextColor3 = self.settings.COLOR_TEXT
	weaponLabel.TextStrokeTransparency = 0
	weaponLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
	weaponLabel.Font = Enum.Font.SourceSans
	weaponLabel.TextSize = 11
	weaponLabel.TextXAlignment = Enum.TextXAlignment.Center
	weaponLabel.Visible = false
	weaponLabel.ZIndex = 3
	weaponLabel.Parent = container

	local distanceLabel = Instance.new("TextLabel")
	distanceLabel.Name = "DistanceLabel"
	distanceLabel.BackgroundTransparency = 1
	distanceLabel.Size = UDim2.new(1, 0, 0, 12)
	distanceLabel.Position = UDim2.new(0.5, 0, 1, 14)
	distanceLabel.AnchorPoint = Vector2.new(0.5, 0)
	distanceLabel.Text = "0m"
	distanceLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
	distanceLabel.TextStrokeTransparency = 0
	distanceLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
	distanceLabel.Font = Enum.Font.SourceSans
	distanceLabel.TextSize = 11
	distanceLabel.TextXAlignment = Enum.TextXAlignment.Center
	distanceLabel.Visible = false
	distanceLabel.ZIndex = 3
	distanceLabel.Parent = container

	local healthBarOutline = Instance.new("Frame")
	healthBarOutline.Name = "HealthBarOutline"
	healthBarOutline.BackgroundColor3 = self.settings.COLOR_HEALTH_OUTLINE
	healthBarOutline.BorderSizePixel = 0
	healthBarOutline.Size = UDim2.new(0, 4, 1, 2)
	healthBarOutline.Position = UDim2.new(0, -7, 0, -1)
	healthBarOutline.Visible = false
	healthBarOutline.ZIndex = 1
	healthBarOutline.Parent = container

	local healthBarBg = Instance.new("Frame")
	healthBarBg.Name = "HealthBarBg"
	healthBarBg.BackgroundColor3 = self.settings.COLOR_HEALTH_BG
	healthBarBg.BorderSizePixel = 0
	healthBarBg.Size = UDim2.new(0, 2, 1, 0)
	healthBarBg.Position = UDim2.new(0, -6, 0, 0)
	healthBarBg.Visible = false
	healthBarBg.ZIndex = 2
	healthBarBg.Parent = container

	local healthBarFill = Instance.new("Frame")
	healthBarFill.Name = "HealthBarFill"
	healthBarFill.BackgroundColor3 = self.settings.COLOR_HEALTH_GREEN
	healthBarFill.BorderSizePixel = 0
	healthBarFill.Size = UDim2.new(1, 0, 1, 0)
	healthBarFill.Position = UDim2.new(0, 0, 0, 0)
	healthBarFill.AnchorPoint = Vector2.new(0, 1)
	healthBarFill.ZIndex = 3
	healthBarFill.Parent = healthBarBg

	local healthText = Instance.new("TextLabel")
	healthText.Name = "HealthText"
	healthText.BackgroundTransparency = 1
	healthText.Size = UDim2.new(0, 30, 0, 10)
	healthText.Position = UDim2.new(0, -12, 0.5, 0)
	healthText.AnchorPoint = Vector2.new(1, 0.5)
	healthText.Text = "100"
	healthText.TextColor3 = self.settings.COLOR_TEXT
	healthText.TextStrokeTransparency = 0
	healthText.TextStrokeColor3 = Color3.new(0, 0, 0)
	healthText.Font = Enum.Font.SourceSans
	healthText.TextSize = 10
	healthText.TextXAlignment = Enum.TextXAlignment.Right
	healthText.Visible = false
	healthText.ZIndex = 3
	healthText.Parent = container

	local tracerLine, tracerOutline = self:createTracerLine(screenGui)
	local skeletonLines = self:createSkeletonLines(screenGui)

	local chams = nil
	if self:shouldShowChams() and targetPlayer.Character then
		chams = self:createChams(targetPlayer.Character, targetPlayer)
	end

	self.espObjects[targetPlayer] = {
		ScreenGui = screenGui,
		Container = container,
		CornerBoxFrame = cornerBoxFrame,
		CornerLines = cornerLines,
		FullBoxFrame = fullBoxFrame,
		FullBoxLines = fullBoxLines,
		NameLabel = nameLabel,
		WeaponLabel = weaponLabel,
		DistanceLabel = distanceLabel,
		HealthBarOutline = healthBarOutline,
		HealthBarBg = healthBarBg,
		HealthBarFill = healthBarFill,
		HealthText = healthText,
		TracerLine = tracerLine,
		TracerOutline = tracerOutline,
		SkeletonLines = skeletonLines,
		Chams = chams,
		Player = targetPlayer
	}

	screenGui.Parent = self.Player:WaitForChild("PlayerGui")
end

function ESP:updateTracer(espData, screenPos, onScreen, targetPlayer)
	if not self.settings.TRACERS_ENABLED then
		espData.TracerLine.Visible = false
		if espData.TracerOutline then
			espData.TracerOutline.Visible = false
		end
		return
	end

	if onScreen then
		local screenSize = self.Camera.ViewportSize
		local startX = screenSize.X / 2
		local startY = self:applyAspectRatio(screenSize.Y)
		local endX = screenPos.X
		local endY = screenPos.Y

		local tracerColor = self.settings.COLOR_TRACER
		if self.settings.TEAM_CHECK and targetPlayer.Team and targetPlayer.TeamColor then
			tracerColor = targetPlayer.TeamColor.Color
		elseif self:isSameTeam(targetPlayer) then
			tracerColor = self.settings.COLOR_CHAMS_FRIENDLY
		else
			tracerColor = self.settings.COLOR_CHAMS_ENEMY
		end

		self:drawLine(espData.TracerLine, startX, startY, endX, endY, tracerColor, 1, espData.TracerOutline)
	else
		espData.TracerLine.Visible = false
		if espData.TracerOutline then
			espData.TracerOutline.Visible = false
		end
	end
end

function ESP:updateESP()
	if not self.settings.ESP_ENABLED then return end

	for targetPlayer, espData in pairs(self.espObjects) do
		if not targetPlayer or not targetPlayer.Parent then
			self:removeESP(targetPlayer)
			continue
		end

		local shouldShow = true
		if targetPlayer == self.Player then
			if not self.settings.SELF_ESP then
				shouldShow = false
			end
		end

		if self.settings.TEAM_HIDE and targetPlayer ~= self.Player then
			if self:isSameTeam(targetPlayer) then
				shouldShow = false
				espData.Container.Visible = false
				espData.TracerLine.Visible = false
				if espData.TracerOutline then
					espData.TracerOutline.Visible = false
				end
				for _, lineData in ipairs(espData.SkeletonLines) do
					lineData.line.Visible = false
					lineData.outline.Visible = false
				end
				if targetPlayer.Character then
					local oldHighlight = targetPlayer.Character:FindFirstChild("ESP_Chams")
					if oldHighlight then
						oldHighlight:Destroy()
					end
				end
				continue
			end
		end

		if shouldShow and targetPlayer.Character then
			local character = targetPlayer.Character
			local humanoid = character:FindFirstChildOfClass("Humanoid")
			local rootPart = character:FindFirstChild("HumanoidRootPart")
			local head = character:FindFirstChild("Head")

			if humanoid and humanoid.Health > 0 and rootPart and head then
				local headTop = head.Position + Vector3.new(0, head.Size.Y / 2 + 0.3, 0)
				local feetBottom = rootPart.Position - Vector3.new(0, 2.8, 0)

				local screenTop, topOnScreen = self:worldToScreen(headTop)
				local screenBottom, bottomOnScreen = self:worldToScreen(feetBottom)

				local adjustedTopY = self:applyAspectRatio(screenTop.Y)
				local adjustedBottomY = self:applyAspectRatio(screenBottom.Y)

				local screenCenterX = (screenTop.X + screenBottom.X) / 2
				local screenCenterY = (adjustedTopY + adjustedBottomY) / 2

				local onScreen = topOnScreen or bottomOnScreen

				if onScreen then
					local distance = (rootPart.Position - self.Camera.CFrame.Position).Magnitude

					local height
					if topOnScreen and bottomOnScreen then
						height = math.abs(adjustedBottomY - adjustedTopY)
						height = math.max(height, 20)
					else
						height = math.floor(self.BASE_HEIGHT * (self.BASE_DISTANCE / math.max(distance, 5)))
						if self.settings.ASPECT_RATIO ~= 1 then
							height = height * self.settings.ASPECT_RATIO
						end
					end

					local width = math.floor(height * 0.6)

					espData.Container.Size = UDim2.new(0, width, 0, height)
					espData.Container.Position = UDim2.new(0, math.floor(screenCenterX), 0, math.floor(screenCenterY))
					espData.Container.Visible = true

					local isVisible = self:isPlayerVisible(targetPlayer)
					local boxColor = self:getTeamColor(targetPlayer, isVisible)

					local useCornerBox = self.settings.CORNER_BOX
					espData.CornerBoxFrame.Visible = useCornerBox
					espData.FullBoxFrame.Visible = not useCornerBox

					if useCornerBox then
						self:updateCornerBox(espData.CornerLines, width, height)
						for _, cornerData in ipairs(espData.CornerLines) do
							cornerData.line.BackgroundColor3 = boxColor
						end
					else
						for _, lineData in ipairs(espData.FullBoxLines) do
							lineData.line.BackgroundColor3 = boxColor
						end
					end

					espData.NameLabel.Text = targetPlayer.DisplayName or targetPlayer.Name

					if self.settings.WEAPON_ESP then
						local weaponName = self:getEquippedWeapon(character)
						if weaponName then
							espData.WeaponLabel.Text = weaponName
							espData.WeaponLabel.Visible = true
						else
							espData.WeaponLabel.Visible = false
						end
					else
						espData.WeaponLabel.Visible = false
					end

					if self.settings.DISTANCE_ESP then
						espData.DistanceLabel.Text = string.format("%dm", math.floor(distance))
						if self.settings.WEAPON_ESP and espData.WeaponLabel.Visible then
							espData.DistanceLabel.Position = UDim2.new(0.5, 0, 1, 14)
						else
							espData.DistanceLabel.Position = UDim2.new(0.5, 0, 1, 2)
						end
						espData.DistanceLabel.Visible = true
					else
						espData.DistanceLabel.Visible = false
					end

					if self.settings.HEALTH_ESP then
						local healthPercent = humanoid.Health / humanoid.MaxHealth
						healthPercent = math.clamp(healthPercent, 0, 1)

						espData.HealthBarFill.Size = UDim2.new(1, 0, healthPercent, 0)
						espData.HealthBarFill.Position = UDim2.new(0, 0, 1, 0)

						local r = math.floor(255 * (1 - healthPercent))
						local g = math.floor(255 * healthPercent)
						espData.HealthBarFill.BackgroundColor3 = Color3.fromRGB(r, g, 0)

						espData.HealthText.Text = string.format("%d", math.floor(humanoid.Health))
						espData.HealthBarOutline.Visible = true
						espData.HealthBarBg.Visible = true

						espData.HealthText.Visible = healthPercent < 1
					else
						espData.HealthBarOutline.Visible = false
						espData.HealthBarBg.Visible = false
						espData.HealthText.Visible = false
					end

					if self:shouldShowChams() then
						local existingHighlight = character:FindFirstChild("ESP_Chams")
						if not existingHighlight then
							self:createChams(character, targetPlayer)
						else
							self:updateChamsAppearance(existingHighlight, targetPlayer, isVisible)
						end
					else
						local existingHighlight = character:FindFirstChild("ESP_Chams")
						if existingHighlight then
							existingHighlight:Destroy()
						end
					end

					self:updateTracer(espData, Vector2.new(screenCenterX, screenCenterY), onScreen, targetPlayer)
					self:updateSkeleton(espData, character, targetPlayer)
				else
					espData.Container.Visible = false
					espData.TracerLine.Visible = false
					if espData.TracerOutline then
						espData.TracerOutline.Visible = false
					end
					for _, lineData in ipairs(espData.SkeletonLines) do
						lineData.line.Visible = false
						lineData.outline.Visible = false
					end
				end
			else
				espData.Container.Visible = false
				espData.TracerLine.Visible = false
				if espData.TracerOutline then
					espData.TracerOutline.Visible = false
				end
				for _, lineData in ipairs(espData.SkeletonLines) do
					lineData.line.Visible = false
					lineData.outline.Visible = false
				end
			end
		else
			if espData.Container then
				espData.Container.Visible = false
			end
			if espData.TracerLine then
				espData.TracerLine.Visible = false
			end
			if espData.TracerOutline then
				espData.TracerOutline.Visible = false
			end
			if espData.SkeletonLines then
				for _, lineData in ipairs(espData.SkeletonLines) do
					lineData.line.Visible = false
					lineData.outline.Visible = false
				end
			end
		end
	end
end

function ESP:initializePlayers()
	for _, targetPlayer in ipairs(self.Players:GetPlayers()) do
		if targetPlayer ~= self.Player or self.settings.SELF_ESP then
			self:createESP(targetPlayer)
		end
	end
end

function ESP:clearESP()
	for targetPlayer, _ in pairs(self.espObjects) do
		self:removeESP(targetPlayer)
	end
	self.espObjects = {}

	for _, targetPlayer in ipairs(self.Players:GetPlayers()) do
		if targetPlayer.Character then
			local highlight = targetPlayer.Character:FindFirstChild("ESP_Chams")
			if highlight then
				highlight:Destroy()
			end
		end
	end

	if self.Player.PlayerGui then
		for _, gui in ipairs(self.Player.PlayerGui:GetChildren()) do
			if gui.Name:match("_ESP$") then
				gui:Destroy()
			end
		end
	end
end

function ESP:SetupPlayerListeners()
	self.Players.PlayerAdded:Connect(function(targetPlayer)
		targetPlayer.CharacterAdded:Connect(function(character)
			task.wait(0.5)
			if self.settings.ESP_ENABLED and (targetPlayer ~= self.Player or self.settings.SELF_ESP) then
				self:removeESP(targetPlayer)
				self:createESP(targetPlayer)
			end
		end)

		targetPlayer.CharacterRemoving:Connect(function()
			self:removeESP(targetPlayer)
		end)

		if self.settings.ESP_ENABLED and (targetPlayer ~= self.Player or self.settings.SELF_ESP) then
			task.wait(1)
			self:createESP(targetPlayer)
		end
	end)

	self.Players.PlayerRemoving:Connect(function(targetPlayer)
		self:removeESP(targetPlayer)
		if targetPlayer == self.Player then
			self:clearESP()
			self:StopESPLoop()
		end
	end)

	for _, targetPlayer in ipairs(self.Players:GetPlayers()) do
		if targetPlayer.Character then
			targetPlayer.CharacterAdded:Connect(function(character)
				task.wait(0.5)
				if self.settings.ESP_ENABLED and (targetPlayer ~= self.Player or self.settings.SELF_ESP) then
					self:removeESP(targetPlayer)
					self:createESP(targetPlayer)
				end
			end)

			targetPlayer.CharacterRemoving:Connect(function()
				self:removeESP(targetPlayer)
			end)
		end
	end
end

function ESP:StartESPLoop()
	if self.espLoop then
		self.espLoop:Disconnect()
	end
	self.espLoop = self.RunService.RenderStepped:Connect(function()
		self:updateESP()
	end)
end

function ESP:StopESPLoop()
	if self.espLoop then
		self.espLoop:Disconnect()
		self.espLoop = nil
	end
end

-- Setters
function ESP:SetEnabled(value)
	self.settings.ESP_ENABLED = value
	if value then
		self:initializePlayers()
		self:StartESPLoop()
		task.defer(function() self:updateESP() end)
	else
		self:clearESP()
		self:StopESPLoop()
	end
end

function ESP:SetSelfESP(value)
	self.settings.SELF_ESP = value
	if self.settings.ESP_ENABLED then
		if value then
			self:createESP(self.Player)
		else
			self:removeESP(self.Player)
		end
	end
end

function ESP:SetHealthESP(value) self.settings.HEALTH_ESP = value end
function ESP:SetDistanceESP(value) self.settings.DISTANCE_ESP = value end
function ESP:SetWeaponESP(value) self.settings.WEAPON_ESP = value end
function ESP:SetTeamCheck(value) self.settings.TEAM_CHECK = value end
function ESP:SetTeamHide(value) self.settings.TEAM_HIDE = value end
function ESP:SetAvailableESP(value) self.settings.AVAILABLE_ESP = value end
function ESP:SetChamsEnabled(value) 
	self.settings.CHAMS_ENABLED = value 
	if self.settings.ESP_ENABLED then
		for targetPlayer in pairs(self.espObjects) do
			if targetPlayer.Character then
				if value and self:shouldShowChams() then
					self:createChams(targetPlayer.Character, targetPlayer)
				else
					local h = targetPlayer.Character:FindFirstChild("ESP_Chams")
					if h then h:Destroy() end
				end
			end
		end
	end
end
function ESP:SetChamsGlow(value) 
	self.settings.CHAMS_GLOW = value 
	if self.settings.ESP_ENABLED then
		for targetPlayer in pairs(self.espObjects) do
			if targetPlayer.Character then
				local h = targetPlayer.Character:FindFirstChild("ESP_Chams")
				if h then
					self:updateChamsAppearance(h, targetPlayer, self:isPlayerVisible(targetPlayer))
				end
			end
		end
	end
end
function ESP:SetChamsOutline(value) 
	self.settings.CHAMS_OUTLINE = value 
	if self.settings.ESP_ENABLED then
		for targetPlayer in pairs(self.espObjects) do
			if targetPlayer.Character then
				local h = targetPlayer.Character:FindFirstChild("ESP_Chams")
				if h then
					self:updateChamsAppearance(h, targetPlayer, self:isPlayerVisible(targetPlayer))
				end
			end
		end
	end
end
function ESP:SetChamsSolid(value) 
	self.settings.CHAMS_SOLID = value 
	if self.settings.ESP_ENABLED then
		for targetPlayer in pairs(self.espObjects) do
			if targetPlayer.Character then
				local h = targetPlayer.Character:FindFirstChild("ESP_Chams")
				if h then
					self:updateChamsAppearance(h, targetPlayer, self:isPlayerVisible(targetPlayer))
				end
			end
		end
	end
end
function ESP:SetTracersEnabled(value) self.settings.TRACERS_ENABLED = value end
function ESP:SetSkeletonEnabled(value) self.settings.SKELETON_ENABLED = value end
function ESP:SetCanHit(value) self.settings.CAN_HIT = value end
function ESP:SetCornerBox(value) self.settings.CORNER_BOX = value end
function ESP:SetAspectRatio(value) self.settings.ASPECT_RATIO = value end

-- Color setters
function ESP:SetColorFriendly(color) self.settings.COLOR_FRIENDLY = color end
function ESP:SetColorEnemy(color) self.settings.COLOR_ENEMY = color end
function ESP:SetColorVisible(color) self.settings.COLOR_VISIBLE = color end
function ESP:SetColorNotVisible(color) self.settings.COLOR_NOT_VISIBLE = color end
function ESP:SetColorText(color) self.settings.COLOR_TEXT = color end
function ESP:SetColorHealthGreen(color) self.settings.COLOR_HEALTH_GREEN = color end
function ESP:SetColorHealthYellow(color) self.settings.COLOR_HEALTH_YELLOW = color end
function ESP:SetColorHealthRed(color) self.settings.COLOR_HEALTH_RED = color end
function ESP:SetColorBoxOutline(color) self.settings.COLOR_BOX_OUTLINE = color end
function ESP:SetColorTracer(color) self.settings.COLOR_TRACER = color end
function ESP:SetColorChamsEnemy(color) self.settings.COLOR_CHAMS_ENEMY = color end
function ESP:SetColorChamsFriendly(color) self.settings.COLOR_CHAMS_FRIENDLY = color end
function ESP:SetColorChamsCanHit(color) self.settings.COLOR_CHAMS_CAN_HIT = color end
function ESP:SetColorChamsCannotHit(color) self.settings.COLOR_CHAMS_CANNOT_HIT = color end
function ESP:SetColorSkeleton(color) self.settings.COLOR_SKELETON = color end
function ESP:SetColorHealthBg(color) self.settings.COLOR_HEALTH_BG = color end
function ESP:SetColorHealthOutline(color) self.settings.COLOR_HEALTH_OUTLINE = color end

function ESP:Start()
	-- ESP будет запускаться только при включении через UI
end

function ESP:GetSettings()
	return self.settings
end

function ESP:ApplySettings(settings)
	if not settings then return end
	for k, v in pairs(settings) do
		if self.settings[k] ~= nil then
			self.settings[k] = v
		end
	end
	
	-- Apply ESP enabled state
	if settings.ESP_ENABLED ~= nil then
		self:SetEnabled(settings.ESP_ENABLED)
	end
end

return ESP
