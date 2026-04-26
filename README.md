if not game:IsLoaded() then
    game.Loaded:Wait()
end
local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local StarterGui = game:GetService("StarterGui")
local LocalPlayer = Players.LocalPlayer

-- CẤU HÌNH RAM API
local USER_SETTINGS = {
    Enable_RAM_API = true,   
    RAM_Port = "7963",       
    RAM_Password = "",       
    DefaultDon = "Chưa có",  
    RAM_IP = "localhost" 
}

local function SendNotification(title, text)
    pcall(function()
        StarterGui:SetCore("SendNotification", { Title = title; Text = text; Duration = 5; })
    end)
end

local requestFunc = (syn and syn.request) or request or (http and http.request) or http_request

local function RAM_API(method, action, bodyText)
    if not requestFunc then return false end
    local url = string.format("http://%s:%s/%s?Account=%s", USER_SETTINGS.RAM_IP, USER_SETTINGS.RAM_Port, action, LocalPlayer.Name)
    if USER_SETTINGS.RAM_Password ~= "" then url = url .. "&Password=" .. USER_SETTINGS.RAM_Password end
    
    local reqData = { Url = url, Method = method }
    if method == "POST" and bodyText then reqData.Body = bodyText end
    
    local success, res = pcall(function() return requestFunc(reqData) end)
    if success and res and res.StatusCode == 200 then return res.Body or true end
    return false
end

local RamConnected = false
local SystemLogMsg = "Dang kiem tra ket noi..."

if USER_SETTINGS.Enable_RAM_API then
    if not requestFunc then
        SystemLogMsg = "Loi: Executor khong ho tro HTTP Request!"
    else
        local testConn = RAM_API("GET", "GetCSRFToken")
        if testConn and testConn ~= "Invalid Account" and testConn ~= "" then
            RamConnected = true
            SystemLogMsg = "RAM: Ket noi API thanh cong!"
        else
            SystemLogMsg = "RAM Loi: Chua bat WebServer / Sai Port!"
        end
    end
else
    SystemLogMsg = "RAM API: Da tat trong Cau hinh."
end

local SAVE_FILE_NAME = "DuLieuDon_" .. LocalPlayer.Name .. ".json"
local Config = { UseRAM = false, CurrentDon = USER_SETTINGS.DefaultDon }

local function SaveLocalData()
    if writefile then writefile(SAVE_FILE_NAME, HttpService:JSONEncode({ Don = Config.CurrentDon })) end
end

local function LoadLocalData()
    if isfile and isfile(SAVE_FILE_NAME) and readfile then
        pcall(function()
            local decoded = HttpService:JSONDecode(readfile(SAVE_FILE_NAME))
            if decoded.Don then Config.CurrentDon = decoded.Don end
        end)
    end
end

if RamConnected then
    Config.UseRAM = true
    local currentRamDesc = RAM_API("GET", "GetDescription")
    if currentRamDesc and currentRamDesc ~= "" then
        local parsedDon = string.match(currentRamDesc, "Đơn: ([^\r\n]*)")
        Config.CurrentDon = parsedDon or USER_SETTINGS.DefaultDon
    else
        LoadLocalData()
    end
else
    LoadLocalData()
end

-- TAO GIAO DIEN UI
local targetGui = LocalPlayer:WaitForChild("PlayerGui")

if targetGui:FindFirstChild("MyCustomUI") then 
    targetGui.MyCustomUI:Destroy() 
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MyCustomUI"
ScreenGui.ResetOnSpawn = false 
ScreenGui.Parent = targetGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 280, 0, 90)
MainFrame.Position = UDim2.new(0.5, -140, 0, 20)
MainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
MainFrame.BorderSizePixel = 3
MainFrame.BorderColor3 = Color3.fromRGB(255, 215, 0)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, -35, 0, 30)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "QUẢN LÝ ĐƠN HÀNG"
Title.TextColor3 = Color3.fromRGB(255, 255, 255)
Title.Font = Enum.Font.SourceSansBold
Title.TextSize = 18
Title.TextXAlignment = Enum.TextXAlignment.Left
Title.Parent = MainFrame

