-- Criar o ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MobileHub"
screenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
screenGui.ResetOnSpawn = false

-- Criar o Hub principal
local hubFrame = Instance.new("Frame")
hubFrame.Name = "HubFrame"
hubFrame.Parent = screenGui
hubFrame.Size = UDim2.new(0.5, 0, 0.3, 0) -- Tamanho proporcional para dispositivos móveis
hubFrame.Position = UDim2.new(0.25, 0, 0.35, 0)
hubFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
hubFrame.BorderSizePixel = 0
hubFrame.Visible = false -- Inicialmente oculto para ser aberto pelo botão
hubFrame.Draggable = true
hubFrame.Active = true

-- Adicionar um título ao Hub
local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "TitleLabel"
titleLabel.Parent = hubFrame
titleLabel.Size = UDim2.new(1, 0, 0.2, 0)
titleLabel.Position = UDim2.new(0, 0, 0, 0)
titleLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
titleLabel.Text = "Mobile Hub v2"
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextScaled = true
titleLabel.Font = Enum.Font.SourceSans

-- Botão para ativar/desativar o loop
local toggleButton = Instance.new("TextButton")
toggleButton.Name = "ToggleButton"
toggleButton.Parent = hubFrame
toggleButton.Size = UDim2.new(0.8, 0, 0.3, 0)
toggleButton.Position = UDim2.new(0.1, 0, 0.3, 0)
toggleButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
toggleButton.Text = "Ativar"
toggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleButton.TextScaled = true
toggleButton.Font = Enum.Font.SourceSans

-- Variável para controle do loop
local loopActive = false
local loopCoroutine

-- Função para o botão de ativar/desativar
toggleButton.MouseButton1Click:Connect(function()
    loopActive = not loopActive
    if loopActive then
        toggleButton.Text = "Desativar Suprimentos"
        loopCoroutine = coroutine.create(function()
            while loopActive do
                wait(0)
                -- Primeiro conjunto de argumentos
                local args1 = {
                    [1] = "PICKUP_ITEM",
                    [2] = "MedKit"
                }
                game:GetService("ReplicatedStorage").NetworkEvents.RemoteFunction:InvokeServer(unpack(args1))

                -- Segundo conjunto de argumentos
                local args2 = {
                    [1] = "PICKUP_ITEM",
                    [2] = "Donut"
                }
                game:GetService("ReplicatedStorage").NetworkEvents.RemoteFunction:InvokeServer(unpack(args2))
            end
        end)
        coroutine.resume(loopCoroutine)
    else
        toggleButton.Text = "Suprimentos"
    end
end)

-- Botão móvel para abrir/fechar o Hub
local toggleHubButton = Instance.new("TextButton")
toggleHubButton.Name = "ToggleHubButton"
toggleHubButton.Parent = screenGui
toggleHubButton.Size = UDim2.new(0.1, 0, 0.1, 0)
toggleHubButton.Position = UDim2.new(0.05, 0, 0.85, 0)
toggleHubButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
toggleHubButton.Text = "Hub"
toggleHubButton.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleHubButton.TextScaled = true
toggleHubButton.Font = Enum.Font.SourceSans
toggleHubButton.Draggable = true
toggleHubButton.Active = true

-- Função para abrir/fechar o Hub
toggleHubButton.MouseButton1Click:Connect(function()
    hubFrame.Visible = not hubFrame.Visible
end)

-- Botão "Curar"
local healButton = Instance.new("TextButton")
healButton.Name = "HealButton"
healButton.Parent = hubFrame
healButton.Size = UDim2.new(0.8, 0, 0.3, 0)
healButton.Position = UDim2.new(0.1, 0, 0.65, 0)
healButton.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
healButton.Text = "Curar"
healButton.TextColor3 = Color3.fromRGB(255, 255, 255)
healButton.TextScaled = true
healButton.Font = Enum.Font.SourceSans

-- Variável para controle da cura
local healActive = false
local healCoroutine

-- Função para o botão de ativar/desativar cura
healButton.MouseButton1Click:Connect(function()
    healActive = not healActive
    if healActive then
        healButton.Text = "Parar Cura"
        healCoroutine = coroutine.create(function()
            while healActive do
                wait(0)
                -- Argumentos para curar o jogador
                local args = {
                    [1] = "HEAL_PLAYER",
                    [2] = game:GetService("Players").LocalPlayer,
                    [3] = 50 -- Quantidade de cura
                }
                game:GetService("ReplicatedStorage").NetworkEvents.RemoteFunction:InvokeServer(unpack(args))
            end
        end)
        coroutine.resume(healCoroutine)
    else
        healButton.Text = "Curar"
    end
end)
