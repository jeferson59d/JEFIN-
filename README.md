-- JEFIN Menu - Universal (Mobile + Desktop)
-- Mantém o menu igual ao original (apenas adiciona controles touch para FOV / Smooth e botão AIM)
-- Não usa Drawing; usa PlayerGui, BillboardGui, UIStroke

--// Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local UIS = UserInputService

--// Variáveis
local aimbotActive = false
local fovActive = false
local fovRadius = 120
local espBoxActive = false
local espHealthActive = false
local espNameActive = false
local holdingAimButton = false -- para mobile: botão virtual
local holdingRightClick = false -- para desktop (mouse right)
local menuVisible = true
local smoothness = 0.15 -- valor inicial (pode ser ajustado pelo slider ou botões)

--// GUI Menu
-- Parent ao PlayerGui (para máxima compatibilidade)
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "JefinScreenGui"
ScreenGui.Parent = PlayerGui
ScreenGui.ResetOnSpawn = false

local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0,240,0,260)
Frame.Position = UDim2.new(0.5,-120,0.5,-130)
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

-- Função rápida para criar botões no mesmo estilo
local function CreateButton(y, text)
    local btn = Instance.new("TextButton", Frame)
    btn.Size = UDim2.new(0,220,0,25)
    btn.Position = UDim2.new(0,10,0,y)
    btn.Text = text
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 14
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.BackgroundColor3 = Color3.fromRGB(0,200,0)
    return btn
end

-- BOTÕES do menu (mantidos iguais)
local AimbotButton = CreateButton(35,"AIMBOT")
AimbotButton.MouseButton1Click:Connect(function()
    aimbotActive = not aimbotActive
    AimbotButton.BackgroundColor3 = aimbotActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
end)

local FovButton = CreateButton(65,"FOV")
FovButton.MouseButton1Click:Connect(function()
    fovActive = not fovActive
    FovButton.BackgroundColor3 = fovActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
    -- mostrar/ocultar círculo também
    if fovActive then
        ScreenGui:FindFirstChild("FOVCircle").Visible = true
    else
        ScreenGui:FindFirstChild("FOVCircle").Visible = false
    end
end)

local EspBoxButton = CreateButton(95,"ESP BOX")
EspBoxButton.MouseButton1Click:Connect(function()
    espBoxActive = not espBoxActive
    EspBoxButton.BackgroundColor3 = espBoxActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
end)

local EspVidaButton = CreateButton(125,"ESP VIDA")
EspVidaButton.MouseButton1Click:Connect(function()
    espHealthActive = not espHealthActive
    EspVidaButton.BackgroundColor3 = espHealthActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
end)

local EspNomeButton = CreateButton(155,"ESP NOME")
EspNomeButton.MouseButton1Click:Connect(function()
    espNameActive = not espNameActive
    EspNomeButton.BackgroundColor3 = espNameActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
end)

-- Slider (visual) FOV
local SliderBack = Instance.new("Frame", Frame)
SliderBack.Name = "SliderBackFOV"
SliderBack.Size = UDim2.new(0,200,0,6)
SliderBack.Position = UDim2.new(0,10,0,190)
SliderBack.BackgroundColor3 = Color3.fromRGB(60,60,60)

local SliderFill = Instance.new("Frame", SliderBack)
SliderFill.Size = UDim2.new(0.4,0,1,0)
SliderFill.BackgroundColor3 = Color3.fromRGB(200,0,0)

local FovValue = Instance.new("TextLabel", Frame)
FovValue.Size = UDim2.new(0,200,0,20)
FovValue.Position = UDim2.new(0,10,0,200)
FovValue.Text = "FOV: "..fovRadius
FovValue.Font = Enum.Font.Gotham
FovValue.TextSize = 14
FovValue.TextColor3 = Color3.fromRGB(255,255,255)
FovValue.BackgroundTransparency = 1

-- Slider Smooth (visual)
local SmoothBack = Instance.new("Frame", Frame)
SmoothBack.Name = "SliderBackSmooth"
SmoothBack.Size = UDim2.new(0,200,0,6)
SmoothBack.Position = UDim2.new(0,10,0,225)
SmoothBack.BackgroundColor3 = Color3.fromRGB(60,60,60)

local SmoothFill = Instance.new("Frame", SmoothBack)
SmoothFill.Size = UDim2.new(0.15,0,1,0)
SmoothFill.BackgroundColor3 = Color3.fromRGB(0,200,200)

