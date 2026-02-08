local Lighting = game:GetService("Lighting")

local World = {}
World.__index = World

function World.new()
	local self = setmetatable({}, World)

	self.ORIGINAL_LIGHTING = {
		Brightness = Lighting.Brightness,
		Ambient = Lighting.Ambient,
		OutdoorAmbient = Lighting.OutdoorAmbient,
		ClockTime = Lighting.ClockTime
	}

	self.LIGHTING_EDIT_ENABLED = false
	self.CUSTOM_BRIGHTNESS = Lighting.Brightness
	self.CUSTOM_AMBIENT = Lighting.Ambient
	self.CUSTOM_OUTDOOR_AMBIENT = Lighting.OutdoorAmbient

	self.WORLD_TIME_ENABLED = false
	self.CUSTOM_CLOCKTIME = Lighting.ClockTime
	self.TIME_PRESET = "Default"
	self.TIME_PRESETS = {
		Default = nil,
		Day = 14,
		Sunset = 18.5,
		Night = 0
	}

self.CUSTOM_SKY_ENABLED = false
self.CUSTOM_SKY_PRESET = "Original"

self.SKY_PRESETS = {
	Original = {
		SkyboxBk = "rbxassetid://148345460",
		SkyboxDn = "rbxassetid://148345469",
		SkyboxFt = "rbxassetid://148345455",
		SkyboxLf = "rbxassetid://148345446",
		SkyboxRt = "rbxassetid://148345440",
		SkyboxUp = "rbxassetid://148345466",
		SunTextureId = "rbxassetid://1345009717",
		MoonTextureId = "rbxasset://sky/moon.jpg",
		StarCount = 3000
	},

	Blue = {
		SkyboxBk = "rbxassetid://124106120273678",
		SkyboxDn = "rbxassetid://72406028001064",
		SkyboxFt = "rbxassetid://139238600782726",
		SkyboxLf = "rbxassetid://139363840022859",
		SkyboxRt = "rbxassetid://122628916393316",
		SkyboxUp = "rbxassetid://104354476381566",
		StarCount = 3000
	},

	Gray = {
		CelestialBodiesShown = true,
		MoonAngularSize = 6,
		MoonTextureId = "rbxasset://sky/moon.jpg",
		SkyboxBk = "rbxassetid://15063412549",
		SkyboxDn = "rbxassetid://15063431496",
		SkyboxFt = "rbxassetid://15063442046",
		SkyboxLf = "rbxassetid://15063637382",
		SkyboxRt = "rbxassetid://15063435355",
		SkyboxUp = "rbxassetid://15063533447",
		StarCount = 3000,
		SunAngularSize = 9,
		SunTextureId = "rbxasset://sky/sun.jpg"
	},

	Dark = {
		CelestialBodiesShown = true,
		MoonAngularSize = 11,
		MoonTextureId = "rbxasset://sky/moon.jpg",
		SkyboxBk = "rbxassetid://2013298",
		SkyboxDn = "rbxassetid://2013298",
		SkyboxFt = "rbxassetid://2013298",
		SkyboxLf = "rbxassetid://2013298",
		SkyboxRt = "rbxassetid://2013298",
		SkyboxUp = "rbxassetid://2013298",
		StarCount = 3000,
		SunAngularSize = 21,
		SunTextureId = "rbxasset://sky/sun.jpg"
	},

	Green = {
		CelestialBodiesShown = false,
		MoonAngularSize = 11,
		MoonTextureId = "rbxasset://sky/moon.jpg",
		SkyboxBk = "rbxassetid://56661187",
		SkyboxDn = "rbxassetid://566611398",
		SkyboxFt = "rbxassetid://566611142",
		SkyboxLf = "rbxassetid://566611266",
		SkyboxRt = "rbxassetid://566611300",
		SkyboxUp = "rbxassetid://566611218",
		StarCount = 3000,
		SunAngularSize = 21,
		SunTextureId = "rbxasset://sky/sun.jpg"
	},

	Pink = {
		CelestialBodiesShown = false,
		MoonAngularSize = 11,
		MoonTextureId = "rbxasset://sky/moon.jpg",
		SkyboxBk = "rbxassetid://271042516",
		SkyboxDn = "rbxassetid://271077243",
		SkyboxFt = "rbxassetid://271042556",
		SkyboxLf = "rbxassetid://271042310",
		SkyboxRt = "rbxassetid://271042467",
		SkyboxUp = "rbxassetid://271077958",
		StarCount = 0,
		SunAngularSize = 21,
		SunTextureId = "rbxasset://sky/sun.jpg"
	},

	Orange = {
		CelestialBodiesShown = true,
		MoonAngularSize = 11,
		MoonTextureId = "rbxasset://sky/moon.jpg",
		SkyboxBk = "rbxassetid://150939022",
		SkyboxDn = "rbxassetid://150939038",
		SkyboxFt = "rbxassetid://150939047",
		SkyboxLf = "rbxassetid://150939056",
		SkyboxRt = "rbxassetid://150939063",
		SkyboxUp = "rbxassetid://150939082",
		StarCount = 3000,
		SunAngularSize = 21,
		SunTextureId = "rbxasset://sky/sun.jpg"
	},

	Purple = {
		CelestialBodiesShown = true,
		MoonAngularSize = 11,
		MoonTextureId = "rbxasset://sky/moon.jpg",
		SkyboxBk = "rbxassetid://75339068650022",
		SkyboxDn = "rbxassetid://103288557260150",
		SkyboxFt = "rbxassetid://131140417030026",
		SkyboxLf = "rbxassetid://91178363674210",
		SkyboxRt = "rbxassetid://91811655162047",
		SkyboxUp = "rbxassetid://70646477316976",
		StarCount = 3000,
		SunAngularSize = 14,
		SunTextureId = "rbxassetid://13683514089"
	},

	Red = {
		CelestialBodiesShown = false,
		MoonAngularSize = 11,
		MoonTextureId = "rbxasset://sky/moon.jpg",
		SkyboxBk = "rbxassetid://108929045660200",
		SkyboxDn = "rbxassetid://78646480540009",
		SkyboxFt = "rbxassetid://90546017435179",
		SkyboxLf = "rbxassetid://109838453114563",
		SkyboxRt = "rbxassetid://94190734796082",
		SkyboxUp = "rbxassetid://126944775797063",
		StarCount = 3000,
		SunAngularSize = 21,
		SunTextureId = "rbxasset://sky/sun.jpg"
	}
}
	local s = Lighting:FindFirstChildOfClass("Sky")
	if s then
		self.ORIGINAL_SKY = s:Clone()
	end

	return self
