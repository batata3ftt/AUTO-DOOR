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

local Workspace = game:GetService("Workspace")

local trackedTraps = {} local ignoreConnection = nil

local function createHighlight(object) if not object then return end if object:FindFirstChild("ESPHighlight") then return end local h = Instance.new("Highlight") h.Name = "ESPHighlight" h.Adornee = object h.FillColor = Color3.fromRGB(255, 0, 0) h.OutlineColor = Color3.fromRGB(255, 0, 0) h.FillTransparency = 0.5 h.OutlineTransparency = 0 h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop h.Parent = object end

local function removeHighlight(object) if not object then return end local h = object:FindFirstChild("ESPHighlight") if h then h:Destroy() end end

local function createNameLabel(object) if not object then return end if object:FindFirstChild("ESPNameLabel") then return end local anchorPart = object.PrimaryPart or object:FindFirstChildWhichIsA("BasePart") if not anchorPart then return end local billboard = Instance.new("BillboardGui") billboard.Name = "ESPNameLabel" billboard.Adornee = anchorPart billboard.AlwaysOnTop = true billboard.Size = UDim2.new(0, 140, 0, 24) billboard.StudsOffset = Vector3.new(0, 3, 0) billboard.ResetOnSpawn = false billboard.Parent = object local label = Instance.new("TextLabel") label.Size = UDim2.new(1, 0, 1, 0) label.BackgroundTransparency = 1 label.Text = "Trap" label.TextColor3 = Color3.fromRGB(255, 0, 0) label.TextSize = 14 label.Font = Enum.Font.GothamBold label.TextStrokeTransparency = 0.4 label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0) label.Parent = billboard end

local function removeNameLabel(object) if not object then return end local b = object:FindFirstChild("ESPNameLabel") if b then b:Destroy() end end

local function setupTrapESP(obj) if not obj then return end if obj.Name ~= "Trap" then return end if not obj:IsA("Model") then return end if trackedTraps[obj] then return end trackedTraps[obj] = true task.wait(0.05) if not obj or not obj.Parent then trackedTraps[obj] = nil; return end createHighlight(obj) createNameLabel(obj) obj.AncestryChanged:Connect(function() if not obj.Parent then trackedTraps[obj] = nil removeHighlight(obj) removeNameLabel(obj) end end) end

local function connectIgnoreFolder(ignoreFolder) for _, obj in pairs(ignoreFolder:GetChildren()) do if obj.Name == "Trap" and obj:IsA("Model") then setupTrapESP(obj) end end if ignoreConnection then ignoreConnection:Disconnect() end ignoreConnection = ignoreFolder.ChildAdded:Connect(function(obj) if obj.Name == "Trap" and obj:IsA("Model") then task.wait(0.05) setupTrapESP(obj) end end) end

local function monitorTraps() task.spawn(function() local waited = 0 while waited < 30 do local ignore = Workspace:FindFirstChild("IGNORE") if ignore then connectIgnoreFolder(ignore); return end task.wait(0.5) waited = waited + 0.5 end Workspace.ChildAdded:Connect(function(child) if child.Name == "IGNORE" then connectIgnoreFolder(child) end end) end) end

local function resetESP() trackedTraps = {} if ignoreConnection then ignoreConnection:Disconnect(); ignoreConnection = nil end for _, obj in pairs(Workspace:GetDescendants()) do pcall(removeHighlight, obj) pcall(removeNameLabel, obj) end end

local function watchForRound() local mapsFolder = Workspace:WaitForChild("MAPS", 30) if not mapsFolder then task.delay(5, watchForRound); return end if mapsFolder:FindFirstChild("GAME MAP") then task.wait(2); monitorTraps() end mapsFolder.ChildAdded:Connect(function(child) if child.Name == "GAME MAP" then resetESP(); task.wait(2); monitorTraps() end end) mapsFolder.ChildRemoved:Connect(function(child) if child.Name == "GAME MAP" then resetESP() end end) end

watchForRound()
