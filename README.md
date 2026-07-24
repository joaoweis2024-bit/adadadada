repeat task.wait() until game:IsLoaded()
local Players,RunService,UIS,TS,Lighting,HS = game:GetService("Players"),game:GetService("RunService"),game:GetService("UserInputService"),game:GetService("TweenService"),game:GetService("Lighting"),game:GetService("HttpService")
local LP = Players.LocalPlayer
local NS,CS,DAS,DAD = 60,30,150,0.2
local LAGGER_SPEED = 15
local speedMode,antiRagdollEnabled,infJumpEnabled = false,false,false
local laggerToggled = false
local lastMoveDir = Vector3.new(0,0,0)
local MOVE_KEYS={[Enum.KeyCode.W]=true,[Enum.KeyCode.A]=true,[Enum.KeyCode.S]=true,[Enum.KeyCode.D]=true,
	[Enum.KeyCode.Up]=true,[Enum.KeyCode.Left]=true,[Enum.KeyCode.Down]=true,[Enum.KeyCode.Right]=true}
local desyncNormalSpeed = 60
local desyncCarrySpeed = 30
local desyncSpeedMode = false
local medusaCounterEnabled = false
local batCounterEnabled = false
local batCounterWasEnabled = false
local _batCounterV2Was = false
local batCounterV2Enabled = false
local setBatCounterV2Visual = nil
local batCounterV2Debounce = false
local batCounterV2LastUsed = 0
local batCounterV2Conns = {}
local unwalkEnabled = false
local lastHealth,medusaDebounce,medusaLastUsed,dropActive = 100,false,0,false
local autoLeftEnabled,autoRightEnabled = false,false
local autoLeftSetVisual,autoRightSetVisual = nil,nil
local speedLabel = nil
local medusaConns = {}
local batCounterConns = {}
local aimbotEnabled = false
local aimbotMoveEnabled = false
local aimbotSetVisual = nil
local autoSwingEnabled = false
local autoSwingSetVisual = nil
local antiLagEnabled = false
local removeAccessoriesEnabled = false
local descendantConnection = nil
local accessoryConnection = nil
local unwalkSavedAnimate = nil
local _anyKeyListening = false
local setK7AntiLagVisual = nil

-- ═══ LOCK IN MODE (Tryhard Animation from Vyse) ═══
local lockInEnabled = false
local lockInHeartbeatConn = nil
local originalAnims = nil
local setLockInVisual = nil

local Anims = {
	idle1    = "rbxassetid://133806214992291",
	idle2    = "rbxassetid://94970088341563",
	walk     = "rbxassetid://707897309",
	run      = "rbxassetid://707861613",
	jump     = "rbxassetid://116936326516985",
	fall     = "rbxassetid://116936326516985",
	climb    = "rbxassetid://116936326516985",
	swim     = "rbxassetid://116936326516985",
	swimidle = "rbxassetid://116936326516985",
}

local function isPackAnim(id)
	if not id then return false end
	for _, v in pairs(Anims) do if v == id then return true end end
	return false
end

local function saveOriginalAnims(char)
	local animate = char:FindFirstChild("Animate")
	if not animate then return end
	local function g(obj) return obj and obj.AnimationId or nil end
	local ids = {
		idle1    = g(animate.idle     and animate.idle.Animation1),
		idle2    = g(animate.idle     and animate.idle.Animation2),
		walk     = g(animate.walk     and animate.walk.WalkAnim),
		run      = g(animate.run      and animate.run.RunAnim),
		jump     = g(animate.jump     and animate.jump.JumpAnim),
		fall     = g(animate.fall     and animate.fall.FallAnim),
		climb    = g(animate.climb    and animate.climb.ClimbAnim),
		swim     = g(animate.swim     and animate.swim.Swim),
		swimidle = g(animate.swimidle and animate.swimidle.SwimIdle),
	}
	if not isPackAnim(ids.walk) then originalAnims = ids end
end

local function applyAnimPack(char)
	local animate = char:FindFirstChild("Animate")
	if not animate then return end
	local function s(obj, id) if obj then obj.AnimationId = id end end
	s(animate.idle     and animate.idle.Animation1,     Anims.idle1)
	s(animate.idle     and animate.idle.Animation2,     Anims.idle2)
	s(animate.walk     and animate.walk.WalkAnim,       Anims.walk)
	s(animate.run      and animate.run.RunAnim,         Anims.run)
	s(animate.jump     and animate.jump.JumpAnim,       Anims.jump)
	s(animate.fall     and animate.fall.FallAnim,       Anims.fall)
	s(animate.climb    and animate.climb.ClimbAnim,     Anims.climb)
	s(animate.swim     and animate.swim.Swim,           Anims.swim)
	s(animate.swimidle and animate.swimidle.SwimIdle,   Anims.swimidle)
end

local function restoreOriginalAnims(char)
	if not originalAnims then return end
	local animate = char:FindFirstChild("Animate")
	if not animate then return end
	local function s(obj, id) if obj and id then obj.AnimationId = id end end
	s(animate.idle     and animate.idle.Animation1,     originalAnims.idle1)
	s(animate.idle     and animate.idle.Animation2,     originalAnims.idle2)
	s(animate.walk     and animate.walk.WalkAnim,       originalAnims.walk)
	s(animate.run      and animate.run.RunAnim,         originalAnims.run)
	s(animate.jump     and animate.jump.JumpAnim,       originalAnims.jump)
	s(animate.fall     and animate.fall.FallAnim,       originalAnims.fall)
	s(animate.climb    and animate.climb.ClimbAnim,     originalAnims.climb)
	s(animate.swim     and animate.swim.Swim,           originalAnims.swim)
	s(animate.swimidle and animate.swimidle.SwimIdle,   originalAnims.swimidle)
	local hum2 = char:FindFirstChildOfClass("Humanoid")
	if hum2 then
		for _, track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end
		hum2:ChangeState(Enum.HumanoidStateType.Running)
	end
end

local function startLockIn()
	if lockInHeartbeatConn then lockInHeartbeatConn:Disconnect(); lockInHeartbeatConn = nil end
	local char = LP.Character
	if char then
		saveOriginalAnims(char)
		applyAnimPack(char)
		local hum2 = char:FindFirstChildOfClass("Humanoid")
		if hum2 then
			for _, track in ipairs(hum2:GetPlayingAnimationTracks()) do track:Stop(0) end
			hum2:ChangeState(Enum.HumanoidStateType.Running)
		end
	end
	lockInHeartbeatConn = RunService.Heartbeat:Connect(function()
		if not lockInEnabled then return end
		local c = LP.Character
		if c then applyAnimPack(c) end
	end)
end

local function stopLockIn()
	if lockInHeartbeatConn then lockInHeartbeatConn:Disconnect(); lockInHeartbeatConn = nil end
	local char = LP.Character
	if char then restoreOriginalAnims(char) end
end

-- ═══ K7 ANTI-LAG (no GUI) ═══


-- ═══ COLOR THEME STATE ═══
local THEME_MODE = "red" -- "red", "black", "white"

