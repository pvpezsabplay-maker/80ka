-- Aim Assist for players (PvP)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

local LocalPlayer = Players.LocalPlayer

-- SETTINGS
local AIM_ASSIST_ENABLED = true
local AIM_RADIUS = 120 -- pixels from crosshair
local AIM_STRENGTH = 0.15 -- 0 = off, 1 = instant snap
local MAX_DISTANCE = 300 -- studs
local TARGET_PART = "Head" -- can be HumanoidRootPart

-- Toggle with Q key
UIS.InputBegan:Connect(function(input, gp)
	if gp then return end
	if input.KeyCode == Enum.KeyCode.Q then
		AIM_ASSIST_ENABLED = not AIM_ASSIST_ENABLED
	end
end)

-- Get closest player to crosshair
local function getClosestPlayer()
	local closest = nil
	local shortest = math.huge

	for _, player in pairs(Players:GetPlayers()) do
		if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild(TARGET_PART) then
			local part = player.Character[TARGET_PART]
			local hum = player.Character:FindFirstChild("Humanoid")
			if hum and hum.Health > 0 then
				local pos, onScreen = Camera:WorldToViewportPoint(part.Position)
				if onScreen then
					local mousePos = UIS:GetMouseLocation()
					local dist2D = (Vector2.new(pos.X, pos.Y) - mousePos).Magnitude
					local dist3D = (Camera.CFrame.Position - part.Position).Magnitude
					if dist2D < AIM_RADIUS and dist3D < MAX_DISTANCE and dist2D < shortest then
						shortest = dist2D
						closest = part
					end
				end
			end
		end
	end

	return closest
end

-- Aim assist loop
RunService.RenderStepped:Connect(function()
	if not AIM_ASSIST_ENABLED then return end

	local target = getClosestPlayer()
	if target then
		local camPos = Camera.CFrame.Position
		local dir = (target.Position - camPos).Unit
		local targetCF = CFrame.new(camPos, camPos + dir)
		Camera.CFrame = Camera.CFrame:Lerp(targetCF, AIM_STRENGTH)
	end
end)
