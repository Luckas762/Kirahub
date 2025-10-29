--== Kira Hub RayField Completo 2025 ==--
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Kira Hub RayField",
   Icon = 0,
   LoadingTitle = "Kira Hub Interface",
   LoadingSubtitle = "by Luckas",
   ShowText = "Kira Hub",
   Theme = "DarkBlue",
   ToggleUIKeybind = "K",
   ConfigurationSaving = {Enabled = true, FolderName = nil, FileName = "KiraHub"},
   Discord = {Enabled = false},
   KeySystem = false
})

local TabVisual = Window:CreateTab("Visual", 4483362458)
local TabMovimento = Window:CreateTab("Movimento", 4483362458)
local TabTeleporte = Window:CreateTab("Teleporte", 4483362458)
local TabConfig = Window:CreateTab("Config", 4483362458)

local Players = game:GetService("Players")
local Player = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local Workspace = game:GetService("Workspace")
local TweenService = game:GetService("TweenService")

local estados = {
    espJogadores=false,
    espItensXP=false,
    velocidade=false,
    velocidadeValor=16,
    brilho=false,
    puloInfinito=false,
    noclip=false,
    teleportItens=false,
    teleportJogadores=false,
    removerTexturas=false,
    modoDebug=false
}

local function ehKiller(player)
    if player.Team and player.Team.Name == "Killer" then return true end
    local role = player:FindFirstChild("Role")
    if role and role.Value == "Killer" then return true end
    if player.Character and player.Character.Name:lower():find("killer") then return true end
    return false
end

local function isBorder(inst)
    if not inst or not inst.Parent then return false end
    return string.find(string.lower(inst.Name), "border") ~= nil
end

local function encontrarAdornee(inst)
    if inst:IsA("BasePart") then return inst end
    if inst:IsA("Model") then
        if inst.PrimaryPart then return inst.PrimaryPart end
        for _, d in ipairs(inst:GetDescendants()) do
            if d:IsA("BasePart") then return d end
        end
    end
    return nil
end

local function repararErros()
    estados.espJogadores = false
    estados.espItensXP = false
    estados.velocidade = false
    estados.brilho = false
    estados.puloInfinito = false
    estados.noclip = false
    estados.teleportItens = false
    estados.teleportJogadores = false
    print("[Debug] Estados do script resetados para evitar erros")
end

local tracked = {}

local function makeBillboardForPart(part)
    if not part or tracked[part] then return end

    local bb = Instance.new("BillboardGui")
    bb.Name = "ESP_Billboard"
    bb.Adornee = part
    bb.AlwaysOnTop = true
    bb.Size = UDim2.new(0,25,0,12)
    bb.StudsOffset = Vector3.new(0,1.5,0)
    bb.MaxDistance = 1000
    bb.Parent = Player:FindFirstChildOfClass("PlayerGui")

    local txt = Instance.new("TextLabel")
    txt.Size = UDim2.new(1,0,1,0)
    txt.BackgroundTransparency = 1
    txt.Text = "XP"
    txt.Font = Enum.Font.Legacy
    txt.TextSize = 8
    txt.TextScaled = false
    txt.TextStrokeTransparency = 1
    txt.TextColor3 = Color3.fromRGB(255,215,0)
    txt.Parent = bb

    local tweenInfo = TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true)
    local goal = {TextColor3 = Color3.fromRGB(255,255,100)}
    local tween = TweenService:Create(txt, tweenInfo, goal)
    tween:Play()

    local emitterPart = Instance.new("Part")
    emitterPart.Anchored = true
    emitterPart.CanCollide = false
    emitterPart.Transparency = 1
    emitterPart.Size = Vector3.new(0.1,0.1,0.1)
    emitterPart.CFrame = part.CFrame + Vector3.new(0,1.5,0)
    emitterPart.Parent = part

    local emitter = Instance.new("ParticleEmitter")
    emitter.Texture = "rbxassetid://3528381440"
    emitter.Color = ColorSequence.new(Color3.fromRGB(255,215,0))
    emitter.LightEmission = 1
    emitter.Rate = 5
    emitter.Lifetime = NumberRange.new(0.5,1)
    emitter.Speed = NumberRange.new(0.5,1)
    emitter.Size = NumberSequence.new(0.2)
    emitter.SpreadAngle = Vector2.new(360,360)
    emitter.Parent = emitterPart

    tracked[part] = {billboard = bb, emitter = emitterPart}
