-- Checa GUI existente
if game.CoreGui:FindFirstChild("Caio_hub") then
    game.CoreGui.Caio_hub:Destroy()
end

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local player = Players.LocalPlayer

-- ===================== FPS BOOST =====================
-- Reduz efeitos gráficos e sombras para aumentar FPS
Lighting.GlobalShadows = false
Lighting.FogEnd = 9e9
Lighting.Brightness = 1
Lighting.OutdoorAmbient = Color3.fromRGB(128,128,128)
pcall(function() workspace:FindFirstChildOfClass("Terrain").WaterWaveSize = 0 end)
for _, v in pairs(workspace:GetDescendants()) do
    if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") then
        v.Enabled = false
    elseif v:IsA("SurfaceGui") then
        v.Enabled = false
    end
end

-- ===================== VARIÁVEIS =====================
local plataformaAtiva = false
local plataforma
local plataformaConnection
local espAtivo = false

-- ===================== FUNÇÕES DE UI =====================
local function addUICorner(frame, radius)
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, radius)
    corner.Parent = frame
end

local function addUIStroke(frame, color, thickness)
    local stroke = Instance.new("UIStroke")
    stroke.Color = color
    stroke.Thickness = thickness
    stroke.Parent = frame
end

-- ===================== SCREEN GUI =====================
local gui = Instance.new("ScreenGui")
gui.Name = "Caio_hub"
gui.Parent = game.CoreGui

-- Ícone arrastável "C"
local icon = Instance.new("TextButton")
icon.Size = UDim2.new(0,60,0,60)
icon.Position = UDim2.new(0.05,0,0.3,0)
icon.BackgroundColor3 = Color3.fromRGB(135,206,250)
icon.Text = "C"
icon.Font = Enum.Font.Arcade
icon.TextSize = 28
icon.TextColor3 = Color3.fromRGB(255,255,255)
icon.TextScaled = true
icon.AutoButtonColor = true
icon.Active = true
icon.Draggable = true
addUICorner(icon, 30)
addUIStroke(icon, Color3.fromRGB(0,191,255), 3)
icon.Parent = gui

-- Menu principal
local main = Instance.new("Frame")
main.Size = UDim2.new(0, 280, 0, 180)
main.Position = UDim2.new(0.15,0,0.3,0)
main.BackgroundColor3 = Color3.fromRGB(135,206,250)
main.Active = true
main.Draggable = true
main.Visible = false
addUICorner(main, 20)
addUIStroke(main, Color3.fromRGB(0,191,255), 3)
main.Parent = gui

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,50)
title.BackgroundTransparency = 1
title.Text = "Caio_hub"
title.Font = Enum.Font.Arcade
title.TextSize = 28
title.TextColor3 = Color3.fromRGB(255,255,255)
title.TextScaled = true
title.Parent = main

-- Função para criar botões
local function createButton(name, posY)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1,-40,0,45)
    btn.Position = UDim2.new(0,20,0,posY)
    btn.BackgroundColor3 = Color3.fromRGB(135,206,250)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.Font = Enum.Font.Arcade
    btn.TextSize = 20
    btn.Text = name
    addUICorner(btn, 15)
    addUIStroke(btn, Color3.fromRGB(0,191,255), 2)
    btn.Parent = main
    return btn
end

-- Alterna menu
icon.MouseButton1Click:Connect(function()
    main.Visible = not main.Visible
end)

-- ===================== BOTÃO 1: Plataforma =====================
local plataformaBtn = createButton("Plataforma",70)
plataformaBtn.MouseButton1Click:Connect(function()
    plataformaAtiva = not plataformaAtiva
    local character = player.Character
    if plataformaAtiva then
        plataformaBtn.BackgroundColor3 = Color3.fromRGB(0,255,0)
        plataforma = Instance.new("Part")
        plataforma.Size = Vector3.new(6,1,6)
        plataforma.Anchored = true
        plataforma.Color = Color3.fromRGB(0,191,255)
        plataforma.Material = Enum.Material.Neon
        plataforma.Parent = workspace
        plataformaConnection = RunService.RenderStepped:Connect(function()
            if plataforma and character and character:FindFirstChild("HumanoidRootPart") then
                plataforma.CFrame = character.HumanoidRootPart.CFrame * CFrame.new(0,-4,0)
            end
        end)
    else
        plataformaBtn.BackgroundColor3 = Color3.fromRGB(135,206,250)
        if plataforma then plataforma:Destroy() plataforma=nil end
        if plataformaConnection then plataformaConnection:Disconnect() plataformaConnection=nil end
    end
end)

