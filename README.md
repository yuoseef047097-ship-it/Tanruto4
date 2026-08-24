-- tanruto4 script
-- نسخة كاملة: الأصدقاء أولاً + علامة "صديق" + قائمة لاعبين + TP/مشاهدة تغلق القائمة

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local old = playerGui:FindFirstChild("tanruto4 script")
if old then old:Destroy() end

local noclip = false
local infiniteJump = false
local flying = false
local espEnabled = false
local spectating = false
local godMode = false

local targetPlayer
local savedTPPoint

local speedList = {16,50,100,150}
local speedIndex = 1

local jumpList = {50,75,100,150,200}
local jumpIndex = 1

local flySpeeds = {40,60,100,150}
local flySpeedIndex = 2
local flySpeed = 60

local flyVelocity
local flyAttachment
local flyConnection
local godConnection

local upHeld = false
local downHeld = false

local espObjects = {}

--------------------------------------------------
-- GUI
--------------------------------------------------

local gui = Instance.new("ScreenGui")
gui.Name = "tanruto4 script"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = playerGui

local frame = Instance.new("Frame")
frame.Size = UDim2.fromOffset(245,285)
frame.Position = UDim2.new(0,60,0.5,-142)
frame.BackgroundColor3 = Color3.fromRGB(17,18,23)
frame.BorderSizePixel = 0
frame.ClipsDescendants = true
frame.Parent = gui

Instance.new("UICorner",frame).CornerRadius = UDim.new(0,12)

local fs = Instance.new("UIStroke",frame)
fs.Color = Color3.fromRGB(0,120,255)
fs.Thickness = 2

--------------------------------------------------
-- Title
--------------------------------------------------

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,38)
title.BackgroundColor3 = Color3.fromRGB(0,105,220)
title.Text = "TANRUTO4 SCRIPT"
title.TextColor3 = Color3.new(1,1,1)
title.TextSize = 15
title.Font = Enum.Font.GothamBold
title.Parent = frame

--------------------------------------------------
-- Drag Main GUI
--------------------------------------------------

local dragging = false
local dragStart
local startPos

title.InputBegan:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.Touch
	or input.UserInputType == Enum.UserInputType.MouseButton1 then

		dragging = true
		dragStart = input.Position
		startPos = frame.Position
	end
end)

UIS.InputChanged:Connect(function(input)

	if not dragging then return end

	if input.UserInputType ~= Enum.UserInputType.Touch
	and input.UserInputType ~= Enum.UserInputType.MouseMovement then
		return
	end

	local delta = input.Position - dragStart

	frame.Position = UDim2.new(
		startPos.X.Scale,
		startPos.X.Offset + delta.X,
		startPos.Y.Scale,
		startPos.Y.Offset + delta.Y
	)
end)

UIS.InputEnded:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.Touch
	or input.UserInputType == Enum.UserInputType.MouseButton1 then

		dragging = false
	end
end)

--------------------------------------------------
-- Mini Button
--------------------------------------------------

local mini = Instance.new("TextButton")
mini.Size = UDim2.fromOffset(50,50)
mini.Position = UDim2.new(0,8,0.5,-25)
mini.BackgroundColor3 = Color3.fromRGB(0,120,255)
mini.Text = "T4"
mini.TextColor3 = Color3.new(1,1,1)
mini.TextSize = 15
mini.Font = Enum.Font.GothamBold
mini.Parent = gui

Instance.new("UICorner",mini).CornerRadius = UDim.new(1,0)

local miniDragging = false
local miniMoved = false
local miniStart
local miniPos

mini.InputBegan:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.Touch
	or input.UserInputType == Enum.UserInputType.MouseButton1 then

		miniDragging = true
		miniMoved = false
		miniStart = input.Position
		miniPos = mini.Position
	end
end)

UIS.InputChanged:Connect(function(input)

	if not miniDragging then return end

	if input.UserInputType ~= Enum.UserInputType.Touch
	and input.UserInputType ~= Enum.UserInputType.MouseMovement then
		return
	end

	local delta = input.Position - miniStart

	if delta.Magnitude > 8 then
		miniMoved = true
	end

	mini.Position = UDim2.new(
		miniPos.X.Scale,
		miniPos.X.Offset + delta.X,
		miniPos.Y.Scale,
		miniPos.Y.Offset + delta.Y
	)
end)

UIS.InputEnded:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.Touch
	or input.UserInputType == Enum.UserInputType.MouseButton1 then

		if miniDragging and not miniMoved then
			frame.Visible = not frame.Visible
		end

		miniDragging = false
	end
end)

--------------------------------------------------
-- Scroll
--------------------------------------------------

local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1,-14,1,-45)
scroll.Position = UDim2.fromOffset(7,42)
scroll.BackgroundTransparency = 1
scroll.BorderSizePixel = 0
scroll.ScrollBarThickness = 5
scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
scroll.Parent = frame

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0,5)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Parent = scroll

local function button(text)

	local b = Instance.new("TextButton")
	b.Size = UDim2.new(1,-4,0,36)
	b.BackgroundColor3 = Color3.fromRGB(35,37,45)
	b.Text = text
	b.TextColor3 = Color3.new(1,1,1)
	b.TextSize = 12
	b.Font = Enum.Font.GothamMedium
	b.Parent = scroll

	Instance.new("UICorner",b).CornerRadius = UDim.new(0,8)

	return b
end

local function textbox(placeholder)

	local b = Instance.new("TextBox")
	b.Size = UDim2.new(1,-4,0,34)
	b.BackgroundColor3 = Color3.fromRGB(27,29,35)
	b.PlaceholderText = placeholder
	b.Text = ""
	b.TextColor3 = Color3.new(1,1,1)
	b.TextSize = 11
	b.ClearTextOnFocus = false
	b.Parent = scroll

	Instance.new("UICorner",b).CornerRadius = UDim.new(0,8)

	return b
end

local function section(text)

	local s = Instance.new("TextLabel")
	s.Size = UDim2.new(1,-4,0,30)
	s.BackgroundColor3 = Color3.fromRGB(0,82,175)
	s.Text = "  "..text
	s.TextColor3 = Color3.new(1,1,1)
	s.TextSize = 12
	s.Font = Enum.Font.GothamBold
	s.TextXAlignment = Enum.TextXAlignment.Left
	s.Parent = scroll

	Instance.new("UICorner",s).CornerRadius = UDim.new(0,8)

	return s
end

--------------------------------------------------
-- Player Section
--------------------------------------------------

section("👤 اللاعب")

