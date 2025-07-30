# Roblox Lua Scripts 

## A collection of beginner-friendly Roblox Studio scripts designed to teach core gameplay scripting concepts using Lua.

---

### Jump Boost Script:

```lua
--[[ CONFIGURABLE VARIABLES ]]
local JUMP_HEIGHT = 150
local JUMP_RESET_HEIGHT = 7.2
local BOOST_DURATION = 2
--[[ END CONFIG ]]

local jumpBoost = script.Parent

local function steppedOn(part)
  local parent = part.Parent
  if game.Players:GetPlayerFromCharacter(parent) then
    parent.Humanoid.JumpHeight = JUMP_HEIGHT
    wait(BOOST_DURATION)
    parent.Humanoid.JumpHeight = JUMP_RESET_HEIGHT
  end
end

jumpBoost.Touched:connect(steppedOn)

```

> Parent: Any part
> 
> Trigger: When a player touches the parent part
> 
> Result: The player can jump higher for a short time
> 
> Customization: Change the value of JumpHeight on line 6

---

### Speed Boost Script:

```lua
--[[ CONFIGURABLE VARIABLES ]]
local BOOST_SPEED = 100
local NORMAL_SPEED = 16
local BOOST_DURATION = 4
--[[ END CONFIG ]]

local speedBoost = script.Parent

local function steppedOn(part)
  local parent = part.Parent
  if game.Players:GetPlayerFromCharacter(parent) then
    parent.Humanoid.WalkSpeed = BOOST_SPEED
    wait(BOOST_DURATION)
    parent.Humanoid.WalkSpeed = NORMAL_SPEED
  end
end

speedBoost.Touched:connect(steppedOn)
```

> Parent: Any part
>
> Trigger: When a player touches the parent part
>
> Result: The player's speed is increased
>
> Customization: Change the value of WalkSpeed on line 6

---

### Kill/Heal Script:

```lua
--[[ CONFIGURABLE VARIABLES ]]
local HEALTH_VALUE = 0 -- Set this to something like 50 to heal or less to damage
--[[ END CONFIG ]]

script.Parent.Touched:connect(function(hit)
  if hit and hit.Parent and hit.Parent:FindFirstChild("Humanoid") then
    hit.Parent.Humanoid.Health = HEALTH_VALUE
  end
end)
```

> Parent: Any part
>
> Trigger: When a player touches the parent part
>
> Result: The player's health is set to 0 and is killed
>
> Customization: Change the value of Health on line 3 to a number other than 0 to hurt or heal the player

---

### Checkpoint Script:

```lua
local spawn = script.Parent
spawn.Touched:connect(function(hit)
  if hit and hit.Parent and hit.Parent:FindFirstChild("Humanoid") then
    local player = game.Players:GetPlayerFromCharacter(hit.Parent)
    local checkpointData = game.ServerStorage:FindFirstChild("CheckpointData")
    if not checkpointData then
      checkpointData = Instance.new("Model", game.ServerStorage)
      checkpointData.Name = "CheckpointData"
    end

    local checkpoint = checkpointData:FindFirstChild(tostring(player.userId))
    if not checkpoint then
      checkpoint = Instance.new("ObjectValue", checkpointData)
      checkpoint.Name = tostring(player.userId)

      player.CharacterAdded:connect(function(character)
        wait()
        local newUser = tostring(player.userId)
        local newData = game.ServerStorage.CheckpointData[newUser].Value.CFrame
        local newCheckpoint = newData + Vector3.new(0, 4, 0)
        character:WaitForChild("HumanoidRootPart").CFrame = newCheckpoint
      end)
    end

    checkpoint.Value = spawn
  end
end)
```

> Parent: Any part
>
> Trigger: When a player touches the parent part
>
> Result: The player will respawn on the parent part instead of the original SpawnLocation
>
> Customization: None

---

### Moving Platform Script:

```lua
--[[ CONFIGURABLE VARIABLES ]]
local AXIS = "x"         -- "x", "y", or "z"
local DISTANCE = 50
local SPEED = 1
--[[ END CONFIG ]]

local factor = 0
local initialPosition = script.Parent.CFrame.Position
local moveDirection = Vector3.new(
  AXIS == "x" and DISTANCE or 0,
  AXIS == "y" and DISTANCE or 0,
  AXIS == "z" and DISTANCE or 0
)
local destinationPosition = initialPosition + moveDirection
local RunService = game:GetService("RunService")

function movePlatform (time, step)
  factor = 0.5 * math.sin(time * SPEED) + 0.5
  local finalPosition = initialPosition:Lerp(destinationPosition, factor)
  script.Parent.CFrame = CFrame.new(finalPosition)
end

RunService.Stepped:Connect(movePlatform)
```

> Parent: Any part
>
> Trigger: None, the script constantly runs
>
> Result: The parent part moves back and forth between two positions
>
> Customization: Set axis on line 1 to "x", "y", or "z" to change the direction
>                Change the value of distance on line 2 to set how far it moves
>                Change the value of speed to alter how fast or slow it moves

---



### Hazard/Damage Script:

```lua
--[[ CONFIGURABLE VARIABLES ]]
local DAMAGE_AMOUNT = 8           -- Amount of health to change per second
local DAMAGE_DURATION = 5         -- How many seconds the effect lasts
local HEAL_INSTEAD = false        -- Set to true to heal instead of hurt
--[[ END CONFIG ]]

local poisonTimers = {}

local function findPlayerFromCharacter(char)
	for i, v in pairs(game.Players:GetPlayers()) do
		if v.Character == char then return v end
	end
end

script.Parent.Touched:Connect(function(part)
	local h = part.Parent:FindFirstChild("Humanoid")
	if h then
		local player = findPlayerFromCharacter(h.Parent)
		if player then
			if poisonTimers[player] then
				poisonTimers[player] = os.time() + DAMAGE_DURATION
			else
				poisonTimers[player] = os.time() + DAMAGE_DURATION
				while true do
					local t = poisonTimers[player]
					if t then
						if os.time() < t then
							if HEAL_INSTEAD then
								h.Health = math.min(h.MaxHealth, h.Health + DAMAGE_AMOUNT)
							else
								h:TakeDamage(DAMAGE_AMOUNT)
							end
						else
							poisonTimers[player] = nil
						end
					else
						break
					end
					wait(1)
				end
			end
		end
	end
end)
```

> Parent: Any part representing a poison or healing zone
>
> Trigger: When a player touches the part
>
> Result: The player either takes damage or is healed over time
>
> Customization:  
> - `DAMAGE_AMOUNT`: how much health to remove or add each second  
> - `DAMAGE_DURATION`: how long the effect lasts  
> - `HEAL_INSTEAD`: set to `true` to make this a healing zone instead of a damaging one



### Disappearing Platform Script:
```lua
--[[ CONFIGURABLE VARIABLES ]]
local DELAY_TO_DISAPPEAR = 1
local DELAY_TO_RESPAWN = 4
--[[ END CONFIG ]]

local part = script.Parent
local state = "Ready"

part.Touched:Connect(function()
	if state == "Ready" then
		state = "Triggered"
		part.Transparency = 0.5
		wait(DELAY_TO_DISAPPEAR)
		part.Transparency = 1
		part.CanCollide = false
		wait(DELAY_TO_RESPAWN)
		part.Transparency = 0
		part.CanCollide = true
		state = "Ready"
	end
end)

```

> Parent: Any part (platform) that you want to disappear and reappear
>
> Trigger: When a player touches the part
>
> Result: The part becomes semi-transparent, then disappears and becomes non-collidable for a short time before reappearing
>
> Customization:  
> - Change `DELAY_TO_DISAPPEAR` to set how long the platform waits before disappearing  
> - Change `DELAY_TO_RESPAWN` to set how long it stays gone before reappearing
---

### Laser Beams:

```lua
--[[ CONFIGURABLE VARIABLES ]]
local BASE_VANISH_TIME = 3         -- Average time laser is active (visible + dangerous)
local BASE_REAPPEAR_TIME = 3       -- Average time laser is inactive (invisible + safe)
local VANISH_VARIANCE = 1          -- Max +/- random variance for vanish time
local REAPPEAR_VARIANCE = 1        -- Max +/- random variance for reappear time
local DAMAGE_AMOUNT = 0.75         -- Amount of health taken away when touched
--[[ END CONFIG ]]

local laser = script.Parent
local active = true

local function getRandomTime(base, variance)
	return base + math.random() * 2 * variance - variance
end

laser.Touched:Connect(function(hit)
	if active and hit and hit.Parent and hit.Parent:FindFirstChild("Humanoid") then
		hit.Parent.Humanoid.Health = DAMAGE_AMOUNT
	end
end)

local function toggleLaser()
	while true do
		-- Laser active (danger!)
		active = true
		laser.Transparency = 0
		laser.CanCollide = true
		wait(getRandomTime(BASE_VANISH_TIME, VANISH_VARIANCE))

		-- Laser inactive (safe)
		active = false
		laser.Transparency = 1
		laser.CanCollide = false
		wait(getRandomTime(BASE_REAPPEAR_TIME, REAPPEAR_VARIANCE))
	end
end

toggleLaser()
```

> Parent: Any part representing a laser beam or trap
>
> Trigger: Script runs on a loop; laser becomes active/inactive on a timer
>
> Result: The laser turns on/off randomly and damages any player who touches it while it's active
>
> Customization:  
> - `BASE_VANISH_TIME`: how long the laser stays active  
> - `BASE_REAPPEAR_TIME`: how long the laser stays off  
> - `VANISH_VARIANCE` / `REAPPEAR_VARIANCE`: add randomness to timing  
> - `DAMAGE_AMOUNT`: how much health is removed on touch

```

---

### Teleporter:

```lua
--[[ CONFIGURABLE VARIABLES ]]
local COOLDOWN_TIME = 1.5     -- Time in seconds between allowed teleports
local HEIGHT_OFFSET = 3       -- Vertical offset to avoid getting stuck in floor
--[[ END CONFIG ]]

local teleporterFolder = script.Parent
local teleportParts = {}

-- Find exactly two teleport parts under the same parent
for _, child in pairs(teleporterFolder:GetChildren()) do
	if child:IsA("BasePart") then
		table.insert(teleportParts, child)
	end
end

-- Safety check
if #teleportParts ~= 2 then
	warn("Teleport system needs exactly 2 BaseParts under the same parent.")
	return
end

local partA = teleportParts[1]
local partB = teleportParts[2]

local lastTeleport = {}

local function canTeleport(player)
	return not lastTeleport[player] or tick() - lastTeleport[player] > COOLDOWN_TIME
end

local function teleportPlayer(player, destination)
	local char = player.Character
	if char and char:FindFirstChild("HumanoidRootPart") then
		char:MoveTo(destination.Position + Vector3.new(0, HEIGHT_OFFSET, 0))
		lastTeleport[player] = tick()
	end
end

local function onTouched(thisPart, otherPart)
	local character = otherPart.Parent
	local humanoid = character and character:FindFirstChild("Humanoid")
	if humanoid then
		local player = game.Players:GetPlayerFromCharacter(character)
		if player and canTeleport(player) then
			local destination = (thisPart == partA) and partB or partA
			teleportPlayer(player, destination)
		end
	end
end

-- Connect both parts
partA.Touched:Connect(function(hit) onTouched(partA, hit) end)
partB.Touched:Connect(function(hit) onTouched(partB, hit) end)

```
> Parent: A **folder** or **model** containing exactly **two** teleporter **parts**
>
> Trigger: When a player touches either part
>
> Result: The player is instantly teleported to the other part
>
> Customization:
> - `COOLDOWN_TIME`: How long a player must wait before teleporting again
> - `HEIGHT_OFFSET`: How far above the target the player will be placed to avoid overlap
>
> ⚠️ Note: The folder or model **must contain exactly two parts**. Add the script to that folder/model.
```

