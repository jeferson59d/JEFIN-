-- JEFIN Menu - Universal (SEM AUTO REVISTAR)
--  Compatível com executores Mobile (Delta) e desktop. Não usa Drawing; usa PlayerGui + BillboardGui.

--// Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

--// Variáveis
local aimbotActive = false
local fovActive = false
local fovRadius = 120
local espBoxActive = false
local espHealthActive = false
local espNameActive = false
local holdingRightClick = false
local menuVisible = true
local smoothness = 0.15

-- Itens para pegar (mantido, sem uso por enquanto)
local ItensParaPegar = {
    "Pt", "Glock", "PtNeon", "G19x", "USPs", "AK47", "G3", "Uzi", "Mac",
    "Fuzil", "TungTung", "m4A1s", "FuzilTrovão", "FuzilPantom", "FuzilHollow",
    "Oitão", "Escudo", "Colote", "Parafal.762", "ArmaDoPd", "PtNike", "PtDragon"
}

--// GUI Menu (em PlayerGui para compatibilidade universal)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.ResetOnSpawn = false

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0,260,0,360)
Frame.Position = UDim2.new(0.5,-130,0.5,-180)
Frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = true

local Title = Instance.new("TextLabel", Frame)
Title.Size = UDim2.new(1,0,0,25)
Title.Text = "JEFIN Menu"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.TextColor3 = Color3.fromRGB(255,255,255)
Title.BackgroundTransparency = 1

-- Função para criar botões com status
local function CreateButtonWithStatus(y, text, initial)
    local btn = Instance.new("TextButton", Frame)
    btn.Size = UDim2.new(0,180,0,25)
    btn.Position = UDim2.new(0,10,0,y)
    btn.Text = text
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 14
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.BackgroundColor3 = Color3.fromRGB(0,200,0)

    local status = Instance.new("TextLabel", Frame)
    status.Size = UDim2.new(0,60,0,25)
    status.Position = UDim2.new(0,200,0,y)
    status.Text = initial and "ON" or "OFF"
    status.TextColor3 = initial and Color3.fromRGB(0,255,0) or Color3.fromRGB(255,0,0)
    status.BackgroundTransparency = 1
    status.Font = Enum.Font.Gotham
    status.TextSize = 14

    return btn, status
end

-- BOTÕES COM STATUS (mantive os mesmos nomes e posições, exceto removi AUTO REVISTAR)
local AimbotButton, AimbotStatus = CreateButtonWithStatus(35,"AIMBOT", aimbotActive)
AimbotButton.MouseButton1Click:Connect(function()
    aimbotActive = not aimbotActive
    AimbotButton.BackgroundColor3 = aimbotActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
    AimbotStatus.Text = aimbotActive and "ON" or "OFF"
    AimbotStatus.TextColor3 = aimbotActive and Color3.fromRGB(0,255,0) or Color3.fromRGB(255,0,0)
end)

local FovButton, FovStatus = CreateButtonWithStatus(65,"FOV", fovActive)
FovButton.MouseButton1Click:Connect(function()
    fovActive = not fovActive
    FovButton.BackgroundColor3 = fovActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
    FovStatus.Text = fovActive and "ON" or "OFF"
    FovStatus.TextColor3 = fovActive and Color3.fromRGB(0,255,0) or Color3.fromRGB(255,0,0)
end)

local EspBoxButton, EspBoxStatus = CreateButtonWithStatus(95,"ESP BOX", espBoxActive)
EspBoxButton.MouseButton1Click:Connect(function()
    espBoxActive = not espBoxActive
    EspBoxButton.BackgroundColor3 = espBoxActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
    EspBoxStatus.Text = espBoxActive and "ON" or "OFF"
    EspBoxStatus.TextColor3 = espBoxActive and Color3.fromRGB(0,255,0) or Color3.fromRGB(255,0,0)
end)

local EspVidaButton, EspVidaStatus = CreateButtonWithStatus(125,"ESP VIDA", espHealthActive)
EspVidaButton.MouseButton1Click:Connect(function()
    espHealthActive = not espHealthActive
    EspVidaButton.BackgroundColor3 = espHealthActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
    EspVidaStatus.Text = espHealthActive and "ON" or "OFF"
    EspVidaStatus.TextColor3 = espHealthActive and Color3.fromRGB(0,255,0) or Color3.fromRGB(255,0,0)
end)