end

local function removeBillboardsOfInstance(inst)
    for part,data in pairs(tracked) do
        if part and part:IsDescendantOf(inst) then
            if data.billboard and data.billboard.Parent then data.billboard:Destroy() end
            if data.emitter and data.emitter.Parent then data.emitter:Destroy() end
            tracked[part] = nil
        end
    end
end

local function createESPForInstance(inst)
    if not inst then return end
    if not (inst:IsA("BasePart") or inst:IsA("Model")) then return end
    if inst:IsA("BasePart") then
        if isBorder(inst) then makeBillboardForPart(inst) end
        return
    end
    for _,child in ipairs(inst:GetDescendants()) do
        if child:IsA("BasePart") and isBorder(child) then
            makeBillboardForPart(child)
        end
    end
end

-- VISUAL
TabVisual:CreateToggle({
    Name = "ESP Jogadores",
    CurrentValue = false,
    Flag = "ESPPlayers",
    Callback = function(Value)
        estados.espJogadores = Value
        if not Value then
            for _, player in pairs(Players:GetPlayers()) do
                if player.Character then
                    local highlight = player.Character:FindFirstChild("ESP_Highlight")
                    if highlight then highlight:Destroy() end
                    local head = player.Character:FindFirstChild("Head")
                    if head then
                        local billboard = head:FindFirstChild("PlayerNameBillboard")
                        if billboard then billboard:Destroy() end
                    end
                end
            end
        end
    end
})

TabVisual:CreateToggle({
    Name = "ESP XP Itens",
    CurrentValue = false,
    Flag = "ESPXPItems",
    Callback = function(Value)
        estados.espItensXP = Value
        if not Value then
            for part,data in pairs(tracked) do
                if data.billboard then data.billboard:Destroy() end
                if data.emitter then data.emitter:Destroy() end
            end
            tracked = {}
        else
            for _, inst in ipairs(Workspace:GetDescendants()) do
                if inst.Name:lower():find("border") then
                    createESPForInstance(inst)
                end
            end
        end
    end
})

TabVisual:CreateToggle({
    Name = "Brilho",
    CurrentValue = false,
    Flag = "Brilho",
    Callback = function(Value)
        estados.brilho = Value
    end
})

-- CONFIG
local function removerTexturasContinuamente()
    while estados.removerTexturas do
        for _, obj in pairs(workspace:GetDescendants()) do
            if obj:IsA("BasePart") then
                obj.Material = Enum.Material.SmoothPlastic
                obj.Reflectance = 0
            end
            if obj:IsA("Texture") or obj:IsA("Decal") then
                obj:Destroy()
            end
        end

        for _, player in ipairs(Players:GetPlayers()) do
            if player.Character then
                for _, obj in ipairs(player.Character:GetDescendants()) do
                    if obj:IsA("ShirtGraphic") or obj:IsA("Shirt") or obj:IsA("Pants") then
                        obj:Destroy()
                    end
                end
            end
        end

        Lighting.GlobalShadows = false
        Lighting.FogEnd = 9e9
        Lighting.Brightness = 1
        Lighting.EnvironmentDiffuseScale = 0
        Lighting.EnvironmentSpecularScale = 0

        for _, p in pairs(workspace:GetDescendants()) do
            if p:IsA("ParticleEmitter") or p:IsA("Sparkles") or p:IsA("Fire") or p:IsA("Smoke") then
                p.Enabled = false
            end
        end

        for _, v in pairs(Lighting:GetDescendants()) do
            if v:IsA("Sky") then v:Destroy() end
        end
        wait(1)
    end
end

TabConfig:CreateToggle({
    Name = "Remover Texturas",
    CurrentValue = false,
    Flag = "RemoverTexturas",
    Callback = function(Value)
        estados.removerTexturas = Value
        if Value then
            spawn(removerTexturasContinuamente)
        end
    end
})

