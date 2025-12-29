-- Kavo UI
local Library = loadstring(game:HttpGet("https://raw.githubusercontent.com/xHeptc/Kavo-UI-Library/main/source.lua"))()
local Window = Library.CreateLib("Luca Hub", "BloodTheme")

-- Abas
local FuncoesTab = Window:NewTab("Funções")
local TeleportsTab = Window:NewTab("Teleports")
local GraphicsTab = Window:NewTab("Graphics")
local VisualsTab = Window:NewTab("Visuals")

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ================= FUNÇÕES =================
local FuncoesSection = FuncoesTab:NewSection("Funções")

-- INVISIBLE (NÃO ALTERADO)
FuncoesSection:NewButton("Invisible", "Ativa Invisible (Keybind + Mobile Button)", function()
    if _G.__INVIS_CONN then
        for _, c in pairs(_G.__INVIS_CONN) do
            pcall(function() c:Disconnect() end)
        end
    end
    _G.__INVIS_CONN = {}

    local Players = game:GetService("Players")
    local UIS = game:GetService("UserInputService")
    local RunService = game:GetService("RunService")
    local StarterGui = game:GetService("StarterGui")

    local player = Players.LocalPlayer
    repeat task.wait() until player.Character

    local character, humanoid, rootPart
    local invisible = false
    local parts = {}

    local isMobile = UIS.TouchEnabled and not UIS.KeyboardEnabled
    local gameIcon = "rbxthumb://type=GameIcon&id=" .. game.PlaceId .. "&w=150&h=150"

    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = "by insonia",
            Text = "Invisible injetado com sucesso!",
            Icon = gameIcon,
            Duration = 3
        })
    end)

    local function setupCharacter()
        character = player.Character or player.CharacterAdded:Wait()
        humanoid = character:WaitForChild("Humanoid")
        rootPart = character:WaitForChild("HumanoidRootPart")
        parts = {}
        for _, obj in pairs(character:GetDescendants()) do
            if obj:IsA("BasePart") and obj.Transparency == 0 then
                table.insert(parts, obj)
            end
        end
    end

    local function setInvisible(state)
        invisible = state
        for _, part in pairs(parts) do
            part.Transparency = invisible and 0.5 or 0
        end
    end

    UIS.InputBegan:Connect(function(input, gp)
        if gp then return end
        if input.KeyCode == Enum.KeyCode.X then
            setInvisible(not invisible)
        end
    end)

    table.insert(_G.__INVIS_CONN,
        RunService.Heartbeat:Connect(function()
            if invisible and rootPart and humanoid then
                local cf = rootPart.CFrame
                local camOffset = humanoid.CameraOffset
                local hidden = cf * CFrame.new(0, -200, 0)
                rootPart.CFrame = hidden
                humanoid.CameraOffset =
                    hidden:ToObjectSpace(CFrame.new(cf.Position)).Position
                RunService.RenderStepped:Wait()
                rootPart.CFrame = cf
                humanoid.CameraOffset = camOffset
            end
        end)
    )

    setupCharacter()
end)

-- ================= HITBOX =================
local HitboxEnabled = false
local OriginalSizes = {}
local BIG_SIZE = Vector3.new(10,10,10)

local function applyHitbox(state)
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = p.Character.HumanoidRootPart
            if state then
                OriginalSizes[p] = OriginalSizes[p] or hrp.Size
                hrp.Size = BIG_SIZE
                hrp.Transparency = 1
                hrp.CanCollide = false
            else
                if OriginalSizes[p] then
                    hrp.Size = OriginalSizes[p]
                end
            end
        end
    end
end

FuncoesSection:NewButton("Hitbox Expander", "Hitbox grande invisível", function()
    HitboxEnabled = not HitboxEnabled
    applyHitbox(HitboxEnabled)
end)

-- ================= ANTI RAGDOLL =================
local AntiRagdoll = false
local AR_Conn

