-- FPS Killer v10 (EDUCACIONAL, uso próprio)
-- ÚNICO BOTÃO que: 1) spam ordenado slots 0->9, 2) aplica efeito visual de "spin" (base 10) sem alterar HRP (evita resets)
-- Agora com toggle pelo teclado G
-- Use APENAS em jogos que você desenvolve / controla.

if game:GetService("CoreGui"):FindFirstChild("FPSKillerGUI") then
	game:GetService("CoreGui").FPSKillerGUI:Destroy()
end

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local player = Players.LocalPlayer
if not player then return end

-- CONFIGURAÇÃO (segura)
local DEFAULT_SPAM_DELAY = 0.05  -- segundos entre equips (50ms). Ajuste se for seu próprio jogo.
local SPIN_BASE_SPEED = 10       -- velocidade base fixa (o que você pediu). NÃO EDITAR em runtime.

-- GUI minimalista (um botão)
local screen = Instance.new("ScreenGui")
screen.Name = "FPSKillerGUI"
screen.Parent = game:GetService("CoreGui")

local frame = Instance.new("Frame", screen)
frame.Size = UDim2.new(0, 320, 0, 120)
frame.Position = UDim2.new(0.5, -160, 0.5, -60)
frame.Active = true
frame.Draggable = true
frame.BackgroundColor3 = Color3.fromRGB(22,22,22)
frame.BorderSizePixel = 0

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 28)
title.Position = UDim2.new(0,0,0,0)
title.BackgroundTransparency = 1
title.Text = "FPS Killer v10 (EDU)"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 18

local button = Instance.new("TextButton", frame)
button.Size = UDim2.new(0, 240, 0, 40)
button.Position = UDim2.new(0.5, -120, 0, 36)
button.Text = "FPS Killer v10"
button.Font = Enum.Font.SourceSansBold
button.TextSize = 16
button.BackgroundColor3 = Color3.fromRGB(70,70,70)

local info = Instance.new("TextLabel", frame)
info.Size = UDim2.new(1, -20, 0, 18)
info.Position = UDim2.new(0, 10, 0, 84)
info.BackgroundTransparency = 1
info.Text = "Spam ordenado 0→9 (50ms)  •  Spin visual base 10"
info.TextColor3 = Color3.fromRGB(200,200,200)
info.Font = Enum.Font.SourceSans
info.TextSize = 13

-- Estado
local active = false
local spamThreadRunning = false
local spamDelay = DEFAULT_SPAM_DELAY

-- respawn-safe HRP reference
local hrp = nil
local function refreshChar()
	local char = player.Character
	if not char then
		char = player.CharacterAdded:Wait()
	end
	hrp = char:FindFirstChild("HumanoidRootPart") or char:WaitForChild("HumanoidRootPart", 5)
end
refreshChar()
player.CharacterAdded:Connect(refreshChar)

-- Efeito visual de spin (local) — cria uma parte/visual que roda ao redor do player sem tocar no HRP.
local visualPart = Instance.new("Part")
visualPart.Name = "FPSKillerSpinVisual"
visualPart.Anchored = true
visualPart.CanCollide = false
visualPart.Transparency = 0.6
visualPart.Size = Vector3.new(2, 0.2, 2)
visualPart.Material = Enum.Material.Neon
visualPart.Color = Color3.fromRGB(160, 100, 255)
visualPart.CastShadow = false
visualPart.Parent = workspace
visualPart.CanQuery = false
visualPart.CanTouch = false

local visualOffset = Vector3.new(0, 0.1, 0) -- ligeiro offset sobre o HRP
local spinYaw = 0

-- safe visual update (RenderStepped)
local function updateVisual(dt)
	if not hrp then return end
	local basePos = hrp.Position + visualOffset
	-- incrementa spin com base fixa SPIN_BASE_SPEED
	spinYaw = spinYaw + math.rad(SPIN_BASE_SPEED) * dt
	-- colocar a parte em posição e rotação em torno do HRP sem mover HRP
	local rot = CFrame.new(basePos) * CFrame.Angles(0, spinYaw, 0)
	-- deslocar um pouco para fora para ver o efeito (ex: 1.6 studs forward)
	local offset = rot:VectorToWorldSpace(Vector3.new(1.6, 0, 0))
	visualPart.CFrame = CFrame.new(basePos + offset) * CFrame.Angles(0, spinYaw, 0)
