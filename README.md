-- เปิด/ปิด Aimbot
_G.Aimbot = true

-- ระยะเล็งเป้าหมาย
local aimDistance = 50 -- ระยะในหน่วยเกม

-- ฟังก์ชันค้นหาเป้าหมายที่ใกล้ที่สุดในระยะ
function findClosestTarget()
    local player = game.Players.LocalPlayer
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end

    local closestTarget = nil
    local closestDistance = aimDistance

    -- ค้นหาผู้เล่นที่ใกล้ที่สุด
    for _, target in pairs(game.Players:GetPlayers()) do
        if target ~= player and target.Character and target.Character:FindFirstChild("Head") then
            local targetHead = target.Character.Head
            local distance = (root.Position - targetHead.Position).Magnitude

            -- ตรวจสอบเงื่อนไขว่าเป้าหมายอยู่ในระยะที่กำหนดและมีสุขภาพที่ดี
            if distance < closestDistance and target.Character:FindFirstChild("Humanoid").Health > 0 then
                closestDistance = distance
                closestTarget = targetHead
            end
        end
    end

    return closestTarget
end

-- ฟังก์ชันเล็งเป้าหมายไปที่หัว
function aimAt(target)
    if not target then return end
    local camera = workspace.CurrentCamera
    -- เปลี่ยน CFrame ของกล้องเพื่อเล็งไปที่หัวของเป้าหมาย
    camera.CFrame = CFrame.new(camera.CFrame.Position, target.Position)
end

-- ฟังก์ชันโจมตีเป้าหมาย
function attackTarget()
    local VirtualInputManager = game:GetService("VirtualInputManager")
    VirtualInputManager:SendKeyEvent(true, "MouseButton1", false, game)
    wait(0.1)
    VirtualInputManager:SendKeyEvent(false, "MouseButton1", false, game)
    wait(0.3)
end

-- ฟังก์ชันตรวจจับการโจมตีหรือเมื่ออยู่ใกล้เป้าหมาย
local function detectProximityOrHit()
    spawn(function()
        while _G.Aimbot do
            local player = game.Players.LocalPlayer
            local char = player.Character
            local humanoid = char and char:FindFirstChild("Humanoid")

            if humanoid and humanoid.Health > 0 then
                -- ค้นหาเป้าหมายใกล้ที่สุด
                local target = findClosestTarget()

                if target then
                    if _G.Aimbot then
                        aimAt(target) -- ล็อคไปที่หัวของเป้าหมาย
                    end

                    -- เรียกใช้การโจมตีเมื่อเข้าใกล้เป้าหมาย
                    if (char.HumanoidRootPart.Position - target.Position).Magnitude <= aimDistance then
                        attackTarget()
                    end
                end
            end

            wait(0.1)
        end
    end)
end

-- ฟังก์ชัน ESP
local function createESP(target)
    -- ตรวจสอบว่ามีเป้าหมายที่เป็นผู้เล่น
    if target and target.Parent and target.Parent:FindFirstChild("Humanoid") then
        local humanoid = target.Parent.Humanoid
        local playerName = target.Parent.Name
        local health = humanoid.Health
        local maxHealth = humanoid.MaxHealth
        local distance = (game.Players.LocalPlayer.Character.HumanoidRootPart.Position - target.Position).Magnitude

        -- สร้าง GUI
        local screenGui = Instance.new("BillboardGui")
        screenGui.Parent = game.CoreGui
        screenGui.Adornee = target.Parent:FindFirstChild("Head")
        screenGui.Size = UDim2.new(0, 120, 0, 60) -- ปรับขนาดของ GUI
        screenGui.StudsOffset = Vector3.new(0, 3, 0)
        screenGui.AlwaysOnTop = true

        -- สร้างกล่องข้อความสำหรับชื่อ
        local playerNameLabel = Instance.new("TextLabel")
        playerNameLabel.Parent = screenGui
        playerNameLabel.Text = playerName
        playerNameLabel.Size = UDim2.new(1, 0, 0.3, 0)
        playerNameLabel.BackgroundTransparency = 1
        playerNameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
        playerNameLabel.TextScaled = true

        -- สร้าง Health Bar
        local healthBarBackground = Instance.new("Frame")
        healthBarBackground.Parent = screenGui
        healthBarBackground.Size = UDim2.new(0.9, 0, 0.2, 0) -- ปรับขนาดของ Health Bar
        healthBarBackground.Position = UDim2.new(0.05, 0, 0.3, 0)
        healthBarBackground.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        healthBarBackground.BackgroundTransparency = 0.6

        -- สร้าง Health Bar ที่แสดงตามสุขภาพ
        local healthBar = Instance.new("Frame")
        healthBar.Parent = healthBarBackground
        healthBar.Size = UDim2.new(health / maxHealth, 0, 1, 0) -- ขนาดจะเปลี่ยนตามสุขภาพ
        -- เปลี่ยนสี Health Bar
        if health / maxHealth > 0.7 then
            healthBar.BackgroundColor3 = Color3.fromRGB(0, 255, 0)  -- สีเขียวเมื่อมีสุขภาพดี
        elseif health / maxHealth > 0.3 then
            healthBar.BackgroundColor3 = Color3.fromRGB(255, 255, 0)  -- สีเหลืองเมื่อสุขภาพลด
        else
            healthBar.BackgroundColor3 = Color3.fromRGB(255, 0, 0)  -- สีแดงเมื่อสุขภาพต่ำ
        end
    end
end

-- เรียกใช้ฟังก์ชัน ESP เมื่อเริ่ม
spawn(function()
    while true do
        for _, target in pairs(game.Players:GetPlayers()) do
            if target ~= game.Players.LocalPlayer then
                local targetHead = target.Character and target.Character:FindFirstChild("Head")
                if targetHead then
                    createESP(targetHead)
                end
            end
        end
        wait(1)
    end
end)

-- เรียกใช้ฟังก์ชันตรวจจับการโจมตีและเข้าใกล้เป้าหมาย
detectProximityOrHit()

-- หยุด Aimbot
function stopAimbot()
    _G.Aimbot = false
end