-- Theme palette builder
local function getThemePalette(mode)
	if mode == "black" then
		return {
			BG    = Color3.fromRGB(2, 2, 2),
			BG2   = Color3.fromRGB(6, 6, 6),
			CARD  = Color3.fromRGB(12, 12, 12),
			HOV   = Color3.fromRGB(22, 22, 22),
			ACCENT= Color3.fromRGB(180, 180, 180),
			ACCDIM= Color3.fromRGB(100, 100, 100),
			STROKE= Color3.fromRGB(80, 80, 80),
			W     = Color3.fromRGB(240,240,240),
			DIM   = Color3.fromRGB(70, 70, 70),
			INP   = Color3.fromRGB(8, 8, 8),
			OFF   = Color3.fromRGB(18, 18, 18),
			MB_C_OFF   = Color3.fromRGB(8, 8, 8),
			MB_C_ON    = Color3.fromRGB(90, 90, 90),
			MB_BRD_OFF = Color3.fromRGB(50, 50, 50),
			MB_BRD_ON  = Color3.fromRGB(200, 200, 200),
			MB_TXT_OFF = Color3.fromRGB(150, 150, 150),
			MB_TXT_ON  = Color3.fromRGB(255, 255, 255),
			PB_BG      = Color3.fromRGB(20, 20, 20),
			SECT_CLR   = Color3.fromRGB(160, 160, 160),
		}
	elseif mode == "white" then
		return {
			BG    = Color3.fromRGB(230, 230, 230),
			BG2   = Color3.fromRGB(215, 215, 215),
			CARD  = Color3.fromRGB(200, 200, 200),
			HOV   = Color3.fromRGB(185, 185, 185),
			ACCENT= Color3.fromRGB(30, 30, 30),
			ACCDIM= Color3.fromRGB(80, 80, 80),
			STROKE= Color3.fromRGB(60, 60, 60),
			W     = Color3.fromRGB(10, 10, 10),
			DIM   = Color3.fromRGB(100, 100, 100),
			INP   = Color3.fromRGB(190, 190, 190),
			OFF   = Color3.fromRGB(175, 175, 175),
			MB_C_OFF   = Color3.fromRGB(200, 200, 200),
			MB_C_ON    = Color3.fromRGB(80, 80, 80),
			MB_BRD_OFF = Color3.fromRGB(120, 120, 120),
			MB_BRD_ON  = Color3.fromRGB(20, 20, 20),
			MB_TXT_OFF = Color3.fromRGB(60, 60, 60),
			MB_TXT_ON  = Color3.fromRGB(10, 10, 10),
			PB_BG      = Color3.fromRGB(185, 185, 185),
			SECT_CLR   = Color3.fromRGB(50, 50, 50),
		}
	else -- red (default)
		return {
			BG    = Color3.fromRGB(6, 0, 0),
			BG2   = Color3.fromRGB(10, 1, 1),
			CARD  = Color3.fromRGB(18, 3, 3),
			HOV   = Color3.fromRGB(35, 5, 5),
			ACCENT= Color3.fromRGB(210, 25, 25),
			ACCDIM= Color3.fromRGB(150, 18, 18),
			STROKE= Color3.fromRGB(190, 25, 25),
			W     = Color3.fromRGB(240,240,240),
			DIM   = Color3.fromRGB(100,25,25),
			INP   = Color3.fromRGB(10, 1, 1),
			OFF   = Color3.fromRGB(30, 4, 4),
			MB_C_OFF   = Color3.fromRGB(8, 0, 0),
			MB_C_ON    = Color3.fromRGB(160, 15, 15),
			MB_BRD_OFF = Color3.fromRGB(100, 12, 12),
			MB_BRD_ON  = Color3.fromRGB(220, 40, 40),
			MB_TXT_OFF = Color3.fromRGB(200, 60, 60),
			MB_TXT_ON  = Color3.fromRGB(255, 255, 255),
			PB_BG      = Color3.fromRGB(20, 2, 2),
			SECT_CLR   = Color3.fromRGB(200, 50, 50),
		}
	end
end

local ICON_ID_RED   = "rbxassetid://124615334262719"
local ICON_ID_BLACK = "rbxassetid://98706655274368"
local ICON_ID_WHITE = "rbxassetid://104402968118489"
local function getIconId()
	if THEME_MODE == "black" then return ICON_ID_BLACK
	elseif THEME_MODE == "white" then return ICON_ID_WHITE
	else return ICON_ID_RED end
end
local ICON_ID = ICON_ID_RED

local POS = {
	L1=Vector3.new(-476.48,-6.28,92.73), L2=Vector3.new(-483.12,-4.95,94.80),
	R1=Vector3.new(-476.16,-6.52,25.62), R2=Vector3.new(-483.04,-5.09,23.14),
}
local AP_L_FACE = Vector3.new(-482.25,-4.96,92.09)
local AP_R_FACE = Vector3.new(-482.06,-6.93,35.47)

local KB = {
	DropBrainrot={kb=Enum.KeyCode.X,gp=nil},
	AutoLeft    ={kb=Enum.KeyCode.Z,gp=nil},
	AutoRight   ={kb=Enum.KeyCode.C,gp=nil},
	AimBot      ={kb=Enum.KeyCode.E,gp=nil},
	TPFloor     ={kb=Enum.KeyCode.F,gp=nil},
	GuiHide     ={kb=Enum.KeyCode.LeftControl,gp=nil},
	SpeedToggle ={kb=Enum.KeyCode.Q,gp=nil},
	LaggerToggle={kb=Enum.KeyCode.R,gp=nil}
}

local Steal = {
	AutoStealEnabled=false, StealRadius=20, StealDuration=0.25,
	Data={}, plotCache={}, plotCacheTime={}, cachedPrompts={}, promptCacheTime=0,
}
local isStealing = false
local stealStartTime = nil
local lastStealTick = 0
local Conns = {autoSteal=nil,antiRag=nil,anchor={},progress=nil,aimbot=nil,batCounter=nil}
local PLOT_CACHE_DURATION = 2
local PROMPT_CACHE_REFRESH = 0.15
local STEAL_COOLDOWN = 0.1
local MEDUSA_COOLDOWN = 25
local BAT_COUNTER_COOLDOWN = 0
local batCounterDebounce = false
local batCounterLastUsed = 0
local progressRadLbl,progressFill,progressPct
local modeValLbl
local _aimTarget = nil
local _aimLastScan = 0

local VYSE_AIMBOT_SPEED = 56.5
local VYSE_HIT_DIST = 5
local SWING_COOLDOWN = 0.08
local hittingCooldown = false

local desyncEnabled = false
local setDesyncVisual = nil
local desyncActive = false
local desyncSpeedConn = nil
local G_desyncAnimate = nil

local fpsBoostEnabled = false

-- ═══ KYRIE DESYNC ═══
local function applyDesyncFFlags(normalSpd, carrySpd)
	pcall(function()
		local setff = syn and syn.set_fflag or (setfflag or nil)
		if setff then
			local flags = {"FFlagDisableGroupGameService","FFlagDisableGroupPassService","FFlagDisableGroupBadgeService",
				"FFlagDisableGroupCurrencyService","FFlagDisableGroupDataStoreService","FFlagDisableGroupAnalyticsService",
				"FFlagDisableGroupLogService","FFlagDisableGroupReportService","FFlagDisableGroupRankService",
				"FFlagDisableGroupMembershipService","FFlagDisableGroupWallService","FFlagDisableGroupAuditService",
				"FFlagDisableGroupRelationshipService","FFlagDisableGroupNotificationService","FFlagDisableGroupPermissionService",
				"FFlagDisableGroupRoleService","FFlagDisableGroupSearchService","FFlagDisableGroupEventService",
				"FFlagDisableGroupInviteService","FFlagDisableGroupShoutService"}
			for _,f in ipairs(flags) do pcall(function() setff(f,"true") end) end
			pcall(function() setff("DFIntUserCameraMaxZoomDistance","2000") end)
		end
	end)
	if desyncSpeedConn then desyncSpeedConn:Disconnect(); desyncSpeedConn = nil end
	desyncSpeedConn = RunService.Heartbeat:Connect(function()
		if not desyncEnabled then if desyncSpeedConn then desyncSpeedConn:Disconnect(); desyncSpeedConn=nil end; return end
		local c = LP.Character; if not c then return end
		local hrp = c:FindFirstChild("HumanoidRootPart"); if not hrp then return end
		local hum = c:FindFirstChildOfClass("Humanoid"); if not hum then return end
		local md = hum.MoveDirection
		local targetSpd = desyncSpeedMode and desyncCarrySpeed or desyncNormalSpeed
		if md.Magnitude > 0 then hrp.AssemblyLinearVelocity = Vector3.new(md.X*targetSpd, hrp.AssemblyLinearVelocity.Y, md.Z*targetSpd) end
	end)
