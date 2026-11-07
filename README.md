-- LocalScript: place in StarterPlayer > StarterPlayerScripts
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local MIN_SECONDS = 5
local MAX_SECONDS = 3600
local DEFAULT_SECONDS = 30
local BANNER_SECONDS = 8

local COLOR_PANEL = Color3.fromRGB(28, 30, 36)
local COLOR_FIELD = Color3.fromRGB(42, 45, 54)
local COLOR_START = Color3.fromRGB(46, 150, 90)
local COLOR_STOP = Color3.fromRGB(170, 60, 60)
local COLOR_ACCENT = Color3.fromRGB(255, 190, 60)
local COLOR_TEXT = Color3.fromRGB(240, 240, 240)
local COLOR_MUTED = Color3.fromRGB(170, 175, 185)

local function make(className, props, parent)
	local object = Instance.new(className)
	for key, value in pairs(props) do
		object[key] = value
	end
	object.Parent = parent
	return object
end

local function makeButton(name, text, position, size, color, parent)
	local button = make("TextButton", {
		Name = name,
		Position = position,
		Size = size,
		BackgroundColor3 = color,
		BorderSizePixel = 0,
		Text = text,
		TextColor3 = COLOR_TEXT,
		TextSize = 16,
		Font = Enum.Font.GothamBold,
	}, parent)
	make("UICorner", {CornerRadius = UDim.new(0, 8)}, button)
	return button
end

local gui = make("ScreenGui", {
	Name = "MovePromptUI",
	ResetOnSpawn = false,
	ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
}, playerGui)

local panel = make("Frame", {
	Name = "Panel",
	AnchorPoint = Vector2.new(0.5, 0),
	Position = UDim2.new(0.5, 0, 0, 16),
	Size = UDim2.fromOffset(300, 200),
	BackgroundColor3 = COLOR_PANEL,
	BorderSizePixel = 0,
}, gui)
make("UICorner", {CornerRadius = UDim.new(0, 12)}, panel)

make("TextLabel", {
	Name = "Title",
	Position = UDim2.fromOffset(16, 12),
	Size = UDim2.new(1, -32, 0, 26),
	BackgroundTransparency = 1,
	Text = "Move Reminder",
	TextColor3 = COLOR_TEXT,
	TextSize = 20,
	Font = Enum.Font.GothamBold,
	TextXAlignment = Enum.TextXAlignment.Left,
}, panel)

make("TextLabel", {
	Name = "Caption",
	Position = UDim2.fromOffset(16, 46),
	Size = UDim2.new(1, -32, 0, 18),
	BackgroundTransparency = 1,
	Text = "Prompt me to move every (seconds)",
	TextColor3 = COLOR_MUTED,
	TextSize = 13,
	Font = Enum.Font.Gotham,
	TextXAlignment = Enum.TextXAlignment.Left,
}, panel)

local input = make("TextBox", {
	Name = "DurationInput",
	Position = UDim2.fromOffset(16, 70),
	Size = UDim2.new(1, -32, 0, 36),
	BackgroundColor3 = COLOR_FIELD,
	BorderSizePixel = 0,
	Text = tostring(DEFAULT_SECONDS),
	PlaceholderText = MIN_SECONDS .. " to " .. MAX_SECONDS,
	PlaceholderColor3 = COLOR_MUTED,
	TextColor3 = COLOR_TEXT,
	TextSize = 18,
	Font = Enum.Font.Gotham,
	ClearTextOnFocus = false,
}, panel)
make("UICorner", {CornerRadius = UDim.new(0, 8)}, input)

local startButton = makeButton("StartButton", "Start", UDim2.fromOffset(16, 118), UDim2.new(0.5, -24, 0, 38), COLOR_START, panel)
local stopButton = makeButton("StopButton", "Stop", UDim2.new(0.5, 8, 0, 118), UDim2.new(0.5, -24, 0, 38), COLOR_STOP, panel)