local EspNomeButton, EspNomeStatus = CreateButtonWithStatus(155,"ESP NOME", espNameActive)
EspNomeButton.MouseButton1Click:Connect(function()
    espNameActive = not espNameActive
    EspNomeButton.BackgroundColor3 = espNameActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
    EspNomeStatus.Text = espNameActive and "ON" or "OFF"
    EspNomeStatus.TextColor3 = espNameActive and Color3.fromRGB(0,255,0) or Color3.fromRGB(255,0,0)
end)

-- Slider FOV
local SliderBack = Instance.new("Frame", Frame)
SliderBack.Size = UDim2.new(0,200,0,6)
SliderBack.Position = UDim2.new(0,10,0,220)
SliderBack.BackgroundColor3 = Color3.fromRGB(60,60,60)

local SliderFill = Instance.new("Frame", SliderBack)
SliderFill.Size = UDim2.new(0.4,0,1,0)
SliderFill.BackgroundColor3 = Color3.fromRGB(200,0,0)

local FovValue = Instance.new("TextLabel", Frame)
FovValue.Size = UDim2.new(0,200,0,20)
FovValue.Position = UDim2.new(0,10,0,230)
FovValue.Text = "FOV: "..fovRadius
FovValue.Font = Enum.Font.Gotham
FovValue.TextSize = 14
FovValue.TextColor3 = Color3.fromRGB(255,255,255)
FovValue.BackgroundTransparency = 1

-- Slider Smooth
local SmoothBack = Instance.new("Frame", Frame)
SmoothBack.Size = UDim2.new(0,200,0,6)
SmoothBack.Position = UDim2.new(0,10,0,255)
SmoothBack.BackgroundColor3 = Color3.fromRGB(60,60,60)

local SmoothFill = Instance.new("Frame", SmoothBack)
SmoothFill.Size = UDim2.new(0.15,0,1,0)
SmoothFill.BackgroundColor3 = Color3.fromRGB(0,200,200)

local SmoothValue = Instance.new("TextLabel", Frame)
SmoothValue.Size = UDim2.new(0,200,0,20)
SmoothValue.Position = UDim2.new(0,10,0,265)
SmoothValue.Text = "Smooth: "..smoothness
SmoothValue.Font = Enum.Font.Gotham
SmoothValue.TextSize = 14
SmoothValue.TextColor3 = Color3.fromRGB(255,255,255)
SmoothValue.BackgroundTransparency = 1

-- Toggle Menu H
UserInputService.InputBegan:Connect(function(input, gp)
    if not gp and input.KeyCode == Enum.KeyCode.H then
        menuVisible = not menuVisible
        Frame.Visible = menuVisible
    end
end)

-- Sliders interactions (mouse location)
local draggingFov = false
local draggingSmooth = false

SliderBack.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingFov = true
    end
end)
SmoothBack.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingSmooth = true
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingFov = false
        draggingSmooth = false
    end
end)

-- Helper para calcular valor de slider a partir da posição do mouse
local function updateSliderFromMouse(backFrame, fillFrame, minVal, maxVal, isFov)
    local mouseX, mouseY = UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y
    local absPos = backFrame.AbsolutePosition
    local absSize = backFrame.AbsoluteSize
    local relativeX = math.clamp((mouseX - absPos.X) / absSize.X, 0, 1)
    fillFrame.Size = UDim2.new(relativeX, 0, 1, 0)
    local value = minVal + (maxVal - minVal) * relativeX
    if isFov then
        fovRadius = math.floor(value)
        FovValue.Text = "FOV: "..fovRadius
    else
        smoothness = math.floor(value*100)/100
        SmoothValue.Text = "Smooth: "..tostring(smoothness)
    end
end

-- Atualiza sliders enquanto arrasta
RunService.RenderStepped:Connect(function()
    if draggingFov then
        updateSliderFromMouse(SliderBack, SliderFill, 30, 1000, true)
    end
    if draggingSmooth then
        updateSliderFromMouse(SmoothBack, SmoothFill, 0.01, 1.0, false)
    end
end)