FuncoesSection:NewButton("Anti Ragdoll", "Bloqueia quedas e stun", function()
    AntiRagdoll = not AntiRagdoll

    if AR_Conn then AR_Conn:Disconnect() end
    if not AntiRagdoll then return end

    AR_Conn = RunService.Heartbeat:Connect(function()
        local char = LocalPlayer.Character
        if not char then return end
        local hum = char:FindFirstChildOfClass("Humanoid")
        if not hum then return end

        hum.PlatformStand = false
        hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
        hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)

        if hum:GetState() == Enum.HumanoidStateType.Ragdoll
        or hum:GetState() == Enum.HumanoidStateType.FallingDown then
            hum:ChangeState(Enum.HumanoidStateType.GettingUp)
        end
    end)
end)

-- ================= NOCLIP (PAINEL) =================
local Noclip = false
local NC_Conn

FuncoesSection:NewButton("Noclip", "Atravessa paredes", function()
    Noclip = not Noclip

    if NC_Conn then NC_Conn:Disconnect() end

    if Noclip then
        NC_Conn = RunService.Stepped:Connect(function()
            if LocalPlayer.Character then
                for _, v in pairs(LocalPlayer.Character:GetDescendants()) do
                    if v:IsA("BasePart") then
                        v.CanCollide = false
                    end
                end
            end
        end)
    else
        if LocalPlayer.Character then
            for _, v in pairs(LocalPlayer.Character:GetDescendants()) do
                if v:IsA("BasePart") then
                    v.CanCollide = true
                end
            end
        end
    end
end)

-- ================= GRAPHICS =================
local GraphicsSection = GraphicsTab:NewSection("Graphics")

do
    local Lighting = game:GetService("Lighting")
    local night = false
    local original = {
        Brightness = Lighting.Brightness,
        ClockTime = Lighting.ClockTime,
        Ambient = Lighting.Ambient,
        OutdoorAmbient = Lighting.OutdoorAmbient
    }

    GraphicsSection:NewButton("Night", "Modo noite", function()
        night = not night
        if night then
            Lighting.Brightness = 1
            Lighting.ClockTime = 0
            Lighting.Ambient = Color3.fromRGB(80,80,80)
            Lighting.OutdoorAmbient = Color3.fromRGB(60,60,60)
        else
            for i,v in pairs(original) do
                Lighting[i] = v
            end
        end
    end)
end

-- RESOLUTION
do
    local conn
    local function setRes(v)
        if conn then conn:Disconnect() end
        conn = RunService.RenderStepped:Connect(function()
            Camera.CFrame = Camera.CFrame *
                CFrame.new(0,0,0, 1,0,0, 0,v,0, 0,0,1)
        end)
    end

    GraphicsSection:NewButton("Tela normal","",function() setRes(1) end)
    GraphicsSection:NewButton("Tela esticada","",function() setRes(0.80) end)
    GraphicsSection:NewButton("Tela muito esticada","",function() setRes(0.70) end)
end

-- ================= VISUALS (ESP) =================
local VisualsSection = VisualsTab:NewSection("ESP")

local ESP_BOX, ESP_NAME, ESP_LINE, ESP_HP = false,false,false,false
local drawings = {}

local function clearESP()
    for _,v in pairs(drawings) do pcall(function() v:Remove() end) end
    table.clear(drawings)
end

local function alive(char)
    local h = char:FindFirstChildOfClass("Humanoid")
    return h and h.Health > 0
end

