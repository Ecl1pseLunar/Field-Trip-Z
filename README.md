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
    Humanoid = Window:AddTab({ Title = "Humanoid", Icon = "" }),
    Teleport = Window:AddTab({ Title = "Teleport", Icon = "" }),
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

-- Botão Supplies
local suppliesToggle = Tabs.Humanoid:AddToggle("Supplies", {Title = "Supplies", Default = false })
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
local godToggle = Tabs.Humanoid:AddToggle("God", {Title = "God Mode", Default = false })
godToggle:OnChanged(function(state)
    godToggleState = state
    while godToggleState do
        wait(0.1)
        local args = { [1] = "HEAL_PLAYER", [2] = game:GetService("Players").LocalPlayer, [3] = 99999 }
        game:GetService("ReplicatedStorage").NetworkEvents.RemoteFunction:InvokeServer(unpack(args))
    end
end)

-- Botão Heal All
local healAllToggle = Tabs.Humanoid:AddToggle("HealAll", {Title = "Heal All", Default = false })
healAllToggle:OnChanged(function(state)
    toggleHealAll(state)
end)

-- Função para teleportar para um local específico
local function teleportToPosition(x, y, z)
    game:GetService("Players").LocalPlayer.Character:SetPrimaryPartCFrame(CFrame.new(x, y, z))
end

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

-- Menu de Configurações (Settings)
Tabs.Settings:AddButton({
    Title = "Test Button",
    Description = "Test Button in Settings",
    Callback = function()
        print("Settings Button Pressed")
    end
})

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