local noclipButton = button("نوكلب: إيقاف")
local jumpButton = button("قفز لا نهائي: إيقاف")
local speedButton = button("السرعة: 16")
local jumpPowerButton = button("قوة القفز: 50")
local godButton = button("عدم الموت: إيقاف")
local flyButton = button("✈ الطيران: إيقاف")
local flySpeedButton = button("سرعة الطيران: 60")

--------------------------------------------------
-- Players Section
--------------------------------------------------

section("👥 اللاعبين")

local playerListButton = button("📋 قائمة اللاعبين")
local espButton = button("ESP: إيقاف")

local spectateBox = textbox("اسم اللاعب للمشاهدة")
local spectateButton = button("👁 مشاهدة")

--------------------------------------------------
-- Teleport Section
--------------------------------------------------

section("📍 الانتقال")

local saveTPButton = button("📍 حفظ TP Point")
local tpButton = button("🚀 الانتقال إلى TP Point")
local clearTPButton = button("🗑 حذف TP Point")

local gotoBox = textbox("اسم اللاعب للانتقال")
local gotoButton = button("📌 انتقال إلى اللاعب")

--------------------------------------------------
-- Players Window
--------------------------------------------------

local playersWindow = Instance.new("Frame")
playersWindow.Size = UDim2.fromOffset(235,300)
playersWindow.Position = UDim2.new(0.5,-117,0.5,-150)
playersWindow.BackgroundColor3 = Color3.fromRGB(18,19,25)
playersWindow.BorderSizePixel = 0
playersWindow.Visible = false
playersWindow.ZIndex = 30
playersWindow.Parent = gui

Instance.new("UICorner",playersWindow).CornerRadius = UDim.new(0,12)

local pwStroke = Instance.new("UIStroke",playersWindow)
pwStroke.Color = Color3.fromRGB(0,120,255)
pwStroke.Thickness = 2

local playersTitle = Instance.new("TextLabel")
playersTitle.Size = UDim2.new(1,0,0,40)
playersTitle.BackgroundColor3 = Color3.fromRGB(0,105,220)
playersTitle.Text = "👥 قائمة اللاعبين"
playersTitle.TextColor3 = Color3.new(1,1,1)
playersTitle.TextSize = 14
playersTitle.Font = Enum.Font.GothamBold
playersTitle.ZIndex = 31
playersTitle.Parent = playersWindow

local closePlayers = Instance.new("TextButton")
closePlayers.Size = UDim2.fromOffset(35,35)
closePlayers.Position = UDim2.new(1,-38,0,2)
closePlayers.Text = "×"
closePlayers.TextSize = 22
closePlayers.TextColor3 = Color3.new(1,1,1)
closePlayers.BackgroundTransparency = 1
closePlayers.ZIndex = 32
closePlayers.Parent = playersWindow

local playersScroll = Instance.new("ScrollingFrame")
playersScroll.Size = UDim2.new(1,-10,1,-48)
playersScroll.Position = UDim2.fromOffset(5,45)
playersScroll.BackgroundTransparency = 1
playersScroll.BorderSizePixel = 0
playersScroll.ScrollBarThickness = 5
playersScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
playersScroll.ZIndex = 31
playersScroll.Parent = playersWindow

local playersLayout = Instance.new("UIListLayout")
playersLayout.Padding = UDim.new(0,6)
playersLayout.Parent = playersScroll

--------------------------------------------------
-- Player Card
--------------------------------------------------

local function createPlayerCard(target,isFriend)

	local card = Instance.new("Frame")
	card.Size = UDim2.new(1,-5,0,80)
	card.BackgroundColor3 = Color3.fromRGB(30,32,40)
	card.ZIndex = 32
	card.Parent = playersScroll

	Instance.new("UICorner",card).CornerRadius = UDim.new(0,9)

	local nameLabel = Instance.new("TextLabel")
	nameLabel.Size = UDim2.new(1,-70,0,25)
	nameLabel.Position = UDim2.fromOffset(5,3)
	nameLabel.BackgroundTransparency = 1
	nameLabel.Text = target.DisplayName.."  @"..target.Name
	nameLabel.TextColor3 = Color3.new(1,1,1)
	nameLabel.TextSize = 10
	nameLabel.Font = Enum.Font.GothamBold
	nameLabel.TextXAlignment = Enum.TextXAlignment.Left
	nameLabel.ZIndex = 33
	nameLabel.Parent = card

	if isFriend then

		local friendLabel = Instance.new("TextLabel")
		friendLabel.Size = UDim2.fromOffset(55,22)
		friendLabel.Position = UDim2.new(1,-60,0,4)
		friendLabel.BackgroundColor3 = Color3.fromRGB(0,170,90)
		friendLabel.TextColor3 = Color3.new(1,1,1)
		friendLabel.Text = "صديق"
		friendLabel.TextSize = 10
		friendLabel.Font = Enum.Font.GothamBold
		friendLabel.ZIndex = 34
		friendLabel.Parent = card

		Instance.new("UICorner",friendLabel).CornerRadius = UDim.new(0,6)
	end

	local tp = Instance.new("TextButton")
	tp.Size = UDim2.new(0.48,-5,0,35)
	tp.Position = UDim2.fromOffset(5,35)
	tp.Text = "📍 TP"
	tp.TextColor3 = Color3.new(1,1,1)
	tp.TextSize = 11
	tp.BackgroundColor3 = Color3.fromRGB(0,105,210)
	tp.ZIndex = 33
	tp.Parent = card

	local spec = Instance.new("TextButton")
	spec.Size = UDim2.new(0.48,-5,0,35)
	spec.Position = UDim2.new(0.52,0,0,35)
	spec.Text = "👁 مشاهدة"
	spec.TextColor3 = Color3.new(1,1,1)
	spec.TextSize = 11
	spec.BackgroundColor3 = Color3.fromRGB(65,67,75)
	spec.ZIndex = 33
	spec.Parent = card

	Instance.new("UICorner",tp).CornerRadius = UDim.new(0,7)
	Instance.new("UICorner",spec).CornerRadius = UDim.new(0,7)

	--------------------------------------------------
	-- TP ثم إغلاق قائمة اللاعبين
	--------------------------------------------------

	tp.MouseButton1Click:Connect(function()

		local char = player.Character
		local targetChar = target.Character

		if char and targetChar then

			local root = char:FindFirstChild("HumanoidRootPart")
			local targetRoot = targetChar:FindFirstChild("HumanoidRootPart")

			if root and targetRoot then
				root.CFrame = targetRoot.CFrame + Vector3.new(0,3,0)
			end
		end

		playersWindow.Visible = false
	end)

	--------------------------------------------------
	-- مشاهدة ثم إغلاق قائمة اللاعبين
	--------------------------------------------------

	spec.MouseButton1Click:Connect(function()

		local targetChar = target.Character

		if targetChar then

			local hum = targetChar:FindFirstChildOfClass("Humanoid")

			if hum then

				targetPlayer = target
				spectating = true

				workspace.CurrentCamera.CameraSubject = hum

				spectateBox.Text = target.Name
				spectateButton.Text = "إيقاف المشاهدة"
			end
		end

		playersWindow.Visible = false
	end)
