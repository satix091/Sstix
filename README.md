-- Satixhub - Script completo para Blox Fruits (Mobile + PC)

repita wait() até que o jogo:IsLoaded()

-- Serviços
Jogadores locais = jogo:GetService("Jogadores")
Armazenamento Replicado local = jogo:GetService("Armazenamento Replicado")
Espaço de trabalho local = jogo:GetService("Espaço de trabalho")
local RunService = jogo:GetService("RunService")
Gerenciador de Entrada Virtual local = jogo:GetService("Gerenciador de Entrada Virtual")
local LocalPlayer = Jogadores.LocalPlayer
Caractere local = LocalPlayer.Character ou LocalPlayer.CharacterAdded:Wait()
local HumanoidRootPart = Personagem:WaitForChild("HumanoidRootPart")

-- Variáveis
AutoFarmEnabled local = verdadeiro
AutoPvPEnabled local = verdadeiro
SpeedHackEnabled local = verdadeiro
AutoESPEnabled local = verdadeiro
local MacroPCOnly = true -- Apenas para PC
AntiReportEnabled local = verdadeiro
AutoQuestEnabled local = verdadeiro
AutoStatsEnabled local = verdadeiro
ItemCollectorEnabled local = verdadeiro

-- ======= ANTI REPORT (Reduzindo Cálculos de Fast Attack) =======
spawn(função()
    enquanto espera(0,1) faça
        pcall(função()
            -- Realiza o ataque de forma mais "normal" com pausas entre os golpes
            se não for AutoPvPEnabled então retorne end
            local attackKey = "Z" -- Pode ser alterado conforme a necessidade
            VirtualInputManager:SendKeyEvent(verdadeiro, chave de ataque, falso, jogo)
            wait(math.random(0.2, 0.5)) -- Pausa estimada para evitar cálculos rápidos
            VirtualInputManager:SendKeyEvent(falso, chave de ataque, falso, jogo)
        fim)
    fim
fim)

-- ======= FAZENDA AUTOMÁTICA =======
multidões locais = {
    ["Primeiro Mar"] = {
        {nome="Bandit", nível=10, missão="BanditQuest1", missãoPart=1, pos=Vector3.new(1152, 16, 1630)}
    },
    ["Segundo Mar"] = {
        {nome="Raider", nível=700, missão="Area1Quest", missãoPart=1, pos=Vector3.new(-5035, 314, -2836)}
    },
    ["Terceiro Mar"] = {
        {name="Pirata Milionário", nível=1900, quest="PiratePortQuest", questPart=2, pos=Vector3.new(-287, 44, 5559)}
    }
}

-- Função para Teletransportar
função TP(pos)
    HumanoidRootPart.CFrame = CFrame.new(pos.X, pos.Y + 3, pos.Z)
fim

-- Função para pegar o mundo (Primeira, Segunda, Terceira Mar)
função GetWorld()
    id local = jogo.PlaceId
    se id == 7449423635 então retorne "Terceiro Mar" fim
    se id == 4442272183 então retorne "Segundo Mar" fim
    retornar "Primeiro Mar"
fim

-- Função para pegar o mob atual para o AutoFarm
função GetMob()
    nível local = LocalPlayer.Data.Level.Value
    para _, m em pares(mobs[GetWorld()]) faça
        se nível >= m.level então retorne m fim
    fim
    retornar mobs[GetWorld()][1]
fim

spawn(função()
    enquanto AutoFarmEnabled e wait(1) fazem
        pcall(função()
            multidão local = GetMob()
            ReplicatedStorage.Remotes.CommF_:InvokeServer("Missão", mob.missão, mob.missãoPart)
            para _, inimigo em pares (Workspace.Enemies:GetChildren()) faça
                se enemy.Name:find(mob.name) e enemy:FindFirstChild("HumanoidRootPart") e enemy.Humanoid.Health > 0 então
                    repita
                        TP(inimigo.HumanoidRootPart.Posição + Vetor3.novo(0,5,0))
                        espere(0,2)
                        VirtualInputManager:SendKeyEvent(verdadeiro, "Z", falso, jogo)
                        espere(0,2)
                        VirtualInputManager:SendKeyEvent(falso, "Z", falso, jogo)
                    até que inimigo.Humanóide.Saúde <= 0 ou não inimigo.Pai
                fim
            fim
        fim)
    fim
fim)

-- ======= PVP AUTOMÁTICO =======
função GetClosestEnemy()
    local mais próximo = nulo
    local mais curto = math.huge
    para _, plr em pares(Players:GetPlayers()) faça
        se plr ~= LocalPlayer e plr.Team ~= LocalPlayer.Team e plr.Character e plr.Character:FindFirstChild("HumanoidRootPart") e plr.Character.Humanoid.Health > 0 então
            dist local = (plr.Character.HumanoidRootPart.Position - HumanoidRootPart.Position).Magnitude
            se dist < mais curto então
                mais curto = dist
                mais próximo = plr
            fim
        fim
    fim
    retornar mais próximo