local status = make("TextLabel", {
	Name = "Status",
	Position = UDim2.fromOffset(16, 166),
	Size = UDim2.new(1, -32, 0, 20),
	BackgroundTransparency = 1,
	Text = "Set a duration and press Start.",
	TextColor3 = COLOR_MUTED,
	TextSize = 13,
	Font = Enum.Font.Gotham,
	TextXAlignment = Enum.TextXAlignment.Left,
}, panel)

local prompt = make("Frame", {
	Name = "MovePrompt",
	AnchorPoint = Vector2.new(0.5, 0.5),
	Position = UDim2.new(0.5, 0, 0.35, 0),
	Size = UDim2.fromOffset(320, 130),
	BackgroundColor3 = COLOR_PANEL,
	BorderSizePixel = 0,
	Visible = false,
}, gui)
make("UICorner", {CornerRadius = UDim.new(0, 12)}, prompt)
local stroke = make("UIStroke", {Color = COLOR_ACCENT, Thickness = 3}, prompt)

make("TextLabel", {
	Name = "PromptTitle",
	Position = UDim2.fromOffset(16, 16),
	Size = UDim2.new(1, -32, 0, 34),
	BackgroundTransparency = 1,
	Text = "Time to move!",
	TextColor3 = COLOR_ACCENT,
	TextSize = 26,
	Font = Enum.Font.GothamBold,
}, prompt)

make("TextLabel", {
	Name = "PromptBody",
	Position = UDim2.fromOffset(16, 52),
	Size = UDim2.new(1, -32, 0, 20),
	BackgroundTransparency = 1,
	Text = "Take a step to keep your session active.",
	TextColor3 = COLOR_TEXT,
	TextSize = 14,
	Font = Enum.Font.Gotham,
	TextWrapped = true,
}, prompt)

local dismissButton = makeButton("DismissButton", "Dismiss", UDim2.new(0.5, -70, 1, -50), UDim2.fromOffset(140, 36), COLOR_FIELD, prompt)

local pulse = TweenService:Create(
	stroke,
	TweenInfo.new(0.6, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut, -1, true),
	{Transparency = 0.8}
)

local sessionId = 0
local duration = DEFAULT_SECONDS
local remaining = DEFAULT_SECONDS
local bannerToken = nil

local function hidePrompt()
	bannerToken = nil
	pulse:Cancel()
	stroke.Transparency = 0
	prompt.Visible = false
end

local function showPrompt()
	prompt.Visible = true
	pulse:Play()
	local token = {}
	bannerToken = token
	task.delay(BANNER_SECONDS, function()
		if bannerToken == token then
			hidePrompt()
		end
	end)
end

local function stopTimer(message)
	sessionId += 1
	hidePrompt()
	status.Text = message
end

local function startTimer()
	local seconds = tonumber(input.Text)
	if not seconds or seconds ~= math.floor(seconds) or seconds < MIN_SECONDS or seconds > MAX_SECONDS then
		status.Text = ("Enter a whole number from %d to %d."):format(MIN_SECONDS, MAX_SECONDS)
		return
	end

	sessionId += 1
	local mySession = sessionId
	duration = seconds
	remaining = seconds
	hidePrompt()

	task.spawn(function()
		while sessionId == mySession do
			status.Text = ("Next move prompt in %ds"):format(remaining)
			task.wait(1)
			if sessionId ~= mySession then
				break
			end
			remaining -= 1
			if remaining <= 0 then
				remaining = duration
				showPrompt()
			end
		end
	end)
end

input.Changed:Connect(function(property)
	if property == "Text" then
		input.Text = (input.Text:gsub("%D", ""))
	end
end)

input.FocusLost:Connect(function(enterPressed)
	if enterPressed then
		startTimer()
	end
end)

startButton.Activated:Connect(startTimer)
stopButton.Activated:Connect(function()
	stopTimer("Stopped.")
end)
dismissButton.Activated:Connect(hidePrompt)