end

--------------------------------------------------
-- Refresh Players
--------------------------------------------------

local function refreshPlayers()

	for _,v in ipairs(playersScroll:GetChildren()) do

		if v:IsA("Frame") then
			v:Destroy()
		end
	end

	local friends = {}
	local others = {}

	for _,p in ipairs(Players:GetPlayers()) do

		if p ~= player then

			local isFriend = false

			pcall(function()
				isFriend = player:IsFriendsWith(p.UserId)
			end)

			if isFriend then
				table.insert(friends,p)
			else
				table.insert(others,p)
			end
		end
	end

	table.sort(friends,function(a,b)
		return a.DisplayName:lower() < b.DisplayName:lower()
	end)

	table.sort(others,function(a,b)
		return a.DisplayName:lower() < b.DisplayName:lower()
	end)

	-- الأصدقاء أولاً
	for _,p in ipairs(friends) do
		createPlayerCard(p,true)
	end

	-- باقي اللاعبين
	for _,p in ipairs(others) do
		createPlayerCard(p,false)
	end
end

playerListButton.MouseButton1Click:Connect(function()

	playersWindow.Visible = not playersWindow.Visible

	if playersWindow.Visible then
		refreshPlayers()
	end
end)

closePlayers.MouseButton1Click:Connect(function()
	playersWindow.Visible = false
end)

Players.PlayerAdded:Connect(function()

	if playersWindow.Visible then
		refreshPlayers()
	end
end)

Players.PlayerRemoving:Connect(function()

	if playersWindow.Visible then
		refreshPlayers()
	end
end)

--------------------------------------------------
-- Noclip
--------------------------------------------------

RunService.Stepped:Connect(function()

	if not noclip then return end

	local char = player.Character

	if char then

		for _,v in ipairs(char:GetDescendants()) do

			if v:IsA("BasePart") then
				v.CanCollide = false
			end
		end
	end
end)

noclipButton.MouseButton1Click:Connect(function()

	noclip = not noclip

	noclipButton.Text =
		noclip and "نوكلب: تشغيل"
		or "نوكلب: إيقاف"
end)

--------------------------------------------------
-- Infinite Jump
--------------------------------------------------

UIS.JumpRequest:Connect(function()

	if not infiniteJump then return end

	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")

	if hum then
		hum:ChangeState(Enum.HumanoidStateType.Jumping)
	end
end)

jumpButton.MouseButton1Click:Connect(function()

	infiniteJump = not infiniteJump

	jumpButton.Text =
		infiniteJump and "قفز لا نهائي: تشغيل"
		or "قفز لا نهائي: إيقاف"
end)

--------------------------------------------------
-- Speed
--------------------------------------------------

speedButton.MouseButton1Click:Connect(function()

	speedIndex += 1

	if speedIndex > #speedList then
		speedIndex = 1
	end

	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")

	if hum then
		hum.WalkSpeed = speedList[speedIndex]
	end

	speedButton.Text = "السرعة: "..speedList[speedIndex]
end)

--------------------------------------------------
-- Jump Power
--------------------------------------------------

jumpPowerButton.MouseButton1Click:Connect(function()

	jumpIndex += 1

	if jumpIndex > #jumpList then
		jumpIndex = 1
	end

	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")

	if hum then
		hum.UseJumpPower = true
		hum.JumpPower = jumpList[jumpIndex]
	end

	jumpPowerButton.Text = "قوة القفز: "..jumpList[jumpIndex]
end)

--------------------------------------------------
-- God Mode
--------------------------------------------------

local function stopGod()

	godMode = false

	if godConnection then
		godConnection:Disconnect()
		godConnection = nil
	end

	godButton.Text = "عدم الموت: إيقاف"
end

local function startGod()

	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")

	if not hum then return end

	godMode = true
	godButton.Text = "عدم الموت: تشغيل"

	if godConnection then
		godConnection:Disconnect()
	end

	godConnection = RunService.Heartbeat:Connect(function()

		if not godMode then return end

		local c = player.Character
		local h = c and c:FindFirstChildOfClass("Humanoid")

		if h and h.Health > 0 then
			h.Health = h.MaxHealth
		end
	end)
end

godButton.MouseButton1Click:Connect(function()

	if godMode then
		stopGod()
	else
		startGod()
	end
end)

--------------------------------------------------
-- ESP
--------------------------------------------------

local function refreshESP()

	for _,h in pairs(espObjects) do

		if h then
			h:Destroy()
		end
	end

	table.clear(espObjects)

	if not espEnabled then return end

	for _,p in ipairs(Players:GetPlayers()) do

		if p ~= player and p.Character then

			local h = Instance.new("Highlight")
			h.Name = "T4_ESP"
			h.Adornee = p.Character
			h.FillTransparency = 0.5
			h.Parent = p.Character

			espObjects[p] = h
		end
	end
end

espButton.MouseButton1Click:Connect(function()

	espEnabled = not espEnabled

	espButton.Text =
		espEnabled and "ESP: تشغيل"
		or "ESP: إيقاف"

	refreshESP()
end)

--------------------------------------------------
-- Spectate
--------------------------------------------------

local function stopSpectating()

	spectating = false
	targetPlayer = nil

	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")

	if hum then
		workspace.CurrentCamera.CameraSubject = hum
	end

	spectateButton.Text = "👁 مشاهدة"
end

spectateButton.MouseButton1Click:Connect(function()

	if spectating then

		stopSpectating()
		return
	end

	local target = findPlayer(spectateBox.Text)

	if target and target.Character then

		local hum = target.Character:FindFirstChildOfClass("Humanoid")

		if hum then

			targetPlayer = target
			spectating = true

			workspace.CurrentCamera.CameraSubject = hum

			spectateButton.Text = "إيقاف المشاهدة"
		end
	end
end)

RunService.RenderStepped:Connect(function()

	if not spectating or not targetPlayer then
		return
	end

	local char = targetPlayer.Character

	if not char then

		stopSpectating()
		return
	end

	local hum = char:FindFirstChildOfClass("Humanoid")

	if hum then
		workspace.CurrentCamera.CameraSubject = hum
	end
end)

--------------------------------------------------
-- TP Point
--------------------------------------------------

