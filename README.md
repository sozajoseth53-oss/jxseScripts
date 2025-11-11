# jxseScripts
-- "aimbot + fly + lock-on + ESP" (safe, universal: PC & Mobile)
-- WARNING: Visual only. No hitboxes, fly works on PC & Mobile

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- ===== CONFIG =====
local config = {
    ESPColor = Color3.fromRGB(0,255,0),
    FOV = 200,
    MaxRange = 1000,
    FlySpeed = 50,
    AimbotEnabled = false,
    LockOnEnabled = false,
    ShowFOV = true,
    ShowSnapLine = true,
    ShowHeadDot = true,
    ShowDistances = true,
    FOVColor = Color3.fromRGB(255,0,0)
}

-- ===== GUI =====
local playerGui = LocalPlayer:WaitForChild("PlayerGui")
local screenGui = Instance.new("ScreenGui", playerGui)
screenGui.Name = "jxseScriptsGui"
screenGui.ResetOnSpawn = false

local frame = Instance.new("Frame", screenGui)
frame.Size = UDim2.new(0, 320, 0, 220)
frame.Position = UDim2.new(0.02,0,0.25,0)
frame.BackgroundTransparency = 0.15
frame.BackgroundColor3 = Color3.fromRGB(20,20,20)
frame.BorderSizePixel = 0
frame.Active = true
frame.Draggable = true

-- ===== TITLE =====
local titleBox = Instance.new("Frame", frame)
titleBox.Size = UDim2.new(0,120,0,24)
titleBox.Position = UDim2.new(0,10,0,5)
titleBox.BackgroundColor3 = Color3.fromRGB(50,50,50)
titleBox.BorderSizePixel = 0

