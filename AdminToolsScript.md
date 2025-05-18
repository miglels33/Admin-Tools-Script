local player = game.Players.LocalPlayer
local backpack = player:WaitForChild("Backpack")
local playerGui = player:WaitForChild("PlayerGui")

-- Selected color
local selectedColor = Color3.fromRGB(255, 255, 255)

-- Create color selection GUI
local function createColorGUI()
	local gui = Instance.new("ScreenGui")
	gui.Name = "ColorPickerGUI"
	gui.ResetOnSpawn = false
	gui.Parent = playerGui

	local frame = Instance.new("Frame", gui)
	frame.Size = UDim2.new(0, 220, 0, 200)
	frame.Position = UDim2.new(0, 20, 0, 100)
	frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
	frame.BorderSizePixel = 0
	frame.AnchorPoint = Vector2.new(0, 0)
	frame.BackgroundTransparency = 0.1

	local uiCorner = Instance.new("UICorner", frame)
	uiCorner.CornerRadius = UDim.new(0, 10)

	local title = Instance.new("TextLabel", frame)
	title.Size = UDim2.new(1, 0, 0, 30)
	title.Text = "Pick a Color"
	title.TextColor3 = Color3.fromRGB(255, 255, 255)
	title.BackgroundTransparency = 1
	title.Font = Enum.Font.SourceSansBold
	title.TextSize = 20

	local function makeTextBox(name, posY)
		local box = Instance.new("TextBox")
		box.Name = name
		box.PlaceholderText = name .. " (0-255)"
		box.Position = UDim2.new(0, 20, 0, posY)
		box.Size = UDim2.new(0, 180, 0, 30)
		box.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
		box.TextColor3 = Color3.new(1, 1, 1)
		box.Font = Enum.Font.SourceSans
		box.TextSize = 18

		local corner = Instance.new("UICorner", box)
		corner.CornerRadius = UDim.new(0, 6)

		box.Parent = frame
		return box
	end

	local rBox = makeTextBox("R", 40)
	local gBox = makeTextBox("G", 80)
	local bBox = makeTextBox("B", 120)

	local applyBtn = Instance.new("TextButton", frame)
	applyBtn.Text = "Apply Color"
	applyBtn.Position = UDim2.new(0, 20, 0, 160)
	applyBtn.Size = UDim2.new(0, 180, 0, 30)
	applyBtn.BackgroundColor3 = Color3.fromRGB(0, 170, 255)
	applyBtn.TextColor3 = Color3.new(1, 1, 1)
	applyBtn.Font = Enum.Font.SourceSansBold
	applyBtn.TextSize = 18

	local corner = Instance.new("UICorner", applyBtn)
	corner.CornerRadius = UDim.new(0, 6)

	applyBtn.MouseButton1Click:Connect(function()
		local r = tonumber(rBox.Text) or 255
		local g = tonumber(gBox.Text) or 255
		local b = tonumber(bBox.Text) or 255
		selectedColor = Color3.fromRGB(r, g, b)
	end)

	return gui
end

-- Create a Tool
local function createTool(name, behavior)
	local tool = Instance.new("Tool")
	tool.Name = name
	tool.RequiresHandle = false

	local gui

	tool.Equipped:Connect(function(mouse)
		if behavior == "color" then
			gui = createColorGUI()
		end

		tool.Activated:Connect(function()
			local connection
			connection = mouse.Button1Down:Connect(function()
				local target = mouse.Target
				if target and target:IsDescendantOf(workspace) and not target:IsDescendantOf(player.Character) then
					
					if behavior == "destroy" then
						local explosion = Instance.new("Explosion")
						explosion.Position = target.Position
						explosion.BlastRadius = 5
						explosion.BlastPressure = 50000
						explosion.Parent = workspace
						target:Destroy()

					elseif behavior == "copy" then
						local clone = target:Clone()
						clone.Parent = workspace
						if clone:IsA("BasePart") then
							clone.CFrame = target.CFrame + Vector3.new(3, 0, 0)
						elseif clone:FindFirstChildWhichIsA("BasePart") then
							local part = clone:FindFirstChildWhichIsA("BasePart")
							part.CFrame = target.CFrame + Vector3.new(3, 0, 0)
						end

					elseif behavior == "color" then
						if target:IsA("BasePart") then
							target.Color = selectedColor
						end
					end
				end
				if connection then
					connection:Disconnect()
				end
			end)
		end)
	end)

	tool.Unequipped:Connect(function()
		if gui then
			gui:Destroy()
		end
	end)

	tool.Parent = backpack
end

-- Give all Tools to player
createTool("Destroyer", "destroy")
createTool("Copy", "copy")
createTool("ColorTool", "color")