-- MOVIMENTO
local VelSlider = TabMovimento:CreateSlider({
   Name = "Velocidade",
   Range = {16,200},
   Increment = 1,
   Suffix = "WalkSpeed",
   CurrentValue = 16,
   Flag = "VelocidadeSlider",
   Callback = function(Value)
       estados.velocidadeValor = Value
   end
})

TabMovimento:CreateToggle({
    Name = "Ativar Velocidade",
    CurrentValue = false,
    Flag = "VelocidadeToggle",
    Callback = function(Value)
        estados.velocidade = Value
    end
})

TabMovimento:CreateToggle({
    Name = "Pulo Infinito",
    CurrentValue = false,
    Flag = "PuloInfinito",
    Callback = function(Value)
        estados.puloInfinito = Value
    end
})

TabMovimento:CreateToggle({
    Name = "Noclip",
    CurrentValue = false,
    Flag = "Noclip",
    Callback = function(Value)
        estados.noclip = Value
    end
})

-- TELEPORTE ITENS substituído pelo seu novo script ultra rápido, sem bolha

local originalPropsPerChar = {}
local running = false
local tpThread = nil

local function saveOriginalState(char)
    if originalPropsPerChar[char] then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if hrp then
        originalPropsPerChar[char] = {
            hrp = {Transparency=hrp.Transparency, CanCollide=hrp.CanCollide, Anchored=hrp.Anchored, CFrame=hrp.CFrame},
            parts = {}
        }
        for _,obj in ipairs(char:GetDescendants()) do
            if obj:IsA("BasePart") and obj ~= hrp then
                originalPropsPerChar[char].parts[obj] = {Transparency=obj.Transparency, CanCollide=obj.CanCollide}
            end
            if obj:IsA("Accessory") and obj:FindFirstChild("Handle") then
                originalPropsPerChar[char].parts[obj.Handle] = {Transparency=obj.Handle.Transparency, CanCollide=obj.Handle.CanCollide}
            end
        end
    end
end

local function restoreOriginalState(char)
    local props = originalPropsPerChar[char]
    if not props then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if hrp and props.hrp then
        hrp.Anchored = true -- ANCORAR para evitar cair
        hrp.CFrame = props.hrp.CFrame -- restaurar posição
        wait(0.2) -- espera estabilizar
        hrp.Anchored = props.hrp.Anchored -- desancora conforme original
        hrp.Transparency = props.hrp.Transparency
        hrp.CanCollide = props.hrp.CanCollide
    end
    for part,p in pairs(props.parts) do
        if part and part.Parent then
            part.Transparency = p.Transparency
            part.CanCollide = p.CanCollide
        end
    end
    originalPropsPerChar[char] = nil
end

local function applyInvisibility(char)
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    saveOriginalState(char)
    pcall(function() hrp.Transparency = 1; hrp.CanCollide = true; hrp.Anchored = false end)
    for _,obj in ipairs(char:GetDescendants()) do
        if obj:IsA("BasePart") and obj ~= hrp then pcall(function() obj.Transparency=1; obj.CanCollide=false end) end
        if obj:IsA("Accessory") and obj:FindFirstChild("Handle") then pcall(function() obj.Handle.Transparency=1; obj.Handle.CanCollide=false end) end
    end
end

local function hidePlayerGuis(char)
    for _,desc in ipairs(char:GetDescendants()) do
        if desc:IsA("BillboardGui") or desc:IsA("SurfaceGui") then
            if not desc:FindFirstChild("OriginalEnabled") then
                local val = Instance.new("BoolValue")
                val.Name = "OriginalEnabled"
                val.Value = desc.Enabled
                val.Parent = desc
            end
            desc.Enabled = false
        end
    end
end

local function restorePlayerGuis(char)
    for _,desc in ipairs(char:GetDescendants()) do
        if desc:IsA("BillboardGui") or desc:IsA("SurfaceGui") then
            local val = desc:FindFirstChild("OriginalEnabled")
            if val then desc.Enabled=val.Value; val:Destroy() end
        end
    end
end

local function getAllBorders()
    local out = {}
    for _,inst in ipairs(Workspace:GetDescendants()) do
        if (inst:IsA("BasePart") or inst:IsA("UnionOperation")) and inst.Name:lower():find("border") then
            table.insert(out, inst)
        end
        if inst:IsA("Model") and inst.Name:lower():find("border") then
            for _,child in ipairs(inst:GetDescendants()) do
                if child:IsA("BasePart") or child:IsA("UnionOperation") then
                    table.insert(out, child)
                end
            end
        end
    end
    return out
