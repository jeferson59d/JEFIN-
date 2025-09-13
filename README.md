-- JEFIN Menu — versão robusta (fallbacks, pcall, universal)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- segurança: aguardar LocalPlayer/Camera se necessário
if not LocalPlayer then
    repeat task.wait() LocalPlayer = Players.LocalPlayer until LocalPlayer
end
if not Camera or not Camera.ClassName then
    repeat task.wait() Camera = workspace.CurrentCamera until Camera and Camera.ClassName
end

local debugMode = false -- true para prints extras

-- Escolhe o melhor parent de GUI (tenta PlayerGui, depois CoreGui)
local function chooseGuiParent()
    local pg = nil
    pcall(function() pg = LocalPlayer:FindFirstChild("PlayerGui") end)
    if pg then
        local ok = pcall(function()
            local t = Instance.new("Frame")
            t.Parent = pg
            t:Destroy()
        end)
        if ok then return pg end
    end
    local cg = game:GetService("CoreGui")
    local ok2 = pcall(function()
        local t = Instance.new("Frame")
        t.Parent = cg
        t:Destroy()
    end)
    if ok2 then return cg end
    -- último recurso
    return LocalPlayer:FindFirstChild("PlayerGui") or game:GetService("CoreGui")
end

local GuiParent = chooseGuiParent()
if debugMode then print("GUI parent:", GuiParent and GuiParent:GetFullName()) end

--// Variáveis de estado
local aimbotActive = false
local fovActive = false
local fovRadius = 120
local espBoxActive = false
local espHealthActive = false
local espNameActive = false
local holdingAimButton = false
local holdingRightClick = false
local menuVisible = true
local smoothness = 0.15

-- Função segura para criar instâncias (evita crash ao setar Parent)
local function safeParent(instance, parent)
    local ok, res = pcall(function() instance.Parent = parent return true end)
    if not ok then
        -- tenta parent alternativo
        pcall(function() instance.Parent = chooseGuiParent() end)
    end
end

--// GUI (mantendo layout muito parecido com o seu)
local PlayerGui = GuiParent -- pode ser PlayerGui ou CoreGui dependendo do ambiente

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "JefinScreenGui"
safeParent(ScreenGui, PlayerGui)
ScreenGui.ResetOnSpawn = false

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0,240,0,260)
Frame.Position = UDim2.new(0.5,-120,0.5,-130)
Frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Draggable = true
Frame.Parent = ScreenGui

local Title = Instance.new("TextLabel", Frame)
Title.Size = UDim2.new(1,0,0,25)
Title.Text = "JEFIN Menu"
Title.Font = Enum.Font.GothamBold
Title.TextSize = 16
Title.TextColor3 = Color3.fromRGB(255,255,255)
Title.BackgroundTransparency = 1

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

-- botões (mesmo visual)
local AimbotButton = CreateButton(35,"AIMBOT")
AimbotButton.MouseButton1Click:Connect(function()
    aimbotActive = not aimbotActive
    AimbotButton.BackgroundColor3 = aimbotActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
end)