saveTPButton.MouseButton1Click:Connect(function()

	local char = player.Character
	local root = char and char:FindFirstChild("HumanoidRootPart")

	if root then

		savedTPPoint = root.CFrame
		saveTPButton.Text = "📍 تم الحفظ ✓"
	end
end)

tpButton.MouseButton1Click:Connect(function()

	if not savedTPPoint then return end

	local char = player.Character
	local root = char and char:FindFirstChild("HumanoidRootPart")

	if root then
		root.CFrame = savedTPPoint + Vector3.new(0,3,0)
	end
end)

clearTPButton.MouseButton1Click:Connect(function()

	savedTPPoint = nil
	saveTPButton.Text = "📍 حفظ TP Point"
end)

--------------------------------------------------
-- Goto
--------------------------------------------------

gotoButton.MouseButton1Click:Connect(function()

	local target = findPlayer(gotoBox.Text)

	if not target then return end

	local char = player.Character
	local targetChar = target.Character

	if not char or not targetChar then return end

	local root = char:FindFirstChild("HumanoidRootPart")
	local targetRoot = targetChar:FindFirstChild("HumanoidRootPart")

	if root and targetRoot then
		root.CFrame = targetRoot.CFrame + Vector3.new(0,3,0)
	end
end)

--------------------------------------------------
-- Fly Controls
--------------------------------------------------

local flightControls = Instance.new("Frame")
flightControls.Size = UDim2.fromOffset(120,105)
flightControls.Position = UDim2.new(1,-130,1,-120)
flightControls.BackgroundTransparency = 1
flightControls.Visible = false
flightControls.Parent = gui

local upButton = Instance.new("TextButton")
upButton.Size = UDim2.fromOffset(55,42)
upButton.Position = UDim2.fromOffset(32,0)
upButton.Text = "▲"
upButton.TextSize = 20
upButton.TextColor3 = Color3.new(1,1,1)
upButton.BackgroundColor3 = Color3.fromRGB(0,120,220)
upButton.Parent = flightControls

local downButton = Instance.new("TextButton")
downButton.Size = UDim2.fromOffset(55,42)
downButton.Position = UDim2.fromOffset(32,50)
downButton.Text = "▼"
downButton.TextSize = 20
downButton.TextColor3 = Color3.new(1,1,1)
downButton.BackgroundColor3 = Color3.fromRGB(0,120,220)
downButton.Parent = flightControls

Instance.new("UICorner",upButton).CornerRadius = UDim.new(0,9)
Instance.new("UICorner",downButton).CornerRadius = UDim.new(0,9)

local function hold(btn,callback)

	btn.InputBegan:Connect(function(input)

		if input.UserInputType == Enum.UserInputType.Touch
		or input.UserInputType == Enum.UserInputType.MouseButton1 then

			callback(true)
		end
	end)

	btn.InputEnded:Connect(function(input)

		if input.UserInputType == Enum.UserInputType.Touch
		or input.UserInputType == Enum.UserInputType.MouseButton1 then

			callback(false)
		end
	end)
end

hold(upButton,function(v)
	upHeld = v
end)

hold(downButton,function(v)
	downHeld = v
end)

--------------------------------------------------
-- Fly
--------------------------------------------------

local function stopFly()

	flying = false
	upHeld = false
	downHeld = false

	flightControls.Visible = false

	if flyConnection then
		flyConnection:Disconnect()
		flyConnection = nil
	end

	if flyVelocity then
		flyVelocity:Destroy()
		flyVelocity = nil
	end

	if flyAttachment then
		flyAttachment:Destroy()
		flyAttachment = nil
	end

	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")
	local root = char and char:FindFirstChild("HumanoidRootPart")

	if hum then
		hum.AutoRotate = true
	end

	if root then
		root.AssemblyLinearVelocity = Vector3.zero
	end

	flyButton.Text = "✈ الطيران: إيقاف"
end

local function startFly()

	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")
	local root = char and char:FindFirstChild("HumanoidRootPart")

	if not char or not hum or not root then
		return
	end

	flying = true
	flightControls.Visible = true
	hum.AutoRotate = false

	flyAttachment = Instance.new("Attachment")
	flyAttachment.Parent = root

	flyVelocity = Instance.new("LinearVelocity")
	flyVelocity.Attachment0 = flyAttachment
	flyVelocity.RelativeTo = Enum.ActuatorRelativeTo.World
	flyVelocity.MaxForce = math.huge
	flyVelocity.VectorVelocity = Vector3.zero
	flyVelocity.Parent = root

	flyConnection = RunService.RenderStepped:Connect(function()

		if not flying then return end

		local currentChar = player.Character
		local currentHum = currentChar and currentChar:FindFirstChildOfClass("Humanoid")

		if not currentChar or not currentHum then

			stopFly()
			return
		end

		local camera = workspace.CurrentCamera
		local move = currentHum.MoveDirection

		if UIS:IsKeyDown(Enum.KeyCode.W) then
			move += camera.CFrame.LookVector
		end

		if UIS:IsKeyDown(Enum.KeyCode.S) then
			move -= camera.CFrame.LookVector
		end

		if UIS:IsKeyDown(Enum.KeyCode.A) then
			move -= camera.CFrame.RightVector
		end

		if UIS:IsKeyDown(Enum.KeyCode.D) then
			move += camera.CFrame.RightVector
		end

		if upHeld or UIS:IsKeyDown(Enum.KeyCode.Space) then
			move += Vector3.new(0,1,0)
		end

		if downHeld or UIS:IsKeyDown(Enum.KeyCode.LeftControl) then
			move -= Vector3.new(0,1,0)
		end

		if move.Magnitude > 0 then
			flyVelocity.VectorVelocity = move.Unit * flySpeed
		else
			flyVelocity.VectorVelocity = Vector3.zero
		end
	end)

	flyButton.Text = "✈ الطيران: تشغيل"
end

flyButton.MouseButton1Click:Connect(function()

	if flying then
		stopFly()
	else
		startFly()
	end
end)

flySpeedButton.MouseButton1Click:Connect(function()

	flySpeedIndex += 1

	if flySpeedIndex > #flySpeeds then
		flySpeedIndex = 1
	end

	flySpeed = flySpeeds[flySpeedIndex]

	flySpeedButton.Text = "سرعة الطيران: "..flySpeed
end)

--------------------------------------------------
-- Respawn
--------------------------------------------------

