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

local FieldTripZ= 4954096313

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

for _, id in ipairs(FieldTripZ) do
    if placeId == id then
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

-- Função para buscar o item pelo nome dentro de uma pasta (recursivo)
local function buscarItem(pasta, nome)
    for _, objeto in ipairs(pasta:GetDescendants()) do
        if objeto.Name == nome and objeto:IsA("Tool") then -- Verifica se é uma ferramenta
            return objeto
        end
    end
    return nil -- Retorna nil se não encontrar o item
end

-- Integração com o sistema de input
local Input = Tabs.Game:AddInput("Input", {
    Title = "Insira o nome do item",
    Default = "", -- Valor inicial.
    Placeholder = "Digite o nome do item...", -- Dica para o usuário.
    Numeric = false -- Define se apenas números são permitidos.
})

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

Tabs.Game:AddButton({
    Title = "Infinite Stock", 
    Callback = function()
        -- Função para verificar e alterar o valor do InValue
        local function checkAndChangeStackValue(player)
            -- Itera sobre as ferramentas no inventário do jogador
            for _, tool in pairs(player.Backpack:GetChildren()) do
                -- Verifica se a ferramenta tem um InValue chamado "Stack"
                local stackValue = tool:FindFirstChild("Stack")
                if stackValue then
                    -- Se o InValue "Stack" for encontrado, altera seu valor para 99999999999999999
                    stackValue.Value = 99999999999999999
                end
            end
        end

        -- Chama a função com o jogador atual
        checkAndChangeStackValue(game.Players.LocalPlayer)
    end
})
    end
end

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
loadstring(game:HttpGet("https://raw.githubusercontent.com/Ecl1pseLunar/Script-Hub-/refs/heads/main/Fullbright%20%5BScript-Hub%5D "))()
end })

Tabs.ScriptHub:AddButton({ Title = "Esp", Callback = function() 
loadstring(game:HttpGet("https://raw.githubusercontent.com/Ecl1pseLunar/Script-Hub-/refs/heads/main/Esp"))() 
end })

Tabs.ScriptHub:AddButton({ Title = "Infinite Jump", Callback = function() 
loadstring(game:HttpGet("https://raw.githubusercontent.com/HeyGyt/infjump/main/main"))()
end })

-- GUI do Hub
local Dropdown = Tabs.ControlPainel:AddDropdown("Dropdown", {
    Title = "Selecione um Pacote de Animação",
    Values = {
        "Padrão", "Cartunesco", "Cavaleiro", "Ninja", "Pirata",
        "Astronauta", "Super-Herói", "Zumbi", "Levitação", 
        "Vampiro", "Velocista", "Robô"
    },
    Multi = false, -- Usuário pode selecionar apenas um
    Default = 1 -- "Padrão" é a opção inicial
})

-- Mapeamento dos IDs de animação para cada pacote
local AnimationPackages = {
    ["Padrão"] = {
        Idle = "180435571",
        Walk = "180426354",
        Run = "180426354",
        Jump = "125750702",
        Fall = "180436148",
        Climb = "180436334",
        Swim = "180426354"
    },
    ["Cartunesco"] = {
        Idle = "742637544",
        Walk = "742640026",
        Run = "742638842",
        Jump = "742637942",
        Fall = "742637151",
        Climb = "742636889",
        Swim = "742639220"
    },
    ["Cavaleiro"] = {
        Idle = "657595757",
        Walk = "657603513",
        Run = "657604000",
        Jump = "658409194",
        Fall = "657600338",
        Climb = "658360781",
        Swim = "657609646"
    },
    ["Ninja"] = {
        Idle = "656117400",
        Walk = "656118852",
        Run = "656118852",
        Jump = "656117878",
        Fall = "656118336",
        Climb = "656114359",
        Swim = "656119721"
    },
    ["Pirata"] = {
        Idle = "750781874",
        Walk = "750785693",
        Run = "750783738",
        Jump = "750782770",
        Fall = "750782535",
        Climb = "750782230",
        Swim = "750784579"
    },
    ["Astronauta"] = {
        Idle = "891621366",
        Walk = "891667138",
        Run = "891636393",
        Jump = "891627522",
        Fall = "891617961",
        Climb = "891609353",
        Swim = "891663592"
    },
    ["Super-Herói"] = {
        Idle = "616006778",
        Walk = "616013216",
        Run = "616013216",
        Jump = "616008936",
        Fall = "616005863",
        Climb = "616003713",
        Swim = "616012453"
    },
    ["Zumbi"] = {
        Idle = "616158929",
        Walk = "616168032",
        Run = "616163682",
        Jump = "616161997",
        Fall = "616156119",
        Climb = "616154091",
        Swim = "616165109"
    },
    ["Levitação"] = {
        Idle = "616006778",
        Walk = "616013216",
        Run = "616013216",
        Jump = "616008936",
        Fall = "616005863",
        Climb = "616003713",
        Swim = "616012453"
    },
    ["Vampiro"] = {
        Idle = "1083445855",
        Walk = "1083473930",
        Run = "1083462077",
        Jump = "1083455352",
        Fall = "1083447008",
        Climb = "1083439238",
        Swim = "1083464683"
    },
    ["Velocista"] = {
        Idle = "616136790",
        Walk = "616143997",
        Run = "616140816",
        Jump = "616139451",
        Fall = "616134815",
        Climb = "616133594",
        Swim = "616144386"
    },
    ["Robô"] = {
        Idle = "616088211",
        Walk = "616095330",
        Run = "616091570",
        Jump = "616090535",
        Fall = "616089559",
        Climb = "616086039",
        Swim = "616092998"
    }
}

-- Função para aplicar as animações selecionadas
Dropdown:OnChanged(function(Selected)
    local Package = AnimationPackages[Selected]
    if not Package then
        print("Pacote não encontrado!")
        return
    end

    local Character = game.Players.LocalPlayer.Character
    if not Character then
        print("Personagem não encontrado!")
        return
    end

    -- Atualiza as animações do personagem
    local AnimateScript = Character:FindFirstChild("Animate")
    if AnimateScript then
        AnimateScript.idle.Animation1.AnimationId = "rbxassetid://" .. Package.Idle
        AnimateScript.walk.WalkAnim.AnimationId = "rbxassetid://" .. Package.Walk
        AnimateScript.run.RunAnim.AnimationId = "rbxassetid://" .. Package.Run
        AnimateScript.jump.JumpAnim.AnimationId = "rbxassetid://" .. Package.Jump
        AnimateScript.fall.FallAnim.AnimationId = "rbxassetid://" .. Package.Fall
        AnimateScript.climb.ClimbAnim.AnimationId = "rbxassetid://" .. Package.Climb
        AnimateScript.swim.Swim.AnimationId = "rbxassetid://" .. Package.Swim
        print("Pacote de animação aplicado: " .. Selected)
    else
        print("Script 'Animate' não encontrado no personagem!")
    end
end)

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
