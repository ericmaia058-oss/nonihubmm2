-- ====================================================================
--            NONI HUB - MURDER MYSTERY 2 (FULL EDITION)
--   Auto Coin TWEEN estavel + Gun anti-parede + Kill Aura TP + resto
-- ====================================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local StarterGui = game:GetService("StarterGui")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local Camera = Workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- ==================== CONFIGURACOES ====================
local Settings = {
    MM2 = {
        ESPEnabled = true,
        GunESPEnabled = true,
        KillAuraEnabled = false,
        AutoKillMurderer = false,
        AutoGrabGun = false,
        AutoCoinCollector = false,
        FleeMode = false,
        AimbotCursor = true,
        AimbotSmooth = 0,
        CoinFallProtection = true,
        AuraRange = 5000,
        CoinRange = 500,
        AttackDelay = 0.1,

        FleeDistance = 40,
        FleeSearchRange = 120,
        FleeTargetMinDistance = 60,
        FleeCooldown = 2.5,

        TPBehindDistance = 4,
        ShotCooldown = 0.25,

        GrabGunRange = 300,

        CoinScanInterval = 1.5
    },
    Colors = {
        Murderer = Color3.fromRGB(255, 40, 40),
        Sheriff  = Color3.fromRGB(40, 120, 255),
        Innocent = Color3.fromRGB(40, 255, 100)
    }
}

local function notify(title, text, duration)
    pcall(function()
        StarterGui:SetCore("SendNotification", {
            Title = title,
            Text = text,
            Duration = duration or 4
        })
    end)
end

-- ==================== NOCLIP ====================
local NOCLIP_ENABLED = false
local noclipConnection = nil