local SmoothValue = Instance.new("TextLabel", Frame)
SmoothValue.Size = UDim2.new(0,200,0,20)
SmoothValue.Position = UDim2.new(0,10,0,235)
SmoothValue.Text = "Smooth: "..smoothness
SmoothValue.Font = Enum.Font.Gotham
SmoothValue.TextSize = 14
SmoothValue.TextColor3 = Color3.fromRGB(255,255,255)
SmoothValue.BackgroundTransparency = 1

-- BOTÕES touch para Mobile (próximos aos sliders)
-- FOV - decrease
local FovDec = Instance.new("TextButton", Frame)
FovDec.Size = UDim2.new(0,30,0,20)
FovDec.Position = UDim2.new(0,215,0,198)
FovDec.Text = "-"
FovDec.Font = Enum.Font.Gotham
FovDec.TextSize = 18
FovDec.TextColor3 = Color3.fromRGB(255,255,255)
FovDec.BackgroundColor3 = Color3.fromRGB(40,40,40)
FovDec.MouseButton1Click:Connect(function()
    fovRadius = math.max(30, fovRadius - 10)
    FovValue.Text = "FOV: "..fovRadius
end)

-- FOV - increase
local FovInc = Instance.new("TextButton", Frame)
FovInc.Size = UDim2.new(0,30,0,20)
FovInc.Position = UDim2.new(0,215,0,182)
FovInc.Text = "+"
FovInc.Font = Enum.Font.Gotham
FovInc.TextSize = 18
FovInc.TextColor3 = Color3.fromRGB(255,255,255)
FovInc.BackgroundColor3 = Color3.fromRGB(40,40,40)
FovInc.MouseButton1Click:Connect(function()
    fovRadius = math.min(1000, fovRadius + 10)
    FovValue.Text = "FOV: "..fovRadius
end)

-- Smooth - decrease
local SmoothDec = Instance.new("TextButton", Frame)
SmoothDec.Size = UDim2.new(0,30,0,20)
SmoothDec.Position = UDim2.new(0,215,0,233)
SmoothDec.Text = "-"
SmoothDec.Font = Enum.Font.Gotham
SmoothDec.TextSize = 18
SmoothDec.TextColor3 = Color3.fromRGB(255,255,255)
SmoothDec.BackgroundColor3 = Color3.fromRGB(40,40,40)
SmoothDec.MouseButton1Click:Connect(function()
    smoothness = math.max(0.01, math.floor((smoothness - 0.01) * 100)/100)
    SmoothValue.Text = "Smooth: "..smoothness
end)

-- Smooth - increase
local SmoothInc = Instance.new("TextButton", Frame)
SmoothInc.Size = UDim2.new(0,30,0,20)
SmoothInc.Position = UDim2.new(0,215,0,217)
SmoothInc.Text = "+"
SmoothInc.Font = Enum.Font.Gotham
SmoothInc.TextSize = 18
SmoothInc.TextColor3 = Color3.fromRGB(255,255,255)
SmoothInc.BackgroundColor3 = Color3.fromRGB(40,40,40)
SmoothInc.MouseButton1Click:Connect(function()
    smoothness = math.min(1.0, math.floor((smoothness + 0.01) * 100)/100)
    SmoothValue.Text = "Smooth: "..smoothness
end)

-- Botão virtual para mirar (usável em Mobile)
local AimHoldButton = Instance.new("TextButton", Frame)
AimHoldButton.Size = UDim2.new(0,220,0,30)
AimHoldButton.Position = UDim2.new(0,10,0,260) -- colocado logo abaixo (pode ficar fora da frame se exceder; vamos ajustar)
-- como Frame tem 260 de altura, posiciono para ficar visível: mover para y = 235
AimHoldButton.Position = UDim2.new(0,10,0,235)
AimHoldButton.Text = "AIM (HOLD)"
AimHoldButton.Font = Enum.Font.Gotham
AimHoldButton.TextSize = 14
AimHoldButton.TextColor3 = Color3.fromRGB(255,255,255)
AimHoldButton.BackgroundColor3 = Color3.fromRGB(40,40,40)
AimHoldButton.AutoButtonColor = true