end

local function enableDesync()
	pcall(function() if raknet then raknet.desync(true) end end)
	local c = LP.Character; if not c then return end
	local hum = c:FindFirstChildOfClass("Humanoid")
	if hum then for _,t in ipairs(hum:GetPlayingAnimationTracks()) do t:Stop() end; hum.WalkSpeed=0; hum.JumpPower=0 end
	local anim = c:FindFirstChild("Animate")
	if anim then G_desyncAnimate = anim:Clone(); anim:Destroy() end
	applyDesyncFFlags(desyncNormalSpeed, desyncCarrySpeed)
end

local function disableDesync()
	pcall(function() if raknet then raknet.desync(false) end end)
	if desyncSpeedConn then desyncSpeedConn:Disconnect(); desyncSpeedConn = nil end
	local c = LP.Character
	if c then
		local hum = c:FindFirstChildOfClass("Humanoid"); if hum then hum.WalkSpeed=16; hum.JumpPower=50 end
		if G_desyncAnimate then G_desyncAnimate:Clone().Parent=c; G_desyncAnimate=nil end
	end
end

-- ═══ COLT INSTA STEAL ═══
local function isMyPlotByName(plotName)
	local ct = tick()
	if Steal.plotCache[plotName] and (ct-(Steal.plotCacheTime[plotName] or 0))<PLOT_CACHE_DURATION then return Steal.plotCache[plotName] end
	local plots = workspace:FindFirstChild("Plots")
	if not plots then Steal.plotCache[plotName]=false;Steal.plotCacheTime[plotName]=ct;return false end
	local plot = plots:FindFirstChild(plotName)
	if not plot then Steal.plotCache[plotName]=false;Steal.plotCacheTime[plotName]=ct;return false end
	local sign = plot:FindFirstChild(">Z-J('")
	if sign then
		local yb = sign:FindFirstChild("YourBase")
		if yb and yb:IsA("BillboardGui") then
			local r = yb.Enabled==true;Steal.plotCache[plotName]=r;Steal.plotCacheTime[plotName]=ct;return r
		end
	end
	Steal.plotCache[plotName]=false;Steal.plotCacheTime[plotName]=ct;return false
end

local function findNearestPrompt()
	local char = LP.Character;if not char then return nil end
	local root = char:FindFirstChild("HumanoidRootPart");if not root then return nil end
	local ct = tick()
	if ct-Steal.promptCacheTime<PROMPT_CACHE_REFRESH and #Steal.cachedPrompts>0 then
		local np,nd = nil,math.huge
		for _,data in ipairs(Steal.cachedPrompts) do
			if data.spawn then
				local dist = (data.spawn.Position-root.Position).Magnitude
				if dist<=Steal.StealRadius and dist<nd then np=data.prompt;nd=dist end
			end
		end
		if np then return np end
	end
	Steal.cachedPrompts={};Steal.promptCacheTime=ct
	local plots = workspace:FindFirstChild("Plots");if not plots then return nil end
	local np,nd = nil,math.huge
	for _,plot in ipairs(plots:GetChildren()) do
		if isMyPlotByName(plot.Name) then continue end
		local pods = plot:FindFirstChild("AnimalPodiums");if not pods then continue end
		for _,pod in ipairs(pods:GetChildren()) do
			pcall(function()
				local base = pod:FindFirstChild("Base");local sp = base and base:FindFirstChild("Spawn")
				if sp then
					local att = sp:FindFirstChild("PromptAttachment")
					if att then
						for _,child in ipairs(att:GetChildren()) do
							if child:IsA("ProximityPrompt") then
								local dist = (sp.Position-root.Position).Magnitude
								table.insert(Steal.cachedPrompts,{prompt=child,spawn=sp,name=pod.Name})
								if dist<=Steal.StealRadius and dist<nd then np=child;nd=dist end
								break
							end
						end
					end
				end
			end)
		end
	end
	return np
end

local function resetProgressBar()
	if progressPct then progressPct.Text="0%" end
	if progressFill then progressFill.Size=UDim2.new(0,0,1,0) end
end

local function executeSteal(prompt)
	local ct = tick()
	if ct-lastStealTick<STEAL_COOLDOWN then return end
	if isStealing then return end
	if not Steal.Data[prompt] then
		Steal.Data[prompt]={hold={},trigger={},ready=true}
		pcall(function()
			if getconnections then
				for _,c in ipairs(getconnections(prompt.PromptButtonHoldBegan)) do if c.Function then table.insert(Steal.Data[prompt].hold,c.Function) end end
				for _,c in ipairs(getconnections(prompt.Triggered)) do if c.Function then table.insert(Steal.Data[prompt].trigger,c.Function) end end
			else Steal.Data[prompt].useFallback=true end
		end)
	end
	local data = Steal.Data[prompt];if not data.ready then return end
	data.ready=false;isStealing=true;stealStartTime=ct;lastStealTick=ct
	if Conns.progress then Conns.progress:Disconnect() end
	Conns.progress = RunService.Heartbeat:Connect(function()
		if not isStealing then Conns.progress:Disconnect();return end
		local prog = math.clamp((tick()-stealStartTime)/Steal.StealDuration,0,1)
		if progressFill then progressFill.Size=UDim2.new(prog,0,1,0) end
		if progressPct then progressPct.Text=math.floor(prog*100).."%"  end
	end)
	task.spawn(function()
		local ok = false
		pcall(function()
			if not data.useFallback then
				for _,fn in ipairs(data.hold) do task.spawn(fn) end
				task.wait(Steal.StealDuration)
				for _,fn in ipairs(data.trigger) do task.spawn(fn) end
				ok=true
			end
		end)
		if not ok and fireproximityprompt then pcall(function() fireproximityprompt(prompt);ok=true end) end
		if not ok then pcall(function() prompt:InputHoldBegin();task.wait(Steal.StealDuration);prompt:InputHoldEnd() end) end
		task.wait(Steal.StealDuration*0.3)
		if Conns.progress then Conns.progress:Disconnect() end
		resetProgressBar()
		task.wait(0.05);data.ready=true;isStealing=false
	end)
end

local function startAutoSteal()
	if Conns.autoSteal then return end
	Conns.autoSteal = RunService.Heartbeat:Connect(function()
		if not Steal.AutoStealEnabled or isStealing then return end
		local p = findNearestPrompt();if p then executeSteal(p) end
	end)
end

local function stopAutoSteal()
	if Conns.autoSteal then Conns.autoSteal:Disconnect();Conns.autoSteal=nil end
	isStealing=false;lastStealTick=0
	Steal.plotCache={};Steal.plotCacheTime={};Steal.cachedPrompts={}
	resetProgressBar()
end

RunService.Stepped:Connect(function()
	for _,p in ipairs(Players:GetPlayers()) do
		if p~=LP and p.Character then
			for _,part in ipairs(p.Character:GetDescendants()) do
				if part:IsA("BasePart") then part.CanCollide=false end
			end
		end
	end
end)

