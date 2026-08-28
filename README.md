local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local hitboxSize = 10
local speed = 16
local flySpeed = 50
local aimAssist = false
local flying = false

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "TestControls"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 260, 0, 300)
frame.Position = UDim2.new(0, 20, 0.5, -150)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = frame

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 8)
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Parent = frame

local padding = Instance.new("UIPadding")
padding.PaddingTop = UDim.new(0, 12)
padding.PaddingLeft = UDim.new(0, 10)
padding.PaddingRight = UDim.new(0, 10)
padding.Parent = frame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "TEST CONTROLS"
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.Parent = frame

local function createSlider(name, min, max, default, callback)
	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1, 0, 0, 25)
	label.Text = name .. ": " .. default
	label.TextColor3 = Color3.new(1, 1, 1)
	label.BackgroundTransparency = 1
	label.TextSize = 14
	label.Font = Enum.Font.Gotham
	label.Parent = frame

	local box = Instance.new("TextBox")
	box.Size = UDim2.new(1, 0, 0, 30)
	box.PlaceholderText = min .. " - " .. max
	box.Text = tostring(default)
	box.TextColor3 = Color3.new(1, 1, 1)
	box.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
	box.ClearTextOnFocus = false
	box.Parent = frame

	local boxCorner = Instance.new("UICorner")
	boxCorner.CornerRadius = UDim.new(0, 6)
	boxCorner.Parent = box

	box.FocusLost:Connect(function()
		local value = tonumber(box.Text)

		if value then
			value = math.clamp(math.floor(value), min, max)
			box.Text = tostring(value)
			label.Text = name .. ": " .. value
			callback(value)
		else
			box.Text = tostring(default)
		end
	end)
end

createSlider("Hitbox", 1, 1000, hitboxSize, function(value)
	hitboxSize = value
end)

createSlider("Speed", 1, 800, speed, function(value)
	speed = value

	local character = player.Character
	local humanoid = character and character:FindFirstChildOfClass("Humanoid")

	if humanoid then
		humanoid.WalkSpeed = value
	end
end)

createSlider("Fly Speed", 1, 200, flySpeed, function(value)
	flySpeed = value
end)

-- AIM ASSIST
local aimButton = Instance.new("TextButton")
aimButton.Size = UDim2.new(1, 0, 0, 35)
aimButton.Text = "Aim Assist: OFF"
aimButton.TextColor3 = Color3.new(1, 1, 1)
aimButton.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
aimButton.Font = Enum.Font.GothamBold
aimButton.TextSize = 14
aimButton.Parent = frame

Instance.new("UICorner", aimButton).CornerRadius = UDim.new(0, 6)

aimButton.MouseButton1Click:Connect(function()
	aimAssist = not aimAssist
	aimButton.Text = "Aim Assist: " .. (aimAssist and "ON" or "OFF")
end)

-- FLY
local flyButton = Instance.new("TextButton")
flyButton.Size = UDim2.new(1, 0, 0, 35)
flyButton.Text = "Fly: OFF"
flyButton.TextColor3 = Color3.new(1, 1, 1)
flyButton.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
flyButton.Font = Enum.Font.GothamBold
flyButton.TextSize = 14
flyButton.Parent = frame

Instance.new("UICorner", flyButton).CornerRadius = UDim.new(0, 6)

flyButton.MouseButton1Click:Connect(function()
	flying = not flying
	flyButton.Text = "Fly: " .. (flying and "ON" or "OFF")
end)