fim

spawn(função()
    enquanto AutoPvPEnabled e wait(2) fazem
        pcall(função()
            inimigo local = GetClosestEnemy()
            se inimigo então
                inimigo localHRP = inimigo.Personagem:EsperaPorCriança("ParteRaizHumanoide")
                repita
                    HumanoidRootPart.CFrame = enemyHRP.CFrame * CFrame.new(0, 0, 2)
                    para _, digite pares({"Z", "X", "C"}) faça
                        VirtualInputManager:SendKeyEvent(verdadeiro, chave, falso, jogo)
                        espere(0,1)
                        VirtualInputManager:SendKeyEvent(falso, chave, falso, jogo)
                    fim
                    espere(0,5)
                até que não seja inimigo.Personagem ou inimigo.Personagem.Humanoide.Saúde <= 0
            fim
        fim)
    fim
fim)

-- ======= HACK DE VELOCIDADE =======
spawn(função()
    enquanto SpeedHackEnabled e wait() fazem
        pcall(função()
            Personagem:FindFirstChildOfClass("Humanoide").WalkSpeed = 120
        fim)
    fim
fim)

-- ======= ESP AUTOMÁTICO =======
spawn(função()
    se não AutoESPEnabled então retorne end
    enquanto espera(3) faça
        para _, obj em pares (Workspace:GetDescendants()) faça
            se obj:IsA("Parte") e (obj.Name == "Peito" ou obj.Name == "Fruta" ou obj.Name == "ParteRaizHumanoide") então
                pcall(função()
                    se não for obj:FindFirstChild("ESP") então
                        esp local = Instance.new("BillboardGui", obj)
                        esp.Nome = "ESP"
                        esp.Size = UDim2.new(0,100,0,40)
                        esp.AlwaysOnTop = verdadeiro
                        rótulo local = Instance.new("TextLabel", esp)
                        rótulo.Tamanho = UDim2.novo(1,0,1,0)
                        rótulo.Texto = obj.Nome
                        label.TextColor3 = Color3.fromRGB(255,255,0)
                        rótulo.BackgroundTransparency = 1
                    fim
                fim)
            fim
        fim
    fim
fim)

-- ======= MACRO (SOMENTE PC) =======
se não for jogo:GetService("UserInputService").TouchEnabled então
    spawn(função()
        enquanto MacroPCOnly e wait(1) fazem
            -- Exemplo simples de combinação
            VirtualInputManager:SendKeyEvent(verdadeiro, "Z", falso, jogo)
            espere(0,1)
            VirtualInputManager:SendKeyEvent(falso, "Z", falso, jogo)
            espere(0,2)
            VirtualInputManager:SendKeyEvent(verdadeiro, "X", falso, jogo)
            espere(0,1)
            VirtualInputManager:SendKeyEvent(falso, "X", falso, jogo)
        fim
    fim)
fim

-- ======= AUTO QUEST (Missões) =======
spawn(função()
    enquanto AutoQuestEnabled e wait(1) fazem
        pcall(função()
            busca local = ReplicatedStorage.Remotes.CommF_:InvokeServer("StartQuest", "BanditQuest1")
            se a missão então
                ReplicatedStorage.Remotes.CommF_:InvokeServer("Missão", "BanditQuest1", 1)
                esperar(10)
            fim
        fim)
    fim
fim)

-- ======= AUTO STATS (Distribuição de pontos) =======
spawn(função()
    enquanto AutoStatsEnabled e wait(1) fazem
        pcall(função()
            estatísticas locais = LocalPlayer:FindFirstChild("Estatísticas")
            se estatísticas então
                estatísticas.Força.Valor = estatísticas.Força.Valor + 1
                estatísticas.Destreza.Valor = estatísticas.Destreza.Valor + 1
                estatísticas.Defesa.Valor = estatísticas.Defesa.Valor + 1
                stats.Intelligence.Value = stats.Intelligence.Value + 1
            fim
        fim)
    fim
fim)

-- ======= COLETA DE ITENS RAROS =======
spawn(função()
    enquanto ItemCollectorEnabled e wait(5) fazem
        pcall(função()
            para _, obj em pares (Workspace:GetDescendants()) faça
                se obj.Name == "DevilFruit" ou obj.Name == "RareChest" então
                    se (obj.Position - Character.HumanoidRootPart.Position).Magnitude <= 50 então
                        TP(obj.Posição)
                        espere(0,2)
                        obj:Destruir()
                    fim
                fim
            fim
        fim)
    fim
fim)

-- ======= RAÇAS V3/V4 =======
spawn(função()
    enquanto espera(1) faça
        pcall(função()
            -- Verifica Raça V3 ou V4 e realiza ações específicas
            se LocalPlayer.Data.Race.Value == "V3" então
                -- Ativa habilidades V3
                LocalPlayer.Character.HumanoidRootPart.CFrame = CFrame.new(5000, 5000, 5000)
            fim
        fim)
    fim
fim)

-- Fim do script
