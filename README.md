--// Serviços
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualInputManager = game:GetService("VirtualInputManager")
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
local autoRevistarActive = false

-- Itens para pegar
local ItensParaPegar = {
    "Pt", "Glock", "PtNeon", "G19x", "USPs", "AK47", "G3", "Uzi", "Mac", 
    "Fuzil", "TungTung", "m4A1s", "FuzilTrovão", "FuzilPantom", "FuzilHollow",
    "Oitão", "Escudo", "Colote", "Parafal.762", "ArmaDoPd", "PtNike", "PtDragon"
}

--// GUI Menu
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
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

-- BOTÕES COM STATUS
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

local AutoRevistarButton, AutoRevistarStatus = CreateButtonWithStatus(185,"AUTO REVISTAR", autoRevistarActive)
AutoRevistarButton.MouseButton1Click:Connect(function()
    autoRevistarActive = not autoRevistarActive
    AutoRevistarButton.BackgroundColor3 = autoRevistarActive and Color3.fromRGB(200,0,0) or Color3.fromRGB(0,200,0)
    AutoRevistarStatus.Text = autoRevistarActive and "ON" or "OFF"
    AutoRevistarStatus.TextColor3 = autoRevistarActive and Color3.fromRGB(0,255,0) or Color3.fromRGB(255,0,0)
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

-- Controle sliders
local draggingFov = false
local draggingSmooth = false
SliderBack.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then draggingFov = true end end)
SmoothBack.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 then draggingSmooth = true end end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        draggingFov = false
        draggingSmooth = false
    end
end)

-- Círculo FOV
local DrawingCircle = Drawing.new("Circle")
DrawingCircle.Visible = false
DrawingCircle.Thickness = 1.5
DrawingCircle.Color = Color3.fromRGB(255,0,0)
DrawingCircle.NumSides = 64
DrawingCircle.Radius = fovRadius
DrawingCircle.Filled = false

-- Função ESP
local ESP = {}
function ESP.New(player)
    local box = Drawing.new("Square")
    box.Visible = false
    box.Color = Color3.fromRGB(255,0,0)
    box.Thickness = 1.5
    box.Filled = false

    local healthBar = Drawing.new("Line")
    healthBar.Visible = false
    healthBar.Thickness = 2

    local nameTag = Drawing.new("Text")
    nameTag.Visible = false
    nameTag.Size = 14
    nameTag.Center = true
    nameTag.Outline = true
    nameTag.Color = Color3.fromRGB(255,255,255)

    return {Box=box, Health=healthBar, Name=nameTag}
end

local espCache = {}
for _,plr in ipairs(Players:GetPlayers()) do
    if plr ~= LocalPlayer then espCache[plr] = ESP.New(plr) end
end
Players.PlayerAdded:Connect(function(plr)
    if plr ~= LocalPlayer then espCache[plr] = ESP.New(plr) end
end)
Players.PlayerRemoving:Connect(function(plr)
    if espCache[plr] then
        for _,obj in pairs(espCache[plr]) do obj:Remove() end
        espCache[plr] = nil
    end
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

-- Toggle Menu H
UserInputService.InputBegan:                                    
