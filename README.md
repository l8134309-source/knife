-- Server Script: لمس لاعب → يطير + انفجار الماب
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local LIFT_FORCE = 80
local LIFT_TIME = 1
local COOLDOWN = 2
local debounces = {}

-- وظيفة لدفع اللاعب للأعلى
local function makeFly(character)
    local player = Players:GetPlayerFromCharacter(character)
    if not player then return end

    if debounces[player] and tick() - debounces[player] < COOLDOWN then return end
    debounces[player] = tick()

    local hrp = character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end

    local bv = Instance.new("BodyVelocity")
    bv.Name = "FlyBoost_BV"
    bv.MaxForce = Vector3.new(1e5,1e5,1e5)
    bv.Velocity = Vector3.new(0,LIFT_FORCE,0)
    bv.P = 1e4
    bv.Parent = hrp

    delay(LIFT_TIME, function()
        if bv and bv.Parent then bv:Destroy() end
    end)
end

-- وظيفة لتفكيك الماب
local function explodeMap()
    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("BasePart") and not obj.Anchored then
            -- دفع الجزء للأعلى مع دوران عشوائي
            local bv = Instance.new("BodyVelocity")
            bv.MaxForce = Vector3.new(1e5,1e5,1e5)
            bv.Velocity = Vector3.new(math.random(-50,50),math.random(50,150),math.random(-50,50))
            bv.P = 1e4
            bv.Parent = obj

            local bg = Instance.new("BodyAngularVelocity")
            bg.MaxTorque = Vector3.new(1e5,1e5,1e5)
            bg.AngularVelocity = Vector3.new(math.random(),math.random(),math.random())*10
            bg.Parent = obj

            -- إزالة القوة بعد 3 ثواني
            delay(3, function()
                if bv and bv.Parent then bv:Destroy() end
                if bg and bg.Parent then bg:Destroy() end
            end)
        end
    end
end

-- الحدث عند لمس لاعب لأي جزء
local function onTouched(hit)
    if not hit or not hit.Parent then return end
    local humanoid = hit.Parent:FindFirstChildOfClass("Humanoid")
    if humanoid then
        makeFly(hit.Parent)  -- اللاعب الذي لمسته يطير
        explodeMap()         -- الماب كله يتفكك
    end
end

-- ربط كل أجزاء اللاعبين الحالية والجديدة
local function connectCharacter(character)
    for _, part in pairs(character:GetChildren()) do
        if part:IsA("BasePart") then
            part.Touched:Connect(onTouched)
        end
    end
    character.ChildAdded:Connect(function(child)
        if child:IsA("BasePart") then
            child.Touched:Connect(onTouched)
        end
    end)
end

for _, player in pairs(Players:GetPlayers()) do
    if player.Character then
        connectCharacter(player.Character)
    end
end

Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        connectCharacter(character)
    end)
end)

print("Fly + Map Explode Script Active!")
