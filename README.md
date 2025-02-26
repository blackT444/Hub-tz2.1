-- Criar interface de controle
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "PlayerESP"
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0.2, 0, 0.1, 0)
Frame.Position = UDim2.new(0.4, 0, 0.85, 0)
Frame.BackgroundTransparency = 0.5
Frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Frame.Parent = ScreenGui

local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(1, 0, 1, 0)
ToggleButton.Text = "ON"
ToggleButton.TextScaled = true
ToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleButton.Parent = Frame

-- Lista de cores únicas para jogadores
local nameColors = {}
for i = 1, 50 do
    table.insert(nameColors, Color3.fromHSV(i / 50, 1, 1)) -- Gerar cores diferentes
end

local assignedColors = {}
local npcColor = Color3.fromRGB(255, 255, 255) -- Branco para NPCs
local objectColor = Color3.fromRGB(255, 255, 255) -- Branco para objetos

-- Pegar uma cor única para cada jogador
local function getUniqueColor(player)
    local playerName = player.Name
    if assignedColors[playerName] then
        return assignedColors[playerName]
    end
    for _, color in ipairs(nameColors) do
        if not table.find(assignedColors, color) then
            assignedColors[playerName] = color
            return color
        end
    end
    return Color3.fromRGB(255, 255, 255) -- Branco se acabar as cores
end

-- Criar ESP (Nome e linha ao redor do jogador ou NPC)
local function createESP(character, name, color)
    local head = character:FindFirstChild("Head")
    if head then
        -- Nome acima do personagem
        local billboardGui = Instance.new("BillboardGui")
        billboardGui.Name = "ESP"
        billboardGui.Adornee = head
        billboardGui.Size = UDim2.new(2, 0, 1, 0)
        billboardGui.AlwaysOnTop = true

        local nameLabel = Instance.new("TextLabel")
        nameLabel.Parent = billboardGui
        nameLabel.BackgroundTransparency = 1
        nameLabel.Size = UDim2.new(1, 0, 1, 0)
        nameLabel.Text = name
        nameLabel.TextColor3 = color
        nameLabel.TextScaled = true
        nameLabel.Font = Enum.Font.SourceSansBold

        billboardGui.Parent = head

        -- Linha ao redor do personagem
        local highlight = Instance.new("Highlight")
        highlight.Name = "ESP_Highlight"
        highlight.Parent = character
        highlight.FillTransparency = 1
        highlight.OutlineColor = color
        highlight.OutlineTransparency = 0
    end
end

-- Adicionar ESP a todos os jogadores
local function addESPToPlayers()
    assignedColors = {} -- Resetar cores
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character then
            createESP(player.Character, player.Name, getUniqueColor(player))
        end
    end
end

-- Adicionar ESP a NPCs
local function addESPToNPCs()
    for _, model in pairs(workspace:GetChildren()) do
        if model:IsA("Model") and not game.Players:GetPlayerFromCharacter(model) then
            if model:FindFirstChild("Humanoid") and model:FindFirstChild("Head") then
                createESP(model, "NPC", npcColor)
            end
        end
    end
end

-- Adicionar ESP a objetos (como computadores)
local function addESPToObjects()
    for _, object in pairs(workspace:GetChildren()) do
        if object:IsA("Part") and object.Name:lower():find("computer") then -- Verifica se o objeto é um computador
            createESP(object, object.Name, objectColor) -- Atribui a cor branca
        end
    end
end

-- Atualizar quando jogadores entram
game.Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        wait(1) -- Aguardar carregamento
        createESP(player.Character, player.Name, getUniqueColor(player))
    end)
end)

-- Atualizar quando jogadores saem (liberar cor)
game.Players.PlayerRemoving:Connect(function(player)
    assignedColors[player.Name] = nil
end)

-- Alternar visibilidade do ESP
local showESP = true
local function toggleESP()
    showESP = not showESP
    for _, player in pairs(game.Players:GetPlayers()) do
        if player.Character then
            local head = player.Character:FindFirstChild("Head")
            if head then
                local esp = head:FindFirstChild("ESP")
                if esp then esp.Enabled = showESP end
            end
            local highlight = player.Character:FindFirstChild("ESP_Highlight")
            if highlight then highlight.Enabled = showESP end
        end
    end

    -- Alternar para NPCs
    for _, model in pairs(workspace:GetChildren()) do
        if model:IsA("Model") and not game.Players:GetPlayerFromCharacter(model) then
            local head = model:FindFirstChild("Head")
            if head then
                local esp = head:FindFirstChild("ESP")
                if esp then esp.Enabled = showESP end
            end
            local highlight = model:FindFirstChild("ESP_Highlight")
            if highlight then highlight.Enabled = showESP end
        end
    end

    -- Alternar para objetos
    for _, object in pairs(workspace:GetChildren()) do
        if object:IsA("Part") and object.Name:lower():find("computer") then
            local esp = object:FindFirstChild("ESP")
            if esp then esp.Enabled = showESP end
        end
    end

    ToggleButton.Text = showESP and "ON" or "OFF"
end

-- Conectar botão ao evento
ToggleButton.MouseButton1Click:Connect(toggleESP)

-- Ativar ESP inicialmente
addESPToPlayers()
addESPToNPCs()
addESPToObjects()
