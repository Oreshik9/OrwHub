getgenv().FOV = 100 
getgenv().Enabled = true

local players = game:GetService("Players")
local replicatedStorage = game:GetService("ReplicatedStorage")

local localPlayer = players.LocalPlayer
local currentCamera = workspace.CurrentCamera

local raycastModule = require(replicatedStorage.Events.Modules.RaycastModule)

local fovCircle =  Drawing.new("Circle")
fovCircle.Position = currentCamera.ViewportSize * 0.5
fovCircle.Visible = true
fovCircle.Color = Color3.fromRGB(255, 255, 255)
fovCircle.Radius = getgenv().FOV
fovCircle.Transparency = 1
fovCircle.Filled = false
fovCircle.NumSides = 0

local function getClosestPlayer()
    local closest = nil
    local closestDistance = math.huge

    for _, player in players:GetPlayers() do
        if player == localPlayer or (player.Team == localPlayer.Team and localPlayer.Team ~= nil) then continue end

        local character = player.Character
        if not character then continue end

        local rootPart = character:FindFirstChild("HumanoidRootPart")
        if not rootPart then continue end

        local screenPosition, onScreen = currentCamera:WorldToViewportPoint(rootPart.Position)
        if not onScreen then continue end

        local screenDistance = (Vector2.new(screenPosition.X, screenPosition.Y) -  currentCamera.ViewportSize * 0.5).Magnitude

        if screenPosition.Z > 0 and screenDistance < getgenv().FOV and screenDistance < closestDistance then
            closest = character
            closestDistance = screenDistance
        end
    end

    return closest
end

for i, func in raycastModule do
    if not typeof(func) == "function" then continue end

    raycastModule[i] = function(...)
        if not getgenv().Enabled then return func(...) end

        local closestPlayer = getClosestPlayer()
        if not closestPlayer then return func(...) end

        return closestPlayer.Head, closestPlayer.Head.Position, Vector3.zero
    end
end 