end

local function teleportLoop()
    local char = Player.Character or Player.CharacterAdded:Wait()
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    while running and hrp.Parent do
        local borders = getAllBorders()
        for _,b in ipairs(borders) do
            if not running then break end
            if b and b.Parent then
                hrp.CFrame = b.CFrame -- teleporte instantâneo
                task.wait(0) -- yield mínimo
            end
        end
        task.wait(0.05)
    end
end

TabTeleporte:CreateToggle({
    Name = "Teleporte Itens",
    CurrentValue = false,
    Flag = "TeleportItens",
    Callback = function(Value)
        estados.teleportItens = Value
        running = Value

        local charNow = Player.Character or Player.CharacterAdded:Wait()

        if Value then
            saveOriginalState(charNow)
            applyInvisibility(charNow)
            hidePlayerGuis(charNow)
            tpThread = task.spawn(teleportLoop)
        else
            if tpThread then
                pcall(task.cancel, tpThread)
                tpThread = nil
            end
            restoreOriginalState(charNow)
            restorePlayerGuis(charNow)
        end
    end
})

TabTeleporte:CreateToggle({
    Name = "Teleporta Jogadores",
    CurrentValue = false,
    Flag = "TeleportPlayers",
    Callback = function(Value)
        estados.teleportJogadores = Value
    end
})

-- NOVO: Botão ESCAPAR (teleporte rápido para o Bouncer)
TabTeleporte:CreateButton({
    Name = "Escapar",
    Description = "Teleporte rápido para o Bouncer",
    Callback = function()
        local player = Players.LocalPlayer
        local function getBouncerPart(bouncer)
            if bouncer:IsA("BasePart") then return bouncer end
            if bouncer:IsA("Model") then
                if bouncer.PrimaryPart then return bouncer.PrimaryPart end
                for _, part in pairs(bouncer:GetDescendants()) do
                    if part:IsA("BasePart") then return part end
                end
            end
            return nil
        end

        local function findNearestBouncer()
            if not player.Character or not player.Character:FindFirstChild("HumanoidRootPart") then return nil end
            local HRP = player.Character.HumanoidRootPart
            local nearestBouncer = nil
            local shortestDistance = math.huge

            for _, obj in pairs(Workspace:GetDescendants()) do
                if obj.Name:lower():find("bouncer") then
                    local part = getBouncerPart(obj)
                    if part then
                        local dist = (part.Position - HRP.Position).Magnitude
                        if dist < shortestDistance then
                            shortestDistance = dist
                            nearestBouncer = part
                        end
                    end
                end
            end
            return nearestBouncer
        end

        local function magnetToBouncer(targetPart)
            if not player.Character or not targetPart then return end
            local HRP = player.Character:FindFirstChild("HumanoidRootPart")
            if not HRP then return end

            local tweenSpeed = 1.2
            local connection
            connection = RunService.RenderStepped:Connect(function()
                if not HRP.Parent or not targetPart.Parent then
                    connection:Disconnect()
                    return
                end

                local newCFrame = HRP.CFrame:Lerp(CFrame.new(targetPart.Position + Vector3.new(0,3,0)), tweenSpeed)
                HRP.CFrame = newCFrame

                if (HRP.Position - targetPart.Position).Magnitude < 1 then
                    connection:Disconnect()
                end
            end)
        end

        local nearest = findNearestBouncer()
        if nearest then
            magnetToBouncer(nearest)
        else
            warn("⚠️ Nenhum Bouncer encontrado!")
        end
    end
})

RunService.RenderStepped:Connect(function()
    if Player.Character and Player.Character:FindFirstChildOfClass("Humanoid") then
        local humanoid = Player.Character:FindFirstChildOfClass("Humanoid")
        humanoid.WalkSpeed = estados.velocidade and estados.velocidadeValor or 16

        if estados.brilho then
            Lighting.Brightness = 2
            Lighting.ClockTime = 14
            Lighting.GlobalShadows = false
            Lighting.FogEnd = 100000
        else
            Lighting.Brightness = 1
            Lighting.GlobalShadows = true
        end
    end
end)

