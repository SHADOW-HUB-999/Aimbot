-- เปิด/ปิด Aimbot
_G.Aimbot = false

-- ฟังก์ชันเปิด/ปิด Aimbot โดยกดปุ่ม Q
local UserInputService = game:GetService("UserInputService")
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.Q then
        _G.Aimbot = not _G.Aimbot
        print("Aimbot is now " .. (_G.Aimbot and "Enabled" or "Disabled"))
    end
end)

-- ฟังก์ชันค้นหาเป้าหมายที่ใกล้ที่สุดในระยะ
function findClosestTarget()
    local player = game.Players.LocalPlayer
    local char = player.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end

    local closestTarget = nil

    -- ค้นหาผู้เล่นที่ใกล้ที่สุด (ไม่มีข้อจำกัดระยะทาง)
    for _, target in pairs(game.Players:GetPlayers()) do
        if target ~= player and target.Character and target.Character:FindFirstChild("Head") then
            local targetHead = target.Character.Head

            -- ตรวจสอบว่าเป้าหมายมีสุขภาพที่ดี
            if target.Character:FindFirstChild("Humanoid").Health > 0 then
                closestTarget = targetHead
                break -- เล็งไปที่เป้าหมายแรกที่พบ
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
                    attackTarget()
                end
            end

            wait(0.1)
        end
    end)
end

-- ฟังก์ชัน ESP Skeleton
local function createSkeletonESP(character)
    if character and character:FindFirstChild("HumanoidRootPart") then
        -- วาดโครงกระดูก (Skeleton)
        local humanoidRootPart = character.HumanoidRootPart
        local humanoid = character:FindFirstChild("Humanoid")
        
        -- แสดงชื่อ
        local screenGui = Instance.new("BillboardGui")
        screenGui.Parent = game.CoreGui
        screenGui.Adornee = humanoidRootPart
        screenGui.Size = UDim2.new(0, 200, 0, 200)
        screenGui.StudsOffset = Vector3.new(0, 3, 0)
        screenGui.AlwaysOnTop = true

        -- วาดโครงกระดูก (โดยใช้เส้นเชื่อมแต่ละจุดในตัวละคร)
        for _, part in pairs(character:GetChildren()) do
            if part:IsA("MeshPart") and part.Name ~= "HumanoidRootPart" then
                local line = Instance.new("LineHandleAdornment")
                line.Parent = game.CoreGui
                line.Adornee = part
                line.Color3 = Color3.fromRGB(0, 255, 255)
                line.Thickness = 0.1
                line.CFrame = part.CFrame
            end
        end
    end
end

-- ฟังก์ชัน ESP สำหรับผู้เล่นทุกคน
spawn(function()
    while true do
        for _, target in pairs(game.Players:GetPlayers()) do
            if target ~= game.Players.LocalPlayer then
                local targetCharacter = target.Character
                if targetCharacter then
                    createSkeletonESP(targetCharacter) -- สร้าง Skeleton ESP
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