-- ===================== BOTÃO 2: ESP Player =====================
local espBtn = createButton("ESP Player",130)
espBtn.MouseButton1Click:Connect(function()
    espAtivo = not espAtivo
    local function aplicarESP()
        for _,plr in pairs(Players:GetPlayers()) do
            if plr.Character and plr.Character:FindFirstChild("Head") then
                if not plr.Character.Head:FindFirstChild("ESP") then
                    local bill = Instance.new("BillboardGui")
                    bill.Name = "ESP"
                    bill.Size = UDim2.new(0,200,0,50)
                    bill.AlwaysOnTop = true
                    bill.Adornee = plr.Character.Head
                    bill.Parent = plr.Character.Head
                    local txt = Instance.new("TextLabel",bill)
                    txt.Size = UDim2.new(1,0,1,0)
                    txt.BackgroundTransparency = 1
                    txt.Text = plr.Name
                    txt.Font = Enum.Font.Arcade
                    txt.TextSize = 14
                    txt.TextColor3 = Color3.fromRGB(0,255,0)
                end
            end
        end
    end
    if espAtivo then
        espBtn.BackgroundColor3 = Color3.fromRGB(0,255,0)
        aplicarESP()
    else
        espBtn.BackgroundColor3 = Color3.fromRGB(135,206,250)
        for _,plr in pairs(Players:GetPlayers()) do
            if plr.Character and plr.Character:FindFirstChild("Head") then
                local esp = plr.Character.Head:FindFirstChild("ESP")
                if esp then esp:Destroy() end
            end
        end
    end
end)

-- ===================== ANTI-MORTE =====================
local function antiMorte(char)
    local humanoid = char:WaitForChild("Humanoid")
    local hrp = char:WaitForChild("HumanoidRootPart")
    humanoid.HealthChanged:Connect(function()
        if humanoid.Health <= 0 then
            humanoid.Health = humanoid.MaxHealth
            hrp.CFrame = hrp.CFrame + Vector3.new(0,5,0)
        end
    end)
end

-- Reconecta sempre que o personagem renasce
player.CharacterAdded:Connect(function(char)
    antiMorte(char)

    -- Reaplicar plataforma se estava ativa
    if plataformaAtiva then
        if plataformaConnection then plataformaConnection:Disconnect() end
        plataformaConnection = RunService.RenderStepped:Connect(function()
            if plataforma and char and char:FindFirstChild("HumanoidRootPart") then
                plataforma.CFrame = char.HumanoidRootPart.CFrame * CFrame.new(0,-4,0)
            end
        end)
    end

    -- Reaplicar ESP se estava ativo
    if espAtivo then
        for _,plr in pairs(Players:GetPlayers()) do
            if plr.Character and plr.Character:FindFirstChild("Head") then
                if not plr.Character.Head:FindFirstChild("ESP") then
                    local bill = Instance.new("BillboardGui")
                    bill.Name = "ESP"
                    bill.Size = UDim2.new(0,200,0,50)
                    bill.AlwaysOnTop = true
                    bill.Adornee = plr.Character.Head
                    bill.Parent = plr.Character.Head
                    local txt = Instance.new("TextLabel",bill)
                    txt.Size = UDim2.new(1,0,1,0)
                    txt.BackgroundTransparency = 1
                    txt.Text = plr.Name
                    txt.Font = Enum.Font.Arcade
                    txt.TextSize = 14
                    txt.TextColor3 = Color3.fromRGB(0,255,0)
                end
            end
        end
    end
end)

-- Chamada inicial para o personagem atual
if player.Character then
    antiMorte(player.Character)
end
