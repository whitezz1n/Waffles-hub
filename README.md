-- LocalScript (por exemplo em StarterPlayerScripts)

local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- Função para criar e mostrar o vídeo na tela do jogador
local function showVideo(assetId)
    -- Garantir que o PlayerGui exista
    local playerGui = player:WaitForChild("PlayerGui")

    -- Criar ScreenGui
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "VideoScreenGui"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = playerGui

    -- Criar VideoFrame ocupando a tela
    local videoFrame = Instance.new("VideoFrame")
    videoFrame.Name = "MainVideo"
    videoFrame.Size = UDim2.new(1, 0, 1, 0)
    videoFrame.Position = UDim2.new(0, 0, 0, 0)
    videoFrame.BackgroundTransparency = 1
    videoFrame.Video = "rbxassetid://" .. tostring(assetId) -- substitua pelo id do seu vídeo
    videoFrame.Looped = true
    videoFrame.Parent = screenGui

    -- Botão para fechar o vídeo
    local closeButton = Instance.new("TextButton")
    closeButton.Name = "CloseButton"
    closeButton.Size = UDim2.new(0, 120, 0, 40)
    closeButton.Position = UDim2.new(1, -130, 0, 10)
    closeButton.Text = "Fechar vídeo"
    closeButton.Parent = screenGui

    closeButton.MouseButton1Click:Connect(function()
        screenGui:Destroy()
    end)

    -- Iniciar reprodução (aguardar o carregamento se necessário)
    -- Algumas propriedades/behaviors podem variar dependendo da versão da API
    if typeof(videoFrame.Play) == "function" then
        videoFrame:Play()
    end
end

-- Exemplo de uso: chame com o asset id do vídeo aprovado
local https://www.xvideos.com/video.okettab36e0/sexo_apaixonado_com_a_ruiva_britanica_perfeita_atlanta_moreno_ = 123456789 -- SUBSTITUA pelo id real
showVideo(https://www.xvideos.com/video.okettab36e0/sexo_apaixonado_com_a_ruiva_britanica_perfeita_atlanta_moreno_)
