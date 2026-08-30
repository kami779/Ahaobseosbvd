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