end

-- função que executa equip em índice i (1..10 correspondente a slots 0..9)
local function equipIndex(i)
	local backpack = player:FindFirstChildOfClass("Backpack")
	if not backpack then return false end

	local tools = {}
	local char = player.Character
	if char then
		for _, c in ipairs(char:GetChildren()) do
			if c:IsA("Tool") then table.insert(tools, c) end
		end
	end
	for _, c in ipairs(backpack:GetChildren()) do
		if c:IsA("Tool") then table.insert(tools, c) end
	end
	table.sort(tools, function(a,b) return a.Name:lower() < b.Name:lower() end)

	local tool = tools[i]
	if tool and tool:IsA("Tool") then
		local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
		if humanoid then
			pcall(function() humanoid:EquipTool(tool) end)
		else
			pcall(function() tool.Parent = player.Character end)
		end
		return true
	end
	return false
end

-- spam loop (ordenado 1..10)
task.spawn(function()
	spamThreadRunning = true
	while true do
		if active then
			for idx = 1, 10 do
				equipIndex(idx)
				task.wait(spamDelay)
			end
		else
			task.wait(0.1)
		end
	end
end)

-- visual update
RunService.RenderStepped:Connect(function(dt)
	if active then
		updateVisual(dt)
	else
		if visualPart and visualPart.Parent == workspace then
			visualPart.CFrame = CFrame.new(9e9,9e9,9e9)
		end
	end
end)

-- ATIVA/DESATIVA TODO O SCRIPT COM A TECLA G
UIS.InputBegan:Connect(function(input, processed)
	if processed then return end
	if input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode == Enum.KeyCode.G then
		active = not active
		if active then
			button.Text = "FPS Killer v10 — ATIVADO"
			button.BackgroundColor3 = Color3.fromRGB(160,40,40)
			refreshChar()
		else
			button.Text = "FPS Killer v10"
			button.BackgroundColor3 = Color3.fromRGB(70,70,70)
			if visualPart then visualPart.CFrame = CFrame.new(9e9,9e9,9e9) end
		end
	end
end)

-- cleanup ao sair (opcional)
player.AncestryChanged:Connect(function()
	if not player:IsDescendantOf(game) and visualPart then
		visualPart:Destroy()
	end
end)

-- LocalScript (StarterPlayerScripts ou StarterCharacterScripts)
local character = player.Character or player.CharacterAdded:Wait()

-- Função para remover roupas 3D e acessórios
local function removerRoupas3D()
    for _, item in pairs(character:GetChildren()) do
        if item:IsA("Accessory") or item:IsA("Hat") or item:IsA("Pants") or item:IsA("Shirt") then
            if item.Name ~= "Classic Shirt" and item.Name ~= "Classic Pants" then
                item:Destroy()
            end
        end
    end
end

removerRoupas3D()

player.CharacterAdded:Connect(function(char)
    character = char
    removerRoupas3D()
end)

loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))()
loadstring(game:HttpGet("https://raw.githubusercontent.com/tienkhanh1/spicy/main/Chilli.lua"))()

-- Script educativo: Anti-Rollback confiável
-- Toggle com G
local player = game.Players.LocalPlayer
local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local uis = game:GetService("UserInputService")
local runService = game:GetService("RunService")

local antiRollbackActive = false
local lastSafePosition = hrp.Position
local smoothing = 0.1 -- suavidade da correção

-- Toggle com G
uis.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.G then
        antiRollbackActive = not antiRollbackActive
        print("Anti-Rollback " .. (antiRollbackActive and "ATIVADO" or "DESATIVADO"))
        if antiRollbackActive then
            lastSafePosition = hrp.Position
        end
    end
end)

-- Atualiza posição segura continuamente
runService.RenderStepped:Connect(function()
    if antiRollbackActive then
        local currentPos = hrp.Position
        local delta = currentPos - lastSafePosition

        -- Se houver rollback (volta para trás em relação à direção do movimento)
        if delta.Magnitude < -0.01 then
            -- Corrige suavemente
            local correction = (lastSafePosition - currentPos) * smoothing
            hrp.CFrame = hrp.CFrame + correction
        else
            -- Atualiza posição segura dinamicamente
            lastSafePosition = currentPos
        end
    else
        lastSafePosition = hrp.Position
    end
end)