end

function World:applyLighting()
	if self.LIGHTING_EDIT_ENABLED then
		Lighting.Brightness = self.CUSTOM_BRIGHTNESS
		Lighting.Ambient = self.CUSTOM_AMBIENT
		Lighting.OutdoorAmbient = self.CUSTOM_OUTDOOR_AMBIENT
	else
		Lighting.Brightness = self.ORIGINAL_LIGHTING.Brightness
		Lighting.Ambient = self.ORIGINAL_LIGHTING.Ambient
		Lighting.OutdoorAmbient = self.ORIGINAL_LIGHTING.OutdoorAmbient
	end
end

function World:applyTime()
	if self.WORLD_TIME_ENABLED then
		local preset = self.TIME_PRESETS[self.TIME_PRESET]
		if preset ~= nil then
			Lighting.ClockTime = preset
		else
			Lighting.ClockTime = self.CUSTOM_CLOCKTIME
		end
	else
		Lighting.ClockTime = self.ORIGINAL_LIGHTING.ClockTime
	end
end

function World:wipeSky()
	for _, v in ipairs(Lighting:GetChildren()) do
		if v:IsA("Sky") then
			v:Destroy()
		end
	end
end

function World:buildSky(name)
	local data = self.SKY_PRESETS[name]
	if not data then return end

	local sky = Instance.new("Sky")
	for k, v in pairs(data) do
		sky[k] = v
	end
	sky.Name = "NemesisSky"
	sky.Parent = Lighting
