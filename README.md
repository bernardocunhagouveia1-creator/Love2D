-- LocalScript: place in StarterPlayer > StarterPlayerScripts
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

local settings = {
    inactivitySeconds = 30,
    minimumSeconds = 5,
    maximumSeconds = 300,
    stepSeconds = 5,
}

local idleTime = 0
local promptVisible = false
local player = { x = 0, y = 0, speed = 220, radius = 16 }
local minusButton, plusButton
local playAreaTop = 0

local function layout()
    local width, height = love.graphics.getDimensions()
    local panelWidth, panelHeight = 440, 150
    local panelX = (width - panelWidth) / 2
    local panelY = 24

    minusButton = { x = panelX + 205, y = panelY + 75, w = 44, h = 40 }
    plusButton = { x = panelX + 295, y = panelY + 75, w = 44, h = 40 }
    playAreaTop = panelY + panelHeight + 20

    if player.x == 0 then
        player.x = width / 2
        player.y = (playAreaTop + height) / 2
    end
end

local function resetInactivity()
    idleTime = 0
    promptVisible = false
end

function love.load()
    love.window.setTitle("Movement Reminder")
    love.window.setMode(800, 600, { resizable = true })
    layout()
end

function love.resize()
    layout()
end

function love.update(dt)
    local dx, dy = 0, 0

    if love.keyboard.isDown("left", "a") then dx = dx - 1 end
    if love.keyboard.isDown("right", "d") then dx = dx + 1 end
    if love.keyboard.isDown("up", "w") then dy = dy - 1 end
    if love.keyboard.isDown("down", "s") then dy = dy + 1 end

    local length = math.sqrt(dx * dx + dy * dy)
    if length > 0 then
        dx, dy = dx / length, dy / length

        local width, height = love.graphics.getDimensions()
        local oldX, oldY = player.x, player.y

        player.x = math.max(player.radius, math.min(width - player.radius,
            player.x + dx * player.speed * dt))
        player.y = math.max(playAreaTop + player.radius,
            math.min(height - player.radius, player.y + dy * player.speed * dt))

        if player.x ~= oldX or player.y ~= oldY then
            resetInactivity()
        end
    else
        idleTime = idleTime + dt
        if idleTime >= settings.inactivitySeconds then
            promptVisible = true
        end
    end
end

local function drawButton(button, label)
    love.graphics.setColor(0.22, 0.27, 0.36)
    love.graphics.rectangle("fill", button.x, button.y, button.w, button.h, 8)
    love.graphics.setColor(1, 1, 1)
    love.graphics.printf(label, button.x, button.y + 8, button.w, "center")
end

function love.draw()
    local width, height = love.graphics.getDimensions()

    love.graphics.clear(0.08, 0.10, 0.14)

    -- Settings panel
    local panelWidth, panelHeight = 440, 150
    local panelX = (width - panelWidth) / 2
    love.graphics.setColor(0.14, 0.17, 0.23)
    love.graphics.rectangle("fill", panelX, 24, panelWidth, panelHeight, 12)

    love.graphics.setColor(1, 1, 1)
    love.graphics.printf("Movement Reminder", panelX, 40, panelWidth, "center")
    love.graphics.printf(
        "Prompt after " .. settings.inactivitySeconds .. " seconds without movement",
        panelX, 79, panelWidth, "center"
    )

    drawButton(minusButton, "−")
    drawButton(plusButton, "+")

    love.graphics.setColor(0.45, 0.75, 1)
    love.graphics.circle("fill", player.x, player.y, player.radius)

    love.graphics.setColor(0.8, 0.84, 0.9)
    love.graphics.printf("Move with WASD or the arrow keys",
        0, height - 30, width, "center")

    if promptVisible then
        local cardWidth, cardHeight = 340, 100
        local cardX = (width - cardWidth) / 2
        local cardY = (height - cardHeight) / 2

        love.graphics.setColor(0, 0, 0, 0.55)
        love.graphics.rectangle("fill", 0, 0, width, height)
        love.graphics.setColor(0.95, 0.72, 0.24)
        love.graphics.rectangle("fill", cardX, cardY, cardWidth, cardHeight, 12)
        love.graphics.setColor(0.12, 0.12, 0.12)
        love.graphics.printf("Move to stay active", cardX, cardY + 20, cardWidth, "center")
        love.graphics.printf("Use WASD or the arrow keys", cardX, cardY + 55, cardWidth, "center")
    end
end

function love.mousepressed(x, y, button)
    if button ~= 1 then return end

    local function contains(rect)
        return x >= rect.x and x <= rect.x + rect.w
            and y >= rect.y and y <= rect.y + rect.h
    end

    if contains(minusButton) then
        settings.inactivitySeconds = math.max(
            settings.minimumSeconds,
            settings.inactivitySeconds - settings.stepSeconds
        )
        resetInactivity()
    elseif contains(plusButton) then
        settings.inactivitySeconds = math.min(
            settings.maximumSeconds,
            settings.inactivitySeconds + settings.stepSeconds
        )
        resetInactivity()
    end
end
