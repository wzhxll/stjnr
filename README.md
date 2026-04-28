-- 独立飞行脚本 - 无相机锁定（完整UI，原样提取）
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local lp = Players.LocalPlayer
local camera = workspace.CurrentCamera
local pgui = lp:WaitForChild("PlayerGui")
local CoreGui = game:GetService("CoreGui")
local ControlModule = require(lp.PlayerScripts:WaitForChild("PlayerModule")):GetControls()

-- 原样复制 FlightSystem
local FlightSystem = {}
FlightSystem.__index = FlightSystem

function FlightSystem.new()
    local self = setmetatable({}, FlightSystem)
    self.isFlying = false
    self.flySpeed = 40
    self.bv = nil
    self.animCache = nil
    self.hrp = nil
    self.hum = nil
    return self
end

function FlightSystem:clearResources()
    local char = lp.Character
    if self.animCache and char then self.animCache.Parent = char end
    if self.bv then self.bv:Destroy() end
    self.bv = nil
    if self.hum and self.hum.Parent then
        self.hum:ChangeState(Enum.HumanoidStateType.Running)
    end
end

function FlightSystem:startFly()
    if self.isFlying then return end
    local char = lp.Character
    if not char then return end
    self.hrp = char:WaitForChild("HumanoidRootPart")
    self.hum = char:WaitForChild("Humanoid")
    local ani = char:FindFirstChild("Animate")
    if ani then
        self.animCache = ani
        ani.Parent = nil
    end
    if self.hrp:FindFirstChild("LeipzigBV") then self.hrp.LeipzigBV:Destroy() end
    self.bv = Instance.new("BodyVelocity", self.hrp)
    self.bv.Name = "LeipzigBV"
    self.bv.MaxForce = Vector3.new(1e6, 1e6, 1e6)
    self.isFlying = true
    task.spawn(function()
        while self.isFlying and char.Parent do
            local mv = ControlModule:GetMoveVector()
            local cf = camera.CFrame
            local dir = (cf.LookVector * -mv.Z) + (cf.RightVector * mv.X)
            if mv.Magnitude > 0 then
                self.bv.Velocity = dir.Unit * self.flySpeed
            else
                self.bv.Velocity = Vector3.new(0, 0.01, 0)
            end
            self.hum:ChangeState(Enum.HumanoidStateType.Climbing)
            RunService.RenderStepped:Wait()
        end
        self:clearResources()
    end)
end

function FlightSystem:stopFly()
    if not self.isFlying then return end
    self.isFlying = false
    self:clearResources()
end

function FlightSystem:setSpeed(speed)
    self.flySpeed = math.clamp(speed, 10, 100)
end

local flight = FlightSystem.new()

local function bindCharacter()
    local char = lp.Character or lp.CharacterAdded:Wait()
    flight.hrp = char:WaitForChild("HumanoidRootPart")
    flight.hum = char:WaitForChild("Humanoid")
    flight:clearResources()
    char.AncestryChanged:Connect(function(_, parent)
        if not parent then
            flight:clearResources()
            bindCharacter()
        end
    end)
end
bindCharacter()

-- 创建UI（原样）
if pgui:FindFirstChild("OriginalFlightUI") then
    pgui.OriginalFlightUI:Destroy()
end

local UI_BG = Color3.fromRGB(200, 230, 255)
local BTN_OFF = Color3.fromRGB(150, 200, 255)
local BTN_ON = Color3.fromRGB(70, 150, 255)
local DESTROY_BTN = Color3.fromRGB(110, 180, 255)
local TEXT_COLOR = Color3.fromRGB(0, 60, 120)
local SPEED_BG = Color3.fromRGB(180, 220, 255)

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "OriginalFlightUI"
ScreenGui.Parent = pgui
ScreenGui.ResetOnSpawn = false
ScreenGui.DisplayOrder = 999

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 150, 0, 145)
MainFrame.Position = UDim2.new(0.5, -75, 0.3, 0)
MainFrame.BackgroundColor3 = UI_BG
MainFrame.BackgroundTransparency = 0.4
MainFrame.Draggable = true
MainFrame.Active = true
MainFrame.Parent = ScreenGui

Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 10)
local stroke = Instance.new("UIStroke", MainFrame)
stroke.Color = Color3.fromRGB(120, 200, 255)
stroke.Thickness = 3
stroke.Transparency = 0.1

