-- BoostFPS Cliente (Legible, ejecutable)

local Players = game:GetService("Players")

local RunService = game:GetService("RunService")

local Lighting = game:GetService("Lighting")

local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer



-- Backups

local backup = {Lighting = {}, Terrain = {}, Descendants = {}}

local applied = false

local effectClasses = {"ParticleEmitter","Trail","Beam","Fire","Smoke","Sparkles"}



-- ===== Lighting =====

local function applyLightingLow()

&nbsp;   backup.Lighting.GlobalShadows = Lighting.GlobalShadows

&nbsp;   backup.Lighting.Brightness = Lighting.Brightness

&nbsp;   backup.Lighting.ClockTime = Lighting.ClockTime

&nbsp;   backup.Lighting.FogEnd = Lighting.FogEnd

&nbsp;   backup.Lighting.Ambient = Lighting.Ambient

&nbsp;   backup.Lighting.OutdoorAmbient = Lighting.OutdoorAmbient

&nbsp;   backup.Lighting.ExposureCompensation = Lighting.ExposureCompensation



&nbsp;   Lighting.GlobalShadows = false

&nbsp;   Lighting.Brightness = 1

&nbsp;   Lighting.ClockTime = 12

&nbsp;   Lighting.FogEnd = 1e6

&nbsp;   Lighting.Ambient = Color3.fromRGB(255,255,255)

&nbsp;   Lighting.OutdoorAmbient = Color3.fromRGB(255,255,255)

&nbsp;   Lighting.ExposureCompensation = 0



&nbsp;   -- PostProcessing

&nbsp;   backup.Lighting.PostProcessing = {}

&nbsp;   for \_,inst in ipairs(Lighting:GetChildren()) do

&nbsp;       if inst:IsA("BloomEffect") or inst:IsA("ColorCorrectionEffect") or inst:IsA("DepthOfFieldEffect") or inst:IsA("SunRaysEffect") or inst:IsA("BlurEffect") then

&nbsp;           table.insert(backup.Lighting.PostProcessing,{inst=inst,Enabled=inst.Enabled})

&nbsp;           inst.Enabled=false

&nbsp;       end

&nbsp;   end

end



local function restoreLighting()

&nbsp;   for k,v in pairs(backup.Lighting) do

&nbsp;       if k~="PostProcessing" then

&nbsp;           Lighting\[k] = v

&nbsp;       end

&nbsp;   end

&nbsp;   if backup.Lighting.PostProcessing then

&nbsp;       for \_,entry in ipairs(backup.Lighting.PostProcessing) do

&nbsp;           entry.inst.Enabled = entry.Enabled

&nbsp;       end

&nbsp;   end

end



-- ===== Terrain =====

local Terrain = Workspace:FindFirstChildOfClass("Terrain")

local function applyTerrainLow()

&nbsp;   if not Terrain then return end

&nbsp;   backup.Terrain.WaterWaveSize = Terrain.WaterWaveSize

&nbsp;   backup.Terrain.WaterWaveSpeed = Terrain.WaterWaveSpeed

&nbsp;   backup.Terrain.WaterReflectance = Terrain.WaterReflectance

&nbsp;   backup.Terrain.WaterTransparency = Terrain.WaterTransparency



&nbsp;   Terrain.WaterWaveSize = 0

&nbsp;   Terrain.WaterWaveSpeed = 0

&nbsp;   Terrain.WaterReflectance = 0

&nbsp;   Terrain.WaterTransparency = 0.7

end

local function restoreTerrain()

&nbsp;   if not Terrain then return end

&nbsp;   Terrain.WaterWaveSize = backup.Terrain.WaterWaveSize

&nbsp;   Terrain.WaterWaveSpeed = backup.Terrain.WaterWaveSpeed

&nbsp;   Terrain.WaterReflectance = backup.Terrain.WaterReflectance

&nbsp;   Terrain.WaterTransparency = backup.Terrain.WaterTransparency

end



-- ===== Descendants =====

local function applyDescendantsLow()

&nbsp;   backup.Descendants = {}

