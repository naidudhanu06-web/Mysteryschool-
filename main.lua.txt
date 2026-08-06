-- ============================================================
-- GAME: MYSTERY SCHOOL - COMPLETE INTEGRATED CODE
-- Engine: Solar2D (Corona SDK)
-- Features: Animated Sprite, D-Pad Controls, Glow/Shadow Coatings, 
--           and Player Record Data Saving (JSON)
-- ============================================================

display.setStatusBar( display.HiddenStatusBar )
local json = require( "json" )

---------------------------------------------------------------
-- 1. RECORD SYSTEM (డేటా సేవ్ & లోడ్ లాజిక్)
---------------------------------------------------------------
local playerRecord = {
    playerName = "Dhanu",
    currentRoom = "School Entrance",
    highScore = 100,
    itemsFound = { "Keycard", "Flashlight" }
}

-- రికార్డ్ డేటాను దాచడానికి (Save Record)
local function saveGameRecord( data )
    local path = system.pathForFile( "player_record.json", system.DocumentsDirectory )
    local file, errorReport = io.open( path, "w" )
    if file then
        file:write( json.encode( data ) )
        io.close( file )
        print( "Record Saved Successfully!" )
    else
        print( "Error saving record: ", errorReport )
    end
end

-- రికార్డ్ డేటాను లోడ్ చేయడానికి (Load Record)
local function loadGameRecord()
    local path = system.pathForFile( "player_record.json", system.DocumentsDirectory )
    local file = io.open( path, "r" )
    if file then
        local contents = file:read( "*a" )
        io.close( file )
        return json.decode( contents )
    end
    return nil
end

-- గేమ్ స్టార్ట్ అయినప్పుడు సేవ్ చేయడం
saveGameRecord( playerRecord )

---------------------------------------------------------------
-- 2. BACKGROUND & ATMOSPHERE COATING
---------------------------------------------------------------
-- బ్యాక్‌గ్రౌండ్ ఇమేజ్
local background = display.newImageRect( "43013.jpg", display.actualContentWidth, display.actualContentHeight )
background.x = display.contentCenterX
background.y = display.contentCenterY

-- మిస్టరీ అట్మాస్ఫియర్ కోసం డార్క్ కోటింగ్ ఓవర్‌లే
local bgCoating = display.newRect( display.contentCenterX, display.contentCenterY, display.actualContentWidth, display.actualContentHeight )
bgCoating:setFillColor( 0.05, 0.05, 0.15, 0.45 )

---------------------------------------------------------------
-- 3. SPRITE SHEET & CHARACTER SETUP
---------------------------------------------------------------
local sheetOptions = {
    width = 64,
    height = 64,
    numFrames = 16,
    sheetContentWidth = 256,
    sheetContentHeight = 256
}

local characterSheet = graphics.newImageSheet( "43017.png", sheetOptions )

local sequenceData = {
    { name = "walkDown",  start = 1,  count = 4, time = 400, loopCount = 0 },
    { name = "walkLeft",  start = 5,  count = 4, time = 400, loopCount = 0 },
    { name = "walkRight", start = 9,  count = 4, time = 400, loopCount = 0 },
    { name = "walkUp",    start = 13, count = 4, time = 400, loopCount = 0 },
}

local playerGroup = display.newGroup()

-- క్యారెక్టర్ షాడో కోటింగ్
local characterShadow = display.newEllipse( playerGroup, 0, 22, 28, 10 )
characterShadow:setFillColor( 0, 0, 0, 0.6 )

-- క్యారెక్టర్ ఆరా గ్లో కోటింగ్
local characterGlow = display.newCircle( playerGroup, 0, 0, 32 )
characterGlow:setFillColor( 0.2, 0.8, 1, 0.25 )
transition.to( characterGlow, { time = 800, alpha = 0.05, xScale = 1.2, yScale = 1.2, iterations = 0, transition = easing.continuousLoop } )

-- స్ప్రైట్ క్యారెక్టర్
local player = display.newSprite( playerGroup, characterSheet, sequenceData )
player.x, player.y = 0, 0
player:setSequence( "walkDown" )

playerGroup.x = display.contentCenterX
playerGroup.y = display.contentCenterY + 40

---------------------------------------------------------------
-- 4. MOVEMENT LOGIC
---------------------------------------------------------------
local moveSpeed = 4
local currentDirection = nil

local function updateMovement()
    if ( currentDirection == "up" ) then
        playerGroup.y = playerGroup.y - moveSpeed
    elseif ( currentDirection == "down" ) then
        playerGroup.y = playerGroup.y + moveSpeed
    elseif ( currentDirection == "left" ) then
        playerGroup.x = playerGroup.x - moveSpeed
    elseif ( currentDirection == "right" ) then
        playerGroup.x = playerGroup.x + moveSpeed
    end
end

Runtime:addEventListener( "enterFrame", updateMovement )

---------------------------------------------------------------
-- 5. GLOWING D-PAD BUTTON CONTROLS
---------------------------------------------------------------
local function createCoatedButton( x, y, label, direction, animName )
    local btnGroup = display.newGroup()
    btnGroup.x, btnGroup.y = x, y

    -- గ్లో లేయర్
    local btnGlow = display.newRect( btnGroup, 0, 0, 50, 50 )
    btnGlow:setFillColor( 0, 0.8, 1, 0.3 )

    -- మెయిన్ బటన్
    local btn = display.newRect( btnGroup, 0, 0, 44, 44 )
    btn:setFillColor( 0.1, 0.1, 0.15, 0.85 )
    btn.strokeWidth = 2
    btn:setStrokeColor( 0, 0.8, 1 )

    local txt = display.newText( btnGroup, label, 0, 0, native.systemFontBold, 18 )
    txt:setFillColor( 1, 1, 1 )

    local function handleTouch( event )
        if ( event.phase == "began" ) then
            currentDirection = direction
            player:setSequence( animName )
            player:play()
            btn:setFillColor( 0, 0.6, 0.9, 0.9 )
            btnGlow.alpha = 0.8
        elseif ( event.phase == "ended" or event.phase == "cancelled" ) then
            currentDirection = nil
            player:pause()
            btn:setFillColor( 0.1, 0.1, 0.15, 0.85 )
            btnGlow.alpha = 0.3
        end
        return true
    end

    btn:addEventListener( "touch", handleTouch )
end

local startX = 70
local startY = display.contentHeight - 70

createCoatedButton( startX, startY - 45, "▲", "up", "walkUp" )
createCoatedButton( startX, startY + 45, "▼", "down", "walkDown" )
createCoatedButton( startX - 45, startY, "◄", "left", "walkLeft" )
createCoatedButton( startX + 45, startY, "►", "right", "walkRight" )

---------------------------------------------------------------
-- 6. UI TITLE & RECORD STATUS DISPLAY
---------------------------------------------------------------
local title = display.newText( "🕵️ MYSTERY SCHOOL", display.contentCenterX, 20, native.systemFontBold, 16 )
title:setFillColor( 0, 0.9, 1 )

local recordText = display.newText( "Record Status: Saved", display.contentCenterX, 42, native.systemFont, 11 )
recordText:setFillColor( 0.8, 0.8, 0.8 )Solar2D Companion