Workspace.DescendantAdded:Connect(function(obj)
    if estados.espItensXP and obj.Name:lower():find("border") then
        createESPForInstance(obj)
    end
end)

Workspace.DescendantRemoving:Connect(function(obj)
    if estados.espItensXP then
        removeBillboardsOfInstance(obj)
    end
end)

RunService.RenderStepped:Connect(function()
    for _,player in pairs(Players:GetPlayers()) do
        if player ~= Player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local HRP = player.Character.HumanoidRootPart

            local highlight = player.Character:FindFirstChild("ESP_Highlight")
            local head = player.Character:FindFirstChild("Head")
            local billboard = head and head:FindFirstChild("PlayerNameBillboard")

            if estados.espJogadores then
                if not highlight then
                    highlight = Instance.new("Highlight", player.Character)
                    highlight.Name = "ESP_Highlight"
                end
                highlight.Adornee = player.Character
                if ehKiller(player) then
                    highlight.FillColor = Color3.fromRGB(255,0,0)
                    highlight.FillTransparency = 0.5
                    highlight.OutlineColor = Color3.fromRGB(255,0,0)
                else
                    highlight.FillColor = Color3.fromRGB(0,162,255)
                    highlight.FillTransparency = 0.5
                    highlight.OutlineColor = Color3.fromRGB(0,162,255)
                end
                highlight.Enabled = true

                if head then
                    if not billboard then
                        billboard = Instance.new("BillboardGui", head)
                        billboard.Name = "PlayerNameBillboard"
                        billboard.Adornee = head
                        billboard.AlwaysOnTop = true
                        billboard.Size = UDim2.new(0,100,0,20)
                        billboard.StudsOffset = Vector3.new(0,2,0)
                        local label = Instance.new("TextLabel", billboard)
                        label.Name = "NameLabel"
                        label.Size = UDim2.new(1,0,1,0)
                        label.BackgroundTransparency = 1
                        label.TextColor3 = Color3.fromRGB(255,255,255)
                        label.Font = Enum.Font.Gotham
                        label.TextScaled = false
                        label.TextSize = 14
                        label.TextStrokeTransparency = 0
                    end
                    local dist = (head.Position - Player.Character.HumanoidRootPart.Position).Magnitude
                    if billboard and billboard:FindFirstChild("NameLabel") then
                        billboard.NameLabel.Text = player.Name.." ["..math.floor(dist).."m]"
                    end
                end
            else
                if highlight then highlight:Destroy() end
                if billboard then billboard:Destroy() end
            end
        end
    end
end)

UIS.JumpRequest:Connect(function()
    if estados.puloInfinito and Player.Character then
        local humanoid = Player.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then humanoid:ChangeState(Enum.HumanoidStateType.Jumping) end
    end
end)

RunService.Stepped:Connect(function()
    if Player.Character then
        for _,part in pairs(Player.Character:GetChildren()) do
            if part:IsA("BasePart") then part.CanCollide = not estados.noclip end
        end
    end
end)

local teleportPlayersList = {}
local teleportPlayerIndex = 1

local function atualizarJogadoresParaTeleporte()
    teleportPlayersList = {}
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= Player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            table.insert(teleportPlayersList, plr)
        end
    end
end

RunService.RenderStepped:Connect(function()
    if estados.teleportJogadores and Player.Character then
        local HRP = Player.Character:FindFirstChild("HumanoidRootPart")
        if HRP then
            if #teleportPlayersList == 0 then atualizarJogadoresParaTeleporte() end
            local alvo = teleportPlayersList[teleportPlayerIndex]
            if alvo and alvo.Character and alvo.Character:FindFirstChild("HumanoidRootPart") then
                HRP.CFrame = alvo.Character.HumanoidRootPart.CFrame + Vector3.new(0,3,0)
                teleportPlayerIndex = teleportPlayerIndex + 1
                if teleportPlayerIndex > #teleportPlayersList then teleportPlayerIndex = 1 end
            else
                atualizarJogadoresParaTeleporte()
            end
        end
    end
end)

print("[Kira Hub RayField] Todas as funções carregadas com sucesso!")
