local ms = game:GetService("MessagingService")
local event = Instance.new("RemoteEvent")
event.Name = "event"
event.Parent = game.ReplicatedStorage
local datastores = require(game.ServerScriptService:WaitForChild("dataservice"))
local role = Instance.new("RemoteEvent")
role.Name = "role"
role.Parent = game.ReplicatedStorage
local role1 = Instance.new("RemoteEvent")
role1.Name = "role1"
role1.Parent = game.ReplicatedStorage
local whitelist = {}
local settings = {
	prefix = "/",--defaul prefix: type this before command
	perplayerprefix = {
		[3831493386] = "!",--an example of a player prefix 
		--dont worry the example wont give the player admin perms
	},--if you want to have a different prefix for a player add there user id and set it to the prefix
	privateserverownerperm = 3, --set this to how much admin private server owners have
	--1 = basic commands like mute and kill
	--2 = can kick players
	--3 = most commands
	--4 = shutdown server
	--5 = do anything to the entire game not recomended
}
local serverbans = {}--this is not saved and is purly temporary
local commands = {
	kick = function(args)
		local target = args.target
		local reason = args.parts[2] or "kicked by admin"
		target:Kick(reason)
	end,

	mute = function(args)
	
		event:FireClient(args.target,args.target, "mute")
	end,

	unmute = function(args)
		event:FireClient(args.target,args.target, "unmute")
	end,

	ban = function(args)
		if args.plr.UserId == args.target.UserId then
			local errclone = script.Parent.error:Clone()
			errclone.Parent = args.plr.PlayerGui
			errclone.errortext.Text = ("you cant ban yourself! you want to be banned?")
			game.Debris:AddItem(errclone,3)
			return 
		end
		if args.plr.UserId == game.PrivateServerOwnerId and args.plr.UserId ~= game.CreatorId then
		local errclone = script.Parent.error:Clone()
		errclone.Parent = args.plr.PlayerGui
		errclone.errortext.Text = ("private server owners please use /sban for server ban")
		game.Debris:AddItem(errclone,3)
			return
		end
		local target = args.target
		local display = args.parts[2] or "banned"
		local private = args.parts[3] or "banned"
		local success,err = pcall(function()
			local banData = {
				UserId = target.UserId,
				DisplayReason = display,
				PrivateReason = private,
				ApplyToUniverse = true,
				Duration = -1,
			}
			game.Players:BanAsync(banData)
		end)
	end,

	shutdown = function(args)
		local reason = args.parts[2] or "server shutdown by admin"
		for _, player in pairs(game.Players:GetPlayers()) do
			player:Kick(reason)
		end
	end,

	tp = function(args)
		local char1 = args.target.Character
		local char2 = args.plr.Character
		char1:PivotTo(char2:GetPivot())
	end,

	kill = function(args)
		local target = args.target
		local char = target.Character
		char:FindFirstChildOfClass("Humanoid").Health = 0
	end,

	ga = function(args)
		local success,err = pcall(function()
			local messagedata = {
				message = table.concat(args.parts, " ", 2) or "" ,
				sender = "from ".. args.plr.Name,
			}
			ms:PublishAsync("global",messagedata)
		end) --end pcall
		if not success then print(err) end
	end,--end function

	test = function(args)
		print("test")
	end, --end test function

	rank = function(args)
		local target = args.target
		local clone = script.Parent.rank:Clone()
		clone.Parent = args.plr.PlayerGui
		local plr = args.plr 
		game.ReplicatedStorage.role1:FireClient(plr,plr,target)
	end,
	fly = function(args)
		event:FireClient(args.target,args.target,"fly")
	end,
	unfly = function(args)
		event:FireClient(args.target,args.target,"unfly")
	end,
	bj = function(args)
		--this command is better for r6
		args.target.Character:BreakJoints()
	end,
	speed = function(args)
		local target = args.target
		local speed = tonumber(args.parts[3]) or 16
		local char = target.Character
		local humanoid = char:FindFirstChildOfClass("Humanoid")
		humanoid.WalkSpeed = speed
	end,
	jump = function(args)
		local targ = args.target
		local hum = targ.Character:FindFirstChildOfClass("Humanoid")
		hum.UseJumpPower = true
		local jumppower = tonumber(args.parts[3]) or 50
		hum.JumpPower = jumppower
	end,
	unrank = function(args)
		local target = args.target
		whitelist[target.UserId] = nil
		args.target:kick("rank removed kicking to refresh perms :C")
		print(whitelist[target.UserId])
	end,
	message = function(args)
		local target = args.target
		local ui = script.Parent.announce.announcement
		local message = table.concat(args.parts, " ", 2) or "" 
		local sender = "from ".. args.plr.Name
		ui.TextLabel.Text = tostring(sender)
		ui.message.Text = tostring(message)
		game.ReplicatedStorage.announce:FireAllClients(ui)
	end,
	
	--add more commands here
}

