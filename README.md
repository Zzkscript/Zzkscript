local flying = false
local player = game.Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoid = character:WaitForChild("Humanoid")
local bodyVelocity = nil
local bodyGyro = nil

local pressCount = 0 -- Compte le nombre de fois que F1 est appuyé
local flySpeed = 50 -- Vitesse initiale du vol
local UIS = game:GetService("UserInputService")

local function startFlying()
    if not flying then
        flying = true
        
        -- Créer des objets pour contrôler le mouvement
        bodyVelocity = Instance.new("BodyVelocity")
        bodyVelocity.Velocity = Vector3.new(0, 0, 0)
        bodyVelocity.MaxForce = Vector3.new(100000, 100000, 100000)
        bodyVelocity.P = 1250
        bodyVelocity.Parent = character.HumanoidRootPart
        
        bodyGyro = Instance.new("BodyGyro")
        bodyGyro.MaxTorque = Vector3.new(100000, 100000, 100000)
        bodyGyro.CFrame = character.HumanoidRootPart.CFrame
        bodyGyro.P = 3000
        bodyGyro.Parent = character.HumanoidRootPart
    end
end

local function stopFlying()
    if flying then
        flying = false
        bodyVelocity:Destroy()
        bodyGyro:Destroy()
    end
end

-- Gestion de l'appui sur F1 pour activer/désactiver le vol
UIS.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.F1 then
        pressCount = pressCount + 1
        if pressCount == 1 then
            startFlying()
        elseif pressCount == 2 then
            stopFlying()
            pressCount = 0 -- Réinitialise le compte pour appuyer deux fois
        end
    end
    
    -- Augmenter la vitesse avec "+" du pavé numérique
    if input.KeyCode == Enum.KeyCode.KeypadPlus then
        flySpeed = flySpeed + 10 -- Augmente la vitesse de 10
        print("Vitesse actuelle : " .. flySpeed)
    end
    
    -- Diminuer la vitesse avec "-" du pavé numérique
    if input.KeyCode == Enum.KeyCode.KeypadMinus then
        flySpeed = flySpeed - 10 -- Diminue la vitesse de 10
        if flySpeed < 0 then flySpeed = 0 end -- Empêche la vitesse d'être négative
        print("Vitesse actuelle : " .. flySpeed)
    end
end)

-- Bouger le personnage en vol selon les touches pressées
game:GetService("RunService").RenderStepped:Connect(function()
    if flying then
        local camera = workspace.CurrentCamera
        local moveDirection = Vector3.new(0, 0, 0)
        
        if UIS:IsKeyDown(Enum.KeyCode.Z) then
            moveDirection = moveDirection + (camera.CFrame.LookVector)
        end
        if UIS:IsKeyDown(Enum.KeyCode.S) then
            moveDirection = moveDirection - (camera.CFrame.LookVector)
        end
        if UIS:IsKeyDown(Enum.KeyCode.Q) then
            moveDirection = moveDirection - (camera.CFrame.RightVector)
        end
        if UIS:IsKeyDown(Enum.KeyCode.D) then
            moveDirection = moveDirection + (camera.CFrame.RightVector)
        end
        
        bodyVelocity.Velocity = moveDirection * flySpeed -- Utilise la vitesse actuelle
        bodyGyro.CFrame = camera.CFrame -- Garder le personnage aligné avec la caméra
    end
end)