-- Atualiza velocidade
RunService.RenderStepped:Connect(function()
	local character = player.Character
	if not character then return end

	local humanoid = character:FindFirstChildOfClass("Humanoid")
	local root = character:FindFirstChild("HumanoidRootPart")

	if humanoid then
		humanoid.WalkSpeed = speed
	end

	-- Fly
	if flying and root then
		local direction = Vector3.zero

		if UserInputService:IsKeyDown(Enum.KeyCode.W) then
			direction += camera.CFrame.LookVector
		end

		if UserInputService:IsKeyDown(Enum.KeyCode.S) then
			direction -= camera.CFrame.LookVector
		end

		if UserInputService:IsKeyDown(Enum.KeyCode.A) then
			direction -= camera.CFrame.RightVector
		end

		if UserInputService:IsKeyDown(Enum.KeyCode.D) then
			direction += camera.CFrame.RightVector
		end

		if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
			direction += Vector3.new(0, 1, 0)
		end

		if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
			direction -= Vector3.new(0, 1, 0)
		end

		if direction.Magnitude > 0 then
			root.AssemblyLinearVelocity = direction.Unit * flySpeed
		else
			root.AssemblyLinearVelocity = Vector3.zero
		end
	end
end)

-- AIM ASSIST
RunService.RenderStepped:Connect(function()
	if not aimAssist then return end

	local closest
	local closestDistance = math.huge

	for _, target in ipairs(Players:GetPlayers()) do
		if target ~= player and target.Character then
			local head = target.Character:FindFirstChild("Head")
			local humanoid = target.Character:FindFirstChildOfClass("Humanoid")

			if head and humanoid and humanoid.Health > 0 then
				local screenPosition, visible =
					camera:WorldToViewportPoint(head.Position)

				if visible then
					local center =
						Vector2.new(camera.ViewportSize.X / 2,
							camera.ViewportSize.Y / 2)

					local distance =
						(Vector2.new(screenPosition.X, screenPosition.Y) - center).Magnitude

					if distance < closestDistance then
						closestDistance = distance
						closest = head
					end
				end
			end
		end
	end

	if closest then
		camera.CFrame = CFrame.lookAt(
			camera.CFrame.Position,
			closest.Position
		)
	end
end)-- SISTEMA DE TESTE - ROBLOX STUDIO
-- Hitbox: 1-1000
-- Speed: 1-800
-- Fly: 1-200
-- Aim Assist: ON/OFF

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local camera = workspace.CurrentCamera

local hitboxSize = 10
local speed = 16
local flySpeed = 50
local aimAssist = false
local flying = false

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "TestControls"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 260, 0, 300)
frame.Position = UDim2.new(0, 20, 0.5, -150)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.BorderSizePixel = 0
frame.Parent = gui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 10)
corner.Parent = frame

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0, 8)
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Parent = frame

local padding = Instance.new("UIPadding")
padding.PaddingTop = UDim.new(0, 12)
padding.PaddingLeft = UDim.new(0, 10)
padding.PaddingRight = UDim.new(0, 10)
padding.Parent = frame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "TEST CONTROLS"
title.TextColor3 = Color3.new(1, 1, 1)
title.BackgroundTransparency = 1
title.TextSize = 20
title.Font = Enum.Font.GothamBold
title.Parent = frame

local function createSlider(name, min, max, default, callback)
	local label = Instance.new("TextLabel")
	label.Size = UDim2.new(1, 0, 0, 25)
	label.Text = name .. ": " .. default
	label.TextColor3 = Color3.new(1, 1, 1)
	label.BackgroundTransparency = 1
	label.TextSize = 14
	label.Font = Enum.Font.Gotham
	label.Parent = frame

	local box = Instance.new("TextBox")
	box.Size = UDim2.new(1, 0, 0, 30)
	box.PlaceholderText = min .. " - " .. max
	box.Text = tostring(default)
	box.TextColor3 = Color3.new(1, 1, 1)
	box.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
	box.ClearTextOnFocus = false
	box.Parent = frame

	local boxCorner = Instance.new("UICorner")
	boxCorner.CornerRadius = UDim.new(0, 6)
	boxCorner.Parent = box

	box.FocusLost:Connect(function()
		local value = tonumber(box.Text)

		if value then
			value = math.clamp(math.floor(value), min, max)
			box.Text = tostring(value)
			label.Text = name .. ": " .. value
			callback(value)
		else
			box.Text = tostring(default)
		end
	end)
end