AimHoldButton.MouseButton1Down:Connect(function()
    holdingAimButton = true
end)
AimHoldButton.MouseButton1Up:Connect(function()
    holdingAimButton = false
end)
-- Touch events também (seguro para mobile)
AimHoldButton.TouchStarted:Connect(function()
    holdingAimButton = true
end)
AimHoldButton.TouchEnded:Connect(function()
    holdingAimButton = false
end)

-- Ajuste para manter frame tamanho original: se botão empurra, deixe visível (não alterei o visual original do menu muito)
-- Caso queira, pode mover o botão para fora do Frame, mas mantive dentro pra não mudar menu drasticamente.

-- Controle sliders por mouse (arrastar)
local draggingFov = false
local draggingSmooth = false

SliderBack.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then draggingFov = true end
end)
SmoothBack.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then draggingSmooth = true end
end)
UIS.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingFov = false
        draggingSmooth = false
    end
end)

-- Círculo FOV (UI) - centralizado na tela
local FOVCircle = Instance.new("Frame", ScreenGui)
FOVCircle.Name = "FOVCircle"
FOVCircle.AnchorPoint = Vector2.new(0.5, 0.5)
FOVCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
FOVCircle.Size = UDim2.new(0, fovRadius*2, 0, fovRadius*2)
FOVCircle.BackgroundTransparency = 1
FOVCircle.Visible = false
-- UIStroke para borda circular (uisquare com uicorner)
local FOVCorner = Instance.new("UICorner", FOVCircle)
FOVCorner.CornerRadius = UDim.new(1, 0) -- torna redondo
local FOVStroke = Instance.new("UIStroke", FOVCircle)
FOVStroke.Thickness = 2
FOVStroke.Transparency = 0
FOVStroke.Color = Color3.fromRGB(255,0,0)

-- Função para ajustar tamanho do círculo baseado em fovRadius (em pixels)
local function updateFOVCircle()
    local diameter = math.max(6, math.floor(fovRadius * 2))
    FOVCircle.Size = UDim2.new(0, diameter, 0, diameter)
    FOVCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
end
updateFOVCircle()

-- ESP: usa BillboardGui criado no LocalPlayer.PlayerGui, adornee = HumanoidRootPart
local espCache = {}

local function createESPFor(plr)
    if espCache[plr] then return end
    local bill = Instance.new("BillboardGui")
    bill.Name = "JefinESP_"..plr.Name
    bill.Adornee = nil
    bill.AlwaysOnTop = true
    bill.Size = UDim2.new(0,150,0,60)
    bill.StudsOffset = Vector3.new(0,2.5,0)
    bill.Parent = PlayerGui

    local container = Instance.new("Frame", bill)
    container.Size = UDim2.new(1,0,1,0)
    container.BackgroundTransparency = 1

    local nameLabel = Instance.new("TextLabel", container)
    nameLabel.Name = "ESPName"
    nameLabel.Size = UDim2.new(1,0,0,20)
    nameLabel.Position = UDim2.new(0,0,0,0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Font = Enum.Font.Gotham
    nameLabel.TextSize = 14
    nameLabel.Text = plr.Name
    nameLabel.TextColor3 = Color3.fromRGB(255,255,255)
    nameLabel.TextStrokeTransparency = 0.6

    local boxFrame = Instance.new("Frame", container)
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
        Bill = bill,
        Name = nameLabel,
        Box = boxFrame,
        Health = healthBar
    }
end

local function removeESP(plr)
    if espCache[plr] then
        pcall(function() espCache[plr].Bill:Destroy() end)
        espCache[plr] = nil
    end
end

-- Cria ESPs para jogadores existentes
for _,plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then
        createESPFor(plr)
    end
end
Players.PlayerAdded:Connect(function(plr)
    if plr ~= LocalPlayer then
        task.spawn(function() createESPFor(plr) end)
    end
end)
Players.PlayerRemoving:Connect(function(plr)
    removeESP(plr)
end)

-- Input botão direito (desktop)
UIS.InputBegan:Connect(function(input, gp)
    if not gp and input.UserInputType == Enum.UserInputType.MouseButton2 then
        holdingRightClick = true
    end
end)
UIS.InputEnded:Connect(function(input, gp)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        holdingRightClick = false
    end
end)

-- Toggle Menu com H (mantive)
UIS.InputBegan:Connect(function(input, gp)
    if not gp and input.KeyCode == Enum.KeyCode.H then
        menuVisible = not menuVisible
        Frame.Visible = menuVisible
    end
end)