RunService.RenderStepped:Connect(function()
	local char=LP.Character;if not char then return end
	local hum=char:FindFirstChildOfClass("Humanoid")
	local hrp=char:FindFirstChild("HumanoidRootPart")
	if not hum or not hrp then return end
	if not desyncEnabled then
		if not(autoLeftEnabled or autoRightEnabled) and not(aimbotEnabled and aimbotMoveEnabled) then
			local md=hum.MoveDirection
			local spd=laggerToggled and LAGGER_SPEED or (speedMode and CS or NS)
			if md.Magnitude>0 then
				lastMoveDir=md
				hrp.Velocity=Vector3.new(md.X*spd,hrp.Velocity.Y,md.Z*spd)
			elseif antiRagdollEnabled and lastMoveDir.Magnitude>0 then
				local anyHeld=false
				for key in pairs(MOVE_KEYS) do if UIS:IsKeyDown(key) then anyHeld=true;break end end
				if anyHeld then hrp.Velocity=Vector3.new(lastMoveDir.X*spd,hrp.Velocity.Y,lastMoveDir.Z*spd) end
			end
		end
	end
	if speedLabel then speedLabel.Text=string.format("Speed: %.1f",Vector3.new(hrp.Velocity.X,0,hrp.Velocity.Z).Magnitude) end
end)

local alConn,arConn = nil,nil
local alPhase,arPhase = 1,1

local function stopAutoLeft()
	if alConn then alConn:Disconnect();alConn=nil end;alPhase=1
	local char=LP.Character;if char then local h=char:FindFirstChildOfClass("Humanoid");if h then h:Move(Vector3.zero,false) end end
end
local function stopAutoRight()
	if arConn then arConn:Disconnect();arConn=nil end;arPhase=1
	local char=LP.Character;if char then local h=char:FindFirstChildOfClass("Humanoid");if h then h:Move(Vector3.zero,false) end end
end

local function startAutoLeft()
	if alConn then alConn:Disconnect() end;alPhase=1
	alConn=RunService.Heartbeat:Connect(function()
		if not autoLeftEnabled then return end
		local char=LP.Character;if not char then return end
		local hrp=char:FindFirstChild("HumanoidRootPart")
		local hum=char:FindFirstChildOfClass("Humanoid")
		if not hrp or not hum then return end
		local spd=NS
		if alPhase==1 then
			local tgt=Vector3.new(POS.L1.X,hrp.Position.Y,POS.L1.Z)
			if (tgt-hrp.Position).Magnitude<1 then alPhase=2; return end
			local d=POS.L1-hrp.Position; local mv=Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false); hrp.Velocity=Vector3.new(mv.X*spd,hrp.Velocity.Y,mv.Z*spd)
		elseif alPhase==2 then
			local tgt=Vector3.new(POS.L2.X,hrp.Position.Y,POS.L2.Z)
			if (tgt-hrp.Position).Magnitude<1 then
				hum:Move(Vector3.zero,false);hrp.Velocity=Vector3.zero
				autoLeftEnabled=false;if alConn then alConn:Disconnect();alConn=nil end
				alPhase=1;if autoLeftSetVisual then autoLeftSetVisual(false) end
				local dir=Vector3.new(AP_L_FACE.X,hrp.Position.Y,AP_L_FACE.Z)-hrp.Position
				if dir.Magnitude>0.01 then hrp.CFrame=CFrame.new(hrp.Position,hrp.Position+Vector3.new(dir.X,0,dir.Z).Unit) end
				return
			end
			local d=POS.L2-hrp.Position; local mv=Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false); hrp.Velocity=Vector3.new(mv.X*spd,hrp.Velocity.Y,mv.Z*spd)
		end
	end)
end

local function startAutoRight()
	if arConn then arConn:Disconnect() end;arPhase=1
	arConn=RunService.Heartbeat:Connect(function()
		if not autoRightEnabled then return end
		local char=LP.Character;if not char then return end
		local hrp=char:FindFirstChild("HumanoidRootPart")
		local hum=char:FindFirstChildOfClass("Humanoid")
		if not hrp or not hum then return end
		local spd=NS
		if arPhase==1 then
			local tgt=Vector3.new(POS.R1.X,hrp.Position.Y,POS.R1.Z)
			if (tgt-hrp.Position).Magnitude<1 then arPhase=2; return end
			local d=POS.R1-hrp.Position; local mv=Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false); hrp.Velocity=Vector3.new(mv.X*spd,hrp.Velocity.Y,mv.Z*spd)
		elseif arPhase==2 then
			local tgt=Vector3.new(POS.R2.X,hrp.Position.Y,POS.R2.Z)
			if (tgt-hrp.Position).Magnitude<1 then
				hum:Move(Vector3.zero,false);hrp.Velocity=Vector3.zero
				autoRightEnabled=false;if arConn then arConn:Disconnect();arConn=nil end
				arPhase=1;if autoRightSetVisual then autoRightSetVisual(false) end
				local dir=Vector3.new(AP_R_FACE.X,hrp.Position.Y,AP_R_FACE.Z)-hrp.Position
				if dir.Magnitude>0.01 then hrp.CFrame=CFrame.new(hrp.Position,hrp.Position+Vector3.new(dir.X,0,dir.Z).Unit) end
				return
			end
			local d=POS.R2-hrp.Position; local mv=Vector3.new(d.X,0,d.Z).Unit
			hum:Move(mv,false); hrp.Velocity=Vector3.new(mv.X*spd,hrp.Velocity.Y,mv.Z*spd)
		end
	end)
end

local function getSpeedLabelColor()
	if THEME_MODE == "black" then return Color3.fromRGB(200,200,200)
	elseif THEME_MODE == "white" then return Color3.fromRGB(30,30,30)
	else return Color3.fromRGB(180,20,40) end
end

local function setupSpeedIndicator(char)
	local head = char:WaitForChild("Head",5);if not head then return end
	local oldBB = head:FindFirstChild("ColtSpeedBB"); if oldBB then oldBB:Destroy() end
	local bb = Instance.new("BillboardGui",head)
	bb.Name = "ColtSpeedBB"
	bb.Size=UDim2.new(0,140,0,25);bb.StudsOffset=Vector3.new(0,3,0);bb.AlwaysOnTop=true
	speedLabel = Instance.new("TextLabel",bb)
	speedLabel.Size=UDim2.new(1,0,1,0);speedLabel.BackgroundTransparency=1
	speedLabel.Text="Speed: 0";speedLabel.TextColor3=getSpeedLabelColor()
	speedLabel.Font=Enum.Font.GothamBold;speedLabel.TextScaled=true
	speedLabel.TextStrokeTransparency=0;speedLabel.TextStrokeColor3=Color3.fromRGB(0,0,0)
end

local otherSpeedLabels = {}
local function setupOtherSpeedLabel(player)
	if player == LP then return end
	local function onChar(char)
		task.spawn(function()
			local head = char:WaitForChild("Head", 5)
			if not head then return end
			local oldBB = head:FindFirstChild("CursedSpeedBB")
			if oldBB then oldBB:Destroy() end
			local bb = Instance.new("BillboardGui", head)
			bb.Name = "CursedSpeedBB"
			bb.Size = UDim2.new(0, 120, 0, 20)
			bb.StudsOffset = Vector3.new(0, 3.2, 0)
			bb.AlwaysOnTop = true
			local lbl = Instance.new("TextLabel", bb)
			lbl.Size = UDim2.new(1, 0, 1, 0)
			lbl.BackgroundTransparency = 1
			lbl.Text = "0"
			lbl.TextColor3 = getSpeedLabelColor()
			lbl.Font = Enum.Font.GothamBold
			lbl.TextScaled = true
			lbl.TextStrokeTransparency = 0
			lbl.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
			otherSpeedLabels[player] = {lbl = lbl, char = char}
		end)
	end
	if player.Character then onChar(player.Character) end
	player.CharacterAdded:Connect(onChar)
end

Players.PlayerAdded:Connect(function(p) if p ~= LP then setupOtherSpeedLabel(p) end end)
Players.PlayerRemoving:Connect(function(p) otherSpeedLabels[p] = nil end)
for _, p in ipairs(Players:GetPlayers()) do if p ~= LP then setupOtherSpeedLabel(p) end end