local SettingsBtn = Instance.new("TextButton")
SettingsBtn.Size = UDim2.new(0, 30, 0, 30)
SettingsBtn.Position = UDim2.new(1, -30, 0, 0)
SettingsBtn.BackgroundTransparency = 1
SettingsBtn.Text = "⚙️"
SettingsBtn.TextSize = 18
SettingsBtn.Parent = MainFrame

local hiddenName = string.sub(LocalPlayer.Name, 1, 4) .. "*****"

-- TAB 1
local ViewMain = Instance.new("Frame")
ViewMain.Size = UDim2.new(1, 0, 1, -30)
ViewMain.Position = UDim2.new(0, 0, 0, 30)
ViewMain.BackgroundTransparency = 1
ViewMain.Parent = MainFrame

local NameLabel = Instance.new("TextLabel")
NameLabel.Size = UDim2.new(1, -20, 0, 25)
NameLabel.Position = UDim2.new(0, 10, 0, 0)
NameLabel.BackgroundTransparency = 1
NameLabel.Text = "Tên: " .. hiddenName
NameLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
NameLabel.Font = Enum.Font.SourceSansBold
NameLabel.TextSize = 16
NameLabel.TextXAlignment = Enum.TextXAlignment.Left
NameLabel.Parent = ViewMain

local OrderLabel = Instance.new("TextLabel")
OrderLabel.Size = UDim2.new(0.75, -10, 0, 25)
OrderLabel.Position = UDim2.new(0, 10, 0, 25)
OrderLabel.BackgroundTransparency = 1
OrderLabel.Text = "Đơn: " .. Config.CurrentDon
OrderLabel.TextColor3 = Color3.fromRGB(255, 255, 0)
OrderLabel.Font = Enum.Font.SourceSansBold
OrderLabel.TextSize = 16
OrderLabel.TextXAlignment = Enum.TextXAlignment.Left
OrderLabel.TextTruncate = Enum.TextTruncate.AtEnd
OrderLabel.Parent = ViewMain

local EditBtn = Instance.new("TextButton")
EditBtn.Size = UDim2.new(0.25, -10, 0, 25)
EditBtn.Position = UDim2.new(0.75, 0, 0, 25)
EditBtn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
EditBtn.Text = "✏️ Sửa"
EditBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
EditBtn.Font = Enum.Font.SourceSansBold
EditBtn.TextSize = 14
EditBtn.Parent = ViewMain

local EditFrame = Instance.new("Frame")
EditFrame.Size = UDim2.new(1, -20, 0, 25)
EditFrame.Position = UDim2.new(0, 10, 0, 25)
EditFrame.BackgroundTransparency = 1
EditFrame.Visible = false
EditFrame.Parent = ViewMain

local InputBox = Instance.new("TextBox")
InputBox.Size = UDim2.new(0.7, -5, 1, 0)
InputBox.Position = UDim2.new(0, 0, 0, 0)
InputBox.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
InputBox.TextColor3 = Color3.fromRGB(0, 0, 0)
InputBox.PlaceholderText = "Nhập đơn..."
InputBox.Text = ""
InputBox.Font = Enum.Font.SourceSansBold
InputBox.TextSize = 15
InputBox.TextXAlignment = Enum.TextXAlignment.Left
InputBox.ClearTextOnFocus = false
InputBox.Parent = EditFrame

local SaveBtn = Instance.new("TextButton")
SaveBtn.Size = UDim2.new(0.3, 0, 1, 0)
SaveBtn.Position = UDim2.new(0.7, 0, 0, 0)
SaveBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
SaveBtn.Text = "💾 Lưu"
SaveBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
SaveBtn.Font = Enum.Font.SourceSansBold
SaveBtn.TextSize = 15
SaveBtn.Parent = EditFrame

-- TAB 2
local ViewSettings = Instance.new("Frame")
ViewSettings.Size = UDim2.new(1, 0, 1, -30)
ViewSettings.Position = UDim2.new(0, 0, 0, 30)
ViewSettings.BackgroundTransparency = 1
ViewSettings.Visible = false 
ViewSettings.Parent = MainFrame

local LogLabel = Instance.new("TextLabel")
LogLabel.Size = UDim2.new(1, -20, 0, 25)
LogLabel.Position = UDim2.new(0, 10, 0, 0)
LogLabel.BackgroundTransparency = 1
LogLabel.TextColor3 = RamConnected and Color3.fromRGB(100, 255, 100) or Color3.fromRGB(255, 100, 100)
LogLabel.TextWrapped = true
LogLabel.Font = Enum.Font.SourceSansBold
LogLabel.TextSize = 14
LogLabel.Text = SystemLogMsg
LogLabel.Parent = ViewSettings