&nbsp;   for \_,obj in ipairs(Workspace:GetDescendants()) do

&nbsp;       local class = obj.ClassName

&nbsp;       for \_,c in ipairs(effectClasses) do

&nbsp;           if class==c then

&nbsp;               table.insert(backup.Descendants,{obj=obj,prop="Enabled",val=obj.Enabled})

&nbsp;               pcall(function() obj.Enabled=false end)

&nbsp;           end

&nbsp;       end

&nbsp;       if obj:IsA("Decal") or obj:IsA("Texture") then

&nbsp;           table.insert(backup.Descendants,{obj=obj,prop="Transparency",val=obj.Transparency})

&nbsp;           pcall(function() obj.Transparency=1 end)

&nbsp;       end

&nbsp;       if (obj:IsA("BillboardGui") or obj:IsA("SurfaceGui")) and obj:IsDescendantOf(Workspace) then

&nbsp;           table.insert(backup.Descendants,{obj=obj,prop="Enabled",val=obj.Enabled})

&nbsp;           pcall(function() obj.Enabled=false end)

&nbsp;       end

&nbsp;       if obj:IsA("BasePart") and not obj:IsDescendantOf(LocalPlayer.Character) then

&nbsp;           table.insert(backup.Descendants,{obj=obj,prop="Material",val=obj.Material})

&nbsp;           pcall(function() obj.Material=Enum.Material.Plastic end)

&nbsp;       end

&nbsp;   end

end



local function restoreDescendants()

&nbsp;   for \_,entry in ipairs(backup.Descendants) do

&nbsp;       local obj=entry.obj

&nbsp;       if obj and obj.Parent then

&nbsp;           pcall(function()

&nbsp;               if entry.prop=="Enabled" then obj.Enabled=entry.val

&nbsp;               elseif entry.prop=="Transparency" then obj.Transparency=entry.val

&nbsp;               elseif entry.prop=="Material" and obj:IsA("BasePart") then obj.Material=entry.val

&nbsp;               end

&nbsp;           end)

&nbsp;       end

&nbsp;   end

end



-- ===== Apply / Restore =====

local function applyLowGraphics()

&nbsp;   if applied then return end

&nbsp;   applied=true

&nbsp;   applyLightingLow()

&nbsp;   applyTerrainLow()

&nbsp;   applyDescendantsLow()

end



local function restoreGraphics()

&nbsp;   if not applied then return end

&nbsp;   applied=false

&nbsp;   restoreDescendants()

&nbsp;   restoreTerrain()

&nbsp;   restoreLighting()

end



-- ===== UI =====

local playerGui = LocalPlayer:WaitForChild("PlayerGui")

local sg = Instance.new("ScreenGui",playerGui)

sg.Name="BoostFPSUI"

sg.ResetOnSpawn=false



local btn = Instance.new("TextButton",sg)

btn.Size=UDim2.new(0,140,0,48)

btn.Position=UDim2.new(0,12,0,12)

btn.Text="Boost FPS: OFF"

btn.Font=Enum.Font.GothamBold

btn.TextSize=18

btn.BackgroundColor3=Color3.fromRGB(40,40,40)

btn.TextColor3=Color3.fromRGB(255,255,255)

local corner = Instance.new("UICorner",btn) corner.CornerRadius=UDim.new(0,8)



btn.MouseButton1Click:Connect(function()

&nbsp;   if applied then

&nbsp;       restoreGraphics()

&nbsp;       btn.Text="Boost FPS: OFF"

&nbsp;       btn.BackgroundColor3=Color3.fromRGB(40,40,40)

&nbsp;   else

&nbsp;       applyLowGraphics()

&nbsp;       btn.Text="Boost FPS: ON"

&nbsp;       btn.BackgroundColor3=Color3.fromRGB(30,160,30)

&nbsp;   end

end)



-- Auto aplicar al cargar

applyLowGraphics()

btn.Text="Boost FPS: ON"

btn.BackgroundColor3=Color3.fromRGB(30,160,30)



print("\[BoostFPS] Script cargado ✅")

### 