player.CharacterAdded:Connect(function()

	if flying then
		stopFly()
	end

	task.wait(0.5)

	if espEnabled then
		refreshESP()
	end

	if godMode then
		startGod()
	end
end)-- tanruto4 script
-- نسخة كاملة: الأصدقاء أولاً + علامة "صديق" + قائمة لاعبين + TP/مشاهدة تغلق القائمة

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local old = playerGui:FindFirstChild("tanruto4 script")
if old then old:Destroy() end

local noclip = false
local infiniteJump = false
local flying = false
local espEnabled = false
local spectating = false
local godMode = false

local targetPlayer
local savedTPPoint

local speedList = {16,50,100,150}
local speedIndex = 1

local jumpList = {50,75,100,150,200}
local jumpIndex = 1

local flySpeeds = {40,60,100,150}
local flySpeedIndex = 2
local flySpeed = 60

local flyVelocity
local flyAttachment
local flyConnection
local godConnection

local upHeld = false
local downHeld = false

local espObjects = {}

--------------------------------------------------
-- GUI
--------------------------------------------------

local gui = Instance.new("ScreenGui")
gui.Name = "tanruto4 script"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = playerGui

local frame = Instance.new("Frame")
frame.Size = UDim2.fromOffset(245,285)
frame.Position = UDim2.new(0,60,0.5,-142)
frame.BackgroundColor3 = Color3.fromRGB(17,18,23)
frame.BorderSizePixel = 0
frame.ClipsDescendants = true
frame.Parent = gui

Instance.new("UICorner",frame).CornerRadius = UDim.new(0,12)

local fs = Instance.new("UIStroke",frame)
fs.Color = Color3.fromRGB(0,120,255)
fs.Thickness = 2

--------------------------------------------------
-- Title
--------------------------------------------------

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,38)
title.BackgroundColor3 = Color3.fromRGB(0,105,220)
title.Text = "TANRUTO4 SCRIPT"
title.TextColor3 = Color3.new(1,1,1)
title.TextSize = 15
title.Font = Enum.Font.GothamBold
title.Parent = frame

--------------------------------------------------
-- Drag Main GUI
--------------------------------------------------

local dragging = false
local dragStart
local startPos

title.InputBegan:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.Touch
	or input.UserInputType == Enum.UserInputType.MouseButton1 then

		dragging = true
		dragStart = input.Position
		startPos = frame.Position
	end
end)

UIS.InputChanged:Connect(function(input)

	if not dragging then return end

	if input.UserInputType ~= Enum.UserInputType.Touch
	and input.UserInputType ~= Enum.UserInputType.MouseMovement then
		return
	end

	local delta = input.Position - dragStart

	frame.Position = UDim2.new(
		startPos.X.Scale,
		startPos.X.Offset + delta.X,
		startPos.Y.Scale,
		startPos.Y.Offset + delta.Y
	)
end)

UIS.InputEnded:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.Touch
	or input.UserInputType == Enum.UserInputType.MouseButton1 then

		dragging = false
	end
end)

--------------------------------------------------
-- Mini Button
--------------------------------------------------

local mini = Instance.new("TextButton")
mini.Size = UDim2.fromOffset(50,50)
mini.Position = UDim2.new(0,8,0.5,-25)
mini.BackgroundColor3 = Color3.fromRGB(0,120,255)
mini.Text = "T4"
mini.TextColor3 = Color3.new(1,1,1)
mini.TextSize = 15
mini.Font = Enum.Font.GothamBold
mini.Parent = gui

Instance.new("UICorner",mini).CornerRadius = UDim.new(1,0)

local miniDragging = false
local miniMoved = false
local miniStart
local miniPos

mini.InputBegan:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.Touch
	or input.UserInputType == Enum.UserInputType.MouseButton1 then

		miniDragging = true
		miniMoved = false
		miniStart = input.Position
		miniPos = mini.Position
	end
end)

UIS.InputChanged:Connect(function(input)

	if not miniDragging then return end

	if input.UserInputType ~= Enum.UserInputType.Touch
	and input.UserInputType ~= Enum.UserInputType.MouseMovement then
		return
	end

	local delta = input.Position - miniStart

	if delta.Magnitude > 8 then
		miniMoved = true
	end

	mini.Position = UDim2.new(
		miniPos.X.Scale,
		miniPos.X.Offset + delta.X,
		miniPos.Y.Scale,
		miniPos.Y.Offset + delta.Y
	)
end)

UIS.InputEnded:Connect(function(input)

	if input.UserInputType == Enum.UserInputType.Touch
	or input.UserInputType == Enum.UserInputType.MouseButton1 then

		if miniDragging and not miniMoved then
			frame.Visible = not frame.Visible
		end

		miniDragging = false
	end
end)

--------------------------------------------------
-- Scroll
--------------------------------------------------

local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1,-14,1,-45)
scroll.Position = UDim2.fromOffset(7,42)
scroll.BackgroundTransparency = 1
scroll.BorderSizePixel = 0
scroll.ScrollBarThickness = 5
scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
scroll.Parent = frame

local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0,5)
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Parent = scroll

local function button(text)

	local b = Instance.new("TextButton")
	b.Size = UDim2.new(1,-4,0,36)
	b.BackgroundColor3 = Color3.fromRGB(35,37,45)
	b.Text = text
	b.TextColor3 = Color3.new(1,1,1)
	b.TextSize = 12
	b.Font = Enum.Font.GothamMedium
	b.Parent = scroll

	Instance.new("UICorner",b).CornerRadius = UDim.new(0,8)

	return b
end

local function textbox(placeholder)

	local b = Instance.new("TextBox")
	b.Size = UDim2.new(1,-4,0,34)
	b.BackgroundColor3 = Color3.fromRGB(27,29,35)
	b.PlaceholderText = placeholder
	b.Text = ""
	b.TextColor3 = Color3.new(1,1,1)
	b.TextSize = 11
	b.ClearTextOnFocus = false
	b.Parent = scroll

	Instance.new("UICorner",b).CornerRadius = UDim.new(0,8)

	return b
end

local function section(text)

	local s = Instance.new("TextLabel")
	s.Size = UDim2.new(1,-4,0,30)
	s.BackgroundColor3 = Color3.fromRGB(0,82,175)
	s.Text = "  "..text
	s.TextColor3 = Color3.new(1,1,1)
	s.TextSize = 12
	s.Font = Enum.Font.GothamBold
	s.TextXAlignment = Enum.TextXAlignment.Left
	s.Parent = scroll

	Instance.new("UICorner",s).CornerRadius = UDim.new(0,8)

	return s
end

--------------------------------------------------
-- Player Section
--------------------------------------------------

section("👤 اللاعب")