RunService.Heartbeat:Connect(function()
	local now2=tick()
	for p, data in pairs(otherSpeedLabels) do
		if data.lbl and data.lbl.Parent and data.char then
			local hrp = data.char:FindFirstChild("HumanoidRootPart")
			if hrp then
				local dt=now2-(data.lastTick or now2)
				if dt>0 and data.lastPos then
					local delta=hrp.Position-data.lastPos
					local spd=Vector3.new(delta.X,0,delta.Z).Magnitude/dt
					data.lbl.Text=string.format("%.1f",spd)
				end
				data.lastPos=hrp.Position
				data.lastTick=now2
			end
		else
			otherSpeedLabels[p] = nil
		end
	end
end)

local function startAntiRagdoll()
	if Conns.antiRag then return end
	Conns.antiRag = RunService.Heartbeat:Connect(function()
		local char = LP.Character;if not char then return end
		local hum = char:FindFirstChildOfClass("Humanoid");local root=char:FindFirstChild("HumanoidRootPart")
		if hum then
			local st = hum:GetState()
			if st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown then
				hum:ChangeState(Enum.HumanoidStateType.Running)
				workspace.CurrentCamera.CameraSubject=hum
				pcall(function() local pm=LP.PlayerScripts:FindFirstChild(">Vz(v^");if pm then require(pm:FindFirstChild("ControlModule")):Enable() end end)
				if root then root.Velocity=Vector3.zero;root.RotVelocity=Vector3.zero end
			end
		end
		for _,obj in ipairs(char:GetDescendants()) do if obj:IsA("Motor6D") and not obj.Enabled then obj.Enabled=true end end
	end)
end

local function stopAntiRagdoll()
	if Conns.antiRag then Conns.antiRag:Disconnect();Conns.antiRag=nil end
end

local IJ_Conn = nil
local function startInfiniteJump()
	if IJ_Conn then IJ_Conn:Disconnect() end
	IJ_Conn = RunService.Heartbeat:Connect(function()
		if not infJumpEnabled then return end
		local char=LP.Character; if not char then return end
		local root=char:FindFirstChild("HumanoidRootPart")
		local hum=char:FindFirstChildOfClass("Humanoid")
		if not root or not hum then return end
		if root.AssemblyLinearVelocity.Y<-120 then
			root.AssemblyLinearVelocity=Vector3.new(root.AssemblyLinearVelocity.X,-120,root.AssemblyLinearVelocity.Z)
		end
		if hum.FloorMaterial==Enum.Material.Air then return end
		if not (UIS:IsKeyDown(Enum.KeyCode.Space) or UIS:IsKeyDown(Enum.KeyCode.ButtonA)) then return end
		local st=hum:GetState()
		if st==Enum.HumanoidStateType.Jumping or st==Enum.HumanoidStateType.Freefall then return end
		root.AssemblyLinearVelocity=Vector3.new(root.AssemblyLinearVelocity.X,55,root.AssemblyLinearVelocity.Z)
	end)
	UIS.JumpRequest:Connect(function()
		if not infJumpEnabled then return end
		local char=LP.Character; if not char then return end
		local root=char:FindFirstChild("HumanoidRootPart")
		if root then root.AssemblyLinearVelocity=Vector3.new(root.AssemblyLinearVelocity.X,55,root.AssemblyLinearVelocity.Z) end
	end)
end

local function stopInfiniteJump()
	if IJ_Conn then IJ_Conn:Disconnect(); IJ_Conn=nil end
end

local function startUnwalk()
	local c=LP.Character;if not c then return end
	local hum=c:FindFirstChildOfClass("Humanoid")
	if hum then for _,t in ipairs(hum:GetPlayingAnimationTracks()) do t:Stop() end end
	local anim=c:FindFirstChild("Animate")
	if anim then unwalkSavedAnimate=anim:Clone();anim:Destroy() end
end

local function stopUnwalk()
	local c=LP.Character
	if c and unwalkSavedAnimate then unwalkSavedAnimate:Clone().Parent=c;unwalkSavedAnimate=nil end
end

local function runTPFloor()
	pcall(function()
		local char=LP.Character;if not char then return end
		local hrp=char:FindFirstChild("HumanoidRootPart");if not hrp then return end
		local rp=RaycastParams.new();rp.FilterDescendantsInstances={char};rp.FilterType=Enum.RaycastFilterType.Exclude
		local res=workspace:Raycast(hrp.Position,Vector3.new(0,-1000,0),rp)
		if res then hrp.CFrame=CFrame.new(res.Position+Vector3.new(0,hrp.Size.Y/2+0.5,0)); hrp.AssemblyLinearVelocity=Vector3.zero end
	end)
end

-- Instant Reset removed

local _dropConns = {}
local dropBrainrotActive = false
local DROP_AUTO_OFF_DELAY = 0.15

local function stopDropBrainrot()
	dropBrainrotActive = false
	for _,cn in ipairs(_dropConns) do pcall(function() cn:Disconnect() end) end; _dropConns = {}
end

local function runDrop()
	if dropBrainrotActive then return end; dropBrainrotActive = true
	task.spawn(function()
		local colConn = RunService.Stepped:Connect(function()
			if not dropBrainrotActive then return end
			for _,p in ipairs(Players:GetPlayers()) do
				if p~=LP and p.Character then
					for _,part in ipairs(p.Character:GetChildren()) do if part:IsA("BasePart") then part.CanCollide=false end end
				end
			end
		end)
		table.insert(_dropConns, colConn)
		task.spawn(function()
			while dropBrainrotActive do
				RunService.Heartbeat:Wait()
				local c=LP.Character; local root=c and c:FindFirstChild("HumanoidRootPart")
				if not root then continue end
				local vel=root.Velocity
				root.Velocity=vel*10000+Vector3.new(0,10000,0)
				RunService.RenderStepped:Wait()
				if root and root.Parent then root.Velocity=vel end
				RunService.Stepped:Wait()
				if root and root.Parent then root.Velocity=vel+Vector3.new(0,0.1,0) end
			end
		end)
		task.wait(DROP_AUTO_OFF_DELAY); stopDropBrainrot()
	end)
end

local function applyFPSBoost()
	pcall(function() setfpscap(999999999) end)
	local function pO(v) pcall(function()
		if v:IsA("Model") then v.LevelOfDetail=Enum.ModelLevelOfDetail.Disabled; v.ModelStreamingMode=Enum.ModelStreamingMode.Nonatomic
		elseif v:IsA("MeshPart") then v.CastShadow=false; v.DoubleSided=false; v.RenderFidelity=Enum.RenderFidelity.Performance
		elseif v:IsA("BasePart") then v.CastShadow=false; v.Material=Enum.Material.Plastic; v.Reflectance=0
		elseif v:IsA("Decal") or v:IsA("Texture") then v.Transparency=1
		elseif v:IsA("SpecialMesh") then v.TextureId=""
		elseif v:IsA("Fire") or v:IsA("SpotLight") or v:IsA("Smoke") or v:IsA("Sparkles") or v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") then v.Enabled=false
		elseif v:IsA("SurfaceAppearance") or v:IsA("1^&Uj{") then v:Destroy()
		elseif v:IsA("Attachment") then v.Visible=false end
	end) end
	for _,v in pairs(workspace:GetDescendants()) do pO(v) end
	pcall(function()
		local L=game:GetService("Lighting")
		for _,v in pairs(L:GetDescendants()) do pcall(function() if v:IsA("Sky") or v:IsA("Atmosphere") or v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("Clouds") or v:IsA("PostEffect") or v:IsA("ColorCorrectionEffect") then v:Destroy() end end) end
		pcall(function() sethiddenproperty(L,"Technology",Enum.Technology.Legacy) end)
		L.GlobalShadows=false; L.FogEnd=9e9; L.Brightness=0
		local ter=workspace:FindFirstChildOfClass("Terrain")
		if ter then pcall(function() sethiddenproperty(ter,"Decoration",false) end); ter.WaterReflectance=0; ter.WaterTransparency=0.7; ter.WaterWaveSize=0; ter.WaterWaveSpeed=0 end
	end)
	workspace.DescendantAdded:Connect(function(v) if fpsBoostEnabled then task.spawn(pO,v) end end)