local Title = Instance.new("TextLabel", MainFrame)
Title.Size = UDim2.new(1,0,0,20)
Title.BackgroundTransparency = 1
Title.Text = "飞行-无相机锁定"
Title.TextColor3 = TEXT_COLOR
Title.TextSize = 12
Title.Font = Enum.Font.GothamBold

local Tip = Instance.new("TextLabel", MainFrame)
Tip.Size = UDim2.new(1,0,0,14)
Tip.Position = UDim2.new(0,0,0,20)
Tip.BackgroundTransparency = 1
Tip.Text = "[无穿墙]"
Tip.TextColor3 = Color3.new(0.9,0,0)
Tip.TextSize = 8

local SpeedInput = Instance.new("TextBox", MainFrame)
SpeedInput.Size = UDim2.new(0,120,0,24)
SpeedInput.Position = UDim2.new(0.5,-60,0, 38)
SpeedInput.BackgroundColor3 = SPEED_BG
SpeedInput.BackgroundTransparency = 0.3
SpeedInput.Text = tostring(flight.flySpeed)
SpeedInput.TextColor3 = TEXT_COLOR
SpeedInput.TextSize = 11
Instance.new("UICorner", SpeedInput).CornerRadius = UDim.new(0,7)

local FlyBtn = Instance.new("TextButton", MainFrame)
FlyBtn.Size = UDim2.new(0,120,0,26)
FlyBtn.Position = UDim2.new(0.5,-60,0, 72)
FlyBtn.BackgroundColor3 = BTN_OFF
FlyBtn.BackgroundTransparency = 0.3
FlyBtn.Text = "飞行"
FlyBtn.TextColor3 = TEXT_COLOR
FlyBtn.TextSize = 11
Instance.new("UICorner", FlyBtn).CornerRadius = UDim.new(0,8)

local DestroyUI = Instance.new("TextButton", MainFrame)
DestroyUI.Size = UDim2.new(0,120,0,26)
DestroyUI.Position = UDim2.new(0.5,-60,0, 108)
DestroyUI.BackgroundColor3 = DESTROY_BTN
DestroyUI.BackgroundTransparency = 0.3
DestroyUI.Text = "销毁UI"
DestroyUI.TextColor3 = TEXT_COLOR
DestroyUI.TextSize = 11
Instance.new("UICorner", DestroyUI).CornerRadius = UDim.new(0,8)

local dragging, dragStart, startPos
MainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = MainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

SpeedInput.FocusLost:Connect(function()
    local val = tonumber(SpeedInput.Text)
    if val then flight:setSpeed(val) else flight:setSpeed(40) end
    SpeedInput.Text = tostring(flight.flySpeed)
end)

FlyBtn.MouseButton1Click:Connect(function()
    if flight.isFlying then
        flight:stopFly()
        FlyBtn.Text = "飞行"
        FlyBtn.BackgroundColor3 = BTN_OFF
    else
        flight:startFly()
        FlyBtn.Text = "飞行开"
        FlyBtn.BackgroundColor3 = BTN_ON
    end
end)

DestroyUI.MouseButton1Click:Connect(function()
    flight:stopFly()
    ScreenGui:Destroy()
end)

MainFrame.Size = UDim2.new(0,0,0,0)
MainFrame:TweenSize(UDim2.new(0,150,0,145), Enum.EasingDirection.Out, Enum.EasingStyle.Back, 0.4, true)