RunService.RenderStepped:Connect(function()
    if not (ESP_BOX or ESP_NAME or ESP_LINE or ESP_HP) then return end

    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and alive(p.Character) then
            local hrp = p.Character.HumanoidRootPart
            local hum = p.Character:FindFirstChildOfClass("Humanoid")
            local pos, on = Camera:WorldToViewportPoint(hrp.Position)
            if not on then continue end

            local dist = (Camera.CFrame.Position - hrp.Position).Magnitude
            local size = math.clamp(2000/dist, 35, 120)

            if ESP_BOX then
                local b = drawings["b"..p.Name] or Drawing.new("Square")
                b.Size = Vector2.new(size, size*1.5)
                b.Position = Vector2.new(pos.X-b.Size.X/2, pos.Y-b.Size.Y/2)
                b.Color = Color3.fromRGB(255,0,0)
                b.Visible = true
                drawings["b"..p.Name] = b
            end

            if ESP_NAME then
                local t = drawings["n"..p.Name] or Drawing.new("Text")
                t.Text = p.DisplayName or p.Name
                t.Center = true
                t.Outline = true
                t.Color = Color3.fromRGB(255,0,0)
                t.Position = Vector2.new(pos.X, pos.Y-size)
                t.Visible = true
                drawings["n"..p.Name] = t
            end

            if ESP_LINE and LocalPlayer.Character then
                local my = Camera:WorldToViewportPoint(LocalPlayer.Character.HumanoidRootPart.Position)
                local l = drawings["l"..p.Name] or Drawing.new("Line")
                l.From = Vector2.new(my.X,my.Y)
                l.To = Vector2.new(pos.X,pos.Y)
                l.Color = Color3.fromRGB(255,0,0)
                l.Visible = true
                drawings["l"..p.Name] = l
            end

            if ESP_HP and hum then
                local hp = hum.Health/hum.MaxHealth*100
                local col = hp>70 and Color3.fromRGB(0,255,0)
                    or hp>50 and Color3.fromRGB(255,255,0)
                    or hp>20 and Color3.fromRGB(255,165,0)
                    or Color3.fromRGB(255,0,0)

                local h = drawings["h"..p.Name] or Drawing.new("Line")
                h.Color = col
                h.Thickness = 4
                h.From = Vector2.new(pos.X-size/2-6, pos.Y+size/2)
                h.To = Vector2.new(pos.X-size/2-6, pos.Y+size/2-(size*(hp/100)))
                h.Visible = true
                drawings["h"..p.Name] = h
            end
        end
    end
end)

VisualsSection:NewButton("ESP Box","",function() ESP_BOX=not ESP_BOX if not ESP_BOX then clearESP() end end)
VisualsSection:NewButton("ESP Name","",function() ESP_NAME=not ESP_NAME if not ESP_NAME then clearESP() end end)
VisualsSection:NewButton("ESP Line","",function() ESP_LINE=not ESP_LINE if not ESP_LINE then clearESP() end end)
VisualsSection:NewButton("ESP Health","",function() ESP_HP=not ESP_HP if not ESP_HP then clearESP() end end)

VisualsSection:NewSlider("FOV Changer","",120,60,function(v)
    Camera.FieldOfView = v
end)

-- ================= TELEPORTS =================
local TeleportsSection = TeleportsTab:NewSection("Locais")

local Teleports = {
    {"Lobby", CFrame.new(-0.29, 7.03, 0)},
    {"Corda", CFrame.new(-241.92, 106, -170.49)},
    {"Jogo Esconder", CFrame.new(-174.30, 5.00, 200.01)},
    {"Quadrado", CFrame.new(300.91, 88.41, 40.99)},
    {"Triângulo", CFrame.new(300.44, 88.56, -43.61)},
    {"Bola", CFrame.new(300.69, 88.49, -142.86)}
}

for _, tp in pairs(Teleports) do
    TeleportsSection:NewButton(tp[1], "", function()
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            LocalPlayer.Character.HumanoidRootPart.CFrame = tp[2]
        end
    end)
end

-- ================= PLAYER =================

local Player = Window:NewTab("Player")
local PlayerSection = Player:NewSection("Player")
 
PlayerSection:NewSlider("Walkspeed", "Change's your speed", 500, 16, function(s)
    game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = s
end)
 
PlayerSection:NewSlider("Jumppower", "Make's you jump High", 500, 50, function(s)
    game.Players.LocalPlayer.Character.Humanoid.JumpPower = s
end)
 
local Tab = Window:NewTab("Settings")
local Section = Tab:NewSection("Discord: w2x4c")
 
Section:NewKeybind("Keybind", "Open/Close Menu", Enum.KeyCode.V, function()
	Library:ToggleUI(V)
end)
