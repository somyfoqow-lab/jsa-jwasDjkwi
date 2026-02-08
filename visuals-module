local VisualsModule = {}
VisualsModule.__index = VisualsModule

function VisualsModule.new(config)
	local self = setmetatable({}, VisualsModule)

	self.player = config.Player
	self.guiParent = config.GuiParent
	self.Services = config.Services or {}

	self.Players = self.Services.Players
	self.TweenService = self.Services.TweenService
	self.RunService = self.Services.RunService
	self.UserInputService = self.Services.UserInputService
	self.ReplicatedStorage = self.Services.ReplicatedStorage
	self.Workspace = self.Services.Workspace
	self.Lighting = self.Services.Lighting
	self.Debris = self.Services.Debris
	self.SoundService = self.Services.SoundService
	self.HttpService = self.Services.HttpService

	self.Notification = config.Notification
	self.IsFakeLagEnabled = config.IsFakeLagEnabled

	self.HITLOG_ENABLED = false
	self.HITSOUND_ENABLED = true
	self.HITSOUND_VOLUME = 1
	self.HITSOUND_PRESET = "bell"

self.GHOST_BASE_TRANSPARENCY = 0.7
self.GHOST_FADE_RADIUS = 1.6
self.GHOST_FADE_POWER = 1.2

	self.KEYBIND_LIST_VISIBLE = true

	self.TRACER_ENABLED = false
	self.TRACER_COLOR = Color3.fromRGB(255, 100, 220)
	self.TRACER_LIFETIME = 0.45
	self.TRACER_WIDTH = 0.12

	self.TRACER_BRIGHTNESS = 20
	self.TRACER_LIGHTEMISSION = 1
	self.TRACER_LIGHTINFLUENCE = 0.95
	self.TRACER_TAIL = 2
	self.TRACER_TRANSPARENCY = NumberSequence.new({
		NumberSequenceKeypoint.new(0.00, 1.00),
		NumberSequenceKeypoint.new(0.125, 0.05),
		NumberSequenceKeypoint.new(0.89, 0.0562),
		NumberSequenceKeypoint.new(1.00, 1.00),
	})

	self.WALLBANG_ENABLED = false
	self.WALLBANG_ENTRY_COLOR = Color3.fromRGB(255, 50, 50)
	self.WALLBANG_EXIT_COLOR = Color3.fromRGB(50, 255, 50)
	self.WALLBANG_MARKER_SIZE = Vector3.new(0.5, 0.5, 0.5)
	self.WALLBANG_LIFETIME = 3

	self.KILLFX_ENABLED = false
	self.KILLFX_TYPE = "ImpactFX"
	self.KILLFX_COLOR = Color3.fromRGB(80, 160, 255)
	self.KILLFX_LIGHT_COLOR = Color3.fromRGB(120, 180, 255)
	self.KILLFX_CUSTOM_DURATION = 0.25
	self.KILLFX_LIGHT_RANGE = 15
	self.KILLFX_LIGHT_BRIGHTNESS = 1

	self.OFFSCREEN_ENABLED = false
	self.OFFSCREEN_COLOR = Color3.fromRGB(255, 80, 80)

	self.BACKTRACK_GHOST_ENABLED = false
	self.BACKTRACK_GHOST_COLOR = Color3.fromRGB(80, 160, 255)
	self.BACKTRACK_GHOST_DURATION = 0.3

	self.KILLIMAGE_ENABLED = false
	self.KILLIMAGE_COLOR = Color3.fromRGB(80, 160, 255)
	self.KILLIMAGE_FADE_IN = 0.08
	self.KILLIMAGE_HOLD = 0.1
	self.KILLIMAGE_FADE_OUT = 0.6
	self.KILLIMAGE_MAX_ALPHA = 0.45

	self.ATMOSPHERE_ENABLED = false
	self.ATMOSPHERE_DENSITY = 0.3
	self.ATMOSPHERE_OFFSET = 0.25
	self.ATMOSPHERE_COLOR = Color3.fromRGB(200, 200, 200)
	self.ATMOSPHERE_DECAY = Color3.fromRGB(128, 128, 128)
	self.ATMOSPHERE_GLARE = 0.5
	self.ATMOSPHERE_HAZE = 0.5

	self.BLOOM_ENABLED = false
	self.BLOOM_INTENSITY = 2
	self.BLOOM_SIZE = 56
	self.BLOOM_THRESHOLD = 0.8

	self.COLOR_CORRECTION_ENABLED = false
	self.COLOR_CORRECTION_BRIGHTNESS = 0
	self.COLOR_CORRECTION_CONTRAST = 0
	self.COLOR_CORRECTION_SATURATION = 0
	self.COLOR_CORRECTION_TINT = Color3.fromRGB(255, 255, 255)

	self.tracerPool = {}
	self.MAX_TRACER_POOL = 12

	self.arrows = {}
	self.hitlogContainer = nil
	self.hitlogOrder = 0

	self.KeybindUI = { Container = nil, ContentContainer = nil, ActiveBinds = {}, Visible = true }
	self._bindInputBegan = nil
	self._bindInputEnded = nil

	self.ghostModel = nil
	self.ghostExpireAt = 0
	self.sourceCharacter = nil
	self.lastFakeLagState = nil
	self.ghostParts = {}
	self.ghostLastCFrames = {}
	self.GHOST_SMOOTH_SPEED = 45
	self._hbLast = tick()
	self._ghostDt = 1 / 60

	self.killImage = nil

	self:Initialize()

	return self
end

function VisualsModule:Initialize()
	self.Camera = self.Workspace.CurrentCamera
	self.FX_ROOT = self.ReplicatedStorage:WaitForChild("FXTemplates")
	self.HITSOUND_STORAGE = self.guiParent:WaitForChild("ScreenGui"):WaitForChild("storage")

	self.ORIGINAL_EFFECTS = {
		Atmosphere = self:SnapshotFirstEffect("Atmosphere"),
		BloomEffect = self:SnapshotFirstEffect("BloomEffect"),
		ColorCorrectionEffect = self:SnapshotFirstEffect("ColorCorrectionEffect"),
	}

	self:CreateHitlogUI()
	self:CreateKeybindUI()
	self:CreateOffscreenArrows()
	self:CreateKillImageGUI()
	self:SetupBacktrackGhost()
	self:SetupKillFXListener()
end

function VisualsModule:CreateHitlogUI()
	local screenGui = Instance.new("ScreenGui")
	screenGui.Name = "NemesisHitlogs"
	screenGui.ResetOnSpawn = false
	screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
	screenGui.Parent = self.guiParent

	local container = Instance.new("Frame")
	container.Name = "HitlogList"
	container.Active = false
	container.Selectable = false
	container.Size = UDim2.new(0, 300, 0, 200)
	container.Position = UDim2.new(0.5, -150, 0.55, 20)
	container.AnchorPoint = Vector2.new(0, 0)
	container.BackgroundTransparency = 1
	container.BorderSizePixel = 0
	container.Parent = screenGui

	local listLayout = Instance.new("UIListLayout")
	listLayout.SortOrder = Enum.SortOrder.LayoutOrder
	listLayout.Padding = UDim.new(0, 2)
	listLayout.VerticalAlignment = Enum.VerticalAlignment.Top
	listLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
	listLayout.Parent = container

	self.hitlogContainer = container
	self.hitlogOrder = 0
end

