-- Script de Auto Farm - Roube um Ovo (Delta Executor)
-- Com teletransporte e teleguiado ao ovo selecionado

local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- Configurações
local coletarDistancia = 50
local tempoEspera = 0.3
local autoTeletransporte = true -- Teleporta automaticamente para o ovo mais próximo
local modoTeleguiado = false -- Se true, você pode selecionar um ovo específico

-- Variável para ovo selecionado manualmente
local ovoSelecionado = nil

-- Função para listar todos os ovos no mapa
local function listarOvos()
    local ovos = {}
    for _, objeto in pairs(workspace:GetDescendants()) do
        if objeto:IsA("BasePart") and objeto.Name:lower():find("ovo") then
            table.insert(ovos, objeto)
        end
    end
    return ovos
end

-- Função para encontrar o ovo mais próximo
local function encontrarOvoMaisProximo()
    local menorDistancia = math.huge
    local ovoProximo = nil
    for _, objeto in pairs(workspace:GetDescendants()) do
        if objeto:IsA("BasePart") and objeto.Name:lower():find("ovo") then
            local distancia = (objeto.Position - humanoidRootPart.Position).Magnitude
            if distancia < menorDistancia then
                menorDistancia = distancia
                ovoProximo = objeto
            end
        end
    end
    return ovoProximo
end

-- Função para teleportar até o ovo
local function teleportarParaOvo(ovo)
    if ovo and ovo.Parent then
        humanoidRootPart.CFrame = CFrame.new(ovo.Position + Vector3.new(0, 3, 0))
    end
end

-- Função para coletar ovo (simula interação)
local function coletarOvo(ovo)
    if ovo and ovo.Parent then
        -- Tenta encontrar RemoteEvent/RemoteFunction
        local remoteEvent = ovo:FindFirstChildOfClass("RemoteEvent") or ovo:FindFirstChildOfClass("RemoteFunction")
        if remoteEvent then
            remoteEvent:FireServer(player)
        else
            -- Tenta ClickDetector
            local clickDetector = ovo:FindFirstChildOfClass("ClickDetector")
            if clickDetector then
                clickDetector:Fire(player)
            end
        end
    end
end

-- Função para teleguiar até o ovo selecionado (movimento suave)
local function teleguiarParaOvo(ovo)
    if ovo and ovo.Parent then
        local alvo = ovo.Position + Vector3.new(0, 3, 0)
        local distancia = (alvo - humanoidRootPart.Position).Magnitude
        
        -- Teleporta diretamente se estiver muito longe
        if distancia > 100 then
            humanoidRootPart.CFrame = CFrame.new(alvo)
        else
            -- Move suavemente (teleguiado)
            local passos = math.ceil(distancia / 10)
            for i = 1, passos do
                local novaPos = humanoidRootPart.Position + (alvo - humanoidRootPart.Position).Unit * 10
                humanoidRootPart.CFrame = CFrame.new(novaPos)
                task.wait(0.05)
            end
            humanoidRootPart.CFrame = CFrame.new(alvo)
        end
    end
end

-- Interface para selecionar ovo manualmente (via console)
local function selecionarOvoManual()
    print("=== OVOS DISPONÍVEIS ===")
    local ovos = listarOvos()
    for i, ovo in pairs(ovos) do
        print(i .. ": " .. ovo.Name .. " - Posição: " .. tostring(ovo.Position))
    end
    print("Digite o número do ovo que deseja mirar (ou 0 para cancelar):")
    
    -- Aguarda entrada do usuário (simplificado - você pode usar uma GUI depois)
    local escolha = tonumber(read()) -- Requer input do console
    if escolha and escolha > 0 and escolha <= #ovos then
        ovoSelecionado = ovos[escolha]
        print("Ovo selecionado: " .. ovoSelecionado.Name)
        return true
    else
        ovoSelecionado = nil
        print("Seleção cancelada")
        return false
    end
end

-- Loop principal
while task.wait(tempoEspera) do
    pcall(function()
        local ovoAlvo = nil
        
        if modoTeleguiado and ovoSelecionado then
            -- Modo teleguiado: usa o ovo selecionado manualmente
            ovoAlvo = ovoSelecionado
            if ovoAlvo and ovoAlvo.Parent then
                teleguiarParaOvo(ovoAlvo)
                coletarOvo(ovoAlvo)
            else
                -- Se o ovo selecionado foi coletado/removido, procura outro
                ovoSelecionado = nil
                print("Ovo selecionado foi coletado! Selecionando novo...")
            end
        elseif autoTeletransporte then
            -- Modo automático: teleporta para o ovo mais próximo
            ovoAlvo = encontrarOvoMaisProximo()
            if ovoAlvo then
                teleportarParaOvo(ovoAlvo)
                coletarOvo(ovoAlvo)
            end
        else
            -- Modo normal: apenas coleta ovos próximos
            for _, objeto in pairs(workspace:GetDescendants()) do
                if objeto:IsA("BasePart") and objeto.Name:lower():find("ovo") then
                    local distancia = (objeto.Position - humanoidRootPart.Position).Magnitude
                    if distancia <= coletarDistancia then
                        coletarOvo(objeto)
                    end
                end
            end
        end
    end)
end

-- Comandos para usar no console (execute manualmente se quiser)
--[[
-- Para ativar modo teleguiado:
modoTeleguiado = true

-- Para selecionar um ovo manualmente:
selecionarOvoManual()

-- Para desativar auto teletransporte:
autoTeletransporte = false

-- Para listar todos os ovos:
local ovos = listarOvos()
for i, ovo in pairs(ovos) do
    print(i .. ": " .. ovo.Name)
end
]]
