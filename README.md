local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local scoreUpdateEvent = Instance.new("RemoteEvent")
scoreUpdateEvent.Name = "ScoreUpdate"
scoreUpdateEvent.Parent = ReplicatedStorage

local treasureFolder = Instance.new("Folder")
treasureFolder.Name = "Treasures"
treasureFolder.Parent = Workspace

local treasures = {}
local treasureCount = 5

local function createTreasure(position)
    local treasure = Instance.new("Part")
    treasure.Size = Vector3.new(1, 1, 1)
    treasure.Position = position
    treasure.Anchored = true
    treasure.BrickColor = BrickColor.new("Bright yellow")
    treasure.Name = "Treasure"
    treasure.Parent = treasureFolder
    table.insert(treasures, treasure)
end

local function placeTreasures()
    for i = 1, treasureCount do
        local randomPosition = Vector3.new(
            math.random(-50, 50),
            5,
            math.random(-50, 50)
        )
        createTreasure(randomPosition)
    end
end

local playerScores = {}

local function onTreasureCollected(player, treasure)
    treasure:Destroy()
    playerScores[player.UserId] = (playerScores[player.UserId] or 0) + 1
    scoreUpdateEvent:FireClient(player, playerScores[player.UserId])
end

local function setupTreasureTouch()
    for _, treasure in pairs(treasures) do
        treasure.Touched:Connect(function(hit)
            local player = Players:GetPlayerFromCharacter(hit.Parent)
            if player then
                onTreasureCollected(player, treasure)
            end
        end)
    end
end

local function resetGame()
    for _, treasure in pairs(treasures) do
        treasure:Destroy()
    end
    treasures = {}
    playerScores = {}
    placeTreasures()
    setupTreasureTouch()
end

placeTreasures()
setupTreasureTouch()

while true do
    wait(60)
    resetGame()
end