function VisualsModule:ShowHitlog(hitType, info)
	if not self.HITLOG_ENABLED then return end
	if not self.hitlogContainer then return end

	self.hitlogOrder += 1
	local textLabel = Instance.new("TextLabel")
	textLabel.Name = "Hitlog_" .. self.hitlogOrder
	textLabel.Active = false
	textLabel.Selectable = false
	textLabel.Size = UDim2.new(1, 0, 0, 12)
	textLabel.BackgroundTransparency = 1
	textLabel.TextTransparency = 1
	textLabel.TextColor3 = (hitType == "Hit") and Color3.fromRGB(150, 255, 150) or Color3.fromRGB(255, 150, 150)
	textLabel.TextSize = 11
	textLabel.Font = Enum.Font.Code
	textLabel.TextStrokeTransparency = 1
	textLabel.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
	textLabel.TextXAlignment = Enum.TextXAlignment.Center
	textLabel.LayoutOrder = self.hitlogOrder
	textLabel.Parent = self.hitlogContainer

	if hitType == "Hit" and info then
		textLabel.Text = string.format("HIT | %s | %d DMG | %d studs",
			tostring(info.bodyPart or "Body"),
			math.floor(tonumber(info.damage) or 0),
			math.floor(tonumber(info.distance) or 0))
	elseif hitType == "Miss" and info then
		local reason = info.reason or "unknown"
		if reason == "hitchance_failed" then
			textLabel.Text = string.format("MISS | %d%% (%d) | %d studs",
				tonumber(info.hitchance) or 0,
				tonumber(info.roll) or 0,
				tonumber(info.distance) or 0)
		elseif reason == "min_damage" then
			textLabel.Text = string.format("MISS | DMG < %d | %d studs",
				tonumber(info.minDamage) or 0,
				tonumber(info.distance) or 0)
		elseif reason == "wall_blocking_target" then
			textLabel.Text = string.format("MISS | WALL | %d studs", tonumber(info.distance) or 0)
		elseif reason == "too_far" then
			textLabel.Text = string.format("MISS | TOO FAR | %d studs", tonumber(info.distance) or 0)
		elseif reason == "bad_angle" then
			textLabel.Text = string.format("MISS | ANGLE | %d°", tonumber(info.angle) or 0)
		elseif reason == "no_target" then
			textLabel.Text = "MISS | NO TARGET"
		elseif reason == "target_dead" then
			textLabel.Text = "MISS | DEAD"
		elseif reason == "friendly_fire" then
			textLabel.Text = "MISS | TEAM"
		else
			textLabel.Text = "MISS"
		end
	elseif hitType == "Hit" then
		textLabel.Text = "HIT"
	else
		textLabel.Text = "MISS"
	end

	local tweenInfoShow = TweenInfo.new(0.25, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
	self.TweenService:Create(textLabel, tweenInfoShow, {TextTransparency = 0, TextStrokeTransparency = 0.4}):Play()

	task.spawn(function()
		task.wait(2.5)
		if not textLabel or not textLabel.Parent then return end
		local tweenInfoHide = TweenInfo.new(0.35, Enum.EasingStyle.Quint, Enum.EasingDirection.In)
		local hideTween = self.TweenService:Create(textLabel, tweenInfoHide, {TextTransparency = 1, TextStrokeTransparency = 1})
		hideTween:Play()
		hideTween.Completed:Connect(function()
			if textLabel and textLabel.Parent then
				textLabel:Destroy()
			end
		end)
	end)
end

function VisualsModule:PlayHitSound()
	if not self.HITSOUND_ENABLED then return end

	local HITSOUND_MAP = {
		correct = "HitSfx13",
		skeet = "HitSfx",
		orchestra = "HitSfx2",
		bow = "HitSfx3",
		uwu = "HitSfx4",
		tf2 = "HitSfx5",
		b8 = "HitSfx6",
		basketball = "HitSfx7",
		idk = "HitSfx8",
		orb = "HitSfx9",
		balltap = "HitSfx10",
		softbell = "HitSfx11",
		softhit = "HitSfx12",
		soft = "HitSfx14",
		bell2 = "HitSfx15",
		tank1 = "HitSfx16",
		rampage = "HitSfx18",
		headshot = "HitSfx19",
		tank2 = "HitSfx20",
		rust = "HitSfx22",
	}

	if self.HITSOUND_PRESET == "bell" then
		local s = Instance.new("Sound")
		s.SoundId = "rbxassetid://7112391013"
		s.Volume = self.HITSOUND_VOLUME
		s.Parent = self.SoundService
		s:Play()
		self.Debris:AddItem(s, 3)
		return
	end

	if not self.HITSOUND_STORAGE then return end

	local assetName = HITSOUND_MAP[self.HITSOUND_PRESET]
	if not assetName then return end

	local src = self.HITSOUND_STORAGE:FindFirstChild(assetName)
	if not src then return end

	local sound
	if src:IsA("Sound") then
		sound = src:Clone()
	else
		sound = src:FindFirstChildWhichIsA("Sound", true)
		if sound then sound = sound:Clone() end
	end
	if not sound then return end

	sound.Volume = self.HITSOUND_VOLUME
	sound.Parent = self.SoundService
	sound:Play()
	self.Debris:AddItem(sound, math.max(sound.TimeLength + 0.3, 2))
end

function VisualsModule:CreateKeybindUI()
	local screenGui = Instance.new("ScreenGui")
	screenGui.Name = "NemesisKeybinds"
	screenGui.ResetOnSpawn = false
	screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
	screenGui.IgnoreGuiInset = true
	screenGui.Parent = self.guiParent

	local container = Instance.new("Frame")
	container.Name = "KeybindContainer"
	container.Size = UDim2.new(0, 220, 0, 0)
	container.Position = UDim2.new(1, -240, 0.5, -150)
	container.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
	container.BackgroundTransparency = 0.15
	container.BorderSizePixel = 0
	container.ClipsDescendants = true
	container.Parent = screenGui

	local blur = Instance.new("ImageLabel")
	blur.Name = "Blur"
	blur.Size = UDim2.new(1, 0, 1, 0)
	blur.BackgroundTransparency = 1
	blur.Image = "rbxasset://textures/ui/GuiImagePlaceholder.png"
	blur.ImageTransparency = 0.95
	blur.ScaleType = Enum.ScaleType.Tile
	blur.TileSize = UDim2.new(0, 128, 0, 128)
	blur.Parent = container

	local corner = Instance.new("UICorner")
	corner.CornerRadius = UDim.new(0, 8)
	corner.Parent = container

	local shadow = Instance.new("ImageLabel")
	shadow.Name = "Shadow"
	shadow.AnchorPoint = Vector2.new(0.5, 0.5)
	shadow.BackgroundTransparency = 1
	shadow.Position = UDim2.new(0.5, 0, 0.5, 0)
	shadow.Size = UDim2.new(1, 30, 1, 30)
	shadow.ZIndex = 0
	shadow.Image = "rbxassetid://5554236805"
	shadow.ImageColor3 = Color3.fromRGB(0, 0, 0)
	shadow.ImageTransparency = 0.5
	shadow.ScaleType = Enum.ScaleType.Slice
	shadow.SliceCenter = Rect.new(23, 23, 277, 277)
	shadow.Parent = container

	local titleBar = Instance.new("Frame")
	titleBar.Name = "TitleBar"
	titleBar.Size = UDim2.new(1, 0, 0, 32)
	titleBar.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
	titleBar.BackgroundTransparency = 0.3
	titleBar.BorderSizePixel = 0
	titleBar.Parent = container

	local titleCorner = Instance.new("UICorner")
	titleCorner.CornerRadius = UDim.new(0, 8)
	titleCorner.Parent = titleBar

	local titleFix = Instance.new("Frame")
	titleFix.Size = UDim2.new(1, 0, 0, 8)
	titleFix.Position = UDim2.new(0, 0, 1, -8)
	titleFix.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
	titleFix.BackgroundTransparency = 0.3
	titleFix.BorderSizePixel = 0
	titleFix.Parent = titleBar

	local titleLabel = Instance.new("TextLabel")
	titleLabel.Name = "Title"
	titleLabel.Size = UDim2.new(1, -16, 1, 0)
	titleLabel.Position = UDim2.new(0, 12, 0, 0)
	titleLabel.BackgroundTransparency = 1
	titleLabel.Text = "keybinds"
	titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
	titleLabel.TextSize = 13
	titleLabel.Font = Enum.Font.GothamBold
	titleLabel.TextXAlignment = Enum.TextXAlignment.Left
	titleLabel.TextYAlignment = Enum.TextYAlignment.Center
	titleLabel.Parent = titleBar

	local contentContainer = Instance.new("Frame")
	contentContainer.Name = "Content"
	contentContainer.Size = UDim2.new(1, 0, 1, -32)
	contentContainer.Position = UDim2.new(0, 0, 0, 32)
	contentContainer.BackgroundTransparency = 1
	contentContainer.BorderSizePixel = 0
	contentContainer.ClipsDescendants = true
	contentContainer.Parent = container

	local listLayout = Instance.new("UIListLayout")
	listLayout.SortOrder = Enum.SortOrder.LayoutOrder
	listLayout.Padding = UDim.new(0, 4)
	listLayout.Parent = contentContainer

	local listPadding = Instance.new("UIPadding")
	listPadding.PaddingTop = UDim.new(0, 8)
	listPadding.PaddingBottom = UDim.new(0, 8)
	listPadding.PaddingLeft = UDim.new(0, 12)
	listPadding.PaddingRight = UDim.new(0, 12)
	listPadding.Parent = contentContainer

	local dragging = false
	local dragInput, dragStart, startPos

	local function update(input)
		local delta = input.Position - dragStart
		local newPos = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
		local tweenInfo = TweenInfo.new(0.15, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
		self.TweenService:Create(container, tweenInfo, {Position = newPos}):Play()
	end

	titleBar.InputBegan:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging = true
			dragStart = input.Position
			startPos = container.Position
			input.Changed:Connect(function()
				if input.UserInputState == Enum.UserInputState.End then
					dragging = false
				end
			end)
		end
	end)

	titleBar.InputChanged:Connect(function(input)
		if input.UserInputType == Enum.UserInputType.MouseMovement then
			dragInput = input
		end
	end)

	if self.UserInputService then
		self.UserInputService.InputChanged:Connect(function(input)
			if input == dragInput and dragging then
				update(input)
			end
		end)
	end

	self.KeybindUI.Container = container
	self.KeybindUI.ContentContainer = contentContainer
	self.KeybindUI.Visible = true
end

function VisualsModule:UpdateKeybindDisplay()
	if not self.KeybindUI.ContentContainer then return end

	for _, child in ipairs(self.KeybindUI.ContentContainer:GetChildren()) do
		if child:IsA("Frame") and child.Name:match("^Bind_") then
			child:Destroy()
		end
	end

	local activeCount = 0
	for name, data in pairs(self.KeybindUI.ActiveBinds) do
		if data.enabled then
			activeCount += 1

			local bindFrame = Instance.new("Frame")
			bindFrame.Name = "Bind_" .. name
			bindFrame.Size = UDim2.new(1, 0, 0, 24)
			bindFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
			bindFrame.BackgroundTransparency = 0.4
			bindFrame.BorderSizePixel = 0
			bindFrame.LayoutOrder = activeCount
			bindFrame.Parent = self.KeybindUI.ContentContainer

			local bindCorner = Instance.new("UICorner")
			bindCorner.CornerRadius = UDim.new(0, 4)
			bindCorner.Parent = bindFrame

			local nameLabel = Instance.new("TextLabel")
			nameLabel.Name = "NameLabel"
			nameLabel.Size = UDim2.new(1, -50, 1, 0)
			nameLabel.Position = UDim2.new(0, 8, 0, 0)
			nameLabel.BackgroundTransparency = 1
			nameLabel.Text = name
			nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
			nameLabel.TextSize = 12
			nameLabel.Font = Enum.Font.Gotham
			nameLabel.TextXAlignment = Enum.TextXAlignment.Left
			nameLabel.TextYAlignment = Enum.TextYAlignment.Center
			nameLabel.TextTransparency = 0
			nameLabel.Parent = bindFrame

			local stateLabel = Instance.new("TextLabel")
			stateLabel.Name = "StateLabel"
			stateLabel.Size = UDim2.new(0, 40, 1, 0)
			stateLabel.Position = UDim2.new(1, -48, 0, 0)
			stateLabel.BackgroundTransparency = 1
			stateLabel.Text = "[" .. (data.state or "on") .. "]"
			stateLabel.TextColor3 = Color3.fromRGB(180, 180, 180)
			stateLabel.TextSize = 11
			stateLabel.Font = Enum.Font.GothamMedium
			stateLabel.TextXAlignment = Enum.TextXAlignment.Right
			stateLabel.TextYAlignment = Enum.TextYAlignment.Center
			stateLabel.TextTransparency = 0
			stateLabel.Parent = bindFrame

			bindFrame.BackgroundTransparency = 1
			nameLabel.TextTransparency = 1
			stateLabel.TextTransparency = 1

			local tweenInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
			self.TweenService:Create(bindFrame, tweenInfo, {BackgroundTransparency = 0.4}):Play()
			self.TweenService:Create(nameLabel, tweenInfo, {TextTransparency = 0}):Play()
			self.TweenService:Create(stateLabel, tweenInfo, {TextTransparency = 0}):Play()
		end
	end

	local totalHeight = 32 + 16 + (activeCount * 28)
	if activeCount == 0 then totalHeight = 32 end

	local tweenInfo = TweenInfo.new(0.4, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
	self.TweenService:Create(self.KeybindUI.Container, tweenInfo, {Size = UDim2.new(0, 220, 0, totalHeight)}):Play()

	if activeCount == 0 and self.KeybindUI.Visible then
		task.wait(0.1)
		local fadeInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
		self.TweenService:Create(self.KeybindUI.Container, fadeInfo, {BackgroundTransparency = 1}):Play()
		self.TweenService:Create(self.KeybindUI.Container.TitleBar, fadeInfo, {BackgroundTransparency = 1}):Play()
		self.TweenService:Create(self.KeybindUI.Container.TitleBar.Title, fadeInfo, {TextTransparency = 1}):Play()
	elseif activeCount > 0 then
		local fadeInfo = TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.Out)
		self.TweenService:Create(self.KeybindUI.Container, fadeInfo, {BackgroundTransparency = 0.15}):Play()
		self.TweenService:Create(self.KeybindUI.Container.TitleBar, fadeInfo, {BackgroundTransparency = 0.3}):Play()
		self.TweenService:Create(self.KeybindUI.Container.TitleBar.Title, fadeInfo, {TextTransparency = 0}):Play()
	end

	if self.KeybindUI.Container then
		self.KeybindUI.Container.Visible = self.KeybindUI.Visible and (activeCount > 0)
	end
end

function VisualsModule:RegisterKeybind(name, keyCode, callback, mode)
	if not self.UserInputService then return end

	mode = mode or "toggle"
	self.KeybindUI.ActiveBinds[name] = {
		key = keyCode,
		callback = callback,
		mode = mode,
		enabled = false,
		state = mode == "toggle" and "off" or "inactive",
	}

	if not self._bindInputBegan then
		self._bindInputBegan = self.UserInputService.InputBegan:Connect(function(input, gpe)
			if gpe then return end
			if input.UserInputType ~= Enum.UserInputType.Keyboard then return end

			for _, bind in pairs(self.KeybindUI.ActiveBinds) do
				if bind.key == input.KeyCode then
					if bind.mode == "toggle" then
						bind.enabled = not bind.enabled
						bind.state = bind.enabled and "on" or "off"
						if bind.callback then bind.callback(bind.enabled) end
					else
						bind.enabled = true
						bind.state = "active"
						if bind.callback then bind.callback(true) end
					end
					self:UpdateKeybindDisplay()
				end
			end
		end)

		self._bindInputEnded = self.UserInputService.InputEnded:Connect(function(input, gpe)
			if input.UserInputType ~= Enum.UserInputType.Keyboard then return end

			for _, bind in pairs(self.KeybindUI.ActiveBinds) do
				if bind.mode == "hold" and bind.key == input.KeyCode then
					bind.enabled = false
					bind.state = "inactive"
					if bind.callback then bind.callback(false) end
					self:UpdateKeybindDisplay()
				end
			end
		end)
	end

	self:UpdateKeybindDisplay()
end

function VisualsModule:CreateOffscreenArrows()
	local OffscreenGui = Instance.new("ScreenGui")
	OffscreenGui.Name = "NemesisOffscreen"
	OffscreenGui.IgnoreGuiInset = true
	OffscreenGui.ResetOnSpawn = false
	OffscreenGui.Parent = self.guiParent

	self.OffscreenGui = OffscreenGui

	local function isEnemy(plr)
		if self.player.Team and plr.Team then
			return self.player.Team ~= plr.Team
		end
		return true
	end

	local function createArrow(plr)
		local img = Instance.new("ImageLabel")
		img.Name = plr.Name .. "_Arrow"
		img.Size = UDim2.fromOffset(45, 45)
img:SetAttribute("BaseSize", 45)
		img.AnchorPoint = Vector2.new(0.5, 0.5)
		img.BackgroundTransparency = 1
		img.Image = "rbxassetid://121771059734689"
		img.Visible = false
		img.ImageTransparency = 0.3
		img.Parent = OffscreenGui
		self.arrows[plr] = img
	end

	local function removeArrow(plr)
		if self.arrows[plr] then
			self.arrows[plr]:Destroy()
			self.arrows[plr] = nil
		end
	end

	self.Players.PlayerAdded:Connect(function(plr)
		if plr ~= self.player and isEnemy(plr) then
			createArrow(plr)
		end
	end)
	self.Players.PlayerRemoving:Connect(removeArrow)

	for _, p in ipairs(self.Players:GetPlayers()) do
		if p ~= self.player and isEnemy(p) then
			createArrow(p)
		end
	end

	local function refreshArrows()
		for plr, arrow in pairs(self.arrows) do
			if not isEnemy(plr) then
				arrow:Destroy()
				self.arrows[plr] = nil
			end
		end

		for _, plr in ipairs(self.Players:GetPlayers()) do
			if plr ~= self.player and isEnemy(plr) and not self.arrows[plr] then
				createArrow(plr)
			end
		end
	end

	self.player:GetPropertyChangedSignal("Team"):Connect(refreshArrows)

	self.RunService.RenderStepped:Connect(function()
local t = tick()
		self.Camera = self.Workspace.CurrentCamera or self.Camera
		local Camera = self.Camera
		if not Camera then return end

		if not self.OFFSCREEN_ENABLED then
			for _, a in pairs(self.arrows) do
				a.Visible = false
			end
			return
		end

		local camCF = Camera.CFrame
		local camPos = camCF.Position
		local vp = Camera.ViewportSize
		local center = Vector2.new(vp.X / 2, vp.Y / 2)

		for plr, arrow in pairs(self.arrows) do
			if not isEnemy(plr) then
				arrow.Visible = false
				continue
			end

			local char = plr.Character
			local hrp = char and char:FindFirstChild("HumanoidRootPart")
			local hum = char and char:FindFirstChild("Humanoid")

			if not hrp or not hum or hum.Health <= 0 then
				arrow.Visible = false
				continue
			end

			local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)

			if onScreen and screenPos.Z > 0 then
				if screenPos.X > 0 and screenPos.X < vp.X and screenPos.Y > 0 and screenPos.Y < vp.Y then
					arrow.Visible = false
					continue
				end
			end

			local toPlayer = (hrp.Position - camPos).Unit
			local camRight = camCF.RightVector
			local x = toPlayer:Dot(camRight)
			local z = toPlayer:Dot(camCF.LookVector)

			local angle = math.atan2(x, z)
			local rot = math.deg(angle)

			local maxRadius = math.min(420, math.min(vp.X, vp.Y) * 0.35)
			local pos = center + Vector2.new(math.sin(angle), -math.cos(angle)) * maxRadius

			arrow.Position = UDim2.fromOffset(pos.X, pos.Y)
			arrow.Rotation = rot + 180
			arrow.ImageColor3 = self.OFFSCREEN_COLOR
			arrow.Visible = true

local pulse = math.abs(math.sin(t * 4.5))
arrow.ImageTransparency = 0.1 + pulse * 0.65
		end
	end)