local noclipButton = button("نوكلب: إيقاف")
local jumpButton = button("قفز لا نهائي: إيقاف")
local speedButton = button("السرعة: 16")
local jumpPowerButton = button("قوة القفز: 50")
local godButton = button("عدم الموت: إيقاف")
local flyButton = button("✈ الطيران: إيقاف")
local flySpeedButton = button("سرعة الطيران: 60")

--------------------------------------------------
-- Players Section
--------------------------------------------------

section("👥 اللاعبين")

local playerListButton = button("📋 قائمة اللاعبين")
local espButton = button("ESP: إيقاف")

local spectateBox = textbox("اسم اللاعب للمشاهدة")
local spectateButton = button("👁 مشاهدة")

--------------------------------------------------
-- Teleport Section
--------------------------------------------------

section("📍 الانتقال")

local saveTPButton = button("📍 حفظ TP Point")
local tpButton = button("🚀 الانتقال إلى TP Point")
local clearTPButton = button("🗑 حذف TP Point")

local gotoBox = textbox("اسم اللاعب للانتقال")
local gotoButton = button("📌 انتقال إلى اللاعب")

--------------------------------------------------
-- Players Window
--------------------------------------------------

local playersWindow = Instance.new("Frame")
playersWindow.Size = UDim2.fromOffset(235,300)
playersWindow.Position = UDim2.new(0.5,-117,0.5,-150)
playersWindow.BackgroundColor3 = Color3.fromRGB(18,19,25)
playersWindow.BorderSizePixel = 0
playersWindow.Visible = false
playersWindow.ZIndex = 30
playersWindow.Parent = gui

Instance.new("UICorner",playersWindow).CornerRadius = UDim.new(0,12)

local pwStroke = Instance.new("UIStroke",playersWindow)
pwStroke.Color = Color3.fromRGB(0,120,255)
pwStroke.Thickness = 2

local playersTitle = Instance.new("TextLabel")
playersTitle.Size = UDim2.new(1,0,0,40)
playersTitle.BackgroundColor3 = Color3.fromRGB(0,105,220)
playersTitle.Text = "👥 قائمة اللاعبين"
playersTitle.TextColor3 = Color3.new(1,1,1)
playersTitle.TextSize = 14
playersTitle.Font = Enum.Font.GothamBold
playersTitle.ZIndex = 31
playersTitle.Parent = playersWindow

local closePlayers = Instance.new("TextButton")
closePlayers.Size = UDim2.fromOffset(35,35)
closePlayers.Position = UDim2.new(1,-38,0,2)
closePlayers.Text = "×"
closePlayers.TextSize = 22
closePlayers.TextColor3 = Color3.new(1,1,1)
closePlayers.BackgroundTransparency = 1
closePlayers.ZIndex = 32
closePlayers.Parent = playersWindow

local playersScroll = Instance.new("ScrollingFrame")
playersScroll.Size = UDim2.new(1,-10,1,-48)
playersScroll.Position = UDim2.fromOffset(5,45)
playersScroll.BackgroundTransparency = 1
playersScroll.BorderSizePixel = 0
playersScroll.ScrollBarThickness = 5
playersScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
playersScroll.ZIndex = 31
playersScroll.Parent = playersWindow

local playersLayout = Instance.new("UIListLayout")
playersLayout.Padding = UDim.new(0,6)
playersLayout.Parent = playersScroll

--------------------------------------------------
-- Player Card
--------------------------------------------------

local function createPlayerCard(target,isFriend)

	local card = Instance.new("Frame")
	card.Size = UDim2.new(1,-5,0,80)
	card.BackgroundColor3 = Color3.fromRGB(30,32,40)
	card.ZIndex = 32
	card.Parent = playersScroll

	Instance.new("UICorner",card).CornerRadius = UDim.new(0,9)

	local nameLabel = Instance.new("TextLabel")
	nameLabel.Size = UDim2.new(1,-70,0,25)
	nameLabel.Position = UDim2.fromOffset(5,3)
	nameLabel.BackgroundTransparency = 1
	nameLabel.Text = target.DisplayName.."  @"..target.Name
	nameLabel.TextColor3 = Color3.new(1,1,1)
	nameLabel.TextSize = 10
	nameLabel.Font = Enum.Font.GothamBold
	nameLabel.TextXAlignment = Enum.TextXAlignment.Left
	nameLabel.ZIndex = 33
	nameLabel.Parent = card

	if isFriend then

		local friendLabel = Instance.new("TextLabel")
		friendLabel.Size = UDim2.fromOffset(55,22)
		friendLabel.Position = UDim2.new(1,-60,0,4)
		friendLabel.BackgroundColor3 = Color3.fromRGB(0,170,90)
		friendLabel.TextColor3 = Color3.new(1,1,1)
		friendLabel.Text = "صديق"
		friendLabel.TextSize = 10
		friendLabel.Font = Enum.Font.GothamBold
		friendLabel.ZIndex = 34
		friendLabel.Parent = card

		Instance.new("UICorner",friendLabel).CornerRadius = UDim.new(0,6)
	end

	local tp = Instance.new("TextButton")
	tp.Size = UDim2.new(0.48,-5,0,35)
	tp.Position = UDim2.fromOffset(5,35)
	tp.Text = "📍 TP"
	tp.TextColor3 = Color3.new(1,1,1)
	tp.TextSize = 11
	tp.BackgroundColor3 = Color3.fromRGB(0,105,210)
	tp.ZIndex = 33
	tp.Parent = card

	local spec = Instance.new("TextButton")
	spec.Size = UDim2.new(0.48,-5,0,35)
	spec.Position = UDim2.new(0.52,0,0,35)
	spec.Text = "👁 مشاهدة"
	spec.TextColor3 = Color3.new(1,1,1)
	spec.TextSize = 11
	spec.BackgroundColor3 = Color3.fromRGB(65,67,75)
	spec.ZIndex = 33
	spec.Parent = card

	Instance.new("UICorner",tp).CornerRadius = UDim.new(0,7)
	Instance.new("UICorner",spec).CornerRadius = UDim.new(0,7)

	--------------------------------------------------
	-- TP ثم إغلاق قائمة اللاعبين
	--------------------------------------------------

	tp.MouseButton1Click:Connect(function()

		local char = player.Character
		local targetChar = target.Character

		if char and targetChar then

			local root = char:FindFirstChild("HumanoidRootPart")
			local targetRoot = targetChar:FindFirstChild("HumanoidRootPart")

			if root and targetRoot then
				root.CFrame = targetRoot.CFrame + Vector3.new(0,3,0)
			end
		end

		playersWindow.Visible = false
	end)

	--------------------------------------------------
	-- مشاهدة ثم إغلاق قائمة اللاعبين
	--------------------------------------------------

	spec.MouseButton1Click:Connect(function()

		local targetChar = target.Character

		if targetChar then

			local hum = targetChar:FindFirstChildOfClass("Humanoid")

			if hum then

				targetPlayer = target
				spectating = true

				workspace.CurrentCamera.CameraSubject = hum

				spectateBox.Text = target.Name
				spectateButton.Text = "إيقاف المشاهدة"
			end
		end

		playersWindow.Visible = false
	end)