end

local function findAnyTool()
	local c=LP.Character
	if c then for _,v in ipairs(c:GetChildren()) do if v:IsA("Tool") then return v end end end
	local bp=LP:FindFirstChildOfClass("Backpack")
	if bp then for _,v in ipairs(bp:GetChildren()) do if v:IsA("Tool") then return v end end end
	return nil
end

local function getClosestPlayer()
	local char=LP.Character; if not char then return nil,math.huge end
	local hrp=char:FindFirstChild("HumanoidRootPart"); if not hrp then return nil,math.huge end
	local cp,cd=nil,math.huge
	for _,p in pairs(Players:GetPlayers()) do
		if p~=LP and p.Character then
			local tr=p.Character:FindFirstChild("HumanoidRootPart")
			local ph=p.Character:FindFirstChildOfClass("Humanoid")
			if tr and ph and ph.Health>0 then
				local d=(hrp.Position-tr.Position).Magnitude
				if d<cd then cd=d; cp=p end
			end
		end
	end
	return cp,cd
end

local function tryHitBat()
	if hittingCooldown then return end; hittingCooldown=true
	pcall(function()
		local c=LP.Character; if not c then return end
		local hum2=c:FindFirstChildOfClass("Humanoid")
		local tool=findAnyTool()
		if tool then
			if tool.Parent~=c and hum2 then pcall(function() hum2:EquipTool(tool) end) end
			local remote=tool:FindFirstChildOfClass("E騵/z{")
			if remote then pcall(function() remote:FireServer() end)
			else pcall(function() tool:Activate() end) end
		end
	end)
	task.delay(SWING_COOLDOWN, function() hittingCooldown=false end)
end

local function startBatAimbot()
	if Conns.aimbot then return end
	Conns.aimbot = RunService.Heartbeat:Connect(function()
		if not aimbotEnabled then return end
		local c=LP.Character; if not c then return end
		local root=c:FindFirstChild("HumanoidRootPart"); if not root then return end
		local hum2=c:FindFirstChildOfClass("Humanoid"); if not hum2 then return end
		local target,dist=getClosestPlayer()
		if target and target.Character then
			local tr=target.Character:FindFirstChild("HumanoidRootPart")
			if tr then
				local fp=tr.Position+tr.CFrame.LookVector*1.5
				local dir=(fp-root.Position).Unit
				root.AssemblyLinearVelocity=Vector3.new(dir.X*VYSE_AIMBOT_SPEED,dir.Y*VYSE_AIMBOT_SPEED,dir.Z*VYSE_AIMBOT_SPEED)
				if dist<=VYSE_HIT_DIST and autoSwingEnabled then tryHitBat() end
			end
		else root.AssemblyLinearVelocity=Vector3.zero end
	end)
end

local function stopBatAimbot()
	if Conns.aimbot then Conns.aimbot:Disconnect(); Conns.aimbot=nil end
	local c=LP.Character; local root=c and c:FindFirstChild("HumanoidRootPart")
	if root then root.AssemblyLinearVelocity=Vector3.zero end; hittingCooldown=false
end

local function enableAimbot()
	if autoLeftEnabled then autoLeftEnabled=false;if autoLeftSetVisual then autoLeftSetVisual(false) end;stopAutoLeft() end
	if autoRightEnabled then autoRightEnabled=false;if autoRightSetVisual then autoRightSetVisual(false) end;stopAutoRight() end
	local char=LP.Character
	if char then
		local hum=char:FindFirstChildOfClass("Humanoid")
		local bp=LP:FindFirstChild("Backpack")
		local bat=(bp and bp:FindFirstChild("Bat")) or char:FindFirstChild("Bat")
		if bat and hum then hum:EquipTool(bat) end
	end
	aimbotMoveEnabled=true
	startBatAimbot()
end

local function disableAimbot(autoSwingRow)
	aimbotMoveEnabled=false
	stopBatAimbot()
	local char=LP.Character
	local hum=char and char:FindFirstChildOfClass("Humanoid")
	local root=char and char:FindFirstChild("HumanoidRootPart")
	if hum then hum.AutoRotate=true end
	if root then
		root.AssemblyLinearVelocity=Vector3.new(0,root.AssemblyLinearVelocity.Y,0)
	end
	autoSwingEnabled=false
	if autoSwingSetVisual then autoSwingSetVisual(false) end
	if autoSwingRow then autoSwingRow.Visible=false end
end