-- Função para checar se um ponto de tela está dentro do FOV (pixels)
local function isInFov(screenX, screenY)
    local mousePos = UIS:GetMouseLocation()
    local dx = screenX - mousePos.X
    local dy = screenY - mousePos.Y
    local dist = math.sqrt(dx*dx + dy*dy)
    return dist <= fovRadius
end

-- Encontrar alvo mais próximo em tela dentro do FOV (simples)
local function getClosestTarget()
    local closest = nil
    local closestDist = math.huge
    local mousePos = UIS:GetMouseLocation()

    for _,plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChildOfClass("Humanoid") then
            local hrp = plr.Character.HumanoidRootPart
            local screenPos, onScreen = Camera:WorldToScreenPoint(hrp.Position)
            if onScreen then
                local dx = screenPos.X - mousePos.X
                local dy = screenPos.Y - mousePos.Y
                local dist = math.sqrt(dx*dx + dy*dy)
                if dist <= fovRadius and dist < closestDist then
                    closest = plr
                    closestDist = dist
                end
            end
        end
    end

    return closest
end

-- Atualizações por frame
RunService.RenderStepped:Connect(function(dt)
    -- sliders: arrastar com mouse (desktop)
    if draggingFov then
        local mouseX = UIS:GetMouseLocation().X
        local rel = math.clamp((mouseX - SliderBack.AbsolutePosition.X) / SliderBack.AbsoluteSize.X, 0, 1)
        SliderFill.Size = UDim2.new(rel, 0, 1, 0)
        fovRadius = math.floor(30 + rel * 970) -- 30 .. 1000
        FovValue.Text = "FOV: "..fovRadius
        updateFOVCircle()
    end
    if draggingSmooth then
        local mouseX = UIS:GetMouseLocation().X
        local rel = math.clamp((mouseX - SmoothBack.AbsolutePosition.X) / SmoothBack.AbsoluteSize.X, 0, 1)
        SmoothFill.Size = UDim2.new(rel, 0, 1, 0)
        smoothness = math.floor((0.01 + rel * 0.99) * 100)/100 -- 0.01 .. 1.00
        SmoothValue.Text = "Smooth: "..smoothness
    end

    -- Atualiza visibilidade do círculo
    FOVCircle.Visible = fovActive

    -- Atualiza ESP para cada jogador
    for plr,widgets in pairs(espCache) do
        if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChildOfClass("Humanoid") then
            local hrp = plr.Character.HumanoidRootPart
            widgets.Bill.Adornee = hrp
            widgets.Name.Text = plr.Name
            widgets.Name.Visible = espNameActive
            widgets.Box.Visible = espBoxActive
            local humanoid = plr.Character:FindFirstChildOfClass("Humanoid")
            if humanoid and humanoid.Health and humanoid.MaxHealth and espHealthActive then
                local percent = math.clamp(humanoid.Health / humanoid.MaxHealth, 0, 1)
                widgets.Health.Size = UDim2.new(percent, 0, 0.2, 0)
                widgets.Health.Visible = true
            else
                widgets.Health.Visible = false
            end
        else
            widgets.Bill.Adornee = nil
            widgets.Name.Visible = false
            widgets.Box.Visible = false
            widgets.Health.Visible = false
        end
    end

    -- AIM: ativado se aimbotActive e (segurando botão direito no PC OU segurando botão virtual no Mobile)
    local aiming = aimbotActive and (holdingRightClick or holdingAimButton)
    if aiming then
        local targetPlayer = getClosestTarget()
        if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = targetPlayer.Character.HumanoidRootPart
            local camC = Camera.CFrame
            local lookCF = CFrame.new(camC.Position, hrp.Position)
            Camera.CFrame = camC:lerp(lookCF, math.clamp(smoothness, 0.01, 1))
        end
    end
end)

-- Garante que espCache entries tenham propriedades corretas (se foram criadas antes)
-- Ajuste: se espCache foi criado sem bill, conserta (compatibilidade)
for plr, entry in pairs(espCache) do
    if entry and entry.Bill and entry.Name and entry.Box and entry.Health then
        -- ok
    else
        removeESP(plr)
        createESPFor(plr)
    end
end

-- Inicial print
print("JEFIN Menu universal carregado — ESP (BillboardGui), FOV circle (UI) e Aimbot com controles Mobile prontos.")