local titleLabel = Instance.new("TextLabel", titleBox)
titleLabel.Size = UDim2.new(1,0,1,0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "jxseScripts"
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 14
titleLabel.TextColor3 = Color3.fromRGB(255,0,0)
titleLabel.TextXAlignment = Enum.TextXAlignment.Center
titleLabel.TextYAlignment = Enum.TextYAlignment.Center

local thankLabel = Instance.new("TextLabel", frame)
thankLabel.Size = UDim2.new(0,200,0,18)
thankLabel.Position = UDim2.new(0,10,0,30)
thankLabel.BackgroundTransparency = 1
thankLabel.Text = "Gracias por usar mi script"
thankLabel.TextColor3 = Color3.fromRGB(255,0,0)
thankLabel.Font = Enum.Font.GothamBold
thankLabel.TextSize = 14
thankLabel.TextXAlignment = Enum.TextXAlignment.Left
thankLabel.TextYAlignment = Enum.TextYAlignment.Top

-- Rainbow title
local hue = 0
RunService.Heartbeat:Connect(function(dt)
    hue = (hue + dt/2) % 1
    titleLabel.TextColor3 = Color3.fromHSV(hue,1,1)
end)

-- ===== BUTTONS =====
local function makeButton(x,y,w,h,text)
    local b = Instance.new("TextButton", frame)
    b.Size = UDim2.new(0,w,0,h)
    b.Position = UDim2.new(0,x,0,y)
    b.Text = text
    b.Font = Enum.Font.GothamBold
    b.TextSize = 14
    b.BackgroundColor3 = Color3.fromRGB(45,45,45)
    b.TextColor3 = Color3.fromRGB(255,255,255)
    return b
end

local toggleMenuBtn = makeButton(10,5,20,20,"-")
local flyBtn = makeButton(40,5,60,20,"Fly: OFF")
local aimbotBtn = makeButton(110,5,80,20,"Aimbot: OFF")
local fovBtn = makeButton(200,5,50,20,"FOV")
local snapBtn = makeButton(260,5,50,20,"Snap")
local headDotBtn = makeButton(10,35,80,20,"Head Dot")
local distBtn = makeButton(100,35,80,20,"Distances")
local lockOnBtn = makeButton(200,35,80,20,"Lock-On: OFF")

local menuVisible = true
toggleMenuBtn.MouseButton1Click:Connect(function()
    menuVisible = not menuVisible
    for _, obj in pairs(frame:GetChildren()) do
        if obj:IsA("TextButton") and obj ~= toggleMenuBtn then
            obj.Visible = menuVisible
        end
    end
    toggleMenuBtn.Text = menuVisible and "-" or "+"
end)

-- ===== FLY =====
local flying = false
local bv,bg
local moveVector = Vector3.new()

flyBtn.MouseButton1Click:Connect(function()
    flying = not flying
    flyBtn.Text = flying and "Fly: ON" or "Fly: OFF"
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        local hrp = char.HumanoidRootPart
        if flying then
            bv = Instance.new("BodyVelocity")
            bv.MaxForce = Vector3.new(1e5,1e5,1e5)
            bv.Velocity = Vector3.new(0,0,0)
            bv.P = 1e4
            bv.Parent = hrp

            bg = Instance.new("BodyGyro")
            bg.MaxTorque = Vector3.new(1e5,1e5,1e5)
            bg.CFrame = hrp.CFrame
            bg.Parent = hrp
        else
            if bv then bv:Destroy() end
            if bg then bg:Destroy() end
        end
    end
end)

-- PC movement
UserInputService.InputBegan:Connect(function(input,gp)
    if flying and not gp then
        if input.KeyCode == Enum.KeyCode.W then moveVector = Vector3.new(0,0,-1) end
        if input.KeyCode == Enum.KeyCode.S then moveVector = Vector3.new(0,0,1) end
        if input.KeyCode == Enum.KeyCode.A then moveVector = Vector3.new(-1,0,0) end
        if input.KeyCode == Enum.KeyCode.D then moveVector = Vector3.new(1,0,0) end
        if input.KeyCode == Enum.KeyCode.Space then moveVector = Vector3.new(0,1,0) end
        if input.KeyCode == Enum.KeyCode.LeftControl then moveVector = Vector3.new(0,-1,0) end
    end
end)

UserInputService.InputEnded:Connect(function(input,gp)
    if flying and not gp then moveVector = Vector3.new() end
end)

-- ===== AIMBOT & LOCK-ON =====
aimbotBtn.MouseButton1Click:Connect(function()
    config.AimbotEnabled = not config.AimbotEnabled
    aimbotBtn.Text = config.AimbotEnabled and "Aimbot: ON" or "Aimbot: OFF"
end)

lockOnBtn.MouseButton1Click:Connect(function()
    config.LockOnEnabled = not config.LockOnEnabled
    lockOnBtn.Text = config.LockOnEnabled and "Lock-On: ON" or "Lock-On: OFF"
end)

-- ===== DRAWING ESP & FOV =====
local successDrawing, Drawing = pcall(function() return Drawing end)
local FOVCircle, SnapLine, HeadDot, MarkCircle

if successDrawing then
    FOVCircle = Drawing.new("Circle")
    FOVCircle.Thickness = 2
    FOVCircle.NumSides = 64
    FOVCircle.Radius = config.FOV
    FOVCircle.Color = config.FOVColor
    FOVCircle.Visible = config.ShowFOV

    SnapLine = Drawing.new("Line")
    SnapLine.Thickness = 2
    SnapLine.Visible = config.ShowSnapLine

    HeadDot = Drawing.new("Circle")
    HeadDot.Thickness = 2
    HeadDot.NumSides = 24
    HeadDot.Radius = 5
    HeadDot.Visible = config.ShowHeadDot

    MarkCircle = Drawing.new("Circle")
    MarkCircle.Thickness = 2
    MarkCircle.NumSides = 36
    MarkCircle.Radius = 12
    MarkCircle.Visible = false
end

-- ===== MAIN LOOP =====
RunService.Heartbeat:Connect(function()
    local char = LocalPlayer.Character
    if flying and char and char:FindFirstChild("HumanoidRootPart") and bv and bg then
        local camCF = Camera.CFrame
        local dir = (camCF.LookVector*moveVector.Z)+(camCF.RightVector*moveVector.X)+Vector3.new(0,moveVector.Y,0)
        bv.Velocity = dir * config.FlySpeed
        bg.CFrame = CFrame.new(char.HumanoidRootPart.Position,char.HumanoidRootPart.Position+camCF.LookVector)
    end

    if successDrawing then
        local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
        if FOVCircle then
            FOVCircle.Position = center
            FOVCircle.Radius = config.FOV
            FOVCircle.Color = config.FOVColor
            FOVCircle.Visible = config.ShowFOV
        end

        for _,player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local head = player.Character:FindFirstChild("Head")
                if head then
                    local sp, onScreen = Camera:WorldToViewportPoint(head.Position)
                    if onScreen then
                        if SnapLine then
                            SnapLine.From = center
                            SnapLine.To = Vector2.new(sp.X, sp.Y)
                            SnapLine.Color = config.ESPColor
                            SnapLine.Visible = config.ShowSnapLine
                        end
                        if HeadDot then
                            HeadDot.Position = Vector2.new(sp.X, sp.Y)
                            HeadDot.Color = config.ESPColor
                            HeadDot.Visible = config.ShowHeadDot
                        end
                    end
                end
            end
        end

        -- Lock-On
        if config.AimbotEnabled and config.LockOnEnabled then
            local closest, shortest = nil, math.huge
            local viewportCenter = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
                    local head = player.Character.Head
                    local dist = (head.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
                    local screenPoint, onScreen = Camera:WorldToViewportPoint(head.Position)
                    if onScreen and dist<shortest and (Vector2.new(screenPoint.X,screenPoint.Y)-viewportCenter).Magnitude <= config.FOV then
                        closest = player
                        shortest = dist
                    end
                end
            end
            if closest and closest.Character then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position,closest.Character.Head.Position)
            end
        end
    end
end)
