local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- ฟังก์ชันหาเป้าหมายที่ใกล้ที่สุด
local function findClosestPlayer()
    local closestPlayer = nil
    local shortestDistance = math.huge

    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local otherHumanoidRootPart = otherPlayer.Character.HumanoidRootPart
            local distance = (humanoidRootPart.Position - otherHumanoidRootPart.Position).Magnitude
            if distance < shortestDistance then
                closestPlayer = otherPlayer
                shortestDistance = distance
            end
        end
    end

    return closestPlayer, shortestDistance
end

-- ฟังก์ชัน Ultra Instinct
local function ultraInstinct()
    local closestPlayer, distance = findClosestPlayer()

    if closestPlayer and distance <= 3 then -- กำหนดระยะที่สามารถหลบได้ (20 หน่วย)
        local targetRootPart = closestPlayer.Character.HumanoidRootPart
        local behindPosition = targetRootPart.Position - (targetRootPart.CFrame.LookVector * 10)

        -- Teleport ตัวละครไปด้านหลังเป้าหมาย
        humanoidRootPart.CFrame = CFrame.new(behindPosition)

        -- อนิเมชันหรือเอฟเฟกต์เสริม (ใส่ได้ตามต้องการ)
        print("Ultra Instinct Activated!")
    else
        print("No player nearby to dodge.")
    end
end

-- เรียกใช้ฟังก์ชันเมื่อถูกโจมตี (ตัวอย่างใช้ RunService ตรวจจับ)
RunService.Stepped:Connect(function()
    -- คุณสามารถแทนเงื่อนไขนี้ด้วยระบบการตรวจจับการโจมตีในเกม
    local isUnderAttack = math.random() > 0.95 -- ตัวอย่างการจำลองการถูกโจมตี
    if isUnderAttack then
        ultraInstinct()
    end
end)

-- เปิด/ปิด ESP, Anti-Stun และ God Mode
_G.ESPEnabled = true
_G.AntiStun = true
_G.GodMode = true

-- ระยะการแสดงผล ESP
local ESPDistance = 300

-- ฟังก์ชันสร้าง UI ของ ESP
local function createESP(player)
    local billboard = Instance.new("BillboardGui")
    local nameLabel = Instance.new("TextLabel")
    local healthBar = Instance.new("Frame")
    local healthBackground = Instance.new("Frame")

    billboard.Name = "ESP"
    billboard.Adornee = player.Character:WaitForChild("HumanoidRootPart")
    billboard.Size = UDim2.new(0, 100, 0, 50)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true

    -- ชื่อผู้เล่น
    nameLabel.Parent = billboard
    nameLabel.Size = UDim2.new(1, 0, 0.5, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = player.Name
    nameLabel.TextColor3 = Color3.new(1, 1, 1)
    nameLabel.TextStrokeTransparency = 0
    nameLabel.Font = Enum.Font.SourceSansBold
    nameLabel.TextSize = 14

    -- กรอบพื้นหลัง Health Bar
    healthBackground.Parent = billboard
    healthBackground.Size = UDim2.new(1, 0, 0.2, 0)
    healthBackground.Position = UDim2.new(0, 0, 0.5, 0)
    healthBackground.BackgroundColor3 = Color3.new(0, 0, 0)

    -- Health Bar
    healthBar.Parent = healthBackground
    healthBar.Size = UDim2.new(1, 0, 1, 0)
    healthBar.Position = UDim2.new(0, 0, 0, 0)
    healthBar.BackgroundColor3 = Color3.new(0, 1, 0)

    return billboard, healthBar
end

-- ฟังก์ชันอัปเดต ESP
local function updateESP()
    while _G.ESPEnabled do
        for _, player in pairs(game.Players:GetPlayers()) do
            if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local humanoid = player.Character:FindFirstChild("Humanoid")
                local rootPart = player.Character:FindFirstChild("HumanoidRootPart")
                local billboard = player.Character:FindFirstChild("ESP")

                if not billboard then
                    billboard, healthBar = createESP(player)
                    billboard.Parent = player.Character
                end

                -- อัปเดตข้อมูล
                local distance = math.floor((game.Players.LocalPlayer.Character.HumanoidRootPart.Position - rootPart.Position).Magnitude)
                billboard.TextLabel.Text = string.format("%s\nDistance: %d", player.Name, distance)

                -- อัปเดตแถบสุขภาพ
                if humanoid then
                    healthBar.Size = UDim2.new(humanoid.Health / humanoid.MaxHealth, 0, 1, 0)
                    if humanoid.Health / humanoid.MaxHealth > 0.5 then
                        healthBar.BackgroundColor3 = Color3.new(0, 1, 0)
                    else
                        healthBar.BackgroundColor3 = Color3.new(1, 0, 0)
                    end
                end
            end
        end
        wait(0.1)
    end
end

-- ฟังก์ชันลบ ESP เมื่อปิดใช้งาน
local function removeESP()
    for _, player in pairs(game.Players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("ESP") then
            player.Character.ESP:Destroy()
        end
    end
end

-- ฟังก์ชันกันสตัน
local function preventStun()
    spawn(function()
        while _G.AntiStun do
            local player = game.Players.LocalPlayer
            local char = player.Character

            if char then
                for _, debuff in pairs(char:GetChildren()) do
                    if debuff:IsA("BoolValue") and debuff.Name:lower():find("stun") then
                        debuff:Destroy()
                    end
                end
            end
            wait(0.1)
        end
    end)
end

-- เริ่มระบบ ESP, Anti-Stun และ God Mode
if _G.ESPEnabled then
    spawn(updateESP)
end

if _G.AntiStun then
    spawn(preventStun)
end

-- ฟังก์ชันปิด ESP
function stopESP()
    _G.ESPEnabled = false
    removeESP()
end

-- ฟังก์ชันปิด Anti-Stun
function stopAntiStun()
    _G.AntiStun = false
end

local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- ฟังก์ชันหาเป้าหมายที่ใกล้ที่สุดหรือกำลังโจมตีเรา
local function findTarget()
    local closestPlayer = nil
    local shortestDistance = math.huge

    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local otherHumanoidRootPart = otherPlayer.Character.HumanoidRootPart
            local distance = (humanoidRootPart.Position - otherHumanoidRootPart.Position).Magnitude

            -- ตรวจสอบระยะทางและเงื่อนไขอื่น (เช่น การพยายามโจมตีเรา)
            local isAttacking = false -- ต้องเชื่อมต่อกับระบบตรวจจับการโจมตีในเกม
            if distance < shortestDistance and (distance <= 50 or isAttacking) then
                closestPlayer = otherPlayer
                shortestDistance = distance
            end
        end
    end

    return closestPlayer
end

-- ฟังก์ชันเล็งเป้าหมาย
local function aimAtTarget(target)
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        local targetPosition = target.Character.HumanoidRootPart.Position
        local aimDirection = (targetPosition - humanoidRootPart.Position).unit
        humanoidRootPart.CFrame = CFrame.new(humanoidRootPart.Position, humanoidRootPart.Position + aimDirection)

        -- เพิ่มความแม่นยำ เช่นการโจมตี
        print("Aiming at:", target.Name)
    else
        print("No valid target found.")
    end
end

-- เรียกใช้ฟังก์ชัน Aim Bot
RunService.RenderStepped:Connect(function()
    local target = findTarget()
    aimAtTarget(target)
end)
