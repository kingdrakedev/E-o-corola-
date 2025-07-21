local ScreenGui = Instance.new("ScreenGui")
local Toggle = Instance.new("TextButton")

ScreenGui.Name = "ESP_GUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game.CoreGui

Toggle.Name = "ToggleESP"
Toggle.Parent = ScreenGui
Toggle.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Toggle.Position = UDim2.new(0, 20, 0, 200)
Toggle.Size = UDim2.new(0, 120, 0, 40)
Toggle.Text = "ESP: OFF"
Toggle.TextColor3 = Color3.fromRGB(255, 255, 255)
Toggle.TextScaled = true

-- ESP Main Code
local espEnabled = false
local players = game:GetService("Players")
local localPlayer = players.LocalPlayer
local runService = game:GetService("RunService")

local function createESPBox(player)
	if player == localPlayer then return end
	if player.Team == localPlayer.Team then return end -- Team Check

	local box = Drawing.new("Text")
	box.Visible = false
	box.Center = true
	box.Outline = true
	box.Font = 2
	box.Size = 13
	box.Color = Color3.fromRGB(255, 0, 0)
	box.Text = player.Name

	return box
end

local espBoxes = {}

local function updateESP()
	for _, player in pairs(players:GetPlayers()) do
		if player ~= localPlayer and not espBoxes[player] then
			espBoxes[player] = createESPBox(player)
		end
	end

	for player, box in pairs(espBoxes) do
		if player.Character and player.Character:FindFirstChild("HumanoidRootPart") and player.Team ~= localPlayer.Team then
			local pos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(player.Character.HumanoidRootPart.Position)
			if onScreen and espEnabled then
				box.Position = Vector2.new(pos.X, pos.Y - 30)
				box.Visible = true
			else
				box.Visible = false
			end
		else
			box.Visible = false
		end
	end
end

runService.RenderStepped:Connect(function()
	if espEnabled then
		updateESP()
	end
end)

-- Toggle ESP
Toggle.MouseButton1Click:Connect(function()
	espEnabled = not espEnabled
	Toggle.Text = espEnabled and "ESP: ON" or "ESP: OFF"

	if not espEnabled then
		for _, box in pairs(espBoxes) do
			box.Visible = false
		end
	end
end)

-- Limpeza ao sair
players.PlayerRemoving:Connect(function(player)
	if espBoxes[player] then
		espBoxes[player]:Remove()
		espBoxes[player] = nil
	end
end)
