-- เปิด/ปิด Aimbot
_G.Aimbot = false

-- ระยะเล็งเป้าหมาย (ไม่จำกัด)
local aimDistance = math.huge -- ไม่มีข้อจำกัดระยะ

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

            -- ตรวจสอบว่าเป้าหมายมีสุขภาพที่ดี
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

-- ฟังก์ชันตรวจจับการคลิกขวา
local function onRightClick(input, gameProcessed)
    if gameProcessed then return end
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        _G.Aimbot = true  -- เปิด Aimbot เมื่อคลิกขวา
        detectProximityOrHit() -- เรียกใช้ฟังก์ชันตรวจจับ
    end
end

-- ฟังก์ชันตรวจจับการปล่อยคลิกขวา
local function onRightClickRelease(input)
    if input.UserInputType == Enum.UserInputType.MouseButton2 then
        _G.Aimbot = false  -- ปิด Aimbot เมื่อปล่อยคลิกขวา
    end
end

-- เชื่อมโยงการคลิกขวา
game:GetService("UserInputService").InputBegan:Connect(onRightClick)
game:GetService("UserInputService").InputEnded:Connect(onRightClickRelease)

-- ฟังก์ชัน ESP Box
local function createESPBox(target)
    if target and target.Parent and target.Parent:FindFirstChild("Humanoid") then
        local humanoidRootPart = target.Parent:FindFirstChild("HumanoidRootPart")
        if not humanoidRootPart then return end

        -- สร้าง BillboardGui
        local screenGui = Instance.new("BillboardGui")
        screenGui.Parent = game.CoreGui
        screenGui.Adornee = humanoidRootPart -- ให้กล่องติดตาม HumanoidRootPart
        screenGui.Size = UDim2.new(0, 200, 0, 200) -- ขนาดเริ่มต้นของกล่อง
        screenGui.StudsOffset = Vector3.new(0, 3, 0) -- ตำแหน่งที่ยกขึ้นจากหัว
        screenGui.AlwaysOnTop = true -- ให้กล่องอยู่เหนือทุกอย่าง

        -- ตรวจสอบทีมของผู้เล่น
        local player = game.Players.LocalPlayer
        local teamColor = Color3.fromRGB(255, 0, 0)  -- ค่าเริ่มต้นสีแดงสำหรับศัตรู
        if target.Parent:FindFirstChild("Team") and player.Team == target.Parent.Team then
            teamColor = Color3.fromRGB(0, 255, 0)  -- สีเขียวสำหรับเพื่อน
        end

        -- สร้างกรอบ (Box) สำหรับ ESP
        local box = Instance.new("Frame")
        box.Parent = screenGui
        box.Size = UDim2.new(1, 0, 1, 0) -- ขนาดของกรอบ
        box.BackgroundColor3 = teamColor -- กำหนดสีตามทีม
        box.BackgroundTransparency = 0.7 -- ความโปร่งใสของกรอบ (ควรโปร่งใส)

        -- สร้างกรอบเพิ่มเติมสำหรับความละเอียดของ ESP
        local outline = Instance.new("Frame")
        outline.Parent = box
        outline.Size = UDim2.new(1, 0, 1, 0)
        outline.BorderSizePixel = 2 -- ขนาดขอบ
        outline.BorderColor3 = Color3.fromRGB(255, 255, 255) -- สีขอบ (สีขาว)
        outline.BackgroundTransparency = 1 -- ทำให้กรอบโปร่งใส

        -- แสดงข้อมูลชื่อของผู้เล่น
        local playerNameLabel = Instance.new("TextLabel")
        playerNameLabel.Parent = screenGui
        playerNameLabel.Text = target.Parent.Name
        playerNameLabel.Size = UDim2.new(1, 0, 0.1, 0) -- ขนาดของชื่อ
        playerNameLabel.Position = UDim2.new(0, 0, -0.15, 0) -- ย้ายไปที่ด้านบนของกล่อง
        playerNameLabel.BackgroundTransparency = 1
        playerNameLabel.TextColor3 = Color3.fromRGB(255, 255, 255) -- สีของข้อความ (สีขาว)
        playerNameLabel.TextScaled = true
        playerNameLabel.TextStrokeTransparency = 0.8 -- การปรับความมืดของข้อความ

        -- แสดง Health Bar
        local healthBarBackground = Instance.new("Frame")
        healthBarBackground.Parent = screenGui
        healthBarBackground.Size = UDim2.new(1, 0, 0.05, 0) -- ขนาดของพื้นหลัง Health Bar
        healthBarBackground.Position = UDim2.new(0, 0, 1, 0)
        healthBarBackground.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        healthBarBackground.BackgroundTransparency = 0.5

        -- สร้าง Health Bar ที่แสดงตามสุขภาพ
        local humanoid = target.Parent:FindFirstChild("Humanoid")
        local healthBar = Instance.new("Frame")
        healthBar.Parent = healthBarBackground
        healthBar.Size = UDim2.new(humanoid.Health / humanoid.MaxHealth, 0, 1, 0) -- ขนาดจะเปลี่ยนตามสุขภาพ
        -- เปลี่ยนสี Health Bar
        if humanoid.Health / humanoid.MaxHealth > 0.7 then
            healthBar.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        elseif humanoid.Health / humanoid.MaxHealth > 0.3 then
            healthBar.BackgroundColor3 = Color3.fromRGB(255, 255, 0)
        else
            healthBar.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        end
    end
end

-- เรียกใช้ฟังก์ชัน ESP Box เมื่อเริ่ม
spawn(function()
    while true do
        for _, target in pairs(game.Players:GetPlayers()) do
            if target ~= game.Players.LocalPlayer then
                local targetHead = target.Character and target.Character:FindFirstChild("Head")
                if targetHead then
                    createESPBox(targetHead)
                end
            end
        end
        wait(1)
    end
end)

