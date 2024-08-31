### Creating new commands
To create new commands you can go into game.ReplicatedStorage (should be installed here) and select the module script labeled 'commands'
The table by default looks like this:
```
module.CMDS = {
	["kick"] = "kick",
	["ban"] = "ban",
	["team"] = "team"
}
```
You can add a new key (example: give command)
```
module.CMDS = { 
	["kick"] = "kick",
	["ban"] = "ban",
	["team"] = "team",
	["give"] = "give"
}
```
The name of the new command should match the key of the logic that were about to make in just a second.

### Adding logic to commands
To start head to the mainmodule.Logic and begin to edit, inside you should see a table titled module.Logic
```
module.Logic = {
	["ban"] = function(Origin, Text) -- UserID and Text will be passed, Do note The number is expected to be in minutes
		local Parse = string.split(Text, " ")
		local PlrName = Parse[2]
		local Time = tonumber(Parse[3])
		local Seconds = Time * 60
		local Reason = Parse[4] or "No reason Provided."
		-- Ban part
		local Playerservice = game:GetService("Players")
		local Config = {
			UserIds = {game.Players:GetUserIdFromNameAsync(PlrName)},
			ApplyToUniverse  = true,
			Duration = Seconds,
			DisplayReason = Reason,
			PrivateReason = "Player has been banned for "..Time.."Reason:"..Reason,
			ExcludeAltAccounts = true -- set to false to ban alts
		}
		Playerservice:BanAsync(Config)
	end,
	["kick"] = function(Origin, Text)
		local Parse = string.split(Text, " ")
		local plr = Parse[2]
		plr = game.Players:GetUserIdFromNameAsync(plr) -- changes plr to player
		local Reason = Parse[3] or "No reason"
		local PLRkick = game.Players:GetPlayerByUserId(plr)
		PLRkick:Kick(Reason)
	end,
	["team"] = function(origin, Text)
		local Parse = string.split(Text, " ")
		local Plr = Parse[2]
		local TeamName = Parse[3]
		local PLR = game.Players:GetUserIdFromNameAsync(Plr)
		local Player = game.Players:GetPlayerByUserId(PLR) -- change to PLR
		Player.Team = game.Teams:FindFirstChild(TeamName)
	end,
}
```
^ is the built in logic, to add logic for your own command you can do this:
["newcommand"] = function(Origin, Text)
  --logic
end,

Origin is textsource and Text is a string. from the origin you can get the UserID of the player. https://create.roblox.com/docs/reference/engine/classes/TextSource
this will be passed to the function unless you edit it, so you can utilize this if you need.

#### Client Code:
Not very important, just here so you can see the code.
```

local Module = require(game:GetService("ReplicatedStorage").ChatCommandsSuite:WaitForChild("Commands"))

wait(2)

local function Onvalue_(Value: TextChatCommand, OriginText: TextSource, Unfiltered: string)
	if Module.Settings_.IsGroup then
		if table.find(Module.Settings_.AllowedUserIds, OriginText.UserId) then
			-- logic
			print("Firing Command")
			local RemoteEvent = game:GetService("ReplicatedStorage").ChatCommandsSuite:WaitForChild("RE")
			RemoteEvent:FireServer(Value.Name,OriginText,Unfiltered) -- passes the command name, the origin, and the text after it.
			return
		end
	end
	if Module.Settings_.CreatorOnly then
		if OriginText.UserId == game.CreatorId then
			-- logic
			print("Firing Command")
			local RemoteEvent = game:GetService("ReplicatedStorage").ChatCommandsSuite:WaitForChild("RE")
			RemoteEvent:FireServer(Value.Name,OriginText,Unfiltered) -- passes the command name, the origin, and the text after it
			return
		end
	end
end

function Add_Function()
	for Index, Value in pairs(game.TextChatService:GetChildren()) do
		if Value:IsA("TextChatCommand") then
			wait()
			print("Loaded Command:"..Value.Name)
			Value.Triggered:Connect(function(OriginText, Unfiltered)
				Onvalue_(Value,OriginText,Unfiltered)
			end)
		end
	end
end

Add_Function()
```
Hopefully this should help you create chat commands easier.
