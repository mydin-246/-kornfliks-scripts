-- Load Rayfield UI
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- Services
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local UserInputService = game:GetService("UserInputService")
local plr = Players.LocalPlayer
local camera = Workspace.CurrentCamera
local mouse = plr:GetMouse()

-- Variables
local weaponModsEnabled, aimbotEnabled, espEnabled, fovCircleVisible, flyEnabled = false, false, false, false, false
local aimFov, aimPrediction = 100, 0.065
local targetPlayer = nil
local ESPObjects = {}
local flySpeed = 1
local flyVelocity = Vector3.new(0,0,0)

-- Window
local Window = Rayfield:CreateWindow({
	Name = "▶ Gun Grounds FFA Ultra PRO MAX ◀",
	LoadingTitle = "Loading...",
	LoadingSubtitle = "Please wait 🔥",
	SaveConfig = {Enabled = true, FolderName = "GunGroundsFFA", FileName = "UltraPROMAX"}
})

-- Tabs
local MainTab = Window:CreateTab("Weapon Mods 🔫")
local AimbotTab = Window:CreateTab("Aimbot ☠️")
local ESPTab = Window:CreateTab("ESP 👀")
local MiscTab = Window:CreateTab("Fun/Misc ✨")

-- ===================
-- Weapon Mods
-- ===================
MainTab:CreateToggle({Name="Enable Weapon Mods", CurrentValue=false, Flag="WeaponModsEnabled", Callback=function(val) weaponModsEnabled=val Rayfield:Notify({Title="Weapon Mods", Content=val and "Enabled!" or "Disabled!", Duration=1, Image=4483362458}) end})
MainTab:CreateSlider({Name="Bullet Speed", Range={50,1000}, Increment=10, CurrentValue=300, Flag="BulletSpeed", Callback=function(val) -- Add bullet speed code here end})
MainTab:CreateToggle({Name="No Recoil", CurrentValue=false, Flag="NoRecoil", Callback=function(val) -- Add recoil remove code end})
MainTab:CreateToggle({Name="Infinite Ammo", CurrentValue=false, Flag="InfiniteAmmo", Callback=function(val) -- Infinite ammo code end})
MainTab:CreateToggle({Name="Rapid Fire", CurrentValue=false, Flag="RapidFire", Callback=function(val) -- Rapid fire code end})

-- ===================
-- Aimbot
-- ===================
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness, fovCircle.NumSides, fovCircle.Radius, fovCircle.Filled, fovCircle.Color, fovCircle.Visible = 2, 12, aimFov, false, Color3.fromRGB(255,0,0), fovCircleVisible

local function isWallBetween(targetCharacter)
	if not targetCharacter:FindFirstChild("Head") then return true end
	local origin = camera.CFrame.Position
	local direction = (targetCharacter.Head.Position-origin).Unit*(targetCharacter.Head.Position-origin).Magnitude
	local rayParams = RaycastParams.new()
	rayParams.FilterDescendantsInstances={plr.Character,targetCharacter}
	rayParams.FilterType=Enum.RaycastFilterType.Blacklist
	local rayResult = Workspace:Raycast(origin,direction,rayParams)
	return rayResult and rayResult.Instance~=nil
end

local function getNearestPlayer()
	local nearest, shortestDistance=nil,math.huge
	for _,player in ipairs(Players:GetPlayers()) do
		if player~=plr and player.Character and player.Character:FindFirstChild("Head") then
			local headPos = camera:WorldToViewportPoint(player.Character.Head.Position)
			local mousePos = Vector2.new(mouse.X,mouse.Y)
			local distance = (Vector2.new(headPos.X,headPos.Y)-mousePos).Magnitude
			if distance<shortestDistance and headPos.Z>0 and not isWallBetween(player.Character) then
				shortestDistance, nearest = distance, player
			end
		end
	end
	return nearest
end

local function predictPosition(player)
	if player and player.Character and player.Character:FindFirstChild("Head") and player.Character:FindFirstChild("HumanoidRootPart") then
		local head, hrp = player.Character.Head, player.Character.HumanoidRootPart
		return head.Position+(hrp.Velocity*aimPrediction)
	end
	return nil