end

--------------------------------------------------
-- Refresh Players
--------------------------------------------------

local function refreshPlayers()

	for _,v in ipairs(playersScroll:GetChildren()) do

		if v:IsA("Frame") then
			v:Destroy()
		end
	end

	local friends = {}
	local others = {}

	for _,p in ipairs(Players:GetPlayers()) do

		if p ~= player then

			local isFriend = false

			pcall(function()
				isFriend = player:IsFriendsWith(p.UserId)
			end)

			if isFriend then
				table.insert(friends,p)
			else
				table.insert(others,p)
			end
		end
	end

	table.sort(friends,function(a,b)
		return a.DisplayName:lower() < b.DisplayName:lower()
	end)

	table.sort(others,function(a,b)
		return a.DisplayName:lower() < b.DisplayName:lower()
	end)

	-- الأصدقاء أولاً
	for _,p in ipairs(friends) do
		createPlayerCard(p,true)
	end

	-- باقي اللاعبين
	for _,p in ipairs(others) do
		createPlayerCard(p,false)
	end
end

playerListButton.MouseButton1Click:Connect(function()

	playersWindow.Visible = not playersWindow.Visible

	if playersWindow.Visible then
		refreshPlayers()
	end
end)

closePlayers.MouseButton1Click:Connect(function()
	playersWindow.Visible = false
end)

Players.PlayerAdded:Connect(function()

	if playersWindow.Visible then
		refreshPlayers()
	end
end)

Players.PlayerRemoving:Connect(function()

	if playersWindow.Visible then
		refreshPlayers()
	end
end)

--------------------------------------------------
-- Noclip
--------------------------------------------------

RunService.Stepped:Connect(function()

	if not noclip then return end

	local char = player.Character

	if char then

		for _,v in ipairs(char:GetDescendants()) do

			if v:IsA("BasePart") then
				v.CanCollide = false
			end
		end
	end
end)

noclipButton.MouseButton1Click:Connect(function()

	noclip = not noclip

	noclipButton.Text =
		noclip and "نوكلب: تشغيل"
		or "نوكلب: إيقاف"
end)

--------------------------------------------------
-- Infinite Jump
--------------------------------------------------

UIS.JumpRequest:Connect(function()

	if not infiniteJump then return end

	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")

	if hum then
		hum:ChangeState(Enum.HumanoidStateType.Jumping)
	end
end)

jumpButton.MouseButton1Click:Connect(function()

	infiniteJump = not infiniteJump

	jumpButton.Text =
		infiniteJump and "قفز لا نهائي: تشغيل"
		or "قفز لا نهائي: إيقاف"
end)

--------------------------------------------------
-- Speed
--------------------------------------------------

speedButton.MouseButton1Click:Connect(function()

	speedIndex += 1

	if speedIndex > #speedList then
		speedIndex = 1
	end

	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")

	if hum then
		hum.WalkSpeed = speedList[speedIndex]
	end

	speedButton.Text = "السرعة: "..speedList[speedIndex]
end)

--------------------------------------------------
-- Jump Power
--------------------------------------------------

jumpPowerButton.MouseButton1Click:Connect(function()

	jumpIndex += 1

	if jumpIndex > #jumpList then
		jumpIndex = 1
	end

	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")

	if hum then
		hum.UseJumpPower = true
		hum.JumpPower = jumpList[jumpIndex]
	end

	jumpPowerButton.Text = "قوة القفز: "..jumpList[jumpIndex]
end)

--------------------------------------------------
-- God Mode
--------------------------------------------------

local function stopGod()

	godMode = false

	if godConnection then
		godConnection:Disconnect()
		godConnection = nil
	end

	godButton.Text = "عدم الموت: إيقاف"
end

local function startGod()

	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")

	if not hum then return end

	godMode = true
	godButton.Text = "عدم الموت: تشغيل"

	if godConnection then
		godConnection:Disconnect()
	end

	godConnection = RunService.Heartbeat:Connect(function()

		if not godMode then return end

		local c = player.Character
		local h = c and c:FindFirstChildOfClass("Humanoid")

		if h and h.Health > 0 then
			h.Health = h.MaxHealth
		end
	end)
end

godButton.MouseButton1Click:Connect(function()

	if godMode then
		stopGod()
	else
		startGod()
	end
end)

--------------------------------------------------
-- ESP
--------------------------------------------------

local function refreshESP()

	for _,h in pairs(espObjects) do

		if h then
			h:Destroy()
		end
	end

	table.clear(espObjects)

	if not espEnabled then return end

	for _,p in ipairs(Players:GetPlayers()) do

		if p ~= player and p.Character then

			local h = Instance.new("Highlight")
			h.Name = "T4_ESP"
			h.Adornee = p.Character
			h.FillTransparency = 0.5
			h.Parent = p.Character

			espObjects[p] = h
		end
	end
end

espButton.MouseButton1Click:Connect(function()

	espEnabled = not espEnabled

	espButton.Text =
		espEnabled and "ESP: تشغيل"
		or "ESP: إيقاف"

	refreshESP()
end)

--------------------------------------------------
-- Spectate
--------------------------------------------------

local function stopSpectating()

	spectating = false
	targetPlayer = nil

	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")

	if hum then
		workspace.CurrentCamera.CameraSubject = hum
	end

	spectateButton.Text = "👁 مشاهدة"
end

spectateButton.MouseButton1Click:Connect(function()

	if spectating then

		stopSpectating()
		return
	end

	local target = findPlayer(spectateBox.Text)

	if target and target.Character then

		local hum = target.Character:FindFirstChildOfClass("Humanoid")

		if hum then

			targetPlayer = target
			spectating = true

			workspace.CurrentCamera.CameraSubject = hum

			spectateButton.Text = "إيقاف المشاهدة"
		end
	end
end)

RunService.RenderStepped:Connect(function()

	if not spectating or not targetPlayer then
		return
	end

	local char = targetPlayer.Character

	if not char then

		stopSpectating()
		return
	end

	local hum = char:FindFirstChildOfClass("Humanoid")

	if hum then
		workspace.CurrentCamera.CameraSubject = hum
	end
end)

--------------------------------------------------
-- TP Point
--------------------------------------------------

