--[[ AUTO COLETAR INSTANTÂNEO - COMPATÍVEL MOBILE E PC ]]--

local player = game.Players.LocalPlayer
local UIS = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local Enabled = false
local RANGE = 15

-- Proteção Anti-Kick
pcall(function()
    local mt = getrawmetatable(game)
    setreadonly(mt, false)
    local kick = mt.__namecall
    mt.__namecall = newcclosure(function(self, ...)
        local method = getnamecallmethod()
        if tostring(method) == "Kick" then
            return warn("Tentativa de Kick Bloqueada.")
        end
        return kick(self, ...)
    end)
end)

-- GUI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AutoColetarGUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = (CoreGui or player:WaitForChild("PlayerGui"))

local Button = Instance.new("TextButton")
Button.Size = UDim2.new(0, 120, 0, 40)
Button.Position = UDim2.new(0, 20, 0, 100)
Button.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Button.TextColor3 = Color3.fromRGB(255, 255, 255)
Button.Text = "Auto Coletar [OFF]"
Button.Font = Enum.Font.SourceSansBold
Button.TextSize = 16
Button.Parent = ScreenGui
Button.Active = true
Button.Draggable = true

-- Anti Lag
local function AntiLag()
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("ParticleEmitter") or obj:IsA("Trail") or obj:IsA("Smoke") or obj:IsA("Fire") or obj:IsA("Explosion") then
            obj:Destroy()
        elseif obj:IsA("Decal") then
            obj.Transparency = 1
        end
    end
end
AntiLag()

-- Loop direto (sem cache) para coleta mais instantânea
task.spawn(function()
    while task.wait(0.1) do
        if Enabled then
            local char = player.Character
            if not (char and char:FindFirstChild("HumanoidRootPart")) then continue end
            local root = char.HumanoidRootPart

            for _, v in pairs(workspace:GetDescendants()) do
                -- ProximityPrompt
                if v:IsA("ProximityPrompt") and v.Enabled then
                    local success, dist = pcall(function()
                        return (v.Parent.Position - root.Position).Magnitude
                    end)
                    if success and dist <= (v.MaxActivationDistance or RANGE) then
                        pcall(function()
                            fireproximityprompt(v)
                        end)
                    end
                end

                -- ClickDetector
                if v:IsA("ClickDetector") then
                    local success, dist = pcall(function()
                        return (v.Parent.Position - root.Position).Magnitude
                    end)
                    if success and dist <= RANGE then
                        pcall(function()
                            fireclickdetector(v)
                        end)
                    end
                end
            end
        end
    end
end)

-- Botão ativar/desativar
Button.MouseButton1Click:Connect(function()
    Enabled = not Enabled
    Button.Text = Enabled and "Auto Coletar [ON]" or "Auto Coletar [OFF]"
    Button.BackgroundColor3 = Enabled and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(30, 30, 30)
end)