createSlider("Hitbox", 1, 1000, hitboxSize, function(value)
	hitboxSize = value
end)

createSlider("Speed", 1, 800, speed, function(value)
	speed = value

	local character = player.Character
	local humanoid = character and character:FindFirstChildOfClass("Humanoid")

	if humanoid then
		humanoid.WalkSpeed = value
	end
end)

createSlider("Fly Speed", 1, 200, flySpeed, function(value)
	flySpeed = value
end)

-- AIM ASSIST
local aimButton = Instance.new("TextButton")
aimButton.Size = UDim2.new(1, 0, 0, 35)
aimButton.Text = "Aim Assist: OFF"
aimButton.TextColor3 = Color3.new(1, 1, 1)
aimButton.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
aimButton.Font = Enum.Font.GothamBold
aimButton.TextSize = 14
aimButton.Parent = frame

Instance.new("UICorner", aimButton).CornerRadius = UDim.new(0, 6)

aimButton.MouseButton1Click:Connect(function()
	aimAssist = not aimAssist
	aimButton.Text = "Aim Assist: " .. (aimAssist and "ON" or "OFF")
end)

-- FLY
local flyButton = Instance.new("TextButton")
flyButton.Size = UDim2.new(1, 0, 0, 35)
flyButton.Text = "Fly: OFF"
flyButton.TextColor3 = Color3.new(1, 1, 1)
flyButton.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
flyButton.Font = Enum.Font.GothamBold
flyButton.TextSize = 14
flyButton.Parent = frame

Instance.new("UICorner", flyButton).CornerRadius = UDim.new(0, 6)

flyButton.MouseButton1Click:Connect(function()
	flying = not flying
	flyButton.Text = "Fly: " .. (flying and "ON" or "OFF")
end)

-- Atualiza velocidade
RunService.RenderStepped:Connect(function()
	local character = player.Character
	if not character then return end

	local humanoid = character:FindFirstChildOfClass("Humanoid")
	local root = character:FindFirstChild("HumanoidRootPart")

	if humanoid then
		humanoid.WalkSpeed = speed
	end

	-- Fly
	if flying and root then
		local direction = Vector3.zero

		if UserInputService:IsKeyDown(Enum.KeyCode.W) then
			direction += camera.CFrame.LookVector
		end

		if UserInputService:IsKeyDown(Enum.KeyCode.S) then
			direction -= camera.CFrame.LookVector
		end

		if UserInputService:IsKeyDown(Enum.KeyCode.A) then
			direction -= camera.CFrame.RightVector
		end

		if UserInputService:IsKeyDown(Enum.KeyCode.D) then
			direction += camera.CFrame.RightVector
		end

		if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
			direction += Vector3.new(0, 1, 0)
		end

		if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
			direction -= Vector3.new(0, 1, 0)
		end

		if direction.Magnitude > 0 then
			root.AssemblyLinearVelocity = direction.Unit * flySpeed
		else
			root.AssemblyLinearVelocity = Vector3.zero
		end
	end
end)

-- AIM ASSIST
RunService.RenderStepped:Connect(function()
	if not aimAssist then return end

	local closest
	local closestDistance = math.huge

	for _, target in ipairs(Players:GetPlayers()) do
		if target ~= player and target.Character then
			local head = target.Character:FindFirstChild("Head")
			local humanoid = target.Character:FindFirstChildOfClass("Humanoid")

			if head and humanoid and humanoid.Health > 0 then
				local screenPosition, visible =
					camera:WorldToViewportPoint(head.Position)

				if visible then
					local center =
						Vector2.new(camera.ViewportSize.X / 2,
							camera.ViewportSize.Y / 2)

					local distance =
						(Vector2.new(screenPosition.X, screenPosition.Y) - center).Magnitude

					if distance < closestDistance then
						closestDistance = distance
						closest = head
					end
				end
			end
		end
	end

	if closest then
		camera.CFrame = CFrame.lookAt(
			camera.CFrame.Position,
			closest.Position
		)
	end
end)
