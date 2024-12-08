_G.ConfigAttack = true
local ActiveRigs, TargetRoots = {}, {}
require(game.ReplicatedStorage.Util.CameraShaker):Stop()
task.spawn(function()
    while task.wait() do
        pcall(function()
            table.clear(ActiveRigs)
            table.clear(TargetRoots)
            for _, rig in ipairs(game:GetService("CollectionService"):GetTagged("ActiveRig")) do
                local humanoid = rig:FindFirstChildOfClass("Humanoid")
                if humanoid and humanoid.Health > 0 and humanoid.RootPart and rig ~= game.Players.LocalPlayer.Character then
                    local player = game.Players:GetPlayerFromCharacter(rig)
                    local isAlly = player and game:GetService("CollectionService"):HasTag(player, "Ally" .. game.Players.LocalPlayer.Name)

                    if not isAlly then
                        table.insert(ActiveRigs, rig)

                        if (rig.HumanoidRootPart.Position - game.Players.LocalPlayer.Character.HumanoidRootPart.Position).Magnitude <= 65 then
                            table.insert(TargetRoots, humanoid.RootPart)
                        end
                    end
                end
            end
        end)
    end
end)
task.spawn(function()
    pcall(function()
        local combatFramework = require(game.Players.LocalPlayer.PlayerScripts.CombatFramework)
        local activeController = getupvalue(combatFramework, 2)
        local animation = Instance.new("Animation")

        local lastAttackTime, attackCount, maxAttacks = 0, 0, 200
        if not shared.orl then
            shared.orl = require(game:GetService("ReplicatedStorage").CombatFramework.RigLib).wrapAttackAnimationAsync
        end

        if not shared.cpc then
            shared.cpc = require(game.Players.LocalPlayer.PlayerScripts.CombatFramework.Particle).play
        end

        if not shared.attack then
            shared.attack = getupvalue(require(game.Players.LocalPlayer.PlayerScripts.CombatFramework.RigController), 2).attack
        end

        
        require(game:GetService("ReplicatedStorage").CombatFramework.RigLib).wrapAttackAnimationAsync = function(anim, target, hitbox, extra, callback)
            if #TargetRoots > 0 then
                require(game.Players.LocalPlayer.PlayerScripts.CombatFramework.Particle).play = function() end
                anim:Play()
                callback(TargetRoots)
                task.wait(anim.length * 0.1)
                anim:Stop()
            end
        end

        while game:GetService("RunService").Stepped:Wait() do
            if #TargetRoots > 0 then
                local controller = activeController.activeController
                if controller and controller.equipped and controller.currentWeaponModel then
                    if tick() > lastAttackTime then
                        local weaponName = controller.currentWeaponModel.Name
                        local attackDelays = { combat = 0.01 }
                        lastAttackTime = tick() + (attackDelays[weaponName:lower()] or 0.1) + (attackCount / maxAttacks) * 0.1

                
                        game:GetService("ReplicatedStorage").RigControllerEvent.FireServer(
                            game:GetService("ReplicatedStorage").RigControllerEvent,
                            "weaponChange",
                            weaponName
                        )

                        attackCount += 1
                        task.delay((attackDelays[weaponName:lower()] or 0.1) + (attackCount + 0.1 / maxAttacks) * 0.1, function()
                            attackCount -= 1
                        end)
                    end

                    if tick() - maxAttacks > 0.1 then
                        controller.timeToNextAttack = 0
                        controller.hitboxMagnitude = 65
                        pcall(task.spawn, controller.attack, controller)
                        maxAttacks = tick()
                    end

                    local animId = controller.anims.basic[3] or controller.anims.basic[2] or nil
                    animation.AnimationId = animId
                    local loadedAnim = controller.humanoid:LoadAnimation(animation)
                    loadedAnim:Play()

                    game:GetService("ReplicatedStorage").RigControllerEvent.FireServer(
                        game:GetService("ReplicatedStorage").RigControllerEvent,
                        "hit",
                        TargetRoots,
                        animId and 3 or 2 or nil,
                        ""
                    )

                    delay(0.1, function()
                        loadedAnim:Stop()
                    end)
                end
            end
        end
    end)
end)