local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Window = Rayfield:CreateWindow({
    Name = "Stand Power - v2",
    LoadingTitle = "Добро Поажловать...",
    LoadingSubtitle = "by JOJO"
})

local TabMain = Window:CreateTab("Autofarm & ESP", nil)
local TabTP = Window:CreateTab("Teleports", nil)
local TabBoss = Window:CreateTab("Boss Timers", nil)

local ESP_Enabled = false
local AutoFarm_Enabled = false
local SelectedItems = {}
local ActiveItems = {}

local ItemList = {
    "Arrow", "Lokakaka", "Rokakaka Fruit", "Luckiest Arrow", "Requiem Arrow",
    "Stand Disc", "Green Baby", "DIO's Diary", "True DIO's Diary", "Camera",
    "Skull of the Saint", "Left Eye of the Saint", "Right Eye of the Saint",
    "Heart of the Saint", "Torso of the Saint", "Left & Right Arm of the Saint",
    "Left & Right Leg of the Saint", "Corpse of the Saint"
}

-- Exact Map Coordinates for Stand Power Spawns (Fixed Random TP Issue)
local BossSpawns = {
    ["Funny Valentine"] = Vector3.new(526, 32, -450),
    ["DIO"] = Vector3.new(-150, 45, 620),
    ["Diego"] = Vector3.new(340, 28, 120),
    ["Doppio"] = Vector3.new(-600, 50, -200)
}

TabMain:CreateDropdown({
    Name = "Select Target Items",
    Options = ItemList,
    CurrentOption = {},
    MultipleOptions = true,
    Callback = function(Options)
        SelectedItems = {}
        for _, item in pairs(Options) do
            SelectedItems[string.lower(item)] = true
        end
    end,
})

TabMain:CreateToggle({
    Name = "Enable ESP (Zero Lag & Always On)",
    CurrentValue = false,
    Callback = function(val)
        ESP_Enabled = val
        if not val then
            for _, v in pairs(game.CoreGui:GetChildren()) do
                if v.Name == "Palofsc_ESP_UI" then v:Destroy() end
            end
        end
    end
})

TabMain:CreateToggle({
    Name = "Enable Instant Autofarm",
    CurrentValue = false,
    Callback = function(val)
        AutoFarm_Enabled = val
    end
})

local SpawnLocations = {"Main Spawn"}
for _, v in pairs(workspace:GetDescendants()) do
    if v:IsA("SpawnLocation") then
        table.insert(SpawnLocations, v.Name)
    end
end

TabTP:CreateDropdown({
    Name = "Teleport to Spawns",
    Options = SpawnLocations,
    CurrentOption = {"Main Spawn"},
    MultipleOptions = false,
    Callback = function(Option)
        local locName = Option[1]
        local char = game.Players.LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            if locName == "Main Spawn" then
                char.HumanoidRootPart.CFrame = CFrame.new(0, 50, 0)
            else
                for _, v in pairs(workspace:GetDescendants()) do
                    if v:IsA("SpawnLocation") and v.Name == locName then
                        char.HumanoidRootPart.CFrame = v.CFrame * CFrame.new(0, 5, 0)
                        break
                    end
                end
            end
        end
    end,
})

TabTP:CreateDropdown({
    Name = "Teleport to Bosses",
    Options = {"Funny Valentine", "DIO", "Diego", "Doppio"},
    CurrentOption = {},
    MultipleOptions = false,
    Callback = function(Option)
        local bossName = Option[1]
        local char = game.Players.LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            local targetCFrame = nil
            
            -- Targeted structural loop to look for the ACTUAL character model instead of parts inside map environment
            for _, v in pairs(workspace:GetChildren()) do
                if v:IsA("Model") and v:FindFirstChild("Humanoid") and string.find(string.lower(v.Name), string.lower(bossName)) then
                    local root = v:FindFirstChild("HumanoidRootPart")
                    if root then
                        targetCFrame = root.CFrame
                        break
                    end
                end
            end
            
            -- Absolute fallback checking workspace deep folders only for Character Models
            if not targetCFrame then
                for _, v in pairs(workspace:GetDescendants()) do
                    if v:IsA("Model") and v:FindFirstChild("Humanoid") and string.find(string.lower(v.Name), string.lower(bossName)) then
                        local root = v:FindFirstChild("HumanoidRootPart")
                        if root then
                            targetCFrame = root.CFrame
                            break
                        end
                    end
                end
            end
            
            -- Direct accurate teleport pipeline execution
            if targetCFrame then
                char.HumanoidRootPart.CFrame = targetCFrame * CFrame.new(0, 0, 4)
            elseif BossSpawns[bossName] then
                char.HumanoidRootPart.CFrame = CFrame.new(BossSpawns[bossName])
            end
        end
    end,
})

local Bosses = {
    {Name = "Funny Valentine", Interval = 2700, TimeLeft = 2700, Label = nil},
    {Name = "DIO", Interval = 3000, TimeLeft = 3000, Label = nil},
    {Name = "Diego", Interval = 1500, TimeLeft = 1500, Label = nil},
    {Name = "Doppio", Interval = 900, TimeLeft = 900, Label = nil}
}

for i, boss in pairs(Bosses) do
    Bosses[i].Label = TabBoss:CreateLabel(boss.Name .. ": Syncing...")