saveTPButton.MouseButton1Click:Connect(function()

	local char = player.Character
	local root = char and char:FindFirstChild("HumanoidRootPart")

	if root then

		savedTPPoint = root.CFrame
		saveTPButton.Text = "📍 تم الحفظ ✓"
	end
end)

tpButton.MouseButton1Click:Connect(function()

	if not savedTPPoint then return end

	local char = player.Character
	local root = char and char:FindFirstChild("HumanoidRootPart")

	if root then
		root.CFrame = savedTPPoint + Vector3.new(0,3,0)
	end
end)

clearTPButton.MouseButton1Click:Connect(function()

	savedTPPoint = nil
	saveTPButton.Text = "📍 حفظ TP Point"
end)

--------------------------------------------------
-- Goto
--------------------------------------------------

gotoButton.MouseButton1Click:Connect(function()

	local target = findPlayer(gotoBox.Text)

	if not target then return end

	local char = player.Character
	local targetChar = target.Character

	if not char or not targetChar then return end

	local root = char:FindFirstChild("HumanoidRootPart")
	local targetRoot = targetChar:FindFirstChild("HumanoidRootPart")

	if root and targetRoot then
		root.CFrame = targetRoot.CFrame + Vector3.new(0,3,0)
	end
end)

--------------------------------------------------
-- Fly Controls
--------------------------------------------------

local flightControls = Instance.new("Frame")
flightControls.Size = UDim2.fromOffset(120,105)
flightControls.Position = UDim2.new(1,-130,1,-120)
flightControls.BackgroundTransparency = 1
flightControls.Visible = false
flightControls.Parent = gui

local upButton = Instance.new("TextButton")
upButton.Size = UDim2.fromOffset(55,42)
upButton.Position = UDim2.fromOffset(32,0)
upButton.Text = "▲"
upButton.TextSize = 20
upButton.TextColor3 = Color3.new(1,1,1)
upButton.BackgroundColor3 = Color3.fromRGB(0,120,220)
upButton.Parent = flightControls

local downButton = Instance.new("TextButton")
downButton.Size = UDim2.fromOffset(55,42)
downButton.Position = UDim2.fromOffset(32,50)
downButton.Text = "▼"
downButton.TextSize = 20
downButton.TextColor3 = Color3.new(1,1,1)
downButton.BackgroundColor3 = Color3.fromRGB(0,120,220)
downButton.Parent = flightControls

Instance.new("UICorner",upButton).CornerRadius = UDim.new(0,9)
Instance.new("UICorner",downButton).CornerRadius = UDim.new(0,9)

local function hold(btn,callback)

	btn.InputBegan:Connect(function(input)

		if input.UserInputType == Enum.UserInputType.Touch
		or input.UserInputType == Enum.UserInputType.MouseButton1 then

			callback(true)
		end
	end)

	btn.InputEnded:Connect(function(input)

		if input.UserInputType == Enum.UserInputType.Touch
		or input.UserInputType == Enum.UserInputType.MouseButton1 then

			callback(false)
		end
	end)
end

hold(upButton,function(v)
	upHeld = v
end)

hold(downButton,function(v)
	downHeld = v
end)

--------------------------------------------------
-- Fly
--------------------------------------------------

local function stopFly()

	flying = false
	upHeld = false
	downHeld = false

	flightControls.Visible = false

	if flyConnection then
		flyConnection:Disconnect()
		flyConnection = nil
	end

	if flyVelocity then
		flyVelocity:Destroy()
		flyVelocity = nil
	end

	if flyAttachment then
		flyAttachment:Destroy()
		flyAttachment = nil
	end

	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")
	local root = char and char:FindFirstChild("HumanoidRootPart")

	if hum then
		hum.AutoRotate = true
	end

	if root then
		root.AssemblyLinearVelocity = Vector3.zero
	end

	flyButton.Text = "✈ الطيران: إيقاف"
end

local function startFly()

	local char = player.Character
	local hum = char and char:FindFirstChildOfClass("Humanoid")
	local root = char and char:FindFirstChild("HumanoidRootPart")

	if not char or not hum or not root then
		return
	end

	flying = true
	flightControls.Visible = true
	hum.AutoRotate = false

	flyAttachment = Instance.new("Attachment")
	flyAttachment.Parent = root

	flyVelocity = Instance.new("LinearVelocity")
	flyVelocity.Attachment0 = flyAttachment
	flyVelocity.RelativeTo = Enum.ActuatorRelativeTo.World
	flyVelocity.MaxForce = math.huge
	flyVelocity.VectorVelocity = Vector3.zero
	flyVelocity.Parent = root

	flyConnection = RunService.RenderStepped:Connect(function()

		if not flying then return end

		local currentChar = player.Character
		local currentHum = currentChar and currentChar:FindFirstChildOfClass("Humanoid")

		if not currentChar or not currentHum then

			stopFly()
			return
		end

		local camera = workspace.CurrentCamera
		local move = currentHum.MoveDirection

		if UIS:IsKeyDown(Enum.KeyCode.W) then
			move += camera.CFrame.LookVector
		end

		if UIS:IsKeyDown(Enum.KeyCode.S) then
			move -= camera.CFrame.LookVector
		end

		if UIS:IsKeyDown(Enum.KeyCode.A) then
			move -= camera.CFrame.RightVector
		end

		if UIS:IsKeyDown(Enum.KeyCode.D) then
			move += camera.CFrame.RightVector
		end

		if upHeld or UIS:IsKeyDown(Enum.KeyCode.Space) then
			move += Vector3.new(0,1,0)
		end

		if downHeld or UIS:IsKeyDown(Enum.KeyCode.LeftControl) then
			move -= Vector3.new(0,1,0)
		end

		if move.Magnitude > 0 then
			flyVelocity.VectorVelocity = move.Unit * flySpeed
		else
			flyVelocity.VectorVelocity = Vector3.zero
		end
	end)

	flyButton.Text = "✈ الطيران: تشغيل"
end

flyButton.MouseButton1Click:Connect(function()

	if flying then
		stopFly()
	else
		startFly()
	end
end)

flySpeedButton.MouseButton1Click:Connect(function()

	flySpeedIndex += 1

	if flySpeedIndex > #flySpeeds then
		flySpeedIndex = 1
	end

	flySpeed = flySpeeds[flySpeedIndex]

	flySpeedButton.Text = "سرعة الطيران: "..flySpeed
end)

--------------------------------------------------
-- Respawn
--------------------------------------------------

player.CharacterAdded:Connect(function()

	if flying then
		stopFly()
	end

	task.wait(0.5)

	if espEnabled then
		refreshESP()
	end

	if godMode then
		startGod()
	end
end)