local function setNoclip(enabled)
    NOCLIP_ENABLED = enabled
    local myChar = LocalPlayer.Character
    if myChar then
        for _, part in ipairs(myChar:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = not enabled
            end
        end
    end

    if noclipConnection then
        noclipConnection:Disconnect()
        noclipConnection = nil
    end

    if enabled then
        noclipConnection = RunService.Stepped:Connect(function()
            if not NOCLIP_ENABLED then return end
            local char = LocalPlayer.Character
            if not char then return end
            for _, part in ipairs(char:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.CanCollide = false
                end
            end
        end)
    end
end

LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(0.5)
    if NOCLIP_ENABLED then
        setNoclip(true)
    end
end)

-- ==================== ESTILOS VISUAIS (cor, material e efeitos - so voce ve) ====================
-- Usado pelas skins "fake" de faca/arma e pelos efeitos de corpo (perna de zumbi, braco de gelo...).
local chromaParts = setmetatable({}, { __mode = "k" })   -- pecas com cor arco-iris
local styleBackup = setmetatable({}, { __mode = "k" })   -- dono -> valores originais

RunService.Heartbeat:Connect(function()
    local c = Color3.fromHSV((tick() * 0.35) % 1, 0.85, 1)
    for part in pairs(chromaParts) do
        if part.Parent then
            part.Color = c
        else
            chromaParts[part] = nil
        end
    end
end)

local Styles = {
    Nebula    = { color = Color3.fromRGB(125, 70, 255),  material = Enum.Material.Neon,        fx = { "stars", "light" } },
    Chroma    = { color = Color3.fromRGB(255, 0, 0),     material = Enum.Material.Neon,        chroma = true, fx = { "sparkles" } },
    Gelo      = { color = Color3.fromRGB(150, 220, 255), material = Enum.Material.Ice,         reflectance = 0.15, fx = { "sparkles", "light" } },
    Lava      = { color = Color3.fromRGB(255, 90, 20),   material = Enum.Material.Neon,        fx = { "fire" }, fxColor2 = Color3.fromRGB(255, 200, 0) },
    Ouro      = { color = Color3.fromRGB(255, 200, 60),  material = Enum.Material.Metal,       reflectance = 0.35, fx = { "sparkles" } },
    Sombra    = { color = Color3.fromRGB(18, 18, 24),    material = Enum.Material.SmoothPlastic, fx = { "smoke" }, fxColor = Color3.fromRGB(60, 0, 90) },
    Sangue    = { color = Color3.fromRGB(160, 10, 20),   material = Enum.Material.Metal,       reflectance = 0.2, fx = { "light" } },
    Toxico    = { color = Color3.fromRGB(90, 255, 60),   material = Enum.Material.Neon,        fx = { "sparkles", "light" } },
    Diamante  = { color = Color3.fromRGB(200, 245, 255), material = Enum.Material.Glass,       reflectance = 0.3, fx = { "sparkles" } },
    Galaxia   = { color = Color3.fromRGB(70, 40, 160),   material = Enum.Material.ForceField,  fx = { "stars" }, fxColor = Color3.fromRGB(180, 150, 255) },
    RosaNeon  = { color = Color3.fromRGB(255, 80, 200),  material = Enum.Material.Neon,        fx = { "light" } },
    Icewing   = { color = Color3.fromRGB(140, 210, 255), material = Enum.Material.Ice,         reflectance = 0.2, fx = { "sparkles", "stars", "light" }, fxColor = Color3.fromRGB(190, 235, 255) },
    Batwing   = { color = Color3.fromRGB(35, 15, 50),    material = Enum.Material.Metal,       reflectance = 0.15, fx = { "smoke", "light" }, fxColor = Color3.fromRGB(150, 60, 220) },
    Ancient   = { color = Color3.fromRGB(205, 150, 55),  material = Enum.Material.Metal,       reflectance = 0.35, fx = { "stars", "light" }, fxColor = Color3.fromRGB(255, 170, 60) },
    FX        = { color = Color3.fromRGB(205, 35, 45),   material = Enum.Material.Metal,       reflectance = 0.25, fx = { "sparkles", "light" }, fxColor = Color3.fromRGB(255, 240, 200) },
    Zumbi     = { color = Color3.fromRGB(99, 140, 74),   material = Enum.Material.Slate },
    Robo      = { color = Color3.fromRGB(150, 156, 168), material = Enum.Material.Metal,       reflectance = 0.5 },
    Esqueleto = { color = Color3.fromRGB(235, 228, 210), material = Enum.Material.SmoothPlastic },
    Fantasma  = { color = Color3.fromRGB(200, 230, 255), material = Enum.Material.ForceField,  transparency = 0.5, fx = { "sparkles" } },
}

-- Skins de arma disponiveis no menu (nome, estilo)
local WeaponStyleList = {
    { name = "Nebula",    style = Styles.Nebula },
    { name = "Chroma",    style = Styles.Chroma },
    { name = "Gelo",      style = Styles.Gelo },
    { name = "Lava",      style = Styles.Lava },
    { name = "Ouro",      style = Styles.Ouro },
    { name = "Sombra",    style = Styles.Sombra },
    { name = "Sangue",    style = Styles.Sangue },
    { name = "Toxico",    style = Styles.Toxico },
    { name = "Diamante",  style = Styles.Diamante },
    { name = "Galaxia",   style = Styles.Galaxia },
    { name = "Rosa Neon", style = Styles.RosaNeon },
    { name = "Icewing (estilo)",  style = Styles.Icewing },
    { name = "Batwing (estilo)",  style = Styles.Batwing },
    { name = "Ancient (estilo)",  style = Styles.Ancient },
    { name = "FX (estilo)",       style = Styles.FX },
}

local function addFx(main, style, registry)
    if not main or not style.fx then return end
    local c = style.fxColor or style.color
    for _, kind in ipairs(style.fx) do
        local fx = nil
        if kind == "sparkles" then
            fx = Instance.new("Sparkles")
            fx.SparkleColor = c
        elseif kind == "fire" then
            fx = Instance.new("Fire")
            fx.Color = c
            fx.SecondaryColor = style.fxColor2 or c
            fx.Size = 2
            fx.Heat = 4
        elseif kind == "smoke" then
            fx = Instance.new("Smoke")
            fx.Color = c
            fx.Size = 2
            fx.Opacity = 0.35
            fx.RiseVelocity = 1
        elseif kind == "light" then
            fx = Instance.new("PointLight")
            fx.Color = c
            fx.Range = 8
            fx.Brightness = 1.5
        elseif kind == "stars" then
            fx = Instance.new("ParticleEmitter")
            fx.Color = ColorSequence.new(c)
            fx.LightEmission = 1
            fx.Rate = 12
            fx.Lifetime = NumberRange.new(0.6, 1.2)
            fx.Speed = NumberRange.new(0.5, 2)
            fx.SpreadAngle = Vector2.new(180, 180)
            fx.Size = NumberSequence.new({
                NumberSequenceKeypoint.new(0, 0.25),
                NumberSequenceKeypoint.new(1, 0),
            })
            fx.Transparency = NumberSequence.new({
                NumberSequenceKeypoint.new(0, 0),
                NumberSequenceKeypoint.new(1, 1),
            })
        end
        if fx then
            fx.Name = "NoniFx"
            fx.Parent = main
            registry[#registry + 1] = fx
        end
    end
end

local function restoreStyleOwner(owner)
    local bk = styleBackup[owner]
    if not bk then return end
    for _, fx in ipairs(bk.fx) do
        if fx and fx.Parent then fx:Destroy() end
    end
    for part, props in pairs(bk.parts) do
        chromaParts[part] = nil
        if part.Parent then
            part.Color = props.Color
            part.Material = props.Material
            part.Reflectance = props.Reflectance
            part.Transparency = props.Transparency
            if props.TextureID ~= nil then
                pcall(function() part.TextureID = props.TextureID end)
            end
        end
    end
    for mesh, tex in pairs(bk.meshes) do
        if mesh.Parent then mesh.TextureId = tex end
    end
    for _, r in ipairs(bk.sas) do
        if r.inst and r.parent and r.parent.Parent then
            r.inst.Parent = r.parent
        end
    end
    styleBackup[owner] = nil
end

local function applyStyleToParts(owner, parts, style, fxTarget)
    restoreStyleOwner(owner)
    local bk = { parts = {}, meshes = {}, sas = {}, fx = {} }
    styleBackup[owner] = bk

    for _, part in ipairs(parts) do
        bk.parts[part] = {
            Color = part.Color,
            Material = part.Material,
            Reflectance = part.Reflectance,
            Transparency = part.Transparency,
            TextureID = part:IsA("MeshPart") and part.TextureID or nil,
        }
        part.Color = style.color
        part.Material = style.material
        part.Reflectance = style.reflectance or 0
        if style.transparency and part.Transparency < 1 then
            part.Transparency = style.transparency
        end
        if part:IsA("MeshPart") and part.TextureID ~= "" then
            pcall(function() part.TextureID = "" end)
        end
        if style.chroma then chromaParts[part] = true end

        for _, d in ipairs(part:GetChildren()) do
            if d:IsA("SurfaceAppearance") then
                bk.sas[#bk.sas + 1] = { inst = d, parent = part }
                d.Parent = nil
            elseif d:IsA("SpecialMesh") then
                bk.meshes[d] = d.TextureId
                d.TextureId = ""
            end
        end
    end
    addFx(fxTarget or parts[1], style, bk.fx)
end

-- ==================== AVATAR VISUAL (so voce ve) ====================
-- Todas as alteracoes aqui sao LOCAIS: os outros jogadores NAO veem.
local AvatarPresets = {
    Headless = { slot = "Head",     id = 134082579 },
    Korblox  = { slot = "RightLeg", id = 139607718 },
}

-- Presets de partes do corpo por ID do catalogo. Adicione os seus aqui, ex:
--   { name = "Perna de zumbi", id = 123456789 },
-- (o ID fica no link do item: roblox.com/catalog/ID)
local BodyPresets = {
    { name = "Perna de zumbi (direita)",    id = 37754710 },
    { name = "Perna de zumbi (esquerda)",   id = 3064930513 },
    { name = "Perna de esqueleto (esq.)",   id = 36781481 },
}

local avatarState = { Headless = false, Korblox = false }
local avatarOriginal = {}   -- guarda o item original de cada slot (fallback HumanoidDescription)
local activeLimbs = {}      -- chave -> id das partes do corpo aplicadas

local function applyHeadlessVisual()
    local char = LocalPlayer.Character
    local head = char and char:FindFirstChild("Head")
    if not head then return end
    local t = avatarState.Headless and 1 or 0
    head.Transparency = t
    for _, d in ipairs(head:GetChildren()) do
        if d:IsA("Decal") then d.Transparency = t end
    end
end

-- Fallback: troca o slot via HumanoidDescription (alguns executores bloqueiam)
local function applyPreset(name)
    task.spawn(function()
        local preset = AvatarPresets[name]
        if not preset then return end
        local ok = pcall(function()
            local char = LocalPlayer.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if not hum then return end
            local desc = hum:GetAppliedDescription()
            if avatarOriginal[preset.slot] == nil then
                avatarOriginal[preset.slot] = desc[preset.slot]
            end
            desc[preset.slot] = avatarState[name] and preset.id or avatarOriginal[preset.slot]
            hum:ApplyDescription(desc)
        end)
        if not ok then
            notify("Noni Hub", name .. " nao funcionou neste executor/jogo", 3)
        end
        applyHeadlessVisual()
    end)
end

-- ---------- Partes do corpo por ID (overlay) ----------
-- Carrega o asset da parte (perna, braco, cabeca...), esconde a sua peca e cola a nova por cima.
local LIMB_NAMES = {}
for _, n in ipairs({
    "Head", "UpperTorso", "LowerTorso",
    "LeftUpperArm", "LeftLowerArm", "LeftHand",
    "RightUpperArm", "RightLowerArm", "RightHand",
    "LeftUpperLeg", "LeftLowerLeg", "LeftFoot",
    "RightUpperLeg", "RightLowerLeg", "RightFoot",
    "Torso", "Left Arm", "Right Arm", "Left Leg", "Right Leg",
}) do
    LIMB_NAMES[n] = true
end

local limbSourceCache = {}

local function loadLimbSource(id)
    if limbSourceCache[id] then return limbSourceCache[id] end
    local ok, objs = pcall(function()
        return game:GetObjects("rbxassetid://" .. tostring(id))
    end)
    if not ok or not objs then return nil end

    local function prio(part)
        local p = part.Parent
        while p do
            if p.Name == "R15ArtistIntent" then return 3 end
            if p.Name == "R15Fixed" then return 2 end
            p = p.Parent
        end
        return 1
    end

    local best = {}
    for _, root in ipairs(objs) do
        local list = root:GetDescendants()
        table.insert(list, root)
        for _, d in ipairs(list) do
            if d:IsA("BasePart") and LIMB_NAMES[d.Name] then
                local pr = prio(d)
                if not best[d.Name] or pr > best[d.Name].prio then
                    best[d.Name] = { part = d, prio = pr }
                end
            end
        end
    end
    limbSourceCache[id] = best
    return best
end

local function removeLimbs(key)
    local char = LocalPlayer.Character
    if not char then return end
    local cname = "NoniLimb_" .. key
    local attr = "NoniLimbKey_" .. key

    for _, d in ipairs(char:GetDescendants()) do
        if d.Name == cname then d:Destroy() end
    end
    for _, d in ipairs(char:GetDescendants()) do
        if d:GetAttribute(attr) then
            d:SetAttribute(attr, nil)
            local cnt = (d:GetAttribute("NoniLimbCount") or 1) - 1
            if cnt <= 0 then
                local base = d:GetAttribute("NoniLimbBase")
                if base ~= nil then d.Transparency = base end
                d:SetAttribute("NoniLimbBase", nil)
                d:SetAttribute("NoniLimbCount", nil)
            else
                d:SetAttribute("NoniLimbCount", cnt)
            end
        end
    end
end

local function applyLimbs(key, id)
    removeLimbs(key)
    local char = LocalPlayer.Character
    if not char then return 0 end
    local best = loadLimbSource(id)
    if not best then return 0 end

    local n = 0
    for name, e in pairs(best) do
        local limb = char:FindFirstChild(name)
        if limb and limb:IsA("BasePart") then
            local clone = e.part:Clone()
            for _, d in ipairs(clone:GetDescendants()) do
                if d:IsA("LuaSourceContainer") or d:IsA("JointInstance") or d:IsA("WeldConstraint") then
                    d:Destroy()
                end
            end
            clone.Name = "NoniLimb_" .. key
            clone.Anchored = false
            clone.CanCollide = false
            clone.CanTouch = false
            clone.CanQuery = false
            clone.Massless = true

            pcall(function() clone.RootPriority = -127 end)
            clone.CFrame = limb.CFrame
            local w = Instance.new("Weld")
            w.Part0 = limb
            w.Part1 = clone
            w.Parent = clone
            clone.Parent = limb

            if limb:GetAttribute("NoniLimbBase") == nil then
                limb:SetAttribute("NoniLimbBase", limb.Transparency)
            end
            limb:SetAttribute("NoniLimbCount", (limb:GetAttribute("NoniLimbCount") or 0) + 1)
            limb:SetAttribute("NoniLimbKey_" .. key, true)
            limb.Transparency = 1
            n = n + 1
        end
    end
    return n
end

local function setLimbs(key, id, enabled)
    if enabled then
        activeLimbs[key] = id
        return applyLimbs(key, id)
    else
        activeLimbs[key] = nil
        removeLimbs(key)
        return 0
    end
end

local function applyCustomLimb(idText)
    local id = tostring(idText or ""):gsub("%D", "")
    if id == "" then
        notify("Noni Hub", "Digite um ID valido", 3)
        return
    end
    task.spawn(function()
        local n = setLimbs("custom_" .. id, id, true)
        if n > 0 then
            notify("Noni Hub", "Parte aplicada (" .. n .. " pecas)", 3)
        else
            activeLimbs["custom_" .. id] = nil
            notify("Noni Hub", "Esse ID nao tem partes do corpo utilizaveis", 4)
        end
    end)
end

local function removeCustomLimbs()
    local keys = {}
    for key in pairs(activeLimbs) do
        if string.sub(key, 1, 7) == "custom_" or string.sub(key, 1, 7) == "preset_" then
            keys[#keys + 1] = key
        end
    end
    for _, key in ipairs(keys) do
        setLimbs(key, nil, false)
    end
    notify("Noni Hub", "Partes custom removidas", 2)
end

local function setBodyPreset(preset, enabled)
    task.spawn(function()
        local key = "preset_" .. preset.name
        local n = setLimbs(key, preset.id, enabled)
        if enabled and n == 0 then
            activeLimbs[key] = nil
            notify("Noni Hub", preset.name .. ": nao consegui aplicar", 3)
        end
    end)
end

-- ---------- Efeitos de corpo (perna de zumbi, braco de gelo...) ----------
local function joinLists(...)
    local out = {}
    for _, list in ipairs({ ... }) do
        for _, v in ipairs(list) do out[#out + 1] = v end
    end
    return out
end

local LIMBS = {
    RightArm = { "RightUpperArm", "RightLowerArm", "RightHand", "Right Arm" },
    LeftArm  = { "LeftUpperArm", "LeftLowerArm", "LeftHand", "Left Arm" },
    RightLeg = { "RightUpperLeg", "RightLowerLeg", "RightFoot", "Right Leg" },
    LeftLeg  = { "LeftUpperLeg", "LeftLowerLeg", "LeftFoot", "Left Leg" },
}
LIMBS.Body = joinLists(LIMBS.RightArm, LIMBS.LeftArm, LIMBS.RightLeg, LIMBS.LeftLeg,
    { "UpperTorso", "LowerTorso", "Torso" })

local BodyStyles = {
    { name = "Perna de zumbi",      limbs = LIMBS.RightLeg, style = Styles.Zumbi },
    { name = "Braco de zumbi",      limbs = LIMBS.LeftArm,  style = Styles.Zumbi },
    { name = "Braco de gelo",       limbs = LIMBS.RightArm, style = Styles.Gelo },
    { name = "Perna de gelo",       limbs = LIMBS.LeftLeg,  style = Styles.Gelo },
    { name = "Braco de fogo",       limbs = LIMBS.RightArm, style = Styles.Lava },
    { name = "Perna de fogo",       limbs = LIMBS.LeftLeg,  style = Styles.Lava },
    { name = "Braco de ouro",       limbs = LIMBS.RightArm, style = Styles.Ouro },
    { name = "Perna de ouro",       limbs = LIMBS.LeftLeg,  style = Styles.Ouro },
    { name = "Braco robotico",      limbs = LIMBS.LeftArm,  style = Styles.Robo },
    { name = "Perna robotica",      limbs = LIMBS.RightLeg, style = Styles.Robo },
    { name = "Braco de esqueleto",  limbs = LIMBS.RightArm, style = Styles.Esqueleto },
    { name = "Perna de esqueleto",  limbs = LIMBS.LeftLeg,  style = Styles.Esqueleto },
    { name = "Corpo Chroma",        limbs = LIMBS.Body,     style = Styles.Chroma },
    { name = "Corpo fantasma",      limbs = LIMBS.Body,     style = Styles.Fantasma },
}

local activeLimbStyles = {}

-- Cola uma copia estilizada (cor/material/efeito) por cima de cada peca e esconde a original.
-- Como a copia nao e parte "de verdade" do corpo, a roupa nao cobre a cor.
local function applyLimbStyle(key, names, style)
    removeLimbs(key)
    local char = LocalPlayer.Character
    if not char then return 0 end

    local n = 0
    local fxDone = false
    for _, name in ipairs(names) do
        local limb = char:FindFirstChild(name)
        if limb and limb:IsA("BasePart") then
            local clone = limb:Clone()
            for _, d in ipairs(clone:GetDescendants()) do
                if d:IsA("LuaSourceContainer") or d:IsA("JointInstance") or d:IsA("WeldConstraint")
                    or d:IsA("SurfaceAppearance") or d:IsA("Decal") or d:IsA("Texture")
                    or d:IsA("Attachment") then
                    d:Destroy()
                end
            end
            for attrName in pairs(clone:GetAttributes()) do
                clone:SetAttribute(attrName, nil)
            end

            clone.Name = "NoniLimb_" .. key
            clone.Anchored = false
            clone.CanCollide = false
            clone.CanTouch = false
            clone.CanQuery = false
            clone.Massless = true
            clone.Transparency = style.transparency or 0
            clone.Color = style.color
            clone.Material = style.material
            clone.Reflectance = style.reflectance or 0
            if clone:IsA("MeshPart") then
                pcall(function() clone.TextureID = "" end)
            end
            if style.chroma then chromaParts[clone] = true end

            pcall(function() clone.RootPriority = -127 end)
            clone.CFrame = limb.CFrame
            local w = Instance.new("Weld")
            w.Part0 = limb
            w.Part1 = clone
            w.Parent = clone
            clone.Parent = limb

            if limb:GetAttribute("NoniLimbBase") == nil then
                limb:SetAttribute("NoniLimbBase", limb.Transparency)
            end
            limb:SetAttribute("NoniLimbCount", (limb:GetAttribute("NoniLimbCount") or 0) + 1)
            limb:SetAttribute("NoniLimbKey_" .. key, true)
            limb.Transparency = 1

            if not fxDone then
                addFx(clone, style, {})
                fxDone = true
            end
            n = n + 1
        end
    end
    return n
end

local function setBodyStyle(def, enabled)
    task.spawn(function()
        local key = "bstyle_" .. def.name
        if enabled then
            activeLimbStyles[key] = def
            local n = applyLimbStyle(key, def.limbs, def.style)
            if n == 0 then
                activeLimbStyles[key] = nil
                notify("Noni Hub", def.name .. ": nao achei as partes do corpo", 3)
            end
        else
            activeLimbStyles[key] = nil
            removeLimbs(key)
        end
    end)
end

-- Korblox usa o mesmo sistema; se falhar, cai no HumanoidDescription
local function applyKorbloxVisual(enabled)
    task.spawn(function()
        if not enabled then
            setLimbs("korblox", nil, false)
            if avatarOriginal.RightLeg ~= nil then applyPreset("Korblox") end
            return
        end
        local n = setLimbs("korblox", AvatarPresets.Korblox.id, true)
        if n == 0 then
            notify("Noni Hub", "Korblox por pecas falhou, tentando outro metodo...", 3)
            applyPreset("Korblox")
        end
    end)
end

-- ---------- Avatar completo (copiar de outro usuario) ----------
-- Le a descricao do avatar da pessoa (roupas, cores, cabelo/acessorios, partes do corpo e rosto)
-- e monta tudo no seu personagem. Funciona com qualquer usuario, mesmo fora do servidor.
local avatarBackup = { removed = {}, added = {}, face = nil, colors = nil }

local function clearAvatarBackup()
    avatarBackup = { removed = {}, added = {}, face = nil, colors = nil }
end

local function loadAssetInstance(id)
    local ok, objs = pcall(function()
        return game:GetObjects("rbxassetid://" .. tostring(id))
    end)
    if ok and objs and #objs > 0 then return objs[1] end

    -- plano B: InsertService (alguns executores so aceitam esse)
    local ok2, model = pcall(function()
        return game:GetService("InsertService"):LoadAsset(tonumber(id))
    end)
    if ok2 and model then
        local first = model:GetChildren()[1]
        if first then
            first.Parent = nil
            model:Destroy()
            return first
        end
    end
    return nil
end

local function restoreAvatar()
    -- partes do corpo copiadas
    local keys = {}
    for key in pairs(activeLimbs) do
        if string.sub(key, 1, 7) == "avatar_" then keys[#keys + 1] = key end
    end
    for _, key in ipairs(keys) do
        setLimbs(key, nil, false)
    end

    local char = LocalPlayer.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    if not char or not hum then
        clearAvatarBackup()
        return
    end

    for _, inst in ipairs(avatarBackup.added) do
        if inst and inst.Parent then inst:Destroy() end
    end
    for _, inst in ipairs(avatarBackup.removed) do
        if inst then
            if inst:IsA("Accessory") then
                pcall(function() hum:AddAccessory(inst) end)
            else
                inst.Parent = char
            end
        end
    end

    local bc = char:FindFirstChildOfClass("BodyColors")
    if bc and avatarBackup.colors then
        local c = avatarBackup.colors
        bc.HeadColor3, bc.TorsoColor3 = c[1], c[2]
        bc.LeftArmColor3, bc.RightArmColor3 = c[3], c[4]
        bc.LeftLegColor3, bc.RightLegColor3 = c[5], c[6]
    end

    if avatarBackup.face then
        local head = char:FindFirstChild("Head")
        local decal = head and head:FindFirstChildOfClass("Decal")
        if decal then decal.Texture = avatarBackup.face end
    end
    clearAvatarBackup()
end

local function copyAvatarFromUserId(uid, label, theirs)
    task.spawn(function()
        local char = LocalPlayer.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if not char or not hum then return end

        -- descricao do avatar (pode falhar em alguns executores; se o jogador esta no servidor,
        -- ainda da pra copiar direto do personagem dele)
        local okDesc, desc = pcall(function()
            return Players:GetHumanoidDescriptionFromUserId(uid)
        end)
        if not okDesc then desc = nil end
        if not desc and not theirs then
            notify("Noni Hub", "Nao consegui ler o avatar de " .. tostring(label), 4)
            return
        end

        restoreAvatar()
        notify("Noni Hub", "Copiando avatar de " .. tostring(label) .. "...", 3)

        local rep = { roupas = 0, acessorios = 0, partes = 0, cores = false, rosto = false }

        local ok, err = pcall(function()
            -- 1) roupas: primeiro do personagem dele, depois pela descricao
            local clothes = {
                { class = "Shirt",        id = desc and desc.Shirt },
                { class = "Pants",        id = desc and desc.Pants },
                { class = "ShirtGraphic", id = desc and desc.GraphicTShirt },
            }
            for _, c in ipairs(clothes) do
                local newInst = nil
                if theirs then
                    local src = theirs:FindFirstChildOfClass(c.class)
                    if src then newInst = src:Clone() end
                end
                if not newInst and c.id and c.id ~= 0 then
                    local inst = loadAssetInstance(c.id)
                    if inst and inst:IsA(c.class) then newInst = inst end
                end
                if newInst then
                    local old = char:FindFirstChildOfClass(c.class)
                    if old then
                        table.insert(avatarBackup.removed, old)
                        old.Parent = nil
                    end
                    newInst.Parent = char
                    table.insert(avatarBackup.added, newInst)
                    rep.roupas = rep.roupas + 1
                end
            end

            -- 2) cores do corpo
            local tbc = theirs and theirs:FindFirstChildOfClass("BodyColors")
            local function pick(bcProp, descProp)
                if tbc then return tbc[bcProp] end
                if desc then return desc[descProp] end
                return nil
            end
            local hc = pick("HeadColor3", "HeadColor")
            if hc then
                local bc = char:FindFirstChildOfClass("BodyColors")
                if not bc then
                    bc = Instance.new("BodyColors")
                    bc.Parent = char
                    table.insert(avatarBackup.added, bc)
                elseif not avatarBackup.colors then
                    avatarBackup.colors = {
                        bc.HeadColor3, bc.TorsoColor3, bc.LeftArmColor3,
                        bc.RightArmColor3, bc.LeftLegColor3, bc.RightLegColor3,
                    }
                end
                bc.HeadColor3 = hc
                bc.TorsoColor3 = pick("TorsoColor3", "TorsoColor")
                bc.LeftArmColor3 = pick("LeftArmColor3", "LeftArmColor")
                bc.RightArmColor3 = pick("RightArmColor3", "RightArmColor")
                bc.LeftLegColor3 = pick("LeftLegColor3", "LeftLegColor")
                bc.RightLegColor3 = pick("RightLegColor3", "RightLegColor")
                rep.cores = true
            end

            -- 3) cabelo / acessorios: os do personagem dele + os da descricao
            local newAccs = {}
            local have = {}
            if theirs then
                for _, a in ipairs(theirs:GetChildren()) do
                    if a:IsA("Accessory") then
                        local ok2, cl = pcall(function() return a:Clone() end)
                        if ok2 and cl then
                            newAccs[#newAccs + 1] = cl
                            have[cl.Name] = true
                        end
                    end
                end
            end
            if desc then
                local okA, accs = pcall(function() return desc:GetAccessories(true) end)
                if okA and accs then
                    for _, a in ipairs(accs) do
                        if not a.IsLayered then
                            local inst = loadAssetInstance(a.AssetId)
                            local acc = nil
                            if inst then
                                acc = inst:IsA("Accessory") and inst or inst:FindFirstChildWhichIsA("Accessory", true)
                            end
                            if acc and not have[acc.Name] then
                                have[acc.Name] = true
                                newAccs[#newAccs + 1] = acc
                            end
                        end
                    end
                end
            end
            if #newAccs > 0 then
                for _, c in ipairs(char:GetChildren()) do
                    if c:IsA("Accessory") then
                        table.insert(avatarBackup.removed, c)
                        c.Parent = nil
                    end
                end
                for _, acc in ipairs(newAccs) do
                    pcall(function() hum:AddAccessory(acc) end)
                    table.insert(avatarBackup.added, acc)
                    rep.acessorios = rep.acessorios + 1
                end
            end

            -- 4) partes do corpo customizadas (cabeca, torso, bracos, pernas)
            if desc then
                local bodyParts = {
                    Head = desc.Head, Torso = desc.Torso,
                    LeftArm = desc.LeftArm, RightArm = desc.RightArm,
                    LeftLeg = desc.LeftLeg, RightLeg = desc.RightLeg,
                }
                for name, id in pairs(bodyParts) do
                    if id and id ~= 0 then
                        local key = "avatar_" .. name
                        local n = setLimbs(key, id, true)
                        if n == 0 then
                            activeLimbs[key] = nil
                        else
                            rep.partes = rep.partes + 1
                        end
                    end
                end
            end

            -- 5) rosto
            local head = char:FindFirstChild("Head")
            if head then
                local faceTex = nil
                local theirHead = theirs and theirs:FindFirstChild("Head")
                local theirDecal = theirHead and theirHead:FindFirstChildOfClass("Decal")
                if theirDecal then
                    faceTex = theirDecal.Texture
                elseif desc and desc.Face and desc.Face ~= 0 then
                    local faceInst = loadAssetInstance(desc.Face)
                    if faceInst and faceInst:IsA("Decal") then faceTex = faceInst.Texture end
                end
                if faceTex and faceTex ~= "" then
                    local target = head:FindFirstChild("NoniLimb_avatar_Head") or head
                    local d = target:FindFirstChildOfClass("Decal")
                    if d then
                        if target == head and avatarBackup.face == nil then
                            avatarBackup.face = d.Texture
                        end
                        d.Texture = faceTex
                    else
                        local nd = Instance.new("Decal")
                        nd.Face = Enum.NormalId.Front
                        nd.Texture = faceTex
                        nd.Parent = target
                        table.insert(avatarBackup.added, nd)
                    end
                    rep.rosto = true
                end
            end
        end)

        local resumo = string.format("roupas %d | acessorios %d | partes %d | cores %s | rosto %s",
            rep.roupas, rep.acessorios, rep.partes, tostring(rep.cores), tostring(rep.rosto))
        print("[NoniHub] Copiar avatar (" .. tostring(label) .. "): " .. resumo)
        if not ok then
            warn("[NoniHub] Copiar avatar erro: " .. tostring(err))
            notify("Noni Hub", "Erro ao copiar (veja F9)", 4)
        else
            notify("Noni Hub", "Copiado: " .. rep.roupas .. " roupas, " .. rep.acessorios
                .. " acessorios, " .. rep.partes .. " partes", 4)
        end
    end)
end

local function copyAvatarByName(text)
    text = tostring(text or ""):match("^%s*(.-)%s*$")
    if text == "" then
        notify("Noni Hub", "Digite um nome de usuario", 3)
        return
    end
    local q = string.lower(text)

    -- jogador que esta no servidor
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer then
            local n1 = string.lower(plr.Name)
            local n2 = string.lower(plr.DisplayName)
            if string.find(n1, q, 1, true) or string.find(n2, q, 1, true) then
                copyAvatarFromUserId(plr.UserId, plr.DisplayName, plr.Character)
                return
            end
        end
    end

    -- qualquer usuario do Roblox (nome exato ou UserId)
    task.spawn(function()
        local ok, uid = pcall(function()
            return tonumber(text) or Players:GetUserIdFromNameAsync(text)
        end)
        if ok and uid then
            copyAvatarFromUserId(uid, text)
        else
            notify("Noni Hub", "Usuario nao encontrado: " .. text, 4)
        end
    end)
end

local function setAvatar(name, enabled)
    avatarState[name] = enabled
    if name == "Headless" then
        applyHeadlessVisual()
    elseif name == "Korblox" then
        applyKorbloxVisual(enabled)
    else
        applyPreset(name)
    end
end

-- mantem o headless aplicado (o jogo pode restaurar a cabeca)
RunService.RenderStepped:Connect(function()
    if avatarState.Headless then
        applyHeadlessVisual()
    end
end)

-- reaplica ao renascer
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(1)
    avatarOriginal = {}
    clearAvatarBackup()
    local drop = {}
    for key in pairs(activeLimbs) do
        if string.sub(key, 1, 7) == "avatar_" then drop[#drop + 1] = key end
    end
    for _, key in ipairs(drop) do activeLimbs[key] = nil end
    for name, on in pairs(avatarState) do
        if on then setAvatar(name, true) end
    end
    for key, id in pairs(activeLimbs) do
        if key ~= "korblox" then
            task.spawn(applyLimbs, key, id)
        end
    end
    for key, def in pairs(activeLimbStyles) do
        task.spawn(applyLimbStyle, key, def.limbs, def.style)
    end
end)

-- ==================== SKIN CHANGER (visual, so voce ve) ====================
-- Troca so a aparencia da SUA faca/arma. Nao da item real e ninguem mais ve.
-- Ele procura modelos de armas em ReplicatedStorage / StarterPack / ReplicatedFirst.
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local SKIN_TAG = "NoniSkin"
local SkinChanger = { Knife = nil, Gun = nil }      -- nome da skin escolhida por slot
local skinVersion = { Knife = 0, Gun = 0 }          -- muda a cada pedido, forca reaplicar
-- ajuste fino de rotacao (graus), se a skin aparecer torta na mao
local SKIN_ROT = {
    Knife = Vector3.new(0, 0, 0),
    Gun   = Vector3.new(0, 0, 0),
}
-- o mesmo, mas para a arma guardada nas costas (KnifeDisplay / GunDisplay)
local SKIN_ROT_BACK = {
    Knife = Vector3.new(0, 0, 0),
    Gun   = Vector3.new(0, 0, 0),
}
local skinIndex = nil

local function scanRootInto(index, root)
    return pcall(function()
        for _, obj in ipairs(root:GetDescendants()) do
            local parent = obj.Parent
            local parentIsContainer = parent and (parent:IsA("Tool") or parent:IsA("Model") or parent:IsA("Accessory"))
            if obj:IsA("Tool") or obj:IsA("Model") or obj:IsA("Accessory") then
                if obj:FindFirstChildWhichIsA("BasePart", true) then
                    index[#index + 1] = { name = obj.Name, obj = obj }
                end
            elseif obj:IsA("Folder") then
                if obj:FindFirstChildWhichIsA("BasePart") then
                    index[#index + 1] = { name = obj.Name, obj = obj }
                end
            elseif obj:IsA("MeshPart") and obj.Name ~= "Handle" and not parentIsContainer then
                index[#index + 1] = { name = obj.Name, obj = obj }
            end
        end
    end)
end

local function buildSkinIndex()
    local index = {}
    scanRootInto(index, ReplicatedStorage)
    scanRootInto(index, game:GetService("StarterPack"))
    scanRootInto(index, game:GetService("ReplicatedFirst"))

    -- se nao achou nada, procura tambem em Lighting e em instancias "nil" (se o executor permitir)
    if #index == 0 then
        scanRootInto(index, game:GetService("Lighting"))
        if getnilinstances then
            local ok, nils = pcall(getnilinstances)
            if ok and nils then
                for _, inst in ipairs(nils) do
                    pcall(function()
                        if (inst:IsA("Tool") or inst:IsA("Model") or inst:IsA("Accessory"))
                            and inst:FindFirstChildWhichIsA("BasePart", true) then
                            index[#index + 1] = { name = inst.Name, obj = inst }
                        end
                        scanRootInto(index, inst)
                    end)
                end
            end
        end
    end
    return index
end

-- Biblioteca de skins: capturadas de jogadores, adicionadas por ID/malha ou importadas do jogo
local SkinLib = {}
local skinLibListeners = {}
local skinOptions = { AutoCapture = true }

local function notifySkinLib()
    for _, fn in ipairs(skinLibListeners) do pcall(fn) end
end

local function skinLibHas(name)
    for _, e in ipairs(SkinLib) do
        if e.name == name then return true end
    end
    return false
end

local function addSkinToLib(name, obj, style)
    for i, e in ipairs(SkinLib) do
        if e.name == name then
            SkinLib[i] = { name = name, obj = obj, style = style }
            notifySkinLib()
            return
        end
    end
    SkinLib[#SkinLib + 1] = { name = name, obj = obj, style = style }
    notifySkinLib()
end

local function findSkin(query)
    local q = string.lower(query)
    for _, e in ipairs(SkinLib) do
        if string.lower(e.name) == q then return e end
    end
    for _, e in ipairs(SkinLib) do
        if string.find(string.lower(e.name), q, 1, true) then return e end
    end
    return nil
end

-- Skins fake iniciais (cor + material + efeitos)
for _, def in ipairs(WeaponStyleList) do
    addSkinToLib(def.name, nil, def.style)
end

-- O MM2 guarda a arma das costas atraves de ObjectValues (DisplayRefKnife / DisplayRefGun)
local function getDisplayObj(char, slot)
    if not char then return nil end
    local ref = char:FindFirstChild("DisplayRef" .. slot)
    if ref and ref:IsA("ObjectValue") and ref.Value and ref.Value.Parent then
        return ref.Value
    end
    return char:FindFirstChild(slot .. "Display")
end

-- "impressao digital" do visual de uma arma (pra nao capturar a mesma skin duas vezes)
local function skinKey(inst)
    local parts = {}
    local list = inst:GetDescendants()
    table.insert(list, inst)
    for _, d in ipairs(list) do
        if d:IsA("MeshPart") then
            parts[#parts + 1] = d.MeshId .. "|" .. d.TextureID
        elseif d:IsA("SpecialMesh") then
            parts[#parts + 1] = d.MeshId .. "|" .. d.TextureId
        end
    end
    table.sort(parts)
    return table.concat(parts, ";")
end

-- Tenta descobrir o nome real da skin (alguns jogos guardam em atributos / StringValue)
local function guessSkinName(obj)
    local keys = { "ItemName", "SkinName", "WeaponName", "DisplayName", "Item", "Skin" }
    local holders = { obj, obj:FindFirstChild("Handle") }
    for _, h in ipairs(holders) do
        for _, k in ipairs(keys) do
            local v = h:GetAttribute(k)
            if typeof(v) == "string" and v ~= "" then return v end
            local sv = h:FindFirstChild(k)
            if sv and sv:IsA("StringValue") and sv.Value ~= "" then return sv.Value end
        end
    end
    return nil
end

local capturedKeys = {}   -- impressao digital -> entrada da biblioteca

-- Guarda o visual REAL da arma de outro jogador. Cada skin guarda duas versoes:
-- a da mao (Tool) e a das costas (Display), pra encaixar certo nos dois lugares.
local function captureFromObject(plr, obj, slot, isDisplay)
    if not obj or not obj.Parent then return end
    local key = skinKey(obj)
    if key == "" then return end

    local entry = capturedKeys[key]
    if not entry then
        local guess = guessSkinName(obj)
        local base
        if guess then
            base = "[real] " .. guess
        else
            base = "[real] " .. ((slot == "Knife") and "Faca" or "Arma")
        end
        local nome, n = base, 1
        while skinLibHas(nome) do
            n = n + 1
            nome = base .. " #" .. n
        end
        addSkinToLib(nome, nil)
        entry = SkinLib[#SkinLib]
        capturedKeys[key] = entry
    end

    local field = isDisplay and "objBack" or "obj"
    if entry[field] then return end
    local ok, clone = pcall(function() return obj:Clone() end)
    if ok and clone then
        entry[field] = clone
        notifySkinLib()
    end
end

-- Copia o visual das armas que OUTROS jogadores estao segurando / carregando nas costas
local function captureAll(force)
    if not force and not skinOptions.AutoCapture then return end
    for _, plr in ipairs(Players:GetPlayers()) do
        local char = plr ~= LocalPlayer and plr.Character
        if char then
            for _, slot in ipairs({ "Knife", "Gun" }) do
                local tool = char:FindFirstChild(slot)
                if tool and tool:IsA("Tool") then
                    captureFromObject(plr, tool, slot, false)
                end
                local ref = char:FindFirstChild("DisplayRef" .. slot)
                if ref and ref:IsA("ObjectValue") and ref.Value then
                    captureFromObject(plr, ref.Value, slot, true)
                end
            end
        end
    end
    if force then
        notify("Noni Hub", #SkinLib .. " skins na lista", 3)
    end
end

local lastCapture = 0
local function updateCapture()
    if tick() - lastCapture < 2 then return end
    lastCapture = tick()
    captureAll(false)
end

local function normalizeAsset(x)
    x = tostring(x or "")
    x = x:gsub("%s", "")
    if x == "" then return "" end
    if x:find("rbxassetid://", 1, true) or x:find("http", 1, true) then return x end
    local digits = x:gsub("%D", "")
    if digits == "" then return "" end
    return "rbxassetid://" .. digits
end

-- Adiciona uma skin carregando um modelo pelo ID (Toolbox / catalogo: espada, acessorio, gear...)
local function loadAssetSkin(idText, customName)
    local id = tostring(idText or ""):gsub("%D", "")
    if id == "" then
        notify("Noni Hub", "Digite um ID valido", 3)
        return
    end
    task.spawn(function()
        local ok, objs = pcall(function()
            return game:GetObjects("rbxassetid://" .. id)
        end)
        if not ok or not objs or #objs == 0 then
            notify("Noni Hub", "Nao consegui carregar o ID " .. id, 4)
            return
        end

        local pick = nil
        for _, o in ipairs(objs) do
            if o:IsA("Tool") or o:IsA("Accessory") or o:IsA("Model") or o:IsA("BasePart") then
                pick = o
                break
            end
        end
        if not pick then
            for _, o in ipairs(objs) do
                local d = o:FindFirstChildWhichIsA("Tool", true)
                    or o:FindFirstChildWhichIsA("Accessory", true)
                    or o:FindFirstChildWhichIsA("Model", true)
                    or o:FindFirstChildWhichIsA("BasePart", true)
                if d then
                    pick = d
                    break
                end
            end
        end
        if not pick or not (pick:IsA("BasePart") or pick:FindFirstChildWhichIsA("BasePart", true)) then
            notify("Noni Hub", "Esse ID nao tem modelo 3D utilizavel", 4)
            return
        end

        local nome = tostring(customName or ""):match("^%s*(.-)%s*$")
        if nome == "" then nome = "ID " .. id end
        addSkinToLib(nome, pick)
        notify("Noni Hub", "Skin adicionada: " .. nome, 3)
    end)
end

-- Cria uma skin a partir de uma malha (MeshId) + textura
local function createMeshSkin(meshText, texText, scaleText, customName)
    local meshId = normalizeAsset(meshText)
    if meshId == "" then
        notify("Noni Hub", "Digite o ID da malha", 3)
        return
    end
    local texId = normalizeAsset(texText)
    local scale = tonumber(scaleText) or 1

    local part = Instance.new("Part")
    part.Name = "Handle"
    part.Size = Vector3.new(1, 1, 1)
    part.Anchored = false
    part.CanCollide = false

    local mesh = Instance.new("SpecialMesh")
    mesh.MeshType = Enum.MeshType.FileMesh
    mesh.MeshId = meshId
    if texId ~= "" then mesh.TextureId = texId end
    mesh.Scale = Vector3.new(scale, scale, scale)
    mesh.Parent = part

    local nome = tostring(customName or ""):match("^%s*(.-)%s*$")
    if nome == "" then nome = "Malha " .. (meshId:gsub("%D", "")) end
    addSkinToLib(nome, part)
    notify("Noni Hub", "Skin criada: " .. nome, 3)
end

-- Adiciona ao menu os modelos que o jogo tem no ReplicatedStorage etc.
local function importGameModels()
    local idx = buildSkinIndex()
    local n = 0
    for _, e in ipairs(idx) do
        local nome = "[jogo] " .. e.name
        if not skinLibHas(nome) then
            addSkinToLib(nome, e.obj)
            n = n + 1
        end
    end
    notify("Noni Hub", n .. " modelos do jogo adicionados", 3)
end

-- Diagnostico: imprime no console (F9) onde o script procura e o que achou
local function diagnoseSkins()
    local lines = {}
    local function add(t) lines[#lines + 1] = t end

    add("=== DIAGNOSTICO DE SKINS ===")
    add("getnilinstances: " .. tostring(getnilinstances ~= nil)
        .. " | firetouchinterest: " .. tostring(firetouchinterest ~= nil))

    local roots = {
        game:GetService("ReplicatedStorage"),
        game:GetService("StarterPack"),
        game:GetService("ReplicatedFirst"),
        game:GetService("Lighting"),
    }
    for _, r in ipairs(roots) do
        local tmp = {}
        scanRootInto(tmp, r)
        add(r.Name .. ": " .. #tmp .. " modelos indexados")
    end

    local rsNames = {}
    for _, c in ipairs(ReplicatedStorage:GetChildren()) do
        rsNames[#rsNames + 1] = c.Name .. "(" .. c.ClassName .. ")"
    end
    add("ReplicatedStorage filhos: " .. table.concat(rsNames, ", "))

    local char = LocalPlayer.Character
    if char then
        local cn = {}
        for _, c in ipairs(char:GetChildren()) do
            cn[#cn + 1] = c.Name .. "(" .. c.ClassName .. ")"
        end
        add("Personagem: " .. table.concat(cn, ", "))
        add("KnifeDisplay: " .. tostring(char:FindFirstChild("KnifeDisplay") ~= nil)
            .. " | GunDisplay: " .. tostring(char:FindFirstChild("GunDisplay") ~= nil))
    else
        add("Personagem: nenhum")
    end

    local bp = LocalPlayer:FindFirstChild("Backpack")
    if bp then
        local bn = {}
        for _, c in ipairs(bp:GetChildren()) do
            bn[#bn + 1] = c.Name .. "(" .. c.ClassName .. ")"
        end
        add("Backpack: " .. table.concat(bn, ", "))
    end

    add("Skin escolhida - faca: " .. tostring(SkinChanger.Knife) .. " | arma: " .. tostring(SkinChanger.Gun))
    add("Biblioteca de skins: " .. #SkinLib .. " itens")
    local ch = LocalPlayer.Character
    if ch then
        for _, slot in ipairs({ "Knife", "Gun" }) do
            local d = getDisplayObj(ch, slot)
            add("Display " .. slot .. ": " .. (d and (d.ClassName .. " " .. d:GetFullName()) or "nao achado"))
        end
    end

    for _, l in ipairs(lines) do
        print("[NoniHub] " .. l)
    end
    notify("Noni Hub", "Diagnostico impresso no console (F9)", 4)
end

-- Nomes das skins da biblioteca, em ordem alfabetica (usado pelo menu)
local function getSkinNames()
    local list = {}
    for _, e in ipairs(SkinLib) do list[#list + 1] = e.name end
    table.sort(list, function(x, y) return string.lower(x) < string.lower(y) end)
    return list
end

-- Aplica um estilo (cor/material/efeitos) na propria arma, sem trocar o modelo
local function applyWeaponStyle(container, style)
    local parts = {}
    local list = container:GetDescendants()
    table.insert(list, container)
    for _, d in ipairs(list) do
        if d:IsA("BasePart") then parts[#parts + 1] = d end
    end
    if #parts == 0 then return false end

    local main = container:FindFirstChild("Handle", true)
    if not (main and main:IsA("BasePart")) then
        main = container:IsA("BasePart") and container or parts[1]
    end
    applyStyleToParts(container, parts, style, main)
    return true
end

local function restoreTool(container)
    restoreStyleOwner(container)
    local f = container:FindFirstChild(SKIN_TAG)
    if f then f:Destroy() end
    local list = container:GetDescendants()
    table.insert(list, container)
    for _, d in ipairs(list) do
        local orig = d:GetAttribute("NoniOrigT")
        if orig ~= nil then
            d.Transparency = orig
            d:SetAttribute("NoniOrigT", nil)
        end
    end
end

-- Aplica a skin em qualquer "container" (Tool na mao OU o modelo das costas),
-- soldando as pecas da skin na peca ancora dele.
local function applySkinCore(container, anchor, srcObj, rot)
    if not anchor or not anchor:IsA("BasePart") then return false end

    local clone = srcObj:Clone()
    -- remove scripts, juntas e humanoides do modelo (vamos soldar tudo na ancora)
    for _, d in ipairs(clone:GetDescendants()) do
        if d:IsA("LuaSourceContainer") or d:IsA("JointInstance") or d:IsA("WeldConstraint")
            or d:IsA("Humanoid") then
            d:Destroy()
        end
    end

    local parts = {}
    if clone:IsA("BasePart") then parts[#parts + 1] = clone end
    for _, d in ipairs(clone:GetDescendants()) do
        if d:IsA("BasePart") then parts[#parts + 1] = d end
    end
    if #parts == 0 then
        clone:Destroy()
        return false
    end

    -- parte de referencia: Handle (Tool), PrimaryPart (Model) ou a primeira parte
    local ref = nil
    if clone:IsA("Tool") then ref = clone:FindFirstChild("Handle") end
    if (not ref or not ref:IsA("BasePart")) and clone:IsA("Model") then ref = clone.PrimaryPart end
    if not ref or not ref:IsA("BasePart") then ref = parts[1] end

    rot = rot or Vector3.new(0, 0, 0)
    local rotCF = CFrame.Angles(math.rad(rot.X), math.rad(rot.Y), math.rad(rot.Z))
    local refInv = ref.CFrame:Inverse()

    -- esconde o visual original
    local list = container:GetDescendants()
    table.insert(list, container)
    for _, d in ipairs(list) do
        if d:IsA("BasePart") or d:IsA("Decal") or d:IsA("Texture") then
            if d:GetAttribute("NoniOrigT") == nil then
                d:SetAttribute("NoniOrigT", d.Transparency)
            end
            d.Transparency = 1
        end
    end

    local folder = Instance.new("Folder")
    folder.Name = SKIN_TAG
    local anchorCF = anchor.CFrame
    for _, part in ipairs(parts) do
        local rel = refInv * part.CFrame
        part.Anchored = false
        part.CanCollide = false
        part.CanTouch = false
        part.CanQuery = false
        part.Massless = true
        pcall(function() part.RootPriority = -127 end)
        -- ja nasce no lugar certo (antes ficava onde o outro jogador estava e puxava o personagem)
        part.CFrame = anchorCF * (rotCF * rel)
        part:SetAttribute("NoniSkinBaseT", part.Transparency)
        local w = Instance.new("Weld")
        w.Part0 = anchor
        w.Part1 = part
        w.C0 = rotCF * rel
        w.Parent = part
        part.Parent = folder
    end
    folder.Parent = container

    if not clone:IsA("BasePart") then clone:Destroy() end
    return true
end

-- peca ancora do modelo que fica nas costas (KnifeDisplay / GunDisplay)
local function getDisplayAnchor(display)
    if display:IsA("BasePart") then return display end
    local h = display:FindFirstChild("Handle", true)
    if h and h:IsA("BasePart") then return h end
    if display:IsA("Model") and display.PrimaryPart then return display.PrimaryPart end
    return display:FindFirstChildWhichIsA("BasePart", true)
end

local function setSkin(slot, name)
    local trimmed = (name or ""):match("^%s*(.-)%s*$")
    if trimmed == "" then
        SkinChanger[slot] = nil
        skinVersion[slot] = skinVersion[slot] + 1
        notify("Noni Hub", "Skin original restaurada", 2)
        return
    end

    local found = findSkin(trimmed)
    if not found then
        skinIndex = nil   -- reconstroi o indice e tenta de novo
        found = findSkin(trimmed)
    end
    if not found then
        notify("Noni Hub", "Skin nao encontrada: " .. trimmed .. " (use Listar skins)", 4)
        return
    end

    SkinChanger[slot] = found.name
    skinVersion[slot] = skinVersion[slot] + 1
    notify("Noni Hub", "Skin: " .. found.name .. " (equipe a arma)", 3)
end

local lastSkinCheck = 0
local function updateSkins()
    if tick() - lastSkinCheck < 0.4 then return end
    lastSkinCheck = tick()

    local char = LocalPlayer.Character
    local bp = LocalPlayer:FindFirstChild("Backpack")
    for _, slot in ipairs({ "Knife", "Gun" }) do
        local targets = {}

        -- 1) a arma na mao (Tool)
        local tool = (char and char:FindFirstChild(slot)) or (bp and bp:FindFirstChild(slot))
        if tool and tool:IsA("Tool") then
            targets[#targets + 1] = { obj = tool, anchor = tool:FindFirstChild("Handle"), rot = SKIN_ROT[slot] }
        end

        -- 2) a arma guardada nas costas (KnifeDisplay / GunDisplay)
        local display = getDisplayObj(char, slot)
        if display then
            targets[#targets + 1] = { obj = display, anchor = getDisplayAnchor(display), rot = SKIN_ROT_BACK[slot], back = true }
        end

        for _, t in ipairs(targets) do
            local inst = t.obj
            if inst:GetAttribute("NoniSkinVer") ~= skinVersion[slot] then
                inst:SetAttribute("NoniSkinVer", skinVersion[slot])
                restoreTool(inst)
                local want = SkinChanger[slot]
                if want then
                    local src = findSkin(want)
                    if src then
                        local ok, res
                        if src.style then
                            ok, res = pcall(applyWeaponStyle, inst, src.style)
                        else
                            local srcObj = t.back and (src.objBack or src.obj) or (src.obj or src.objBack)
                            ok, res = pcall(applySkinCore, inst, t.anchor, srcObj, t.rot)
                        end
                        if not ok or not res then
                            warn("[NoniHub] skin '" .. want .. "' em " .. inst.Name .. " falhou: "
                                .. (ok and "sem peca ancora / sem partes" or tostring(res)))
                            notify("Noni Hub", "Falha ao aplicar a skin " .. want .. " (veja F9)", 3)
                            restoreTool(inst)
                        end
                    else
                        warn("[NoniHub] modelo da skin '" .. want .. "' nao encontrado")
                    end
                end
            end
        end
    end
end

-- A skin das costas some enquanto a arma esta na mao (igual ao jogo faz com a original)
local function syncDisplaySkins()
    local char = LocalPlayer.Character
    if not char then return end
    for _, slot in ipairs({ "Knife", "Gun" }) do
        local display = getDisplayObj(char, slot)
        if display then
            local bk = styleBackup[display]
            if bk then
                local eq = char:FindFirstChild(slot) ~= nil
                for _, fx in ipairs(bk.fx) do
                    if fx.Parent then fx.Enabled = not eq end
                end
            end
        end
        local folder = display and display:FindFirstChild(SKIN_TAG)
        if folder then
            local equipped = char:FindFirstChild(slot) ~= nil
            for _, part in ipairs(folder:GetChildren()) do
                if part:IsA("BasePart") then
                    local base = part:GetAttribute("NoniSkinBaseT") or 0
                    part.Transparency = equipped and 1 or base
                end
            end
        end
    end
end

-- ==================== INTRO ====================
local function createIntroScreen()
    local playerGui = LocalPlayer:WaitForChild("PlayerGui")

    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "NoniHubIntro"
    screenGui.ResetOnSpawn = false
    screenGui.IgnoreGuiInset = true
    screenGui.DisplayOrder = 99999999
    screenGui.Parent = playerGui

    local background = Instance.new("Frame")
    background.Size = UDim2.new(1, 0, 1, 0)
    background.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    background.BorderSizePixel = 0
    background.ZIndex = 100
    background.Parent = screenGui

    local uiGradient = Instance.new("UIGradient")
    uiGradient.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0,   Color3.fromRGB(15, 0, 0)),
        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(120, 15, 20)),
        ColorSequenceKeypoint.new(1,   Color3.fromRGB(20, 0, 0))
    })
    uiGradient.Parent = background

    local rotateTween = TweenService:Create(uiGradient,
        TweenInfo.new(4, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut, -1, true),
        {Rotation = 360})
    rotateTween:Play()

    local titleLabel = Instance.new("TextLabel")
    titleLabel.Size = UDim2.new(0.8, 0, 0.2, 0)
    titleLabel.Position = UDim2.new(0.1, 0, 0.4, 0)
    titleLabel.BackgroundTransparency = 1
    titleLabel.Text = "NONI HUB"
    titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleLabel.TextScaled = true
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.ZIndex = 101
    titleLabel.Parent = background

    local creditsLabel = Instance.new("TextLabel")
    creditsLabel.Size = UDim2.new(0.4, 0, 0.05, 0)
    creditsLabel.Position = UDim2.new(0.58, 0, 0.92, 0)
    creditsLabel.BackgroundTransparency = 1
    creditsLabel.Text = "feito por Ds k s k s/hoaid"
    creditsLabel.TextColor3 = Color3.fromRGB(200, 120, 120)
    creditsLabel.TextScaled = true
    creditsLabel.Font = Enum.Font.Gotham
    creditsLabel.TextXAlignment = Enum.TextXAlignment.Right
    creditsLabel.ZIndex = 101
    creditsLabel.Parent = background

    task.delay(3, function()
        local ti = TweenInfo.new(1, Enum.EasingStyle.Linear, Enum.EasingDirection.Out)
        local t1 = TweenService:Create(background, ti, {BackgroundTransparency = 1})
        local t2 = TweenService:Create(titleLabel, ti, {TextTransparency = 1})
        local t3 = TweenService:Create(creditsLabel, ti, {TextTransparency = 1})
        t1:Play(); t2:Play(); t3:Play()
        t1.Completed:Connect(function()
            rotateTween:Cancel()
            screenGui:Destroy()
        end)
    end)
end

-- ==================== UI PRINCIPAL ====================
local function createMainUI()
    local playerGui = LocalPlayer:WaitForChild("PlayerGui")

    local mainGui = Instance.new("ScreenGui")
    mainGui.Name = "NoniHubMM2"
    mainGui.ResetOnSpawn = false
    mainGui.DisplayOrder = 99999
    mainGui.Parent = playerGui

    local Theme = {
        bg     = Color3.fromRGB(13, 13, 19),
        panel  = Color3.fromRGB(22, 22, 32),
        panel2 = Color3.fromRGB(31, 31, 45),
        field  = Color3.fromRGB(17, 17, 25),
        text   = Color3.fromRGB(240, 240, 248),
        sub    = Color3.fromRGB(150, 150, 172),
        off    = Color3.fromRGB(62, 62, 84),
        hover  = Color3.fromRGB(44, 44, 62),
    }
    local currentAccent = Color3.fromRGB(200, 40, 50)
    local accentRefs = {}
    local toggleRefreshers = {}
    local tabs = {}
    local activeTab = nil
    local orders = {}
    local applyAccent

    -- ---------- helpers ----------
    local function useAccent(inst, prop)
        accentRefs[#accentRefs + 1] = { inst = inst, prop = prop }
        inst[prop] = currentAccent
    end

    local function corner(inst, radius)
        local c = Instance.new("UICorner")
        c.CornerRadius = UDim.new(0, radius or 8)
        c.Parent = inst
        return c
    end

    local function stroke(inst, color, thickness, transparency)
        local st = Instance.new("UIStroke")
        st.Color = color
        st.Thickness = thickness or 1
        st.Transparency = transparency or 0
        st.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
        st.Parent = inst
        return st
    end

    local function nextOrder(page)
        orders[page] = (orders[page] or 0) + 1
        return orders[page]
    end

    local function hoverEffect(btn, normal, over)
        btn.MouseEnter:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.12), { BackgroundColor3 = over }):Play()
        end)
        btn.MouseLeave:Connect(function()
            TweenService:Create(btn, TweenInfo.new(0.12), { BackgroundColor3 = normal }):Play()
        end)
    end

    -- ---------- janela ----------
    local defaultMainPos = UDim2.new(0.5, -270, 0.5, -180)
    local defaultMinPos = UDim2.new(0, 15, 0, 65)

    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 540, 0, 360)
    frame.Position = defaultMainPos
    frame.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    frame.BorderSizePixel = 0
    frame.Active = true
    frame.Draggable = true
    frame.Parent = mainGui
    corner(frame, 14)
    useAccent(stroke(frame, currentAccent, 1.5), "Color")

    local grad = Instance.new("UIGradient")
    grad.Color = ColorSequence.new(Color3.fromRGB(22, 22, 33), Color3.fromRGB(11, 11, 16))
    grad.Rotation = 90
    grad.Parent = frame

    local topBar = Instance.new("Frame")
    topBar.Size = UDim2.new(1, 0, 0, 46)
    topBar.BackgroundColor3 = Theme.panel
    topBar.BorderSizePixel = 0
    topBar.Parent = frame
    corner(topBar, 14)

    local topFix = Instance.new("Frame")
    topFix.Size = UDim2.new(1, 0, 0, 14)
    topFix.Position = UDim2.new(0, 0, 1, -14)
    topFix.BackgroundColor3 = Theme.panel
    topFix.BorderSizePixel = 0
    topFix.Parent = topBar

    local topLine = Instance.new("Frame")
    topLine.Size = UDim2.new(1, 0, 0, 2)
    topLine.Position = UDim2.new(0, 0, 1, -2)
    topLine.BorderSizePixel = 0
    topLine.ZIndex = 2
    topLine.Parent = topBar
    useAccent(topLine, "BackgroundColor3")

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(0, 100, 1, 0)
    title.Position = UDim2.new(0, 18, 0, 0)
    title.BackgroundTransparency = 1
    title.Text = "NONI HUB"
    title.Font = Enum.Font.GothamBold
    title.TextSize = 17
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = topBar
    useAccent(title, "TextColor3")

    local subtitle = Instance.new("TextLabel")
    subtitle.Size = UDim2.new(0, 150, 1, 0)
    subtitle.Position = UDim2.new(0, 118, 0, 2)
    subtitle.BackgroundTransparency = 1
    subtitle.Text = "MM2 Edition"
    subtitle.TextColor3 = Theme.sub
    subtitle.Font = Enum.Font.Gotham
    subtitle.TextSize = 12
    subtitle.TextXAlignment = Enum.TextXAlignment.Left
    subtitle.Parent = topBar

    local minimizeBtn = Instance.new("TextButton")
    minimizeBtn.Size = UDim2.new(0, 30, 0, 30)
    minimizeBtn.Position = UDim2.new(1, -42, 0, 8)
    minimizeBtn.BackgroundColor3 = Theme.panel2
    minimizeBtn.BorderSizePixel = 0
    minimizeBtn.Text = "-"
    minimizeBtn.TextColor3 = Theme.text
    minimizeBtn.Font = Enum.Font.GothamBold
    minimizeBtn.TextSize = 18
    minimizeBtn.ZIndex = 3
    minimizeBtn.Parent = topBar
    corner(minimizeBtn, 8)
    useAccent(stroke(minimizeBtn, currentAccent, 1, 0.4), "Color")
    hoverEffect(minimizeBtn, Theme.panel2, Theme.hover)

    local minFrame = Instance.new("TextButton")
    minFrame.Size = UDim2.new(0, 52, 0, 52)
    minFrame.Position = defaultMinPos
    minFrame.BackgroundColor3 = Theme.panel
    minFrame.BorderSizePixel = 0
    minFrame.Text = "N"
    minFrame.Font = Enum.Font.GothamBold
    minFrame.TextSize = 26
    minFrame.Visible = false
    minFrame.Active = true
    minFrame.Draggable = true
    minFrame.Parent = mainGui
    corner(minFrame, 14)
    useAccent(minFrame, "TextColor3")
    useAccent(stroke(minFrame, currentAccent, 1.8), "Color")

    local function minimizeUI()
        frame.Visible = false
        minFrame.Position = defaultMinPos
        minFrame.Visible = true
    end
    local function expandUI()
        minFrame.Visible = false
        frame.Position = defaultMainPos
        frame.Visible = true
    end
    minimizeBtn.MouseButton1Click:Connect(minimizeUI)
    minFrame.MouseButton1Click:Connect(expandUI)

    -- ---------- sidebar e conteudo ----------
    local sidebar = Instance.new("Frame")
    sidebar.Size = UDim2.new(0, 124, 1, -62)
    sidebar.Position = UDim2.new(0, 8, 0, 54)
    sidebar.BackgroundColor3 = Theme.panel
    sidebar.BorderSizePixel = 0
    sidebar.Parent = frame
    corner(sidebar, 10)

    local sideLayout = Instance.new("UIListLayout")
    sideLayout.Padding = UDim.new(0, 6)
    sideLayout.SortOrder = Enum.SortOrder.LayoutOrder
    sideLayout.Parent = sidebar
    local sidePad = Instance.new("UIPadding")
    sidePad.PaddingTop = UDim.new(0, 8)
    sidePad.PaddingLeft = UDim.new(0, 8)
    sidePad.PaddingRight = UDim.new(0, 8)
    sidePad.Parent = sidebar

    local content = Instance.new("Frame")
    content.Size = UDim2.new(1, -148, 1, -62)
    content.Position = UDim2.new(0, 140, 0, 54)
    content.BackgroundColor3 = Theme.panel
    content.BorderSizePixel = 0
    content.ClipsDescendants = true
    content.Parent = frame
    corner(content, 10)

    local function selectTab(activeBtn)
        activeTab = activeBtn
        for _, t in ipairs(tabs) do
            local on = (t.btn == activeBtn)
            t.page.Visible = on
            t.btn.BackgroundColor3 = on and currentAccent or Theme.panel2
            t.btn.TextColor3 = on and Color3.fromRGB(255, 255, 255) or Theme.sub
        end
    end

    local function addTab(name, page)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, 0, 0, 36)
        btn.LayoutOrder = #tabs + 1
        btn.BackgroundColor3 = Theme.panel2
        btn.BorderSizePixel = 0
        btn.Text = name
        btn.Font = Enum.Font.GothamBold
        btn.TextSize = 12
        btn.Parent = sidebar
        corner(btn, 8)
        tabs[#tabs + 1] = { btn = btn, page = page }
        btn.MouseButton1Click:Connect(function() selectTab(btn) end)
        return btn
    end

    local function newPage()
        local page = Instance.new("ScrollingFrame")
        page.Size = UDim2.new(1, -8, 1, -8)
        page.Position = UDim2.new(0, 4, 0, 4)
        page.BackgroundTransparency = 1
        page.BorderSizePixel = 0
        page.ScrollBarThickness = 3
        page.CanvasSize = UDim2.new(0, 0, 0, 0)
        page.AutomaticCanvasSize = Enum.AutomaticSize.Y
        page.Visible = false
        page.Parent = content
        useAccent(page, "ScrollBarImageColor3")

        local l = Instance.new("UIListLayout")
        l.Padding = UDim.new(0, 6)
        l.SortOrder = Enum.SortOrder.LayoutOrder
        l.Parent = page
        local pad = Instance.new("UIPadding")
        pad.PaddingLeft = UDim.new(0, 4)
        pad.PaddingRight = UDim.new(0, 8)
        pad.PaddingTop = UDim.new(0, 4)
        pad.PaddingBottom = UDim.new(0, 10)
        pad.Parent = page
        return page
    end

    -- ---------- componentes ----------
    local function addSection(page, text)
        local l = Instance.new("TextLabel")
        l.Size = UDim2.new(1, 0, 0, 22)
        l.LayoutOrder = nextOrder(page)
        l.BackgroundTransparency = 1
        l.Text = string.upper(text)
        l.Font = Enum.Font.GothamBold
        l.TextSize = 11
        l.TextXAlignment = Enum.TextXAlignment.Left
        l.Parent = page
        useAccent(l, "TextColor3")
        return l
    end

    local function addNote(page, text)
        local l = Instance.new("TextLabel")
        l.Size = UDim2.new(1, 0, 0, 0)
        l.AutomaticSize = Enum.AutomaticSize.Y
        l.LayoutOrder = nextOrder(page)
        l.BackgroundTransparency = 1
        l.Text = text
        l.TextColor3 = Theme.sub
        l.Font = Enum.Font.Gotham
        l.TextSize = 11
        l.TextWrapped = true
        l.TextXAlignment = Enum.TextXAlignment.Left
        l.TextYAlignment = Enum.TextYAlignment.Top
        l.Parent = page
        return l
    end

    local function addToggle(page, name, default, callback)
        local row = Instance.new("Frame")
        row.Size = UDim2.new(1, 0, 0, 36)
        row.LayoutOrder = nextOrder(page)
        row.BackgroundColor3 = Theme.panel2
        row.BorderSizePixel = 0
        row.Parent = page
        corner(row, 8)

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, -70, 1, 0)
        label.Position = UDim2.new(0, 12, 0, 0)
        label.BackgroundTransparency = 1
        label.Text = name
        label.TextColor3 = Theme.text
        label.Font = Enum.Font.GothamMedium
        label.TextSize = 12
        label.TextXAlignment = Enum.TextXAlignment.Left
        label.Parent = row

        local pill = Instance.new("Frame")
        pill.Size = UDim2.new(0, 40, 0, 20)
        pill.Position = UDim2.new(1, -52, 0.5, -10)
        pill.BorderSizePixel = 0
        pill.Parent = row
        corner(pill, 10)

        local knob = Instance.new("Frame")
        knob.Size = UDim2.new(0, 16, 0, 16)
        knob.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
        knob.BorderSizePixel = 0
        knob.Parent = pill
        corner(knob, 8)

        local state = default
        local function refresh(animate)
            local goal = state and UDim2.new(1, -18, 0.5, -8) or UDim2.new(0, 2, 0.5, -8)
            pill.BackgroundColor3 = state and currentAccent or Theme.off
            if animate then
                TweenService:Create(knob, TweenInfo.new(0.15, Enum.EasingStyle.Quad), { Position = goal }):Play()
            else
                knob.Position = goal
            end
        end
        refresh(false)
        toggleRefreshers[#toggleRefreshers + 1] = function() refresh(false) end

        local hit = Instance.new("TextButton")
        hit.Size = UDim2.new(1, 0, 1, 0)
        hit.BackgroundTransparency = 1
        hit.Text = ""
        hit.Parent = row
        hit.MouseButton1Click:Connect(function()
            state = not state
            refresh(true)
            callback(state)
        end)
        return row
    end

    local function addButton(page, text, callback)
        local b = Instance.new("TextButton")
        b.Size = UDim2.new(1, 0, 0, 32)
        b.LayoutOrder = nextOrder(page)
        b.BackgroundColor3 = Theme.panel2
        b.BorderSizePixel = 0
        b.Text = text
        b.TextColor3 = Theme.text
        b.Font = Enum.Font.GothamBold
        b.TextSize = 12
        b.Parent = page
        corner(b, 8)
        useAccent(stroke(b, currentAccent, 1, 0.5), "Color")
        hoverEffect(b, Theme.panel2, Theme.hover)
        b.MouseButton1Click:Connect(callback)
        return b
    end

    local function addInput(page, placeholder)
        local box = Instance.new("TextBox")
        box.Size = UDim2.new(1, 0, 0, 30)
        box.LayoutOrder = nextOrder(page)
        box.BackgroundColor3 = Theme.field
        box.BorderSizePixel = 0
        box.Text = ""
        box.PlaceholderText = placeholder
        box.PlaceholderColor3 = Theme.sub
        box.TextColor3 = Theme.text
        box.Font = Enum.Font.Gotham
        box.TextSize = 12
        box.ClearTextOnFocus = false
        box.Parent = page
        corner(box, 8)
        stroke(box, Theme.off, 1, 0.3)
        return box
    end

    -- ---------- menu suspenso de skins ----------
    local dropdowns = {}

    local function addSkinDropdown(page, slot, title)
        local holder = Instance.new("Frame")
        holder.Size = UDim2.new(1, 0, 0, 0)
        holder.AutomaticSize = Enum.AutomaticSize.Y
        holder.LayoutOrder = nextOrder(page)
        holder.BackgroundTransparency = 1
        holder.Parent = page
        local hl = Instance.new("UIListLayout")
        hl.Padding = UDim.new(0, 4)
        hl.SortOrder = Enum.SortOrder.LayoutOrder
        hl.Parent = holder

        local head = Instance.new("TextButton")
        head.Size = UDim2.new(1, 0, 0, 36)
        head.LayoutOrder = 1
        head.BackgroundColor3 = Theme.panel2
        head.BorderSizePixel = 0
        head.TextColor3 = Theme.text
        head.Font = Enum.Font.GothamBold
        head.TextSize = 12
        head.TextTruncate = Enum.TextTruncate.AtEnd
        head.Parent = holder
        corner(head, 8)
        useAccent(stroke(head, currentAccent, 1, 0.4), "Color")
        hoverEffect(head, Theme.panel2, Theme.hover)

        local box = Instance.new("Frame")
        box.Size = UDim2.new(1, 0, 0, 150)
        box.LayoutOrder = 2
        box.BackgroundColor3 = Theme.field
        box.BorderSizePixel = 0
        box.Visible = false
        box.Parent = holder
        corner(box, 8)

        local search = Instance.new("TextBox")
        search.Size = UDim2.new(1, -12, 0, 26)
        search.Position = UDim2.new(0, 6, 0, 6)
        search.BackgroundColor3 = Theme.panel2
        search.BorderSizePixel = 0
        search.Text = ""
        search.PlaceholderText = "buscar..."
        search.PlaceholderColor3 = Theme.sub
        search.TextColor3 = Theme.text
        search.Font = Enum.Font.Gotham
        search.TextSize = 12
        search.ClearTextOnFocus = false
        search.Parent = box
        corner(search, 6)

        local scroll = Instance.new("ScrollingFrame")
        scroll.Size = UDim2.new(1, -12, 1, -42)
        scroll.Position = UDim2.new(0, 6, 0, 36)
        scroll.BackgroundTransparency = 1
        scroll.BorderSizePixel = 0
        scroll.ScrollBarThickness = 3
        scroll.CanvasSize = UDim2.new(0, 0, 0, 0)
        scroll.AutomaticCanvasSize = Enum.AutomaticSize.Y
        scroll.Parent = box
        useAccent(scroll, "ScrollBarImageColor3")
        local sl = Instance.new("UIListLayout")
        sl.Padding = UDim.new(0, 3)
        sl.SortOrder = Enum.SortOrder.LayoutOrder
        sl.Parent = scroll

        local function refreshHead()
            head.Text = title .. ":  " .. (SkinChanger[slot] or "Original") .. "   v"
        end

        local function refreshList()
            for _, c in ipairs(scroll:GetChildren()) do
                if c:IsA("TextButton") or c:IsA("TextLabel") then c:Destroy() end
            end
            local filter = string.lower(search.Text or "")

            local function addItem(text, order, onClick)
                local it = Instance.new(onClick and "TextButton" or "TextLabel")
                it.Size = UDim2.new(1, -6, 0, 26)
                it.LayoutOrder = order
                it.BackgroundColor3 = Theme.panel2
                it.BorderSizePixel = 0
                it.Text = text
                it.TextColor3 = onClick and Theme.text or Theme.sub
                it.Font = Enum.Font.Gotham
                it.TextSize = 12
                it.TextTruncate = Enum.TextTruncate.AtEnd
                it.Parent = scroll
                corner(it, 6)
                if onClick then
                    hoverEffect(it, Theme.panel2, Theme.hover)
                    it.MouseButton1Click:Connect(onClick)
                end
            end

            addItem("Original (sem skin)", 0, function()
                setSkin(slot, "")
                refreshHead()
                box.Visible = false
            end)

            local names = getSkinNames()
            if #names == 0 then
                addItem("Vazio: capture de jogadores ou adicione por ID", 1, nil)
                return
            end
            for i, name in ipairs(names) do
                if filter == "" or string.find(string.lower(name), filter, 1, true) then
                    addItem(name, i, function()
                        setSkin(slot, name)
                        refreshHead()
                        box.Visible = false
                    end)
                end
            end
        end

        head.MouseButton1Click:Connect(function()
            box.Visible = not box.Visible
            if box.Visible then
                search.Text = ""
                refreshList()
            end
        end)
        search:GetPropertyChangedSignal("Text"):Connect(function()
            if box.Visible then refreshList() end
        end)

        refreshHead()
        dropdowns[slot] = {
            refreshHead = refreshHead,
            refreshList = function()
                if box.Visible then refreshList() end
            end,
        }
    end

    skinLibListeners[#skinLibListeners + 1] = function()
        for _, d in pairs(dropdowns) do d.refreshList() end
    end

    local function addRotRow(page, label, tbl, slot)
        local row = Instance.new("Frame")
        row.Size = UDim2.new(1, 0, 0, 34)
        row.LayoutOrder = nextOrder(page)
        row.BackgroundColor3 = Theme.panel2
        row.BorderSizePixel = 0
        row.Parent = page
        corner(row, 8)

        local lbl = Instance.new("TextLabel")
        lbl.Size = UDim2.new(0, 92, 1, 0)
        lbl.Position = UDim2.new(0, 10, 0, 0)
        lbl.BackgroundTransparency = 1
        lbl.Text = label
        lbl.TextColor3 = Theme.text
        lbl.Font = Enum.Font.GothamMedium
        lbl.TextSize = 11
        lbl.TextXAlignment = Enum.TextXAlignment.Left
        lbl.Parent = row

        local v = tbl[slot]
        local initial = { v.X, v.Y, v.Z }
        local boxes = {}
        for i = 1, 3 do
            local b = Instance.new("TextBox")
            b.Size = UDim2.new(0, 46, 0, 24)
            b.Position = UDim2.new(0, 108 + (i - 1) * 52, 0.5, -12)
            b.BackgroundColor3 = Theme.field
            b.BorderSizePixel = 0
            b.Text = tostring(initial[i])
            b.TextColor3 = Theme.text
            b.Font = Enum.Font.Gotham
            b.TextSize = 11
            b.ClearTextOnFocus = false
            b.Parent = row
            corner(b, 6)
            boxes[i] = b
            b.FocusLost:Connect(function()
                tbl[slot] = Vector3.new(
                    tonumber(boxes[1].Text) or 0,
                    tonumber(boxes[2].Text) or 0,
                    tonumber(boxes[3].Text) or 0
                )
                skinVersion[slot] = skinVersion[slot] + 1
            end)
        end
    end

    -- ==================== PAGINAS ====================
    local pgMain = newPage()
    local pgSkins = newPage()
    local pgAvatar = newPage()
    local pgConfig = newPage()

    local tabMain = addTab("Principal", pgMain)
    addTab("Armas", pgSkins)
    addTab("Avatar", pgAvatar)
    addTab("Config", pgConfig)

    -- ----- Principal -----
    addSection(pgMain, "Visao")
    addToggle(pgMain, "ESP Roles", Settings.MM2.ESPEnabled, function(v) Settings.MM2.ESPEnabled = v end)
    addToggle(pgMain, "Gun ESP", Settings.MM2.GunESPEnabled, function(v) Settings.MM2.GunESPEnabled = v end)
    addSection(pgMain, "Combate")
    addToggle(pgMain, "Kill All (TP)", Settings.MM2.KillAuraEnabled, function(v) Settings.MM2.KillAuraEnabled = v end)
    addToggle(pgMain, "Auto Shoot Murder", Settings.MM2.AutoKillMurderer, function(v) Settings.MM2.AutoKillMurderer = v end)
    addToggle(pgMain, "Aimbot Cursor", Settings.MM2.AimbotCursor, function(v) Settings.MM2.AimbotCursor = v end)
    addToggle(pgMain, "Flee Mode", Settings.MM2.FleeMode, function(v) Settings.MM2.FleeMode = v end)
    addSection(pgMain, "Coleta")
    addToggle(pgMain, "Auto Grab Gun", Settings.MM2.AutoGrabGun, function(v) Settings.MM2.AutoGrabGun = v end)
    addToggle(pgMain, "Auto Coin", Settings.MM2.AutoCoinCollector, function(v) Settings.MM2.AutoCoinCollector = v end)

    -- ----- Armas (skin changer) -----
    addSection(pgSkins, "Skin da faca e da arma")
    addNote(pgSkins, "Skins visuais so pra voce: mudam cor, material e efeitos da sua faca ou arma (na mao e nas costas). Escolha e equipe a arma.")
    addSkinDropdown(pgSkins, "Knife", "Faca")
    addSkinDropdown(pgSkins, "Gun", "Arma")

    addSection(pgSkins, "Skins reais (copiadas do servidor)")
    addNote(pgSkins, "Se alguem no servidor estiver com uma skin (Icewing, Batwing, Ancient, FX...), o script copia o visual real e ela aparece na lista com [real]. Precisa do outro jogador estar com a arma na mao ou nas costas.")
    addToggle(pgSkins, "Captura automatica", skinOptions.AutoCapture, function(v) skinOptions.AutoCapture = v end)
    addButton(pgSkins, "Capturar agora", function() captureAll(true) end)

    addButton(pgSkins, "Restaurar skins originais", function()
        setSkin("Knife", "")
        setSkin("Gun", "")
        for _, d in pairs(dropdowns) do d.refreshHead() end
    end)

    -- ----- Avatar -----
    addSection(pgAvatar, "Visual rapido")
    addToggle(pgAvatar, "Headless", false, function(v) setAvatar("Headless", v) end)
    addToggle(pgAvatar, "Korblox (perna direita)", false, function(v) setAvatar("Korblox", v) end)

    -- ----- Config -----
    local palette = {
        { name = "Roxo",     color = Color3.fromRGB(120, 90, 200) },
        { name = "Azul",     color = Color3.fromRGB(70, 140, 255) },
        { name = "Verde",    color = Color3.fromRGB(70, 200, 120) },
        { name = "Vermelho", color = Color3.fromRGB(200, 40, 50) },
        { name = "Rosa",     color = Color3.fromRGB(240, 110, 200) },
        { name = "Dourado",  color = Color3.fromRGB(240, 190, 70) },
        { name = "Ciano",    color = Color3.fromRGB(80, 220, 230) },
        { name = "Laranja",  color = Color3.fromRGB(255, 140, 40) },
        { name = "Lima",     color = Color3.fromRGB(180, 240, 80) },
        { name = "Violeta",  color = Color3.fromRGB(180, 80, 255) },
        { name = "Branco",   color = Color3.fromRGB(240, 240, 240) },
        { name = "Sangue",   color = Color3.fromRGB(150, 15, 20) },
    }

    local function applyTransparency(v)
        frame.BackgroundTransparency = v
        topBar.BackgroundTransparency = v
        topFix.BackgroundTransparency = v
        sidebar.BackgroundTransparency = v
        content.BackgroundTransparency = v
        minFrame.BackgroundTransparency = v
    end

    addSection(pgConfig, "Cor de destaque")
    local colorGrid = Instance.new("Frame")
    colorGrid.Size = UDim2.new(1, 0, 0, 76)
    colorGrid.LayoutOrder = nextOrder(pgConfig)
    colorGrid.BackgroundTransparency = 1
    colorGrid.Parent = pgConfig
    local cg = Instance.new("UIGridLayout")
    cg.CellSize = UDim2.new(0, 30, 0, 30)
    cg.CellPadding = UDim2.new(0, 8, 0, 8)
    cg.SortOrder = Enum.SortOrder.LayoutOrder
    cg.Parent = colorGrid
    for i, item in ipairs(palette) do
        local cb = Instance.new("TextButton")
        cb.LayoutOrder = i
        cb.BackgroundColor3 = item.color
        cb.BorderSizePixel = 0
        cb.Text = ""
        cb.Parent = colorGrid
        corner(cb, 8)
        stroke(cb, Color3.fromRGB(255, 255, 255), 1, 0.6)
        cb.MouseButton1Click:Connect(function()
            applyAccent(item.color)
            notify("Noni Hub", "Cor: " .. item.name, 2)
        end)
    end

    addSection(pgConfig, "Transparencia")
    local tGrid = Instance.new("Frame")
    tGrid.Size = UDim2.new(1, 0, 0, 68)
    tGrid.LayoutOrder = nextOrder(pgConfig)
    tGrid.BackgroundTransparency = 1
    tGrid.Parent = pgConfig
    local tg = Instance.new("UIGridLayout")
    tg.CellSize = UDim2.new(0, 34, 0, 28)
    tg.CellPadding = UDim2.new(0, 8, 0, 8)
    tg.SortOrder = Enum.SortOrder.LayoutOrder
    tg.Parent = tGrid
    for i, val in ipairs({ 0, 0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9 }) do
        local tb = Instance.new("TextButton")
        tb.LayoutOrder = i
        tb.BackgroundColor3 = Theme.panel2
        tb.BorderSizePixel = 0
        tb.Text = tostring(val)
        tb.TextColor3 = Theme.text
        tb.Font = Enum.Font.GothamBold
        tb.TextSize = 11
        tb.Parent = tGrid
        corner(tb, 6)
        hoverEffect(tb, Theme.panel2, Theme.hover)
        tb.MouseButton1Click:Connect(function() applyTransparency(val) end)
    end

    addButton(pgConfig, "Resetar aparencia", function()
        applyTransparency(0)
        applyAccent(Color3.fromRGB(200, 40, 50))
        notify("Noni Hub", "Aparencia resetada", 2)
    end)
    addNote(pgConfig, "feito por Ds k s k s/hoaid")

    -- ---------- accent ----------
    applyAccent = function(color)
        currentAccent = color
        for _, r in ipairs(accentRefs) do
            pcall(function() r.inst[r.prop] = color end)
        end
        for _, fn in ipairs(toggleRefreshers) do fn() end
        if activeTab then selectTab(activeTab) end
    end

    selectTab(tabMain)

    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if not gameProcessed and input.KeyCode == Enum.KeyCode.K then
            if frame.Visible then minimizeUI() else expandUI() end
        end
    end)
end

-- ==================== LOGICA MM2 ====================

local function getPlayerRole(plr)
    if not plr or not plr.Character then return "Innocent" end
    local backpack = plr:FindFirstChild("Backpack")
    local character = plr.Character
    if (backpack and backpack:FindFirstChild("Knife")) or character:FindFirstChild("Knife") then
        return "Murderer"
    end
    if (backpack and backpack:FindFirstChild("Gun")) or character:FindFirstChild("Gun") then
        return "Sheriff"
    end
    return "Innocent"
end

local function isRoundActive()
    for _, plr in ipairs(Players:GetPlayers()) do
        local backpack = plr:FindFirstChild("Backpack")
        local character = plr.Character
        if (backpack and backpack:FindFirstChild("Knife")) or (character and character:FindFirstChild("Knife")) then
            return true
        end
    end
    return false
end

local function isAlive()
    local myChar = LocalPlayer.Character
    if not myChar then return false end
    if not myChar:FindFirstChild("HumanoidRootPart") then return false end
    local hum = myChar:FindFirstChildOfClass("Humanoid")
    if not hum then return false end
    if hum.Health <= 0 then return false end
    if hum:GetState() == Enum.HumanoidStateType.Dead then return false end
    return true
end

local function isPlayerInMap(plr)
    if not plr or not plr.Character then return false end
    local hrp = plr.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end
    local y = hrp.Position.Y
    if y < -100 or y > 500 then return false end
    return true
end

local function canRunMatchFunctions()
    return isAlive() and isRoundActive()
end

local function hasGun()
    local myChar = LocalPlayer.Character
    if not myChar then return false end
    if myChar:FindFirstChild("Gun") then return true end
    local backpack = LocalPlayer:FindFirstChild("Backpack")
    if backpack and backpack:FindFirstChild("Gun") then return true end
    return false
end

local function hasWeaponEquipped()
    local myChar = LocalPlayer.Character
    if not myChar then return false end
    if myChar:FindFirstChild("Knife") then return true end
    if myChar:FindFirstChild("Gun") then return true end
    for _, tool in ipairs(myChar:GetChildren()) do
        if tool:IsA("Tool") then
            local n = tool.Name:lower()
            if n:find("knife") or n:find("gun") or n:find("pistol") or n:find("revolver") or n:find("sword") then
                return true
            end
        end
    end
    return false
end

local function isInsideAnyPlayer(obj)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr.Character and obj:IsDescendantOf(plr.Character) then return true end
        local bp = plr:FindFirstChild("Backpack")
        if bp and obj:IsDescendantOf(bp) then return true end
    end
    return false
end

local function getClosestMurderer()
    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return nil, nil end
    local myHRP = myChar.HumanoidRootPart
    local target, dist = nil, math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and getPlayerRole(plr) == "Murderer" and isPlayerInMap(plr) then
            local mChar = plr.Character
            if mChar and mChar:FindFirstChild("HumanoidRootPart") then
                local mHum = mChar:FindFirstChildOfClass("Humanoid")
                if mHum and mHum.Health > 0 then
                    local d = (mChar.HumanoidRootPart.Position - myHRP.Position).Magnitude
                    if d < dist then
                        dist = d
                        target = mChar
                    end
                end
            end
        end
    end
    return target, dist
end

local function findSafeFleeTarget(murdererChar)
    if not murdererChar then return nil end
    local mHRP = murdererChar:FindFirstChild("HumanoidRootPart")
    if not mHRP then return nil end
    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return nil end
    local myHRP = myChar.HumanoidRootPart
    local bestTarget, bestScore = nil, -math.huge
    local searchRange = Settings.MM2.FleeSearchRange
    local minDistFromMurderer = Settings.MM2.FleeTargetMinDistance

    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character ~= murdererChar then
            if plr.Character and getPlayerRole(plr) ~= "Murderer" then
                local pChar = plr.Character
                local pHRP = pChar:FindFirstChild("HumanoidRootPart")
                local pHum = pChar:FindFirstChildOfClass("Humanoid")
                if pHRP and pHum and pHum.Health > 0 then
                    local distToMe = (pHRP.Position - myHRP.Position).Magnitude
                    if distToMe <= searchRange and distToMe > 30 then
                        local distToMurderer = (pHRP.Position - mHRP.Position).Magnitude
                        if distToMurderer >= minDistFromMurderer then
                            local score = distToMurderer + (distToMe * 0.3)
                            if score > bestScore then
                                bestScore = score
                                bestTarget = pChar
                            end
                        end
                    end
                end
            end
        end
    end
    return bestTarget
end

local function applyHighlight(plr)
    if plr == LocalPlayer then return end
    local function update()
        local char = plr.Character
        if char then
            local role = getPlayerRole(plr)
            local hl = char:FindFirstChild("NoniESP")
            if Settings.MM2.ESPEnabled then
                if not hl then
                    hl = Instance.new("Highlight")
                    hl.Name = "NoniESP"
                    hl.FillTransparency = 0.4
                    hl.OutlineTransparency = 0
                    hl.Parent = char
                end
                hl.FillColor = Settings.Colors[role] or Settings.Colors.Innocent
                hl.OutlineColor = hl.FillColor
                hl.Enabled = true
            elseif hl then
                hl.Enabled = false
            end
        end
    end
    if plr.Character then update() end
    plr.CharacterAdded:Connect(function()
        task.wait(0.5)
        update()
    end)
end

-- ================================================================
-- KILL AURA TP
-- ================================================================
local lastAttack = 0
local isKilling = false
local knifeRemotesPrinted = false

local function findStabRemote(knife)
    for _, d in ipairs(knife:GetDescendants()) do
        if d:IsA("RemoteEvent") and d.Name == "Stab" then return d end
    end
    return nil
end

-- Mostra no console (F9) os remotes da faca, pra ajudar a ajustar se algo nao matar
local function printKnifeRemotes(knife)
    if knifeRemotesPrinted then return end
    knifeRemotesPrinted = true
    local names = {}
    for _, d in ipairs(knife:GetDescendants()) do
        if d:IsA("RemoteEvent") or d:IsA("RemoteFunction") or d:IsA("BindableEvent") then
            names[#names + 1] = d:GetFullName() .. " (" .. d.ClassName .. ")"
        end
    end
    print("[NoniHub] Remotes da faca: " .. (#names > 0 and table.concat(names, " | ") or "nenhum"))
end

-- Ataca um alvo por tres caminhos ao mesmo tempo: evento de stab, ativar a faca e toque forcado
local function stabTarget(knife, tChar, tHRP)
    local stab = findStabRemote(knife)
    if stab then
        pcall(function() stab:FireServer("Slash") end)
    end
    pcall(function() knife:Activate() end)

    local handle = knife:FindFirstChild("Handle")
    if handle and firetouchinterest then
        local touchParts = { tHRP }
        local head = tChar:FindFirstChild("Head")
        if head then touchParts[#touchParts + 1] = head end
        local torso = tChar:FindFirstChild("UpperTorso") or tChar:FindFirstChild("Torso")
        if torso then touchParts[#touchParts + 1] = torso end
        for _, part in ipairs(touchParts) do
            pcall(function()
                firetouchinterest(part, handle, 0)
                firetouchinterest(part, handle, 1)
            end)
        end
    end
end

-- KILL ALL: passa por TODOS os jogadores vivos de uma vez (do mais perto ao mais longe)
local function runKillAura()
    if not Settings.MM2.KillAuraEnabled or isKilling then return end
    if tick() - lastAttack < Settings.MM2.AttackDelay then return end
    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return end
    if getPlayerRole(LocalPlayer) ~= "Murderer" then return end

    local knife = myChar:FindFirstChild("Knife")
        or (LocalPlayer.Backpack and LocalPlayer.Backpack:FindFirstChild("Knife"))
    if not knife then return end

    isKilling = true
    local ok, err = pcall(function()
        local hum = myChar:FindFirstChildOfClass("Humanoid")
        if knife.Parent ~= myChar and hum then
            hum:EquipTool(knife)
            task.wait(0.05)
        end
        printKnifeRemotes(knife)

        local myHRP = myChar.HumanoidRootPart
        local origin = myHRP.CFrame

        local targets = {}
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character then
                local tChar = plr.Character
                local tHRP = tChar:FindFirstChild("HumanoidRootPart")
                local tHum = tChar:FindFirstChildOfClass("Humanoid")
                if tHRP and tHum and tHum.Health > 0 and isPlayerInMap(plr) then
                    local d = (tHRP.Position - myHRP.Position).Magnitude
                    if d <= Settings.MM2.AuraRange then
                        targets[#targets + 1] = { char = tChar, hrp = tHRP, hum = tHum, dist = d }
                    end
                end
            end
        end
        if #targets == 0 then return end
        table.sort(targets, function(x, y) return x.dist < y.dist end)

        setNoclip(true)
        for _, t in ipairs(targets) do
            if not Settings.MM2.KillAuraEnabled or not myHRP.Parent then break end
            for attempt = 1, 4 do
                if not t.hrp.Parent or not t.hum.Parent or t.hum.Health <= 0 then break end
                if knife.Parent ~= myChar and hum then
                    hum:EquipTool(knife)
                end
                myHRP.CFrame = t.hrp.CFrame * CFrame.new(0, 0, 1.5)
                myHRP.AssemblyLinearVelocity = Vector3.zero
                stabTarget(knife, t.char, t.hrp)
                RunService.Heartbeat:Wait()
            end
        end

        if myHRP.Parent then
            myHRP.CFrame = origin
        end
        setNoclip(false)
    end)

    if not ok then
        warn("[NoniHub] KillAll erro: " .. tostring(err))
        setNoclip(false)
    end
    isKilling = false
    lastAttack = tick()
end

-- ================================================================
-- AUTO SHOOT MURDERER (TP atrás)
-- ================================================================
local lastGunShot = 0
local currentAimTarget = nil
local aimActive = false
local shiftLockActive = false
local oldRotationType = nil
local AIM_BIND = "NoniAimLock"

-- Liga o Shift Lock: o mouse fica travado no centro da tela,
-- entao onde a camera aponta e exatamente onde o tiro vai
local function enableShiftLock()
    if shiftLockActive then return end
    shiftLockActive = true
    pcall(function()
        local gs = UserSettings():GetService("UserGameSettings")
        oldRotationType = gs.RotationType
        gs.RotationType = Enum.RotationType.CameraRelative
    end)
    pcall(function()
        UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter
    end)
end

local function disableShiftLock()
    if not shiftLockActive then return end
    shiftLockActive = false
    pcall(function()
        local gs = UserSettings():GetService("UserGameSettings")
        gs.RotationType = oldRotationType or Enum.RotationType.MovementRelative
    end)
    pcall(function()
        UserInputService.MouseBehavior = Enum.MouseBehavior.Default
    end)
end

-- Mira no centro do corpo com uma pequena previsao de movimento
local function getAimPoint(part)
    return part.Position + part.AssemblyLinearVelocity * 0.08
end

local function startAimLoop()
    if aimActive then return end
    aimActive = true
    -- prioridade logo depois da camera padrao, pra ela nao sobrescrever a mira
    RunService:BindToRenderStep(AIM_BIND, Enum.RenderPriority.Camera.Value + 1, function()
        if not Settings.MM2.AutoKillMurderer then return end
        if not currentAimTarget or not currentAimTarget.Parent then
            currentAimTarget = nil
            return
        end

        pcall(function()
            UserInputService.MouseBehavior = Enum.MouseBehavior.LockCenter
        end)

        local camPos = Camera.CFrame.Position
        local newCF = CFrame.new(camPos, getAimPoint(currentAimTarget))
        if Settings.MM2.AimbotSmooth > 0 then
            Camera.CFrame = Camera.CFrame:Lerp(newCF, 1 - Settings.MM2.AimbotSmooth)
        else
            Camera.CFrame = newCF
        end
    end)
end

local function stopAimLoop()
    if aimActive then
        aimActive = false
        pcall(function()
            RunService:UnbindFromRenderStep(AIM_BIND)
        end)
    end
    currentAimTarget = nil
    disableShiftLock()
end

local function getGun()
    local myChar = LocalPlayer.Character
    if not myChar then return nil end
    local gun = myChar:FindFirstChild("Gun")
    if gun and gun:IsA("Tool") then return gun end
    local bp = LocalPlayer:FindFirstChild("Backpack")
    if bp then
        gun = bp:FindFirstChild("Gun")
        if gun and gun:IsA("Tool") then
            local hum = myChar:FindFirstChildOfClass("Humanoid")
            if hum then hum:EquipTool(gun) end
            return gun
        end
    end
    return nil
end

local gunRemotesPrinted = false
local function printGunRemotes(gun)
    if gunRemotesPrinted then return end
    gunRemotesPrinted = true
    local names = {}
    for _, d in ipairs(gun:GetDescendants()) do
        if d:IsA("RemoteEvent") or d:IsA("RemoteFunction") then
            names[#names + 1] = d:GetFullName() .. " (" .. d.ClassName .. ")"
        end
    end
    print("[NoniHub] Remotes da arma: " .. (#names > 0 and table.concat(names, " | ") or "nenhum"))
end

-- Dispara direto pro servidor com a posicao do alvo (sem mirar e sem teleportar)
local function shootMurdererRemote(gun, targetPart)
    local rf, re = nil, nil
    for _, d in ipairs(gun:GetDescendants()) do
        if d:IsA("RemoteFunction") and not rf then rf = d end
        if d:IsA("RemoteEvent") and not re and (d.Name == "Shoot" or d.Name == "ShootGun") then re = d end
    end
    if not rf and not re then return false end

    local pos = targetPart.Position
    if rf then
        task.spawn(function()
            pcall(function() rf:InvokeServer(1, pos, "AH2") end)
        end)
    end
    if re then
        pcall(function() re:FireServer(CFrame.new(pos), pos) end)
    end
    return true
end

local function runAutoKillMurdererInner()
    if not Settings.MM2.AutoKillMurderer then
        stopAimLoop()
        return
    end
    if tick() - lastGunShot < Settings.MM2.ShotCooldown then return end
    if not canRunMatchFunctions() then return end

    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return end
    local myHRP = myChar.HumanoidRootPart

    local gun = getGun()
    if not gun then
        stopAimLoop()
        return
    end
    if gun.Parent ~= myChar then
        local hum = myChar:FindFirstChildOfClass("Humanoid")
        if hum then hum:EquipTool(gun) end
        task.wait(0.12)
        if gun.Parent ~= myChar then return end
    end

    local targetMurderer = nil
    local closestDist = math.huge
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and getPlayerRole(plr) == "Murderer" and isPlayerInMap(plr) then
            local mChar = plr.Character
            if mChar then
                local mHRP = mChar:FindFirstChild("HumanoidRootPart")
                local mHum = mChar:FindFirstChildOfClass("Humanoid")
                if mHRP and mHum and mHum.Health > 0 then
                    local d = (mHRP.Position - myHRP.Position).Magnitude
                    if d < closestDist then
                        closestDist = d
                        targetMurderer = mChar
                    end
                end
            end
        end
    end
    if not targetMurderer then
        stopAimLoop()
        return
    end

    local mHRP = targetMurderer:FindFirstChild("HumanoidRootPart")
    if not mHRP then return end

    -- TIRO INSTANTANEO: manda direto pro servidor, sem mirar e sem teleportar
    printGunRemotes(gun)
    local fastHum = targetMurderer:FindFirstChildOfClass("Humanoid")
    for attempt = 1, 3 do
        if not fastHum or not mHRP.Parent or fastHum.Health <= 0 then break end
        if not shootMurdererRemote(gun, mHRP) then break end
        task.wait(0.1)
    end
    if not fastHum or not mHRP.Parent or fastHum.Health <= 0 then
        lastGunShot = tick()
        return
    end

    -- se nao matou, cai no metodo antigo: teleporta atras do murder e mira
    setNoclip(true)
    local mLook = mHRP.CFrame.LookVector
    local behindPos = mHRP.Position - mLook * Settings.MM2.TPBehindDistance
    myHRP.CFrame = CFrame.new(behindPos, mHRP.Position)
    RunService.Heartbeat:Wait()

    if not targetMurderer.Parent or not mHRP.Parent then
        stopAimLoop()
        setNoclip(false)
        return
    end
    local mHum = targetMurderer:FindFirstChildOfClass("Humanoid")
    if not mHum or mHum.Health <= 0 then
        stopAimLoop()
        setNoclip(false)
        return
    end

    -- mira no centro do corpo (hitbox maior que a cabeca) + Shift Lock ligado
    currentAimTarget = mHRP
    enableShiftLock()
    startAimLoop()

    -- espera a camera realmente alinhar com o murder antes de atirar
    local aimStart = tick()
    while tick() - aimStart < 0.5 do
        if not mHRP.Parent or not mHum.Parent or mHum.Health <= 0 then break end
        if not myHRP.Parent then break end

        -- continua colado atras do murder enquanto ele se mexe
        local look = mHRP.CFrame.LookVector
        local behind = mHRP.Position - look * Settings.MM2.TPBehindDistance
        myHRP.CFrame = CFrame.new(behind, mHRP.Position)

        RunService.RenderStepped:Wait()

        local toTarget = (getAimPoint(mHRP) - Camera.CFrame.Position)
        if toTarget.Magnitude > 0.1 then
            local dot = Camera.CFrame.LookVector:Dot(toTarget.Unit)
            if tick() - aimStart >= 0.12 and dot > 0.995 then
                break
            end
        end
    end

    if mHRP.Parent and mHum.Parent and mHum.Health > 0 then
        pcall(function()
            if gun:IsA("Tool") and gun.Parent == myChar then
                gun:Activate()
            end
        end)
        -- segura a mira um instante pro tiro registrar
        task.wait(0.15)
    end

    stopAimLoop()
    setNoclip(false)
    lastGunShot = tick()
end

-- Wrapper: impede varias execucoes ao mesmo tempo (o loop principal chama toda hora)
local isAutoShooting = false
local function runAutoKillMurderer()
    if isAutoShooting then return end
    isAutoShooting = true
    local ok, err = pcall(runAutoKillMurdererInner)
    if not ok then
        warn("[NoniHub] AutoShoot erro: " .. tostring(err))
        stopAimLoop()
        setNoclip(false)
    end
    isAutoShooting = false
end

-- ================================================================
-- AUTO GRAB GUN v5 (anti-parede)
-- ================================================================
local isGrabbingGun = false
local lastGrabAttempt = 0
local lastGunScan = 0
local cachedGunDrop = nil
local GUN_SCAN_INTERVAL = 1.5

-- Checa apenas se a gun esta dentro da faixa de altura do mapa.
-- (Filtros de chao/teto/parede foram removidos: bloqueavam guns validas.)
local function isGunAccessible(gunPart)
    if not gunPart or not gunPart.Parent then return false end
    local y = gunPart.Position.Y
    return y > -100 and y < 500
end

-- Mantida so por compatibilidade: com noclip nao precisa de caminho livre
local function hasPathToGun(fromPos, gunPos)
    return true
end

local function shallowFindGunDrops()
    local results = {}
    local function scan(container, depth)
        if depth > 3 then return end
        for _, obj in ipairs(container:GetChildren()) do
            local n = obj.Name:lower()
            if n == "gundrop" or n == "gun" or n:find("gundrop") then
                if obj:IsA("BasePart") then
                    if not isInsideAnyPlayer(obj) and isGunAccessible(obj) then
                        table.insert(results, obj)
                    end
                elseif obj:IsA("Model") then
                    if not isInsideAnyPlayer(obj) then
                        for _, p in ipairs(obj:GetDescendants()) do
                            if p:IsA("BasePart") and not isInsideAnyPlayer(p) and isGunAccessible(p) then
                                table.insert(results, p)
                            end
                        end
                    end
                end
            end
            if (obj:IsA("Folder") or obj:IsA("Model")) and depth < 3 then
                scan(obj, depth + 1)
            end
        end
    end
    scan(Workspace, 0)
    return results
end

local function getGunPosition(gunObj)
    if not gunObj then return nil end
    if gunObj:IsA("BasePart") then return gunObj.Position end
    if gunObj:IsA("Model") then
        if gunObj.PrimaryPart then return gunObj.PrimaryPart.Position end
        local part = gunObj:FindFirstChildWhichIsA("BasePart", true)
        if part then return part.Position end
    end
    return nil
end

local function findClosestGunDrop()
    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return nil, nil end
    local myHRP = myChar.HumanoidRootPart

    -- caminho rapido: no MM2 a gun caida se chama "GunDrop" direto no Workspace
    local direct = Workspace:FindFirstChild("GunDrop")
    if direct and not isInsideAnyPlayer(direct) then
        local pos = getGunPosition(direct)
        if pos then
            return direct, (pos - myHRP.Position).Magnitude
        end
    end

    -- fallback: varredura rasa
    local closestGun, closestDist = nil, math.huge
    for _, obj in ipairs(shallowFindGunDrops()) do
        if not obj:IsDescendantOf(myChar) and not isInsideAnyPlayer(obj) then
            local d = (obj.Position - myHRP.Position).Magnitude
            if d < closestDist then
                closestDist = d
                closestGun = obj
            end
        end
    end
    return closestGun, closestDist
end

local function hasMyGun(myChar)
    if myChar and myChar:FindFirstChild("Gun") then return true end
    local bp = LocalPlayer:FindFirstChild("Backpack")
    return bp ~= nil and bp:FindFirstChild("Gun") ~= nil
end

local lastGrabNotify = 0
local function grabNotify(text)
    if tick() - lastGrabNotify < 3 then return end
    lastGrabNotify = tick()
    notify("Noni Hub", text, 2)
end

local function runAutoGrabGun()
    if not Settings.MM2.AutoGrabGun or isGrabbingGun then return end
    if tick() - lastGrabAttempt < 0.5 then return end
    if not isAlive() then return end

    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return end
    if getPlayerRole(LocalPlayer) == "Murderer" then return end
    if hasMyGun(myChar) then return end

    if tick() - lastGunScan >= GUN_SCAN_INTERVAL then
        lastGunScan = tick()
        cachedGunDrop = findClosestGunDrop()
    end

    local gunDrop = cachedGunDrop
    if not gunDrop or not gunDrop.Parent then
        cachedGunDrop = nil
        -- rescan rapido quando a gun some do cache
        lastGunScan = 0
        return
    end
    if isInsideAnyPlayer(gunDrop) then
        cachedGunDrop = nil
        return
    end

    local gunPos = getGunPosition(gunDrop)
    if not gunPos then
        cachedGunDrop = nil
        return
    end

    local hrp = myChar.HumanoidRootPart
    local dist = (gunPos - hrp.Position).Magnitude
    if dist > Settings.MM2.GrabGunRange then return end

    lastGrabAttempt = tick()
    isGrabbingGun = true
    grabNotify("Gun encontrada, pegando...")

    setNoclip(true)
    local oldCFrame = hrp.CFrame
    local targetCF = CFrame.new(gunPos + Vector3.new(0, 0.5, 0))

    -- fica "preso" em cima da gun (sem cair pelo chao por causa do noclip) ate pegar
    local grabbed = false
    local startT = tick()
    local lastTouch = 0
    while tick() - startT < 1.5 do
        if not myChar.Parent or not hrp.Parent then break end
        if hasMyGun(myChar) then grabbed = true break end
        if not gunDrop.Parent then grabbed = true break end
        if isInsideAnyPlayer(gunDrop) then grabbed = true break end

        -- acompanha a gun caso ela se mexa
        local curPos = getGunPosition(gunDrop)
        if curPos then
            targetCF = CFrame.new(curPos + Vector3.new(0, 0.5, 0))
        end
        hrp.CFrame = targetCF
        hrp.AssemblyLinearVelocity = Vector3.zero

        -- reforca o toque se o executor tiver firetouchinterest
        if firetouchinterest and gunDrop:IsA("BasePart") and tick() - lastTouch > 0.15 then
            lastTouch = tick()
            pcall(function()
                firetouchinterest(hrp, gunDrop, 0)
                firetouchinterest(hrp, gunDrop, 1)
            end)
        end

        task.wait()
    end

    if grabbed then
        grabNotify("Gun pega!")
    else
        grabNotify("Nao consegui pegar a gun")
        if myChar.Parent and hrp.Parent then
            hrp.CFrame = oldCFrame
        end
        cachedGunDrop = nil
    end

    setNoclip(false)
    isGrabbingGun = false
end

-- ================================================================
-- FLEE MODE v2
-- ================================================================
local lastFlee = 0
local fleeingActive = false

local function isMurdererLookingAtMe(murdererChar, myChar)
    local mHRP = murdererChar:FindFirstChild("HumanoidRootPart")
    local mHead = murdererChar:FindFirstChild("Head")
    local myHRP = myChar:FindFirstChild("HumanoidRootPart")
    if not mHRP or not myHRP then return false end
    local lookPart = mHead or mHRP
    local lookDir = lookPart.CFrame.LookVector
    local toMe = (myHRP.Position - lookPart.Position)
    if toMe.Magnitude < 1 then return true end
    toMe = toMe.Unit
    return lookDir:Dot(toMe) > 0.5
end

local function isSafePosition(pos)
    if pos.Y < -50 or pos.Y > 500 then return false end
    local rayParams = RaycastParams.new()
    rayParams.FilterType = Enum.RaycastFilterType.Exclude
    rayParams.FilterDescendantsInstances = { LocalPlayer.Character }
    local result = Workspace:Raycast(pos, Vector3.new(0, -30, 0), rayParams)
    return result ~= nil
end

local function findEscapePoint(myHRP, murdererChar)
    local mHRP = murdererChar:FindFirstChild("HumanoidRootPart")
    if not mHRP then return nil end

    local away = (myHRP.Position - mHRP.Position)
    if away.Magnitude < 1 then away = Vector3.new(1, 0, 0) end
    away = Vector3.new(away.X, 0, away.Z).Unit

    local right = Vector3.new(-away.Z, 0, away.X)
    local candidates = {
        away,
        (away + right * 0.5).Unit,
        (away - right * 0.5).Unit,
        (away + right).Unit,
        (away - right).Unit,
    }

    local fleeRange = Settings.MM2.FleeSearchRange or 120
    for _, dir in ipairs(candidates) do
        local testPos = myHRP.Position + dir * fleeRange
        if isSafePosition(testPos) then
            return testPos
        end
    end
    return nil
end

RunService.Heartbeat:Connect(function()
    if not Settings.MM2.FleeMode then
        fleeingActive = false
        return
    end
    if not canRunMatchFunctions() then return end
    if tick() - lastFlee < (Settings.MM2.FleeCooldown or 2.5) then return end
    if hasGun() then return end

    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return end
    local myHRP = myChar.HumanoidRootPart

    local murderer, dist = getClosestMurderer()
    if not murderer or not dist then return end

    if dist > (Settings.MM2.FleeDistance or 40) then
        fleeingActive = false
        return
    end
    if not isMurdererLookingAtMe(murderer, myChar) then return end

    local safeTarget = findSafeFleeTarget(murderer)
    if safeTarget then
        local targetHRP = safeTarget:FindFirstChild("HumanoidRootPart")
        if targetHRP and isSafePosition(targetHRP.Position) then
            setNoclip(true)
            myHRP.CFrame = CFrame.new(targetHRP.Position + Vector3.new(3, 0, 3))
            task.delay(0.3, function() setNoclip(false) end)
            lastFlee = tick()
            fleeingActive = true
            return
        end
    end

    local escapePos = findEscapePoint(myHRP, murderer)
    if escapePos then
        setNoclip(true)
        myHRP.CFrame = CFrame.new(escapePos + Vector3.new(0, 5, 0))
        task.delay(0.3, function() setNoclip(false) end)
        lastFlee = tick()
        fleeingActive = true
    end
end)

-- ================================================================
-- AUTO COIN v15 (TWEEN LINEAR - velocidade estavel)
-- ================================================================
local isCollectingCoin = false
local lastCoinScan = 0
local failedCoins = {}

-- Configs (ajuste aqui)
local COIN_SPEED = 28          -- studs por segundo (WalkSpeed padrao = 16; se kickar, baixe para 22 ou 18)
local COIN_TOUCH_WAIT = 0.2    -- tempo parado na moeda pra coletar
local COIN_GAP = 0.1
local COIN_MAX_DISTANCE = 300
local COIN_MIN_Y = -20
local COIN_HEIGHT_OFFSET = 1

local function coinStillExists(coin)
    return coin and coin.Parent ~= nil
end

local function isCoinSafe(coin)
    if not coin or not coin.Parent then return false end
    local pos = coin.Position
    if pos.Y < COIN_MIN_Y or pos.Y > 500 then return false end
    local rayParams = RaycastParams.new()
    rayParams.FilterType = Enum.RaycastFilterType.Exclude
    rayParams.FilterDescendantsInstances = { LocalPlayer.Character, coin }
    local result = Workspace:Raycast(pos, Vector3.new(0, -60, 0), rayParams)
    if not result then return false end
    if (result.Position - pos).Magnitude > 25 then return false end
    return true
end

-- Tween linear: tempo = distancia / velocidade (velocidade sempre constante)
local function tweenTo(targetPos, coin)
    local myChar = LocalPlayer.Character
    if not myChar then return false end
    local hrp = myChar:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end

    local dist = (targetPos - hrp.Position).Magnitude
    if dist < 1 then return true end

    local duration = dist / COIN_SPEED
    local tween = TweenService:Create(
        hrp,
        TweenInfo.new(duration, Enum.EasingStyle.Linear, Enum.EasingDirection.Out),
        { CFrame = CFrame.new(targetPos) * hrp.CFrame.Rotation }
    )

    local finished = false
    local conn = tween.Completed:Connect(function()
        finished = true
    end)
    tween:Play()

    while not finished do
        if not Settings.MM2.AutoCoinCollector
            or not hrp.Parent
            or hasWeaponEquipped()
            or (coin and not coinStillExists(coin)) then
            tween:Cancel()
            break
        end
        -- evita a fisica empurrar/derrubar o personagem durante o tween
        hrp.AssemblyLinearVelocity = Vector3.zero
        task.wait()
    end

    conn:Disconnect()
    return finished
end

local function shallowFindCoins()
    local results = {}
    local function scan(container, depth)
        if depth > 4 then return end
        for _, obj in ipairs(container:GetChildren()) do
            if obj:IsA("BasePart") then
                local n = obj.Name:lower()
                if n:find("coin") and not n:find("gui") then
                    table.insert(results, obj)
                end
            elseif obj:IsA("Folder") or obj:IsA("Model") then
                scan(obj, depth + 1)
            end
        end
    end
    scan(Workspace, 0)
    return results
end

local function runAutoCoinCollector()
    if not Settings.MM2.AutoCoinCollector then
        if isCollectingCoin then
            isCollectingCoin = false
            setNoclip(false)
        end
        return
    end

    if isCollectingCoin then return end
    if tick() - lastCoinScan < Settings.MM2.CoinScanInterval then return end
    if hasWeaponEquipped() then return end

    lastCoinScan = tick()

    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return end
    local hum = myChar:FindFirstChildOfClass("Humanoid")
    if not hum or hum.Health <= 0 then return end

    local myHRP = myChar.HumanoidRootPart
    local originPos = myHRP.Position

    local coins = {}
    for _, obj in ipairs(shallowFindCoins()) do
        if not failedCoins[obj] then
            local dist = (obj.Position - myHRP.Position).Magnitude
            if dist <= COIN_MAX_DISTANCE and isCoinSafe(obj) then
                table.insert(coins, obj)
            end
        end
    end

    if #coins == 0 then
        failedCoins = {}
        return
    end

    table.sort(coins, function(a, b)
        return (a.Position - myHRP.Position).Magnitude < (b.Position - myHRP.Position).Magnitude
    end)

    isCollectingCoin = true
    setNoclip(true)

    for _, coin in ipairs(coins) do
        if not Settings.MM2.AutoCoinCollector then break end
        if not myChar.Parent or not myHRP.Parent then break end
        if hasWeaponEquipped() then break end

        if coinStillExists(coin) then
            local p = coin.Position
            tweenTo(Vector3.new(p.X, p.Y + COIN_HEIGHT_OFFSET, p.Z), coin)

            if coinStillExists(coin) then
                task.wait(COIN_TOUCH_WAIT)
            end
            if coinStillExists(coin) then
                failedCoins[coin] = true
                task.wait(COIN_GAP)
            end
        end
    end

    -- volta pro ponto de origem, tambem por tween
    if myChar.Parent and myHRP.Parent and Settings.MM2.AutoCoinCollector then
        tweenTo(originPos, nil)
    end

    isCollectingCoin = false
    setNoclip(false)
end

-- ================================================================
-- GUN ESP
-- ================================================================
local lastGunESPScan = 0
local GUN_ESP_SCAN_INTERVAL = 0.5
local gunESPObjects = {}

local function applyGunHighlight(part)
    if not part or not part.Parent then return end
    if part:IsDescendantOf(LocalPlayer.Character) then return end
    if isInsideAnyPlayer(part) then return end

    local hl = part:FindFirstChild("NoniGunESP")
    if not hl then
        hl = Instance.new("Highlight")
        hl.Name = "NoniGunESP"
        hl.FillColor = Color3.fromRGB(255, 220, 0)
        hl.FillTransparency = 0.3
        hl.OutlineColor = Color3.fromRGB(255, 255, 255)
        hl.OutlineTransparency = 0
        hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        hl.Parent = part
    end
    hl.Enabled = true

    local billboard = part:FindFirstChild("NoniGunBillboard")
    if not billboard then
        billboard = Instance.new("BillboardGui")
        billboard.Name = "NoniGunBillboard"
        billboard.Size = UDim2.new(0, 100, 0, 30)
        billboard.StudsOffset = Vector3.new(0, 2, 0)
        billboard.AlwaysOnTop = true
        billboard.MaxDistance = 500
        billboard.Parent = part

        local label = Instance.new("TextLabel")
        label.Size = UDim2.new(1, 0, 1, 0)
        label.BackgroundTransparency = 1
        label.Text = "GUN"
        label.TextColor3 = Color3.fromRGB(255, 220, 0)
        label.TextStrokeTransparency = 0
        label.TextStrokeColor3 = Color3.fromRGB(0, 0, 0)
        label.Font = Enum.Font.GothamBold
        label.TextScaled = true
        label.Parent = billboard
    end
    billboard.Enabled = true
end

local function disableGunESP(part)
    if not part or not part.Parent then return end
    local hl = part:FindFirstChild("NoniGunESP")
    if hl then hl.Enabled = false end
    local bb = part:FindFirstChild("NoniGunBillboard")
    if bb then bb.Enabled = false end
end

local function updateGunESP()
    if not Settings.MM2.GunESPEnabled then
        if next(gunESPObjects) then
            for obj, _ in pairs(gunESPObjects) do
                disableGunESP(obj)
            end
            gunESPObjects = {}
        end
        return
    end

    if tick() - lastGunESPScan < GUN_ESP_SCAN_INTERVAL then return end
    lastGunESPScan = tick()

    local newGuns = shallowFindGunDrops()
    local newSet = {}

    for _, part in ipairs(newGuns) do
        newSet[part] = true
        applyGunHighlight(part)
    end

    for obj, _ in pairs(gunESPObjects) do
        if not newSet[obj] then
            disableGunESP(obj)
        end
    end

    gunESPObjects = newSet
end

-- ==================== LOOP PRINCIPAL ====================

for _, plr in ipairs(Players:GetPlayers()) do
    applyHighlight(plr)
end

Players.PlayerAdded:Connect(function(plr)
    applyHighlight(plr)
end)

RunService.Heartbeat:Connect(function()
    if Settings.MM2.ESPEnabled then
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character then
                local char = plr.Character
                local hl = char:FindFirstChild("NoniESP")
                local role = getPlayerRole(plr)
                if not hl then
                    hl = Instance.new("Highlight")
                    hl.Name = "NoniESP"
                    hl.FillTransparency = 0.4
                    hl.OutlineTransparency = 0
                    hl.Parent = char
                end
                hl.FillColor = Settings.Colors[role] or Settings.Colors.Innocent
                hl.OutlineColor = hl.FillColor
                hl.Enabled = true
            end
        end
    else
        for _, plr in ipairs(Players:GetPlayers()) do
            if plr ~= LocalPlayer and plr.Character then
                local hl = plr.Character:FindFirstChild("NoniESP")
                if hl then hl.Enabled = false end
            end
        end
    end

    runKillAura()
    task.spawn(runAutoKillMurderer)
    runAutoGrabGun()
    runAutoCoinCollector()
    updateGunESP()
end)

-- Skins: loop separado e protegido, pra nao depender das outras funcoes
RunService.Heartbeat:Connect(function()
    pcall(updateSkins)
    pcall(syncDisplaySkins)
    pcall(updateCapture)
end)

-- Execucao
createIntroScreen()
task.delay(3.5, function()
    createMainUI()
    notify("Noni Hub", "MM2 Edition carregado com sucesso!", 5)
end)