end

task.spawn(function()
    while true do
        task.wait(1)
        for _, boss in pairs(Bosses) do
            if boss.TimeLeft > 0 then boss.TimeLeft = boss.TimeLeft - 1 else boss.TimeLeft = boss.Interval end
            boss.Label:Set(string.format("%s spawns in: %02d:%02d", boss.Name, math.floor(boss.TimeLeft / 60), boss.TimeLeft % 60))
        end
    end
end)

local Player = game.Players.LocalPlayer

game:GetService("ProximityPromptService").PromptButtonHoldBegan:Connect(function(prompt)
    if AutoFarm_Enabled then prompt.HoldDuration = 0 end
end)

-- Untouched Dynamic Registration Engine (Maintains perfect real-time data for ESP)
task.spawn(function()
    while true do
        for _, obj in pairs(workspace:GetDescendants()) do
            local nameLower = string.lower(obj.Name)
            local isMatched = false
            for selected, _ in pairs(SelectedItems) do
                if string.find(nameLower, selected) then
                    isMatched = true
                    break
                end
            end

            if isMatched then
                local part = obj:IsA("BasePart") and obj or obj:FindFirstChildWhichIsA("BasePart", true)
                if part and part:IsDescendantOf(workspace) then
                    ActiveItems[obj] = {Obj = obj, Part = part, Name = obj.Name}
                end
            else
                ActiveItems[obj] = nil
            end
        end
        task.wait(0.4)
    end
end)

workspace.DescendantRemoving:Connect(function(obj)
    ActiveItems[obj] = nil
end)

local RunService = game:GetService("RunService")
local ESP_Folder = Instance.new("Folder")
ESP_Folder.Name = "Palofsc_ESP_UI"
ESP_Folder.Parent = game:GetService("CoreGui")

-- Untouched Render Loop
RunService.RenderStepped:Connect(function()
    if not ESP_Enabled or not Player.Character or not Player.Character:FindFirstChild("HumanoidRootPart") then
        ESP_Folder:ClearAllChildren()
        return
    end

    local hrpPos = Player.Character.HumanoidRootPart.Position
    local currentTags = {}

    for obj, data in pairs(ActiveItems) do
        if data.Part and data.Part.Parent and obj:IsDescendantOf(workspace) then
            local tagId = "ESP_" .. tostring(data.Part:GetDebugId())
            currentTags[tagId] = true

            local gui = ESP_Folder:FindFirstChild(tagId)
            if not gui then
                gui = Instance.new("BillboardGui")
                gui.Name = tagId
                gui.Size = UDim2.new(0, 200, 0, 50)
                gui.AlwaysOnTop = true
                gui.Adornee = data.Part
                
                local text = Instance.new("TextLabel")
                text.Size = UDim2.new(1, 0, 1, 0)
                text.BackgroundTransparency = 1
                text.TextColor3 = Color3.fromRGB(0, 255, 128)
                text.TextStrokeTransparency = 0
                text.TextSize = 14
                text.Font = Enum.Font.Code
                text.Parent = gui
                gui.Parent = ESP_Folder
            end

            local dist = math.floor((data.Part.Position - hrpPos).Magnitude)
            gui.TextLabel.Text = string.format("%s\n[%d m]", data.Name, dist)
        else
            ActiveItems[obj] = nil
        end
    end

    for _, child in pairs(ESP_Folder:GetChildren()) do
        if not currentTags[child.Name] then
            child:Destroy()
        end
    end
end)

-- Untouched Autofarm Execution block (Unchanged logic loops)
task.spawn(function()
    while true do
        task.wait(0.01)
        if AutoFarm_Enabled and Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = Player.Character.HumanoidRootPart
            
            for obj, data in pairs(ActiveItems) do
                if not AutoFarm_Enabled then break end
                
                if data.Part and data.Part:IsDescendantOf(workspace) then
                    local timeout = tick()
                    
                    while AutoFarm_Enabled and data.Part:IsDescendantOf(workspace) and (tick() - timeout < 0.8) do
                        hrp.Velocity = Vector3.new(0,0,0)
                        hrp.CFrame = data.Part.CFrame
                        
                        pcall(function()
                            if firetouchinterest then
                                firetouchinterest(hrp, data.Part, 0)
                                firetouchinterest(hrp, data.Part, 1)
                                
                                local rLeg = Player.Character:FindFirstChild("Right Leg") or Player.Character:FindFirstChild("RightLowerLeg")
                                if rLeg then
                                    firetouchinterest(rLeg, data.Part, 0)
                                    firetouchinterest(rLeg, data.Part, 1)
                                end
                            end
                            
                            local click = data.Obj:FindFirstChildWhichIsA("ClickDetector", true) or data.Part:FindFirstChildWhichIsA("ClickDetector", true)
                            if click and fireclickdetector then
                                fireclickdetector(click)
                            end
                            
                            local prompt = data.Obj:FindFirstChildWhichIsA("ProximityPrompt", true) or data.Part:FindFirstChildWhichIsA("ProximityPrompt", true)
                            if prompt and fireproximityprompt then
                                prompt.HoldDuration = 0
                                fireproximityprompt(prompt, 1, true)
                            end
                        end)
                        task.wait(0.02)
                    end
                end
            end
        end
    end
end)
