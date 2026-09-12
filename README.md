local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")

local lp = Players.LocalPlayer
local smokeRedActive = false
local activeSmokes = {}

-- Lógica estricta de selección por ID: Solo agarra lo que tenga exactamente el id "SmokeRed"
local function refreshSmokes()
    activeSmokes = {}
    local com = workspace:FindFirstChild("WorkspaceCom")
    if not com then return end

    local function collectProps(parent)
        for _, child in ipairs(parent:GetChildren()) do
            -- Filtro estricto: Solo ID "SmokeRed" y perteneciente al jugador local
            if (child.Name == "Prop" .. lp.Name or child.Name == "SmokeRed") and child:GetAttribute("id") == "SmokeRed" then
                local remote = child:FindFirstChild("SetCurrentCFrame")
                if remote and remote:IsA("RemoteFunction") then
                    table.insert(activeSmokes, {prop = child, remote = remote})
                end
            end
            if child:IsA("Folder") or child:IsA("Model") then
                collectProps(child)
            end
        end
    end
    
    collectProps(com)
end

-- Bucle constante: mantiene los SmokeRed estrictamente abajo de los pies (-3.5 en Y)
RunService.Heartbeat:Connect(function()
    if not smokeRedActive then return end
    
    local char = lp.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    
    local tRoot = char.HumanoidRootPart
    local legCF = tRoot.CFrame * CFrame.new(0, -3.5, 0)
    
    if #activeSmokes == 0 then
        refreshSmokes()
    end
    
    for _, item in ipairs(activeSmokes) do
        if item.prop and item.prop.Parent then
            pcall(function()
                if item.prop:IsA("Model") then
                    item.prop:PivotTo(legCF)
                elseif item.prop:IsA("BasePart") then
                    item.prop.CFrame = legCF
                end
                item.remote:InvokeServer(legCF)
            end)
        end
    end
end)

local function toggleSmokeRed(state)
    smokeRedActive = state
    if state then
        refreshSmokes()
    else
        local undergroundCF = CFrame.new(0, -1000, 0)
        for _, item in ipairs(activeSmokes) do
            if item.prop and item.prop.Parent then
                pcall(function()
                    if item.prop:IsA("Model") then
                        item.prop:PivotTo(undergroundCF)
                    elseif item.prop:IsA("BasePart") then
                        item.prop.CFrame = undergroundCF
                    end
                    item.remote:InvokeServer(undergroundCF)
                end)
            end
        end
        activeSmokes = {}
    end
end

-- ================= INTERFAZ GRÁFICA DIMINUTA =================
local targetParent = (pcall(function() return CoreGui.Name end) and CoreGui) or lp:WaitForChild("PlayerGui")
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "SmokeRedMiniPanel"
ScreenGui.Parent = targetParent
ScreenGui.ResetOnSpawn = false

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 190, 0, 100)
Frame.Position = UDim2.new(0.5, -95, 0.5, -50)
Frame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
Frame.Active = true
Frame.Draggable = true
Frame.Parent = ScreenGui
Instance.new("UICorner", Frame).CornerRadius = UDim.new(0, 8)

local topBar = Instance.new("Frame", Frame)
topBar.Size = UDim2.new(1, 0, 0, 24)
topBar.BackgroundColor3 = Color3.fromRGB(30, 30, 38)
topBar.BorderSizePixel = 0
Instance.new("UICorner", topBar).CornerRadius = UDim.new(0, 8)

local Title = Instance.new("TextLabel", topBar)
Title.Position = UDim2.new(0, 8, 0, 0)
Title.Size = UDim2.new(1, -26, 1, 0)
Title.Text = "💨 SmokeRed Estricto"
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 11
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.BackgroundTransparency = 1

local CloseButton = Instance.new("TextButton", topBar)
CloseButton.Size = UDim2.new(0, 20, 0, 20)
CloseButton.Position = UDim2.new(1, -22, 0, 2)
CloseButton.Text = "X"
CloseButton.TextSize = 10
CloseButton.TextColor3 = Color3.fromRGB(200, 50, 50)
CloseButton.BackgroundTransparency = 1
Instance.new("UICorner", CloseButton)

local ToggleBtn = Instance.new("TextButton", Frame)
ToggleBtn.Size = UDim2.new(0.85, 0, 0, 42)
ToggleBtn.Position = UDim2.new(0.075, 0, 0, 38)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.TextSize = 12
ToggleBtn.Font = Enum.Font.GothamBold
ToggleBtn.Text = "SmokeRed: OFF"
Instance.new("UICorner", ToggleBtn).CornerRadius = UDim.new(0, 6)

ToggleBtn.MouseButton1Click:Connect(function()
    local newState = not smokeRedActive
    toggleSmokeRed(newState)
    if newState then
        ToggleBtn.Text = "SmokeRed: ON"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(40, 160, 80)
    else
        ToggleBtn.Text = "SmokeRed: OFF"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
    end
end)

CloseButton.MouseButton1Click:Connect(function()
    toggleSmokeRed(false)
    ScreenGui:Destroy()
end)
