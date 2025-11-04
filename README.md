local ESPFolder = Instance.new("Folder", game.CoreGui)
ESPFolder.Name = "BrainrotESPVisual"

local function getBrainrots()
    local brainrots = {}
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Model") and obj:FindFirstChild("BrainrotTag") then
            local brainrotValue = obj:FindFirstChild("Value") and obj.Value.Value or 0
            local brainrotName = obj.Name
            local rotsPerSecond = obj:FindFirstChild("RotsPerSecond") and obj.RotsPerSecond.Value or 0
            table.insert(brainrots, {
                Instance = obj,
                Value = brainrotValue,
                Name = brainrotName,
                RotsPerSec = rotsPerSecond
            })
        end
    end
    return brainrots
end

local function getBestBrainrot(brainrots)
    table.sort(brainrots, function(a, b)
        return a.Value > b.Value
    end)
    return brainrots[1]
end

local function drawESPLine(target, rotsPerSec, name)
    local camera = workspace.CurrentCamera
    if ESPFolder:FindFirstChild("BrainrotLine") then
        ESPFolder.BrainrotLine:Destroy()
    end
    if not target or not target:FindFirstChild("HumanoidRootPart") then return end

    local rootPart = target.HumanoidRootPart
    local l = Drawing.new("Line")
    l.Color = Color3.fromRGB(255, 0, 0)
    l.Thickness = 3
    l.Transparency = 1
    l.From = camera:WorldToViewportPoint(game.Players.LocalPlayer.Character.HumanoidRootPart.Position)
    l.To = camera:WorldToViewportPoint(rootPart.Position)
    l.ZIndex = 2
    l.Visible = true
    l.Name = "BrainrotLine"
    l.Parent = ESPFolder

    -- Info Text
    local t = Drawing.new("Text")
    t.Text = string.format("🧠 %s | %.2f rot/s", name, rotsPerSec)
    t.Color = Color3.fromRGB(255,0,0)
    t.Size = 18
    t.Center = true
    t.Outline = true
    t.Position = camera:WorldToViewportPoint(rootPart.Position) + Vector3.new(0, -30, 0)
    t.Visible = true
    t.Name = "BrainrotInfo"
    t.Parent = ESPFolder
end

game:GetService("RunService").RenderStepped:Connect(function()
    for _, v in ipairs(ESPFolder:GetChildren()) do v:Destroy() end
    local brainrots = getBrainrots()
    if #brainrots == 0 then return end
    local best = getBestBrainrot(brainrots)
    drawESPLine(best.Instance, best.RotsPerSec, best.Name)
end)
