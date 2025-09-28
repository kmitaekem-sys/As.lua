-- === Utility HUD (Q/N/M) ===
-- Speed 39 (Q) + ESP ผู้เล่น (N) + Night Vision (M) + ปุ่ม 3 อัน (มือถือ/PC)

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local LP = Players.LocalPlayer

-- ===== Config =====
local DEFAULT_SPEED = 16
local FAST_SPEED    = 39
_G.__ESP_ON   = _G.__ESP_ON   or false
_G.__SPEED_ON = _G.__SPEED_ON or true
_G.__NIGHT_ON = _G.__NIGHT_ON or true

-- ===== Speed =====
local function applySpeed()
    local hum = LP.Character and LP.Character:FindFirstChildOfClass("Humanoid")
    if hum then hum.WalkSpeed = (_G.__SPEED_ON and FAST_SPEED or DEFAULT_SPEED) end
end
applySpeed()
LP.CharacterAdded:Connect(function() task.wait(0.2); applySpeed() end)

-- ===== ESP (ชื่อเล็ก + กรอบขาว) =====
local function headOf(char)
    return char and (char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart") or char.PrimaryPart)
end
local function removeESP(char)
    if not char then return end
    local h = headOf(char)
    if h and h:FindFirstChild("NameESP") then h.NameESP:Destroy() end
    if char:FindFirstChild("BoxESP") then char.BoxESP:Destroy() end
    for _,p in ipairs(char:GetDescendants()) do
        if p:IsA("BoxHandleAdornment") and p.Name=="BoxESP_BOX" then p:Destroy() end
    end
end
local function addNameESP(plr, char)
    local head = headOf(char); if not head then return end
    local old = head:FindFirstChild("NameESP"); if old then old:Destroy() end
    local bill = Instance.new("BillboardGui")
    bill.Name, bill.Adornee, bill.AlwaysOnTop = "NameESP", head, true
    bill.Size, bill.StudsOffset = UDim2.new(0,120,0,20), Vector3.new(0,2,0)
    local label = Instance.new("TextLabel")
    label.BackgroundTransparency, label.Size = 1, UDim2.new(1,0,1,0)
    label.Text = (plr.DisplayName~="" and plr.DisplayName or plr.Name)
    label.TextColor3, label.TextStrokeTransparency = Color3.new(1,1,1), 0.3
    label.Font, label.TextScaled, label.TextSize = Enum.Font.SourceSansBold, false, 16
    label.Parent = bill; bill.Parent = head
end
local function addBoxESP_Highlight(char)
    if char:FindFirstChild("BoxESP") then char.BoxESP:Destroy() end
    local ok,hl = pcall(function()
        local h = Instance.new("Highlight")
        h.Name, h.Adornee = "BoxESP", char
        h.FillTransparency, h.OutlineTransparency = 1, 0
        h.OutlineColor, h.DepthMode = Color3.new(1,1,1), Enum.HighlightDepthMode.AlwaysOnTop
        h.Parent = char; return h
    end)
    return ok and hl
end
local function addBoxESP_Fallback(char)
    for _,bp in ipairs(char:GetDescendants()) do
        if bp:IsA("BasePart") then
            if bp:FindFirstChild("BoxESP_BOX") then bp.BoxESP_BOX:Destroy() end
            local box = Instance.new("BoxHandleAdornment")
            box.Name, box.Adornee, box.AlwaysOnTop, box.ZIndex = "BoxESP_BOX", bp, true, 10
            box.Transparency, box.Size, box.Color3 = 0.5, bp.Size + Vector3.new(0.05,0.05,0.05), Color3.new(1,1,1)
            box.Parent = bp
        end
    end
end
local function addESPToChar(plr, char)
    if not _G.__ESP_ON then return end
    addNameESP(plr, char)
    if not addBoxESP_Highlight(char) then addBoxESP_Fallback(char) end
end
local function applyESPToPlayer(p)
    if p==LP then return end
    local function onChar(c) if _G.__ESP_ON then removeESP(c); addESPToChar(p,c) else removeESP(c) end end
    if p.Character then onChar(p.Character) end
    p.CharacterAdded:Connect(onChar)
end
local function applyESPToAll()
    for _,p in ipairs(Players:GetPlayers()) do applyESPToPlayer(p) end
end
local function removeESPFromAll()
    for _,p in ipairs(Players:GetPlayers()) do if p~=LP and p.Character then removeESP(p.Character) end end
end
Players.PlayerAdded:Connect(applyESPToPlayer)

-- ===== Night Vision =====
local nightConn
local function enforceNight()
    local L = game.Lighting
    L.Brightness = 3; L.Ambient = Color3.new(1,1,1); L.OutdoorAmbient = Color3.new(1,1,1)
    if L:FindFirstChild("Atmosphere") then L.Atmosphere.Density, L.Atmosphere.Haze = 0, 0 end
    L.FogEnd, L.ClockTime = 100000, 12
end
local function setNight(on)
    if on then if not nightConn then nightConn = RunService.Heartbeat:Connect(enforceNight) end
    else if nightConn then nightConn:Disconnect(); nightConn=nil end end
end
setNight(_G.__NIGHT_ON)

-- ===== UI: 3 ปุ่ม =====
local function ensureUI()
    local pg = LP:WaitForChild("PlayerGui")
    if pg:FindFirstChild("UTIL_Toggles_QNM") then pg.UTIL_Toggles_QNM:Destroy() end
    local gui = Instance.new("ScreenGui")
    gui.Name, gui.ResetOnSpawn, gui.IgnoreGuiInset = "UTIL_Toggles_QNM", false, true
    gui.Parent = pg

    local function makeBtn(name, text, row)
        local b = Instance.new("TextButton")
        b.Name, b.Text, b.AutoButtonColor = name, text, true
        b.BackgroundTransparency = 0.15
        b.Size = UDim2.new(0,160,0,40)
        b.AnchorPoint = Vector2.new(1,1)
        b.Position = UDim2.new(1,-16,1,-16-(row-1)*48)
        b.Parent = gui
        return b
    end
    local btnESP  = makeBtn("BtnESP",  _G.__ESP_ON   and "ESP: ON"        or "ESP: OFF",        1)
    local btnSPD  = makeBtn("BtnSPD",  _G.__SPEED_ON and "Speed: ON (39)" or "Speed: OFF (16)", 2)
    local btnNIGH = makeBtn("BtnNIGH", _G.__NIGHT_ON and "Night: ON"      or "Night: OFF",      3)

    local function refresh()
        btnESP.Text  = _G.__ESP_ON   and "ESP: ON"        or "ESP: OFF"
        btnSPD.Text  = _G.__SPEED_ON and "Speed: ON (39)" or "Speed: OFF (16)"
        btnNIGH.Text = _G.__NIGHT_ON and "Night: ON"      or "Night: OFF"
    end
    btnESP.MouseButton1Click:Connect(function()
        _G.__ESP_ON = not _G.__ESP_ON
        if _G.__ESP_ON then applyESPToAll() else removeESPFromAll() end
        refresh()
    end)
    btnSPD.MouseButton1Click:Connect(function()
        _G.__SPEED_ON = not _G.__SPEED_ON
        applySpeed()
        refresh()
    end)
    btnNIGH.MouseButton1Click:Connect(function()
        _G.__NIGHT_ON = not _G.__NIGHT_ON
        setNight(_G.__NIGHT_ON)
        refresh()
    end)

    -- ===== Hotkeys: N = ESP, Q = Speed, M = Night =====
    UIS.InputBegan:Connect(function(inp, gpe)
        if gpe then return end
        if inp.KeyCode == Enum.KeyCode.N then btnESP:Activate()
        elseif inp.KeyCode == Enum.KeyCode.Q then btnSPD:Activate()  -- เปลี่ยนเป็น Q ตามที่ขอ
        elseif inp.KeyCode == Enum.KeyCode.M then btnNIGH:Activate()
        end
    end)
end
ensureUI()

-- sync เริ่มต้น
if _G.__ESP_ON then applyESPToAll() else removeESPFromAll() end
applySpeed(); setNight(_G.__NIGHT_ON)