end

function World:applySky()
	self:wipeSky()
	if self.CUSTOM_SKY_ENABLED then
		self:buildSky(self.CUSTOM_SKY_PRESET)
	else
		if self.ORIGINAL_SKY then
			self.ORIGINAL_SKY:Clone().Parent = Lighting
		end
	end
end

function World:SetLightingEnabled(v)
	self.LIGHTING_EDIT_ENABLED = v
	self:applyLighting()
end

function World:SetBrightness(v)
	self.CUSTOM_BRIGHTNESS = v
	self:applyLighting()
end

function World:SetAmbient(v)
	self.CUSTOM_AMBIENT = v
	self:applyLighting()
end

function World:SetOutdoorAmbient(v)
	self.CUSTOM_OUTDOOR_AMBIENT = v
	self:applyLighting()
end

function World:SetTimeEnabled(v)
	self.WORLD_TIME_ENABLED = v
	self:applyTime()
end

function World:SetClockTime(v)
	self.CUSTOM_CLOCKTIME = v
	self:applyTime()
end

function World:SetTimePreset(v)
	self.TIME_PRESET = v
	self:applyTime()
end

function World:SetSkyEnabled(v)
	self.CUSTOM_SKY_ENABLED = v
	self:applySky()
end

function World:SetSkyPreset(v)
	self.CUSTOM_SKY_PRESET = v
	self:applySky()
end

function World:Export()
	return {
		LIGHTING_EDIT_ENABLED = self.LIGHTING_EDIT_ENABLED,
		CUSTOM_BRIGHTNESS = self.CUSTOM_BRIGHTNESS,
		CUSTOM_AMBIENT = {self.CUSTOM_AMBIENT.R * 255, self.CUSTOM_AMBIENT.G * 255, self.CUSTOM_AMBIENT.B * 255},
		CUSTOM_OUTDOOR_AMBIENT = {self.CUSTOM_OUTDOOR_AMBIENT.R * 255, self.CUSTOM_OUTDOOR_AMBIENT.G * 255, self.CUSTOM_OUTDOOR_AMBIENT.B * 255},
		WORLD_TIME_ENABLED = self.WORLD_TIME_ENABLED,
		CUSTOM_CLOCKTIME = self.CUSTOM_CLOCKTIME,
		TIME_PRESET = self.TIME_PRESET,
		CUSTOM_SKY_ENABLED = self.CUSTOM_SKY_ENABLED,
		CUSTOM_SKY_PRESET = self.CUSTOM_SKY_PRESET
	}
end

function World:Import(t)
	local function c(a)
		return Color3.fromRGB(a[1], a[2], a[3])
	end

	if t.LIGHTING_EDIT_ENABLED ~= nil then self.LIGHTING_EDIT_ENABLED = t.LIGHTING_EDIT_ENABLED end
	if t.CUSTOM_BRIGHTNESS then self.CUSTOM_BRIGHTNESS = t.CUSTOM_BRIGHTNESS end
	if t.CUSTOM_AMBIENT then self.CUSTOM_AMBIENT = c(t.CUSTOM_AMBIENT) end
	if t.CUSTOM_OUTDOOR_AMBIENT then self.CUSTOM_OUTDOOR_AMBIENT = c(t.CUSTOM_OUTDOOR_AMBIENT) end
	if t.WORLD_TIME_ENABLED ~= nil then self.WORLD_TIME_ENABLED = t.WORLD_TIME_ENABLED end
	if t.CUSTOM_CLOCKTIME then self.CUSTOM_CLOCKTIME = t.CUSTOM_CLOCKTIME end
	if t.TIME_PRESET then self.TIME_PRESET = t.TIME_PRESET end
	if t.CUSTOM_SKY_ENABLED ~= nil then self.CUSTOM_SKY_ENABLED = t.CUSTOM_SKY_ENABLED end
	if t.CUSTOM_SKY_PRESET then self.CUSTOM_SKY_PRESET = t.CUSTOM_SKY_PRESET end

	self:applyLighting()
	self:applyTime()
	self:applySky()
end

return World