local FovButton = CreateButton(65,"FOV")
FovButton.MouseButton1Click:Connect(function()
    fovActive = not fovActive
    FovButton.BackgroundColor3 = fovActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
    pcall(function() ScreenGui:FindFirstChild("FOVCircle").Visible = fovActive end)
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

-- Sliders visuais
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

-- botões + / - para mobile
local function createSmallBtn(x,y,txt)
    local b = Instance.new("TextButton", Frame)
    b.Size = UDim2.new(0,30,0,20)
    b.Position = UDim2.new(0,x,0,y)
    b.Text = txt
    b.Font = Enum.Font.Gotham
    b.TextSize = 18
    b.TextColor3 = Color3.fromRGB(255,255,255)
    b.BackgroundColor3 = Color3.fromRGB(40,40,40)
    return b
end

local FovInc = createSmallBtn(0.89,0.71,"+")
FovInc.MouseButton1Click:Connect(function()
    fovRadius = math.min(1000, fovRadius + 10)
    FovValue.Text = "FOV: "..fovRadius
end)
local FovDec = createSmallBtn(0.89,0.77,"-")
FovDec.MouseButton1Click:Connect(function()
    fovRadius = math.max(30, fovRadius - 10)
    FovValue.Text = "FOV: "..fovRadius
end)
local SmoothInc = createSmallBtn(0.89,0.85,"+")
SmoothInc.MouseButton1Click:Connect(function()
    smoothness = math.min(1, math.floor((smoothness + 0.01)*100)/100)
    SmoothValue.Text = "Smooth: "..smoothness
end)
local SmoothDec = createSmallBtn(0.89,0.91,"-")
SmoothDec.MouseButton1Click:Connect(function()
    smoothness = math.max(0.01, math.floor((smoothness - 0.01)*100)/100)
    SmoothValue.Text = "Smooth: "..smoothness
end)

-- Botão AIM (HOLD) touch-friendly
local AimHoldButton = Instance.new("TextButton")
AimHoldButton.Size = UDim2.new(0,220,0,30)
AimHoldButton.Position = UDim2.new(0,10,0,235)
AimHoldButton.Text = "AIM (HOLD)"
AimHoldButton.Font = Enum.Font.Gotham
AimHoldButton.TextSize = 14
AimHoldButton.TextColor3 = Color3.fromRGB(255,255,255)
AimHoldButton.BackgroundColor3 = Color3.fromRGB(40,40,40)
AimHoldButton.Parent = Frame

AimHoldButton.MouseButton1Down:Connect(function() holdingAimButton = true end)
AimHoldButton.MouseButton1Up:Connect(function() holdingAimButton = false end)
AimHoldButton.TouchStarted:Connect(function() holdingAimButton = true end)
AimHoldButton.TouchEnded:Connect(function() holdingAimButton = false end)

-- FOV circle UI
local FOVCircle = Instance.new("Frame")
FOVCircle.Name = "FOVCircle"
FOVCircle.AnchorPoint = Vector2.new(0.5,0.5)
FOVCircle.Position = UDim2.new(0.5,0,0.5,0)
FOVCircle.Size = UDim2.new(0, fovRadius*2, 0, fovRadius*2)
FOVCircle.BackgroundTransparency = 1
FOVCircle.Parent = ScreenGui

local corner = Instance.new("UICorner", FOVCircle)
corner.CornerRadius = UDim.new(1,0)
local stroke = Instance.new("UIStroke", FOVCircle)
stroke.Thickness = 2
stroke.Color = Color3.fromRGB(255,0,0)

local function updateFOVCircle()
    local d = math.max(6, math.floor(fovRadius*2))
    FOVCircle.Size = UDim2.new(0, d, 0, d)
    FOVCircle.Position = UDim2.new(0.5,0,0.5,0)
end
updateFOVCircle()
FOVCircle.Visible = fovActive

-- Esp cache
local espCache = {}

local function createESPFor(plr)
    if espCache[plr] then return end
    local bill = Instance.new("BillboardGui")
    bill.Name = "JefinESP_"..plr.Name
    bill.Adornee = nil
    bill.AlwaysOnTop = true
    bill.Size = UDim2.new(0,150,0,60)
    bill.StudsOffset = Vector3.new(0,2.5,0)
    safeParent(bill, PlayerGui)

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

-- init esp
for _,plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then createESPFor(plr) end
end
Players.PlayerAdded:Connect(function(plr) if plr ~= LocalPlayer then task.spawn(function() createESPFor(plr) end) end end)
Players.PlayerRemoving:Connect(removeESP)

-- input desktop right click
UIS.InputBegan:Connect(function(input, gp)
    if not gp and input.UserInputType == Enum.UserInputType.MouseButton2 then holdingRightClick = true end
end)
UIS.InputEnded:Connect(function(input, gp)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then holdingRightClick = false end
end)

-- toggle menu H
UIS.InputBegan:Connect(function(input, gp)
    if not gp and input.KeyCode == Enum.KeyCode.H then
        menuVisible = not menuVisible
        pcall(function() Frame.Visible = menuVisible end)
    end
end)

-- helpers
local function getMousePos()
    local ok, res = pcall(function() return UIS:GetMouseLocation() end)
    if ok and res then
        return res.X or res.x or res.Position.X, res.Y or res.y or res.Position.Y
    end
    -- fallback center
    local vs = Camera and Camera.ViewportSize or Vector2.new(800,600)
    return vs.X/2, vs.Y/2
end

local function getClosestTarget()
    local mx, my = getMousePos()
    local best, bestDist = nil, math.huge
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChildOfClass("Humanoid") then
            local hrp = plr.Character.HumanoidRootPart
            local screenPos, onScreen = Camera:WorldToScreenPoint(hrp.Position)
            if onScreen then
                local dx = screenPos.X - mx
                local dy = screenPos.Y - my
                local d = math.sqrt(dx*dx + dy*dy)
                if d <= fovRadius and d < bestDist then
                    best = plr
                    bestDist = d
                end
            end
        end
    end
    return best
end

-- Render loop com pcall: previne crash total
RunService.RenderStepped:Connect(function(dt)
    local ok, err = pcall(function()
        -- sliders por arraste (desktop)
        if SliderBack and SliderBack.AbsolutePosition then
            if UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) and UIS:GetMouseLocation() then
                -- checa se mouse está sobre a barra (simples)
                -- dragging detection simplificado: se mouse Y está aproximadamente no slider Y
                local mx,my = getMousePos()
                local abs = SliderBack.AbsolutePosition
                local size = SliderBack.AbsoluteSize
                if my >= abs.Y and my <= abs.Y + size.Y and mx >= abs.X and mx <= abs.X + size.X then
                    local rel = math.clamp((mx - abs.X)/size.X, 0, 1)
                    SliderFill.Size = UDim2.new(rel,0,1,0)
                    fovRadius = math.floor(30 + rel * 970)
                    FovValue.Text = "FOV: "..fovRadius
                    updateFOVCircle()
                end
            end
            if UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) and UIS:GetMouseLocation() then
                local mx,my = getMousePos()
                local abs = SmoothBack.AbsolutePosition
                local size = SmoothBack.AbsoluteSize
                if my >= abs.Y and my <= abs.Y + size.Y and mx >= abs.X and mx <= abs.X + size.X then
                    local rel = math.clamp((mx - abs.X)/size.X, 0, 1)
                    SmoothFill.Size = UDim2.new(rel,0,1,0)
                    smoothness = math.floor((0.01 + rel*0.99)*100)/100
                    SmoothValue.Text = "Smooth: "..smoothness
                end
            end
        end

        -- atualiza círculo
        FOVCircle.Visible = fovActive
        updateFOVCircle()

        -- atualiza ESP
        for plr, widget in pairs(espCache) do
            local okChar = plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") and plr.Character:FindFirstChildOfClass("Humanoid")
            if okChar then
                widget.Bill.Adornee = plr.Character.HumanoidRootPart
                widget.Name.Text = plr.Name
                widget.Name.Visible = espNameActive
                widget.Box.Visible = espBoxActive
                local humanoid = plr.Character:FindFirstChildOfClass("Humanoid")
                if humanoid and humanoid.Health and humanoid.MaxHealth and espHealthActive then
                    local percent = math.clamp(humanoid.Health / humanoid.MaxHealth, 0, 1)
                    widget.Health.Size = UDim2.new(percent, 0, 0.2, 0)
                    widget.Health.Visible = true
                else
                    widget.Health.Visible = false
                end
            else
                widget.Bill.Adornee = nil
                widget.Name.Visible = false
                widget.Box.Visible = false
                widget.Health.Visible = false
            end
        end

        -- AIM logic
        local aiming = aimbotActive and (holdingRightClick or holdingAimButton)
        if aiming then
            local target = getClosestTarget()
            if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = target.Character.HumanoidRootPart
                local cam = Camera and Camera.CFrame
                if cam then
                    local look = CFrame.new(cam.Position, hrp.Position)
                    Camera.CFrame = cam:lerp(look, math.clamp(smoothness, 0.01, 1))
                end
            end
        end
    end)
    if not ok and debugMode then
        warn("RenderStepped pcall erro:", err)
    end
end)

-- Garantir que espCache esteja ok (recria se corrompido)
for plr,_ in pairs(espCache) do
    if not (espCache[plr] and espCache[plr].Bill) then
        removeESP(plr)
        createESPFor(plr)
    end
end

print("JEFIN universal carregado (tente usar o botão AIM e os + / - do menu).")