local ToggleRAMBtn = Instance.new("TextButton")
ToggleRAMBtn.Size = UDim2.new(1, -20, 0, 25)
ToggleRAMBtn.Position = UDim2.new(0, 10, 0, 25)
ToggleRAMBtn.Text = Config.UseRAM and "Trạng thái: LƯU LÊN RAM" or "Trạng thái: LƯU LOCAL"
ToggleRAMBtn.BackgroundColor3 = Config.UseRAM and Color3.fromRGB(50, 200, 50) or Color3.fromRGB(200, 50, 50)
ToggleRAMBtn.Font = Enum.Font.SourceSansBold
ToggleRAMBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleRAMBtn.TextSize = 14
ToggleRAMBtn.Parent = ViewSettings

SettingsBtn.MouseButton1Click:Connect(function()
    if ViewMain.Visible then
        ViewMain.Visible = false
        ViewSettings.Visible = true
        SettingsBtn.Text = "🔙"
        Title.Text = "CÀI ĐẶT & LOG"
    else
        ViewMain.Visible = true
        ViewSettings.Visible = false
        SettingsBtn.Text = "⚙️"
        Title.Text = "QUẢN LÝ ĐƠN HÀNG"
    end
end)

EditBtn.MouseButton1Click:Connect(function()
    OrderLabel.Visible = false
    EditBtn.Visible = false
    EditFrame.Visible = true
    InputBox.Text = Config.CurrentDon
    InputBox:CaptureFocus()
end)

SaveBtn.MouseButton1Click:Connect(function()
    local newNote = InputBox.Text
    if newNote == "" then return end
    Config.CurrentDon = newNote
    OrderLabel.Text = "Đơn: " .. Config.CurrentDon
    EditFrame.Visible = false
    OrderLabel.Visible = true
    EditBtn.Visible = true

    task.spawn(function()
        if Config.UseRAM and RamConnected then
            local currentDesc = RAM_API("GET", "GetDescription") or ""
            local cleanDesc = string.gsub(currentDesc, "Đơn: [^\r\n]*[\r\n]*", "")
            cleanDesc = string.gsub(cleanDesc, "^%s+", "")
            local finalDesc = "Đơn: " .. newNote
            if cleanDesc ~= "" then finalDesc = finalDesc .. "\n" .. cleanDesc end
            
            local successSet = RAM_API("POST", "SetDescription", finalDesc)
            if successSet then SendNotification("Thành công", "Đã GHIM ĐƠN lên RAM!")
            else SendNotification("Lỗi RAM", "Không gửi được dữ liệu.") end
        else
            SaveLocalData()
            SendNotification("Thành công", "Đã lưu Local!")
        end
    end)
end)

ToggleRAMBtn.MouseButton1Click:Connect(function()
    if not RamConnected and not Config.UseRAM then
        SendNotification("Cảnh báo", "RAM chưa kết nối!") return
    end
    Config.UseRAM = not Config.UseRAM
    ToggleRAMBtn.Text = Config.UseRAM and "Trạng thái: LƯU LÊN RAM" or "Trạng thái: LƯU LOCAL"
    ToggleRAMBtn.BackgroundColor3 = Config.UseRAM and Color3.fromRGB(50, 200, 50) or Color3.fromRGB(200, 50, 50)
end)

task.spawn(function()
    while task.wait(1) do
        if Config.UseRAM and RamConnected then
            local currentDesc = RAM_API("GET", "GetDescription")
            if currentDesc and currentDesc ~= "" then
                local parsedDon = string.match(currentDesc, "Đơn: ([^\r\n]*)")
                if parsedDon and parsedDon ~= Config.CurrentDon then
                    Config.CurrentDon = parsedDon
                    OrderLabel.Text = "Đơn: " .. Config.CurrentDon
                    if EditFrame.Visible and not InputBox:IsFocused() then InputBox.Text = Config.CurrentDon end
                end
            end
        end
    end
end)

game.Players.PlayerRemoving:Connect(function(player)
    if player == LocalPlayer and not Config.UseRAM then SaveLocalData() end
end)
