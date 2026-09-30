
--[[
    ROUBE UM OVO - OPEN SOURCE
    Roblox / Luau

    Funcionalidades:
    - Pegar ovo
    - Levar ovo para a base
    - Recompensa em Coins
    - Dados dos animais
    - Ranking dos melhores animais
]]

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

local EggsFolder = workspace:WaitForChild("Eggs")
local BasesFolder = workspace:WaitForChild("Bases")

--------------------------------------------------
-- 🐾 ANIMAIS
--------------------------------------------------

local Animals = {
	{
		Name = "Dragão",
		Rarity = "Lendário",
		Value = 10000
	},

	{
		Name = "Fênix",
		Rarity = "Lendário",
		Value = 8000
	},

	{
		Name = "Tigre Dourado",
		Rarity = "Épico",
		Value = 5000
	},

	{
		Name = "Lobo",
		Rarity = "Raro",
		Value = 2500
	},

	{
		Name = "Raposa",
		Rarity = "Incomum",
		Value = 1000
	}
}

-- Ordena do maior valor para o menor
table.sort(Animals, function(a, b)
	return a.Value > b.Value
end)

--------------------------------------------------
-- 🏆 MOSTRAR MELHORES ANIMAIS
--------------------------------------------------

print("========== MELHORES ANIMAIS ==========")

for i, animal in ipairs(Animals) do
	print(
		"#" .. i,
		animal.Name,
		"| Raridade:",
		animal.Rarity,
		"| Valor:",
		animal.Value
	)
end

print("=======================================")

--------------------------------------------------
-- 🥚 SISTEMA DE OVOS
--------------------------------------------------

local carryingEgg = {}

local function getBase(player)
	for _, base in ipairs(BasesFolder:GetChildren()) do
		if base:GetAttribute("Owner") == player.UserId then
			return base
		end
	end

	return nil
end

--------------------------------------------------
-- PEGAR O OVO
--------------------------------------------------

local function pickupEgg(player, egg)

	if carryingEgg[player] then
		return
	end

	if egg:GetAttribute("Taken") then
		return
	end

	local character = player.Character
	local root = character and character:FindFirstChild("HumanoidRootPart")

	if not root then
		return
	end

	egg:SetAttribute("Taken", true)

	carryingEgg[player] = egg

	egg.Anchored = false
	egg.CanCollide = false

	egg.CFrame = root.CFrame * CFrame.new(0, 2, -2)

	local weld = Instance.new("WeldConstraint")
	weld.Name = "EggCarryWeld"
	weld.Part0 = egg
	weld.Part1 = root
	weld.Parent = egg
end

--------------------------------------------------
-- ENTREGAR OVO NA BASE
--------------------------------------------------

local function deliverEgg(player)

	local egg = carryingEgg[player]

	if not egg then
		return
	end

	local base = getBase(player)

	if not base then
		return
	end

	local character = player.Character
	local root = character and character:FindFirstChild("HumanoidRootPart")

	local deliveryPoint = base:FindFirstChild("DeliveryPoint")

	if not root or not deliveryPoint then
		return
	end

	local distance =
		(root.Position - deliveryPoint.Position).Magnitude

	if distance > 12 then
		return
	end

	--------------------------------------------------
	-- REMOVE OVO
	--------------------------------------------------

	carryingEgg[player] = nil

	egg:Destroy()

	--------------------------------------------------
	-- RECOMPENSA
	--------------------------------------------------

	local leaderstats = player:FindFirstChild("leaderstats")

	if leaderstats then

		local coins = leaderstats:FindFirstChild("Coins")

		if coins then
			coins.Value += 100
		end

	end

	print(player.Name .. " entregou um ovo e ganhou 100 Coins!")
end

--------------------------------------------------
-- CONFIGURAR OVOS
--------------------------------------------------

for _, egg in ipairs(EggsFolder:GetChildren()) do

	if egg:IsA("BasePart") then

		egg:SetAttribute("Taken", false)

		local prompt =
			egg:FindFirstChildOfClass("ProximityPrompt")

		if prompt then

			prompt.Triggered:Connect(function(player)

				pickupEgg(player, egg)

			end)

		end
	end
end

--------------------------------------------------
-- JOGADORES
--------------------------------------------------

Players.PlayerAdded:Connect(function(player)

	-- Leaderstats
	local leaderstats = Instance.new("Folder")
	leaderstats.Name = "leaderstats"
	leaderstats.Parent = player

	local coins = Instance.new("IntValue")
	coins.Name = "Coins"
	coins.Value = 0
	coins.Parent = leaderstats

	--------------------------------------------------
	-- PERSONAGEM
	--------------------------------------------------

	player.CharacterAdded:Connect(function(character)

		character:WaitForChild("HumanoidRootPart")

		task.spawn(function()

			while character.Parent do

				task.wait(0.25)

				if carryingEgg[player] then
					deliverEgg(player)
				end

			end

		end)

	end)

end)

--------------------------------------------------
-- SAÍDA DO JOGADOR
--------------------------------------------------

Players.PlayerRemoving:Connect(function(player)

	local egg = carryingEgg[player]

	if egg and egg.Parent then

		egg:SetAttribute("Taken", false)
		egg.Anchored = true
		egg.CanCollide = true

		local weld =
			egg:FindFirstChild("EggCarryWeld")

		if weld then
			weld:Destroy()
		end

	end

	carryingEgg[player] = nil

end)
Estrutura necessária no Roblox Studio
Workspace
├── Eggs
│   ├── Egg1
│   ├── Egg2
│   └── Egg3
│
└── Bases
    ├── Base1
    │   └── DeliveryPoint
    ├── Base2
    │   └── DeliveryPoint
    └── Base3
        └── DeliveryPoint
Cada Egg precisa ter um ProximityPrompt. Para cada base, o atributo Owner deve conter o UserId do jogador dono da base.
Se quiser, também posso transformar isso em um sistema mais completo, com ovos que dão animais aleatórios, raridades, dinheiro, inventário e uma GUI mostrando os 10 melhores animais.