end

function VisualsModule:CreateTracer(startPos, endPos)
	if not self.TRACER_ENABLED then return end

	local distance = (endPos - startPos).Magnitude
	if distance < 0.5 then return end

	if self.WALLBANG_ENABLED then
		local direction = (endPos - startPos)
		local dirUnit = direction.Unit

		local params = RaycastParams.new()
		params.FilterType = Enum.RaycastFilterType.Exclude
		local char = self.player.Character
		if char then
			params.FilterDescendantsInstances = { char }
		end

		local rayForward = self.Workspace:Raycast(startPos, dirUnit * distance, params)
		if rayForward then
			local rayBackward = self.Workspace:Raycast(endPos, -dirUnit * distance, params)
			if rayBackward then
				local penetrationDepth = (rayForward.Position - rayBackward.Position).Magnitude
				if penetrationDepth > 0.5 and penetrationDepth < (distance * 0.95) then
					task.spawn(function()
						self:CreateWallbangMarker(rayForward.Position, self.WALLBANG_ENTRY_COLOR)
						self:CreateWallbangMarker(rayBackward.Position, self.WALLBANG_EXIT_COLOR)
					end)
				end
			end
		end
	end

	local tracerObj = table.remove(self.tracerPool)
	if not tracerObj then
		local folder = Instance.new("Folder")
		folder.Name = "tracer_fx"

		local mover = Instance.new("Part")
		mover.Name = "Mover"
		mover.Anchored = true
		mover.Transparency = 1
		mover.CanCollide = false
		mover.CanQuery = false
		mover.CanTouch = false
		mover.Size = Vector3.new(0.05, 0.05, 0.05)
		mover.Parent = folder

		local a0 = Instance.new("Attachment")
		a0.Name = "A0"
		a0.Parent = mover

		local a1 = Instance.new("Attachment")
		a1.Name = "A1"
		a1.Parent = mover

		local trail = Instance.new("Trail")
		trail.Name = "Trace"
		trail.Attachment0 = a0
		trail.Attachment1 = a1
		trail.Enabled = false
		trail.FaceCamera = true
		trail.Brightness = self.TRACER_BRIGHTNESS
		trail.LightEmission = self.TRACER_LIGHTEMISSION
		trail.LightInfluence = self.TRACER_LIGHTINFLUENCE
		trail.TextureMode = Enum.TextureMode.Stretch
		trail.Texture = "rbxassetid://93746968407218"
		trail.TextureLength = 3
		trail.Transparency = self.TRACER_TRANSPARENCY
		trail.Parent = mover

		tracerObj = { folder = folder, mover = mover, a0 = a0, a1 = a1, trail = trail }
	end

	local w = math.clamp(self.TRACER_WIDTH, 0.02, 0.25)

	tracerObj.trail.Color = ColorSequence.new(self.TRACER_COLOR)
	tracerObj.trail.Lifetime = self.TRACER_LIFETIME
	tracerObj.trail.WidthScale = NumberSequence.new(w)
	tracerObj.trail.Transparency = self.TRACER_TRANSPARENCY
	tracerObj.trail.Brightness = self.TRACER_BRIGHTNESS
	tracerObj.trail.LightEmission = self.TRACER_LIGHTEMISSION
	tracerObj.trail.LightInfluence = self.TRACER_LIGHTINFLUENCE

	local dir = (endPos - startPos)
	local unit = dir.Unit

	tracerObj.folder.Parent = self.Workspace
	tracerObj.mover.CFrame = CFrame.lookAt(startPos, startPos + unit)
	tracerObj.a0.Position = Vector3.new(0, 0, 0)
	tracerObj.a1.Position = Vector3.new(0, 0, -self.TRACER_TAIL)
	tracerObj.trail.Enabled = true

	local flyTime = math.clamp(distance / 2200, 0.06, 0.22)
	local tween = self.TweenService:Create(tracerObj.mover, TweenInfo.new(flyTime, Enum.EasingStyle.Linear), {
		CFrame = CFrame.lookAt(endPos, endPos + unit),
	})
	tween:Play()

	tween.Completed:Once(function()
		task.delay(tracerObj.trail.Lifetime + 0.05, function()
			if not tracerObj or not tracerObj.folder then return end
			tracerObj.trail.Enabled = false
			tracerObj.folder.Parent = nil
			if #self.tracerPool < self.MAX_TRACER_POOL then
				table.insert(self.tracerPool, tracerObj)
			else
				tracerObj.folder:Destroy()
			end
		end)
	end)