local BAT_COUNTER_SLAP_LIST={"Bat","Slap","Iron Slap","Gold Slap","Diamond Slap","Emerald Slap","Ruby Slap","Dark Matter Slap","Flame Slap","Nuclear Slap","Galaxy Slap","Glitched Slap"}
local function findBatForCounter()
	local c=LP.Character; if not c then return nil end
	local bp=LP:FindFirstChildOfClass("Backpack")
	for _,name in ipairs(BAT_COUNTER_SLAP_LIST) do
		local t=c:FindFirstChild(name) or (bp and bp:FindFirstChild(name))
		if t then return t end
	end
	for _,ch in ipairs(c:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end
	if bp then for _,ch in ipairs(bp:GetChildren()) do if ch:IsA("Tool") and ch.Name:lower():find("bat") then return ch end end end
	return nil
end

local function swingBatForCounter(bat,char)
	local hum2=char:FindFirstChildOfClass("Humanoid")
	if bat.Parent~=char then if hum2 then pcall(function() hum2:EquipTool(bat) end) end; task.wait(0.05) end
	local remote=bat:FindFirstChildOfClass("E騵/z{") or bat:FindFirstChildOfClass("RemoteFunction")
	if remote and remote:IsA("E騵/z{") then
		pcall(function() remote:FireServer() end); task.wait(0.15); pcall(function() remote:FireServer() end)
	else pcall(function() bat:Activate() end); task.wait(0.15); pcall(function() bat:Activate() end) end
end

local setBatCounterVisual = nil

local function startBatCounter()
	if Conns.batCounter then return end
	Conns.batCounter = RunService.Heartbeat:Connect(function()
		if not batCounterEnabled then return end
		if batCounterDebounce then return end
		local char=LP.Character; if not char then return end
		local hum2=char:FindFirstChildOfClass("Humanoid"); if not hum2 then return end
		local st=hum2:GetState()
		local isRagdolled=st==Enum.HumanoidStateType.Physics or st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown
		if isRagdolled then
			batCounterDebounce=true
			task.spawn(function()
				local bat=findBatForCounter()
				if bat then swingBatForCounter(bat,char) end
				task.wait(0.5); batCounterDebounce=false
			end)
		end
	end)
end

local function stopBatCounter()
	if Conns.batCounter then Conns.batCounter:Disconnect(); Conns.batCounter=nil end
	batCounterDebounce=false
end

local function findMedusa()
	local char = LP.Character;if not char then return nil end
	for _,t in ipairs(char:GetChildren()) do if t:IsA("Tool") and t.Name:lower():find("medusa") then return t end end
	local bp = LP:FindFirstChild("Backpack")
	if bp then for _,t in ipairs(bp:GetChildren()) do if t:IsA("Tool") and t.Name:lower():find("medusa") then return t end end end
end

local function useMedusa()
	if medusaDebounce or tick()-medusaLastUsed<MEDUSA_COOLDOWN then return end
	local char = LP.Character;if not char then return end
	medusaDebounce=true
	local med = findMedusa()
	if med then
		if med.Parent~=char then local h=char:FindFirstChildOfClass("Humanoid");if h then h:EquipTool(med) end end
		pcall(function() med:Activate() end)
		medusaLastUsed=tick()
	end
	medusaDebounce=false
end

local function onAnchorChanged(part)
	return part:GetPropertyChangedSignal("Anchored"):Connect(function()
		if part.Anchored and part.Transparency==1 and medusaCounterEnabled then useMedusa() end
	end)
end

local function setupMedusa(char)
	for _,c in pairs(medusaConns) do pcall(function() c:Disconnect() end) end;medusaConns={}
	if not char then return end
	for _,part in ipairs(char:GetDescendants()) do if part:IsA("BasePart") then table.insert(medusaConns,onAnchorChanged(part)) end end
	table.insert(medusaConns,char.DescendantAdded:Connect(function(part)
		if part:IsA("BasePart") then table.insert(medusaConns,onAnchorChanged(part)) end
	end))
end

local autoTPEnabled=false
local autoTPHeight=20
local autoTPConn=nil
local setAutoTPVisual=nil

local function startAutoTP()
	if autoTPConn then task.cancel(autoTPConn);autoTPConn=nil end
	autoTPConn=task.spawn(function()
		while autoTPEnabled do
			task.wait(0.1)
			pcall(function()
				local char=LP.Character;if not char then return end
				local hrp=char:FindFirstChild("HumanoidRootPart");if not hrp then return end
				local hum2=char:FindFirstChildOfClass("Humanoid");if not hum2 then return end
				if hum2.FloorMaterial~=Enum.Material.Air then return end
				if hrp.Position.Y<autoTPHeight then return end
				hrp.CFrame=CFrame.new(hrp.Position.X,-8.80,hrp.Position.Z)
				hrp.AssemblyLinearVelocity=Vector3.zero
			end)
		end
	end)
end

local function stopAutoTP()
	autoTPEnabled=false
	if autoTPConn then task.cancel(autoTPConn);autoTPConn=nil end
end

local optimizerThreads = {}
local optimizerConns   = {}
local BK_FFLAGS = {
	["DFIntTaskSchedulerTargetFps"]=999,
	["DFIntTaskSchedulerTargetFpsMax"]=999,
	["FFlagDebugGraphicsPreferVulkan"]=true,
	["FFlagDisablePostFx"]=true,
	["FIntRenderShadowIntensity"]=0,
	["DFIntParticleMaxCount"]=0,
	["DFIntGlobalPointLightMaxCount"]=0,
}

local bkCleanedChars = {}
local function bkCleanChar(char)
	if not char then return end
	pcall(function()
		for _,child in ipairs(char:GetChildren()) do
			if child:IsA("Accessory") or child:IsA("Hat") then child:Destroy()
			elseif child:IsA("Shirt") or child:IsA("Pants") or child:IsA("ShirtGraphic") then child:Destroy()
			elseif child:IsA("BodyColors") then child:Destroy()
			elseif child:IsA("CharacterMesh") then child:Destroy()
			end
		end
		for _,child in ipairs(char:GetDescendants()) do
			pcall(function()
				if child:IsA("ParticleEmitter") or child:IsA("Trail") or child:IsA("Beam") then child:Destroy()
				elseif child:IsA("PointLight") or child:IsA("SpotLight") or child:IsA("SurfaceLight") then child:Destroy()
				elseif child:IsA("Fire") or child:IsA("Smoke") or child:IsA("Sparkles") then child:Destroy()
				elseif child:IsA("BasePart") then
					child.CastShadow=false;child.Material=Enum.Material.Plastic;child.Reflectance=0
				end
			end)
		end
	end)
	bkCleanedChars[char]=true
end

local function bkCleanBackpack(player)
	pcall(function()
		local bp=player:FindFirstChild("Backpack"); if not bp then return end
		for _,tool in ipairs(bp:GetChildren()) do
			if tool:IsA("Tool") then
				for _,desc in ipairs(tool:GetDescendants()) do
					pcall(function()
						if desc:IsA("ParticleEmitter") or desc:IsA("Trail") or desc:IsA("Beam")
							or desc:IsA("PointLight") or desc:IsA("SpotLight") or desc:IsA("SurfaceLight")
							or desc:IsA("Fire") or desc:IsA("Smoke") or desc:IsA("Sparkles") then
							desc:Destroy()
						end
					end)
				end
			end
		end
	end)
end

local function enableAntiLag()
	if antiLagEnabled then return end
	antiLagEnabled=true
	bkCleanedChars={}
	pcall(function() for flag,val in pairs(BK_FFLAGS) do pcall(function() setfflag(flag,tostring(val)) end) end end)
	pcall(function() local r=settings().Rendering; r.QualityLevel=Enum.QualityLevel.Level01 end)
	pcall(function()
		Lighting.GlobalShadows=false;Lighting.Brightness=3;Lighting.FogEnd=9e9
		for _,e in ipairs(Lighting:GetChildren()) do
			if e:IsA("PostEffect") or e:IsA("Atmosphere") then pcall(function() e:Destroy() end) end
		end
	end)
	pcall(function() setfpscap(999) end)
	for _,p in ipairs(Players:GetPlayers()) do
		bkCleanChar(p.Character)
		bkCleanBackpack(p)
	end
end

local function disableAntiLag()
	antiLagEnabled=false
	bkCleanedChars={}
	for _,c in ipairs(optimizerConns) do pcall(function() c:Disconnect() end) end
	optimizerConns={}
	for _,t in ipairs(optimizerThreads) do pcall(function() task.cancel(t) end) end
	optimizerThreads={}
	if descendantConnection then descendantConnection:Disconnect();descendantConnection=nil end
end

local function enableRemoveAccessories()
	removeAccessoriesEnabled=true
	for _,p in pairs(Players:GetPlayers()) do
		if p.Character then for _,obj in ipairs(p.Character:GetDescendants()) do if obj:IsA("Accessory") or obj:IsA("Hat") then obj:Destroy() end end end
	end
end

local function disableRemoveAccessories()
	removeAccessoriesEnabled=false
	if accessoryConnection then accessoryConnection:Disconnect();accessoryConnection=nil end
end

LP.CharacterAdded:Connect(function(char)
	lastHealth=100;task.wait(0.5)
	setupSpeedIndicator(char)
	if medusaCounterEnabled then setupMedusa(char) end
	if batCounterEnabled then startBatCounter() end
	if unwalkEnabled then task.wait(0.5);startUnwalk() end
	if lockInEnabled then
		task.wait(0.3)
		saveOriginalAnims(char)
		applyAnimPack(char)
	end
end)
if LP.Character then setupSpeedIndicator(LP.Character) end

local function saveConfig()
	local function ks(e) return {kb=e.kb and e.kb.Name or nil,gp=e.gp and e.gp.Name or nil} end
	local cfg = {
		normalSpeed=NS,carrySpeed=CS,
		dropBrainrotKey=ks(KB.DropBrainrot),autoLeftKey=ks(KB.AutoLeft),autoRightKey=ks(KB.AutoRight),
		aimbotKey=ks(KB.AimBot),laggerToggleKey=ks(KB.LaggerToggle),tpFloorKey=ks(KB.TPFloor),guiHideKey=ks(KB.GuiHide),
		speedToggleKey=ks(KB.SpeedToggle),
		grabRadius=Steal.StealRadius,stealDuration=Steal.StealDuration,
		antiRagdoll=antiRagdollEnabled,autoStealEnabled=Steal.AutoStealEnabled,
		infiniteJump=infJumpEnabled,medusaCounter=medusaCounterEnabled,
		batCounter=batCounterEnabled,
		carryMode=speedMode,laggerMode=laggerToggled,laggerSpeed=LAGGER_SPEED,
		autoTPEnabled=autoTPEnabled,autoTPHeight=autoTPHeight,
		aimbot=aimbotEnabled,aimbotMove=aimbotMoveEnabled,autoSwing=autoSwingEnabled,
		unwalkEnabled=unwalkEnabled,
		antiLag=antiLagEnabled,removeAccessories=removeAccessoriesEnabled,
		lockInEnabled=lockInEnabled,
		themeMode=THEME_MODE,
	}
	if writefile then pcall(function() writefile("ColtHubPC.json",HS:JSONEncode(cfg)) end) end
end

task.spawn(function() while task.wait(5) do saveConfig() end end)

local setInstaGrab,setInfJumpVisual,setAntiRagVisual,setMedusaVisual
local setUnwalkVisual,setAntiLagVisual,setRemoveAccVisual,setFpsBoostVisual
local normalBox,carryBox,laggerBox,radInput,floatHeightBox,stealDurBox

-- ═══ THEME APPLY FUNCTION (wires closed over in buildGui) ═══
local buildGui  -- forward declare so rebuildWithTheme can reference it

local function rebuildWithTheme(mode)
	THEME_MODE = mode
	ICON_ID = getIconId()
	saveConfig()
	-- Recolor own speed billboard
	if speedLabel then speedLabel.TextColor3 = getSpeedLabelColor() end
	-- Recolor other players' speed billboards
	for _, data in pairs(otherSpeedLabels) do
		if data.lbl then data.lbl.TextColor3 = getSpeedLabelColor() end
	end
	local old = game:GetService("CoreGui"):FindFirstChild("ColtHub")
	if old then old:Destroy() end
	local pg = LP:FindFirstChild("PlayerGui")
	if pg then
		local o = pg:FindFirstChild("ColtHub"); if o then o:Destroy() end
		local m = pg:FindFirstChild("ColtMobileGui"); if m then m:Destroy() end
	end
	buildGui()
end

buildGui = function()
	local P = getThemePalette(THEME_MODE)
	local BG    = P.BG
	local BG2   = P.BG2
	local CARD  = P.CARD
	local HOV   = P.HOV
	local RED   = P.ACCENT
	local REDDIM= P.ACCDIM
	local STROKE= P.STROKE
	local W     = P.W
	local DIM   = P.DIM
	local INP   = P.INP
	local OFF   = P.OFF
	local MB_C_OFF  = P.MB_C_OFF
	local MB_C_ON   = P.MB_C_ON
	local MB_BRD_OFF= P.MB_BRD_OFF
	local MB_BRD_ON = P.MB_BRD_ON
	local MB_TXT_OFF= P.MB_TXT_OFF
	local MB_TXT_ON = P.MB_TXT_ON

	local old=game:GetService("CoreGui"):FindFirstChild("ColtHub");if old then old:Destroy() end
	local pg=LP:FindFirstChild("PlayerGui");if pg then local o=pg:FindFirstChild("ColtHub");if o then o:Destroy() end end
	local gui=Instance.new("ScreenGui")
	gui.Name="ColtHub";gui.ResetOnSpawn=false;gui.DisplayOrder=10;gui.IgnoreGuiInset=true
	pcall(function() if syn and syn.protect_gui then syn.protect_gui(gui) end end)
	if not pcall(function() gui.Parent=game:GetService("CoreGui") end) then gui.Parent=LP:WaitForChild("PlayerGui") end

	local main=Instance.new("Frame",gui)
	main.Size=UDim2.new(0,300,0,510);main.Position=UDim2.new(0,20,0,20)
	main.BackgroundColor3=BG;main.BorderSizePixel=0;main.ClipsDescendants=false
	Instance.new("UICorner",main).CornerRadius=UDim.new(0,22)
	local mainStroke=Instance.new("UIStroke",main)
	mainStroke.Color=STROKE;mainStroke.Thickness=2

	local bgImg=Instance.new("ImageLabel",main)
	bgImg.Size=UDim2.new(1,0,1,0);bgImg.Position=UDim2.new(0,0,0,0)
	bgImg.BackgroundTransparency=1;bgImg.Image=getIconId()
	bgImg.ImageTransparency=0.88;bgImg.ScaleType=Enum.ScaleType.Crop
	bgImg.ZIndex=0
	Instance.new("UICorner",bgImg).CornerRadius=UDim.new(0,12)

	local function drag(f)
		local dn,ds,sp,di=false
		f.InputBegan:Connect(function(i)
			if i.UserInputType==Enum.UserInputType.MouseButton1 or i.UserInputType==Enum.UserInputType.Touch then
				dn=true;ds=i.Position;sp=f.Position
				i.Changed:Connect(function() if i.UserInputState==Enum.UserInputState.End then dn=false end end)
			end
		end)
		f.InputChanged:Connect(function(i)
			if i.UserInputType==Enum.UserInputType.MouseMovement or i.UserInputType==Enum.UserInputType.Touch then di=i end
		end)
		UIS.InputChanged:Connect(function(i)
			if i==di and dn then
				local nX=sp.X.Offset+(i.Position.X-ds.X)
				local nY=sp.Y.Offset+(i.Position.Y-ds.Y)
				f.Position=UDim2.new(sp.X.Scale,nX,sp.Y.Scale,nY)
			end
		end)
	end
	drag(main)

	local hdr=Instance.new("Frame",main)
	hdr.Size=UDim2.new(1,0,0,52);hdr.BackgroundColor3=BG2;hdr.BorderSizePixel=0
	Instance.new("UICorner",hdr).CornerRadius=UDim.new(0,12)

	local hdrIcon=Instance.new("ImageLabel",hdr)
	hdrIcon.Size=UDim2.new(0,26,0,26);hdrIcon.Position=UDim2.new(0,8,0.5,-13)
	hdrIcon.BackgroundColor3=RED;hdrIcon.BackgroundTransparency=0
	hdrIcon.Image=getIconId();hdrIcon.ImageTransparency=0.2;hdrIcon.ScaleType=Enum.ScaleType.Crop
	hdrIcon.BorderSizePixel=0;hdrIcon.ZIndex=3
	Instance.new("UICorner",hdrIcon).CornerRadius=UDim.new(0,5)

	local ttl=Instance.new("TextLabel",hdr)
	ttl.Size=UDim2.new(1,-80,1,0);ttl.Position=UDim2.new(0,42,0,0)
	ttl.BackgroundTransparency=1;ttl.Text="COLT HUB"
	ttl.TextColor3=Color3.fromRGB(235,235,235);ttl.Font=Enum.Font.GothamBlack;ttl.TextSize=15
	ttl.TextXAlignment=Enum.TextXAlignment.Left;ttl.ZIndex=3
	local sub=Instance.new("TextLabel",hdr)
	sub.Size=UDim2.new(1,-80,0,12);sub.Position=UDim2.new(0,42,0,28)
	sub.BackgroundTransparency=1;sub.Text="crimson edition"
	sub.TextColor3=REDDIM;sub.Font=Enum.Font.Gotham;sub.TextSize=9
	sub.TextXAlignment=Enum.TextXAlignment.Left;sub.ZIndex=3

	local closeBtn=Instance.new("TextButton",hdr)
	closeBtn.Size=UDim2.new(0,28,0,28);closeBtn.Position=UDim2.new(1,-34,0.5,-14)
	closeBtn.BackgroundColor3=BG2;closeBtn.BorderSizePixel=0
	closeBtn.Text="-";closeBtn.TextColor3=REDDIM;closeBtn.Font=Enum.Font.GothamBold;closeBtn.TextSize=22;closeBtn.ZIndex=4
	Instance.new("UICorner",closeBtn).CornerRadius=UDim.ne... (40 KB restante(s))