local cmdpower = {
	kick = 2,
	mute = 1, 
	unmute = 1, 
	ban = 3, 
	shutdown = 4, 
	tp = 1,
	kill = 1, 
	ga = 4,
	rank = 4,
	test = 6, 
	jump = 1,
	speed = 1,
	fly = 2,
	unfly = 2,
	unrank = 4,
	message = 3,
	bj = 2,
	
}

local StrictTarget = {
	kick = true,
	ban = true
}

--save and load whitelist
local dss = game:GetService("DataStoreService")
local datastore = dss:GetDataStore("shadowadmin3.0")
local module = require(game.ServerScriptService.dataservice)



print(whitelist)
function canrun(cmd,plr)
	local pp = whitelist[plr.UserId] or 0
	local cp = cmdpower[string.lower(cmd)] or 0
	if pp >= cp then
		return true
	else
		return false
	end
end

game.Players.PlayerAdded:Connect(function(plr)
	--check if they are serevr banned
	if table.find(serverbans,plr.UserId) then
		plr:Kick("you are serve")
	end
	--chat handler
	
	local roletag = Instance.new("IntValue")
	roletag.Name = "role"
	roletag.Parent = plr
	roletag.Value = module.load(plr.UserId,datastore) or 0
	if plr.UserId == game.CreatorId then
		roletag.Value = 5
	end
	if roletag.Value > 0 then
		whitelist[plr.UserId] = roletag.Value
	end
	plr.Chatted:Connect(function(message)
		if message:sub(1,1) ~= settings.prefix and message:sub(1,1) ~= settings.perplayerprefix[plr.UserId] then
			return 
		end
		local target = nil
		local parts = string.split(message," ")
		if parts[2] then
			for i,v in ipairs(game.Players:GetPlayers()) do
				if string.find(string.lower(v.Name),string.lower(parts[2])) then
					target = v 
					break
				end
			end
		end
		local cmd = parts[1]:sub(2):lower()
		--build args
		local args = {
			plr = plr,
			parts = parts,
			target = target or (not StrictTarget[cmd] and plr or nil)
		}

		--fire command
		if commands[cmd] and canrun(cmd,plr) then
			commands[cmd](args)
		elseif not canrun(cmd, plr) then
			print("NAHHHH")	

		end
	end)
end)


game.ReplicatedStorage:WaitForChild("role").OnServerEvent:Connect(function(plr,target,rank)
	local role = plr:WaitForChild("role",2)
	if plr.role.Value <= rank and plr.role.Value < 4 then
		local clone = script.Parent.error:Clone()
		clone.Parent = plr.PlayerGui
		clone.errortext.Text = "ERROR: you dont have permision to give this rank"
		warn("player exicuted command with little or no permision")
			task.wait(3)
			clone:Destroy()
			return
	end
	whitelist[target.UserId] = rank
	module.save(plr.UserId,rank,datastore)
end)
--data stores
game.Players.PlayerRemoving:Connect(function(plr)
	if whitelist[plr.UserId] then
		module.save(plr.UserId,whitelist[plr.UserId],datastore)
	end
end)
