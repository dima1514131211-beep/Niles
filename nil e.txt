print("[Oblivion] Начинаем загрузку...")
repo = "https://cdn.jsdelivr.net/gh/deividcomsono/Obsidian@main/"
Library, ThemeManager, SaveManager = nil, nil, nil
libDone, themeDone, saveDone = false, false, false
task.spawn(function()
print("[Oblivion] Скачиваем Library с GitHub...")
local success, res = pcall(function()
local src = game:HttpGet(repo .. "Library.lua")
local function plainReplace(s, old, new)
local idx = string.find(s, old, 1, true)
if idx then
return string.sub(s, 1, idx - 1) .. new .. string.sub(s, idx + #old)
end
return s
end
src = plainReplace(src,
'[Enum.UserInputType.MouseButton3] = "MB3",\n        }\n        pcall(function()\n            SpecialKeys["MB4"] = Enum.KeyCode.ButtonX1\n            SpecialKeys["MB5"] = Enum.KeyCode.ButtonX2\n            SpecialKeysInput[Enum.KeyCode.ButtonX1] = "MB4"\n            SpecialKeysInput[Enum.KeyCode.ButtonX2] = "MB5"\n        end)\n\n        -- Modifiers')
src = plainReplace(src,
'if SpecialKeysInput[InputObj.UserInputType] ~= nil then\n                    KeyName = SpecialKeysInput[InputObj.UserInputType]\n                elseif InputObj.UserInputType == Enum.UserInputType.Keyboard then',
'if SpecialKeysInput[InputObj.UserInputType] ~= nil then\n'
..'                    KeyName = SpecialKeysInput[InputObj.UserInputType]\n'
..'                elseif SpecialKeysInput[InputObj.KeyCode] ~= nil then\n'
..'                    KeyName = SpecialKeysInput[InputObj.KeyCode]\n'
..'                elseif InputObj.UserInputType == Enum.UserInputType.Keyboard then')
src = plainReplace(src,
'if SpecialKeys[Key] ~= nil then\n                    if Library.Toggled then\n                        return false\n                    end\n\n                    return UserInputService:IsMouseButtonPressed(SpecialKeys[Key])\n                        and not UserInputService:GetFocusedTextBox()',
'if SpecialKeys[Key] ~= nil then\n'
..'                    if Library.Toggled then\n                        return false\n                    end\n'
..'                    if (Key == "MB4" or Key == "MB5") and SpecialKeys[Key] then\n'
..'                        return UserInputService:IsKeyDown(SpecialKeys[Key])\n'
..'                            and not UserInputService:GetFocusedTextBox()\n'
..'                    end\n'
..'                    return UserInputService:IsMouseButtonPressed(SpecialKeys[Key])\n'
..'                        and not UserInputService:GetFocusedTextBox()')
src = plainReplace(src,
'if SpecialKeysInput[CurrentInput.UserInputType] ~= nil then\n                Key = SpecialKeysInput[CurrentInput.UserInputType]\n            elseif CurrentInput.UserInputType == Enum.UserInputType.Keyboard then',
'if SpecialKeysInput[CurrentInput.UserInputType] ~= nil then\n'
..'                Key = SpecialKeysInput[CurrentInput.UserInputType]\n'
..'            elseif SpecialKeysInput[CurrentInput.KeyCode] ~= nil then\n'
..'                Key = SpecialKeysInput[CurrentInput.KeyCode]\n'
..'            elseif CurrentInput.UserInputType == Enum.UserInputType.Keyboard then')
src = plainReplace(src,
'SpecialKeysInput[Input.UserInputType] == Key\n                    or (Input.UserInputType == Enum.UserInputType.Keyboard and Input.KeyCode.Name == Key)',
'SpecialKeysInput[Input.UserInputType] == Key\n'
..'                    or SpecialKeysInput[Input.KeyCode] == Key\n'
..'                    or (Input.UserInputType == Enum.UserInputType.Keyboard and Input.KeyCode.Name == Key)')
local srcLenP7 = #src
  src = plainReplace(src, [[
        local Label = New("TextLabel", {
            BackgroundTransparency = 1,
            Size = UDim2.new(1, -40, 1, 0),
            Text = Toggle.Text,
            TextSize = 14,
            TextTransparency = 0.4,
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = Button,
        })

        New("UIListLayout", {
            FillDirection = Enum.FillDirection.Horizontal,
            HorizontalAlignment = Enum.HorizontalAlignment.Right,
            Padding = UDim.new(0, 6),
            Parent = Label,
        })

        local Switch = New("Frame", {
            AnchorPoint = Vector2.new(1, 0),
            BackgroundColor3 = "MainColor",
            Position = UDim2.fromScale(1, 0),
            Size = UDim2.fromOffset(32, 18),
            Parent = Button,
        })
        New("UICorner", {
            CornerRadius = UDim.new(1, 0),
            Parent = Switch,
        })
        New("UIPadding", {
            PaddingBottom = UDim.new(0, 2),
            PaddingLeft = UDim.new(0, 2),
            PaddingRight = UDim.new(0, 2),
            PaddingTop = UDim.new(0, 2),
            Parent = Switch,
        })
        local SwitchStroke = New("UIStroke", {
            Color = "OutlineColor",
            Parent = Switch,
        })

        local Ball = New("Frame", {
            BackgroundColor3 = "FontColor",
            Size = UDim2.fromScale(1, 1),
            SizeConstraint = Enum.SizeConstraint.RelativeYY,
            Parent = Switch,
        })
        New("UICorner", {
            CornerRadius = UDim.new(1, 0),
            Parent = Ball,
        })

        function Toggle:UpdateColors()
            Toggle:Display()
        end

        function Toggle:Display()
            if Library.Unloaded then
                return
            end

            local Offset = Toggle.Value and 1 or 0

            Switch.BackgroundTransparency = Toggle.Disabled and 0.75 or 0
            SwitchStroke.Transparency = Toggle.Disabled and 0.75 or 0

            Switch.BackgroundColor3 = Toggle.Value and Library.Scheme.AccentColor or Library.Scheme.MainColor
            SwitchStroke.Color = Toggle.Value and Library.Scheme.AccentColor or Library.Scheme.OutlineColor

            Library.Registry[Switch].BackgroundColor3 = Toggle.Value and "AccentColor" or "MainColor"
            Library.Registry[SwitchStroke].Color = Toggle.Value and "AccentColor" or "OutlineColor"

            if Toggle.Disabled then
                Label.TextTransparency = 0.8
                Ball.AnchorPoint = Vector2.new(Offset, 0)
                Ball.Position = UDim2.fromScale(Offset, 0)

                Ball.BackgroundColor3 = Library:GetDarkerColor(Library.Scheme.FontColor)
                Library.Registry[Ball].BackgroundColor3 = function()
                    return Library:GetDarkerColor(Library.Scheme.FontColor)
                end

                return
            end

            TweenService:Create(Label, Library.TweenInfo, {
                TextTransparency = Toggle.Value and 0 or 0.4,
            }):Play()
            TweenService:Create(Ball, Library.TweenInfo, {
                AnchorPoint = Vector2.new(Offset, 0),
                Position = UDim2.fromScale(Offset, 0),
            }):Play()

            Ball.BackgroundColor3 = Library.Scheme.FontColor
            Library.Registry[Ball].BackgroundColor3 = "FontColor"
        end]], [[
        local Label = New("TextLabel", {
            BackgroundTransparency = 1,
            Size = UDim2.new(1, -46, 1, 0),
            Text = Toggle.Text,
            TextSize = 14,
            TextTransparency = 0.4,
            TextXAlignment = Enum.TextXAlignment.Left,
            Parent = Button,
        })

        New("UIListLayout", {
            FillDirection = Enum.FillDirection.Horizontal,
            HorizontalAlignment = Enum.HorizontalAlignment.Right,
            Padding = UDim.new(0, 6),
            Parent = Label,
        })

        local Switch = New("Frame", {
            AnchorPoint = Vector2.new(1, 0),
            BackgroundColor3 = "MainColor",
            Position = UDim2.fromScale(1, 0),
            Size = UDim2.fromOffset(38, 18),
            Parent = Button,
        })
        New("UICorner", {
            CornerRadius = UDim.new(1, 0),
            Parent = Switch,
        })
        New("UIPadding", {
            PaddingBottom = UDim.new(0, 2),
            PaddingLeft = UDim.new(0, 2),
            PaddingRight = UDim.new(0, 2),
            PaddingTop = UDim.new(0, 2),
            Parent = Switch,
        })
        local SwitchStroke = New("UIStroke", {
            Color = "OutlineColor",
            Parent = Switch,
        })

        -- Мягкое акцентное свечение пилюли, когда тогл включён
        local SwitchGlow = New("UIStroke", {
            Color = "AccentColor",
            Thickness = 3,
            Transparency = 1,
            Parent = Switch,
        })

        -- Лёгкий вертикальный градиент — объём пилюли
        New("UIGradient", {
            Rotation = 90,
            Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
                ColorSequenceKeypoint.new(1, Color3.fromRGB(212, 212, 212)),
            }),
            Parent = Switch,
        })

        local Ball = New("Frame", {
            BackgroundColor3 = "FontColor",
            Size = UDim2.fromScale(1, 1),
            SizeConstraint = Enum.SizeConstraint.RelativeYY,
            Parent = Switch,
        })
        New("UICorner", {
            CornerRadius = UDim.new(1, 0),
            Parent = Ball,
        })
        -- Градиент шарика — имитация мягкой тени снизу
        New("UIGradient", {
            Rotation = 90,
            Color = ColorSequence.new({
                ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
                ColorSequenceKeypoint.new(1, Color3.fromRGB(219, 219, 219)),
            }),
            Parent = Ball,
        })

        local ToggleTweenInfo = TweenInfo.new(0.22, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
        local PressTweenInfo = TweenInfo.new(0.12, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)

        function Toggle:UpdateColors()
            Toggle:Display()
        end

        function Toggle:Display()
            if Library.Unloaded then
                return
            end

            local Offset = Toggle.Value and 1 or 0

            Switch.BackgroundTransparency = Toggle.Disabled and 0.75 or 0

            Switch.BackgroundColor3 = Toggle.Value and Library.Scheme.AccentColor or Library.Scheme.MainColor
            SwitchStroke.Color = Toggle.Value and Library.Scheme.AccentColor or Library.Scheme.OutlineColor

            Library.Registry[Switch].BackgroundColor3 = Toggle.Value and "AccentColor" or "MainColor"
            Library.Registry[SwitchStroke].Color = Toggle.Value and "AccentColor" or "OutlineColor"

            if Toggle.Disabled then
                SwitchStroke.Transparency = 0.75
                SwitchGlow.Transparency = 1

                Label.TextTransparency = 0.8
                Ball.AnchorPoint = Vector2.new(Offset, 0)
                Ball.Position = UDim2.fromScale(Offset, 0)

                Ball.BackgroundColor3 = Library:GetDarkerColor(Library.Scheme.FontColor)
                Library.Registry[Ball].BackgroundColor3 = function()
                    return Library:GetDarkerColor(Library.Scheme.FontColor)
                end

                return
            end

            TweenService:Create(Label, Library.TweenInfo, {
                TextTransparency = Toggle.Value and 0 or 0.4,
            }):Play()
            TweenService:Create(Ball, ToggleTweenInfo, {
                AnchorPoint = Vector2.new(Offset, 0),
                Position = UDim2.fromScale(Offset, 0),
            }):Play()
            TweenService:Create(Switch, ToggleTweenInfo, {
                BackgroundColor3 = Toggle.Value and Library.Scheme.AccentColor or Library.Scheme.MainColor,
            }):Play()
            TweenService:Create(SwitchStroke, ToggleTweenInfo, {
                Transparency = Toggle.Value and 1 or 0,
            }):Play()
            TweenService:Create(SwitchGlow, ToggleTweenInfo, {
                Transparency = Toggle.Value and 0.45 or 1,
            }):Play()

            Ball.BackgroundColor3 = Library.Scheme.FontColor
            Library.Registry[Ball].BackgroundColor3 = "FontColor"
        end

        -- Наведение: пилюля мягко подсвечивается акцентом
        table.insert(Toggle.Connections, Button.MouseEnter:Connect(function()
            if Toggle.Disabled or Library.Unloaded then
                return
            end

            TweenService:Create(SwitchGlow, ToggleTweenInfo, {
                Transparency = Toggle.Value and 0.2 or 0.55,
            }):Play()
        end))
        table.insert(Toggle.Connections, Button.MouseLeave:Connect(function()
            if Library.Unloaded then
                return
            end

            TweenService:Create(Ball, PressTweenInfo, {
                Size = UDim2.fromScale(1, 1),
            }):Play()
            TweenService:Create(SwitchGlow, ToggleTweenInfo, {
                Transparency = (Toggle.Disabled or not Toggle.Value) and 1 or 0.45,
            }):Play()
        end))
        -- Нажатие: шарик слегка сжимается — тактильный отклик
        table.insert(Toggle.Connections, Button.MouseButton1Down:Connect(function()
            if Toggle.Disabled or Library.Unloaded then
                return
            end

            TweenService:Create(Ball, PressTweenInfo, {
                Size = UDim2.fromScale(0.88, 0.88),
            }):Play()
        end))
        table.insert(Toggle.Connections, Button.MouseButton1Up:Connect(function()
            if Library.Unloaded then
                return
            end

            TweenService:Create(Ball, PressTweenInfo, {
                Size = UDim2.fromScale(1, 1),
            }):Play()
        end))]])
if #src ~= srcLenP7 then
print("[Oblivion] P7: новые тоглы применены!")
else
src = plainReplace(src, "            TweenService:Create(Ball, Library.TweenInfo, {\n                AnchorPoint = Vector2.new(Offset, 0),\n                Position = UDim2.fromScale(Offset, 0),\n            }):Play()", "            TweenService:Create(Ball, TweenInfo.new(0.25, Enum.EasingStyle.Back, Enum.EasingDirection.Out), {\n                AnchorPoint = Vector2.new(Offset, 0),\n                Position = UDim2.fromScale(Offset, 0),\n            }):Play()")
print("[Oblivion] P7: точный патч не совпал — применён запасной (пружинная анимация)")
end
return loadstring(src)()
end)
if success then
Library = res
libDone = true
print("[Oblivion] Library успешно скачана!")
else
warn("[Oblivion] ОШИБКА загрузки Library: " .. tostring(res))
end
end)
task.spawn(function()
local s, r = pcall(function()
ThemeManager = loadstring(game:HttpGet(repo .. "addons/ThemeManager.lua"))()
end)
if not s then warn("[Oblivion] ThemeManager не загружен: " .. tostring(r)) end
themeDone = true
end)
task.spawn(function()
local s, r = pcall(function()
SaveManager = loadstring(game:HttpGet(repo .. "addons/SaveManager.lua"))()
end)
if not s then warn("[Oblivion] SaveManager не загружен: " .. tostring(r)) end
saveDone = true
end)
t0 = tick()
while not libDone do
if tick() - t0 > 30 then
warn("[Oblivion] ТАЙМАУТ: Библиотека не скачалась за 30 секунд!")
break
end
task.wait(0.05)
end

if not Library then
warn("[Oblivion] КРИТИЧЕСКАЯ ОШИБКА: Library = nil. Скрипт остановлен.")
return
end
print("[Oblivion] Создаем главное окно...")
Players = game:GetService("Players")
RunService = game:GetService("RunService")
LocalPlayer = Players.LocalPlayer

UserInputService = game:GetService("UserInputService")
Library.ForceCheckbox = true

Window = Library:CreateWindow({
Icon = "96010887466311",
Title = "Oblivion | FTAP",
Font = Enum.Font.GothamMedium, 
SidebarCompacted = true, 
Footer = "Oblivion Project v3.38",
Center = true,
AutoShow = true,
Resizable = true,
NotifySide = "Right",
ShowCustomCursor = true,
})

Tabs = {
Home = Window:AddTab("Home", "house"),
Combat = Window:AddTab("Combat", "swords"),
Def = Window:AddTab("Def", "shield"),
Movement = Window:AddTab("Movement", "person-standing"),
Visual = Window:AddTab("Visual", "eye"),
Target = Window:AddTab("Target", "target"),
Bloodman = Window:AddTab("Bloodman", "skull"),
Fun = Window:AddTab("Fun", "party-popper"),
Misc = Window:AddTab("Misc", "component"),
Settings = Window:AddTab("Settings", "settings")
}

for _, TabBtn in ipairs(Library.TabButtons) do
if TabBtn.Label then TabBtn.Label.Visible = false end
end

pcall(function() Library:SetFont(Enum.Font.GothamMedium) end)

task.defer(function()
pcall(function()
local mainFrame = Window.MainFrame
if not mainFrame or not Library.AddToRegistry then return end
if mainFrame:FindFirstChild("_obvWinGlow") then return end
for _, wst in ipairs(mainFrame:GetChildren()) do
if wst:IsA("UIStroke") then wst.Transparency = 1 end
end
local glow = Instance.new("UIStroke")
glow.Name = "_obvWinGlow"
glow.ApplyStrokeMode = Enum.ApplyStrokeMode.Border
glow.Thickness = 3
glow.Transparency = 0.5
glow.Color = Library.Scheme.AccentColor
glow.Parent = mainFrame
Library:AddToRegistry(glow, { Color = "AccentColor" })
task.spawn(function()
local TS0 = game:GetService("TweenService")
local breatheUp = false
while not Library.Unloaded and glow.Parent do
breatheUp = not breatheUp
TS0:Create(glow, TweenInfo.new(1.5, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), { Transparency = breatheUp and 0.4 or 0.62 }):Play()
task.wait(1.5)
end
end)
local TS = game:GetService("TweenService")
local origWinPos = mainFrame.Position
mainFrame.Position = origWinPos + UDim2.fromOffset(0, 36)
TS:Create(mainFrame, TweenInfo.new(0.6, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), { Position = origWinPos }):Play()
end)
end)

if Library and Library.Notify then
local _origNotify = Library.Notify
local _scriptIcon = "rbxassetid://96010887466311"
Library.Notify = function(self, ...)
local args = {...}
local notifText = (type(args[1]) == "table" and args[1].Title) or args[1] or ""
_origNotify(self, ...)
task.spawn(function()
for attempt = 1, 10 do
task.wait(0.05)
local found = nil
local services = {game:GetService("CoreGui"), game:GetService("PlayerGui")}
for _, svc in ipairs(services) do
for _, gui in ipairs(svc:GetChildren()) do
for _, tl in ipairs(gui:GetDescendants()) do
if tl:IsA("TextLabel") and tl.Text == notifText and tl.Visible then
found = tl
break
end
end
if found then break end
end
if found then break end
end
if found then
local notifFrame = found.Parent
if notifFrame and notifFrame:FindFirstChild("_obvIcon") then return end
local icon = Instance.new("ImageLabel")
icon.Name = "_obvIcon"
icon.Image = _scriptIcon
icon.Size = UDim2.new(0, 30, 0, 30)
icon.BackgroundTransparency = 1
icon.BorderSizePixel = 0
icon.ZIndex = found.ZIndex + 1
if notifFrame:IsA("Frame") or notifFrame:IsA("ScrollingFrame") then
icon.Position = UDim2.new(0, 4, 0.5, -15)
icon.Parent = notifFrame
for _, child in ipairs(notifFrame:GetChildren()) do
if child:IsA("TextLabel") or child:IsA("TextButton") then
local xOff = child.Position.X.Offset
child.Position = UDim2.new(child.Position.X.Scale, xOff + 38, child.Position.Y.Scale, child.Position.Y.Offset)
end
end
else
icon.Position = UDim2.new(0, 4, 0.5, -15)
icon.Parent = found
found.Position = UDim2.new(found.Position.X.Scale, found.Position.X.Offset + 38, found.Position.Y.Scale, found.Position.Y.Offset)
end
return
end
end
end)
end
end

Toggles = Library.Toggles
Options = Library.Options

currentLang = "English"
if readfile then
pcall(function()
local savedLang = readfile("obvilion_lang.txt")
if savedLang == "English" or savedLang == "Русский" then
currentLang = savedLang
end
end)
end

function L(ru, en)
if currentLang == "English" then return en end
return ru
end
local HomeLeft = Tabs.Home:AddLeftGroupbox("Profile", "user")
local HomeRight = Tabs.Home:AddRightGroupbox("About", "info")

task.defer(function()
local container = nil
if HomeLeft.Container then
container = HomeLeft.Container
elseif HomeLeft.Frame then
container = HomeLeft.Frame
end
if not container then
for _, obj in ipairs(Window:GetDescendants()) do
if obj:IsA("ScrollingFrame") or obj:IsA("Frame") then
for _, child in ipairs(obj:GetChildren()) do
if child:IsA("Frame") and child.Name:find("Profile") then
container = obj
break
end
end
if container then break end
end
end
end
if container then
local avatar = Instance.new("ImageLabel")
avatar.Name = "HomeAvatar"
avatar.Size = UDim2.new(0, 240, 0, 240)
avatar.Position = UDim2.new(0, 0, 0, 0)
avatar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
avatar.BorderSizePixel = 0
avatar.Image = string.format("rbxthumb://type=AvatarHeadShot&id=%d&w=150&h=150", LocalPlayer.UserId)
avatar.ScaleType = Enum.ScaleType.Fit
avatar.BackgroundTransparency = 0.3
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 12)
corner.Parent = avatar
avatar.Parent = container
avatar.ZIndex = 50
for _, child in ipairs(container:GetChildren()) do
if child ~= avatar and child:IsA("GuiObject") then
child.Position = child.Position + UDim2.new(0, 0, 0, 250)
end
end
end
end)

HomeLeft:AddLabel(LocalPlayer.DisplayName)
HomeLeft:AddLabel("@" .. LocalPlayer.Name)
HomeLeft:AddLabel("ID: " .. LocalPlayer.UserId)
local accTimestamp = os.time() - LocalPlayer.AccountAge * 86400
HomeLeft:AddLabel("Created: " .. os.date("%d.%m.%Y", accTimestamp))

HomeLeft:AddButton({
Text = "Rejoin Server",
Tooltip = L("Перезайти на другой сервер", "Rejoin another server"),
Func = function()
game:GetService("TeleportService"):TeleportToPlaceInstance(game.PlaceId, game.JobId, LocalPlayer)
end,
})

HomeRight:AddLabel("Free balance & comfort")
HomeRight:AddLabel("script for FTAP.")
HomeRight:AddLabel("Made to enhance your")
HomeRight:AddLabel("gameplay experience.")
HomeRight:AddLabel("")
HomeRight:AddLabel("Owner: quvode")
HomeRight:AddLabel("Co-Owner: Hunter")

do

CombatGrab = Tabs.Combat:AddLeftGroupbox("Grab Modifier", "hand")

CombatGrab:AddToggle("EnableGrabModifier", {
Text = "Grab Modifier",
Default = false,
Tooltip = L("Один тогл на все виды грабов. Выбирай нужные в списке ниже", "One toggle for all grab types. Pick the ones you need in the list below"),
})

CombatGrab:AddDropdown("GrabModifierTypes", {
Text = L("Виды грабов", "Grab Types"),
Values = {"Ragdoll", "Fling", "Kill", "Freeze", "Spin", "Massless", "Invisible", "Noclip", "Kick"},
Multi = true,
Tooltip = L("Выбери грабы: можно один или сразу все. Anchor Grab — отдельно ниже", "Pick grabs: one or all of them. Anchor Grab is separate below"),
})

CombatGrab:AddToggle("EnableCombatGrab", {
Text = "Ragdoll Grab", Default = false, Visible = false,
Tooltip = L("Автоматически рагдаллит игрока, которого вы захватили", "Automatically ragdolls the grabbed player"),
})
CombatGrab:AddToggle("EnableFlingGrab", {
Text = "Fling Grab", Default = false, Visible = false,
Tooltip = L("Запускает игрока высоко в небо (Fling)", "Launches player high into the sky (Fling)"),
})
CombatGrab:AddToggle("EnableKillGrab", {
Text = "Kill Grab", Default = false, Visible = false,
Tooltip = L("Мгновенно убивает игрока (бросок в бездну под карту)", "Instantly kills player (throws into void)"),
})
CombatGrab:AddToggle("EnableFreezeGrab", {
Text = "Freeze Grab", Default = false, Visible = false,
Tooltip = "Freeze Grab",
})
CombatGrab:AddToggle("EnableSpinGrab", {
Text = "Spin Grab", Default = false, Visible = false,
Tooltip = L("Вращает захваченного игрока/предмет вокруг своей оси", "Spins grabbed player/item around its axis"),
})

CombatGrab:AddSlider("SpinGrabSpeed", {
Text = "Spin Speed",
Default = 30,
Min = 1,
Max = 200,
Increment = 1,
Tooltip = L("Скорость вращения при Spin Grab", "Spin speed for Spin Grab"),
})

CombatGrab:AddToggle("EnableMasslessGrab", {
Text = "Massless Grab", Default = false, Visible = false,
Tooltip = L("Линия захвата без физики — цель становится очень лёгкой, без массы", "Physics-less grab line — target becomes weightless"),
})
CombatGrab:AddToggle("EnableInvisibleGrab", {
Text = "Invisible Grab", Default = false, Visible = false,
Tooltip = L("Спам CreateGrabLine → граблайн невидим для всех", "Spam CreateGrabLine → grab line invisible to everyone"),
})
CombatGrab:AddToggle("EnableNoclipGrab", {
Text = "Noclip Grab", Default = false, Visible = false,
Tooltip = "Target passes through walls",
})

GrabFlagMap = {
["Ragdoll"] = "EnableCombatGrab",
["Fling"] = "EnableFlingGrab",
["Kill"] = "EnableKillGrab",
["Freeze"] = "EnableFreezeGrab",
["Spin"] = "EnableSpinGrab",
["Massless"] = "EnableMasslessGrab",
["Invisible"] = "EnableInvisibleGrab",
["Noclip"] = "EnableNoclipGrab",
["Kick"] = "EnableKickGrab",
}
local function syncGrabToggles()
local master = Toggles.EnableGrabModifier and Toggles.EnableGrabModifier.Value or false
local sel = Options.GrabModifierTypes and Options.GrabModifierTypes.Value or {}
for label, flag in pairs(GrabFlagMap) do
local t = Toggles[flag]
if t then
local want = (master and sel[label] == true) or false
if t.Value ~= want then
pcall(function() t:SetValue(want) end)
end
end
end
end
Toggles.EnableGrabModifier:OnChanged(syncGrabToggles)
Options.GrabModifierTypes:OnChanged(syncGrabToggles)

local function mirrorGrabDropdown()
local dd = Options.GrabModifierTypes
if not dd then return end
local sel = {}
for label, flag in pairs(GrabFlagMap) do
local t = Toggles[flag]
if t and t.Value then sel[label] = true end
end
local cur = dd.Value or {}
local same = true
for k in pairs(cur) do
if not sel[k] then same = false break end
end
if same then
for k in pairs(sel) do
if not cur[k] then same = false break end
end
end
if not same then
pcall(function() dd:SetValue(sel) end)
end
end
for _, flag in pairs(GrabFlagMap) do
local t = Toggles[flag]
if t and type(t.SetValue) == "function" and not t._grabMirror then
local orig = t.SetValue
t._grabMirror = true
t.SetValue = function(self, v)
orig(self, v)
mirrorGrabDropdown()
end
end
end

CombatGrab:AddToggle("EnableAnchorGrab", {
Text = "Anchor Grab",
Default = false,
Tooltip = "Anchor Held + Auto Recover Parts (9rr). Grab and release = freeze, nobody can take it",
}):AddKeyPicker("AnchorGrabKeybind", {
Default = "T",
SyncToggleState = false,
Mode = "Hold",
Text = "Anchor Key",
NoUI = false,
})

local freezeGrabCurrentTarget = nil

local CombatGrabRight = Tabs.Combat:AddRightGroupbox("Grab Options", "settings")

CombatGrabRight:AddToggle("EnableInfiniteLine", {
Text = "Infinite Line Extend",
Default = false,
Tooltip = "Infinite grab line - scroll mouse wheel to extend/retract",
})

CombatGrabRight:AddSlider("LineExtendIncrease", {
Text = "Line Extend Step",
Default = 7,
Min = 1,
Max = 10,
Increment = 1,
Tooltip = "How much to extend per scroll step",
})

local infiniteLineConn = nil
local infiniteLineWheelConn = nil
local infiniteLineDistance = 0
local infiniteLineIncrease = 7

Toggles.EnableInfiniteLine:OnChanged(function()
if Toggles.EnableInfiniteLine.Value then
local cam = workspace.CurrentCamera
local UIS = game:GetService("UserInputService")
infiniteLineDistance = 0
infiniteLineIncrease = Options.LineExtendIncrease and Options.LineExtendIncrease.Value or 7

infiniteLineWheelConn = UIS.InputChanged:Connect(function(input)
if input.UserInputType == Enum.UserInputType.MouseWheel then
if infiniteLineDistance <= 3 then
infiniteLineDistance = 3
end
if input.Position.Z > 0 then
infiniteLineDistance = infiniteLineDistance + infiniteLineIncrease
elseif input.Position.Z < 0 then
infiniteLineDistance = infiniteLineDistance - infiniteLineIncrease
end
end
end)

infiniteLineConn = workspace.ChildAdded:Connect(function(child)
if child.Name == "GrabParts" and child:IsA("Model") then
if UIS.MouseEnabled then
local grabPartsModel = child
grabPartsModel:WaitForChild("GrabPart")
grabPartsModel:WaitForChild("DragPart")

local clonedDragPart = grabPartsModel.DragPart:Clone()
clonedDragPart.Name = "DragPart1"
if clonedDragPart:FindFirstChild("AlignPosition") and clonedDragPart:FindFirstChild("DragAttach") then
clonedDragPart.AlignPosition.Attachment1 = clonedDragPart.DragAttach
end
clonedDragPart.Parent = grabPartsModel

infiniteLineDistance = (clonedDragPart.Position - cam.CFrame.Position).Magnitude

if grabPartsModel.DragPart:FindFirstChild("AlignPosition") then
grabPartsModel.DragPart.AlignPosition.Enabled = false
end
if clonedDragPart:FindFirstChild("AlignOrientation") then
clonedDragPart.AlignOrientation.Enabled = false
end

task.spawn(function()
while grabPartsModel.Parent do
clonedDragPart.Position = cam.CFrame.Position + cam.CFrame.LookVector * infiniteLineDistance
task.wait()
end
infiniteLineDistance = 0
end)
end
end
end)
else
if infiniteLineWheelConn then
infiniteLineWheelConn:Disconnect()
infiniteLineWheelConn = nil
end
if infiniteLineConn then
infiniteLineConn:Disconnect()
infiniteLineConn = nil
end
infiniteLineDistance = 0
end
end)

Options.LineExtendIncrease:OnChanged(function()
infiniteLineIncrease = Options.LineExtendIncrease.Value
end)

local spinGrabTargets = {}

local invisibleGrabTargets = {}
local noclipGrabTargets = {}

InvisLineOn = false
_invisLineTask = nil

local function ApplyInvisLine(v)
InvisLineOn = v
_G.InvisLine = v or nil

if _invisLineTask then
pcall(function() task.cancel(_invisLineTask) end)
_invisLineTask = nil
end

if v then
local grabFolder = ReplicatedStorage:FindFirstChild("GrabEvents")
if not grabFolder then grabFolder = ReplicatedStorage:WaitForChild("GrabEvents", 5) end
local createLine = grabFolder and grabFolder:FindFirstChild("CreateGrabLine")
if not createLine then
Library:Notify(L("Invisible Grab: CreateGrabLine не найден", "Invisible Grab: CreateGrabLine not found"), 3)
Toggles.EnableInvisibleGrab:SetValue(false)
return
end

_invisLineTask = task.spawn(function()
while InvisLineOn do
pcall(function() createLine:FireServer() end)
RunService.Heartbeat:Wait()
end
end)
end
end

Toggles.EnableInvisibleGrab:OnChanged(function()
ApplyInvisLine(Toggles.EnableInvisibleGrab.Value)

if not Toggles.EnableInvisibleGrab.Value then
for obj, data in pairs(invisibleGrabTargets) do
if obj and obj.Parent then
if data.isBeam then
pcall(function() obj.Enabled = data.enabled end)
else
pcall(function() obj.Transparency = data.trans end)
end
end
end
invisibleGrabTargets = {}
end
end)

CombatGrab:AddToggle("EnableKickGrab", {
Text = "Kick Grab",
Default = false, Visible = false,
})

do
local t = Toggles.EnableKickGrab
if t and type(t.SetValue) == "function" and not t._grabMirror then
local orig = t.SetValue
t._grabMirror = true
t.SetValue = function(self, v)
orig(self, v)
mirrorGrabDropdown()
end
end
end
Toggles.EnableKickGrab:OnChanged(function(state)
if not state then
getgenv().KickGrabActive = false
getgenv().FKeyAttackActive = false

if getgenv().FKeyInputConnection then
getgenv().FKeyInputConnection:Disconnect()
getgenv().FKeyInputConnection = nil
end

return
end
if getgenv().KickGrabActive then
return
end

local conflictToggles = {
"EnableCombatGrab", 
"EnableKillGrab",
"EnableFlingGrab",
"EnableSpinGrab",
"EnableMasslessGrab",
"EnableAnchorGrab",
}
for _, flag in ipairs(conflictToggles) do
if Toggles[flag] and Toggles[flag].Value then
pcall(function() Toggles[flag]:SetValue(false) end)
Library:Notify("Kick Grab: " .. flag .. " выключен (конфликт)", 2)
end
end

getgenv().KickGrabActive = true
getgenv().FKeyAttackActive = false

local Players = game:GetService('Players')
local ReplicatedStorage = game:GetService('ReplicatedStorage')
local UserInputService = game:GetService('UserInputService')
local RunService = game:GetService('RunService')
local plr = Players.LocalPlayer
local camera = workspace.CurrentCamera
local GrabEvents = ReplicatedStorage:WaitForChild('GrabEvents')
local CreateGrabLine = GrabEvents:WaitForChild('CreateGrabLine')
local SetNetworkOwner = GrabEvents:WaitForChild('SetNetworkOwner')
local DestroyGrabLine = GrabEvents:WaitForChild('DestroyGrabLine')

task.spawn(function()
while getgenv().KickGrabActive do
local grabParts = workspace:FindFirstChild('GrabParts')

if not grabParts then
task.wait()
continue
end

local gp = grabParts:FindFirstChild('GrabPart')
local weld = gp and gp:FindFirstChildOfClass('WeldConstraint')
local part1 = weld and weld.Part1

if part1 then
local ownerPlayer = nil

for _, pl in ipairs(Players:GetPlayers()) do
if pl.Character and part1:IsDescendantOf(pl.Character) then
ownerPlayer = pl
break
end
end

if not ownerPlayer then
task.wait()
continue
end

while getgenv().KickGrabActive and workspace:FindFirstChild('GrabParts') do
if ownerPlayer then
local tgtTorso = ownerPlayer.Character and ownerPlayer.Character:FindFirstChild('HumanoidRootPart')
local tgtHead = ownerPlayer.Character and (ownerPlayer.Character:FindFirstChild('Head') or ownerPlayer.Character:FindFirstChild('Torso') or ownerPlayer.Character:FindFirstChild('UpperTorso'))
local myTorso = plr.Character and plr.Character:FindFirstChild('HumanoidRootPart')

if tgtTorso and myTorso and tgtHead then
pcall(function()
SetNetworkOwner:FireServer(tgtTorso, CFrame.lookAt(myTorso.Position, tgtTorso.Position))
end)
task.wait()
pcall(function()
DestroyGrabLine:FireServer(tgtHead)
end)
end
end

task.wait()
end
end

task.wait()
end
end)
end)

Toggles.EnableNoclipGrab:OnChanged(function()
if not Toggles.EnableNoclipGrab.Value then
for part, origCollide in pairs(noclipGrabTargets) do
if part and part.Parent then
pcall(function() part.CanCollide = origCollide end)
end
end
noclipGrabTargets = {}
end
end)

Toggles.EnableSpinGrab:OnChanged(function()
if not Toggles.EnableSpinGrab.Value then
for primary, bav in pairs(spinGrabTargets) do
if bav and bav.Parent then bav:Destroy() end
end
spinGrabTargets = {}
end
end)
local freezeGrabData = {} 

local function freezeGrabGetNetworkOwner()
local rs = game:GetService("ReplicatedStorage")
local folder = rs:FindFirstChild("GrabEvents") or rs:WaitForChild("GrabEvents", 10)
if folder then
return folder:FindFirstChild("SetNetworkOwner") or folder:WaitForChild("SetNetworkOwner", 5)
end
return nil
end

local function freezeGrabGetPrimaryPart(model)
if not model then return nil end
if model:IsA("Model") and model.PrimaryPart then return model.PrimaryPart end
local hum = model:FindFirstChildOfClass("Humanoid")
if hum and model:FindFirstChild("HumanoidRootPart") then return model.HumanoidRootPart end
for _, part in ipairs(model:GetDescendants()) do
if part:IsA("BasePart") then return part end
end
if model:IsA("BasePart") then return model end
return nil
end

local function freezeGrabModel(model)
if not model or not model.Parent then return end
if freezeGrabData[model] then return end

local primary = freezeGrabGetPrimaryPart(model)
if not primary then return end

local hum = model:FindFirstChildOfClass("Humanoid")
local setOwner = freezeGrabGetNetworkOwner()

local savedState = {
primary = primary,
anchored = primary.Anchored,
platformStand = hum and hum.PlatformStand or nil,
sit = hum and hum.Sit or nil,
pos = primary.Position,
}

if setOwner then
for _, part in ipairs(model:GetDescendants()) do
if part:IsA("BasePart") then
pcall(function() setOwner:FireServer(part, part.CFrame) end)
end
end
if model:IsA("BasePart") then
pcall(function() setOwner:FireServer(model, model.CFrame) end)
end
end

local bp = Instance.new("BodyPosition")
bp.Name = "FreezeGrabBodyPosition"
bp.MaxForce = Vector3.new(1e9, 1e9, 1e9)
bp.P = 1e7
bp.D = 1e5
bp.Position = primary.Position
bp.Parent = primary

local bg = Instance.new("BodyGyro")
bg.Name = "FreezeGrabBodyGyro"
bg.MaxTorque = Vector3.new(1e9, 1e9, 1e9)
bg.P = 1e7
bg.D = 1e5
bg.CFrame = primary.CFrame
bg.Parent = primary

pcall(function() primary.Anchored = true end)

if hum then
pcall(function()
hum.PlatformStand = true
hum.Sit = true
end)
end

local heartbeat = game:GetService("RunService").Heartbeat:Connect(function()
local owner = freezeGrabGetNetworkOwner()
if primary and primary.Parent then
pcall(function()
primary.Anchored = true
primary.AssemblyLinearVelocity = Vector3.zero
primary.AssemblyAngularVelocity = Vector3.zero
end)
if bp and bp.Parent then
pcall(function() bp.Position = savedState.pos end)
end
if owner then
pcall(function() owner:FireServer(primary, primary.CFrame) end)
end
else
if freezeGrabData[model] then
freezeGrabData[model].heartbeat:Disconnect()
freezeGrabData[model] = nil
end
end
if hum and hum.Parent then
pcall(function()
hum.PlatformStand = true
hum.Sit = true
end)
end
end)

freezeGrabData[model] = {
savedState = savedState,
heartbeat = heartbeat,
bodyPosition = bp,
bodyGyro = bg,
}

Library:Notify(L("Freeze Grab: заморожено", "Freeze Grab: frozen"), 1)
end

local function unfreezeGrabModel(model)
if not model then return end
local data = freezeGrabData[model]
if not data then return end

if data.heartbeat then
data.heartbeat:Disconnect()
end
if data.bodyPosition then
pcall(function() data.bodyPosition:Destroy() end)
end
if data.bodyGyro then
pcall(function() data.bodyGyro:Destroy() end)
end

local saved = data.savedState
if saved then
local primary = saved.primary
if primary and primary.Parent then
pcall(function()
primary.Anchored = false
primary.AssemblyLinearVelocity = Vector3.zero
primary.AssemblyAngularVelocity = Vector3.zero
primary.Velocity = Vector3.new(0, -5, 0)
primary.AssemblyLinearVelocity = Vector3.new(0, -5, 0)
end)
local setOwner = freezeGrabGetNetworkOwner()
if setOwner then
pcall(function() setOwner:FireServer(primary, primary.CFrame) end)
end
end
local hum = saved.primary and saved.primary.Parent and saved.primary.Parent:FindFirstChildOfClass("Humanoid")
if hum then
pcall(function()
hum.PlatformStand = saved.platformStand
hum.Sit = saved.sit
hum:ChangeState(Enum.HumanoidStateType.GettingUp)
hum:ChangeState(Enum.HumanoidStateType.Running)
end)
end
end

freezeGrabData[model] = nil
Library:Notify(L("Freeze Grab: разморожено", "Freeze Grab: unfrozen"), 1)
end

Toggles.EnableFreezeGrab:OnChanged(function()
if Toggles.EnableFreezeGrab.Value then
if freezeGrabCurrentTarget and not freezeGrabData[freezeGrabCurrentTarget] then
freezeGrabModel(freezeGrabCurrentTarget)
end
else
local frozenModels = {}
for model, _ in pairs(freezeGrabData) do
table.insert(frozenModels, model)
end
for _, model in ipairs(frozenModels) do
unfreezeGrabModel(model)
task.wait(0.05)
end
end
end)

workspace.ChildAdded:Connect(function(child)
if child.Name == "GrabParts" then
local grabPart = child:WaitForChild("GrabPart", 1)
if not grabPart then return end
local weld = grabPart:WaitForChild("WeldConstraint", 1)
if not weld then return end
local targetPart = weld.Part1
local targetModel = targetPart and targetPart.Parent
local targetPlayer = targetModel and Players:GetPlayerFromCharacter(targetModel)
if targetModel and targetModel ~= LocalPlayer.Character and targetPlayer and targetPlayer ~= LocalPlayer then
freezeGrabCurrentTarget = targetModel
if Toggles.EnableFreezeGrab and Toggles.EnableFreezeGrab.Value then
if freezeGrabData[targetModel] then
unfreezeGrabModel(targetModel)
else
freezeGrabModel(targetModel)
end
end
end
end
end)

workspace.ChildRemoved:Connect(function(child)
if child.Name == "GrabParts" then
freezeGrabCurrentTarget = nil
for primary, bav in pairs(spinGrabTargets) do
if bav and bav.Parent then bav:Destroy() end
end
spinGrabTargets = {}
end
end)

local AnchoredObjects = {}
local AnchorRecoverCoro = nil
local AnchorRecoverEnabled = false
local AnchorInputConn = nil

local Anchor_SetNW = game:GetService("ReplicatedStorage"):WaitForChild("GrabEvents"):WaitForChild("SetNetworkOwner")

local function setanchorObject(part)
if typeof(part) ~= 'Instance' or not part.Parent then return end
if not (part.Parent:IsA('Model') or part.Parent:IsA('Folder')) then return end
local parentModel = part.Parent
if parentModel:IsA('Folder') or parentModel == workspace then
parentModel = part
end
if parentModel:GetAttribute('IsAnchored') then
local data = AnchoredObjects[parentModel]
if data then
if data.BodyPosition and data.BodyPosition.Parent then
data.BodyPosition.MaxForce = Vector3.new(0, 0, 0)
data.BodyPosition:Destroy()
end
if data.BodyGyro and data.BodyGyro.Parent then
data.BodyGyro.MaxTorque = Vector3.new(0, 0, 0)
data.BodyGyro:Destroy()
end
if data.Highlight and data.Highlight.Parent then
data.Highlight:Destroy()
end
for _, conn in ipairs(data.Connections) do
if conn and conn.Connected then conn:Disconnect() end
end
parentModel:SetAttribute('IsAnchored', false)
AnchoredObjects[parentModel] = nil
Library:Notify("Anchor Grab: unanchored", 1)
end
else
local originalPosition = part.Position
local maxTorque = Vector3.new(math.huge, math.huge, math.huge)
local zeroVector = Vector3.new(0, 0, 0)

local bodyPosition = Instance.new('BodyPosition')
bodyPosition.Name = 'AnchorPositionBody'
bodyPosition.P = 40000
bodyPosition.D = 950
bodyPosition.MaxForce = maxTorque
bodyPosition.Position = part.Position
bodyPosition.Parent = part

local bodyGyro = Instance.new('BodyGyro')
bodyGyro.Name = 'AnchorGyroBody'
bodyGyro.P = 40000
bodyGyro.D = 950
bodyGyro.MaxTorque = maxTorque
bodyGyro.CFrame = part.CFrame
bodyGyro.Parent = part

local highlight = Instance.new('Highlight')
highlight.Name = 'GrabAnchorHL'
highlight.DepthMode = Enum.HighlightDepthMode.Occluded
highlight.FillTransparency = 1
highlight.OutlineColor = Color3.new(0, 0, 1)
highlight.OutlineTransparency = 0.5
highlight.Parent = parentModel

local connections = {}
local anchoredPart = part

connections[1] = parentModel.DescendantAdded:Connect(function(descendant)
if descendant.Name == 'PartOwner' then
local h = parentModel:FindFirstChild('GrabAnchorHL')
if h then
h.OutlineColor = descendant.Value ~= LocalPlayer.Name and Color3.new(1, 0, 0) or Color3.new(0, 0, 1)
end
if descendant.Value ~= LocalPlayer.Name then
bodyGyro.MaxTorque = zeroVector
bodyPosition.MaxForce = zeroVector
else
bodyGyro.MaxTorque = maxTorque
bodyPosition.MaxForce = maxTorque
end
end
end)
connections[2] = parentModel.DescendantRemoving:Connect(function(descendant)
if descendant.Name == 'PartOwner' and descendant.Value == LocalPlayer.Name then
bodyGyro.MaxTorque = zeroVector
bodyPosition.MaxForce = zeroVector
end
end)

task.spawn(function()
while bodyPosition.Parent do
if parentModel:GetAttribute('IsAnchored') then
bodyGyro.MaxTorque = maxTorque
bodyPosition.MaxForce = maxTorque
else
bodyGyro.MaxTorque = zeroVector
bodyPosition.MaxForce = zeroVector
end
bodyPosition.Position = originalPosition + Vector3.new(0, 0.001, 0)
task.wait()
bodyPosition.Position = originalPosition
end
end)

AnchoredObjects[parentModel] = {
BodyPosition = bodyPosition,
BodyGyro = bodyGyro,
PartAnchored = part,
Highlight = highlight,
Connections = connections,
Model = parentModel,
OriginalPosition = originalPosition,
}

local targetPlayer = Players:GetPlayerFromCharacter(parentModel)
if targetPlayer then
AnchoredObjects[parentModel].Player = targetPlayer
AnchoredObjects[parentModel].AnchorCFrame = part.CFrame
end

parentModel:SetAttribute('IsAnchored', true)
Library:Notify("Anchor Grab: anchored", 1)
end
end

local function anchorfunc()
local grabParts = workspace:FindFirstChild('GrabParts')
local didGrab = false
if grabParts then
local grabPart = grabParts:FindFirstChild('GrabPart')
if grabPart then
local weld = grabPart:FindFirstChild('WeldConstraint')
if weld and weld.Part1 then
local heldPart = weld.Part1
if heldPart and heldPart.Parent and not heldPart.Anchored then
setanchorObject(heldPart)
didGrab = true
end
end
end
end
if not didGrab then
local char = LocalPlayer.Character
if not char then return end
local cam = workspace.CurrentCamera
if not cam then return end
local origin = cam.CFrame.Position
local direction = cam.CFrame.LookVector * 5000
local result = workspace:Raycast(origin, direction, {char})
if result and result.Instance then
local hitPart = result.Instance
local targetModel = nil
local check = hitPart
while check and check.Parent do
if check:IsA('Model') then
if check:GetAttribute('IsAnchored') then
targetModel = check
break
end
if not targetModel then
targetModel = check
end
end
check = check.Parent
end
if not targetModel and hitPart.Parent then
if hitPart.Parent == workspace or hitPart.Parent:IsA('Folder') then
targetModel = hitPart
end
end
if targetModel then
setanchorObject(targetModel:IsA('BasePart') and targetModel or targetModel:FindFirstChildWhichIsA('BasePart'))
end
end
end
end

local function unAnchorAll()
for model, data in pairs(AnchoredObjects) do
if data.BodyPosition and data.BodyPosition.Parent then data.BodyPosition:Destroy() end
if data.BodyGyro and data.BodyGyro.Parent then data.BodyGyro:Destroy() end
if data.Highlight and data.Highlight.Parent then data.Highlight:Destroy() end
for _, conn in ipairs(data.Connections) do
if conn and conn.Connected then conn:Disconnect() end
end
if typeof(model) == 'Instance' and model.Parent then
model:SetAttribute('IsAnchored', false)
end
end
table.clear(AnchoredObjects)
end

local function RecoverParts()
while AnchorRecoverEnabled do
pcall(function()
local character = LocalPlayer.Character
if character and character:FindFirstChild('HumanoidRootPart') then
local humanoidRootPart = character.HumanoidRootPart
for model, data in pairs(AnchoredObjects) do
coroutine.wrap(function()
if data.PartAnchored and data.PartAnchored.Parent then
local part = data.PartAnchored
local distance = (part.Position - humanoidRootPart.Position).Magnitude
if distance > 30 then
local partOwner = part:FindFirstChild('PartOwner')
if not partOwner or partOwner.Value ~= LocalPlayer.Name then
humanoidRootPart.CFrame = CFrame.new(part.Position + Vector3.new(0, 5, 0))
task.wait(0.1)
Anchor_SetNW:FireServer(part, CFrame.lookAt(humanoidRootPart.Position, part.Position))
end
else
local partOwner = part:FindFirstChild('PartOwner')
if not partOwner or partOwner.Value ~= LocalPlayer.Name then
Anchor_SetNW:FireServer(part, CFrame.lookAt(humanoidRootPart.Position, part.Position))
end
end
end
end)()
end
local toReanchor = {}
for model, data in pairs(AnchoredObjects) do
if data.Player and data.AnchorCFrame and data.OriginalPosition then
local p = data.Player
if p.Character and p.Character ~= model then
local newHRP = p.Character:FindFirstChild('HumanoidRootPart')
if newHRP then
table.insert(toReanchor, {player = p, oldModel = model, cframe = data.AnchorCFrame, pos = data.OriginalPosition})
end
end
end
end
for _, entry in ipairs(toReanchor) do
pcall(function()
AnchoredObjects[entry.oldModel] = nil
if typeof(entry.oldModel) == 'Instance' and entry.oldModel.Parent then
entry.oldModel:SetAttribute('IsAnchored', false)
end
local newHRP = entry.player.Character:FindFirstChild('HumanoidRootPart')
if newHRP then
newHRP.CFrame = entry.cframe
setanchorObject(newHRP)
end
end)
end
end
end)
task.wait(0.05)
end
end

local function AnchorStartInput()
if AnchorInputConn then return end
AnchorInputConn = UserInputService.InputBegan:Connect(function(input, gpe)
if gpe then return end
if not Toggles.EnableAnchorGrab.Value then return end
local kb = Options.AnchorGrabKeybind
if not kb then return end
local boundKey = kb.Value
local isMatch = false
if input.KeyCode.Name == boundKey or input.UserInputType.Name == boundKey then isMatch = true end
if boundKey == "MB4" and input.KeyCode and input.KeyCode.Name == "ButtonX1" then isMatch = true end
if boundKey == "MB5" and input.KeyCode and input.KeyCode.Name == "ButtonX2" then isMatch = true end
if boundKey == "RightClick" or boundKey == "MB2" then
if input.UserInputType == Enum.UserInputType.MouseButton2 then isMatch = true end
end
if boundKey == "LeftClick" or boundKey == "MB1" then
if input.UserInputType == Enum.UserInputType.MouseButton1 then isMatch = true end
end
if not isMatch then return end
anchorfunc()
end)
end

local function AnchorStopInput()
if AnchorInputConn then AnchorInputConn:Disconnect(); AnchorInputConn = nil end
end

Toggles.EnableAnchorGrab:OnChanged(function()
if Toggles.EnableAnchorGrab.Value then
AnchorStartInput()
AnchorRecoverEnabled = true
if not AnchorRecoverCoro or coroutine.status(AnchorRecoverCoro) == 'dead' then
AnchorRecoverCoro = coroutine.create(RecoverParts)
coroutine.resume(AnchorRecoverCoro)
end
Library:Notify("Anchor Grab ON", 2)
else
AnchorStopInput()
AnchorRecoverEnabled = false
if AnchorRecoverCoro and coroutine.status(AnchorRecoverCoro) ~= 'dead' then
coroutine.close(AnchorRecoverCoro)
AnchorRecoverCoro = nil
end
unAnchorAll()
Library:Notify("Anchor Grab OFF", 2)
end
end)

local function combatGrabGetTargetModel(child)
local grabPart = child:WaitForChild("GrabPart", 1)
if not grabPart then return nil end
local weld = grabPart:WaitForChild("WeldConstraint", 1)
if not weld then return nil end
local targetPart = weld.Part1
local targetModel = targetPart and targetPart.Parent
if not targetModel then return nil end
local targetPlayer = Players:GetPlayerFromCharacter(targetModel)
if targetModel == LocalPlayer.Character then return nil end
if targetPlayer == LocalPlayer then return nil end
return targetModel
end

local function combatGrabGetDestroyLine()
local rs = game:GetService("ReplicatedStorage")
local folder = rs:FindFirstChild("GrabEvents") or rs:WaitForChild("GrabEvents", 10)
if folder then
return folder:FindFirstChild("DestroyGrabLine") or folder:WaitForChild("DestroyGrabLine", 5)
end
return nil
end

local function combatGrabReleaseHold(targetModel, targetRoot)
local destroyLine = combatGrabGetDestroyLine()
if not destroyLine then return end

if targetModel then
for _, v in ipairs(targetModel:GetDescendants()) do
if v.Name == "PartOwner" then
pcall(function() destroyLine:FireServer(v.Parent) end)
end
end
end

if targetRoot then
for _ = 1, 3 do
pcall(function() destroyLine:FireServer(targetRoot) end)
end
end
end

ragdollPalete = nil

local function ragdollGrabGetDestroyLine()
local RS = game:GetService("ReplicatedStorage")
local folder = RS:FindFirstChild("GrabEvents") or RS:WaitForChild("GrabEvents", 10)
if folder then
return folder:FindFirstChild("DestroyGrabLine") or folder:WaitForChild("DestroyGrabLine", 5)
end
return nil
end

local function ragdollGrabGetSpawnRemote()
local RS = game:GetService("ReplicatedStorage")
local menuToys = RS:FindFirstChild("MenuToys") or RS:WaitForChild("MenuToys", 10)
if menuToys then
return menuToys:FindFirstChild("SpawnToyRemoteFunction") or menuToys:WaitForChild("SpawnToyRemoteFunction", 5)
end
return nil
end

local function ragdollGrabCheckOwner(part)
return part:FindFirstChild("PartOwner") and part["PartOwner"].Value == LocalPlayer.Name
end

local function ragdollGrabSpawnPalete()
local spawnRemote = ragdollGrabGetSpawnRemote()
if not spawnRemote then return nil end
local canSpawn = LocalPlayer:FindFirstChild("CanSpawnToy") or LocalPlayer:WaitForChild("CanSpawnToy", 10)
if not canSpawn then return nil end
local t0 = tick()
while not canSpawn.Value do
if tick() - t0 > 5 then return nil end
task.wait(0.1)
end
local myHRP = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
if not myHRP then return nil end
local spawnCF = myHRP.CFrame * CFrame.new(0, 14, 20)
local inv = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
if not inv then return nil end
local spawned = nil
local conn
conn = inv.ChildAdded:Connect(function(child)
if child.Name == "PalletLightBrown" then spawned = child end
end)
task.spawn(function()
pcall(function() spawnRemote:InvokeServer("PalletLightBrown", spawnCF, Vector3.zero) end)
end)
local startT = tick()
repeat task.wait() until spawned or (tick() - startT) > 2.5
conn:Disconnect()
if spawned then
spawned:WaitForChild("SoundPart", 3)
end
return spawned
end

local function ragdollGrabPreparePalete()
local inv = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
if inv and inv:FindFirstChild("RagdollPalete") then
ragdollPalete = inv:FindFirstChild("RagdollPalete")
return
end
ragdollPalete = ragdollGrabSpawnPalete()
if not ragdollPalete then return end
local soundPart = ragdollPalete:FindFirstChild("SoundPart")
if not soundPart then return end
local setOwner = freezeGrabGetNetworkOwner()
local oldCF = LocalPlayer.Character and LocalPlayer.Character:GetPivot()
while not ragdollGrabCheckOwner(soundPart) do
if not soundPart or not soundPart.Parent then return end
if LocalPlayer.Character then
pcall(function() LocalPlayer.Character:PivotTo(soundPart.CFrame) end)
end
if setOwner then pcall(function() setOwner:FireServer(soundPart, soundPart.CFrame) end) end
task.wait(0.05)
end
for _, v in pairs(ragdollPalete:GetChildren()) do
if v:IsA("BasePart") then
v.Transparency = 0.8
v.CanCollide = false
v.CanQuery = false
end
end
if oldCF and LocalPlayer.Character then
pcall(function() LocalPlayer.Character:PivotTo(oldCF) end)
end
ragdollPalete.Name = "RagdollPalete"
local bv = Instance.new("BodyVelocity")
bv.MaxForce = Vector3.new(0, math.huge, 0)
bv.Velocity = Vector3.new(0, 900, 0)
bv.Parent = soundPart
end

local function ragdollGrabApply(targetModel)
if not targetModel then return end
local hum = targetModel:FindFirstChildOfClass("Humanoid")
if not hum then return end
local root = targetModel:FindFirstChild("HumanoidRootPart")
if not root then return end

local setOwner = freezeGrabGetNetworkOwner()

task.spawn(function()
if not ragdollPalete or not ragdollPalete.Parent then
ragdollGrabPreparePalete()
end
if not ragdollPalete or not ragdollPalete.Parent then return end
local soundPart = ragdollPalete:FindFirstChild("SoundPart")
if not soundPart then return end

if not ragdollGrabCheckOwner(soundPart) then
local t0 = tick()
while not ragdollGrabCheckOwner(soundPart) and tick() - t0 < 5 do
if setOwner then pcall(function() setOwner:FireServer(soundPart, soundPart.CFrame) end) end
task.wait(0.05)
end
end

while true do
local ragdolled = hum:FindFirstChild("Ragdolled")
if not root or not root.Parent then break end
if not workspace:FindFirstChild("GrabParts") then break end
if ragdolled and ragdolled.Value then break end
soundPart.Position = root.Position
task.wait(0.1)
end
end)

Library:Notify(L("Ragdoll Grab: цель рагдоллнута", "Ragdoll Grab: target ragdolled"), 1)
end

flingGrabTargetPart = nil

local function flingGrabApply(targetPart)
if not targetPart or not targetPart:IsA("BasePart") or not targetPart.Parent then return end

local setOwner = freezeGrabGetNetworkOwner()
if setOwner then
pcall(function() setOwner:FireServer(targetPart, targetPart.CFrame) end)
end

pcall(function()
targetPart.AssemblyLinearVelocity = Vector3.new(0, 400, 0)
end)

local bv = Instance.new("BodyVelocity")
bv.Name = "FlingGrabVelocity"
bv.MaxForce = Vector3.new(0, math.huge, 0)
bv.P = 100000
bv.Velocity = Vector3.new(0, 400, 0)
bv.Parent = targetPart
game:GetService("Debris"):AddItem(bv, 0.5)

Library:Notify(L("Fling Grab: цель подброшена в небо", "Fling Grab: target launched into sky"), 1)
end

local function killGrabApply(targetModel)
if not targetModel then return end
local hum = targetModel:FindFirstChildOfClass("Humanoid")
if not hum then return end
local root = targetModel:FindFirstChild("HumanoidRootPart")
if not root then return end

local setOwner = freezeGrabGetNetworkOwner()
local destroyLine = combatGrabGetDestroyLine()

task.spawn(function()
local head = targetModel:FindFirstChild("Head")
local t0 = tick()
while head and head:FindFirstChild("PartOwner") and head["PartOwner"].Value ~= LocalPlayer.Name do
if not workspace:FindFirstChild("GrabParts") then return end
if tick() - t0 > 10 then return end
if setOwner then
pcall(function() setOwner:FireServer(root, root.CFrame) end)
end
task.wait(0.05)
end

local deathCF = CFrame.new(-999999999999, 9999999999999, -999999999999)
for _, part in pairs(targetModel:GetChildren()) do
if part:IsA("BasePart") then
pcall(function() part.CFrame = deathCF end)
end
end
task.wait()
for _, part in pairs(targetModel:GetChildren()) do
if part:IsA("BasePart") then
pcall(function() part.CFrame = deathCF end)
end
end

local bv = Instance.new("BodyVelocity")
bv.Velocity = Vector3.new(0, 99999999999, 0)
bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bv.P = 100000075
bv.Parent = root

pcall(function()
hum.Sit = false
hum.Jump = true
hum.BreakJointsOnDeath = false
hum:ChangeState(Enum.HumanoidStateType.Dead)
end)

task.delay(2, function()
if bv and bv.Parent then bv:Destroy() end
end)

if setOwner then
pcall(function() destroyLine:FireServer(root) end)
end
end)

Library:Notify(L("Kill Grab: цель убита", "Kill Grab: target killed"), 1)
end

workspace.ChildAdded:Connect(function(child)
if child.Name ~= "GrabParts" then return end

task.spawn(function()
task.wait(0.05)
local targetModel = combatGrabGetTargetModel(child)
if not targetModel then return end

if Toggles.EnableCombatGrab and Toggles.EnableCombatGrab.Value then
ragdollGrabApply(targetModel)
end

if Toggles.EnableKillGrab and Toggles.EnableKillGrab.Value then
killGrabApply(targetModel)
end

if Toggles.EnableFlingGrab and Toggles.EnableFlingGrab.Value then
local hum = targetModel:FindFirstChildOfClass("Humanoid")
local root = targetModel:FindFirstChild("HumanoidRootPart")
if root and hum and hum.Health > 0 then
flingGrabTargetPart = root
end
end

if Toggles.EnableSpinGrab and Toggles.EnableSpinGrab.Value then
local primary = targetModel.PrimaryPart or targetModel:FindFirstChild("HumanoidRootPart") or targetModel:FindFirstChildWhichIsA("BasePart")
if primary then
local bav = Instance.new("BodyAngularVelocity")
bav.Name = "SpinGrabBAV"
bav.AngularVelocity = Vector3.new(0, Options.SpinGrabSpeed and Options.SpinGrabSpeed.Value or 30, 0)
bav.MaxTorque = Vector3.new(0, math.huge, 0)
bav.Parent = primary
spinGrabTargets[primary] = bav
end
end

if Toggles.EnableMasslessGrab and Toggles.EnableMasslessGrab.Value then
task.spawn(function()
local dragPart = child:WaitForChild("DragPart", 5)
if not dragPart then return end
local alignPos = dragPart:WaitForChild("AlignPosition", 5)
local alignOri = dragPart:WaitForChild("AlignOrientation", 5)
if not alignPos or not alignOri then return end
local setOwner = freezeGrabGetNetworkOwner()
if setOwner then
pcall(function() setOwner:FireServer(dragPart, dragPart.CFrame) end)
task.wait(0.1)
end
local dragPart1 = child:FindFirstChild("DragPart1")
if dragPart1 then
local dp1AlignPos = dragPart1:WaitForChild("AlignPosition", 5)
if setOwner then
pcall(function() setOwner:FireServer(dragPart1, dragPart1.CFrame) end)
task.wait(0.1)
end
while workspace:FindFirstChild("GrabParts") and task.wait() do
if dp1AlignPos then
dp1AlignPos.Responsiveness = 200
dp1AlignPos.MaxForce = math.huge
dp1AlignPos.MaxVelocity = math.huge
end
alignOri.Responsiveness = 200
alignOri.MaxTorque = math.huge
end
else
while workspace:FindFirstChild("GrabParts") and task.wait() do
alignPos.Responsiveness = 200
alignOri.Responsiveness = 200
alignPos.MaxForce = math.huge
alignPos.MaxVelocity = math.huge
alignOri.MaxTorque = math.huge
end
end
end)
end

if Toggles.EnableNoclipGrab and Toggles.EnableNoclipGrab.Value then
task.spawn(function()
local dragPart = child:WaitForChild("DragPart", 5)
local setOwner = freezeGrabGetNetworkOwner()
if setOwner and dragPart then
pcall(function() setOwner:FireServer(dragPart, dragPart.CFrame) end)
task.wait(0.1)
end
local dragPart1 = child:FindFirstChild("DragPart1")
if setOwner and dragPart1 then
pcall(function() setOwner:FireServer(dragPart1, dragPart1.CFrame) end)
task.wait(0.1)
end
if dragPart then noclipGrabTargets[dragPart] = dragPart.CanCollide end
if dragPart1 then noclipGrabTargets[dragPart1] = dragPart1.CanCollide end
for _, part in ipairs(targetModel:GetDescendants()) do
if part:IsA("BasePart") then
noclipGrabTargets[part] = part.CanCollide
end
end
local lastGroundedG = nil
local lastFloorYG = nil
while workspace:FindFirstChild("GrabParts") and Toggles.EnableNoclipGrab and Toggles.EnableNoclipGrab.Value and task.wait() do
if dragPart then pcall(function() dragPart.CanCollide = false end) end
if dragPart1 then pcall(function() dragPart1.CanCollide = false end) end
for _, part in ipairs(targetModel:GetDescendants()) do
if part:IsA("BasePart") then
pcall(function() part.CanCollide = false end)
end
end
local myCharG = LocalPlayer.Character
local myHRPG = myCharG and myCharG:FindFirstChild("HumanoidRootPart")
local tRootG = targetModel and (targetModel:FindFirstChild("HumanoidRootPart") or targetModel.PrimaryPart or targetModel:FindFirstChildWhichIsA("BasePart"))
if myHRPG then
local gpG = workspace:FindFirstChild("GrabParts")
local exclG = {myCharG}
if targetModel then table.insert(exclG, targetModel) end
if gpG then table.insert(exclG, gpG) end
for _, plG in ipairs(Players:GetPlayers()) do
local chG = plG.Character
if chG then table.insert(exclG, chG) end
end
local rpG = RaycastParams.new()
rpG.FilterType = Enum.RaycastFilterType.Exclude
rpG.FilterDescendantsInstances = exclG
local hitU = workspace:Raycast(myHRPG.Position, Vector3.new(0, -10, 0), rpG)
if hitU and myHRPG.AssemblyLinearVelocity.Y > -2 then
lastGroundedG = myHRPG.Position
lastFloorYG = hitU.Position.Y
end
local hitF = workspace:Raycast(myHRPG.Position, Vector3.new(0, -150, 0), rpG)
local underMapG = (not hitF) or (hitF.Position.Y > myHRPG.Position.Y + 1)
if tRootG then
local rpT = RaycastParams.new()
rpT.FilterType = Enum.RaycastFilterType.Exclude
rpT.FilterDescendantsInstances = exclG
local hitT = workspace:Raycast(tRootG.Position + Vector3.new(0, 4, 0), Vector3.new(0, -150, 0), rpT)
local sunkY = nil
if hitT and tRootG.Position.Y < hitT.Position.Y + 1 then
sunkY = hitT.Position.Y
elseif not hitT and lastFloorYG and tRootG.Position.Y < lastFloorYG - 2 then
sunkY = lastFloorYG
end
if hitT then lastFloorYG = hitT.Position.Y end
if sunkY then
local pushUpG = (sunkY - tRootG.Position.Y) > 3
for _, partG in ipairs(targetModel:GetDescendants()) do
if partG:IsA("BasePart") then
pcall(function()
local vG = partG.AssemblyLinearVelocity
if vG.Y <= 0 then
partG.AssemblyLinearVelocity = Vector3.new(vG.X, pushUpG and 25 or 0, vG.Z)
end
end)
end
end
end
if (myHRPG.Position - tRootG.Position).Magnitude > 40 then
for partR, origR in pairs(noclipGrabTargets) do
if partR and partR.Parent then pcall(function() partR.CanCollide = origR end) end
end
noclipGrabTargets = {}
pcall(function() combatGrabReleaseHold(targetModel, tRootG) end)
Library:Notify(L("Noclip Grab: линия растянута — захват сброшен", "Noclip Grab: line overstretched — grab released"), 3)
break
end
end
if lastGroundedG and underMapG and myHRPG.Position.Y < lastGroundedG.Y - 12 then
for partR, origR in pairs(noclipGrabTargets) do
if partR and partR.Parent then pcall(function() partR.CanCollide = origR end) end
end
noclipGrabTargets = {}
pcall(function() combatGrabReleaseHold(targetModel, tRootG) end)
pcall(function()
myHRPG.CFrame = CFrame.new(lastGroundedG.X, lastGroundedG.Y + 4, lastGroundedG.Z)
myHRPG.AssemblyLinearVelocity = Vector3.zero
end)
Library:Notify(L("Noclip Grab: вас утянуло под пол — вернул на место", "Noclip Grab: you were dragged under the map — teleported back"), 3)
break
end
end
end
if next(noclipGrabTargets) ~= nil then
for partP, origP in pairs(noclipGrabTargets) do
if partP and partP.Parent then pcall(function() partP.CanCollide = origP end) end
end
noclipGrabTargets = {}
end
local myCharE = LocalPlayer.Character
local myHRPE = myCharE and myCharE:FindFirstChild("HumanoidRootPart")
if myHRPE and lastGroundedG and myHRPE.Position.Y < lastGroundedG.Y - 12 then
local rpE = RaycastParams.new()
local exclE = {myCharE}
if targetModel then table.insert(exclE, targetModel) end
local gpE = workspace:FindFirstChild("GrabParts")
if gpE then table.insert(exclE, gpE) end
rpE.FilterType = Enum.RaycastFilterType.Exclude
rpE.FilterDescendantsInstances = exclE
local hitE = workspace:Raycast(myHRPE.Position, Vector3.new(0, -150, 0), rpE)
if (not hitE) or (hitE.Position.Y > myHRPE.Position.Y + 1) then
pcall(function()
myHRPE.CFrame = CFrame.new(lastGroundedG.X, lastGroundedG.Y + 4, lastGroundedG.Z)
myHRPE.AssemblyLinearVelocity = Vector3.zero
end)
Library:Notify(L("Noclip Grab: вас утянуло под пол — вернул на место", "Noclip Grab: you were dragged under the map — teleported back"), 3)
end
end
end)
end
end)
end)

workspace.ChildRemoved:Connect(function(child)
if child.Name ~= "GrabParts" then return end
if Toggles.EnableFlingGrab and Toggles.EnableFlingGrab.Value and flingGrabTargetPart and flingGrabTargetPart.Parent then
flingGrabApply(flingGrabTargetPart)
end
for primary, bav in pairs(spinGrabTargets) do
if bav and bav.Parent then bav:Destroy() end
end
spinGrabTargets = {}
for obj, data in pairs(invisibleGrabTargets) do
if obj and obj.Parent then
if data.isBeam then
pcall(function() obj.Enabled = data.enabled end)
else
pcall(function() obj.Transparency = data.trans end)
end
end
end
invisibleGrabTargets = {}
for part, origCollide in pairs(noclipGrabTargets) do
if part and part.Parent then
pcall(function() part.CanCollide = origCollide end)
end
end
noclipGrabTargets = {}
flingGrabTargetPart = nil
end)

local StrongThrow = Tabs.Combat:AddLeftGroupbox("Strong Throw + Super Grab", "arrow-up")

StrongThrow:AddToggle("EnableStrongThrow", {
Text = "Strong Throw",
Default = false,
Tooltip = L("ПКМ бросает захваценную цель с усиленной силой", "RMB throws grabbed target with boosted force"),
})

StrongThrow:AddSlider("StrongThrowPower", {
Text = L("Сила броска", "Throw Power"),
Default = 15,
Min = 1,
Max = 1000,
Rounding = 0,
Compact = false,
Tooltip = L("Сила броска ПКМ — это прямое значение скорости, без процентов", "RMB throw power is a direct speed value, not percentage"),
})

strongThrowObj = nil
strongThrowConnections = {}

local function cleanupStrongThrow()
for _, conn in ipairs(strongThrowConnections) do
pcall(function() conn:Disconnect() end)
end
strongThrowConnections = {}
strongThrowObj = nil
end

Toggles.EnableStrongThrow:OnChanged(function()
if not Toggles.EnableStrongThrow.Value then
cleanupStrongThrow()
return
end

table.insert(strongThrowConnections, workspace.ChildAdded:Connect(function(c)
if c.Name == "GrabParts" then
local part = c:FindFirstChild("GrabPart") or c:WaitForChild("GrabPart", 1)
if part then
local weld = part:FindFirstChild("WeldConstraint") or part:WaitForChild("WeldConstraint", 1)
if weld then
strongThrowObj = weld.Part1
end
end
end
end))

table.insert(strongThrowConnections, UserInputService.InputBegan:Connect(function(inp)
if inp.UserInputType == Enum.UserInputType.MouseButton2 then
local obj = strongThrowObj
if obj and obj.Parent then
local bv = Instance.new("BodyVelocity", obj)
local Camera = workspace.CurrentCamera
bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bv.Velocity = Camera.CFrame.LookVector * Options.StrongThrowPower.Value
game:GetService("Debris"):AddItem(bv, 4)
strongThrowObj = nil
Library:Notify(L("Strong Throw: бросок ПКМ", "Strong Throw: RMB throw"), 1)
end
end
end))

table.insert(strongThrowConnections, workspace.ChildRemoved:Connect(function(desc)
if desc.Name == "GrabParts" then
task.delay(1, function()
strongThrowObj = nil
end)
end
end))

Library:Notify(L("Strong Throw ВКЛ: ПКМ бросает цель", "Strong Throw ON: RMB throws target"), 2)
end)

local SuperGrab = StrongThrow 

SuperGrab:AddToggle("EnableSuperGrab", {
Text = "Super Grab",
Default = false,
Tooltip = L("Бросает захваченную цель вперёд при естественном отпускании захвата", "Throws grabbed target forward on natural release"),
})

SuperGrab:AddSlider("SuperGrabPower", {
Text = L("Сила Super Grab", "Super Grab Power"),
Default = 15,
Min = 1,
Max = 1000,
Rounding = 0,
Compact = false,
Tooltip = L("Сила броска при отпускании (1-1000)", "Throw power on release (1-1000)"),
})

SuperGrab:AddDropdown("SuperGrabTargets", {
Text = L("На что действует", "Affects"),
Default = {"Players", "Items"},
Values = {"Players", "Items"},
Multi = true,
Tooltip = L("Выбери на что будет действовать сила", "Choose what the force affects"),
})

superGrabTargetPart = nil

local function applySuperGrab(targetPart)
if not targetPart or not targetPart:IsA("BasePart") or not targetPart.Parent then return end

local targetModel = targetPart.Parent
local throwPart = targetPart
if targetModel and targetModel:FindFirstChildOfClass("Humanoid") and targetModel:FindFirstChild("HumanoidRootPart") then
throwPart = targetModel.HumanoidRootPart
end

if throwPart.Parent == LocalPlayer.Character then return end
local throwPlayer = throwPart.Parent and Players:GetPlayerFromCharacter(throwPart.Parent)
if throwPlayer == LocalPlayer then return end

local power = Options.SuperGrabPower and Options.SuperGrabPower.Value or 100
local cam = workspace.CurrentCamera
if not cam then return end

local dir = cam.CFrame.LookVector
dir = dir
local force = dir * (power * 20)

local setOwner = freezeGrabGetNetworkOwner()
if setOwner then
pcall(function() setOwner:FireServer(throwPart, throwPart.CFrame) end)
end

pcall(function()
throwPart.AssemblyLinearVelocity = force
end)

local bv = Instance.new("BodyVelocity")
bv.Name = "SuperGrabVelocity"
bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bv.P = 100000
bv.Velocity = force
bv.Parent = throwPart
game:GetService("Debris"):AddItem(bv, 0.15)

Library:Notify("Super Grab: бросок (сила " .. tostring(power) .. ")", 1)
end

workspace.ChildAdded:Connect(function(child)
if child.Name == "GrabParts" then
if not (Toggles.EnableSuperGrab and Toggles.EnableSuperGrab.Value) then return end
superGrabTargetPart = nil
local grabPart = child:WaitForChild("GrabPart", 1)
if not grabPart then return end
local weld = grabPart:WaitForChild("WeldConstraint", 1)
if not weld then return end
local targetPart = weld.Part1
local targetModel = targetPart and targetPart.Parent
local targetPlayer = targetModel and Players:GetPlayerFromCharacter(targetModel)
local sgTargets = Options.SuperGrabTargets and Options.SuperGrabTargets.Value or {}
local sgHitPlayers = sgTargets["Players"] or false
local sgHitObjects = sgTargets["Items"] or false
if targetPart and targetPart.Parent and targetModel ~= LocalPlayer.Character and ((sgHitPlayers and targetPlayer and targetPlayer ~= LocalPlayer) or (sgHitObjects and not targetPlayer and not targetPart.Anchored)) then
superGrabTargetPart = targetPart
end
end
end)

workspace.ChildRemoved:Connect(function(child)
if child.Name == "GrabParts" then
if Toggles.EnableSuperGrab and Toggles.EnableSuperGrab.Value and superGrabTargetPart and superGrabTargetPart.Parent then
applySuperGrab(superGrabTargetPart)
end
superGrabTargetPart = nil
end
end)

end
local CombatAuras = Tabs.Combat:AddRightGroupbox("Auras (Targets & Range)", "sparkles")

CombatAuras:AddSlider("AuraDistance", {
Text = "Aura Distance",
Default = 20,
Min = 0,
Max = 30,
Rounding = 0,
Compact = false,
Tooltip = L("Радиус действия всех аур", "Radius for all auras"),
})

CombatAuras:AddToggle("TargetPlayers", {
Text = "Target Players",
Default = true,
Tooltip = L("Действует ли на игроков", "Affects players"),
})

CombatAuras:AddToggle("TargetItems", {
Text = "Target Items",
Default = false,
Tooltip = L("Действует ли на предметы", "Affects items"),
})

CombatAuras:AddToggle("EnableAurasMaster", {
Text = "Auras",
Default = false,
Tooltip = L("Один тогл на все ауры. Выбирай нужные в списке ниже", "One toggle for all auras. Pick the ones you need in the list below"),
})

CombatAuras:AddDropdown("AuraTypes", {
Text = L("Виды аур", "Aura Types"),
Values = {"Fling Aura", "Void Aura", "Kill Aura", "Spin Aura", "Telekinesis Aura", "Click Aura", "Anti Anti Kick Aura", "Anti Banana Aura"},
Multi = true,
Tooltip = L("Выбери ауры: можно одну или сразу все", "Pick auras: one or all of them"),
})

AuraFlagMap = {
["Fling Aura"] = "FlingAura",
["Void Aura"] = "VoidAura",
["Kill Aura"] = "KillAura",
["Spin Aura"] = "SpinAura",
["Telekinesis Aura"] = "TelekinesisAura",
["Click Aura"] = "ClickAura",
["Anti Anti Kick Aura"] = "DestroyAntiKickAura",
["Anti Banana Aura"] = "AntiBananaAura",
}
local function syncAuraToggles()
local master = Toggles.EnableAurasMaster and Toggles.EnableAurasMaster.Value or false
local sel = Options.AuraTypes and Options.AuraTypes.Value or {}
for label, flag in pairs(AuraFlagMap) do
local t = Toggles[flag]
if t then
local want = (master and sel[label] == true) or false
if t.Value ~= want then
pcall(function() t:SetValue(want) end)
end
end
end
end
Toggles.EnableAurasMaster:OnChanged(syncAuraToggles)
Options.AuraTypes:OnChanged(syncAuraToggles)

local function mirrorAuraDropdown()
local dd = Options.AuraTypes
if not dd then return end
local sel = {}
for label, flag in pairs(AuraFlagMap) do
local t = Toggles[flag]
if t and t.Value then sel[label] = true end
end
local cur = dd.Value or {}
local same = true
for k in pairs(cur) do
if not sel[k] then same = false break end
end
if same then
for k in pairs(sel) do
if not cur[k] then same = false break end
end
end
if not same then
pcall(function() dd:SetValue(sel) end)
end
end

local function getAuraTargets(rootPos)
local targets = {}
local dist = Options.AuraDistance.Value

if Toggles.TargetPlayers.Value then
for _, player in pairs(game:GetService("Players"):GetPlayers()) do
if player ~= game:GetService("Players").LocalPlayer and player.Character then
local hrp = player.Character:FindFirstChild("HumanoidRootPart")
if hrp and (hrp.Position - rootPos).Magnitude <= dist then
table.insert(targets, hrp)
end
end
end
end

if Toggles.TargetItems.Value then
for _, obj in ipairs(workspace:GetPartBoundsInRadius(rootPos, dist)) do
if obj:IsA("BasePart") and not obj.Anchored and obj.Parent ~= game:GetService("Players").LocalPlayer.Character then
local isPlayerPart = false
if not Toggles.TargetPlayers.Value then
for _, p in pairs(game:GetService("Players"):GetPlayers()) do
if p.Character and p.Character:IsAncestorOf(obj) then
isPlayerPart = true
break
end
end
end
if not isPlayerPart then
table.insert(targets, obj)
end
end
end
end
return targets
end

local function getMyRoot()
local c = game:GetService("Players").LocalPlayer.Character
return c and c:FindFirstChild("HumanoidRootPart")
end

local function setNetOwner(part)
pcall(function() game:GetService("ReplicatedStorage").GrabEvents.SetNetworkOwner:FireServer(part, part.CFrame) end)
end

CombatAuras:AddToggle("FlingAura", {Text = "Fling Aura", Default = false, Visible = false})
CombatAuras:AddSlider("FlingAuraPower", {Text = "Fling Aura Power", Default = 5, Min = 1, Max = 100, Rounding = 0, Compact = false})
runningFlingAura = false
Toggles.FlingAura:OnChanged(function()
runningFlingAura = Toggles.FlingAura.Value
if runningFlingAura then
task.spawn(function()
while runningFlingAura do
local root = getMyRoot()
if root then
for _, target in ipairs(getAuraTargets(root.Position)) do
pcall(function()
setNetOwner(target)
local pwr = Options.FlingAuraPower.Value
local bv = Instance.new("BodyVelocity", target)
bv.Velocity = Vector3.new(
math.random(-1, 1) * (6 * pwr),
20 * pwr,
math.random(-1, 1) * (6 * pwr)
)
bv.MaxForce = Vector3.one * math.huge
game:GetService("Debris"):AddItem(bv, 0.2)
end)
end
end
task.wait()
end
end)
end
end)

CombatAuras:AddToggle("VoidAura", {Text = "Void Aura", Default = false, Visible = false})
runningVoidAura = false
Toggles.VoidAura:OnChanged(function()
runningVoidAura = Toggles.VoidAura.Value
if runningVoidAura then
task.spawn(function()
while runningVoidAura do
local root = getMyRoot()
if root then
for _, target in ipairs(getAuraTargets(root.Position)) do
pcall(function()
setNetOwner(target)
target.CFrame = target.CFrame - Vector3.new(0, 1500, 0)
local bv = Instance.new("BodyVelocity", target)
bv.Velocity = Vector3.new(0, -9e9, 0)
bv.MaxForce = Vector3.one * math.huge
game:GetService("Debris"):AddItem(bv, 0.2)
end)
end
end
task.wait(0.05)
end
end)
end
end)

CombatAuras:AddToggle("KillAura", {Text = "Kill Aura", Default = false, Visible = false})
runningKillAura = false
killAuraTask = nil
Toggles.KillAura:OnChanged(function()
runningKillAura = Toggles.KillAura.Value
if runningKillAura then
killAuraTask = task.spawn(function()
local GE = game:GetService("ReplicatedStorage"):WaitForChild("GrabEvents")
local SetNetworkOwner = GE:WaitForChild("SetNetworkOwner")
local DestroyGrabLine = GE:WaitForChild("DestroyGrabLine")
while runningKillAura do
local root = getMyRoot()
if root then
for _, target in ipairs(getAuraTargets(root.Position)) do
pcall(function()
local targetModel = target.Parent
local head = targetModel:FindFirstChild("Head")
local hum = targetModel:FindFirstChildOfClass("Humanoid")
if hum and hum.Health > 0 and head then
SetNetworkOwner:FireServer(target, target.CFrame)
task.wait(0.1)
DestroyGrabLine:FireServer(target)
if head:FindFirstChild("PartOwner") and head.PartOwner.Value == LocalPlayer.Name then
for _, part in pairs(targetModel:GetChildren()) do
if part:IsA("BasePart") then
part.CFrame = CFrame.new(-1000000000, 1000000000, -1000000000)
end
end
task.wait()
for _, part in pairs(targetModel:GetChildren()) do
if part:IsA("BasePart") then
part.CFrame = CFrame.new(-1000000000, 1000000000, -1000000000)
end
end
local bv = Instance.new("BodyVelocity")
bv.Velocity = Vector3.new(0, -9999999, 0)
bv.MaxForce = Vector3.new(9000000000, 9000000000, 9000000000)
bv.P = 100000075
bv.Parent = target
hum.Sit = false
hum.Jump = true
hum.BreakJointsOnDeath = false
hum:ChangeState(Enum.HumanoidStateType.Dead)
task.delay(2, function()
if bv and bv.Parent then bv:Destroy() end
end)
end
end
end)
end
end
task.wait(0.1)
end
end)
else
if killAuraTask then task.cancel(killAuraTask) killAuraTask = nil end
end
end)

CombatAuras:AddToggle("SpinAura", {Text = "Spin Aura", Default = false, Visible = false})
CombatAuras:AddSlider("SpinAuraSpeed", {Text = "Spin Speed", Default = 5, Min = 1, Max = 100, Rounding = 0, Compact = false})
CombatAuras:AddSlider("SpinAuraHeight", {Text = "Spin Height", Default = 5, Min = -50, Max = 50, Rounding = 0, Compact = false})
runningSpinAura = false
spinAuraTask = nil
Toggles.SpinAura:OnChanged(function()
runningSpinAura = Toggles.SpinAura.Value
if runningSpinAura then
spinAuraTask = task.spawn(function()
local spinAngle = 0
while runningSpinAura do
pcall(function()
local root = getMyRoot()
if not root then return end
local spd = Options.SpinAuraSpeed.Value
local heightVal = Options.SpinAuraHeight.Value
spinAngle = spinAngle + (spd / 100)
if spinAngle >= 6.28 then spinAngle = 0 end

local isOrbitMode = heightVal >= 20
local radius = isOrbitMode and 6 or 10
local yOffset = isOrbitMode and math.max(heightVal, 5) or heightVal

for _, target in ipairs(getAuraTargets(root.Position)) do
local targetPosition = root.Position + Vector3.new(
math.cos(spinAngle) * radius,
yOffset,
math.sin(spinAngle) * radius
)
pcall(function()
setNetOwner(target)
local p = target.Parent
local hum = p and p:FindFirstChildOfClass("Humanoid")
if hum then
hum.PlatformStand = true
end
local bv = target:FindFirstChild("SpinAuraBV") or Instance.new("BodyVelocity")
bv.Name = "SpinAuraBV"
local pullStrength = isOrbitMode and 35 or 25
bv.Velocity = (targetPosition - target.Position) * pullStrength
bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bv.P = 10000
bv.Parent = target
local bg = target:FindFirstChild("SpinAuraBG") or Instance.new("BodyGyro")
bg.Name = "SpinAuraBG"
bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
bg.P = 10000
bg.D = 500
bg.CFrame = CFrame.new(target.Position, root.Position)
bg.Parent = target
end)
end
end)
task.wait(0.02)
end
end)
else
if spinAuraTask then task.cancel(spinAuraTask) spinAuraTask = nil end
for _, p in pairs(game:GetService("Players"):GetPlayers()) do
if p.Character then
local hum = p.Character:FindFirstChildOfClass("Humanoid")
if hum then hum.PlatformStand = false end
local t = p.Character:FindFirstChild("HumanoidRootPart") or p.Character:FindFirstChild("Torso")
if t then
if t:FindFirstChild("SpinAuraBV") then t.SpinAuraBV:Destroy() end
if t:FindFirstChild("SpinAuraBG") then t.SpinAuraBG:Destroy() end
end
end
end
end
end)

CombatAuras:AddToggle("TelekinesisAura", {Text = "Telekinesis Aura", Default = false, Visible = false})
runningTKAura = false
tkAuraTask = nil
Toggles.TelekinesisAura:OnChanged(function()
runningTKAura = Toggles.TelekinesisAura.Value
if runningTKAura then
tkAuraTask = task.spawn(function()
local GE = game:GetService("ReplicatedStorage"):WaitForChild("GrabEvents")
local SetNetworkOwner = GE:WaitForChild("SetNetworkOwner")
local Camera = workspace.CurrentCamera
while runningTKAura do
pcall(function()
local root = getMyRoot()
if not root then return end
local lookDir = Camera.CFrame.LookVector
for _, target in ipairs(getAuraTargets(root.Position)) do
pcall(function()
local targetModel = target.Parent
for _, desc in ipairs(targetModel:GetDescendants()) do
if desc:IsA("BasePart") then desc.CanCollide = false end
end
SetNetworkOwner:FireServer(target, root.CFrame)
local pos = target:FindFirstChild("HellAuraPos") or Instance.new("BodyPosition")
pos.Name = "HellAuraPos"
pos.MaxForce = Vector3.new(100000, 100000, 100000)
pos.D = 500
pos.P = 50000
pos.Position = root.Position + lookDir * 15 + Vector3.new(0, 5, 0)
pos.Parent = target
local gyro = target:FindFirstChild("HellAuraGyro") or Instance.new("BodyGyro")
gyro.Name = "HellAuraGyro"
gyro.MaxTorque = Vector3.new(100000, 100000, 100000)
gyro.D = 500
gyro.P = 50000
gyro.CFrame = CFrame.new(target.Position, root.Position)
gyro.Parent = target
end)
end
end)
task.wait(0.05)
end
end)
else
if tkAuraTask then task.cancel(tkAuraTask) tkAuraTask = nil end
for _, p in pairs(game:GetService("Players"):GetPlayers()) do
if p.Character then
local t = p.Character:FindFirstChild("HumanoidRootPart") or p.Character:FindFirstChild("Torso")
if t then
if t:FindFirstChild("HellAuraPos") then t.HellAuraPos:Destroy() end
if t:FindFirstChild("HellAuraGyro") then t.HellAuraGyro:Destroy() end
end
end
end
for _, obj in ipairs(workspace:GetDescendants()) do
if obj:IsA("BasePart") then
if obj:FindFirstChild("HellAuraPos") then obj.HellAuraPos:Destroy() end
if obj:FindFirstChild("HellAuraGyro") then obj.HellAuraGyro:Destroy() end
end
end
end
end)

CombatAuras:AddToggle("ClickAura", {Text = "Click Aura", Default = false, Visible = false})
clickAuraEnabled = false
clickAuraTask = nil
Toggles.ClickAura:OnChanged(function()
clickAuraEnabled = Toggles.ClickAura.Value
if clickAuraEnabled then
clickAuraTask = task.spawn(function()
while clickAuraEnabled do
pcall(function()
local root = getMyRoot()
if not root then return end
for _, target in ipairs(getAuraTargets(root.Position)) do
pcall(function()
setNetOwner(target)
if not target:FindFirstChild("ClickAuraBV") then
local bv = Instance.new("BodyVelocity")
bv.Name = "ClickAuraBV"
bv.Velocity = Vector3.zero
bv.MaxForce = Vector3.one * math.huge
bv.Parent = target
local bg = Instance.new("BodyGyro")
bg.Name = "ClickAuraBG"
bg.MaxTorque = Vector3.one * math.huge
bg.CFrame = CFrame.new(target.Position, target.Position + Vector3.new(0, 0, 1))
bg.Parent = target
end
end)
end
end)
task.wait(0.05)
end
end)
else
if clickAuraTask then task.cancel(clickAuraTask) clickAuraTask = nil end
end
end)

CombatAuras:AddToggle("DestroyAntiKickAura", {Text = "Anti Anti Kick Aura", Default = false, Visible = false})
runningDestroyAntiKickAura = false
Toggles.DestroyAntiKickAura:OnChanged(function()
runningDestroyAntiKickAura = Toggles.DestroyAntiKickAura.Value
if runningDestroyAntiKickAura then
task.spawn(function()
local GE = game:GetService("ReplicatedStorage"):WaitForChild("GrabEvents")
local setNE = GE:WaitForChild("SetNetworkOwner")
local createGL = GE:WaitForChild("CreateGrabLine")
local destroyGL = GE:WaitForChild("DestroyGrabLine")
local auraFrame = 0
while runningDestroyAntiKickAura do
local root = getMyRoot()
if root then
for _, obj in ipairs(workspace:GetPartBoundsInRadius(root.Position, Options.AuraDistance.Value)) do
if obj:IsA("BasePart") and obj.Name == "StickyPart" and not game:GetService("Players").LocalPlayer.Character:IsAncestorOf(obj) then
if auraFrame % 3 == 0 then pcall(function() setNE:FireServer(obj, obj.CFrame) end)
elseif auraFrame % 3 == 1 then pcall(function() createGL:FireServer(obj, Vector3.zero, obj.Position, false) end)
else pcall(function() destroyGL:FireServer(obj) end) end
end
end
auraFrame = auraFrame + 1
end
task.wait()
end
end)
end
end)

CombatAuras:AddToggle("AntiBananaAura", {Text = "Anti Banana Aura", Default = false, Visible = false})
runningAntiBananaAura = false
Toggles.AntiBananaAura:OnChanged(function()
runningAntiBananaAura = Toggles.AntiBananaAura.Value
if runningAntiBananaAura then
task.spawn(function()
local GE = game:GetService("ReplicatedStorage"):WaitForChild("GrabEvents")
local setNE = GE:WaitForChild("SetNetworkOwner")
local createGL = GE:WaitForChild("CreateGrabLine")
local destroyGL = GE:WaitForChild("DestroyGrabLine")
local auraFrame = 0
while runningAntiBananaAura do
local root = getMyRoot()
if root then
for _, obj in ipairs(workspace:GetPartBoundsInRadius(root.Position, Options.AuraDistance.Value)) do
if obj:IsA("BasePart") and obj.Name == "HitboxPart" and not game:GetService("Players").LocalPlayer.Character:IsAncestorOf(obj) then
if auraFrame % 3 == 0 then pcall(function() setNE:FireServer(obj, obj.CFrame) end)
elseif auraFrame % 3 == 1 then pcall(function() createGL:FireServer(obj, Vector3.zero, obj.Position, false) end)
else pcall(function() destroyGL:FireServer(obj) end) end

pcall(function()
local direction = (obj.Position - root.Position).Unit
local bv = Instance.new("BodyVelocity")
bv.Velocity = direction * 5000 + Vector3.new(0, 2000, 0)
bv.MaxForce = Vector3.one * math.huge
bv.Parent = obj
game:GetService("Debris"):AddItem(bv, 0.3)
end)
end
end
auraFrame = auraFrame + 1
end
task.wait()
end
end)
end
end)

for _, flag in pairs(AuraFlagMap) do
local t = Toggles[flag]
if t and type(t.SetValue) == "function" and not t._auraMirror then
local orig = t.SetValue
t._auraMirror = true
t.SetValue = function(self, v)
orig(self, v)
mirrorAuraDropdown()
end
end
end

do
_lagAutoBlock = true

local lagSection = Tabs.Combat:AddRightGroupbox("Server Lag", "radio-tower")

local function getGrabEventsFolder()
return game:GetService("ReplicatedStorage"):FindFirstChild("GrabEvents")
end

lagSection:AddSlider("LineLagPower", {
Text = "Line Lag Ping",
Default = 5000,
Min = 0,
Max = 10000,
Rounding = 0,
Compact = false,
Tooltip = L("Пинг в мс — чем выше, тем сильнее лаг", "Ping in ms - higher = more lag"),
})

local lineLagEnabled = false
local lineLagPart = nil

local function startLineLag()
if lineLagEnabled then return end
lineLagEnabled = true
pcall(notifyLineLagStart)
task.spawn(function()
local grabEvents = getGrabEventsFolder()
if not grabEvents then lineLagEnabled = false; return end
local createLine = grabEvents:FindFirstChild("CreateGrabLine")
if not createLine then lineLagEnabled = false; return end
while lineLagEnabled do
local pingMs = Options.LineLagPower and Options.LineLagPower.Value or 5000
local power = math.floor(pingMs / 10)
local spawnLocation = workspace:FindFirstChild("SpawnLocation") or workspace:FindFirstChild("Spawn") or (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart"))
if not spawnLocation then task.wait(); continue end
lineLagPart = spawnLocation
for i = 1, power do
if not lineLagEnabled then break end
local randomX = math.random(-1e9, 1e9)
local randomZ = math.random(-1e9, 1e9)
local directions = {
CFrame.new(randomX, 0, randomZ),
CFrame.new(-randomX, 0, -randomZ),
CFrame.new(randomX, 0, -randomZ),
CFrame.new(-randomX, 0, randomZ)
}
for _, pos in pairs(directions) do
pcall(function() createLine:FireServer(spawnLocation, pos) end)
end
task.wait()
end
task.wait()
end
lineLagEnabled = false
end)
end

local function purgeGrabLines(part)
if not part then return end
local grabEvents = getGrabEventsFolder()
if not grabEvents then return end
local createLine = grabEvents:FindFirstChild("CreateGrabLine")
local destroyLine = grabEvents:FindFirstChild("DestroyGrabLine")
if not createLine or not destroyLine then return end
task.spawn(function()
pcall(function() createLine:FireServer(part, CFrame.new(3e9, 4e9, 2e9)) end)
task.wait()
for _ = 1, 8 do
pcall(function() destroyLine:FireServer(part) end)
task.wait(0.1)
end
end)
end

local function stopLineLag()
lineLagEnabled = false
pcall(purgeGrabLines, lineLagPart)
lineLagPart = nil
pcall(notifyLineLagStop)
end

lagSection:AddToggle("EnableLineLag", {
Text = "Line Lag",
Default = false,
Tooltip = L("Спамит CreateGrabLine с огромными координатами", "Spams CreateGrabLine with huge coordinates"),
Callback = function(v)
if v and _lagAutoBlock then return end
if v then startLineLag() else stopLineLag() end
end,
})

lagSection:AddSlider("PacketLagMB", {
Text = "Size of Packets to PACKET LAG",
Default = 0.1,
Min = 0.01,
Max = 20,
Rounding = 2,
Compact = false,
Tooltip = L("Метод NoName: тик каждые 0.1 сек. До 1.6 МБ — один пакет за тик; выше тик делится на пакеты по ~0.8 МБ (20 МБ = 15 пакетов = ~116 МБ/сек трафика — при аплоаде ниже лагает ТВОЙ клиент, опусти слайдер)", "NoName method: one tick every 0.1 sec. Up to 1.6 MB - one packet per tick; above that the tick is split into ~0.8 MB packets (20 MB = 15 packets = ~116 MB/s of traffic - if your upload is lower YOUR client lags, lower the slider)"),
})

local _packetRedeemLabel = lagSection:AddLabel("🛜🔽 Redeemed packets: 0")
local _packetSentLabel = lagSection:AddLabel("🛜🔼 Sent packets: 0")

local _packetLagTask = nil
local _packetSentCount = 0
local _packetRedeemCount = 0
local _packetWasLagged = false
local _packetDetectorConn = nil

local NN_PACKET_STRING = "metaballs metaballs metaballs metaballs metaballs metaballs metaballs metaballs"

local NN_MAX_REPEATS = 10126

local function fireOnePacket()
local mb = (Options.PacketLagMB and Options.PacketLagMB.Value) or 0.1
local totalRepeats = math.max(1, math.floor(mb * 1024 * 1024 / 143))
local packets = math.max(1, math.ceil(totalRepeats / NN_MAX_REPEATS))
local base = math.floor(totalRepeats / packets)
local extra = totalRepeats - base * packets
local grabEvents = getGrabEventsFolder()
if not grabEvents then return end
local extendGL = grabEvents:FindFirstChild("ExtendGrabLine")
if not extendGL then return end
for i = 1, packets do
local n = base + (i <= extra and 1 or 0)
if n > NN_MAX_REPEATS then n = NN_MAX_REPEATS end
extendGL:FireServer(string.rep(NN_PACKET_STRING, n))
_packetSentCount = _packetSentCount + 1
if i < packets then
task.wait()
end
end
if _packetSentLabel then
pcall(function() _packetSentLabel:SetText("🛜🔼 Sent packets: " .. _packetSentCount) end)
end
end

local function startPacketLag()
if _packetLagTask then return end
pcall(notifyPacketLagStart)
_packetLagTask = task.spawn(function()
while true do
task.wait(0.1)
pcall(fireOnePacket)
end
end)
end

local function stopPacketLag()
if _packetLagTask then
task.cancel(_packetLagTask)
_packetLagTask = nil
end
pcall(notifyPacketLagStop)
end

local function fireSinglePacket()
task.spawn(fireOnePacket)
end

do
local grabEvents0 = getGrabEventsFolder()
local extendGL0 = grabEvents0 and grabEvents0:FindFirstChild("ExtendGrabLine")
if extendGL0 then
_packetDetectorConn = extendGL0.OnClientEvent:Connect(function(arg1, data)
if typeof(data) == "string" and not _packetWasLagged then
_packetWasLagged = true
local stringLen = string.len(data)
if stringLen > 300 then
_packetRedeemCount = _packetRedeemCount + 1
if _packetRedeemLabel then
pcall(function() _packetRedeemLabel:SetText("🛜🔽 Redeemed packets: " .. _packetRedeemCount) end)
end
if Toggles.LagDetectNotify and Toggles.LagDetectNotify.Value then
local sizeRounded = math.round((stringLen / (1024 * 1024)) * 1000) / 1000
Library:Notify({
Title = "Oblivion",
Description = "PACKET LAGS DETECTED\nSource: " .. tostring(arg1) .. "\nSize: " .. tostring(sizeRounded) .. " MB",
Time = 5,
})
end
end
task.delay(5, function()
_packetWasLagged = false
end)
end
end)
end
end

lagSection:AddToggle("EnablePacketLag", {
Text = "Packet Lag",
Default = false,
Tooltip = L("Метод NoName: metaballs-пакеты каждые 0.1 сек, размер тика = Packet Size (MB)", "NoName method: metaballs packets every 0.1 sec, tick size = Packet Size (MB)"),
Callback = function(v)
if v and _lagAutoBlock then return end
if v then startPacketLag() else stopPacketLag() end
end,
})

lagSection:AddButton("SinglePacketLag", {
Text = "1 Packet Lag",
Tooltip = L("Отправить 1 порцию (Packet Size МБ, кусками по ~0.8 МБ)", "Send 1 dose (Packet Size MB, in ~0.8 MB chunks)"),
Callback = function()
fireSinglePacket()
end,
})

lagSection:AddLabel("1 Packet Key"):AddKeyPicker("SinglePacketKey", {
Default = "None",
SyncToggleState = false,
Mode = "Press",
Text = "1 Packet Key",
NoUI = false,
})

local uis = game:GetService("UserInputService")
uis.InputBegan:Connect(function(input, gpe)
if gpe then return end
local key = Options.SinglePacketKey and Options.SinglePacketKey.Value
if not key or key == "None" then return end
local isMatch = false
if input.KeyCode.Name == key or input.UserInputType.Name == key then isMatch = true end
if key == "MB4" and input.KeyCode and input.KeyCode.Name == "ButtonX1" then isMatch = true end
if key == "MB5" and input.KeyCode and input.KeyCode.Name == "ButtonX2" then isMatch = true end
if (key == "RightClick" or key == "MB2" or key == "MouseButton2") and input.UserInputType == Enum.UserInputType.MouseButton2 then isMatch = true end
if (key == "LeftClick" or key == "MB1" or key == "MouseButton1") and input.UserInputType == Enum.UserInputType.MouseButton1 then isMatch = true end
if isMatch then
fireSinglePacket()
end
end)
end

do
DefSection = Tabs.Def:AddLeftGroupbox("", "shield", nil, nil, true)
DefAntiGrab = DefSection

DefAntiGrab:AddToggle("EnableAntiGrab", {
Text = "Anti Grab [BEST]",
Default = false,
Tooltip = L("Ходишь не замечая что тебя берут", "Walk without being affected by grabs"),
})

antiGrabRS = game:GetService("ReplicatedStorage")
antiGrabProc = false
antiGrabConns = {}

function antiGrabStruggleEvent()
local ce = antiGrabRS:FindFirstChild("CharacterEvents")
return ce and ce:FindFirstChild("Struggle")
end

function antiGrabRagdollRemote()
local ce = antiGrabRS:FindFirstChild("CharacterEvents")
return ce and (ce:FindFirstChild("RagdollRemote") or ce:WaitForChild("RagdollRemote", 5))
end

function antiGrabStopVelocityEvent()
local ge = antiGrabRS:FindFirstChild("GameCorrectionEvents")
return ge and ge:FindFirstChild("StopAllVelocity")
end

function antiGrabIsHeld()
local char = LocalPlayer.Character
if not char then return false end
local head = char:FindFirstChild("Head")
local po = head and head:FindFirstChild("PartOwner")
if po then
local ownerName = tostring(po.Value)
if ownerName ~= "" and ownerName ~= LocalPlayer.Name then
local ownerPlayer = game:GetService("Players"):FindFirstChild(ownerName)
if ownerPlayer and ownerPlayer.Character then
for _, obj in ipairs(workspace:GetChildren()) do
if obj.Name == "CreatureBlobman" and obj:IsA("Model") then
if ownerPlayer.Character:IsDescendantOf(obj) then return false end
end
end
end
return true
end
end
local isHeld = LocalPlayer:FindFirstChild("IsHeld")
if isHeld and isHeld.Value then return true end
return false
end

function antiGrabDisableRagdoll(char)
for _, v in pairs(char:GetChildren()) do
if v:IsA("BasePart") and v:FindFirstChild("BallSocketConstraint") and v.Name ~= "Head" then
v.BallSocketConstraint.Enabled = false
if v:FindFirstChild("RagdollLimbPart") then
v.RagdollLimbPart.WeldConstraint.Enabled = false
end
end
end
end

function antiGrabEnableRagdoll(char)
for _, v in pairs(char:GetChildren()) do
if v:IsA("BasePart") and v:FindFirstChild("BallSocketConstraint") and v.Name ~= "Head" then
v.BallSocketConstraint.Enabled = true
if v:FindFirstChild("RagdollLimbPart") then
v.RagdollLimbPart.WeldConstraint.Enabled = true
end
end
end
end

function antiGrabApply(char)
if not char then return end
local hrp = char:FindFirstChild("HumanoidRootPart")
local hum = char:FindFirstChild("Humanoid")
local head = char:FindFirstChild("Head")
if not (hrp and hum and head) then return end

antiGrabDisableRagdoll(char)

if antiGrabConns["Head"] then antiGrabConns["Head"]:Disconnect() end
antiGrabConns["Head"] = head.ChildAdded:Connect(function(partOwner)
if partOwner.Name ~= "PartOwner" or antiGrabProc then return end
antiGrabProc = true
hum.Sit = false

local struggle = antiGrabStruggleEvent()
local ragdollRemote = antiGrabRagdollRemote()

hrp.Anchored = true
task.spawn(function()
while (head and head:FindFirstChild("PartOwner")) or (LocalPlayer:FindFirstChild("IsHeld") and LocalPlayer.IsHeld.Value) do
if struggle then pcall(function() struggle:FireServer(LocalPlayer) end) end
if ragdollRemote then pcall(function() ragdollRemote:FireServer(hrp, 0) end) end
pcall(function()
hrp.CFrame = hrp.CFrame + hum.MoveDirection * (hum.WalkSpeed / 60)
hum.PlatformStand = false
hum.Sit = false
hum.AutoRotate = true
end)
if hrp:FindFirstChild("WeldHRP") and hrp.WeldHRP.Enabled then
pcall(function() head.CFrame = hrp.CFrame + Vector3.new(0, 1.35, 0) end)
end
task.wait()
end
pcall(function() hrp.Anchored = false end)
task.spawn(function()
task.wait(0.3)
local ge = antiGrabRS:FindFirstChild("GrabEvents")
local dgl = ge and ge:FindFirstChild("DestroyGrabLine")
if dgl then
for _, plr in ipairs(game:GetService("Players"):GetPlayers()) do
if plr ~= LocalPlayer and plr.Character then
local tRoot = plr.Character:FindFirstChild("HumanoidRootPart")
local tHead = plr.Character:FindFirstChild("Head")
if tRoot then
for _ = 1, 3 do
pcall(function() dgl:FireServer(tRoot) end)
end
end
if tHead then
pcall(function() dgl:FireServer(tHead) end)
end
for _, v in ipairs(plr.Character:GetDescendants()) do
if v.Name == "PartOwner" then
pcall(function() dgl:FireServer(v.Parent) end)
end
end
end
end
end
end)
antiGrabProc = false
end)
end)

if antiGrabConns["Hum"] then antiGrabConns["Hum"]:Disconnect() end
antiGrabConns["Hum"] = hum.Changed:Connect(function(prop)
if prop == "Sit" and hum.Sit then
if not (hum.SeatPart and hum.SeatPart.Parent and hum.SeatPart.Parent.Name == "CreatureBlobman") then
hum:SetStateEnabled(Enum.HumanoidStateType.Jumping, true)
hum.Sit = false
end
end
end)

local weldHRP = hrp:FindFirstChild("WeldHRP")
if weldHRP then
if antiGrabConns["Weld"] then antiGrabConns["Weld"]:Disconnect() end
antiGrabConns["Weld"] = weldHRP.Changed:Connect(function()
if not hrp.WeldHRP.Enabled then return end
task.spawn(function()
while not hum.Sit do task.wait() end
hum.Sit = false
hum.AutoRotate = true
while hrp.WeldHRP.Enabled do
pcall(function() head.CFrame = hrp.CFrame + Vector3.new(0, 1.35, 0) end)
task.wait()
end
end)
end)
end

local ragdolled = hum:FindFirstChild("Ragdolled")
if ragdolled then
if antiGrabConns["Ragdoll"] then antiGrabConns["Ragdoll"]:Disconnect() end
antiGrabConns["Ragdoll"] = ragdolled.Changed:Connect(function()
if hum.Ragdolled.Value then
antiGrabDisableRagdoll(char)
pcall(function() hum:ChangeState(Enum.HumanoidStateType.GettingUp) end)
end
end)
end
end

function antiGrabCleanup()
for k, conn in pairs(antiGrabConns) do
if conn then conn:Disconnect() end
end
antiGrabConns = {}
antiGrabProc = false
end

Toggles.EnableAntiGrab:OnChanged(function()
if Toggles.EnableAntiGrab.Value then
antiGrabCleanup()
antiGrabApply(LocalPlayer.Character)
antiGrabConns["CharAdded"] = LocalPlayer.CharacterAdded:Connect(function(newChar)
task.wait(0.5)
antiGrabApply(newChar)
end)
else
antiGrabCleanup()
local char = LocalPlayer.Character
if char then
antiGrabEnableRagdoll(char)
local hrp = char:FindFirstChild("HumanoidRootPart")
if hrp then hrp.Anchored = false end
end
end
end)

DefAntiLoop = DefSection

DefAntiLoop:AddToggle("EnableAntiLoopTP", {
Text = "Anti Loop TP",
Default = false,
Tooltip = L("Возвращает на место, если читер пытается кидать тебя по карте", "Returns you to position if someone tries to fling you"),
})

DefAntiLoop:AddSlider("AntiLoopThreshold", {
Text = "Порог телепорт����",
Default = 50,
Min = 20,
Max = 200,
Rounding = 0,
Compact = false,
Tooltip = L("Если перемещение за кадр больше этого числа, тебя вернёт обратно", "If movement per frame exceeds this, you get teleported back"),
})

lastSafeCF = nil
antiLoopConn = nil

Toggles.EnableAntiLoopTP:OnChanged(function()
if Toggles.EnableAntiLoopTP.Value then
lastSafeCF = nil
if antiLoopConn then antiLoopConn:Disconnect() end
antiLoopConn = RunService.Heartbeat:Connect(function()
local char = LocalPlayer.Character
local hrp = char and char:FindFirstChild("HumanoidRootPart")
local hum = char and char:FindFirstChildOfClass("Humanoid")
if not hrp or not hum then return end

local threshold = Options.AntiLoopThreshold and Options.AntiLoopThreshold.Value or 50

local isFlying = (Toggles.EnableFly and Toggles.EnableFly.Value) or (Toggles.EnableBlobFly and Toggles.EnableBlobFly.Value)

local isGrounded = (hum.FloorMaterial ~= Enum.Material.Air) or isFlying
local vel = hrp.AssemblyLinearVelocity.Magnitude

if isGrounded and vel < (isFlying and 500 or 150) then
lastSafeCF = hrp.CFrame
end

if not lastSafeCF then return end

local dist = (hrp.Position - lastSafeCF.Position).Magnitude

if dist > threshold or (not isFlying and vel > 350) then
pcall(function()
hum.Sit = false
hum.PlatformStand = false
end)
local ce = game:GetService("ReplicatedStorage"):FindFirstChild("CharacterEvents")
local struggle = ce and ce:FindFirstChild("Struggle")
if struggle then pcall(function() struggle:FireServer(LocalPlayer) end) end

hrp.AssemblyLinearVelocity = Vector3.zero
hrp.AssemblyAngularVelocity = Vector3.zero
hrp.CFrame = lastSafeCF
end
end)
else
if antiLoopConn then antiLoopConn:Disconnect() end
antiLoopConn = nil
lastSafeCF = nil
end
end)

DefAntiExplode = DefSection

DefAntiExplode:AddToggle("EnableAntiExplode", {
Text = "Anti Explosion",
Default = false,
Tooltip = L("Гасит отлёт от взрывов момбы: при взрыве тебя не швыряет", "Negates explosion knockback from Momba"),
})

antiExplodeRS = game:GetService("ReplicatedStorage")
antiExplodeConn = nil

function antiExplodeStopVelocityEvent()
local ge = antiExplodeRS:FindFirstChild("GameCorrectionEvents")
return ge and ge:FindFirstChild("StopAllVelocity")
end

function antiExplodeIsRagdolled(hum)
local r = hum and hum:FindFirstChild("Ragdolled")
return r ~= nil and r.Value == true
end

function antiExplodeKillVelocity(char)
for _, part in ipairs(char:GetDescendants()) do
if part:IsA("BasePart") then
pcall(function()
part.AssemblyLinearVelocity = Vector3.zero
part.AssemblyAngularVelocity = Vector3.zero
end)
end
end
end

function antiExplodeTick()
local char = LocalPlayer.Character
if not char then return end
local hum = char:FindFirstChildOfClass("Humanoid")
if antiExplodeIsRagdolled(hum) then
local stopVel = antiExplodeStopVelocityEvent()
if stopVel then pcall(function() stopVel:FireServer() end) end
antiExplodeKillVelocity(char)
end
end

Toggles.EnableAntiExplode:OnChanged(function()
if Toggles.EnableAntiExplode.Value then
if antiExplodeConn then antiExplodeConn:Disconnect() end
antiExplodeConn = RunService.Heartbeat:Connect(antiExplodeTick)
else
if antiExplodeConn then antiExplodeConn:Disconnect() end
antiExplodeConn = nil
end
end)

DefAntiLag = DefSection

DefAntiLag:AddToggle("EnableAntiLag", {
Text = "Anti Lag",
Default = false,
Tooltip = L("Убирает лаги от спама грабящих линий (lag server): чистит и глушит отрисовку лучей", "Removes lag from grab line spam: cleans and suppresses beam rendering"),
})

antiLagConn = nil 
antiLagBeamScript = nil 

function antiLagIsLine(obj)
if obj:IsA("Beam") then return true end
local n = string.lower(obj.Name)
return n:find("grabbeam") ~= nil or n:find("grabline") ~= nil
end

function antiLagSweep()
for _, obj in ipairs(workspace:GetDescendants()) do
if antiLagIsLine(obj) then
pcall(function() obj:Destroy() end)
end
end
end

Toggles.EnableAntiLag:OnChanged(function()
if Toggles.EnableAntiLag.Value then
local ps = LocalPlayer:FindFirstChild("PlayerScripts")
local beamScript = ps and ps:FindFirstChild("CharacterAndBeamMove")
if beamScript then
antiLagBeamScript = beamScript
pcall(function() beamScript.Disabled = true end)
end
antiLagSweep()
if antiLagConn then antiLagConn:Disconnect() end
antiLagConn = workspace.DescendantAdded:Connect(function(obj)
if not (Toggles.EnableAntiLag and Toggles.EnableAntiLag.Value) then return end
if antiLagIsLine(obj) then
pcall(function() obj:Destroy() end)
end
end)
Library:Notify(L("Anti Lag ВКЛ — чищу спам линий", "Anti Lag ON — clearing line spam"), 2)
else
if antiLagConn then antiLagConn:Disconnect(); antiLagConn = nil end
if antiLagBeamScript then
pcall(function() antiLagBeamScript.Disabled = false end)
antiLagBeamScript = nil
end
Library:Notify(L("Anti Lag ВЫКЛ", "Anti Lag OFF"), 2)
end
end)

do
DefAntiKick = Tabs.Def:AddLeftGroupbox("Anti Kick", "shield-check")

local LimbSelfSection = Tabs.Def:AddLeftGroupbox(L("Отрубание конечностей", "Limb Removal (Self)"), "scissors")

LimbSelfSection:AddLabel(L("Ноги: попытка удаления НА СЕРВЕРЕ (пропадут у всех игроков)", "Legs: tries REAL server removal (gone for everyone)"), false)

local function limbSelfGetJoints(names)
local char = game.Players.LocalPlayer.Character
if not char then return {} end
local wanted = {}
for _, nm in ipairs(names) do
local limb = char:FindFirstChild(nm)
if limb and limb:IsA("BasePart") then
wanted[limb] = { limb = limb, motors = {} }
end
end
for _, d in ipairs(char:GetDescendants()) do
if d:IsA("Motor6D") then
local j0 = wanted[d.Part0]
if j0 then table.insert(j0.motors, d) end
local j1 = wanted[d.Part1]
if j1 then table.insert(j1.motors, d) end
end
end
local list = {}
for _, w in pairs(wanted) do
table.insert(list, w)
end
return list
end

local function limbSelfAlive(joints)
for _, w in ipairs(joints) do
if w.limb.Parent then return true end
end
return false
end

local function limbSelfRemotes()
local RS = game:GetService("ReplicatedStorage")
local mt = RS:FindFirstChild("MenuToys")
local ce = RS:FindFirstChild("CharacterEvents")
local destroyToy = mt and mt:FindFirstChild("DestroyToy")
local ragdollRE = ce and ce:FindFirstChild("RagdollRemote")
return destroyToy, ragdollRE
end

local function limbSelfRemove(names)
local char = game.Players.LocalPlayer.Character
local hum = char and char:FindFirstChildOfClass("Humanoid")
if not (char and hum) then
Library:Notify(L("Конечности: персонаж не найден", "Limbs: character not found"), 2)
return
end
local joints = limbSelfGetJoints(names)
if #joints == 0 then
Library:Notify(L("Конечности: не найдены (уже отрублены?)", "Limbs: not found (already removed?)"), 2)
return
end
pcall(function() hum.RequiresNeck = false end)
local function disableMotors()
for _, w in ipairs(joints) do
for _, m in ipairs(w.motors) do
pcall(function() m.Enabled = false end)
end
end
end
disableMotors()
pcall(function() hum:ChangeState(Enum.HumanoidStateType.Physics) end)
task.wait(0.15)

local destroyToy, ragdollRE = limbSelfRemotes()
local hrp = char:FindFirstChild("HumanoidRootPart")
for pass = 1, 2 do
if limbSelfAlive(joints) then
disableMotors()
if pass == 1 and destroyToy then
for _, w in ipairs(joints) do
if w.limb.Parent then
pcall(function() destroyToy:FireServer(w.limb) end)
end
end
task.wait(0.4)
end
if limbSelfAlive(joints) then
for _, w in ipairs(joints) do
if w.limb.Parent then
pcall(function() w.limb.Anchored = true end)
pcall(function() w.limb.CFrame = CFrame.new(0, -100000, 0) end)
end
end
if ragdollRE and hrp and hrp.Parent then
pcall(function() ragdollRE:FireServer(hrp, 1.2) end)
end
for _ = 1, 5 do
task.wait(0.25)
if not limbSelfAlive(joints) then break end
for _, w in ipairs(joints) do
if w.limb.Parent then
pcall(function() w.limb.CFrame = CFrame.new(0, -100000, 0) end)
end
end
end
end
end
end
local serverKilled = not limbSelfAlive(joints)
for _, w in ipairs(joints) do
if w.limb.Parent then
pcall(function() w.limb:Destroy() end)
end
end
pcall(function() hum:ChangeState(Enum.HumanoidStateType.GettingUp) end)
if serverKilled then
Library:Notify(L("Конечности: удалены НА СЕРВЕРЕ (у всех игроков)", "Limbs: removed ON SERVER (for everyone)"), 2)
else
Library:Notify(L("Конечности: отрублены (сервер не принял — удалено локально)", "Limbs: severed (server refused — removed locally)"), 2)
end
end

local limbLoopLegsConn = nil
local limbLegsList = { "Left Leg", "Right Leg", "LeftUpperLeg", "LeftLowerLeg", "LeftFoot", "RightUpperLeg", "RightLowerLeg", "RightFoot" }

LimbSelfSection:AddToggle("EnableLoopLegs", {
Text = L("Loop ломание ног", "Loop leg break"),
Default = false,
Tooltip = L("Ломает ноги сразу при включении и после КАЖДОГО респавна, пока включён", "Breaks legs on enable and after EVERY respawn while enabled"),
})

Toggles.EnableLoopLegs:OnChanged(function()
if limbLoopLegsConn then limbLoopLegsConn:Disconnect() limbLoopLegsConn = nil end
if Toggles.EnableLoopLegs.Value then
limbSelfRemove(limbLegsList)
limbLoopLegsConn = LocalPlayer.CharacterAdded:Connect(function(char)
task.spawn(function()
task.wait(0.25)
if Toggles.EnableLoopLegs.Value and char and char.Parent then
limbSelfRemove(limbLegsList)
end
end)
end)
end
end)

local autoLeaveConn = nil
DefAntiKick:AddToggle("EnableAutoLeave", {
Text = "Auto Leave (Anti-Ban)",
Default = false,
Tooltip = L("Автоматически выходит из игры, если сервер отправляет слишком много предупреждений 'Flying'", "Auto-leaves if server sends too many 'Flying' warnings"),
Callback = function(Value)
if autoLeaveConn then autoLeaveConn:Disconnect() autoLeaveConn = nil end
if Value then
local warnTimestamps = {}
local notify = game:GetService("ReplicatedStorage"):FindFirstChild("GameCorrectionEvents")
if notify then
notify = notify:FindFirstChild("GameCorrectionsNotify")
end
if not notify then return end

autoLeaveConn = notify.OnClientEvent:Connect(function(reason)
if reason == "Flying" then
local currentTime = os.clock()
table.insert(warnTimestamps, currentTime)

for i = #warnTimestamps, 1, -1 do
if currentTime - warnTimestamps[i] > 1 then
table.remove(warnTimestamps, i)
end
end

if #warnTimestamps >= 3 then
LocalPlayer:Kick("Oblivion Safety: Отключено для предотвращения бана.")
end
end
end)
end
end
})

DefAntiKick:AddToggle("EnableAntiVoid", {
Text = "Anti Void",
Default = false,
Tooltip = L("Не даёт упасть в void: убирает FallenPartsDestroyHeight и телепортирует на спавн при падении", "Prevents void fall: removes FallenPartsDestroyHeight and teleports to spawn")
})

DefAntiKick:AddToggle("EnableAntiKickShuriken", {
Text = "Anti Kick Sticky",
Default = false,
Tooltip = L("Спавнит выбранный стик в грудь и держит его залоченным на тебе — сбивает попытки кикнуть", "Spawns selected sticky on chest and keeps it locked — disrupts kick attempts"),
})

DefAntiKick:AddDropdown("AntiKickStickyType", {
Text = L("Стик", "Sticky"),
Values = {"NinjaShuriken", "NinjaKatana", "NinjaKunai", "ToolPencil", "ToolDiggingForkRusty", "ToolCleaver", "ToolPickaxe"},
Default = 1,
Multi = false,
Tooltip = L("Какой стик спавнить для Anti Kick Sticky", "Which sticky to spawn for Anti Kick Sticky"),
})

DefAntiKick:AddDropdown("AntiKickStickOrient", {
Text = L("Ориентация стика", "Sticky orientation"),
Values = {"Диагонально", "Вертикально", "Горизонтально", "Вперёд"},
Default = 1,
Multi = false,
Tooltip = L("Вертикально = стоит боком (смотрит вправо/влево, не вперёд/назад); также диагональ/горизонталь/вперёд (v42)", "Vertical = sideways (faces left/right, not front/back); also diagonal/horizontal/forward (v42)"),
})

DefAntiKick:AddToggle("EnableAntiKickItem", {
Text = "Anti Kick Item",
Default = false,
Tooltip = L("Спавнит выбранный предмет (свеча/часы/тарелка) в грудь и приклеивает Weld'ом — сбивает попытки кикнуть", "Spawns selected item (candle/clock/plate) on chest and Welds it — disrupts kick attempts"),
})

DefAntiKick:AddDropdown("AntiKickItemName", {
Text = L("Предмет", "Item"),
Values = {"SpookyCandle1", "ClockAlarm", "FoodPlate", "SprayCanWD", "JapaneseLantern"},
Default = 1,
Multi = false,
Tooltip = L("Какой предмет спавнить для Anti Kick Item (SprayCanWD / JapaneseLantern добавлены)", "Which item to spawn for Anti Kick Item (SprayCanWD / JapaneseLantern added)"),
})

local antiVoidConnections = {}
local originalFallenHeight = nil
local lastSafePosition = nil

function cleanupAntiVoid()
for _, conn in ipairs(antiVoidConnections) do
pcall(function() conn:Disconnect() end)
end
antiVoidConnections = {}
if originalFallenHeight ~= nil then
workspace.FallenPartsDestroyHeight = originalFallenHeight
originalFallenHeight = nil
end
end

Toggles.EnableAntiVoid:OnChanged(function()
if Toggles.EnableAntiVoid.Value then
originalFallenHeight = workspace.FallenPartsDestroyHeight
workspace.FallenPartsDestroyHeight = -50000
lastSafePosition = CFrame.new(0, 10, 0)

table.insert(antiVoidConnections, RunService.Heartbeat:Connect(function()
local char = LocalPlayer.Character
if not char then return end
local root = char:FindFirstChild("HumanoidRootPart")
local hum = char:FindFirstChildOfClass("Humanoid")
if not root then return end
if root.Position.Y > -100 then
lastSafePosition = root.CFrame
end
if root.Position.Y < -800 then
root.CFrame = CFrame.new(0, 10, 0)
root.AssemblyLinearVelocity = Vector3.zero
root.AssemblyAngularVelocity = Vector3.zero
if hum then hum.PlatformStand = false end
Library:Notify(L("Anti Void: телепорт на спавн", "Anti Void: teleport to spawn"), 3)
end
end))

table.insert(antiVoidConnections, game:GetService("Workspace").ChildRemoved:Connect(function(child)
if child.Name == LocalPlayer.Name then
cleanupAntiVoid()
Toggles.EnableAntiVoid:SetValue(false)
end
end))

Library:Notify(L("Anti Void ВКЛ", "Anti Void ON"), 2)
else
cleanupAntiVoid()
Library:Notify(L("Anti Void ВЫКЛ", "Anti Void OFF"), 2)
end
end)

DefAntiKick:AddToggle("EnableAntiBurn", {
Text = "Anti Burn",
Default = false,
Tooltip = L("Не даёт сгореть: тушит костёр/плиту через огнетушитель (firetouchinterest)", "Prevents burning: extinguishes fire/stove via fire extinguisher")
})

antiBurnConnections = {}

function cleanupAntiBurn()
for _, conn in ipairs(antiBurnConnections) do
pcall(function() conn:Disconnect() end)
end
antiBurnConnections = {}
end

function getExtinguishPart()
local p1 = workspace:FindFirstChild("Map")
and workspace.Map:FindFirstChild("Hole")
and workspace.Map.Hole:FindFirstChild("PoisonBigHole")
and workspace.Map.Hole.PoisonBigHole:FindFirstChild("ExtinguishPart")
if p1 and p1:IsA("BasePart") then return p1 end
for _, obj in ipairs(workspace:GetDescendants()) do
if obj:IsA("BasePart") and obj.Name == "ExtinguishPart" then
return obj
end
end
return nil
end

function setupAntiBurn(char)
local hum = char:FindFirstChildOfClass("Humanoid")
local hrp = char:FindFirstChild("HumanoidRootPart")
if not hum or not hrp then return end

local apagarfogo = getExtinguishPart()
if apagarfogo and apagarfogo:IsA("BasePart") then
apagarfogo.Size = Vector3.new(0.5, 0.5, 0.5)
apagarfogo.Transparency = 1
local tex = apagarfogo:FindFirstChild("Tex")
if tex and tex:IsA("Decal") then tex.Transparency = 1 end
end

local function doExtinguishLoop()
local firePart = char:FindFirstChild("FirePlayerPart", true)
if not firePart or not apagarfogo then return end
local oldCF = apagarfogo.CFrame
local oldSize = apagarfogo.Size
task.spawn(function()
while firePart and firePart.Parent do
local fd = hum:FindFirstChild("FireDebounce")
local cb = firePart:FindFirstChild("CanBurn")
if (not fd or not fd.Value) and (not cb or not cb.Value) then break end
if cb then cb.Value = false end
if fd then fd.Value = false end
apagarfogo.CFrame = firePart.CFrame * CFrame.new(math.random(-1, 1), math.random(-1, 1), math.random(-1, 1))
if firetouchinterest then
firetouchinterest(firePart, apagarfogo, 0)
task.wait()
firetouchinterest(firePart, apagarfogo, 1)
else
task.wait(0.05)
end
end
pcall(function()
apagarfogo.CFrame = oldCF
apagarfogo.Size = oldSize
end)
end)
end

local fireDebounce = hum:FindFirstChild("FireDebounce")
if fireDebounce then
table.insert(antiBurnConnections, fireDebounce:GetPropertyChangedSignal("Value"):Connect(function()
if fireDebounce.Value then doExtinguishLoop() end
end))
if fireDebounce.Value then doExtinguishLoop() end
end

local firePart = char:FindFirstChild("FirePlayerPart", true)
if firePart and firePart:FindFirstChild("CanBurn") then
table.insert(antiBurnConnections, firePart.CanBurn:GetPropertyChangedSignal("Value"):Connect(function()
if firePart.CanBurn.Value then doExtinguishLoop() end
end))
if firePart.CanBurn.Value then doExtinguishLoop() end
end

table.insert(antiBurnConnections, char.DescendantAdded:Connect(function(desc)
if desc.Name == "FirePlayerPart" then
local cb = desc:FindFirstChild("CanBurn")
if cb then
table.insert(antiBurnConnections, cb:GetPropertyChangedSignal("Value"):Connect(function()
if cb.Value then doExtinguishLoop() end
end))
if cb.Value then doExtinguishLoop() end
end
local fd = hum:FindFirstChild("FireDebounce")
if fd and fd.Value then doExtinguishLoop() end
end
end))
end

Toggles.EnableAntiBurn:OnChanged(function()
if Toggles.EnableAntiBurn.Value then
cleanupAntiBurn()
if LocalPlayer.Character then
setupAntiBurn(LocalPlayer.Character)
end
table.insert(antiBurnConnections, LocalPlayer.CharacterAdded:Connect(setupAntiBurn))
Library:Notify(L("Anti Burn ВКЛ", "Anti Burn ON"), 2)
else
cleanupAntiBurn()
Library:Notify(L("Anti Burn ВЫКЛ", "Anti Burn OFF"), 2)
end
end)

function antiKickClear()
local inv = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
local menuToys = game:GetService("ReplicatedStorage"):FindFirstChild("MenuToys")
local destroyrem = menuToys and menuToys:FindFirstChild("DestroyToy")
if inv and destroyrem then
for _, v in pairs(inv:GetChildren()) do
if v.Name == "AntiKick" or v.Name == "AntiKickItem" or v.Name == "NinjaShuriken" or v.Name == "NinjaKatana" or v.Name == "NinjaKunai" or v.Name == "ToolPencil" or v.Name == "ToolDiggingForkRusty" or v.Name == "ToolCleaver" or v.Name == "ToolPickaxe" or v.Name == "SpookyCandle1" or v.Name == "JapaneseLantern" or v.Name == "ClockAlarm" or v.Name == "FoodPlate" or v.Name == "SprayCanWD" then
pcall(function() destroyrem:FireServer(v) end)
end
end
end
end

Toggles.EnableAntiKickShuriken:OnChanged(function()
local plr = LocalPlayer
local RS = game:GetService("ReplicatedStorage")

local function getAntiKickItemName()
local sel = Options.AntiKickStickyType and Options.AntiKickStickyType.Value
return sel or "NinjaShuriken"
end

if Toggles.EnableAntiKickShuriken.Value then
Library:Notify(L("Anti Kick Sticky ВКЛ", "Anti Kick Sticky ON"), 2)
task.spawn(function()
local setOwner = RS:WaitForChild("GrabEvents"):WaitForChild("SetNetworkOwner")
local stickyEvent = RS:WaitForChild("PlayerEvents"):WaitForChild("StickyPartEvent")
local spawnRemote = RS:WaitForChild("MenuToys"):WaitForChild("SpawnToyRemoteFunction")
local canSpawn = plr:WaitForChild("CanSpawnToy")

local function getHRP()
if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
return plr.Character.HumanoidRootPart
else
local character = plr.CharacterAdded:Wait()
return character:WaitForChild("HumanoidRootPart")
end
end

local function CheckForHome()
local plotItems = workspace:FindFirstChild("PlotItems")
local playersInPlots = plotItems and plotItems:FindFirstChild("PlayersInPlots")
if not (playersInPlots and playersInPlots:FindFirstChild(plr.Name)) then
return false
end
local plots = workspace:FindFirstChild("Plots")
if not plots then return false end
for _, v in pairs(plots:GetChildren()) do
local sign = v:FindFirstChild("PlotSign")
local owners = sign and sign:FindFirstChild("ThisPlotsOwners")
if owners then
for _, b in pairs(owners:GetChildren()) do
if b.Value == plr.Name then
local folder = plotItems:FindFirstChild(v.Name)
if folder then
return true, folder
end
end
end
end
end
return false
end

local function getAttachPart(obj)
return obj:FindFirstChild("StickyPart") or obj:FindFirstChild("HoldPart") or obj:FindFirstChild("Main") or obj:FindFirstChildWhichIsA("BasePart")
end
local function StickKunai(kunai)
local attachPart = getAttachPart(kunai)
if not kunai or not attachPart then
return
end
local currentHRP = getHRP()
if not currentHRP then
return
end
local ownerPart = kunai:FindFirstChild("SoundPart") or attachPart
if ownerPart then
if not ownerPart:FindFirstChild("PartOwner") or ownerPart.PartOwner.Value ~= plr.Name then
pcall(function() setOwner:FireServer(ownerPart, ownerPart.CFrame) end)
end
end
local firePart = currentHRP:FindFirstChild("FirePlayerPart") or currentHRP:WaitForChild("FirePlayerPart", 5)
if firePart then
local sz = attachPart.Size
local stickOrient = Options.AntiKickStickOrient and Options.AntiKickStickOrient.Value or "Диагонально"
local rx, ry, rz = 0, 0, 0
if stickOrient == "Горизонтально" then
if sz.Z >= sz.X and sz.Z >= sz.Y then
ry = math.rad(90)
elseif sz.Y >= sz.X and sz.Y >= sz.Z then
rz = math.rad(90)
end
elseif stickOrient == "Вертикально" then
if sz.X >= sz.Y and sz.X >= sz.Z then
rz = math.rad(90)
elseif sz.Z >= sz.X and sz.Z >= sz.Y then
rx = math.rad(90)
end
elseif stickOrient == "Вперёд" then
if sz.X >= sz.Y and sz.X >= sz.Z then
ry = math.rad(90)
elseif sz.Y >= sz.X and sz.Y >= sz.Z then
rx = -math.rad(90)
end
else
if sz.X >= sz.Y and sz.X >= sz.Z then
rz = math.rad(45)
elseif sz.Z >= sz.X and sz.Z >= sz.Y then
rx = -math.rad(45)
else
rz = -math.rad(45)
end
end
local baseRot = CFrame.Angles(rx, ry, rz)
if stickOrient == "Вертикально" then
baseRot = CFrame.Angles(0, math.rad(90), 0) * baseRot
end
local sideCFrame = CFrame.new(0, 0, 0) * baseRot
pcall(function()
stickyEvent:FireServer(
attachPart,
firePart,
sideCFrame
)
end)
if not attachPart:FindFirstChild("LocalFakeWeld") then
attachPart.CFrame = firePart.CFrame * sideCFrame
local w = Instance.new("WeldConstraint")
w.Name = "LocalFakeWeld"
w.Part0 = attachPart
w.Part1 = firePart
w.Parent = attachPart
end
end
for _, obj in pairs(kunai:GetChildren()) do
if obj:IsA("BasePart") then
obj.CanTouch = false
obj.CanCollide = false
obj.CanQuery = false
if obj.Name == "StickyPart" or obj.Name == "SoundPart" or obj.Name == "Hitbox" or obj.Transparency == 1 then
obj.Transparency = 1
else
obj.Transparency = 0.8
if not obj:FindFirstChild("Highlight") then
local high = Instance.new("Highlight", obj)
high.FillColor = Color3.fromRGB(255, 255, 255)
high.OutlineColor = Color3.fromRGB(0, 0, 0)
end
end
end
end
end

local function SpawnToy(name)
local t = tick()
while not canSpawn.Value do
if not Toggles.EnableAntiKickShuriken.Value or tick() - t > 5 then
return nil
end
task.wait(0.1)
end
local currentHRP = getHRP()
if currentHRP then
task.spawn(function()
pcall(function()
spawnRemote:InvokeServer(name, currentHRP.CFrame * CFrame.new(0, 2, 0), Vector3.new(0, 0, 0))
end)
end)
end
local boolik, house = CheckForHome()
local inv = workspace:FindFirstChild(plr.Name .. "SpawnedInToys")
local plotItems = workspace:FindFirstChild("PlotItems")
local playersInPlots = plotItems and plotItems:FindFirstChild("PlayersInPlots")
if boolik and house then
return house:WaitForChild(name, 2)
elseif not (playersInPlots and playersInPlots:FindFirstChild(plr.Name)) and inv then
return inv:WaitForChild(name, 2)
end
return nil
end

while Toggles.EnableAntiKickShuriken.Value do
task.wait(0.005)
if not plr.Character or not plr.Character:FindFirstChild("Humanoid") or plr.Character.Humanoid.Health <= 0 then
continue
end
local inv = workspace:FindFirstChild(plr.Name .. "SpawnedInToys")
local kunai = (inv and inv:FindFirstChild("AntiKick")) or (inv and inv:FindFirstChild(getAntiKickItemName()))
local plotItems = workspace:FindFirstChild("PlotItems")
local playersInPlots = plotItems and plotItems:FindFirstChild("PlayersInPlots")

pcall(function()
if playersInPlots and playersInPlots:FindFirstChild(plr.Name) then
local boolik, house = CheckForHome()
local plots = workspace:FindFirstChild("Plots")
if boolik and house and plots and plots:FindFirstChild(house.Name) then
local sign = plots[house.Name]:FindFirstChild("PlotSign")
local signReady = false
if sign then
local ownersVal = sign:FindFirstChild("ThisPlotsOwners")
local ownersObj = ownersVal and ownersVal.Value
local trn = ownersObj and ownersObj:FindFirstChild("TimeRemainingNum")
signReady = (trn ~= nil and trn.Value ~= nil and trn.Value > 89)
end
if signReady then
local k = SpawnToy(getAntiKickItemName())
if k then
k.Name = "AntiKick"
StickKunai(k)
kunai = k
end
end
end
end
end)

if not kunai then
if playersInPlots and playersInPlots:FindFirstChild(plr.Name) then
continue
end
kunai = SpawnToy(getAntiKickItemName())
if kunai == nil then
continue
end
kunai.Name = "AntiKick"
if not kunai then
continue
end
end

repeat
if kunai and kunai:FindFirstChild("StickyPart") and kunai.StickyPart.CanTouch == true then
StickKunai(kunai)
kunai.Name = "AntiKick"
end
task.wait(0.05)
until not kunai or not Toggles.EnableAntiKickShuriken.Value
or not kunai:FindFirstChild("StickyPart")
or kunai.StickyPart.CanTouch == false
or not plr.Character or not plr.Character:FindFirstChild("HumanoidRootPart")
or (plr.Character.HumanoidRootPart.Position - kunai.StickyPart.Position).Magnitude >= 40

if not kunai or not kunai:FindFirstChild("StickyPart") or not plr.Character or not plr.Character:FindFirstChild("HumanoidRootPart") or (plr.Character.HumanoidRootPart.Position - kunai.StickyPart.Position).Magnitude >= 40 then
antiKickClear()
end

pcall(function()
repeat
task.wait(0.05)
local a = getAttachPart(kunai)
until not Toggles.EnableAntiKickShuriken.Value or not plr.Character or not plr.Character:FindFirstChild("Humanoid") or not kunai or not kunai.Parent or not a

local a2 = kunai and getAttachPart(kunai)
if not a2 or not kunai.Parent or (plr.Character and plr.Character:FindFirstChild("Humanoid") and plr.Character.Humanoid.Health <= 0) then
antiKickClear()
end
end)
end
end)
else
antiKickClear()
Library:Notify(L("Anti Kick Sticky ВЫКЛ", "Anti Kick Sticky OFF"), 2)
end
end)

Toggles.EnableAntiKickItem:OnChanged(function()
local plr = LocalPlayer
local RS = game:GetService("ReplicatedStorage")

if Toggles.EnableAntiKickItem.Value then
Library:Notify(L("Anti Kick Item ВКЛ", "Anti Kick Item ON"), 2)
task.spawn(function()
local setOwner = RS:WaitForChild("GrabEvents"):WaitForChild("SetNetworkOwner")
local spawnRemote = RS:WaitForChild("MenuToys"):WaitForChild("SpawnToyRemoteFunction")
local canSpawn = plr:WaitForChild("CanSpawnToy")

local function getHRP()
if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
return plr.Character.HumanoidRootPart
else
local character = plr.CharacterAdded:Wait()
return character:WaitForChild("HumanoidRootPart")
end
end

local function getSelectedItem()
local sel = Options.AntiKickItemName and Options.AntiKickItemName.Value
if sel and sel ~= "" then
return sel
end
return "SpookyCandle1"
end

local function FWD(parent, part, time)
return parent:FindFirstChild(part) or parent:WaitForChild(part, time)
end

local function itemSpawn(name)
local t = tick()
while not canSpawn.Value do
if not Toggles.EnableAntiKickItem.Value or tick() - t > 5 then
return nil
end
task.wait(0.1)
end
local currentHRP = getHRP()
if not currentHRP then return nil end
pcall(function()
spawnRemote:InvokeServer(name, currentHRP.CFrame * CFrame.new(0, 2, 0), Vector3.new(0, 0, 0))
end)
local inv = workspace:FindFirstChild(plr.Name .. "SpawnedInToys")
if inv then
return inv:WaitForChild(name, 3)
end
return nil
end

local function sno(part)
pcall(function() setOwner:FireServer(part, part.CFrame) end)
end

local function isNetworkOwner(part)
local po = part and part:FindFirstChild("PartOwner")
return po and po.Value == plr.Name
end

local function getSoundPart(item, name)
if name == "SpookyCandle1" then
return FWD(item, "Hitbox", 0.5)
end
return item:FindFirstChild("Hitbox") or item:FindFirstChild("SoundPart") or item:FindFirstChild("Main") or item:FindFirstChildWhichIsA("BasePart")
end

local akItemLastChar = plr.Character
while Toggles.EnableAntiKickItem.Value do
task.wait(0.05)
if plr.Character ~= akItemLastChar then
akItemLastChar = plr.Character
pcall(function()
local d = RS:FindFirstChild("MenuToys")
d = d and d:FindFirstChild("DestroyToy")
if d then
local oldInv = workspace:FindFirstChild(plr.Name .. "SpawnedInToys")
if oldInv then
for _, v in pairs(oldInv:GetChildren()) do
if v.Name == "AntiKickItem" then d:FireServer(v) end
end
end
end
end)
continue
end
if not plr.Character or not plr.Character:FindFirstChildOfClass("Humanoid") or plr.Character:FindFirstChildOfClass("Humanoid").Health <= 0 then
continue
end

local inv = workspace:FindFirstChild(plr.Name .. "SpawnedInToys")
local itemName = getSelectedItem()
local Item = inv and inv:FindFirstChild("AntiKickItem")
local SoundPart = Item and getSoundPart(Item, itemName)

if not Item or not SoundPart then
if inv then
for _, v in pairs(inv:GetChildren()) do
if v.Name == "AntiKickItem" then
pcall(function()
local d = RS:FindFirstChild("MenuToys")
d = d and d:FindFirstChild("DestroyToy")
if d then d:FireServer(v) end
end)
end
end
end
Item = itemSpawn(itemName)
if not Item then continue end
SoundPart = getSoundPart(Item, itemName)
if not SoundPart then continue end
sno(SoundPart)
for _, v in pairs(Item:GetChildren()) do
if v:IsA("BasePart") then
v.CanCollide = false
v.CanQuery = false
v.Transparency = 0.8
end
end
local currentHRP = getHRP()
if currentHRP then
local firePart = currentHRP:FindFirstChild("FirePlayerPart") or currentHRP
local Weld
if itemName == "SpookyCandle1" then
SoundPart.CFrame = currentHRP.CFrame * CFrame.new(0, 1, 0)
Weld = Instance.new("Weld")
Weld.C0 = CFrame.new(0, 1, 0)
Weld.C1 = CFrame.new(0, 0, 0)
else
SoundPart.CFrame = currentHRP.CFrame
Weld = Instance.new("WeldConstraint")
end
Weld.Name = "WeldBlabla"
Weld.Part0 = SoundPart
Weld.Part1 = firePart
Weld.Parent = SoundPart
end
Item.Name = "AntiKickItem"
for _ = 1, 30 do
if not Toggles.EnableAntiKickItem.Value or not Item.Parent then break end
if isNetworkOwner(SoundPart) then break end
sno(SoundPart)
task.wait(0.03)
end
end

if not Item or not SoundPart then continue end
if not isNetworkOwner(SoundPart) then
local hrpNow = plr.Character and plr.Character:FindFirstChild("HumanoidRootPart")
if hrpNow and (SoundPart.Position - hrpNow.Position).Magnitude > 40 then
pcall(function()
local d = RS:FindFirstChild("MenuToys")
d = d and d:FindFirstChild("DestroyToy")
if d then d:FireServer(Item) end
end)
continue
end
sno(SoundPart)
end
local currentHRP = getHRP()
if currentHRP then
local firePart = currentHRP:FindFirstChild("FirePlayerPart") or currentHRP
local Weld = SoundPart:FindFirstChild("WeldBlabla")
if itemName == "SpookyCandle1" then
if not Weld then
SoundPart.CFrame = currentHRP.CFrame * CFrame.new(0, 1, 0)
Weld = Instance.new("Weld")
Weld.Name = "WeldBlabla"
Weld.C0 = CFrame.new(0, 1, 0)
Weld.C1 = CFrame.new(0, 0, 0)
Weld.Part0 = SoundPart
Weld.Part1 = firePart
Weld.Parent = SoundPart
elseif Weld.Part1 ~= firePart then
Weld.Part1 = firePart
end
else
if not Weld then
SoundPart.CFrame = currentHRP.CFrame
Weld = Instance.new("WeldConstraint")
Weld.Name = "WeldBlabla"
Weld.Part0 = SoundPart
Weld.Part1 = firePart
Weld.Parent = SoundPart
elseif Weld.Part1 ~= firePart then
Weld.Enabled = false
SoundPart.CFrame = currentHRP.CFrame
Weld.Part1 = firePart
Weld.Enabled = true
end
end
end
end
end)
else
antiKickClear()
Library:Notify(L("Anti Kick Item ВЫКЛ", "Anti Kick Item OFF"), 2)
end
end)
end

antiOwnFoods = {
["Pizza (Cheese)"] = "FoodPizzaCheese",
["Pizza (Pepperoni)"] = "FoodPizzaPepperoni",
["Burger"] = "FoodHamburger",
["Hotdog"] = "FoodHotdog",
["Banana"] = "FoodBanana",
["Donut"] = "FoodDonut",
["Cake"] = "FoodCakePink",
["Bread"] = "FoodBread",
["Mayonnaise"] = "FoodMayonnaise",
["French Fries"] = "FoodFrenchFries",
["Dippy Egg"] = "FoodDippyEgg",
["Coconut"] = "FoodCoconut",
["Meat Stick"] = "FoodMeatStick",
["Poison Mushroom"] = "FoodMushroomPoison",
["Snare Drum"] = "InstrumentDrumSnare",
["Bongos"] = "InstrumentDrumBongos",
["Bugle"] = "InstrumentBrassBugle",
["Trumpet"] = "InstrumentBrassTrumpet",
["Vuvuzela"] = "InstrumentBrassVuvuzela",
["Acoustic Guitar"] = "InstrumentGuitarAcoustic",
["Banjo"] = "InstrumentGuitarBanjo",
["Lyre"] = "InstrumentGuitarLyre",
["Ukulele"] = "InstrumentGuitarUkulele",
["Melodica"] = "InstrumentPianoMelodica",
["Microphone"] = "InstrumentVoiceMicrophone",
["Ocarina"] = "InstrumentWoodwindOcarina",
["Saxophone"] = "InstrumentWoodwindSaxophone",
}
antiOwnFoodOrder = {
"Pizza (Cheese)",
"Pizza (Pepperoni)",
"Burger",
"Hotdog",
"Banana",
"Donut",
"Cake",
"Bread",
"Mayonnaise",
"French Fries",
"Dippy Egg",
"Coconut",
"Meat Stick",
"Poison Mushroom",
"Snare Drum",
"Bongos",
"Bugle",
"Trumpet",
"Vuvuzela",
"Acoustic Guitar",
"Banjo",
"Lyre",
"Ukulele",
"Melodica",
"Microphone",
"Ocarina",
"Saxophone",
}

DefAntiOwnership = DefSection

DefAntiOwnership:AddDropdown("AntiOwnFood", {
Text = L("Еда", "Food"),
Values = antiOwnFoodOrder,
Default = 1,
Multi = false,
Tooltip = L("Что спавнить и есть для защиты", "What to spawn and eat for protection"),
})

DefAntiOwnership:AddToggle("EnableAntiOwnership", {
Text = "Anti Ownership",
Default = false,
Tooltip = L("Спавнит еду, берёт в руки (ПКМ) и быстро ест (ЛКМ) — держит тебя 'занятым', сбивая захват/кик", "Spawns food, picks up (RMB) and eats quickly (LMB) — keeps you 'busy', disrupting grab/kick"),
})

DefAntiOwnership:AddSlider("AntiOwnCycleSpeed", {
Text = L("Скорость цикла", "Cycle Speed"),
Default = 0.02,
Min = 0.01,
Max = 1,
Rounding = 3,
Compact = false,
Tooltip = L("Задержка между циклами (сек). Меньше = быстрее. 0.02 = норма из XOCO", "Delay between cycles (sec). Lower = faster. 0.02 = XOCO default"),
})

Toggles.EnableAntiOwnership:OnChanged(function()
local plr = LocalPlayer
local RS = game:GetService("ReplicatedStorage")

local function antiOwnCheckForHome()
local plotItems = workspace:FindFirstChild("PlotItems")
local playersInPlots = plotItems and plotItems:FindFirstChild("PlayersInPlots")
if not (playersInPlots and playersInPlots:FindFirstChild(plr.Name)) then
return false
end
local plots = workspace:FindFirstChild("Plots")
if not plots then return false end
for _, v in pairs(plots:GetChildren()) do
local sign = v:FindFirstChild("PlotSign")
local owners = sign and sign:FindFirstChild("ThisPlotsOwners")
if owners then
for _, b in pairs(owners:GetChildren()) do
if b.Value == plr.Name then
local folder = plotItems:FindFirstChild(v.Name)
if folder then
return true, folder
end
end
end
end
end
return false
end

local function antiOwnInv()
local inHome, house = antiOwnCheckForHome()
if inHome and house then return house end
return workspace:FindFirstChild(plr.Name .. "SpawnedInToys")
end

local function antiOwnFoodName()
local sel = Options.AntiOwnFood and Options.AntiOwnFood.Value
return antiOwnFoods[sel] or "FoodHamburger"
end

local function antiOwnClear()
local inv = antiOwnInv()
local menuToys = RS:FindFirstChild("MenuToys")
local destroyrem = menuToys and menuToys:FindFirstChild("DestroyToy")
if inv and destroyrem then
for _, v in pairs(inv:GetChildren()) do
if string.sub(v.Name, 1, 4) == "Food" then
pcall(function() destroyrem:FireServer(v) end)
end
end
end
end

if Toggles.EnableAntiOwnership.Value then
Library:Notify(L("Anti Ownership ВКЛ", "Anti Ownership ON"), 2)
task.spawn(function()
local menuToys = RS:WaitForChild("MenuToys", 10)
local spawnRemote = menuToys and menuToys:WaitForChild("SpawnToyRemoteFunction", 10)
local destroyRemote = menuToys and menuToys:WaitForChild("DestroyToy", 10)
local holdEvents = RS:WaitForChild("HoldEvents", 10)
local holdUse = holdEvents and holdEvents:WaitForChild("Use", 10)
local canSpawn = plr:WaitForChild("CanSpawnToy", 10)

if not spawnRemote then Library:Notify(L("Anti Ownership: не найден SpawnToyRemoteFunction", "Anti Ownership: SpawnToyRemoteFunction not found"), 5) return end
if not holdUse then Library:Notify(L("Anti Ownership: не найден HoldEvents.Use", "Anti Ownership: HoldEvents.Use not found"), 5) return end
if not canSpawn then Library:Notify(L("Anti Ownership: нет CanSpawnToy — спавню без гейта", "Anti Ownership: no CanSpawnToy — spawning without gate"), 4) end

local function antiOwnGetHRP()
if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
return plr.Character.HumanoidRootPart
end
local char = plr.CharacterAdded:Wait()
return char:WaitForChild("HumanoidRootPart", 5)
end

local antiOwnWarned = false
local antiOwnHeldNotified = false
local function antiOwnSpawn(name)
local t = tick()
while canSpawn and not canSpawn.Value do
if not Toggles.EnableAntiOwnership.Value or tick() - t > 3 then break end
RunService.Heartbeat:Wait()
end
local hrp = antiOwnGetHRP()
if not hrp then return nil end
task.spawn(function()
local okSpawn, errSpawn = pcall(function()
spawnRemote:InvokeServer(name, hrp.CFrame * CFrame.new(0, 5, -5), Vector3.new(0, 0, 0))
end)
if not okSpawn and not antiOwnWarned then
antiOwnWarned = true
Library:Notify("Anti Ownership: спавн '" .. tostring(name) .. "' упал (" .. tostring(errSpawn) .. ")", 6)
end
end)
local t1 = tick()
repeat
RunService.Heartbeat:Wait()
local inv = antiOwnInv()
local toy = inv and inv:FindFirstChild(name)
if toy then return toy end
until not Toggles.EnableAntiOwnership.Value or tick() - t1 > 2
if not antiOwnWarned then
antiOwnWarned = true
Library:Notify("Anti Ownership: '" .. tostring(name) .. "' не появилась в инве��т��ре — проверь имя еды/ремоут", 6)
end
return nil
end

while Toggles.EnableAntiOwnership.Value do
local CycleSpeed = Options.AntiOwnCycleSpeed and Options.AntiOwnCycleSpeed.Value or 0.02
local HoldDuration = CycleSpeed
local char = plr.Character
local hrp = char and char:FindFirstChild("HumanoidRootPart")
if not hrp or not char then task.wait(CycleSpeed) continue end
local hum = char:FindFirstChildOfClass("Humanoid")
if not hum or hum.Health <= 0 then task.wait(CycleSpeed) continue end

local name = antiOwnFoodName()
local inv = antiOwnInv()
local item = inv and inv:FindFirstChild(name)

if not item or not item.Parent then
task.spawn(function()
pcall(function()
spawnRemote:InvokeServer(name, hrp.CFrame * CFrame.new(0, -12, 0), Vector3.zero)
end)
end)
task.wait(0.1)
continue
end

local holdPart = item:FindFirstChild("HoldPart")
if not holdPart then
pcall(function() destroyRemote:FireServer(item) end)
task.wait(CycleSpeed)
continue
end

for _, v in pairs(item:GetDescendants()) do
if v:IsA("BasePart") then
v.CanCollide = false
v.Massless = true
end
end

local holdRF = holdPart:FindFirstChild("HoldItemRemoteFunction")
local dropRF = holdPart:FindFirstChild("DropItemRemoteFunction")

if holdRF then
task.spawn(function()
pcall(function()
holdRF:InvokeServer(item, char)
end)
end)
end

task.wait(HoldDuration)

if dropRF then
task.spawn(function()
pcall(function()
dropRF:InvokeServer(item, CFrame.new(0, 5000, 0), Vector3.zero)
end)
end)
end

if not antiOwnHeldNotified then
antiOwnHeldNotified = true
Library:Notify(L("Anti Ownership: защита активна (grab-drop)", "Anti Ownership: protection active (grab-drop)"), 3)
end

task.wait(CycleSpeed)
end

end)
else
antiOwnClear()
Library:Notify(L("Anti Ownership ВЫКЛ", "Anti Ownership OFF"), 2)
end
end)
end

local DefAntiLoopKick = Tabs.Def:AddRightGroupbox("Anti Loop Kick", "refresh-cw")

antiLoopKickConn = nil
antiLoopKickHouse = nil
antiLoopKickRadius = 10

DefAntiLoopKick:AddDropdown("AntiLoopKickHouse", {
Text = L("Дом для Loop TP", "Loop TP House"),
Default = "Green House",
Values = {"Green House", "Pink House", "Witch House", "Blue House", "China House"},
Multi = false,
Tooltip = L("Выбери дом", "Select a house"),
Callback = function(Value)
antiLoopKickHouse = Value
end
})

DefAntiLoopKick:AddToggle("EnableAntiLoopKick", {
Text = "Anti Loop Kick",
Default = false,
Tooltip = "Zone-based, radius 10",
Callback = function(Value)
if Value then
local houseName = antiLoopKickHouse or (Options.AntiLoopKickHouse and Options.AntiLoopKickHouse.Value) or "Green House"
local cf = houseMapPoints and houseMapPoints[houseName]
if not cf then
local pts = {
["Green House"] = CFrame.new(-548.305054, -2.45424771, 79.3213348),
["Pink House"] = CFrame.new(-475.493835, -2.70774508, -159.395279),
["Witch House"] = CFrame.new(270.225922, -2.48055029, 458.186493),
["Blue House"] = CFrame.new(501.939911, 88.2323608, -349.129211),
["China House"] = CFrame.new(545.441833, 128.004593, -99.4881439)
}
cf = pts[houseName]
end
if not cf then return end
local housePos = cf.Position
antiLoopKickConn = RunService.Heartbeat:Connect(function()
local char = LocalPlayer.Character
if not char then return end
local root = char:FindFirstChild("HumanoidRootPart")
if not root then return end
if (root.Position - housePos).Magnitude > antiLoopKickRadius then
char:PivotTo(cf)
end
end)
Library:Notify("Anti Loop Kick ВКЛ: " .. houseName, 2)
else
if antiLoopKickConn then antiLoopKickConn:Disconnect() antiLoopKickConn = nil end
local char = LocalPlayer.Character
if char and char:FindFirstChild("HumanoidRootPart") then
char.HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
char.HumanoidRootPart.AssemblyAngularVelocity = Vector3.zero
end
Library:Notify(L("Anti Loop Kick ВЫКЛ", "Anti Loop Kick OFF"), 2)
end
end
})

DefCounterMode = Tabs.Def:AddRightGroupbox("Counter Mode", "swords")

DefCounterMode:AddToggle("EnableCounterMode", {
Text = "Counter Mode",
Default = false,
Tooltip = L("Когда тебя трогают — применяет эффект к игроку", "When someone touches you — applies effect to them"),
})

DefCounterMode:AddDropdown("CounterModeType", {
Text = L("Режим", "Mode"),
Default = "Bounce",
Values = {"Bounce", "Die", "Fling"},
Multi = false,
Tooltip = L("Что произойдет с игроком который тебя тронет", "What happens to the player who touches you"),
})

DefCounterMode:AddSlider("CounterRepulsionForce", {
Text = L("Сила отскока", "Bounce Force"),
Default = 15,
Min = 10,
Max = 500,
Rounding = 0,
Increment = 10,
Tooltip = L("Сила с которой игрок отлетит", "Force with which player gets knocked back"),
})

DefCounterMode:AddSlider("CounterFlingForce", {
Text = L("Сила флинга", "Fling Force"),
Default = 400,
Min = 100,
Max = 5000,
Rounding = 0,
Increment = 50,
Tooltip = L("Сила с которой игрок улетит вверх", "Force with which player gets launched up"),
})

counterModeConn = nil

local function counterLookAt(startPosition, targetPosition)
local directionVector = (targetPosition - startPosition).Unit
local rightVector = directionVector:Cross(Vector3.new(0, 1, 0))
local upVector = rightVector:Cross(directionVector)
return CFrame.fromMatrix(startPosition, rightVector, upVector)
end

local function counterCheckNetOwn(potentialPlayer)
if typeof(potentialPlayer) == "Instance" and potentialPlayer:IsA("Player") and potentialPlayer.Character then
if potentialPlayer.Character:FindFirstChild("Head") then
local head = potentialPlayer.Character.Head
if head:FindFirstChild("PartOwner") and head.PartOwner.Value == LocalPlayer.Name then
return true
end
end
end
return false
end

local function counterSNOWship(otherPlayer, callbackFunction)
if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
if typeof(otherPlayer) == "Instance" and otherPlayer:IsA("Player") and otherPlayer.Character and otherPlayer.Character:FindFirstChild("HumanoidRootPart") then
local otherPlayerHRP = otherPlayer.Character.HumanoidRootPart
local dist = LocalPlayer:DistanceFromCharacter(otherPlayerHRP.Position)
if counterCheckNetOwn(otherPlayer) then
if type(callbackFunction) == "function" then
callbackFunction()
end
return true
end
if dist <= 30 then
local setOwner = ReplicatedStorage:FindFirstChild("GrabEvents")
if setOwner then
setOwner = setOwner:FindFirstChild("SetNetworkOwner")
end
if setOwner then
setOwner:FireServer(otherPlayerHRP, counterLookAt(LocalPlayer.Character.HumanoidRootPart.Position, otherPlayerHRP.Position))
end
end
end
end
end

local function counterCreateSkyVelocity(part)
local bv = Instance.new("BodyVelocity")
bv.Name = "SkyVelocity"
bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bv.Velocity = Vector3.new(0, 100000, 0)
bv.Parent = part
return bv
end

function counterModeApply(otherPlayer)
if not otherPlayer or not otherPlayer.Character then return end
local otherChar = otherPlayer.Character
local otherHumanoid = otherChar:FindFirstChildOfClass("Humanoid")
local otherHumanoidRootPart = otherChar:FindFirstChild("HumanoidRootPart")
if not otherHumanoid or not otherHumanoidRootPart then return end

local mode = Options.CounterModeType and Options.CounterModeType.Value or "Bounce"
local destroyGrabLineEvent = ReplicatedStorage:FindFirstChild("GrabEvents")
if destroyGrabLineEvent then
destroyGrabLineEvent = destroyGrabLineEvent:FindFirstChild("DestroyGrabLine")
end

local counterAction

if mode == "Bounce" then
counterAction = function()
local lookAtCFrame = counterLookAt(LocalPlayer.Character.HumanoidRootPart.Position, otherHumanoidRootPart.Position)
local bodyVelocity = Instance.new("BodyVelocity", otherPlayer.Character.HumanoidRootPart)
bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
local force = (Options.CounterRepulsionForce and Options.CounterRepulsionForce.Value) or 100
bodyVelocity.Velocity = Vector3.new(lookAtCFrame.lookVector.X, 0.5, lookAtCFrame.lookVector.Z) * force
wait()
bodyVelocity:Destroy()
if destroyGrabLineEvent then
destroyGrabLineEvent:FireServer(otherHumanoidRootPart)
end
end
elseif mode == "Die" then
counterAction = function()
local humanoidInstance = otherHumanoid
if humanoidInstance then
counterCreateSkyVelocity(otherHumanoidRootPart)
for _ = 0, 20 do
pcall(function()
humanoidInstance.BreakJointsOnDeath = false
humanoidInstance:ChangeState(Enum.HumanoidStateType.Dead)
humanoidInstance.Jump = true
humanoidInstance.Sit = true
end)
end
task.wait()
if destroyGrabLineEvent then
destroyGrabLineEvent:FireServer(otherHumanoidRootPart)
end
end
end
elseif mode == "Fling" then
counterAction = function()
local force = (Options.CounterFlingForce and Options.CounterFlingForce.Value) or 400
counterCreateSkyVelocity(otherHumanoidRootPart)
wait(0.5)
if destroyGrabLineEvent then
destroyGrabLineEvent:FireServer(otherHumanoidRootPart)
end
end
else
return
end

for _ = 1, 50 do
if counterSNOWship(otherPlayer, counterAction) then
break
end
task.wait()
end
end

function counterModeHook(char)
if not char then return end
char.DescendantAdded:Connect(function(hitPart)
if hitPart.Name == "PartOwner" and Toggles.EnableCounterMode and Toggles.EnableCounterMode.Value then
local heldObjectName = tostring(hitPart.Value)
if heldObjectName == LocalPlayer.Name then return end
local otherPlayer = game:GetService("Players"):FindFirstChild(heldObjectName)
if otherPlayer and otherPlayer ~= LocalPlayer then
local otherHumanoid = nil
local otherHumanoidRootPart = nil
if otherPlayer.Character then
otherHumanoid = otherPlayer.Character:FindFirstChildOfClass("Humanoid")
otherHumanoidRootPart = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
end
if otherPlayer and otherHumanoid and otherHumanoidRootPart then
task.spawn(function() counterModeApply(otherPlayer) end)
end
end
end
end)
end

Toggles.EnableCounterMode:OnChanged(function()
if Toggles.EnableCounterMode.Value then
if LocalPlayer.Character then counterModeHook(LocalPlayer.Character) end
counterModeConn = LocalPlayer.CharacterAdded:Connect(function(char)
task.wait(0.5)
counterModeHook(char)
end)
else
if counterModeConn then counterModeConn:Disconnect() counterModeConn = nil end
end
end)

DefAntiPaint = DefAntiGrab 

DefAntiPaint:AddToggle("EnableAntiPaint", {
Text = "Anti Paint",
Default = false,
Tooltip = "\u{0423}\u{0434}\u{0430}\u{043b}\u{044f}\u{0435}\u{0442} \u{043a}\u{0440}\u{0430}\u{0441}\u{043a}\u{0443} \u{0438} \u{043d}\u{0435} \u{0434}\u{0430}\u{0451}\u{0442} \u{043d}\u{043e}\u{0432}\u{043e}\u{0439} \u{043b}\u{0435}\u{0447}\u{044c}",
})

antiPaintConns = {}

function antiPaintDeleteAll()
for _, obj in ipairs(workspace:GetDescendants()) do
if obj:IsA("BasePart") and obj.Name == "PaintPlayerPart" then
pcall(function() obj:Destroy() end)
end
end
end

function antiPaintWatch()
table.insert(antiPaintConns, workspace.DescendantAdded:Connect(function(obj)
if obj:IsA("BasePart") and obj.Name == "PaintPlayerPart" then
task.defer(function()
if obj and obj.Parent then pcall(function() obj:Destroy() end) end
end)
end
end))
end

function antiPaintSetTouch(state)
local char = LocalPlayer.Character
if not char then return end
for _, v in ipairs(char:GetChildren()) do
if v:IsA("Part") or v:IsA("BasePart") then
v.CanTouch = state
v.CanQuery = state
end
end
end

function antiPaintCleanup()
for _, conn in ipairs(antiPaintConns) do
pcall(function() conn:Disconnect() end)
end
antiPaintConns = {}
antiPaintSetTouch(true)
end

Toggles.EnableAntiPaint:OnChanged(function()
if Toggles.EnableAntiPaint.Value then
antiPaintDeleteAll()
antiPaintWatch()
antiPaintSetTouch(false)
Library:Notify("Anti Paint \u{0412}\u{041a}\u{041b}", 2)
else
antiPaintCleanup()
Library:Notify("Anti Paint \u{0412}\u{042b}\u{041a}\u{041b}", 2)
end
end)

DefAntiItems = Tabs.Def:AddRightGroupbox("Anti Items", "package-x")

DefAntiItems:AddToggle("EnableAntiStick", {
Text = "Anti Sticky",
Default = false,
Tooltip = "Disables StickyPartsTouchDetection script (XOCO style)",
})

Toggles.EnableAntiStick:OnChanged(function()
local v = Toggles.EnableAntiStick.Value
local sptd = LocalPlayer.PlayerScripts:FindFirstChild("StickyPartsTouchDetection")
if sptd then
sptd.Disabled = v
end
if v then
Library:Notify("Anti Sticky ON", 2)
else
Library:Notify("Anti Sticky OFF", 2)
end
end)

DefAntiItems:AddToggle("EnableAntiSnowball", {
Text = "Anti Snowball",
Default = false,
Tooltip = "Loop ragdoll cancel (XOCO style)",
})

local loopRagdoll = false
local antiSnowballWalkConn = nil
local antiSnowballWeldConn = nil
Toggles.EnableAntiSnowball:OnChanged(function()
local v = Toggles.EnableAntiSnowball.Value
loopRagdoll = v
if antiSnowballWalkConn then antiSnowballWalkConn:Disconnect(); antiSnowballWalkConn = nil end
if antiSnowballWeldConn then antiSnowballWeldConn:Disconnect(); antiSnowballWeldConn = nil end
if v then
local function breakLimbs(char)
for _, part in pairs(char:GetChildren()) do
if part:IsA("BasePart") and part:FindFirstChild("BallSocketConstraint") and part.Name ~= "Head" then
part.BallSocketConstraint.Enabled = false
if part:FindFirstChild("RagdollLimbPart") then
part.RagdollLimbPart.WeldConstraint.Enabled = false
end
end
end
end
local char = LocalPlayer.Character
if char then
local hrp = char:FindFirstChild("HumanoidRootPart")
local hum = char:FindFirstChild("Humanoid")
if hrp and hum then
local weldHRP = hrp:FindFirstChild("WeldHRP")
if weldHRP then
antiSnowballWeldConn = weldHRP.Changed:Connect(function()
if hrp.WeldHRP.Enabled then
task.spawn(function()
while not hum.Sit do task.wait() end
hum.Sit = false
hum.AutoRotate = true
hum.HipHeight = 1
while hrp.WeldHRP.Enabled and task.wait() do
local head = char:FindFirstChild("Head")
if head then head.CFrame = hrp.CFrame + Vector3.new(0, 1.35, 0) end
end
hum.HipHeight = 0
end)
end
end)
end
end
end
antiSnowballWalkConn = RunService.Heartbeat:Connect(function()
if not loopRagdoll then return end
local c = LocalPlayer.Character
if not c or not c.Parent then return end
local h = c:FindFirstChild("Humanoid")
if h and h.Ragdolled and h.Ragdolled.Value then
breakLimbs(c)
local hrp2 = c:FindFirstChild("HumanoidRootPart")
if hrp2 and h.MoveDirection.Magnitude > 0 then
hrp2.CFrame = hrp2.CFrame + h.MoveDirection * 0.43
end
end
end)
task.spawn(function()
while loopRagdoll and task.wait(0.05) do
pcall(function()
local c = LocalPlayer.Character
local hrp2 = c and c:FindFirstChild("HumanoidRootPart")
if hrp2 then
local ce = game:GetService("ReplicatedStorage"):FindFirstChild("CharacterEvents")
local ragdollRemote = ce and ce:FindFirstChild("RagdollRemote")
if ragdollRemote then
ragdollRemote:FireServer(hrp2, 0.5)
end
breakLimbs(c)
end
end)
end
end)
Library:Notify("Anti Snowball ON", 2)
else
local char = LocalPlayer.Character
if char then
for _, part in pairs(char:GetChildren()) do
if part:IsA("BasePart") and part:FindFirstChild("RagdollLimbPart") then
part.RagdollLimbPart.WeldConstraint.Enabled = true
end
end
end
Library:Notify("Anti Snowball OFF", 2)
end
end)

DefAntiItems:AddToggle("EnableAntiRagdoll", {
Text = "Anti Ragdoll",
Default = false,
Tooltip = "Forces body to follow HRP during ragdoll, no animations, free movement",
})

local antiRagdollRunning = false
local antiRagdollConn = nil
Toggles.EnableAntiRagdoll:OnChanged(function()
local v = Toggles.EnableAntiRagdoll.Value
antiRagdollRunning = v
if antiRagdollConn then antiRagdollConn:Disconnect(); antiRagdollConn = nil end
if not v then
local c = LocalPlayer.Character
if c then
for _, part in pairs(c:GetChildren()) do
local fw = part:FindFirstChild("AntiRagdollWeld")
if fw then fw:Destroy() end
end
end
Library:Notify("Anti Ragdoll OFF", 2)
return
end
Library:Notify("Anti Ragdoll ON", 2)
local function stopAnims(char)
local hum = char and char:FindFirstChild("Humanoid")
if not hum then return end
local animator = hum:FindFirstChildOfClass("Animator")
if animator then
for _, track in pairs(animator:GetPlayingAnimationTracks()) do
track:Stop()
end
end
end
local function breakLimbs(char)
for _, part in pairs(char:GetChildren()) do
if part:IsA("BasePart") and part.Name ~= "Head" and part.Name ~= "HumanoidRootPart" then
local bsc = part:FindFirstChild("BallSocketConstraint")
if bsc then bsc.Enabled = false end
local rlp = part:FindFirstChild("RagdollLimbPart")
if rlp then
rlp.WeldConstraint.Enabled = false
end
end
end
end
local function weldLimbsToHRP(char)
local hrp = char:FindFirstChild("HumanoidRootPart")
if not hrp then return end
local limbNames = {"Left Arm", "Right Arm", "Left Leg", "Right Leg", "LowerTorso", "UpperTorso", "Head"}
for _, name in pairs(limbNames) do
local limb = char:FindFirstChild(name)
if limb and limb:IsA("BasePart") then
if not limb:FindFirstChild("AntiRagdollWeld") then
local w = Instance.new("Weld")
w.Name = "AntiRagdollWeld"
w.Part0 = hrp
w.Part1 = limb
w.C0 = hrp.CFrame:ToObjectSpace(limb.CFrame)
w.Parent = limb
end
end
end
end
local function removeFakeWelds(char)
for _, part in pairs(char:GetChildren()) do
local fw = part:FindFirstChild("AntiRagdollWeld")
if fw then fw:Destroy() end
end
end
local function isRagdolled(char)
for _, part in pairs(char:GetChildren()) do
if part:IsA("BasePart") and part.Name ~= "Head" then
local bsc = part:FindFirstChild("BallSocketConstraint")
if bsc and bsc.Enabled then return true end
end
end
local h = char:FindFirstChild("Humanoid")
if h and h:FindFirstChild("Ragdolled") and h.Ragdolled.Value then return true end
return false
end
local wasRagdoll = false
local savedCFrame = nil
antiRagdollConn = RunService.Heartbeat:Connect(function()
if not antiRagdollRunning then return end
local c = LocalPlayer.Character
if not c or not c.Parent then return end
local h = c:FindFirstChild("Humanoid")
if not h then return end
local hrp = c:FindFirstChild("HumanoidRootPart")
if not hrp then return end
local rag = isRagdolled(c)
if rag then
if not wasRagdoll then
wasRagdoll = true
savedCFrame = hrp.CFrame
weldLimbsToHRP(c)
end
breakLimbs(c)
stopAnims(c)
pcall(function()
local ce = game:GetService("ReplicatedStorage"):FindFirstChild("CharacterEvents")
local rd = ce and ce:FindFirstChild("RagdollRemote")
if rd then rd:FireServer(hrp, 0) end
end)
if h.MoveDirection.Magnitude > 0 then
local speed = h.WalkSpeed / 60
savedCFrame = savedCFrame + h.MoveDirection * speed
end
hrp.CFrame = savedCFrame
else
if wasRagdoll then
wasRagdoll = false
savedCFrame = nil
removeFakeWelds(c)
end
end
end)
end)

DefAntiItems:AddToggle("EnableAntiNetworkOwnership", {
Text = "Anti Network Ownership",
Default = false,
Tooltip = "Anti grab: no anims, no ragdoll, free movement when grabbed",
})

local antiNetOwnRunning = false
local antiNetOwnHbConn = nil
local antiNetOwnWelds = {}
local antiNetOwnActive = false 

local function isGrabbed()
local isHeld = LocalPlayer:FindFirstChild("IsHeld")
if isHeld and isHeld.Value then return true end
local c = LocalPlayer.Character
if c then
local head = c:FindFirstChild("Head")
if head and head:FindFirstChild("PartOwner") then return true end
end
return false
end

local function weldLimbsToHRP(char)
local hrp = char:FindFirstChild("HumanoidRootPart")
if not hrp then return end
local limbNames = {"Left Arm", "Right Arm", "Left Leg", "Right Leg", "LowerTorso", "UpperTorso", "Head"}
for _, name in ipairs(limbNames) do
local limb = char:FindFirstChild(name)
if limb and limb:IsA("BasePart") and not limb:FindFirstChild("AntiNetOwnWeld") then
local w = Instance.new("Weld")
w.Name = "AntiNetOwnWeld"
w.Part0 = hrp
w.Part1 = limb
w.C0 = hrp.CFrame:ToObjectSpace(limb.CFrame)
w.Parent = limb
table.insert(antiNetOwnWelds, w)
end
end
end

local function removeNetOwnWelds()
for _, w in ipairs(antiNetOwnWelds) do
pcall(function() w:Destroy() end)
end
antiNetOwnWelds = {}
end

Toggles.EnableAntiNetworkOwnership:OnChanged(function()
local v = Toggles.EnableAntiNetworkOwnership.Value
antiNetOwnRunning = v
if antiNetOwnHbConn then antiNetOwnHbConn:Disconnect(); antiNetOwnHbConn = nil end
removeNetOwnWelds()
antiNetOwnActive = false
if not v then
Library:Notify("Anti Network Ownership OFF", 2)
return
end
Library:Notify("Anti Network Ownership ON", 2)
antiNetOwnHbConn = RunService.Heartbeat:Connect(function()
if not antiNetOwnRunning then return end
local grabbed = isGrabbed()
local c = LocalPlayer.Character
if not c or not c.Parent then return end
local h = c:FindFirstChild("Humanoid")
if not h then return end
local hrp = c:FindFirstChild("HumanoidRootPart")
if not hrp then return end
if grabbed then
if not antiNetOwnActive then
antiNetOwnActive = true
weldLimbsToHRP(c)
end
local animator = h:FindFirstChildOfClass("Animator")
if animator then
for _, track in pairs(animator:GetPlayingAnimationTracks()) do
track:Stop()
end
end
for _, part in pairs(c:GetChildren()) do
if part:IsA("BasePart") and part.Name ~= "Head" and part.Name ~= "HumanoidRootPart" then
local bsc = part:FindFirstChild("BallSocketConstraint")
if bsc then bsc.Enabled = false end
local rlp = part:FindFirstChild("RagdollLimbPart")
if rlp then
local wc = rlp:FindFirstChild("WeldConstraint")
if wc then wc.Enabled = false end
end
end
end
pcall(function()
local ce = game:GetService("ReplicatedStorage"):FindFirstChild("CharacterEvents")
local st = ce and ce:FindFirstChild("Struggle")
if st then st:FireServer(LocalPlayer) end
end)
if h.MoveDirection.Magnitude > 0 then
local speed = h.WalkSpeed / 60
hrp.CFrame = hrp.CFrame + h.MoveDirection * speed
end
hrp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
else
if antiNetOwnActive then
antiNetOwnActive = false
removeNetOwnWelds()
end
end
end)
end)

DefAntiItems:AddToggle("EnableAntiBanana", {
Text = "Anti Banana",
Default = false,
Tooltip = "Prevents banana slip",
})

antiBananaRunning = false

Toggles.EnableAntiBanana:OnChanged(function()
if Toggles.EnableAntiBanana.Value then
antiBananaRunning = true
task.spawn(function()
while antiBananaRunning do
local char = LocalPlayer.Character
if char then
local hum = char:FindFirstChildOfClass("Humanoid")
local hrp = char:FindFirstChild("HumanoidRootPart")
if hum and hrp and hum.Health > 0 then
hum.Sit = true
hum:ChangeState(Enum.HumanoidStateType.Running)
local cam = workspace.CurrentCamera
local vec = cam and cam.CFrame.LookVector or Vector3.new(0, 0, -1)
hrp.CFrame = CFrame.new(hrp.Position, hrp.Position + Vector3.new(vec.X, 0, vec.Z))
end
end
task.wait()
end
end)
Library:Notify("Anti Banana ON", 2)
else
antiBananaRunning = false
Library:Notify("Anti Banana OFF", 2)
end
end)

DefAntiItems:AddToggle("EnableAntiPoison", {
Text = "Anti Poison",
Default = false,
Tooltip = "Destroy poison mushrooms",
})

antiPoisonConns = {}

antiPoisonConns = {}

function antiPoisonCleanup()
for _, conn in ipairs(antiPoisonConns) do
pcall(function() conn:Disconnect() end)
end
antiPoisonConns = {}
end

Toggles.EnableAntiPoison:OnChanged(function()
if Toggles.EnableAntiPoison.Value then
local setOwner = freezeGrabGetNetworkOwner()
local function destroyPoison()
for _, obj in ipairs(workspace:GetDescendants()) do
if obj:IsA("BasePart") then
local n = obj.Name:lower()
if n:find("poison") or n:find("mushroom") or n:find("foodmush") then
if setOwner then
pcall(function() setOwner:FireServer(obj, obj.CFrame) end)
end
pcall(function() obj:Destroy() end)
end
end
end
end
destroyPoison()
table.insert(antiPoisonConns, workspace.DescendantAdded:Connect(function(obj)
if obj:IsA("BasePart") then
local n = obj.Name:lower()
if n:find("poison") or n:find("mushroom") or n:find("foodmush") then
task.defer(function()
if setOwner and obj and obj.Parent then
pcall(function() setOwner:FireServer(obj, obj.CFrame) end)
end
if obj and obj.Parent then pcall(function() obj:Destroy() end) end
end)
end
end
end))
local healConn
healConn = game:GetService("RunService").Heartbeat:Connect(function()
local char = LocalPlayer.Character
local hum = char and char:FindFirstChildOfClass("Humanoid")
if hum and hum.Health > 0 and hum.Health < hum.MaxHealth then
pcall(function() hum.Health = hum.MaxHealth end)
end
if char then
for _, v in ipairs(char:GetDescendants()) do
local vn = v.Name:lower()
if vn:find("poison") or vn:find("sick") or vn:find("toxic") then
if v:IsA("BoolValue") then pcall(function() v.Value = false end) end
if v:IsA("NumberValue") then pcall(function() v.Value = 0 end) end
if v:IsA("IntValue") then pcall(function() v.Value = 0 end) end
if v:IsA("StringValue") then pcall(function() v.Value = "" end) end
end
end
end
end)
table.insert(antiPoisonConns, healConn)
Library:Notify("Anti Poison ON", 2)
else
antiPoisonCleanup()
Library:Notify("Anti Poison OFF", 2)
end
end)

local DefAntiAFK = DefAntiGrab 

antiAFKConn = nil
antiAFKLastPos = nil
antiAFKLastMove = 0

local function stopAntiAFK()
if antiAFKConn then antiAFKConn:Disconnect(); antiAFKConn = nil end
antiAFKLastPos = nil
end

DefAntiAFK:AddToggle("EnableAntiAFK", {
Text = "Anti AFK",
Default = false,
Tooltip = L("Делает маленький шаг если не двигаешься 1 минуту", "Takes a small step if you don't move for 1 minute"),
Callback = function(val)
if val then
antiAFKLastMove = tick()
antiAFKConn = game:GetService("RunService").Heartbeat:Connect(function()
local char = LocalPlayer.Character
if not char then return end
local hrp = char:FindFirstChild("HumanoidRootPart")
if not hrp then return end
local pos = hrp.Position
if antiAFKLastPos and (pos - antiAFKLastPos).Magnitude > 0.5 then
antiAFKLastMove = tick()
end
antiAFKLastPos = pos
if tick() - antiAFKLastMove >= 60 then
local fwd = hrp.CFrame.LookVector
hrp.CFrame = hrp.CFrame + fwd * 0.5
antiAFKLastMove = tick()
end
end)
Library:Notify("Anti AFK ON", 2)
else
stopAntiAFK()
Library:Notify("Anti AFK OFF", 2)
end
end
})

local DefAutoReset = Tabs.Def:AddRightGroupbox("Auto Reset", "rotate-ccw")

do
do
local pcldActive = false
local pcldConn = nil
local pcldDeathConn = nil

local function stopPCLD()
pcldActive = false
if pcldConn then pcldConn:Disconnect(); pcldConn = nil end
if pcldDeathConn then pcldDeathConn:Disconnect(); pcldDeathConn = nil end
end

local function startPCLD()
stopPCLD()
pcldActive = true
task.spawn(function()
local isFirstCycle = true
while pcldActive do
local char = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart", 5)
local hum = char:WaitForChild("Humanoid", 5)
if not hrp or not hum then task.wait(0.5) continue end
if hum.Health <= 0 then
char = LocalPlayer.CharacterAdded:Wait()
hrp = char:WaitForChild("HumanoidRootPart", 5)
hum = char:WaitForChild("Humanoid", 5)
end
if not hrp or not hum then continue end

if isFirstCycle then
local savedCF = hrp.CFrame
hrp.CFrame = CFrame.new(hrp.Position.X, 50000, hrp.Position.Z)
task.wait(0.05)
hum.Health = 0
char = LocalPlayer.CharacterAdded:Wait()
if not pcldActive then break end
hrp = char:WaitForChild("HumanoidRootPart", 5)
hum = char:WaitForChild("Humanoid", 5)
if hrp and hum then
task.wait(0.1)
hrp.CFrame = savedCF
task.wait(0.05)
hum.Health = 0
end
isFirstCycle = false
else
task.wait(0.1)
if hum then
hum.Health = 0
pcall(function() char:BreakJoints() end)
end
end

char = LocalPlayer.CharacterAdded:Wait()
if not pcldActive then break end
hum = char:WaitForChild("Humanoid", 5)
if hum then
hum.Died:Wait()
end
end
end)
end

DefAutoReset:AddToggle("EnablePCLDBreak", {
Text = "Auto PCLD Break",
Default = false,
Tooltip = L("Хитрая система смертей, которая ломает хитбоксы и делает тебя бессмертным", "Clever death system that breaks hitboxes and makes you immortal"),
Callback = function(Value)
if Value then
startPCLD()
else
stopPCLD()
end
end
})
end

do
local DefGucci = Tabs.Def:AddRightGroupbox("Gucci (Godmode & Invis)", "crown")

local gucciConn = nil
local gucciActive = false
local gucciVehicle = nil

local function stopGucci()
gucciActive = false
if gucciConn then gucciConn:Disconnect(); gucciConn = nil end

local char = LocalPlayer.Character
local hum = char and char:FindFirstChildOfClass("Humanoid")
if hum then
hum.Sit = false
hum:ChangeState(Enum.HumanoidStateType.GettingUp)
end

if gucciVehicle and gucciVehicle.Parent then
pcall(function()
local RS = game:GetService("ReplicatedStorage")
RS.MenuToys.DestroyToy:FireServer(gucciVehicle)
end)
end
gucciVehicle = nil
end

local function startGucci(mode)
stopGucci()
gucciActive = true

task.spawn(function()
local char = LocalPlayer.Character
local hrp = char and char:WaitForChild("HumanoidRootPart", 3)
local hum = char and char:WaitForChild("Humanoid", 3)
if not hrp or not hum then return end

local RS = game:GetService("ReplicatedStorage")
local savedCF = hrp.CFrame

local seat = nil
local isTrain = false

if mode == "Anti Kill (Train)" then
isTrain = true
local trainFolder = workspace:FindFirstChild("Map") and workspace.Map:FindFirstChild("AlwaysHereTweenedObjects")
local train = trainFolder and trainFolder:FindFirstChild("Train")
if train then
for _, d in ipairs(train:GetDescendants()) do
if d:IsA("Seat") then
seat = d
break
end
end
end
if not seat then
Library:Notify(L("Поезд не найден на карте!", "Train not found on map!"), 3)
stopGucci()
return
end
else
local toyName = "TractorGreen"
if mode == "Godmode (Blobman)" then
toyName = "CreatureBlobman"
end

pcall(function()
RS.MenuToys.SpawnToyRemoteFunction:InvokeServer(toyName, CFrame.new(0, 50000, 0), Vector3.zero)
end)

local folder = workspace:WaitForChild(LocalPlayer.Name .. "SpawnedInToys", 5)
if folder then
local toy = folder:WaitForChild(toyName, 5)
if toy then
gucciVehicle = toy
seat = toy:WaitForChild("VehicleSeat", 3) or toy:FindFirstChildWhichIsA("VehicleSeat", true) or toy:FindFirstChildWhichIsA("Seat", true)
end
end
end

if not seat then return end

local t0 = tick()
while seat.Occupant ~= hum and (tick() - t0) < 3 and gucciActive do
if not isTrain then
seat.CFrame = CFrame.new(0, 50000, 0)
end
hrp.CFrame = seat.CFrame + Vector3.new(0, 2, 0)
seat:Sit(hum)
task.wait(0.05)
end

if not gucciActive then return end

hrp.CFrame = savedCF

gucciConn = game:GetService("RunService").Heartbeat:Connect(function()
if not hrp or not hrp.Parent or not hum then return end

pcall(function() RS.CharacterEvents.RagdollRemote:FireServer(hrp, 0) end)

if not isTrain and seat and seat.Parent then
seat.CFrame = CFrame.new(0, 50000, 0)
seat.AssemblyLinearVelocity = Vector3.zero
seat.AssemblyAngularVelocity = Vector3.zero
end

if hum.Sit then
hrp.CFrame = savedCF
else
savedCF = hrp.CFrame
end
end)

Library:Notify("Gucci Mode (" .. mode .. ") Активирован!", 2)
end)
end

DefGucci:AddToggle("EnableGucci", {
Text = L("Включить Gucci Mode", "Enable Gucci Mode"),
Default = false,
Tooltip = L("Делает тебя бессмертным или невидимым за счет бага", "Makes you immortal or invisible via exploit"),
Callback = function(Value)
local mode = Options.GucciType and Options.GucciType.Value or "Invisible (Tractor)"
if mode == "Invisible Gucci (Train)" then
invisGucciActive = Value
if Value then
startInvisGucciFull()
Library:Notify(L("Invisible Gucci ВКЛ", "Invisible Gucci ON"), 2)
else
stopInvisGucci()
Library:Notify(L("Invisible Gucci ВЫКЛ", "Invisible Gucci OFF"), 2)
end
elseif Value then
startGucci(mode)
else
stopGucci()
Library:Notify(L("Gucci Mode Отключен", "Gucci Mode Disabled"), 2)
end
end
})

DefGucci:AddDropdown("GucciType", {
Text = L("Тип Gucci", "Gucci Type"),
Default = "Invisible (Tractor)",
Values = {"Invisible (Tractor)", "Godmode (Blobman)", "Anti Kill (Train)", "Invisible Gucci (Train)"},
Tooltip = L("Tractor - невидимость, Blobman - Годмод, Train - защита через поезд", "Tractor - invisibility, Blobman - Godmode, Train - protection via train"),
})

Options.GucciType:OnChanged(function()
if Toggles.EnableGucci and Toggles.EnableGucci.Value then
if invisGucciActive then
invisGucciActive = false
stopInvisGucci()
else
stopGucci()
end
local mode = Options.GucciType and Options.GucciType.Value
if mode == "Invisible Gucci (Train)" then
invisGucciActive = true
startInvisGucciFull()
Library:Notify(L("Invisible Gucci ВКЛ", "Invisible Gucci ON"), 2)
else
startGucci(mode)
end
end
end)
end

end

invisGucciConn = nil
invisGucciSafePos = nil
invisGucciRestoreFrames = 0
invisGucciActive = false

function startInvisGucci()
local char = LocalPlayer.Character
if not char then return end
local hum = char:WaitForChild("Humanoid")
local hrp = char:WaitForChild("HumanoidRootPart")
invisGucciSafePos = hrp.Position
local folder = workspace:FindFirstChild("Map")
local tweened = folder and folder:FindFirstChild("AlwaysHereTweenedObjects")
local train = tweened and tweened:FindFirstChild("Train")
local seat = nil
if train then
for _, d in ipairs(train:GetDescendants()) do
if d:IsA("Seat") then
seat = d
break
end
end
end
if seat then
hrp.CFrame = seat.CFrame + Vector3.new(0, 2, 0)
seat:Sit(hum)
end
hum:GetPropertyChangedSignal("Jump"):Connect(function()
if hum.Jump and hum.Sit then
invisGucciRestoreFrames = 15
invisGucciSafePos = hrp.Position
end
end)
if invisGucciConn then invisGucciConn:Disconnect() end
invisGucciConn = game:GetService("RunService").Heartbeat:Connect(function()
if not hrp or not hum then return end
local rs = game:GetService("ReplicatedStorage")
local ce = rs:FindFirstChild("CharacterEvents")
if ce then
local ragdoll = ce:FindFirstChild("RagdollRemote")
if ragdoll then ragdoll:FireServer(hrp, 0) end
end
if invisGucciRestoreFrames > 0 then
hrp.CFrame = CFrame.new(invisGucciSafePos)
invisGucciRestoreFrames = invisGucciRestoreFrames - 1
end
end)
task.spawn(function()
while hum.Sit do task.wait(1) end
task.wait(0)
if hrp then hrp.CFrame = CFrame.new(invisGucciSafePos) end
end)
end

function stopInvisGucci()
if invisGucciConn then
invisGucciConn:Disconnect()
invisGucciConn = nil
end
local char = LocalPlayer.Character
if char then pcall(function() char:BreakJoints() end) end
end

function startInvisGucciFull()
startInvisGucci()
task.spawn(function()
while invisGucciActive do
local folder = workspace:FindFirstChild("Map")
local tweened = folder and folder:FindFirstChild("AlwaysHereTweenedObjects")
if not (tweened and tweened:FindFirstChild("Train")) then
stopInvisGucci()
local retries = 0
repeat
task.wait(0.2)
retries = retries + 1
folder = workspace:FindFirstChild("Map")
tweened = folder and folder:FindFirstChild("AlwaysHereTweenedObjects")
until (tweened and tweened:FindFirstChild("Train")) or retries > 25 or not invisGucciActive
if invisGucciActive and tweened and tweened:FindFirstChild("Train") then
startInvisGucci()
end
end
task.wait(0.5)
end
end)
end

DefAutoReset:AddToggle("EnableAutoReset", {
Text = "Auto Reset",
Default = false,
Tooltip = "\u{0410}\u{0432}\u{0442}\u{043e}\u{043c}\u{0430}\u{0442}\u{0438}\u{0447}\u{0435}\u{0441}\u{043a}\u{0438} \u{0440}\u{0435}\u{0441}\u{0435}\u{0442}\u{0438}\u{0442} \u{0435}\u{0441}\u{043b}\u{0438} \u{0441}\u{0435}\u{0440}\u{0432}\u{0435}\u{0440} \u{043f}\u{0438}\u{0448}\u{0435}\u{0442} Flying (\u{0430}\u{043d}\u{0442}\u{0438}-\u{0431}\u{0430}\u{043d})",
})

autoResetConn = nil

Toggles.EnableAutoReset:OnChanged(function()
if Toggles.EnableAutoReset.Value then
local rs = game:GetService("ReplicatedStorage")
local gce = rs:FindFirstChild("GameCorrectionEvents")
local notify = gce and gce:FindFirstChild("GameCorrectionsNotify")
if notify then
autoResetConn = notify.OnClientEvent:Connect(function(r)
if r == "Flying" then
local char = LocalPlayer.Character
local hum = char and char:FindFirstChildOfClass("Humanoid")
if hum then
Library:Notify("Auto Reset: \u{0440}\u{0435}\u{0441}\u{0435}\u{0442} \u{043e}\u{0442} \u{0431}\u{0430}\u{043d}\u{0430}", 3)
pcall(function() char:BreakJoints() end)
pcall(function() hum.Health = 0 end)
end
end
end)
end
Library:Notify(L("Auto Reset ВКЛ", "Auto Reset ON"), 2)
else
if autoResetConn then autoResetConn:Disconnect() autoResetConn = nil end
Library:Notify(L("Auto Reset ВЫКЛ", "Auto Reset OFF"), 2)
end
end)

DefAutoReset:AddToggle("EnableLoopReset", {
Text = "Loop Reset",
Default = false,
Tooltip = L("Бесконечно ресетает персонажа (BreakJoints + Health=0)", "Infinitely resets character (BreakJoints + Health=0)"),
})

loopResetConn = nil

Toggles.EnableLoopReset:OnChanged(function()
if Toggles.EnableLoopReset.Value then
loopResetConn = task.spawn(function()
while Toggles.EnableLoopReset and Toggles.EnableLoopReset.Value do
local char = LocalPlayer.Character
if char then
local hum = char:FindFirstChildOfClass("Humanoid")
if hum and hum.Health > 0 then
pcall(function() char:BreakJoints() end)
pcall(function() hum.Health = 0 end)
end
end
task.wait(1)
end
end)
Library:Notify(L("Loop Reset ВКЛ", "Loop Reset ON"), 2)
else
if loopResetConn then task.cancel(loopResetConn) loopResetConn = nil end
DefAntiBlobman = DefAntiGrab 
end
end)

DefAntiBlobman = DefAntiGrab 

DefAntiBlobman:AddToggle("EnableAntiBlobman", {
Text = "Anti Blobman",
Default = false,
Tooltip = L("Нельзя тебя схватить + удаляет детекторы", "Cannot be grabbed + removes detectors"),
})

antiBlobmanRunning = false
antiBlobmanDescConn = nil
antiBlobmanHeartbeat = nil
antiBlobmanLoopConn = nil

function antiBlobmanFindAllBlobs()
local blobs = {}
local plotItems = workspace:FindFirstChild("PlotItems")
if plotItems then
for _, plot in pairs(plotItems:GetChildren()) do
if plot.Name ~= "PlayersInPlots" then
for _, item in pairs(plot:GetChildren()) do
if item.Name == "CreatureBlobman" then
table.insert(blobs, item)
end
end
end
end
end
for _, plr in pairs(game:GetService("Players"):GetPlayers()) do
local toyFolder = workspace:FindFirstChild(plr.Name .. "SpawnedInToys")
if toyFolder then
for _, item in pairs(toyFolder:GetChildren()) do
if item.Name == "CreatureBlobman" then
table.insert(blobs, item)
end
end
end
end
for _, child in pairs(workspace:GetChildren()) do
if child.Name == "CreatureBlobman" then
table.insert(blobs, child)
end
end
return blobs
end

function antiBlobmanFireDrop(blob, hrp)
local sObj = blob:FindFirstChild("BlobmanSeatAndOwnerScript")
local rightDetector = blob:FindFirstChild("RightDetector")
local leftDetector = blob:FindFirstChild("LeftDetector")
if not sObj then return end
local dropEvent = sObj:FindFirstChild("CreatureDrop")
if not dropEvent then return end
if rightDetector then
local rightWeld = rightDetector:FindFirstChild("RightWeld")
if rightWeld then pcall(function() dropEvent:FireServer(rightWeld, hrp) end) end
end
if leftDetector then
local leftWeld = leftDetector:FindFirstChild("LeftWeld")
if leftWeld then pcall(function() dropEvent:FireServer(leftWeld, hrp) end) end
end
local ce = game:GetService("ReplicatedStorage"):FindFirstChild("CharacterEvents")
if ce then
local struggle = ce:FindFirstChild("Struggle")
if struggle then pcall(function() struggle:FireServer(LocalPlayer) end) end
end
end

Toggles.EnableAntiBlobman:OnChanged(function()
if Toggles.EnableAntiBlobman.Value then
antiBlobmanRunning = true
local char = LocalPlayer.Character
local hrp = char and char:FindFirstChild("HumanoidRootPart")
if hrp and char and not char:FindFirstChild("TruePositionPart") then
local tpp = Instance.new("Part")
tpp.Parent = char
tpp.Name = "TruePositionPart"
tpp.Anchored = true
tpp.Transparency = 1
tpp.CanCollide = false
tpp.Size = Vector3.new(0.1, 0.1, 0.1)
tpp.CFrame = CFrame.new(0, -10000000, 0)
end
antiBlobmanDescConn = workspace.DescendantAdded:Connect(function(toy)
if toy.Name == "CreatureBlobman" and antiBlobmanRunning then
task.defer(function()
local leftDetector = toy:WaitForChild("LeftDetector", 3)
local rightDetector = toy:WaitForChild("RightDetector", 3)
local c = LocalPlayer.Character
local h = c and c:FindFirstChild("HumanoidRootPart")
if h and toy:FindFirstChild("Head") then
local d = (toy.Head.Position - h.Position).Magnitude
local curDist = 15
if d <= curDist then
if leftDetector then pcall(function() leftDetector:Destroy() end) end
if rightDetector then pcall(function() rightDetector:Destroy() end) end
end
end
end)
end
end)
antiBlobmanHeartbeat = RunService.Heartbeat:Connect(function()
local c = LocalPlayer.Character
if not c then return end
local h = c:FindFirstChild("HumanoidRootPart")
if not h then return end
local tpp = c:FindFirstChild("TruePositionPart")
if tpp and h then
local rootAtt = h:FindFirstChild("RootAttachment")
if rootAtt and rootAtt.Parent == h then
pcall(function() rootAtt.Parent = tpp end)
end
end
local isGrabbed = false
for _, part in pairs(c:GetChildren()) do
if part:IsA("Part") and part.Massless then
part.Massless = false
isGrabbed = true
end
end
if isGrabbed then
h.AssemblyLinearVelocity = Vector3.new(0, 15000000, 0)
local blobs = antiBlobmanFindAllBlobs()
for _, blob in ipairs(blobs) do
pcall(function() antiBlobmanFireDrop(blob, h) end)
end
end
end)
antiBlobmanLoopConn = RunService.Heartbeat:Connect(function()
local c = LocalPlayer.Character
local h = c and c:FindFirstChild("HumanoidRootPart")
if not h then return end
local curDist = 15
local blobs = antiBlobmanFindAllBlobs()
for _, blob in ipairs(blobs) do
local blobHead = blob:FindFirstChild("Head")
if blobHead then
local d = (blobHead.Position - h.Position).Magnitude
if d <= curDist then
local ld = blob:FindFirstChild("LeftDetector")
local rd = blob:FindFirstChild("RightDetector")
if ld and ld.Parent then pcall(function() ld:Destroy() end) end
if rd and rd.Parent then pcall(function() rd:Destroy() end) end
end
end
end
end)
Library:Notify(L("Anti Blobman ВКЛ", "Anti Blobman ON"), 2)
else
antiBlobmanRunning = false
if antiBlobmanDescConn then antiBlobmanDescConn:Disconnect() antiBlobmanDescConn = nil end
if antiBlobmanHeartbeat then antiBlobmanHeartbeat:Disconnect() antiBlobmanHeartbeat = nil end
if antiBlobmanLoopConn then antiBlobmanLoopConn:Disconnect() antiBlobmanLoopConn = nil end
local c = LocalPlayer.Character
local h = c and c:FindFirstChild("HumanoidRootPart")
local tpp = c and c:FindFirstChild("TruePositionPart")
if h and tpp then
local rootAtt = tpp:FindFirstChild("RootAttachment")
if rootAtt then pcall(function() rootAtt.Parent = h end) end
pcall(function() tpp:Destroy() end)
end
Library:Notify(L("Anti Blobman ВЫКЛ", "Anti Blobman OFF"), 2)
end
end)

local DefSecretPlatform = Tabs.Def:AddLeftGroupbox("Secret Platform", "square-stack")

secretPlatformActive = false
secretPlatformPart = nil
secretPlatformConn = nil
secretPlatformSavedPos = nil
secretPlatformOrigFallenHeight = nil

local SECRET_PLATFORM_Y = -300
local SECRET_PLATFORM_SIZE = Vector3.new(30, 2, 30)

local function createSecretPlatform()
local char = LocalPlayer.Character
if not char then return end
local hrp = char:FindFirstChild("HumanoidRootPart")
if not hrp then return end

secretPlatformSavedPos = hrp.CFrame

secretPlatformOrigFallenHeight = workspace.FallenPartsDestroyHeight
workspace.FallenPartsDestroyHeight = -999999

secretPlatformPart = Instance.new("Part")
secretPlatformPart.Name = "SecretPlatform_Oblivion"
secretPlatformPart.Anchored = true
secretPlatformPart.Size = SECRET_PLATFORM_SIZE
secretPlatformPart.Position = Vector3.new(0, SECRET_PLATFORM_Y, 0)
secretPlatformPart.Transparency = 0.4
secretPlatformPart.Material = Enum.Material.Neon
secretPlatformPart.Color = Color3.fromRGB(80, 0, 255)
secretPlatformPart.CanCollide = true
secretPlatformPart.Parent = workspace

local light = Instance.new("PointLight")
light.Color = Color3.fromRGB(120, 0, 255)
light.Brightness = 3
light.Range = 40
light.Parent = secretPlatformPart

hrp.CFrame = CFrame.new(0, SECRET_PLATFORM_Y + 5, 0)
hrp.AssemblyLinearVelocity = Vector3.zero
hrp.AssemblyAngularVelocity = Vector3.zero
local hum = char:FindFirstChildOfClass("Humanoid")
if hum then hum.PlatformStand = false end

secretPlatformConn = RunService.Heartbeat:Connect(function()
if not secretPlatformActive then return end
local c = LocalPlayer.Character
if not c then return end
local h = c:FindFirstChild("HumanoidRootPart")
if not h then return end
if h.Position.Y < SECRET_PLATFORM_Y - 50 then
h.CFrame = CFrame.new(0, SECRET_PLATFORM_Y + 5, 0)
h.AssemblyLinearVelocity = Vector3.zero
h.AssemblyAngularVelocity = Vector3.zero
end
if workspace.FallenPartsDestroyHeight > -999999 then
workspace.FallenPartsDestroyHeight = -999999
end
end)

Library:Notify(L("Secret Platform: телепорт под карту!", "Secret Platform: teleport under map!"), 3)
end

local function removeSecretPlatform()
if secretPlatformConn then
secretPlatformConn:Disconnect()
secretPlatformConn = nil
end

if secretPlatformOrigFallenHeight ~= nil then
workspace.FallenPartsDestroyHeight = secretPlatformOrigFallenHeight
secretPlatformOrigFallenHeight = nil
end

local char = LocalPlayer.Character
if char and secretPlatformSavedPos then
local hrp = char:FindFirstChild("HumanoidRootPart")
if hrp then
hrp.CFrame = secretPlatformSavedPos
hrp.AssemblyLinearVelocity = Vector3.zero
hrp.AssemblyAngularVelocity = Vector3.zero
local hum = char:FindFirstChildOfClass("Humanoid")
if hum then hum.PlatformStand = false end
end
end
secretPlatformSavedPos = nil

if secretPlatformPart then
pcall(function() secretPlatformPart:Destroy() end)
secretPlatformPart = nil
end

Library:Notify(L("Secret Platform: возврат на карту", "Secret Platform: back to map"), 3)
end

DefSecretPlatform:AddToggle("EnableSecretPlatform", {
Text = "Secret Platform",
Default = false,
Tooltip = L("Телепорт на скрытую платформу под картой (видна только тебе). Анти-войд включается автоматически.", "Teleport to hidden platform under map (visible only to you). Anti-void enabled automatically."),
}):AddKeyPicker("SecretPlatformKeybind", {
Default = "None",
SyncToggleState = true,
Mode = "Toggle",
Text = "Platform Key",
NoUI = false,
})

Toggles.EnableSecretPlatform:OnChanged(function()
if Toggles.EnableSecretPlatform.Value then
secretPlatformActive = true
createSecretPlatform()
else
secretPlatformActive = false
removeSecretPlatform()
end
end)

LocalPlayer.CharacterAdded:Connect(function(char)
if secretPlatformActive then
secretPlatformActive = false
removeSecretPlatform()
Toggles.EnableSecretPlatform:SetValue(false)
end
end)

VisualCamera = Tabs.Visual:AddLeftGroupbox("Camera", "camera")
defaultFOV = 70

function UpdateFOV()
local camera = workspace.CurrentCamera
if camera then
if Toggles.EnableFOV.Value then
camera.FieldOfView = Options.FOVValue.Value
else
camera.FieldOfView = defaultFOV
end
end
end

VisualCamera:AddToggle("EnableFOV", {
Text = "Custom FOV",
Default = false,
Tooltip = L("Включить кастомный Field of View", "Enable custom Field of View"),
Callback = function(Value)
UpdateFOV()
end
})

VisualCamera:AddSlider("FOVValue", {
Text = "FOV Amount",
Default = 70,
Min = 0,
Max = 120,
Rounding = 0,
Compact = false,
Tooltip = L("Настройка угла обзора", "FOV setting"),
Callback = function(Value)
UpdateFOV()
end
})

task.spawn(function()
while task.wait(0.1) do
if Toggles.EnableFOV and Toggles.EnableFOV.Value then
local camera = workspace.CurrentCamera
if camera and camera.FieldOfView ~= Options.FOVValue.Value then
camera.FieldOfView = Options.FOVValue.Value
end
end
end
end)

do
tpDefCamMode = LocalPlayer.CameraMode
tpDefMaxZoom = LocalPlayer.CameraMaxZoomDistance
tpDefMinZoom = LocalPlayer.CameraMinZoomDistance

VisualCamera:AddToggle("EnableThirdPerson", {
Text = "3rd Person",
Default = false,
Tooltip = L("Вид от третьего лица: отдаляй камеру колесиком мыши", "Third person view: zoom out with mouse wheel"),
Callback = function(Value)
LocalPlayer.CameraMaxZoomDistance = 1e9
if Value then
LocalPlayer.CameraMode = Enum.CameraMode.Classic
else
LocalPlayer.CameraMode = Enum.CameraMode.LockFirstPerson
end
end
})

VisualCamera:AddSlider("ThirdPersonDistance", {
Text = L("Дальность камеры", "Camera Distance"),
Default = 128,
Min = 5,
Max = 400,
Rounding = 0,
Compact = false,
Tooltip = L("Максимальное отдаление камеры колесиком мыши", "Max camera zoom with mouse wheel"),
Callback = function(Value)
if Toggles.EnableThirdPerson and Toggles.EnableThirdPerson.Value then
LocalPlayer.CameraMaxZoomDistance = Value
end
end
})

VisualCamera:AddToggle("Enable4x3", {
Text = "Camera 4:3",
Default = false,
Tooltip = L("Сплющить экран в 4:3 (через FOV)", "Squash screen to 4:3 (via FOV)"),
Callback = function(Value)
local cam = workspace.CurrentCamera
if not cam then return end
if Value then
_G.Cam4x3DefaultFOV = cam.FieldOfView
local vFOV = math.rad(cam.FieldOfView)
local newFOV = math.deg(2 * math.atan(math.tan(vFOV / 2) * 0.75))
cam.FieldOfView = newFOV
_G.Cam4x3Conn = RunService.RenderStepped:Connect(function()
if workspace.CurrentCamera and _G.Cam4x3DefaultFOV then
local cur = math.rad(_G.Cam4x3DefaultFOV)
local nf = math.deg(2 * math.atan(math.tan(cur / 2) * 0.75))
workspace.CurrentCamera.FieldOfView = nf
end
end)
else
if _G.Cam4x3Conn then _G.Cam4x3Conn:Disconnect() _G.Cam4x3Conn = nil end
if _G.Cam4x3DefaultFOV then
if workspace.CurrentCamera then workspace.CurrentCamera.FieldOfView = _G.Cam4x3DefaultFOV end
_G.Cam4x3DefaultFOV = nil
end
end
end
})

local VisualShaders = Tabs.Visual:AddLeftGroupbox("Shaders", "palette")
local Lighting = game:GetService("Lighting")

originalLight = {
Brightness = Lighting.Brightness,
Ambient = Lighting.Ambient,
OutdoorAmbient = Lighting.OutdoorAmbient,
TimeOfDay = Lighting.TimeOfDay,
ClockTime = Lighting.ClockTime,
FogEnd = Lighting.FogEnd,
FogStart = Lighting.FogStart,
ExposureCompensation = Lighting.ExposureCompensation,
}

local ShaderThemes = {
["Night"] = {
Brightness = 2.0,
Ambient = Color3.fromRGB(60, 70, 130),
OutdoorAmbient = Color3.fromRGB(70, 80, 150),
ClockTime = 12,
FogEnd = 100000,
FogStart = 90000,
ExposureCompensation = 0.3,
CC = { Brightness = 0, Contrast = 0.1, Saturation = -0.1, Tint = Color3.fromRGB(120, 140, 220) },
Bloom = { Intensity = 0.6, Size = 32, Threshold = 0.7 },
},
["Dusk"] = {
Brightness = 2.5,
Ambient = Color3.fromRGB(170, 110, 75),
OutdoorAmbient = Color3.fromRGB(190, 120, 85),
ClockTime = 12,
FogEnd = 100000,
FogStart = 90000,
ExposureCompensation = 0.4,
CC = { Brightness = 0, Contrast = 0.1, Saturation = 0.2, Tint = Color3.fromRGB(255, 200, 160) },
Bloom = { Intensity = 0.6, Size = 32, Threshold = 0.7 },
},
["Sunset"] = {
Brightness = 2.5,
Ambient = Color3.fromRGB(210, 120, 75),
OutdoorAmbient = Color3.fromRGB(230, 130, 85),
ClockTime = 12,
FogEnd = 100000,
FogStart = 90000,
ExposureCompensation = 0.4,
CC = { Brightness = 0, Contrast = 0.15, Saturation = 0.35, Tint = Color3.fromRGB(255, 170, 120) },
Bloom = { Intensity = 0.8, Size = 40, Threshold = 0.5 },
},
["Fog"] = {
Brightness = 2.5,
Ambient = Color3.fromRGB(200, 200, 210),
OutdoorAmbient = Color3.fromRGB(200, 200, 210),
ClockTime = 12,
FogEnd = 300,
FogStart = 50,
ExposureCompensation = 0,
CC = { Brightness = 0.05, Contrast = -0.1, Saturation = -0.2, Tint = Color3.fromRGB(220, 220, 230) },
Bloom = { Intensity = 0.3, Size = 48, Threshold = 0.9 },
},
["Bright"] = {
Brightness = 3.0,
Ambient = Color3.fromRGB(220, 220, 220),
OutdoorAmbient = Color3.fromRGB(240, 240, 240),
ClockTime = 12,
FogEnd = 100000,
FogStart = 90000,
ExposureCompensation = 0.5,
CC = { Brightness = 0.1, Contrast = 0.2, Saturation = 0.5, Tint = Color3.fromRGB(255, 255, 240) },
Bloom = { Intensity = 1.0, Size = 32, Threshold = 0.4 },
},
["Neon"] = {
Brightness = 2.5,
Ambient = Color3.fromRGB(80, 40, 120),
OutdoorAmbient = Color3.fromRGB(90, 50, 140),
ClockTime = 22,
FogEnd = 1500,
FogStart = 200,
ExposureCompensation = 0.3,
CC = { Brightness = 0, Contrast = 0.4, Saturation = 0.8, Tint = Color3.fromRGB(180, 100, 255) },
Bloom = { Intensity = 1.3, Size = 44, Threshold = 0.35 },
},
["Cold"] = {
Brightness = 2.5,
Ambient = Color3.fromRGB(120, 140, 180),
OutdoorAmbient = Color3.fromRGB(140, 160, 200),
ClockTime = 12,
FogEnd = 100000,
FogStart = 90000,
ExposureCompensation = 0.1,
CC = { Brightness = 0, Contrast = 0.1, Saturation = 0.1, Tint = Color3.fromRGB(150, 180, 255) },
Bloom = { Intensity = 0.4, Size = 24, Threshold = 0.8 },
},
["Warm"] = {
Brightness = 2.5,
Ambient = Color3.fromRGB(180, 140, 100),
OutdoorAmbient = Color3.fromRGB(200, 160, 120),
ClockTime = 12,
FogEnd = 100000,
FogStart = 90000,
ExposureCompensation = 0.2,
CC = { Brightness = 0.02, Contrast = 0.1, Saturation = 0.3, Tint = Color3.fromRGB(255, 200, 150) },
Bloom = { Intensity = 0.5, Size = 28, Threshold = 0.6 },
},
["Realistic"] = {
Brightness = 3.5,
Ambient = Color3.fromRGB(80, 80, 80),
OutdoorAmbient = Color3.fromRGB(120, 120, 120),
ClockTime = 14,
FogEnd = 50000,
FogStart = 5000,
ExposureCompensation = 0.3,
GlobalShadows = true,
Use2022Materials = true,
CC = { Brightness = 0.05, Contrast = 0.2, Saturation = 0.3, Tint = Color3.fromRGB(255, 250, 245) },
Bloom = { Intensity = 0.3, Size = 24, Threshold = 0.8 },
SunRays = { Intensity = 0.05, Spread = 0.1 },
},
["UltraRealism"] = {
Brightness = 3.0,
Ambient = Color3.fromRGB(50, 50, 50),
OutdoorAmbient = Color3.fromRGB(130, 130, 130),
ClockTime = 15.5,
FogEnd = 8000,
FogStart = 500,
ExposureCompensation = 0.2,
GlobalShadows = true,
Use2022Materials = true,
ForceMaterials = true,
EnvDiffuse = 1.0,
EnvSpecular = 1.0,
CC = { Brightness = 0.05, Contrast = 0.25, Saturation = 0.3, Tint = Color3.fromRGB(255, 250, 240) },
Bloom = { Intensity = 0.05, Size = 10, Threshold = 2.0 },
SunRays = { Intensity = 0.3, Spread = 0.2 },
DOF = { FocusDistance = 25, InFocusRadius = 50, NearIntensity = 0.1, FarIntensity = 0.3 },
},
["Twilight"] = {
Brightness = 3.5,
Ambient = Color3.fromRGB(59, 33, 27),
OutdoorAmbient = Color3.fromRGB(34, 0, 49),
ClockTime = 6.7,
FogEnd = 1000,
FogStart = 0,
FogColor = Color3.fromRGB(94, 76, 106),
ExposureCompensation = 0.24,
ColorShift_Top = Color3.fromRGB(240, 127, 14),
ColorShift_Bottom = Color3.fromRGB(11, 0, 20),
CC = { Brightness = 0, Contrast = 0, Saturation = 0.05, Tint = Color3.fromRGB(255, 224, 219) },
Bloom = { Intensity = 0.1, Size = 100, Threshold = 0 },
SunRays = { Intensity = 0.05, Spread = 0.8 },
CustomSkybox = { Bk = "rbxassetid://323494035", Dn = "rbxassetid://323494368", Ft = "rbxassetid://323494130", Lf = "rbxassetid://323494252", Rt = "rbxassetid://323494067", Up = "rbxassetid://323493360" },
},
["Luminous"] = {
Brightness = 2,
Ambient = Color3.fromRGB(50, 50, 50),
OutdoorAmbient = Color3.fromRGB(150, 150, 150),
ClockTime = 10,
FogEnd = 10000,
FogStart = 0,
FogColor = Color3.fromRGB(20, 20, 20),
ExposureCompensation = 0.5,
ColorShift_Top = Color3.fromRGB(250, 250, 250),
ColorShift_Bottom = Color3.fromRGB(250, 250, 250),
CC = { Brightness = 0, Contrast = 0, Saturation = 0, Tint = Color3.fromRGB(255, 255, 255) },
Bloom = { Intensity = 0.1, Size = 100, Threshold = 0 },
CustomSkybox = { Bk = "rbxassetid://323494035", Dn = "rbxassetid://323494368", Ft = "rbxassetid://323494130", Lf = "rbxassetid://323494252", Rt = "rbxassetid://323494067", Up = "rbxassetid://323493360" },
},
["Sandstorm"] = {
Brightness = 2.5,
Ambient = Color3.fromRGB(80, 40, 10),
OutdoorAmbient = Color3.fromRGB(100, 50, 10),
ClockTime = 7,
FogEnd = 1000,
FogStart = 0,
FogColor = Color3.fromRGB(100, 55, 20),
ExposureCompensation = 0.5,
ColorShift_Top = Color3.fromRGB(240, 127, 14),
ColorShift_Bottom = Color3.fromRGB(240, 120, 20),
CC = { Brightness = 0, Contrast = 0, Saturation = 0, Tint = Color3.fromRGB(255, 220, 180) },
Bloom = { Intensity = 0.1, Size = 100, Threshold = 0 },
CustomSkybox = { Bk = "rbxassetid://323494035", Dn = "rbxassetid://323494368", Ft = "rbxassetid://323494130", Lf = "rbxassetid://323494252", Rt = "rbxassetid://323494067", Up = "rbxassetid://323493360" },
},
["Arctic"] = {
Brightness = 2,
Ambient = Color3.fromRGB(0, 50, 100),
OutdoorAmbient = Color3.fromRGB(150, 170, 200),
GlobalShadows = true,
ExposureCompensation = 0,
CC = { Brightness = 0, Contrast = 0.2, Saturation = -0.3, Tint = Color3.fromRGB(220, 240, 255) },
Bloom = { Intensity = 0.8, Size = 24, Threshold = 0.6 },
Atmosphere = { Density = 0.45, Color = Color3.fromRGB(180, 200, 255), Decay = Color3.fromRGB(255, 255, 255), Glare = 0.6 },
},
}

local origMaterials = {}
local function applyRealMaterials()
task.spawn(function()
for _, v in ipairs(workspace:GetDescendants()) do
if v:IsA("BasePart") then
if not origMaterials[v] then
origMaterials[v] = v.Material
end
if v.Material == Enum.Material.Plastic or v.Material == Enum.Material.SmoothPlastic then
v.Material = Enum.Material.Concrete
end
end
end
end)
end
local function restoreMaterials()
task.spawn(function()
for k, v in pairs(origMaterials) do
if k and k.Parent then
k.Material = v
end
end
origMaterials = {}
end)
end

local ccEffect, bloomEffect, sunRaysEffect, dofEffect
currentShaderTheme = nil

existingEffects = {}
local function clearExistingEffects()
for _, v in ipairs(Lighting:GetChildren()) do
if v:IsA("ColorCorrectionEffect") or v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") then
table.insert(existingEffects, v)
v.Enabled = false
end
end
end
local function restoreExistingEffects()
for _, v in ipairs(existingEffects) do
v.Enabled = true
end
existingEffects = {}
end

local function applyShaderTheme(themeName)
local theme = ShaderThemes[themeName]
if not theme then return end
currentShaderTheme = themeName

clearExistingEffects()

Lighting.Brightness = theme.Brightness
Lighting.Ambient = theme.Ambient
Lighting.OutdoorAmbient = theme.OutdoorAmbient
if theme.ClockTime then
Lighting.ClockTime = theme.ClockTime
elseif theme.TimeOfDay then
Lighting.TimeOfDay = theme.TimeOfDay
end
Lighting.FogEnd = theme.FogEnd
if theme.FogStart then
Lighting.FogStart = theme.FogStart
end
if theme.ExposureCompensation then
Lighting.ExposureCompensation = theme.ExposureCompensation
end

if theme.GlobalShadows ~= nil then
Lighting.GlobalShadows = theme.GlobalShadows
else
Lighting.GlobalShadows = false
end

if theme.Use2022Materials then
pcall(function() game:GetService("MaterialService").Use2022Materials = true end)
else
pcall(function() game:GetService("MaterialService").Use2022Materials = false end)
end

if theme.EnvDiffuse then pcall(function() Lighting.EnvironmentDiffuseScale = theme.EnvDiffuse end) end
if theme.EnvSpecular then pcall(function() Lighting.EnvironmentSpecularScale = theme.EnvSpecular end) end

local atm = Lighting:FindFirstChildOfClass("Atmosphere")
if atm then atm.Enabled = false end

if theme.CC then
if not ccEffect then
ccEffect = Instance.new("ColorCorrectionEffect")
ccEffect.Parent = Lighting
end
ccEffect.Brightness = theme.CC.Brightness
ccEffect.Contrast = theme.CC.Contrast
ccEffect.Saturation = theme.CC.Saturation
ccEffect.TintColor = theme.CC.Tint
ccEffect.Enabled = true
end

if theme.Bloom then
if not bloomEffect then
bloomEffect = Instance.new("BloomEffect")
bloomEffect.Parent = Lighting
end
bloomEffect.Intensity = theme.Bloom.Intensity
bloomEffect.Size = theme.Bloom.Size
bloomEffect.Threshold = theme.Bloom.Threshold
bloomEffect.Enabled = true
else
if bloomEffect then bloomEffect.Enabled = false end
end

if theme.SunRays then
if not sunRaysEffect then
sunRaysEffect = Instance.new("SunRaysEffect")
sunRaysEffect.Parent = Lighting
end
sunRaysEffect.Intensity = theme.SunRays.Intensity
sunRaysEffect.Spread = theme.SunRays.Spread
sunRaysEffect.Enabled = true
else
if sunRaysEffect then sunRaysEffect.Enabled = false end
end

if theme.DOF then
if not dofEffect then
dofEffect = Instance.new("DepthOfFieldEffect")
dofEffect.Parent = Lighting
end
dofEffect.FocusDistance = theme.DOF.FocusDistance
dofEffect.InFocusRadius = theme.DOF.InFocusRadius
dofEffect.NearIntensity = theme.DOF.NearIntensity
dofEffect.FarIntensity = theme.DOF.FarIntensity
dofEffect.Enabled = true
else
if dofEffect then dofEffect.Enabled = false end
end
if theme.CustomSkybox then
local sky = Lighting:FindFirstChildOfClass("Sky")
if not sky then
sky = Instance.new("Sky")
sky.Parent = Lighting
end
sky.SkyboxBk = theme.CustomSkybox.Bk
sky.SkyboxDn = theme.CustomSkybox.Dn
sky.SkyboxFt = theme.CustomSkybox.Ft
sky.SkyboxLf = theme.CustomSkybox.Lf
sky.SkyboxRt = theme.CustomSkybox.Rt
sky.SkyboxUp = theme.CustomSkybox.Up
sky.SunAngularSize = 14
sky.Parent = nil
sky.Parent = Lighting
else
end
if theme.Atmosphere then
local atmos = Lighting:FindFirstChildOfClass("Atmosphere")
if not atmos then
atmos = Instance.new("Atmosphere")
atmos.Parent = Lighting
end
atmos.Density = theme.Atmosphere.Density
atmos.Color = theme.Atmosphere.Color
atmos.Decay = theme.Atmosphere.Decay
atmos.Glare = theme.Atmosphere.Glare
end
if theme.ColorShift_Top then Lighting.ColorShift_Top = theme.ColorShift_Top end
if theme.ColorShift_Bottom then Lighting.ColorShift_Bottom = theme.ColorShift_Bottom end
if theme.FogColor then Lighting.FogColor = theme.FogColor end
end
local function restoreLighting()
Lighting.Brightness = originalLight.Brightness
Lighting.Ambient = originalLight.Ambient
Lighting.OutdoorAmbient = originalLight.OutdoorAmbient
if originalLight.ClockTime then
Lighting.ClockTime = originalLight.ClockTime
else
Lighting.TimeOfDay = originalLight.TimeOfDay
end
Lighting.FogEnd = originalLight.FogEnd
Lighting.FogStart = originalLight.FogStart
Lighting.ExposureCompensation = originalLight.ExposureCompensation
Lighting.GlobalShadows = true
local atm = Lighting:FindFirstChildOfClass("Atmosphere")
if atm then atm.Enabled = true end
if ccEffect then ccEffect:Destroy(); ccEffect = nil end
if bloomEffect then bloomEffect:Destroy(); bloomEffect = nil end
if sunRaysEffect then sunRaysEffect:Destroy(); sunRaysEffect = nil end
if dofEffect then dofEffect:Destroy(); dofEffect = nil end
pcall(function() game:GetService("MaterialService").Use2022Materials = false end)
if _G.MaterialsForced then
_G.MaterialsForced = false
restoreMaterials()
end
restoreExistingEffects()
currentShaderTheme = nil
end

VisualShaders:AddToggle("EnableShaders", {
Text = "Shaders",
Default = false,
Tooltip = L("Включить шейдеры (изменяет освещение игры)", "Enable shaders (changes game lighting)"),
Callback = function(Value)
if Value then
applyShaderTheme(Options.ShaderTheme.Value)
else
restoreLighting()
end
end
})

task.spawn(function()
while task.wait(0.05) do
if Toggles.EnableShaders and Toggles.EnableShaders.Value and currentShaderTheme then
local theme = ShaderThemes[currentShaderTheme]
if theme then
Lighting.Brightness = theme.Brightness
Lighting.Ambient = theme.Ambient
Lighting.OutdoorAmbient = theme.OutdoorAmbient
if theme.ClockTime then
Lighting.ClockTime = theme.ClockTime
end
if theme.ExposureCompensation then
Lighting.ExposureCompensation = theme.ExposureCompensation
end
if theme.GlobalShadows ~= nil then
Lighting.GlobalShadows = theme.GlobalShadows
else
Lighting.GlobalShadows = false
end
Lighting.FogEnd = theme.FogEnd
if theme.FogStart then Lighting.FogStart = theme.FogStart end

if theme.Use2022Materials then
pcall(function() game:GetService("MaterialService").Use2022Materials = true end)
else
pcall(function() game:GetService("MaterialService").Use2022Materials = false end)
end

if theme.ForceMaterials then
if not _G.MaterialsForced then
_G.MaterialsForced = true
applyRealMaterials()
end
else
if _G.MaterialsForced then
_G.MaterialsForced = false
restoreMaterials()
end
end

local atm = Lighting:FindFirstChildOfClass("Atmosphere")
if atm then
if Toggles.EnableCustomSky and Toggles.EnableCustomSky.Value then
atm.Enabled = true
else
atm.Enabled = false
end
end
for _, v in ipairs(Lighting:GetChildren()) do
if v ~= ccEffect and v ~= bloomEffect and v ~= sunRaysEffect then
if v:IsA("ColorCorrectionEffect") or v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") then
v.Enabled = false
end
end
end
if ccEffect and theme.CC then
ccEffect.Enabled = true
if not ccEffect.Parent then ccEffect.Parent = Lighting end
end
if bloomEffect and theme.Bloom then
bloomEffect.Enabled = true
if not bloomEffect.Parent then bloomEffect.Parent = Lighting end
end
if sunRaysEffect and theme.SunRays then
sunRaysEffect.Enabled = true
if not sunRaysEffect.Parent then sunRaysEffect.Parent = Lighting end
end
if dofEffect and theme.DOF then
dofEffect.Enabled = true
if not dofEffect.Parent then dofEffect.Parent = Lighting end
end

if theme.EnvDiffuse then pcall(function() Lighting.EnvironmentDiffuseScale = theme.EnvDiffuse end) end
if theme.EnvSpecular then pcall(function() Lighting.EnvironmentSpecularScale = theme.EnvSpecular end) end
end
end
end
end)

VisualShaders:AddDropdown("ShaderTheme", {
Text = L("Тема", "Theme"),
Default = "Night",
Values = {"Night", "Dusk", "Sunset", "Fog", "Bright", "Neon", "Cold", "Warm", "UltraRealism", "Twilight", "Luminous", "Sandstorm", "Arctic"},
Multi = false,
Tooltip = L("Выбери тему освещения", "Choose lighting theme"),
Callback = function(Value)
if Toggles.EnableShaders and Toggles.EnableShaders.Value then
applyShaderTheme(Value)
end
end
})

do
VisualAtmo = Tabs.Visual:AddLeftGroupbox(L("Атмосфера", "Atmosphere"), "cloud-sun")

atmoOriginal = {
saved = false,
ClockTime = Lighting.ClockTime,
TimeOfDay = Lighting.TimeOfDay,
FogEnd = Lighting.FogEnd,
FogStart = Lighting.FogStart,
FogColor = Lighting.FogColor,
OutdoorAmbient = Lighting.OutdoorAmbient,
ColorShift_Top = Lighting.ColorShift_Top,
ColorShift_Bottom = Lighting.ColorShift_Bottom,
}

atmoOriginalSky = {}
atmoCustomObjects = {}

local function saveAtmoOriginal()
if atmoOriginal.saved then return end
atmoOriginal.saved = true
for _, obj in ipairs(Lighting:GetChildren()) do
if obj:IsA("Sky") or obj:IsA("Atmosphere") then
atmoOriginalSky[obj] = {
parent = obj.Parent,
props = {}
}
if obj:IsA("Atmosphere") then
atmoOriginalSky[obj].props = {
Density = obj.Density,
Offset = obj.Offset,
Glare = obj.Glare,
Haze = obj.Haze,
Color = obj.Color,
Decay = obj.Decay,
Enabled = obj.Enabled,
}
end
end
end
end

SkyboxList = {
["HD"]={Bk="http://www.roblox.com/asset/?id=16553658937",Dn="http://www.roblox.com/asset/?id=16553660713",Ft="http://www.roblox.com/asset/?id=16553662144",Lf="http://www.roblox.com/asset/?id=16553664042",Rt="http://www.roblox.com/asset/?id=16553665766",Up="http://www.roblox.com/asset/?id=16553667750"},
["Black Storm"]={Bk="rbxassetid://15502511288",Dn="rbxassetid://15502508460",Ft="rbxassetid://15502510289",Lf="rbxassetid://15502507918",Rt="rbxassetid://15502509398",Up="rbxassetid://15502511911"},
["Snow"]={Bk="http://www.roblox.com/asset/?id=155657655",Dn="http://www.roblox.com/asset/?id=155674246",Ft="http://www.roblox.com/asset/?id=155657609",Lf="http://www.roblox.com/asset/?id=155657671",Rt="http://www.roblox.com/asset/?id=155657619",Up="http://www.roblox.com/asset/?id=155674931"},
["Blue Space"]={Bk="rbxassetid://15536110634",Dn="rbxassetid://15536112543",Ft="rbxassetid://15536116141",Lf="rbxassetid://15536114370",Rt="rbxassetid://15536118762",Up="rbxassetid://15536117282"},
["Realistic"]={Bk="rbxassetid://653719502",Dn="rbxassetid://653718790",Ft="rbxassetid://653719067",Lf="rbxassetid://653719190",Rt="rbxassetid://653718931",Up="rbxassetid://653719321"},
["Sunset"]={Bk="rbxassetid://600830446",Dn="rbxassetid://600831635",Ft="rbxassetid://600832720",Lf="rbxassetid://600886090",Rt="rbxassetid://600833862",Up="rbxassetid://600835177"},
["Stormy"]={Bk="http://www.roblox.com/asset/?id=18703245834",Dn="http://www.roblox.com/asset/?id=18703243349",Ft="http://www.roblox.com/asset/?id=18703240532",Lf="http://www.roblox.com/asset/?id=18703237556",Rt="http://www.roblox.com/asset/?id=18703235430",Up="http://www.roblox.com/asset/?id=18703232671"},
["Pink"]={Bk="rbxassetid://12216109205",Dn="rbxassetid://12216109875",Ft="rbxassetid://12216109489",Lf="rbxassetid://12216110170",Rt="rbxassetid://12216110471",Up="rbxassetid://12216108877"},
["Arctic"]={Bk="http://www.roblox.com/asset/?id=225469390",Dn="http://www.roblox.com/asset/?id=225469395",Ft="http://www.roblox.com/asset/?id=225469403",Lf="http://www.roblox.com/asset/?id=225469450",Rt="http://www.roblox.com/asset/?id=225469471",Up="http://www.roblox.com/asset/?id=225469481"},
["Space"]={Bk="http://www.roblox.com/asset/?id=166509999",Dn="http://www.roblox.com/asset/?id=166510057",Ft="http://www.roblox.com/asset/?id=166510116",Lf="http://www.roblox.com/asset/?id=166510092",Rt="http://www.roblox.com/asset/?id=166510131",Up="http://www.roblox.com/asset/?id=166510114"},
["Red Night"]={Bk="http://www.roblox.com/asset/?id=401664839",Dn="http://www.roblox.com/asset/?id=401664862",Ft="http://www.roblox.com/asset/?id=401664960",Lf="http://www.roblox.com/asset/?id=401664881",Rt="http://www.roblox.com/asset/?id=401664901",Up="http://www.roblox.com/asset/?id=401664936"},
["Deep Space 1"]={Bk="http://www.roblox.com/asset/?id=149397692",Dn="http://www.roblox.com/asset/?id=149397686",Ft="http://www.roblox.com/asset/?id=149397697",Lf="http://www.roblox.com/asset/?id=149397684",Rt="http://www.roblox.com/asset/?id=149397688",Up="http://www.roblox.com/asset/?id=149397702"},
["Pink Skies"]={Bk="http://www.roblox.com/asset/?id=151165214",Dn="http://www.roblox.com/asset/?id=151165197",Ft="http://www.roblox.com/asset/?id=151165224",Lf="http://www.roblox.com/asset/?id=151165191",Rt="http://www.roblox.com/asset/?id=151165206",Up="http://www.roblox.com/asset/?id=151165227"},
["Purple Sunset"]={Bk="rbxassetid://264908339",Dn="rbxassetid://264907909",Ft="rbxassetid://264909420",Lf="rbxassetid://264909758",Rt="rbxassetid://264908886",Up="rbxassetid://264907379"},
["Blue Night"]={Bk="http://www.roblox.com/asset/?id=12064107",Dn="http://www.roblox.com/asset/?id=12064152",Ft="http://www.roblox.com/asset/?id=12064121",Lf="http://www.roblox.com/asset/?id=12063984",Rt="http://www.roblox.com/asset/?id=12064115",Up="http://www.roblox.com/asset/?id=12064131"},
["Blue Nebula"]={Bk="http://www.roblox.com/asset?id=135207744",Dn="http://www.roblox.com/asset?id=135207662",Ft="http://www.roblox.com/asset?id=135207770",Lf="http://www.roblox.com/asset?id=135207615",Rt="http://www.roblox.com/asset?id=135207695",Up="http://www.roblox.com/asset?id=135207794"},
["Blue Planet"]={Bk="rbxassetid://218955819",Dn="rbxassetid://218953419",Ft="rbxassetid://218954524",Lf="rbxassetid://218958493",Rt="rbxassetid://218957134",Up="rbxassetid://218950090"},
["Deep Space 2"]={Bk="http://www.roblox.com/asset/?id=159248188",Dn="http://www.roblox.com/asset/?id=159248183",Ft="http://www.roblox.com/asset/?id=159248187",Lf="http://www.roblox.com/asset/?id=159248173",Rt="http://www.roblox.com/asset/?id=159248192",Up="http://www.roblox.com/asset/?id=159248176"},
["Summer"]={Bk="rbxassetid://16648590964",Dn="rbxassetid://16648617436",Ft="rbxassetid://16648595424",Lf="rbxassetid://16648566370",Rt="rbxassetid://16648577071",Up="rbxassetid://16648598180"},
["Galaxy"]={Bk="rbxassetid://15983968922",Dn="rbxassetid://15983966825",Ft="rbxassetid://15983965025",Lf="rbxassetid://15983967420",Rt="rbxassetid://15983966246",Up="rbxassetid://15983964246"},
["Stylized"]={Bk="rbxassetid://18351376859",Dn="rbxassetid://18351374919",Ft="rbxassetid://18351376800",Lf="rbxassetid://18351376469",Rt="rbxassetid://18351376457",Up="rbxassetid://18351377189"},
["Minecraft"]={Bk="rbxassetid://8735166756",Dn="http://www.roblox.com/asset/?id=8735166707",Ft="http://www.roblox.com/asset/?id=8735231668",Lf="http://www.roblox.com/asset/?id=8735166755",Rt="http://www.roblox.com/asset/?id=8735166751",Up="http://www.roblox.com/asset/?id=8735166729"},
["Cloudy Rain"]={Bk="http://www.roblox.com/asset/?id=4498828382",Dn="http://www.roblox.com/asset/?id=4498828812",Ft="http://www.roblox.com/asset/?id=4498829917",Lf="http://www.roblox.com/asset/?id=4498830911",Rt="http://www.roblox.com/asset/?id=4498830417",Up="http://www.roblox.com/asset/?id=4498831746"},
["Black Cloudy Rain"]={Bk="http://www.roblox.com/asset/?id=149679669",Dn="http://www.roblox.com/asset/?id=149681979",Ft="http://www.roblox.com/asset/?id=149679690",Lf="http://www.roblox.com/asset/?id=149679709",Rt="http://www.roblox.com/asset/?id=149679722",Up="http://www.roblox.com/asset/?id=149680199"},
["Blossom Daylight"]={Bk="http://www.roblox.com/asset/?id=271042516",Dn="http://www.roblox.com/asset/?id=271077243",Ft="http://www.roblox.com/asset/?id=271042556",Lf="http://www.roblox.com/asset/?id=271042310",Rt="http://www.roblox.com/asset/?id=271042467",Up="http://www.roblox.com/asset/?id=271077958"},
["Roblox Default"]={Bk="rbxasset://textures/sky/sky512_bk.tex",Dn="rbxasset://textures/sky/sky512_dn.tex",Ft="rbxasset://textures/sky/sky512_ft.tex",Lf="rbxasset://textures/sky/sky512_lf.tex",Rt="rbxasset://textures/sky/sky512_rt.tex",Up="rbxasset://textures/sky/sky512_up.tex"},
} end

ActiveSky = nil

local function applySkybox(skyName)
skyName = skyName or "HD"
local skyData = SkyboxList[skyName]
if not skyData then return end
if ActiveSky then
pcall(function() ActiveSky:Destroy() end)
ActiveSky = nil
end
for obj, data in pairs(atmoOriginalSky) do
if obj:IsA("Sky") and obj.Parent then
data.parent = obj.Parent
obj.Parent = nil
end
end
local sky = Instance.new("Sky")
sky.Name = "OblivionSky"
sky.SkyboxBk = skyData.Bk
sky.SkyboxDn = skyData.Dn
sky.SkyboxFt = skyData.Ft
sky.SkyboxLf = skyData.Lf
sky.SkyboxRt = skyData.Rt
sky.SkyboxUp = skyData.Up
sky.SunAngularSize = 0
sky.Parent = Lighting
ActiveSky = sky
end

local function disableSkybox()
if ActiveSky then
pcall(function() ActiveSky:Destroy() end)
ActiveSky = nil
end
for obj, data in pairs(atmoOriginalSky) do
if obj:IsA("Sky") and not obj.Parent then
pcall(function() obj.Parent = data.parent end)
end
end
end

function restoreAtmoOriginal()
if not atmoOriginal.saved then return end

Lighting.ClockTime = atmoOriginal.ClockTime
Lighting.TimeOfDay = atmoOriginal.TimeOfDay
Lighting.FogEnd = atmoOriginal.FogEnd
Lighting.FogStart = atmoOriginal.FogStart
Lighting.FogColor = atmoOriginal.FogColor
Lighting.OutdoorAmbient = atmoOriginal.OutdoorAmbient
Lighting.ColorShift_Top = atmoOriginal.ColorShift_Top
if atmoOriginal.ColorShift_Bottom then Lighting.ColorShift_Bottom = atmoOriginal.ColorShift_Bottom end

disableSkybox()
for _, obj in ipairs(atmoCustomObjects) do
pcall(function() obj:Destroy() end)
end
atmoCustomObjects = {}

for obj, data in pairs(atmoOriginalSky) do
if obj and obj.Parent then
if obj:IsA("Atmosphere") and data.props then
obj.Density = data.props.Density
obj.Offset = data.props.Offset
obj.Glare = data.props.Glare
obj.Haze = data.props.Haze
obj.Color = data.props.Color
obj.Decay = data.props.Decay
obj.Enabled = data.props.Enabled
end
elseif obj and not obj.Parent then
pcall(function() obj.Parent = data.parent end)
end
end
end

local function maintainAtmosphere()
if Toggles.EnableCustomTime and Toggles.EnableCustomTime.Value then
local timeVal = Options.CustomTimeValue and Options.CustomTimeValue.Value or 12
Lighting.ClockTime = timeVal
end

if Toggles.EnableCustomFog and Toggles.EnableCustomFog.Value then
local fogColor = Options.CustomFogColor and Options.CustomFogColor.Value or Color3.fromRGB(200, 200, 200)
local fogDist = Options.CustomFogDistance and Options.CustomFogDistance.Value or 1000
Lighting.FogColor = fogColor
Lighting.FogEnd = fogDist
Lighting.FogStart = 0
end

end

function applyAtmosphere()
saveAtmoOriginal()

local shadersActive = Toggles.EnableShaders and Toggles.EnableShaders.Value

if Toggles.EnableCustomTime and Toggles.EnableCustomTime.Value then
local timeVal = Options.CustomTimeValue and Options.CustomTimeValue.Value or 12
Lighting.ClockTime = timeVal
Lighting.TimeOfDay = tostring(timeVal) .. ":00:00"
else
if not shadersActive then
Lighting.ClockTime = atmoOriginal.ClockTime
Lighting.TimeOfDay = atmoOriginal.TimeOfDay
end
end

if Toggles.EnableCustomFog and Toggles.EnableCustomFog.Value then
fogColor = Options.CustomFogColor and Options.CustomFogColor.Value or Color3.fromRGB(200, 200, 200)
fogDist = Options.CustomFogDistance and Options.CustomFogDistance.Value or 1000
Lighting.FogColor = fogColor
Lighting.FogEnd = fogDist
Lighting.FogStart = 0
else
if not shadersActive then
Lighting.FogEnd = atmoOriginal.FogEnd
Lighting.FogStart = atmoOriginal.FogStart
Lighting.FogColor = atmoOriginal.FogColor
end
end

if Toggles.EnableCustomSky and Toggles.EnableCustomSky.Value then
applySkybox(Options.SkyboxType and Options.SkyboxType.Value or "HD")
else
disableSkybox()
for obj, data in pairs(atmoOriginalSky) do
if obj:IsA("Sky") and not obj.Parent then
pcall(function() obj.Parent = data.parent end)
end
end
end

if Toggles.EnableSkyboxColor and Toggles.EnableSkyboxColor.Value then
local sc = Options.SkyboxColor and Options.SkyboxColor.Value or Color3.fromRGB(128, 128, 128)
Lighting.ColorShift_Top = sc
Lighting.ColorShift_Bottom = sc
else
Lighting.ColorShift_Top = atmoOriginal.ColorShift_Top or Color3.fromRGB(0, 0, 0)
Lighting.ColorShift_Bottom = atmoOriginal.ColorShift_Bottom or Color3.fromRGB(0, 0, 0)
end
end

VisualAtmo:AddToggle("EnableCustomTime", {
Text = L("Время суток", "Time of Day"),
Default = false,
Tooltip = L("Заморозить и изменить время суток", "Freeze and change time of day"),
Callback = function(Value)
if Value then
applyAtmosphere()
else
if not (Toggles.EnableShaders and Toggles.EnableShaders.Value) then
Lighting.ClockTime = atmoOriginal.ClockTime
Lighting.TimeOfDay = atmoOriginal.TimeOfDay
end
end
end
})
VisualAtmo:AddSlider("CustomTimeValue", {
Text = L("Час", "Hour"),
Default = 12,
Min = 0,
Max = 24,
Rounding = 1,
Compact = false,
Callback = function(Value)
if Toggles.EnableCustomTime and Toggles.EnableCustomTime.Value then
applyAtmosphere()
end
end
})

VisualAtmo:AddToggle("EnableCustomSky", {
Text = "Skybox",
Default = false,
Tooltip = L("Заменяет небо на кастомный скайбокс", "Replaces sky with custom skybox"),
Callback = function(Value)
if Value then
applySkybox(Options.SkyboxType and Options.SkyboxType.Value or "HD")
else
disableSkybox()
if not (Toggles.EnableShaders and Toggles.EnableShaders.Value) then
restoreAtmoOriginal()
end
end
end
})

VisualAtmo:AddDropdown("SkyboxType", {
Text = L("Список скайбоксов", "Skybox List"),
Default = "HD",
Values = {"HD", "Black Storm", "Snow", "Blue Space", "Realistic", "Sunset", "Stormy", "Pink", "Arctic", "Space", "Red Night", "Deep Space 1", "Pink Skies", "Purple Sunset", "Blue Night", "Blue Nebula", "Blue Planet", "Deep Space 2", "Summer", "Galaxy", "Stylized", "Minecraft", "Cloudy Rain", "Black Cloudy Rain", "Blossom Daylight", "Roblox Default"},
Multi = false,
Tooltip = L("Выбери скайбокс", "Choose skybox"),
Callback = function(Value)
if Toggles.EnableCustomSky and Toggles.EnableCustomSky.Value then
applySkybox(Value)
end
end
})

VisualAtmo:AddToggle("EnableSkyboxColor", {
Text = L("Цвет скайбокса", "Skybox Color"),
Default = false,
Tooltip = L("Изменить цвет неба на кастомный через цветовую палитру", "Change sky color via color picker"),
Callback = function(Value)
if Value then
local sc = Options.SkyboxColor and Options.SkyboxColor.Value or Color3.fromRGB(128, 128, 128)
Lighting.ColorShift_Top = sc
Lighting.ColorShift_Bottom = sc
else
Lighting.ColorShift_Top = atmoOriginal.ColorShift_Top or Color3.fromRGB(0, 0, 0)
Lighting.ColorShift_Bottom = atmoOriginal.ColorShift_Bottom or Color3.fromRGB(0, 0, 0)
end
end
})

VisualAtmo:AddLabel(L("Цвет неба", "Sky Color")):AddColorPicker("SkyboxColor", {
Default = Color3.fromRGB(128, 128, 128),
Title = L("Цвет скайбокса", "Skybox Color"),
Callback = function(Value)
if Toggles.EnableSkyboxColor and Toggles.EnableSkyboxColor.Value then
Lighting.ColorShift_Top = Value
Lighting.ColorShift_Bottom = Value
end
end
})

VisualAtmo:AddToggle("EnableCustomFog", {
Text = L("Туман", "Fog"),
Default = false,
Tooltip = L("Густой туман с настраиваемым цветом", "Dense fog with custom color"),
Callback = function(Value)
applyAtmosphere()
if not Value and not (Toggles.EnableShaders and Toggles.EnableShaders.Value) then
Lighting.FogEnd = atmoOriginal.FogEnd
Lighting.FogStart = atmoOriginal.FogStart
Lighting.FogColor = atmoOriginal.FogColor
end
end
}):AddColorPicker("CustomFogColor", {
Default = Color3.fromRGB(200, 200, 200),
Title = L("Цвет тумана", "Fog Color"),
Callback = function(Value)
if Toggles.EnableCustomFog and Toggles.EnableCustomFog.Value then
applyAtmosphere()
end
end
})
VisualAtmo:AddSlider("CustomFogDistance", {
Text = L("Дальность тумана", "Fog Distance"),
Default = 1000,
Min = 50,
Max = 10000,
Rounding = 0,
Compact = false,
Callback = function(Value)
if Toggles.EnableCustomFog and Toggles.EnableCustomFog.Value then
applyAtmosphere()
end
end
})

RunService.Heartbeat:Connect(function()
local anyAtmo = (Toggles.EnableCustomTime and Toggles.EnableCustomTime.Value)
or (Toggles.EnableCustomFog and Toggles.EnableCustomFog.Value)
or (Toggles.EnableCustomSky and Toggles.EnableCustomSky.Value)
if anyAtmo then
maintainAtmosphere()
end
end)

end
do
local VisualBright = Tabs.Visual:AddLeftGroupbox(L("Освещение", "Lighting"), "sun")

VisualBright:AddToggle("EnableFullbright", {
Text = "Fullbright",
Default = false,
Tooltip = L("Всё видно в темноте — максимальная яркость", "Everything visible in darkness — max brightness"),
Callback = function(Value)
if Value then
if not originalLight._fbSaved then
originalLight._fbSaved = true
originalLight._fbBrightness = Lighting.Brightness
originalLight._fbAmbient = Lighting.Ambient
originalLight._fbOutdoorAmbient = Lighting.OutdoorAmbient
originalLight._fbClockTime = Lighting.ClockTime
originalLight._fbGlobalShadows = Lighting.GlobalShadows
originalLight._fbExposure = Lighting.ExposureCompensation
end
Lighting.Brightness = 3
Lighting.Ambient = Color3.fromRGB(255, 255, 255)
Lighting.OutdoorAmbient = Color3.fromRGB(255, 255, 255)
Lighting.ClockTime = 12
Lighting.GlobalShadows = false
Lighting.ExposureCompensation = 0.5
else
if originalLight._fbSaved then
Lighting.Brightness = originalLight._fbBrightness
Lighting.Ambient = originalLight._fbAmbient
Lighting.OutdoorAmbient = originalLight._fbOutdoorAmbient
Lighting.ClockTime = originalLight._fbClockTime
Lighting.GlobalShadows = originalLight._fbGlobalShadows
Lighting.ExposureCompensation = originalLight._fbExposure
end
end
end
})
end

do
VisualTex = Tabs.Visual:AddRightGroupbox(L("Текстуры карты", "Map Textures"), "image")

VisualTex:AddToggle("EnableGeneralTint", {
Text = L("Общее", "General"),
Default = false,
Tooltip = "Красит всё глобальное освещение и атмосферу кар��ы",
})
VisualTex:AddLabel(L("Цвет общего освещения", "General Lighting Color")):AddColorPicker("GeneralTintColor", {
Default = Color3.fromRGB(150, 200, 255),
Title = L("Общий цвет", "General Color"),
})

origPartColors = {}
recoloredMats = {
[Enum.Material.Grass] = Color3.fromRGB(75, 150, 75),
[Enum.Material.Wood] = Color3.fromRGB(130, 100, 75),
[Enum.Material.Concrete] = Color3.fromRGB(150, 150, 150),
[Enum.Material.Plastic] = Color3.fromRGB(200, 200, 200),
[Enum.Material.SmoothPlastic] = Color3.fromRGB(200, 200, 200),
}

function applyTextureColors()
if not Toggles or not Toggles.EnableTexColors then return end
if not Toggles.EnableTexColors.Value then return end

if Options.TexColorGrass then recoloredMats[Enum.Material.Grass] = Options.TexColorGrass.Value end
if Options.TexColorWood then recoloredMats[Enum.Material.Wood] = Options.TexColorWood.Value end
if Options.TexColorConcrete then recoloredMats[Enum.Material.Concrete] = Options.TexColorConcrete.Value end
if Options.TexColorPlastic then
recoloredMats[Enum.Material.Plastic] = Options.TexColorPlastic.Value
recoloredMats[Enum.Material.SmoothPlastic] = Options.TexColorPlastic.Value
end

task.spawn(function()
for _, part in ipairs(workspace:GetDescendants()) do
if part:IsA("BasePart") then
local tColor = recoloredMats[part.Material]
if tColor then
if not origPartColors[part] then
origPartColors[part] = part.Color
end
part.Color = tColor
end
end
end
pcall(function()
if Options.TexColorGrass then workspace.Terrain:SetMaterialColor(Enum.Material.Grass, Options.TexColorGrass.Value) end
if Options.TexColorWood then workspace.Terrain:SetMaterialColor(Enum.Material.Wood, Options.TexColorWood.Value) end
if Options.TexColorConcrete then workspace.Terrain:SetMaterialColor(Enum.Material.Concrete, Options.TexColorConcrete.Value) end
if Options.TexColorPlastic then workspace.Terrain:SetMaterialColor(Enum.Material.Plastic, Options.TexColorPlastic.Value) end
end)
end)
end

VisualTex:AddToggle("EnableTexColors", {
Text = L("Изменить цвета текстур", "Change Texture Colors"),
Default = false,
Tooltip = L("Перекрашивает объекты на карте по их материалу", "Recolors map objects by their material"),
Callback = function(Value)
if Value then
applyTextureColors()
else
for part, color in pairs(origPartColors) do
if part and part.Parent then
part.Color = color
end
end
origPartColors = {}
pcall(function()
workspace.Terrain:SetMaterialColor(Enum.Material.Grass, Color3.fromRGB(106, 127, 63))
workspace.Terrain:SetMaterialColor(Enum.Material.Wood, Color3.fromRGB(212, 175, 55))
workspace.Terrain:SetMaterialColor(Enum.Material.Concrete, Color3.fromRGB(127, 127, 127))
workspace.Terrain:SetMaterialColor(Enum.Material.Plastic, Color3.fromRGB(200, 200, 200))
end)
end
end
})

VisualTex:AddLabel(L("Цвет травы", "Grass Color")):AddColorPicker("TexColorGrass", {
Default = Color3.fromRGB(75, 150, 75),
Title = L("Цвет травы", "Grass Color"),
Callback = applyTextureColors
})
VisualTex:AddLabel(L("Цвет дерева", "Wood Color")):AddColorPicker("TexColorWood", {
Default = Color3.fromRGB(130, 100, 75),
Title = L("Цвет дерева", "Wood Color"),
Callback = applyTextureColors
})
VisualTex:AddLabel(L("Цвет камня/бетона", "Concrete Color")):AddColorPicker("TexColorConcrete", {
Default = Color3.fromRGB(150, 150, 150),
Title = L("Цвет камня/бетона", "Concrete Color"),
Callback = applyTextureColors
})
VisualTex:AddLabel(L("Цвет пластика", "Plastic Color")):AddColorPicker("TexColorPlastic", {
Default = Color3.fromRGB(200, 200, 200),
Title = L("Цвет пластика", "Plastic Color"),
Callback = applyTextureColors
})

origGeneralTint = {}

RunService.Heartbeat:Connect(function()
if not Toggles then return end
local lighting = game:GetService("Lighting")

if Toggles.EnableGeneralTint and Toggles.EnableGeneralTint.Value then
local cColor = Options.GeneralTintColor and Options.GeneralTintColor.Value or Color3.fromRGB(150, 200, 255)

if not origGeneralTint["OutdoorAmbient"] then
origGeneralTint["OutdoorAmbient"] = lighting.OutdoorAmbient
origGeneralTint["ColorShift_Top"] = lighting.ColorShift_Top
end
lighting.OutdoorAmbient = cColor
lighting.ColorShift_Top = cColor

local atm = lighting:FindFirstChildOfClass("Atmosphere")
if not atm then
atm = Instance.new("Atmosphere")
atm.Name = "CustomAtmosphere"
atm.Density = 0.3
atm.Offset = 0.25
atm.Parent = lighting
origGeneralTint[atm] = "Created"
elseif not origGeneralTint[atm] then
origGeneralTint[atm] = {Color = atm.Color, Decay = atm.Decay, Enabled = atm.Enabled}
end

atm.Enabled = true
atm.Color = cColor
atm.Decay = Color3.new(cColor.R * 0.5, cColor.G * 0.5, cColor.B * 0.5)
else
if origGeneralTint["OutdoorAmbient"] then
lighting.OutdoorAmbient = origGeneralTint["OutdoorAmbient"]
lighting.ColorShift_Top = origGeneralTint["ColorShift_Top"]
origGeneralTint["OutdoorAmbient"] = nil
end
for obj, data in pairs(origGeneralTint) do
if typeof(obj) == "Instance" and obj:IsA("Atmosphere") then
if data == "Created" then
obj:Destroy()
else
obj.Color = data.Color
obj.Decay = data.Decay
obj.Enabled = data.Enabled
end
end
end
origGeneralTint = {}
end
end)

do
local VisualEffects = Tabs.Visual:AddLeftGroupbox("Effects", "wand-sparkles")

VisualEffects:AddToggle("EnableBlur", {
Text = "Blur",
Default = false,
Tooltip = L("Размытие экрана", "Screen blur"),
Callback = function(Value)
if Value then
local blur = Instance.new("BlurEffect")
blur.Name = "OblivionBlur"
blur.Size = Options.BlurSize.Value
blur.Parent = game:GetService("Lighting")
else
local ex = game:GetService("Lighting"):FindFirstChild("OblivionBlur")
if ex then ex:Destroy() end
end
end
})

VisualEffects:AddSlider("BlurSize", {
Text = "Blur Size",
Default = 10,
Min = 0,
Max = 24,
Rounding = 1,
Increment = 0.5,
Tooltip = L("Сила размытия", "Blur strength"),
Callback = function(Value)
local blur = game:GetService("Lighting"):FindFirstChild("OblivionBlur")
if blur then blur.Size = Value end
end
})

local Visual3rdEffects = Tabs.Visual:AddLeftGroupbox("3rd Person Effects", "scan-eye")

Visual3rdEffects:AddToggle("Enable3rdPersonEffects", {
Text = L("Эффекты (3rd Person)", "Effects (3rd Person)"),
Default = false,
Tooltip = L("Визуальные эффекты на персонаже, видны только от 3 лица", "Visual effects on character, visible only in 3rd person"),
Callback = function(Value)
if Value then
apply3rdPersonEffect(Options.EffectType and Options.EffectType.Value or "Fire")
else
remove3rdPersonEffect()
end
end
})

Visual3rdEffects:AddDropdown("EffectType", {
Text = L("Список эффектов", "Effect List"),
Default = "Fire",
Values = {"Fire", "Sparkles", "Toxic", "Godly", "Super Sayien", "North Star", "Blue Lord", "Pink Aura", "Angel Wing", "Sweet Heart", "Ethereal Aura"},
Multi = false,
Tooltip = L("Выбери эффект для персонажа", "Choose character effect"),
Callback = function(Value)
if Toggles.Enable3rdPersonEffects and Toggles.Enable3rdPersonEffects.Value then
apply3rdPersonEffect(Value)
end
end
})

Visual3rdEffects:AddLabel(L("Цвет эффектов", "Effect Color")):AddColorPicker("EffectColor", {
Default = Color3.fromRGB(255, 100, 0),
Title = L("Цвет эффекта", "Effect Color"),
Callback = function(Value)
if Toggles.Enable3rdPersonEffects and Toggles.Enable3rdPersonEffects.Value then
apply3rdPersonEffect(Options.EffectType and Options.EffectType.Value or "Fire")
end
end
})

local current3rdEffectName = nil
local current3rdEffectParts = {}

local function clear3rdEffectParts()
for _, part in ipairs(current3rdEffectParts) do
pcall(function() part:Destroy() end)
end
current3rdEffectParts = {}
end

AuraModels = {
["Godly"] = "rbxassetid://16699750981",
["Super Sayien"] = "rbxassetid://116109508364297",
["North Star"] = "rbxassetid://83945069652732",
["Blue Lord"] = "rbxassetid://10974316799",
["Pink Aura"] = "rbxassetid://115980859615239",
["Angel Wing"] = "rbxassetid://90022969696073",
["Sweet Heart"] = "rbxassetid://91724768175470",
["Ethereal Aura"] = "rbxassetid://97041568674250",
}
currentAuraModel = nil
lastLoadedAura = nil

function loadAuraModel(effectName)
local id = AuraModels[effectName]
if not id then return false end
local ok, m = pcall(function() return game:GetObjects(id)[1] end)
if ok and m then
currentAuraModel = m
return true
end
return false
end

function enableAuraModel(char)
if not currentAuraModel then return end
local tmp = currentAuraModel:Clone()
local allowedTypes = {
ParticleEmitter = true, Fire = true, Smoke = true, Sparkles = true,
PointLight = true, SpotLight = true, SurfaceLight = true,
Beam = true, Trail = true, BillboardGui = true,
}
local ec = (Options.EffectColor and Options.EffectColor.Value) or nil
for _, o in ipairs(tmp:GetDescendants()) do
if allowedTypes[o.ClassName] then
local cl = o:Clone()
local pn = o.Parent and o.Parent.Name
local tgt = pn and char:FindFirstChild(pn) or char:FindFirstChildWhichIsA("BasePart")
if tgt and not tgt:FindFirstChild(cl.Name) then
cl.Parent = tgt
if ec then
pcall(function()
if cl:IsA("ParticleEmitter") then
cl.Color = ColorSequence.new(ec)
elseif cl:IsA("Fire") then
cl.Color = ec
cl.SecondaryColor = ec
elseif cl:IsA("Smoke") then
cl.Color = ec
elseif cl:IsA("Sparkles") then
cl.SparkleColor = ec
elseif cl:IsA("PointLight") or cl:IsA("SpotLight") or cl:IsA("SurfaceLight") then
cl.Color = ec
elseif cl:IsA("Beam") then
cl.Color = ColorSequence.new(ec)
elseif cl:IsA("Trail") then
cl.Color = ColorSequence.new(ec)
end
end)
end
table.insert(current3rdEffectParts, cl)
end
end
end
tmp:Destroy()
end

function apply3rdPersonEffect(effectName)
remove3rdPersonEffect()
local char = LocalPlayer.Character
if not char then return end
local hrp = char:FindFirstChild("HumanoidRootPart")
local torso = char:FindFirstChild("Torso") or char:FindFirstChild("UpperTorso")
if not hrp or not torso then return end

current3rdEffectName = effectName

if effectName == "Fire" then
local ec = (Options.EffectColor and Options.EffectColor.Value) or Color3.fromRGB(255, 100, 0)
local fire = Instance.new("Fire")
fire.Name = "Oblivion3rdEffect"
fire.Size = 5
fire.Color = ec
fire.SecondaryColor = ec
fire.Heat = 5
fire.Parent = torso
table.insert(current3rdEffectParts, fire)
local fire2 = Instance.new("Fire")
fire2.Name = "Oblivion3rdEffect2"
fire2.Size = 3
fire2.Color = ec
fire2.SecondaryColor = ec
fire2.Heat = 3
fire2.Parent = hrp
table.insert(current3rdEffectParts, fire2)
local pl = Instance.new("PointLight")
pl.Name = "Oblivion3rdEffectLight"
pl.Color = ec
pl.Range = 12
pl.Brightness = 3
pl.Parent = hrp
table.insert(current3rdEffectParts, pl)

elseif effectName == "Sparkles" then
local ec = (Options.EffectColor and Options.EffectColor.Value) or Color3.fromRGB(255, 255, 255)
local sparkles = Instance.new("Sparkles")
sparkles.Name = "Oblivion3rdEffect"
sparkles.SparkleColor = ec
sparkles.Parent = torso
table.insert(current3rdEffectParts, sparkles)
local att = Instance.new("Attachment")
att.Name = "Oblivion3rdEffectAtt"
att.Parent = hrp
local pe = Instance.new("ParticleEmitter")
pe.Texture = "rbxassetid://243660364"
pe.Color = ColorSequence.new(ec)
pe.Size = NumberSequence.new({NumberSequenceKeypoint.new(0, 0.5, 0), NumberSequenceKeypoint.new(1, 1, 0)})
pe.Transparency = NumberSequence.new({NumberSequenceKeypoint.new(0, 0, 0), NumberSequenceKeypoint.new(1, 1, 0)})
pe.Lifetime = NumberRange.new(1)
pe.Rate = 30
pe.Speed = NumberRange.new(1, 3)
pe.Parent = att
table.insert(current3rdEffectParts, att)
local pl = Instance.new("PointLight")
pl.Name = "Oblivion3rdEffectLight"
pl.Color = ec
pl.Range = 10
pl.Brightness = 2
pl.Parent = hrp
table.insert(current3rdEffectParts, pl)

elseif effectName == "Toxic" then
local ec = (Options.EffectColor and Options.EffectColor.Value) or Color3.fromRGB(50, 255, 0)
local smoke = Instance.new("Smoke")
smoke.Name = "Oblivion3rdEffect"
smoke.Color = ec
smoke.Size = 5
smoke.RiseVelocity = 2
smoke.Opacity = 0.6
smoke.Parent = torso
table.insert(current3rdEffectParts, smoke)
local smoke2 = Instance.new("Smoke")
smoke2.Name = "Oblivion3rdEffect2"
smoke2.Color = ec
smoke2.Size = 3
smoke2.RiseVelocity = 1
smoke2.Opacity = 0.5
smoke2.Parent = hrp
table.insert(current3rdEffectParts, smoke2)
local pl = Instance.new("PointLight")
pl.Name = "Oblivion3rdEffectLight"
pl.Color = ec
pl.Range = 12
pl.Brightness = 3
pl.Parent = hrp
table.insert(current3rdEffectParts, pl)

elseif AuraModels[effectName] then
if lastLoadedAura ~= effectName then
if currentAuraModel then pcall(function() currentAuraModel:Destroy() end) end
currentAuraModel = nil
if loadAuraModel(effectName) then
lastLoadedAura = effectName
end
end
if currentAuraModel then
enableAuraModel(char)
end
end
end

function remove3rdPersonEffect()
clear3rdEffectParts()
current3rdEffectName = nil
local char = LocalPlayer.Character
if char then
for _, child in ipairs(char:GetDescendants()) do
if child.Name == "Oblivion3rdEffect" or child.Name == "Oblivion3rdEffect2" or child.Name == "Oblivion3rdEffectAtt" or child.Name == "Oblivion3rdEffectLight" then
pcall(function() child:Destroy() end)
end
end
end
end

LocalPlayer.CharacterAdded:Connect(function()
task.wait(1)
if Toggles.Enable3rdPersonEffects and Toggles.Enable3rdPersonEffects.Value then
apply3rdPersonEffect(Options.EffectType and Options.EffectType.Value or "Fire")
end
end)

local VisualCoin = Tabs.Visual:AddLeftGroupbox("Coin", "circle-dollar-sign")

VisualCoin:AddButton({
Text = "Set Coins",
Tooltip = L("Введите число и нажмите Enter (как в venom X)", "Enter a number and press Enter (like in venom X)"),
Func = function()
local sg = Instance.new("ScreenGui")
sg.Name = "CoinInputGui"
sg.ResetOnSpawn = false
sg.IgnoreGuiInset = true
sg.DisplayOrder = 9999
sg.Parent = (gethui and gethui()) or game:GetService("CoreGui")

local overlay = Instance.new("Frame")
overlay.Size = UDim2.new(1, 0, 1, 0)
overlay.BackgroundColor3 = Color3.new(0, 0, 0)
overlay.BackgroundTransparency = 0.5
overlay.BorderSizePixel = 0
overlay.Parent = sg

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0, 340, 0, 180)
frame.Position = UDim2.new(0.5, -170, 0.5, -90)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
frame.BorderSizePixel = 0
frame.Parent = sg

local fc = Instance.new("UICorner")
fc.CornerRadius = UDim.new(0, 12)
fc.Parent = frame

local fs = Instance.new("UIStroke")
fs.Color = Color3.fromRGB(100, 80, 200)
fs.Thickness = 2
fs.Transparency = 0.2
fs.Parent = frame

local fg = Instance.new("UIGradient")
fg.Rotation = 90
fg.Color = ColorSequence.new(Color3.fromRGB(35, 35, 50), Color3.fromRGB(20, 20, 30))
fg.Parent = frame

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 40)
titleBar.BackgroundColor3 = Color3.fromRGB(100, 80, 200)
titleBar.BorderSizePixel = 0
titleBar.Parent = frame

local tbc = Instance.new("UICorner")
tbc.CornerRadius = UDim.new(0, 12)
tbc.Parent = titleBar

local tbg = Instance.new("UIGradient")
tbg.Rotation = 90
tbg.Color = ColorSequence.new(Color3.fromRGB(120, 100, 220), Color3.fromRGB(80, 60, 180))
tbg.Parent = titleBar

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -40, 1, 0)
title.Position = UDim2.new(0, 15, 0, 0)
title.BackgroundTransparency = 1
title.Text = L("💰 Монеты", "💰 Coins")
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.Font = Enum.Font.SourceSansBold
title.TextSize = 18
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = titleBar

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -35, 0, 5)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.SourceSansBold
closeBtn.TextSize = 16
closeBtn.Text = "X"
closeBtn.BorderSizePixel = 0
closeBtn.Parent = titleBar

local cbc = Instance.new("UICorner")
cbc.CornerRadius = UDim.new(0, 6)
cbc.Parent = closeBtn

closeBtn.MouseButton1Click:Connect(function() sg:Destroy() end)

local hint = Instance.new("TextLabel")
hint.Size = UDim2.new(1, -30, 0, 20)
hint.Position = UDim2.new(0, 15, 0, 48)
hint.BackgroundTransparency = 1
hint.Text = L("Введите количество монет:", "Enter coin amount:")
hint.TextColor3 = Color3.fromRGB(180, 180, 200)
hint.Font = Enum.Font.SourceSans
hint.TextSize = 14
hint.TextXAlignment = Enum.TextXAlignment.Left
hint.Parent = frame

local box = Instance.new("TextBox")
box.Size = UDim2.new(1, -30, 0, 40)
box.Position = UDim2.new(0, 15, 0, 75)
box.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
box.TextColor3 = Color3.fromRGB(255, 255, 255)
box.Font = Enum.Font.SourceSansBold
box.TextSize = 20
box.PlaceholderText = L("Например: 999999", "Example: 999999")
box.PlaceholderColor3 = Color3.fromRGB(120, 120, 140)
box.Text = ""
box.ClearTextOnFocus = false
box.BorderSizePixel = 0
box.Parent = frame

local bxc = Instance.new("UICorner")
bxc.CornerRadius = UDim.new(0, 8)
bxc.Parent = box

local bxs = Instance.new("UIStroke")
bxs.Color = Color3.fromRGB(100, 80, 200)
bxs.Thickness = 1.5
bxs.Transparency = 0.3
bxs.Parent = box

box:CaptureFocus()

local btn = Instance.new("TextButton")
btn.Size = UDim2.new(1, -30, 0, 38)
btn.Position = UDim2.new(0, 15, 0, 128)
btn.BackgroundColor3 = Color3.fromRGB(0, 170, 80)
btn.TextColor3 = Color3.fromRGB(255, 255, 255)
btn.Font = Enum.Font.SourceSansBold
btn.TextSize = 16
btn.Text = L("✓ Применить", "✓ Apply")
btn.BorderSizePixel = 0
btn.Parent = frame

local bnc = Instance.new("UICorner")
bnc.CornerRadius = UDim.new(0, 8)
bnc.Parent = btn

local bng = Instance.new("UIGradient")
bng.Rotation = 90
bng.Color = ColorSequence.new(Color3.fromRGB(0, 200, 100), Color3.fromRGB(0, 140, 60))
bng.Parent = btn

local function applyCoins()
local amt = tonumber(box.Text) or 0
pcall(function() LocalPlayer.PlayerGui.MenuGui.TopRight.CoinsFrame.CoinsDisplay.Coins.Text = tostring(amt) end)
Library:Notify("Монеты: " .. tostring(amt), 2)
sg:Destroy()
end

btn.MouseButton1Click:Connect(applyCoins)
box.FocusLost:Connect(function(ep) if ep then applyCoins() end end)
overlay.MouseButton1Click:Connect(function() sg:Destroy() end)
end
})

VisualESP = Tabs.Visual:AddRightGroupbox("ESP", "eye")

espColor = Color3.fromRGB(0, 255, 0)
guiParent = (gethui and gethui()) or game:GetService("CoreGui")
playerGui = LocalPlayer:FindFirstChild("PlayerGui") or LocalPlayer:WaitForChild("PlayerGui", 10)

espObjects = {}

VisualESP:AddToggle("EnableESP", {
Text = "ESP",
Default = false,
Tooltip = "Включить/��ыключить ESP",
}):AddColorPicker("ESPColor", {
Default = Color3.fromRGB(0, 255, 0),
Title = "Цвет ESP",
Callback = function(Value)
espColor = Value
for _, obj in pairs(espObjects) do
if obj.hl then obj.hl.FillColor = Value; obj.hl.OutlineColor = Value end
if obj.box then obj.box.Color3 = Value end
if obj.name then obj.name.TextColor3 = Value end
if obj.dist then obj.dist.TextColor3 = Value end
if obj.drawing then obj.drawing.Color = Value end
end
end,
})

VisualESP:AddDropdown("ESPMode", {
Text = L("Режим", "Mode"),
Default = "Outline",
Values = {"Outline", "Fill", "Box"},
Multi = false,
Tooltip = L("Режим отображения ESP (PCLD ESP теперь отдельным тумблером ниже)", "ESP display mode (PCLD ESP is a separate toggle below)"),
})

pcldColor = Color3.fromRGB(0, 255, 255)
pcldEspConns = {}
pcldEspAdorns = {}

local function pcldEspAdd(obj)
if obj.Name ~= "PlayerCharacterLocationDetector" then return end
if obj:FindFirstChild("PCLD_ESP") then return end
local adorn = Instance.new("BoxHandleAdornment")
adorn.Name = "PCLD_ESP"
adorn.Size = obj.Size
adorn.AlwaysOnTop = true
adorn.ZIndex = 10
adorn.Color3 = pcldColor
adorn.Transparency = 0.25
adorn.Adornee = obj
adorn.Parent = obj
pcldEspAdorns[obj] = adorn
end

local function pcldEspRemove(obj)
if obj.Name ~= "PlayerCharacterLocationDetector" then return end
local esp = obj:FindFirstChild("PCLD_ESP")
if esp then esp:Destroy() end
pcldEspAdorns[obj] = nil
end

local function pcldEspRemoveAll()
for _, v in ipairs(workspace:GetChildren()) do
pcldEspRemove(v)
end
for _, v in ipairs(workspace:GetDescendants()) do
if v.Name == "PlayerCharacterLocationDetector" and v:FindFirstChild("PCLD_ESP") then
v.PCLD_ESP:Destroy()
end
end
pcldEspAdorns = {}
end

local function pcldEspAddAll()
for _, v in ipairs(workspace:GetDescendants()) do
if v.Name == "PlayerCharacterLocationDetector" then
pcldEspAdd(v)
end
end
end

VisualESP:AddToggle("EnablePCLDESP", {
Text = "PCLD ESP",
Default = false,
Tooltip = L("BoxHandleAdornment на каждом PlayerCharacterLocationDetector (как в NoName)", "BoxHandleAdornment on each PlayerCharacterLocationDetector (like NoName)"),
Callback = function(Value)
if Value then
pcldEspAddAll()
local conn = workspace.ChildAdded:Connect(function(obj)
pcldEspAdd(obj)
local sub = obj.DescendantAdded:Connect(function(d)
pcldEspAdd(d)
end)
table.insert(pcldEspConns, sub)
end)
table.insert(pcldEspConns, conn)
else
for _, c in ipairs(pcldEspConns) do
pcall(function() c:Disconnect() end)
end
pcldEspConns = {}
pcldEspRemoveAll()
end
end,
}):AddColorPicker("PCLDESPColor", {
Default = Color3.fromRGB(0, 255, 255),
Title = "Цвет PCLD ESP",
Callback = function(Value)
pcldColor = Value
for obj, adorn in pairs(pcldEspAdorns) do
if adorn and adorn.Parent then
adorn.Color3 = Value
end
end
for _, v in ipairs(workspace:GetDescendants()) do
if v.Name == "PlayerCharacterLocationDetector" and v:FindFirstChild("PCLD_ESP") then
v.PCLD_ESP.Color3 = Value
end
end
end,
})

updateESP = nil

VisualESP:AddDropdown("ESPElements", {
Text = L("Элементы", "Elements"),
Default = {},
Values = {"Name", "Distance", "Icon"},
Multi = true,
Tooltip = "Сними все галочки — останутся только боксы/контур. Или на��ми кнопку ниже.",
Callback = function(Value)
end,
})

VisualESP:AddButton({
Text = L("Сбросить элементы", "Reset Elements"),
Tooltip = L("Снимает ВСЕ элементы ESP (Ник, Дистанция, Иконка)", "Removes ALL ESP elements (Name, Distance, Icon)"),
Func = function()
if Options.ESPElements and Options.ESPElements.SetValue then
pcall(function() Options.ESPElements:SetValue({}) end)
end
end,
})

VisualESP:AddSlider("ESPSize", {
Text = L("Размер", "Size"),
Default = 12,
Min = 6,
Max = 30,
Rounding = 0,
Tooltip = L("Размер текста и иконок", "Text and icon size"),
})

antiKickEspConns = {}
antiKickEspItems = {}
antiKickEspColor = Color3.fromRGB(255, 0, 255)
antiKickEspActive = false

local stickyNames = {
["NinjaShuriken"] = true,
["NinjaKatana"] = true,
["NinjaKunai"] = true,
["ToolPencil"] = true,
["ToolDiggingForkRusty"] = true,
["ToolCleaver"] = true,
["ToolPickaxe"] = true,
}

local function antiKickEspClear()
for model, data in pairs(antiKickEspItems) do
if data.hl and data.hl.Parent then data.hl:Destroy() end
if data.bb and data.bb.Parent then data.bb:Destroy() end
end
antiKickEspItems = {}
end

local function antiKickEspStop()
antiKickEspActive = false
for _, c in ipairs(antiKickEspConns) do
pcall(function() c:Disconnect() end)
end
antiKickEspConns = {}
antiKickEspClear()
end

local function antiKickEspFindAllStickies()
local found = {}
for _, obj in ipairs(workspace:GetDescendants()) do
if stickyNames[obj.Name] and obj:IsA("Model") and obj:FindFirstChild("StickyPart") then
table.insert(found, obj)
end
end
return found
end

local function antiKickEspScan()
if not antiKickEspActive then return end
local stickies = antiKickEspFindAllStickies()
local taggedModels = {}
for model, _ in pairs(antiKickEspItems) do
taggedModels[model] = true
end
local newTagged = {}
for _, plr in ipairs(game:GetService("Players"):GetPlayers()) do
if plr == LocalPlayer then continue end
local char = plr.Character
if not char then continue end
local hrp = char:FindFirstChild("HumanoidRootPart")
if not hrp then continue end
local hum = char:FindFirstChild("Humanoid")
if not hum or hum.Health <= 0 then continue end
for _, sticky in ipairs(stickies) do
if newTagged[sticky] then continue end
local sp = sticky:FindFirstChild("StickyPart")
if not sp then continue end
local attached = false
for _, v in ipairs(sticky:GetDescendants()) do
if v:IsA("WeldConstraint") then
local p0, p1 = v.Part0, v.Part1
if (p0 and p0:IsDescendantOf(char)) or (p1 and p1:IsDescendantOf(char)) then
attached = true
break
end
end
end
if not attached then
local d = (hrp.Position - sp.Position).Magnitude
if d < 12 then attached = true end
end
if attached then
newTagged[sticky] = true
if not antiKickEspItems[sticky] then
local hl = Instance.new("Highlight")
hl.Name = "AKESP"
hl.FillColor = antiKickEspColor
hl.OutlineColor = antiKickEspColor
hl.FillTransparency = 0.5
hl.OutlineTransparency = 0
hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
hl.Parent = sticky
local bb = Instance.new("BillboardGui")
bb.Name = "AKESP"
bb.Size = UDim2.new(0, 100, 0, 20)
bb.StudsOffset = Vector3.new(0, 1.5, 0)
bb.AlwaysOnTop = true
bb.LightInfluence = 0
bb.MaxDistance = 300
bb.Parent = sp or sticky
local lbl = Instance.new("TextLabel")
lbl.Size = UDim2.new(1, 0, 1, 0)
lbl.BackgroundTransparency = 0.4
lbl.BackgroundColor3 = Color3.new(0, 0, 0)
lbl.TextColor3 = antiKickEspColor
lbl.Font = Enum.Font.SourceSansBold
lbl.TextSize = 12
lbl.TextStrokeTransparency = 0.3
lbl.Text = plr.Name
lbl.Parent = bb
antiKickEspItems[sticky] = {hl = hl, bb = bb}
end
end
end
end
for model, data in pairs(antiKickEspItems) do
if not newTagged[model] or not model.Parent then
if data.hl and data.hl.Parent then data.hl:Destroy() end
if data.bb and data.bb.Parent then data.bb:Destroy() end
antiKickEspItems[model] = nil
end
end
end

VisualESP:AddToggle("EnableAntiKickESP", {
Text = L("Анти Кик ESP", "Anti Kick ESP"),
Default = false,
Tooltip = L("Подсвечивает липкие предметы (сюрикены, кунаи) на игроках", "Highlights sticky items (shurikens, kunai) on players"),
Callback = function(val)
if val then
antiKickEspActive = true
antiKickEspScan()
task.spawn(function()
while antiKickEspActive do
task.wait(2)
pcall(antiKickEspScan)
end
end)
Library:Notify(L("Анти Кик ESP ON", "Anti Kick ESP ON"), 2)
else
antiKickEspStop()
Library:Notify(L("Анти Кик ESP OFF", "Anti Kick ESP OFF"), 2)
end
end
}):AddColorPicker("AntiKickESPColor", {
Default = Color3.fromRGB(255, 0, 255),
Title = L("Цвет Анти Кик ESP", "Anti Kick ESP Color"),
Callback = function(val)
antiKickEspColor = val
for _, data in pairs(antiKickEspItems) do
if data.hl and data.hl.Parent then
data.hl.FillColor = val
data.hl.OutlineColor = val
end
if data.bb and data.bb.Parent then
local lbl = data.bb:FindFirstChildOfClass("TextLabel")
if lbl then lbl.TextColor3 = val end
end
end
end
})

function createESP(plr)
if espObjects[plr] then return end
local hl = Instance.new("Highlight")
hl.FillColor = espColor
hl.OutlineColor = espColor
hl.FillTransparency = 1
hl.OutlineTransparency = 1
hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
hl.Enabled = false
hl.Parent = workspace

local box = Instance.new("SelectionBox")
box.Color3 = espColor
box.Transparency = 0
box.Adornee = nil 
box.LineThickness = 0.05
box.Parent = workspace

local boxPart = Instance.new("Part")
boxPart.Name = "ESPBox"
boxPart.Size = Vector3.new(4, 6, 2)
boxPart.Transparency = 1
boxPart.CanCollide = false
boxPart.Anchored = true
boxPart.Parent = workspace

local bb = Instance.new("BillboardGui")
bb.Size = UDim2.new(0, 200, 0, 80)
bb.StudsOffset = Vector3.new(0, 3, 0)
bb.AlwaysOnTop = true
bb.LightInfluence = 0
bb.MaxDistance = 600
bb.Enabled = false
bb.ResetOnSpawn = false 
bb.Parent = guiParent 

local nLabel = Instance.new("TextLabel")
nLabel.Size = UDim2.new(1, 0, 0, 20)
nLabel.BackgroundTransparency = 1
nLabel.Text = ""
nLabel.TextColor3 = espColor
nLabel.Font = Enum.Font.SourceSansBold
nLabel.TextSize = 14
nLabel.TextStrokeTransparency = 0.5
nLabel.Parent = bb

local dLabel = Instance.new("TextLabel")
dLabel.Size = UDim2.new(1, 0, 0, 16)
dLabel.Position = UDim2.new(0, 0, 0, 20)
dLabel.BackgroundTransparency = 1
dLabel.Text = ""
dLabel.TextColor3 = espColor
dLabel.Font = Enum.Font.SourceSans
dLabel.TextSize = 12
dLabel.TextStrokeTransparency = 0.5
dLabel.Parent = bb

local iLabel = Instance.new("ImageLabel")
iLabel.Size = UDim2.new(0, 40, 0, 40)
iLabel.Position = UDim2.new(0.5, -20, 0, 40)
iLabel.BackgroundTransparency = 1
iLabel.Image = ""
iLabel.Parent = bb

espObjects[plr] = {hl = hl, box = box, boxPart = boxPart, bb = bb, name = nLabel, dist = dLabel, icon = iLabel}

local draw = Drawing.new("Square")
draw.Thickness = 1.5
draw.Filled = false
draw.Color = espColor
draw.Visible = false
espObjects[plr].drawing = draw
end

function removeESP(plr)
local obj = espObjects[plr]
if not obj then return end
if obj.hl then obj.hl:Destroy() end
if obj.box then obj.box:Destroy() end
if obj.boxPart then obj.boxPart:Destroy() end
if obj.bb then obj.bb:Destroy() end
if obj.drawing then obj.drawing:Remove() end
espObjects[plr] = nil
end

function hasElement(el, name)
if type(el) ~= "table" then return false end
if el[name] == true then return true end
for _, v in pairs(el) do
if v == name then return true end
end
return false
end

updateESP = function(plr)
local obj = espObjects[plr]
if not obj then return end

local function hideElements()
if obj.bb then
obj.bb.Adornee = nil
obj.bb.Enabled = false
end
if obj.name then obj.name.Visible = false end
if obj.dist then obj.dist.Visible = false end
if obj.icon then obj.icon.Visible = false end
end

local function hideVisuals()
if obj.hl then obj.hl.Enabled = false; obj.hl.Adornee = nil end
if obj.box then obj.box.Adornee = nil end
if obj.drawing then obj.drawing.Visible = false end
end

if not (Toggles.EnableESP and Toggles.EnableESP.Value) then
hideVisuals()
hideElements()
return
end
local char = plr.Character
if not char then
hideVisuals()
hideElements()
return
end
local root = char:FindFirstChild("HumanoidRootPart")
local hum = char:FindFirstChild("Humanoid")
local head = char:FindFirstChild("Head")
if not root or not hum or hum.Health <= 0 then
hideVisuals()
hideElements()
return
end

obj.hl.FillColor = espColor
obj.hl.OutlineColor = espColor
obj.name.TextColor3 = espColor
obj.dist.TextColor3 = espColor
obj.box.Color3 = espColor

local mode = Options.ESPMode and Options.ESPMode.Value or "Конт��р"
if mode == "Outline" then
obj.hl.FillTransparency = 1
obj.hl.OutlineTransparency = 0
obj.hl.Adornee = char
obj.hl.Enabled = true
obj.box.Adornee = nil
elseif mode == "Fill" then
obj.hl.FillTransparency = 0.5
obj.hl.OutlineTransparency = 0.2
obj.hl.Adornee = char
obj.hl.Enabled = true
obj.box.Adornee = nil
elseif mode == "Box" then
obj.hl.Enabled = false; obj.hl.Adornee = nil
obj.box.Adornee = nil
end
if mode ~= "Box" and obj.drawing then
obj.drawing.Visible = false
end

local elements = Options.ESPElements and Options.ESPElements.Value or {}
local showN = hasElement(elements, "Name")
local showD = hasElement(elements, "Distance")
local showI = hasElement(elements, "Icon")
local anyElement = showN or showD or showI

if not anyElement then
hideElements()
return 
end

obj.bb.Adornee = head or root
obj.bb.Enabled = true

local ts = Options.ESPSize and Options.ESPSize.Value or 12
obj.name.TextSize = ts
obj.dist.TextSize = math.max(ts - 2, 6)
obj.icon.Size = UDim2.new(0, ts * 3, 0, ts * 3)
obj.icon.Position = UDim2.new(0.5, -ts * 1.5, 0, ts + 20)

obj.name.Visible = showN
obj.name.Text = showN and plr.DisplayName or ""

if showD then
local cam = workspace.CurrentCamera
if cam and root then
obj.dist.Text = string.format("%.0f studs", (root.Position - cam.CFrame.Position).Magnitude)
obj.dist.Visible = true
end
else
obj.dist.Visible = false
end

obj.icon.Visible = showI
if showI and obj.icon.Image == "" then
obj.icon.Image = string.format("rbxthumb://type=AvatarHeadShot&id=%d&w=150&h=150", plr.UserId)
end
end

task.spawn(function()
while task.wait(0.1) do
if Toggles.EnableESP and Toggles.EnableESP.Value then
for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer then
local obj = espObjects[plr]
if obj and (not obj.bb.Parent or not obj.hl.Parent) then
removeESP(plr)
obj = nil
end
if not obj then createESP(plr) end
updateESP(plr)
end
end
end
end
end)

Players.PlayerRemoving:Connect(removeESP)

Toggles.EnableESP:OnChanged(function()
if not Toggles.EnableESP.Value then
for _, obj in pairs(espObjects) do
obj.hl.Enabled = false
obj.box.Adornee = nil
obj.bb.Enabled = false
if obj.drawing then obj.drawing.Visible = false end
end
end
end)

RunService.RenderStepped:Connect(function()
if not (Toggles.EnableESP and Toggles.EnableESP.Value) then return end
if not (Options.ESPMode and Options.ESPMode.Value == "Box") then return end
local cam = workspace.CurrentCamera
if not cam then return end
for plr, obj in pairs(espObjects) do
if plr ~= LocalPlayer and obj.drawing then
local char = plr.Character
if not char then obj.drawing.Visible = false; continue end
local root = char:FindFirstChild("HumanoidRootPart")
local hum = char:FindFirstChild("Humanoid")
local head = char:FindFirstChild("Head")
if not root or not hum or hum.Health <= 0 then
obj.drawing.Visible = false
continue
end
local topWorld = (head and head.Position or root.Position) + Vector3.new(0, 0.5, 0)
local botWorld = root.Position - Vector3.new(0, 3, 0)
local screenTop, onTop = cam:WorldToViewportPoint(topWorld)
local screenBot, onBot = cam:WorldToViewportPoint(botWorld)
if onTop and onBot then
local height = math.abs(screenTop.Y - screenBot.Y)
local width = height * 0.5
width = math.max(width, 15)
height = math.max(height, 25)
local centerX = (screenTop.X + screenBot.X) / 2
obj.drawing.Size = Vector2.new(width, height)
obj.drawing.Position = Vector2.new(centerX - width / 2, screenTop.Y)
obj.drawing.Color = espColor
obj.drawing.Thickness = 1.5
obj.drawing.Visible = true
else
obj.drawing.Visible = false
end
end
end
end)

end

do
local CBHGroup = Tabs.Visual:AddRightGroupbox("Custom Black Hole", "circle-dot")

local CBH = {}
CBH.RS = game:GetService("ReplicatedStorage")
CBH.WS = game:GetService("Workspace")
CBH.RunService = game:GetService("RunService")
CBH.Players = game:GetService("Players")
CBH.LocalPlayer = CBH.Players.LocalPlayer

CBH.settings = {
colorMode = "Default",       
neonGlow = false,
silent = false,             
rainbow = false,            
hideBillboard = false,      
beamWidth0 = 1,
beamWidth1 = 1,
beamTransparency = 0,        
billboardSize = 10,          
realistic = false,          
customColor = Color3.fromRGB(255, 0, 255),
customTexture = "",
}

CBH.rainbowConn = nil
CBH.watcherConn = nil

CBH.palette = {
["Default"]   = { hole = Color3.fromRGB(0,   0,   0),   beam = Color3.fromRGB(170, 0, 255),  gui = Color3.fromRGB(150, 0, 255) },
["White"]     = { hole = Color3.fromRGB(255, 255, 255), beam = Color3.fromRGB(255, 255, 255),gui = Color3.fromRGB(255, 255, 255) },
["Red"]       = { hole = Color3.fromRGB(180,  0,   0),   beam = Color3.fromRGB(255, 50, 50),  gui = Color3.fromRGB(200, 30, 30) },
["Blue"]      = { hole = Color3.fromRGB(0,   50, 180),   beam = Color3.fromRGB(50, 120, 255), gui = Color3.fromRGB(30,  80, 220) },
["Green"]     = { hole = Color3.fromRGB(0,  120,  30),   beam = Color3.fromRGB(50, 255, 100), gui = Color3.fromRGB(20, 180, 60) },
["Gold"]      = { hole = Color3.fromRGB(180,140,   0),   beam = Color3.fromRGB(255, 220, 50), gui = Color3.fromRGB(220, 180, 20) },
["Cyan"]      = { hole = Color3.fromRGB(0,  180, 200),   beam = Color3.fromRGB(50, 230, 255), gui = Color3.fromRGB(0,  200, 230) },
["Pink"]      = { hole = Color3.fromRGB(220, 50, 180),   beam = Color3.fromRGB(255, 100, 220),gui = Color3.fromRGB(230, 60, 200) },
}

CBH.colorList = {}
for k, _ in pairs(CBH.palette) do table.insert(CBH.colorList, k) end
table.sort(CBH.colorList, function(a, b)
if a == "Default" then return true end
if b == "Default" then return false end
return a < b
end)

CBH.applyToModel = function(model)
if not model then return end
local hole = model:FindFirstChild("Hole")
if not hole then return end

hole.Material = CBH.settings.neonGlow and Enum.Material.Neon or Enum.Material.Plastic

if not CBH.settings.rainbow then
local ct = CBH.palette[CBH.settings.colorMode]
if ct then
if CBH.settings.colorMode == "Custom" then
ct = {
hole = CBH.settings.customColor,
beam = CBH.settings.customColor,
gui = CBH.settings.customColor,
}
end
hole.Color = ct.hole
local beam = hole:FindFirstChild("Attachment") and hole.Attachment:FindFirstChild("Beam")
if beam then beam.Color = ColorSequence.new(ct.beam) end
local gui = hole:FindFirstChild("BillboardGui")
if gui then
if gui:FindFirstChild("Large") then gui.Large.ImageColor3 = ct.gui end
if gui:FindFirstChild("Small") then gui.Small.ImageColor3 = ct.gui end
end
end
end

if CBH.settings.customTexture ~= "" then
local tex = CBH.settings.customTexture
local gui = hole:FindFirstChild("BillboardGui")
if gui then
if gui:FindFirstChild("Large") then gui.Large.Image = tex end
if gui:FindFirstChild("Small") then gui.Small.Image = tex end
end
for _, child in ipairs(hole:GetChildren()) do
if child:IsA("Texture") or child:IsA("Decal") then
child.Texture = tex
end
end
end

local beam = hole:FindFirstChild("Attachment") and hole.Attachment:FindFirstChild("Beam")
if beam then
beam.Width0 = CBH.settings.beamWidth0
beam.Width1 = CBH.settings.beamWidth1
beam.Transparency = NumberSequence.new(CBH.settings.beamTransparency / 100)
end

local gui = hole:FindFirstChild("BillboardGui")
if gui then
gui.Size = UDim2.new(CBH.settings.billboardSize, 0, CBH.settings.billboardSize, 0)
gui.Enabled = not CBH.settings.hideBillboard
end

local drone  = hole:FindFirstChild("Drone")
local scream = hole:FindFirstChild("Scream")
if drone  then drone.Volume  = CBH.settings.silent and 0 or 1 end
if scream then scream.Volume = CBH.settings.silent and 0 or 1 end
end

CBH.applyCurrent = function()
CBH.applyToModel(CBH.WS:FindFirstChild("BlackHoleKick"))
end

CBH.applyRealistic = function(model)
if not model then return end
local hole = model:FindFirstChild("Hole")
if not hole then return end

local success, realisticModel = pcall(function()
return game:GetObjects("rbxassetid://16797584940")[1]
end)
if not success or not realisticModel then return end

for _, obj in ipairs(realisticModel:GetDescendants()) do
if obj:IsA("ParticleEmitter") or obj:IsA("Beam") or obj:IsA("Sound") or obj:IsA("Trail") then
local clone = obj:Clone()
clone.Parent = hole
end
if obj:IsA("BillboardGui") then
local curGui = hole:FindFirstChild("BillboardGui")
if curGui then
local newGui = obj:Clone()
newGui.Parent = hole
if curGui:FindFirstChild("Large") and newGui:FindFirstChild("Large") then
curGui.Large.Image = newGui.Large.Image
end
if curGui:FindFirstChild("Small") and newGui:FindFirstChild("Small") then
curGui.Small.Image = newGui.Small.Image
end
newGui:Destroy()
end
end
end

realisticModel:Destroy()
end

CBH.setupWatcher = function()
if CBH.watcherConn then CBH.watcherConn:Disconnect() CBH.watcherConn = nil end
CBH.watcherConn = CBH.WS.ChildAdded:Connect(function(child)
if child.Name == "BlackHoleKick" then
task.wait(0.1)
CBH.applyToModel(child)
if CBH.settings.realistic then
CBH.applyRealistic(child)
end
end
end)
end

CBH.setupWatcher()

CBHGroup:AddDropdown("CBHColorMode", {
Text = L("Цвет дыры", "Hole Color"),
Values = CBH.colorList,
Default = "Default",
Multi = false,
Tooltip = L("Цвет Hole + Beam + Billboard иконки", "Hole + Beam + Billboard icon color"),
Callback = function(Value)
CBH.settings.colorMode = Value
if CBH.settings.rainbow then
CBH.settings.rainbow = false
if Toggles.CBHRainbow then Toggles.CBHRainbow:SetValue(false) end
if CBH.rainbowConn then CBH.rainbowConn:Disconnect() CBH.rainbowConn = nil end
end
CBH.applyCurrent()
end,
})

CBHGroup:AddLabel("Custom Color"):AddColorPicker("CBHCustomColorPicker", {
Default = Color3.fromRGB(255, 0, 255),
Title = L("Цвет Custom Black Hole", "Custom Black Hole Color"),
Callback = function(Value)
CBH.settings.customColor = Value
if CBH.settings.colorMode == "Custom" then
CBH.applyCurrent()
end
end
})

CBHGroup:AddInput("CBHCustomTexture", {
Text = "Custom Texture (URL/ID)",
Default = "",
Tooltip = L("Вставь ссылку на картинку (https://...) или rbxassetid://... Применяется к Hole + Billboard", "Paste image URL (https://...) or rbxassetid://... Applied to Hole + Billboard"),
Placeholder = "https://... или rbxassetid://...",
Callback = function(val)
CBH.settings.customTexture = tostring(val)
CBH.applyCurrent()
end,
})

CBHGroup:AddToggle("CBHNeonGlow", {
Text = "Neon Glow",
Default = false,
Tooltip = L("Дыра светится (Material = Neon)", "Hole glows (Material = Neon)"),
Callback = function(Value)
CBH.settings.neonGlow = Value
CBH.applyCurrent()
end,
})

CBHGroup:AddToggle("CBHRainbow", {
Text = "Rainbow Mode",
Default = false,
Tooltip = L("Дыра переливается цветами радуги (по Hue)", "Hole shimmers with rainbow colors (by Hue)"),
Callback = function(Value)
CBH.settings.rainbow = Value
if CBH.rainbowConn then CBH.rainbowConn:Disconnect() CBH.rainbowConn = nil end
if Value then
local hue = 0
CBH.rainbowConn = CBH.RunService.Heartbeat:Connect(function(dt)
hue = (hue + dt * 0.3) % 1
local c = Color3.fromHSV(hue, 1, 1)
local model = CBH.WS:FindFirstChild("BlackHoleKick")
if not model then return end
local hole = model:FindFirstChild("Hole")
if not hole then return end
hole.Color = c
local beam = hole:FindFirstChild("Attachment") and hole.Attachment:FindFirstChild("Beam")
if beam then beam.Color = ColorSequence.new(c) end
local gui = hole:FindFirstChild("BillboardGui")
if gui then
if gui:FindFirstChild("Large") then gui.Large.ImageColor3 = c end
if gui:FindFirstChild("Small") then gui.Small.ImageColor3 = c end
end
end)
else
CBH.applyCurrent()
end
end,
})

CBHGroup:AddToggle("CBHRealistic", {
Text = "Realistic Black Hole",
Default = false,
Tooltip = L("Загружает реалистичную модель с частицами/звуками/трейлами", "Loads realistic model with particles/sounds/trails"),
Callback = function(Value)
CBH.settings.realistic = Value
if Value then
local current = CBH.WS:FindFirstChild("BlackHoleKick")
if current then
CBH.applyRealistic(current)
end
end
end,
})

CBHGroup:AddDivider()

CBHGroup:AddToggle("CBHHideBillboard", {
Text = "Hide Billboard",
Default = false,
Tooltip = L("Прячет иконку-картинку (остаётся только Hole + Beam)", "Hides icon image (only Hole + Beam remain)"),
Callback = function(Value)
CBH.settings.hideBillboard = Value
CBH.applyCurrent()
end,
})

CBHGroup:AddToggle("CBHSilent", {
Text = L("Silent (без звука)", "Silent (no sound)"),
Default = false,
Tooltip = L("Убирает звуки Drone + Scream", "Removes Drone + Scream sounds"),
Callback = function(Value)
CBH.settings.silent = Value
CBH.applyCurrent()
end,
})

CBHGroup:AddDivider()

CBHGroup:AddSlider("CBHBeamW0", {
Text = L("Beam ширина (внутренняя)", "Beam Width (inner)"),
Default = 1,
Min = 0,
Max = 20,
Rounding = 1,
Compact = false,
Tooltip = L("Толщина луча у центра Hole", "Beam thickness at Hole center"),
Callback = function(Value)
CBH.settings.beamWidth0 = Value
local model = CBH.WS:FindFirstChild("BlackHoleKick")
if model then
local hole = model:FindFirstChild("Hole")
local beam = hole and hole:FindFirstChild("Attachment") and hole.Attachment:FindFirstChild("Beam")
if beam then beam.Width0 = Value end
end
end,
})

CBHGroup:AddSlider("CBHBeamW1", {
Text = L("Beam ширина (внешняя)", "Beam Width (outer)"),
Default = 1,
Min = 0,
Max = 20,
Rounding = 1,
Compact = false,
Tooltip = L("Толщина луча у края Hole", "Beam thickness at Hole edge"),
Callback = function(Value)
CBH.settings.beamWidth1 = Value
local model = CBH.WS:FindFirstChild("BlackHoleKick")
if model then
local hole = model:FindFirstChild("Hole")
local beam = hole and hole:FindFirstChild("Attachment") and hole.Attachment:FindFirstChild("Beam")
if beam then beam.Width1 = Value end
end
end,
})

CBHGroup:AddSlider("CBHBeamTransparency", {
Text = L("Beam прозрачность (%)", "Beam Transparency (%)"),
Default = 0,
Min = 0,
Max = 100,
Rounding = 0,
Compact = false,
Tooltip = L("0 = полностью непрозрачный, 100 = невидимый", "0 = fully opaque, 100 = invisible"),
Callback = function(Value)
CBH.settings.beamTransparency = Value
local model = CBH.WS:FindFirstChild("BlackHoleKick")
if model then
local hole = model:FindFirstChild("Hole")
local beam = hole and hole:FindFirstChild("Attachment") and hole.Attachment:FindFirstChild("Beam")
if beam then
beam.Transparency = NumberSequence.new(Value / 100)
end
end
end,
})

CBHGroup:AddSlider("CBHBillboardSize", {
Text = L("Billboard размер", "Billboard Size"),
Default = 10,
Min = 2,
Max = 40,
Rounding = 0,
Compact = false,
Tooltip = L("Размер иконки-картинки над Hole (studs)", "Icon image size above Hole (studs)"),
Callback = function(Value)
CBH.settings.billboardSize = Value
local model = CBH.WS:FindFirstChild("BlackHoleKick")
if model then
local hole = model:FindFirstChild("Hole")
local gui  = hole and hole:FindFirstChild("BillboardGui")
if gui then gui.Size = UDim2.new(Value, 0, Value, 0) end
end
end,
})

CBHGroup:AddButton({
Text = L("Применить к текущей дыре", "Apply to current hole"),
Tooltip = L("Применяет все настройки к workspace.BlackHoleKick если она существует", "Applies all settings to workspace.BlackHoleKick if it exists"),
Func = function()
CBH.applyCurrent()
Library:Notify(L("Custom Black Hole: настройки применены", "Custom Black Hole: settings applied"), 3)
end,
})

CBHGroup:AddButton({
Text = L("Сброс к Default", "Reset to Default"),
Tooltip = L("Сбрасывает все настройки к стандартным", "Resets all settings to default"),
Func = function()
CBH.settings.colorMode = "Default"
CBH.settings.neonGlow = false
CBH.settings.silent = false
CBH.settings.hideBillboard = false
CBH.settings.beamWidth0 = 1
CBH.settings.beamWidth1 = 1
CBH.settings.beamTransparency = 0
CBH.settings.billboardSize = 10

if CBH.settings.rainbow then
CBH.settings.rainbow = false
if Toggles.CBHRainbow then Toggles.CBHRainbow:SetValue(false) end
if CBH.rainbowConn then CBH.rainbowConn:Disconnect() CBH.rainbowConn = nil end
end

if Options.CBHColorMode then Options.CBHColorMode:SetValue("Default") end
if Toggles.CBHNeonGlow then Toggles.CBHNeonGlow:SetValue(false) end
if Toggles.CBHHideBillboard then Toggles.CBHHideBillboard:SetValue(false) end
if Toggles.CBHSilent then Toggles.CBHSilent:SetValue(false) end
if Options.CBHBeamW0 then Options.CBHBeamW0:SetValue(1) end
if Options.CBHBeamW1 then Options.CBHBeamW1:SetValue(1) end
if Options.CBHBeamTransparency then Options.CBHBeamTransparency:SetValue(0) end
if Options.CBHBillboardSize then Options.CBHBillboardSize:SetValue(10) end
CBH.settings.customColor = Color3.fromRGB(255, 0, 255)
CBH.settings.customTexture = ""
if Options.CBHCustomTexture then Options.CBHCustomTexture:SetValue("") end

CBH.applyCurrent()
Library:Notify(L("Custom Black Hole: сброс к Default", "Custom Black Hole: reset to Default"), 3)
end,
})
end

end
local VisualWeather = Tabs.Visual:AddLeftGroupbox(L("Погода", "Weather"), "cloud-rain")

VisualWeather:AddToggle("EnableWeather", {
Text = L("Погода", "Weather"),
Default = false,
Tooltip = L("Включить красивые погодные эффекты", "Enable weather effects"),
})

VisualWeather:AddDropdown("WeatherType", {
Text = L("Тип погоды", "Weather Type"),
Default = "Snow",
Values = {"Snow", "Rain", "Rainbow", "Thunderstorm", "Fairy Dust"},
Multi = false,
Tooltip = "Выбери ��ффект",
})

VisualWeather:AddSlider("WeatherAmount", {
Text = L("Количество", "Amount"),
Default = 1,
Min = 0.2,
Max = 3,
Rounding = 1,
Compact = false,
Tooltip = "Сила снега, дождя �� грозы",
})

VisualWeather:AddSlider("WeatherSpeed", {
Text = L("Скорость", "Speed"),
Default = 1,
Min = 0.2,
Max = 3,
Rounding = 1,
Compact = false,
Tooltip = L("Скорость падения снега, дождя и грозы", "Snow, rain and storm fall speed"),
})

VisualWeather:AddLabel(L("Цвет всего", "Overall Color")):AddColorPicker("WeatherColor", {
Default = Color3.fromRGB(255, 255, 255),
Title = "Цвет всей погод��",
})

VisualWeather:AddButton({
Text = L("Сброс цвета", "Reset Color"),
Tooltip = "Вернуть обычные цв��та снега, дождя, радуги и грозы",
Func = function()
if Options.WeatherColor and Options.WeatherColor.SetValue then
pcall(function() Options.WeatherColor:SetValue(Color3.fromRGB(255, 255, 255)) end)
end
local rb = workspace:FindFirstChild("RainbowFixed")
if rb then rb:Destroy() end
end,
})

task.spawn(function()
local weatherPart = nil
local weatherEmitter = nil
local weatherLight = nil
local weatherObjects = {}
local weatherDrops = {}
local weatherAnchorPos = nil
local lastDropSpawn = 0
local lastLightning = 0
local lastRainbowColorKey = nil

local function addWeatherObject(obj)
weatherObjects[#weatherObjects + 1] = obj
return obj
end

local function clearDrops()
for i = #weatherDrops, 1, -1 do
pcall(function() weatherDrops[i].part:Destroy() end)
weatherDrops[i] = nil
end
end

local function clearWeatherVisuals(keepPart)
for _, obj in ipairs(weatherObjects) do
pcall(function() obj:Destroy() end)
end
weatherObjects = {}
clearDrops()
if weatherEmitter then weatherEmitter.Enabled = false end
if weatherLight then weatherLight.Enabled = false end
if not keepPart and weatherPart then
weatherPart:Destroy()
weatherPart = nil
weatherEmitter = nil
weatherLight = nil
end
end

local function getHRP()
local char = LocalPlayer.Character
return char and char:FindFirstChild("HumanoidRootPart")
end

local function getWeatherAmount()
return math.clamp((Options.WeatherAmount and Options.WeatherAmount.Value) or 1, 0.2, 3)
end

local function getWeatherSpeed()
return math.clamp((Options.WeatherSpeed and Options.WeatherSpeed.Value) or 1, 0.2, 3)
end

local function getWeatherColor()
return (Options.WeatherColor and Options.WeatherColor.Value) or Color3.fromRGB(255, 255, 255)
end

local function isCustomWeatherColor()
local c = getWeatherColor()
return not (c.R > 0.98 and c.G > 0.98 and c.B > 0.98)
end

local function weatherColor(defaultColor)
if isCustomWeatherColor() then
return getWeatherColor()
end
return defaultColor
end

local function getGroundY(pos, fallbackY)
local params = RaycastParams.new()
params.FilterType = Enum.RaycastFilterType.Blacklist
params.FilterDescendantsInstances = {LocalPlayer.Character, weatherPart}
local result = workspace:Raycast(pos + Vector3.new(0, 35, 0), Vector3.new(0, -500, 0), params)
if result then
return result.Position.Y + 0.35
end
return fallbackY or (pos.Y - 90)
end

local function makePart(name, parent, size, cframe, color, transparency)
local p = Instance.new("Part")
p.Name = name
p.Anchored = true
p.CanCollide = false
p.CanQuery = false
p.CanTouch = false
p.Material = Enum.Material.Neon
p.Color = color
p.Transparency = transparency or 0
p.Size = size
p.CFrame = cframe
p.Parent = parent
return addWeatherObject(p)
end

local function spawnDrop(kind, basePos, speedMul)
speedMul = speedMul or getWeatherSpeed()
local maxDrops = kind == "snow" and math.floor(420 * getWeatherAmount()) or math.floor(260 * getWeatherAmount())
if #weatherDrops >= maxDrops then return end
local p = Instance.new("Part")
p.Anchored = true
p.CanCollide = false
p.CanQuery = false
p.CanTouch = false
p.Material = Enum.Material.Neon
if kind == "snow" then
p.Name = "WeatherSnowflake"
p.Shape = Enum.PartType.Ball
p.Color = weatherColor(Color3.fromRGB(255, 255, 255))
p.Transparency = 0.03
local sz = math.random(7, 14) / 10
local spawnPos = basePos + Vector3.new(math.random(-190,190), math.random(55,95), math.random(-190,190))
p.Size = Vector3.new(sz, sz, sz)
p.CFrame = CFrame.new(spawnPos)
weatherDrops[#weatherDrops + 1] = {
part = p,
vel = Vector3.new(math.random(-4,4)/10, -math.random(80,140)/10 * speedMul, math.random(-4,4)/10),
life = 16,
groundLife = 1.8,
groundY = getGroundY(spawnPos, basePos.Y - 4),
kind = kind,
phase = math.random()*10,
landed = false,
}
elseif kind == "rain" then
p.Name = "WeatherRainDrop"
p.Color = weatherColor(Color3.fromRGB(120, 175, 255))
p.Transparency = 0.02
p.Size = Vector3.new(0.08, math.random(38, 62) / 10, 0.08)
p.CFrame = CFrame.new(basePos + Vector3.new(math.random(-210,210), math.random(70,115), math.random(-210,210)))
weatherDrops[#weatherDrops + 1] = {part = p, vel = Vector3.new(math.random(-8,8)/10, -math.random(120,170) * speedMul, math.random(-8,8)/10), life = 1.8 / speedMul, kind = kind, phase = 0}
elseif kind == "fairy" then
p.Name = "WeatherFairy"
p.Shape = Enum.PartType.Ball
p.Color = weatherColor(Color3.fromRGB(220, 180, 255))
p.Transparency = 0.1
local sz = math.random(5, 12) / 100
local spawnPos = basePos + Vector3.new(math.random(-120,120), math.random(0,20), math.random(-120,120))
p.Size = Vector3.new(sz, sz, sz)
p.CFrame = CFrame.new(spawnPos)
weatherDrops[#weatherDrops + 1] = {
part = p,
vel = Vector3.new(math.random(-6,6)/10, math.random(15,45)/10 * speedMul, math.random(-6,6)/10),
life = 3,
kind = kind,
phase = math.random()*10,
landed = false,
}
end
p.Parent = workspace
end

local function updateDrops(dt, basePos, kind)
for i = #weatherDrops, 1, -1 do
local d = weatherDrops[i]
if not d.part or not d.part.Parent or d.kind ~= kind then
if d.part then pcall(function() d.part:Destroy() end) end
table.remove(weatherDrops, i)
else
d.life = d.life - dt
if kind == "snow" then
if d.landed then
d.groundLife = d.groundLife - dt
d.part.Transparency = math.clamp(1 - (d.groundLife / 1.8), 0.03, 1)
if d.groundLife <= 0 then
pcall(function() d.part:Destroy() end)
table.remove(weatherDrops, i)
end
else
local sway = Vector3.new(math.sin(tick()*1.1 + d.phase) * 0.45, 0, math.cos(tick()*1.0 + d.phase) * 0.45)
d.part.CFrame = d.part.CFrame * CFrame.Angles(0, math.rad(18*dt), math.rad(12*dt))
d.part.CFrame = d.part.CFrame + ((d.vel + sway) * dt)
if d.part.Position.Y <= d.groundY then
d.landed = true
d.part.CFrame = CFrame.new(d.part.Position.X, d.groundY, d.part.Position.Z)
d.part.Transparency = 0.1
elseif d.life <= 0 then
pcall(function() d.part:Destroy() end)
table.remove(weatherDrops, i)
end
end
elseif kind == "rain" then
d.part.CFrame = d.part.CFrame + (d.vel * dt)
local pos = d.part.Position
if d.life <= 0 or pos.Y < basePos.Y - 15 or (Vector3.new(pos.X, basePos.Y, pos.Z) - Vector3.new(basePos.X, basePos.Y, basePos.Z)).Magnitude > 300 then
pcall(function() d.part:Destroy() end)
table.remove(weatherDrops, i)
end
elseif kind == "fairy" then
local t = tick()
local sparkle = Vector3.new(math.sin(t*5 + d.phase)*3, 0, math.cos(t*4 + d.phase)*3)
d.part.CFrame = d.part.CFrame + ((d.vel + sparkle) * dt)
d.part.Transparency = 0.1 + math.sin(t * 6 + d.phase) * 0.3
if d.life <= 0 then
pcall(function() d.part:Destroy() end)
table.remove(weatherDrops, i)
end
end
end
end
end

function makeLightning(origin)
local boltColor = weatherColor(Color3.fromRGB(215, 235, 255))
local lastPos = origin + Vector3.new(math.random(-30,30), 380, math.random(-30,30))
local segments = math.random(8, 12)
for i = 1, segments do
local nextPos = origin + Vector3.new(math.random(-70, 70), 380 - (i * (380 / segments)), math.random(-70, 70))
local mid = (lastPos + nextPos) / 2
local len = (lastPos - nextPos).Magnitude
local part = makePart("WeatherLightning", workspace, Vector3.new(7, 7, len + 2), CFrame.lookAt(mid, nextPos), boltColor, 0)
task.delay(0.22, function() if part then part:Destroy() end end)
lastPos = nextPos
end
if math.random(1, 2) == 1 then
local branchStart = origin + Vector3.new(math.random(-80,80), 260, math.random(-80,80))
local branchEnd = branchStart + Vector3.new(math.random(-160,160), -math.random(80,150), math.random(-160,160))
local mid = (branchStart + branchEnd) / 2
local len = (branchStart - branchEnd).Magnitude
local branch = makePart("WeatherLightningBranch", workspace, Vector3.new(5, 5, len + 2), CFrame.lookAt(mid, branchEnd), boltColor, 0)
task.delay(0.22, function() if branch then branch:Destroy() end end)
end
end

function makeRainbowSegment(parent, p1, p2, color, thick)
local mid = (p1 + p2) / 2
local len = (p1 - p2).Magnitude
local seg = Instance.new("Part")
seg.Name = "RainbowSegment"
seg.Anchored = true
seg.CanCollide = false
seg.CanQuery = false
seg.CanTouch = false
seg.Material = Enum.Material.Neon
seg.Color = color
seg.Transparency = 0.08
seg.Size = Vector3.new(thick, thick, len + 3)
seg.CFrame = CFrame.lookAt(mid, p2)
seg.Parent = parent
end

function ensureRainbow(hrp)
local c = getWeatherColor()
local colorKey = tostring(math.floor(c.R*255))..":"..tostring(math.floor(c.G*255))..":"..tostring(math.floor(c.B*255))
local oldRainbow = workspace:FindFirstChild("RainbowFixed")
if oldRainbow and lastRainbowColorKey == colorKey then return end
if oldRainbow then oldRainbow:Destroy() end
lastRainbowColorKey = colorKey
clearWeatherVisuals(true)
local model = Instance.new("Model")
model.Name = "RainbowFixed"
model.Parent = workspace
addWeatherObject(model)

weatherAnchorPos = weatherAnchorPos or (hrp.Position + Vector3.new(0, 0, -1150))
local center = weatherAnchorPos + Vector3.new(0, 80, 0)
local colors = {
weatherColor(Color3.fromRGB(255, 0, 0)), weatherColor(Color3.fromRGB(255, 120, 0)), weatherColor(Color3.fromRGB(255, 255, 0)),
weatherColor(Color3.fromRGB(0, 255, 0)), weatherColor(Color3.fromRGB(0, 120, 255)), weatherColor(Color3.fromRGB(100, 0, 200)), weatherColor(Color3.fromRGB(180, 0, 255)),
}
local radiusBase = 620
local steps = 120
for band, color in ipairs(colors) do
local radius = radiusBase - (band * 16)
local last = nil
for i = 0, steps do
local a = math.rad(180 - (i * 180 / steps))
local pos = center + Vector3.new(math.cos(a) * radius, math.sin(a) * radius, 0)
if last then makeRainbowSegment(model, last, pos, color, 14) end
last = pos
end
end
end

function ensureWeatherPart()
if weatherPart then return end
weatherPart = Instance.new("Part")
weatherPart.Name = "OblivionWeatherPart"
weatherPart.Transparency = 1
weatherPart.CanCollide = false
weatherPart.CanQuery = false
weatherPart.CanTouch = false
weatherPart.Anchored = true
weatherPart.Size = Vector3.new(460, 1, 460)
weatherPart.Parent = workspace
weatherEmitter = Instance.new("ParticleEmitter")
weatherEmitter.Parent = weatherPart
weatherEmitter.EmissionDirection = Enum.NormalId.Bottom
weatherLight = Instance.new("PointLight")
weatherLight.Parent = weatherPart
weatherLight.Enabled = false
end

function updateWeather()
if not (Toggles.EnableWeather and Toggles.EnableWeather.Value) then
clearWeatherVisuals(false)
weatherAnchorPos = nil
return
end

local hrp = getHRP()
if not hrp then return end
ensureWeatherPart()
local wType = Options.WeatherType and Options.WeatherType.Value or "Snow"
local now = tick()
local amountMul = getWeatherAmount()
local speedMul = getWeatherSpeed()
weatherPart.CFrame = CFrame.new(hrp.Position + Vector3.new(0, 85, 0))

if wType ~= "Rainbow" and workspace:FindFirstChild("RainbowFixed") then clearWeatherVisuals(true) end

if wType == "Snow" then
weatherEmitter.Enabled = false
weatherEmitter.Texture = ""
weatherEmitter.Acceleration = Vector3.new(0, -6, 0)
weatherEmitter.Drag = 12
weatherEmitter.Rate = 0
weatherEmitter.Lifetime = NumberRange.new(9, 14)
weatherEmitter.Speed = NumberRange.new(0, 0)
weatherEmitter.Rotation = NumberRange.new(0, 360)
weatherEmitter.RotSpeed = NumberRange.new(-55, 55)
weatherEmitter.SpreadAngle = Vector2.new(70, 70)
weatherEmitter.Size = NumberSequence.new({NumberSequenceKeypoint.new(0, 1.7), NumberSequenceKeypoint.new(1, 2.6)})
weatherEmitter.Transparency = NumberSequence.new(0.02)
weatherEmitter.Color = ColorSequence.new(weatherColor(Color3.fromRGB(255,255,255)))
weatherLight.Enabled = false
updateDrops(1/60, hrp.Position, "snow")
if now - lastDropSpawn > math.max(0.035, 0.075 / amountMul) then
lastDropSpawn = now
for i = 1, math.max(1, math.floor(4 * amountMul)) do spawnDrop("snow", hrp.Position, speedMul) end
end
elseif wType == "Rain" then
weatherEmitter.Enabled = true
weatherEmitter.Texture = "rbxassetid://182285145"
weatherEmitter.Acceleration = Vector3.new(0, -320 * speedMul, 0)
weatherEmitter.Drag = 0
weatherEmitter.Rate = 12000 * amountMul
weatherEmitter.Lifetime = NumberRange.new(0.9 / speedMul, 1.4 / speedMul)
weatherEmitter.Speed = NumberRange.new(120 * speedMul, 180 * speedMul)
weatherEmitter.SpreadAngle = Vector2.new(18, 18)
weatherEmitter.Rotation = NumberRange.new(0, 0)
weatherEmitter.RotSpeed = NumberRange.new(0, 0)
weatherEmitter.Size = NumberSequence.new(3.2)
weatherEmitter.Transparency = NumberSequence.new(0.02)
weatherEmitter.Color = ColorSequence.new(weatherColor(Color3.fromRGB(95, 160, 255)))
weatherLight.Enabled = false
updateDrops(1/60, hrp.Position, "rain")
if now - lastDropSpawn > math.max(0.008, 0.02 / amountMul) then
lastDropSpawn = now
for i = 1, math.max(1, math.floor(12 * amountMul)) do spawnDrop("rain", hrp.Position, speedMul) end
end
elseif wType == "Rainbow" then
clearDrops()
weatherEmitter.Enabled = false
weatherLight.Enabled = false
ensureRainbow(hrp)
elseif wType == "Thunderstorm" then
weatherEmitter.Enabled = true
weatherEmitter.Texture = "rbxassetid://182285145"
weatherEmitter.Acceleration = Vector3.new(0, -360 * speedMul, 0)
weatherEmitter.Drag = 0
weatherEmitter.Rate = 15000 * amountMul
weatherEmitter.Lifetime = NumberRange.new(0.8 / speedMul, 1.2 / speedMul)
weatherEmitter.Speed = NumberRange.new(150 * speedMul, 210 * speedMul)
weatherEmitter.SpreadAngle = Vector2.new(20, 20)
weatherEmitter.Rotation = NumberRange.new(0, 0)
weatherEmitter.RotSpeed = NumberRange.new(0, 0)
weatherEmitter.Size = NumberSequence.new(3.5)
weatherEmitter.Transparency = NumberSequence.new(0.01)
weatherEmitter.Color = ColorSequence.new(weatherColor(Color3.fromRGB(80, 130, 230)))
updateDrops(1/60, hrp.Position, "rain")
if now - lastDropSpawn > math.max(0.006, 0.016 / amountMul) then
lastDropSpawn = now
for i = 1, math.max(1, math.floor(16 * amountMul)) do spawnDrop("rain", hrp.Position, speedMul) end
end
if now - lastLightning > (math.random(15, 35) / 10) / amountMul then
lastLightning = now
for i = 1, math.random(math.max(1, math.floor(2 * amountMul)), math.max(2, math.floor(4 * amountMul))) do
local side = math.random(1, 2) == 1 and -1 or 1
local origin = hrp.Position + Vector3.new(math.random(-1000,1000), 0, side * math.random(900,1500))
makeLightning(origin)
end
end
elseif wType == "Fairy Dust" then
weatherEmitter.Enabled = false
weatherLight.Enabled = true
weatherLight.Color = weatherColor(Color3.fromRGB(220, 180, 255))
weatherLight.Brightness = 2 * amountMul
weatherLight.Range = 25 * amountMul
updateDrops(1/60, hrp.Position, "fairy")
if now - lastDropSpawn > math.max(0.015, 0.03 / amountMul) then
lastDropSpawn = now
for i = 1, math.max(1, math.floor(4 * amountMul)) do spawnDrop("fairy", hrp.Position, speedMul) end
end
end
end

RunService.Heartbeat:Connect(updateWeather)
end)

do
local VisualWater = Tabs.Visual:AddRightGroupbox("World", "earth")

local waterSavedData = {}

VisualWater:AddToggle("EnableRealisticWater", {
Text = "Realistic Water",
Default = false,
Tooltip = L("Заменяет плоскую текстуру океана за картой на НАСТОЯЩУЮ воду, в которой можно плавать.", "Replaces flat ocean texture with REAL water you can swim in."),
Callback = function(Value)
pcall(function()
local terrain = workspace.Terrain
local model = workspace:FindFirstChild("Map")
model = model and model:FindFirstChild("AlwaysHereTweenedObjects")
model = model and model:FindFirstChild("Ocean")
model = model and model:FindFirstChild("Object")
model = model and model:FindFirstChild("ObjectModel")

if model then
if Value then
for _, part in ipairs(model:GetChildren()) do
if part.Name == "Ocean" and part:IsA("BasePart") then
if not waterSavedData[part] then
local cf = part.CFrame
local size = part.Size
local region = Region3.new(
cf.Position - (size / 2),
cf.Position + (size / 2) - Vector3.new(0, 1, 0)
):ExpandToGrid(4)

waterSavedData[part] = {
region = region,
transparency = part.Transparency,
cancollide = part.CanCollide
}
end

local data = waterSavedData[part]
part.Transparency = 1
part.CanCollide = false
terrain:FillRegion(data.region, 4, Enum.Material.Water)
end
end
else
for part, data in pairs(waterSavedData) do
if part and part.Parent then
part.Transparency = data.transparency
if Toggles.EnableWaterWalk and Toggles.EnableWaterWalk.Value then
part.CanCollide = true
else
part.CanCollide = data.cancollide
end
end
terrain:FillRegion(data.region, 4, Enum.Material.Air)
end
end
end
end)
end
})

VisualWater:AddSlider("WaterWaveSize", {
Text = L("Размер волн", "Wave Size"),
Default = 0.15,
Min = 0,
Max = 3,
Rounding = 2,
Compact = false,
Tooltip = L("0 - штиль (гладкая вода), 3 - гигантские волны", "0 - calm (smooth water), 3 - giant waves"),
Callback = function(Value)
pcall(function()
workspace.Terrain.WaterWaveSize = Value
workspace.Terrain.WaterWaveSpeed = Value * 20
end)
end
})

end
VisualTracers = Tabs.Visual:AddRightGroupbox("Tracer Lines", "route")

tracerColor = Color3.fromRGB(255, 0, 0)
tracerObjects = {}

VisualTracers:AddToggle("EnableTracers", {
Text = "Tracer Lines",
Default = false,
Tooltip = L("Линии от экрана к игрокам", "Lines from screen to players"),
}):AddColorPicker("TracerColor", {
Default = Color3.fromRGB(255, 0, 0),
Title = L("Цвет линий", "Line Color"),
Callback = function(Value)
tracerColor = Value
for _, obj in pairs(tracerObjects) do
if obj.line then obj.line.Color = tracerColor end
end
end,
})

VisualTracers:AddSlider("TracerWidth", {
Text = L("Толщина линии", "Line Thickness"),
Default = 1,
Min = 1,
Max = 5,
Rounding = 0,
Tooltip = L("Ширина tracer линии", "Tracer line width"),
})

VisualTracers:AddDropdown("TracerOrigin", {
Text = L("Откуда вести", "Origin Point"),
Default = "Screen Center",
Values = {"Top Center", "Screen Center", "Bottom Center"},
Multi = false,
Tooltip = L("Точка откуда ведутся линии", "Origin point for lines"),
})

VisualTracers:AddToggle("TracerThroughWalls", {
Text = L("Показывать за спиной", "Show Behind Camera"),
Default = false,
Tooltip = L("Показывать линии к игрокам даже если они за камерой", "Show lines to players even if behind camera"),
})

function getTracerOrigin()
local cam = workspace.CurrentCamera
if not cam then return Vector2.new(0, 0) end
local vp = cam.ViewportSize
local origin = Options.TracerOrigin and Options.TracerOrigin.Value or "Screen Center"
if origin == "Top Center" then
return Vector2.new(vp.X / 2, 0)
elseif origin == "Bottom Center" then
return Vector2.new(vp.X / 2, vp.Y)
else
return Vector2.new(vp.X / 2, vp.Y / 2)
end
end

function createTracer(plr)
if tracerObjects[plr] then return end
local line = Drawing.new("Line")
line.Thickness = 1
line.Color = tracerColor
line.Visible = false
tracerObjects[plr] = {line = line}
end

function removeTracer(plr)
local obj = tracerObjects[plr]
if not obj then return end
if obj.line then obj.line:Remove() end
tracerObjects[plr] = nil
end

function updateTracer(plr)
local obj = tracerObjects[plr]
if not obj then return end
if not (Toggles.EnableTracers and Toggles.EnableTracers.Value) then
obj.line.Visible = false
return
end
local char = plr.Character
if not char then obj.line.Visible = false; return end
local root = char:FindFirstChild("HumanoidRootPart")
local hum = char:FindFirstChild("Humanoid")
if not root or not hum or hum.Health <= 0 then
obj.line.Visible = false
return
end
local cam = workspace.CurrentCamera
if not cam then obj.line.Visible = false; return end
local origin = getTracerOrigin()
local throughWalls = Toggles.TracerThroughWalls and Toggles.TracerThroughWalls.Value or false
local screenPos, onScreen = cam:WorldToViewportPoint(root.Position)
if onScreen then
obj.line.From = origin
obj.line.To = Vector2.new(screenPos.X, screenPos.Y)
elseif throughWalls then
local camCF = cam.CFrame
local toPlayer = root.Position - camCF.Position
local dx = toPlayer:Dot(camCF.RightVector)
local dy = toPlayer:Dot(camCF.UpVector)
local angle = math.atan2(-dy, dx)
local lineLen = 200
obj.line.From = origin
obj.line.To = origin + Vector2.new(math.cos(angle), math.sin(angle)) * lineLen
else
obj.line.Visible = false
return
end
obj.line.Color = tracerColor
obj.line.Thickness = Options.TracerWidth and Options.TracerWidth.Value or 1
obj.line.Visible = true
end

RunService.RenderStepped:Connect(function()
if not (Toggles.EnableTracers and Toggles.EnableTracers.Value) then return end
if Window and Window.Visible == true then return end
for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer then
local obj = tracerObjects[plr]
if not obj then createTracer(plr); obj = tracerObjects[plr] end
if obj then updateTracer(plr) end
end
end
end)

Players.PlayerRemoving:Connect(removeTracer)

Toggles.EnableTracers:OnChanged(function()
if not Toggles.EnableTracers.Value then
for _, obj in pairs(tracerObjects) do
if obj.line then obj.line.Visible = false end
end
end
end)

do
VisualGrabLine = Tabs.Visual:AddRightGroupbox("Grab Line", "move")

grabLineOriginals = setmetatable({}, { __mode = "k" }) 
grabBeams = {} 

function isGrabBeam(obj)
return obj:IsA("Beam") and obj.Name == "GrabBeam"
end

function saveGrabBeamOriginal(beam)
if grabLineOriginals[beam] then return end
grabLineOriginals[beam] = {
Width0 = beam.Width0, Width1 = beam.Width1,
Texture = beam.Texture, TextureLength = beam.TextureLength,
TextureSpeed = beam.TextureSpeed, TextureMode = beam.TextureMode,
LightEmission = beam.LightEmission,
Color = beam.Color,
Transparency = beam.Transparency,
}
end

function restoreGrabBeam(beam)
local o = grabLineOriginals[beam]
if not o then return end
pcall(function()
beam.Width0 = o.Width0; beam.Width1 = o.Width1
beam.Texture = o.Texture; beam.TextureLength = o.TextureLength
beam.TextureSpeed = o.TextureSpeed; beam.TextureMode = o.TextureMode
beam.LightEmission = o.LightEmission
beam.Color = o.Color
beam.Transparency = o.Transparency
end)
end

grabTextures = {
["Lightning"] = { id = "rbxasset://textures/particles/sparkles_main.dds", length = 0.7, speed = 5 },
["Explosion"] = { id = "rbxasset://textures/particles/explosion01_implosion_main.dds", length = 2.5, speed = 1.5 },
["Flash"] = { id = "rbxasset://textures/particles/explosion01_seeds.dds", length = 1.4, speed = 2.5 },
["Plasma"] = { id = "rbxasset://textures/particles/explosion01_layer2.dds", length = 3, speed = 1.5, width = 1.0 },
["Smoke"] = { id = "rbxasset://textures/particles/smoke_main.dds", length = 2, speed = 1, width = 1.4 },
["Fire"] = { id = "rbxasset://textures/particles/fire_main.dds", length = 1.6, speed = 3, width = 1.2 },
["Sparks"] = { id = "rbxasset://textures/particles/sparkles_main.dds", length = 1.2, speed = 2, width = 0.9 },
["Non-Gamepass"] = { id = "rbxassetid://8933346550", length = 1, speed = 1, width = 1.0 },
["Gamepass"] = { id = "rbxassetid://8933355899", length = 1, speed = 1, width = 1.0 },
["Chain"] = { id = "rbxassetid://81358145120405", length = 1, speed = 1, width = 1.0 },
["Chain 2"] = { id = "rbxassetid://132910145874066", length = 1, speed = 1, width = 1.0 },
["Chain 3"] = { id = "rbxassetid://128466395060514", length = 1, speed = 1, width = 1.0 },
["Chain 4"] = { id = "rbxassetid://73368670987191", length = 1, speed = 1, width = 1.0 },
["Rope"] = { id = "rbxassetid://78999022056924", length = 1, speed = 1, width = 1.0 },
["Spring"] = { id = "rbxassetid://18837732116", length = 1, speed = 1, width = 1.0 },
["Circle"] = { id = "rbxassetid://5367817750", length = 1, speed = 1, width = 1.0 },
["Circle-Outline"] = { id = "rbxassetid://12201347372", length = 1, speed = 1, width = 1.0 },
["Triangle"] = { id = "rbxassetid://4704920160", length = 1, speed = 1, width = 1.0 },
["Triangle-Outline"] = { id = "rbxassetid://94666748694025", length = 1, speed = 1, width = 1.0 },
["Square"] = { id = "rbxassetid://15007588972", length = 1, speed = 1, width = 1.0 },
["Square-Outline"] = { id = "rbxassetid://15420927706", length = 1, speed = 1, width = 1.0 },
["Heart"] = { id = "rbxassetid://89015294175898", length = 1, speed = 1, width = 1.0 },
["Heart-Outline"] = { id = "rbxassetid://125373934805238", length = 1, speed = 1, width = 1.0 },
["Moon"] = { id = "rbxassetid://9013498676", length = 1, speed = 1, width = 1.0 },
["Dots"] = { id = "rbxassetid://9169659357", length = 1, speed = 1, width = 1.0 },
["Bubble"] = { id = "rbxassetid://1249690853", length = 1, speed = 1, width = 1.0 },
["Star"] = { id = "rbxassetid://5639840603", length = 1, speed = 1, width = 1.0 },
["Robux"] = { id = "rbxassetid://11560341132", length = 1, speed = 1, width = 1.0 },
["Roblox-Logo"] = { id = "rbxassetid://12348119032", length = 1, speed = 1, width = 1.0 },
["Brick"] = { id = "rbxassetid://4430903072", length = 1, speed = 1, width = 1.0 },
["Studs"] = { id = "rbxassetid://15539356451", length = 1, speed = 1, width = 1.0 },
["Fire"] = { id = "rbxassetid://18654087326", length = 1, speed = 1, width = 1.0 },
["Lazar"] = { id = "rbxassetid://8922958725", length = 1, speed = 1, width = 1.0 },
["Spider-Web"] = { id = "rbxassetid://123815660139244", length = 1, speed = 1, width = 1.0 },
["Smoke"] = { id = "rbxassetid://12900071392", length = 1, speed = 1, width = 1.0 },
["Audio-Visualiser"] = { id = "rbxassetid://81588563590679", length = 1, speed = 1, width = 1.0 },
["Pulse"] = { id = "rbxassetid://82163767314193", length = 1, speed = 1, width = 1.0 },
["Arrow"] = { id = "rbxassetid://9006027964", length = 1, speed = 1, width = 1.0 },
["Arrow 2"] = { id = "rbxassetid://10249261576", length = 1, speed = 1, width = 1.0 },
}

grabDashSeq = NumberSequence.new({
NumberSequenceKeypoint.new(0, 0),
NumberSequenceKeypoint.new(0.12, 0),
NumberSequenceKeypoint.new(0.13, 1),
NumberSequenceKeypoint.new(0.24, 1),
NumberSequenceKeypoint.new(0.25, 0),
NumberSequenceKeypoint.new(0.37, 0),
NumberSequenceKeypoint.new(0.38, 1),
NumberSequenceKeypoint.new(0.49, 1),
NumberSequenceKeypoint.new(0.5, 0),
NumberSequenceKeypoint.new(0.62, 0),
NumberSequenceKeypoint.new(0.63, 1),
NumberSequenceKeypoint.new(0.74, 1),
NumberSequenceKeypoint.new(0.75, 0),
NumberSequenceKeypoint.new(0.87, 0),
NumberSequenceKeypoint.new(0.88, 1),
NumberSequenceKeypoint.new(1, 1),
})
grabDotSeq = NumberSequence.new({
NumberSequenceKeypoint.new(0, 0),
NumberSequenceKeypoint.new(0.05, 0),
NumberSequenceKeypoint.new(0.06, 1),
NumberSequenceKeypoint.new(0.24, 1),
NumberSequenceKeypoint.new(0.25, 0),
NumberSequenceKeypoint.new(0.3, 0),
NumberSequenceKeypoint.new(0.31, 1),
NumberSequenceKeypoint.new(0.49, 1),
NumberSequenceKeypoint.new(0.5, 0),
NumberSequenceKeypoint.new(0.55, 0),
NumberSequenceKeypoint.new(0.56, 1),
NumberSequenceKeypoint.new(0.74, 1),
NumberSequenceKeypoint.new(0.75, 0),
NumberSequenceKeypoint.new(0.8, 0),
NumberSequenceKeypoint.new(0.81, 1),
NumberSequenceKeypoint.new(0.99, 1),
NumberSequenceKeypoint.new(1, 0),
})
grabCometSeq = NumberSequence.new({
NumberSequenceKeypoint.new(0, 1),
NumberSequenceKeypoint.new(0.6, 0.55),
NumberSequenceKeypoint.new(1, 0),
})
grabSolidSeq = NumberSequence.new(0)

grabRainbowConn = nil
grabRainbowHue = 0

grabPulseVal = 0
grabPulseDir = 1

function applyGrabBeamStyle(beam)
local style = (Options.GrabLineStyle and Options.GrabLineStyle.Value) or "Default"
local o = grabLineOriginals[beam]
local t = tick()
local tex = grabTextures[style]
local baseW = (Options.GrabLineWidth and Options.GrabLineWidth.Value) or 1

if style == "Default" then
if o then
beam.Texture = o.Texture
beam.TextureLength = o.TextureLength
beam.TextureSpeed = o.TextureSpeed
beam.TextureMode = o.TextureMode
beam.Width0 = o.Width0; beam.Width1 = o.Width1
beam.LightEmission = o.LightEmission
beam.Transparency = o.Transparency
beam.Color = o.Color
end
return
end

if style == "Low Quality" then
beam.Texture = ""
if o then
beam.TextureLength = o.TextureLength
beam.TextureSpeed = o.TextureSpeed
beam.TextureMode = o.TextureMode
end
beam.Width0 = baseW; beam.Width1 = baseW
beam.LightEmission = (Options.GrabLineLightEm and Options.GrabLineLightEm.Value) or 1
beam.Transparency = grabSolidSeq
elseif style == "Custom" then
local customId = (Options.GrabCustomTexId and Options.GrabCustomTexId.Value) or ""
local cleanId = tostring(customId):gsub("%D", "")
if cleanId ~= "" then
beam.Texture = "rbxassetid://" .. cleanId
beam.TextureMode = Enum.TextureMode.Wrap
beam.TextureLength = (Options.GrabLineLengthEnabled and Options.GrabLineLengthEnabled.Value and Options.GrabLineLength and Options.GrabLineLength.Value) or 1
beam.TextureSpeed = (Options.GrabLineSpeedEnabled and Options.GrabLineSpeedEnabled.Value and Options.GrabLineSpeed and Options.GrabLineSpeed.Value) or 1
elseif o then
beam.Texture = o.Texture
beam.TextureLength = o.TextureLength
beam.TextureSpeed = o.TextureSpeed
beam.TextureMode = o.TextureMode
end
beam.Width0 = baseW; beam.Width1 = baseW
beam.LightEmission = (Options.GrabLineLightEm and Options.GrabLineLightEm.Value) or 1
beam.Transparency = grabSolidSeq
elseif tex then
local texId = tex.id
if texId == "__CUSTOM__" then
local customId = (Options.GrabCustomTexId and Options.GrabCustomTexId.Value) or ""
local cleanId = tostring(customId):gsub("%D", "")
texId = cleanId ~= "" and ("rbxassetid://" .. cleanId) or ""
end
beam.Texture = texId
beam.TextureMode = Enum.TextureMode.Wrap
if Options.GrabLineLengthEnabled and Options.GrabLineLengthEnabled.Value and Options.GrabLineLength then
beam.TextureLength = Options.GrabLineLength.Value
else
beam.TextureLength = tex.length
end
if Options.GrabLineSpeedEnabled and Options.GrabLineSpeedEnabled.Value and Options.GrabLineSpeed then
beam.TextureSpeed = Options.GrabLineSpeed.Value
else
beam.TextureSpeed = tex.speed
end
beam.Width0 = baseW; beam.Width1 = baseW
beam.LightEmission = (Options.GrabLineLightEm and Options.GrabLineLightEm.Value) or 1
beam.Transparency = grabSolidSeq
else
beam.Width0 = baseW; beam.Width1 = baseW
beam.LightEmission = (Options.GrabLineLightEm and Options.GrabLineLightEm.Value) or 1
beam.Transparency = grabSolidSeq
end

if Toggles.GrabLineRainbow and Toggles.GrabLineRainbow.Value then
local h = (tick() * 0.5) % 1
local c = Color3.fromHSV(h, 1, 1)
beam.Color = ColorSequence.new(c, c)
elseif Toggles.GrabLineColorEnabled and Toggles.GrabLineColorEnabled.Value then
local c0 = (Options.GrabLineColor0 and Options.GrabLineColor0.Value) or Color3.fromRGB(255, 255, 255)
local c1 = (Options.GrabLineColor1 and Options.GrabLineColor1.Value) or c0
beam.Color = ColorSequence.new(c0, c1)
elseif o then
beam.Color = o.Color
end

if Toggles.GrabLinePulse and Toggles.GrabLinePulse.Value then
local pulseSpeed = (Options.GrabLinePulseSpeed and Options.GrabLinePulseSpeed.Value) or 2
local w = baseW * (0.55 + 0.45 * (0.5 + 0.5 * math.sin(t * pulseSpeed * 2)))
beam.Width0 = w; beam.Width1 = w
end
end

VisualGrabLine:AddToggle("EnableGrabLine", {
Text = "Custom Grab Line",
Default = false,
Tooltip = "Меняет форму и вид линии захвата (Grab), кото��ой ты хватаешь",
})

VisualGrabLine:AddDropdown("GrabLineStyle", {
Text = L("Форма линии", "Line Style"),
Default = "Default",
Values = {
"Default",
"Smoke", "Fire", "Sparks", "Lightning", "Plasma", "Explosion", "Flash",
"Non-Gamepass", "Gamepass",
"Chain", "Chain 2", "Chain 3", "Chain 4",
"Rope", "Spring",
"Circle", "Circle-Outline",
"Triangle", "Triangle-Outline",
"Square", "Square-Outline",
"Heart", "Heart-Outline",
"Moon", "Dots", "Bubble", "Star",
"Robux", "Roblox-Logo",
"Brick", "Studs",
"Fire", "Lazar",
"Spider-Web", "Smoke",
"Audio-Visualiser", "Pulse",
"Arrow", "Arrow 2",
},
Multi = false,
Tooltip = L("Вид линии захвата: 38 разных текстур (из unstable.txt + встроенные)", "Grab line style: 38 textures (from unstable.txt + built-in)"),
})

VisualGrabLine:AddSlider("GrabLineWidth", {
Text = L("Ширина линии", "Line Width"),
Default = 1,
Min = 0.05,
Max = 3,
Rounding = 2,
Compact = false,
Tooltip = L("Толщина линии захвата (для всех стилей, кроме «Обычная»)", "Grab line thickness (for all styles except 'Normal')"),
})

VisualGrabLine:AddInput("GrabCustomTexId", {
Text = "Custom Texture ID",
Default = "",
Placeholder = "rbxassetid://1234567890",
Tooltip = L("Введи asset ID текстуры (например 5639840603). Активируй «Custom» в дропдауне выше.", "Enter texture asset ID (e.g. 5639840603). Activate 'Custom' in dropdown above."),
})

VisualGrabLine:AddLabel(L("Цвет линии — Start", "Line Color — Start")):AddColorPicker("GrabLineColor0", {
Default = Color3.fromRGB(255, 255, 255),
Title = L("Цвет начала линии", "Line Start Color"),
})

VisualGrabLine:AddLabel(L("Цвет линии — End", "Line Color — End")):AddColorPicker("GrabLineColor1", {
Default = Color3.fromRGB(255, 255, 255),
Title = L("Цвет конца линии", "Line End Color"),
})

VisualGrabLine:AddToggle("GrabLineColorEnabled", {
Text = L("Включить цвет линии", "Enable Line Color"),
Default = false,
Tooltip = L("Применять выбранные цвета к линии (иначе — цвет игры)", "Apply selected colors to line (otherwise game color)"),
})

VisualGrabLine:AddToggle("GrabLineRainbow", {
Text = L("Rainbow (перелив)", "Rainbow"),
Default = false,
Tooltip = L("Линия автоматически переливается всеми цветами радуги", "Line automatically shimmers with all rainbow colors"),
})

VisualGrabLine:AddToggle("GrabLineSpeedEnabled", {
Text = L("Своя скорость текстуры", "Custom Texture Speed"),
Default = false,
Tooltip = L("Включить ползунок скорости прокрутки текстуры", "Enable texture scroll speed slider"),
})

VisualGrabLine:AddSlider("GrabLineSpeed", {
Text = L("Скорость текстуры", "Texture Speed"),
Default = 1,
Min = -10,
Max = 10,
Rounding = 1,
Compact = false,
Tooltip = L("Скорость прокрутки текстуры вдоль линии (отрицательная = реверс)", "Texture scroll speed along line (negative = reverse)"),
})

VisualGrabLine:AddToggle("GrabLineLengthEnabled", {
Text = L("Своя длина текстуры", "Custom Texture Length"),
Default = false,
Tooltip = L("Включить ползунок длины текстуры", "Enable texture length slider"),
})

VisualGrabLine:AddSlider("GrabLineLength", {
Text = L("Длина текстуры", "Texture Length"),
Default = 1,
Min = 0.1,
Max = 10,
Rounding = 2,
Compact = false,
Tooltip = L("Длина (размер) текстуры вдоль линии", "Texture length (size) along line"),
})

VisualGrabLine:AddSlider("GrabLineLightEm", {
Text = L("Свечение (LightEmission)", "Glow (LightEmission)"),
Default = 1,
Min = 0,
Max = 1,
Rounding = 2,
Compact = false,
Tooltip = L("Сила свечения текстуры (0 = нет, 1 = максимум)", "Texture glow intensity (0 = none, 1 = max)"),
})

VisualGrabLine:AddToggle("GrabLinePulse", {
Text = L("Пульсация ширины", "Width Pulse"),
Default = false,
Tooltip = L("Ширина линии пульсирует (как дыхание)", "Line width pulses (like breathing)"),
})

VisualGrabLine:AddSlider("GrabLinePulseSpeed", {
Text = L("Скорость пульсации", "Pulse Speed"),
Default = 2,
Min = 0.1,
Max = 10,
Rounding = 1,
Compact = false,
Tooltip = L("Как быстро пульсирует ширина", "How fast the width pulses"),
})

task.spawn(function()
for _, obj in ipairs(workspace:GetDescendants()) do
if isGrabBeam(obj) then grabBeams[obj] = true end
end
end)
workspace.DescendantAdded:Connect(function(obj)
if isGrabBeam(obj) then grabBeams[obj] = true end
end)
workspace.DescendantRemoving:Connect(function(obj)
if grabBeams[obj] then
grabBeams[obj] = nil
grabLineOriginals[obj] = nil
end
end)

Toggles.EnableGrabLine:OnChanged(function()
if not Toggles.EnableGrabLine.Value then
for beam in pairs(grabBeams) do
if beam and beam.Parent then restoreGrabBeam(beam) end
end
end
end)

RunService.RenderStepped:Connect(function()
if not (Toggles.EnableGrabLine and Toggles.EnableGrabLine.Value) then return end
for beam in pairs(grabBeams) do
if beam.Parent then
saveGrabBeamOriginal(beam)
pcall(applyGrabBeamStyle, beam)
else
grabBeams[beam] = nil
end
end
end)

do
local VisualHoverHL = Tabs.Visual:AddRightGroupbox("Target ESP", "crosshair")

VisualHoverHL:AddDropdown("TargetESPMode", {
Text = "Target ESP Mode",
Default = "None",
Values = {"None", "Style 1 (Highlight)", "Style 2 (Box)", "Style 3 (Brackets)", "Style 4 (Particles)"},
Tooltip = L("Выделять игрока при наведении", "Highlight player on hover"),
})

VisualHoverHL:AddLabel(L("Цвет Target ESP", "Target ESP Color")):AddColorPicker("TargetESPColor", {
Default = Color3.fromRGB(200, 0, 255),
Title = L("Цвет Target ESP", "Target ESP Color"),
})

VisualHoverHL:AddDropdown("HoverHighlightDist", {
Text = L("Дистанция", "Distance"),
Default = "30 studs",
Values = {"20 studs", "30 studs"},
Multi = false,
Tooltip = L("Максимальная дистанция для выделения", "Max distance for highlighting"),
})

do
local targetEspHl = nil
local targetEspBox = nil
local targetEspBrackets = nil
local targetEspParticles = nil

local function removeTargetESP()
if targetEspHl then pcall(function() targetEspHl:Destroy() end) targetEspHl = nil end
if targetEspBox then pcall(function() targetEspBox:Destroy() end) targetEspBox = nil end
if targetEspBrackets then pcall(function() targetEspBrackets:Destroy() end) targetEspBrackets = nil end
if targetEspParticles then pcall(function() targetEspParticles:Destroy() end) targetEspParticles = nil end
end

local function createBrackets(parent, color)
local bgui = Instance.new("BillboardGui")
bgui.Name = "OblivionTargetBrackets"
bgui.Adornee = parent
bgui.Size = UDim2.new(4, 0, 5.5, 0)
bgui.AlwaysOnTop = true
bgui.LightInfluence = 0

local thickness = 10
local length = 0.25

local function makeLine(size, pos, anchor)
local f = Instance.new("Frame")
f.BorderSizePixel = 0
f.BackgroundColor3 = color
f.Size = size
f.Position = pos
f.AnchorPoint = anchor
f.Parent = bgui
local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(1, 0)
uiCorner.Parent = f
return f
end

makeLine(UDim2.new(length, 0, 0, thickness), UDim2.new(0, 0, 0, 0), Vector2.new(0, 0))
makeLine(UDim2.new(0, thickness, length, 0), UDim2.new(0, 0, 0, 0), Vector2.new(0, 0))
makeLine(UDim2.new(length, 0, 0, thickness), UDim2.new(1, 0, 0, 0), Vector2.new(1, 0))
makeLine(UDim2.new(0, thickness, length, 0), UDim2.new(1, 0, 0, 0), Vector2.new(1, 0))
makeLine(UDim2.new(length, 0, 0, thickness), UDim2.new(0, 0, 1, 0), Vector2.new(0, 1))
makeLine(UDim2.new(0, thickness, length, 0), UDim2.new(0, 0, 1, 0), Vector2.new(0, 1))
makeLine(UDim2.new(length, 0, 0, thickness), UDim2.new(1, 0, 1, 0), Vector2.new(1, 1))
makeLine(UDim2.new(0, thickness, length, 0), UDim2.new(1, 0, 1, 0), Vector2.new(1, 1))

bgui.Parent = parent
return bgui
end

local function createParticles(parent, color)
local att = Instance.new("Attachment")
att.Name = "OblivionTargetParticlesAtt"
att.Position = Vector3.new(0, -1.5, 0)

local pe = Instance.new("ParticleEmitter")
pe.Name = "OblivionTargetParticles"
pe.Texture = "rbxassetid://284205403"
pe.Color = ColorSequence.new(color)
pe.Size = NumberSequence.new({NumberSequenceKeypoint.new(0, 0), NumberSequenceKeypoint.new(0.5, 2), NumberSequenceKeypoint.new(1, 0)})
pe.Rate = 40
pe.Lifetime = NumberRange.new(1, 2)
pe.Speed = NumberRange.new(3, 7)
pe.VelocitySpread = 60
pe.EmissionDirection = Enum.NormalId.Top
pe.Parent = att

att.Parent = parent
return att
end

RunService.RenderStepped:Connect(function()
local mode = Options.TargetESPMode and Options.TargetESPMode.Value or "None"
if mode == "None" then
removeTargetESP()
return
end

local mouse = LocalPlayer:GetMouse()
local hitPart = mouse.Target
if not hitPart then
removeTargetESP()
return
end

local distStr = Options.HoverHighlightDist and Options.HoverHighlightDist.Value or "30 studs"
local maxDist = tonumber(distStr:match("(%d+)")) or 30
local targetChar = hitPart:FindFirstAncestorOfClass("Model")
if not targetChar then
removeTargetESP()
return
end
local plr = Players:GetPlayerFromCharacter(targetChar)
if not plr or plr == LocalPlayer then
removeTargetESP()
return
end
local myChar = LocalPlayer.Character
local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
local theirRoot = targetChar:FindFirstChild("HumanoidRootPart")
if not myRoot or not theirRoot then
removeTargetESP()
return
end
local dist = (myRoot.Position - theirRoot.Position).Magnitude
if dist > maxDist then
removeTargetESP()
return
end

local root = targetChar:FindFirstChild("HumanoidRootPart")
if not root then removeTargetESP() return end

local espColor = Options.TargetESPColor and Options.TargetESPColor.Value or Color3.fromRGB(200, 0, 255)

if mode == "Style 1 (Highlight)" then
if targetEspBox then pcall(function() targetEspBox:Destroy() end) targetEspBox = nil end
if targetEspBrackets then pcall(function() targetEspBrackets:Destroy() end) targetEspBrackets = nil end
if targetEspParticles then pcall(function() targetEspParticles:Destroy() end) targetEspParticles = nil end

if not targetEspHl or targetEspHl.Parent ~= targetChar then
if targetEspHl then pcall(function() targetEspHl:Destroy() end) end
targetEspHl = Instance.new("Highlight")
targetEspHl.Name = "OblivionTargetESP"
targetEspHl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
targetEspHl.FillTransparency = 0.5
targetEspHl.OutlineTransparency = 0
targetEspHl.Parent = targetChar
end
targetEspHl.FillColor = espColor
targetEspHl.OutlineColor = espColor

elseif mode == "Style 2 (Box)" then
if targetEspHl then pcall(function() targetEspHl:Destroy() end) targetEspHl = nil end
if targetEspBrackets then pcall(function() targetEspBrackets:Destroy() end) targetEspBrackets = nil end
if targetEspParticles then pcall(function() targetEspParticles:Destroy() end) targetEspParticles = nil end

if not targetEspBox or targetEspBox.Parent ~= root then
if targetEspBox then pcall(function() targetEspBox:Destroy() end) end
targetEspBox = Instance.new("SelectionBox")
targetEspBox.Name = "OblivionTargetESPBox"
targetEspBox.LineThickness = 0.05
targetEspBox.SurfaceTransparency = 0.8
targetEspBox.Adornee = root
targetEspBox.Parent = root
end
targetEspBox.Color3 = espColor
targetEspBox.SurfaceColor3 = espColor

elseif mode == "Style 3 (Brackets)" then
if targetEspHl then pcall(function() targetEspHl:Destroy() end) targetEspHl = nil end
if targetEspBox then pcall(function() targetEspBox:Destroy() end) targetEspBox = nil end
if targetEspParticles then pcall(function() targetEspParticles:Destroy() end) targetEspParticles = nil end

if not targetEspBrackets or targetEspBrackets.Parent ~= root then
if targetEspBrackets then pcall(function() targetEspBrackets:Destroy() end) end
targetEspBrackets = createBrackets(root, espColor)
else
for _, child in ipairs(targetEspBrackets:GetChildren()) do
if child:IsA("Frame") then child.BackgroundColor3 = espColor end
end
end

elseif mode == "Style 4 (Particles)" then
if targetEspHl then pcall(function() targetEspHl:Destroy() end) targetEspHl = nil end
if targetEspBox then pcall(function() targetEspBox:Destroy() end) targetEspBox = nil end
if targetEspBrackets then pcall(function() targetEspBrackets:Destroy() end) targetEspBrackets = nil end

if not targetEspParticles or targetEspParticles.Parent ~= root then
if targetEspParticles then pcall(function() targetEspParticles:Destroy() end) end
targetEspParticles = createParticles(root, espColor)
else
local pe = targetEspParticles:FindFirstChild("OblivionTargetParticles")
if pe then pe.Color = ColorSequence.new(espColor) end
end
end
end)end

end
local VisualPallet = Tabs.Visual:AddLeftGroupbox("Pallet Color", "palette")

do
local HatSection = Tabs.Visual:AddRightGroupbox("Hats", "hat-glasses")
local hatTypes = { "Chinese Hat", "Crown", "Propeller", "Cat Ears", "Wizard Hat", "Cowboy Hat", "Demon Horns", "Knight Helmet" }

HatSection:AddToggle("EnableHat", {
Text = L("Надеть шляпу", "Wear Hat"),
Default = false,
Tooltip = L("Локальная визуальная шляпа", "Local visual hat"),
}):AddColorPicker("HatColor", {
Default = Color3.fromRGB(255, 50, 50),
Title = L("Цвет шляпы", "Hat Color"),
})

HatSection:AddDropdown("HatType", {
Text = L("Тип шляпы", "Hat Type"),
Values = hatTypes,
Default = "Chinese Hat",
Multi = false,
})

HatSection:AddSlider("HatSize", {
Text = L("Размер шляпы", "Hat Size"),
Default = 1,
Min = 0.5,
Max = 3,
Rounding = 1,
Compact = false,
})

HatSection:AddToggle("RainbowHat", {
Text = L("Радужная шляпа", "Rainbow Hat"),
Default = false,
})

local hatModel = nil
local hatParts = {}
local hatConn = nil
local hatMeshPart = nil
local hatMesh = nil

local hatMeshIds = {
["Wizard Hat"] = "rbxassetid://716245457",
["Cowboy Hat"] = "rbxassetid://1029586",
["Knight Helmet"] = "rbxassetid://8570385",
}
local hatYOffsets = {
["Wizard Hat"] = 1.1,
["Cowboy Hat"] = 0.7,
["Knight Helmet"] = 0.2,
}
local hatZOffsets = {
["Wizard Hat"] = 0.2,
["Cowboy Hat"] = 0,
["Knight Helmet"] = 0,
}
local hatMeshAsset = nil

local function clearHat()
if hatConn then hatConn:Disconnect(); hatConn = nil end
if hatModel then pcall(function() hatModel:Destroy() end); hatModel = nil end
if hatMeshPart then pcall(function() hatMeshPart:Destroy() end); hatMeshPart = nil end
hatMesh = nil
hatParts = {}
end

local function addHatPart(className, size, color)
local p = Instance.new(className or "Part")
p.Name = "OblivionHatPart"
p.Anchored = true
p.CanCollide = false
p.CanTouch = false
p.CanQuery = false
p.CastShadow = false
p.Material = Enum.Material.SmoothPlastic
p.Color = color
p.Size = size
p.Parent = hatModel
table.insert(hatParts, p)
return p
end

local function createHat()
clearHat()
local char = LocalPlayer.Character
local head = char and char:FindFirstChild("Head")
if not head then return end
local typ = (Options.HatType and Options.HatType.Value) or "Chinese Hat"
local s = (Options.HatSize and Options.HatSize.Value) or 1
local col = (Options.HatColor and Options.HatColor.Value) or Color3.fromRGB(255, 50, 50)

local meshId = hatMeshIds[typ]
if meshId then
hatMeshPart = Instance.new("Part")
hatMeshPart.Name = "OblivionHatMesh"
hatMeshPart.Anchored = true
hatMeshPart.CanCollide = false
hatMeshPart.CanTouch = false
hatMeshPart.CanQuery = false
hatMeshPart.CastShadow = false
hatMeshPart.Massless = true
hatMeshPart.Transparency = 0
hatMeshPart.Material = Enum.Material.SmoothPlastic
hatMeshPart.Color = col
hatMeshPart.Size = Vector3.new(0.1, 0.1, 0.1)
hatMeshPart.Parent = workspace
hatMesh = Instance.new("SpecialMesh")
hatMesh.MeshType = Enum.MeshType.FileMesh
hatMesh.MeshId = meshId
hatMesh.Scale = Vector3.new(s, s, s)
hatMesh.Parent = hatMeshPart
else
hatModel = Instance.new("Model")
hatModel.Name = "OblivionHat"
hatModel.Parent = workspace
if typ == "Chinese Hat" then
for i = 1, 16 do
addHatPart("WedgePart", Vector3.new(0.34*s, 0.22*s, 1.55*s), col)
end
addHatPart("Part", Vector3.new(0.08*s, 0.9*s, 0.08*s), Color3.fromRGB(40, 24, 10))
elseif typ == "Crown" then
addHatPart("Part", Vector3.new(1.55*s, 0.18*s, 1.55*s), col)
for i = 1, 12 do addHatPart("WedgePart", Vector3.new(0.20*s, 0.58*s, 0.28*s), col) end
for i = 1, 12 do addHatPart("Part", Vector3.new(0.16*s, 0.16*s, 0.16*s), Color3.fromHSV((i-1)/12, 0.9, 1)) end
addHatPart("Part", Vector3.new(0.38*s, 0.38*s, 0.38*s), Color3.fromRGB(255, 240, 90))
elseif typ == "Propeller" then
addHatPart("Part", Vector3.new(0.3*s,0.3*s,0.3*s), col).Shape = Enum.PartType.Ball
addHatPart("Part", Vector3.new(1.8*s,0.08*s,0.35*s), col)
addHatPart("Part", Vector3.new(0.35*s,0.08*s,1.8*s), col)
elseif typ == "Cat Ears" then
addHatPart("WedgePart", Vector3.new(0.5*s,0.65*s,0.22*s), col)
addHatPart("WedgePart", Vector3.new(0.5*s,0.65*s,0.22*s), col)
elseif typ == "Demon Horns" then
for i = 1, 6 do addHatPart("WedgePart", Vector3.new(0.18*s, 0.35*s, 0.6*s), col) end
for i = 1, 6 do addHatPart("WedgePart", Vector3.new(0.18*s, 0.35*s, 0.6*s), col) end
end
end

local spin = 0
hatConn = RunService.RenderStepped:Connect(function(dt)
local c = LocalPlayer.Character
local h = c and c:FindFirstChild("Head")
if not h then return end
typ = (Options.HatType and Options.HatType.Value) or typ
s = (Options.HatSize and Options.HatSize.Value) or s
col = (Options.HatColor and Options.HatColor.Value) or col
if Toggles.RainbowHat and Toggles.RainbowHat.Value then
col = Color3.fromHSV((tick() % 5) / 5, 1, 1)
end
local cam = workspace.CurrentCamera
local fp = cam and (cam.CFrame.Position - h.Position).Magnitude < 1.2
local tr = fp and 1 or 0
spin = spin + dt * 8

if hatMeshPart and hatMesh then
local yOff = hatYOffsets[typ] or 0
local zOff = hatZOffsets[typ] or 0
hatMeshPart.CFrame = h.CFrame * CFrame.new(0, yOff, zOff)
hatMeshPart.Color = col
hatMesh.Scale = Vector3.new(s, s, s)
hatMeshPart.Transparency = fp and 1 or 0
return
end

if not hatModel then return end

if typ == "Chinese Hat" and #hatParts >= 17 then
for i = 1, 16 do
local a = math.rad((i-1) * 22.5)
local p = hatParts[i]
p.Size = Vector3.new(0.34*s, 0.22*s, 1.55*s)
p.CFrame = h.CFrame * CFrame.new(math.cos(a)*0.70*s, 0.72*s, math.sin(a)*0.70*s)
* CFrame.Angles(0, -a, math.rad(-18))
p.Color = col
end
hatParts[17].Size = Vector3.new(0.08*s, 0.9*s, 0.08*s)
hatParts[17].CFrame = h.CFrame * CFrame.new(0, 0.18*s, 0)
hatParts[17].Color = Color3.fromRGB(40, 24, 10)
elseif typ == "Crown" and #hatParts >= 26 then
hatParts[1].Size = Vector3.new(1.55*s, 0.18*s, 1.55*s)
hatParts[1].CFrame = h.CFrame * CFrame.new(0, 0.76*s, 0)
hatParts[1].Color = col
for i = 1, 12 do
local a = math.rad((i-1) * 30)
local spike = hatParts[i+1]
spike.Size = Vector3.new(0.20*s, 0.58*s, 0.28*s)
spike.CFrame = h.CFrame * CFrame.new(math.cos(a)*0.62*s, 1.05*s, math.sin(a)*0.62*s)
* CFrame.Angles(0, -a, math.rad(10))
spike.Color = col
local gem = hatParts[i+13]
gem.Size = Vector3.new(0.16*s, 0.16*s, 0.16*s)
gem.CFrame = h.CFrame * CFrame.new(math.cos(a)*0.65*s, 1.38*s, math.sin(a)*0.65*s)
gem.Color = Color3.fromHSV((i-1)/12, 0.9, 1)
gem.Material = Enum.Material.Neon
end
hatParts[26].Size = Vector3.new(0.38*s, 0.38*s, 0.38*s)
hatParts[26].CFrame = h.CFrame * CFrame.new(0, 1.50*s, 0)
hatParts[26].Color = Color3.fromRGB(255, 240, 90)
hatParts[26].Material = Enum.Material.Neon
elseif typ == "Propeller" and #hatParts >= 3 then
hatParts[1].CFrame = h.CFrame * CFrame.new(0, 0.88*s, 0)
hatParts[2].CFrame = h.CFrame * CFrame.new(0, 1.08*s, 0) * CFrame.Angles(0, spin, 0)
hatParts[3].CFrame = h.CFrame * CFrame.new(0, 1.08*s, 0) * CFrame.Angles(0, spin, 0)
for _, p in ipairs(hatParts) do p.Color = col end
elseif typ == "Cat Ears" and #hatParts >= 2 then
hatParts[1].CFrame = h.CFrame * CFrame.new(-0.43*s, 0.86*s, 0) * CFrame.Angles(0, 0, math.rad(-20))
hatParts[2].CFrame = h.CFrame * CFrame.new(0.43*s, 0.86*s, 0) * CFrame.Angles(0, 0, math.rad(20))
for _, p in ipairs(hatParts) do p.Color = col end
elseif typ == "Demon Horns" and #hatParts >= 12 then
for i = 1, 6 do
local p = hatParts[i]
local h_frac = (i-1) / 5
local y = 0.8 + h_frac * 1.2
p.Size = Vector3.new(0.18*s, 0.35*s, 0.6*s)
p.CFrame = h.CFrame * CFrame.new(-0.4*s - h_frac*0.15*s, y*s, 0)
* CFrame.Angles(math.rad(20 + h_frac*30), 0, math.rad(-15 - h_frac*10))
p.Color = col
end
for i = 7, 12 do
local j = i - 6
local h_frac = (j-1) / 5
local y = 0.8 + h_frac * 1.2
p = hatParts[i]
p.Size = Vector3.new(0.18*s, 0.35*s, 0.6*s)
p.CFrame = h.CFrame * CFrame.new(0.4*s + h_frac*0.15*s, y*s, 0)
* CFrame.Angles(math.rad(20 + h_frac*30), 0, math.rad(15 + h_frac*10))
p.Color = col
end
end
for _, p in ipairs(hatParts) do
p.Transparency = fp and 1 or 0
end
end)
end

Toggles.EnableHat:OnChanged(function()
if Toggles.EnableHat.Value then createHat() else clearHat() end
end)
if Options.HatType then Options.HatType:OnChanged(function() if Toggles.EnableHat and Toggles.EnableHat.Value then createHat() end end) end
if Options.HatSize then Options.HatSize:OnChanged(function() if Toggles.EnableHat and Toggles.EnableHat.Value then createHat() end end) end
LocalPlayer.CharacterAdded:Connect(function()
task.wait(0.8)
if Toggles.EnableHat and Toggles.EnableHat.Value then createHat() end
end)
end

do
local PlayerEffects = Tabs.Visual:AddLeftGroupbox("Player Effects", "sparkles")

PlayerEffects:AddToggle("EnableJumpEffect", {
Text = "Jump Effect",
Default = false,
Tooltip = L("Рисует эффект при прыжке", "Draws effect on jump"),
})

PlayerEffects:AddDropdown("JumpEffectType", {
Text = L("Тип прыжка", "Jump Type"),
Values = {"Star Burst", "Smoke Ring", "Sparkle"},
Default = "Smoke Ring",
Multi = false,
})

PlayerEffects:AddSlider("JumpEffectRadius", {
Text = L("Радиус", "Radius"),
Default = 25,
Min = 5,
Max = 100,
Rounding = 1,
Compact = false,
})

PlayerEffects:AddSlider("JumpEffectDuration", {
Text = L("Длительность", "Duration"),
Default = 0.8,
Min = 0.1,
Max = 3,
Rounding = 2,
Compact = false,
})

PlayerEffects:AddLabel(L("Цвет прыжка", "Jump Color")):AddColorPicker("JumpEffectColor", {
Default = Color3.fromRGB(0, 255, 255),
Title = L("Цвет прыжка", "Jump Color"),
})

PlayerEffects:AddToggle("EnableWalkTrail", {
Text = "Walk Trail",
Default = false,
Tooltip = L("Рисует след за спиной при ходьбе", "Draws trail behind when walking"),
})

PlayerEffects:AddDropdown("WalkTrailType", {
Text = L("Тип следа", "Trail Type"),
Values = {"GlowThread", "Crystals", "Shadow Smoke"},
Default = "GlowThread",
Multi = false,
})

PlayerEffects:AddSlider("WalkTrailLength", {
Text = L("Длина следа", "Trail Length"),
Default = 200,
Min = 10,
Max = 1000,
Rounding = 0,
Compact = false,
})

PlayerEffects:AddLabel(L("Цвет следа", "Trail Color")):AddColorPicker("WalkTrailColor", {
Default = Color3.fromRGB(255, 0, 255),
Title = L("Цвет следа", "Trail Color"),
})

local jumpEffectObjects = {}
local walkTrailCustomParts = {}
local walkTrailAttach0 = nil
local walkTrailAttach1 = nil
local walkTrail = nil
local walkTrailCore = nil
local walkTrailEmitter = nil

local function cleanupJumpEffects()
for _, obj in ipairs(jumpEffectObjects) do
pcall(function() obj:Destroy() end)
end
jumpEffectObjects = {}
end

local function cleanupWalkTrail()
for _, p in ipairs(walkTrailCustomParts) do
pcall(function() p:Destroy() end)
end
walkTrailCustomParts = {}
if walkTrail then pcall(function() walkTrail:Destroy() end); walkTrail = nil end
if walkTrailCore then pcall(function() walkTrailCore:Destroy() end); walkTrailCore = nil end
if walkTrailEmitter then pcall(function() walkTrailEmitter:Destroy() end); walkTrailEmitter = nil end
if walkTrailAttach0 then pcall(function() walkTrailAttach0:Destroy() end); walkTrailAttach0 = nil end
if walkTrailAttach1 then pcall(function() walkTrailAttach1:Destroy() end); walkTrailAttach1 = nil end
end

local function createJumpEffect()
if not (Toggles.EnableJumpEffect and Toggles.EnableJumpEffect.Value) then return end
local char = LocalPlayer.Character
local root = char and char:FindFirstChild("HumanoidRootPart")
if not root then return end
local typ = Options.JumpEffectType and Options.JumpEffectType.Value or "Smoke Ring"
local radius = Options.JumpEffectRadius and Options.JumpEffectRadius.Value or 25
local duration = Options.JumpEffectDuration and Options.JumpEffectDuration.Value or 0.8
local color = Options.JumpEffectColor and Options.JumpEffectColor.Value or Color3.fromRGB(0, 255, 255)
local pos = root.Position - Vector3.new(0, 2.9, 0)

if typ == "Smoke Ring" then
local part = Instance.new("Part")
part.Anchored = true
part.CanCollide = false
part.CanQuery = false
part.CanTouch = false
part.Transparency = 1
part.Size = Vector3.new(0.1, 0.1, 0.1)
part.CFrame = CFrame.new(pos)
part.Parent = workspace
table.insert(jumpEffectObjects, part)
local emitter = Instance.new("ParticleEmitter")
emitter.Texture = "rbxassetid://7083340510"
emitter.Color = ColorSequence.new(color)
emitter.Size = NumberSequence.new({
NumberSequenceKeypoint.new(0, radius * 0.1),
NumberSequenceKeypoint.new(0.5, radius * 0.2),
NumberSequenceKeypoint.new(1, 0)
})
emitter.Transparency = NumberSequence.new({
NumberSequenceKeypoint.new(0, 0),
NumberSequenceKeypoint.new(0.7, 0.3),
NumberSequenceKeypoint.new(1, 1)
})
emitter.Lifetime = NumberRange.new(duration, duration)
emitter.Rate = 0
emitter.Rotation = NumberRange.new(0, 360)
emitter.RotSpeed = NumberRange.new(-90, 90)
emitter.Speed = NumberRange.new(0, 0)
emitter.LightEmission = 1
emitter.Parent = part
emitter:Emit(30)
task.delay(duration + 0.5, function()
pcall(function() part:Destroy() end)
end)
elseif typ == "Star Burst" then
local part = Instance.new("Part")
part.Anchored = true
part.CanCollide = false
part.CanQuery = false
part.CanTouch = false
part.Transparency = 1
part.Size = Vector3.new(0.1, 0.1, 0.1)
part.CFrame = CFrame.new(pos)
part.Parent = workspace
table.insert(jumpEffectObjects, part)
local emitter = Instance.new("ParticleEmitter")
emitter.Texture = "rbxassetid://16857504782"
emitter.Color = ColorSequence.new(color)
emitter.Size = NumberSequence.new({
NumberSequenceKeypoint.new(0, radius * 0.15),
NumberSequenceKeypoint.new(1, 0)
})
emitter.Transparency = NumberSequence.new({
NumberSequenceKeypoint.new(0, 0),
NumberSequenceKeypoint.new(1, 1)
})
emitter.Lifetime = NumberRange.new(duration * 0.5, duration)
emitter.Rate = 0
emitter.Rotation = NumberRange.new(0, 360)
emitter.RotSpeed = NumberRange.new(-180, 180)
emitter.Speed = NumberRange.new(radius * 2, radius * 4)
emitter.SpreadAngle = Vector2.new(180, 180)
emitter.LightEmission = 1
emitter.Acceleration = Vector3.new(0, -10, 0)
emitter.Parent = part
emitter:Emit(50)
task.delay(duration + 1, function()
pcall(function() part:Destroy() end)
end)
else
local part = Instance.new("Part")
part.Anchored = true
part.CanCollide = false
part.CanQuery = false
part.CanTouch = false
part.Transparency = 1
part.Size = Vector3.new(0.1, 0.1, 0.1)
part.CFrame = CFrame.new(pos + Vector3.new(0, 1, 0))
part.Parent = workspace
table.insert(jumpEffectObjects, part)
local light = Instance.new("PointLight")
light.Color = color
light.Range = radius * 3
light.Brightness = 20
light.Shadows = false
light.Parent = part
local emitter = Instance.new("ParticleEmitter")
emitter.Texture = "rbxassetid://16857504782"
emitter.Color = ColorSequence.new(color)
emitter.Size = NumberSequence.new({
NumberSequenceKeypoint.new(0, radius * 0.2),
NumberSequenceKeypoint.new(1, 0)
})
emitter.Transparency = NumberSequence.new({
NumberSequenceKeypoint.new(0, 0),
NumberSequenceKeypoint.new(1, 1)
})
emitter.Lifetime = NumberRange.new(duration * 0.3, duration * 0.5)
emitter.Rate = 0
emitter.Rotation = NumberRange.new(0, 360)
emitter.Speed = NumberRange.new(0, 0)
emitter.LightEmission = 1
emitter.Parent = part
emitter:Emit(15)
task.spawn(function()
local t = 0
while t < duration and part.Parent do
t = t + task.wait()
local alpha = t / duration
light.Range = radius * 3 * (1 - alpha)
light.Brightness = 20 * (1 - alpha)
end
pcall(function() part:Destroy() end)
end)
end
end

local function setupTrailLine()
local char = LocalPlayer.Character
local root = char and char:FindFirstChild("HumanoidRootPart")
if not root then return end
walkTrailAttach0 = Instance.new("Attachment")
walkTrailAttach0.Position = Vector3.new(0, -2.5, 2.0)
walkTrailAttach0.Parent = root
walkTrailAttach1 = Instance.new("Attachment")
walkTrailAttach1.Position = Vector3.new(0, -2.5, -2.0)
walkTrailAttach1.Parent = root
walkTrail = Instance.new("Trail")
walkTrail.Name = "OblivionWalkTrail"
walkTrail.Attachment0 = walkTrailAttach0
walkTrail.Attachment1 = walkTrailAttach1
walkTrail.Texture = ""
walkTrail.LightEmission = 1
walkTrail.LightInfluence = 0
walkTrail.WidthScale = NumberSequence.new({
NumberSequenceKeypoint.new(0, 8),
NumberSequenceKeypoint.new(1, 0)
})
walkTrail.Parent = root
walkTrailCore = Instance.new("Trail")
walkTrailCore.Name = "OblivionWalkTrailCore"
walkTrailCore.Attachment0 = walkTrailAttach0
walkTrailCore.Attachment1 = walkTrailAttach1
walkTrailCore.Texture = ""
walkTrailCore.Color = ColorSequence.new(Color3.fromRGB(255, 255, 255))
walkTrailCore.LightEmission = 1
walkTrailCore.LightInfluence = 0
walkTrailCore.WidthScale = NumberSequence.new({
NumberSequenceKeypoint.new(0, 3),
NumberSequenceKeypoint.new(1, 0)
})
walkTrailCore.Parent = root
end

local lastWalkPos = nil

local function updateWalkTrail()
if not (Toggles.EnableWalkTrail and Toggles.EnableWalkTrail.Value) then return end
local char = LocalPlayer.Character
local root = char and char:FindFirstChild("HumanoidRootPart")
if not root then return end
local typ = Options.WalkTrailType and Options.WalkTrailType.Value or "GlowThread"
local color = Options.WalkTrailColor and Options.WalkTrailColor.Value or Color3.fromRGB(255, 0, 255)
local length = Options.WalkTrailLength and Options.WalkTrailLength.Value or 200

if typ == "GlowThread" then
if not walkTrail then setupTrailLine() end
if walkTrail then
walkTrail.Color = ColorSequence.new(color)
walkTrail.Lifetime = math.clamp(length / 2, 0.5, 40)
walkTrail.Enabled = true
end
if walkTrailCore then
walkTrailCore.Lifetime = walkTrail.Lifetime
walkTrailCore.Enabled = true
end
return
end

if walkTrail then walkTrail.Enabled = false end
if walkTrailCore then walkTrailCore.Enabled = false end

if typ == "Shadow Smoke" then
if not walkTrailEmitter then
walkTrailEmitter = Instance.new("ParticleEmitter")
walkTrailEmitter.Name = "OblivionSmokeTrail"
walkTrailEmitter.Texture = "rbxassetid://7083340510"
walkTrailEmitter.Color = ColorSequence.new(color, Color3.fromRGB(50, 50, 50))
walkTrailEmitter.Size = NumberSequence.new({
NumberSequenceKeypoint.new(0, 3),
NumberSequenceKeypoint.new(1, 0)
})
walkTrailEmitter.Transparency = NumberSequence.new({
NumberSequenceKeypoint.new(0, 0.3),
NumberSequenceKeypoint.new(1, 1)
})
walkTrailEmitter.Lifetime = NumberRange.new(1, 2)
walkTrailEmitter.Speed = NumberRange.new(1, 3)
walkTrailEmitter.SpreadAngle = Vector2.new(30, 30)
walkTrailEmitter.Rate = 40
walkTrailEmitter.Rotation = NumberRange.new(0, 360)
walkTrailEmitter.RotSpeed = NumberRange.new(-45, 45)
walkTrailEmitter.LightEmission = 0.3
walkTrailEmitter.Acceleration = Vector3.new(0, 3, 0)
walkTrailEmitter.Parent = root
end
walkTrailEmitter.Color = ColorSequence.new(color, Color3.fromRGB(50, 50, 50))
walkTrailEmitter.Enabled = true
return
end

if walkTrailEmitter then walkTrailEmitter.Enabled = false end

if typ == "Crystals" then
local backPos = root.Position - Vector3.new(0, 3, 0)
if lastWalkPos and (backPos - lastWalkPos).Magnitude < 1 then return end
lastWalkPos = backPos
local p = Instance.new("Part")
p.Anchored = true
p.CanCollide = false
p.CanQuery = false
p.CanTouch = false
p.Material = Enum.Material.Neon
p.Color = color
p.Transparency = 0.2
local height = math.random(20, 60) * 0.1
p.Size = Vector3.new(0.3, height, 0.3)
local offset = Vector3.new(math.random(-30, 30)*0.1, height/2, math.random(-30, 30)*0.1)
p.CFrame = CFrame.new(backPos + offset) * CFrame.Angles(math.rad(math.random(-40, 40)), math.rad(math.random(0, 360)), math.rad(math.random(-20, 20)))
p.Parent = workspace
table.insert(walkTrailCustomParts, p)
while #walkTrailCustomParts > length do
local old = table.remove(walkTrailCustomParts, 1)
pcall(function() old:Destroy() end)
end
end
end

Toggles.EnableJumpEffect:OnChanged(function()
if not Toggles.EnableJumpEffect.Value then
cleanupJumpEffects()
end
end)

Toggles.EnableWalkTrail:OnChanged(function()
if Toggles.EnableWalkTrail.Value then
else
cleanupWalkTrail()
end
end)

if Options.WalkTrailType then
Options.WalkTrailType:OnChanged(function()
if Toggles.EnableWalkTrail and Toggles.EnableWalkTrail.Value and LocalPlayer.Character then
cleanupWalkTrail()
end
end)
end

if Options.WalkTrailColor then
Options.WalkTrailColor:OnChanged(function()
if walkTrail and Toggles.EnableWalkTrail and Toggles.EnableWalkTrail.Value then
walkTrail.Color = ColorSequence.new(Options.WalkTrailColor.Value)
end
if walkTrailEmitter and Toggles.EnableWalkTrail and Toggles.EnableWalkTrail.Value then
walkTrailEmitter.Color = ColorSequence.new(Options.WalkTrailColor.Value, Color3.fromRGB(50, 50, 50))
end
end)
end

local function hookJumpEffect(char)
local hum = char:FindFirstChildOfClass("Humanoid")
if not hum then return end
hum.StateChanged:Connect(function(_, newState)
if newState == Enum.HumanoidStateType.Jumping then
createJumpEffect()
end
end)
end

if LocalPlayer.Character then
task.delay(0.5, function()
hookJumpEffect(LocalPlayer.Character)
end)
end

LocalPlayer.CharacterAdded:Connect(function(char)
task.wait(0.5)
cleanupJumpEffects()
cleanupWalkTrail()
hookJumpEffect(char)
end)

RunService.Heartbeat:Connect(function()
updateWalkTrail()
end)
end
palletColor = Color3.fromRGB(0, 255, 128)
palletColorData = {} 

VisualPallet:AddToggle("EnablePalletColor", {
Text = L("Цвет палетки под ногами", "Pallet Color Under Feet"),
Default = false,
Tooltip = L("Красит палетку, на которой стоишь; сброс через N сек после схода", "Colors the pallet you stand on; resets N sec after stepping off"),
}):AddColorPicker("PalletColor", {
Default = Color3.fromRGB(0, 255, 128),
Title = L("Цвет палетки", "Pallet Color"),
Callback = function(Value)
palletColor = Value
end,
})

VisualPallet:AddSlider("PalletResetDelay", {
Text = L("Сброс через (сек)", "Reset After (sec)"),
Default = 3,
Min = 0,
Max = 10,
Rounding = 1,
Compact = false,
Tooltip = L("Через сколько секунд после схода палетка вернёт исходный цвет", "Seconds after stepping off before pallet resets color"),
})

VisualPallet:AddSlider("PalletTransparency", {
Text = L("Прозрачность", "Transparency"),
Default = 0.5,
Min = 0,
Max = 1,
Rounding = 2,
Compact = false,
Tooltip = L("0 — непрозрачная, 1 — полностью невидимая", "0 = opaque, 1 = fully invisible"),
})

VisualPallet:AddSlider("PalletGlow", {
Text = L("Подсветка", "Highlight"),
Default = 0,
Min = 0,
Max = 1,
Rounding = 2,
Compact = false,
Tooltip = "0 ��� без подсветки, 1 — максимальное свечение со всех сторон",
})

VisualPallet:AddSlider("PalletSmoothness", {
Text = L("Плавность появления", "Fade In Smoothness"),
Default = 0,
Min = 0,
Max = 1,
Rounding = 2,
Compact = false,
Tooltip = L("0 — цвет появляется сразу; 1 — очень плавное появление", "0 = color appears instantly; 1 = very smooth fade in"),
})

local function getPalletFromPart(part)
local cur = part
while cur and cur ~= workspace do
if string.find(string.lower(cur.Name), "pallet") then
return cur
end
cur = cur.Parent
end
return nil
end

local function getPalletParts(pallet)
local parts = {}
if pallet:IsA("BasePart") then
table.insert(parts, pallet)
end
for _, d in ipairs(pallet:GetDescendants()) do
if d:IsA("BasePart") then
table.insert(parts, d)
end
end
return parts
end

local function restorePallet(data)
for part, orig in pairs(data.originals) do
if part and part.Parent then
pcall(function()
part.Color = orig.color
part.Transparency = orig.transparency
local light = part:FindFirstChild("OblivionPalletLight")
if light then light:Destroy() end
if part.Material == Enum.Material.Neon then
part.Material = Enum.Material.SmoothPlastic
end
end)
end
end
if data.boxes then
for p, box in pairs(data.boxes) do
pcall(function() box:Destroy() end)
data.boxes[p] = nil
end
end
if data.highlight then
pcall(function() data.highlight:Destroy() end)
data.highlight = nil
end
end

local function restoreAllPallets()
for _, data in pairs(palletColorData) do
restorePallet(data)
end
palletColorData = {}
end

Toggles.EnablePalletColor:OnChanged(function()
if not Toggles.EnablePalletColor.Value then
restoreAllPallets()
end
end)

inputOverlayGui = nil
inputOverlayKeys = {}
inputOverlayMouse = {}
inputOverlayConn = nil

local function createInputOverlay()
if inputOverlayGui then return end
local sg = Instance.new("ScreenGui")
sg.Name = "InputOverlay"
sg.ResetOnSpawn = false
sg.IgnoreGuiInset = true
sg.DisplayOrder = 9999
sg.Parent = game:GetService("CoreGui")
local container = Instance.new("Frame")
container.Name = "Container"
container.Size = UDim2.new(0, 230, 0, 110)
container.Position = UDim2.new(0, 20, 1, -130)
container.BackgroundTransparency = 1
container.Parent = sg
inputOverlayGui = sg
local keySize = 35
local keyGap = 4
local keyStartX = 0
local keyStartY = 35
local keyLayout = {
{key = "W", x = keyStartX + keySize + keyGap, y = keyStartY},
{key = "A", x = keyStartX, y = keyStartY + keySize + keyGap},
{key = "S", x = keyStartX + keySize + keyGap, y = keyStartY + keySize + keyGap},
{key = "D", x = keyStartX + (keySize + keyGap) * 2, y = keyStartY + keySize + keyGap},
}
for _, layout in ipairs(keyLayout) do
local frame = Instance.new("Frame")
frame.Name = "Key_" .. layout.key
frame.Size = UDim2.new(0, keySize, 0, keySize)
frame.Position = UDim2.new(0, layout.x, 0, layout.y)
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.BackgroundTransparency = 0.3
frame.BorderSizePixel = 1
frame.BorderColor3 = Color3.fromRGB(60, 60, 60)
frame.Parent = container
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 6)
corner.Parent = frame
local label = Instance.new("TextLabel")
label.Size = UDim2.new(1, 0, 1, 0)
label.BackgroundTransparency = 1
label.Text = layout.key
label.TextColor3 = Color3.fromRGB(200, 200, 200)
label.Font = Enum.Font.GothamBold
label.TextSize = 16
label.Parent = frame
inputOverlayKeys[layout.key] = frame
end
local mouseX = keyStartX + (keySize + keyGap) * 3 + 18
local mouseY = keyStartY - 2
local mouseBody = Instance.new("Frame")
mouseBody.Name = "MouseBody"
mouseBody.Size = UDim2.new(0, 40, 0, 72)
mouseBody.Position = UDim2.new(0, mouseX, 0, mouseY)
mouseBody.BackgroundColor3 = Color3.fromRGB(35, 35, 40)
mouseBody.BackgroundTransparency = 0.1
mouseBody.BorderSizePixel = 0
mouseBody.Parent = container
local mbCorner = Instance.new("UICorner")
mbCorner.CornerRadius = UDim.new(0, 12)
mbCorner.Parent = mouseBody
local mbStroke = Instance.new("UIStroke")
mbStroke.Color = Color3.fromRGB(80, 80, 90)
mbStroke.Thickness = 1.5
mbStroke.Transparency = 0.15
mbStroke.Parent = mouseBody
local mbGrad = Instance.new("UIGradient")
mbGrad.Rotation = 90
mbGrad.Color = ColorSequence.new({
ColorSequenceKeypoint.new(0, Color3.fromRGB(50, 50, 58)),
ColorSequenceKeypoint.new(0.5, Color3.fromRGB(38, 38, 44)),
ColorSequenceKeypoint.new(1, Color3.fromRGB(28, 28, 33)),
})
mbGrad.Parent = mouseBody
local splitLine = Instance.new("Frame")
splitLine.Name = "SplitLine"
splitLine.Size = UDim2.new(0, 1, 0, 32)
splitLine.Position = UDim2.new(0, 19.5, 0, 3)
splitLine.BackgroundColor3 = Color3.fromRGB(70, 70, 80)
splitLine.BackgroundTransparency = 0.3
splitLine.BorderSizePixel = 0
splitLine.Parent = mouseBody
local mouseLeft = Instance.new("Frame")
mouseLeft.Name = "MouseLeft"
mouseLeft.Size = UDim2.new(0, 17, 0, 32)
mouseLeft.Position = UDim2.new(0, 2, 0, 2)
mouseLeft.BackgroundColor3 = Color3.fromRGB(45, 45, 52)
mouseLeft.BackgroundTransparency = 0.05
mouseLeft.BorderSizePixel = 0
mouseLeft.Parent = mouseBody
local mlCorner = Instance.new("UICorner")
mlCorner.CornerRadius = UDim.new(0, 8)
mlCorner.Parent = mouseLeft
local mlStroke = Instance.new("UIStroke")
mlStroke.Color = Color3.fromRGB(65, 65, 75)
mlStroke.Thickness = 1
mlStroke.Transparency = 0.25
mlStroke.Parent = mouseLeft
local mouseRight = Instance.new("Frame")
mouseRight.Name = "MouseRight"
mouseRight.Size = UDim2.new(0, 17, 0, 32)
mouseRight.Position = UDim2.new(0, 21, 0, 2)
mouseRight.BackgroundColor3 = Color3.fromRGB(45, 45, 52)
mouseRight.BackgroundTransparency = 0.05
mouseRight.BorderSizePixel = 0
mouseRight.Parent = mouseBody
local mrCorner = Instance.new("UICorner")
mrCorner.CornerRadius = UDim.new(0, 8)
mrCorner.Parent = mouseRight
local mrStroke = Instance.new("UIStroke")
mrStroke.Color = Color3.fromRGB(65, 65, 75)
mrStroke.Thickness = 1
mrStroke.Transparency = 0.25
mrStroke.Parent = mouseRight
inputOverlayMouse.Left = mouseLeft
inputOverlayMouse.Right = mouseRight
inputOverlayMouse.LeftStroke = mlStroke
inputOverlayMouse.RightStroke = mrStroke
end

local function destroyInputOverlay()
if inputOverlayGui and inputOverlayGui.Parent then inputOverlayGui:Destroy() end
inputOverlayGui = nil
inputOverlayKeys = {}
inputOverlayMouse = {}
end

local function startInputOverlayTracking()
if inputOverlayConn then return end
local UIS = game:GetService("UserInputService")
inputOverlayConn = RunService.RenderStepped:Connect(function()
if not inputOverlayGui then return end
for key, frame in pairs(inputOverlayKeys) do
local pressed = false
if key == "W" then pressed = UIS:IsKeyDown(Enum.KeyCode.W) end
if key == "A" then pressed = UIS:IsKeyDown(Enum.KeyCode.A) end
if key == "S" then pressed = UIS:IsKeyDown(Enum.KeyCode.S) end
if key == "D" then pressed = UIS:IsKeyDown(Enum.KeyCode.D) end
if pressed then
frame.BackgroundColor3 = Color3.fromRGB(100, 200, 100)
frame.BackgroundTransparency = 0.1
else
frame.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
frame.BackgroundTransparency = 0.3
end
end
if inputOverlayMouse.Left then
if UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
inputOverlayMouse.Left.BackgroundColor3 = Color3.fromRGB(80, 140, 255)
inputOverlayMouse.Left.BackgroundTransparency = 0
if inputOverlayMouse.LeftStroke then
inputOverlayMouse.LeftStroke.Color = Color3.fromRGB(120, 180, 255)
inputOverlayMouse.LeftStroke.Transparency = 0
end
else
inputOverlayMouse.Left.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
inputOverlayMouse.Left.BackgroundTransparency = 0.1
if inputOverlayMouse.LeftStroke then
inputOverlayMouse.LeftStroke.Color = Color3.fromRGB(60, 60, 70)
inputOverlayMouse.LeftStroke.Transparency = 0.3
end
end
end
if inputOverlayMouse.Right then
if UIS:IsMouseButtonPressed(Enum.UserInputType.MouseButton2) then
inputOverlayMouse.Right.BackgroundColor3 = Color3.fromRGB(255, 80, 80)
inputOverlayMouse.Right.BackgroundTransparency = 0
if inputOverlayMouse.RightStroke then
inputOverlayMouse.RightStroke.Color = Color3.fromRGB(255, 140, 140)
inputOverlayMouse.RightStroke.Transparency = 0
end
else
inputOverlayMouse.Right.BackgroundColor3 = Color3.fromRGB(50, 50, 55)
inputOverlayMouse.Right.BackgroundTransparency = 0.1
if inputOverlayMouse.RightStroke then
inputOverlayMouse.RightStroke.Color = Color3.fromRGB(60, 60, 70)
inputOverlayMouse.RightStroke.Transparency = 0.3
end
end
end
end)
end

local function stopInputOverlayTracking()
if inputOverlayConn then inputOverlayConn:Disconnect() inputOverlayConn = nil end
end

VisualPallet:AddToggle("EnableInputOverlay", {
Text = "Input Overlay",
Default = false,
Tooltip = "Shows WASD + mouse clicks at bottom left",
}):OnChanged(function()
if Toggles.EnableInputOverlay.Value then
createInputOverlay()
startInputOverlayTracking()
else
stopInputOverlayTracking()
destroyInputOverlay()
end
end)

RunService.Heartbeat:Connect(function()
if not (Toggles.EnablePalletColor and Toggles.EnablePalletColor.Value) then
return
end
local char = LocalPlayer.Character
local root = char and char:FindFirstChild("HumanoidRootPart")
local hum = char and char:FindFirstChildOfClass("Humanoid")
local now = tick()
local delay = Options.PalletResetDelay and Options.PalletResetDelay.Value or 3

local standingPallet = nil
if root and hum and hum.Health > 0 and hum.FloorMaterial ~= Enum.Material.Air then
local params = RaycastParams.new()
params.FilterType = Enum.RaycastFilterType.Exclude
params.FilterDescendantsInstances = { char }
local result = workspace:Raycast(root.Position, Vector3.new(0, -10, 0), params)
if result then
standingPallet = getPalletFromPart(result.Instance)
end
end

if standingPallet then
local data = palletColorData[standingPallet]
if not data then
data = { originals = {}, lastStand = now }
for _, part in ipairs(getPalletParts(standingPallet)) do
data.originals[part] = { color = part.Color, transparency = part.Transparency }
end
palletColorData[standingPallet] = data
end
data.lastStand = now
local wantT = (Options.PalletTransparency and Options.PalletTransparency.Value) or 0
local smooth = (Options.PalletSmoothness and Options.PalletSmoothness.Value) or 0

for part in pairs(data.originals) do
if part and part.Parent then
if smooth > 0 then
local dt = 1/60 
local alpha = math.clamp(dt / math.max(0.05, smooth), 0.01, 1)
if part.Color ~= palletColor then
part.Color = part.Color:Lerp(palletColor, alpha)
end
if math.abs(part.Transparency - wantT) > 0.01 then
part.Transparency = part.Transparency + (wantT - part.Transparency) * alpha
end
else
if part.Color ~= palletColor then part.Color = palletColor end
if part.Transparency ~= wantT then part.Transparency = wantT end
end
local glow = (Options.PalletGlow and Options.PalletGlow.Value) or 0
local light = part:FindFirstChild("OblivionPalletLight")
if glow > 0 then
if glow >= 0.55 then part.Material = Enum.Material.Neon else part.Material = Enum.Material.SmoothPlastic end
part.Color = palletColor:Lerp(Color3.new(1, 1, 1), glow * 0.35)
if not light then
light = Instance.new("PointLight")
light.Name = "OblivionPalletLight"
light.Shadows = false
light.Parent = part
end
light.Enabled = true
light.Color = palletColor
light.Brightness = 0.2 + glow * 7
light.Range = 3 + glow * 20
else
if light then light.Enabled = false end
if part.Material == Enum.Material.Neon then part.Material = Enum.Material.SmoothPlastic end
end
end
end
end

for pallet, data in pairs(palletColorData) do
if not pallet.Parent then
palletColorData[pallet] = nil
elseif pallet ~= standingPallet and now - data.lastStand >= delay then
restorePallet(data)
palletColorData[pallet] = nil
end
end
end)

do
local XocoSkinSection = Tabs.Visual:AddLeftGroupbox("Custom Skin", "shirt")

local skinConn = nil
local skinOriginals = {}

local function getXocoColor()
if Options.CustomSkinColor and Options.CustomSkinColor.Value then
return Options.CustomSkinColor.Value
end
return Color3.fromRGB(255, 255, 255)
end

local function getXocoMaterial()
local name = Options.CustomSkinMaterial and Options.CustomSkinMaterial.Value or "Default"
if name == "Default" then return nil end
local ok, mat = pcall(function() return Enum.Material[name] end)
if ok then return mat end
return nil
end

local function applyXocoSkin()
local char = LocalPlayer.Character
if not char then return end
local color = getXocoColor()
local mat = getXocoMaterial()
for _, part in ipairs(char:GetDescendants()) do
if part:IsA("Clothing") or part:IsA("ShirtGraphic") or part:IsA("Decal") then
if not skinOriginals[part] then
skinOriginals[part] = { Type = "Clothing", Parent = part.Parent }
end
pcall(function() part.Parent = game:GetService("Lighting") end)
elseif part:IsA("BasePart") then
if not skinOriginals[part] then
local tex = part:IsA("MeshPart") and part.TextureID or nil
skinOriginals[part] = { Type = "BasePart", Material = part.Material, Color = part.Color, Transparency = part.Transparency, TextureID = tex }
end
pcall(function()
part.Color = color
if mat then part.Material = mat end
if part:IsA("MeshPart") then part.TextureID = "" end
end)
elseif part:IsA("SpecialMesh") then
if not skinOriginals[part] then
skinOriginals[part] = { Type = "SpecialMesh", TextureId = part.TextureId }
end
pcall(function() part.TextureId = "" end)
end
end
end

XocoSkinSection:AddToggle("EnableCustomSkin", {
Text = "Custom Skin",
Default = false,
Tooltip = L("Красит твою модель игрока в выбранный цвет", "Colors your player model in selected color"),
Callback = function(Value)
if Value then
if not skinConn then
skinConn = RunService.Heartbeat:Connect(function()
applyXocoSkin()
end)
end
else
if skinConn then
skinConn:Disconnect()
skinConn = nil
end
for part, orig in pairs(skinOriginals) do
if part then
if orig.Type == "Clothing" and orig.Parent then
pcall(function() part.Parent = orig.Parent end)
elseif orig.Type == "BasePart" and part.Parent then
pcall(function()
part.Material = orig.Material
part.Color = orig.Color
part.Transparency = orig.Transparency
if part:IsA("MeshPart") and orig.TextureID then part.TextureID = orig.TextureID end
end)
elseif orig.Type == "SpecialMesh" and part.Parent then
pcall(function() part.TextureId = orig.TextureId end)
end
end
end
skinOriginals = {}
end
end
})

XocoSkinSection:AddLabel("Paint Color"):AddColorPicker("CustomSkinColor", {
Default = Color3.fromRGB(255, 255, 255),
Title = L("Цвет краски", "Paint Color"),
})

XocoSkinSection:AddDropdown("CustomSkinMaterial", {
Text = "Material",
Values = {"Default", "ForceField", "Neon", "Glass", "Foil", "Ice", "CorrodedMetal", "DiamondPlate", "Wood", "WoodPlanks", "Marble", "Granite", "Slate", "Brick", "Fabric", "Sand", "Plastic", "SmoothPlastic", "Cobblestone", "Grass"},
Default = "Default",
Tooltip = L("Материал частей тела", "Body part material"),
})

LocalPlayer.CharacterAdded:Connect(function(char)
if Toggles.EnableCustomSkin and Toggles.EnableCustomSkin.Value then
task.wait(0.2)
applyXocoSkin()
end
end)
end

do
end

end
do
local MovementSection = Tabs.Movement:AddLeftGroupbox("Player Movement", "person-standing")

local function BypassPCLD()
for _, obj in ipairs(workspace:GetDescendants()) do
if obj.Name == "PCLD" or obj.Name == "PCLDPart" then
obj.Transparency = 1
obj.CanCollide = false
end
end
end
task.spawn(BypassPCLD)

workspace.DescendantAdded:Connect(function(obj)
if obj.Name == "PCLD" or obj.Name == "PCLDPart" then
task.wait()
obj.Transparency = 1
obj.CanCollide = false
end
end)

local function getRoot(char)
return char:FindFirstChild("HumanoidRootPart")
end

MovementSection:AddToggle("EnableFly", {
Text = "Fly (Bypass)",
Default = false,
Tooltip = L("Полет с обходом гравитации и скорости (CFrame)", "Flight bypassing gravity and speed (CFrame)"),
}):AddKeyPicker("FlyKeybind", {
Default = "None",
SyncToggleState = true,
Mode = "Toggle",
Text = "Fly Key",
NoUI = false,
})
flyHeight = nil 

MovementSection:AddSlider("FlySpeed", {
Text = "Fly Speed",
Default = 50,
Min = 10,
Max = 300,
Rounding = 0,
Compact = false,
Tooltip = L("Настройка скорости полета", "Flight speed setting"),
})

MovementSection:AddToggle("EnableSpeed", {
Text = "CFrame Speed",
Default = false,
Tooltip = L("Универсальный спидхак (работает везде)", "Universal speed hack (works everywhere)"),
})

MovementSection:AddSlider("SpeedValue", {
Text = "Speed Multiplier",
Default = 1,
Min = 1,
Max = 100,
Rounding = 1,
Compact = false,
Tooltip = L("Множитель скорости (CFrame)", "Speed multiplier (CFrame)"),
})

flyAntiKickConn = nil
local function startFlyAntiKick()
if flyAntiKickConn then return end
local rs = game:GetService("ReplicatedStorage")
local ce = rs:FindFirstChild("GameCorrectionEvents")
local notify = ce and ce:FindFirstChild("GameCorrectionsNotify")
local che = rs:FindFirstChild("CharacterEvents")
local struggle = che and che:FindFirstChild("Struggle")
if not notify then
Library:Notify(L("Fly Anti-Kick: событие детекта не найдено", "Fly Anti-Kick: detect event not found"), 4)
return
end
flyAntiKickConn = notify.OnClientEvent:Connect(function(reason)
if reason ~= "Flying" then return end
if not (Toggles.EnableFly and Toggles.EnableFly.Value) then return end
if struggle then pcall(function() struggle:FireServer(LocalPlayer) end) end
end)
end
local function stopFlyAntiKick()
if flyAntiKickConn then
flyAntiKickConn:Disconnect()
flyAntiKickConn = nil
end
end

flyStartedSticky = false

Toggles.EnableFly:OnChanged(function()
if Toggles.EnableFly.Value then
startFlyAntiKick()
if Toggles.EnableAntiKickShuriken and not Toggles.EnableAntiKickShuriken.Value then
flyStartedSticky = true
Toggles.EnableAntiKickShuriken:SetValue(true)
end
Library:Notify(L("Fly ВКЛ (anti-kick + sticky)", "Fly ON (anti-kick + sticky)"), 2)
else
stopFlyAntiKick()
if flyStartedSticky and Toggles.EnableAntiKickShuriken and Toggles.EnableAntiKickShuriken.Value then
Toggles.EnableAntiKickShuriken:SetValue(false)
end
flyStartedSticky = false
flyHeight = nil
local char = LocalPlayer.Character
local hrp = char and char:FindFirstChild("HumanoidRootPart")
local hum = char and char:FindFirstChild("Humanoid")
if hrp then
if hrp.Anchored then hrp.Anchored = false end
hrp.AssemblyLinearVelocity = Vector3.zero
if hrp:FindFirstChild("FlyVelocity") then hrp.FlyVelocity:Destroy() end
end
if hum then
hum.AutoRotate = true
end
Library:Notify(L("Fly ВЫКЛ", "Fly OFF"), 2)
end

end)

RunService.Heartbeat:Connect(function(delta)
local char = LocalPlayer.Character
if not char then return end
local root = getRoot(char)
local hum = char:FindFirstChild("Humanoid")
if not root or not hum then return end

if Toggles.EnableFly and Toggles.EnableFly.Value then
root.AssemblyLinearVelocity = Vector3.zero
if hum.PlatformStand then hum.PlatformStand = false end

local cam = workspace.CurrentCamera
if not cam then return end
local moveDir = Vector3.new(0, 0, 0)

if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + cam.CFrame.LookVector end
if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - cam.CFrame.LookVector end
if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + cam.CFrame.RightVector end
if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - cam.CFrame.RightVector end
if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDir = moveDir + Vector3.new(0, 1, 0) end
if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) or UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then moveDir = moveDir - Vector3.new(0, 1, 0) end

local speed = Options.FlySpeed and Options.FlySpeed.Value or 50

if moveDir.Magnitude > 0 then
moveDir = moveDir.Unit
local newPos = root.CFrame.Position + (moveDir * speed * delta)
root.CFrame = CFrame.new(newPos) * (root.CFrame - root.CFrame.Position)
flyHeight = newPos
else
if not flyHeight then flyHeight = root.CFrame.Position end
root.CFrame = CFrame.new(flyHeight) * (root.CFrame - root.CFrame.Position)
end
else
flyHeight = nil
if root.Anchored then root.Anchored = false end
end
if Toggles.EnableSpeed and Toggles.EnableSpeed.Value and not (Toggles.EnableFly and Toggles.EnableFly.Value) then
if hum.MoveDirection.Magnitude > 0 then
root.CFrame = root.CFrame + (hum.MoveDirection * Options.SpeedValue.Value * delta * 20)
end
end
end)

MovementSection:AddToggle("EnableNoClip", {
Text = "NoClip",
Default = false,
Tooltip = "П��охождение сквозь стены",
})

MovementSection:AddToggle("EnableWaterWalk", {
Text = "Water Walk",
Default = false,
Tooltip = L("Делает воду за картой твердой, позволяя ходить по ней", "Makes water behind map solid, allowing you to walk on it"),
Callback = function(Value)
pcall(function()
local model = workspace:FindFirstChild("Map")
model = model and model:FindFirstChild("AlwaysHereTweenedObjects")
model = model and model:FindFirstChild("Ocean")
model = model and model:FindFirstChild("Object")
model = model and model:FindFirstChild("ObjectModel")

if model then
for _, obj in pairs(model:GetChildren()) do
if obj.Name == "Ocean" and obj:IsA("BasePart") then
obj.CanCollide = Value
end
end
end
end)
end
})

RunService.Heartbeat:Connect(function()
if Toggles.EnableWaterWalk and Toggles.EnableWaterWalk.Value then
pcall(function()
local model = workspace:FindFirstChild("Map")
model = model and model:FindFirstChild("AlwaysHereTweenedObjects")
model = model and model:FindFirstChild("Ocean")
model = model and model:FindFirstChild("Object")
model = model and model:FindFirstChild("ObjectModel")

if model then
for _, obj in pairs(model:GetChildren()) do
if obj.Name == "Ocean" and obj:IsA("BasePart") and not obj.CanCollide then
obj.CanCollide = true
end
end
end
end)
end
end)

RunService.Stepped:Connect(function()
if Toggles.EnableNoClip and Toggles.EnableNoClip.Value then
local char = LocalPlayer.Character
if char then
for _, part in ipairs(char:GetDescendants()) do
if part:IsA("BasePart") and part.CanCollide then
part.CanCollide = false
end
end
end
end
end)

originalJumpPower = 50
originalJumpHeight = 7.2
originalUseJumpPower = false
wasJumpEnabled = false

MovementSection:AddToggle("EnableJumpPower", {
Text = "Custom Jump Power",
Default = false,
Tooltip = L("Включить кастомную высоту прыжка", "Enable custom jump height"),
})

MovementSection:AddSlider("JumpPowerValue", {
Text = "Jump Power Amount",
Default = 50,
Min = 50,
Max = 300,
Rounding = 0,
Compact = false,
Tooltip = L("Настройка высоты прыжка", "Jump height setting"),
})

Toggles.EnableJumpPower:OnChanged(function()
local char = LocalPlayer.Character
local hum = char and char:FindFirstChild("Humanoid")
if not hum then return end

if Toggles.EnableJumpPower.Value then
if not wasJumpEnabled then
originalJumpPower = hum.JumpPower
originalJumpHeight = hum.JumpHeight
originalUseJumpPower = hum.UseJumpPower
wasJumpEnabled = true
end
hum.UseJumpPower = true
hum.JumpPower = Options.JumpPowerValue.Value
else
if wasJumpEnabled then
hum.UseJumpPower = originalUseJumpPower
hum.JumpPower = originalJumpPower
hum.JumpHeight = originalJumpHeight
wasJumpEnabled = false
end
end
end)

Options.JumpPowerValue:OnChanged(function()
if Toggles.EnableJumpPower.Value then
local char = LocalPlayer.Character
local hum = char and char:FindFirstChild("Humanoid")
if hum then
hum.JumpPower = Options.JumpPowerValue.Value
end
end
end)

MovementSection:AddToggle("EnableInfiniteJump", {
Text = "Infinite Jump",
Default = false,
Tooltip = L("Бесконечный прыжок: можно прыгать сколько угодно раз в воздухе", "Infinite jump: jump as many times in the air"),
})

UserInputService.JumpRequest:Connect(function()
if not (Toggles.EnableInfiniteJump and Toggles.EnableInfiniteJump.Value) then return end
pcall(function()
local char = LocalPlayer.Character
local hum = char and char:FindFirstChildOfClass("Humanoid")
if not hum or hum.Health <= 0 then return end
if Toggles.EnableJumpPower and Toggles.EnableJumpPower.Value and Options.JumpPowerValue then
hum.UseJumpPower = true
hum.JumpPower = Options.JumpPowerValue.Value
end
hum:ChangeState(Enum.HumanoidStateType.Jumping)
hum.Jump = true
end)
end)

jerkOffAnimTrack = nil
jerkOffActive = false

function stopJerkOff()
jerkOffActive = false
if jerkOffAnimTrack then
jerkOffAnimTrack:Stop()
jerkOffAnimTrack = nil
end
end

function startJerkOff()
if jerkOffActive then return end
local char = LocalPlayer.Character
if not char then return end
local hum = char:FindFirstChildOfClass("Humanoid")
if not hum then return end
local animator = hum:FindFirstChild("Animator")
if not animator then
animator = Instance.new("Animator")
animator.Parent = hum
end

local anim = Instance.new("Animation")
anim.AnimationId = "rbxassetid://168268306"

jerkOffAnimTrack = animator:LoadAnimation(anim)
jerkOffAnimTrack.Priority = Enum.AnimationPriority.Action
jerkOffAnimTrack:Play()
jerkOffActive = true

task.spawn(function()
while jerkOffActive and jerkOffAnimTrack and jerkOffAnimTrack.IsPlaying do
task.wait(0.1)
pcall(function() jerkOffAnimTrack.TimePosition = 0.3 end)
end
end)
end

MovementSection:AddToggle("EnableJerkOff", {
Text = "Jerk Off",
Default = false,
Tooltip = L("Анимация дрочки (видна всем игрокам на сервере)", "Dancing animation (visible to all players)"),
Callback = function(Value)
if Value then
startJerkOff()
else
stopJerkOff()
end
end
}):AddKeyPicker("JerkOffKeybind", {
Default = "None",
SyncToggleState = true,
Mode = "Toggle",
Text = "Jerk Off Key",
NoUI = false,
})

standBehindConn = nil
standBehindTarget = nil

local function standBehindList()
local list = {}
for _, p in ipairs(Players:GetPlayers()) do
if p ~= LocalPlayer then
table.insert(list, p.Name)
end
end
return list
end

MovementSection:AddDropdown("StandBehindTarget", {
Text = L("Stand Behind — Цель", "Stand Behind — Target"),
Values = standBehindList(),
Default = (standBehindList())[1] or nil,
Tooltip = L("Выбери игрока из списка", "Select a player from the list"),
Callback = function(Value)
standBehindTarget = Value
end,
})

local function standBehindRefresh()
if not Options.StandBehindTarget then return end
local list = standBehindList()
pcall(function() Options.StandBehindTarget:SetValues(list) end)
local cur = Options.StandBehindTarget.Value
local stillHere = false
for _, n in ipairs(list) do if n == cur then stillHere = true break end end
if (not stillHere) and #list > 0 then
pcall(function() Options.StandBehindTarget:SetValue(list[1]) end)
end
standBehindTarget = Options.StandBehindTarget.Value
end

Players.PlayerAdded:Connect(function() task.wait(0.3) standBehindRefresh() end)
Players.PlayerRemoving:Connect(function() task.defer(standBehindRefresh) end)

local function standBehindStop()
if standBehindConn then standBehindConn:Disconnect(); standBehindConn = nil end
local char = LocalPlayer.Character
if not char then return end
for _, p in pairs(char:GetChildren()) do
if p:IsA("BasePart") then
p.Velocity = Vector3.zero
p.RotVelocity = Vector3.zero
end
end
end

local function standBehindStart()
standBehindStop()
standBehindConn = game:GetService("RunService").Heartbeat:Connect(function()
local char = LocalPlayer.Character
if not char then return end
local hrp = char:FindFirstChild("HumanoidRootPart")
if not hrp then return end
local targetName = standBehindTarget or (Options.StandBehindTarget and Options.StandBehindTarget.Value)
if not targetName then return end
local targetPlayer = Players:FindFirstChild(targetName)
if not targetPlayer or not targetPlayer.Character then return end
local tHrp = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
if not tHrp then return end
hrp.Velocity = Vector3.zero
hrp.RotVelocity = Vector3.zero
hrp.CFrame = tHrp.CFrame * CFrame.new(0, 0, 2)
end)
end

MovementSection:AddToggle("EnableStandBehind", {
Text = "Stand Behind",
Default = false,
Tooltip = L("Встать за выбранным игроком (следует за ним)", "Stand behind selected player (follows them)"),
Callback = function(Value)
if Value then
standBehindStart()
else
standBehindStop()
end
end
}):AddKeyPicker("StandBehindKeybind", {
Default = "None",
SyncToggleState = true,
Mode = "Toggle",
Text = "Stand Behind Key",
NoUI = false,
})

spiderConn = nil
spiderSpeed = 30

local function spiderStop()
if spiderConn then spiderConn:Disconnect(); spiderConn = nil end
end

local function spiderStart()
local char = LocalPlayer.Character
if not char then return end
local hrp = char:FindFirstChild("HumanoidRootPart")
if not hrp then return end
spiderConn = game:GetService("RunService").Heartbeat:Connect(function(dt)
local c = LocalPlayer.Character
if not c then return end
local h = c:FindFirstChild("HumanoidRootPart")
local hum = c:FindFirstChildOfClass("Humanoid")
if not h or not hum then return end
if not Toggles.EnableSpider or not Toggles.EnableSpider.Value then return end
local cam = workspace.CurrentCamera
local moveDir = Vector3.zero
local moving = false
local keys = {W=false,A=false,S=false,D=false}
pcall(function()
keys.W = UserInputService:IsKeyDown(Enum.KeyCode.W)
keys.A = UserInputService:IsKeyDown(Enum.KeyCode.A)
keys.S = UserInputService:IsKeyDown(Enum.KeyCode.S)
keys.D = UserInputService:IsKeyDown(Enum.KeyCode.D)
end)
if keys.W or keys.S or keys.A or keys.D then
moving = true
local camCF = cam.CFrame
local forward = camCF.LookVector * ((keys.W and 1 or 0) - (keys.S and 1 or 0))
local right = camCF.RightVector * ((keys.D and 1 or 0) - (keys.A and 1 or 0))
moveDir = (forward + right)
if moveDir.Magnitude > 0 then moveDir = moveDir.Unit end
end
local speed = spiderSpeed
if Options.SpiderSpeed then speed = Options.SpiderSpeed.Value end
local rayParams = RaycastParams.new()
rayParams.FilterType = Enum.RaycastFilterType.Blacklist
rayParams.FilterDescendantsInstances = {c}
local ray = workspace:Raycast(h.Position, h.CFrame.LookVector * 2, rayParams)
local wall = ray and ray.Instance
if wall then
hum.PlatformStand = false
hum.Sit = false
local climbDir = Vector3.new(0, 1, 0)
if moving then
climbDir = (moveDir + Vector3.new(0, 1, 0)).Unit
end
h.Velocity = climbDir * speed
h.CFrame = CFrame.new(h.Position, h.Position + wall.Normal * -1)
elseif moving then
local floorRay = workspace:Raycast(h.Position, Vector3.new(0, -4, 0), rayParams)
if not floorRay then
h.Velocity = moveDir * speed + Vector3.new(0, hum.WalkSpeed * 0.5, 0)
end
end
end)
end

MovementSection:AddToggle("EnableSpider", {
Text = "Spider",
Default = false,
Tooltip = L("Лазание по стенам (WASD + автоприлипание к стенам)", "Wall climbing (WASD + auto-stick to walls)"),
Callback = function(Value)
if Value then
spiderStart()
else
spiderStop()
end
end
}):AddKeyPicker("SpiderKeybind", {
Default = "None",
SyncToggleState = true,
Mode = "Toggle",
Text = "Spider Key",
NoUI = false,
})

MovementSection:AddSlider("SpiderSpeed", {
Text = "Spider Speed",
Default = 30,
Min = 10,
Max = 200,
Rounding = 0,
Compact = false,
Tooltip = L("Скорость лазания по стенам", "Wall climb speed"),
})
playBangActive = false
bangAnimTrack = nil
bangAnimId = "rbxassetid://148840371"
bangSpeed = 10

local function bangStop()
playBangActive = false
if bangAnimTrack then
bangAnimTrack:Stop()
bangAnimTrack = nil
end
end

local function bangStart()
local char = LocalPlayer.Character
if not char then return end
local hum = char:FindFirstChildOfClass("Humanoid")
if not hum then return end
local animator = hum:FindFirstChildOfClass("Animator")
if not animator then
animator = Instance.new("Animator")
animator.Parent = hum
end
local anim = Instance.new("Animation")
anim.AnimationId = bangAnimId
bangAnimTrack = animator:LoadAnimation(anim)
bangAnimTrack.Priority = Enum.AnimationPriority.Action
bangAnimTrack:Play()
local speed = (Options.BangSpeed and Options.BangSpeed.Value) or 10
bangAnimTrack:AdjustSpeed(speed)
playBangActive = true
task.spawn(function()
while playBangActive do
task.wait(0.1)
if bangAnimTrack and bangAnimTrack.IsPlaying then
bangAnimTrack.TimePosition = 0.1
end
end
end)
end

MovementSection:AddToggle("EnableBang", {
Text = "Bang",
Default = false,
Tooltip = L("Анимация bang (как в XOCO / Infinite Yield)", "Bang animation (like XOCO / Infinite Yield)"),
Callback = function(Value)
if Value then
bangStart()
else
bangStop()
end
end
}):AddKeyPicker("BangKeybind", {
Default = "None",
SyncToggleState = true,
Mode = "Toggle",
Text = "Bang Key",
NoUI = false,
})

MovementSection:AddSlider("BangSpeed", {
Text = "Bang Speed",
Default = 10,
Min = 0.1,
Max = 20,
Rounding = 1,
Compact = false,
Tooltip = L("Скорость анимации Bang (как в XOCO: 1 = нормально, 10 = быстро)", "Bang animation speed (like XOCO: 1 = normal, 10 = fast)"),
})

ReplicatedStorage = game:GetService("ReplicatedStorage")
BlobFlySection = Tabs.Movement:AddRightGroupbox("Blob Fly", "bird")

do
TPCrosshairSection = Tabs.Movement:AddRightGroupbox("Teleport to Crosshair", "crosshair")

TPCrosshairSection:AddToggle("EnableTPCrosshair", {
Text = L("Телепорт к прицелу", "Teleport to Crosshair"),
Default = false,
Tooltip = L("Вкл/Выкл телепорт туда, куда смотришь", "Toggle teleport to where you look"),
}):AddKeyPicker("TPCrosshairKey", {
Default = "None",
SyncToggleState = false,
Mode = "Toggle",
Text = L("Кнопка ТП", "TP Button"),
NoUI = false,
})

TPCrosshairSection:AddSlider("TPCrosshairSmoothness", {
Text = L("Плавность ТП", "TP Smoothness"),
Default = 0,
Min = 0,
Max = 1,
Rounding = 2,
Compact = false,
Tooltip = L("0 — моментально; 1 — очень плавно", "0 = instant; 1 = very smooth"),
})

TweenService = game:GetService("TweenService")

function tpKeyMatches(input, boundKey)
if not boundKey then return false end
if input.KeyCode.Name == boundKey or input.UserInputType.Name == boundKey then return true end
if (boundKey == "MB1" or boundKey == "MouseButton1") and input.UserInputType == Enum.UserInputType.MouseButton1 then return true end
if (boundKey == "MB2" or boundKey == "MouseButton2") and input.UserInputType == Enum.UserInputType.MouseButton2 then return true end
if (boundKey == "MB4") and input.KeyCode and input.KeyCode.Name == "ButtonX1" then return true end
if (boundKey == "MB5") and input.KeyCode and input.KeyCode.Name == "ButtonX2" then return true end
return false
end

function doTPCrosshair()
if not (Toggles.EnableTPCrosshair and Toggles.EnableTPCrosshair.Value) then return end
local char = LocalPlayer.Character
local hrp = char and char:FindFirstChild("HumanoidRootPart")
local cam = workspace.CurrentCamera
if not (hrp and cam) then return end
local params = RaycastParams.new()
params.FilterType = Enum.RaycastFilterType.Exclude
params.FilterDescendantsInstances = { char }
local origin = cam.CFrame.Position
local dir = cam.CFrame.LookVector * 5000
local hit = workspace:Raycast(origin, dir, params)
local targetPos = hit and (hit.Position + Vector3.new(0, 3, 0)) or (origin + cam.CFrame.LookVector * 200)
local smooth = Options.TPCrosshairSmoothness and Options.TPCrosshairSmoothness.Value or 0
if smooth <= 0.05 then
hrp.CFrame = CFrame.new(targetPos)
else
TweenService:Create(hrp, TweenInfo.new(smooth, Enum.EasingStyle.Linear), { CFrame = CFrame.new(targetPos) }):Play()
end
end

UserInputService.InputBegan:Connect(function(input, gpe)
if gpe then return end
if Options.TPCrosshairKey and tpKeyMatches(input, Options.TPCrosshairKey.Value) then
doTPCrosshair()
end
end)
end

blobFlyBV = nil
blobFlyBG = nil

function spawnBlobman()
local char = LocalPlayer.Character
local hrp = char and char:FindFirstChild("HumanoidRootPart")
local spawnCF = hrp and hrp.CFrame or CFrame.new(0, 100, 0)
pcall(function()
ReplicatedStorage.MenuToys.SpawnToyRemoteFunction:InvokeServer("CreatureBlobman", spawnCF, Vector3.zero)
end)
local folder = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
if not folder then
folder = workspace:WaitForChild(LocalPlayer.Name .. "SpawnedInToys", 5)
end
if folder then
local blob = folder:WaitForChild("CreatureBlobman", 5)
if blob then
Library:Notify(L("Blobman заспавнен!", "Blobman spawned!"), 3)
return blob
end
end
return nil
end

function GetBlobRoot()
local char = LocalPlayer.Character
local hum = char and char:FindFirstChild("Humanoid")
if hum and hum.SeatPart and hum.SeatPart.Parent and hum.SeatPart.Parent.Name == "CreatureBlobman" then
return hum.SeatPart.Parent:FindFirstChild("HumanoidRootPart") or hum.SeatPart.Parent.PrimaryPart
end
local folder = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
if folder then
local blob = folder:FindFirstChild("CreatureBlobman")
if blob then
return blob:FindFirstChild("HumanoidRootPart") or blob.PrimaryPart
end
end
return nil
end

bmKickingInProgress = false 

function sitOnBlobman()
local char = LocalPlayer.Character
local hrp = char and char:FindFirstChild("HumanoidRootPart")
local hum = char and char:FindFirstChild("Humanoid")
if not hrp or not hum then return false end
local folder = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
local blob = folder and folder:FindFirstChild("CreatureBlobman")
if blob and not bmKickingInProgress then
local seat = blob:FindFirstChild("VehicleSeat")
if not seat or (seat and seat.Occupant and seat.Occupant ~= hum) then
pcall(function()
local destroyRem = ReplicatedStorage:FindFirstChild("MenuToys")
destroyRem = destroyRem and destroyRem:FindFirstChild("DestroyToy")
if destroyRem then destroyRem:FireServer(blob) end
end)
local dw = tick()
while folder and folder:FindFirstChild("CreatureBlobman") and tick() - dw < 2 do
task.wait(0.05)
end
blob = nil
end
end
if not blob then
blob = spawnBlobman()
task.wait(0.5)
end
if not blob then return false end
local seat = blob:WaitForChild("VehicleSeat", 5)
if not seat then return false end
if seat.Occupant and seat.Occupant ~= hum then
pcall(function()
local destroyRem = ReplicatedStorage:FindFirstChild("MenuToys")
destroyRem = destroyRem and destroyRem:FindFirstChild("DestroyToy")
if destroyRem then destroyRem:FireServer(blob) end
end)
task.wait(0.3)
blob = spawnBlobman()
if not blob then return false end
seat = blob:WaitForChild("VehicleSeat", 5)
if not seat then return false end
end
local t = tick()
repeat
if not hum.SeatPart then
hrp.CFrame = seat.CFrame + Vector3.new(0, 1, 0)
hrp.AssemblyLinearVelocity = Vector3.zero
seat:Sit(hum)
end
RunService.Heartbeat:Wait()
until hum.SeatPart == seat or tick() - t > 3
return hum.SeatPart == seat
end

function cleanupBlobFly()
if blobFlyBV then blobFlyBV:Destroy(); blobFlyBV = nil end
if blobFlyBG then blobFlyBG:Destroy(); blobFlyBG = nil end
end

BlobFlySection:AddToggle("EnableBlobFly", {
Text = "Blob Fly",
Default = false,
Tooltip = L("Спавнит CreatureBlobman, сажает тебя на него и даёт летать", "Spawns CreatureBlobman, seats you on it and lets you fly"),
})

blobFlySpeedMode = "Normal"
BlobFlySection:AddDropdown("BlobFlySpeedMode", {
Text = L("Скорость", "Speed"),
Values = {"Low", "Normal", "Fast"},
Default = "Normal",
Multi = false,
Tooltip = L("Low = 40 (медленнее) | Normal = 60 (база) | Fast = 120 (в 2 раза быстрее)", "Low = 40 (slower) | Normal = 60 (base) | Fast = 120 (2x faster)"),
Callback = function(v) blobFlySpeedMode = v end,
})

BlobFlySection:AddSlider("BlobFlySpeed", {
Text = "Blob Fly Speed",
Default = 60,
Min = 10,
Max = 300,
Rounding = 0,
Compact = false,
Tooltip = L("Скорость полёта блоба", "Blob flight speed"),
})

BlobFlySection:AddButton({
Text = L("Заспавнить + сесть на блоба", "Spawn + Sit on Blob"),
Tooltip = L("Ручной спавн и посадка (если авто не сработал)", "Manual spawn and sit (if auto didn't work)"),
Func = function()
task.spawn(sitOnBlobman)
end,
})

Toggles.EnableBlobFly:OnChanged(function()
if Toggles.EnableBlobFly.Value then
task.spawn(function()
local ok = sitOnBlobman()
if not ok then
Library:Notify(L("Не удалось сесть на блоба — нажми кнопку ниже", "Failed to sit on blob — press button below"), 4)
end
end)
else
cleanupBlobFly()
end
end)

RunService.Heartbeat:Connect(function()
if not (Toggles.EnableBlobFly and Toggles.EnableBlobFly.Value) then
if blobFlyBV or blobFlyBG then cleanupBlobFly() end
return
end
local root = GetBlobRoot()
if not root then return end

if not root:FindFirstChild("BlobFlyVelocity") then
blobFlyBV = Instance.new("BodyVelocity")
blobFlyBV.Name = "BlobFlyVelocity"
blobFlyBV.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
blobFlyBV.P = 10000
blobFlyBV.Parent = root
else
blobFlyBV = root.BlobFlyVelocity
end
if not root:FindFirstChild("BlobFlyGyro") then
blobFlyBG = Instance.new("BodyGyro")
blobFlyBG.Name = "BlobFlyGyro"
blobFlyBG.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
blobFlyBG.P = 20000
blobFlyBG.D = 100
blobFlyBG.Parent = root
else
blobFlyBG = root.BlobFlyGyro
end

cam = workspace.CurrentCamera
moveDir = Vector3.zero
if UserInputService:IsKeyDown(Enum.KeyCode.W) then moveDir = moveDir + cam.CFrame.LookVector end
if UserInputService:IsKeyDown(Enum.KeyCode.S) then moveDir = moveDir - cam.CFrame.LookVector end
if UserInputService:IsKeyDown(Enum.KeyCode.A) then moveDir = moveDir - cam.CFrame.RightVector end
if UserInputService:IsKeyDown(Enum.KeyCode.D) then moveDir = moveDir + cam.CFrame.RightVector end
if UserInputService:IsKeyDown(Enum.KeyCode.Space) then moveDir = moveDir + Vector3.new(0, 1, 0) end
if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then moveDir = moveDir - Vector3.new(0, 1, 0) end

local blobBaseSpeed = Options.BlobFlySpeed and Options.BlobFlySpeed.Value or 60
local speedMulti = (blobFlySpeedMode == "Fast" and 2) or (blobFlySpeedMode == "Low" and 0.667) or 1
speed = blobBaseSpeed * speedMulti
if blobFlyBV then blobFlyBV.Velocity = moveDir * speed end
if blobFlyBG then blobFlyBG.CFrame = cam.CFrame end
end)

local HouseTPSection = Tabs.Movement:AddRightGroupbox("House Teleport", "house")

houseMapPoints = {
["Green House"] = CFrame.new(-548.305054, -2.45424771, 79.3213348),
["Pink House"] = CFrame.new(-475.493835, -2.70774508, -159.395279),
["Witch House"] = CFrame.new(270.225922, -2.48055029, 458.186493),
["Blue House"] = CFrame.new(501.939911, 88.2323608, -349.129211),
["China House"] = CFrame.new(545.441833, 128.004593, -99.4881439)
}

selectedHouseTP = nil

HouseTPSection:AddDropdown("HouseTPSelect", {
Text = L("Выбор дома", "House Select"),
Default = "Green House",
Values = {"Green House", "Pink House", "Witch House", "Blue House", "China House"},
Multi = false,
Tooltip = L("Выбери дом для телепортации", "Select house for teleportation"),
Callback = function(Value)
selectedHouseTP = Value
end
})

HouseTPSection:AddButton({
Text = L("Телепортироваться", "Teleport"),
Tooltip = L("Мгновенно телепортирует в выбранный дом", "Instantly teleports to selected house"),
Func = function()
local houseName = selectedHouseTP or (Options.HouseTPSelect and Options.HouseTPSelect.Value) or "Green House"
local cf = houseMapPoints[houseName]
if not cf then return end
local char = LocalPlayer.Character
if not char then return end
char:PivotTo(cf)
Library:Notify("Телепорт: " .. houseName, 2)
end
})

do
local fgLimbs = { "Torso", "Right Leg", "Left Leg", "Right Arm", "Left Arm", "Head" }
local fgSocketDefault = {
["Right Leg"] = Vector3.new(0, 1, 0),
["Left Leg"] = Vector3.new(0, 1, 0),
["Right Arm"] = Vector3.new(1, 0.5, 0),
["Left Arm"] = Vector3.new(-1, 0.5, 0),
Head = Vector3.new(0, 1, 0),
}
local fgPoses = {
Default = { ["Right Arm"] = { 0, 0, 0 }, ["Left Arm"] = { 0, 0, 0 }, ["Right Leg"] = { 0, 0, 0 }, ["Left Leg"] = { 0, 0, 0 }, Head = { 0, 0, 0 }, Torso = { 0, 0, 0 }, BodyPos = { 0, 0, -5 } },
["Shoulder-Sitting"] = { ["Right Arm"] = { 30, 0, -40 }, ["Left Arm"] = { 30, 0, 40 }, ["Right Leg"] = { 76, 7, 29 }, ["Left Leg"] = { 76, 7, -29 }, Head = { -20, 0, 0 }, Torso = { 8, 0, 0 }, BodyPos = { 0, 2.3, 1 } },
["JoJo Stand"] = { ["Right Arm"] = { -16, 0.3, -6.7 }, ["Left Arm"] = { -2, 0, 0 }, ["Right Leg"] = { -16, 22.5, -1 }, ["Left Leg"] = { -18.4, 40, -2 }, Head = { -21.9, 17.8, 0 }, Torso = { 9.6, 27.1, 0 }, BodyPos = { -2.8, 1.3, 2.75 } },
Blowjob = { ["Right Arm"] = { 90, 0, 0 }, ["Left Arm"] = { 90, 0, 0 }, ["Right Leg"] = { -95, 0, 15 }, ["Left Leg"] = { -95, 0, -15 }, Head = { -25, 0, 0 }, Torso = { 0, 0, 0 }, BodyPos = { 0, -2.1, -1.4 } },
["Standing Sex"] = { ["Right Arm"] = { 110, 0, -30 }, ["Left Arm"] = { 110, 0, 30 }, ["Right Leg"] = { 75, 0, 20 }, ["Left Leg"] = { 75, 0, -20 }, Head = { -10, 0, 0 }, Torso = { 0, 0, 0 }, BodyPos = { 0, 0, -1.2 } },
["Doggy-Pose Sex"] = { ["Right Arm"] = { 100, 0, 0 }, ["Left Arm"] = { 100, 0, 0 }, ["Right Leg"] = { 90, 0, 40 }, ["Left Leg"] = { 90, 0, -40 }, Head = { 90, 0, 0 }, Torso = { 100, -180, 0 }, BodyPos = { 0, -1.4, -1.4 } },
["Missionary Sex"] = { ["Right Arm"] = { 0, 0, 0 }, ["Left Arm"] = { 0, 0, 0 }, ["Right Leg"] = { 0, 0, 45 }, ["Left Leg"] = { 0, 0, -45 }, Head = { -45, 0, 0 }, Torso = { 90, 0, 0 }, BodyPos = { 0, -1.4, -1.4 } },
}
local fgRotation = {
["Right Arm"] = Vector3.zero,
["Left Arm"] = Vector3.zero,
["Right Leg"] = Vector3.zero,
["Left Leg"] = Vector3.zero,
Head = Vector3.zero,
Torso = Vector3.zero,
}
local fgOffset = {
["Right Leg"] = Vector3.new(0, 1, 0),
["Left Leg"] = Vector3.new(0, 1, 0),
["Right Arm"] = Vector3.new(1, 0.5, 0),
["Left Arm"] = Vector3.new(-1, 0.5, 0),
Head = Vector3.new(0, 1, 0),
}
local fgBodyOffset = CFrame.new(0, 0, -5)
local fgTransparency = 0
local fgEnabled = false
local fgTargets = {}
local fgSelectedPreset = "Default"
local fgActive = setmetatable({}, { __mode = "k" })

function OBV_IsPartOwner(part)
if not (part and part:IsA("BasePart")) then return false end
local head = part.Name == "HumanoidRootPart" and part.Parent and part.Parent:FindFirstChild("Head")
head = head and head:FindFirstChild("PartOwner")
if head and head.Value == LocalPlayer.Name then return true end
local po = part:FindFirstChild("PartOwner")
if po and po.Value == LocalPlayer.Name then return true end
po = part.Parent and part.Parent:FindFirstChild("PartOwner", true)
if po then return po.Value == LocalPlayer.Name end
return false
end

function OBV_Grab(part, posCf)
local char = LocalPlayer.Character
local root = char and (char:FindFirstChild("realHrp") or char:FindFirstChild("HumanoidRootPart"))
local hum = char and char:FindFirstChildOfClass("Humanoid")
if not root or not hum or hum.Health == 0 or not part or not part.Parent or not part:IsA("BasePart") then return end
local rs = game:GetService("ReplicatedStorage")
local ge = rs:FindFirstChild("GrabEvents")
local snw = ge and ge:FindFirstChild("SetNetworkOwner")
if not snw then return end
local grabCf = (posCf or root.CFrame) * CFrame.new(0, 1.593, 0)
pcall(function() snw:FireServer(part, CFrame.new(grabCf.Position, part.Position)) end)
end

function OBV_SpawnToy(toyName)
if type(toyName) ~= "string" or toyName == "" then return nil end
local char = LocalPlayer.Character
local root = char and (char:FindFirstChild("realHrp") or char:FindFirstChild("HumanoidRootPart"))
local hum = char and char:FindFirstChildOfClass("Humanoid")
if not root or not hum or hum.Health == 0 then return nil end
local rs = game:GetService("ReplicatedStorage")
local toys = rs:FindFirstChild("MenuToys")
local spawnRF = toys and toys:FindFirstChild("SpawnToyRemoteFunction")
if not spawnRF then return nil end
local canSpawn = LocalPlayer:FindFirstChild("CanSpawnToy")
local tWait = tick()
while canSpawn and canSpawn.Value == false and tick() - tWait < 10 do
task.wait()
end
if canSpawn and canSpawn.Value == false then return nil end
local myToys = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys") or workspace:WaitForChild(LocalPlayer.Name .. "SpawnedInToys", 5)
if not myToys then return nil end
local found = nil
local conn = myToys.ChildAdded:Connect(function(child)
if child.Name == toyName and not found then
found = child
if conn then conn:Disconnect() conn = nil end
end
end)
local spawnPos = CFrame.new(root.CFrame.Position + Vector3.new(0, 15, 0)) + Vector3.new(0, 0, 20)
local rot = Vector3.zero
local rotp = workspace:FindFirstChild("RotateOrientPart")
if rotp then rot = rotp.Rotation end
local t0 = tick()
task.spawn(function()
pcall(function() spawnRF:InvokeServer(toyName, spawnPos, rot) end)
end)
while not found and tick() - t0 < 1.3 do
game:GetService("RunService").Heartbeat:Wait()
end
if conn then conn:Disconnect() conn = nil end
return found
end

local obvOwning = false
function OBV_SNO(part, timeout, cond)
if obvOwning then return false end
local char = LocalPlayer.Character
local head = char and char:FindFirstChild("Head")
local root = char and (char:FindFirstChild("realHrp") or char:FindFirstChild("HumanoidRootPart"))
local hum = char and char:FindFirstChildOfClass("Humanoid")
local rag = hum and hum:FindFirstChild("Ragdolled")
if not (head and rag and root and hum and hum.Health > 0) then return false end
if not part or not part:IsA("BasePart") or not part.Parent then return false end
local spawnLoc = workspace:FindFirstChild("SpawnLocation")
if not spawnLoc then return false end
if (part.Position - spawnLoc.Position).Magnitude > 10000000 then return false end
if OBV_IsPartOwner(part) then return true end
if cond and not cond() then return false end
obvOwning = true
local far = (root.Position - part.Position).Magnitude >= 31
local realRoot, realHead = root.CFrame, head.CFrame
local moveCf = root.CFrame
local hbConn = nil
local rs = game:GetService("RunService")
if far then
rs:BindToRenderStep("OBLIVION_SNO_FREEZE", Enum.RenderPriority.Camera.Value - 1, function()
root.CFrame = realRoot
head.CFrame = realHead
end)
hbConn = rs.Heartbeat:Connect(function()
realRoot = root.CFrame
realHead = head.CFrame
root.CFrame = moveCf
head.CFrame = moveCf
end)
end
local ok = pcall(function()
local t0 = tick()
local t = timeout or 0.65
while hum.Health ~= 0 and part.Parent and root.Parent and not OBV_IsPartOwner(part) and (not cond or cond()) and tick() - t0 < t do
if far then
moveCf = part.CFrame - Vector3.new(0, 22, 0)
end
rs.Heartbeat:Wait()
OBV_Grab(part, far and moveCf or nil)
end
end)
if far then
pcall(function() rs:UnbindFromRenderStep("OBLIVION_SNO_FREEZE") end)
if hbConn then hbConn:Disconnect() hbConn = nil end
head.CFrame = realHead
root.CFrame = realRoot
end
obvOwning = false
if not ok then return false end
return OBV_IsPartOwner(part)
end

local FGGroup = Tabs.Movement:AddLeftGroupbox("Figure Grab", "mouse-pointer-click")

FGGroup:AddDropdown("FG_Targets", {
SpecialType = "Player",
ExcludeLocalPlayer = true,
Default = nil,
Multi = true,
AllowNull = true,
Searchable = true,
EnablePlayerImages = true,
Text = "Targets",
Tooltip = L("Выбери игроков — они станут фигурами и будут держаться в позе рядом с тобой", "Pick players — they become figures held in the pose next to you"),
Callback = function(values)
table.clear(fgTargets)
if values then
for player, state in pairs(values) do
if state then
fgTargets[player] = true
end
end
end
end,
})

FGGroup:AddToggle("FG_Enabled", {
Text = "Enabled",
Default = false,
Tooltip = L("Ставит выбранных игроков в позу и удерживает их рядом с тобой (Auto-Clicker Resonance)", "Poses the selected players and holds them next to you (Resonance Auto-Clicker)"),
Callback = function(value)
fgEnabled = value
end,
})

FGGroup:AddDropdown("FG_PosePreset", {
Text = L("Поза (пресет Resonance)", "Pose (Resonance preset)"),
Values = { "Blowjob", "Default", "Doggy-Pose Sex", "JoJo Stand", "Missionary Sex", "Shoulder-Sitting", "Standing Sex" },
Default = "Default",
Tooltip = L("7 поз Resonance — нажми Load Preset, чтобы применить к ползункам", "7 Resonance poses — press Load Preset to apply them to the sliders"),
Callback = function(name)
fgSelectedPreset = name
end,
})

FGGroup:AddButton({
Text = "Load Preset",
Tooltip = L("Применить выбранную позу к ползункам Offset/Rotation", "Apply the selected pose to the Offset/Rotation sliders"),
Func = function()
local pose = fgPoses[fgSelectedPreset or "Default"]
if not pose then return end
local bodyPos = pose.BodyPos
if bodyPos then
Options.FG_BodyX:SetValue(tonumber(bodyPos[1]) or 0)
Options.FG_BodyY:SetValue(tonumber(bodyPos[2]) or 0)
Options.FG_BodyZ:SetValue(tonumber(bodyPos[3]) or -5)
end
for limb, rot in pairs(fgRotation) do
local r = pose[limb]
if r then
Options["FG_RotX" .. limb]:SetValue(tonumber(r[1]) or 0)
Options["FG_RotY" .. limb]:SetValue(tonumber(r[2]) or 0)
Options["FG_RotZ" .. limb]:SetValue(tonumber(r[3]) or 0)
end
end
for limb in pairs(fgOffset) do
for _, axis in ipairs({ "X", "Y", "Z" }) do
local opt = Options["FG_Off" .. axis .. limb]
if opt then opt:SetValue(opt.Default) end
end
end
Library:Notify(L("Figure Grab: поза загружена", "Figure Grab: pose loaded"), 2)
end,
})

local FGConfig = Tabs.Movement:AddLeftGroupbox("Pose Config", "settings")

FGConfig:AddSlider("FG_BodyX", {
Text = "X Offset",
Default = 0,
Min = -10,
Max = 10,
Rounding = 1,
Compact = true,
Tooltip = L("Смещение фигуры относительно тебя (X)", "Figure offset relative to you (X)"),
Callback = function(value)
local o = fgBodyOffset
fgBodyOffset = CFrame.new(value, o.Y, o.Z)
end,
})

FGConfig:AddSlider("FG_BodyY", {
Text = "Y Offset",
Default = 0,
Min = -5,
Max = 20,
Rounding = 1,
Compact = true,
Tooltip = L("Смещение фигуры относительно тебя (Y)", "Figure offset relative to you (Y)"),
Callback = function(value)
local o = fgBodyOffset
fgBodyOffset = CFrame.new(o.X, value, o.Z)
end,
})

FGConfig:AddSlider("FG_BodyZ", {
Text = "Z Offset",
Default = -5,
Min = -10,
Max = 10,
Rounding = 1,
Compact = true,
Tooltip = L("Смещение фигуры относительно тебя (Z)", "Figure offset relative to you (Z)"),
Callback = function(value)
local o = fgBodyOffset
fgBodyOffset = CFrame.new(o.X, o.Y, value)
end,
})

FGConfig:AddSlider("FG_Transparency", {
Text = "Targets Transparency",
Default = 0,
Min = 0,
Max = 1,
Rounding = 2,
Compact = true,
Tooltip = L("Прозрачность фигур (только для тебя)", "Figure transparency (only for you)"),
Callback = function(value)
fgTransparency = value
end,
})

local FGTabbox = Tabs.Movement:AddRightTabbox("Pose Editor")
local FGOffsetTab = FGTabbox:AddTab("Offset", "move-diagonal")
local FGRotationTab = FGTabbox:AddTab("Rotation", "refresh-cw")

FGOffsetTab:AddDropdown("FG_LimbO", {
Text = "Select limb",
Values = { "Head", "Right Leg", "Right Arm", "Left Leg", "Left Arm" },
Default = "Head",
})

FGRotationTab:AddDropdown("FG_LimbR", {
Text = "Select limb",
Values = { "Head", "Torso", "Right Leg", "Right Arm", "Left Leg", "Left Arm" },
Default = "Head",
})

for _, limbName in ipairs({ "Head", "Torso", "Right Leg", "Right Arm", "Left Leg", "Left Arm" }) do
local rotBox = FGRotationTab:AddDependencyBox()
rotBox:SetupDependencies({ { Options.FG_LimbR, limbName } })
rotBox:AddSlider("FG_RotX" .. limbName, {
Text = "X Rotation",
Default = 0,
Min = -180,
Max = 180,
Rounding = 1,
Compact = true,
Callback = function(value)
local r = fgRotation[limbName]
fgRotation[limbName] = Vector3.new(value, r.Y, r.Z)
end,
})
rotBox:AddSlider("FG_RotY" .. limbName, {
Text = "Y Rotation",
Default = 0,
Min = -180,
Max = 180,
Rounding = 1,
Compact = true,
Callback = function(value)
local r = fgRotation[limbName]
fgRotation[limbName] = Vector3.new(r.X, value, r.Z)
end,
})
rotBox:AddSlider("FG_RotZ" .. limbName, {
Text = "Z Rotation",
Default = 0,
Min = -180,
Max = 180,
Rounding = 1,
Compact = true,
Callback = function(value)
local r = fgRotation[limbName]
fgRotation[limbName] = Vector3.new(r.X, r.Y, value)
end,
})
if limbName ~= "Torso" then
local offBox = FGOffsetTab:AddDependencyBox()
offBox:SetupDependencies({ { Options.FG_LimbO, limbName } })
offBox:AddSlider("FG_OffX" .. limbName, {
Text = "X Offset",
Default = fgOffset[limbName].X,
Min = -5,
Max = 5,
Rounding = 1,
Compact = true,
Callback = function(value)
local o = fgOffset[limbName]
fgOffset[limbName] = Vector3.new(value, o.Y, o.Z)
end,
})
offBox:AddSlider("FG_OffY" .. limbName, {
Text = "Y Offset",
Default = fgOffset[limbName].Y,
Min = -5,
Max = 5,
Rounding = 1,
Compact = true,
Callback = function(value)
local o = fgOffset[limbName]
fgOffset[limbName] = Vector3.new(o.X, value, o.Z)
end,
})
offBox:AddSlider("FG_OffZ" .. limbName, {
Text = "Z Offset",
Default = fgOffset[limbName].Z,
Min = -5,
Max = 5,
Rounding = 1,
Compact = true,
Callback = function(value)
local o = fgOffset[limbName]
fgOffset[limbName] = Vector3.new(o.X, o.Y, value)
end,
})
end
end

local function fgApplyTransparency(st)
for _, part in ipairs(st.Character:GetDescendants()) do
if part:IsA("BasePart") then
if st.Transparency[part] == nil then
st.Transparency[part] = part.LocalTransparencyModifier
end
part.LocalTransparencyModifier = fgTransparency
end
end
st.TransparencyValue = fgTransparency
end

local function fgCleanup(player)
local st = fgActive[player]
fgActive[player] = nil
if not st then return end
if st.TransparencyConn then
st.TransparencyConn:Disconnect()
st.TransparencyConn = nil
end
for part, saved in pairs(st.Transparency) do
if part.Parent then
part.LocalTransparencyModifier = saved
end
end
local char = st.Character
local torso = char and char:FindFirstChild("Torso")
local bp = torso and torso:FindFirstChild("AutoClickerBP")
local bg = torso and torso:FindFirstChild("AutoClickerBG")
if bp then bp:Destroy() end
if bg then bg:Destroy() end
if char then
for _, limbName in ipairs(fgLimbs) do
local limb = char:FindFirstChild(limbName)
if limb then
if (limbName == "Torso" or limbName == "Head") and limb:IsA("BasePart") then
limb.CanCollide = true
end
local socket = limb:FindFirstChild("BallSocketConstraint")
if socket then
socket.LimitsEnabled = true
socket.TwistLimitsEnabled = true
local def = fgSocketDefault[limbName]
if def and socket.Attachment1 then
socket.Attachment1.Position = def
end
end
local limbBg = limb:FindFirstChild("AutoClickerBG")
if limbBg then limbBg:Destroy() end
end
end
end
end

task.spawn(function()
while task.wait() do
for player in pairs(fgActive) do
if not fgTargets[player] then
fgCleanup(player)
end
end
if not fgEnabled then
for player in pairs(fgTargets) do
fgCleanup(player)
end
else
local char = LocalPlayer.Character
local root = char and (char:FindFirstChild("realHrp") or char:FindFirstChild("HumanoidRootPart"))
local myTorso = char and char:FindFirstChild("Torso")
local myHum = char and char:FindFirstChildOfClass("Humanoid")
if myHum then
local head = char:FindFirstChild("Head")
if head and head:IsA("BasePart") and head.CanCollide then head.CanCollide = false end
if myTorso and myTorso:IsA("BasePart") and myTorso.CanCollide then myTorso.CanCollide = false end
end
if root and myHum and myHum.Health > 0 and myTorso then
for player in pairs(fgTargets) do
local tChar = player.Character
local tHum = tChar and tChar:FindFirstChildOfClass("Humanoid")
local tTorso = tChar and tChar:FindFirstChild("Torso")
local st = fgActive[player]
if st and st.Character ~= tChar then
fgCleanup(player)
st = nil
end
if player ~= LocalPlayer and tHum and tHum.Health > 0 and tChar and tTorso then
if not st then
st = {
Character = tChar,
Parts = {},
RagdollParts = {},
Sockets = {},
SocketAttachments = {},
Gyros = {},
GyroTargets = {},
LimbOffsets = {},
RotationValues = {},
RotationFrames = {},
Transparency = {},
}
fgActive[player] = st
st.TransparencyConn = tChar.DescendantAdded:Connect(function(descendant)
if fgActive[player] ~= st or not descendant:IsA("BasePart") then return end
st.Transparency[descendant] = descendant.LocalTransparencyModifier
descendant.LocalTransparencyModifier = fgTransparency
end)
end
if st.TransparencyValue ~= fgTransparency then
fgApplyTransparency(st)
end
local bodyPos = st.BodyPosition
if not bodyPos or bodyPos.Parent ~= tTorso then
bodyPos = tTorso:FindFirstChild("AutoClickerBP") or Instance.new("BodyPosition")
bodyPos.Name = "AutoClickerBP"
bodyPos.D = 450
bodyPos.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bodyPos.Parent = tTorso
st.BodyPosition = bodyPos
st.BodyTarget = nil
end
local wantPos = (root.CFrame * fgBodyOffset).Position
if st.BodyTarget ~= wantPos then
bodyPos.Position = wantPos
st.BodyTarget = wantPos
end
local faceCf = CFrame.lookAt(tTorso.Position, Vector3.new(myTorso.Position.X, tTorso.Position.Y, myTorso.Position.Z))
for _, limbName in ipairs(fgLimbs) do
local limb = st.Parts[limbName]
if not limb or limb.Parent ~= tChar then
limb = tChar:FindFirstChild(limbName)
st.Parts[limbName] = limb
st.RagdollParts[limbName] = limb and limb:FindFirstChild("RagdollLimbPart")
st.Sockets[limbName] = limb and limb:FindFirstChild("BallSocketConstraint")
st.SocketAttachments[limbName] = st.Sockets[limbName] and st.Sockets[limbName].Attachment1
st.Gyros[limbName] = nil
st.GyroTargets[limbName] = nil
st.LimbOffsets[limbName] = nil
end
if limb then
if limb:IsA("BasePart") and limb.CanCollide then limb.CanCollide = false end
local ragPart = st.RagdollParts[limbName]
if not ragPart or not ragPart.Parent then
ragPart = limb:FindFirstChild("RagdollLimbPart")
st.RagdollParts[limbName] = ragPart
end
if ragPart and ragPart.CanCollide then ragPart.CanCollide = false end
local socket = st.Sockets[limbName]
if not socket or not socket.Parent then
socket = limb:FindFirstChild("BallSocketConstraint")
st.Sockets[limbName] = socket
st.SocketAttachments[limbName] = socket and socket.Attachment1
st.LimbOffsets[limbName] = nil
end
if socket then
if socket.LimitsEnabled then socket.LimitsEnabled = false end
if socket.TwistLimitsEnabled then socket.TwistLimitsEnabled = false end
local off = fgOffset[limbName]
local att = st.SocketAttachments[limbName]
if off and att and st.LimbOffsets[limbName] ~= off then
att.Position = off
st.LimbOffsets[limbName] = off
end
end
local gyro = st.Gyros[limbName]
if not gyro or gyro.Parent ~= limb then
gyro = limb:FindFirstChild("AutoClickerBG") or Instance.new("BodyGyro")
gyro.Name = "AutoClickerBG"
gyro.D = 500
gyro.P = 7500
gyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
gyro.Parent = limb
st.Gyros[limbName] = gyro
st.GyroTargets[limbName] = nil
end
local rot = fgRotation[limbName]
if st.RotationValues[limbName] ~= rot then
st.RotationValues[limbName] = rot
st.RotationFrames[limbName] = CFrame.Angles(math.rad(rot.X), math.rad(rot.Y), math.rad(rot.Z))
end
local frame = st.RotationFrames[limbName]
local gyroCf
if limbName == "Torso" then
gyroCf = faceCf * frame
else
gyroCf = tTorso.CFrame * frame
end
if st.GyroTargets[limbName] ~= gyroCf then
gyro.CFrame = gyroCf
st.GyroTargets[limbName] = gyroCf
end
end
end
local dv = root.Position - tTorso.Position
if dv:Dot(dv) < 900 or OBV_SNO(tTorso) then
OBV_Grab(tTorso)
local ragFlag = tHum:FindFirstChild("Ragdolled")
if ragFlag and not ragFlag.Value and OBV_RagdollTarget then
task.spawn(OBV_RagdollTarget, tChar)
end
end
end
end
end
end
end
end)
end

do
local BhopSection = Tabs.Movement:AddRightGroupbox("Bhop", "rabbit")

BhopSection:AddToggle("EnableBhop", {
Text = "Bhop (Bunny Hop)",
Default = false,
Tooltip = L("Начни двигаться — персонаж сам начнёт распрыгиваться", "Start moving — character begins bunnyhopping automatically"),
})

BhopSection:AddToggle("EnableBhopStrafe", {
Text = L("Strafes (разгон мышкой)", "Strafes (mouse boost)"),
Default = false,
Tooltip = L("В полёте крути мышку в сторону движения — набираешь скорость, как в CS", "In air, move mouse in movement direction — gain speed like in CS"),
})

BhopSection:AddSlider("BhopJumpPower", {
Text = L("Высота прыжка", "Jump Height"),
Default = 50,
Min = 10,
Max = 200,
Rounding = 0,
Compact = false,
})

BhopSection:AddSlider("BhopSpeedGain", {
Text = L("Разгон за прыжок", "Jump Boost"),
Default = 4,
Min = 0,
Max = 20,
Rounding = 1,
Compact = false,
Tooltip = L("Сколько скорости добавляется при каждом отталкивании от земли", "Speed added per ground bounce"),
})

BhopSection:AddSlider("BhopStrafeSpeed", {
Text = L("Сила стрейфа", "Strafe Force"),
Default = 0.05,
Min = 0.01,
Max = 0.3,
Rounding = 2,
Compact = false,
Tooltip = L("Насколько сильно мышка подгоняет скорость в полёте", "How strongly mouse adjusts speed in air"),
})

local bhopConn = nil
local bhopLastYaw = nil
local bhopCurSpeed = 0

local function stopBhop()
if bhopConn then
bhopConn:Disconnect()
bhopConn = nil
end
bhopLastYaw = nil
bhopCurSpeed = 0
end

Toggles.EnableBhop:OnChanged(function()
stopBhop()
if not Toggles.EnableBhop.Value then return end
bhopConn = RunService.Heartbeat:Connect(function()
local char = LocalPlayer.Character
local hum = char and char:FindFirstChildOfClass("Humanoid")
local root = char and char:FindFirstChild("HumanoidRootPart")
if not (hum and root) then return end

local jumpPower = Options.BhopJumpPower and Options.BhopJumpPower.Value or 50
local speedGain = Options.BhopSpeedGain and Options.BhopSpeedGain.Value or 4
local strafeGain = Options.BhopStrafeSpeed and Options.BhopStrafeSpeed.Value or 0.05
local maxSpeed = 200

local vel = root.AssemblyLinearVelocity
local flatVel = Vector3.new(vel.X, 0, vel.Z)

local moving = hum.MoveDirection.Magnitude > 0.05
if not moving and hum.FloorMaterial ~= Enum.Material.Air then
bhopCurSpeed = 0
end

if hum.FloorMaterial ~= Enum.Material.Air and moving then
local dir = hum.MoveDirection.Unit
bhopCurSpeed = math.max(flatVel.Magnitude, 16) + speedGain
if bhopCurSpeed > maxSpeed then bhopCurSpeed = maxSpeed end
root.AssemblyLinearVelocity = Vector3.new(dir.X * bhopCurSpeed, jumpPower, dir.Z * bhopCurSpeed)
end

if Toggles.EnableBhopStrafe and Toggles.EnableBhopStrafe.Value then
if hum.FloorMaterial == Enum.Material.Air then
local cam = workspace.CurrentCamera
if cam and flatVel.Magnitude > 5 then
local _, yaw = cam.CFrame:ToEulerAnglesYXZ()
if bhopLastYaw then
local delta = yaw - bhopLastYaw
if delta > math.pi then delta = delta - math.pi * 2 end
if delta < -math.pi then delta = delta + math.pi * 2 end
if math.abs(delta) > 0.0005 then
local rotated = CFrame.Angles(0, delta, 0) * flatVel
local newSpeed = rotated.Magnitude + math.abs(delta) * strafeGain * 100
if newSpeed > maxSpeed then newSpeed = maxSpeed end
local newFlat = rotated.Unit * newSpeed
root.AssemblyLinearVelocity = Vector3.new(newFlat.X, vel.Y, newFlat.Z)
end
end
bhopLastYaw = yaw
end
else
bhopLastYaw = nil
end
end
end)
end)

end

do
do
local SpinSection = Tabs.Movement:AddRightGroupbox("Spin Character", "rotate-3d")

SpinSection:AddToggle("EnableSpinChar", {
Text = "Spin Character",
Default = false,
Tooltip = L("Быстрое вращение персонажа. Видно только от 3 лица — камера не крутится", "Fast character spin. Visible only in 3rd person — camera doesn't rotate"),
})

SpinSection:AddSlider("SpinCharSpeed", {
Text = L("Скорость вращения", "Rotation Speed"),
Default = 5,
Min = 1,
Max = 50,
Rounding = 0,
Compact = false,
})

local spinCharConn = nil

local function stopSpinChar()
if spinCharConn then
spinCharConn:Disconnect()
spinCharConn = nil
end
end

Toggles.EnableSpinChar:OnChanged(function()
stopSpinChar()
if not Toggles.EnableSpinChar.Value then return end
spinCharConn = RunService.Heartbeat:Connect(function()
local char = LocalPlayer.Character
local root = char and char:FindFirstChild("HumanoidRootPart")
local head = char and char:FindFirstChild("Head")
if not root then return end
local cam = workspace.CurrentCamera
if cam and head then
local dist = (cam.CFrame.Position - head.Position).Magnitude
if dist < 1.5 then return end
end
local speed = Options.SpinCharSpeed and Options.SpinCharSpeed.Value or 5
root.CFrame = root.CFrame * CFrame.Angles(0, math.rad(speed), 0)
end)
end)

end
end

do
do
local AnimSection = Tabs.Movement:AddLeftGroupbox("Animations", "music-2")

AnimSection:AddDropdown("SelectedAnimation", {
Text = "Animation",
Values = {"Crazy", "Insane", "Collapse", "Zombie", "Moon Dance", "Full Punch", "Full Swing", "Arm Turbine", "Barrel Roll", "Arm Detach", "Insane Arms", "Spinner", "Crazy Slash", "Zombie Attack"},
Default = "Crazy",
Tooltip = L("Выбери анимацию", "Choose animation"),
})

local animTrack_Custom = nil

local animIds_Custom = {
["Crazy"]    = "rbxassetid://248263260",
["Insane"]   = "rbxassetid://35654637",
["Collapse"] = "rbxassetid://35154961",
["Zombie"]   = "rbxassetid://33796059",
["Moon Dance"] = "rbxassetid://45834924",
["Full Punch"] = "rbxassetid://204062532",
["Full Swing"] = "rbxassetid://218504594",
["Arm Turbine"]= "rbxassetid://259438880",
["Barrel Roll"]= "rbxassetid://136801964",
["Arm Detach"] = "rbxassetid://33169583",
["Insane Arms"]= "rbxassetid://27432691",
["Spinner"]    = "rbxassetid://754658275",
["Crazy Slash"]= "rbxassetid://674871189",
["Zombie Attack"] = "rbxassetid://708553116",
}

local function stopCustomAnim()
if animTrack_Custom then
animTrack_Custom:Stop()
animTrack_Custom = nil
end
end

local function playCustomAnim()
stopCustomAnim()
local char = LocalPlayer.Character
if not char then return end
local hum = char:FindFirstChildOfClass("Humanoid")
if not hum then return end
local animator = hum:FindFirstChildOfClass("Animator")
if not animator then
animator = Instance.new("Animator")
animator.Parent = hum
end
local animName = Options.SelectedAnimation and Options.SelectedAnimation.Value or "Crazy"
local animId = animIds_Custom[animName]
if not animId then return end
local anim = Instance.new("Animation")
anim.AnimationId = animId
animTrack_Custom = animator:LoadAnimation(anim)
animTrack_Custom.Priority = Enum.AnimationPriority.Action
animTrack_Custom.Looped = true
animTrack_Custom:Play()

task.spawn(function()
while Toggles.EnableAnimation and Toggles.EnableAnimation.Value and animTrack_Custom do
pcall(function()
if animTrack_Custom.TimePosition > 0.9 then
animTrack_Custom.TimePosition = 0.3
end
end)
task.wait(0.05)
end
end)
end

AnimSection:AddToggle("EnableAnimation", {
Text = "Play Animation",
Default = false,
Tooltip = L("Запустить выбранную анимацию", "Play selected animation"),
}):AddKeyPicker("AnimationKeybind", {
Default = "None",
SyncToggleState = true,
Mode = "Toggle",
Text = "Animation Key",
NoUI = false,
})

Toggles.EnableAnimation:OnChanged(function()
if Toggles.EnableAnimation.Value then
playCustomAnim()
else
stopCustomAnim()
end
end)

Options.SelectedAnimation:OnChanged(function()
if Toggles.EnableAnimation and Toggles.EnableAnimation.Value then
playCustomAnim()
end
end)
end

end
do
local TractorSpeedSection = Tabs.Movement:AddLeftGroupbox("Tractor Speed", "gauge")

local tractorState = {
enabled = false,
nitroActive = false,
conn = nil,
gyro = nil,
speed = 0,
}

TractorSpeedSection:AddToggle("EnableTractorSpeed", {
Text = "Speed Tractor",
Default = false,
Tooltip = "Увеличивает скорость трактора (TractorGreen/Red/Orange).\nНитро — удерживайте бинд ниже.",
Callback = function(v)
tractorState.enabled = v
if v then
tractorState.gyro = nil
tractorState.speed = 0
tractorState.conn = RunService.Heartbeat:Connect(function(dt)
if not tractorState.enabled then
if tractorState.gyro then tractorState.gyro:Destroy() tractorState.gyro = nil end
tractorState.speed = 0
return
end
local char = LocalPlayer.Character
local hum = char and char:FindFirstChildOfClass("Humanoid")
if not hum then return end
local seatPart = hum.SeatPart
if not seatPart or not seatPart:IsA("VehicleSeat") then
if tractorState.gyro then tractorState.gyro:Destroy() tractorState.gyro = nil end
tractorState.speed = 0
return
end
local model = seatPart.Parent
if not model then return end
local name = model.Name
if name ~= "TractorGreen" and name ~= "TractorRed" and name ~= "TractorOrange" then
if tractorState.gyro then tractorState.gyro:Destroy() tractorState.gyro = nil end
tractorState.speed = 0
return
end
local root = model.PrimaryPart or seatPart
if not root then return end

if not tractorState.gyro or tractorState.gyro.Parent ~= root then
if tractorState.gyro then tractorState.gyro:Destroy() end
local gyro = Instance.new("BodyGyro")
gyro.MaxTorque = Vector3.new(1e6, 0, 1e6)
gyro.P = 5e4
gyro.D = 5e3
gyro.CFrame = root.CFrame
gyro.Parent = root
tractorState.gyro = gyro
end
local gyroObj = tractorState.gyro
gyroObj.CFrame = CFrame.new(root.Position, root.Position + Vector3.new(root.CFrame.LookVector.X, 0, root.CFrame.LookVector.Z).Unit)

root.AssemblyAngularVelocity = Vector3.new(0, root.AssemblyAngularVelocity.Y, 0)

local vel = root.AssemblyLinearVelocity
local flatVel = Vector3.new(vel.X, 0, vel.Z)
local throttle = seatPart.Throttle
local pressing = math.abs(throttle) > 0

local baseSpeed = Options.TractorBaseSpeed and Options.TractorBaseSpeed.Value or 100
local nitroSpeed = Options.TractorNitroSpeed and Options.TractorNitroSpeed.Value or 300
local targetSpeed = pressing and (tractorState.nitroActive and nitroSpeed or baseSpeed) or 0

tractorState.speed = tractorState.speed + (targetSpeed - tractorState.speed) * math.clamp(dt * 4, 0, 1)

if not pressing then
local friction = flatVel.Magnitude > 0.5 and flatVel.Unit * -1 * 80 * dt or Vector3.zero
root.AssemblyLinearVelocity = vel + friction
elseif tractorState.speed > 0.5 then
local seatLook = Vector3.new(seatPart.CFrame.LookVector.X, 0, seatPart.CFrame.LookVector.Z).Unit
local dir = throttle > 0 and seatLook or -seatLook
root.AssemblyLinearVelocity = dir * tractorState.speed + Vector3.new(0, vel.Y, 0)
end
end)
else
if tractorState.conn then
tractorState.conn:Disconnect()
tractorState.conn = nil
end
if tractorState.gyro then tractorState.gyro:Destroy() tractorState.gyro = nil end
tractorState.speed = 0
end
end,
})

TractorSpeedSection:AddSlider("TractorBaseSpeed", {
Text = "Base Speed",
Default = 15,
Min = 50,
Max = 200,
Rounding = 0,
Compact = false,
Tooltip = L("Базовая скорость трактора", "Base tractor speed"),
})

TractorSpeedSection:AddSlider("TractorNitroSpeed", {
Text = "Nitro Speed",
Default = 300,
Min = 150,
Max = 500,
Rounding = 0,
Compact = false,
Tooltip = L("Скорость при удержании Nitro", "Speed while holding Nitro"),
})

TractorSpeedSection:AddToggle("EnableTractorNitro", {
Text = "Tractor Nitro",
Default = false,
NoUI = true,
}):AddKeyPicker("NitroKey", {
Default = "LeftShift",
SyncToggleState = false,
Mode = "Hold",
Text = "Nitro Key (Hold)",
NoUI = false,
})

TractorSpeedSection:AddToggle("EnableTractorJump", {
Text = "Tractor Jump",
Default = false,
NoUI = true,
}):AddKeyPicker("TractorJumpKey", {
Default = "Space",
SyncToggleState = false,
Mode = "Toggle",
Text = "Jump Key",
NoUI = false,
})

UserInputService.InputBegan:Connect(function(input, gpe)
if gpe then return end
local boundKey = Options.TractorJumpKey and Options.TractorJumpKey.Value
if not boundKey then return end
local keyMatch = input.KeyCode.Name == boundKey or input.UserInputType.Name == boundKey
or (boundKey == "MB4" and input.KeyCode and input.KeyCode.Name == "ButtonX1")
or (boundKey == "MB5" and input.KeyCode and input.KeyCode.Name == "ButtonX2")
if not keyMatch then return end
local char = LocalPlayer.Character
local hum = char and char:FindFirstChildOfClass("Humanoid")
if not hum or not hum.SeatPart or not hum.SeatPart:IsA("VehicleSeat") then return end
local seatPart = hum.SeatPart
local model = seatPart.Parent
if not model then return end
local name = model.Name
if name ~= "TractorGreen" and name ~= "TractorRed" and name ~= "TractorOrange" then return end
local root = model.PrimaryPart or seatPart
if not root then return end
root.AssemblyLinearVelocity = Vector3.new(
root.AssemblyLinearVelocity.X,
100,
root.AssemblyLinearVelocity.Z
)
end)

local function checkNitro(input, isHeld)
local boundKey = Options.NitroKey and Options.NitroKey.Value
if not boundKey then return end
if input.KeyCode.Name == boundKey or input.UserInputType.Name == boundKey then
tractorState.nitroActive = isHeld
elseif boundKey == "MB4" and input.KeyCode and input.KeyCode.Name == "ButtonX1" then
tractorState.nitroActive = isHeld
elseif boundKey == "MB5" and input.KeyCode and input.KeyCode.Name == "ButtonX2" then
tractorState.nitroActive = isHeld
end
end

UserInputService.InputBegan:Connect(function(input, gpe)
if gpe then return end
checkNitro(input, true)
end)

UserInputService.InputEnded:Connect(function(input)
checkNitro(input, false)
end)

_friendWhitelist = {}
_friendWhitelistEnabled = false

function LoadFriendsIntoWhitelist()
_friendWhitelist = {}
if not LocalPlayer then return end
local success, pages = pcall(function()
return LocalPlayer:GetFriendsAsync()
end)
if not success or not pages then return end
repeat
local currentPage = pages:GetCurrentPage()
for _, friend in ipairs(currentPage) do
if friend and friend.Id then
_friendWhitelist[friend.Id] = true
end
end
if not pages.IsFinished then
pages:AdvanceToNextPageAsync()
else
break
end
until false
Library:Notify("Друзья загружены: " .. #_friendWhitelist .. " в вайтлисте", 2)
end

function isFriendWhitelisted(plr)
if not _friendWhitelistEnabled then return false end
if not plr then return false end
return _friendWhitelist[plr.UserId] == true
end

local TargetPlayers = Tabs.Target:AddLeftGroupbox(L("Игроки на сервере", "Players on Server"), "users")
local TargetAction = Tabs.Target:AddRightGroupbox(L("Действие", "Action"), "crosshair")

TargetAvatarImage = TargetPlayers:AddImage("TargetAvatarDisplay", {
Image = "rbxassetid://0",
Size = Vector2.new(150, 150),
})

TargetAvatarLabel = TargetPlayers:AddLabel(L("<b><font color='#aaaaaa'>Цель не выбрана</font></b>", "<b><font color='#aaaaaa'>No target selected</font></b>"))

TargetStatName        = TargetPlayers:AddLabel("<b><font color='#aaaaaa'>Name:</font></b> <font color='#888888'>—</font>")
TargetStatUsername    = TargetPlayers:AddLabel("<b><font color='#aaaaaa'>Username:</font></b> <font color='#888888'>—</font>")
TargetStatHealth      = TargetPlayers:AddLabel("<b><font color='#aaaaaa'>Health:</font></b> <font color='#888888'>—</font>")
TargetStatStuds        = TargetPlayers:AddLabel("<b><font color='#aaaaaa'>Studs:</font></b> <font color='#888888'>—</font>")
TargetStatCord        = TargetPlayers:AddLabel("<b><font color='#aaaaaa'>Cord:</font></b> <font color='#888888'>—</font>")
TargetStatInPlot      = TargetPlayers:AddLabel("<b><font color='#aaaaaa'>In plot:</font></b> <font color='#888888'>—</font>")
TargetStatRagdoll     = TargetPlayers:AddLabel("<b><font color='#aaaaaa'>Ragdoll:</font></b> <font color='#888888'>—</font>")
TargetStatKick        = TargetPlayers:AddLabel("<b><font color='#aaaaaa'>Kick:</font></b> <font color='#888888'>—</font>")
TargetStatLeft        = TargetPlayers:AddLabel("<b><font color='#aaaaaa'>Left:</font></b> <font color='#888888'>—</font>")
TargetStatRejoin      = TargetPlayers:AddLabel("<b><font color='#aaaaaa'>Rejoin:</font></b> <font color='#888888'>—</font>")
TargetStatLastGrab    = TargetPlayers:AddLabel("<b><font color='#aaaaaa'>Last grab:</font></b> <font color='#888888'>None</font>")
TargetStatPCLDBreak   = TargetPlayers:AddLabel("<b><font color='#aaaaaa'>PCLD Break:</font></b> <font color='#888888'>—</font>")

_targetAvatarCurrentUid = nil
_tgtStatConn = nil

function tgtUpdateStats()
if not Options.TargetPlayer then return end
local value = Options.TargetPlayer.Value
if not value or value == "" then return end
local plr = tgtGetPlayerByName(value)
if not plr or not plr.Character then return end

local char = plr.Character
local hrp = char:FindFirstChild("HumanoidRootPart")
local hum = char:FindFirstChildOfClass("Humanoid")
local lpHrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")

pcall(function() TargetStatName:SetText("<b><font color='#aaaaaa'>Name:</font></b> <font color='#ffffff'>" .. tostring(plr.DisplayName) .. "</font>") end)
pcall(function() TargetStatUsername:SetText("<b><font color='#aaaaaa'>Username:</font></b> <font color='#ffffff'>" .. tostring(plr.Name) .. "</font>") end)

if hum then
local hp = math.floor(hum.Health)
local hpMax = math.floor(hum.MaxHealth)
local pct = hpMax > 0 and math.floor((hp / hpMax) * 100) or 0
local color = "#55ff55"
if pct <= 0 then color = "#ff5555"
elseif pct <= 50 then color = "#ffff55" end
pcall(function() TargetStatHealth:SetText("<b><font color='#aaaaaa'>Health:</font></b> <font color='" .. color .. "'>" .. pct .. "%</font>") end)
end

if hrp and lpHrp then
local studs = math.floor((lpHrp.Position - hrp.Position).Magnitude)
pcall(function() TargetStatStuds:SetText("<b><font color='#aaaaaa'>Studs:</font></b> <font color='#ffffff'>" .. studs .. "</font>") end)
end

if hrp then
local pos = hrp.Position
pcall(function() TargetStatCord:SetText("<b><font color='#aaaaaa'>Cord:</font></b> <font color='#ffffff'>" .. math.floor(pos.X) .. ", " .. math.floor(pos.Y) .. ", " .. math.floor(pos.Z) .. "</font>") end)
end

local inPlot = false
local okPlot = pcall(function()
inPlot = plr:FindFirstChild("InPlot") and plr.InPlot.Value or false
end)
local plotColor = inPlot and "#55ff55" or "#ff5555"
pcall(function() TargetStatInPlot:SetText("<b><font color='#aaaaaa'>In plot:</font></b> <font color='" .. plotColor .. "'>" .. (inPlot and "Yes" or "No") .. "</font>") end)

local isRagdolled = false
if hum then
local ragVal = hum:FindFirstChild("Ragdolled")
isRagdolled = ragVal and ragVal.Value or false
end
local ragColor = isRagdolled and "#ff5555" or "#55ff55"
pcall(function() TargetStatRagdoll:SetText("<b><font color='#aaaaaa'>Ragdoll:</font></b> <font color='" .. ragColor .. "'>" .. (isRagdolled and "Yes" or "No") .. "</font>") end)

local kickText = "—"
local kickColor = "#888888"
pcall(function()
local kickAttr = char:GetAttribute("KickedBy")
if kickAttr and kickAttr ~= "" then
kickText = tostring(kickAttr)
kickColor = "#ff5555"
end
end)
pcall(function() TargetStatKick:SetText("<b><font color='#aaaaaa'>Kick:</font></b> <font color='" .. kickColor .. "'>" .. kickText .. "</font>") end)

local leftText = "—"
pcall(function()
local leaveTime = plr:GetAttribute("TimeUntilKick")
if leaveTime and leaveTime > 0 then
local h = math.floor(leaveTime / 3600)
local m = math.floor((leaveTime % 3600) / 60)
local s = math.floor(leaveTime % 60)
leftText = string.format("%d (%02d:%02d:%02d)", leaveTime, h, m, s)
end
end)
pcall(function() TargetStatLeft:SetText("<b><font color='#aaaaaa'>Left:</font></b> <font color='#ffffff'>" .. leftText .. "</font>") end)

local rejoinText = "—"
pcall(function()
local rejoinTime = plr:GetAttribute("TimeUntilRejoin")
if rejoinTime and rejoinTime > 0 then
local h = math.floor(rejoinTime / 3600)
local m = math.floor((rejoinTime % 3600) / 60)
local s = math.floor(rejoinTime % 60)
rejoinText = string.format("%d (%02d:%02d:%02d)", rejoinTime, h, m, s)
end
end)
pcall(function() TargetStatRejoin:SetText("<b><font color='#aaaaaa'>Rejoin:</font></b> <font color='#ffffff'>" .. rejoinText .. "</font>") end)

local lastGrabText = "None"
local lastGrabColor = "#ff9955"
pcall(function()
local grabber = char:GetAttribute("LastGrabber")
if grabber and grabber ~= "" then
lastGrabText = tostring(grabber)
lastGrabColor = "#ffaa55"
end
end)
pcall(function() TargetStatLastGrab:SetText("<b><font color='#aaaaaa'>Last grab:</font></b> <font color='" .. lastGrabColor .. "'>" .. lastGrabText .. "</font>") end)

local pcldBreak = false
pcall(function()
local pcld = char:GetAttribute("PCLDBreak")
pcldBreak = pcld == true
end)
local pcldColor = pcldBreak and "#ff5555" or "#55ff55"
pcall(function() TargetStatPCLDBreak:SetText("<b><font color='#aaaaaa'>PCLD Break:</font></b> <font color='" .. pcldColor .. "'>" .. (pcldBreak and "Yes" or "No") .. "</font>") end)
end

task.spawn(function()
while true do
if Options.TargetPlayer and Options.TargetPlayer.Value and Options.TargetPlayer.Value ~= "" then
pcall(tgtUpdateStats)
end
task.wait(0.5)
end
end)

function tgtUpdateAvatarDisplay()
if not Options.TargetPlayer then return end
local value = Options.TargetPlayer.Value
if not value or value == "" then
_targetAvatarCurrentUid = nil
pcall(function() TargetAvatarImage:SetImage("rbxassetid://0") end)
pcall(function() TargetAvatarLabel:SetText("<b><font color='#aaaaaa'>Цель не выбрана</font></b>") end)
pcall(function() TargetStatName:SetText("<b><font color='#aaaaaa'>Name:</font></b> <font color='#888888'>—</font>") end)
pcall(function() TargetStatUsername:SetText("<b><font color='#aaaaaa'>Username:</font></b> <font color='#888888'>—</font>") end)
pcall(function() TargetStatHealth:SetText("<b><font color='#aaaaaa'>Health:</font></b> <font color='#888888'>—</font>") end)
pcall(function() TargetStatStuds:SetText("<b><font color='#aaaaaa'>Studs:</font></b> <font color='#888888'>—</font>") end)
pcall(function() TargetStatCord:SetText("<b><font color='#aaaaaa'>Cord:</font></b> <font color='#888888'>—</font>") end)
pcall(function() TargetStatInPlot:SetText("<b><font color='#aaaaaa'>In plot:</font></b> <font color='#888888'>—</font>") end)
pcall(function() TargetStatRagdoll:SetText("<b><font color='#aaaaaa'>Ragdoll:</font></b> <font color='#888888'>—</font>") end)
pcall(function() TargetStatKick:SetText("<b><font color='#aaaaaa'>Kick:</font></b> <font color='#888888'>—</font>") end)
pcall(function() TargetStatLeft:SetText("<b><font color='#aaaaaa'>Left:</font></b> <font color='#888888'>—</font>") end)
pcall(function() TargetStatRejoin:SetText("<b><font color='#aaaaaa'>Rejoin:</font></b> <font color='#888888'>—</font>") end)
pcall(function() TargetStatLastGrab:SetText("<b><font color='#aaaaaa'>Last grab:</font></b> <font color='#888888'>None</font>") end)
pcall(function() TargetStatPCLDBreak:SetText("<b><font color='#aaaaaa'>PCLD Break:</font></b> <font color='#888888'>—</font>") end)
return
end
local plr = tgtGetPlayerByName(value)
if not plr then
_targetAvatarCurrentUid = nil
pcall(function() TargetAvatarImage:SetImage("rbxassetid://0") end)
pcall(function() TargetAvatarLabel:SetText("<b><font color='#ff5555'>Игрок не найден</font></b>") end)
return
end
local uid = plr.UserId
if uid == _targetAvatarCurrentUid then return end
_targetAvatarCurrentUid = uid
pcall(function()
TargetAvatarLabel:SetText("<b><font color='#ffffff'>" .. plr.DisplayName .. "</font></b> <font color='#888888'>(@" .. plr.Name .. ")</font>")
end)
task.spawn(function()
local ok, content = pcall(Players.GetUserThumbnailAsync, Players, uid, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size420x420)
if ok and content and content ~= "" and plr.Parent then
if _targetAvatarCurrentUid == uid then
pcall(function() TargetAvatarImage:SetImage(content) end)
end
end
end)
end

local function tgtPlayerList()
local list = {}
for _, p in ipairs(Players:GetPlayers()) do
if p ~= LocalPlayer then
table.insert(list, p.DisplayName .. " (@" .. p.Name .. ")")
end
end
return list
end

function tgtGetPlayerByName(value)
if not value or value == "" then return nil end
local username = value:match("@([^)]+)")
if username then
return Players:FindFirstChild(username)
end
return Players:FindFirstChild(value)
end

TargetPlayers:AddDropdown("TargetPlayer", {
Text = L("Цель", "Target"),
Values = tgtPlayerList(),
Default = (tgtPlayerList())[1] or nil,
Tooltip = L("Список игроков обновляется сам", "Player list updates automatically"),
Callback = function() tgtUpdateAvatarDisplay() end,
})

local function tgtRefresh()
if not Options.TargetPlayer then return end
local list = tgtPlayerList()
pcall(function() Options.TargetPlayer:SetValues(list) end)
local cur = Options.TargetPlayer.Value
local stillHere = false
for _, n in ipairs(list) do if n == cur then stillHere = true break end end
if (not stillHere) and #list > 0 then
pcall(function() Options.TargetPlayer:SetValue(list[1]) end)
end
tgtUpdateAvatarDisplay()
end

Players.PlayerAdded:Connect(function() task.wait(0.3) tgtRefresh() end)
Players.PlayerRemoving:Connect(function() task.defer(tgtRefresh) end)

task.spawn(function() task.wait(1) tgtRefresh(); tgtUpdateAvatarDisplay() end)

TargetPlayers:AddButton({
Text = L("Обновить список", "Refresh List"),
Func = function() tgtRefresh() end,
})

TargetPlayers:AddToggle("TargetLoopPlayer", {
Text = "Loop Player",
Default = false,
Tooltip = L("Бесконечно применять действие к выбранной цели", "Infinitely apply action to selected target"),
})

TargetPlayers:AddToggle("TargetLoopServer", {
Text = "Loop Server",
Default = false,
Tooltip = L("Бесконечно применять действие ко всему серверу", "Infinitely apply action to entire server"),
})

TargetPlayers:AddToggle("TargetSkipFriends", {
Text = L("Не трогать друзей", "Skip Friends"),
Default = false,
Tooltip = L("Исключает друзей из кика по всему серверу (Loop Server / Server кнопка)", "Excludes friends from server-wide kick (Loop Server / Server button)"),
})

TargetAction:AddDropdown("DestroyHeightMode", {
Text = "Destroy Height",
Values = {"Spawn", "Heaven"},
Default = "Spawn",
Multi = false,
Tooltip = L("Spawn = высота 35 (как в 9rr). Heaven = 1e9 (в космос)", "Spawn = height 35 (like 9rr). Heaven = 1e9 (in space)"),
})

Toggles.TargetLoopServer:OnChanged(function()
if Toggles.TargetLoopServer.Value then
Library:Notify(L("Destroy Server: запускаю (с лагами!)...", "Destroy Server: starting (with lag!)..."), 2)
task.spawn(function()
while Toggles.TargetLoopServer.Value do
local heightMode = (Options.DestroyHeightMode and Options.DestroyHeightMode.Value) or "Spawn"
local height = (heightMode == "Heaven") and 1e9 or 35

tgtStartLineLag()
task.wait(1)

local players = {}
for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer and not isFriendWhitelisted(plr) then
table.insert(players, plr)
end
end
if #players == 0 then tgtStopLineLag() task.wait(1) continue end

local myChar = LocalPlayer.Character
local myHrp = myChar and myChar:FindFirstChild("HumanoidRootPart")
if not myHrp then tgtStopLineLag() task.wait(1) continue end

local playerData = {}
for _, plr in ipairs(players) do
local char = plr.Character
local hrp = char and char:FindFirstChild("HumanoidRootPart")
if hrp then table.insert(playerData, {player = plr, hrp = hrp}) end
end

for _, data in ipairs(playerData) do
pcall(function() myHrp.CFrame = data.hrp.CFrame * CFrame.new(0, 5, 5) end)
task.wait(0.2)
local GE = ReplicatedStorage:FindFirstChild("GrabEvents")
local setOwner = GE and GE:FindFirstChild("SetNetworkOwner")
if setOwner and data.hrp then
pcall(function() setOwner:FireServer(data.hrp, data.hrp.CFrame) end)
end
task.wait()
end

pcall(function()
myHrp.CFrame = myHrp.CFrame
myHrp.AssemblyLinearVelocity = Vector3.zero
end)

local radius = 40
local angleStep = (math.pi * 2) / #playerData
for idx, data in ipairs(playerData) do
local angle = (idx - 1) * angleStep
local x = math.cos(angle) * radius
local z = math.sin(angle) * radius

pcall(function()
data.hrp.CFrame = CFrame.new(x, height, z)
data.hrp.AssemblyLinearVelocity = Vector3.zero
end)

local bp = Instance.new("BodyPosition")
bp.MaxForce = Vector3.new(1e9, 1e9, 1e9)
bp.P = 40000000
bp.Position = Vector3.new(x, height, z)
bp.Parent = data.hrp
task.delay(2, function() pcall(function() bp:Destroy() end) end)
task.wait()
end

local GE = ReplicatedStorage:FindFirstChild("GrabEvents")
local destroyLine = GE and GE:FindFirstChild("DestroyGrabLine")
if destroyLine then
for i = 1, 8 do
for _, data in ipairs(playerData) do
pcall(function() destroyLine:FireServer(data.hrp) end)
end
task.wait(0.3)
end
end

tgtStopLineLag()
task.wait(1)
end
end)
else
tgtStopLineLag()
Library:Notify(L("Destroy Server: остановлено", "Destroy Server: stopped"), 2)
end
end)

tgtBringLoop = false
tgtBringInit = false  

function getGE()
return ReplicatedStorage:FindFirstChild("GrabEvents")
end

function tgtBringTeleport(targetName, isFirstFrame)
local target = Players:FindFirstChild(targetName)
if not target then return end
local tChar = target.Character
local tRoot = tChar and tChar:FindFirstChild("HumanoidRootPart")
local tHum = tChar and tChar:FindFirstChild("Humanoid")
local myChar = LocalPlayer.Character
local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
local myHum = myChar and myChar:FindFirstChildOfClass("Humanoid")
if not (tRoot and tHum and tHum.Health > 0 and myRoot and myHum) then return end

local GE = GE_Bloodman or getGE()
if not GE then return end
local sno = GE:FindFirstChild("SetNetworkOwner")
if not sno then return end
local cgl = createGrabLineEvent or GE:FindFirstChild("CreateGrabLine")
local dgl = GE:FindFirstChild("DestroyGrabLine")

if isFirstFrame then
local savedPos = myRoot.CFrame
local bringPos = savedPos * CFrame.new(0, 0, 3)

if not sitOnBlobman() then return end
task.wait(0.2)

local seat = myHum.SeatPart
if not (seat and seat.Parent and seat.Parent.Name == "CreatureBlobman") then return end
local blob = seat.Parent
local blobRoot = blob:FindFirstChild("HumanoidRootPart") or blob.PrimaryPart

local remoteFolder = blob:FindFirstChild("BlobmanSeatAndOwnerScript")
local grab = remoteFolder and remoteFolder:FindFirstChild("CreatureGrab")
local drop = remoteFolder and remoteFolder:FindFirstChild("CreatureDrop")
local L_Det = blob:FindFirstChild("LeftDetector")
local R_Det = blob:FindFirstChild("RightDetector")
local L_Weld = L_Det and (L_Det:FindFirstChild("LeftWeld") or L_Det:FindFirstChild("RigidConstraint"))
local R_Weld = R_Det and (R_Det:FindFirstChild("RightWeld") or R_Det:FindFirstChild("RigidConstraint"))

myRoot.CFrame = tRoot.CFrame
if blobRoot then blobRoot.CFrame = tRoot.CFrame end
task.wait(0.1)

if grab and drop and L_Weld and R_Weld then
pcall(function()
grab:FireServer(L_Det, tRoot, L_Weld)
grab:FireServer(R_Det, tRoot, R_Weld)
drop:FireServer(L_Weld, tRoot)
drop:FireServer(R_Weld, tRoot)
end)
end

pcall(function()
tHum.PlatformStand = true
tRoot.AssemblyLinearVelocity = Vector3.zero
sno:FireServer(tRoot, myRoot.CFrame)
if cgl then
cgl:FireServer(tRoot, Vector3.zero, tRoot.Position, false)
end
end)

task.wait(0.5)

myRoot.CFrame = savedPos
myRoot.AssemblyLinearVelocity = Vector3.zero
if blobRoot then
blobRoot.CFrame = savedPos
blobRoot.AssemblyLinearVelocity = Vector3.zero
end

pcall(function()
tRoot.CFrame = bringPos
tRoot.AssemblyLinearVelocity = Vector3.zero
tHum.PlatformStand = true
sno:FireServer(tRoot, bringPos)
if dgl then dgl:FireServer(tRoot) end
if cgl then
cgl:FireServer(tRoot, Vector3.zero, tRoot.Position, false)
end
end)
task.wait(0.1)

if dgl then
for _ = 1, 5 do
pcall(function() dgl:FireServer(tRoot) end)
task.wait(0.01)
end
end

pcall(function() myHum.Sit = false end)
task.wait(0.1)

local folder = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
local blobToDestroy = folder and folder:FindFirstChild("CreatureBlobman")
if blobToDestroy then
local destroyRem = ReplicatedStorage:FindFirstChild("MenuToys")
destroyRem = destroyRem and destroyRem:FindFirstChild("DestroyToy")
if destroyRem then destroyRem:FireServer(blobToDestroy) end
end
else
local bringPos = myRoot.CFrame * CFrame.new(0, 0, 3)
pcall(function()
tHum.PlatformStand = true
sno:FireServer(tRoot, bringPos)
if cgl then
cgl:FireServer(tRoot, Vector3.zero, tRoot.Position, false)
end
tRoot.CFrame = bringPos
tRoot.AssemblyLinearVelocity = Vector3.zero
end)
end
end

function tgtBringAll(isFirstFrame)
for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer then
tgtBringTeleport(plr.Name, isFirstFrame)
end
end
end

local tbl17, tbl18, tbl19, tbl20

TargetAction:AddDropdown("TargetMethod", {
Text = L("Что делаем (можно несколько)", "Action (multi-select)"),
Values = { "Kick", "Kill", "Lock", "Teleport", "View", "BlackHole", "BombMissile", "Banana", "Snowball", "AntiAntiKickWD", "AntiAntiKickClick", "AntiAntiKickBlackHole", "DontGiveChance", "PalletRagdoll", "RemoveTargetGucci", "Destroy Food", "Remove Invisibility", "Delete Seatables", "FireworkMissile", "BombBalloon", "PresentBig", "PresentSmall" },
Default = "Teleport",
Multi = true,
Tooltip = L("Можно выбрать несколько действий — они выполнятся параллельно. Зажми Ctrl/Cmd для множественного выбора.", "Select multiple actions — they run in parallel. Hold Ctrl/Cmd for multi-select."),
Callback = function() end,
})

TargetAction:AddSlider("ToyCount", {
Text = L("Количество игрушек", "Toy Count"),
Default = 3,
Min = 1,
Max = 10,
Rounding = 0,
Compact = false,
Tooltip = L("Сколько игрушек спавнить (BlackHole, BombMissile, Firework, Balloon, Present)", "How many toys to spawn (BlackHole, BombMissile, Firework, Balloon, Present)"),
})

local tgtRestartBusy = false
TargetAction:AddButton({
Text = L("Перезапустить кик", "Restart Kick"),
Tooltip = L("Полный перезапуск действий на текущей цели: останавливает циклы, освобождает линии и берёт цель заново", "Full restart of actions on the current target: stops the loops, releases the lines and re-grabs the target"),
Func = function()
if tgtRestartBusy then return end
tgtRestartBusy = true
task.spawn(function()
local rawValue = Options.TargetPlayer and Options.TargetPlayer.Value or ""
local targetPlayer = tgtGetPlayerByName(rawValue)
if not targetPlayer then
Library:Notify(L("Target: выбери игрока", "Target: select a player"), 3)
tgtRestartBusy = false
return
end
Library:Notify(L("Target: перезапускаю кик...", "Target: restarting the kick..."), 2)
local loopToggle = Toggles.TargetLoopPlayer
if loopToggle and loopToggle.Value then
loopToggle:SetValue(false)
end
local t0 = tick()
while tick() - t0 < 5 do
local busy = (tbl19.LoopApplyMethodT == true) or (tbl20.SelectedPlayers and tbl20.SelectedPlayers[targetPlayer] == true) or (tbl20.NetworkKickQueue and tbl20.NetworkKickQueue[targetPlayer] ~= nil)
if not busy then break end
task.wait(0.1)
end
task.wait(0.4)
if loopToggle and not loopToggle.Value then
loopToggle:SetValue(true)
end
tgtRestartBusy = false
end)
end,
})

local PHKickSettings = TargetAction:AddDependencyBox()
PHKickSettings:SetupDependencies({ { Options.TargetMethod, "Kick" } })

PHKickSettings:AddDivider(L("Kick Settings", "Kick Settings"))

PHKickSettings:AddDropdown("NKDelayConf", {
Text = L("Режим задержки", "Delay Mode"),
Values = { "Normal", "Slower but better", "Faster but worse" },
Multi = false,
Default = 1,
Tooltip = L("Режим снятия овнершипа при кике", "Kick ownership release delay mode"),
Callback = function(nkDelayMode)
tbl18.NKDelayMode = nkDelayMode
end,
})

PHKickSettings:AddToggle("RagdollNK", {
Text = L("Pallet Ragdoll (Invis)", "Pallet Ragdoll (Invis)"),
Default = true,
Tooltip = L("Pallet Ragdoll (Invis): невидимая паллета бьёт цель гиперскоростью и рагдоллит её при кике (порт XOCU)", "Pallet Ragdoll (Invis): invisible pallet strikes the target with hyper-velocity and ragdolls it during kick (XOCU port)"),
Callback = function(ragdollInNK)
tbl19.RagdollInNK = ragdollInNK
if not ragdollInNK and tgtPalletRagdollStopFor then
pcall(tgtPalletRagdollStopFor, nil)
end
end,
})

PHKickSettings:AddToggle("AntiKickRemoverNK", {
Text = L("Remove Anti-Kick", "Remove Anti-Kick"),
Default = true,
Tooltip = L("Удалять анти-кик шурикены у цели", "Deletes target anti-kick shurikens"),
Callback = function(deleteShurInNK)
tbl19.DeleteShurInNK = deleteShurInNK
end,
})

PHKickSettings:AddSlider("HeightNK", {
Text = L("Kick Height", "Kick Height"),
Default = 15,
Min = 0,
Max = 28,
Compact = true,
Rounding = 0,
Tooltip = L("Высота, на которой висит цель", "Height where the target will hang"),
Callback = function(nkOffset)
tbl18.NKOffset = nkOffset
end,
})

PHKickSettings:AddButton({
Text = L("TP To Void Platform", "TP To Void Platform"),
Tooltip = L("Телепорт на платформу, где жертва не сможет спавнить анти-кик шурикены (влияет и на тебя)", "Teleports you to a platform where victim cant spawn anti-kick shurikens, though it affects you as well"),
Func = function()
tgtPoopTPVoidPlatform()
end,
})

local PHGucciSettings = TargetAction:AddDependencyBox()
PHGucciSettings:SetupDependencies({ { Options.TargetMethod, "RemoveTargetGucci" } })

PHGucciSettings:AddDivider(L("Remove-Gucci Settings", "Remove-Gucci Settings"))

PHGucciSettings:AddToggle("RagdollInRG", {
Text = L("Break Player? [Inf Ragdoll]", "Break Player? [Inf Ragdoll]"),
Default = false,
Tooltip = L("Это занимает 1.5 секунды, используй только если есть время!", "This takes 1.5 seconds, use only if you have time!"),
Callback = function(ragdollInRG)
tbl19.RagdollInRG = ragdollInRG
end,
})

PHGucciSettings:AddButton({
Text = L("Fix Gucci Remover", "Fix Gucci Remover"),
Tooltip = L("Чинит баги Remove Gucci, если не работает", "Fixes Remove Gucci bugs if it's not working"),
Func = function()
for k, selectedPlayer in pairs(tbl20.SelectedPlayers) do
if selectedPlayer then
local v27, v28 = tbl17.getInventory(k)

if v27 then
for _, child in ipairs(v27:GetChildren()) do
local vehicleSeat = child:FindFirstChild("VehicleSeat")

if vehicleSeat then
vehicleSeat:SetAttribute("broken", false)
end
end
end

if v28 then
for _, child in ipairs(v28:GetChildren()) do
local vehicleSeat = child:FindFirstChild("VehicleSeat")

if vehicleSeat then
vehicleSeat:SetAttribute("broken", false)
end
end
end
end
end
end,
})

tgtTeleportLoop = false
tgtViewLoop = false

Toggles.TargetLoopPlayer:OnChanged(function()
local actionsDict = Options.TargetMethod and Options.TargetMethod.Value or {}
local actions = {}
if type(actionsDict) == "string" then
actions = {actionsDict}
else
for actionName, isOn in pairs(actionsDict) do
if isOn then table.insert(actions, actionName) end
end
end
local rawValue = Options.TargetPlayer and Options.TargetPlayer.Value or ""
local targetPlayer = tgtGetPlayerByName(rawValue)
local targetName = targetPlayer and targetPlayer.Name or ""

if Toggles.TargetLoopPlayer.Value then
if targetName == "" then
Library:Notify(L("Target: выбери игрока", "Target: select a player"), 3)
Toggles.TargetLoopPlayer:SetValue(false)
return
end
if #actions == 0 then
Library:Notify(L("Target: выбери действие в списке", "Target: select an action"), 3)
Toggles.TargetLoopPlayer:SetValue(false)
return
end

for _, action in ipairs(actions) do
if action == "Teleport" then
tgtTeleportLoop = true
task.spawn(function()
Library:Notify("Teleport loop: " .. targetName, 2)
while tgtTeleportLoop do
local target = Players:FindFirstChild(targetName)
local tChar = target and target.Character
local tRoot = tChar and tChar:FindFirstChild("HumanoidRootPart")
local myChar = LocalPlayer.Character
local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
if tRoot and myRoot then
myRoot.CFrame = tRoot.CFrame * CFrame.new(0, 0, 5)
end
RunService.Heartbeat:Wait()
end
end)
elseif action == "View" then
tgtViewLoop = true
task.spawn(function()
Library:Notify("View loop: " .. targetName, 2)
while tgtViewLoop do
local target = Players:FindFirstChild(targetName)
local tChar = target and target.Character
local tHead = tChar and tChar:FindFirstChild("Head")
local cam = workspace.CurrentCamera
local myChar = LocalPlayer.Character
local myHead = myChar and myChar:FindFirstChild("Head")
if cam and tHead and myHead then
cam.CFrame = CFrame.lookAt(myHead.Position, tHead.Position)
end
RunService.RenderStepped:Wait()
end
end)
else
task.spawn(function()
tgtRunAction(action, targetName, function() return Toggles.TargetLoopPlayer.Value end)
end)
end
end
else
tgtTeleportLoop = false
tgtViewLoop = false
OwnershipKickEnabled = false
end
end)

do

local function getGrabEventsT()
return ReplicatedStorage:FindFirstChild("GrabEvents") or ReplicatedStorage:WaitForChild("GrabEvents", 10)
end
local function getSpawnToyRF_T()
local mt = ReplicatedStorage:FindFirstChild("MenuToys") or ReplicatedStorage:WaitForChild("MenuToys", 10)
return mt and (mt:FindFirstChild("SpawnToyRemoteFunction") or mt:WaitForChild("SpawnToyRemoteFunction", 5))
end
local function getDestroyToyRE_T()
local mt = ReplicatedStorage:FindFirstChild("MenuToys") or ReplicatedStorage:WaitForChild("MenuToys", 10)
return mt and (mt:FindFirstChild("DestroyToy") or mt:WaitForChild("DestroyToy", 5))
end

do
local _GE = getGrabEventsT()
setNetworkOwnerEvent = _GE and (_GE:FindFirstChild("SetNetworkOwner") or _GE:WaitForChild("SetNetworkOwner", 5))
createGrabLineEvent = _GE and (_GE:FindFirstChild("CreateGrabLine") or _GE:WaitForChild("CreateGrabLine", 5))
destroyGrabLineEvent = _GE and (_GE:FindFirstChild("DestroyGrabLine") or _GE:WaitForChild("DestroyGrabLine", 5))
end

local function tgtSno(part)
if not part or not part.Parent then return end
pcall(function() getGrabEventsT().SetNetworkOwner:FireServer(part, part.CFrame) end)
end

local function getTgtSelectedTarget()
if not Options.TargetPlayer then return nil end
local rawValue = Options.TargetPlayer and Options.TargetPlayer.Value or ""
local targetPlayer = tgtGetPlayerByName(rawValue)
local name = targetPlayer and targetPlayer.Name or ""
if not name or name == "" then return nil end
return targetPlayer
end

local function cleanupKickAlign(tHRP)
if not tHRP or not tHRP.Parent then return end
local align = tHRP:FindFirstChild("KickAlign")
local rot = tHRP:FindFirstChild("KickRot")
local att0 = tHRP:FindFirstChild("KickAtt0")
if align then
if align.Attachment1 then align.Attachment1:Destroy() end
align:Destroy()
end
if rot then rot:Destroy() end
if att0 then att0:Destroy() end
pcall(function() getGrabEventsT().DestroyGrabLine:FireServer(tHRP) end)
end

function tgtV3Action(mode, targetName, keepGoing)
local target = Players:FindFirstChild(targetName)
if not target then return end

local myChar = LocalPlayer.Character
local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
if not (myChar and myHRP) then return end

local savedPos = myHRP.CFrame
local startTime = tick()
local done = false

if mode == "kill" then
local KILL_HEIGHT_LIMIT = 10000
local KILL_OFFSET = Vector3.new(5, -18.5, 0)

local function killSnoIsAboveLimit(plr)
local c = plr and plr.Character
local hrp = c and c:FindFirstChild("HumanoidRootPart")
if hrp then
return hrp.Position.Y > KILL_HEIGHT_LIMIT
end
return true
end

local function killSnoDisableCollisions(character)
for _, desc in pairs(character:GetDescendants()) do
if desc:IsA("BasePart") then
desc.CanCollide = false
end
end
end

local function killSnoForceDeath(root, humanoid)
for _, part in pairs(humanoid.Parent:GetChildren()) do
if part:IsA("BasePart") then
part.CFrame = CFrame.new(-999999999999, 9999999999999, -999999999999)
end
end
task.wait()
for _, part2 in pairs(humanoid.Parent:GetChildren()) do
if part2:IsA("BasePart") then
part2.CFrame = CFrame.new(-999999999999, 9999999999999, -999999999999)
end
end
local bv = Instance.new("BodyVelocity")
bv.Velocity = Vector3.new(0, 99999999999, 0)
bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bv.P = 100000075
bv.Parent = root
humanoid.Sit = false
humanoid.Jump = true
humanoid.BreakJointsOnDeath = false
humanoid:ChangeState(Enum.HumanoidStateType.Dead)
task.delay(2, function()
if bv and bv.Parent then bv:Destroy() end
end)
end

while keepGoing() do
local target = Players:FindFirstChild(targetName)
if not target or not target.Parent then break end
local myChar = LocalPlayer.Character
local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
if not myHRP then
task.wait(0.1)
continue
end
local savedCFrame = myChar:GetPivot()
local inPlot = false
pcall(function()
local pi = Workspace:FindFirstChild("PlotItems")
local pip = pi and pi:FindFirstChild("PlayersInPlots")
inPlot = (pip and pip:FindFirstChild(target.Name) ~= nil) or false
end)
if not inPlot then
local tChar = target.Character
local tRoot = tChar and tChar:FindFirstChild("HumanoidRootPart")
local tHead = tChar and tChar:FindFirstChild("Head")
local tHum = tChar and tChar:FindFirstChild("Humanoid")
if tRoot and tHead and tHum and tHum.Health > 0 and not killSnoIsAboveLimit(target) then
pcall(function()
if not keepGoing() then return end
if not (tbl17 and tbl17.sno and tbl17.grab) then return end
killSnoDisableCollisions(tChar)
local owned = tbl17.ispartowner(tRoot)
if not owned then
owned = tbl17.sno(tRoot) == true
end
if not owned then
for _ = 1, 3 do
if not keepGoing() or not tRoot.Parent then return end
if tbl17.sno(tRoot) == true then
owned = true
break
end
task.wait(0.05)
end
end
if not owned then return end
tbl17.grab(tRoot)
task.wait(0.1)
killSnoForceDeath(tRoot, tHum)
task.wait(0.2)
pcall(function() tbl17.unsno(tRoot) end)
end)
task.wait(0.1)
end
end
task.wait(0.05)
end
return
end

local lastRemoteFire = tick()
while keepGoing() do
target = Players:FindFirstChild(targetName)
if not target or not target.Parent then break end
myChar = LocalPlayer.Character
myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
local myHead = myChar and myChar:FindFirstChild("Head")
local tChar = target.Character
local tHRP = tChar and tChar:FindFirstChild("HumanoidRootPart")
local tHum = tChar and tChar:FindFirstChildOfClass("Humanoid")
if not (myChar and myHRP and myHead) or not (tHRP and tHum) or tHum.Health <= 0 then
RunService.Heartbeat:Wait()
continue
end
local dist = (tHRP.Position - myHRP.Position).Magnitude
if dist <= 30 then
if not tHRP:FindFirstChild("KickAlign") then
local oldBp = tHRP:FindFirstChildOfClass("BodyPosition")
if oldBp then oldBp:Destroy() end
local att0 = Instance.new("Attachment", tHRP)
att0.Name = "KickAtt0"
local att1 = Instance.new("Attachment", workspace.Terrain)
att1.Name = "KickAtt1"
local alignPos = Instance.new("AlignPosition")
alignPos.Name = "KickAlign"
alignPos.Attachment0 = att0
alignPos.Attachment1 = att1
alignPos.MaxForce = math.huge
alignPos.Responsiveness = 200
alignPos.Parent = tHRP
local alignRot = Instance.new("AlignOrientation")
alignRot.Name = "KickRot"
alignRot.Attachment0 = att0
alignRot.Mode = Enum.OrientationAlignmentMode.OneAttachment
alignRot.CFrame = CFrame.new()
alignRot.MaxTorque = math.huge
alignRot.Responsiveness = 200
alignRot.Parent = tHRP
end
tgtSno(tHRP)
local align = tHRP:FindFirstChild("KickAlign")
if align and align.Attachment1 then
align.Attachment1.WorldPosition = Vector3.new(0, 15, 0)
end
local rot = tHRP:FindFirstChild("KickRot")
if rot then rot.CFrame = CFrame.Angles(0, 0, 0) end
if tick() - lastRemoteFire > getKickRemoteDelay() then
pcall(function() getGrabEventsT().DestroyGrabLine:FireServer(tHRP) end)
lastRemoteFire = tick()
end
else
pcall(function() myChar:PivotTo(tHRP.CFrame * CFrame.new(0, 2, 4)) end)
tgtSno(tHRP)
end
RunService.Heartbeat:Wait()
end
if target and target.Character then
local tH = target.Character:FindFirstChild("HumanoidRootPart")
if tH then cleanupKickAlign(tH) end
end
pcall(function() if LocalPlayer.Character then LocalPlayer.Character:PivotTo(savedPos) end end)
end

function tgtLockLoop(targetName, keepGoing)
if not (tbl17 and tbl17.sno and tbl17.grab and tbl17.charp) then
Library:Notify(L("Lock: ядро Resonance не загружено", "Lock: Resonance core not loaded"), 3)
return
end
local LOCK_HEIGHT = (tbl18 and tbl18.NKOffset) or 15

while keepGoing() do
local target = Players:FindFirstChild(targetName)
if not target or not target.Parent or not target.Character then break end
local myChar, myHRP, myHum = tbl17.charp()
if not (myChar and myHRP and myHum) or myHum.Health <= 0 then
task.wait(0.1)
continue
end
local tChar, tRoot, tHum = tbl17.charp(target)
if not (tChar and tRoot and tHum) or tHum.Health <= 0 then
task.wait(0.1)
continue
end

if not tbl17.ispartowner(tRoot) then
local holdPos0 = myHRP.Position + Vector3.new(0, LOCK_HEIGHT, 0)
prepareNetworkKickMovers(tRoot, holdPos0)
if tbl17.getMagnitude(myHRP, tRoot) > 33 then
if not tbl17.sno(tRoot) then
task.wait(0.1)
continue
end
tbl17.teleport(tRoot, holdPos0)
end
tbl17.grab(tRoot)
tbl17.ToggleCollisions(tChar, false)
tbl17.RepositionLimbs(tChar, true)
end

local bp = tRoot:FindFirstChild("KickAP")
local gyro = tRoot:FindFirstChild("KickAO")
local holdPos = myHRP.Position + Vector3.new(0, LOCK_HEIGHT, 0)
if bp then bp.Position = holdPos end
if gyro then gyro.CFrame = CFrame.new() end
tRoot.AssemblyLinearVelocity = Vector3.zero
RunService.Heartbeat:Wait()
end

local target = Players:FindFirstChild(targetName)
if target and target.Character then
local tRoot = target.Character:FindFirstChild("HumanoidRootPart")
if tRoot then
pcall(function() tbl17.unsno(tRoot) end)
local ap = tRoot:FindFirstChild("KickAP")
local ao = tRoot:FindFirstChild("KickAO")
if ap then pcall(function() ap:Destroy() end) end
if ao then pcall(function() ao:Destroy() end) end
end
end
end


function tgtBombMissileLoop(targetName, keepGoing)
local target0 = Players:FindFirstChild(targetName)
if target0 and target0.Character then
local tRoot0 = target0.Character:FindFirstChild("HumanoidRootPart")
local tHead0 = target0.Character:FindFirstChild("Head")
local GE0 = getGrabEventsT()
if GE0 then
local destroyGL0 = GE0:FindFirstChild("DestroyGrabLine")
if destroyGL0 then
if tRoot0 then pcall(function() destroyGL0:FireServer(tRoot0) end) end
if tHead0 then pcall(function() destroyGL0:FireServer(tHead0) end) end
end
local sno0 = GE0:FindFirstChild("SetNetworkOwner")
if sno0 and tRoot0 then
pcall(function() sno0:FireServer(tRoot0, tRoot0.CFrame) end)
end
end
end
task.wait(0.05) 
local spawnRF = getSpawnToyRF_T()
local destroyRE = getDestroyToyRE_T()
if not spawnRF then return end
local toyFolderName = LocalPlayer.Name .. "SpawnedInToys"

local folder = workspace:FindFirstChild(toyFolderName) or workspace:WaitForChild(toyFolderName, 5)
if not folder then return end

local bombEvents = ReplicatedStorage:FindFirstChild("BombEvents")
local bombExplode = bombEvents and bombEvents:FindFirstChild("BombExplode")

local pendingRockets = {}

local conn
conn = folder.ChildAdded:Connect(function(child)
if not keepGoing() then return end
if child.Name == "BombMissile" and child:WaitForChild("ThisToysNumber", 3) then
task.spawn(function()
pcall(function()
local body = child:FindFirstChild("Body") or child:FindFirstChild("PartHitDetector")
if not body then return end
pcall(function() setNetworkOwnerEvent:FireServer(body, body.CFrame) end)
pcall(function() createGrabLineEvent:FireServer(body, Vector3.zero, body.Position, false) end)
task.wait(0.05)
pcall(function() destroyGrabLineEvent:FireServer(body) end)
task.wait(0.05)
local stableBV = Instance.new("BodyVelocity", body)
stableBV.Name = "Stable"
stableBV.Velocity = Vector3.new(0, 0, 0)
stableBV.MaxForce = Vector3.new(1, 1, 1) * math.huge
body.CFrame = CFrame.new(math.random(-1000, 1000), 10000, math.random(-1000, 1000))
end)
table.insert(pendingRockets, child)
end)
end
end)

while keepGoing() do
local count = 3
if Options.ToyCount then
count = Options.ToyCount.Value or 3
end

if #pendingRockets >= count then
local target = Players:FindFirstChild(targetName)
local tRoot = target and target.Character and target.Character:FindFirstChild("HumanoidRootPart")
if tRoot then
local toExplode = {}
for i = #pendingRockets, 1, -1 do
local rocket = pendingRockets[i]
if rocket and rocket.Parent then
table.insert(toExplode, rocket)
end
table.remove(pendingRockets, i)
end
for _, child in ipairs(toExplode) do
task.spawn(function()
pcall(function()
local body = child:FindFirstChild("Body") or child:FindFirstChild("PartHitDetector")
if not body then return end
for _ = 1, 3 do
if not body or not body.Parent then return end
body.CFrame = CFrame.new(tRoot.Position)
task.wait(0.05)
end
task.wait(0.05)
if bombExplode then
pcall(function()
bombExplode:FireServer({
["Radius"] = 17.5,
["TimeLength"] = 2,
["Hitbox"] = child:FindFirstChild("PartHitDetector") or body,
["ExplodesByFire"] = false,
["MaxForcePerStudSquared"] = 225,
["Model"] = child,
["ImpactSpeed"] = 100,
["ExplodesByPointy"] = false,
["DestroysModel"] = false,
["PositionPart"] = body
}, tRoot.Position)
end)
end
end)
end)
end
end
end

if #pendingRockets < count then
local myChar = LocalPlayer.Character
local myHead = myChar and myChar:FindFirstChild("Head")
if myHead then
task.spawn(function()
pcall(function() spawnRF:InvokeServer("BombMissile", myHead.CFrame, Vector3.zero) end)
end)
end
end
task.wait(0.5)
end

if conn then conn:Disconnect() end
for _ = 1, #pendingRockets do table.remove(pendingRockets) end

if folder and destroyRE then
for _, missile in ipairs(folder:GetChildren()) do
if missile.Name == "BombMissile" then
pcall(function() destroyRE:FireServer(missile) end)
end
end
end
end

function tgtBlackHoleLoop(targetName, keepGoing)
local target0 = Players:FindFirstChild(targetName)
if target0 and target0.Character then
local tRoot0 = target0.Character:FindFirstChild("HumanoidRootPart")
local tHead0 = target0.Character:FindFirstChild("Head")
local GE0 = getGrabEventsT()
if GE0 then
local destroyGL0 = GE0:FindFirstChild("DestroyGrabLine")
if destroyGL0 then
if tRoot0 then pcall(function() destroyGL0:FireServer(tRoot0) end) end
if tHead0 then pcall(function() destroyGL0:FireServer(tHead0) end) end
end
local sno0 = GE0:FindFirstChild("SetNetworkOwner")
if sno0 and tRoot0 then
pcall(function() sno0:FireServer(tRoot0, tRoot0.CFrame) end)
end
end
end
task.wait(0.05) 
local spawnRF = getSpawnToyRF_T()
local destroyRE = getDestroyToyRE_T()
if not spawnRF then return end
local toyFolderName = LocalPlayer.Name .. "SpawnedInToys"

local folder = workspace:FindFirstChild(toyFolderName) or workspace:WaitForChild(toyFolderName, 5)
if not folder then return end

local function launchDarkMatter(body, targetPos)
if not body or not body.Parent then return end
pcall(function() setNetworkOwnerEvent:FireServer(body, body.CFrame) end)
pcall(function() createGrabLineEvent:FireServer(body, Vector3.zero, body.Position, false) end)
task.wait(0.05)
pcall(function() destroyGrabLineEvent:FireServer(body) end)
if targetPos then
for _ = 1, 3 do
if not body or not body.Parent then return end
body.CFrame = CFrame.new(targetPos)
task.wait(0.05)
end
end
end

local conn
conn = folder.ChildAdded:Connect(function(child)
if not keepGoing() then return end
if child.Name == "BombDarkMatter" and child:WaitForChild("ThisToysNumber", 3) then
local toyNum = child.ThisToysNumber.Value
local toyNumber = folder:FindFirstChild("ToyNumber")
if toyNumber and toyNum == (toyNumber.Value - 1) then
local body = child:FindFirstChild("Body") or child:FindFirstChild("Pyramid") or child:FindFirstChild("PartHitDetector")
if body then
local target = Players:FindFirstChild(targetName)
local tHRP = target and target.Character and target.Character:FindFirstChild("HumanoidRootPart")
task.spawn(function() launchDarkMatter(body, tHRP and tHRP.Position) end)
end
end
end
end)

while keepGoing() do
local count = 3
if Options.ToyCount then
count = Options.ToyCount.Value or 3
end
local activeCount = 0
for _, c in ipairs(folder:GetChildren()) do
if c.Name == "BombDarkMatter" then activeCount = activeCount + 1 end
end
if activeCount < count then
local myChar = LocalPlayer.Character
local myHead = myChar and myChar:FindFirstChild("Head")
if myHead then
task.spawn(function()
pcall(function() spawnRF:InvokeServer("BombDarkMatter", myHead.CFrame, Vector3.zero) end)
end)
end
end
task.wait(1)
end

if conn then conn:Disconnect() end

if folder and destroyRE then
for _, bh in ipairs(folder:GetChildren()) do
if bh.Name == "BombDarkMatter" then
pcall(function() destroyRE:FireServer(bh) end)
end
end
end
end

function tgtSnowballLoop(targetName, keepGoing)
local target0 = Players:FindFirstChild(targetName)
if target0 and target0.Character then
local tRoot0 = target0.Character:FindFirstChild("HumanoidRootPart")
local tHead0 = target0.Character:FindFirstChild("Head")
local GE0 = getGrabEventsT()
if GE0 then
local destroyGL0 = GE0:FindFirstChild("DestroyGrabLine")
if destroyGL0 then
if tRoot0 then pcall(function() destroyGL0:FireServer(tRoot0) end) end
if tHead0 then pcall(function() destroyGL0:FireServer(tHead0) end) end
end
local sno0 = GE0:FindFirstChild("SetNetworkOwner")
if sno0 and tRoot0 then
pcall(function() sno0:FireServer(tRoot0, tRoot0.CFrame) end)
end
end
end
task.wait(0.05) 
local spawnRF = getSpawnToyRF_T()
local destroyRE = getDestroyToyRE_T()
if not spawnRF then return end
local toyFolderName = LocalPlayer.Name .. "SpawnedInToys"

while keepGoing() do
local target = Players:FindFirstChild(targetName)
if target and target.Character then
local tChar = target.Character
local torso = tChar:FindFirstChild("HumanoidRootPart") or tChar:FindFirstChild("Torso")
if torso then
local offset = Vector3.new(
math.random(-5, 5) / 10,
math.random(-5, 5) / 10,
math.random(-5, 5) / 10
)
task.spawn(function()
pcall(function() spawnRF:InvokeServer("BallSnowball", torso.CFrame * CFrame.new(offset), Vector3.zero) end)
end)

local folder = workspace:FindFirstChild(toyFolderName)
if folder then
for _, snowball in ipairs(folder:GetChildren()) do
if snowball.Name == "BallSnowball" then
local part = snowball:IsA("BasePart") and snowball or snowball.PrimaryPart or snowball:FindFirstChildWhichIsA("BasePart")
if part then
pcall(function()
part.CFrame = torso.CFrame
part.AssemblyLinearVelocity = Vector3.zero
end)
end
end
end
end
end
end
RunService.Heartbeat:Wait()
end

local folder = workspace:FindFirstChild(toyFolderName)
if folder and destroyRE then
for _, snowball in ipairs(folder:GetChildren()) do
if snowball.Name == "BallSnowball" then
pcall(function() destroyRE:FireServer(snowball) end)
end
end
end
end

function tgtBananaLoop(targetName, keepGoing)
local target0 = Players:FindFirstChild(targetName)
if target0 and target0.Character then
local tRoot0 = target0.Character:FindFirstChild("HumanoidRootPart")
local tHead0 = target0.Character:FindFirstChild("Head")
local GE0 = getGrabEventsT()
if GE0 then
local destroyGL0 = GE0:FindFirstChild("DestroyGrabLine")
if destroyGL0 then
if tRoot0 then pcall(function() destroyGL0:FireServer(tRoot0) end) end
if tHead0 then pcall(function() destroyGL0:FireServer(tHead0) end) end
end
local sno0 = GE0:FindFirstChild("SetNetworkOwner")
if sno0 and tRoot0 then
pcall(function() sno0:FireServer(tRoot0, tRoot0.CFrame) end)
end
end
end
task.wait(0.05) 
local spawnRF = getSpawnToyRF_T()
local destroyRE = getDestroyToyRE_T()
if not spawnRF then return end
local toyFolderName = LocalPlayer.Name .. "SpawnedInToys"

local RS = ReplicatedStorage
local holdEvents = RS:FindFirstChild("HoldEvents") or RS:WaitForChild("HoldEvents", 10)

local function FWD(parent, partName, time)
return parent:FindFirstChild(partName) or parent:WaitForChild(partName, time)
end

local function CFP(parent, partName)
return parent:FindFirstChild(partName) ~= nil
end

local function SpawnToyWait(toyName)
local myChar = LocalPlayer.Character
local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
if not myHRP then return nil end

local spawnCF = myHRP.CFrame * CFrame.new(0, 14, 20)
local container = workspace:FindFirstChild(toyFolderName)
if not container then return nil end

local spawnedObject = nil
local conn
conn = container.ChildAdded:Connect(function(child)
if child.Name == toyName then
spawnedObject = child
end
end)

task.spawn(function()
pcall(function() spawnRF:InvokeServer(toyName, spawnCF, Vector3.zero) end)
end)

local startT = tick()
repeat task.wait() until spawnedObject or (tick() - startT) > 2.5
conn:Disconnect()
return spawnedObject
end

local alignPos = nil

while keepGoing() do
local target = Players:FindFirstChild(targetName)
if not target or not target.Character then task.wait(0.1) continue end
local tChar = target.Character
local tLeftLeg = tChar:FindFirstChild("Left Leg")
if not tLeftLeg then task.wait(0.1) continue end

local folder = workspace:FindFirstChild(toyFolderName)
if not folder then task.wait(0.1) continue end

local banana = folder:FindFirstChild("FoodBanana")
local soundPart = banana and banana:FindFirstChild("SoundPart")

if not soundPart then
for _, v in pairs(folder:GetChildren()) do
if v.Name == "FoodBanana" then
pcall(function() destroyRE:FireServer(v) end)
end
end
banana = SpawnToyWait("FoodBanana")
if not banana then task.wait(0.1) continue end
soundPart = FWD(banana, "SoundPart", 5)
if not soundPart then task.wait(0.1) continue end

local holdPart = FWD(banana, "HoldPart", 5)
if holdPart then
local holdRF = FWD(holdPart, "HoldItemRemoteFunction", 5)
if holdRF then
pcall(function() holdRF:InvokeServer(banana, LocalPlayer.Character) end)
end
end

if holdEvents then
local holdUse = holdEvents:FindFirstChild("Use")
if holdUse then
pcall(function() holdUse:FireServer(banana) end)
while CFP(banana, "EdiblePart") and keepGoing() do
task.wait()
end
pcall(function() holdUse:FireServer(banana) end)
end
end

local holdPart2 = banana:FindFirstChild("HoldPart")
if holdPart2 then
local dropRF = holdPart2:FindFirstChild("DropItemRemoteFunction")
if dropRF and LocalPlayer.Character then
pcall(function()
dropRF:InvokeServer(banana, LocalPlayer.Character:GetPivot() * CFrame.new(0, 15, -10), Vector3.zero)
end)
end
end

repeat
task.wait(0.01)
soundPart = banana and banana:FindFirstChild("SoundPart")
if not soundPart then break end
tgtSno(soundPart)
until not soundPart or CFP(soundPart, "PartOwner") or not keepGoing()
if soundPart then
pcall(function() destroyGrabLineEvent:FireServer(soundPart) end)
end

local atach = Instance.new("Attachment")
atach.Parent = soundPart
alignPos = Instance.new("AlignPosition")
alignPos.Responsiveness = 100
alignPos.Parent = soundPart
alignPos.Attachment0 = atach
end

if banana then
for _, v in pairs(banana:GetChildren()) do
if CFP(v, "PartOwner") and not CheckNetworkOwnerShipOnPart(v) then
pcall(function() destroyRE:FireServer(banana) end)
banana = nil
break
end
end
end
if not banana then task.wait(0.1) continue end

alignPos = soundPart:FindFirstChild("AlignPosition")
if not alignPos then
pcall(function() destroyRE:FireServer(banana) end)
banana = nil
task.wait(0.1)
continue
end

local atachNew = tLeftLeg:FindFirstChild("LeftFootAttachment")
if atachNew then
alignPos.Attachment1 = atachNew
end
task.wait()
end

local folder = workspace:FindFirstChild(toyFolderName)
if folder and destroyRE then
for _, banana in ipairs(folder:GetChildren()) do
if banana.Name == "FoodBanana" then
local sp = banana:FindFirstChild("SoundPart")
local ap = sp and sp:FindFirstChild("AlignPosition")
if ap then ap:Destroy() end
pcall(function() destroyRE:FireServer(banana) end)
end
end
end
end

function tgtAntiAntiKickWDLoop(targetName, keepGoing)
local spawnRF = getSpawnToyRF_T()
if not spawnRF then return end

local myInv = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
if not myInv then return end

local GEW = getGrabEventsT()
local snoEvtW = GEW and GEW:FindFirstChild("SetNetworkOwner")
local function snoW(part)
if not part or not part.Parent or not snoEvtW then return end
pcall(function() snoEvtW:FireServer(part, part.CFrame) end)
end

local function FWD(parent, partName, time)
return parent:FindFirstChild(partName) or parent:WaitForChild(partName, time)
end

local function changeCollision(obj, val)
for _, v in pairs(obj:GetChildren()) do
if v:IsA("BasePart") and v.Name ~= "GrabParts" then
v.CanCollide = val
end
end
end

local function spawnToyWD(toyName)
local myChar = LocalPlayer.Character
local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
if not myHRP then return nil end
local spawnCF = myHRP.CFrame * CFrame.new(0, 14, 20)
local container = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
if not container then return nil end
local spawnedObject = nil
local conn
conn = container.ChildAdded:Connect(function(child)
if child.Name == toyName then spawnedObject = child end
end)
task.spawn(function()
pcall(function() spawnRF:InvokeServer(toyName, spawnCF, Vector3.zero) end)
end)
local startT = tick()
repeat task.wait() until spawnedObject or (tick() - startT) > 2.5
conn:Disconnect()
return spawnedObject
end

local function renameInShop(instance, originalName, toRename)
local index = nil
local toCheckName = instance.Name or instance
for i, v in pairs(myInv:GetChildren()) do if v.Name == toCheckName then index = i end end
if index == nil then return end
local contents = LocalPlayer.PlayerGui.MenuGui.Menu.TabContents.ToyDestroy.Contents
for i, v in ipairs(contents:GetChildren()) do
if v.Name == originalName and i == index then
local view = v.ViewItemButton
view.Text = toRename
view.TextScaled = true
view.LowResImage.Image = ""
end
end
end

local WD = myInv:FindFirstChild("SprayCanWD")
local SoundPart = WD and WD:FindFirstChild("SoundPart")

if not WD then
WD = spawnToyWD("SprayCanWD")
if not WD then return end
repeat task.wait() until WD and WD:FindFirstChild("SoundPart") or not keepGoing()
if not WD then return end
SoundPart = WD:FindFirstChild("SoundPart") or WD:WaitForChild("SoundPart", 3)
changeCollision(WD, false)
task.delay(1, function()
if WD and WD.Parent then pcall(renameInShop, WD, WD.Name, "Anti-AntIkick") end
end)
end

local HitBox = FWD(WD, "Hitbox", 5)
if HitBox then
repeat
snoW(HitBox)
task.wait(0.05)
until not HitBox.Parent or not keepGoing() or HitBox:FindFirstChild("PartOwner")
end

Library:Notify(L("Anti-AntiKick [WD]: работаем по " .. targetName, "Anti-AntiKick [WD]: running on " .. targetName), 2)

while keepGoing() and task.wait(0.2) do
SoundPart = WD and WD.Parent and WD:FindFirstChild("SoundPart")
local target = Players:FindFirstChild(targetName)
local tRoot = target and target.Character and target.Character:FindFirstChild("HumanoidRootPart")

if not tRoot then
repeat
task.wait(0.1)
target = Players:FindFirstChild(targetName)
tRoot = target and target.Character and target.Character:FindFirstChild("HumanoidRootPart")
until tRoot or not keepGoing()
end

if not SoundPart then
WD = spawnToyWD("SprayCanWD")
if not WD then task.wait(0.5) continue end
repeat task.wait() until WD and WD:FindFirstChild("SoundPart") or not keepGoing()
if not WD then task.wait(0.5) continue end
SoundPart = WD:FindFirstChild("SoundPart") or WD:WaitForChild("SoundPart", 3)
changeCollision(WD, false)
HitBox = FWD(WD, "Hitbox", 5)
if HitBox then
repeat
snoW(HitBox)
task.wait(0.05)
until not HitBox.Parent or HitBox:FindFirstChild("PartOwner") or not keepGoing()
end
task.delay(1, function()
if WD and WD.Parent then pcall(renameInShop, WD, WD.Name, "Anti-Antikick") end
end)
end

if SoundPart and tRoot then
pcall(function() SoundPart.CFrame = tRoot.CFrame * CFrame.new(0, 0, 4) end)
task.wait()
pcall(function()
SoundPart.CFrame = CFrame.new(-382.838318, 41.6491699, 665.570251)
SoundPart.AssemblyAngularVelocity = Vector3.zero
SoundPart.AssemblyLinearVelocity = Vector3.zero
end)
end
end
end

function tgtAntiAntiKickBlackHoleLoop(targetName, keepGoing)
local target0 = Players:FindFirstChild(targetName)
if target0 and target0.Character then
local tRoot0 = target0.Character:FindFirstChild("HumanoidRootPart")
local tHead0 = target0.Character:FindFirstChild("Head")
local GE0 = getGrabEventsT()
if GE0 then
local destroyGL0 = GE0:FindFirstChild("DestroyGrabLine")
if destroyGL0 then
if tRoot0 then pcall(function() destroyGL0:FireServer(tRoot0) end) end
if tHead0 then pcall(function() destroyGL0:FireServer(tHead0) end) end
end
local sno0 = GE0:FindFirstChild("SetNetworkOwner")
if sno0 and tRoot0 then
pcall(function() sno0:FireServer(tRoot0, tRoot0.CFrame) end)
end
end
end
task.wait(0.05) 
local spawnRF = getSpawnToyRF_T()
local destroyRE = getDestroyToyRE_T()
if not spawnRF then return end

local myInv = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
if not myInv then return end

local function FWD(parent, partName, time)
return parent:FindFirstChild(partName) or parent:WaitForChild(partName, time)
end

local playerEvents = ReplicatedStorage:FindFirstChild("PlayerEvents")
local stickyEvent = playerEvents and (playerEvents:FindFirstChild("StickyPartEvent") or playerEvents:WaitForChild("StickyPartEvent", 5))
if not stickyEvent then
for _, obj in ipairs(ReplicatedStorage:GetDescendants()) do
if obj.Name == "StickyPartEvent" and obj:IsA("RemoteEvent") then
stickyEvent = obj
break
end
end
end

local shurikensToBlackhole = {}

while keepGoing() do
local target = Players:FindFirstChild(targetName)
local tChar = target and target.Character
local tHRP = tChar and tChar:FindFirstChild("HumanoidRootPart")
local firePlayerPart = tHRP and tHRP:FindFirstChild("FirePlayerPart")

if not stickyEvent then
Library:Notify(L("AntiAntiKick BlackHole: StickyPartEvent не найден в PlayerEvents", "AntiAntiKick BlackHole: StickyPartEvent not found in PlayerEvents"), 3)
return
end
if not firePlayerPart then
Library:Notify(L("AntiAntiKick BlackHole: FirePlayerPart не найден у цели", "AntiAntiKick BlackHole: FirePlayerPart not found on target"), 3)
return
end

for _ = 1, #shurikensToBlackhole do table.remove(shurikensToBlackhole) end

local myChar = LocalPlayer.Character
local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
if not myHRP then task.wait(0.5) continue end
local spawnCF = myHRP.CFrame * CFrame.new(0, 14, 20)

for i = 1, 10 do
if not keepGoing() then break end
local Shur = nil
pcall(function() Shur = spawnRF:InvokeServer("NinjaKunai", spawnCF, Vector3.zero) end)
if not Shur or not Shur.Parent then
Shur = myInv:FindFirstChild("NinjaKunai")
end
if Shur then
local StickyPart = Shur:FindFirstChild("StickyPart")
if StickyPart then
pcall(function() setNetworkOwnerEvent:FireServer(StickyPart, StickyPart.CFrame) end)
local bodyPos = Instance.new("BodyPosition")
bodyPos.Position = Vector3.new(math.random(-100, 100), 1e3, math.random(-100, 100))
bodyPos.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bodyPos.Parent = StickyPart
pcall(function()
stickyEvent:FireServer(StickyPart, firePlayerPart, CFrame.Angles(0, math.rad(90), math.rad(90)))
end)
table.insert(shurikensToBlackhole, StickyPart)
Shur.Name = "Anti-AntiKick(Kunai)"
end
end
end

for repeatIdx = 1, 2 do
for _, v in pairs(shurikensToBlackhole) do
if not v or not v.Parent then continue end
pcall(function()
stickyEvent:FireServer(v, firePlayerPart, CFrame.Angles(0, math.rad(90), math.rad(90)))
end)
end
end

task.wait(2)
end

if myInv and destroyRE then
for _, shur in ipairs(myInv:GetChildren()) do
if shur.Name == "Anti-AntiKick(Kunai)" or shur.Name == "NinjaKunai" then
pcall(function() destroyRE:FireServer(shur) end)
end
end
end
end

function tgtAntiAntiKickClickLoop(targetName, keepGoing)
local GEC = getGrabEventsT()
local snoEvtC = GEC and GEC:FindFirstChild("SetNetworkOwner")
if not snoEvtC then
Library:Notify(L("AntiAntiKick CLICK: SetNetworkOwner не найден", "AntiAntiKick CLICK: SetNetworkOwner not found"), 3)
return
end
local function snoC(part)
if not part or not part.Parent then return end
pcall(function() snoEvtC:FireServer(part, part.CFrame) end)
end
local TargetInv = workspace:FindFirstChild(targetName .. "SpawnedInToys")
local TargetPLR = Players:FindFirstChild(targetName)
Library:Notify(L("Anti-AntiKick [CLICK]: кликаем по " .. targetName, "Anti-AntiKick [CLICK]: clicking on " .. targetName), 2)
while keepGoing() and task.wait() do
TargetInv = (TargetInv and TargetInv.Parent) and TargetInv or workspace:FindFirstChild(targetName .. "SpawnedInToys")
TargetPLR = (TargetPLR and TargetPLR.Parent) and TargetPLR or Players:FindFirstChild(targetName)
local Root = TargetPLR and TargetPLR.Character and TargetPLR.Character:FindFirstChild("HumanoidRootPart")
if not Root then continue end
local FirePlayerPart = Root:FindFirstChild("FirePlayerPart")
if not FirePlayerPart then continue end
if not TargetInv then continue end
for _, v in pairs(TargetInv:GetChildren()) do
local StickyPart = v:FindFirstChild("StickyPart")
if StickyPart then
if StickyPart.CanQuery then
task.spawn(function()
for _, part in pairs(v:GetChildren()) do
if part:IsA("BasePart") then
part.CanCollide = false
part.CanTouch = false
part.CanQuery = false
end
end
end)
end
local po = StickyPart:FindFirstChild("PartOwner")
if not (po and po.Value == LocalPlayer.Name) then
snoC(StickyPart)
end
end
end
end
end

function tgtDontGiveChanceLoop(targetName, keepGoing)
local GED = getGrabEventsT()
local snoEvtD = GED and GED:FindFirstChild("SetNetworkOwner")
if not snoEvtD then
Library:Notify(L("DontGiveChance: SetNetworkOwner не найден", "DontGiveChance: SetNetworkOwner not found"), 3)
return
end
local function snoD(part)
if not part or not part.Parent then return end
pcall(function() snoEvtD:FireServer(part, part.CFrame) end)
end
Library:Notify(L("DontGiveChance: следим за " .. targetName, "DontGiveChance: watching " .. targetName), 2)
while keepGoing() do
task.wait(0.1)
local target = Players:FindFirstChild(targetName)
local tChar = target and target.Character
local tHum = tChar and tChar:FindFirstChildOfClass("Humanoid")
if not (tChar and tHum) then continue end
if tHum.SeatPart ~= nil then
local tRoot = tChar:FindFirstChild("HumanoidRootPart")
local tHead = tChar:FindFirstChild("Head")
local myChar = LocalPlayer.Character
local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
if not (tRoot and tHead and myHRP) then continue end
local oldCF = myHRP.CFrame
task.wait(0.1)
pcall(function() myHRP.CFrame = tRoot.CFrame * CFrame.new(0, 5, 5) end)
task.wait(0.2)
snoD(tHead)
pcall(function() myHRP.CFrame = oldCF end)
task.wait(0.5)
end
end
end

function tgtPalletRagdollLoop(targetName, keepGoing) 
local target = Players:FindFirstChild(targetName)
if not target or not target.Character then return end
local RS = game:GetService("ReplicatedStorage")
local GE = RS:WaitForChild("GrabEvents")
local RunService = game:GetService("RunService")
local skyPos = CFrame.new(0, 800000, 0)
RS.MenuToys.SpawnToyRemoteFunction:InvokeServer("PalletLightBrown", skyPos, Vector3.zero)
local pallet
repeat
pallet = workspace:FindFirstChild(LocalPlayer.Name.."SpawnedInToys")
and workspace[LocalPlayer.Name.."SpawnedInToys"]:FindFirstChild("PalletLightBrown")
RunService.Heartbeat:Wait()
until pallet or not keepGoing()
if not pallet then return end
local mainPart = pallet:FindFirstChild("SoundPart")
if not mainPart then return end
mainPart.CanCollide = false
mainPart.Anchored = false
local function claim(part)
GE.SetNetworkOwner:FireServer(part, part.CFrame)
GE.CreateGrabLine:FireServer(part, Vector3.zero, part.Position, false)
GE.DestroyGrabLine:FireServer(part)
end
claim(mainPart)
Library:Notify(L("PalletRagdoll v45: молотим " .. targetName, "PalletRagdoll v45: smashing " .. targetName), 2) 
while keepGoing() do
for i = 1, 20 do
RunService.Heartbeat:Wait()
end
if not target or not target.Parent or not target.Character then break end
local head = target.Character:FindFirstChild("Head")
if not head then continue end
local targetPos = head.Position
mainPart.CFrame = CFrame.new(targetPos.X, targetPos.Y + 0.2, targetPos.Z) 
mainPart.AssemblyLinearVelocity = Vector3.zero
mainPart.AssemblyAngularVelocity = Vector3.new(1000, 1000, 1000)
claim(mainPart)
mainPart.CanCollide = true
for i = 1, 3 do
RunService.Heartbeat:Wait()
end
mainPart.CanCollide = false
mainPart.CFrame = skyPos
mainPart.AssemblyAngularVelocity = Vector3.zero
end
if pallet then
pcall(function()
RS.MenuToys.DestroyToy:FireServer(pallet)
end)
if pallet.Parent then
pallet:Destroy()
end
end
end

function tgtRemoveTargetGucciLoop(targetName, keepGoing)
local myChar = LocalPlayer.Character
local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
if not myRoot then
Library:Notify(L("Remove Target Gucci: нет персонажа", "Remove Target Gucci: no character"), 3)
return
end
local SafeSpot = myRoot.CFrame
Library:Notify("Remove Target Gucci: слежу за " .. targetName, 2)

while keepGoing() do
local target = Players:FindFirstChild(targetName)
if not target or not target.Parent then
task.wait(0.5)
continue
end

local folderName = target.Name .. "SpawnedInToys"
local toysFolder = workspace:FindFirstChild(folderName)
if not toysFolder then
task.wait(1)
continue
end

local foundBlobman = false
for _, obj in ipairs(toysFolder:GetChildren()) do
if not keepGoing() then break end
if obj.Name == "CreatureBlobman" then
foundBlobman = true
local seat = obj:FindFirstChild("VehicleSeat") or obj:FindFirstChildWhichIsA("VehicleSeat", true)
if seat then
local myChar2 = LocalPlayer.Character
local myRoot2 = myChar2 and myChar2:FindFirstChild("HumanoidRootPart")
local myHum2 = myChar2 and myChar2:FindFirstChildOfClass("Humanoid")
if myRoot2 and myHum2 then
if myHum2.SeatPart ~= seat then
Library:Notify(L("Remove Target Gucci: сажусь на блобман цели", "Remove Target Gucci: sitting on target blobman"), 2)
local magnetConn = RunService.Stepped:Connect(function()
if myRoot2 and seat and seat.Parent then
myRoot2.CFrame = seat.CFrame
myRoot2.Velocity = Vector3.zero
if obj.PrimaryPart then
obj.PrimaryPart.Velocity = Vector3.zero
obj.PrimaryPart.RotVelocity = Vector3.zero
end
end
end)
local sitStart = tick()
while tick() - sitStart < 1 do
if not keepGoing() then break end
if myHum2.SeatPart == seat then break end
seat:Sit(myHum2)
task.wait()
end
if magnetConn then magnetConn:Disconnect() end
if myHum2.SeatPart == seat then
task.wait(0.3)
myHum2.Sit = false
myHum2.Jump = true
task.wait(0.05)
myRoot2.CFrame = SafeSpot
myRoot2.Velocity = Vector3.zero
Library:Notify("Remove Target Gucci: Gucci украден у " .. targetName, 2)
task.wait(0.5)
else
myRoot2.CFrame = SafeSpot
end
end
end
end
end
end

if not foundBlobman then
task.wait(1)
end
end

local finalChar = LocalPlayer.Character
local finalRoot = finalChar and finalChar:FindFirstChild("HumanoidRootPart")
if finalRoot and SafeSpot then
finalRoot.CFrame = SafeSpot
finalRoot.Velocity = Vector3.zero
end
end

function tgtHeavenLoop(targetName, keepGoing)
local target0 = Players:FindFirstChild(targetName)
if target0 and target0.Character then
local tRoot0 = target0.Character:FindFirstChild("HumanoidRootPart")
local GE0 = getGrabEventsT()
if GE0 and tRoot0 then
local destroyGL0 = GE0:FindFirstChild("DestroyGrabLine")
if destroyGL0 then pcall(function() destroyGL0:FireServer(tRoot0) end) end
local sno0 = GE0:FindFirstChild("SetNetworkOwner")
if sno0 then pcall(function() sno0:FireServer(tRoot0, tRoot0.CFrame) end) end
end
end
task.wait(0.05)

local GE = getGrabEventsT()
if not GE then return end
local SetNetworkOwner = GE:FindFirstChild("SetNetworkOwner")
local DestroyGrabLine = GE:FindFirstChild("DestroyGrabLine")
if not SetNetworkOwner then return end
local Debris = game:GetService("Debris")

while keepGoing() do
local target = Players:FindFirstChild(targetName)
if not target or not target.Character then task.wait(0.1) continue end
local tRoot = target.Character:FindFirstChild("HumanoidRootPart")
local tHum = target.Character:FindFirstChildOfClass("Humanoid")
if not (tRoot and tHum) or tHum.Health <= 0 then task.wait(0.1) continue end

pcall(function()
SetNetworkOwner:FireServer(tRoot, tRoot.CFrame)
if DestroyGrabLine then DestroyGrabLine:FireServer(tRoot) end
tRoot.CFrame = CFrame.new(0, 200, 0)
local bv = Instance.new("BodyVelocity")
bv.Name = "OblivionHeaven"
bv.MaxForce = Vector3.new(0, math.huge, 0)
bv.Velocity = Vector3.new(0, 200, 0)
bv.P = 12500
bv.Parent = tRoot
Debris:AddItem(bv, 0.01)
end)
RunService.Heartbeat:Wait()
end
end

OwnershipKickEnabled = false
OwnershipKickTask = nil

local function getKickRemoteDelay()
local bv = Options.BmSpamDelay and Options.BmSpamDelay.Value or 0.05
return bv
end

local function getKickGrabWait()
local bv = Options.BmSpamDelay and Options.BmSpamDelay.Value or 0.05
return bv * 6 
end

function tgtOwnershipKickLoop(targetName, keepGoing) 
OwnershipKickEnabled = true
local target = Players:FindFirstChild(targetName)
if not target then
OwnershipKickEnabled = false
return
end
local RS = game:GetService("ReplicatedStorage")
local GE = RS:WaitForChild("GrabEvents")
local RunService = game:GetService("RunService")
local myChar = LocalPlayer.Character
local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
if not myRoot then
OwnershipKickEnabled = false
return
end
local savedPos = myRoot.CFrame 
local lastTargetChar = target.Character 
local lastMyChar = myChar 
local respawnWaitUntil = 0 
local spawnFFDeadline = 0 
local failApproaches = 0 
local dragging = false
local grabStartTime = 0
local checkStartTime = 0
local currentFPS = 60
local fpsConnection = RunService.RenderStepped:Connect(function(dt)
currentFPS = 1 / dt
end)
local bodyPos = nil
local bodyGyro = nil
local function cleanupBodies()
pcall(function()
if bodyPos then bodyPos:Destroy() bodyPos = nil end
if bodyGyro then bodyGyro:Destroy() bodyGyro = nil end
end)
end
local function createBodies(targetRoot, pos)
cleanupBodies()
for _, v in pairs(targetRoot:GetChildren()) do
if v:IsA("BodyPosition") or v:IsA("BodyGyro") then
v:Destroy()
end
end
bodyPos = Instance.new("BodyPosition")
bodyPos.MaxForce = Vector3.new(9e9, 9e9, 9e9)
bodyPos.D = 100
bodyPos.Position = pos
bodyPos.Parent = targetRoot
bodyGyro = Instance.new("BodyGyro")
bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
bodyPos.D = 100
bodyGyro.CFrame = CFrame.new(pos)
bodyGyro.Parent = targetRoot
end
while keepGoing() and OwnershipKickEnabled do
local currentTarget = Players:FindFirstChild(target.Name)
if not currentTarget or not currentTarget.Parent then
cleanupBodies()
break
end
myChar = LocalPlayer.Character
myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
local tChar = currentTarget.Character
local tRoot = tChar and tChar:FindFirstChild("HumanoidRootPart")
local tHum = tChar and tChar:FindFirstChild("Humanoid")
if myChar ~= lastMyChar then 
lastMyChar = myChar
if myRoot then savedPos = myRoot.CFrame end
end
if tChar ~= lastTargetChar then 
lastTargetChar = tChar
dragging = false
grabStartTime = 0
checkStartTime = 0
failApproaches = 0
cleanupBodies()
respawnWaitUntil = tick() + 1.2
spawnFFDeadline = tick() + 5
end
if tRoot and tHum and tHum.Health > 0 and myRoot then
if not dragging then
local spawnFF = tChar:FindFirstChildOfClass("ForceField") 
if tick() < respawnWaitUntil or (spawnFF and tick() < spawnFFDeadline) then
RunService.Heartbeat:Wait()
continue
end
myRoot.CFrame = tRoot.CFrame * CFrame.new(0, 0, 3) 
cleanupBodies()
checkStartTime = 0
pcall(function()
tHum.PlatformStand = true
tHum.Sit = true
GE.SetNetworkOwner:FireServer(tRoot, tRoot.CFrame)
GE.SetNetworkOwner:FireServer(tRoot, tRoot.CFrame)
GE.DestroyGrabLine:FireServer(tRoot)
end)
myRoot.AssemblyLinearVelocity = Vector3.zero
myRoot.AssemblyAngularVelocity = Vector3.zero
if grabStartTime == 0 then grabStartTime = tick() end
if tick() - grabStartTime > 0.35 then
dragging = true
grabStartTime = 0
checkStartTime = tick()
local lockPos = savedPos * CFrame.new(0, 17, 0)
createBodies(tRoot, lockPos.Position)
end
else
myRoot.CFrame = savedPos 
local lockPos = savedPos * CFrame.new(0, 17, 0)
myRoot.AssemblyLinearVelocity = Vector3.zero
myRoot.AssemblyAngularVelocity = Vector3.zero
if bodyPos and bodyPos.Parent then
bodyPos.Position = lockPos.Position
if bodyGyro then
bodyGyro.CFrame = lockPos
end
else
createBodies(tRoot, lockPos.Position)
end
tHum.PlatformStand = true
pcall(function()
if currentFPS > 200 then
GE.SetNetworkOwner:FireServer(tRoot, lockPos)
GE.SetNetworkOwner:FireServer(tRoot, lockPos)
GE.DestroyGrabLine:FireServer(tRoot)
elseif currentFPS >= 155 and currentFPS <= 200 then
GE.SetNetworkOwner:FireServer(tRoot, lockPos)
GE.SetNetworkOwner:FireServer(tRoot, lockPos)
GE.SetNetworkOwner:FireServer(tRoot, lockPos)
GE.DestroyGrabLine:FireServer(tRoot)
else
GE.SetNetworkOwner:FireServer(tRoot, lockPos)
GE.SetNetworkOwner:FireServer(tRoot, lockPos)
GE.SetNetworkOwner:FireServer(tRoot, lockPos)
GE.SetNetworkOwner:FireServer(tRoot, lockPos)
GE.DestroyGrabLine:FireServer(tRoot)
end
end)
if checkStartTime > 0 and tick() - checkStartTime > 0.30 then
local currentDist = (tRoot.Position - lockPos.Position).Magnitude
if currentDist > 10 then
dragging = false
grabStartTime = 0
checkStartTime = 0
cleanupBodies()
failApproaches = failApproaches + 1 
if failApproaches >= 5 then 
failApproaches = 0
respawnWaitUntil = tick() + 2
end
myRoot.CFrame = tRoot.CFrame * CFrame.new(0, 0, 3)
else
checkStartTime = tick()
failApproaches = 0 
end
end
end
else
dragging = false
grabStartTime = 0
checkStartTime = 0
cleanupBodies()
end
RunService.Heartbeat:Wait()
end
fpsConnection:Disconnect()
cleanupBodies()
myChar = LocalPlayer.Character
myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
if myRoot then myRoot.CFrame = savedPos end
OwnershipKickEnabled = false
end

_tgtLineLagEnabled = false
_tgtLineLagThread = nil
_tgtLineLagPart = nil

function tgtStartLineLag()
if _tgtLineLagEnabled then return end
_tgtLineLagEnabled = true
_tgtLineLagThread = coroutine.create(function()
local GrabEvents = ReplicatedStorage:FindFirstChild("GrabEvents")
if not GrabEvents then return end
local createLine = GrabEvents:FindFirstChild("CreateGrabLine")
if not createLine then return end
while _tgtLineLagEnabled do
local spawnLocation = workspace:FindFirstChild("SpawnLocation") or workspace:FindFirstChild("Spawn") or (LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart"))
if spawnLocation then
_tgtLineLagPart = spawnLocation
local randomX = math.random(-1e9, 1e9)
local randomZ = math.random(-1e9, 1e9)
local directions = {
CFrame.new(randomX, 0, randomZ),
CFrame.new(-randomX, 0, -randomZ),
CFrame.new(randomX, 0, -randomZ),
CFrame.new(-randomX, 0, randomZ)
}
for _, pos in pairs(directions) do
pcall(function() createLine:FireServer(spawnLocation, pos) end)
end
end
task.wait()
end
end)
coroutine.resume(_tgtLineLagThread)
end

function tgtStopLineLag()
_tgtLineLagEnabled = false
if _tgtLineLagThread then
pcall(function() coroutine.close(_tgtLineLagThread) end)
_tgtLineLagThread = nil
end
local purgePart = _tgtLineLagPart
_tgtLineLagPart = nil
if purgePart then
local GE2 = ReplicatedStorage:FindFirstChild("GrabEvents")
local createLine2 = GE2 and GE2:FindFirstChild("CreateGrabLine")
local destroyLine2 = GE2 and GE2:FindFirstChild("DestroyGrabLine")
if createLine2 and destroyLine2 then
task.spawn(function()
pcall(function() createLine2:FireServer(purgePart, CFrame.new(3e9, 4e9, 2e9)) end)
task.wait()
for _ = 1, 8 do
pcall(function() destroyLine2:FireServer(purgePart) end)
task.wait(0.1)
end
end)
end
end
end

function bmTestBombaMassKick(keepGoing)
while true do
if keepGoing and not keepGoing() then break end
if not keepGoing and not bmLoopServer then break end

local heightMode = (Options.DestroyHeightMode and Options.DestroyHeightMode.Value) or "Spawn"
local height = (heightMode == "Heaven") and 1e9 or 35

tgtStartLineLag()
task.wait(1)

local players = {}
for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer and not isFriendWhitelisted(plr) then
table.insert(players, plr)
end
end
if #players == 0 then
tgtStopLineLag()
task.wait(1)
continue
end

local myChar = LocalPlayer.Character
local myHrp = myChar and myChar:FindFirstChild("HumanoidRootPart")
if not myHrp then tgtStopLineLag() task.wait(1) continue end

local savedPos = myHrp.CFrame

local playerData = {}
for _, plr in ipairs(players) do
local char = plr.Character
local hrp = char and char:FindFirstChild("HumanoidRootPart")
if hrp then table.insert(playerData, {player = plr, hrp = hrp}) end
end

for _, data in ipairs(playerData) do
pcall(function() myHrp.CFrame = data.hrp.CFrame * CFrame.new(0, 5, 5) end)
task.wait(0.2)
local GE = ReplicatedStorage:FindFirstChild("GrabEvents")
local setOwner = GE and GE:FindFirstChild("SetNetworkOwner")
if setOwner and data.hrp then
pcall(function() setOwner:FireServer(data.hrp, data.hrp.CFrame) end)
end
task.wait()
end

pcall(function()
myHrp.CFrame = savedPos
myHrp.AssemblyLinearVelocity = Vector3.zero
end)

local radius = 40
local angleStep = (math.pi * 2) / #playerData
for idx, data in ipairs(playerData) do
local angle = (idx - 1) * angleStep
local x = math.cos(angle) * radius
local z = math.sin(angle) * radius

pcall(function()
data.hrp.CFrame = CFrame.new(x, height, z)
data.hrp.AssemblyLinearVelocity = Vector3.zero
end)

local bp = Instance.new("BodyPosition")
bp.MaxForce = Vector3.new(1e9, 1e9, 1e9)
bp.P = 40000000
bp.Position = Vector3.new(x, height, z)
bp.Parent = data.hrp
task.delay(2, function() pcall(function() bp:Destroy() end) end)
task.wait()
end

local GE = ReplicatedStorage:FindFirstChild("GrabEvents")
local destroyLine = GE and GE:FindFirstChild("DestroyGrabLine")
if destroyLine then
for i = 1, 8 do
for _, data in ipairs(playerData) do
pcall(function() destroyLine:FireServer(data.hrp) end)
end
task.wait(0.3)
end
end

tgtStopLineLag()

pcall(function()
if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
LocalPlayer.Character.HumanoidRootPart.CFrame = savedPos
LocalPlayer.Character.HumanoidRootPart.AssemblyLinearVelocity = Vector3.zero
end
end)

task.wait(1)
end
end

function bmBlobHoldEngine(targetName, keepGoing, reacquire, maxSeconds, mode)
local fnStart = tick()
bmKickingInProgress = true

local myChar = LocalPlayer.Character
local myHum = myChar and myChar:FindFirstChildOfClass("Humanoid")
local seat = myHum and myHum.SeatPart
if not (seat and seat.Parent and seat.Parent.Name == "CreatureBlobman") then
if not sitOnBlobman() then
Library:Notify(L("Blobman: не удалось сесть в блодмана", "Blobman: blobman sit failed"), 3)
bmKickingInProgress = false
return
end
task.wait(0.3)
end

local GE = GE_Bloodman or ReplicatedStorage:FindFirstChild("GrabEvents")
if not GE then
Library:Notify(L("Blobman: GrabEvents не найден", "Blobman: GrabEvents not found"), 3)
bmKickingInProgress = false
return
end

myChar = LocalPlayer.Character
local myHum0 = myChar and myChar:FindFirstChildOfClass("Humanoid")
local seat0 = myHum0 and myHum0.SeatPart
local blobRoot0 = nil
if seat0 and seat0.Parent and seat0.Parent.Name == "CreatureBlobman" then
local blob0 = seat0.Parent
blobRoot0 = blob0:FindFirstChild("HumanoidRootPart") or blob0.PrimaryPart
end
if not blobRoot0 then
bmKickingInProgress = false
return
end

local cFrame = nil
local skyFloor = nil

if mode == "sky" then
skyFloor = Instance.new("Part")
skyFloor.Name = "OblivionSkyKickFloor"
skyFloor.Anchored = true
skyFloor.CanCollide = true
skyFloor.Size = Vector3.new(50, 1, 50)
skyFloor.CFrame = CFrame.new(20000, 20000, 20000)
skyFloor.BrickColor = BrickColor.new("White")
skyFloor.Material = Enum.Material.SmoothPlastic
skyFloor.Parent = Workspace
cFrame = CFrame.new(20000, 20010.5, 20000)
pcall(function() blobRoot0.CFrame = cFrame end)
local myHRP0 = myChar:FindFirstChild("HumanoidRootPart")
if myHRP0 then pcall(function() myHRP0.CFrame = cFrame end) end
task.wait(0.2)
elseif mode == "lock" or mode == "spin" then
cFrame = nil
else
cFrame = blobRoot0.CFrame
end

local flag = false
local n = 0
local n2 = 0
local preDragCFrame = nil

while keepGoing() do
if maxSeconds and (tick() - fnStart) > maxSeconds then break end

local target = Players:FindFirstChild(targetName)
if not target or not target.Parent then
if not reacquire then break end
flag = false
n2 = 0
RunService.Heartbeat:Wait()
continue
end

local myCharL = LocalPlayer.Character
local myHumL = myCharL and myCharL:FindFirstChildOfClass("Humanoid")
local seatL = myHumL and myHumL.SeatPart
if not (seatL and seatL.Parent and seatL.Parent.Name == "CreatureBlobman") then
break
end
local blobL = seatL.Parent
local blobRootL = blobL:FindFirstChild("HumanoidRootPart") or blobL.PrimaryPart
local remoteFolderL = blobL:FindFirstChild("BlobmanSeatAndOwnerScript")
local creatureGrabL = remoteFolderL and remoteFolderL:FindFirstChild("CreatureGrab")
local creatureDropL = remoteFolderL and remoteFolderL:FindFirstChild("CreatureDrop")
local rightDetectorL = blobL:FindFirstChild("RightDetector")

local tChar = target.Character
local tRoot = tChar and tChar:FindFirstChild("HumanoidRootPart")
local tHum = tChar and tChar:FindFirstChildOfClass("Humanoid")

if tRoot and tHum and tHum.Health > 0 and blobRootL then
tRoot.Velocity = Vector3.zero

if not flag then
if n2 == 0 then
n2 = tick()
preDragCFrame = blobRootL.CFrame
end
blobRootL.CFrame = tRoot.CFrame
blobRootL.Velocity = Vector3.zero

if tick() - n >= 0.002 then
n = tick()
pcall(function()
tHum.PlatformStand = true
tHum.Sit = true
if GE.SetNetworkOwner then GE.SetNetworkOwner:FireServer(tRoot, blobRootL.CFrame) end
if GE.DestroyGrabLine then GE.DestroyGrabLine:FireServer(tRoot) end
end)
end

if tick() - n2 > 0.35 then
local backCFrame = (mode == "lock" or mode == "spin") and preDragCFrame or cFrame
if backCFrame then
blobRootL.CFrame = backCFrame
blobRootL.Velocity = Vector3.zero
end
flag = true
n2 = 0
end
else
local cFrame2 = nil
if mode == "spin" then
local spinR = (Options.SpinKickRadius and Options.SpinKickRadius.Value) or bmSpinRadius or 25
local spinS = (Options.SpinKickSpeed and Options.SpinKickSpeed.Value) or bmSpinSpeed or 0.25
local spinH = (Options.SpinKickHeight and Options.SpinKickHeight.Value) or bmSpinKickHeight or 20
bmSpinAngle = bmSpinAngle + spinS
if bmSpinAngle > 6.283185 then bmSpinAngle = bmSpinAngle - 6.283185 end
cFrame2 = blobRootL.CFrame * CFrame.new(math.cos(bmSpinAngle) * spinR, spinH, math.sin(bmSpinAngle) * spinR)
else
if mode ~= "lock" then
blobRootL.CFrame = cFrame
blobRootL.Velocity = Vector3.zero
end
cFrame2 = (mode == "lock") and (blobRootL.CFrame * CFrame.new(0, 23, 0)) or (cFrame * CFrame.new(0, 23, 0))
end
tRoot.CFrame = cFrame2
tHum.PlatformStand = true
tHum.Sit = true

if tick() - n >= 0.002 then
n = tick()
pcall(function()
if GE.SetNetworkOwner then GE.SetNetworkOwner:FireServer(tRoot, cFrame2) end
if GE.DestroyGrabLine then GE.DestroyGrabLine:FireServer(tRoot) end
local rightWeld = rightDetectorL and (rightDetectorL:FindFirstChild("RightWeld") or rightDetectorL:FindFirstChildWhichIsA("Weld"))
if rightWeld and creatureDropL and creatureGrabL then
creatureDropL:FireServer(rightWeld)
creatureGrabL:FireServer(rightDetectorL, tRoot, rightWeld)
end
end)
end
end
else
flag = false
n2 = 0
end

RunService.Heartbeat:Wait()
end

if blobRoot0 and blobRoot0.Parent and cFrame then
pcall(function()
blobRoot0.CFrame = cFrame
blobRoot0.Velocity = Vector3.zero
end)
end
if skyFloor and skyFloor.Parent then skyFloor:Destroy() end
bmKickingInProgress = false
end

function bmTargetInPlot(targetName)
local ok, res = pcall(function()
local plr = Players:FindFirstChild(targetName)
if not plr then return false end
local v = plr:FindFirstChild("InPlot")
if v and v.Value == true then return true end
local pi = Workspace:FindFirstChild("PlotItems")
local pip = pi and pi:FindFirstChild("PlayersInPlots")
return (pip and pip:FindFirstChild(plr.Name) ~= nil) or false
end)
return (ok and res) or false
end

function bmKickThroughPlot(targetName, keepGoing, maxSeconds)
local target = Players:FindFirstChild(targetName)
if not target then
Library:Notify(L("Блодман: игрок не найден", "Blobman: player not found"), 3)
return
end
local t0 = tick()
while keepGoing() and target.Parent do
tbl20.SelectedPlayers[target] = true
tbl20.TargetMethods = { Kick = true }
tbl19.LoopApplyMethodT = true
if maxSeconds and (tick() - t0) > maxSeconds then break end
task.wait(0.05)
end
tbl20.SelectedPlayers[target] = nil
if next(tbl20.SelectedPlayers) == nil then
tbl19.LoopApplyMethodT = false
pcall(tbl17.clearAllQueues, "LoopApplyMethodT")
end
task.spawn(function()
pcall(cleanupNetworkKickPlayer, target)
end)
end

function bmKickTarget(targetName, keepGoing, reacquire, maxSeconds)
if bmTargetInPlot(targetName) then
Library:Notify(L("Блодман: цель в плоте — сетевой кик", "Blobman: target in plot — network kick"), 3)
bmKickThroughPlot(targetName, keepGoing, maxSeconds)
return
end
bmBlobHoldEngine(targetName, keepGoing, reacquire, maxSeconds, "blob")
end

function bmLockTarget(targetName, keepGoing, reacquire, maxSeconds)
bmBlobHoldEngine(targetName, keepGoing, reacquire, maxSeconds, "lock")
end

function bmLockServer(keepGoing)
for _, plr in ipairs(Players:GetPlayers()) do
if keepGoing and not keepGoing() then return end
if plr ~= LocalPlayer and not isFriendWhitelisted(plr) then
bmLockTarget(plr.Name, function()
return (not keepGoing) or keepGoing()
end, false, 12)
end
end
end


function bmKickServer(keepGoing)
for _, plr in ipairs(Players:GetPlayers()) do
if keepGoing and not keepGoing() then return end
if plr ~= LocalPlayer and not isFriendWhitelisted(plr) then
bmKickTarget(plr.Name, function()
return (not keepGoing) or keepGoing()
end, false, 12)
end
end
end

_toyHitboxNames = {
BombMissile = "PartHitDetector",
BombDarkMatter = "PartHitDetector",
FireworkMissile = "PartHitDetector",
BombBalloon = "Balloon",
PresentBig = "Box",
PresentSmall = "Box",
}
_toySetupParts = {
BombMissile = "Body",
BombDarkMatter = "Pyramid",
FireworkMissile = "Hitbox",
BombBalloon = "Balloon",
PresentBig = "Box",
PresentSmall = "Box",
}

function tgtToyExplosionLoop(toyType, targetName, keepGoing)
local spawnRF = getSpawnToyRF_T()
local destroyRE = getDestroyToyRE_T()
if not spawnRF then return end
local GE = getGrabEventsT()
local SetNetworkOwner = GE and GE:FindFirstChild("SetNetworkOwner")
local BombEvents = ReplicatedStorage:FindFirstChild("BombEvents")
local BuyToy = ReplicatedStorage:FindFirstChild("MenuToys") and ReplicatedStorage.MenuToys:FindFirstChild("BuyToyRemoteFunction")
if not SetNetworkOwner then return end
local toyFolderName = LocalPlayer.Name .. "SpawnedInToys"
local hitboxName = _toyHitboxNames[toyType] or "PartHitDetector"
local setupPartName = _toySetupParts[toyType] or "Body"
local explosionAmount = (Options.ToyCount and Options.ToyCount.Value) or 3

local function getAllToys()
local toys = workspace:FindFirstChild(toyFolderName)
if not toys then return {} end
local result = {}
for _, toy in pairs(toys:GetChildren()) do
if toy.Name == toyType then table.insert(result, toy) end
end
return result
end

local function spawnOneToy()
local char = LocalPlayer.Character
local hrp = char and char:FindFirstChild("HumanoidRootPart")
if not hrp then return end
pcall(function()
spawnRF:InvokeServer(toyType, CFrame.new(hrp.Position + Vector3.new(0, 5, 0)), Vector3.zero)
if BuyToy then BuyToy:InvokeServer(toyType) end
end)
end

local function setupBomb(bomb)
if not bomb or not bomb.PrimaryPart then return end
local hitPart = bomb:FindFirstChild(setupPartName)
if not hitPart then return end
pcall(function() SetNetworkOwner:FireServer(hitPart, hitPart.CFrame) end)
task.wait(0.05)
pcall(function()
for _, v in pairs(bomb.PrimaryPart:GetChildren()) do
if v:IsA("BodyVelocity") or v.Name == "Stable" then v:Destroy() end
end
local bodyVel = Instance.new("BodyVelocity")
bodyVel.Velocity = Vector3.zero
bodyVel.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bodyVel.Name = "Stable"
bodyVel.Parent = bomb.PrimaryPart
bomb:PivotTo(CFrame.new(math.random(-500, 500), 10000, math.random(-500, 500)))
end)
end

local function explodeBomb(bomb, targetHRP)
if not bomb or not targetHRP then return end
local hitbox = bomb:FindFirstChild(hitboxName)
if not hitbox then return end
local targetPos = targetHRP.Position
if BombEvents and BombEvents:FindFirstChild("BombExplode") then
pcall(function()
BombEvents.BombExplode:FireServer({Hitbox = hitbox, PositionPart = targetHRP}, targetPos)
end)
end
end

local function deleteAllToys()
for _, bomb in pairs(getAllToys()) do
pcall(function() destroyRE:FireServer(bomb) end)
end
end

while keepGoing() do
local target = Players:FindFirstChild(targetName)
if not target or not target.Character then task.wait(0.2) continue end
local tHRP = target.Character:FindFirstChild("HumanoidRootPart")
local tHum = target.Character:FindFirstChildOfClass("Humanoid")
if not (tHRP and tHum) or tHum.Health <= 0 then task.wait(0.2) continue end
deleteAllToys()
task.wait(0.1)
while #getAllToys() < explosionAmount and keepGoing() do
task.spawn(spawnOneToy)
task.wait(0.02)
end
task.wait(0.15)
for _, bomb in pairs(getAllToys()) do
if not keepGoing() then break end
task.spawn(function() setupBomb(bomb) end)
task.wait(0.02)
end
task.wait(0.2)
tHRP = target.Character and target.Character:FindFirstChild("HumanoidRootPart")
if tHRP and tHum and tHum.Health > 0 then
for _, bomb in pairs(getAllToys()) do
if not keepGoing() then break end
task.spawn(function() explodeBomb(bomb, tHRP) end)
task.wait(0.01)
end
end
task.wait(0.15)
deleteAllToys()
task.wait(0.5)
end
deleteAllToys()
end

function tgtRunAction(action, targetName, keepGoing)
if action == "Kick" then
tgtPoopKickLoop(targetName, keepGoing)
elseif action == "Kill" then
tgtV3Action("kill", targetName, keepGoing)
elseif action == "Lock" then
tgtLockLoop(targetName, keepGoing)
elseif action == "BlackHole" then
tgtBlackHoleLoop(targetName, keepGoing)
elseif action == "BombMissile" then
tgtBombMissileLoop(targetName, keepGoing)
elseif action == "Banana" then
tgtBananaLoop(targetName, keepGoing)
elseif action == "Snowball" then
tgtSnowballLoop(targetName, keepGoing)
elseif action == "AntiAntiKickWD" then
tgtAntiAntiKickWDLoop(targetName, keepGoing)
elseif action == "AntiAntiKickClick" then
tgtAntiAntiKickClickLoop(targetName, keepGoing)
elseif action == "AntiAntiKickBlackHole" then
tgtAntiAntiKickBlackHoleLoop(targetName, keepGoing)
elseif action == "PalletRagdoll" then
tgtPalletRagdollLoop(targetName, keepGoing)
elseif action == "DontGiveChance" then
tgtDontGiveChanceLoop(targetName, keepGoing)
elseif action == "RemoveTargetGucci" then
tgtPoopGucciLoop(targetName, keepGoing)
elseif action == "Destroy Food" then
tgtPoopDestroyFoodLoop(targetName, keepGoing)
elseif action == "Remove Invisibility" then
tgtPoopRemoveInvisLoop(targetName, keepGoing)
elseif action == "Delete Seatables" then
tgtPoopDeleteSeatablesLoop(targetName, keepGoing)
elseif action == "FireworkMissile" then
tgtToyExplosionLoop("FireworkMissile", targetName, keepGoing)
elseif action == "BombBalloon" then
tgtToyExplosionLoop("BombBalloon", targetName, keepGoing)
elseif action == "PresentBig" then
tgtToyExplosionLoop("PresentBig", targetName, keepGoing)
elseif action == "PresentSmall" then
tgtToyExplosionLoop("PresentSmall", targetName, keepGoing)
end
end

end  


Workspace = Workspace or game:GetService("Workspace")
replicatedStorage = replicatedStorage or game:GetService("ReplicatedStorage")
runService = runService or game:GetService("RunService")
localPlayer = localPlayer or Players.LocalPlayer

function tgtPoopSyncSelected(targetName, addOnly)
local target = Players:FindFirstChild(targetName)
if not addOnly then
tbl20.SelectedPlayers = {}
end
if target then
tbl20.SelectedPlayers[target] = true
end
return target
end

function tgtPoopKickLoop(targetName, keepGoing)
local target = tgtPoopSyncSelected(targetName, false)
if not target then
Library:Notify(L("Kick: игрок не найден", "Kick: player not found"), 3)
return
end
if keepGoing() == false then
tbl17.NetworkKick(target, false, false)
return
end
while keepGoing() and target.Parent do
tbl20.SelectedPlayers[target] = true
tbl20.TargetMethods = { Kick = true }
tbl19.LoopApplyMethodT = true
task.wait(0.05)
end
tbl20.SelectedPlayers[target] = nil
if next(tbl20.SelectedPlayers) == nil then
tbl19.LoopApplyMethodT = false
pcall(tbl17.clearAllQueues, "LoopApplyMethodT")
end
task.spawn(function()
pcall(cleanupNetworkKickPlayer, target)
end)
end

local function tgtPHQueueLoop(targetName, keepGoing, method, fn)
local target = tgtPoopSyncSelected(targetName, false)
if not target then
Library:Notify(L(method .. ": игрок не найден", method .. ": player not found"), 3)
return
end
while keepGoing() and target.Parent do
tbl17.queue(method, fn, target)
task.wait()
end
pcall(tbl17.clearAllQueues, "LoopApplyMethodT")
end

function tgtPoopGucciLoop(targetName, keepGoing)
tgtPHQueueLoop(targetName, keepGoing, "Remove Gucci", tbl17.RemoveGucci)
end

local tgtPHFoodSet = {}
if type(antiOwnFoods) == "table" then
for _, itemName in pairs(antiOwnFoods) do
tgtPHFoodSet[itemName] = true
end
end
local tgtPHMusicKeywords = {
"drum", "snare", "bongo", "guitar", "banjo", "lyre", "ukulele", "trumpet",
"bugle", "vuvuzela", "melodica", "microphone", "ocarina", "saxophone",
"instrument", "music", "boombox", "speaker", "piano", "flute", "horn",
}
local function tgtPHIsFoodOrMusic(name)
if tgtPHFoodSet[name] then return true end
if string.sub(name, 1, 4) == "Food" then return true end
local low = string.lower(name)
for _, kw in ipairs(tgtPHMusicKeywords) do
if string.find(low, kw, 1, true) then return true end
end
return false
end

function tgtPoopDestroyFoodLoop(targetName, keepGoing)
local target = tgtPoopSyncSelected(targetName, false)
if not target then
Library:Notify(L("Destroy Food: игрок не найден", "Destroy Food: player not found"), 3)
return
end
if keepGoing() == false then
return
end
while keepGoing() and target.Parent do
local myChar = localPlayer.Character
local myGrip = myChar and myChar:FindFirstChild("Left Arm")
myGrip = myGrip and myGrip:FindFirstChild("LeftGripAttachment")
local targetInv = Workspace:FindFirstChild(target.Name .. "SpawnedInToys")
if targetInv and myGrip and myChar then
for _, v in pairs(targetInv:GetDescendants()) do
if not keepGoing() then break end
if v.Name == "HoldPart" and v.Parent then
local item = v.Parent
if tgtPHIsFoodOrMusic(item.Name) then
local rigid = v:FindFirstChild("RigidConstraint")
local holdFn = item:FindFirstChild("HoldItemRemoteFunction") or v:FindFirstChild("HoldItemRemoteFunction")
local dropFn = item:FindFirstChild("DropItemRemoteFunction") or v:FindFirstChild("DropItemRemoteFunction")
if holdFn then
local attached = false
for _ = 1, 20 do
if not keepGoing() or not item.Parent or not v.Parent then break end
task.spawn(function()
pcall(function() holdFn:InvokeServer(item, myChar) end)
end)
task.wait(0.05)
if rigid and rigid.Attachment1 == myGrip then
attached = true
break
end
end
if dropFn then
task.spawn(function()
pcall(function() dropFn:InvokeServer(item) end)
end)
end
v.Name = "NotHoldPart"
end
end
end
end
end
task.wait(0.25)
end
end

function tgtPoopDeleteSeatablesLoop(targetName, keepGoing)
tgtPHQueueLoop(targetName, keepGoing, "Delete Seatables", tbl17.DeleteSeatables)
end

function tgtPoopRemoveInvisLoop(targetName, keepGoing)
local target = tgtPoopSyncSelected(targetName, false)
if not target then
Library:Notify(L("Remove Invisibility: игрок не найден", "Remove Invisibility: player not found"), 3)
return
end
while keepGoing() and target.Parent do
pcall(tbl17.RemoveInvisibility, target)
task.wait()
end
pcall(tbl17.clearAllQueues, "LoopApplyMethodT")
end

local phVoidPlatform = nil
function tgtPoopTPVoidPlatform()
local myChar, myHrp, myHum = tbl17.charp()
if not myHrp or not myHum or myHum.Health <= 0 then
Library:Notify(L("TP: ты мёртв/без персонажа", "TP: you are dead / no character"), 2)
return
end
if not (phVoidPlatform and phVoidPlatform.Parent) then
phVoidPlatform = Instance.new("Part")
phVoidPlatform.Name = "_OblivionVoidPlatform"
phVoidPlatform.Anchored = true
phVoidPlatform.CanCollide = true
phVoidPlatform.Size = Vector3.new(30, 1, 30)
phVoidPlatform.CFrame = CFrame.new(0, 50000, 0)
phVoidPlatform.Parent = workspace
end
tbl17.teleport(myHrp, phVoidPlatform.CFrame + Vector3.new(0, 10, 0))
Library:Notify(L("TP: ты на войд-платформе", "TP: you are on the void platform"), 2)
end
tbl17 = {}
local isHeld
isHeld = game:GetService("Players").LocalPlayer:WaitForChild("IsHeld", 5)

tbl18 = {
        CounterMethod = "Kill",
        TargetedLimbCA = "All",
        AntiLagSens = 3,
        AGType = "Normal",
        GucciType = "CreatureBlobman",
        AntiInputType = "FoodHamburger",
        InputLgDelay = 0.05,
        AILTransparency = 1,
        NKOffset = 15,
        NKDelayMode = "Normal",
}


tbl19 = {
        AntiGrab = false,
        GucciAntiGrab = false,
        AntiBlobman = false,
        AntiInputLag = false,
        AntiExplosion = false,
        AntiBurn = false,
        AntiVoid = false,
        AntiKick = false,
        AntiKick2 = false,
        AntiSticky = false,
        AntiSnowball = false,
        AntiBanana = false,
        AntiPaint = false,
        AntiPoison = false,
        AntiLag = false,
        AutoAntiLag = true,
        AntiInvis = false,
        AutoReset = true,
        LoopTP = false,
        DestroyVoid = true,
        WaterWalk = false,
        DeleteBarriers = false,
        OtherInvCollisions = false,
        CounterAttack = false,
        LoopApplyMethodT = false,
        LoopApplyMethodTServer = false,
        RagdollInNK = true,
        RagdollInRG = false,
        DeleteShurInNK = true,
}


tbl20 = {
        SelectedPlayers = {},
        TargetMethods = {},
        Whitelisted = {},
        PlayerDropdowns = {},
        WhoGrabbed = nil,
        KickFlags = 0,
        LastKickFlagTime = 0,
        States = { Owning = false, Spawning = false, Ragdolling = false, Escaping = false },
        BrokenPlots = {},
        FailedToys = {},
        GrabbableParts = setmetatable({}, { __mode = "k" }),
        LimbOffsetsRL = {
                Head = CFrame.new(0, 1.5, 0),
                ["Left Arm"] = CFrame.new(-1.5, 0, 0),
                ["Right Arm"] = CFrame.new(1.5, 0, 0),
                ["Left Leg"] = CFrame.new(-0.5, -2, 0),
                ["Right Leg"] = CFrame.new(0.5, -2, 0),
                Torso = CFrame.new(),
        },
        NetworkKickQueue = {},
        DestroyThread = nil,
        AKType = "Shuriken",
        AntiKickSettings = {
                Highlight = false,
                HighlightFill = Color3.new(1, 0, 0),
                HighlightOutl = Color3.new(1, 1, 1),
                HighlightFT = 0.5,
                HighlightOT = 0,
                Transparency = 0.9,
        },
}

local tbl21, tbl22, DisblVoid, tbl23, v21

do
        local resonanceExtractAdapters = getgenv and getgenv().ResonanceExtractAdapters or {}
        tbl21 = {}
        tbl22 = {}
        local tbl24 = {}
        local obj = setmetatable({}, { __mode = "k" })
        local obj2 = setmetatable({}, { __mode = "k" })
        local tbl25 = {}
        local tbl26 = {}
        DisblVoid = nil

        local notify = function(arg, arg2)
                task.spawn(function()
                        Library:Notify((arg2 and arg2 .. ": " or "") .. tostring(arg), 3)
                end)
        end

        methodSelected = function(arg, arg2)
                if type(arg) ~= "table" then
                        return arg == arg2
                end

                if arg[arg2] ~= nil then
                        return arg[arg2] == true or arg[arg2] == arg2
                end

                for k, v22 in pairs(arg) do
                        if k == arg2 or v22 == arg2 then
                                return true
                        end
                end

                return false
        end

        tbl23 = {}
        v21 = nil

        locateRemoteRoot = function()
                local isRemoteRoot = function(arg)
                        return arg and arg:FindFirstChild("GrabEvents") and arg:FindFirstChild("MenuToys") and arg:FindFirstChild("CharacterEvents")
                end

                if isRemoteRoot(replicatedStorage) then
                        return replicatedStorage
                end

                for _, descendant in ipairs(replicatedStorage:GetDescendants()) do
                        if isRemoteRoot(descendant) then
                                return descendant
                        end
                end

                return replicatedStorage
        end

        refreshRemotes = function()
                v21 = locateRemoteRoot()
                local grabEvents2 = replicatedStorage:FindFirstChild("GrabEvents")
                local menuToys = v21 and v21:FindFirstChild("MenuToys")
                local characterEvents = v21 and v21:FindFirstChild("CharacterEvents")
                local playerEvents = v21 and v21:FindFirstChild("PlayerEvents")
                tbl23.sno = grabEvents2 and grabEvents2:FindFirstChild("SetNetworkOwner") or nil
                tbl23.cgl = grabEvents2 and grabEvents2:FindFirstChild("CreateGrabLine") or nil
                tbl23.dgl = grabEvents2 and grabEvents2:FindFirstChild("DestroyGrabLine") or nil
                tbl23.Struggle = characterEvents and characterEvents:FindFirstChild("Struggle") or nil
                tbl23.RagdollRemote = characterEvents and characterEvents:FindFirstChild("RagdollRemote") or nil
                tbl23.spe = playerEvents and playerEvents:FindFirstChild("StickyPartEvent") or nil
                tbl23.spawnEvent = menuToys and menuToys:FindFirstChild("SpawnToyRemoteFunction") or nil
                tbl23.destroyToyEvent = menuToys and menuToys:FindFirstChild("DestroyToy") or nil
                return v21
        end

        refreshRemotes()

        fireRemote = function(arg, ...)
                if not arg then
                        return false, nil
                end

                local ok2, result2 = pcall(function(...)
                        if arg:IsA("RemoteFunction") then
                                return arg:InvokeServer(...)
                        end
                        arg:FireServer(...)
                        return true
                end, ...)

                return ok2, result2
        end

        tbl17.charp = function(arg)
                local character = arg or localPlayer
                character = character and character.Character
                if not character then
                        return nil, nil, nil
                end
                return character, character:FindFirstChild("HumanoidRootPart") or character:FindFirstChild("Torso"), (character:FindFirstChildWhichIsA("Humanoid"))
        end

        tbl17.getMagnitude = function(arg, arg2)
                if not arg or not arg2 or not arg:IsA("BasePart") or not arg2:IsA("BasePart") then
                        return math.huge
                end
                return (arg.Position - arg2.Position).Magnitude
        end

        tbl17.isnetworkowner = function(arg)
                if typeof(arg) ~= "Instance" or not arg:IsA("BasePart") then
                        return false
                end
                local assemblyRootPart = arg

                if arg.Massless and arg.AssemblyRootPart then
                        assemblyRootPart = arg.AssemblyRootPart
                end

                if not assemblyRootPart or assemblyRootPart.Anchored then
                        return false
                end

                if type(isnetworkowner) == "function" then
                        local ok2, result2 = pcall(isnetworkowner, assemblyRootPart)
                        if ok2 and type(result2) == "boolean" then
                                return result2
                        end
                end

                local ok2, result2 = pcall(function()
                        return assemblyRootPart.ReceiveAge
                end)

                return ok2 and result2 == 0
        end

        tbl17.ispartowner = function(arg)
                if typeof(arg) ~= "Instance" or not arg:IsA("BasePart") then
                        return false
                end

                if arg.Name == "HumanoidRootPart" then
                        local parent = arg.Parent

                        if parent then
                                local head_ = parent:FindFirstChild("Head")

                                if head_ then
                                        local partOwner = head_:FindFirstChild("PartOwner")
                                        if partOwner then
                                                return partOwner.Value == localPlayer.Name
                                        end
                                end
                        end
                end

                local partOwner = arg:FindFirstChild("PartOwner")
                if partOwner and partOwner.Value == localPlayer.Name then
                        return true
                end
                local parent = arg.Parent
                if not parent then
                        return false
                end
                local partOwner2 = parent:FindFirstChild("PartOwner", true)
                if not partOwner2 then
                        return false
                end
                return partOwner2.Value == localPlayer.Name
        end

        tbl17.WhichPlot = function(arg)
                if typeof(arg) ~= "Instance" or not arg:IsA("Player") then
                        return nil, nil
                end
                local plots = Workspace:FindFirstChild("Plots")
                if not plots then
                        return nil, nil
                end

                for _, child in pairs(plots:GetChildren()) do
                        local plotArea = child:FindFirstChild("PlotArea")

                        if plotArea and plotArea:IsA("BasePart") then
                                local ok2, result2 = pcall(function()
                                        return Workspace:GetPartsInPart(plotArea)
                                end)

                                if ok2 and type(result2) == "table" then
                                        for _, v22 in pairs(result2) do
                                                v22 = v22 and v22.Parent
                                                if (v22 and Players:GetPlayerFromCharacter(v22)) == arg then
                                                        local inOwnedPlot = arg:FindFirstChild("InOwnedPlot")
                                                        return child, inOwnedPlot and inOwnedPlot.Value
                                                end
                                        end
                                end
                        end
                end

                return nil, nil
        end

        removeSpawnSound = function(arg)
                task.spawn(function()
                        local main = arg and arg:WaitForChild("Main", 0.4)
                        local itemSpawnSound = main and main:WaitForChild("ItemSpawnSound", 0.4)

                        if itemSpawnSound then
                                itemSpawnSound:Destroy()
                        end
                end)
        end

        tbl17.BreakPlot = function(arg)
                if resonanceExtractAdapters.BreakPlot then
                        return resonanceExtractAdapters.BreakPlot(arg)
                end

                if typeof(arg) ~= "Instance" then
                        return false
                end
                local v22 = arg
                local inPlot = nil

                if arg:IsA("Player") then
                        inPlot = arg:FindFirstChild("InPlot")
                        if not inPlot then
                                return false
                        end
                        v22 = tbl17.WhichPlot(arg)
                        if not v22 then
                                return false
                        end
                end

                if typeof(v22) ~= "Instance" then
                        return false
                end

                if tbl20.BrokenPlots[v22.Name] then
                        return true
                end
                local plotArea = v22:FindFirstChild("PlotArea")
                if not plotArea then
                        return false
                end
                refreshRemotes()
                if not tbl23.spe then
                        return false
                end
                local NinjaShuriken = tbl17.st("NinjaShuriken")
                if not NinjaShuriken then
                        return false
                end
                removeSpawnSound(NinjaShuriken)
                NinjaShuriken.Parent = nil
                local stickyPart = NinjaShuriken:WaitForChild("StickyPart", 0.4)
                local stickyWeld = stickyPart and stickyPart:WaitForChild("StickyWeld", 0.4)
                if not stickyPart or not stickyWeld then
                        tbl17.dt(NinjaShuriken)
                        return false
                end
                local cframe = CFrame.new(1099511627776, 1099511627776, 1099511627776)

                if v22.Name == "Plot5" then
                        cframe = CFrame.new(2147483648, 2147483648, 2147483648)
                end

                if not tbl17.sno(stickyPart) then
                        tbl17.dt(NinjaShuriken)
                        return false
                end
                fireRemote(tbl23.spe, stickyPart, plotArea, cframe)
                local now = tick()

                while true do
                        if stickyWeld.Parent and tick() - now <= 0.8 then
                                if not (stickyWeld.Part1 == plotArea and (not inPlot or inPlot.Value ~= true)) then
                                        task.wait()
                                        continue
                                end
                        end

                        break
                end

                if not stickyWeld.Parent or stickyWeld.Part1 ~= plotArea or inPlot and inPlot.Value == true then
                        tbl17.dt(NinjaShuriken)
                        return false
                end
                tbl20.BrokenPlots[v22.Name] = NinjaShuriken

                pcall(function()
                        NinjaShuriken.Destroying:Once(function()
                                if tbl20.BrokenPlots[v22.Name] == NinjaShuriken then
                                        tbl20.BrokenPlots[v22.Name] = nil
                                end
                        end)
                end)

                return NinjaShuriken
        end

        performOwnershipGrabLoop = function(arg, arg2, arg3, arg4, arg5, arg6)
                local cframe = CFrame.new()
                local cframe2 = CFrame.new()
                local cframe3 = CFrame.new()
                local connection3 = nil

                local setupHiddenMove = function()
                        cframe = arg3.CFrame
                        cframe2 = arg4.CFrame
                        cframe3 = arg3.CFrame

                        pcall(function()
                                runService:UnbindFromRenderStep("RSCClient")
                        end)

                        runService:BindToRenderStep("RSCClient", Enum.RenderPriority.Camera.Value - 1, function()
                                if arg3.Parent then
                                        arg3.CFrame = cframe
                                end

                                if arg4.Parent then
                                        arg4.CFrame = cframe2
                                end
                        end)

                        connection3 = runService.Heartbeat:Connect(function()
                                if not arg3.Parent or not arg4.Parent then
                                        return
                                end
                                cframe = arg3.CFrame
                                cframe2 = arg4.CFrame
                                arg3.CFrame = cframe3
                                arg4.CFrame = cframe3
                        end)
                end

                local cleanupHiddenMove = function()
                        pcall(function()
                                runService:UnbindFromRenderStep("RSCClient")
                        end)

                        if connection3 then
                                connection3:Disconnect()
                                connection3 = nil
                        end

                        if arg4.Parent then
                                arg4.CFrame = cframe2
                        end

                        if arg3.Parent then
                                arg3.CFrame = cframe
                        end
                end

                local n8 = tonumber(arg2) or 0.65
                local now = tick()

                local ok2, result2 = pcall(function()
                        if arg6 then
                                setupHiddenMove()
                        end

                        while arg.Parent and not tbl17.ispartowner(arg) and arg5.Health ~= 0 and arg3.Parent and tick() - now < n8 do
                                if arg6 then
                                        cframe3 = arg.CFrame - Vector3.new(0, 22, 0)
                                end

                                runService.Heartbeat:Wait()
                                tbl17.grab(arg, arg6 and cframe3 or nil)
                        end
                end)

                if arg6 then
                        pcall(cleanupHiddenMove)
                end

                return ok2, result2
        end

        tbl17.sno = function(arg, arg2)
                if resonanceExtractAdapters.sno then
                        return resonanceExtractAdapters.sno(arg, arg2)
                end

                if typeof(arg) ~= "Instance" or not arg:IsA("BasePart") or not arg.Parent then
                        return false
                end
                local v22, v23, v24 = tbl17.charp()
                local head_ = v22 and v22:FindFirstChild("Head")
                local ragdolled = v24 and v24:FindFirstChild("Ragdolled")
                if not v22 or not head_ or not v23 or not v24 or not ragdolled or v24.Health <= 0 then
                        return false
                end
                local spawnLocation = Workspace:FindFirstChild("SpawnLocation")
                if not spawnLocation or tbl17.getMagnitude(arg, spawnLocation) > 10000000 then
                        return false
                end

                if tbl17.ispartowner(arg) then
                        return true
                end
                local flag2 = (v23.Position - arg.Position).Magnitude >= 31
                if flag2 and tbl20.States.Owning == true then
                        return false
                end
                local flag3 = arg.Name == "HumanoidRootPart" or arg.Name == "Torso"
                local v25 = nil

                if flag3 then
                        local parent = arg.Parent and Players:FindFirstChild(arg.Parent.Name)
                        local isPlayer = parent and parent:IsA("Player")
                        v25 = nil

                        if isPlayer then
                                local v26, v27, v28 = tbl17.charp(parent)
                                if not v28 or v28.Health == 0 then
                                        return false
                                end
                                local inPlot = parent:FindFirstChild("InPlot")
                                inPlot = inPlot and inPlot.Value == true
                                v25 = nil

                                if inPlot then
                                        v25 = tbl17.BreakPlot(parent)
                                        if not v25 then
                                                return false
                                        end
                                end
                        end
                end

                tbl20.States.Owning = true
                local v26, v27 = performOwnershipGrabLoop(arg, arg2, v23, head_, v24, flag2)
                tbl20.States.Owning = false

                if typeof(v25) == "Instance" then
                        task.delay(1, tbl17.dt, v25)
                end

                if not v26 then
                        warn("[Resonance sno] " .. tostring(v27))
                        return false
                end
                return tbl17.ispartowner(arg)
        end

        tbl17.unsno = function(arg)
                if resonanceExtractAdapters.unsno then
                        return resonanceExtractAdapters.unsno(arg)
                end

                if not arg then
                        return false
                end
                refreshRemotes()
                if tbl23.dgl then
                        return (fireRemote(tbl23.dgl, arg))
                end
                return false
        end

        tbl17.teleport = function(arg, arg2, arg3)
                if not arg then
                        return false
                end

                if typeof(arg) == "Instance" and arg:IsA("Model") then
                        arg = arg.PrimaryPart or arg:FindFirstChildWhichIsA("BasePart", true)
                end

                if typeof(arg) ~= "Instance" or not arg:IsA("BasePart") then
                        return false
                end
                local cframe

                if typeof(arg2) == "CFrame" then
                        cframe = arg2
                elseif typeof(arg2) == "Vector3" then
                        cframe = CFrame.new(arg2)
                else
                        cframe = nil

                        if typeof(arg2) == "Instance" then
                                if arg2:IsA("BasePart") then
                                        cframe = arg2.CFrame
                                else
                                        cframe = nil

                                        if arg2:IsA("Model") then
                                                cframe = arg2:GetPivot()
                                        end
                                end
                        end
                end

                if not cframe then
                        return false
                end

                if typeof(arg3) == "CFrame" then
                        cframe *= arg3
                end

                arg.CFrame = cframe
                tbl17.resetVelocity(arg)
                return true
        end

        tbl17.resetVelocity = function(arg)
                if arg and arg:IsA("BasePart") then
                        arg.AssemblyLinearVelocity = Vector3.zero
                        arg.AssemblyAngularVelocity = Vector3.zero
                end
        end

        tbl17.FormatPlayer = function(arg)
                if typeof(arg) ~= "Instance" or not arg:IsA("Player") then
                        return tostring(arg)
                end

                if arg == localPlayer then
                        return "You"
                end

                if arg.DisplayName == arg.Name then
                        return arg.Name
                end
                return string.format("%s (@%s)", arg.DisplayName, arg.Name)
        end

        tbl17.findNicePart = function(arg)
                if typeof(arg) ~= "Instance" or not arg:IsA("Model") then
                        return nil
                end
                local v22 = tbl20.GrabbableParts[arg]

                if v22 then
                        if v22.Parent and v22:IsA("BasePart") and v22.CanQuery and v22.Name ~= "Button" then
                                return v22
                        end
                        tbl20.GrabbableParts[arg] = nil
                end

                for _, child in pairs(arg:GetChildren()) do
                        if child:IsA("BasePart") and child.CanQuery and child.Name ~= "Button" then
                                tbl20.GrabbableParts[arg] = child
                                return child
                        end
                end

                return nil
        end

        tbl17.DestroyObject = function(arg)
                if resonanceExtractAdapters.DestroyObject then
                        return resonanceExtractAdapters.DestroyObject(arg)
                end

                if typeof(arg) ~= "Instance" or not arg:IsA("Model") then
                        return false
                end
                local v22 = tbl17.findNicePart(arg)
                if not v22 or not tbl17.sno(v22) then
                        return false
                end

                if DisblVoid and type(DisblVoid.SetDisabled) == "function" then
                        pcall(function()
                                DisblVoid:SetDisabled(true)
                        end)
                end

                Workspace.FallenPartsDestroyHeight = (0/0)
                arg:PivotTo(CFrame.new(0, -11111, 0))

                if tbl20.DestroyThread then
                        pcall(task.cancel, tbl20.DestroyThread)
                end

                tbl20.DestroyThread = task.delay(0.25, function()
                        Workspace.FallenPartsDestroyHeight = -10000
                        task.wait()
                        Workspace.FallenPartsDestroyHeight = tbl19.DestroyVoid and (0/0) or -100

                        if DisblVoid and type(DisblVoid.SetDisabled) == "function" then
                                pcall(function()
                                        DisblVoid:SetDisabled(false)
                                end)
                        end
                end)

                return true
        end

        tbl17.getInventory = function(arg)
                if resonanceExtractAdapters.getInventory then
                        return resonanceExtractAdapters.getInventory(arg)
                end

                if typeof(arg) ~= "Instance" or not arg:IsA("Player") then
                        return nil, nil
                end
                local v22 = Workspace:FindFirstChild(arg.Name .. "SpawnedInToys")
                local plots = Workspace:FindFirstChild("Plots")
                local plotItems = Workspace:FindFirstChild("PlotItems")
                local v23 = plots and plotItems
                local v24 = nil

                if v23 then
                        v24 = nil

                        for _, child in pairs(plots:GetChildren()) do
                                local plotSign = child:FindFirstChild("PlotSign")
                                plotSign = plotSign and plotSign:FindFirstChild("ThisPlotsOwners")

                                if plotSign then
                                        for _, child2 in pairs(plotSign:GetChildren()) do
                                                if child2:IsA("StringValue") and child2.Value == arg.Name then
                                                        v24 = plotItems:FindFirstChild(child.Name)
                                                end
                                        end
                                end
                        end
                end

                return v22, v24
        end

        tbl17.kill = function(arg)
                local v22, v23, v24 = tbl17.charp(arg)

                if v24 and v24.Health ~= 0 and v23 then
                        if Workspace:FindFirstChild("SpawnLocation") then
                                tbl17.getMagnitude(Workspace.SpawnLocation, v23)
                        end

                        if tbl17.sno(v23) then
                                v24.BreakJointsOnDeath = false
                                v24.Sit = false
                                v24:ChangeState(Enum.HumanoidStateType.Dead)
                                tbl17.teleport(v23, CFrame.new(0, 9e9, 0))
                                task.delay(2, tbl17.unsno, v23)
                        end
                end
        end

        tbl17.void = function(arg)
                local v22, v23, v24 = tbl17.charp(arg)

                if v24 and v24.Health ~= 0 and v23 then
                        local spawnLocation = Workspace:FindFirstChild("SpawnLocation")

                        if (not spawnLocation or tbl17.getMagnitude(spawnLocation, v23) < 15000) and tbl17.sno(v23) then
                                tbl17.teleport(v23, CFrame.new(0, 9e9, 0))
                                v23.Velocity = Vector3.new(0, 9e9, 0)
                                task.delay(0.35, tbl17.unsno, v23)
                        end
                end
        end

        tbl17.grab = function(arg, arg2)
                if resonanceExtractAdapters.grab then
                        return resonanceExtractAdapters.grab(arg, arg2)
                end
                local v22, v23, v24 = tbl17.charp()
                if not v23 or not v24 or v24.Health == 0 then
                        return
                end

                if not arg or not arg.Parent or not arg:IsA("BasePart") then
                        return
                end
                refreshRemotes()
                if not tbl23.sno then
                        return
                end
                arg2 = arg2 or v23.CFrame
                if typeof(arg2) ~= "CFrame" then
                        return
                end
                local position = arg.Position
                local cframe = CFrame.new((arg2 * CFrame.new(0, 1.593, 0)).Position, position)
                tbl23.sno:FireServer(arg, cframe)
        end

        limbMatchesMode = function(arg, arg2)
                if not arg:IsA("BasePart") then
                        return false
                end
                local flag2 = arg.Name:find("Arm") ~= nil
                local flag3 = arg.Name:find("Leg") ~= nil
                if arg2 == "Arms" then
                        return flag2
                end

                if arg2 == "Legs" then
                        return flag3
                end

                if arg2 == "All" then
                        return flag2 or flag3
                end
                return false
        end

        tbl17.breaklimb = function(arg, arg2)
                if resonanceExtractAdapters.breaklimb then
                        return resonanceExtractAdapters.breaklimb(arg, arg2)
                end
                local flag2 = arg == localPlayer
                if flag2 and tbl19.GucciAntiGrab then
                        notify("Can't break limbs while in Gucci!")
                        return false
                end
                local v22, v23, v24 = tbl17.charp(arg)
                local ragdolled = v24 and v24:FindFirstChild("Ragdolled")
                if not v23 or not v24 or v24.Health == 0 or not ragdolled then
                        return false
                end
                local children = v22:GetChildren()
                local flag3 = false

                for _, child in ipairs(children) do
                        if limbMatchesMode(child, arg2) then
                                flag3 = true
                                break
                        end
                end

                if not flag3 then
                        return false
                end

                if flag2 then
                        refreshRemotes()
                        if not tbl23.RagdollRemote then
                                return false
                        end
                        fireRemote(tbl23.RagdollRemote, v23, 0.35)
                        local now = tick()

                        while true do
                                task.wait()

                                if ragdolled.Value then
                                        break
                                elseif tick() - now > 0.5 then
                                        break
                                end
                        end
                else
                        if not tbl17.sno(v23) then
                                return false
                        end

                        if not tbl17.ragdoll(v22) then
                                return false
                        end
                end

                if not ragdolled.Value then
                        return false
                end

                for _, child in ipairs(children) do
                        if limbMatchesMode(child, arg2) then
                                child.CFrame = CFrame.new(0, -100000, 0)
                        end
                end

                task.wait(0.15)

                if flag2 then
                        tbl17.grab(v23)
                        task.wait()
                        refreshRemotes()

                        if tbl23.RagdollRemote then
                                fireRemote(tbl23.RagdollRemote, v23, 0)
                        end
                else
                        Workspace.FallenPartsDestroyHeight = -1000
                        tbl17.teleport(v23, CFrame.new(0, -1100, 0))
                        task.wait()
                        Workspace.FallenPartsDestroyHeight = tbl19.DestroyVoid and (0/0) or -100
                        tbl17.unsno(v23)
                end

                return true
        end

        recoveredYield = function(arg)
                local n8 = arg or 0
                local now = tick()

                repeat
                        runService.Heartbeat:Wait()
                until n8 <= tick() - now

                return true
        end

        tbl17.ExtraEscape = function()
                local v22, v23, v24 = tbl17.charp()
                if not v23 or not v24 or v24.Health == 0 or tbl20.States.Escaping then
                        return
                end
                local inPlot = localPlayer:FindFirstChild("InPlot")
                if not inPlot or inPlot.Value then
                        return
                end
                tbl20.States.Escaping = true
                local waypoints = Workspace:FindFirstChild("Waypoints")
                waypoints = waypoints and waypoints:FindFirstChild("TudorHouse")
                if not waypoints then
                        tbl20.States.Escaping = false
                        return
                end
                local TractorGreen = tbl17.st("TractorGreen", waypoints.CFrame)
                if not TractorGreen then
                        tbl20.States.Escaping = false
                        return
                end
                local vehicleSeat = TractorGreen:WaitForChild("VehicleSeat", 1)
                if not vehicleSeat then
                        tbl20.States.Escaping = false
                        return tbl17.dt(TractorGreen)
                end
                local now = tick()
                local flag2 = false
                v24.Sit = false
                vehicleSeat:Sit(v24)

                local connection3 = vehicleSeat.ChildAdded:Connect(function(child)
                        if child.Name == "SeatWeld" then
                                child.Enabled = false
                                flag2 = true
                                child:Destroy()
                        end
                end)

                while true do
                        recoveredYield()

                        if flag2 then
                                break
                        elseif tick() - now > 0.7 then
                                break
                        end
                end

                connection3:Disconnect()
                v24.Sit = false
                tbl20.States.Escaping = false
                return tbl17.dt(TractorGreen)
        end

        collectInventoryModels = function(arg)
                local v22, v23 = tbl17.getInventory(arg)
                local tbl27 = {}

                if v22 then
                        for _, child in ipairs(v22:GetChildren()) do
                                table.insert(tbl27, child)
                        end
                end

                if v23 then
                        for _, child in ipairs(v23:GetChildren()) do
                                table.insert(tbl27, child)
                        end
                end

                return tbl27
        end

        breakVehicleSeat = function(arg, arg2, arg3, arg4)
                if tbl19.GucciAntiGrab then
                        return false
                end

                if not arg or not arg.Parent then
                        return false
                end

                if not (arg:IsA("VehicleSeat") or arg:IsA("Seat")) then
                        return false
                end

                if arg.Occupant or arg:GetAttribute("broken") then
                        return false
                end

                if not arg2 or arg2.Health == 0 or not arg3 then
                        return false
                end
                arg2.Sit = false

                if tbl19.RagdollInRG and arg4 then
                        tbl17.sno(arg4, 1.5)
                end

                arg.CanCollide = false
                local flag2 = false

                local connection3 = arg.ChildAdded:Connect(function(child)
                        if child.Name == "SeatWeld" then
                                pcall(function()
                                        child.Enabled = false
                                end)

                                child:Destroy()
                                flag2 = true
                        end
                end)

                local now = tick()

                while arg.Parent and arg2.Parent and not flag2 and tick() - now <= 0.5 do
                        runService.Heartbeat:Wait()
                        arg:Sit(arg2)
                end

                arg2.Sit = false
                local parent = arg.Parent

                task.delay(0.5, function()
                        if arg and parent then
                                pcall(function()
                                        arg.Parent = replicatedStorage
                                        task.wait()
                                        arg.Parent = parent
                                end)
                        end
                end)

                if flag2 then
                        arg:SetAttribute("broken", true)

                        if connection3 then
                                connection3:Disconnect()
                        end

                        local connection4 = nil

                        connection4 = arg.ChildAdded:Connect(function(child)
                                if child.Name ~= "SeatWeld" then
                                        return
                                end
                                local v22
                                v22, v22 = tbl17.charp()
                                local part1 = nil

                                pcall(function()
                                        part1 = child.Part1
                                end)

                                if part1 ~= v22 then
                                        arg:SetAttribute("broken", false)

                                        if connection4 then
                                                connection4:Disconnect()
                                        end
                                end
                        end)

                        return true
                end

                arg2.Sit = true
                task.wait(0.05)
                arg2.Sit = false

                if connection3 then
                        connection3:Disconnect()
                end

                return false
        end

        applySeatBreakerToInventory = function(arg, arg2, arg3, arg4)
                if not arg then
                        return
                end

                for _, child in ipairs(arg:GetChildren()) do
                        if child:IsA("Model") then
                                local vehicleSeat = child:FindFirstChildWhichIsA("VehicleSeat") or child:FindFirstChildWhichIsA("Seat")

                                if vehicleSeat then
                                        breakVehicleSeat(vehicleSeat, arg2, arg3, arg4)
                                end
                        end
                end
        end

        tbl17.RemoveGucci = function(arg)
                if resonanceExtractAdapters.RemoveGucci then
                        return resonanceExtractAdapters.RemoveGucci(arg)
                end
                local v22, v23, v24 = tbl17.charp()
                local v25, v26, v27 = tbl17.charp(arg)
                local v28, v29 = tbl17.getInventory(arg)
                if not v24 or v24.Health <= 0 or not v23 or not v28 or not v27 then
                        return false
                end
                local seatPart = v27.SeatPart

                if not seatPart then
                        applySeatBreakerToInventory(v28, v24, v23, v26)

                        if v29 then
                                applySeatBreakerToInventory(v29, v24, v23, v26)
                        end

                        return nil
                end

                if v27.Sit then
                        applySeatBreakerToInventory(v28, v24, v23, v26)

                        if v29 then
                                applySeatBreakerToInventory(v29, v24, v23, v26)
                        end

                        return nil
                end

                seatPart:SetAttribute("broken", false)
                return breakVehicleSeat(seatPart, v24, v23, v26)
        end

        tbl17.DeleteSeatables = function(arg)
                if resonanceExtractAdapters.DeleteSeatables then
                        return resonanceExtractAdapters.DeleteSeatables(arg)
                end
                local v22, v23, v24 = tbl17.charp()
                local v25, v26 = tbl17.getInventory(arg)
                if not v24 or v24.Health <= 0 or not v23 or not v25 then
                        return false
                end

                local processFolder = function(arg2)
                        if not arg2 then
                                return
                        end

                        for _, child in ipairs(arg2:GetChildren()) do
                                if child:IsA("Model") then
                                        local primaryPart = child.PrimaryPart or tbl17.findNicePart(child)

                                        if primaryPart and primaryPart:IsA("BasePart") and primaryPart.Velocity.Magnitude <= 10 then
                                                local vehicleSeat = child:FindFirstChild("VehicleSeat")

                                                if vehicleSeat and not vehicleSeat.Occupant then
                                                        tbl17.DestroyObject(child)
                                                end
                                        end
                                end
                        end
                end

                processFolder(v25)

                if v26 then
                        processFolder(v26)
                end

                return nil
        end

        tbl17.EatAntiInput = function(arg)
                if resonanceExtractAdapters.EatAntiInput then
                        return resonanceExtractAdapters.EatAntiInput(arg)
                end
                local character = arg and arg.Character
                character = character and character:FindFirstChild("Left Arm")
                local v22, v23, v24 = tbl17.charp()
                local leftArm = v22 and v22:FindFirstChild("Left Arm")
                leftArm = leftArm and leftArm:FindFirstChild("LeftGripAttachment")
                local plotItems = Workspace:FindFirstChild("PlotItems")
                if not character or not v22 or not leftArm or not v24 or v24.Health == 0 or not plotItems then
                        return false
                end
                local flag2 = false

                for _, v25 in pairs(character:GetConnectedParts()) do
                        if v25.Name == "HoldPart" and not v25:IsDescendantOf(plotItems) then
                                local parent = v25.Parent
                                local holdItemRemoteFunction = v25:WaitForChild("HoldItemRemoteFunction", 0.45)
                                local dropItemRemoteFunction = v25:WaitForChild("DropItemRemoteFunction", 0.45)
                                local rigidConstraint = v25:WaitForChild("RigidConstraint", 0.45)

                                if holdItemRemoteFunction and dropItemRemoteFunction and rigidConstraint then
                                        rigidConstraint.Enabled = false
                                        local now = tick()

                                        while v22.Parent and parent.Parent and leftArm.Parent and rigidConstraint.Parent and rigidConstraint.Attachment1 ~= leftArm and v24.Health ~= 0 and tick() - now < 1 do
                                                task.spawn(function()
                                                        local ok2, result2 = pcall(function()
                                                                holdItemRemoteFunction:InvokeServer(parent, v22)
                                                        end)

                                                        if not ok2 then
                                                                warn("[Resonance Destroy Food/hold] " .. tostring(result2))
                                                        end
                                                end)

                                                task.wait()
                                        end

                                        if rigidConstraint.Parent and rigidConstraint.Attachment1 == leftArm then
                                                task.spawn(function()
                                                        local ok2, result2 = pcall(function()
                                                                dropItemRemoteFunction:InvokeServer(parent)
                                                        end)

                                                        if not ok2 then
                                                                warn("[Resonance Destroy Food/drop] " .. tostring(result2))
                                                        end
                                                end)

                                                pcall(function()
                                                        dropItemRemoteFunction:Destroy()
                                                end)

                                                flag2 = true
                                        end
                                end
                        end
                end

                return flag2
        end

        tbl17.RemoveInvisibility = function(arg)
                if resonanceExtractAdapters.RemoveInvisibility then
                        return resonanceExtractAdapters.RemoveInvisibility(arg)
                end
                local v22, v23, v24 = tbl17.charp(arg)
                if not v23 or not v24 or v24.Health == 0 then
                        return false
                end
                v23.Massless = false
                return true
        end

        destroyNamedMover = function(arg, arg2)
                arg = arg and arg:FindFirstChild(arg2)

                if arg then
                        arg:Destroy()
                end
        end

        releaseKickOwnership = function(arg, arg2)
                if arg2 == "Faster but worse" then
                        tbl17.unsno(arg)
                        return
                end

                if arg2 == "Normal" then
                        task.delay(0.2, tbl17.unsno, arg)
                        return
                end

                if arg2 == "Slower but better" then
                        task.spawn(function()
                                task.wait()
                                tbl17.unsno(arg)
                        end)
                end
        end

        tbl17.ToggleCollisions = function(arg, arg2, arg3, arg4)
                if resonanceExtractAdapters.ToggleCollisions then
                        return resonanceExtractAdapters.ToggleCollisions(arg, arg2, arg3, arg4)
                end

                if typeof(arg) ~= "Instance" or not arg:IsA("Model") then
                        return
                end

                for _, child in pairs(arg:GetChildren()) do
                        if child:IsA("BasePart") then
                                if arg2 then
                                        if child.Name == "Torso" or child.Name == "Head" then
                                                child.CanCollide = true
                                        end
                                else
                                        child.CanCollide = false

                                        if not arg3 then
                                                local ragdollLimbPart = child:FindFirstChild("RagdollLimbPart")

                                                if ragdollLimbPart then
                                                        ragdollLimbPart.CanCollide = false
                                                end
                                        end
                                end

                                if child.Name ~= "HumanoidRootPart" then
                                        local physicalProperties = nil

                                        if arg4 then
                                                physicalProperties = PhysicalProperties.new(0.01, 0.01, 0.01, 0.01, 0.01)
                                        end

                                        child.CustomPhysicalProperties = physicalProperties
                                end
                        end
                end
        end

        tbl17.ToggleMotor6D = function(arg, enabled, arg2)
                if resonanceExtractAdapters.ToggleMotor6D then
                        return resonanceExtractAdapters.ToggleMotor6D(arg, enabled, arg2)
                end

                if typeof(arg) ~= "Instance" or not arg:IsA("Model") then
                        return false
                end
                local humanoid_ = arg:FindFirstChild("Humanoid")
                local torso_ = arg:FindFirstChild("Torso")
                if not humanoid_ or not torso_ then
                        return false
                end
                local ragdolled = humanoid_:FindFirstChild("Ragdolled")
                if not ragdolled then
                        return false
                end

                if ragdolled.Value ~= true then
                        for _, child in pairs(torso_:GetChildren()) do
                                if child:IsA("Motor6D") then
                                        child.Enabled = enabled
                                end
                        end

                        return true
                end

                if not arg2 then
                        return false
                end

                for _, v22 in pairs({ "Left Leg", "Right Leg", "Left Arm", "Right Arm", "Head" }) do
                        local v23 = arg:FindFirstChild(v22)

                        if v23 then
                                local ballSocketConstraint = v23:FindFirstChild("BallSocketConstraint")

                                if ballSocketConstraint then
                                        ballSocketConstraint.Enabled = enabled
                                end
                        end
                end

                return true
        end

        tbl17.RepositionLimbs = function(arg, arg2)
                if resonanceExtractAdapters.RepositionLimbs then
                        return resonanceExtractAdapters.RepositionLimbs(arg, arg2)
                end

                if typeof(arg) ~= "Instance" or not arg:IsA("Model") then
                        return false
                end
                local humanoidRootPart = arg:FindFirstChild("HumanoidRootPart")
                if not humanoidRootPart then
                        return false
                end
                local weldHRP = humanoidRootPart:FindFirstChild("WeldHRP")
                local torso_ = arg:FindFirstChild("Torso")
                if not torso_ then
                        return false
                end
                local neck = torso_:FindFirstChild("Neck")
                if neck and neck.Enabled == true then
                        return false
                end

                if weldHRP then
                end

                for k, v22 in pairs(tbl20.LimbOffsetsRL) do
                        local v23 = arg:FindFirstChild(k)

                        if v23 and v23:IsA("BasePart") then
                                v23.CFrame = humanoidRootPart.CFrame * v22

                                if arg2 then
                                        tbl17.resetVelocity(v23)
                                end
                        end
                end

                return true
        end

        tbl17.removesticky = function(arg)
                if resonanceExtractAdapters.removesticky then
                        return resonanceExtractAdapters.removesticky(arg)
                end

                if typeof(arg) ~= "Instance" or not arg:IsA("Model") or not arg.Parent then
                        return false
                end
                local humanoidRootPart = arg:FindFirstChild("HumanoidRootPart") or arg:FindFirstChild("Torso")
                if not humanoidRootPart or not humanoidRootPart:IsA("BasePart") then
                        return false
                end
                local plotItems = Workspace:FindFirstChild("PlotItems")
                local obj3 = setmetatable({}, { __mode = "k" })

                local registerModel = function(arg2)
                        if typeof(arg2) ~= "Instance" then
                                return
                        end
                        local parent

                        if arg2:IsA("BasePart") then
                                parent = arg2.Parent
                        else
                                parent = arg2
                        end

                        if parent and parent:IsA("Model") and parent ~= arg and parent.Parent then
                                obj3[parent] = true
                        end
                end

                for _, descendant in pairs(arg:GetDescendants()) do
                        if descendant:IsA("BasePart") then
                                for _, v22 in pairs(descendant:GetConnectedParts()) do
                                        if v22.Name == "StickyPart" then
                                                for _, child in pairs(v22:GetChildren()) do
                                                        if child:IsA("TouchInterest") then
                                                                child:Destroy()
                                                        end
                                                end

                                                registerModel(v22)
                                        end
                                end
                        end
                end

                local playerFromCharacter = Players:GetPlayerFromCharacter(arg)

                if playerFromCharacter then
                        local v22, v23 = tbl17.getInventory(playerFromCharacter)

                        if v22 then
                                for _, child in pairs(v22:GetChildren()) do
                                        registerModel(child)
                                end
                        end

                        if v23 then
                                for _, child in pairs(v23:GetChildren()) do
                                        registerModel(child)
                                end
                        end
                end

                local flag2 = false

                for k in pairs(obj3) do
                        if k.Parent then
                                local ok2, result2 = pcall(k.GetPivot, k)

                                if ok2 and (result2.Position - humanoidRootPart.Position).Magnitude <= 10 then
                                        local v22 = tbl17.findNicePart(k)

                                        if v22 and v22:IsA("BasePart") then
                                                local rscBV = v22:FindFirstChild("RscBV")

                                                if rscBV and not rscBV:IsA("BodyVelocity") then
                                                        rscBV:Destroy()
                                                        rscBV = nil
                                                end

                                                if not rscBV then
                                                        local bodyVelocity = Instance.new("BodyVelocity")
                                                        bodyVelocity.Velocity = Vector3.new(0, -50000, 0)
                                                        bodyVelocity.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
                                                        bodyVelocity.Name = "RscBV"
                                                        bodyVelocity.Parent = v22
                                                end

                                                tbl17.ToggleCollisions(k, false)

                                                if not (plotItems and k:IsDescendantOf(plotItems)) then
                                                        tbl17.grab(v22)
                                                end

                                                local stickyPart = k:FindFirstChild("StickyPart")

                                                if stickyPart then
                                                        for _, child in pairs(stickyPart:GetChildren()) do
                                                                child:Destroy()
                                                        end
                                                end

                                                k:PivotTo(CFrame.new(0, -50000, 0))
                                                task.delay(1, tbl17.unsno, v22)
                                                flag2 = true
                                        end
                                end
                        end
                end

                return flag2
        end

        tbl17.ragdoll = function(arg, arg2)
                if resonanceExtractAdapters.ragdoll then
                        return resonanceExtractAdapters.ragdoll(arg, arg2)
                end

                if typeof(arg) ~= "Instance" then
                        return
                end
                local torso_, humanoid_

                if arg:IsA("Player") then
                        local character = arg.Character
                        torso_ = nil
                        humanoid_ = nil

                        if character then
                                torso_ = character:FindFirstChild("Torso")
                                humanoid_ = character:FindFirstChild("Humanoid")
                        end
                elseif arg:IsA("Model") then
                        local head_ = arg:FindFirstChild("Head")
                        torso_ = arg:FindFirstChild("Torso")
                        humanoid_ = arg:FindFirstChild("Humanoid")
                        if not head_ then
                                return
                        end
                else
                        if not arg:IsA("BasePart") then
                                return
                        end
                        local isModel = arg.Parent and arg.Parent:IsA("Model")
                        humanoid_ = nil

                        if isModel then
                                humanoid_ = arg.Parent:FindFirstChild("Humanoid")
                                torso_ = arg
                        else
                                torso_ = arg
                        end
                end

                if not torso_ or not torso_:IsA("BasePart") or not torso_.Parent or not humanoid_ then
                        return
                end
                local ragdolled = humanoid_:FindFirstChild("Ragdolled")
                if not ragdolled then
                        return
                end

                if ragdolled.Value ~= false then
                        return
                end

                if tbl20.States.Ragdolling then
                        return
                end
                torso_.CanCollide = false
                tbl20.States.Ragdolling = true
                local v22 = Workspace:FindFirstChild(localPlayer.Name .. "SpawnedInToys")
                if not v22 then
                        tbl20.States.Ragdolling = false
                        return
                end
                local ragdollRSC = v22:FindFirstChild("RagdollRSC") or tbl17.st("PalletLightBrown")
                if not ragdollRSC then
                        tbl20.States.Ragdolling = false
                        return
                end
                local soundPart = ragdollRSC:WaitForChild("SoundPart", 0.5)

                if not soundPart or not soundPart:IsA("BasePart") then
                        tbl20.States.Ragdolling = false
                        tbl17.dt(ragdollRSC)
                        return
                end

                local main = ragdollRSC:WaitForChild("Main", 0.5)

                if not main or not main:IsA("BasePart") then
                        tbl20.States.Ragdolling = false
                        tbl17.dt(ragdollRSC)
                        return
                end

                if not tbl17.sno(soundPart) then
                        tbl20.States.Ragdolling = false
                        tbl17.dt(ragdollRSC)
                        return
                end

                ragdollRSC.Name = "RagdollRSC"
                pcall(function()
                        for _, v in pairs(ragdollRSC:GetDescendants()) do
                                if v:IsA("BasePart") then
                                        v.Transparency = 1
                                        v.CanCollide = false
                                        v.CanQuery = false
                                end
                        end
                end)
                local ragdollBP = main:FindFirstChild("RagdollBP")

                if ragdollBP and not ragdollBP:IsA("BodyPosition") then
                        ragdollBP:Destroy()
                        ragdollBP = nil
                end

                if not ragdollBP then
                        local bodyPosition = Instance.new("BodyPosition")
                        bodyPosition.Name = "RagdollBP"
                        bodyPosition.D = 1500
                        bodyPosition.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
                        bodyPosition.Position = Vector3.new(0, 100000, 0)
                        bodyPosition.Parent = main
                else
                        ragdollBP.D = 1500
                        ragdollBP.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
                        ragdollBP.Position = Vector3.new(0, 100000, 0)
                end

                local now = tick()

                while ragdollRSC.Parent and soundPart.Parent and torso_.Parent and ragdolled.Parent and ragdolled.Value ~= true and tick() - now <= 0.5 do
                        soundPart.Position = torso_.Position
                        soundPart.Velocity = Vector3.new(0, -10, 0)
                        task.wait(0.1)
                end

                tbl20.States.Ragdolling = false
                return ragdolled.Value
        end

        tbl17.st = function(arg, arg2)
                if resonanceExtractAdapters.st then
                        return resonanceExtractAdapters.st(arg, arg2)
                end

                if type(arg) ~= "string" or tbl20.FailedToys[arg] then
                        return nil
                end
                local v22, v23 = tbl17.charp()
                if not v23 or tbl20.States.Spawning then
                        return nil
                end
                local inPlot = localPlayer:FindFirstChild("InPlot")
                local inOwnedPlot = localPlayer:FindFirstChild("InOwnedPlot")
                local flag2 = inPlot ~= nil and inPlot.Value == true

                if flag2 then
                        flag2 = not (inOwnedPlot ~= nil and inOwnedPlot.Value == true)
                end

                if flag2 and tbl20.States.Owning then
                        return nil
                end
                refreshRemotes()
                if not tbl23.spawnEvent or not tbl23.spawnEvent:IsA("RemoteFunction") then
                        return nil
                end
                local v24 = Workspace:FindFirstChild(localPlayer.Name .. "SpawnedInToys")
                if not v24 then
                        return nil
                end
                tbl20.States.Spawning = true

                if flag2 then
                        tbl20.States.Owning = true
                end

                local connection3 = nil
                local connection4 = nil
                local flag3 = false
                local cFrame = v23.CFrame
                local v25 = nil

                local cleanup = function()
                        if flag3 then
                                pcall(function()
                                        runService:UnbindFromRenderStep("STClient")
                                end)

                                flag3 = false
                        end

                        if connection4 then
                                connection4:Disconnect()
                                connection4 = nil
                        end

                        if connection3 then
                                connection3:Disconnect()
                                connection3 = nil
                        end

                        if flag2 and v23 and v23.Parent then
                                v23.CFrame = cFrame
                        end

                        tbl20.States.Spawning = false

                        if flag2 then
                                tbl20.States.Owning = false
                        end
                end

                local ok2, result2 = pcall(function()
                        local canSpawnToy = localPlayer:FindFirstChild("CanSpawnToy")

                        if canSpawnToy then
                                while canSpawnToy.Value == false do
                                        task.wait()
                                        if not v23.Parent then
                                                return
                                        end
                                end
                        end

                        local cframe = flag2 and CFrame.new(0, 25, 0) or v23.CFrame + Vector3.new(0, 15, 0)
                        local n8 = CFrame.new((typeof(arg2) == "CFrame" and arg2 or cframe).Position) + Vector3.new(0, 0, 20)
                        local rotateOrientPart = Workspace:FindFirstChild("RotateOrientPart")
                        local rotation = rotateOrientPart and rotateOrientPart.Rotation or Vector3.zero

                        connection3 = v24.ChildAdded:Connect(function(child)
                                if child.Name == arg then
                                        v25 = child

                                        if connection3 then
                                                connection3:Disconnect()
                                                connection3 = nil
                                        end
                                end
                        end)

                        cFrame = v23.CFrame

                        if flag2 then
                                local cframe2 = CFrame.new(0, 10, 0)

                                pcall(function()
                                        runService:UnbindFromRenderStep("STClient")
                                end)

                                runService:BindToRenderStep("STClient", Enum.RenderPriority.Camera.Value - 1, function()
                                        if v23.Parent then
                                                v23.CFrame = cFrame
                                        end
                                end)

                                flag3 = true

                                connection4 = runService.Heartbeat:Connect(function()
                                        if v23.Parent then
                                                cFrame = v23.CFrame
                                                v23.CFrame = cframe2
                                        end
                                end)
                        end

                        local invokeSpawn = function()
                                fireRemote(tbl23.spawnEvent, arg, n8, rotation)
                        end

                        task.spawn(invokeSpawn)
                        local now = tick()

                        while true do
                                runService.Heartbeat:Wait()

                                if flag2 then
                                        task.spawn(invokeSpawn)
                                end

                                local v26 = v25
                                local flag4

                                if v25 then
                                        flag4 = v26
                                else
                                        flag4 = tick() - now > 1.3
                                end

                                if not flag4 then
                                        continue
                                end
                                break
                        end
                end)

                cleanup()
                if not ok2 then
                        warn(("[Resonance st] %s"):format(tostring(result2)))
                        return nil
                end

                if not v25 then
                        tbl20.FailedToys[arg] = true

                        task.delay(2, function()
                                tbl20.FailedToys[arg] = nil
                        end)
                end

                return v25
        end

        tbl17.dt = function(arg)
                if resonanceExtractAdapters.dt then
                        return resonanceExtractAdapters.dt(arg)
                end

                if typeof(arg) ~= "Instance" or not arg:IsA("Model") then
                        return false
                end
                refreshRemotes()
                if not tbl23.destroyToyEvent then
                        return false
                end
                return fireRemote(tbl23.destroyToyEvent, arg) == true
        end

        removeTargetAntiKickShuriken = function(arg)
                if not arg then
                        return
                end

                local destroyStickyOwner = function(arg2)
                        if not arg2 or not arg2:IsA("BasePart") then
                                return
                        end

                        for _, v22 in ipairs(arg2:GetConnectedParts()) do
                                if v22.Name == "StickyPart" then
                                        local parent = v22.Parent

                                        if parent and (parent.Name == "NinjaShuriken" or parent.Name == "ShurikenRSC") then
                                                tbl17.dt(parent)
                                        end
                                end
                        end
                end

                for _, descendant in ipairs(arg:GetDescendants()) do
                        if descendant:IsA("BasePart") then
                                destroyStickyOwner(descendant)
                        end
                end
        end

        cleanupNetworkKickPlayer = function(arg)
                pcall(tgtKickHoldStop, arg)
                pcall(tgtPalletRagdollStopFor, arg)
                local v22, v23, v24 = tbl17.charp(arg)
                if not v22 or not v23 or not v24 then
                        return false
                end
                local kickAP = v23:FindFirstChild("KickAP")
                local kickAO = v23:FindFirstChild("KickAO")
                local weldHRP = v23:FindFirstChild("WeldHRP")

                if kickAP then
                        kickAP:Destroy()
                end

                if kickAO then
                        kickAO:Destroy()
                end

                local ragdolled = v24:FindFirstChild("Ragdolled")

                if ragdolled and ragdolled.Value == true and weldHRP then
                        weldHRP.Enabled = true
                end

                task.spawn(tbl17.ToggleMotor6D, v22, true, true)
                task.spawn(tbl17.ToggleCollisions, v22, true)
                return true
        end

        createNetworkKickMovers = function(parent, position)
                local bodyPosition = Instance.new("BodyPosition")
                bodyPosition.Name = "KickAP"
                bodyPosition.D = 500
                bodyPosition.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
                bodyPosition.Position = position
                bodyPosition.Parent = parent
                local bodyGyro = Instance.new("BodyGyro")
                bodyGyro.Name = "KickAO"
                bodyGyro.D = 500
                bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
                bodyGyro.CFrame = CFrame.new()
                bodyGyro.Parent = parent
                return bodyPosition, bodyGyro
        end

        prepareNetworkKickMovers = function(arg, position)
                local kickAP = arg:FindFirstChild("KickAP")
                local kickAO = arg:FindFirstChild("KickAO")
                local weldHRP = arg:FindFirstChild("WeldHRP")

                if not kickAP or not kickAO then
                        if kickAP then
                                kickAP:Destroy()
                        end

                        if kickAO then
                                kickAO:Destroy()
                        end

                        kickAP, kickAO = createNetworkKickMovers(arg, position)
                end

                if weldHRP then
                        weldHRP.Enabled = false
                end

                kickAP.Position = position
                return kickAP, kickAO, weldHRP
        end

        getRecoveredKickPosition = function(arg, arg2, arg3)
                local n8 = (arg2 - 1) * 6.2831853071795862 / arg3
                local nkOffset = tbl18.NKOffset
                return arg.Position + Vector3.new(math.cos(n8) * 10, nkOffset, math.sin(n8) * 10)
        end

        tgtKickHoldState = {}

        tgtKickHoldStop = function(player)
        local st = tgtKickHoldState[player]
        if not st then
        return false
        end
        tgtKickHoldState[player] = nil
        st.active = false
        return true
        end

        tgtKickHoldEnsure = function(player)
        if not player or not player.Parent or tgtKickHoldState[player] then
        return
        end
        local st = { active = true }
        tgtKickHoldState[player] = st
        task.spawn(function()
        while st.active and player and player.Parent do
        local ok2 = pcall(function()
        local char2 = player.Character
        local hrp2 = char2 and char2:FindFirstChild("HumanoidRootPart")
        local myChar2 = localPlayer.Character
        local myRoot2 = myChar2 and myChar2:FindFirstChild("HumanoidRootPart")
        local myHum2 = myChar2 and myChar2:FindFirstChildOfClass("Humanoid")
        local tHum2 = char2 and char2:FindFirstChildOfClass("Humanoid")
        if not hrp2 or not myRoot2 or not myHum2 or myHum2.Health <= 0 then
        return
        end
        if not tHum2 or tHum2.Health <= 0 or not char2:FindFirstChild("Torso") then
        return
        end
        local inPlot2 = localPlayer:FindFirstChild("InPlot")
        if inPlot2 and inPlot2.Value == true then
        return
        end
        local kickPos2 = getRecoveredKickPosition(myRoot2, 1, 1)
        prepareNetworkKickMovers(hrp2, kickPos2)
        if tbl17.getMagnitude(myRoot2, hrp2) > 33 then
        if not tbl17.sno(hrp2) then
        return
        end
        tbl17.teleport(hrp2, kickPos2)
        end
        tbl17.grab(hrp2)
        releaseKickOwnership(hrp2, tbl18.NKDelayMode)
        if tbl19.RagdollInNK then
        task.spawn(tbl17.ragdoll, char2)
        end
        tbl17.ToggleCollisions(char2, false)
        tbl17.RepositionLimbs(char2, true)
        end)
        if not ok2 then
        break
        end
        runService.Heartbeat:Wait()
        end
        end)
        end

        tgtPalletRagdollState = { active = false, player = nil, pallet = nil, soundPart = nil, worker = nil, steppedConn = nil }

        tgtPalletRagdollStopFor = function(player)
        local st = tgtPalletRagdollState
        if not st.active and not st.pallet then
        return
        end
        st.active = false
        st.player = nil
        st.worker = nil
        if st.steppedConn then
        pcall(function() st.steppedConn:Disconnect() end)
        st.steppedConn = nil
        end
        local pallet = st.pallet
        st.pallet = nil
        st.soundPart = nil
        if pallet and pallet.Parent then
        pcall(tbl17.dt, pallet)
        end
        local toysFolder = Workspace:FindFirstChild(localPlayer.Name .. "SpawnedInToys")
        if toysFolder then
        local ghost = toysFolder:FindFirstChild("PalletForRagdoll")
        if ghost and ghost.Parent then
        pcall(tbl17.dt, ghost)
        end
        end
        end

        tgtPalletRagdollSpawn = function()
        local st = tgtPalletRagdollState
        if not st.active or (st.pallet and st.pallet.Parent) then
        return
        end
        local toy = tbl17.st("PalletLightBrown")
        if toy and toy.Parent then
        if st.active and not (st.pallet and st.pallet.Parent) then
        st.pallet = toy
        else
        pcall(tbl17.dt, toy)
        end
        end
        end

        tgtPalletRagdollEnsure = function(player)
        local st = tgtPalletRagdollState
        if not player or not player.Parent then
        return
        end
        st.player = player
        if st.active then
        return
        end
        st.active = true
        st.worker = task.spawn(function()
        refreshRemotes()
        if not tbl23.sno or not tbl23.destroyToyEvent then
        tgtPalletRagdollStopFor(nil)
        return
        end
        while st.active do
        if not st.player or not st.player.Parent then
        tgtPalletRagdollStopFor(nil)
        break
        end
        local myChar = localPlayer.Character
        local myHum = myChar and myChar:FindFirstChildOfClass("Humanoid")
        if not myHum or myHum.Health <= 0 then
        task.wait(0.5)
        continue
        end
        if not (st.pallet and st.pallet.Parent) then
        tgtPalletRagdollSpawn()
        if not (st.pallet and st.pallet.Parent) then
        task.wait(0.3)
        end
        continue
        end
        local pallet = st.pallet
        local soundPart = pallet:FindFirstChild("SoundPart")
        if not soundPart then
        soundPart = pallet:WaitForChild("SoundPart", 0.5)
        end
        if not soundPart or not soundPart.Parent then
        pcall(tbl17.dt, pallet)
        st.pallet = nil
        st.soundPart = nil
        continue
        end
        pcall(function()
        for _, v in pairs(pallet:GetDescendants()) do
        if v:IsA("BasePart") then
        v.CanCollide = false
        v.CanQuery = false
        v.Transparency = 1
        end
        end
        end)
        local claimed = false
        pcall(function()
        tbl23.sno:FireServer(soundPart, soundPart.CFrame)
        if tbl23.dgl then
        fireRemote(tbl23.dgl, soundPart)
        end
        local partOwner = soundPart:WaitForChild("PartOwner", 1)
        claimed = partOwner ~= nil and partOwner.Value == localPlayer.Name
        end)
        if not claimed then
        pcall(tbl17.dt, pallet)
        st.pallet = nil
        st.soundPart = nil
        task.wait(0.05)
        continue
        end
        pcall(function()
        pallet.Name = "PalletForRagdoll"
        end)
        st.soundPart = soundPart
        local strikePhase = false
        if st.steppedConn then
        pcall(function() st.steppedConn:Disconnect() end)
        st.steppedConn = nil
        end
        local steppedConn = nil
        steppedConn = runService.Stepped:Connect(function()
        if not st.active or not pallet.Parent or not soundPart.Parent then
        if steppedConn then
        steppedConn:Disconnect()
        if st.steppedConn == steppedConn then
        st.steppedConn = nil
        end
        end
        return
        end
        local tChar = st.player and st.player.Character
        local tRoot = tChar and tChar:FindFirstChild("HumanoidRootPart")
        local tHum = tChar and tChar:FindFirstChildOfClass("Humanoid")
        if tRoot and tHum and soundPart.Parent and tHum.Health > 0 then
        local ragVal = tHum:FindFirstChild("Ragdolled")
        local isRag = ragVal and ragVal.Value or false
        if not isRag then
        strikePhase = not strikePhase
        if strikePhase then
        soundPart.CFrame = tRoot.CFrame * CFrame.new(0, 2, 0)
        soundPart.AssemblyLinearVelocity = Vector3.new(0, -9e5, 0)
        else
        soundPart.CFrame = tRoot.CFrame * CFrame.new(0, -1, 0)
        soundPart.AssemblyLinearVelocity = Vector3.new(0, 9e5, 0)
        end
        else
        soundPart.CFrame = CFrame.new(0, 9e9, 0)
        soundPart.AssemblyLinearVelocity = Vector3.zero
        end
        else
        soundPart.CFrame = CFrame.new(0, 9e9, 0)
        soundPart.AssemblyLinearVelocity = Vector3.zero
        end
        end)
        st.steppedConn = steppedConn
        while st.active and pallet.Parent and soundPart.Parent and st.player and st.player.Parent do
        task.wait(0.1)
        end
        if st.steppedConn == steppedConn then
        pcall(function() steppedConn:Disconnect() end)
        st.steppedConn = nil
        end
        if st.active and st.pallet == pallet then
        st.pallet = nil
        st.soundPart = nil
        if pallet.Parent then
        pcall(tbl17.dt, pallet)
        end
        task.wait(0.05)
        end
        end
        end)
        end

        performNetworkKickPass = function(arg, arg2, arg3, arg4)
                if resonanceExtractAdapters.NetworkKick then
                        return resonanceExtractAdapters.NetworkKick(arg, arg4 == true, false, tbl18, tbl19, tbl20)
                end

                if typeof(arg) ~= "Instance" or not arg:IsA("Player") or arg == localPlayer or not arg.Parent then
                        return false
                end
                local v22, v23, v24 = tbl17.charp()
                if not v22 or not v23 or not v24 or v24.Health <= 0 then
                        return false
                end
                local inPlot = localPlayer:FindFirstChild("InPlot")
                if inPlot and inPlot.Value == true then
                        return false
                end
                local v25, v26, v27 = tbl17.charp(arg)
                if not v25 or not v26 or not v27 or v27.Health <= 0 then
                        return false
                end

                if not v25:FindFirstChild("Torso") then
                        return false
                end
                local v28 = getRecoveredKickPosition(v23, arg2, arg3)
                prepareNetworkKickMovers(v26, v28)

                if tbl17.getMagnitude(v23, v26) > 33 then
                        if not tbl17.sno(v26) then
                                return false
                        end
                        tbl17.teleport(v26, v28)
                end

                tbl17.grab(v26)
                releaseKickOwnership(v26, tbl18.NKDelayMode)

                if tbl19.RagdollInNK then
                        task.spawn(tbl17.ragdoll, v25)
                end

                if tbl19.DeleteShurInNK then
                        task.spawn(tbl17.removesticky, v25)
                end

                tbl17.ToggleCollisions(v25, false)
                tbl17.RepositionLimbs(v25, true)

                if not arg4 then
                        task.delay(0.5, function()
                                pcall(cleanupNetworkKickPlayer, arg)
                                obj[arg] = nil
                        end)
                end

                return true
        end

        tbl17.NetworkKick = function(arg, arg2, arg3)
                if typeof(arg) ~= "Instance" or not arg:IsA("Player") or arg == localPlayer then
                        return false
                end
                tbl20.NetworkKickQueue[arg] = { loopCall = arg2 == true, isServer = arg3 == true, queuedAt = tick() }
                return true
        end

        runNetworkKickWorker = function()
                while task.wait() do
                        local tbl27 = {}

                        for k, v22 in pairs(tbl20.NetworkKickQueue) do
                                tbl20.NetworkKickQueue[k] = nil

                                if k.Parent and not tbl20.Whitelisted[k] and not obj[k] then
                                        table.insert(tbl27, { player = k, request = v22 })
                                end
                        end

                        local n8 = #tbl27

                        for i, v22 in ipairs(tbl27) do
                                local player = v22.player

                                if player.Parent and not obj[player] then
                                        obj[player] = true
                                        local ok2, result2 = pcall(performNetworkKickPass, player, i, n8, false)
                                        local flag2 = not ok2

                                        if flag2 or result2 ~= true then
                                                obj[player] = nil
                                        end

                                        if flag2 then
                                                warn(("[Resonance Target/Kick] %s: %s"):format(player.Name, tostring(result2)))
                                        end
                                end
                        end
                end
        end

        tbl17.queue = function(arg, arg2, arg3)
                if arg == nil or type(arg2) ~= "function" or not arg3 then
                        return false
                end
                local tbl27 = tbl25[arg]

                if not tbl27 then
                        tbl27 = {}
                        tbl25[arg] = tbl27
                end

                if not table.find(tbl27, arg3) then
                        table.insert(tbl27, arg3)
                end

                if tbl26[arg] then
                        return true
                end
                tbl26[arg] = true

                task.spawn(function()
                        while #tbl27 > 0 do
                                local v22 = table.remove(tbl27, 1)

                                if v22 and v22.Parent then
                                        local ok2, result2 = pcall(arg2, v22)

                                        if not ok2 then
                                                local v23 = tostring
                                                warn(("[Resonance Target/queue:%s] %s"):format(tostring(arg), v23(result2)))
                                        end

                                        task.wait()
                                end
                        end

                        tbl26[arg] = nil
                end)

                return true
        end

        tbl17.clearAllQueues = function(arg)
                if arg then
                        tbl22[arg] = (tbl22[arg] or 0) + 1
                else
                        for k in pairs(tbl22) do
                                tbl22[k] = tbl22[k] + 1
                        end
                end

                for _, v22 in pairs(tbl25) do
                        table.clear(v22)
                end

                table.clear(tbl20.NetworkKickQueue)
        end

        tbl17.GetTargets = function(arg, arg2)
                local tbl27 = {}

                if arg then
                        for _, player in ipairs(Players:GetPlayers()) do
                                if player ~= localPlayer and not tbl20.Whitelisted[player] then
                                        table.insert(tbl27, player)
                                end
                        end

                        return tbl27
                end

                local v22 = pairs
                local tbl28 = arg2 or {}

                for k, v23 in v22(tbl28) do
                        local flag2 = v23 ~= false

                        if not (typeof(k) == "Instance" and k:IsA("Player")) then
                                if type(k) == "string" then
                                        k = Players:FindFirstChild(k)
                                elseif typeof(v23) == "Instance" and v23:IsA("Player") then
                                        flag2 = true
                                        k = v23
                                else
                                        k = nil

                                        if type(v23) == "string" then
                                                k = Players:FindFirstChild(v23)
                                                flag2 = true
                                        end
                                end
                        end

                        if k and flag2 and k ~= localPlayer and not tbl20.Whitelisted[k] then
                                table.insert(tbl27, k)
                        end
                end

                return tbl27
        end

        tbl17.ProcessPlayer = function(arg, arg2, arg3, arg4)
                if arg4 then
                        return
                end

                if not arg or arg == localPlayer or tbl20.Whitelisted[arg] then
                        return
                end
                local targetMethods = tbl20.TargetMethods

                if methodSelected(targetMethods, "Kick") then
                        if not arg2 or arg3 then
                                tbl17.NetworkKick(arg, arg2, arg3)
                        end
                end

                if methodSelected(targetMethods, "Kill") then
                        if arg2 then
                                tbl17.queue("Kill", tbl17.kill, arg)
                        else
                                tbl17.kill(arg)
                        end
                end

                if methodSelected(targetMethods, "Void") then
                        if arg2 then
                                tbl17.queue("Void", tbl17.void, arg)
                        else
                                tbl17.void(arg)
                        end
                end

                if methodSelected(targetMethods, "Remove Gucci") then
                        if arg2 then
                                tbl17.queue("Remove Gucci", tbl17.RemoveGucci, arg)
                        else
                                tbl17.RemoveGucci(arg)
                        end
                end

                if methodSelected(targetMethods, "Destroy Food") then
                        if arg2 then
                                tbl17.queue("Destroy Food", tbl17.EatAntiInput, arg)
                        else
                                tbl17.EatAntiInput(arg)
                        end
                end

                if methodSelected(targetMethods, "Remove Invisibility") then
                        tbl17.RemoveInvisibility(arg)
                end

                if methodSelected(targetMethods, "Delete Seatables") then
                        if arg2 then
                                tbl17.queue("Delete Seatables", tbl17.DeleteSeatables, arg)
                        else
                                tbl17.DeleteSeatables(arg)
                        end
                end

                if methodSelected(targetMethods, "Bring") then
                        local v22, v23, v24 = tbl17.charp(arg)
                        local v25
                        v25, v25 = tbl17.charp()

                        if v23 and v25 and v24 and v24.Health ~= 0 and tbl17.sno(v23) then
                                tbl17.teleport(v23, v25, CFrame.new(0, 0, -5))
                                task.delay(0.2, tbl17.unsno, v23)
                        end
                end
        end

        runSelectedNetworkKickLoop = function()
                while task.wait() do
                        local flag2 = tbl19.LoopApplyMethodT == true
                        local v22 = methodSelected(tbl20.TargetMethods, "Kick")
                        local inPlot = localPlayer:FindFirstChild("InPlot")
                        inPlot = inPlot and inPlot.Value == true

                        for k in pairs(obj2) do
                                if not k or not k.Parent or not flag2 or not v22 or inPlot or tbl20.SelectedPlayers[k] ~= true then
                                        pcall(cleanupNetworkKickPlayer, k)
                                        obj2[k] = nil
                                end
                        end

                        if flag2 and v22 and not inPlot then
                                local v23, v24, v25 = tbl17.charp()

                                if v24 and v25 and v25.Health > 0 then
                                        local tbl27 = {}

                                        for k, selectedPlayer in pairs(tbl20.SelectedPlayers) do
                                                if selectedPlayer == true and typeof(k) == "Instance" and k:IsA("Player") and k.Parent then
                                                        table.insert(tbl27, k)
                                                        obj2[k] = true
                                                end
                                        end

                                        local n8 = #tbl27

                                        if n8 > 0 then
                                                for i, v26 in ipairs(tbl27) do
                                                        if tbl19.LoopApplyMethodT == true then
                                                                local inPlot2 = localPlayer:FindFirstChild("InPlot")

                                                                if not (inPlot2 and inPlot2.Value == true) then
                                                                        local v27, v28, v29 = tbl17.charp()
                                                                        if not (not v28 or not v29 or v29.Health <= 0) then
                                                                                pcall(performNetworkKickPass, v26, i, n8, true)
                                                                                continue
                                                                        end
                                                                end
                                                        end

                                                        break
                                                end

                                                if tbl18.NKDelayMode == "Slower but better" then
                                                        task.wait()
                                                end
                                        end
                                end
                        end
                end
        end

        safeProcessPlayer = function(arg, arg2, arg3, arg4)
                local ok2, result2 = pcall(tbl17.ProcessPlayer, arg, arg2, arg3, arg4)

                if not ok2 then
                        local name = typeof(arg) == "Instance" and arg.Name or tostring(arg)
                        local now = tick()

                        if now - (tbl24[name] or 0) >= 1 then
                                tbl24[name] = now
                                warn(("[Resonance Target] %s: %s"):format(name, tostring(result2)))
                        end

                        return false, result2
                end

                return true, result2
        end
end

validateTargetRuntime = function()
        refreshRemotes()
        local tbl24 = {}

        if not Workspace:FindFirstChild("SpawnLocation") then
                table.insert(tbl24, "workspace.SpawnLocation")
        end

        if not Workspace:FindFirstChild("Plots") then
                table.insert(tbl24, "workspace.Plots")
        end

        if not Workspace:FindFirstChild("PlotItems") then
                table.insert(tbl24, "workspace.PlotItems")
        end

        if not tbl23.sno then
                table.insert(tbl24, "GrabEvents.SetNetworkOwner")
        end

        if not tbl23.dgl then
                table.insert(tbl24, "GrabEvents.DestroyGrabLine")
        end

        if #tbl24 > 0 then
                warn("[Resonance Target/self-check] missing: " .. table.concat(tbl24, ", "))
                return false, tbl24
        end
        return true, tbl24
end

tbl17.ValidateTargetRuntime = validateTargetRuntime
task.spawn(runNetworkKickWorker)
task.spawn(runSelectedNetworkKickLoop)
task.defer(validateTargetRuntime)

syncSelectedPlayers = function(arg)
        table.clear(tbl20.SelectedPlayers)

        if typeof(arg) == "Instance" and arg:IsA("Player") then
                tbl20.SelectedPlayers[arg] = true
        elseif type(arg) == "table" then
                for k, v22 in pairs(arg) do
                        if typeof(k) == "Instance" and k:IsA("Player") then
                                tbl20.SelectedPlayers[k] = v22 ~= false
                        elseif typeof(v22) == "Instance" and v22:IsA("Player") then
                                tbl20.SelectedPlayers[v22] = true
                        end
                end
        end
end


BloodmanTargets = Tabs.Bloodman:AddLeftGroupbox(L("Игроки на сервере", "Players on Server"), "users")
BloodmanAction = Tabs.Bloodman:AddRightGroupbox(L("Действие", "Action"), "hand")

GE_Bloodman = ReplicatedStorage:FindFirstChild("GrabEvents")
or ReplicatedStorage:WaitForChild("GrabEvents", 10)

bmLoopPlayer = false
bmLoopServer = false

function bmPlayerList()
local list = {}
for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer then
table.insert(list, plr.DisplayName .. " (@" .. plr.Name .. ")")
end
end
return list
end

function bmGetPlayerByName(value)
if not value or value == "" then return nil end
local username = value:match("@([^)]+)")
if username then
return Players:FindFirstChild(username)
end
return Players:FindFirstChild(value)
end

BloodmanAvatarImage = BloodmanTargets:AddImage("BloodmanAvatarDisplay", {
Image = "rbxassetid://0",
Size = Vector2.new(150, 150),
})

BloodmanAvatarLabel = BloodmanTargets:AddLabel(L("<b><font color='#aaaaaa'>Цель не выбрана</font></b>", "<b><font color='#aaaaaa'>No target selected</font></b>"))

BloodmanStatName        = BloodmanTargets:AddLabel("<b><font color='#aaaaaa'>Name:</font></b> <font color='#888888'>—</font>")
BloodmanStatUsername    = BloodmanTargets:AddLabel("<b><font color='#aaaaaa'>Username:</font></b> <font color='#888888'>—</font>")
BloodmanStatHealth      = BloodmanTargets:AddLabel("<b><font color='#aaaaaa'>Health:</font></b> <font color='#888888'>—</font>")
BloodmanStatStuds        = BloodmanTargets:AddLabel("<b><font color='#aaaaaa'>Studs:</font></b> <font color='#888888'>—</font>")
BloodmanStatCord        = BloodmanTargets:AddLabel("<b><font color='#aaaaaa'>Cord:</font></b> <font color='#888888'>—</font>")
BloodmanStatInPlot      = BloodmanTargets:AddLabel("<b><font color='#aaaaaa'>In plot:</font></b> <font color='#888888'>—</font>")
BloodmanStatRagdoll     = BloodmanTargets:AddLabel("<b><font color='#aaaaaa'>Ragdoll:</font></b> <font color='#888888'>—</font>")
BloodmanStatKick        = BloodmanTargets:AddLabel("<b><font color='#aaaaaa'>Kick:</font></b> <font color='#888888'>—</font>")
BloodmanStatLeft        = BloodmanTargets:AddLabel("<b><font color='#aaaaaa'>Left:</font></b> <font color='#888888'>—</font>")
BloodmanStatRejoin      = BloodmanTargets:AddLabel("<b><font color='#aaaaaa'>Rejoin:</font></b> <font color='#888888'>—</font>")
BloodmanStatLastGrab    = BloodmanTargets:AddLabel("<b><font color='#aaaaaa'>Last grab:</font></b> <font color='#888888'>None</font>")
BloodmanStatPCLDBreak   = BloodmanTargets:AddLabel("<b><font color='#aaaaaa'>PCLD Break:</font></b> <font color='#888888'>—</font>")

_bloodmanAvatarCurrentUid = nil

function bmUpdateStats()
if not Options.BloodmanTarget then return end
local value = Options.BloodmanTarget.Value
if not value or value == "" then return end
local plr = bmGetPlayerByName(value)
if not plr or not plr.Character then return end

local char = plr.Character
local hrp = char:FindFirstChild("HumanoidRootPart")
local hum = char:FindFirstChildOfClass("Humanoid")
local lpHrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")

pcall(function() BloodmanStatName:SetText("<b><font color='#aaaaaa'>Name:</font></b> <font color='#ffffff'>" .. tostring(plr.DisplayName) .. "</font>") end)
pcall(function() BloodmanStatUsername:SetText("<b><font color='#aaaaaa'>Username:</font></b> <font color='#ffffff'>" .. tostring(plr.Name) .. "</font>") end)

if hum then
local hp = math.floor(hum.Health)
local hpMax = math.floor(hum.MaxHealth)
local pct = hpMax > 0 and math.floor((hp / hpMax) * 100) or 0
local color = "#55ff55"
if pct <= 0 then color = "#ff5555"
elseif pct <= 50 then color = "#ffff55" end
pcall(function() BloodmanStatHealth:SetText("<b><font color='#aaaaaa'>Health:</font></b> <font color='" .. color .. "'>" .. pct .. "%</font>") end)
end

if hrp and lpHrp then
local studs = math.floor((lpHrp.Position - hrp.Position).Magnitude)
pcall(function() BloodmanStatStuds:SetText("<b><font color='#aaaaaa'>Studs:</font></b> <font color='#ffffff'>" .. studs .. "</font>") end)
end

if hrp then
local pos = hrp.Position
pcall(function() BloodmanStatCord:SetText("<b><font color='#aaaaaa'>Cord:</font></b> <font color='#ffffff'>" .. math.floor(pos.X) .. ", " .. math.floor(pos.Y) .. ", " .. math.floor(pos.Z) .. "</font>") end)
end

local inPlot = false
pcall(function() inPlot = plr:FindFirstChild("InPlot") and plr.InPlot.Value or false end)
local plotColor = inPlot and "#55ff55" or "#ff5555"
pcall(function() BloodmanStatInPlot:SetText("<b><font color='#aaaaaa'>In plot:</font></b> <font color='" .. plotColor .. "'>" .. (inPlot and "Yes" or "No") .. "</font>") end)

local isRagdolled = false
if hum then
local ragVal = hum:FindFirstChild("Ragdolled")
isRagdolled = ragVal and ragVal.Value or false
end
local ragColor = isRagdolled and "#ff5555" or "#55ff55"
pcall(function() BloodmanStatRagdoll:SetText("<b><font color='#aaaaaa'>Ragdoll:</font></b> <font color='" .. ragColor .. "'>" .. (isRagdolled and "Yes" or "No") .. "</font>") end)

local kickText = "—"
local kickColor = "#888888"
pcall(function()
local kickAttr = char:GetAttribute("KickedBy")
if kickAttr and kickAttr ~= "" then
kickText = tostring(kickAttr)
kickColor = "#ff5555"
end
end)
pcall(function() BloodmanStatKick:SetText("<b><font color='#aaaaaa'>Kick:</font></b> <font color='" .. kickColor .. "'>" .. kickText .. "</font>") end)

local leftText = "—"
pcall(function()
local leaveTime = plr:GetAttribute("TimeUntilKick")
if leaveTime and leaveTime > 0 then
local h = math.floor(leaveTime / 3600)
local m = math.floor((leaveTime % 3600) / 60)
local s = math.floor(leaveTime % 60)
leftText = string.format("%d (%02d:%02d:%02d)", leaveTime, h, m, s)
end
end)
pcall(function() BloodmanStatLeft:SetText("<b><font color='#aaaaaa'>Left:</font></b> <font color='#ffffff'>" .. leftText .. "</font>") end)

local rejoinText = "—"
pcall(function()
local rejoinTime = plr:GetAttribute("TimeUntilRejoin")
if rejoinTime and rejoinTime > 0 then
local h = math.floor(rejoinTime / 3600)
local m = math.floor((rejoinTime % 3600) / 60)
local s = math.floor(rejoinTime % 60)
rejoinText = string.format("%d (%02d:%02d:%02d)", rejoinTime, h, m, s)
end
end)
pcall(function() BloodmanStatRejoin:SetText("<b><font color='#aaaaaa'>Rejoin:</font></b> <font color='#ffffff'>" .. rejoinText .. "</font>") end)

local lastGrabText = "None"
local lastGrabColor = "#ff9955"
pcall(function()
local grabber = char:GetAttribute("LastGrabber")
if grabber and grabber ~= "" then
lastGrabText = tostring(grabber)
lastGrabColor = "#ffaa55"
end
end)
pcall(function() BloodmanStatLastGrab:SetText("<b><font color='#aaaaaa'>Last grab:</font></b> <font color='" .. lastGrabColor .. "'>" .. lastGrabText .. "</font>") end)

local pcldBreak = false
pcall(function() pcldBreak = char:GetAttribute("PCLDBreak") == true end)
local pcldColor = pcldBreak and "#ff5555" or "#55ff55"
pcall(function() BloodmanStatPCLDBreak:SetText("<b><font color='#aaaaaa'>PCLD Break:</font></b> <font color='" .. pcldColor .. "'>" .. (pcldBreak and "Yes" or "No") .. "</font>") end)

end

task.spawn(function()
while true do
if Options.BloodmanTarget and Options.BloodmanTarget.Value and Options.BloodmanTarget.Value ~= "" then
pcall(bmUpdateStats)
end
task.wait(0.5)
end
end)

function bmUpdateAvatarDisplay()
if not Options.BloodmanTarget then return end
local value = Options.BloodmanTarget.Value
if not value or value == "" then
_bloodmanAvatarCurrentUid = nil
pcall(function() BloodmanAvatarImage:SetImage("rbxassetid://0") end)
pcall(function() BloodmanAvatarLabel:SetText("<b><font color='#aaaaaa'>Цель не выбрана</font></b>") end)
pcall(function() BloodmanStatName:SetText("<b><font color='#aaaaaa'>Name:</font></b> <font color='#888888'>—</font>") end)
pcall(function() BloodmanStatUsername:SetText("<b><font color='#aaaaaa'>Username:</font></b> <font color='#888888'>—</font>") end)
pcall(function() BloodmanStatHealth:SetText("<b><font color='#aaaaaa'>Health:</font></b> <font color='#888888'>—</font>") end)
pcall(function() BloodmanStatStuds:SetText("<b><font color='#aaaaaa'>Studs:</font></b> <font color='#888888'>—</font>") end)
pcall(function() BloodmanStatCord:SetText("<b><font color='#aaaaaa'>Cord:</font></b> <font color='#888888'>—</font>") end)
pcall(function() BloodmanStatInPlot:SetText("<b><font color='#aaaaaa'>In plot:</font></b> <font color='#888888'>—</font>") end)
pcall(function() BloodmanStatRagdoll:SetText("<b><font color='#aaaaaa'>Ragdoll:</font></b> <font color='#888888'>—</font>") end)
pcall(function() BloodmanStatKick:SetText("<b><font color='#aaaaaa'>Kick:</font></b> <font color='#888888'>—</font>") end)
pcall(function() BloodmanStatLeft:SetText("<b><font color='#aaaaaa'>Left:</font></b> <font color='#888888'>—</font>") end)
pcall(function() BloodmanStatRejoin:SetText("<b><font color='#aaaaaa'>Rejoin:</font></b> <font color='#888888'>—</font>") end)
pcall(function() BloodmanStatLastGrab:SetText("<b><font color='#aaaaaa'>Last grab:</font></b> <font color='#888888'>None</font>") end)
pcall(function() BloodmanStatPCLDBreak:SetText("<b><font color='#aaaaaa'>PCLD Break:</font></b> <font color='#888888'>—</font>") end)
return
end
local plr = bmGetPlayerByName(value)
if not plr then
_bloodmanAvatarCurrentUid = nil
pcall(function() BloodmanAvatarImage:SetImage("rbxassetid://0") end)
pcall(function() BloodmanAvatarLabel:SetText("<b><font color='#ff5555'>Игрок не найден</font></b>") end)
return
end
local uid = plr.UserId
if uid == _bloodmanAvatarCurrentUid then return end
_bloodmanAvatarCurrentUid = uid
pcall(function()
BloodmanAvatarLabel:SetText("<b><font color='#ffffff'>" .. plr.DisplayName .. "</font></b> <font color='#888888'>(@" .. plr.Name .. ")</font>")
end)
task.spawn(function()
local ok, content = pcall(Players.GetUserThumbnailAsync, Players, uid, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size420x420)
if ok and content and content ~= "" and plr.Parent then
if _bloodmanAvatarCurrentUid == uid then
pcall(function() BloodmanAvatarImage:SetImage(content) end)
end
end
end)
end

BloodmanTargets:AddDropdown("BloodmanTarget", {
Text = L("Цель", "Target"),
Values = bmPlayerList(),
Default = (bmPlayerList())[1] or nil,
Tooltip = L("Список игроков обновляется сам", "Player list updates automatically"),
Callback = function() bmUpdateAvatarDisplay() end,
})

function bmRefresh()
if not Options.BloodmanTarget then return end
local list = bmPlayerList()
pcall(function() Options.BloodmanTarget:SetValues(list) end)
local cur = Options.BloodmanTarget.Value
local stillHere = false
for _, n in ipairs(list) do if n == cur then stillHere = true break end end
if (not stillHere) and #list > 0 then
pcall(function() Options.BloodmanTarget:SetValue(list[1]) end)
end
bmUpdateAvatarDisplay()
end

Players.PlayerAdded:Connect(function() task.wait(0.3) bmRefresh() end)
Players.PlayerRemoving:Connect(function() task.defer(bmRefresh) end)

task.spawn(function() task.wait(1) bmRefresh(); bmUpdateAvatarDisplay() end)

BloodmanTargets:AddButton({
Text = L("Обновить список", "Refresh List"),
Func = function() bmRefresh() end,
})

BloodmanTargets:AddToggle("BloodmanLoopPlayer", {
Text = "Loop Player",
Default = false,
Tooltip = L("Держит и кикает выбранную цель, пока она не выйдет (даже после ресета)", "Holds and kicks selected target until they leave (even after reset)"),
})

BloodmanTargets:AddToggle("BloodmanLoopServer", {
Text = "Loop Server",
Default = false,
Tooltip = L("Кикает весь сервер по кругу, пока включено", "Kicks entire server in rotation while enabled"),
})

BloodmanTargets:AddToggle("BloodmanSkipFriends", {
Text = L("Не трогать друзей", "Skip Friends"),
Default = false,
Tooltip = L("Исключает друзей из кика по всему серверу", "Excludes friends from server-wide kick"),
})

BloodmanAction:AddDropdown("BloodmanMethod", {
Text = L("Что делаем", "Action"),
Values = { "Kick", "Void", "Kill", "Spin Kick", "Lock" },
Default = "Kick",
Tooltip = L("Работают: Кик (порт PoopHub), Килл, Войд, Спин Кик (радиус/скорость/высота), Лок", "Works: Kick (PoopHub port), Kill, Void, Spin Kick (radius/speed/height), Lock"),
Callback = function() end,
})

BloodmanAction:AddSlider("BmSpamDelay", {
Text = L("Спам задержка", "Spam Delay"),
Default = 0.05,
Min = 0.01,
Max = 1,
Rounding = 2,
Compact = false,
Tooltip = L("Задержка между спамом grab/drop. Меньше = быстрее", "Delay between grab/drop spam. Lower = faster"),
Callback = function() end,
})

BloodmanAction:AddSlider("SpinKickRadius", {
Text = "Spin Radius",
Default = 25,
Min = 5,
Max = 50,
Rounding = 0,
Compact = false,
Tooltip = L("Радиус орбиты цели вокруг тебя", "Target orbit radius around you"),
Callback = function(v)
bmSpinRadius = v
end,
})

BloodmanAction:AddSlider("SpinKickSpeed", {
Text = "Spin Speed",
Default = 0.25,
Min = 0.05,
Max = 1,
Rounding = 2,
Compact = false,
Tooltip = L("Скорость вращения (больше = быстрее)", "Rotation speed (higher = faster)"),
Callback = function(v)
bmSpinSpeed = v
end,
})

BloodmanAction:AddSlider("SpinKickHeight", {
Text = "Spin Height",
Default = 20,
Min = 5,
Max = 100,
Rounding = 0,
Compact = false,
Tooltip = L("Высота на которой цель вращается над тобой", "Height at which target rotates above you"),
Callback = function(v)
bmSpinKickHeight = v
end,
})

local bmAutoSitRunning = false

BloodmanAction:AddToggle("AutoSitBloodman", {
Text = "Auto Sit Bloodman",
Default = false,
Tooltip = L("Авто-спавнит блодмана и сажает в него. Если уже сидишь — не спавнит нового.", "Auto-spawns blobman and seats you. If already seated — doesn't spawn new one."),
})

Toggles.AutoSitBloodman:OnChanged(function()
if Toggles.AutoSitBloodman.Value then
bmAutoSitRunning = true
task.spawn(function()
while bmAutoSitRunning do
local char = LocalPlayer.Character
local hum = char and char:FindFirstChild("Humanoid")
local seat = hum and hum.SeatPart
if seat and seat.Parent and seat.Parent.Name == "CreatureBlobman" then
task.wait(0.5)
else
local ok = sitOnBlobman()
if not ok then
task.wait(1)
end
end
end
end)
else
bmAutoSitRunning = false
end
end)

local bmFreezeRunning = false
local bmFreezeConn = nil
local bmFreezeBP = nil

BloodmanAction:AddToggle("FreezeBloodman", {
Text = "Freeze Bloodman",
Default = false,
Tooltip = L("Замораживает блодмана на месте — не можешь двигаться.", "Freezes blobman in place — cannot move."),
})
Toggles.FreezeBloodman:OnChanged(function()
if Toggles.FreezeBloodman.Value then
bmFreezeRunning = true
bmFreezeConn = RunService.Heartbeat:Connect(function()
if not bmFreezeRunning then return end
local char = LocalPlayer.Character
local hum = char and char:FindFirstChild("Humanoid")
if not hum then return end
local seat = hum.SeatPart
if not (seat and seat.Parent and seat.Parent.Name == "CreatureBlobman") then return end
local blob = seat.Parent
local root = blob:FindFirstChild("HumanoidRootPart") or blob.PrimaryPart
if not root then return end
pcall(function()
root.AssemblyLinearVelocity = Vector3.zero
root.AssemblyAngularVelocity = Vector3.zero
end)
if not bmFreezeBP or not bmFreezeBP.Parent then
bmFreezeBP = Instance.new("BodyPosition")
bmFreezeBP.Name = "BmFreezeBP"
bmFreezeBP.P = 50000
bmFreezeBP.D = 1000
bmFreezeBP.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bmFreezeBP.Position = root.Position
bmFreezeBP.Parent = root
else
bmFreezeBP.Position = root.Position
end
end)
else
bmFreezeRunning = false
if bmFreezeConn then bmFreezeConn:Disconnect(); bmFreezeConn = nil end
if bmFreezeBP and bmFreezeBP.Parent then bmFreezeBP:Destroy() end
bmFreezeBP = nil
end
end)

local bmRestartBusy = false
BloodmanAction:AddButton({
Text = L("Перезапустить кик", "Restart Kick"),
Tooltip = L("Полный перезапуск кика на текущей цели: останавливает движки, освобождает линии и берёт цель заново — если холд иногда слетает", "Full kick restart on the current target: stops the engines, releases the lines and re-grabs the target — for when the hold sometimes slips"),
Func = function()
if bmRestartBusy then return end
bmRestartBusy = true
task.spawn(function()
local targetValue = Options.BloodmanTarget and Options.BloodmanTarget.Value or ""
local loopToggle = Toggles.BloodmanLoopPlayer
if not loopToggle or targetValue == "" then
Library:Notify(L("Bloodman: выбери игрока", "Bloodman: select a player"), 3)
else
Library:Notify(L("Bloodman: перезапускаю кик...", "Bloodman: restarting the kick..."), 2)
if loopToggle.Value then
loopToggle:SetValue(false)
end
local t0 = tick()
while bmKickingInProgress and tick() - t0 < 5 do
task.wait(0.1)
end
task.wait(0.4)
if not loopToggle.Value then
loopToggle:SetValue(true)
end
end
bmRestartBusy = false
end)
end,
})

function bmCheckMethod()
local m = (Options.BloodmanMethod and Options.BloodmanMethod.Value) or "Kick"
if m == "Kick" or m == "Kill" or m == "Void" or m == "Spin Kick" or m == "Lock" then
return true
end
Library:Notify("Bloodman: неизвестный метод " .. m, 4)
return false
end

_antiGrabState = {}
_antiGrabNotifyAt = {}

function bmAntiGrabNotify(name, kind)
local now = tick()
if _antiGrabNotifyAt[name] and now - _antiGrabNotifyAt[name] < 30 then return end
_antiGrabNotifyAt[name] = now
Library:Notify(L("Anti-Grab у " .. name .. ": " .. kind, "Anti-Grab on " .. name .. ": " .. kind), 4)
end

function bmAntiGrabSample(plr, now)
local st = _antiGrabState[plr.Name]
if not st then
st = { owned = false, ownedAt = 0, grabs = 0, escapes = 0, unowned = 0, lastT = now, lastSeen = now, engaged = false, detected = false, kind = nil }
_antiGrabState[plr.Name] = st
end
local char = plr.Character
local head = char and char:FindFirstChild("Head")
local hum = char and char:FindFirstChildOfClass("Humanoid")
if not char or not head or not hum or hum.Health <= 0 then
if char and st.owned and st.grabs > 0 and (not hum or hum.Health <= 0) and (now - st.ownedAt) < 1.2 then
st.escapes = st.escapes + 1
st.kind = "авто-ресет на захвате"
st.detected = true
bmAntiGrabNotify(plr.Name, st.kind)
end
st.owned = false
st.lastT = now
st.lastSeen = now
return
end
if hum.SeatPart then
st.unowned = 0
st.owned = false
st.lastT = now
st.lastSeen = now
return
end
local dt = math.min(now - st.lastT, 0.5)
local po = head:FindFirstChild("PartOwner")
local poName = po and tostring(po.Value) or ""
local mine = poName == LocalPlayer.Name
if mine then
st.unowned = 0
if not st.owned then
st.owned = true
st.ownedAt = now
st.grabs = st.grabs + 1
end
elseif poName ~= "" then
st.unowned = 0
st.owned = false
else
if st.owned then
st.owned = false
if now - st.ownedAt < 0.45 then
st.escapes = st.escapes + 1
st.kind = "страгл-эскейп (Unbind)"
if st.escapes >= 3 then
st.detected = true
bmAntiGrabNotify(plr.Name, st.kind)
end
end
end
st.unowned = st.unowned + dt
local myChar = LocalPlayer.Character
local myRoot = myChar and myChar:FindFirstChild("HumanoidRootPart")
local tRoot = char:FindFirstChild("HumanoidRootPart")
if st.grabs == 0 and st.unowned > 3 and not st.detected and myRoot and tRoot and (tRoot.Position - myRoot.Position).Magnitude < 60 then
st.detected = true
st.kind = "владение не берётся (якорь/анти-граб)"
bmAntiGrabNotify(plr.Name, st.kind)
end
end
st.lastT = now
st.lastSeen = now
end


task.spawn(function()
while true do
task.wait(0.1)
local watchOn = false
local now = tick()
local cands = {}
if watchOn then
pcall(function()
for pl in pairs(tbl20.SelectedPlayers) do
if typeof(pl) == "Instance" and pl:IsA("Player") and pl ~= LocalPlayer then
cands[pl] = true
end
end
end)
pcall(function()
do 
local mc = LocalPlayer.Character
local mh = mc and mc:FindFirstChildOfClass("Humanoid")
local seat = mh and mh.SeatPart
if seat and seat.Parent and seat.Parent.Name == "CreatureBlobman" then
local br = seat.Parent:FindFirstChild("HumanoidRootPart") or seat.Parent.PrimaryPart
if br then
for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer then
local pc = plr.Character
local pr = pc and pc:FindFirstChild("HumanoidRootPart")
if pr and (pr.Position - br.Position).Magnitude < 40 then
cands[plr] = true
end
end
end
end
end
end
end)
end
for plr in pairs(cands) do
if plr.Parent then
pcall(bmAntiGrabSample, plr, now)
end
end
for name, st in pairs(_antiGrabState) do
local still = false
for c in pairs(cands) do
if c.Name == name then still = true break end
end
if still then
st.engaged = true
elseif now - st.lastSeen > 120 then
_antiGrabState[name] = nil
else
st.engaged = false
end
end
end
end)

_antiGrabPunishHb = RunService.Heartbeat:Connect(function()
local bmMethod = Options.BloodmanMethod and Options.BloodmanMethod.Value or "Kick"
local bmKickMode = bmKickingInProgress and (bmMethod == "Kick" or bmMethod == "Kill" or bmMethod == "Void") or false
local punish = false
if not punish then return end
local myName = LocalPlayer.Name
local now = tick()
for name, st in pairs(_antiGrabState) do
local plr = Players:FindFirstChild(name)
local char = plr and plr.Character
local head = char and char:FindFirstChild("Head")
local po = head and head:FindFirstChild("PartOwner")
if po and tostring(po.Value) == myName then
local armed = st.detected or st.escapes >= 2 or (bmKickingInProgress and st.escapes >= 1)
if armed and (not st.snapUntil or now - st.snapUntil > 2) then
st.snapUntil = now + 0.5
st.escapes = 0
st.detected = false
st.kind = nil
st.owned = false
Library:Notify(L("Анти-граб " .. name .. " пробит: войд-снап", "Anti-grab on " .. name .. " pierced: void snap"), 4)
end
if st.snapUntil and now <= st.snapUntil then
local tRoot = char:FindFirstChild("HumanoidRootPart") or head
pcall(function()
tRoot.AssemblyLinearVelocity = Vector3.zero
tRoot.CFrame = tRoot.CFrame - Vector3.new(0, 100000, 0)
end)
end
end
end
end)

_antiGrabRetryAt = {}
task.spawn(function()
while true do
task.wait(0.5)
if false then
local now = tick()
pcall(function()
for p in pairs(tbl20.SelectedPlayers) do
if typeof(p) == "Instance" and p:IsA("Player") and p ~= LocalPlayer and p.Parent then
local st = _antiGrabState[p.Name]
local char = p.Character
local hum = char and char:FindFirstChildOfClass("Humanoid")
if st and (st.detected or st.escapes >= 1) and hum and hum.Health > 0 then
if not _antiGrabRetryAt[p.Name] or now - _antiGrabRetryAt[p.Name] > 2.5 then
_antiGrabRetryAt[p.Name] = now
tbl20.NetworkKickQueue[p] = { loopCall = false, isServer = false, queuedAt = now }
bmAntiGrabNotify(p.Name, "перезапуск кика")
end
end
end
end
end)
end
end
end)

local prSNW = game:GetService("ReplicatedStorage"):FindFirstChild("GrabEvents")
prSNW = prSNW and prSNW:FindFirstChild("SetNetworkOwner")
local prToys = game:GetService("ReplicatedStorage"):FindFirstChild("MenuToys")
local prSpawnRF = prToys and prToys:FindFirstChild("SpawnToyRemoteFunction")
local prDestroy = prToys and prToys:FindFirstChild("DestroyToy")
local prPallet, prSound, prMain, prBusy = nil, nil, nil, false

local function prSpawnPallet()
local myToys = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
if not myToys then return nil end
local pallet = myToys:FindFirstChild("RagdollRSC") or myToys:FindFirstChild("PalletLightBrown")
if pallet and pallet.Parent then return pallet end
if not prSpawnRF then return nil end
local myChar = LocalPlayer.Character
local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
if not myHRP then return nil end
pcall(function() prSpawnRF:InvokeServer("PalletLightBrown", myHRP.CFrame * CFrame.new(0, 50, 0), Vector3.zero) end)
local t = tick() + 5
repeat task.wait(0.05) until (myToys:FindFirstChild("PalletLightBrown") or tick() > t)
return myToys:FindFirstChild("PalletLightBrown")
end

local function palletRagdoll(targetChar)
if prBusy or not targetChar or not targetChar.Parent then return false end
local head = targetChar:FindFirstChild("Head")
local torso = head and targetChar:FindFirstChild("Torso")
local humanoid = torso and targetChar:FindFirstChild("Humanoid")
humanoid = humanoid and humanoid:FindFirstChild("Ragdolled")
if not humanoid then return false end
if humanoid.Value then return true end
prBusy = true
local ok = false
local pallet = prSpawnPallet()
if pallet and pallet.Parent then
local soundPart = pallet:FindFirstChild("SoundPart")
local main = pallet:FindFirstChild("Main")
if soundPart and main then
if prSNW then
local t2 = tick() + 2
repeat
pcall(function() prSNW:FireServer(soundPart, soundPart.CFrame) end)
task.wait()
until soundPart:FindFirstChild("PartOwner") or tick() > t2
end
pallet.Name = "RagdollRSC"
for _, child in pairs(pallet:GetChildren()) do
if child:IsA("BasePart") then child.CanCollide = false end
end
if not main:FindFirstChild("RagdollBP") then
local bp = Instance.new("BodyPosition")
bp.Name = "RagdollBP"
bp.D = 1500
bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bp.Position = Vector3.new(0, 100000, 0)
bp.Parent = main
end
prPallet, prSound, prMain = pallet, soundPart, main
local started = tick()
while true do
if not (prSound and prSound.Parent and torso.Parent and head.Parent) then break end
prSound.Position = torso.Position
prSound.Velocity = Vector3.new(0, -10, 0)
task.wait(0.1)
if humanoid.Value or not prMain or not prMain.Parent or not targetChar.Parent or not head:FindFirstChild("PartOwner") or tick() - started > 0.5 then break end
end
ok = humanoid.Value
end
end
prBusy = false
return ok
end

OBV_RagdollTarget = palletRagdoll

task.spawn(function()
while true do
task.wait(0.2)
local targets = {}
if bmKickingInProgress then
local pl = bmGetPlayerByName(Options.BloodmanTarget and Options.BloodmanTarget.Value or "")
if pl and pl.Parent then targets[pl] = true end
end
pcall(function()
for p in pairs(tbl20.NetworkKickQueue) do
if typeof(p) == "Instance" and p:IsA("Player") and p.Parent then
targets[p] = true
end
end
end)
for pl in pairs(targets) do
local char = pl.Character
if char and char:FindFirstChild("Torso") then
task.spawn(pcall, palletRagdoll, char)
end
end
end
end)

Toggles.BloodmanLoopPlayer:OnChanged(function()
if Toggles.BloodmanLoopPlayer.Value then
if not bmCheckMethod() then
Toggles.BloodmanLoopPlayer:SetValue(false)
return
end
local rawValue = Options.BloodmanTarget and Options.BloodmanTarget.Value or ""
local targetPlayer = bmGetPlayerByName(rawValue)
local targetName = targetPlayer and targetPlayer.Name or ""
if not targetName or targetName == "" then
Library:Notify(L("Bloodman: выбери игрока", "Bloodman: select a player"), 3)
Toggles.BloodmanLoopPlayer:SetValue(false)
return
end
local m = (Options.BloodmanMethod and Options.BloodmanMethod.Value) or "Kick"
bmLoopPlayer = true
task.spawn(function()
Library:Notify("Bloodman: " .. m .. " → " .. targetName, 2)
if m == "Kill" then
bmKillTarget(targetName, function() return bmLoopPlayer end, true, nil)
elseif m == "Void" then
bmVoidTarget(targetName, function() return bmLoopPlayer end, true, nil)
elseif m == "Spin Kick" then
bmSpinKickTarget(targetName, function() return bmLoopPlayer end, true, nil)
elseif m == "Lock" then
bmLockTarget(targetName, function() return bmLoopPlayer end, true, nil)
else
bmKickTarget(targetName, function() return bmLoopPlayer end, true, nil)
end
bmLoopPlayer = false
if Toggles.BloodmanLoopPlayer.Value then
Toggles.BloodmanLoopPlayer:SetValue(false)
end
end)
else
bmLoopPlayer = false
end
end)

Toggles.BloodmanLoopServer:OnChanged(function()
if Toggles.BloodmanLoopServer.Value then
if not bmCheckMethod() then
Toggles.BloodmanLoopServer:SetValue(false)
return
end
bmLoopServer = true
task.spawn(function()
local m = (Options.BloodmanMethod and Options.BloodmanMethod.Value) or "Kick"
Library:Notify("Bloodman: " .. m .. " → сервер", 2)
while bmLoopServer do
if m == "Kill" then
bmKillServer(function() return bmLoopServer end)
elseif m == "Void" then
bmVoidServer(function() return bmLoopServer end)
elseif m == "Spin Kick" then
bmSpinKickServer(function() return bmLoopServer end)
elseif m == "Kick" then
bmKickServer(function() return bmLoopServer end)
elseif m == "Lock" then
bmLockServer(function() return bmLoopServer end)
end
task.wait(0.2)
end
end)
else
bmLoopServer = false
end
end)

Toggles.BloodmanSkipFriends:OnChanged(function()
_friendWhitelistEnabled = Toggles.BloodmanSkipFriends.Value
if _friendWhitelistEnabled then
task.spawn(function()
LoadFriendsIntoWhitelist()
end)
else
_friendWhitelist = {}
end
end)

function bmGrabTarget(targetName, keepGoing, reacquire, maxSeconds)
fnStart = tick()
grabSavedPos = nil
dragging = false
grabStartTime = 0
blobRoot = nil

while keepGoing() do
if maxSeconds and (tick() - fnStart) > maxSeconds then break end

local target = Players:FindFirstChild(targetName)
if not target then break end

local char = LocalPlayer.Character
local myHum = char and char:FindFirstChildOfClass("Humanoid")
local myHRP = char and char:FindFirstChild("HumanoidRootPart")
if not (myHum and myHRP) then
RunService.Heartbeat:Wait()
continue
end

if not grabSavedPos then
grabSavedPos = myHRP.CFrame
end

local seat = myHum.SeatPart
if not (seat and seat.Parent and seat.Parent.Name == "CreatureBlobman") then
if not bmKickingInProgress then
if not sitOnBlobman() then
RunService.Heartbeat:Wait()
continue
end
task.wait(0.1)
seat = myHum.SeatPart
if seat and seat.Parent then
blobRoot = seat.Parent:FindFirstChild("HumanoidRootPart") or seat.Parent.PrimaryPart
if blobRoot then
blobRoot.CFrame = grabSavedPos
blobRoot.AssemblyLinearVelocity = Vector3.zero
end
end
myHRP.CFrame = grabSavedPos
myHRP.AssemblyLinearVelocity = Vector3.zero
RunService.Heartbeat:Wait()
continue
else
RunService.Heartbeat:Wait()
continue
end
end

local blob = seat.Parent
blobRoot = blob:FindFirstChild("HumanoidRootPart") or blob.PrimaryPart

myHRP.CFrame = grabSavedPos
myHRP.AssemblyLinearVelocity = Vector3.zero
if blobRoot then
blobRoot.CFrame = grabSavedPos
blobRoot.AssemblyLinearVelocity = Vector3.zero
end

local tChar = target.Character
local tRoot = tChar and tChar:FindFirstChild("HumanoidRootPart")
local tHum = tChar and tChar:FindFirstChild("Humanoid")
if not (tRoot and tHum and tHum.Health > 0) then
dragging = false
grabStartTime = 0
RunService.Heartbeat:Wait()
continue
end

bmKickingInProgress = true

local remoteFolder = blob:FindFirstChild("BlobmanSeatAndOwnerScript")
local grab = remoteFolder and remoteFolder:FindFirstChild("CreatureGrab")
local drop = remoteFolder and remoteFolder:FindFirstChild("CreatureDrop")
local L_Det = blob:FindFirstChild("LeftDetector")
local R_Det = blob:FindFirstChild("RightDetector")
local L_Weld = L_Det and (L_Det:FindFirstChild("LeftWeld") or L_Det:FindFirstChild("RigidConstraint"))
local R_Weld = R_Det and (R_Det:FindFirstChild("RightWeld") or R_Det:FindFirstChild("RigidConstraint"))

if grab and drop and L_Weld and R_Weld then
pcall(function()
grab:FireServer(L_Det, tRoot, L_Weld)
grab:FireServer(R_Det, tRoot, R_Weld)
drop:FireServer(L_Weld, tRoot)
drop:FireServer(R_Weld, tRoot)
end)
end

if not dragging then
myHRP.CFrame = tRoot.CFrame
if blobRoot then blobRoot.CFrame = tRoot.CFrame end
pcall(function()
tHum.PlatformStand = true
tRoot.AssemblyLinearVelocity = Vector3.zero
GE_Bloodman.SetNetworkOwner:FireServer(tRoot, myHRP.CFrame)
if createGrabLineEvent then
createGrabLineEvent:FireServer(tRoot, Vector3.zero, tRoot.Position, false)
end
end)

if grabStartTime == 0 then grabStartTime = tick() end
if tick() - grabStartTime > getKickGrabWait() then
dragging = true
grabStartTime = 0
end
else
local holdPos = grabSavedPos * CFrame.new(0, 0, 3)
myHRP.CFrame = grabSavedPos
myHRP.AssemblyLinearVelocity = Vector3.zero
if blobRoot then
blobRoot.CFrame = grabSavedPos
blobRoot.AssemblyLinearVelocity = Vector3.zero
end

pcall(function()
tRoot.CFrame = holdPos
tRoot.AssemblyLinearVelocity = Vector3.zero
tHum.PlatformStand = true
GE_Bloodman.SetNetworkOwner:FireServer(tRoot, holdPos)
GE_Bloodman.DestroyGrabLine:FireServer(tRoot)
if createGrabLineEvent then
createGrabLineEvent:FireServer(tRoot, Vector3.zero, tRoot.Position, false)
end
end)
end

RunService.Heartbeat:Wait()
end

bmKickingInProgress = false
if myHRP and myHRP.Parent then
myHRP.CFrame = grabSavedPos or myHRP.CFrame
myHRP.AssemblyLinearVelocity = Vector3.zero
end
grabSavedPos = nil
dragging = false

folder = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
blobToDestroy = folder and folder:FindFirstChild("CreatureBlobman")
if blobToDestroy then
local destroyRem = ReplicatedStorage:FindFirstChild("MenuToys")
destroyRem = destroyRem and destroyRem:FindFirstChild("DestroyToy")
if destroyRem then destroyRem:FireServer(blobToDestroy) end
end
end

function bmGrabServer(keepGoing)
for _, plr in ipairs(Players:GetPlayers()) do
if keepGoing and not keepGoing() then return end
if plr ~= LocalPlayer then
bmGrabTarget(plr.Name, function()
return (not keepGoing) or keepGoing()
end, false, 12)
end
end
end

bmSpinAngle = 0
bmSpinRadius = 25
bmSpinSpeed = 0.25
bmSpinKickHeight = 20

function bmSpinKickTarget(targetName, keepGoing, reacquire, maxSeconds)
bmSpinAngle = 0
bmBlobHoldEngine(targetName, keepGoing, reacquire, maxSeconds, "spin")
end

function bmSpinKickServer(keepGoing)
for _, plr in ipairs(Players:GetPlayers()) do
if keepGoing and not keepGoing() then return end
if plr ~= LocalPlayer and not isFriendWhitelisted(plr) then
bmSpinKickTarget(plr.Name, function()
return (not keepGoing) or keepGoing()
end, false, 12)
end
end
end

function bmKillTarget(targetName, keepGoing, reacquire, maxSeconds)
fnStart = tick()
killSavedPos = nil
highPos = nil
myHRP = nil  

local function isOwningTarget(targetHead)
if not targetHead then return false end
local po = targetHead:FindFirstChild("PartOwner")
return po and po.Value == LocalPlayer.Name
end

while keepGoing() do
if maxSeconds and (tick() - fnStart) > maxSeconds then break end

local target = Players:FindFirstChild(targetName)
if not target then
if not reacquire then break end
RunService.Heartbeat:Wait()
continue
end

local char = LocalPlayer.Character
local myHum = char and char:FindFirstChildOfClass("Humanoid")
local myHRP = char and char:FindFirstChild("HumanoidRootPart")
if not (myHum and myHRP) then
RunService.Heartbeat:Wait()
continue
end

if not killSavedPos then
killSavedPos = myHRP.CFrame
highPos = killSavedPos * CFrame.new(0, 500, 0)
myHRP.CFrame = highPos
myHRP.AssemblyLinearVelocity = Vector3.zero
task.wait(0.1)
end

local seat = myHum.SeatPart
if not (seat and seat.Parent and seat.Parent.Name == "CreatureBlobman") then
if not bmKickingInProgress then
if not sitOnBlobman() then
RunService.Heartbeat:Wait()
continue
end
task.wait(0.1)
seat = myHum.SeatPart
if seat and seat.Parent then
local br = seat.Parent:FindFirstChild("HumanoidRootPart") or seat.Parent.PrimaryPart
if br then
br.CFrame = highPos
br.AssemblyLinearVelocity = Vector3.zero
end
end
myHRP.CFrame = highPos
myHRP.AssemblyLinearVelocity = Vector3.zero
RunService.Heartbeat:Wait()
continue
else
RunService.Heartbeat:Wait()
continue
end
end

local blob = seat.Parent
local blobRoot = blob:FindFirstChild("HumanoidRootPart") or blob.PrimaryPart

myHRP.CFrame = highPos
myHRP.AssemblyLinearVelocity = Vector3.zero

local tChar = target.Character
local tRoot = tChar and tChar:FindFirstChild("HumanoidRootPart")
local tHead = tChar and tChar:FindFirstChild("Head")
local tHum = tChar and tChar:FindFirstChildOfClass("Humanoid")
if not (tRoot and tHum and tHum.Health > 0) then
RunService.Heartbeat:Wait()
continue
end

bmKickingInProgress = true

if blobRoot then
blobRoot.CFrame = tRoot.CFrame + Vector3.new(0, 5, 0)
blobRoot.AssemblyLinearVelocity = Vector3.zero
end
task.wait(0.1)

pcall(function() GE_Bloodman.SetNetworkOwner:FireServer(tRoot, myHRP.CFrame) end)
task.wait(0.1)
pcall(function() GE_Bloodman.SetNetworkOwner:FireServer(tRoot, myHRP.CFrame) end)
task.wait(0.1)

local script = blob:FindFirstChild("BlobmanSeatAndOwnerScript")
local L_Det = blob:FindFirstChild("LeftDetector")
local R_Det = blob:FindFirstChild("RightDetector")
local L_Weld = L_Det and (L_Det:FindFirstChild("LeftWeld") or L_Det:FindFirstChild("RigidConstraint"))
local R_Weld = R_Det and (R_Det:FindFirstChild("RightWeld") or R_Det:FindFirstChild("RigidConstraint"))

if script and (L_Det or R_Det) and (L_Weld or R_Weld) then
local grab = script:FindFirstChild("CreatureGrab")
local release = script:FindFirstChild("CreatureRelease")
local drop = script:FindFirstChild("CreatureDrop")
if grab then
for _ = 1, 10 do
if not keepGoing() then break end
pcall(function()
if L_Det and L_Weld then grab:FireServer(L_Det, tRoot, L_Weld) end
if R_Det and R_Weld then grab:FireServer(R_Det, tRoot, R_Weld) end
end)
task.wait(0.05)
end
end
end
task.wait(0.2)

pcall(function()
tHum.BreakJointsOnDeath = false
tHum.WalkSpeed = 0
tHum.JumpPower = 0
tHum.PlatformStand = true
end)

if tHum.SeatPart == nil then
pcall(function()
if tHum.RigType ~= Enum.HumanoidRigType.R15 then
tHum.RigType = Enum.HumanoidRigType.R15
end
if tHum.RigType ~= Enum.HumanoidRigType.R6 then
tHum.RigType = Enum.HumanoidRigType.R6
end
if tHum.RigType ~= Enum.HumanoidRigType.R15 then
tHum.RigType = Enum.HumanoidRigType.R15
end
end)
end
task.wait(0.05)

if script and L_Det and L_Weld then
pcall(function()
local grab = script:FindFirstChild("CreatureGrab")
local release = script:FindFirstChild("CreatureRelease")
if grab and release then
for _ = 1, 4 do
grab:FireServer(L_Det, tRoot, L_Weld)
task.wait(0.05)
release:FireServer(L_Weld, tRoot)
end
end
end)
end

pcall(function() GE_Bloodman.SetNetworkOwner:FireServer(tRoot, tRoot.CFrame) end)
task.wait(0.05)
pcall(function() tRoot.AssemblyLinearVelocity = Vector3.new(0, 1e10, 0) end)
task.wait(0.05)
if script and L_Det and L_Weld then
pcall(function()
local grab = script:FindFirstChild("CreatureGrab")
local release = script:FindFirstChild("CreatureRelease")
if grab and release then
for _ = 1, 4 do
grab:FireServer(L_Det, tRoot, L_Weld)
task.wait(0.05)
release:FireServer(L_Weld, tRoot)
end
end
end)
end
pcall(function() tRoot.AssemblyLinearVelocity = Vector3.new(0, 1e10, 0) end)
task.wait(0.1)

if script and L_Det and L_Weld then
pcall(function()
local grab = script:FindFirstChild("CreatureGrab")
local release = script:FindFirstChild("CreatureRelease")
if grab and release then
for _ = 1, 4 do
grab:FireServer(L_Det, tRoot, L_Weld)
task.wait(0.05)
release:FireServer(L_Weld, tRoot)
end
end
end)
end

if blobRoot then
blobRoot.CFrame = highPos
blobRoot.AssemblyLinearVelocity = Vector3.zero
end
myHRP.CFrame = highPos
myHRP.AssemblyLinearVelocity = Vector3.zero

bmKickingInProgress = false

if not reacquire then break end

RunService.Heartbeat:Wait()
end

bmKickingInProgress = false
finalChar = LocalPlayer.Character
finalHRP = finalChar and finalChar:FindFirstChild("HumanoidRootPart")
if finalHRP and killSavedPos then
finalHRP.CFrame = killSavedPos
finalHRP.AssemblyLinearVelocity = Vector3.zero
end
killSavedPos = nil
highPos = nil
end

function bmKillServer(keepGoing)
while (not keepGoing or keepGoing()) do
for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer and plr.Character then
local tHum = plr.Character:FindFirstChildOfClass("Humanoid")
if tHum and tHum.Health > 0 then
bmKillTarget(plr.Name, function() return true end, false, 8)
end
end
end
task.wait(0.2)
end
end

function bmVoidTarget(targetName, keepGoing, reacquire, maxSeconds)
fnStart = tick()
voidSavedPos = nil
highPos = nil
myHRP = nil  

local function isOwningTarget(targetHead)
if not targetHead then return false end
local po = targetHead:FindFirstChild("PartOwner")
return po and po.Value == LocalPlayer.Name
end

while keepGoing() do
if maxSeconds and (tick() - fnStart) > maxSeconds then break end

local target = Players:FindFirstChild(targetName)
if not target then
if not reacquire then break end
RunService.Heartbeat:Wait()
continue
end

local char = LocalPlayer.Character
local myHum = char and char:FindFirstChildOfClass("Humanoid")
local myHRP = char and char:FindFirstChild("HumanoidRootPart")
if not (myHum and myHRP) then
RunService.Heartbeat:Wait()
continue
end

if not voidSavedPos then
voidSavedPos = myHRP.CFrame
highPos = voidSavedPos * CFrame.new(0, 500, 0)
myHRP.CFrame = highPos
myHRP.AssemblyLinearVelocity = Vector3.zero
task.wait(0.1)
end

local seat = myHum.SeatPart
if not (seat and seat.Parent and seat.Parent.Name == "CreatureBlobman") then
if not bmKickingInProgress then
if not sitOnBlobman() then
RunService.Heartbeat:Wait()
continue
end
task.wait(0.1)
seat = myHum.SeatPart
if seat and seat.Parent then
local br = seat.Parent:FindFirstChild("HumanoidRootPart") or seat.Parent.PrimaryPart
if br then
br.CFrame = highPos
br.AssemblyLinearVelocity = Vector3.zero
end
end
myHRP.CFrame = highPos
myHRP.AssemblyLinearVelocity = Vector3.zero
RunService.Heartbeat:Wait()
continue
else
RunService.Heartbeat:Wait()
continue
end
end

local blob = seat.Parent
local blobRoot = blob:FindFirstChild("HumanoidRootPart") or blob.PrimaryPart

myHRP.CFrame = highPos
myHRP.AssemblyLinearVelocity = Vector3.zero

local tChar = target.Character
local tRoot = tChar and tChar:FindFirstChild("HumanoidRootPart")
local tHead = tChar and tChar:FindFirstChild("Head")
local tHum = tChar and tChar:FindFirstChildOfClass("Humanoid")
if not (tRoot and tHum and tHum.Health > 0) then
RunService.Heartbeat:Wait()
continue
end

bmKickingInProgress = true

if blobRoot then
blobRoot.CFrame = tRoot.CFrame + Vector3.new(0, 5, 0)
blobRoot.AssemblyLinearVelocity = Vector3.zero
end
task.wait(0.1)

pcall(function() GE_Bloodman.SetNetworkOwner:FireServer(tRoot, myHRP.CFrame) end)
task.wait(0.1)
pcall(function() GE_Bloodman.SetNetworkOwner:FireServer(tRoot, myHRP.CFrame) end)
task.wait(0.1)

local script = blob:FindFirstChild("BlobmanSeatAndOwnerScript")
local L_Det = blob:FindFirstChild("LeftDetector")
local R_Det = blob:FindFirstChild("RightDetector")
local L_Weld = L_Det and (L_Det:FindFirstChild("LeftWeld") or L_Det:FindFirstChild("RigidConstraint"))
local R_Weld = R_Det and (R_Det:FindFirstChild("RightWeld") or R_Det:FindFirstChild("RigidConstraint"))

if script and (L_Det or R_Det) and (L_Weld or R_Weld) then
local grab = script:FindFirstChild("CreatureGrab")
local release = script:FindFirstChild("CreatureRelease")
local drop = script:FindFirstChild("CreatureDrop")
if grab then
for _ = 1, 10 do
if not keepGoing() then break end
pcall(function()
if L_Det and L_Weld then grab:FireServer(L_Det, tRoot, L_Weld) end
if R_Det and R_Weld then grab:FireServer(R_Det, tRoot, R_Weld) end
end)
task.wait(0.05)
end
end
end
task.wait(0.2)

pcall(function()
tHum.PlatformStand = true
tHum.WalkSpeed = 0
tHum.JumpPower = 0
end)

for _ = 1, 8 do
pcall(function()
tRoot.CFrame = tRoot.CFrame - Vector3.new(0, 100000, 0)
local bv2 = Instance.new("BodyVelocity", tRoot)
bv2.Velocity = Vector3.new(0, -9e9, 0)
bv2.MaxForce = Vector3.one * math.huge
game:GetService("Debris"):AddItem(bv2, 0.1)
end)
task.wait(0.05)
end

if script and L_Det and L_Weld then
pcall(function()
local grab = script:FindFirstChild("CreatureGrab")
local release = script:FindFirstChild("CreatureRelease")
local drop = script:FindFirstChild("CreatureDrop")
for _ = 1, 3 do
if grab then grab:FireServer(L_Det, tRoot, L_Weld) end
task.wait(0.05)
if drop then drop:FireServer(L_Weld, tRoot) end
if release then release:FireServer(L_Weld, tRoot) end
task.wait(0.05)
end
end)
end

pcall(function()
tRoot.CFrame = tRoot.CFrame - Vector3.new(0, 1000000, 0)
local bvFinal = Instance.new("BodyVelocity", tRoot)
bvFinal.Velocity = Vector3.new(0, -9e9, 0)
bvFinal.MaxForce = Vector3.one * math.huge
game:GetService("Debris"):AddItem(bvFinal, 0.5)
end)
task.wait(0.3)

if blobRoot then
blobRoot.CFrame = highPos
blobRoot.AssemblyLinearVelocity = Vector3.zero
end
myHRP.CFrame = highPos
myHRP.AssemblyLinearVelocity = Vector3.zero

bmKickingInProgress = false

if not reacquire then break end

RunService.Heartbeat:Wait()
end

bmKickingInProgress = false
finalChar = LocalPlayer.Character
finalHRP = finalChar and finalChar:FindFirstChild("HumanoidRootPart")
if finalHRP and voidSavedPos then
finalHRP.CFrame = voidSavedPos
finalHRP.AssemblyLinearVelocity = Vector3.zero
end
voidSavedPos = nil
highPos = nil
end

function bmVoidServer(keepGoing)
while (not keepGoing or keepGoing()) do
for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer and plr.Character then
local tHum = plr.Character:FindFirstChildOfClass("Humanoid")
if tHum and tHum.Health > 0 then
bmVoidTarget(plr.Name, function() return true end, false, 8)
end
end
end
task.wait(0.2)
end
end

print("[OBLIVION DEBUG] Before MISC section")
do
MiscSection = Tabs.Misc:AddLeftGroupbox("Combat & Automation", "bot")

do
MiscSection:AddToggle("EnableFPSBoost", {
Text = "FPS Boost",
Default = false,
Tooltip = L("Меняет текстуры/материалы/эффекты. Дальность камеры НЕ трогает.", "Changes textures/materials/effects. Does NOT affect camera distance."),
})

local Lighting = game:GetService("Lighting")
local UserGameSettings = UserSettings():GetService("UserGameSettings")

fpsOriginal = {
saved = false,
GlobalShadows = nil,
QualityLevel = nil,
materials = {},
castShadow = {},
decals = {},
enabled = {},
}
fpsProcessing = false

function isCharacterPart(inst)
local cur = inst
while cur and cur ~= workspace do
if cur:FindFirstChildOfClass("Humanoid") then return true end
if Players:GetPlayerFromCharacter(cur) then return true end
cur = cur.Parent
end
return false
end

function saveOnce()
if fpsOriginal.saved then return end
pcall(function() fpsOriginal.GlobalShadows = Lighting.GlobalShadows end)
pcall(function() fpsOriginal.QualityLevel = UserGameSettings.GraphicsQualityLevel end)
fpsOriginal.saved = true
end

function restoreFps()
pcall(function() Lighting.GlobalShadows = fpsOriginal.GlobalShadows end)
pcall(function() UserGameSettings.GraphicsQualityLevel = fpsOriginal.QualityLevel end)
for obj, mat in pairs(fpsOriginal.materials) do
if obj and obj.Parent then obj.Material = mat end
end
for obj, cs in pairs(fpsOriginal.castShadow) do
if obj and obj.Parent then pcall(function() obj.CastShadow = cs end) end
end
for obj, tr in pairs(fpsOriginal.decals) do
if obj and obj.Parent then obj.Transparency = tr end
end
for obj, en in pairs(fpsOriginal.enabled) do
if obj and obj.Parent then pcall(function() obj.Enabled = en end) end
end
end

function applyFpsChunked()
if fpsProcessing then return end
fpsProcessing = true
saveOnce()

pcall(function() Lighting.GlobalShadows = false end)
pcall(function() UserGameSettings.GraphicsQualityLevel = Enum.QualityLevel.Level01 end)
pcall(function() sethiddenproperty(workspace.Terrain, "Decoration", false) end)

pcall(function()
local types = {"Texture", "Decal", "SurfaceGui"}
for _, t in ipairs(types) do
settings():GetService("RenderingSettings")["TextureQuality"] = 1
end
end)

local all = workspace:GetDescendants()
for i, v in ipairs(all) do
if not (Toggles.EnableFPSBoost and Toggles.EnableFPSBoost.Value) then break end
if v:IsA("BasePart") and not isCharacterPart(v) then
if fpsOriginal.materials[v] == nil then fpsOriginal.materials[v] = v.Material end
if fpsOriginal.castShadow[v] == nil then pcall(function() fpsOriginal.castShadow[v] = v.CastShadow end) end
v.Material = Enum.Material.SmoothPlastic
pcall(function() v.CastShadow = false end)
elseif v:IsA("Decal") or v:IsA("Texture") then
if fpsOriginal.decals[v] == nil then fpsOriginal.decals[v] = v.Transparency end
v.Transparency = 1
elseif v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") then
if fpsOriginal.enabled[v] == nil then pcall(function() fpsOriginal.enabled[v] = v.Enabled end) end
pcall(function() v.Enabled = false end)
end
if i % 250 == 0 then task.wait() end
end
fpsProcessing = false
end

fpsDescendantConn = nil
Toggles.EnableFPSBoost:OnChanged(function()
if Toggles.EnableFPSBoost.Value then
task.spawn(applyFpsChunked)
if fpsDescendantConn then fpsDescendantConn:Disconnect() end
fpsDescendantConn = workspace.DescendantAdded:Connect(function(v)
if not (Toggles.EnableFPSBoost and Toggles.EnableFPSBoost.Value) then return end
if v:IsA("BasePart") and not isCharacterPart(v) then
v.Material = Enum.Material.SmoothPlastic
pcall(function() v.CastShadow = false end)
elseif v:IsA("Decal") or v:IsA("Texture") then
v.Transparency = 1
elseif v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Beam") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") then
pcall(function() v.Enabled = false end)
end
end)
else
fpsProcessing = false
if fpsDescendantConn then fpsDescendantConn:Disconnect(); fpsDescendantConn = nil end
restoreFps()
pcall(function() sethiddenproperty(workspace.Terrain, "Decoration", true) end)
end
end)
end

VirtualInputManager = game:GetService("VirtualInputManager")

MiscSection:AddToggle("EnableLockTap", {
Text = "Lock Tap",
Default = false,
Tooltip = "Бинд: камера к ближайшему + ЛКМ",
}):AddKeyPicker("LockTapKeybind", {
Default = "None",
SyncToggleState = false,
Mode = "Hold",
Text = "Lock Tap Key",
NoUI = true,
})

MiscSection:AddSlider("LockDistance", {
Text = "Lock Distance",
Default = 50,
Min = 10,
Max = 500,
Rounding = 0,
Compact = false,
Tooltip = "Дистанция для Lock Tap",
})

MiscSection:AddToggle("EnableTriggerBot", {
Text = "TriggerBot",
Default = false,
Tooltip = "Автоклик при наведении прицела на игрока",
}):AddKeyPicker("TriggerBotKeybind", {
Default = "None",
SyncToggleState = false,
Mode = "Hold",
Text = "Trigger Bot Key",
NoUI = false,
})

MiscSection:AddDropdown("TriggerBotMode", {
Text = "Срабатывание",
Default = "Always",
Values = { "Always", "Hold", "Toggle" },
Multi = false,
Tooltip = "Всегда — работает постоянно; Зажатие — пока держишь бинд; Переключатель — вкл/выкл по нажатию бинда",
})

MiscSection:AddSlider("TriggerBotDistance", {
Text = "Дистанция TriggerBot",
Default = 30,
Min = 10,
Max = 30,
Rounding = 0,
Compact = false,
Tooltip = "Дальность захвата. Максимум в игре — 30 метров (с геймпассом).",
})

local function getNearestPlayer(maxDist)
local nearest = nil
local minDist = maxDist
local cam = workspace.CurrentCamera
if not cam then return nil end
local myPos = cam.CFrame.Position
for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
local root = plr.Character.HumanoidRootPart
local dist = (root.Position - myPos).Magnitude
if dist < minDist then
minDist = dist
nearest = plr
end
end
end
return nearest
end

UserInputService.InputBegan:Connect(function(input, gpe)
if gpe then return end
if not (Toggles.EnableLockTap and Toggles.EnableLockTap.Value) then return end
local boundKey = Options.LockTapKeybind.Value
local isMatch = false

if input.KeyCode.Name == boundKey or input.UserInputType.Name == boundKey then
isMatch = true
end
if boundKey == "MB4" and input.KeyCode and input.KeyCode.Name == "ButtonX1" then isMatch = true end
if boundKey == "MB5" and input.KeyCode and input.KeyCode.Name == "ButtonX2" then isMatch = true end

if boundKey == "RightClick" or boundKey == "MB2" or boundKey == "MouseButton2" then
if input.UserInputType == Enum.UserInputType.MouseButton2 then isMatch = true end
end
if boundKey == "LeftClick" or boundKey == "MB1" or boundKey == "MouseButton1" then
if input.UserInputType == Enum.UserInputType.MouseButton1 then isMatch = true end
end

if isMatch then
local target = getNearestPlayer(Options.LockDistance.Value)
if not target or not target.Character then return end
local root = target.Character:FindFirstChild("HumanoidRootPart")
if not root then return end
local cam = workspace.CurrentCamera
if not cam then return end
local aimCF = CFrame.new(cam.CFrame.Position, root.Position)
cam.CFrame = aimCF
pcall(function() VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 1) end)
pcall(function() VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 1) end)
end
end)

Triggerbot = {
Enabled = false,
Connection = nil,
canGrab = true,
maxDistance = 20,
preGrabDelay = 0.00001,
postGrabDelay = 0.05,
lastTarget = nil,
lastHitTime = 0,
targetMemoryDuration = 0.1,
checkThrottle = 0.008,
lastCheck = 0,
}
tbRayParams = RaycastParams.new()
tbRayParams.FilterType = Enum.RaycastFilterType.Exclude

tbHeld = false
tbToggled = false

local function tbKeyMatches(input, boundKey)
if boundKey == nil then return false end
if input.KeyCode.Name == boundKey or input.UserInputType.Name == boundKey then return true end
if (boundKey == "MB2" or boundKey == "MouseButton2" or boundKey == "RightClick") and input.UserInputType == Enum.UserInputType.MouseButton2 then return true end
if (boundKey == "MB1" or boundKey == "MouseButton1" or boundKey == "LeftClick") and input.UserInputType == Enum.UserInputType.MouseButton1 then return true end
if boundKey == "MB4" and input.KeyCode and input.KeyCode.Name == "ButtonX1" then return true end
if boundKey == "MB5" and input.KeyCode and input.KeyCode.Name == "ButtonX2" then return true end
return false
end

UserInputService.InputBegan:Connect(function(input, gpe)
if gpe then return end
if not (Toggles.EnableTriggerBot and Toggles.EnableTriggerBot.Value) then return end
local boundKey = Options.TriggerBotKeybind and Options.TriggerBotKeybind.Value
if tbKeyMatches(input, boundKey) then
tbHeld = true
tbToggled = not tbToggled
if Options.TriggerBotMode and Options.TriggerBotMode.Value == "Toggle" then
Library:Notify("Trigger Bot: " .. (tbToggled and "ВКЛ" or "ВЫКЛ"), 1)
end
end
end)

UserInputService.InputEnded:Connect(function(input)
local boundKey = Options.TriggerBotKeybind and Options.TriggerBotKeybind.Value
if tbKeyMatches(input, boundKey) then
tbHeld = false
end
end)

function Triggerbot:GetTarget()
local c = LocalPlayer.Character
if not c or not c:FindFirstChild("HumanoidRootPart") then
return
end
if workspace:FindFirstChild("GrabParts") then
return
end
local cam = workspace.CurrentCamera
if not cam then return end
local origin, dir = cam.CFrame.Position, cam.CFrame.LookVector
tbRayParams.FilterDescendantsInstances = { c, workspace.Terrain }
local result = workspace:Raycast(origin, dir * 1000, tbRayParams)
if not result then
local dirs = {
dir,
(dir + Vector3.new(0, 0.075, 0)).Unit,
(dir - Vector3.new(0, 0.075, 0)).Unit,
}
for _, d in ipairs(dirs) do
result = workspace:Raycast(origin, d * 1000, tbRayParams)
if result then break end
end
end
if not result then return end
local model = result.Instance:FindFirstAncestorOfClass("Model")
if not model or model == c then return end
local hum = model:FindFirstChildOfClass("Humanoid")
if not hum or hum.Health <= 0 then return end
local root = model:FindFirstChild("HumanoidRootPart")
if not root then return end
local dist = (c.HumanoidRootPart.Position - root.Position).Magnitude
local maxDist = Options.TriggerBotDistance and Options.TriggerBotDistance.Value or 20
if dist > maxDist then return end
return model
end

function Triggerbot:OnHeartbeat()
if not self.Enabled or not self.canGrab then return end

local mode = Options.TriggerBotMode and Options.TriggerBotMode.Value or "Always"
local active = false
if mode == "Always" then
active = true
elseif mode == "Hold" then
active = tbHeld
elseif mode == "Toggle" then
active = tbToggled
end
if not active then return end

if UserInputService:GetFocusedTextBox() then return end
if tick() - self.lastCheck < self.checkThrottle then return end
self.lastCheck = tick()
local t = self:GetTarget()
if t then
self.lastTarget = t
self.lastHitTime = tick()
elseif self.lastTarget and tick() - self.lastHitTime > self.targetMemoryDuration then
self.lastTarget = nil
end
local c = LocalPlayer.Character
local root = self.lastTarget and self.lastTarget:FindFirstChild("HumanoidRootPart")
if not (self.lastTarget and c and c:FindFirstChild("HumanoidRootPart") and root) then
return
end

local maxDist = Options.TriggerBotDistance and Options.TriggerBotDistance.Value or 20
if (c.HumanoidRootPart.Position - root.Position).Magnitude > maxDist then
self.lastTarget = nil
return
end
self.canGrab = false
task.spawn(function()
task.wait(self.preGrabDelay)
VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 1)
task.wait(0.01)
VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 1)
local t0 = tick()
repeat
task.wait(0.02)
until not workspace:FindFirstChild("GrabParts") or tick() - t0 > 1.6
task.wait(self.postGrabDelay)
self.canGrab = true
self.lastTarget = nil
end)
end

Toggles.EnableTriggerBot:OnChanged(function()
Triggerbot.Enabled = Toggles.EnableTriggerBot.Value
if Triggerbot.Enabled and not Triggerbot.Connection then
Library:Notify(L("TriggerBot ВКЛЮЧЁН!", "TriggerBot ENABLED!"), 3)
Triggerbot.Connection = RunService.Heartbeat:Connect(function()
Triggerbot:OnHeartbeat()
end)
elseif not Triggerbot.Enabled and Triggerbot.Connection then
Triggerbot.Connection:Disconnect()
Triggerbot.Connection = nil
end
end)

local AimAssistSection = Tabs.Misc:AddLeftGroupbox("Aim Assist", "target")

AimAssistSection:AddToggle("EnableAimAssist", {
Text = "Aim Assist",
Default = false,
Tooltip = L("Плавно доводит камеру на ближайшего к прицелу врага", "Smoothly moves camera to nearest enemy to crosshair"),
}):AddKeyPicker("AimAssistKeybind", {
Default = "None",
SyncToggleState = false,
Mode = "Hold",
Text = "Aim Assist Key",
NoUI = false,
})

AimAssistSection:AddDropdown("AimAssistMode", {
Text = L("Срабатывание", "Trigger Mode"),
Default = "Hold",
Values = { "Always", "Hold�", "Toggle" },
Multi = false,
Tooltip = L("Всегда — работает постоянно; Зажатие — пока держишь бинд; Переключатель — вкл/выкл по нажатию бинда", "Always — always on; Hold — while holding bind; Toggle — on/off on bind press"),
})

AimAssistSection:AddDropdown("AimAssistPart", {
Text = L("Куда наводить", "Aim Part"),
Default = "Head",
Values = { "Head", "Torso", "Center" },
Multi = false,
Tooltip = L("Часть тела, в которую целится помощник", "Body part the helper aims at"),
})

AimAssistSection:AddSlider("AimAssistSmooth", {
Text = L("Плавность", "Smoothness"),
Default = 10,
Min = 1,
Max = 30,
Rounding = 0,
Compact = false,
Tooltip = "Больше — пла��нее и медленнее; 1 — почти мгновенно",
})

AimAssistSection:AddSlider("AimAssistFOV", {
Text = L("FOV (радиус)", "FOV (radius)"),
Default = 120,
Min = 20,
Max = 600,
Rounding = 0,
Compact = false,
Tooltip = L("Берёт цель только в этом радиусе от центра экрана (в пикселях)", "Only targets within this radius from screen center (in pixels)"),
})

AimAssistSection:AddSlider("AimAssistDistance", {
Text = L("Дальность", "Range"),
Default = 150,
Min = 10,
Max = 1000,
Rounding = 0,
Compact = false,
Tooltip = L("Максимальная 3D-дистанция до цели, дальше которой помощник не срабатывает", "Max 3D distance to target, beyond which helper won't activate"),
})

aimAssistHeld = false
aimAssistToggled = false

local function aimAssistKeyMatches(input, boundKey)
if boundKey == nil then return false end
if input.KeyCode.Name == boundKey or input.UserInputType.Name == boundKey then
return true
end
if (boundKey == "MB2" or boundKey == "MouseButton2" or boundKey == "RightClick")
and input.UserInputType == Enum.UserInputType.MouseButton2 then
return true
end
if (boundKey == "MB1" or boundKey == "MouseButton1" or boundKey == "LeftClick")
and input.UserInputType == Enum.UserInputType.MouseButton1 then
return true
end
if boundKey == "MB4" and input.KeyCode and input.KeyCode.Name == "ButtonX1" then return true end
if boundKey == "MB5" and input.KeyCode and input.KeyCode.Name == "ButtonX2" then return true end
return false
end

UserInputService.InputBegan:Connect(function(input, gpe)
if gpe then return end
if not (Toggles.EnableAimAssist and Toggles.EnableAimAssist.Value) then return end
local boundKey = Options.AimAssistKeybind and Options.AimAssistKeybind.Value
if aimAssistKeyMatches(input, boundKey) then
aimAssistHeld = true
aimAssistToggled = not aimAssistToggled
if Options.AimAssistMode and Options.AimAssistMode.Value == "Toggle" then
Library:Notify("Aim Assist: " .. (aimAssistToggled and L("ВКЛ", "ON") or "��ЫКЛ"), 1)
end
end
end)

UserInputService.InputEnded:Connect(function(input)
local boundKey = Options.AimAssistKeybind and Options.AimAssistKeybind.Value
if aimAssistKeyMatches(input, boundKey) then
aimAssistHeld = false
end
end)

Toggles.EnableAimAssist:OnChanged(function()
if not Toggles.EnableAimAssist.Value then
aimAssistToggled = false
aimAssistHeld = false
end
end)

local function getAimAssistPartName()
local v = Options.AimAssistPart and Options.AimAssistPart.Value or "Head"
if v == "Torso" then return "Torso" end
if v == "Center" then return "HumanoidRootPart" end
return "Head"
end

local function getAimAssistTarget()
local cam = workspace.CurrentCamera
if not cam then return nil end
local center = Vector2.new(cam.ViewportSize.X / 2, cam.ViewportSize.Y / 2)
local fov = Options.AimAssistFOV and Options.AimAssistFOV.Value or 120
local maxDist = Options.AimAssistDistance and Options.AimAssistDistance.Value or 150
local lpChar = LocalPlayer.Character
local lpRoot = lpChar and lpChar:FindFirstChild("HumanoidRootPart")
local partName = getAimAssistPartName()
local best, bestDist = nil, fov
for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer and plr.Character then
local part = plr.Character:FindFirstChild(partName)
or plr.Character:FindFirstChild("HumanoidRootPart")
or plr.Character:FindFirstChild("Torso")
or plr.Character:FindFirstChild("Head")
local hum = plr.Character:FindFirstChildOfClass("Humanoid")
if part and hum and hum.Health > 0 then
local within = true
if lpRoot then
within = (part.Position - lpRoot.Position).Magnitude <= maxDist
end
if within then
local sp, onScreen = cam:WorldToViewportPoint(part.Position)
if onScreen then
local d = (Vector2.new(sp.X, sp.Y) - center).Magnitude
if d < bestDist then
bestDist = d
best = part
end
end
end
end
end
end
return best
end

RunService.RenderStepped:Connect(function()
if not (Toggles.EnableAimAssist and Toggles.EnableAimAssist.Value) then return end
local mode = Options.AimAssistMode and Options.AimAssistMode.Value or "Hold"
local active = false
if mode == "Always" then
active = true
elseif mode == "Hold" then
active = aimAssistHeld
elseif mode == "Toggle" then
active = aimAssistToggled
end
if not active then return end
local target = getAimAssistTarget()
if not target then return end
local cam = workspace.CurrentCamera
if not cam then return end
local smooth = Options.AimAssistSmooth and Options.AimAssistSmooth.Value or 10
if smooth < 1 then smooth = 1 end
local goal = CFrame.new(cam.CFrame.Position, target.Position)
local alpha = math.clamp(1 / smooth, 0.02, 1)
cam.CFrame = cam.CFrame:Lerp(goal, alpha)
end)

local SpawnPalletSection = Tabs.Misc:AddRightGroupbox("Toy Spawner", "package-plus")

SpawnPalletSection:AddToggle("EnableSpawnPallet", {
Text = L("Спавн палетки (бинд)", "Pallet Spawn (bind)"),
Default = false,
Tooltip = L("Когда включено — по бинду спавнит палетку перед собой", "When enabled — spawns pallet in front on bind"),
}):AddKeyPicker("SpawnPalletKeybind", {
Default = "None",
SyncToggleState = false,
Mode = "Hold",
Text = "Spawn Pallet Key",
NoUI = false,
})

local function spawnPalletKeyMatches(input, boundKey)
if boundKey == nil then return false end
if input.KeyCode.Name == boundKey or input.UserInputType.Name == boundKey then
return true
end
if (boundKey == "MB2" or boundKey == "MouseButton2" or boundKey == "RightClick")
and input.UserInputType == Enum.UserInputType.MouseButton2 then
return true
end
if (boundKey == "MB1" or boundKey == "MouseButton1" or boundKey == "LeftClick")
and input.UserInputType == Enum.UserInputType.MouseButton1 then
return true
end
if boundKey == "MB4" and input.KeyCode and input.KeyCode.Name == "ButtonX1" then return true end
if boundKey == "MB5" and input.KeyCode and input.KeyCode.Name == "ButtonX2" then return true end
return false
end

lastPalletSpawn = 0
local function doSpawnPallet()
if tick() - lastPalletSpawn < 1 then return end
lastPalletSpawn = tick()

local char = LocalPlayer.Character
if not char then return end
local camPart = char:FindFirstChild("CamPart")
if not camPart then return end
local RS = game:GetService("ReplicatedStorage")
local menuToys = RS:FindFirstChild("MenuToys")
if not menuToys then return end
local spawnRemote = menuToys:FindFirstChild("SpawnToyRemoteFunction")
if not spawnRemote then return end
local canSpawn = LocalPlayer:FindFirstChild("CanSpawnToy")
if canSpawn and not canSpawn.Value then
local t0 = tick()
while canSpawn and not canSpawn.Value do
if tick() - t0 > 3 then return end
task.wait(0.1)
end
end
task.spawn(function()
pcall(function()
spawnRemote:InvokeServer("PalletLightBrown", camPart.CFrame, Vector3.new(0, camPart.Orientation.Y, 0))
end)
end)
end

UserInputService.InputBegan:Connect(function(input, gpe)
if gpe then return end
if not (Toggles.EnableSpawnPallet and Toggles.EnableSpawnPallet.Value) then return end
local boundKey = Options.SpawnPalletKeybind and Options.SpawnPalletKeybind.Value
if spawnPalletKeyMatches(input, boundKey) then
doSpawnPallet()
end
end)

do
local RocketSpawnerSection = Tabs.Misc:AddRightGroupbox("Rocket Spawner", "rocket")

RocketSpawnerSection:AddToggle("EnableRocketSpawner", {
Text = L("Включить Rocket Spawner", "Enable Rocket Spawner"),
Default = false,
Tooltip = L("Разрешает спавнить ракеты по бинду", "Allows spawning rockets on bind"),
}):AddKeyPicker("RocketSpawnKeybind", {
Default = "None",
SyncToggleState = false,
Mode = "Hold",
Text = L("Кнопка спавна ракеты", "Rocket Spawn Button"),
NoUI = false,
})

RocketSpawnerSection:AddDropdown("RocketSpawnMode", {
Text = L("Режим спавна", "Spawn Mode"),
Default = "Spawn Only",
Values = {"Spawn Only", "Spawn + Shoot"},
Tooltip = L("Выстрел запустит ракету туда, куда ты смотришь (через прицел)", "Fire launches rocket where you aim (via crosshair)"),
})

RocketSpawnerSection:AddSlider("RocketShootSpeed", {
Text = L("Скорость выстрела", "Fire Rate"),
Default = 200,
Min = 50,
Max = 1000,
Rounding = 0,
})

do
local function getShootCFrame()
local cam = workspace.CurrentCamera
local mousePos = UserInputService:GetMouseLocation()
local ray = cam:ViewportPointToRay(mousePos.X, mousePos.Y)
return CFrame.lookAt(cam.CFrame.Position, cam.CFrame.Position + ray.Direction * 1000)
end

UserInputService.InputBegan:Connect(function(input, gpe)
if gpe then return end
if not (Toggles.EnableRocketSpawner and Toggles.EnableRocketSpawner.Value) then return end
local boundKey = Options.RocketSpawnKeybind and Options.RocketSpawnKeybind.Value

local isMatch = false
if typeof(boundKey) == "EnumItem" then
if input.KeyCode == boundKey or input.UserInputType == boundKey then isMatch = true end
elseif type(boundKey) == "string" then
if input.KeyCode.Name == boundKey or input.UserInputType.Name == boundKey then isMatch = true end
if boundKey == "MB4" and input.KeyCode and input.KeyCode.Name == "ButtonX1" then isMatch = true end
if boundKey == "MB5" and input.KeyCode and input.KeyCode.Name == "ButtonX2" then isMatch = true end
end

if isMatch then
local char = LocalPlayer.Character
local hrp = char and char:FindFirstChild("HumanoidRootPart")
if not hrp then return end

local mode = Options.RocketSpawnMode.Value
local spawnName = "BombMissile"

local RS = game:GetService("ReplicatedStorage")
local spawnRemote = RS:FindFirstChild("MenuToys") and RS.MenuToys:FindFirstChild("SpawnToyRemoteFunction")
local canSpawn = LocalPlayer:FindFirstChild("CanSpawnToy")

if not spawnRemote then return end

if canSpawn and canSpawn.Value == false then
Library:Notify(L("Подожди перезарядки спавна игрушек!", "Wait for toy spawn cooldown!"), 2)
return
end

local cam = workspace.CurrentCamera
local mousePos = UserInputService:GetMouseLocation()
local ray = cam:ViewportPointToRay(mousePos.X, mousePos.Y)
local rayParams = RaycastParams.new()
rayParams.FilterType = Enum.RaycastFilterType.Exclude
rayParams.FilterDescendantsInstances = {LocalPlayer.Character}
local result = workspace:Raycast(ray.Origin, ray.Direction * 500, rayParams)

local hitPos = result and result.Position or (cam.CFrame.Position + ray.Direction * 100)
local lookCF = CFrame.lookAt(cam.CFrame.Position, hitPos)

if mode == "Spawn Only" then
task.spawn(function()
local shootPos = cam.CFrame.Position + lookCF.LookVector * 5
local shootCF = CFrame.new(shootPos, shootPos + lookCF.LookVector)
pcall(function() spawnRemote:InvokeServer(spawnName, shootCF, Vector3.zero) end)
end)
elseif mode == "Spawn + Shoot" then
task.spawn(function()
local folder = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
if not folder then return end

local rocket = nil
local conn = folder.ChildAdded:Connect(function(child)
if child.Name == spawnName then
rocket = child
end
end)

local spawnCF = hrp.CFrame * CFrame.new(0, 14, 20)
pcall(function() spawnRemote:InvokeServer(spawnName, spawnCF, Vector3.zero) end)

local t0 = tick()
while not rocket and tick() - t0 < 3 do task.wait() end
conn:Disconnect()

if not rocket then return end

local primary = rocket.PrimaryPart or rocket:FindFirstChildWhichIsA("BasePart")
if not primary then
while not primary and tick() - t0 < 4 do
task.wait()
primary = rocket.PrimaryPart or rocket:FindFirstChildWhichIsA("BasePart")
end
end
if not primary then return end

local partToGrab = nil
for _, v in ipairs(rocket:GetChildren()) do
pcall(function()
if v:IsA("BasePart") and v.CanQuery and v.CanTouch then
partToGrab = v
end
end)
if partToGrab then break end
end
if not partToGrab then partToGrab = primary end

local setOwner = RS:FindFirstChild("GrabEvents") and RS.GrabEvents:FindFirstChild("SetNetworkOwner")
if setOwner then pcall(function() setOwner:FireServer(partToGrab, partToGrab.CFrame) end) end

task.wait(0.2)

local camCF = workspace.CurrentCamera.CFrame
local shootPos = camCF.Position + lookCF.LookVector * 8
primary.CFrame = CFrame.new(shootPos, shootPos + lookCF.LookVector)

local speed = Options.RocketShootSpeed and Options.RocketShootSpeed.Value or 200
primary.AssemblyLinearVelocity = lookCF.LookVector * speed

local bv = Instance.new("BodyVelocity")
bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bv.Velocity = lookCF.LookVector * speed
bv.Parent = primary

local bg = Instance.new("BodyGyro")
bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
bg.CFrame = CFrame.new(shootPos, shootPos + lookCF.LookVector)
bg.Parent = primary

game:GetService("Debris"):AddItem(rocket, 5)
end)
end
end
end)
end

end
local GamepassSection = Tabs.Misc:AddLeftGroupbox("Gamepass", "ticket")

local LimbRemovalSection = Tabs.Misc:AddRightGroupbox("Limb Removal (binds)", "scissors")

LimbRemovalSection:AddLabel("Требует: grab + ragdoll цели", false)

LimbRemovalSection:AddLabel("Remove Left Leg", false):AddKeyPicker("RemoveLeftLeg", {
Default = "None",
SyncToggleState = false,
Mode = "Press",
Text = "Remove Left Leg",
NoUI = false,
Callback = function()
if not (workspace:FindFirstChild("GrabParts") and workspace.GrabParts:FindFirstChild("GrabPart")) then
Library:Notify(L("Limb Removal: сначала захвати цель", "Limb Removal: grab target first"), 2)
return
end
local target = workspace.GrabParts.GrabPart:FindFirstChild("WeldConstraint")
target = target and target.Part1 and target.Part1.Parent
if not target then return end
local leg = target:FindFirstChild("Left Leg")
local hum = target:FindFirstChildOfClass("Humanoid")
if not (leg and hum and hum:FindFirstChild("Ragdolled")) then
Library:Notify(L("Limb Removal: нет Left Leg или Humanoid", "Limb Removal: no Left Leg or Humanoid"), 2)
return
end
if not hum.Ragdolled.Value then
Library:Notify(L("Limb Removal: цель не рагдоллнута", "Limb Removal: target not ragdolled"), 2)
return
end
local pos = target:FindFirstChild("Torso") and target.Torso.CFrame
if not pos then return end
workspace.FallenPartsDestroyHeight = -100
leg.CFrame = CFrame.new(0, -1000, 0)
task.wait(0.1)
target.Torso.CFrame = CFrame.new(0, -950, 0)
task.wait(0)
target.Torso.CFrame = pos
Library:Notify(L("Limb Removal: Left Leg удалена", "Limb Removal: Left Leg removed"), 1)
end,
})

LimbRemovalSection:AddLabel("Remove Right Leg", false):AddKeyPicker("RemoveRightLeg", {
Default = "None",
SyncToggleState = false,
Mode = "Press",
Text = "Remove Right Leg",
NoUI = false,
Callback = function()
if not (workspace:FindFirstChild("GrabParts") and workspace.GrabParts:FindFirstChild("GrabPart")) then
Library:Notify(L("Limb Removal: сначала захвати цель", "Limb Removal: grab target first"), 2)
return
end
local target = workspace.GrabParts.GrabPart:FindFirstChild("WeldConstraint")
target = target and target.Part1 and target.Part1.Parent
if not target then return end
local leg = target:FindFirstChild("Right Leg")
local hum = target:FindFirstChildOfClass("Humanoid")
if not (leg and hum and hum:FindFirstChild("Ragdolled")) then
Library:Notify(L("Limb Removal: нет Right Leg или Humanoid", "Limb Removal: no Right Leg or Humanoid"), 2)
return
end
if not hum.Ragdolled.Value then
Library:Notify(L("Limb Removal: цель не рагдоллнута", "Limb Removal: target not ragdolled"), 2)
return
end
local pos = target:FindFirstChild("Torso") and target.Torso.CFrame
if not pos then return end
workspace.FallenPartsDestroyHeight = -100
leg.CFrame = CFrame.new(0, -1000, 0)
task.wait(0.1)
target.Torso.CFrame = CFrame.new(0, -950, 0)
task.wait(0)
target.Torso.CFrame = pos
Library:Notify(L("Limb Removal: Right Leg удалена", "Limb Removal: Right Leg removed"), 1)
end,
})

LimbRemovalSection:AddLabel("Remove Left Arm", false):AddKeyPicker("RemoveLeftArm", {
Default = "None",
SyncToggleState = false,
Mode = "Press",
Text = "Remove Left Arm",
NoUI = false,
Callback = function()
if not (workspace:FindFirstChild("GrabParts") and workspace.GrabParts:FindFirstChild("GrabPart")) then
Library:Notify(L("Limb Removal: сначала захвати цель", "Limb Removal: grab target first"), 2)
return
end
local target = workspace.GrabParts.GrabPart:FindFirstChild("WeldConstraint")
target = target and target.Part1 and target.Part1.Parent
if not target then return end
local arm = target:FindFirstChild("Left Arm")
local hum = target:FindFirstChildOfClass("Humanoid")
if not (arm and hum and hum:FindFirstChild("Ragdolled")) then
Library:Notify(L("Limb Removal: нет Left Arm или Humanoid", "Limb Removal: no Left Arm or Humanoid"), 2)
return
end
if not hum.Ragdolled.Value then
Library:Notify(L("Limb Removal: цель не рагдоллнута", "Limb Removal: target not ragdolled"), 2)
return
end
local pos = target:FindFirstChild("Torso") and target.Torso.CFrame
if not pos then return end
workspace.FallenPartsDestroyHeight = -100
arm.CFrame = CFrame.new(0, -1000, 0)
task.wait(0.1)
target.Torso.CFrame = CFrame.new(0, -950, 0)
task.wait(0)
target.Torso.CFrame = pos
Library:Notify(L("Limb Removal: Left Arm удалена", "Limb Removal: Left Arm removed"), 1)
end,
})

LimbRemovalSection:AddLabel("Remove Right Arm", false):AddKeyPicker("RemoveRightArm", {
Default = "None",
SyncToggleState = false,
Mode = "Press",
Text = "Remove Right Arm",
NoUI = false,
Callback = function()
if not (workspace:FindFirstChild("GrabParts") and workspace.GrabParts:FindFirstChild("GrabPart")) then
Library:Notify(L("Limb Removal: сначала захвати цель", "Limb Removal: grab target first"), 2)
return
end
local target = workspace.GrabParts.GrabPart:FindFirstChild("WeldConstraint")
target = target and target.Part1 and target.Part1.Parent
if not target then return end
local arm = target:FindFirstChild("Right Arm")
local hum = target:FindFirstChildOfClass("Humanoid")
if not (arm and hum and hum:FindFirstChild("Ragdolled")) then
Library:Notify(L("Limb Removal: нет Right Arm или Humanoid", "Limb Removal: no Right Arm or Humanoid"), 2)
return
end
if not hum.Ragdolled.Value then
Library:Notify(L("Limb Removal: цель не рагдоллнута", "Limb Removal: target not ragdolled"), 2)
return
end
local pos = target:FindFirstChild("Torso") and target.Torso.CFrame
if not pos then return end
workspace.FallenPartsDestroyHeight = -100
arm.CFrame = CFrame.new(0, -1000, 0)
task.wait(0.1)
target.Torso.CFrame = CFrame.new(0, -950, 0)
task.wait(0)
target.Torso.CFrame = pos
Library:Notify(L("Limb Removal: Right Arm удалена", "Limb Removal: Right Arm removed"), 1)
end,
})

GamepassSection:AddToggle("EnableFurtherReach", {
Text = "Further Reach (Gamepass)",
Default = false,
Tooltip = "Геймпасс на дальнюю линию захвата. Увеличивает дальность граба.",
})

GamepassSection:AddToggle("EnableAutoEscape", {
Text = "Auto Escape (Break Free)",
Default = false,
Tooltip = L("Автоматически вырывается из чужого захвата. С геймпассом — 3 сек, без — 12 сек.", "Auto-escapes from foreign grabs. With gamepass — 3 sec, without — 12 sec."),
})

furtherReachActive = false
furtherReachCharConn = nil

local gamepassRS = game:GetService("ReplicatedStorage")
local gamepassRF = game:GetService("ReplicatedFirst")
local gamepassEvents = gamepassRS:FindFirstChild("GamepassEvents")
local menuToys = gamepassRS:FindFirstChild("MenuToys")
local gamepassScriptNotify = gamepassEvents and gamepassEvents:FindFirstChild("FurtherReachBoughtNotifier")
local gamepassActivator = menuToys and menuToys:FindFirstChild("LimitedTimeToyEvent")

local function furtherReachEnable()
local char = LocalPlayer.Character
if not char then return end

local existing = LocalPlayer:FindFirstChild("FartherReach")
if existing then existing:Destroy() end
local fr = Instance.new("BoolValue")
fr.Name = "FartherReach"
fr.Value = true
fr.Parent = LocalPlayer

if gamepassScriptNotify then
gamepassScriptNotify.Parent = gamepassRF
end

if gamepassActivator then
gamepassActivator.Parent = gamepassEvents
gamepassActivator.Name = "FurtherReachBoughtNotifier"
end

pcall(function()
local gs = char:FindFirstChild("GrabbingScript")
if gs then gs.Enabled = false; gs.Enabled = true end
end)

task.delay(0.1, function()
if gamepassActivator then pcall(function() gamepassActivator:FireServer() end) end
end)

furtherReachActive = true
end

local function furtherReachDisable()
local existing = LocalPlayer:FindFirstChild("FartherReach")
if existing then existing:Destroy() end

if gamepassScriptNotify then
gamepassScriptNotify.Parent = gamepassEvents
end

if gamepassActivator then
gamepassActivator.Name = "LimitedTimeToyEvent"
gamepassActivator.Parent = menuToys
end

local char = LocalPlayer.Character
if char then
pcall(function()
local gs = char:FindFirstChild("GrabbingScript")
if gs then gs.Enabled = false; gs.Enabled = true end
end)
end

furtherReachActive = false
end

Toggles.EnableFurtherReach:OnChanged(function()
if Toggles.EnableFurtherReach.Value then
furtherReachEnable()
furtherReachCharConn = LocalPlayer.CharacterAdded:Connect(function(newChar)
newChar:WaitForChild("GrabbingScript", 5)
task.wait(0.1)
if Toggles.EnableFurtherReach and Toggles.EnableFurtherReach.Value then
furtherReachEnable()
end
end)
else
furtherReachDisable()
if furtherReachCharConn then
furtherReachCharConn:Disconnect()
furtherReachCharConn = nil
end
end
end)

autoEscapeRunning = false

local function autoEscapeStart()
if autoEscapeRunning then return end
autoEscapeRunning = true

task.spawn(function()
local holdTimer = 0
while Toggles.EnableAutoEscape and Toggles.EnableAutoEscape.Value do
local char = LocalPlayer.Character
if char then
local head = char:FindFirstChild("Head")
local po = head and head:FindFirstChild("PartOwner")
local isHeld = LocalPlayer:FindFirstChild("IsHeld")
local beingHeld = (po ~= nil) or (isHeld and isHeld.Value)

if beingHeld then
holdTimer = holdTimer + 0.1
local escapeTime = (Toggles.EnableFurtherReach and Toggles.EnableFurtherReach.Value) and 3 or 12

if holdTimer >= escapeTime then
local RS = game:GetService("ReplicatedStorage")
local ge = RS:FindFirstChild("GrabEvents")
if ge then
local destroyLine = ge:FindFirstChild("DestroyGrabLine")
if destroyLine then
local hrp = char:FindFirstChild("HumanoidRootPart")
if hrp then pcall(function() destroyLine:FireServer(hrp) end) end
if head then pcall(function() destroyLine:FireServer(head) end) end
end
end

local ce = RS:FindFirstChild("CharacterEvents")
if ce then
local struggle = ce:FindFirstChild("Struggle")
if struggle then
pcall(function() struggle:FireServer(LocalPlayer) end)
end
end

pcall(function()
local gs = char:FindFirstChild("GrabbingScript")
if gs then gs.Enabled = false; gs.Enabled = true end
end)

if po then
pcall(function() po:Destroy() end)
end

holdTimer = 0
Library:Notify(L("Auto Escape: вырвался из захвата!", "Auto Escape: escaped from grab!"), 2)
end
else
holdTimer = 0
end
else
holdTimer = 0
end
task.wait(0.1)
end
holdTimer = 0
autoEscapeRunning = false
end)
end

Toggles.EnableAutoEscape:OnChanged(function()
if Toggles.EnableAutoEscape.Value then
autoEscapeStart()
end
end)

local HitboxSection = Tabs.Misc:AddRightGroupbox("Hitboxes", "box")

hitboxData = {}
hitboxColor = Color3.fromRGB(255, 60, 60)

local hitboxOriginals
if getgenv then
hitboxOriginals = getgenv().OblivionHitboxOriginals
if not hitboxOriginals then
hitboxOriginals = setmetatable({}, { __mode = "k" })
getgenv().OblivionHitboxOriginals = hitboxOriginals
end
else
hitboxOriginals = setmetatable({}, { __mode = "k" })
end

HitboxSection:AddToggle("EnableHitbox", {
Text = L("Хитбоксы", "Hitboxes"),
Default = false,
Tooltip = L("Раздувает область попадания у игроков — легче захватывать; сквозь них можно проходить", "Enlarges player hitboxes — easier to grab; can walk through them"),
}):AddKeyPicker("HitboxKeybind", {
Default = "None",
SyncToggleState = true,
Mode = "Toggle",
Text = "Hitbox Key",
NoUI = false,
}):AddColorPicker("HitboxColor", {
Default = Color3.fromRGB(255, 60, 60),
Title = L("Цвет хитбокса", "Hitbox Color"),
Callback = function(Value)
hitboxColor = Value
for _, data in pairs(hitboxData) do
if data.box then
data.box.Color3 = Value
data.box.SurfaceColor3 = Value
end
end
end,
})

HitboxSection:AddToggle("ShowHitbox", {
Text = L("Показывать хитбокс", "Show Hitbox"),
Default = true,
Tooltip = L("Выкл — хитбокс станет невидимым (контур убирается)", "Off — hitbox becomes invisible (outline removed)"),
Callback = function(Value)
if not Value then
for _, data in pairs(hitboxData) do
if data.box then data.box.Adornee = nil end
end
end
end,
})

HitboxSection:AddSlider("HitboxTransparency", {
Text = L("Прозрачность боксов", "Box Transparency"),
Default = 0.65,
Min = 0,
Max = 1,
Rounding = 2,
Compact = false,
Tooltip = L("0 — сплошной бокс, 1 — полностью прозрачный", "0 = solid box, 1 = fully transparent"),
})

HitboxSection:AddSlider("HitboxSize", {
Text = L("Размер хитбоксов", "Hitbox Size"),
Default = 8,
Min = 3,
Max = 30,
Rounding = 0,
Compact = false,
Tooltip = L("Размер области попадания игроков (в studs)", "Player hitbox size (in studs)"),
})

local function getHitboxPart(char)
return char:FindFirstChild("Torso")
or char:FindFirstChild("UpperTorso")
or char:FindFirstChild("LowerTorso")
end

local function restoreHitbox(plr)
local data = hitboxData[plr]
if not data then return end
if data.part and data.part.Parent then
pcall(function()
data.part.Size = data.size
data.part.Massless = data.massless
data.part.Transparency = data.transparency
data.part.LocalTransparencyModifier = 0
data.part.CanCollide = data.cancollide
end)
end
if data.box then data.box:Destroy() end
hitboxData[plr] = nil
end

local function restoreAllHitboxes()
for plr in pairs(hitboxData) do
restoreHitbox(plr)
end
for part, orig in pairs(hitboxOriginals) do
if part and part.Parent then
pcall(function()
part.Size = orig.size
part.Massless = orig.massless
part.Transparency = orig.transparency
part.LocalTransparencyModifier = 0
part.CanCollide = orig.cancollide
end)
end
end
end

Toggles.EnableHitbox:OnChanged(function()
if not Toggles.EnableHitbox.Value then
restoreAllHitboxes()
end
end)

Players.PlayerRemoving:Connect(function(plr)
restoreHitbox(plr)
end)

RunService.Heartbeat:Connect(function()
if not (Toggles.EnableHitbox and Toggles.EnableHitbox.Value) then return end
local size = (Options.HitboxSize and Options.HitboxSize.Value) or 8
local sizeVec = Vector3.new(size, size, size)
local showBox = Toggles.ShowHitbox and Toggles.ShowHitbox.Value
local boxT = (Options.HitboxTransparency and Options.HitboxTransparency.Value) or 0.65
for _, plr in ipairs(Players:GetPlayers()) do
if plr ~= LocalPlayer and plr.Character then
local part = getHitboxPart(plr.Character)
local hum = plr.Character:FindFirstChildOfClass("Humanoid")
if part and hum and hum.Health > 0 then
local data = hitboxData[plr]
if not data or data.part ~= part then
if data and data.box then data.box:Destroy() end
local orig = hitboxOriginals[part]
if not orig then
orig = { size = part.Size, massless = part.Massless, transparency = part.Transparency, cancollide = part.CanCollide }
hitboxOriginals[part] = orig
end
data = { part = part, size = orig.size, massless = orig.massless, transparency = orig.transparency, cancollide = orig.cancollide }
local sb = Instance.new("SelectionBox")
sb.Adornee = part
sb.LineThickness = 0.2
sb.SurfaceTransparency = 0.65
sb.SurfaceColor3 = hitboxColor
sb.Color3 = hitboxColor
sb.Parent = workspace
data.box = sb
hitboxData[plr] = data
end
if not part.Massless then part.Massless = true end
if part.CanCollide then part.CanCollide = false end
if part.Transparency ~= data.transparency then part.Transparency = data.transparency end
part.LocalTransparencyModifier = 1
if part.Size ~= sizeVec then part.Size = sizeVec end
if data.box then
if showBox then
if data.box.Adornee ~= part then data.box.Adornee = part end
if not data.box.Parent then data.box.Parent = workspace end
data.box.Color3 = hitboxColor
data.box.SurfaceColor3 = hitboxColor
data.box.SurfaceTransparency = boxT
elseif data.box.Adornee ~= nil then
data.box.Adornee = nil
end
end
end
end
end
end)
do
local CurrentPlayer = LocalPlayer
local game_Workspace = workspace
local game_Players = Players
local game_UserInputService = UserInputService
local game_RunService = RunService
local game_Lighting = game:GetService("Lighting")
local game_TweenService = game:GetService("TweenService")

local GrabEventsFolder = ReplicatedStorage:WaitForChild("GrabEvents")
local SetOwnershipEvent = GrabEventsFolder:WaitForChild("SetNetworkOwner")
local DestroyLineEvent = GrabEventsFolder:WaitForChild("DestroyGrabLine")
local CreateLineEvent = GrabEventsFolder:FindFirstChild("CreateGrabLine", 3)

_G.ControllingCreature = nil
local ConnectionNoclip = nil
local NoclipEnabled = false
local RaycastParameters = RaycastParams.new()
RaycastParameters.FilterDescendantsInstances = { CurrentPlayer.Character }
RaycastParameters.FilterType = Enum.RaycastFilterType.Exclude

local function CreateLookAtCFrame(startPosition, endPosition)
local DirectionVector = (endPosition - startPosition).Unit
local RightAxis = DirectionVector:Cross(Vector3.new(0, 1, 0))
local UpAxis = RightAxis:Cross(DirectionVector)
return CFrame.fromMatrix(startPosition, RightAxis, UpAxis)
end

local function GetCurrentCharacter()
if CurrentPlayer.Character and CurrentPlayer.Character:FindFirstChild("HumanoidRootPart") and CurrentPlayer.Character:FindFirstChildOfClass("Humanoid") then
return CurrentPlayer.Character
end
end

local function ValidatePartOwnership(partInstance, shouldReturnOwner)
if typeof(partInstance) == "Instance" and partInstance:FindFirstChild("PartOwner") and partInstance.PartOwner.Value == CurrentPlayer.Name then
return not shouldReturnOwner and true or partInstance.PartOwner
end
end

local function RequestOwnershipSingle(partInstance)
local DistanceFromPlayer = CurrentPlayer:DistanceFromCharacter(partInstance.Position)
if CurrentPlayer.Character and CurrentPlayer.Character:FindFirstChild("HumanoidRootPart") then
if ValidatePartOwnership(partInstance) then return true end
if DistanceFromPlayer <= 30 then
pcall(function()
SetOwnershipEvent:FireServer(partInstance, CreateLookAtCFrame(CurrentPlayer.Character.HumanoidRootPart.Position, partInstance.Position))
end)
end
end
end

local function RequestOwnershipAndRemoveLine(partInstance)
local DistanceCalculated = CurrentPlayer:DistanceFromCharacter(partInstance.Position)
local IsConnected = partInstance:GetAttribute("Connected")
local HasCreatedConnection = partInstance:GetAttribute("CreatedConnected")
if CurrentPlayer.Character and CurrentPlayer.Character:FindFirstChild("HumanoidRootPart") then
if ValidatePartOwnership(partInstance) then
partInstance:SetAttribute("Connected", true)
pcall(function() DestroyLineEvent:FireServer(partInstance) end)
if not HasCreatedConnection then
partInstance:SetAttribute("CreatedConnected", true)
partInstance.ChildAdded:Connect(function(childAdded)
if childAdded.Name == "PartOwner" and childAdded.Value ~= CurrentPlayer.Name then
partInstance:SetAttribute("Connected", false)
end
end)
end
elseif DistanceCalculated <= 30 and not IsConnected then
pcall(function()
SetOwnershipEvent:FireServer(partInstance, CreateLookAtCFrame(CurrentPlayer.Character.HumanoidRootPart.Position, partInstance.Position))
end)
end
end
end

local function ActivateNoclipMode()
if not ConnectionNoclip then
NoclipEnabled = false
local function NoclipLoopFunction()
if NoclipEnabled == false and game_Players.LocalPlayer.Character ~= nil then
for _, child in ipairs(game_Players.LocalPlayer.Character:GetChildren()) do
if child:IsA("BasePart") and child.CanCollide then
child.CanCollide = false
end
end
end
task.wait(0.21)
end
ConnectionNoclip = game_RunService.Stepped:Connect(NoclipLoopFunction)
end
end

local function DeactivateNoclipMode()
if not _G.NoclipToggle then
if ConnectionNoclip then
ConnectionNoclip:Disconnect()
ConnectionNoclip = nil
end
NoclipEnabled = true
end
end

local function DisableCharacterQuery(characterModel)
for _, part in ipairs(characterModel:GetChildren()) do
if part:IsA("Part") then part.CanQuery = false end
end
end

local function EnableCharacterQuery(characterModel)
for _, part in ipairs(characterModel:GetChildren()) do
if part:IsA("Part") then part.CanQuery = true end
end
end

local OblivionControl = {}
OblivionControl.ColorCorrection = Instance.new("ColorCorrectionEffect")
OblivionControl.ColorCorrection.Name = "OblivionControlCC"
OblivionControl.ColorCorrection.Parent = game_Lighting
OblivionControl.ColorCorrection.Enabled = false
OblivionControl.ColorCorrection.TintColor = Color3.fromRGB(255, 255, 255)
OblivionControl.ColorCorrection.Brightness = 0
OblivionControl.ColorCorrection.Contrast = 0
OblivionControl.ColorCorrection.Saturation = 0

OblivionControl.Bloom = Instance.new("BloomEffect")
OblivionControl.Bloom.Name = "OblivionControlBloom"
OblivionControl.Bloom.Parent = game_Lighting
OblivionControl.Bloom.Enabled = false
OblivionControl.Bloom.Intensity = 0
OblivionControl.Bloom.Size = 0
OblivionControl.Bloom.Threshold = 0.95

OblivionControl.SunRays = Instance.new("SunRaysEffect")
OblivionControl.SunRays.Name = "OblivionControlSunRays"
OblivionControl.SunRays.Parent = game_Lighting
OblivionControl.SunRays.Enabled = false
OblivionControl.SunRays.Intensity = 0
OblivionControl.SunRays.Spread = 0

OblivionControl.WarpSound = Instance.new("Sound")
OblivionControl.WarpSound.SoundId = "rbxassetid://1846275076"  
OblivionControl.WarpSound.Volume = 0.6
OblivionControl.WarpSound.PlaybackSpeed = 0.85
OblivionControl.WarpSound.Parent = game_Workspace

OblivionControl.ExitSound = Instance.new("Sound")
OblivionControl.ExitSound.SoundId = "rbxassetid://3068220893"  
OblivionControl.ExitSound.Volume = 0.4
OblivionControl.ExitSound.PlaybackSpeed = 1.1
OblivionControl.ExitSound.Parent = game_Workspace

OblivionControl.Active = false
OblivionControl.TweenConn = nil
OblivionControl.Highlight = nil
OblivionControl.SelectionBox = nil

local function startControlVisuals(targetModel)
OblivionControl.Active = true

pcall(function() OblivionControl.WarpSound:Play() end)

local cam = game_Workspace.CurrentCamera
if cam then
pcall(function()
cam.FieldOfView = 45
local tween = game_TweenService:Create(
cam,
TweenInfo.new(0.5, Enum.EasingStyle.Back, Enum.EasingDirection.Out, 0, true),
{FieldOfView = 65}
)
tween:Play()
end)
end

OblivionControl.ColorCorrection.Enabled = true
pcall(function()
OblivionControl.ColorCorrection.TintColor = Color3.fromRGB(180, 100, 255)
OblivionControl.ColorCorrection.Saturation = 0.8
OblivionControl.ColorCorrection.Contrast = 0.3
OblivionControl.ColorCorrection.Brightness = -0.05

local ccTween = game_TweenService:Create(
OblivionControl.ColorCorrection,
TweenInfo.new(0.6, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
{
TintColor = Color3.fromRGB(100, 220, 255),
Saturation = 0.3,
Contrast = 0.15,
Brightness = 0,
}
)
ccTween:Play()
end)

OblivionControl.Bloom.Enabled = true
OblivionControl.Bloom.Intensity = 2.5
OblivionControl.Bloom.Size = 35
OblivionControl.Bloom.Threshold = 0.3
pcall(function()
local bloomTween = game_TweenService:Create(
OblivionControl.Bloom,
TweenInfo.new(0.7, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
{ Intensity = 0.6, Size = 24, Threshold = 0.8 }
)
bloomTween:Play()
end)

OblivionControl.SunRays.Enabled = true
OblivionControl.SunRays.Intensity = 0.3
OblivionControl.SunRays.Spread = 0.5

if targetModel then
pcall(function()
if OblivionControl.Highlight then OblivionControl.Highlight:Destroy() end
OblivionControl.Highlight = Instance.new("Highlight")
OblivionControl.Highlight.Name = "OblivionControlHighlight"
OblivionControl.Highlight.Adornee = targetModel
OblivionControl.Highlight.FillColor = Color3.fromRGB(100, 255, 200)
OblivionControl.Highlight.FillTransparency = 0.7
OblivionControl.Highlight.OutlineColor = Color3.fromRGB(100, 255, 200)
OblivionControl.Highlight.OutlineTransparency = 0
OblivionControl.Highlight.Parent = targetModel
end)
end

OblivionControl.TweenConn = game_RunService.Heartbeat:Connect(function()
if not OblivionControl.Active then return end
local t = tick()
pcall(function()
OblivionControl.Bloom.Intensity = 0.6 + 0.2 * math.sin(t * 3)
end)
pcall(function()
local hueShift = (math.sin(t * 0.8) + 1) * 0.5  
OblivionControl.ColorCorrection.TintColor = Color3.fromRGB(
80 + math.floor(40 * hueShift),
220,
255 - math.floor(20 * hueShift)
)
end)
end)
end

local function endControlVisuals()
OblivionControl.Active = false

pcall(function() OblivionControl.ExitSound:Play() end)

local cam = game_Workspace.CurrentCamera
if cam then
pcall(function()
local tween = game_TweenService:Create(
cam,
TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
{FieldOfView = 70}
)
tween:Play()
end)
end

pcall(function()
local ccTween = game_TweenService:Create(
OblivionControl.ColorCorrection,
TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
{
TintColor = Color3.fromRGB(255, 255, 255),
Saturation = 0,
Contrast = 0,
Brightness = 0,
}
)
ccTween:Play()
ccTween.Completed:Once(function()
OblivionControl.ColorCorrection.Enabled = false
end)
end)

pcall(function()
OblivionControl.Bloom.Intensity = 3
OblivionControl.Bloom.Size = 50
local bloomTween = game_TweenService:Create(
OblivionControl.Bloom,
TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
{ Intensity = 0, Size = 0 }
)
bloomTween:Play()
bloomTween.Completed:Once(function()
OblivionControl.Bloom.Enabled = false
end)
end)

pcall(function()
local srTween = game_TweenService:Create(
OblivionControl.SunRays,
TweenInfo.new(0.4),
{ Intensity = 0, Spread = 0 }
)
srTween:Play()
srTween.Completed:Once(function()
OblivionControl.SunRays.Enabled = false
end)
end)

if OblivionControl.Highlight then
pcall(function() OblivionControl.Highlight:Destroy() end)
OblivionControl.Highlight = nil
end

if OblivionControl.TweenConn then
OblivionControl.TweenConn:Disconnect()
OblivionControl.TweenConn = nil
end
end

local function MovePlayerToPosition(targetCFrame, offsetVector)
local PlayerCharacter = GetCurrentCharacter()
if PlayerCharacter and typeof(targetCFrame) == "CFrame" then
local RootPartInstance = PlayerCharacter.HumanoidRootPart
local HumanoidInstance = PlayerCharacter:FindFirstChildOfClass("Humanoid")
pcall(function()
RootPartInstance.CFrame = RootPartInstance.CFrame.Rotation + targetCFrame.Position
end)
if HumanoidInstance.SeatPart == nil or tostring(HumanoidInstance.SeatPart.Parent) ~= "CreatureBlobman" then
HumanoidInstance.Sit = false
end
end
end

function BeginControllingTarget(targetModelInstance)
if typeof(targetModelInstance) == "Instance" and targetModelInstance:IsA("Model") then
local TargetModel = targetModelInstance
local TargetHumanoid = TargetModel:FindFirstChildOfClass("Humanoid")
local TargetRootPart = TargetModel:FindFirstChild("HumanoidRootPart")
local TargetHead = TargetModel:FindFirstChild("Head")
local IsValidTargetType = (function()
if not game_Players:GetPlayerFromCharacter(targetModelInstance) and (targetModelInstance.Name == "YouDecoy" or targetModelInstance.Name == "CreatureBlobman" or tostring(targetModelInstance.Parent.Name) == "Robloxians") then
return true
end
end)()

if TargetModel and TargetHumanoid and TargetRootPart then
local ConnectionsTable = {}
local function CleanupAllConnections()
for _, conn in pairs(ConnectionsTable) do
if typeof(conn) == "RBXScriptConnection" then
pcall(function() conn:Disconnect() end)
end
end
table.clear(ConnectionsTable)
end

_G.ControllingCreature = TargetModel
TargetHumanoid.WalkSpeed = 0
TargetHumanoid.JumpPower = 24
TargetHumanoid.CameraOffset = Vector3.new(0, 0, -0.7)

ConnectionsTable[1] = TargetHumanoid.Died:Connect(function()
_G.ControllingCreature = nil
end)

local VelocityController = Instance.new("BodyVelocity", TargetRootPart)
local PlayerVelocityController = Instance.new("BodyVelocity")
PlayerVelocityController.MaxForce = Vector3.new(0, math.huge, 0)
PlayerVelocityController.Velocity = Vector3.new()
VelocityController.MaxForce = Vector3.new(math.huge, 0, math.huge)

DisableCharacterQuery(TargetModel)

task.spawn(function()
ActivateNoclipMode()
while TargetModel.Parent and _G.ControllingCreature ~= nil do
if IsValidTargetType then
RequestOwnershipAndRemoveLine(TargetHead)
else
RequestOwnershipSingle(TargetHead)
end
TargetHumanoid.AutoRotate = true
task.wait()
end
end)

game_Workspace.CurrentCamera.CameraSubject = TargetHumanoid
startControlVisuals(TargetModel)

local PlayerCharacterCurrent = GetCurrentCharacter()
local PlayerRootPartReference = nil
local PlayerHumanoidReference = nil

if PlayerCharacterCurrent then
PlayerHumanoidReference = PlayerCharacterCurrent:FindFirstChildOfClass("Humanoid")
PlayerRootPartReference = PlayerCharacterCurrent:FindFirstChild("HumanoidRootPart")
PlayerVelocityController.Parent = PlayerRootPartReference

ConnectionsTable[2] = PlayerHumanoidReference.Died:Connect(function()
_G.ControllingCreature = nil
end)
ConnectionsTable[3] = game_UserInputService.JumpRequest:Connect(function()
TargetHumanoid:ChangeState("Jumping")
end)
ConnectionsTable[5] = PlayerHumanoidReference.Changed:Connect(function(propertyName)
if propertyName == "MoveDirection" then
VelocityController.Velocity = PlayerHumanoidReference.MoveDirection * 20
end
end)
ConnectionsTable[6] = workspace.CurrentCamera.Changed:Connect(function(cameraProperty)
if cameraProperty == "CameraSubject" then
game_Workspace.CurrentCamera.CameraSubject = TargetHumanoid
end
end)

local CameraDirectionVector = nil
ConnectionsTable[7] = TargetHead.Changed:Connect(function(headProperty)
if headProperty == "CFrame" then
CameraDirectionVector = game_Workspace.CurrentCamera.CFrame.lookVector
TargetHumanoid.CameraOffset = -Vector3.new(CameraDirectionVector.X, 5, CameraDirectionVector.Z) * 1.7
end
end)

TargetHumanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)

task.spawn(function()
while TargetModel.Parent and _G.ControllingCreature ~= nil and PlayerCharacterCurrent and PlayerCharacterCurrent.Parent do
MovePlayerToPosition(CFrame.new(TargetRootPart.Position + Vector3.new(0, -10, 0)))
task.wait()
end
CleanupAllConnections()
DeactivateNoclipMode()
pcall(function() MovePlayerToPosition(CFrame.new(TargetRootPart.Position + Vector3.new(5, 15, 5))) end)
EnableCharacterQuery(TargetModel)
pcall(function() VelocityController:Destroy() end)
pcall(function() PlayerVelocityController:Destroy() end)
pcall(function() game_Workspace.CurrentCamera.CameraSubject = PlayerHumanoidReference end)
_G.ControllingCreature = nil
pcall(function() PlayerRootPartReference.Velocity = Vector3.new() end)
endControlVisuals()
end)
else
_G.ControllingCreature = nil
end
end
end
end

function ExecuteControlAction()
local ActiveCharacter = GetCurrentCharacter()
if ActiveCharacter then
local CharacterHead = ActiveCharacter:FindFirstChild("Head")
if not CharacterHead then return end
local ActiveCamera = game_Workspace.CurrentCamera
local CharacterHumanoid = ActiveCharacter:FindFirstChildOfClass("Humanoid")
local RaycastResult = game_Workspace:Raycast(CharacterHead.Position, ActiveCamera.CFrame.lookVector * 50, RaycastParameters)
if RaycastResult and CharacterHumanoid and CharacterHumanoid.Health > 0 then
local HitInstanceParent = RaycastResult.Instance.Parent
if HitInstanceParent and HitInstanceParent:FindFirstChildOfClass("Humanoid") then
BeginControllingTarget(HitInstanceParent)
end
end
end
end

function ExecuteControlToggle()
if _G.ControllingCreature then
_G.ControllingCreature = nil
else
ExecuteControlAction()
end
end

CurrentPlayer.CharacterAdded:Connect(function(newCharacter)
RaycastParameters.FilterDescendantsInstances = { newCharacter }
end)

local ControlSection = Tabs.Fun:AddRightGroupbox("Player + NPC Control", "gamepad-2")

local ControlToggle = ControlSection:AddToggle("ControlCharactersToggle", {
Text = "Player + NPC Control",
Default = false,
Tooltip = L("Управляй NPC/игроком на которого смотришь. Наведи камеру → жми бинд.", "Control NPC/player you aim at. Aim camera → press bind."),
})

ControlToggle:AddKeyPicker("ControlKeybind", {
Default = "V",
SyncToggleState = false,
Mode = "Toggle",
Text = "Control Key",
NoUI = false,
Callback = function(isActive)
if not ControlToggle.Value then
return
end
ExecuteControlToggle()
end,
})

ControlToggle:OnChanged(function(enabled)
if enabled then
Library:Notify(L("Player + NPC Control ВКЛ — наведи на цель и жми бинд", "Player + NPC Control ON — aim at target and press bind"), 2)
else
if _G.ControllingCreature then
_G.ControllingCreature = nil
end
Library:Notify(L("Player + NPC Control ВЫКЛ", "Player + NPC Control OFF"), 1)
end
end)
end

function initBombDrone()
FunBombDrone = Tabs.Fun:AddLeftGroupbox("Rocket Drone", "plane")

droneToggle = FunBombDrone:AddToggle("EnableBombDrone", {
Text = "Rocket Drone Mode",
Default = false,
Tooltip = L("WASD — движение, Space — вверх, Shift — вниз. ПКМ — камера.", "WASD — move, Space — up, Shift — down. RMB — camera."),
})

FunBombDrone:AddSlider("DroneSpeed", {
Text = L("Скорость ракеты", "Rocket Speed"),
Default = 85,
Min = 20,
Max = 500,
Rounding = 0,
})

pcall(function()
droneToggle:AddKeyPicker("DroneAddKey", {
Default = "None",
SyncToggleState = false,
Mode = "Toggle",
Text = L("Бинд", "Bind"),
})
end)

FunBombDrone:AddButton({
Text = L("Добавить ракету", "Add Rocket"),
Tooltip = L("Наведи прицел на ракету и нажми F. Можно несколько!", "Aim at rocket and press F. Multiple allowed!"),
Func = function()
if not (Toggles.EnableBombDrone and Toggles.EnableBombDrone.Value) then
Library:Notify(L("Сначала включи Rocket Drone Mode!", "Enable Rocket Drone Mode first!"), 3)
return
end
local targetPart = getTargetUnderCrosshair()
if targetPart then
attachInsideRocket(targetPart)
else
Library:Notify("❌ Наведи прицел на ракету!", 2)
end
end,
})

FunBombDrone:AddButton({
Text = L("Сбросить все ракеты", "Reset All Rockets"),
Func = function()
resetRocketCamera()
Library:Notify(L("Rocket Drone: сброшен", "Rocket Drone: reset"), 2)
end,
})

isCamActive = false
trackedParts = {}
renderConn = nil
savedCamSubject = nil
savedCamMode = nil
savedCamMax = nil
savedCamMin = nil

local function disableRocketMovers(model)
for _, obj in ipairs(model:GetDescendants()) do
if obj:IsA("BodyVelocity") or obj:IsA("BodyGyro") or obj:IsA("BodyAngularVelocity") or obj:IsA("BodyPosition") or obj:IsA("BodyForce") or obj:IsA("BodyThrust") or obj:IsA("LinearVelocity") or obj:IsA("AngularVelocity") then
pcall(function() obj:Destroy() end)
end
end
end

local function ensureBV(part)
local bv = part:FindFirstChild("RocketMove")
if not bv then
bv = Instance.new("BodyVelocity")
bv.Name = "RocketMove"
bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bv.Velocity = Vector3.zero
bv.Parent = part
end
return bv
end

local function ensureBG(part)
local bg = part:FindFirstChild("RocketGyro")
if not bg then
bg = Instance.new("BodyGyro")
bg.Name = "RocketGyro"
bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
bg.CFrame = part.CFrame
bg.Parent = part
end
return bg
end

function resetRocketCamera()
isCamActive = false
if renderConn then
renderConn:Disconnect()
renderConn = nil
end
for _, p in ipairs(trackedParts) do
if p and p.Parent then
local bv = p:FindFirstChild("RocketMove")
if bv then pcall(function() bv:Destroy() end) end
local bg = p:FindFirstChild("RocketGyro")
if bg then pcall(function() bg:Destroy() end) end
end
end
trackedParts = {}
local cam = workspace.CurrentCamera
if cam then
if savedCamSubject then
cam.CameraSubject = savedCamSubject
savedCamSubject = nil
else
local char = LocalPlayer.Character
local hum = char and char:FindFirstChildOfClass("Humanoid")
if hum then cam.CameraSubject = hum end
end
cam.CameraType = Enum.CameraType.Custom
end
if savedCamMode then LocalPlayer.CameraMode = savedCamMode; savedCamMode = nil end
if savedCamMax then LocalPlayer.CameraMaxZoomDistance = savedCamMax; savedCamMax = nil end
if savedCamMin then LocalPlayer.CameraMinZoomDistance = savedCamMin; savedCamMin = nil end
end

function getTargetUnderCrosshair()
local char = LocalPlayer.Character
if not char then return nil end
local cam = workspace.CurrentCamera
if not cam then return nil end
local vc = Vector2.new(cam.ViewportSize.X / 2, cam.ViewportSize.Y / 2)
local ray = cam:ViewportPointToRay(vc.X, vc.Y)
local rp = RaycastParams.new()
rp.FilterType = Enum.RaycastFilterType.Exclude
rp.FilterDescendantsInstances = {char}
rp.IgnoreWater = true
local result = workspace:Raycast(ray.Origin, ray.Direction * 500, rp)
if result and result.Instance then
local hitPart = result.Instance
local scanFloor = hitPart
while scanFloor and scanFloor ~= workspace do
if scanFloor.Name == "Map" or scanFloor.Name == "Baseplate" or scanFloor:IsA("Terrain") then
scanFloor = nil
break
end
scanFloor = scanFloor.Parent
end
if scanFloor and not hitPart:IsA("Terrain") and hitPart.Name ~= "Baseplate" then
return hitPart
end
end
return nil
end

function attachInsideRocket(part)
if not part or not part:IsA("BasePart") then return end
for _, p in ipairs(trackedParts) do
if p == part then
Library:Notify(L("Эта ракета уже управляется!", "This rocket is already controlled!"), 2)
return
end
end
table.insert(trackedParts, part)
if not isCamActive then
isCamActive = true
local cam = workspace.CurrentCamera
if cam then
savedCamSubject = cam.CameraSubject
savedCamMode = LocalPlayer.CameraMode
savedCamMax = LocalPlayer.CameraMaxZoomDistance
savedCamMin = LocalPlayer.CameraMinZoomDistance
end
end
Library:Notify("🚀 Ракета #" .. #trackedParts .. " добавлена! Всего: " .. #trackedParts, 2)
local model = part.Parent
if model and model:IsA("Model") then
disableRocketMovers(model)
end
ensureBV(part)
ensureBG(part)
local cam = workspace.CurrentCamera
if cam then
cam.CameraSubject = trackedParts[1]
cam.CameraType = Enum.CameraType.Custom
end
LocalPlayer.CameraMode = Enum.CameraMode.Classic
LocalPlayer.CameraMaxZoomDistance = 50
LocalPlayer.CameraMinZoomDistance = 15
if renderConn then renderConn:Disconnect() end
renderConn = RunService.RenderStepped:Connect(function(dt)
if isCamActive and #trackedParts > 0 then
local cam2 = workspace.CurrentCamera
if not cam2 then return end
local speed = (Options.DroneSpeed and Options.DroneSpeed.Value) or 85
if LocalPlayer.CameraMode ~= Enum.CameraMode.Classic then
LocalPlayer.CameraMode = Enum.CameraMode.Classic
end
if LocalPlayer.CameraMinZoomDistance < 15 then
LocalPlayer.CameraMinZoomDistance = 15
end
if #trackedParts > 0 and trackedParts[1] and trackedParts[1].Parent then
if cam2.CameraSubject ~= trackedParts[1] then
cam2.CameraSubject = trackedParts[1]
end
end
cam2.CameraType = Enum.CameraType.Custom
local moveVector = Vector3.zero
if UserInputService:IsKeyDown(Enum.KeyCode.W) then
moveVector = moveVector + cam2.CFrame.LookVector
end
if UserInputService:IsKeyDown(Enum.KeyCode.S) then
moveVector = moveVector - cam2.CFrame.LookVector
end
if UserInputService:IsKeyDown(Enum.KeyCode.A) then
moveVector = moveVector - cam2.CFrame.RightVector
end
if UserInputService:IsKeyDown(Enum.KeyCode.D) then
moveVector = moveVector + cam2.CFrame.RightVector
end
if UserInputService:IsKeyDown(Enum.KeyCode.Space) then
moveVector = moveVector + Vector3.new(0, 1, 0)
end
if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) or UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then
moveVector = moveVector - Vector3.new(0, 1, 0)
end
local validParts = {}
for _, p in ipairs(trackedParts) do
if p and p.Parent then
pcall(function() p.Anchored = false end)
local bv = ensureBV(p)
local bg = ensureBG(p)
if moveVector.Magnitude > 0 then
bv.Velocity = moveVector.Unit * speed
bg.CFrame = CFrame.new(p.Position, p.Position + moveVector.Unit)
else
bv.Velocity = Vector3.zero
end
table.insert(validParts, p)
end
end
trackedParts = validParts
else
resetRocketCamera()
end
end)
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
if gameProcessed then return end
if not (Toggles.EnableBombDrone and Toggles.EnableBombDrone.Value) then return end
if Options.DroneAddKey and tpKeyMatches(input, Options.DroneAddKey.Value) then
local targetPart = getTargetUnderCrosshair()
if targetPart then
attachInsideRocket(targetPart)
else
Library:Notify("❌ Наведи прицел на ракету!", 2)
end
end
end)

Toggles.EnableBombDrone:OnChanged(function()
if Toggles.EnableBombDrone.Value then
Library:Notify(L("Rocket Drone: нажми бинд чтобы добавить ракету. WASD — движение, ПКМ — камера", "Rocket Drone: press bind to add rocket. WASD — move, RMB — camera"), 4)
else
resetRocketCamera()
Library:Notify(L("Rocket Drone: сброшен", "Rocket Drone: reset"), 2)
end
end)
end
pcall(initBombDrone)

do
print("[OBLIVION DEBUG] Before Settings section")
local SettingsSection = Tabs.Settings:AddLeftGroupbox("Menu", "wrench")

local BGImageSection = Tabs.Settings:AddLeftGroupbox(L("Фон картинки", "Background Image"), "image")

BGImageSection:AddInput("BGImageUrl", {
Text = L("URL картинки", "Image URL"),
Default = "",
Placeholder = "https://...",
Tooltip = L("Вставь прямую ссылку на картинку (jpg/png/webp)", "Paste direct image URL (jpg/png/webp)"),
})

BGImageSection:AddButton({
Text = L("Установить фон", "Set Background"),
Func = function()
local url = Options.BGImageUrl and Options.BGImageUrl.Value or ""
if url == "" then
Library:Notify(L("Введи URL картинки", "Enter image URL"), 2)
return
end
if Window and Window.SetBackgroundImage then
Window:SetBackgroundImage(url)
Library:Notify(L("Фон установлен", "Background set"), 2)
else
Library:Notify(L("Окно не найдено", "Window not found"), 2)
end
end
})

BGImageSection:AddButton({
Text = L("Убрать фон", "Remove Background"),
Func = function()
if Window and Window.SetBackgroundImage then
Window:SetBackgroundImage("")
Library:Notify(L("Фон убран", "Background removed"), 2)
end
end
})

do
local NotifySection = Tabs.Settings:AddRightGroupbox("Notifications", "bell")

_notifyDB = {}

local function _db(key, cd)
local now = tick()
if _notifyDB[key] and now - _notifyDB[key] < (cd or 3) then
return false
end
_notifyDB[key] = now
return true
end

local function _rp(ref)
if not ref or ref == '' then return 'Unknown' end
if typeof(ref) == 'Instance' and ref:IsA('Player') then
local u, d = ref.Name, ref.DisplayName or ''
return (d ~= '' and d ~= u) and (u .. ' (' .. d .. ')') or u
end
if type(ref) == 'string' then
local p = Players:FindFirstChild(ref)
if p then
local u, d = p.Name, p.DisplayName or ''
return (d ~= '' and d ~= u) and (u .. ' (' .. d .. ')') or u
end
return ref
end
return tostring(ref)
end

_notifyCons = {}

local function addCon(group, key, conn)
if not _notifyCons[group] then _notifyCons[group] = {} end
if _notifyCons[group][key] then
pcall(function() _notifyCons[group][key]:Disconnect() end)
end
_notifyCons[group][key] = conn
end

local function clearCon(group, key)
if _notifyCons[group] and _notifyCons[group][key] then
pcall(function() _notifyCons[group][key]:Disconnect() end)
_notifyCons[group][key] = nil
end
end

_rejoinMemory = {}

NotifySection:AddCheckbox('JoinNotify', {
Text = 'Join Notify',
Default = false,
Callback = function(v)
clearCon('join', 'add')
if not v then return end
addCon('join', 'add', Players.PlayerAdded:Connect(function(plr)
plr.CharacterAdded:Wait()
if not _db('join_' .. plr.UserId, 5) then return end
Library:Notify({
Title = 'Oblivion',
Description = _rp(plr) .. ' Joined the server',
Time = 6,
})
end))
end,
})

NotifySection:AddCheckbox('LeaveNotify', {
Text = 'Leave Notify',
Default = false,
Callback = function(v)
clearCon('join', 'remove')
if not v then return end
addCon('join', 'remove', Players.PlayerRemoving:Connect(function(plr)
if not _db('leave_' .. plr.UserId, 5) then return end
Library:Notify({
Title = 'Oblivion',
Description = _rp(plr) .. ' Left the server',
Time = 4,
})
end))
end,
})

NotifySection:AddCheckbox('RejoinNotify', {
Text = 'Rejoin Notify',
Default = false,
Callback = function(v)
clearCon('join', 'rejoin_add')
clearCon('join', 'rejoin_rem')
if not v then return end
addCon('join', 'rejoin_rem', Players.PlayerRemoving:Connect(function(plr)
_rejoinMemory[plr.UserId] = { t = tick(), display = _rp(plr) }
end))
addCon('join', 'rejoin_add', Players.PlayerAdded:Connect(function(plr)
local entry = _rejoinMemory[plr.UserId]
if not entry then return end
plr.CharacterAdded:Wait()
if not _db('rejoin_' .. plr.UserId, 5) then return end
local away = math.floor(tick() - entry.t)
Library:Notify({
Title = 'Oblivion',
Description = _rp(plr) .. ' Rejoined\nAway for: ' .. away .. 's',
Time = 6,
})
_rejoinMemory[plr.UserId] = nil
end))
end,
})

NotifySection:AddCheckbox('GrabbedNotify', {
Text = 'Grabbed Notify',
Default = false,
Callback = function(v)
clearCon('grab', 'self')
if not v then return end
local isHeld = LocalPlayer:WaitForChild('IsHeld', 5)
if not isHeld then return end
addCon('grab', 'self', isHeld.Changed:Connect(function()
if isHeld.Value then
if not _db('grabbed_self', 3) then return end
local head = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild('Head')
local po = head and head:FindFirstChild('PartOwner')
local who = (po and po.Value ~= '') and _rp(po.Value) or 'Someone'
Library:Notify({
Title = 'Oblivion',
Description = who .. ' Is grabbing you!',
Time = 5,
})
else
if _db('released_self', 3) then
Library:Notify({
Title = 'Oblivion',
Description = 'You were released',
Time = 3,
})
end
end
end))
end,
})

NotifySection:AddCheckbox('KickNotify', {
Text = 'Kick Notify',
Default = false,
Callback = function(v)
clearCon('world', 'bh_add')
clearCon('world', 'bh_remove')
if not v then return end
addCon('world', 'bh_add', workspace.ChildAdded:Connect(function(child)
if child.Name ~= 'BlackHoleKick' then return end
addCon('world', 'bh_remove', Players.PlayerRemoving:Connect(function(plr2)
if not _db('kick_' .. plr2.UserId, 5) then return end
Library:Notify({
Title = 'Oblivion',
Description = _rp(plr2) .. ' has been KICKED',
Time = 6,
})
child.Name = plr2.Name .. 'KICK'
end))
end))
end,
})

NotifySection:AddCheckbox('LagDetectNotify', {
Text = 'Lag Detect Notify',
Default = false,
Tooltip = 'Уведомлять когда кто-то лагает сервер',
Callback = function(v)
clearCon('lag', 'detect')
if not v then return end
local lastLagNotify = 0
addCon('lag', 'detect', workspace.ChildAdded:Connect(function(child)
if not child:IsA('Model') and not child:IsA('BasePart') then return end
local nameLower = child.Name:lower()
if nameLower:find('grabline') or nameLower:find('grab') or nameLower:find('beam') then
task.wait(0.5)
local count = 0
for _, c in ipairs(workspace:GetChildren()) do
local n = c.Name:lower()
if n:find('grabline') or n:find('grab') or n:find('beam') then
count = count + 1
end
end
if count > 50 and tick() - lastLagNotify > 10 then
lastLagNotify = tick()
Library:Notify({
Title = 'Oblivion',
Description = 'Server lag detected! (' .. count .. ' grab objects)',
Time = 5,
})
end
end
end))
end,
})

function notifyLineLagStart()
if Toggles.LagDetectNotify and Toggles.LagDetectNotify.Value then
Library:Notify({
Title = 'Oblivion',
Description = LocalPlayer.Name .. ' started Line Lag',
Time = 3,
})
end
end

function notifyLineLagStop()
if Toggles.LagDetectNotify and Toggles.LagDetectNotify.Value then
Library:Notify({
Title = 'Oblivion',
Description = LocalPlayer.Name .. ' stopped Line Lag',
Time = 3,
})
end
end

function notifyPacketLagStart()
if Toggles.LagDetectNotify and Toggles.LagDetectNotify.Value then
Library:Notify({
Title = 'Oblivion',
Description = LocalPlayer.Name .. ' started Packet Lag',
Time = 3,
})
end
end

function notifyPacketLagStop()
if Toggles.LagDetectNotify and Toggles.LagDetectNotify.Value then
Library:Notify({
Title = 'Oblivion',
Description = LocalPlayer.Name .. ' stopped Packet Lag',
Time = 3,
})
end
end

local menuOk = pcall(function()
SettingsSection:AddLabel("Menu bind")
:AddKeyPicker("MenuKeybind", { Default = "RightShift", NoUI = true, Text = "Menu keybind" })
end)
if not menuOk then warn("[Oblivion] MenuKeybind failed") end

local cursorOk = pcall(function()
SettingsSection:AddToggle("ShowCustomCursor", {
Text = "Custom Cursor",
Default = true,
Callback = function(Value)
Library.ShowCustomCursor = Value
end,
})
end)
if not cursorOk then warn("[Oblivion] ShowCustomCursor failed") end

local notifyOk = pcall(function()
SettingsSection:AddDropdown("NotificationSide", {
Values = { "Left", "Right" },
Default = "Right",
Text = "Notification Side",
Callback = function(Value)
pcall(function() Library:SetNotifySide(Value) end)
end,
})
end)
if not notifyOk then warn("[Oblivion] NotificationSide failed") end

local langOk = pcall(function()
SettingsSection:AddDropdown("LanguageDropdown", {
Values = {"Русский", "English"},
Default = "Русский",
Text = "Language / Язык",
Tooltip = "Выбери язык интерфейса: Русский или English",
Callback = function(Value)
currentLang = Value
if writefile then pcall(function() writefile("obvilion_lang.txt", Value) end) end
task.wait(0.3)
Library:Unload()
task.wait(0.5)
if readfile and loadstring then
pcall(function()
local scriptContent = readfile("Obvilion (2).lua")
loadstring(scriptContent)()
end)
end
end,
})
end)
if not langOk then warn("[Oblivion] LanguageDropdown failed") end

local dpiOk = pcall(function()
SettingsSection:AddDropdown("DPIDropdown", {
Values = { "50%", "75%", "100%", "125%", "150%", "175%", "200%" },
Default = "100%",
Text = "DPI Scale",
Callback = function(Value)
Value = Value:gsub("%%", "")
pcall(function() Library:SetDPIScale(tonumber(Value)) end)
end,
})
end)
if not dpiOk then warn("[Oblivion] DPI failed") end

local cornerOk = pcall(function()
local defCorner = 5
pcall(function() defCorner = Library.CornerRadius or 5 end)
SettingsSection:AddSlider("UICornerSlider", {
Text = "Corner Radius",
Default = defCorner,
Min = 0,
Max = 20,
Rounding = 0,
Callback = function(value)
pcall(function() Window:SetCornerRadius(value) end)
end
})
end)
if not cornerOk then warn("[Oblivion] Corner slider failed") end

task.spawn(function()
local CoreGui = (gethui and gethui()) or game:GetService("CoreGui")
local oldWm = CoreGui:FindFirstChild("OblivionWatermark")
if oldWm then oldWm:Destroy() end

local watermarkGui = Instance.new("ScreenGui")
watermarkGui.Name = "OblivionWatermark"
watermarkGui.ResetOnSpawn = false
watermarkGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
watermarkGui.Parent = CoreGui

local watermarkFrame = Instance.new("Frame")
watermarkFrame.Size = UDim2.new(0, 0, 0, 30)
watermarkFrame.AutomaticSize = Enum.AutomaticSize.X
watermarkFrame.Position = UDim2.new(0, 20, 0, 20)
watermarkFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 25)
watermarkFrame.BackgroundTransparency = 0.2
watermarkFrame.BorderSizePixel = 0
watermarkFrame.Visible = false
watermarkFrame.Parent = watermarkGui

local wmCorner = Instance.new("UICorner")
wmCorner.CornerRadius = UDim.new(0, 8)
wmCorner.Parent = watermarkFrame

local wmStroke = Instance.new("UIStroke")
wmStroke.Color = Color3.fromRGB(60, 60, 75)
wmStroke.Thickness = 1.5
wmStroke.Parent = watermarkFrame

local wmText = Instance.new("TextLabel")
wmText.Size = UDim2.new(0, 0, 1, 0)
wmText.AutomaticSize = Enum.AutomaticSize.X
wmText.Position = UDim2.new(0, 0, 0, 0)
wmText.BackgroundTransparency = 1
wmText.Text = "<b>Oblivion</b> | FPS: 0 | Ping: 0ms"
wmText.TextColor3 = Color3.fromRGB(230, 230, 230)
wmText.Font = Enum.Font.GothamMedium
wmText.TextSize = 14
wmText.RichText = true
wmText.Parent = watermarkFrame

local wmPadding = Instance.new("UIPadding")
wmPadding.PaddingLeft = UDim.new(0, 14)
wmPadding.PaddingRight = UDim.new(0, 14)
wmPadding.Parent = wmText

watermarkFrame.Active = true
local wmUIS = game:GetService("UserInputService")
local wmDragging = false
local wmDragStart = nil
local wmStartPos = nil
pcall(function()
local savedWmPos = readfile and readfile("oblivion_wm_pos.txt")
if savedWmPos then
local sx, sy = savedWmPos:match("^(%-?%d+),(%-?%d+)$")
if sx and sy then
watermarkFrame.Position = UDim2.new(0, tonumber(sx) or 20, 0, tonumber(sy) or 20)
end
end
end)
watermarkFrame.InputBegan:Connect(function(input)
if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
wmDragging = true
wmDragStart = input.Position
wmStartPos = watermarkFrame.Position
input.Changed:Connect(function()
if input.UserInputState == Enum.UserInputState.End then
wmDragging = false
pcall(function()
if writefile then
writefile("oblivion_wm_pos.txt", math.floor(watermarkFrame.Position.X.Offset) .. "," .. math.floor(watermarkFrame.Position.Y.Offset))
end
end)
end
end)
end
end)
wmUIS.InputChanged:Connect(function(input)
if wmDragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
local delta = input.Position - wmDragStart
watermarkFrame.Position = UDim2.new(wmStartPos.X.Scale, wmStartPos.X.Offset + delta.X, wmStartPos.Y.Scale, wmStartPos.Y.Offset + delta.Y)
end
end)

local wmFrames = 0
local wmLastUpdate = tick()
local wmFPS = 0

RunService.RenderStepped:Connect(function()
wmFrames = wmFrames + 1
local now = tick()
if now - wmLastUpdate >= 1 then
wmFPS = wmFrames
wmFrames = 0
wmLastUpdate = now
end

if Toggles.EnableWatermark and Toggles.EnableWatermark.Value then
watermarkFrame.Visible = true
local ping = 0
pcall(function()
ping = math.floor(LocalPlayer:GetNetworkPing() * 1000)
end)
if ping == 0 then
pcall(function()
ping = math.floor(game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue())
end)
end
wmText.Text = string.format("<b>Oblivion</b> | FPS: %d | Ping: %d ms", wmFPS, ping)
else
watermarkFrame.Visible = false
end
end)

local wmOk = pcall(function()
SettingsSection:AddToggle("EnableWatermark", {
Text = "Watermark",
Default = true,
Tooltip = L("Показывать красивый ватермарк (Oblivion, FPS, Ping) на экране", "Show watermark (Oblivion, FPS, Ping) on screen"),
})
end)
if not wmOk then warn("[Oblivion] Watermark toggle failed") end
end)
SettingsSection:AddDivider()

SettingsSection:AddButton({
Text = "Unload",
Func = function()
if workspace.CurrentCamera then
workspace.CurrentCamera.FieldOfView = defaultFOV
end
Library:Unload()
end,
Tooltip = L("Полностью выгрузить чит из игры", "Completely unload cheat from game")
})

pcall(function() Library.ToggleKeybind = Options.MenuKeybind end)

pcall(function()
Library:OnUnload(function()
if workspace.CurrentCamera then
workspace.CurrentCamera.FieldOfView = defaultFOV
end
if watermarkGui then watermarkGui:Destroy() end
end)
end)

task.spawn(function()
local t0 = tick()
while not (themeDone and saveDone) do
if tick() - t0 > 15 then break end 
task.wait(0.05)
end
pcall(function()
do
do
local CoconutToys = Tabs.Fun:AddRightGroupbox("Coconut Visuals", "citrus")

CoconutToys:AddToggle("EnableCocoPenis", {
Text = "Coconut Penis",
Default = false,
Tooltip = L("Создает пенис из кокосов", "Creates penis from coconuts"),
})

CoconutToys:AddSlider("CocoPenisLength", {
Text = "Penis Length",
Default = 5,
Min = 1,
Max = 20,
Rounding = 0,
Tooltip = L("Длина (в кокосах)", "Length (in coconuts)"),
})

CoconutToys:AddToggle("EnableCocoBreasts", {
Text = "Coconut Breasts",
Default = false,
Tooltip = L("2 кокоса на груди", "2 coconuts on chest"),
})

CoconutToys:AddToggle("EnableCocoButt", {
Text = "Coconut Butt",
Default = false,
Tooltip = L("2 кокоса на попке", "2 coconuts on butt"),
})

task.spawn(function()
local RS = game:GetService("ReplicatedStorage")
local GrabEvents = RS:WaitForChild("GrabEvents", 5)
local MenuToys = RS:WaitForChild("MenuToys", 5)
if not (GrabEvents and MenuToys) then return end

local SetNetworkOwner = GrabEvents:FindFirstChild("SetNetworkOwner")
local SpawnToy = MenuToys:FindFirstChild("SpawnToyRemoteFunction")
local DestroyToy = MenuToys:FindFirstChild("DestroyToy")

while task.wait(0.05) do
local penisEn = Toggles.EnableCocoPenis and Toggles.EnableCocoPenis.Value
local breastsEn = Toggles.EnableCocoBreasts and Toggles.EnableCocoBreasts.Value
local buttEn = Toggles.EnableCocoButt and Toggles.EnableCocoButt.Value

if not (penisEn or breastsEn or buttEn) then continue end

local char = LocalPlayer.Character
local root = char and char:FindFirstChild("HumanoidRootPart")
if not root then continue end

local len = Options.CocoPenisLength and Options.CocoPenisLength.Value or 5
local neededCount = 0
if penisEn then neededCount = neededCount + len + 2 end
if breastsEn then neededCount = neededCount + 2 end
if buttEn then neededCount = neededCount + 2 end

local spawnedToysFolder = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
if not spawnedToysFolder then continue end

local allCoconuts = {}
for _, toy in ipairs(spawnedToysFolder:GetChildren()) do
if toy.Name == "FoodCoconut" then
table.insert(allCoconuts, toy)
end
end

if #allCoconuts < neededCount and SpawnToy then
task.spawn(function()
pcall(function()
SpawnToy:InvokeServer("FoodCoconut", root.CFrame * CFrame.new(-5, 5, 10), Vector3.zero)
end)
end)
end

local cocoIndex = 1
local velOffset = CFrame.new(root.Velocity / 100)

local function positionCoco(coco, offsetCFrame)
local part = coco:FindFirstChild("SoundPart")
local holdPart = coco:FindFirstChild("HoldPart")
local rigid = holdPart and holdPart:FindFirstChild("RigidConstraint")
local owner = part and part:FindFirstChild("PartOwner")

if part and holdPart and rigid then
if owner and owner.Value == LocalPlayer.Name then
part.CFrame = root.CFrame * offsetCFrame * velOffset
part.Velocity = Vector3.zero
end

if not owner or owner.Value ~= LocalPlayer.Name then
if SetNetworkOwner then
pcall(function() SetNetworkOwner:FireServer(part, part.CFrame) end)
end
end

if rigid.Attachment1 and DestroyToy then
pcall(function() DestroyToy:FireServer(coco) end)
end

for _, p in ipairs(coco:GetChildren()) do
if p:IsA("BasePart") then
p.CanCollide = false
p.CanQuery = false
if p.Transparency ~= 1 then p.Transparency = 0 end
end
end
end
end

if penisEn then
local balls = {CFrame.new(-0.55, -1.2, -0.8), CFrame.new(0.55, -1.2, -0.8)}
for i = 1, 2 do
if allCoconuts[cocoIndex] then
positionCoco(allCoconuts[cocoIndex], balls[i])
cocoIndex = cocoIndex + 1
end
end
for i = 1, len do
if allCoconuts[cocoIndex] then
local offsetZ = -0.8 - (i * 0.9)
positionCoco(allCoconuts[cocoIndex], CFrame.new(0, -1.2, offsetZ))
cocoIndex = cocoIndex + 1
end
end
end

if breastsEn then
local breastOffsets = {CFrame.new(-0.6, 0.5, -0.9), CFrame.new(0.6, 0.5, -0.9)}
for i = 1, 2 do
if allCoconuts[cocoIndex] then
positionCoco(allCoconuts[cocoIndex], breastOffsets[i])
cocoIndex = cocoIndex + 1
end
end
end

if buttEn then
local buttOffsets = {CFrame.new(-0.55, -1, 0.75), CFrame.new(0.55, -1, 0.75)}
for i = 1, 2 do
if allCoconuts[cocoIndex] then
positionCoco(allCoconuts[cocoIndex], buttOffsets[i])
cocoIndex = cocoIndex + 1
end
end
end
end
end)

end

end
do
local SparklerBuildGroup = Tabs.Fun:AddLeftGroupbox("Sparkler Build", "wand-sparkles")

local buildHighRun = false
local buildConnection = nil
local buildToy = nil
local buildPart = nil
local buildBp = nil
local buildBg = nil
local multiSparklers = {}
local currentBuildShape = "Heart"

SparklerBuildGroup:AddToggle("EnableSparklerBuild", {
Text = "Sparkler Build",
Default = false,
Tooltip = L("Спавнит бенгальские огни и рисует выбранную фигуру", "Spawns sparklers and draws selected shape"),
})

SparklerBuildGroup:AddDropdown("SparklerBuildShape", {
Text = "Shape",
Values = {"Heart", "Star", "Tree", "Saturn", "Cross", "Sword", "Wings", "Crown", "Tornado", "Rocket", "Umbrella", "Atom", "DNA", "Planets"},
Default = 1,
Multi = false,
Tooltip = L("Фигура для рисования бенгальскими огнями", "Shape for sparkler drawing"),
})

local function setupSparklerPart(toy, part)
for _, v in ipairs(toy:GetDescendants()) do
if v:IsA("BasePart") then
v.Anchored = false
v.CanCollide = false
v.Massless = true
elseif v:IsA("JointInstance") or v:IsA("Weld") or v:IsA("WeldConstraint") or v:IsA("Motor6D") then
pcall(function() v:Destroy() end)
elseif v:IsA("Tool") then
v.Enabled = false
end
end
part:BreakJoints()
local bp = Instance.new("BodyPosition")
bp.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bp.P = 20000
bp.D = 500
bp.Parent = part
local bg = Instance.new("BodyGyro")
bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
bg.P = 3000
bg.CFrame = CFrame.new()
bg.Parent = part
return bp, bg
end

local function setupMultiPart(toy, part)
for _, v in ipairs(toy:GetDescendants()) do
if v:IsA("BasePart") then
v.Anchored = false
v.CanCollide = false
v.Massless = true
elseif v:IsA("JointInstance") or v:IsA("Weld") or v:IsA("WeldConstraint") or v:IsA("Motor6D") then
pcall(function() v:Destroy() end)
elseif v:IsA("Tool") then
v.Enabled = false
end
end
part:BreakJoints()
return part
end

local function spawnSingleSparkler(RS, player, hrp)
pcall(function()
RS.MenuToys.SpawnToyRemoteFunction:InvokeServer("FireworkSparkler", hrp.CFrame * CFrame.new(0, 50, 0), Vector3.zero)
end)
local folder = workspace:WaitForChild(player.Name .. "SpawnedInToys", 5)
if not folder then return nil end
local toy = folder:WaitForChild("FireworkSparkler", 5)
if not toy then return nil end
local part = toy:FindFirstChild("Handle") or toy:FindFirstChildWhichIsA("BasePart")
if not part then return nil end
task.wait(0.2)
local bp, bg = setupSparklerPart(toy, part)
return toy, part, bp, bg
end

local function clearMultiSparklers()
for _, s in ipairs(multiSparklers) do
pcall(function() s.toy:Destroy() end)
end
multiSparklers = {}
end

multiShapeConfigs = {
Saturn = {
count = 5,
update = function(i, t, centerCf)
if i <= 3 then
local idx = i
local radius = 8
local angle = t * 3.6 + (idx * (2 * math.pi / 3))
local tiltAngle = math.rad(idx * 60)
local offset = Vector3.new(radius * math.cos(angle), radius * math.sin(angle), 0)
local rotated = CFrame.Angles(tiltAngle, 0, 0) * offset
return centerCf * CFrame.new(rotated)
else
local idx = i - 3
local ringRadius = 15
local angle = t * 2.4 + (idx * math.pi)
local offset = Vector3.new(ringRadius * math.cos(angle), 0, ringRadius * math.sin(angle))
local tilted = CFrame.Angles(math.rad(15), 0, 0) * offset
return centerCf * CFrame.new(tilted)
end
end
},
Atom = {
count = 4,
update = function(i, t, centerCf)
if i == 1 then
return centerCf
else
local idx = i - 1
local radius = 12
local angle = t * 2.0 + (idx * (2 * math.pi / 3))
local offset
if idx == 1 then
offset = Vector3.new(radius * math.cos(angle), radius * math.sin(angle), 0)
elseif idx == 2 then
offset = Vector3.new(0, radius * math.sin(angle), radius * math.cos(angle))
else
offset = Vector3.new(radius * math.cos(angle), 0, radius * math.sin(angle))
end
return centerCf * CFrame.new(offset)
end
end
},
Rocket = {
count = 5,
height = 15,
update = function(i, t, centerCf)
if i == 1 then
return centerCf * CFrame.new(0, 8, 0)
elseif i == 2 or i == 3 then
local y = i == 2 and 2 or -2
local radius = 2
local angle = (i - 2) * math.pi + t * 4
local offset = Vector3.new(radius * math.cos(angle), 0, radius * math.sin(angle))
return centerCf * CFrame.new(0, y, 0) * CFrame.new(offset)
else
local side = i == 4 and 1 or -1
local x = side * 5
return centerCf * CFrame.new(x, -6, 0)
end
end
},
Umbrella = {
count = 5,
height = 15,
update = function(i, t, centerCf)
if i == 1 then
return centerCf * CFrame.new(0, 5, 0)
else
local idx = i - 1
local radius = 8
local angle = idx * (2 * math.pi / 4)
local offset = Vector3.new(radius * math.cos(angle), -1, radius * math.sin(angle))
return centerCf * CFrame.new(offset)
end
end
},
Planets = {
count = 5,
update = function(i, t, centerCf)
if i == 1 then
return centerCf
else
local idx = i - 1
local ringRadius = 15
local angle = t * 1.5 + idx * (math.pi / 2)
local offset = Vector3.new(ringRadius * math.cos(angle), 0, ringRadius * math.sin(angle))
return centerCf * CFrame.new(offset)
end
end
}
}

local function isMultiShape(shape)
return multiShapeConfigs[shape] ~= nil
end

local function spawnMultiSparklers(RS, player, hrp, shape)
clearMultiSparklers()
local config = multiShapeConfigs[shape]
if not config then return end
local folder = workspace:WaitForChild(player.Name .. "SpawnedInToys", 5)
if not folder then return end

for _, child in ipairs(folder:GetChildren()) do
if child.Name == "FireworkSparkler" then
pcall(function() child:Destroy() end)
end
end

local function claimSparkler(toy)
if toy:FindFirstChild("ClaimedByBuild") then return false end
local tag = Instance.new("BoolValue")
tag.Name = "ClaimedByBuild"
tag.Value = true
tag.Parent = toy
return true
end

local function spawnAndClaim(cf)
pcall(function()
RS.MenuToys.SpawnToyRemoteFunction:InvokeServer("FireworkSparkler", cf, Vector3.zero)
end)
local start = tick()
while (tick() - start) < 3 do
for _, child in ipairs(folder:GetChildren()) do
if child.Name == "FireworkSparkler" and claimSparkler(child) then
child.Parent = folder
return child
end
end
task.wait(0.05)
end
return nil
end

for i = 1, config.count do
local height = config.height or 25
local cf = hrp.CFrame * CFrame.new(0, height, 0)
local toy = spawnAndClaim(cf)
if toy then
local part = toy:FindFirstChild("Handle") or toy:FindFirstChildWhichIsA("BasePart")
if part then
pcall(function()
RS.GrabEvents.SetNetworkOwner:FireServer(part, part.CFrame)
end)
task.wait(0.1)
setupMultiPart(toy, part)
table.insert(multiSparklers, { toy = toy, part = part, index = i })
end
end
task.wait(0.1)
end
end

Toggles.EnableSparklerBuild:OnChanged(function()
buildHighRun = Toggles.EnableSparklerBuild.Value
local RS = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local player = LocalPlayer

if buildHighRun then
currentBuildShape = Options.SparklerBuildShape and Options.SparklerBuildShape.Value or "Heart"
task.spawn(function()
if not player.Character then return end
local hrp = player.Character:FindFirstChild("HumanoidRootPart")
if not hrp then return end

if isMultiShape(currentBuildShape) then
spawnMultiSparklers(RS, player, hrp, currentBuildShape)
if #multiSparklers == 0 then return end
else
buildToy, buildPart, buildBp, buildBg = spawnSingleSparkler(RS, player, hrp)
if not buildToy then return end
end

local t = 0
if buildConnection then buildConnection:Disconnect() end
buildConnection = RunService.Heartbeat:Connect(function(dt)
local ok, err = pcall(function()
if not buildHighRun then
if buildConnection then buildConnection:Disconnect() end
return
end

local char = player.Character
local currentHrp = char and char:FindFirstChild("HumanoidRootPart")
if not currentHrp then return end

t = t + (8 * dt)

local config = multiShapeConfigs[currentBuildShape]
if config then
local height = config.height or 25
local centerCf = currentHrp.CFrame * CFrame.new(0, height, 0)
for _, s in ipairs(multiSparklers) do
if not s.part or not s.part.Parent then continue end
local targetCf = config.update(s.index, t, centerCf)
s.part.AssemblyLinearVelocity = Vector3.zero
s.part.AssemblyAngularVelocity = Vector3.zero
s.part.CFrame = targetCf
end
return
end

if not buildPart or not buildPart.Parent then
if buildToy and buildToy.Parent then
pcall(function() buildToy:Destroy() end)
end
buildToy = nil
buildPart = nil
buildBp = nil
buildBg = nil
task.spawn(function()
if not buildHighRun then return end
local char2 = player.Character
local hrp2 = char2 and char2:FindFirstChild("HumanoidRootPart")
if not hrp2 then return end
buildToy, buildPart, buildBp, buildBg = spawnSingleSparkler(RS, player, hrp2)
end)
return
end

pcall(function()
RS.GrabEvents.SetNetworkOwner:FireServer(buildPart, buildPart.CFrame)
end)

local scale = 1.5
local x, y, z
if currentBuildShape == "Heart" then
x = 16 * math.sin(t) ^ 3
y = 13 * math.cos(t) - 5 * math.cos(2 * t) - 2 * math.cos(3 * t) - math.cos(4 * t)
z = 3
elseif currentBuildShape == "Star" then
local r = 10 * (1 + 0.5 * math.cos(5 * t))
x = r * math.cos(t)
y = r * math.sin(t)
z = 3
elseif currentBuildShape == "Tree" then
local cycle = t % (2 * math.pi)
local height = cycle / (2 * math.pi) * 15
local progress = height / 15
local radius = (1 - progress) * 3.5
local angle = cycle * 10
x = radius * math.cos(angle)
y = height - 7.5
z = radius * math.sin(angle)
elseif currentBuildShape == "Cross" then
local cycle = t % 4
if cycle < 1 then
x = 10 * (cycle - 0.5) * 2
y = 0
elseif cycle < 2 then
x = 0
y = 10 * (cycle - 1.5) * 2
elseif cycle < 3 then
x = -10 * (cycle - 2.5) * 2
y = 0
else
x = 0
y = -10 * (cycle - 3.5) * 2
end
z = 3
elseif currentBuildShape == "Sword" then
local cycle = t % 2
if cycle < 0.8 then
x = 0
y = 15 * cycle - 6
elseif cycle < 1.0 then
x = 8 * (cycle - 0.9) * 10
y = 6
elseif cycle < 1.6 then
x = 0
y = 6 - 10 * (cycle - 1.0)
else
x = 0
y = 0
end
z = 3
elseif currentBuildShape == "Wings" then
local cycle = t % 2
local side = cycle < 1 and 1 or -1
local c = cycle < 1 and cycle or cycle - 1
x = side * (5 + 10 * c)
y = 5 * math.sin(c * math.pi)
z = 3
elseif currentBuildShape == "Crown" then
local n = 5
local cycle = (t % 1) * n
local idx = math.floor(cycle)
local localT = cycle - idx
local angle = idx * (2 * math.pi / n)
local nextAngle = (idx + 1) * (2 * math.pi / n)
local baseRadius = 8
local x1 = baseRadius * math.cos(angle)
local z1 = baseRadius * math.sin(angle)
local x2 = baseRadius * math.cos(nextAngle)
local z2 = baseRadius * math.sin(nextAngle)
x = x1 + (x2 - x1) * localT
z = z1 + (z2 - z1) * localT
y = (localT < 0.5 and localT * 2 or 2 - localT * 2) * 6
elseif currentBuildShape == "Tornado" then
local height = (t % 3) / 3 * 25
local radius = 8 * (1 - height / 25)
local angle = t * 8
x = radius * math.cos(angle)
z = radius * math.sin(angle)
y = height - 12
elseif currentBuildShape == "DNA" then
local cycle = t % 2
local progress = cycle / 2
local spiralHeight = 20
local radius = 5
local turns = 2
local angle = t * 8 + progress * turns * 2 * math.pi
x = radius * math.cos(angle)
z = radius * math.sin(angle)
y = (progress - 0.5) * spiralHeight
elseif currentBuildShape == "Umbrella" then
local n = 5
local cycle = (t % 1) * n
local idx = math.floor(cycle)
local localT = cycle - idx
local angle = idx * (2 * math.pi / n)
local radius = 12 * localT
x = radius * math.cos(angle)
z = radius * math.sin(angle)
y = 0
elseif currentBuildShape == "Planets" then
local cycle = t % 3
local ring = math.floor(cycle) + 1
local localT = cycle - (ring - 1)
local ringRadius = 8 + ring * 5
local angle = t * 2 + localT * 2 * math.pi
x = ringRadius * math.cos(angle)
z = ringRadius * math.sin(angle)
y = 0
else
x = 0; y = 0; z = 0
end

local relPos
if currentBuildShape == "Crown" then
relPos = Vector3.new(x * scale, (y * scale) + 5, z)
elseif currentBuildShape == "Wings" then
relPos = Vector3.new(x * scale, (y * scale) + 15, -5)
else
relPos = Vector3.new(x * scale, (y * scale) + 25, z)
end
buildBp.Position = currentHrp.CFrame:PointToWorldSpace(relPos)
buildBg.CFrame = currentHrp.CFrame
end)
if not ok then
warn("[Sparkler Build Heartbeat Error] " .. tostring(err))
end
end)
end)
else
if buildConnection then
buildConnection:Disconnect()
buildConnection = nil
end
if buildToy then
pcall(function() buildToy:Destroy() end)
buildToy = nil
buildPart = nil
buildBp = nil
buildBg = nil
end
clearMultiSparklers()
local backpack = player:FindFirstChild("Backpack")
if backpack then
for _, v in ipairs(backpack:GetChildren()) do
if v.Name == "FireworkSparkler" then
pcall(function() v:Destroy() end)
end
end
end
end
end)
end
end)

do
local TsunamiSection = Tabs.Fun and Tabs.Fun:AddLeftGroupbox("Tsunami", "waves-horizontal") or nil
if not TsunamiSection then
warn("[OBLIVION] TsunamiSection: AddLeftGroupbox вернул nil — пропускаем")
return
end

tsunamiWall = nil
tsunamiFill = nil
tsunamiSweepConn = nil
tsunamiSweepX = nil
tsunamiStyle = nil
tsunamiRefPart = nil
tsunamiMaxH = nil
tsunamiSaveCF = nil 
tsunamiHalfW = nil 
tsunamiBaseY = nil 
tsunamiBaseZ = nil 
tsunamiSweepAccum = nil 
local TSUNAMI_START_X = -2000 
local TSUNAMI_END_X = 2000 
local TSUNAMI_DEPTH_Z = 4000 
local TSUNAMI_THICK = 30
local TSUNAMI_SPEED = 100 

local function tsunamiHeight()
local v = Options.TsunamiSize and Options.TsunamiSize.Value or 5
return v * 10
end

local function tsunamiGetSNW()
local ge = game:GetService("ReplicatedStorage"):FindFirstChild("GrabEvents")
return ge and ge:FindFirstChild("SetNetworkOwner") or nil
end

local function tsunamiDestroy()
if tsunamiSweepConn then tsunamiSweepConn:Disconnect() tsunamiSweepConn = nil end
if tsunamiWall then pcall(function() tsunamiWall:Destroy() end) tsunamiWall = nil end
if tsunamiFill then pcall(function() tsunamiFill:Destroy() end) tsunamiFill = nil end
if tsunamiStyle == "part" and tsunamiRefPart and tsunamiRefPart.Parent and tsunamiSaveCF then
local snw = tsunamiGetSNW()
if snw then
pcall(function() snw:FireServer(tsunamiRefPart, tsunamiSaveCF) end)
end
end
if tsunamiStyle == "terrain" and tsunamiSweepX then
local terrain = workspace:FindFirstChildOfClass("Terrain")
local w = tsunamiSweepX - TSUNAMI_START_X
local hh = tsunamiMaxH or tsunamiHeight()
if terrain and w > 0 then
local x = TSUNAMI_START_X
while x < tsunamiSweepX do
local x2 = math.min(tsunamiSweepX, x + 100)
pcall(function()
terrain:FillBlock(
CFrame.new((x + x2) / 2, hh / 2, 0),
Vector3.new(x2 - x + 2, hh, TSUNAMI_DEPTH_Z),
Enum.Material.Air
)
end)
x = x2
end
end
end
tsunamiStyle = nil
tsunamiRefPart = nil
tsunamiMaxH = nil
tsunamiSaveCF = nil
tsunamiHalfW = nil
tsunamiBaseY = nil
tsunamiBaseZ = nil
tsunamiSweepAccum = nil
end

local function tsunamiFindWaterRef()
local best = nil
local bestScore = -1
local function consider(p)
local nm = string.lower(p.Name)
local named = p.Material == Enum.Material.Water
or string.find(nm, "water", 1, true) ~= nil
or string.find(nm, "sea", 1, true) ~= nil
or string.find(nm, "ocean", 1, true) ~= nil
or string.find(nm, "lake", 1, true) ~= nil
or string.find(nm, "river", 1, true) ~= nil
if named then
if p.Size.Y > 300 then return end
else
if p.Size.Y > 60 then return end
end
if p.Size.X * p.Size.Z < 2500 then return end
if p.Transparency >= 0.98 then return end
local pos = p.Position
local outside = math.abs(pos.X) > 350 or math.abs(pos.Z) > 350
local giant = p.Size.X > 1000 or p.Size.Z > 1000
if not (named or outside or giant) then return end
local score = p.Size.X * p.Size.Z
if outside or giant then score = score + 1000000000 end
if named then score = score + 500000000 end
if score > bestScore then
best = p
bestScore = score
end
end
for _, obj in ipairs(workspace:GetDescendants()) do
if obj:IsA("BasePart") then
consider(obj)
end
end
if best then return "part", best end
local terrain = workspace:FindFirstChildOfClass("Terrain")
if terrain then
local heights = { -8, -20, -50, 0, 25, 60, 120 }
local radii = { 1000, 2000, 3000 }
for _, rr in ipairs(radii) do
local probes = {
Vector3.new(-rr, 0, 0),
Vector3.new(rr, 0, 0),
Vector3.new(0, 0, -rr),
Vector3.new(0, 0, rr),
}
for _, pr in ipairs(probes) do
for _, yy in ipairs(heights) do
local found = false
pcall(function()
local c = Vector3.new(pr.X, yy, pr.Z)
local r = Region3.new(c - Vector3.new(8, 12, 8), c + Vector3.new(8, 12, 8)):ExpandToGrid(4)
local mats = terrain:ReadVoxels(r, 4)
for x = 1, mats.Size.X do
for y = 1, mats.Size.Y do
for z = 1, mats.Size.Z do
if mats[x][y][z] == Enum.Material.Water then found = true end
end
end
end
end)
if found then return "terrain", terrain end
end
end
end
end
return nil, nil
end

local function tsunamiApplyLook(p)
if tsunamiStyle == "part" and tsunamiRefPart and tsunamiRefPart.Parent then
local ref = tsunamiRefPart
p.Material = ref.Material
p.Color = ref.Color
p.Transparency = ref.Transparency
p.Reflectance = ref.Reflectance
p.CanCollide = false 
for _, ch in ipairs(ref:GetChildren()) do
if ch:IsA("Texture") or ch:IsA("Decal") then
local c = ch:Clone()
c.Parent = p
end
end
end
end

local function tsunamiNewPart()
local p = Instance.new("Part")
p.Name = "MassiveTsunami"
p.Material = Enum.Material.Water
p.BrickColor = BrickColor.new("Deep blue")
p.Transparency = 0.15
p.Anchored = true
p.CanCollide = false 
p.TopSurface = Enum.SurfaceType.Smooth
p.BottomSurface = Enum.SurfaceType.Smooth
p.Parent = workspace
tsunamiApplyLook(p)
return p
end

local function tsunamiStartLocal()
local h = tsunamiHeight()
tsunamiWall = tsunamiNewPart()
tsunamiWall.Size = Vector3.new(TSUNAMI_THICK, h, TSUNAMI_DEPTH_Z)
tsunamiWall.CFrame = CFrame.new(TSUNAMI_START_X, h / 2, 0)
tsunamiFill = tsunamiNewPart()
tsunamiFill.Size = Vector3.new(1, h, TSUNAMI_DEPTH_Z)
tsunamiFill.CFrame = CFrame.new(TSUNAMI_START_X, h / 2, 0)
tsunamiSweepX = TSUNAMI_START_X
tsunamiSweepConn = RunService.Heartbeat:Connect(function(dt)
if not (tsunamiWall and tsunamiFill and tsunamiWall.Parent and tsunamiFill.Parent) then return end
local hy = tsunamiWall.Size.Y
if hy > tsunamiMaxH then tsunamiMaxH = hy end
tsunamiSweepX = math.min(TSUNAMI_END_X, tsunamiSweepX + TSUNAMI_SPEED * dt)
tsunamiWall.CFrame = CFrame.new(tsunamiSweepX, hy / 2, 0)
local width = math.max(1, tsunamiSweepX - TSUNAMI_START_X)
tsunamiFill.Size = Vector3.new(width, hy, TSUNAMI_DEPTH_Z)
tsunamiFill.CFrame = CFrame.new((TSUNAMI_START_X + tsunamiSweepX) / 2, hy / 2, 0)
end)
end

local function tsunamiStart()
tsunamiDestroy()
tsunamiStyle, tsunamiRefPart = tsunamiFindWaterRef()
local h = tsunamiHeight()
tsunamiMaxH = h
if tsunamiStyle == "terrain" then
tsunamiSweepX = TSUNAMI_START_X
local terrain = workspace:FindFirstChildOfClass("Terrain")
tsunamiSweepConn = RunService.Heartbeat:Connect(function(dt)
if not terrain then return end
local prevX = tsunamiSweepX
tsunamiSweepX = math.min(TSUNAMI_END_X, tsunamiSweepX + TSUNAMI_SPEED * dt)
local w = tsunamiSweepX - prevX
if w <= 0 then return end
local hh = tsunamiHeight()
if hh > tsunamiMaxH then tsunamiMaxH = hh end
pcall(function()
terrain:FillBlock(
CFrame.new((prevX + tsunamiSweepX) / 2, hh / 2, 0),
Vector3.new(w + 2, hh, TSUNAMI_DEPTH_Z),
Enum.Material.Water
)
end)
end)
return
end
if tsunamiStyle == "part" then
local snw = tsunamiGetSNW()
local wp = tsunamiRefPart
if snw and wp and wp.Parent then
tsunamiSaveCF = wp.CFrame
tsunamiHalfW = wp.Size.X / 2
tsunamiBaseY = wp.Position.Y
tsunamiBaseZ = wp.Position.Z
tsunamiSweepX = TSUNAMI_START_X
tsunamiSweepAccum = 0
pcall(function()
snw:FireServer(wp, CFrame.new(TSUNAMI_START_X - tsunamiHalfW, tsunamiBaseY, tsunamiBaseZ) * tsunamiSaveCF.Rotation)
end)
tsunamiSweepConn = RunService.Heartbeat:Connect(function(dt)
if not (tsunamiStyle == "part" and tsunamiRefPart and tsunamiRefPart.Parent) then return end
tsunamiSweepAccum = tsunamiSweepAccum + dt
if tsunamiSweepAccum < 0.1 then return end
local step = math.min(0.25, tsunamiSweepAccum)
tsunamiSweepAccum = 0
tsunamiSweepX = math.min(TSUNAMI_END_X, tsunamiSweepX + TSUNAMI_SPEED * step)
local cf = CFrame.new(tsunamiSweepX - tsunamiHalfW, tsunamiBaseY, tsunamiBaseZ) * tsunamiSaveCF.Rotation
pcall(function() snw:FireServer(tsunamiRefPart, cf) end)
end)
task.delay(0.7, function()
if not (tsunamiStyle == "part" and tsunamiRefPart and tsunamiRefPart.Parent and tsunamiSaveCF) then return end
local want = TSUNAMI_START_X - (tsunamiHalfW or 0)
if math.abs(tsunamiRefPart.Position.X - want) > 600 then
if tsunamiSweepConn then tsunamiSweepConn:Disconnect() tsunamiSweepConn = nil end
local s2 = tsunamiGetSNW()
if s2 then pcall(function() s2:FireServer(tsunamiRefPart, tsunamiSaveCF) end) end
tsunamiStyle = "partLocal"
tsunamiStartLocal()
pcall(function()
Library:Notify(L("Цунами: сервер не принял волну — включена локальная (видишь только ты)", "Tsunami: server rejected the wave — local one enabled (only you see it)"), 4)
end)
end
end)
return
end
end
tsunamiStartLocal()
end

TsunamiSection:AddToggle("EnableTsunami", {
Text = "Tsunami",
Default = false,
Tooltip = L("Двигает НАСТОЯЩУЮ воду за картой через всю карту: текстура та самая, видно весь сервер. Сквозь локальную волну можно проходить", "Moves the REAL out-of-map water across the map: same texture, visible to the whole server"),
Callback = function(Value)
if Value then
pcall(tsunamiStart)
if tsunamiStyle == "terrain" then
Library:Notify(L("Цунами: настоящая терра-вода заливает карту (видишь только ты)", "Tsunami: real terrain water floods the map (only you see it)"), 3)
elseif tsunamiStyle == "part" then
Library:Notify(L("Цунами: настоящая вода за картой едет по карте — видно всему серверу", "Tsunami: real out-of-map water sweeps the map — visible to the whole server"), 4)
else
Library:Notify(L("Цунами: вода за картой не найдена — локальный стандартный вид", "Tsunami: out-of-map water not found — local default look"), 3)
end
else
pcall(tsunamiDestroy)
end
end
})

TsunamiSection:AddSlider("TsunamiSize", {
Text = "Tsunami Height",
Default = 5,
Min = 1,
Max = 200,
Rounding = 1,
Compact = false,
Tooltip = L("Высота волны (Y)", "Wave Height (Y)"),
Callback = function(Value)
if tsunamiWall and tsunamiFill and tsunamiWall.Parent and tsunamiFill.Parent then
local h = Value * 10
tsunamiWall.Size = Vector3.new(TSUNAMI_THICK, h, TSUNAMI_DEPTH_Z)
tsunamiWall.CFrame = CFrame.new(tsunamiWall.Position.X, h / 2, 0)
local width = math.max(1, tsunamiFill.Size.X)
tsunamiFill.Size = Vector3.new(width, h, TSUNAMI_DEPTH_Z)
tsunamiFill.CFrame = CFrame.new(tsunamiFill.Position.X, h / 2, 0)
end
end
})

TsunamiSection:AddButton({
Text = "Make Tsunami",
Tooltip = L("Порт Resonance: 12 сюрикенов уносят кусок океана на гигантский офсет — вода поднимается и идёт стеной", "Resonance port: 12 shurikens drag the ocean chunk with a giant offset — water rises and moves as a wall"),
Func = function()
task.spawn(function()
local map = workspace:FindFirstChild("Map")
local tweened = map and map:FindFirstChild("AlwaysHereTweenedObjects")
local ocean = tweened and tweened:FindFirstChild("Ocean")
local obj = ocean and ocean:FindFirstChild("Object")
local om = obj and obj:FindFirstChild("ObjectModel")
local children = om and om:GetChildren()
local refPart = children and children[22]
if not refPart then
Library:Notify(L("Tsunami: кусок океана не найден", "Tsunami: ocean chunk not found"), 3)
return
end
if not OBV_SpawnToy or not OBV_SNO then
Library:Notify(L("Tsunami: хелперы не готовы", "Tsunami: helpers not ready"), 3)
return
end
local rs = game:GetService("ReplicatedStorage")
local pe = rs:FindFirstChild("PlayerEvents")
local sticky = pe and pe:FindFirstChild("StickyPartEvent")
if not sticky then
Library:Notify(L("Tsunami: ремоуты не найдены", "Tsunami: remotes not found"), 3)
return
end
local list = {}
for _ = 1, 12 do
local shur = OBV_SpawnToy("NinjaShuriken")
if shur then
local sp = shur:FindFirstChild("StickyPart") or shur:WaitForChild("StickyPart", 0.25)
local sop = shur:FindFirstChild("SoundPart") or shur:WaitForChild("SoundPart", 0.25)
if sp and sop and OBV_SNO(sop) then
sop.CFrame = CFrame.new(0, 1e10, 0)
table.insert(list, shur)
end
end
end
local n = 0
for _, shur in ipairs(list) do
local sp = shur:FindFirstChild("StickyPart")
if shur.Parent and shur:FindFirstChild("SoundPart") and sp then
pcall(function() sticky:FireServer(sp, refPart, CFrame.new(4294967296, 2147483648, 2147483648)) end)
n = n + 1
end
end
Library:Notify(L("Tsunami: волна запущена (" .. n .. "/12)", "Tsunami: wave launched (" .. n .. "/12)"), 3)
end)
end,
})
end

do
local BrickHouseSection = Tabs.Fun and Tabs.Fun:AddLeftGroupbox("Brick House", "target") or nil
if not BrickHouseSection then
warn("[OBLIVION] BrickHouseSection: AddLeftGroupbox вернул nil — пропускаем")
else
local brickHouseConn = nil
local brickHouseModule = nil
local brickHouseOldCast = nil
local brickHouseOldCastThrough = nil

BrickHouseSection:AddToggle("EnableBrickHouse", {
Text = "Silent Aim (Brick House)",
Default = false,
Tooltip = L("Сайлент-аим: хук Raycast cast/castThrough, пули летят в ближайшего врага в поле зрения", "Silent aim: hooks Raycast cast/castThrough, shots redirect to closest visible enemy"),
Callback = function(Value)
if Value then
pcall(function()
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")

local camera = Workspace.CurrentCamera

local function GetClosestPlayer()
local closestDistance = math.huge
local closest = nil
local myTeam = LocalPlayer:GetAttribute("Team")
local lchar = LocalPlayer.Character
if not lchar then return end
local lhrp = lchar:FindFirstChild("Head")
if not lhrp then return end
for _, v in Players:GetPlayers() do
if v == LocalPlayer then continue end
local char = v.Character
if not char then continue end
if char:GetAttribute("Dead") then continue end
if v:GetAttribute("Team") == myTeam then continue end
local hrp = char:FindFirstChild("Head")
if not hrp then continue end
local screenPos, onScreen = camera:WorldToViewportPoint(hrp.Position)
if not onScreen then continue end
local dist = (Vector2.new(screenPos.X, screenPos.Y) - camera.ViewportSize / 2).Magnitude
if dist < closestDistance then
closestDistance = dist
closest = hrp
end
end
return closest
end

_G._silentAimData = {
target = nil,
localPlayer = LocalPlayer,
camera = camera,
oldCast = nil,
oldCastThrough = nil,
}

brickHouseConn = RunService.RenderStepped:Connect(function()
_G._silentAimData.target = GetClosestPlayer()
end)

local raycastMODULE

for _, obj in getgc(true) do
if type(obj) == "table" and rawget(obj, "cast") and rawget(obj, "castThrough") then
if type(rawget(obj, "cast")) == "function" and type(rawget(obj, "castThrough")) == "function" then
raycastMODULE = obj
break
end
end
end

if not raycastMODULE then
local success, result = pcall(function()
return require(game:GetService("ReplicatedStorage"):WaitForChild("Shared"):WaitForChild("Raycast"))
end)
if success and type(result) == "table" and type(result.cast) == "function" and type(result.castThrough) == "function" then
raycastMODULE = result
end
end

if not raycastMODULE then
warn("no raycast module found, script probably wont work")
return
end

local origiENV = getfenv()
local newENV = {}
setmetatable(newENV, {
__index = function(_, key)
if key == "getgenv" or key == "hookfunction" or key == "hookfunc" or key == "replaceclosure" or key == "oth" or key == "debug" then
return nil
end
return origiENV[key]
end
})

newENV._silentAimData = _G._silentAimData
newENV.typeof = typeof
newENV.Vector3 = Vector3
newENV.warn = warn
newENV.select = select

local cast = loadstring([[
    return function(origin, direction, ...)
        local data = _silentAimData
        local t = data.target
        if t and typeof(direction) == "Vector3" then
            local lchar = data.localPlayer.Character
            if lchar and lchar:FindFirstChild("Head") then
                if (origin - lchar.Head.Position).Magnitude < 12 then
                    direction = (t.Position - origin)
                end
            end
        end
        return data.oldCast(origin, direction, ...)
    end
]])

local castthrough = loadstring([[
    return function(origin, direction, ...)
        local data = _silentAimData
        local t = data.target
        if t and typeof(direction) == "Vector3" then
            local lchar = data.localPlayer.Character
            if lchar and lchar:FindFirstChild("Head") then
                if (origin - lchar.Head.Position).Magnitude < 12 then
                    direction = (t.Position - origin)
                end
            end
        end
        return data.oldCastThrough(origin, direction, ...)
    end
]])

pcall(function()
setfenv(cast, newENV)
end)
pcall(function()
setfenv(castthrough, newENV)
end)

local castHook = cast()
local castThroughHook = castthrough()

_G._silentAimData.oldCast = hookfunction(raycastMODULE.cast, castHook)
_G._silentAimData.oldCastThrough = hookfunction(raycastMODULE.castThrough, castThroughHook)

brickHouseModule = raycastMODULE
brickHouseOldCast = _G._silentAimData.oldCast
brickHouseOldCastThrough = _G._silentAimData.oldCastThrough
end)
else
pcall(function()
if brickHouseConn then
brickHouseConn:Disconnect()
brickHouseConn = nil
end
if brickHouseModule then
if brickHouseOldCast then
pcall(function() hookfunction(brickHouseModule.cast, brickHouseOldCast) end)
end
if brickHouseOldCastThrough then
pcall(function() hookfunction(brickHouseModule.castThrough, brickHouseOldCastThrough) end)
end
end
brickHouseModule = nil
brickHouseOldCast = nil
brickHouseOldCastThrough = nil
_G._silentAimData = nil
end)
end
end
})
end
end

pcall(function() ThemeManager:SetLibrary(Library) end)
pcall(function()
ThemeManager:SetDefaultTheme({
BackgroundColor = Color3.fromRGB(8, 12, 22),
MainColor = Color3.fromRGB(14, 20, 36),
AccentColor = Color3.fromRGB(0, 145, 255),
OutlineColor = Color3.fromRGB(30, 44, 70),
FontColor = Color3.fromRGB(237, 233, 248),
FontFace = "GothamMedium",
})
end)
local _smWaitT0 = tick()
while not (saveDone and ThemeManager and SaveManager) and tick() - _smWaitT0 < 20 do
task.wait(0.1)
end
if not SaveManager then
warn("[Oblivion] SaveManager не загрузился за 20 сек — конфиги недоступны")
end
pcall(function() SaveManager:SetLibrary(Library) end)
_oblivionCfgLoading = true

if SaveManager then
local HttpService = game:GetService("HttpService")
SaveManager.LoadJSON = function(self, Content)
local ok, Decoded = pcall(HttpService.JSONDecode, HttpService, Content)
if not ok or typeof(Decoded) ~= "table" or typeof(Decoded.objects) ~= "table" then
warn("[Oblivion] Config load error: invalid JSON")
return false, "Failed to decode config data"
end

local Ignore = self.Ignore

local queue = {}

pcall(function()
if Library.KeybindFrame and typeof(Decoded.keybindMenu) == "table" then
local KF = Decoded.keybindMenu
Library.KeybindFrame.Visible = (KF.visible == true)
local KeybindMenuToggle = Library.Options and Library.Options.KeybindMenuOpen
if KeybindMenuToggle then
KeybindMenuToggle:SetValue(KF.visible == true)
end
end
end)

for _, Data in ipairs(Decoded.objects) do
local Index = Data and Data.idx
local ElType = Data and Data.type
if not ElType then continue end
if Ignore and Ignore[Index] then continue end
table.insert(queue, { Index = Index, ElType = ElType, Data = Data })
end

task.spawn(function()
for _, Item in ipairs(queue) do
local Index, ElType, Data = Item.Index, Item.ElType, Item.Data
print("[Oblivion] cfg: applying " .. tostring(ElType) .. " '" .. tostring(Index) .. "'")
local okEl, errEl = pcall(function()
if ElType == "Toggle" then
local Toggle = Library.Toggles and Library.Toggles[Index]
if not Toggle then return end
if Toggle.Value == Data.value then return end
Toggle:SetValue(Data.value)
elseif ElType == "Slider" then
local Slider = Library.Options and Library.Options[Index]
if not Slider then return end
local v = tonumber(Data.value)
if not v then return end
if Slider.Min and v < Slider.Min then v = Slider.Min end
if Slider.Max and v > Slider.Max then v = Slider.Max end
if Slider.Value == v then return end
Slider:SetValue(v)
elseif ElType == "Dropdown" then
if Index == "LanguageDropdown" then
warn("[Oblivion] cfg: 'LanguageDropdown' skipped (unloads script on change)")
return
end
local Dropdown = Library.Options and Library.Options[Index]
if not Dropdown then return end
local values = Dropdown.Values
if typeof(values) == "table" then
if typeof(Data.value) == "table" then
local filtered = {}
for _, v in ipairs(Data.value) do
if table.find(values, v) then
table.insert(filtered, v)
end
end
Dropdown:SetValue(filtered)
else
if table.find(values, Data.value) then
Dropdown:SetValue(Data.value)
else
warn("[Oblivion] Config: dropdown value '" .. tostring(Data.value) .. "' for '" .. tostring(Index) .. "' not found — skipped")
end
end
else
Dropdown:SetValue(Data.value)
end
elseif ElType == "ColorPicker" then
local Picker = Library.Options and Library.Options[Index]
if not Picker then return end
Picker:SetValueRGB(Color3.fromHex(Data.value), Data.transparency)
elseif ElType == "KeyPicker" then
local Key = Library.Options and Library.Options[Index]
if not Key then return end
Key:SetValue({ Data.key, Data.mode, Data.modifiers })
if Data.mode == "Toggle" and Data.toggled ~= nil then
Key.Toggled = Data.toggled
if Key.Update then Key:Update() end
end
elseif ElType == "Input" then
local Input = Library.Options and Library.Options[Index]
if not Input or typeof(Data.text) ~= "string" then return end
if Input.Value == Data.text then return end
Input:SetValue(Data.text)
else
local Element = Library.Options and Library.Options[Index]
if Element and Element.SetValue and Data.value ~= nil then
Element:SetValue(Data.value)
end
end
end)
if not okEl then
warn("[Oblivion] cfg: FAILED '" .. tostring(Index) .. "' (" .. tostring(ElType) .. "): " .. tostring(errEl))
else
print("[Oblivion] cfg: applied OK '" .. tostring(Index) .. "'")
end
end
print("[Oblivion] cfg: очередь конфига полностью применена")
end)
return true
end
end 
pcall(function() SaveManager:IgnoreThemeSettings() end)
pcall(function() SaveManager:SetIgnoreIndexes({
"MenuKeybind",
"TargetLoopPlayer",
"TargetLoopServer",
"BloodmanLoopPlayer",
"BloodmanLoopServer",
"KillAura",
"VoidAura",
"FlingAura",
"SpinAura",
"TelekinesisAura",
"ClickAura",
"DestroyAntiKickAura",
"AntiBananaAura",
"EnableCombatGrab",
"EnableKickGrab",
"EnableKillGrab",
"EnableFlingGrab",
"EnableSpinGrab",
"EnableCounterMode",
"EnableBombDrone",
"EnableTsunami",
"EnableLoopLegs",
"EnableAntiKickShuriken",
"EnableAntiKickItem",
"ControlCharactersToggle",
"EnableTriggerBot",
"EnableLockTap",
"EnableLineLag",
"EnablePacketLag",
"EnableGrabModifier",
"GrabModifierTypes",
"EnableAurasMaster",
"AuraTypes",
"FG_Enabled",
}) end)
pcall(function() ThemeManager:SetFolder("Oblivion_FTAP") end)
pcall(function() SaveManager:SetFolder("Oblivion_FTAP/Configs") end)
pcall(function() SaveManager:BuildConfigSection(Tabs.Settings) end)
pcall(function() ThemeManager:ApplyToTab(Tabs.Settings) end)
pcall(function() SaveManager:LoadAutoloadConfig() end)

task.delay(2, function()
_oblivionCfgLoading = false
_lagAutoBlock = false
pcall(function() if Toggles.EnableLineLag and Toggles.EnableLineLag.Value then Toggles.EnableLineLag:SetValue(false) end end)
pcall(function() if Toggles.EnablePacketLag and Toggles.EnablePacketLag.Value then Toggles.EnablePacketLag:SetValue(false) end end)
pcall(function() stopLineLag() end)
pcall(function() stopPacketLag() end)
pcall(function() tgtStopLineLag() end)

end)
end)

Library:Notify("Oblivion Loaded!", 3)

do
local VirtualInputManager = game:GetService("VirtualInputManager")
local UserInputService = game:GetService("UserInputService")
local isMobile = UserInputService.TouchEnabled and not UserInputService.MouseEnabled

local mobileKeyboardGui = nil
local keyboardVisible = false

local keyMap = {
A = Enum.KeyCode.A, B = Enum.KeyCode.B, C = Enum.KeyCode.C, D = Enum.KeyCode.D,
E = Enum.KeyCode.E, F = Enum.KeyCode.F, G = Enum.KeyCode.G, H = Enum.KeyCode.H,
I = Enum.KeyCode.I, J = Enum.KeyCode.J, K = Enum.KeyCode.K, L = Enum.KeyCode.L,
M = Enum.KeyCode.M, N = Enum.KeyCode.N, O = Enum.KeyCode.O, P = Enum.KeyCode.P,
Q = Enum.KeyCode.Q, R = Enum.KeyCode.R, S = Enum.KeyCode.S, T = Enum.KeyCode.T,
U = Enum.KeyCode.U, V = Enum.KeyCode.V, W = Enum.KeyCode.W, X = Enum.KeyCode.X,
Y = Enum.KeyCode.Y, Z = Enum.KeyCode.Z,
["1"] = Enum.KeyCode.One, ["2"] = Enum.KeyCode.Two, ["3"] = Enum.KeyCode.Three,
["4"] = Enum.KeyCode.Four, ["5"] = Enum.KeyCode.Five, ["6"] = Enum.KeyCode.Six,
["7"] = Enum.KeyCode.Seven, ["8"] = Enum.KeyCode.Eight, ["9"] = Enum.KeyCode.Nine,
["0"] = Enum.KeyCode.Zero,
F1 = Enum.KeyCode.F1, F2 = Enum.KeyCode.F2, F3 = Enum.KeyCode.F3, F4 = Enum.KeyCode.F4,
F5 = Enum.KeyCode.F5, F6 = Enum.KeyCode.F6, F7 = Enum.KeyCode.F7, F8 = Enum.KeyCode.F8,
Space = Enum.KeyCode.Space, Shift = Enum.KeyCode.LeftShift,
LMB = "MouseButton1", RMB = "MouseButton2",
}

local rows = {
{ keys = { "1","2","3","4","5","6","7","8","9","0" }, leftIndent = 0 },
{ keys = { "Q","W","E","R","T","Y","U","I","O","P" }, leftIndent = 0 },
{ keys = { "A","S","D","F","G","H","J","K","L" }, leftIndent = 0.5 },  
{ keys = { "Z","X","C","V","B","N","M" }, leftIndent = 1.0 },         
{ keys = { "F1","F2","F3","F4","F5","F6","F7","F8" }, leftIndent = 0 },
{ keys = { "Shift","Space","LMB","RMB" }, leftIndent = 0 },
}

local function pressKey(keyCode)
pcall(function()
if keyCode == "MouseButton1" then
VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 1)
task.wait(0.05)
VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 1)
elseif keyCode == "MouseButton2" then
VirtualInputManager:SendMouseButtonEvent(0, 0, 1, true, game, 1)
task.wait(0.05)
VirtualInputManager:SendMouseButtonEvent(0, 0, 1, false, game, 1)
elseif typeof(keyCode) == "EnumItem" then
VirtualInputManager:SendKeyEvent(true, keyCode, false, game)
task.wait(0.05)
VirtualInputManager:SendKeyEvent(false, keyCode, false, game)
end
end)
end

local function createMobileKeyboard()
if mobileKeyboardGui then return mobileKeyboardGui end

local sg = Instance.new("ScreenGui")
sg.Name = "OblivionMobileKeyboard"
sg.ResetOnSpawn = false
sg.IgnoreGuiInset = true
sg.DisplayOrder = 99999
sg.Parent = (gethui and gethui()) or game:GetService("CoreGui")

local toggleBtn = Instance.new("TextButton")
toggleBtn.Name = "ToggleBtn"
toggleBtn.Size = UDim2.new(0, 90, 0, 50)
toggleBtn.Position = UDim2.new(1, -100, 1, -60)
toggleBtn.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
toggleBtn.BackgroundTransparency = 0.2
toggleBtn.BorderSizePixel = 0
toggleBtn.Text = "⌨"
toggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
toggleBtn.Font = Enum.Font.GothamBold
toggleBtn.TextScaled = true
toggleBtn.Parent = sg

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 8)
toggleCorner.Parent = toggleBtn

local kbFrame = Instance.new("Frame")
kbFrame.Name = "KeyboardFrame"
kbFrame.Size = UDim2.new(1, 0, 0, 280)
kbFrame.Position = UDim2.new(0, 0, 1, 0)  
kbFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
kbFrame.BackgroundTransparency = 0.15
kbFrame.BorderSizePixel = 0
kbFrame.Visible = false
kbFrame.Parent = sg

local kbCorner = Instance.new("UICorner")
kbCorner.CornerRadius = UDim.new(0, 12)
kbCorner.Parent = kbFrame

local kbPadding = Instance.new("UIPadding")
kbPadding.PaddingTop = UDim.new(0, 8)
kbPadding.PaddingBottom = UDim.new(0, 8)
kbPadding.PaddingLeft = UDim.new(0, 8)
kbPadding.PaddingRight = UDim.new(0, 8)
kbPadding.Parent = kbFrame

local keySize = 40        
local keyGap = 4          
local rowHeight = keySize
local rowSpacing = 4
local maxKeysInRow = 10
local totalRowWidth = maxKeysInRow * keySize + (maxKeysInRow - 1) * keyGap  

local yOffset = 0

for rowIdx, row in ipairs(rows) do
local keys = row.keys
local indent = row.leftIndent or 0
local numKeys = #keys
local rowWidth = numKeys * keySize + (numKeys - 1) * keyGap
local leftOffsetPx = indent * (keySize + keyGap) / 2

local rowFrame = Instance.new("Frame")
rowFrame.Name = "Row" .. rowIdx
rowFrame.Size = UDim2.new(0, totalRowWidth, 0, rowHeight)
rowFrame.Position = UDim2.new(0.5, -totalRowWidth/2, 0, yOffset)  
rowFrame.BackgroundTransparency = 1
rowFrame.Parent = kbFrame

local currentX = leftOffsetPx  
for i, keyText in ipairs(keys) do
local keyBtn = Instance.new("TextButton")
keyBtn.Name = "Key_" .. keyText
local isWide = (keyText == "Space" or keyText == "Shift")
local widthMult = isWide and 2.5 or 1
local keyWidth = keySize * widthMult + (widthMult - 1) * keyGap

keyBtn.Size = UDim2.new(0, keyWidth, 0, keySize)
keyBtn.Position = UDim2.new(0, currentX, 0, 0)
keyBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
keyBtn.BackgroundTransparency = 0.1
keyBtn.BorderSizePixel = 0
keyBtn.Text = keyText
keyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
keyBtn.Font = Enum.Font.GothamBold
keyBtn.TextScaled = true
keyBtn.Parent = rowFrame

local keyCorner = Instance.new("UICorner")
keyCorner.CornerRadius = UDim.new(0, 6)
keyCorner.Parent = keyBtn

keyBtn.MouseButton1Down:Connect(function()
keyBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 120)
end)
keyBtn.MouseButton1Up:Connect(function()
keyBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
end)
keyBtn.MouseLeave:Connect(function()
keyBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
end)

keyBtn.Activated:Connect(function()
local keyCode = keyMap[keyText]
if keyCode then
pressKey(keyCode)
end
end)

currentX = currentX + keyWidth + keyGap
end

yOffset = yOffset + rowHeight + rowSpacing
end

kbFrame.Size = UDim2.new(1, 0, 0, yOffset + 16)

local closeBtn = Instance.new("TextButton")
closeBtn.Name = "CloseBtn"
closeBtn.Size = UDim2.new(0, 40, 0, 32)
closeBtn.Position = UDim2.new(1, -48, 0, 8)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeBtn.BorderSizePixel = 0
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.Font = Enum.Font.GothamBold
closeBtn.TextScaled = true
closeBtn.Parent = kbFrame

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 6)
closeCorner.Parent = closeBtn

local function setKeyboardVisible(visible)
keyboardVisible = visible
kbFrame.Visible = true
local targetY = visible and (-kbFrame.Size.Y.Offset) or 0
pcall(function()
local TweenService = game:GetService("TweenService")
local tween = TweenService:Create(
kbFrame,
TweenInfo.new(0.25, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
{ Position = UDim2.new(0, 0, 1, targetY) }
)
tween:Play()
end)
task.delay(0.3, function()
if keyboardVisible == visible then
kbFrame.Position = UDim2.new(0, 0, 1, targetY)
end
end)
toggleBtn.Text = visible and "✕" or "⌨"
end

toggleBtn.Activated:Connect(function()
setKeyboardVisible(not keyboardVisible)
end)
closeBtn.Activated:Connect(function()
setKeyboardVisible(false)
end)

local dragging = false
local dragStart, startPos
toggleBtn.InputBegan:Connect(function(input)
if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
dragging = true
dragStart = input.Position
startPos = toggleBtn.Position
end
end)
toggleBtn.InputChanged:Connect(function(input)
if dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
local delta = input.Position - dragStart
local newPos = UDim2.new(
startPos.X.Scale, startPos.X.Offset + delta.X,
startPos.Y.Scale, startPos.Y.Offset + delta.Y
)
local x = math.clamp(newPos.X.Offset, 0, sg.AbsoluteSize.X - 90)
local y = math.clamp(newPos.Y.Offset, 0, sg.AbsoluteSize.Y - 50)
toggleBtn.Position = UDim2.new(0, x, 0, y)
end
end)
toggleBtn.InputEnded:Connect(function(input)
if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
dragging = false
end
end)

mobileKeyboardGui = sg
return sg
end

pcall(createMobileKeyboard)

MiscSection:AddToggle("EnableMobileKeyboard", {
Text = L("Mobile Keyboard (кнопка ⌨)", "Mobile Keyboard (⌨ button)"),
Default = isMobile,  
Tooltip = L("Показать кнопку ⌨ внизу справа — открывает виртуальную клавиатуру для биндов", "Show ⌨ button bottom right — opens virtual keyboard for binds"),
})

Toggles.EnableMobileKeyboard:OnChanged(function()
if mobileKeyboardGui then
local toggleBtn = mobileKeyboardGui:FindFirstChild("ToggleBtn")
if toggleBtn then
toggleBtn.Visible = Toggles.EnableMobileKeyboard.Value
end
if not Toggles.EnableMobileKeyboard.Value then
local kbFrame = mobileKeyboardGui:FindFirstChild("KeyboardFrame")
if kbFrame then kbFrame.Visible = false end
keyboardVisible = false
end
end
end)

if Toggles.EnableMobileKeyboard then
if mobileKeyboardGui then
local toggleBtn = mobileKeyboardGui:FindFirstChild("ToggleBtn")
if toggleBtn then
toggleBtn.Visible = Toggles.EnableMobileKeyboard.Value
end
end
end
end

do
local HouseDestroySection = nil
local ok, err = pcall(function()
HouseDestroySection = Tabs.Fun:AddRightGroupbox("Destroy Houses", "house")
end)
if not ok then print("[DEBUG] Destroy Houses error: " .. tostring(err)) end
if not HouseDestroySection then print("[DEBUG] HouseDestroySection is nil!") return end

local HD = {}
HD.plotColors = {
["Plot1"] = "Зелёный",
["Plot2"] = "Красный",
["Plot3"] = "Жёлтый",
["Plot4"] = "Синий",
["Plot5"] = "Фиолетовый",
}
HD.colorToPlot = {}
for k,v in pairs(HD.plotColors) do HD.colorToPlot[v] = k end
HD.colorList = {}
for _, color in pairs(HD.plotColors) do table.insert(HD.colorList, color) end
table.sort(HD.colorList)

HouseDestroySection:AddDropdown("HouseTarget", {
Text = L("Дом", "House"),
Values = HD.colorList,
Default = HD.colorList[1],
Multi = false,
Tooltip = L("Какой дом разнести", "Which house to destroy"),
Callback = function() end,
})

HD.getSpawnRF = function()
local mt = game:GetService("ReplicatedStorage"):FindFirstChild("MenuToys")
return mt and (mt:FindFirstChild("SpawnToyRemoteFunction") or mt:WaitForChild("SpawnToyRemoteFunction", 5))
end
HD.getDestroyRE = function()
local mt = game:GetService("ReplicatedStorage"):FindFirstChild("MenuToys")
return mt and (mt:FindFirstChild("DestroyToy") or mt:WaitForChild("DestroyToy", 5))
end

HouseDestroySection:AddSlider("HouseDestroyDuration", {
Text = L("Длительность разноса (сек)", "Destroy Duration (sec)"),
Default = 10,
Min = 3,
Max = 60,
Rounding = 0,
Compact = false,
Tooltip = L("Сколько секунд decoy будет разносить предметы в доме", "Seconds decoy will destroy items in house"),
})

HouseDestroySection:AddButton({
Text = L("Разнести дом", "Destroy House"),
Tooltip = L("Спавнит YouDecoy, забирает ownership и флингает все предметы в выбранном доме", "Spawns YouDecoy, takes ownership and flings all items in selected house"),
Func = function()
local function destroyPlot(plotName, colorName)
local spawnRF = HD.getSpawnRF()
local destroyRE = HD.getDestroyRE()
local GE = game:GetService("ReplicatedStorage"):FindFirstChild("GrabEvents")
local sno = GE and GE:FindFirstChild("SetNetworkOwner")
local destroyGL = GE and GE:FindFirstChild("DestroyGrabLine")
if not spawnRF then return end

local plot = workspace:FindFirstChild("PlotItems") and workspace.PlotItems:FindFirstChild(plotName)
if not plot then return end

local myChar = LocalPlayer.Character
local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
if not myHRP then return end

local inv = workspace:FindFirstChild(LocalPlayer.Name .. "SpawnedInToys")
if not inv then return end

for _, old in ipairs(inv:GetChildren()) do
if old.Name == "YouDecoy" or old.Name == "blablabalbb1" then
pcall(function() destroyRE:FireServer(old) end)
end
end

task.spawn(function() pcall(function() spawnRF:InvokeServer("YouDecoy", myHRP.CFrame, Vector3.zero) end) end)
local clone = nil
local t0 = tick()
while not clone and tick() - t0 < 3 do
clone = inv:FindFirstChild("YouDecoy")
task.wait(0.05)
end
if not clone then return end
clone.Name = "blablabalbb1"

local decoyHRP = clone:FindFirstChild("HumanoidRootPart")
local decoyHead = clone:FindFirstChild("Head")
if not decoyHRP then return end

if sno then
local attempts = 0
repeat
pcall(function() sno:FireServer(decoyHRP, decoyHRP.CFrame) end)
task.wait(0.05)
attempts = attempts + 1
until (decoyHead and decoyHead:FindFirstChild("PartOwner") and decoyHead.PartOwner.Value == LocalPlayer.Name) or attempts > 30
end

local bodyPos = Instance.new("BodyPosition")
bodyPos.P = 90000
bodyPos.D = 100
bodyPos.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
bodyPos.Parent = decoyHRP

local bodyGyro = Instance.new("BodyGyro")
bodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
bodyGyro.P = 20000
bodyGyro.D = 200
bodyGyro.Parent = decoyHRP

if destroyGL then pcall(function() destroyGL:FireServer(decoyHRP) end) end

local function flingObject(object)
if not object or not object.PrimaryPart then return end
local targetPos = object.PrimaryPart.Position
bodyPos.Position = targetPos + Vector3.new(0, 2, 0)
while decoyHRP and decoyHRP.Parent and object and object.PrimaryPart do
local distance = (decoyHRP.Position - object.PrimaryPart.Position).Magnitude
if distance < 3 then break end
task.wait()
end
if decoyHRP and decoyHRP.Parent then
local direction = (decoyHRP.Position - object.PrimaryPart.Position).Unit
decoyHRP.AssemblyLinearVelocity = direction * 800
task.spawn(function()
for _ = 1, 15 do
task.wait()
decoyHRP.AssemblyAngularVelocity = Vector3.new(
math.random(-100, 100) * 100,
math.random(-100, 100) * 100,
math.random(-100, 100) * 100
)
end
end)
task.wait(0.3)
end
end

local startTime = tick()
local duration = Options.HouseDestroyDuration and Options.HouseDestroyDuration.Value or 10
while tick() - startTime < duration do
local foundAny = false
for _, v in ipairs(plot:GetChildren()) do
if v.Name ~= "RemoveLagToys" and v.Name ~= "ToysLimitNum" and v.PrimaryPart then
flingObject(v)
foundAny = true
if tick() - startTime >= duration then break end
end
end
if not foundAny then break end
task.wait(0.1)
end

task.wait(2)
if bodyPos then bodyPos:Destroy() end
if bodyGyro then bodyGyro:Destroy() end
pcall(function() destroyRE:FireServer(clone) end)
Library:Notify("Destroy House: " .. colorName .. " — готово!", 2)
end

local colorName = Options.HouseTarget and Options.HouseTarget.Value or "Зелёный"
local plotName = HD.colorToPlot[colorName] or "Plot1"
Library:Notify("Destroy House: " .. colorName .. " — погнали!", 2)
task.spawn(function() destroyPlot(plotName, colorName) end)
end,
})

do
local MapBreakGroup = Tabs.Fun:AddLeftGroupbox("Map Break (Shuriken)", "bomb")

local MB = {}
MB.RS = game:GetService("ReplicatedStorage")
MB.WS = game:GetService("Workspace")
MB.Players = game:GetService("Players")
MB.LocalPlayer = MB.Players.LocalPlayer

MB.spawnRF  = MB.RS:WaitForChild("MenuToys"):WaitForChild("SpawnToyRemoteFunction", 10)
MB.destroyToy = MB.RS:WaitForChild("MenuToys"):FindFirstChild("DestroyToy") or MB.RS:WaitForChild("MenuToys"):WaitForChild("DestroyToy", 10)
MB.GE = MB.RS:WaitForChild("GrabEvents", 10)
MB.SetNetworkOwner = MB.GE:WaitForChild("SetNetworkOwner", 10)
MB.DestroyGrabLine = MB.GE:WaitForChild("DestroyGrabLine", 10)
MB.StickyPartEvent = MB.RS:WaitForChild("PlayerEvents"):WaitForChild("StickyPartEvent", 10)

MapBreakGroup:AddSlider("MapBreakShurikenCount", {
Text = L("Количество сюрикенов", "Shuriken Count"),
Default = 10,
Min = 5,
Max = 50,
Rounding = 0,
Compact = false,
Tooltip = L("Сколько NinjaShuriken спавнить и прилипать к объекту (больше = сильнее разнос)", "NinjaShuriken count to spawn and stick to object (more = stronger destroy)"),
})

MB.findTransport = function(modelName)
local map = MB.WS:FindFirstChild("Map")
if not map then return nil end
local tweened = map:FindFirstChild("AlwaysHereTweenedObjects")
if not tweened then return nil end
local m = tweened:FindFirstChild(modelName)
if not m then return nil end
return m
end

MB.findBody = function(transportModel)
if not transportModel then return nil end
local obj = transportModel:FindFirstChild("Object")
if not obj then return nil end
local om = obj:FindFirstChild("ObjectModel")
if not om then return nil end
local body = om:FindFirstChild("Body")
if body and body:IsA("BasePart") then return body end
for _, v in ipairs(om:GetChildren()) do
if v:IsA("BasePart") then return v end
end
return nil
end

MB.findTargetByIndex = function(transportModel, idxList)
if not transportModel then return nil end
local obj = transportModel:FindFirstChild("Object")
if not obj then return nil end
local om = obj:FindFirstChild("ObjectModel")
if not om then return nil end
local children = om:GetChildren()
for _, idx in ipairs(idxList) do
local t = children[idx]
if t and t:IsA("BasePart") then return t end
end
for _, v in ipairs(children) do
if v:IsA("BasePart") then return v end
end
return nil
end

MB.breakTransport = function(modelName, targetPart, label)
pcall(function()
local me = MB.LocalPlayer
local char = me.Character
local hrp = char and char:FindFirstChild("HumanoidRootPart")
if not hrp then
Library:Notify(L("Map Break: нет HRP", "Map Break: no HRP"), 3)
return
end

local BackPack = MB.WS:FindFirstChild(me.Name .. "SpawnedInToys")
if not BackPack then
Library:Notify(L("Map Break: нет SpawnedInToys папки", "Map Break: no SpawnedInToys folder"), 3)
return
end

if not targetPart then
Library:Notify("Map Break: target не найден (" .. (label or "?") .. ")", 3)
return
end

local count = Options.MapBreakShurikenCount and Options.MapBreakShurikenCount.Value or 10

local spawnCFrame = hrp.CFrame * CFrame.new(0, 3, 0)
for i = 1, count do
pcall(function()
MB.spawnRF:InvokeServer("NinjaShuriken", spawnCFrame, Vector3.zero)
end)
task.wait(0.05)
end

task.wait(1)

for i = 1, count do
local shur = BackPack:FindFirstChild("NinjaShuriken")
if shur then
local stickyPart = shur:FindFirstChild("StickyPart")
if stickyPart then
shur.Name = tostring(i)
pcall(function()
MB.StickyPartEvent:FireServer(stickyPart, targetPart, CFrame.Angles(0, 0, 0))
end)
end
end
task.wait(0.1)
end

for i = 1, 100 do
pcall(function()
MB.SetNetworkOwner:FireServer(targetPart, targetPart.CFrame)
end)
task.wait()
end

local attach = targetPart:FindFirstChild("ObjectModelAttachment")
if attach then
pcall(function() attach:Destroy() end)
end

local obj = targetPart.Parent
if obj and obj.Parent then
local objParent = obj.Parent
if objParent then
local followPart = objParent:FindFirstChild("FollowThisPart")
if followPart then
local alignPos = followPart:FindFirstChild("AlignPosition")
local alignRot = followPart:FindFirstChild("AlignOrientation")
if alignPos then
pcall(function() alignPos.Attachment0 = nil end)
end
if alignRot then
pcall(function() alignRot.Attachment0 = nil end)
end
end
end
end

Library:Notify("Map Break: " .. (label or "?") .. " сломан!", 3)
end)
end

MapBreakGroup:AddButton({
Text = "Break Outer UFO",
Tooltip = L("Спавнит N сюрикенов и прилипает их к OuterUFO Body, ломает через SetNetworkOwner spam", "Spawns N shurikens and sticks them to OuterUFO Body, breaks via SetNetworkOwner spam"),
Func = function()
local transport = MB.findTransport("OuterUFO")
if not transport then
Library:Notify(L("Map Break: OuterUFO не найден", "Map Break: OuterUFO not found"), 3)
return
end
local body = MB.findBody(transport)
if not body then
Library:Notify(L("Map Break: OuterUFO Body не найден", "Map Break: OuterUFO Body not found"), 3)
return
end
task.spawn(function() MB.breakTransport("OuterUFO", body, "Outer UFO") end)
end,
})

MapBreakGroup:AddButton({
Text = "Break Inner UFO",
Tooltip = L("Аналогично для InnerUFO", "Same for InnerUFO"),
Func = function()
local transport = MB.findTransport("InnerUFO")
if not transport then
Library:Notify(L("Map Break: InnerUFO не найден", "Map Break: InnerUFO not found"), 3)
return
end
local body = MB.findBody(transport)
if not body then
Library:Notify(L("Map Break: InnerUFO Body не найден", "Map Break: InnerUFO Body not found"), 3)
return
end
task.spawn(function() MB.breakTransport("InnerUFO", body, "Inner UFO") end)
end,
})

MapBreakGroup:AddButton({
Text = "Break CaveCart",
Tooltip = L("Ломает CaveCart, target = children[13] из ObjectModel", "Breaks CaveCart, target = children[13] from ObjectModel"),
Func = function()
local transport = MB.findTransport("CaveCart")
if not transport then
Library:Notify(L("Map Break: CaveCart не найден", "Map Break: CaveCart not found"), 3)
return
end
local target = MB.findTargetByIndex(transport, {13, 2, 3, 1})
if not target then
Library:Notify(L("Map Break: CaveCart target не найден", "Map Break: CaveCart target not found"), 3)
return
end
task.spawn(function() MB.breakTransport("CaveCart", target, "CaveCart") end)
end,
})

MapBreakGroup:AddButton({
Text = "Break Train",
Tooltip = L("Ломает Train, target = children[2]/3/1 из ObjectModel", "Breaks Train, target = children[2]/3/1 from ObjectModel"),
Func = function()
local transport = MB.findTransport("Train")
if not transport then
Library:Notify(L("Map Break: Train не найден", "Map Break: Train not found"), 3)
return
end
local target = MB.findTargetByIndex(transport, {2, 3, 1})
if not target then
Library:Notify(L("Map Break: Train target не найден", "Map Break: Train target not found"), 3)
return
end
task.spawn(function() MB.breakTransport("Train", target, "Train") end)
end,
})

end

do
do
local BSPGroup = Tabs.Fun:AddRightGroupbox("Break Spawn", "hammer")
local bspRS = game:GetService("ReplicatedStorage")
local bspToys = bspRS:FindFirstChild("MenuToys")
local bspSpawnRF = bspToys and bspToys:FindFirstChild("SpawnToyRemoteFunction")
local bspDestroyToy = bspToys and bspToys:FindFirstChild("DestroyToy")
local bspGE = bspRS:FindFirstChild("GrabEvents")
local bspSNW = bspGE and bspGE:FindFirstChild("SetNetworkOwner")
local bspPE = bspRS:FindFirstChild("PlayerEvents")
local bspSticky = bspPE and bspPE:FindFirstChild("StickyPartEvent")
local bspForced = setmetatable({}, { __mode = "k" })

BSPGroup:AddDropdown("BreakSpawnMethod", {
Text = "Break Method",
Values = { "Non-collidable part", "Kickable part" },
Default = "Non-collidable part",
Tooltip = L("Non-collidable part — спавн уносится в небо, Kickable part — уносится под карту", "Non-collidable part — spawn flies to the sky, Kickable part — flies below the map"),
})

BSPGroup:AddButton({
Text = "Break Spawn",
Tooltip = L("Ломает спавн-плиту: все стоящие на спавне падают вниз и умирают", "Breaks the spawn plate: everyone standing on spawn falls down and dies"),
Func = function()
task.spawn(function()
local map = workspace:FindFirstChild("Map")
local ground = map and map:FindFirstChild("BaseGround")
if not ground then
Library:Notify(L("Break Spawn: BaseGround не найден", "Break Spawn: BaseGround not found"), 3)
return
end
local plate = nil
for _, child in ipairs(ground:GetChildren()) do
if child:IsA("BasePart") then
if (child.Position - Vector3.new(107.5, -101.35, 43.375)).Magnitude <= 0.1 then
plate = child
break
else
plate = nil
end
else
plate = nil
end
end
if not plate then
Library:Notify(L("Break Spawn: спавн-плита не найдена", "Break Spawn: spawn plate not found"), 3)
return
end
local shur = OBV_SpawnToy("NinjaShuriken")
if not shur then
Library:Notify(L("Break Spawn: сюрикен не заспавнился", "Break Spawn: shuriken did not spawn"), 3)
return
end
local stickyPart = shur:WaitForChild("StickyPart", 0.5)
local stickyWeld = stickyPart and stickyPart:WaitForChild("StickyWeld", 0.5)
if not (stickyPart and stickyWeld) then
if bspDestroyToy then pcall(function() bspDestroyToy:FireServer(shur) end) end
Library:Notify(L("Break Spawn: StickyWeld не найден", "Break Spawn: StickyWeld not found"), 3)
return
end
if not OBV_SNO(stickyPart) then
if bspDestroyToy then pcall(function() bspDestroyToy:FireServer(shur) end) end
Library:Notify(L("Break Spawn: не удалось взять овнершип — попробуй ещё раз", "Break Spawn: failed to take ownership — try again"), 3)
return
end
if not bspForced[stickyWeld] then
bspForced[stickyWeld] = true
stickyWeld:GetPropertyChangedSignal("Enabled"):Connect(function()
if bspForced[stickyWeld] and stickyWeld.Parent and stickyWeld.Enabled then
stickyWeld.Enabled = false
end
end)
end
stickyWeld.Enabled = false
local method = Options.BreakSpawnMethod and Options.BreakSpawnMethod.Value or "Non-collidable part"
local offset = method == "Kickable part" and CFrame.new(0, -2.3e9, 0) or CFrame.new(1e15, 1e15, 1e15)
pcall(function() bspSticky:FireServer(stickyPart, plate, offset) end)
Library:Notify(L("Break Spawn: спавн сломан — все на спавне падают", "Break Spawn: spawn broken — everyone on spawn falls"), 3)
end)
end,
})

BSPGroup:AddLabel(L("Fix part: удали шурикен, который заспавнился в инвентаре!", "Fix part: delete the shuriken that was spawned in your inventory!"), true)
end

local BreakSZGroup = Tabs.Fun:AddRightGroupbox("Break SafeZone", "shield-off")

local BSZ = {}
BSZ.RS = game:GetService("ReplicatedStorage")
BSZ.WS = game:GetService("Workspace")
BSZ.Players = game:GetService("Players")
BSZ.LocalPlayer = BSZ.Players.LocalPlayer

BSZ.MenuToys = BSZ.RS:WaitForChild("MenuToys", 10)
BSZ.spawnRF  = BSZ.MenuToys:WaitForChild("SpawnToyRemoteFunction", 10)
BSZ.destroyToy = BSZ.MenuToys:FindFirstChild("DestroyToy") or BSZ.MenuToys:WaitForChild("DestroyToy", 10)
BSZ.GE = BSZ.RS:WaitForChild("GrabEvents", 10)
BSZ.SetNetworkOwner = BSZ.GE:WaitForChild("SetNetworkOwner", 10)
BSZ.DestroyGrabLine = BSZ.GE:WaitForChild("DestroyGrabLine", 10)

local function getSnowballs()
local snowballs = {}
local folder = BSZ.WS:FindFirstChild(BSZ.LocalPlayer.Name .. "SpawnedInToys")
if folder then
for _, toy in ipairs(folder:GetChildren()) do
if toy.Name == "BallSnowball" then
table.insert(snowballs, toy)
end
end
end
return snowballs
end

local function spawnSnowball()
local initialCFrame = CFrame.new(263.5, -4.5, 486.9)
task.spawn(function()
pcall(function()
BSZ.spawnRF:InvokeServer("BallSnowball", initialCFrame, Vector3.new(0, -120.21099853515625, 0))
end)
end)
end

local function processSnowball(snowball)
local soundPart = snowball:FindFirstChild("SoundPart")
if soundPart then
task.spawn(function()
pcall(function()
BSZ.SetNetworkOwner:FireServer(soundPart, soundPart.CFrame)
end)
end)
end
end

local function teleportSnowballs(snowballs)
local targetCFrame = CFrame.new(264.5792541503906, -5.477070331573486, 433.4557800292969)
for _, snowball in ipairs(snowballs) do
task.spawn(function()
for _, part in ipairs(snowball:GetDescendants()) do
if part:IsA("BasePart") then
part.CFrame = targetCFrame
end
end
end)
end
end

local function testWithOven()
local targetPosition = CFrame.new(242.66055297851562, -9.196549415588379, 444.3758850097656)
pcall(function()
BSZ.spawnRF:InvokeServer("OvenDarkGray", targetPosition, Vector3.new(0, -74.0790023803711, 0))
end)
task.wait(0.5)
local folder = BSZ.WS:FindFirstChild(BSZ.LocalPlayer.Name .. "SpawnedInToys")
if not folder then return false end
for _, child in ipairs(folder:GetChildren()) do
if child.Name == "OvenDarkGray" and child:GetAttribute("AtSpawned") == nil then
return true
end
end
return false
end

local function cleanupToys()
local folder = BSZ.WS:FindFirstChild(BSZ.LocalPlayer.Name .. "SpawnedInToys")
if folder then
for _, item in ipairs(folder:GetChildren()) do
if item.Name == "BallSnowball" or item.Name == "OvenDarkGray" then
pcall(function() BSZ.destroyToy:FireServer(item) end)
end
end
end
end

local function setAntiBarrier(val)
local plots = BSZ.WS:FindFirstChild("Plots")
if not plots then return end
for _, plot in ipairs(plots:GetChildren()) do
local barrierModel = plot:FindFirstChild("Barrier")
if barrierModel then
for _, part in ipairs(barrierModel:GetChildren()) do
if part:IsA("BasePart") and part.Name == "PlotBarrier" then
part.CanCollide = not val
end
end
end
end
end

BreakSZGroup:AddButton({
Text = "Break SafeZone",
Tooltip = L("Спам снежками по барьеру + снимает CanCollide (Anti Barrier). Слэм из 9rr", "Snowball spam on barrier + removes CanCollide (Anti Barrier). Slam from 9rr"),
Func = function()
task.spawn(function()
Library:Notify(L("Break SZ: ломаем барьер...", "Break SZ: breaking barrier..."), 2)

setAntiBarrier(true)

local startTime = tick()
local broken = false
while (tick() - startTime) < 15 and not broken do
local snowballs = getSnowballs()

if #snowballs < 2 then
if #snowballs < 1 then
spawnSnowball()
task.wait(0.02)
end
spawnSnowball()
task.wait(0.02)
snowballs = getSnowballs()
end

if #snowballs == 2 then
for _, snowball in ipairs(snowballs) do
processSnowball(snowball)
end
teleportSnowballs(snowballs)

if testWithOven() then
broken = true
else
task.wait(0.05)
end
end

task.wait(0.05)
end

cleanupToys()

if broken then
Library:Notify(L("Break SZ: барьер сломан + Anti Barrier ВКЛ!", "Break SZ: barrier broken + Anti Barrier ON!"), 3)
else
Library:Notify(L("Break SZ: Anti Barrier ВКЛ (снежки не успели, но проходить можно)", "Break SZ: Anti Barrier ON (snowballs didn't work, but you can pass)"), 3)
end
end)
end,
})
end

end
end
end
end
end
end