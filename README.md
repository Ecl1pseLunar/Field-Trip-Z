local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

local Window = Fluent:CreateWindow({
    Title = "System " .. Fluent.Version,
    SubTitle = "by... ",
    TabWidth = 160,
    Size = UDim2.fromOffset(580, 460),
    Acrylic = true,
    Theme = "Dark",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Main = Window:AddTab({ Title = "Main", Icon = "" }),
    Game = Window:AddTab({ Title = "Game", Icon = "" }),
    Teleport = Window:AddTab({ Title = "Teleport", Icon = "" }),
    ControlPainel = Window:AddTab({ Title = "Control Painel", Icon = "" }),
    ScriptHub = Window:AddTab({ Title = "Script Hub", Icon = "" }),
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" })
}

local Options = Fluent.Options

local humanoidToggleState = false
local godToggleState = false
local healAllToggleState = false
local isHealAllActive = false
local playerList = {}
local isLoopActive = true

-- Função para curar jogadores (usado no Heal All)
local function healPlayers()
    if not isLoopActive or not healAllToggleState then return end

    if #playerList == 0 then
        playerList = {}  -- Limpar lista de jogadores
        for _, otherPlayer in ipairs(game:GetService("Players"):GetPlayers()) do
            if otherPlayer ~= game:GetService("Players").LocalPlayer then
                table.insert(playerList, otherPlayer.Name)
            end
        end
    end

    if #playerList > 0 then
        local currentPlayerName = playerList[1]
        table.remove(playerList, 1)

        local args = {
            [1] = "HEAL_PLAYER",
            [2] = game:GetService("Players")[currentPlayerName],
            [3] = 99999
        }

        game:GetService("ReplicatedStorage").NetworkEvents.RemoteFunction:InvokeServer(unpack(args))
        wait(0.2)
        healPlayers() -- Recursão para curar todos
    end
end

-- Função para ativar/desativar o Heal All
local function toggleHealAll(state)
    healAllToggleState = state
    if state then
        healPlayers()  -- Iniciar o loop de cura
    end
end

 -- Abar Game:
 
Tabs.Game:AddParagraph({ Title = "Game", Content = "Use exploiter para alterar o jogo: obtenha itens, torne-se imortal, cure aliados e explore outras funções." })

-- Botão Supplies
local suppliesToggle = Tabs.Game:AddToggle("Supplies", {Title = "Supplies", Default = false })
suppliesToggle:OnChanged(function(state)
    humanoidToggleState = state
    while humanoidToggleState do
        wait(0.1)
        -- Fornecer MedKit
        local args = { [1] = "PICKUP_ITEM", [2] = "MedKit" }
        game:GetService("ReplicatedStorage").NetworkEvents.RemoteFunction:InvokeServer(unpack(args))
        -- Fornecer Donut
        args = { [1] = "PICKUP_ITEM", [2] = "Donut" }
        game:GetService("ReplicatedStorage").NetworkEvents.RemoteFunction:InvokeServer(unpack(args))
    end
end)

-- Botão God Mode
local godToggle = Tabs.Game:AddToggle("God", {Title = "God Mode", Default = false })
godToggle:OnChanged(function(state)
    godToggleState = state
    while godToggleState do
        wait(0.1)
        local args = { [1] = "HEAL_PLAYER", [2] = game:GetService("Players").LocalPlayer, [3] = 99999 }
        game:GetService("ReplicatedStorage").NetworkEvents.RemoteFunction:InvokeServer(unpack(args))
    end
end)

-- Botão Heal All
local healAllToggle = Tabs.Game:AddToggle("HealAll", {Title = "Heal All", Default = false })
healAllToggle:OnChanged(function(state)
    toggleHealAll(state)
end)

-- Função para teleportar para um local específico
local function teleportToPosition(x, y, z)
    game:GetService("Players").LocalPlayer.Character:SetPrimaryPartCFrame(CFrame.new(x, y, z))
end

Tabs.Game:AddParagraph({ Title = "Tool", Content = "Digite o nome do item que deseja!" })

-- Integração com o sistema de input
local Input = Tabs.Game:AddInput("Input", {
    Title = "Insira o nome do item",
    Default = "NameTool", -- Valor inicial.
    Placeholder = "Digite o nome do item...", -- Dica para o usuário.
    Numeric = false -- Define se apenas números são permitidos.
})

-- Função para buscar o item pelo nome dentro de uma pasta (recursivo)
local function buscarItem(pasta, nome)
    for _, objeto in ipairs(pasta:GetDescendants()) do
        if objeto.Name == nome then
            return objeto
        end
    end
    return nil -- Retorna nil se não encontrar o item
end

-- Evento para capturar o valor digitado no input
Input:OnChanged(function(valorDigitado)
    print("Você digitou:", valorDigitado)
    
    local localizacao = game:GetService("ReplicatedStorage") -- Local onde os itens estão (altere se necessário)
    
    -- Busca o item
    local itemEncontrado = buscarItem(localizacao, valorDigitado)
    
    if itemEncontrado then
        -- Clona o item encontrado
        local itemClonado = itemEncontrado:Clone()
        -- Move o item clonado para o inventário do jogador
        itemClonado.Parent = game.Players.LocalPlayer.Backpack
        print("Item '" .. valorDigitado .. "' clonado e movido para o inventário!")
    else
        print("Item '" .. valorDigitado .. "' não encontrado! Verifique o nome e o local.")
    end
end)


 -- Abar do teleport

Tabs.Teleport:AddParagraph({ Title = "Teleport", Content = "Permite teletransportar-se para locais específicos do jogo com facilidade e rapidez." })

-- Botões de Teleport
Tabs.Teleport:AddButton({
    Title = "Field Trip Z",
    Description = "Teleport to Field Trip Z",
    Callback = function()
        game:GetService("TeleportService"):Teleport(4954096313, game:GetService("Players").LocalPlayer)
    end
})

Tabs.Teleport:AddButton({
    Title = "Field Trip Z [Hardcore]",
    Description = "Teleport to Field Trip Z [Hardcore]",
    Callback = function()
        game:GetService("TeleportService"):Teleport(5941839954, game:GetService("Players").LocalPlayer)
    end
})

Tabs.Teleport:AddButton({
    Title = "Field Trip Z [Game]",
    Description = "Teleport to Field Trip Z [Game]",
    Callback = function()
        game:GetService("TeleportService"):Teleport(5096191125, game:GetService("Players").LocalPlayer)
    end
})

Tabs.Teleport:AddButton({
    Title = "Sala De História",
    Description = "Teleport to Story Room",
    Callback = function()
        teleportToPosition(-19, 7, -18)
    end
})

Tabs.Teleport:AddButton({
    Title = "Diretoria",
    Description = "Teleport to Directorate",
    Callback = function()
        teleportToPosition(83, 7, -48)
    end
})

Tabs.Teleport:AddButton({
    Title = "Sala De Ciência",
    Description = "Teleport to Science Room",
    Callback = function()
        teleportToPosition(185, 7, -123)
    end
})

Tabs.Teleport:AddButton({
    Title = "Cafeteira",
    Description = "Teleport to Coffee Maker",
    Callback = function()
        teleportToPosition(76, 7, -133)
    end
})

Tabs.Teleport:AddButton({
    Title = "Cozinha",
    Description = "Teleport to Kitchen",
    Callback = function()
        teleportToPosition(97, 7, -134)
    end
})

Tabs.Teleport:AddButton({
    Title = "Blox`N go",
    Description = "Teleport to Blox`N go",
    Callback = function()
        teleportToPosition(-104, 3, 342)
    end
})

 -- Abar Script HubHub
 
Tabs.ScriptHub:AddParagraph({ Title = "Script Hub", Content = "Central para scripts, com funções de exploiter e customizações para modificar o jogo de diversas maneiras." })
 
Tabs.ScriptHub:AddButton({ Title = "Dex Mobile", Callback = function() 
loadstring(game:HttpGet("https://raw.githubusercontent.com/realredz/DEX-Explorer/refs/heads/main/Mobile.lua"))()
end })
 
Tabs.ScriptHub:AddButton({ Title = "Fullbright", Callback = function() 
 -- Start
-- Script para remover o fog e desativar as sombras
local lighting = game:GetService("Lighting")

-- Remover o fog (névoa) de maneira definitiva
lighting.FogStart = 1000000  -- Início do fog extremamente distante
lighting.FogEnd = 10000000  -- Fim do fog mais distante ainda
lighting.FogColor = Color3.fromRGB(255, 255, 255)  -- Cor do fog configurada para branco, praticamente invisível

-- Desativar sombras
lighting.GlobalShadows = false  -- Desativa as sombras globais, removendo todas as sombras do jogo

-- Iluminação fullbright (sem sombras e com iluminação intensa)
lighting.Ambient = Color3.fromRGB(255, 255, 255)  -- Cor do ambiente totalmente iluminada
lighting.Brightness = 2  -- Brilho ajustado para garantir iluminação, mas sem exagero
lighting.OutdoorAmbient = Color3.fromRGB(255, 255, 255)  -- Luz externa clara, sem sombras
lighting.TimeOfDay = "14:00:00"  -- Definindo uma hora do dia que ilumina intensamente (tarde)
 -- End
end })

-- Salvando Configurações
SaveManager:SetLibrary(Fluent)
InterfaceManager:SetLibrary(Fluent)
SaveManager:IgnoreThemeSettings()
SaveManager:SetIgnoreIndexes({})
InterfaceManager:SetFolder("FluentScriptHub")
SaveManager:SetFolder("FluentScriptHub/specific-game")
InterfaceManager:BuildInterfaceSection(Tabs.Settings)
SaveManager:BuildConfigSection(Tabs.Settings)

-- Seleciona a Tab Inicial
Window:SelectTab(1)

Fluent:Notify({
    Title = "Fluent",
    Content = "The script has been loaded.",
    Duration = 8
})

-- Carrega as configurações autoload
SaveManager:LoadAutoloadConfig()