-- ESP usando BillboardGui (universal)
local function createESPForPlayer(plr)
    local bill = Instance.new("BillboardGui")
    bill.Name = "JefinESP"
    bill.Size = UDim2.new(0,150,0,60)
    bill.Adornee = nil
    bill.AlwaysOnTop = true
    bill.StudsOffset = Vector3.new(0, 2.5, 0)
    bill.Parent = plr:WaitForChild("PlayerGui") -- safe parent; if PlayerGui não existir, este call espera

    local outer = Instance.new("Frame", bill)
    outer.Size = UDim2.new(1,0,1,0)
    outer.BackgroundTransparency = 1

    local nameLabel = Instance.new("TextLabel", outer)
    nameLabel.Name = "ESPName"
    nameLabel.Size = UDim2.new(1,0,0,20)
    nameLabel.Position = UDim2.new(0,0,0,0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Font = Enum.Font.Gotham
    nameLabel.TextSize = 14
    nameLabel.Text = plr.Name
    nameLabel.TextColor3 = Color3.fromRGB(255,255,255)
    nameLabel.TextStrokeTransparency = 0.6

    local boxFrame = Instance.new("Frame", outer)
    boxFrame.Name = "ESPBox"
    boxFrame.Size = UDim2.new(0,80,0,30)
    boxFrame.Position = UDim2.new(0.5,-40,0,20)
    boxFrame.BackgroundTransparency = 0.6
    boxFrame.BackgroundColor3 = Color3.fromRGB(0,0,0)
    boxFrame.BorderSizePixel = 1

    local healthBar = Instance.new("Frame", boxFrame)
    healthBar.Name = "HealthBar"
    healthBar.Size = UDim2.new(1,0,0.2,0)
    healthBar.Position = UDim2.new(0,0,0.8,0)
    healthBar.BackgroundColor3 = Color3.fromRGB(0,255,0)
    healthBar.BorderSizePixel = 0

    return {
        Billboard = bill,
        Name = nameLabel,
        Box = boxFrame,
        Health = healthBar,
    }
end

-- Gerenciar cache de ESP
local espCache = {}
local function ensureESP(plr)
    if not espCache[plr] then
        -- tente anexar ao PlayerGui do LocalPlayer para que o jogador veja
        local container = LocalPlayer:FindFirstChild("PlayerGui")
        -- cria Billboards no PlayerGui do local (necessário para mobile executors sem Drawing)
        local bill = Instance.new("BillboardGui")
        bill.Name = "JefinESP_"..plr.Name
        bill.Size = UDim2.new(0,150,0,60)
        bill.Adornee = nil
        bill.AlwaysOnTop = true
        bill.StudsOffset = Vector3.new(0, 2.5, 0)
        bill.Parent = LocalPlayer:WaitForChild("PlayerGui")

        local outer = Instance.new("Frame", bill)
        outer.Size = UDim2.new(1,0,1,0)
        outer.BackgroundTransparency = 1

        local nameLabel = Instance.new("TextLabel", outer)
        nameLabel.Name = "ESPName"
        nameLabel.Size = UDim2.new(1,0,0,20)
        nameLabel.Position = UDim2.new(0,0,0,0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Font = Enum.Font.Gotham
        nameLabel.TextSize = 14
        nameLabel.Text = plr.Name
        nameLabel.TextColor3 = Color3.fromRGB(255,255,255)
        nameLabel.TextStrokeTransparency = 0.6

        local boxFrame = Instance.new("Frame", outer)
        boxFrame.Name = "ESPBox"
        boxFrame.Size = UDim2.new(0,80,0,30)
        boxFrame.Position = UDim2.new(0.5,-40,0,20)
        boxFrame.BackgroundTransparency = 0.6
        boxFrame.BackgroundColor3 = Color3.fromRGB(0,0,0)
        boxFrame.BorderSizePixel = 1

        local healthBar = Instance.new("Frame", boxFrame)
        healthBar.Name = "HealthBar"
        healthBar.Size = UDim2.new(1,0,0.2,0)
        healthBar.Position = UDim2.new(0,0,0.8,0)
        healthBar.BackgroundColor3 = Color3.fromRGB(0,255,0)
        healthBar.BorderSizePixel = 0

        espCache[plr] = {
            Billboard = bill,
            Name = nameLabel,
            Box = boxFrame,
            Health = healthBar,
        }
    end
end

-- Remove ESP
local function removeESP(plr)
    if espCache[plr] then
        pcall(function()
            espCache[plr].Billboard:Destroy()
        end)
        espCache[plr] = nil
    end
end

-- Atualiza ESP cada frame
RunService.RenderStepped:Connect(function()
    for plr,widgets in pairs(espCache) do
        if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChild("Humanoid") then
            local hrp = plr.Character.HumanoidRootPart
            widgets.Billboard.Adornee = hrp
            -- name
            widgets.Name.Text = plr.Name
            widgets.Name.Visible = espNameActive
            -- box visibility
            widgets.Box.Visible = espBoxActive
            -- health
            local humanoid = plr.Character:FindFirstChild("Humanoid")
            if humanoid and humanoid.Health and humanoid.MaxHealth then
                local healthPercent = math.clamp(humanoid.Health / humanoid.MaxHealth, 0, 1)
                widgets.Health.Size = UDim2.new(healthPercent,0,0.2,0)
                widgets.Health.Visible = espHealthActive
            else
                widgets.Health.Visible = false
            end
        else
            widgets.Billboard.Adornee = nil
            widgets.Name.Visible = false
            widgets.Box.Visible = false
            widgets.Health.Visible = false
        end
    end
end)

-- Cria ESP para jogadores já presentes
for _,plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then
        ensureESP(plr)
    end
end
Players.PlayerAdded:Connect(function(plr)
    if plr ~= LocalPlayer then
        -- pequeno delay para garantir PlayerGui
        task.spawn(function()
            ensureESP(plr)
        end)
    end
end)
Players.PlayerRemoving:Connect(function(plr)
    removeESP(plr)
end)

-- Input botão direito
UserInputService.InputBegan:Connect(function(input, gp)
    if not gp and input.UserInputType == Enum.UserInputType.MouseButton2 then
        holdingRightClick = true
    end
end)
UserInputService.InputEnded:Connect(function(input, gp)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        holdingRightClick = false
    end
end)

-- Função utilitária: verificar se ponto da tela está dentro do FOV (eu uso distância em pixels)
local function isInFov(screenPos)
    local mousePos = UserInputService:GetMouseLocation()
    local dx = screenPos.X - mousePos.X
    local dy = screenPos.Y - mousePos.Y
    local dist = math.sqrt(dx*dx + dy*dy)
    return dist <= fovRadius
end

-- Encontrar alvo válido mais próximo (em tela) dentro do FOV e linha de visão simples
local function getClosestTarget()
    local closest = nil
    local closestDist = math.huge
    local mousePos = UserInputService:GetMouseLocation()

    for _,plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChild("Humanoid") and plr.Character.Humanoid.Health > 0 then
            local hrp = plr.Character.HumanoidRootPart
            local screenPos, onScreen = Camera:WorldToScreenPoint(hrp.Position)
            if onScreen then
                local dx = screenPos.X - mousePos.X
                local dy = screenPos.Y - mousePos.Y
                local dist = math.sqrt(dx*dx + dy*dy)
                if dist <= fovRadius and dist < closestDist then
                    -- opcional: checar raycast para linha de visão (pode ser deixado leve)
                    closest = plr
                    closestDist = dist
                end
            end
        end
    end

    return closest
end

-- Aimbot loop (RenderStepped para suavidade)
RunService.RenderStepped:Connect(function(dt)
    -- atualizar sliders já tratados acima
    if aimbotActive and holdingRightClick then
        local target = getClosestTarget()
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = target.Character.HumanoidRootPart
            -- calcular look at CFrame a partir da câmera para o ponto do head/HRP
            local camCFrame = Camera.CFrame
            local targetPos = hrp.Position
            local lookCF = CFrame.new(camCFrame.Position, targetPos)
            -- suavizar usando Lerp
            local newCFrame = camCFrame:lerp(lookCF, math.clamp(smoothness, 0.01, 1))
            Camera.CFrame = newCFrame
        end
    end
end)

-- Inicializar espCache para todos os players (já feito), garantir ausência do AutoRevistar
-- OBS: Menu visual foi preservado (apenas removido o botão AutoRevistar)

-- Mensagem de confirmação (opcional)
print("JEFIN Menu carregado — Aimbot, FOV e ESP prontos (sem Auto Revistar).")
