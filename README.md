local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local placeId = 116495829188952 -- Dead Rails
local currentJobId = game.JobId
local triedServers = {} -- Para evitar servidores repetidos

-- Marca o servidor atual como já visitado
triedServers[currentJobId] = true

-- Função para encontrar um servidor diferente
local function findNewServer()
    local nextPageCursor = nil

    repeat
        local url = ("https://games.roblox.com/v1/games/%d/servers/Public?sortOrder=Asc&limit=100"):format(placeId)
        if nextPageCursor then
            url = url .. "&cursor=" .. nextPageCursor
        end

        local success, response = pcall(function()
            return game:HttpGet(url)
        end)

        if not success then
            warn("Erro ao buscar servidores:", response)
            return nil
        end

        local data = HttpService:JSONDecode(response)
        nextPageCursor = data.nextPageCursor

        for _, server in ipairs(data.data) do
            if server.playing < server.maxPlayers and not triedServers[server.id] then
                triedServers[server.id] = true
                return server.id
            end
        end

    until not nextPageCursor

    return nil
end

-- Loop de troca de servidor
while true do
    wait(180) -- Espera 3 minutos
    local newServerId = findNewServer()
    if newServerId then
        TeleportService:TeleportToPlaceInstance(placeId, newServerId, player)
        break -- Jogador será teleportado; script para
    else
        warn("Nenhum servidor novo encontrado. Tentando novamente em 3 minutos.")
    end
end
