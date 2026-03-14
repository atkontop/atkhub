local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local localPlayer = Players.LocalPlayer
local ATK_USERNAME = "atk_matt65"

local nametags = {}

-- Create nametag above a character
local function createNametagForCharacter(character, displayText)
	if not character then return end
	local head = character:FindFirstChild("Head")
	if not head then return end
	if head:FindFirstChild("CyberpunkNametag") then return end

	local billboard = Instance.new("BillboardGui")
	billboard.Name = "CyberpunkNametag"
	billboard.Adornee = head
	billboard.Size = UDim2.new(0,200,0,50)
	billboard.StudsOffset = Vector3.new(0,2,0)
	billboard.AlwaysOnTop = true
	billboard.Parent = head

	local bg = Instance.new("Frame")
	bg.Size = UDim2.new(0.75,0,1,0)
	bg.Position = UDim2.new(0,0,0,0)
	bg.BackgroundColor3 = Color3.fromRGB(255,0,0)
	bg.BorderSizePixel = 0
	bg.Parent = billboard

	local uicorner = Instance.new("UICorner")
	uicorner.CornerRadius = UDim.new(0,10)
	uicorner.Parent = bg

	local glow = Instance.new("UIStroke")
	glow.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
	glow.LineJoinMode = Enum.LineJoinMode.Round
	glow.Thickness = 2
	glow.Color = Color3.fromRGB(255,255,255)
	glow.Parent = bg

	local scanline = Instance.new("Frame")
	scanline.Size = UDim2.new(1,0,0,2)
	scanline.BackgroundColor3 = Color3.fromRGB(255,255,255)
	scanline.BackgroundTransparency = 0.5
	scanline.BorderSizePixel = 0
	scanline.Parent = bg

	-- Letters
	local letters = {}
	for i = 1, #displayText do
		local l = Instance.new("TextLabel")
		-- Make letters smaller and narrower for SS USER
		local letterScale = displayText == "ATK" and 1/#displayText or 0.8/#displayText
		l.Size = UDim2.new(letterScale,0,1,0)
		l.Position = UDim2.new((i-1) * letterScale,0,0,0)
		l.BackgroundTransparency = 1
		l.Text = displayText:sub(i,i)
		l.Font = Enum.Font.GothamBold
		l.TextSize = displayText == "ATK" and 22 or 16  -- smaller for SS USER
		l.TextColor3 = Color3.fromRGB(255,255,255)
		l.TextStrokeTransparency = 0.5
		l.Parent = bg
		table.insert(letters,l)
	end

	-- Skull icon
	local skullFrame = Instance.new("Frame")
	skullFrame.Size = UDim2.new(0,40,0,40)
	-- shift skull slightly for SS USER so it doesn't overlap
	skullFrame.Position = displayText == "ATK" and UDim2.new(0.77,0,0.5,-20) or UDim2.new(0.8,0,0.5,-20)
	skullFrame.BackgroundColor3 = Color3.fromRGB(20,20,20)
	skullFrame.BackgroundTransparency = 0.25
	skullFrame.BorderSizePixel = 0
	skullFrame.Parent = billboard

	local skullCorner = Instance.new("UICorner")
	skullCorner.CornerRadius = UDim.new(1,0)
	skullCorner.Parent = skullFrame

	local skullGlow = Instance.new("UIStroke")
	skullGlow.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
	skullGlow.LineJoinMode = Enum.LineJoinMode.Round
	skullGlow.Thickness = 2
	skullGlow.Color = Color3.fromRGB(255,0,0)
	skullGlow.Parent = skullFrame

	local skull = Instance.new("ImageLabel")
	skull.Size = UDim2.new(0,28,0,28)
	skull.Position = UDim2.new(0.5,-14,0.5,-14)
	skull.BackgroundTransparency = 1
	skull.Image = "rbxassetid://6031090992"
	skull.Parent = skullFrame

	-- Animation variables
	local glitchTimer = 0
	local glitchOffset = 2
	local pulseDirection = 1
	local pulseValue = 0
	local scanOffset = 0
	local fadeAlpha = 1
	local fadeDirection = -1

	RunService.RenderStepped:Connect(function(dt)
		local cam = Workspace.CurrentCamera
		local distance = (cam.CFrame.Position - head.Position).Magnitude
		local scale = math.clamp(1 / (distance/20), 0.5, 1) -- smaller when far
		billboard.Size = UDim2.new(0, 200*scale, 0, 50*scale)

		if displayText == "ATK" then
			-- Background pulse
			local t = tick()
			bg.BackgroundColor3 = Color3.fromRGB(255, 255 * (0.5 + 0.5 * math.sin(t*2)), 255 * (0.5 + 0.5 * math.cos(t*2)))

			-- Glitch letters
			glitchTimer = glitchTimer + dt
			if glitchTimer > 0.05 then
				glitchTimer = 0
				for _,l in pairs(letters) do
					local offsetX = math.random(-glitchOffset,glitchOffset)
					local offsetY = math.random(-glitchOffset,glitchOffset)
					l.Position = UDim2.new(l.Position.X.Scale, offsetX, l.Position.Y.Scale, offsetY)
					l.TextColor3 = Color3.fromRGB(255, math.random(180,255), math.random(180,255))
				end
			end
		else
			-- "SS USER" fade in/out
			fadeAlpha = fadeAlpha + fadeDirection * dt
			if fadeAlpha > 1 then fadeAlpha = 1; fadeDirection = -1 end
			if fadeAlpha < 0 then fadeAlpha = 0; fadeDirection = 1 end
			for _,l in pairs(letters) do
				l.TextTransparency = 1 - fadeAlpha
			end
			-- Keep background solid red/white pulse
			local t = tick()
			bg.BackgroundColor3 = Color3.fromRGB(255, 255 * (0.5 + 0.5 * math.sin(t*1)), 255 * (0.5 + 0.5 * math.cos(t*1)))
		end

		-- Skull glow pulse
		pulseValue = pulseValue + dt * pulseDirection * 2
		if pulseValue > 1 then pulseValue = 1; pulseDirection = -1 end
		if pulseValue < 0 then pulseValue = 0; pulseDirection = 1 end
		glow.Color = Color3.fromRGB(255, 255 * pulseValue, 255 * pulseValue)
		skullGlow.Color = Color3.fromRGB(255, 0, 0 + 155 * pulseValue)

		-- Scanline animation
		scanOffset = scanOffset + dt * 80
		if scanOffset > bg.AbsoluteSize.Y then scanOffset = 0 end
		scanline.Position = UDim2.new(0,0,0,scanOffset)
	end)

	return billboard
end

-- Create nametag for a player
local function createNametagForPlayer(player)
	local displayText = player.Name == ATK_USERNAME and "ATK" or "SS USER"
	if player.Character then
		nametags[player] = createNametagForCharacter(player.Character, displayText)
	end
	player.CharacterAdded:Connect(function(char)
		if nametags[player] then
			nametags[player]:Destroy()
			nametags[player] = nil
		end
		nametags[player] = createNametagForCharacter(char, displayText)
	end)
end

-- Clean up when player leaves
Players.PlayerRemoving:Connect(function(player)
	if nametags[player] then
		nametags[player]:Destroy()
		nametags[player] = nil
	end
end)

-- Initial creation
for _,player in pairs(Players:GetPlayers()) do
	createNametagForPlayer(player)
end

-- New player joins
Players.PlayerAdded:Connect(function(player)
	createNametagForPlayer(player)
end)
