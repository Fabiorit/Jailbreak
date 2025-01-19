kopiujesz od punktu 6 do 116




local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer
local HighlightTable = {}
local Target = nil
local FollowActive = false
local ScriptActive = true

function CreateHighlight(player)
    if player.Team and player.Team.Name == "Criminal" then
        local highlight = Instance.new("Highlight")
        highlight.Adornee = player.Character
        highlight.FillColor = Color3.new(1, 0, 0)
        highlight.OutlineTransparency = 1
        highlight.Parent = game.CoreGui
        HighlightTable[player] = highlight
    end
end

function RemoveHighlight(player)
    if HighlightTable[player] then
        HighlightTable[player]:Destroy()
        HighlightTable[player] = nil
    end
end

Players.PlayerAdded:Connect(function(player)
    if not ScriptActive then return end
    player.CharacterAdded:Connect(function()
        if player ~= LocalPlayer then
            if player.Team and player.Team.Name == "Criminal" then
                CreateHighlight(player)
            end
        end
    end)

    if player.Character and player.Team and player.Team.Name == "Criminal" then
        CreateHighlight(player)
    end

    player:GetPropertyChangedSignal("Team"):Connect(function()
        if player.Team and player.Team.Name ~= "Criminal" then
            RemoveHighlight(player)
        elseif player.Team and player.Team.Name == "Criminal" then
            CreateHighlight(player)
        end
    end)
end)

Players.PlayerRemoving:Connect(function(player)
    RemoveHighlight(player)
end)

for _, player in pairs(Players:GetPlayers()) do
    if player ~= LocalPlayer and player.Character then
        if player.Team and player.Team.Name == "Criminal" then
            CreateHighlight(player)
        end
    end
end

function FindAttackablePlayer()
    local closest, distance = nil, math.huge
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if player.Team and player.Team.Name == "Criminal" then
                local magnitude = (player.Character.HumanoidRootPart.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
                if magnitude < distance then
                    closest = player
                    distance = magnitude
                end
            end
        end
    end
    return closest
end

game:GetService("UserInputService").InputBegan:Connect(function(input, processed)
    if not processed then
        if input.KeyCode == Enum.KeyCode.X then
            FollowActive = not FollowActive
            if FollowActive then
                Target = FindAttackablePlayer()
            else
                Target = nil
            end
        elseif input.KeyCode == Enum.KeyCode.RightShift then
            ScriptActive = not ScriptActive
            if not ScriptActive then
                for _, player in pairs(Players:GetPlayers()) do
                    RemoveHighlight(player)
                end
                Target = nil
                FollowActive = false
            else
                for _, player in pairs(Players:GetPlayers()) do
                    if player.Team and player.Team.Name == "Criminal" then
                        CreateHighlight(player)
                    end
                end
            end
        end
    end
end)

RunService.RenderStepped:Connect(function()
    if ScriptActive and FollowActive and Target and Target.Character and Target.Character:FindFirstChild("HumanoidRootPart") then
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, Target.Character.HumanoidRootPart.Position)
    end
end)