end

local function aimAtPlayer(player)
	local predictedPos = predictPosition(player)
	if predictedPos then
		camera.CFrame = CFrame.new(camera.CFrame.Position,predictedPos)
	end
end

-- ===================
-- ESP
-- ===================
RunService.RenderStepped:Connect(function()
	if aimbotEnabled then
		targetPlayer=getNearestPlayer()
		if targetPlayer then aimAtPlayer(targetPlayer) end
	end

	if espEnabled then
		for _,p in ipairs(Players:GetPlayers()) do
			if p~=plr and p.Character and p.Character:FindFirstChild("Head") then
				if not ESPObjects[p] then
					ESPObjects[p]=Drawing.new("Square")
					ESPObjects[p].Thickness, ESPObjects[p].Color, ESPObjects[p].Filled = 1, Color3.new(1,0,0), false
				end
				local headPos,onScreen=camera:WorldToViewportPoint(p.Character.Head.Position)
				if onScreen then
					ESPObjects[p].Position,ESPObjects[p].Size,ESPObjects[p].Visible = Vector2.new(headPos.X-5,headPos.Y-5),Vector2.new(10,10),true
				else ESPObjects[p].Visible=false end
			end
		end
	end

	-- Fly Mode
	if flyEnabled then
		local char=plr.Character
		if char and char:FindFirstChild("HumanoidRootPart") then
			local hrp=char.HumanoidRootPart
			local moveVec = Vector3.new(0,0,0)
			if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveVec = moveVec + (hrp.CFrame.LookVector) end
			if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveVec = moveVec - (hrp.CFrame.LookVector) end
			if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveVec = moveVec - (hrp.CFrame.RightVector) end
			if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveVec = moveVec + (hrp.CFrame.RightVector) end
			if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveVec = moveVec + Vector3.new(0,1,0) end
			if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveVec = moveVec - Vector3.new(0,1,0) end
			hrp.Velocity = moveVec*flySpeed*50
		end
	end
end)

-- ===================
-- GUI Toggles & Sliders
-- ===================
AimbotTab:CreateToggle({Name="Enable Aimbot", CurrentValue=false, Flag="Aimbot", Callback=function(val) aimbotEnabled=fovCircle.Visible=val end})
AimbotTab:CreateSlider({Name="Aimbot FOV", Range={0,360}, Increment=1, CurrentValue=aimFov, Flag="AimbotFOV", Callback=function(val) aimFov=fovCircle.Radius=val end})

ESPTab:CreateToggle({Name="Enable ESP", CurrentValue=false, Flag="ESP", Callback=function(val) espEnabled=val end})
ESPTab:CreateToggle({Name="ESP Names", CurrentValue=false, Flag="ESPNames", Callback=function(val) -- Names code here end})
ESPTab:CreateToggle({Name="ESP Health", CurrentValue=false, Flag="ESPHealth", Callback=function(val) -- Health bars code here end})

MiscTab:CreateToggle({Name="Jump Boost", CurrentValue=false, Flag="JumpBoost", Callback=function(val) plr.Character.Humanoid.JumpPower = val and 150 or 50 end})
MiscTab:CreateToggle({Name="Speed Boost", CurrentValue=false, Flag="SpeedBoost", Callback=function(val) plr.Character.Humanoid.WalkSpeed = val and 100 or 16 end})
MiscTab:CreateToggle({Name="Wall Clip", CurrentValue=false, Flag="WallClip", Callback=function(val) plr.Character.HumanoidRootPart.CanCollide = not val end})
MiscTab:CreateToggle({Name="Gravity Mod", CurrentValue=false, Flag="GravityMod", Callback=function(val) workspace.Gravity = val and 30 or 196.2 end})
MiscTab:CreateToggle({Name="Fly Mode", CurrentValue=false, Flag="FlyMode", Callback=function(val) flyEnabled=val end})
MiscTab:CreateSlider({Name="Fly Speed", Range={1,20}, Increment=1, CurrentValue=5, Flag="FlySpeed", Callback=function(val) flySpeed=val end})
