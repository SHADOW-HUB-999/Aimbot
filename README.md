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
-- ฟังก์ชัน God Mode แบบปรับปรุง
local function enableGodMode()
    spawn(function()
        while _G.GodMode do
            local player = game.Players.LocalPlayer
            local char = player.Character
            local humanoid = char and char:FindFirstChild("Humanoid")

            if humanoid then
                -- ล็อคค่า Health ให้เท่ากับ MaxHealth เสมอ
                humanoid.Health = humanoid.MaxHealth
                humanoid:GetPropertyChangedSignal("Health"):Connect(function()
                    if humanoid.Health < humanoid.MaxHealth then
                        humanoid.Health = humanoid.MaxHealth
                    end
                end)

                -- ลบสคริปต์ที่อาจลด Health
                for _, obj in pairs(char:GetChildren()) do
                    if obj:IsA("Script") or obj:IsA("LocalScript") then
                        if obj.Name:lower():find("damage") or obj.Name:lower():find("hurt") then
                            obj:Destroy()
                        end
                    end
                end

                -- ตรวจจับ RemoteEvent ที่อาจลด Health
                for _, remote in pairs(game.ReplicatedStorage:GetChildren()) do
                    if remote:IsA("RemoteEvent") or remote:IsA("RemoteFunction") then
                        remote.OnClientEvent:Connect(function(...)
                            -- บล็อกข้อมูลที่อาจลด Health
                            humanoid.Health = humanoid.MaxHealth
                        end)
                    end
                end
            end

            wait(0.1)
        end
    end)
end

-- เปิดใช้งาน God Mode
if _G.GodMode then
    enableGodMode()
end

-- เริ่มระบบ ESP, Anti-Stun และ God Mode
if _G.ESPEnabled then
    spawn(updateESP)
end

if _G.AntiStun then
    spawn(preventStun)
end

if _G.GodMode then
    enableGodMode()
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

-- ฟังก์ชันปิด God Mode
function stopGodMode()
    _G.GodMode = false
end
