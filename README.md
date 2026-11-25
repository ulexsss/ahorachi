local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Interfaz principal del menú
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "VisualMenuUI"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 220, 0, 175)
mainFrame.Position = UDim2.new(0.03, 0, 0.38, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(28, 28, 32)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui

local mainCorner = Instance.new("UICorner")
mainCorner.CornerRadius = UDim.new(0, 8)
mainCorner.Parent = mainFrame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 28)
title.BackgroundColor3 = Color3.fromRGB(165, 32, 32)
title.Text = "VISUAL MENU"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.GothamBold
title.TextSize = 13
title.Parent = mainFrame

local titleCorner = Instance.new("UICorner")
titleCorner.CornerRadius = UDim.new(0, 8)
titleCorner.Parent = title

-- Cartel flotante ("Objeto Desconocido") del video
local tooltip = Instance.new("Frame")
tooltip.Name = "ObjetoDesconocidoTooltip"
tooltip.Size = UDim2.new(0, 210, 0, 145)
tooltip.BackgroundColor3 = Color3.fromRGB(20, 20, 24)
tooltip.BackgroundTransparency = 0.08
tooltip.BorderSizePixel = 0
tooltip.Visible = false
tooltip.ZIndex = 50
tooltip.Parent = screenGui

local tipCorner = Instance.new("UICorner")
tipCorner.CornerRadius = UDim.new(0, 8)
tipCorner.Parent = tooltip

local tipStroke = Instance.new("UIStroke")
tipStroke.Color = Color3.fromRGB(255, 255, 255)
tipStroke.Thickness = 1.2
tipStroke.Parent = tooltip

local tipText = Instance.new("TextLabel")
tipText.Size = UDim2.new(1, -16, 1, -12)
tipText.Position = UDim2.new(0, 8, 0, 6)
tipText.BackgroundTransparency = 1
tipText.RichText = true
tipText.TextWrapped = true
tipText.TextYAlignment = Enum.TextYAlignment.Top
tipText.Font = Enum.Font.GothamSemibold
tipText.TextSize = 12
tipText.TextColor3 = Color3.fromRGB(240, 240, 240)
tipText.ZIndex = 51
tipText.Text = [[<b><font size="14">Objeto Desconocido</font></b>
<font color="#ff4d4d"><b>Sin Verificar</b></font>
<b>???</b>
<font size="10" color="#cccccc">La información del objetivo está temporalmente oculta debido a modificaciones visuales activas.</font>

<font size="10" color="#ffaa44">si experimentas problemas durante el intercambio, confirma su placa de usuario</font>]]
tipText.Parent = tooltip

-- Estado y referencias
local hiddenObjects = {}
local hideMode = false
local textMode = false

local function createButton(text, yPos)
	local btn = Instance.new("TextButton")
	btn.Size = UDim2.new(1, -20, 0, 32)
	btn.Position = UDim2.new(0, 10, 0, yPos)
	btn.BackgroundColor3 = Color3.fromRGB(45, 45, 52)
	btn.Text = text
	btn.TextColor3 = Color3.fromRGB(255, 255, 255)
	btn.Font = Enum.Font.GothamBold
	btn.TextSize = 12
	btn.Parent = mainFrame

	local c = Instance.new("UICorner")
	c.CornerRadius = UDim.new(0, 6)
	c.Parent = btn
	return btn
end

local btnX = createButton("desaparecer [X]", 36)
local btnC = createButton("texto [C]", 74)
local btnV = createButton("activado [V]", 112)

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 24, 0, 22)
closeBtn.Position = UDim2.new(0, 10, 0, 148)
closeBtn.BackgroundColor3 = Color3.fromRGB(165, 32, 32)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextSize = 12
closeBtn.Parent = mainFrame

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 5)
closeCorner.Parent = closeBtn

closeBtn.MouseButton1Click:Connect(function()
	mainFrame.Visible = not mainFrame.Visible
end)

-- Oculta o restaura el elemento de UI bajo el mouse (sin tocar el propio menú)
local function toggleHideUnderMouse()
	hideMode = not hideMode
	if hideMode then
		btnX.Text = "aparecer [X]"
		btnX.BackgroundColor3 = Color3.fromRGB(46, 155, 75)
		local mousePos = UserInputService:GetMouseLocation()
		local guis = playerGui:GetGuiObjectsAtPosition(mousePos.X, mousePos.Y)
		for _, obj in ipairs(guis) do
			if not obj:IsDescendantOf(screenGui) and obj.Visible then
				hiddenObjects[obj] = true
				obj.Visible = false
				break
			end
		end
	else
		btnX.Text = "desaparecer [X]"
		btnX.BackgroundColor3 = Color3.fromRGB(45, 45, 52)
		for obj in pairs(hiddenObjects) do
			if obj and obj.Parent then
				obj.Visible = true
			end
		end
		table.clear(hiddenObjects)
	end
end

-- Activa o desactiva el cartel "Objeto Desconocido"
local function toggleTooltip()
	textMode = not textMode
	tooltip.Visible = textMode
	if textMode then
		btnC.Text = "restaurar texto [C]"
		btnC.BackgroundColor3 = Color3.fromRGB(46, 155, 75)
	else
		btnC.Text = "texto [C]"
		btnC.BackgroundColor3 = Color3.fromRGB(45, 45, 52)
	end
end

-- Activa o desactiva ambos modos con V
local function toggleAll()
	local targetState = not (hideMode or textMode)
	if targetState ~= hideMode then
		toggleHideUnderMouse()
	end
	if targetState ~= textMode then
		toggleTooltip()
	end
	btnV.BackgroundColor3 = targetState and Color3.fromRGB(46, 155, 75) or Color3.fromRGB(45, 45, 52)
end

btnX.MouseButton1Click:Connect(toggleHideUnderMouse)
btnC.MouseButton1Click:Connect(toggleTooltip)
btnV.MouseButton1Click:Connect(toggleAll)

UserInputService.InputBegan:Connect(function(input, gameProcessed)
	if gameProcessed then return end
	if input.KeyCode == Enum.KeyCode.X then
		toggleHideUnderMouse()
	elseif input.KeyCode == Enum.KeyCode.C then
		toggleTooltip()
	elseif input.KeyCode == Enum.KeyCode.V then
		toggleAll()
	end
end)

RunService.RenderStepped:Connect(function()
	if tooltip.Visible then
		local pos = UserInputService:GetMouseLocation()
		tooltip.Position = UDim2.fromOffset(pos.X + 14, pos.Y - 10)
	end
end)