end

function VisualsModule:CreateWallbangMarker(pos, color)
	local part = Instance.new("Part")
	part.Name = "WallbangMarker"
	part.Anchored = true
	part.CanCollide = false
	part.CanQuery = false
	part.CastShadow = false
	part.Material = Enum.Material.ForceField
	part.Color = color
	part.Size = self.WALLBANG_MARKER_SIZE
	part.CFrame = CFrame.new(pos)
	part.Parent = self.Workspace

	part.Transparency = 0
	local tIn = self.TweenService:Create(part, TweenInfo.new(0.3), { Transparency = 0 })
	tIn:Play()

	self.Debris:AddItem(part, self.WALLBANG_LIFETIME)

	task.delay(self.WALLBANG_LIFETIME - 0.5, function()
		if part.Parent then
			self.TweenService:Create(part, TweenInfo.new(0.5), { Transparency = 1, Size = Vector3.new(0, 0, 0) }):Play()
		end
	end)
end

function VisualsModule:CreateKillImageGUI()
	local gui = Instance.new("ScreenGui")
	gui.Name = "NemesisKillImage"
	gui.IgnoreGuiInset = true
	gui.ResetOnSpawn = false
	gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
	gui.DisplayOrder = 9999
	gui.Parent = self.guiParent

	local img = Instance.new("ImageLabel")
	img.Name = "Effect"
	img.Size = UDim2.fromScale(1, 1)
	img.Position = UDim2.fromScale(0.5, 0.5)
	img.AnchorPoint = Vector2.new(0.5, 0.5)
	img.BackgroundTransparency = 1
	img.Image = "http://www.roblox.com/asset/?id=3124804643"
	img.ImageTransparency = 1
	img.ScaleType = Enum.ScaleType.Stretch
	img.ZIndex = 999
	img.Parent = gui

	self.killImage = img
