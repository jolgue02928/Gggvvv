local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")

local localPlayer = Players.LocalPlayer
local placeId = game.PlaceId
local currentJobId = game.JobId

-- Função para buscar um servidor diferente
local function findDifferentServer()
	local success, response = pcall(function()
		return game:HttpGet("https://games.roblox.com/v1/games/" .. placeId .. "/servers/Public?sortOrder=Asc&limit=100")
	end)

	if not success then
		warn("Erro ao obter servidores:", response)
		return nil
	end

	local data = HttpService:JSONDecode(response)

	for _, server in pairs(data.data) do
		if server.id ~= currentJobId and server.playing < server.maxPlayers then
			return server.id
		end
	end

	return nil
end

-- Função que faz a troca de servidor de 3 em 3 minutos
local function loopTeleport()
	while true do
		wait(180) -- 3 minutos
		local serverId = findDifferentServer()
		if serverId then
			TeleportService:TeleportToPlaceInstance(placeId, serverId, localPlayer)
			break -- este break é opcional; em geral o Teleport desconecta o script
		else
			warn("Nenhum servidor disponível no momento.")
		end
	end
end

-- Inicia o processo
loopTeleport()
