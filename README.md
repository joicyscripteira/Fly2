-- Fly Script com botão para celular - por @joicyscripteira (modificado para voar a 60 metros)

local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

local flying = false
local speed = 50
local fixedHeight = 60
local control = {F = 0, B = 0, L = 0, R = 0}
local flyConnection = nil

-- Função para voar a 60m
local function fly()
	if flying then return end
	flying = true

	local bodyGyro = Instance.new("BodyGyro")
	bodyGyro.P = 9e4
	bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
	bodyGyro.CFrame = humanoidRootPart.CFrame
	bodyGyro.Parent = humanoidRootPart

	local bodyPosition = Instance.new("BodyPosition")
	bodyPosition.MaxForce = Vector3.new(9e9, 9e9, 9e9)
	bodyPosition.D = 1000
	bodyPosition.P = 10000
	bodyPosition.Position = humanoidRootPart.Position
	bodyPosition.Parent = humanoidRootPart

	flyConnection = RunService.RenderStepped:Connect(function()
		if not flying then return end

		-- Detectar o chão abaixo
		local rayOrigin = humanoidRootPart.Position
		local rayDirection = Vector3.new(0, -200, 0)
		local raycastParams = RaycastParams.new()
		raycastParams.FilterDescendantsInstances = {character}
		raycastParams.FilterType = Enum.RaycastFilterType.Blacklist

		local raycastResult = Workspace:Raycast(rayOrigin, rayDirection, raycastParams)

		local groundY = raycastResult and raycastResult.Position.Y or 0
		local desiredY = groundY + fixedHeight

		local move = Vector3.new(control.L + control.R, 0, control.F + control.B)
		local camCFrame = Workspace.CurrentCamera.CFrame
		local direction = move.Magnitude > 0 and camCFrame:VectorToWorldSpace(move.Unit) * speed or Vector3.zero

		bodyPosition.Position = Vector3.new(
			humanoidRootPart.Position.X + direction.X,
			desiredY,
			humanoidRootPart.Position.Z + direction.Z
		)

		bodyGyro.CFrame = camCFrame
	end)

	player.CharacterRemoving:Connect(function()
		flying = false
		bodyGyro:Destroy()
		bodyPosition:Destroy()
	end)
end

-- Parar voo
local function stopFlying()
	flying = false
	if flyConnection then flyConnection:Disconnect() end
	if humanoidRootPart:FindFirstChild("BodyGyro") then
		humanoidRootPart.BodyGyro:Destroy()
	end
	if humanoidRootPart:FindFirstChild("BodyPosition") then
		humanoidRootPart.BodyPosition:Destroy()
	end
end

-- Interface botão lateral direita
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FlyUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 100, 0, 40)
toggleButton.Position = UDim2.new(1, -110, 0.85, 0)
toggleButton.AnchorPoint = Vector2.new(0, 0)
toggleButton.BackgroundColor3 = Color3.fromRGB(200, 30, 30)
toggleButton.Text = "Fly: OFF"
toggleButton.TextScaled = true
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Parent = screenGui

toggleButton.MouseButton1Click:Connect(function()
	if flying then
		stopFlying()
		toggleButton.Text = "Fly: OFF"
		toggleButton.BackgroundColor3 = Color3.fromRGB(200, 30, 30)
	else
		fly()
		toggleButton.Text = "Fly: ON"
		toggleButton.BackgroundColor3 = Color3.fromRGB(30, 200, 30)
	end
end)

-- Controles WASD
UIS.InputBegan:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.W then control.F = -1 end
	if input.KeyCode == Enum.KeyCode.S then control.B = 1 end
	if input.KeyCode == Enum.KeyCode.A then control.L = -1 end
	if input.KeyCode == Enum.KeyCode.D then control.R = 1 end
end)

UIS.InputEnded:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.W then control.F = 0 end
	if input.KeyCode == Enum.KeyCode.S then control.B = 0 end
	if input.KeyCode == Enum.KeyCode.A then control.L = 0 end
	if input.KeyCode == Enum.KeyCode.D then control.R = 0 end
end)