end

function VisualsModule:PlayKillImage()
	if not self.KILLIMAGE_ENABLED then return end
	if not self.killImage then return end

	self.killImage.ImageColor3 = self.KILLIMAGE_COLOR
	self.killImage.ImageTransparency = 1

	local tIn = self.TweenService:Create(
		self.killImage,
		TweenInfo.new(self.KILLIMAGE_FADE_IN, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
		{ ImageTransparency = 1 - self.KILLIMAGE_MAX_ALPHA }
	)

	local tOut = self.TweenService:Create(
		self.killImage,
		TweenInfo.new(self.KILLIMAGE_FADE_OUT, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
		{ ImageTransparency = 1 }
	)

	tIn:Play()
	tIn.Completed:Wait()
	task.wait(self.KILLIMAGE_HOLD)
	tOut:Play()
end

function VisualsModule:FixFxInstance(fx, cf)
	if fx:IsA("Model") then
		for _, d in ipairs(fx:GetDescendants()) do
			if d:IsA("BasePart") then
				d.Anchored = true
				d.CanCollide = false
				d.CanQuery = false
				d.CanTouch = false
			end
		end
		fx:PivotTo(cf)
	elseif fx:IsA("BasePart") then
		fx.Anchored = true
		fx.CanCollide = false
		fx.CanQuery = false
		fx.CanTouch = false
		fx.CFrame = cf
	end
end

function VisualsModule:RecolorParticles(root, color)
	for _, d in ipairs(root:GetDescendants()) do
		if d:IsA("ParticleEmitter") then
			d.Color = ColorSequence.new(color)
		end
	end
end

function VisualsModule:PlayImpactFX(position)
	local template = self.FX_ROOT:FindFirstChild("ImpactFX")
	if not template then return end

	local fx = template:Clone()
	self:FixFxInstance(fx, CFrame.new(position))
	fx.Parent = self.Workspace

	self:RecolorParticles(fx, self.KILLFX_COLOR)

	for _, d in ipairs(fx:GetDescendants()) do
		if d:IsA("ParticleEmitter") then
			d.Enabled = true
		end
	end

	task.delay(0.15, function()
		if fx and fx.Parent then
			for _, d in ipairs(fx:GetDescendants()) do
				if d:IsA("ParticleEmitter") then
					d.Enabled = false
				end
			end
		end
	end)

	self.Debris:AddItem(fx, 2)
end

function VisualsModule:PlayCustomFX(position)
	local template = self.FX_ROOT:FindFirstChild("CustomFX")
	if not template then return end

	local fx = template:Clone()
	self:FixFxInstance(fx, CFrame.new(position))
	fx.Parent = self.Workspace

	self:RecolorParticles(fx, self.KILLFX_COLOR)

	local light = fx:FindFirstChild("PointLight") or fx:FindFirstChildWhichIsA("PointLight", true)
	if light then
		light.Color = self.KILLFX_LIGHT_COLOR
		light.Range = 0
		light.Brightness = self.KILLFX_LIGHT_BRIGHTNESS
		light.Enabled = true

		local tIn = self.TweenService:Create(light, TweenInfo.new(self.KILLFX_CUSTOM_DURATION * 0.5, Enum.EasingStyle.Quad), { Range = self.KILLFX_LIGHT_RANGE })
		local tOut = self.TweenService:Create(light, TweenInfo.new(self.KILLFX_CUSTOM_DURATION * 0.5, Enum.EasingStyle.Quad), { Range = 0 })

		tIn:Play()
		tIn.Completed:Once(function()
			tOut:Play()
		end)
	end

	for _, d in ipairs(fx:GetDescendants()) do
		if d:IsA("ParticleEmitter") then
			d.Enabled = true
		end
	end

	task.delay(self.KILLFX_CUSTOM_DURATION, function()
		if fx and fx.Parent then
			for _, d in ipairs(fx:GetDescendants()) do
				if d:IsA("ParticleEmitter") then
					d.Enabled = false
				end
			end
		end
	end)

	self.Debris:AddItem(fx, 2)
end

function VisualsModule:SetupKillFXListener()
	local InfEvent = self.ReplicatedStorage:WaitForChild("inf", 10)
	if InfEvent then
		InfEvent.OnClientEvent:Connect(function(killedPlayerName)
			if not self.KILLFX_ENABLED then return end

			local killedPlayer = self.Players:FindFirstChild(killedPlayerName)
			if not killedPlayer then return end

			local char = killedPlayer.Character
			if not char then return end

			local head = char:FindFirstChild("Head")
			if not head then return end

			local pos = head.Position
			if self.KILLFX_TYPE == "ImpactFX" then
				self:PlayImpactFX(pos)
			elseif self.KILLFX_TYPE == "CustomFX" then
				self:PlayCustomFX(pos)
			end
		end)
	end

	task.spawn(function()
		local htl = self.ReplicatedStorage:WaitForChild("htl", 10)
		if not htl then return end

		htl.OnClientEvent:Connect(function(hitType, info)
			if hitType == "Hit" then
				self:PlayHitSound()
			end

			if hitType == "Hit" and info and self.KILLIMAGE_ENABLED then
				local dmg = info.damage or 0
				if dmg > 99 then
					self:PlayKillImage()
				end
			end

			if self.HITLOG_ENABLED then
				self:ShowHitlog(hitType, info)
			end
		end)
	end)
end

function VisualsModule:IsFakeLagEnabledFallback()
	if typeof(self.IsFakeLagEnabled) == "function" then
		local ok, v = pcall(self.IsFakeLagEnabled)
		if ok then return v == true end
	end
	if _G.ConfigSystem and _G.ConfigSystem.settings then
		return _G.ConfigSystem.settings["FakeLag"] == true
	end
	return false
end

function VisualsModule:SetupBacktrackGhost()
	local backtrackRemote = self.ReplicatedStorage:WaitForChild("btl")
	local lastSend = 0

	local ghostsFolder = self.Workspace:FindFirstChild("BacktrackGhosts") or Instance.new("Folder", self.Workspace)
	ghostsFolder.Name = "BacktrackGhosts"
	self._ghostsFolder = ghostsFolder

	self.RunService.Heartbeat:Connect(function()
		local now = tick()
		local dt = now - self._hbLast
		self._hbLast = now
		self._ghostDt = dt

		if self.ghostModel and now > self.ghostExpireAt then
			self:ClearBacktrackGhost()
		end

		if self.BACKTRACK_GHOST_ENABLED and self.player.Character then
			local hrp = self.player.Character:FindFirstChild("HumanoidRootPart")
			if hrp then
				if now - lastSend > 0.05 then
					lastSend = now
					backtrackRemote:FireServer(self:IsFakeLagEnabledFallback())
				end
			end
		end
	end)

	backtrackRemote.OnClientEvent:Connect(function(cframe, isFakelag, ping)
		if not self.BACKTRACK_GHOST_ENABLED then return end

		if not isFakelag and ping then
			local char = self.player.Character
			if char then
				local hrp = char:FindFirstChild("HumanoidRootPart")
				if hrp then
					local velocity = hrp.AssemblyLinearVelocity
					local pingOffset = velocity * (ping * 0.5)
					cframe = hrp.CFrame - pingOffset
				end
			end
		end

		self:UpdateBacktrackGhostPose(cframe, isFakelag)
	end)

	self.player.CharacterAdded:Connect(function()
		self:ClearBacktrackGhost()
	end)

	self.player.CharacterRemoving:Connect(function()
		self:ClearBacktrackGhost()
	end)
end

function VisualsModule:ClearBacktrackGhost()
	if self.ghostModel then
		self.ghostModel:Destroy()
		self.ghostModel = nil
		table.clear(self.ghostParts)
		table.clear(self.ghostLastCFrames)
	end
	self.sourceCharacter = nil
	self.lastFakeLagState = nil
end

function VisualsModule:CreateBacktrackGhost()
	local srcChar = self.player.Character
	if not srcChar then return nil end

	local root = srcChar:FindFirstChild("HumanoidRootPart")
	if not root then return nil end

	srcChar.Archivable = true
	local ghost = srcChar:Clone()
	srcChar.Archivable = false
	if not ghost then return nil end

	ghost.Name = self.player.Name .. "_BacktrackGhost"

	for _, v in pairs(ghost:GetDescendants()) do
		if v:IsA("Script") or v:IsA("LocalScript") or v:IsA("Sound")
			or v:IsA("BillboardGui") or v:IsA("Humanoid")
			or v:IsA("AlignPosition") or v:IsA("AlignOrientation")
			or v:IsA("BodyMover") or v:IsA("BodyGyro") or v:IsA("BodyVelocity")
			or v:IsA("BodyPosition") or v:IsA("BodyForce") or v:IsA("VectorForce")
			or v:IsA("LineForce") or v:IsA("Animator") then
			v:Destroy()
		end
	end

	for _, v in pairs(ghost:GetDescendants()) do
		if v:IsA("Motor6D") or v:IsA("Weld") or v:IsA("WeldConstraint")
			or v:IsA("HingeConstraint") or v:IsA("BallSocketConstraint")
			or v:IsA("Constraint") or v:IsA("JointInstance") then
			v:Destroy()
		end
	end

	local hasParts = false
	for _, v in pairs(ghost:GetDescendants()) do
		if v:IsA("BasePart") then
			hasParts = true
			v.Anchored = true
			v.CanCollide = false
			v.CanTouch = false
			v.CanQuery = false
			v.Massless = true
			v.Material = Enum.Material.ForceField
			v.Transparency = self.GHOST_BASE_TRANSPARENCY
			v.Color = self.BACKTRACK_GHOST_COLOR
			v.CastShadow = false

			if v.Name == "Head" then
				local face = v:FindFirstChild("face")
				if face then face:Destroy() end
			end
		end
	end

	if not hasParts then
		ghost:Destroy()
		return nil
	end

	table.clear(self.ghostParts)
	for _, v in ipairs(ghost:GetDescendants()) do
		if v:IsA("BasePart") then
			table.insert(self.ghostParts, v)
		end
	end

	ghost.Parent = self._ghostsFolder
	return ghost
end

function VisualsModule:UpdateBacktrackGhostPose(targetCFrame, isFakelag)
	if not self.BACKTRACK_GHOST_ENABLED then return end

	local srcChar = self.player.Character
	if not srcChar then return end

	local humanoid = srcChar:FindFirstChild("Humanoid")
	if humanoid and humanoid.Health <= 0 then
		self:ClearBacktrackGhost()
		return
	end

	if (not self.ghostModel) or (self.sourceCharacter ~= srcChar) or (self.lastFakeLagState ~= isFakelag) or (not self.ghostModel.Parent) then
		self:ClearBacktrackGhost()
		self.ghostModel = self:CreateBacktrackGhost()
		if self.ghostModel then
			self.sourceCharacter = srcChar
			self.lastFakeLagState = isFakelag
		else
			return
		end
	end

	local srcRoot = srcChar:FindFirstChild("HumanoidRootPart")
	if not srcRoot then return end

	local currentRootCFrame = srcRoot.CFrame
	local offset = targetCFrame * currentRootCFrame:Inverse()

	local dt = tonumber(self._ghostDt) or (1 / 60)
	local alpha = math.clamp(self.GHOST_SMOOTH_SPEED * dt, 0, 1)

for _, ghostPart in ipairs(self.ghostParts) do
	local srcPart = srcChar:FindFirstChild(ghostPart.Name, true)
	if srcPart then
		local targetCF = offset * srcPart.CFrame
		local lastCF = self.ghostLastCFrames[ghostPart] or ghostPart.CFrame
		local newCF = lastCF:Lerp(targetCF, alpha)
		ghostPart.CFrame = newCF
		self.ghostLastCFrames[ghostPart] = newCF

		local dist = (newCF.Position - srcPart.Position).Magnitude

		if dist < self.GHOST_FADE_RADIUS then
			local k = math.clamp(1 - (dist / self.GHOST_FADE_RADIUS), 0, 1)
			k = k ^ self.GHOST_FADE_POWER
			ghostPart.Transparency = math.clamp(
				self.GHOST_BASE_TRANSPARENCY + k * (1 - self.GHOST_BASE_TRANSPARENCY),
				0,
				1
			)
		else
			ghostPart.Transparency = self.GHOST_BASE_TRANSPARENCY
		end
	end
end

	self.ghostExpireAt = tick() + self.BACKTRACK_GHOST_DURATION
end

function VisualsModule:SnapshotFirstEffect(className)
	local inst = self.Lighting:FindFirstChildOfClass(className)
	if not inst then return nil end
	local c = inst:Clone()
	c.Name = "NemesisOriginal_" .. className
	return c
end

function VisualsModule:WipeClass(className)
	for _, inst in ipairs(self.Lighting:GetDescendants()) do
		if inst.ClassName == className then
			inst:Destroy()
		end
	end
	for _, inst in ipairs(self.Lighting:GetChildren()) do
		if inst.ClassName == className then
			inst:Destroy()
		end
	end
end

function VisualsModule:ApplySingleEffect(className, enabled, applyFn)
	if enabled then
		self:WipeClass(className)
		local fx = Instance.new(className)
		fx.Name = "Nemesis_" .. className
		fx.Parent = self.Lighting
		applyFn(fx)
	else
		self:WipeClass(className)
		local original = self.ORIGINAL_EFFECTS[className]
		if original then
			original:Clone().Parent = self.Lighting
		end
	end
end

function VisualsModule:UpdateAtmosphere()
	self:ApplySingleEffect("Atmosphere", self.ATMOSPHERE_ENABLED, function(atmosphere)
		atmosphere.Density = self.ATMOSPHERE_DENSITY
		atmosphere.Offset = self.ATMOSPHERE_OFFSET
		atmosphere.Color = self.ATMOSPHERE_COLOR
		atmosphere.Decay = self.ATMOSPHERE_DECAY
		atmosphere.Glare = self.ATMOSPHERE_GLARE
		atmosphere.Haze = self.ATMOSPHERE_HAZE
	end)
end

function VisualsModule:SetAtmosphereEnabled(value)
	self.ATMOSPHERE_ENABLED = value
	self:UpdateAtmosphere()
end

function VisualsModule:SetAtmosphereDensity(value)
	self.ATMOSPHERE_DENSITY = value
	if self.ATMOSPHERE_ENABLED then self:UpdateAtmosphere() end
end

function VisualsModule:SetAtmosphereOffset(value)
	self.ATMOSPHERE_OFFSET = value
	if self.ATMOSPHERE_ENABLED then self:UpdateAtmosphere() end
end

function VisualsModule:SetAtmosphereColor(value)
	self.ATMOSPHERE_COLOR = value
	if self.ATMOSPHERE_ENABLED then self:UpdateAtmosphere() end
end

function VisualsModule:SetAtmosphereDecay(value)
	self.ATMOSPHERE_DECAY = value
	if self.ATMOSPHERE_ENABLED then self:UpdateAtmosphere() end
end

function VisualsModule:SetAtmosphereGlare(value)
	self.ATMOSPHERE_GLARE = value
	if self.ATMOSPHERE_ENABLED then self:UpdateAtmosphere() end
end

function VisualsModule:SetAtmosphereHaze(value)
	self.ATMOSPHERE_HAZE = value
	if self.ATMOSPHERE_ENABLED then self:UpdateAtmosphere() end
end

function VisualsModule:UpdateBloom()
	self:ApplySingleEffect("BloomEffect", self.BLOOM_ENABLED, function(bloom)
		bloom.Enabled = true
		bloom.Intensity = self.BLOOM_INTENSITY
		bloom.Size = self.BLOOM_SIZE
		bloom.Threshold = self.BLOOM_THRESHOLD
	end)
end

function VisualsModule:SetBloomEnabled(value)
	self.BLOOM_ENABLED = value
	self:UpdateBloom()
end

function VisualsModule:SetBloomIntensity(value)
	self.BLOOM_INTENSITY = value
	if self.BLOOM_ENABLED then self:UpdateBloom() end
end

function VisualsModule:SetBloomSize(value)
	self.BLOOM_SIZE = value
	if self.BLOOM_ENABLED then self:UpdateBloom() end
end

function VisualsModule:SetBloomThreshold(value)
	self.BLOOM_THRESHOLD = value
	if self.BLOOM_ENABLED then self:UpdateBloom() end
end

function VisualsModule:UpdateColorCorrection()
	self:ApplySingleEffect("ColorCorrectionEffect", self.COLOR_CORRECTION_ENABLED, function(cc)
		cc.Brightness = self.COLOR_CORRECTION_BRIGHTNESS
		cc.Contrast = self.COLOR_CORRECTION_CONTRAST
		cc.Saturation = self.COLOR_CORRECTION_SATURATION
		cc.TintColor = self.COLOR_CORRECTION_TINT
	end)
end

function VisualsModule:SetColorCorrectionEnabled(value)
	self.COLOR_CORRECTION_ENABLED = value
	self:UpdateColorCorrection()
end

function VisualsModule:SetColorCorrectionBrightness(value)
	self.COLOR_CORRECTION_BRIGHTNESS = value
	if self.COLOR_CORRECTION_ENABLED then self:UpdateColorCorrection() end
end

function VisualsModule:SetColorCorrectionContrast(value)
	self.COLOR_CORRECTION_CONTRAST = value
	if self.COLOR_CORRECTION_ENABLED then self:UpdateColorCorrection() end
end

function VisualsModule:SetColorCorrectionSaturation(value)
	self.COLOR_CORRECTION_SATURATION = value
	if self.COLOR_CORRECTION_ENABLED then self:UpdateColorCorrection() end
end

function VisualsModule:SetColorCorrectionTint(value)
	self.COLOR_CORRECTION_TINT = value
	if self.COLOR_CORRECTION_ENABLED then self:UpdateColorCorrection() end
end

function VisualsModule:SetHitlogEnabled(value)
	self.HITLOG_ENABLED = value
	if self.hitlogContainer then
		self.hitlogContainer.Visible = value
	end
end

function VisualsModule:SetHitSoundEnabled(value)
	self.HITSOUND_ENABLED = value
end

function VisualsModule:SetHitSoundPreset(value)
	self.HITSOUND_PRESET = value
end

function VisualsModule:SetHitSoundVolume(value)
	self.HITSOUND_VOLUME = value
end

function VisualsModule:SetKeybindListVisible(value)
	self.KeybindUI.Visible = value
	self.KEYBIND_LIST_VISIBLE = value
	if self.KeybindUI.Container then
		self.KeybindUI.Container.Visible = value
	end
	self:UpdateKeybindDisplay()
end

function VisualsModule:SetTracerEnabled(value)
	self.TRACER_ENABLED = value
end

function VisualsModule:SetTracerColor(value)
	self.TRACER_COLOR = value
end

function VisualsModule:SetTracerLifetime(value)
	self.TRACER_LIFETIME = value
end

function VisualsModule:SetTracerWidth(value)
	self.TRACER_WIDTH = value
end

function VisualsModule:SetWallbangEnabled(value)
	self.WALLBANG_ENABLED = value
end

function VisualsModule:SetWallbangEntryColor(value)
	self.WALLBANG_ENTRY_COLOR = value
end

function VisualsModule:SetWallbangExitColor(value)
	self.WALLBANG_EXIT_COLOR = value
end

function VisualsModule:SetWallbangMarkerSize(value)
	if typeof(value) == "number" then
		self.WALLBANG_MARKER_SIZE = Vector3.new(value, value, value)
	elseif typeof(value) == "Vector3" then
		self.WALLBANG_MARKER_SIZE = value
	end
end

function VisualsModule:SetWallbangLifetime(value)
	self.WALLBANG_LIFETIME = value
end

function VisualsModule:SetKillFXEnabled(value)
	self.KILLFX_ENABLED = value
end

function VisualsModule:SetKillFXType(value)
	self.KILLFX_TYPE = value
end

function VisualsModule:SetKillFXColor(value)
	self.KILLFX_COLOR = value
end

function VisualsModule:SetKillFXLightColor(value)
	self.KILLFX_LIGHT_COLOR = value
end

function VisualsModule:SetKillFXDuration(value)
	self.KILLFX_CUSTOM_DURATION = value
end

function VisualsModule:SetKillFXLightRange(value)
	self.KILLFX_LIGHT_RANGE = value
end

function VisualsModule:SetKillFXLightBrightness(value)
	self.KILLFX_LIGHT_BRIGHTNESS = value
end

function VisualsModule:SetOffscreenEnabled(value)
	self.OFFSCREEN_ENABLED = value
end

function VisualsModule:SetOffscreenColor(value)
	self.OFFSCREEN_COLOR = value
end

function VisualsModule:SetBacktrackGhostEnabled(value)
	self.BACKTRACK_GHOST_ENABLED = value
	if not value then
		self:ClearBacktrackGhost()
	end
end

function VisualsModule:SetBacktrackGhostColor(value)
	self.BACKTRACK_GHOST_COLOR = value
	for _, p in ipairs(self.ghostParts) do
		p.Color = value
		p.Transparency = self.GHOST_BASE_TRANSPARENCY
	end
end

function VisualsModule:SetBacktrackGhostDuration(value)
	self.BACKTRACK_GHOST_DURATION = value
end

function VisualsModule:SetKillImageEnabled(value)
	self.KILLIMAGE_ENABLED = value
end

function VisualsModule:SetKillImageColor(value)
	self.KILLIMAGE_COLOR = value
end

function VisualsModule:SetKillImageMaxAlpha(value)
	self.KILLIMAGE_MAX_ALPHA = value
end

function VisualsModule:GetSettings()
	local function colorToTable(c)
		return { math.floor(c.R * 255), math.floor(c.G * 255), math.floor(c.B * 255) }
	end

	return {
		HITLOG_ENABLED = self.HITLOG_ENABLED,
		KEYBIND_LIST_VISIBLE = self.KeybindUI.Visible,
		HITSOUND_ENABLED = self.HITSOUND_ENABLED,
		HITSOUND_PRESET = self.HITSOUND_PRESET,
		HITSOUND_VOLUME = self.HITSOUND_VOLUME,

		OFFSCREEN_ENABLED = self.OFFSCREEN_ENABLED,
		OFFSCREEN_COLOR = colorToTable(self.OFFSCREEN_COLOR),

		WALLBANG_ENABLED = self.WALLBANG_ENABLED,
		WALLBANG_ENTRY_COLOR = colorToTable(self.WALLBANG_ENTRY_COLOR),
		WALLBANG_EXIT_COLOR = colorToTable(self.WALLBANG_EXIT_COLOR),
		WALLBANG_MARKER_SIZE = self.WALLBANG_MARKER_SIZE.X,
		WALLBANG_LIFETIME = self.WALLBANG_LIFETIME,

		KILLFX_ENABLED = self.KILLFX_ENABLED,
		KILLFX_TYPE = self.KILLFX_TYPE,
		KILLFX_COLOR = colorToTable(self.KILLFX_COLOR),
		KILLFX_LIGHT_COLOR = colorToTable(self.KILLFX_LIGHT_COLOR),
		KILLFX_CUSTOM_DURATION = self.KILLFX_CUSTOM_DURATION,
		KILLFX_LIGHT_RANGE = self.KILLFX_LIGHT_RANGE,
		KILLFX_LIGHT_BRIGHTNESS = self.KILLFX_LIGHT_BRIGHTNESS,

		KILLIMAGE_ENABLED = self.KILLIMAGE_ENABLED,
		KILLIMAGE_COLOR = colorToTable(self.KILLIMAGE_COLOR),
		KILLIMAGE_MAX_ALPHA = self.KILLIMAGE_MAX_ALPHA,

		BACKTRACK_GHOST_ENABLED = self.BACKTRACK_GHOST_ENABLED,
		BACKTRACK_GHOST_COLOR = colorToTable(self.BACKTRACK_GHOST_COLOR),
		BACKTRACK_GHOST_DURATION = self.BACKTRACK_GHOST_DURATION,

		TRACER_ENABLED = self.TRACER_ENABLED,
		TRACER_COLOR = colorToTable(self.TRACER_COLOR),
		TRACER_LIFETIME = self.TRACER_LIFETIME,
		TRACER_WIDTH = self.TRACER_WIDTH,

		ATMOSPHERE_ENABLED = self.ATMOSPHERE_ENABLED,
		ATMOSPHERE_DENSITY = self.ATMOSPHERE_DENSITY,
		ATMOSPHERE_OFFSET = self.ATMOSPHERE_OFFSET,
		ATMOSPHERE_COLOR = colorToTable(self.ATMOSPHERE_COLOR),
		ATMOSPHERE_DECAY = colorToTable(self.ATMOSPHERE_DECAY),
		ATMOSPHERE_GLARE = self.ATMOSPHERE_GLARE,
		ATMOSPHERE_HAZE = self.ATMOSPHERE_HAZE,

		BLOOM_ENABLED = self.BLOOM_ENABLED,
		BLOOM_INTENSITY = self.BLOOM_INTENSITY,
		BLOOM_SIZE = self.BLOOM_SIZE,
		BLOOM_THRESHOLD = self.BLOOM_THRESHOLD,

		COLOR_CORRECTION_ENABLED = self.COLOR_CORRECTION_ENABLED,
		COLOR_CORRECTION_BRIGHTNESS = self.COLOR_CORRECTION_BRIGHTNESS,
		COLOR_CORRECTION_CONTRAST = self.COLOR_CORRECTION_CONTRAST,
		COLOR_CORRECTION_SATURATION = self.COLOR_CORRECTION_SATURATION,
		COLOR_CORRECTION_TINT = colorToTable(self.COLOR_CORRECTION_TINT),
	}
end

function VisualsModule:ApplySettings(settings)
	if not settings then return end

	local function toColor3(arr)
		if type(arr) == "table" and #arr >= 3 then
			return Color3.fromRGB(arr[1], arr[2], arr[3])
		end
		return nil
	end

	if settings.HITLOG_ENABLED ~= nil then self:SetHitlogEnabled(settings.HITLOG_ENABLED) end
	if settings.KEYBIND_LIST_VISIBLE ~= nil then self:SetKeybindListVisible(settings.KEYBIND_LIST_VISIBLE) end
	if settings.HITSOUND_ENABLED ~= nil then self:SetHitSoundEnabled(settings.HITSOUND_ENABLED) end
	if settings.HITSOUND_PRESET ~= nil then self:SetHitSoundPreset(settings.HITSOUND_PRESET) end
	if settings.HITSOUND_VOLUME ~= nil then self:SetHitSoundVolume(settings.HITSOUND_VOLUME) end

	if settings.OFFSCREEN_ENABLED ~= nil then self:SetOffscreenEnabled(settings.OFFSCREEN_ENABLED) end
	if settings.OFFSCREEN_COLOR then self:SetOffscreenColor(toColor3(settings.OFFSCREEN_COLOR) or self.OFFSCREEN_COLOR) end

	if settings.WALLBANG_ENABLED ~= nil then self:SetWallbangEnabled(settings.WALLBANG_ENABLED) end
	if settings.WALLBANG_ENTRY_COLOR then self:SetWallbangEntryColor(toColor3(settings.WALLBANG_ENTRY_COLOR) or self.WALLBANG_ENTRY_COLOR) end
	if settings.WALLBANG_EXIT_COLOR then self:SetWallbangExitColor(toColor3(settings.WALLBANG_EXIT_COLOR) or self.WALLBANG_EXIT_COLOR) end
	if settings.WALLBANG_MARKER_SIZE ~= nil then self:SetWallbangMarkerSize(settings.WALLBANG_MARKER_SIZE) end
	if settings.WALLBANG_LIFETIME ~= nil then self:SetWallbangLifetime(settings.WALLBANG_LIFETIME) end

	if settings.KILLFX_ENABLED ~= nil then self:SetKillFXEnabled(settings.KILLFX_ENABLED) end
	if settings.KILLFX_TYPE ~= nil then self:SetKillFXType(settings.KILLFX_TYPE) end
	if settings.KILLFX_COLOR then self:SetKillFXColor(toColor3(settings.KILLFX_COLOR) or self.KILLFX_COLOR) end
	if settings.KILLFX_LIGHT_COLOR then self:SetKillFXLightColor(toColor3(settings.KILLFX_LIGHT_COLOR) or self.KILLFX_LIGHT_COLOR) end
	if settings.KILLFX_CUSTOM_DURATION ~= nil then self:SetKillFXDuration(settings.KILLFX_CUSTOM_DURATION) end
	if settings.KILLFX_LIGHT_RANGE ~= nil then self:SetKillFXLightRange(settings.KILLFX_LIGHT_RANGE) end
	if settings.KILLFX_LIGHT_BRIGHTNESS ~= nil then self:SetKillFXLightBrightness(settings.KILLFX_LIGHT_BRIGHTNESS) end

	if settings.KILLIMAGE_ENABLED ~= nil then self:SetKillImageEnabled(settings.KILLIMAGE_ENABLED) end
	if settings.KILLIMAGE_COLOR then self:SetKillImageColor(toColor3(settings.KILLIMAGE_COLOR) or self.KILLIMAGE_COLOR) end
	if settings.KILLIMAGE_MAX_ALPHA ~= nil then self:SetKillImageMaxAlpha(settings.KILLIMAGE_MAX_ALPHA) end

	if settings.BACKTRACK_GHOST_ENABLED ~= nil then self:SetBacktrackGhostEnabled(settings.BACKTRACK_GHOST_ENABLED) end
	if settings.BACKTRACK_GHOST_COLOR then self:SetBacktrackGhostColor(toColor3(settings.BACKTRACK_GHOST_COLOR) or self.BACKTRACK_GHOST_COLOR) end
	if settings.BACKTRACK_GHOST_DURATION ~= nil then self:SetBacktrackGhostDuration(settings.BACKTRACK_GHOST_DURATION) end

	if settings.TRACER_ENABLED ~= nil then self:SetTracerEnabled(settings.TRACER_ENABLED) end
	if settings.TRACER_COLOR then self:SetTracerColor(toColor3(settings.TRACER_COLOR) or self.TRACER_COLOR) end
	if settings.TRACER_LIFETIME ~= nil then self:SetTracerLifetime(settings.TRACER_LIFETIME) end
	if settings.TRACER_WIDTH ~= nil then self:SetTracerWidth(settings.TRACER_WIDTH) end

	if settings.ATMOSPHERE_ENABLED ~= nil then self:SetAtmosphereEnabled(settings.ATMOSPHERE_ENABLED) end
	if settings.ATMOSPHERE_DENSITY ~= nil then self:SetAtmosphereDensity(settings.ATMOSPHERE_DENSITY) end
	if settings.ATMOSPHERE_OFFSET ~= nil then self:SetAtmosphereOffset(settings.ATMOSPHERE_OFFSET) end
	if settings.ATMOSPHERE_COLOR then self:SetAtmosphereColor(toColor3(settings.ATMOSPHERE_COLOR) or self.ATMOSPHERE_COLOR) end
	if settings.ATMOSPHERE_DECAY then self:SetAtmosphereDecay(toColor3(settings.ATMOSPHERE_DECAY) or self.ATMOSPHERE_DECAY) end
	if settings.ATMOSPHERE_GLARE ~= nil then self:SetAtmosphereGlare(settings.ATMOSPHERE_GLARE) end
	if settings.ATMOSPHERE_HAZE ~= nil then self:SetAtmosphereHaze(settings.ATMOSPHERE_HAZE) end

	if settings.BLOOM_ENABLED ~= nil then self:SetBloomEnabled(settings.BLOOM_ENABLED) end
	if settings.BLOOM_INTENSITY ~= nil then self:SetBloomIntensity(settings.BLOOM_INTENSITY) end
	if settings.BLOOM_SIZE ~= nil then self:SetBloomSize(settings.BLOOM_SIZE) end
	if settings.BLOOM_THRESHOLD ~= nil then self:SetBloomThreshold(settings.BLOOM_THRESHOLD) end

	if settings.COLOR_CORRECTION_ENABLED ~= nil then self:SetColorCorrectionEnabled(settings.COLOR_CORRECTION_ENABLED) end
	if settings.COLOR_CORRECTION_BRIGHTNESS ~= nil then self:SetColorCorrectionBrightness(settings.COLOR_CORRECTION_BRIGHTNESS) end
	if settings.COLOR_CORRECTION_CONTRAST ~= nil then self:SetColorCorrectionContrast(settings.COLOR_CORRECTION_CONTRAST) end
	if settings.COLOR_CORRECTION_SATURATION ~= nil then self:SetColorCorrectionSaturation(settings.COLOR_CORRECTION_SATURATION) end
	if settings.COLOR_CORRECTION_TINT then self:SetColorCorrectionTint(toColor3(settings.COLOR_CORRECTION_TINT) or self.COLOR_CORRECTION_TINT) end
end

function VisualsModule:Start()
end

return VisualsModule
