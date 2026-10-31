local UNiHub = {}
UNiHub.__index = UNiHub

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")

local Player = Players.LocalPlayer
local Theme = {
    Background = Color3.fromRGB(17, 15, 27),
    Surface = Color3.fromRGB(25, 22, 39),
    Surface2 = Color3.fromRGB(34, 29, 51),
    Accent = Color3.fromRGB(145, 92, 255),
    Accent2 = Color3.fromRGB(105, 65, 205),
    Text = Color3.fromRGB(245, 242, 255),
    Muted = Color3.fromRGB(163, 155, 184),
    Stroke = Color3.fromRGB(62, 53, 83),
    Good = Color3.fromRGB(85, 205, 135),
    Bad = Color3.fromRGB(235, 92, 105)
}

local function create(className, properties, parent)
    local object = Instance.new(className)
    for property, value in pairs(properties or {}) do
        object[property] = value
    end
    object.Parent = parent
    return object
end

local function corner(parent, radius)
    return create("UICorner", {CornerRadius = UDim.new(0, radius or 8)}, parent)
end

local function stroke(parent, color, transparency)
    return create("UIStroke", {
        Color = color or Theme.Stroke,
        Transparency = transparency or 0,
        Thickness = 1
    }, parent)
end

local function tween(object, duration, properties)
    local animation = TweenService:Create(object, TweenInfo.new(duration or 0.18, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), properties)
    animation:Play()
    return animation
end

local function makeText(parent, text, size, color, font)
    return create("TextLabel", {
        BackgroundTransparency = 1,
        Text = tostring(text or ""),
        TextColor3 = color or Theme.Text,
        TextSize = size or 13,
        Font = font or Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Center,
        Size = UDim2.new(1, 0, 1, 0)
    }, parent)
end

local function normalizeKey(value)
    if typeof(value) == "EnumItem" then
        return value
    end
    if type(value) == "string" then
        return Enum.KeyCode[value]
    end
    return nil
end

function UNiHub:Notify(title, message, duration)
    if self.Destroyed then return end
    local toast = create("Frame", {
        AnchorPoint = Vector2.new(1, 0),
        Position = UDim2.new(1, 320, 0, 18),
        Size = UDim2.fromOffset(280, 76),
        BackgroundColor3 = Theme.Surface,
        BorderSizePixel = 0
    }, self.ToastHolder)
    corner(toast, 10)
    stroke(toast)
    local titleLabel = makeText(create("Frame", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(13, 8),
        Size = UDim2.new(1, -26, 0, 20)
    }, toast), title or "UNi Hub", 14, Theme.Text, Enum.Font.GothamBold)
    local messageLabel = makeText(create("Frame", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(13, 31),
        Size = UDim2.new(1, -26, 0, 34)
    }, toast), message or "", 12, Theme.Muted)
    messageLabel.TextWrapped = true
    tween(toast, 0.25, {Position = UDim2.new(1, -18, 0, 18)})
    task.delay(duration or 3, function()
        if toast and toast.Parent then
            tween(toast, 0.2, {Position = UDim2.new(1, 320, 0, 18)})
            task.wait(0.22)
            if toast then toast:Destroy() end
        end
    end)
end

function UNiHub:_register(id, value, setter)
    if not id or id == "" then return end
    self.Values[id] = value
    self.Setters[id] = setter
end

function UNiHub:GetValue(id)
    return self.Values[id]
end

function UNiHub:SetValue(id, value)
    local setter = self.Setters[id]
    if setter then
        setter(value)
        return true
    end
    return false
end

function UNiHub:GetConfig()
    local data = {}
    for id, value in pairs(self.Values) do
        if typeof(value) == "EnumItem" then
            data[id] = value.Name
        else
            data[id] = value
        end
    end
    return HttpService:JSONEncode(data)
end

function UNiHub:LoadConfig(json)
    local success, data = pcall(function()
        return HttpService:JSONDecode(json)
    end)
    if not success or type(data) ~= "table" then
        self:Notify("Config", "Invalid configuration data.", 3)
        return false
    end
    for id, value in pairs(data) do
        if self.Setters[id] then
            self:SetValue(id, value)
        end
    end
    self:Notify("Config", "Configuration loaded.", 2)
    return true
end

function UNiHub:Destroy()
    if self.Destroyed then return end
    self.Destroyed = true
    if self.ScreenGui then self.ScreenGui:Destroy() end
    if self.InputConnection then self.InputConnection:Disconnect() end
end

function UNiHub:CreateWindow(options)
    options = options or {}
    local self = setmetatable({}, UNiHub)
    self.Values = {}
    self.Setters = {}
    self.Tabs = {}
    self.Destroyed = false
    self.Visible = true
    self.ToggleKey = normalizeKey(options.ToggleKey or Enum.KeyCode.RightControl) or Enum.KeyCode.RightControl

    local playerGui = Player:WaitForChild("PlayerGui")
    local old = playerGui:FindFirstChild("UNiHub")
    if old then old:Destroy() end

    self.ScreenGui = create("ScreenGui", {
        Name = "UNiHub",
        ResetOnSpawn = false,
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    }, playerGui)

    self.Main = create("Frame", {
        Name = "Main",
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.fromScale(0.5, 0.5),
        Size = options.Size or UDim2.fromOffset(650, 460),
        BackgroundColor3 = Theme.Background,
        BorderSizePixel = 0,
        ClipsDescendants = true
    }, self.ScreenGui)
    corner(self.Main, 12)
    stroke(self.Main, Theme.Stroke)

    local top = create("Frame", {
        Name = "Topbar",
        Size = UDim2.new(1, 0, 0, 48),
        BackgroundColor3 = Theme.Surface,
        BorderSizePixel = 0
    }, self.Main)
    local title = makeText(create("Frame", {
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(16, 0),
        Size = UDim2.new(1, -120, 1, 0)
    }, top), options.Title or "UNi Hub", 15, Theme.Text, Enum.Font.GothamBold)

    local close = create("TextButton", {
        Text = "×",
        Font = Enum.Font.GothamBold,
        TextSize = 23,
        TextColor3 = Theme.Muted,
        BackgroundTransparency = 1,
        Size = UDim2.fromOffset(42, 42),
        Position = UDim2.new(1, -46, 0, 3),
        AutoButtonColor = false
    }, top)
    close.MouseButton1Click:Connect(function() self:Destroy() end)

    local minimize = create("TextButton", {
        Text = "—",
        Font = Enum.Font.GothamBold,
        TextSize = 18,
        TextColor3 = Theme.Muted,
        BackgroundTransparency = 1,
        Size = UDim2.fromOffset(38, 42),
        Position = UDim2.new(1, -86, 0, 3),
        AutoButtonColor = false
    }, top)
    minimize.MouseButton1Click:Connect(function()
        self.Content.Visible = not self.Content.Visible
        tween(self.Main, 0.18, {Size = self.Content.Visible and (options.Size or UDim2.fromOffset(650, 460)) or UDim2.fromOffset(650, 48)})
    end)

    self.Content = create("Frame", {
        Name = "Content",
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(0, 48),
        Size = UDim2.new(1, 0, 1, -48)
    }, self.Main)

    self.TabBar = create("ScrollingFrame", {
        Name = "Tabs",
        BackgroundColor3 = Theme.Surface,
        BorderSizePixel = 0,
        Position = UDim2.fromOffset(0, 0),
        Size = UDim2.new(0, 148, 1, 0),
        CanvasSize = UDim2.new(),
        ScrollBarThickness = 2,
        ScrollBarImageColor3 = Theme.Accent
    }, self.Content)
    local tabLayout = create("UIListLayout", {
        Padding = UDim.new(0, 6),
        SortOrder = Enum.SortOrder.LayoutOrder
    }, self.TabBar)
    create("UIPadding", {
        PaddingTop = UDim.new(0, 12),
        PaddingLeft = UDim.new(0, 9),
        PaddingRight = UDim.new(0, 9)
    }, self.TabBar)
    tabLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        self.TabBar.CanvasSize = UDim2.fromOffset(0, tabLayout.AbsoluteContentSize.Y + 20)
    end)

    self.Pages = create("Frame", {
        Name = "Pages",
        BackgroundTransparency = 1,
        Position = UDim2.fromOffset(148, 0),
        Size = UDim2.new(1, -148, 1, 0)
    }, self.Content)

    self.ToastHolder = create("Frame", {
        BackgroundTransparency = 1,
        Position = UDim2.new(1, -300, 0, 0),
        Size = UDim2.fromOffset(300, 110),
        ZIndex = 20
    }, self.ScreenGui)

    local dragging, dragStart, startPosition
    top.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPosition = self.Main.Position
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
            local delta = input.Position - dragStart
            self.Main.Position = UDim2.new(startPosition.X.Scale, startPosition.X.Offset + delta.X, startPosition.Y.Scale, startPosition.Y.Offset + delta.Y)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
    end)

    self.InputConnection = UserInputService.InputBegan:Connect(function(input, processed)
        if not processed and input.KeyCode == self.ToggleKey and not self.Destroyed then
            self.Visible = not self.Visible
            self.Main.Visible = self.Visible
        end
    end)

    self:Notify("UNi Hub", "Interface loaded.", 2)
    return self
end

function UNiHub:Tab(name)
    local window = self
    local page = create("ScrollingFrame", {
        Name = tostring(name),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        Size = UDim2.fromScale(1, 1),
        CanvasSize = UDim2.new(),
        ScrollBarThickness = 3,
        ScrollBarImageColor3 = Theme.Accent,
        Visible = #self.Tabs == 0
    }, self.Pages)
    local layout = create("UIListLayout", {
        Padding = UDim.new(0, 9),
        SortOrder = Enum.SortOrder.LayoutOrder
    }, page)
    create("UIPadding", {
        PaddingTop = UDim.new(0, 12),
        PaddingLeft = UDim.new(0, 12),
        PaddingRight = UDim.new(0, 12),
        PaddingBottom = UDim.new(0, 12)
    }, page)
    layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        page.CanvasSize = UDim2.fromOffset(0, layout.AbsoluteContentSize.Y + 24)
    end)

    local button = create("TextButton", {
        Name = tostring(name) .. "Tab",
        Text = tostring(name),
        Font = Enum.Font.Gotham,
        TextSize = 12,
        TextColor3 = #self.Tabs == 0 and Theme.Text or Theme.Muted,
        BackgroundColor3 = #self.Tabs == 0 and Theme.Accent2 or Theme.Surface,
        BorderSizePixel = 0,
        Size = UDim2.new(1, 0, 0, 34),
        AutoButtonColor = false
    }, self.TabBar)
    corner(button, 7)

    local tab = {Window = window, Page = page, Button = button}
    table.insert(self.Tabs, tab)
    button.MouseButton1Click:Connect(function()
        for _, item in ipairs(window.Tabs) do
            item.Page.Visible = false
            tween(item.Button, 0.12, {BackgroundColor3 = Theme.Surface, TextColor3 = Theme.Muted})
        end
        page.Visible = true
        tween(button, 0.12, {BackgroundColor3 = Theme.Accent2, TextColor3 = Theme.Text})
    end)

    function tab:Section(sectionName)
        local section = create("Frame", {
            Name = tostring(sectionName),
            BackgroundColor3 = Theme.Surface,
            BorderSizePixel = 0,
            Size = UDim2.new(1, 0, 0, 42),
            AutomaticSize = Enum.AutomaticSize.Y
        }, self.Page)
        corner(section, 9)
        stroke(section, Theme.Stroke)
        local header = makeText(create("Frame", {
            BackgroundTransparency = 1,
            Position = UDim2.fromOffset(12, 4),
            Size = UDim2.new(1, -24, 0, 28)
        }, section), sectionName, 13, Theme.Text, Enum.Font.GothamBold)
        local holder = create("Frame", {
            BackgroundTransparency = 1,
            Position = UDim2.fromOffset(10, 36),
            Size = UDim2.new(1, -20, 0, 0),
            AutomaticSize = Enum.AutomaticSize.Y
        }, section)
        local list = create("UIListLayout", {
            Padding = UDim.new(0, 7),
            SortOrder = Enum.SortOrder.LayoutOrder
        }, holder)
        list:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            holder.Size = UDim2.new(1, -20, 0, list.AbsoluteContentSize.Y)
            section.Size = UDim2.new(1, 0, 0, list.AbsoluteContentSize.Y + 46)
        end)

        local api = {Window = self.Window, Holder = holder}

        local function row(label, height)
            local frame = create("Frame", {
                BackgroundColor3 = Theme.Surface2,
                BorderSizePixel = 0,
                Size = UDim2.new(1, 0, 0, height or 36)
            }, holder)
            corner(frame, 7)
            local textLabel = makeText(create("Frame", {
                BackgroundTransparency = 1,
                Position = UDim2.fromOffset(10, 0),
                Size = UDim2.new(1, -20, 1, 0)
            }, frame), label, 12, Theme.Text)
            return frame, textLabel
        end

        function api:Label(options)
            options = type(options) == "table" and options or {Text = options}
            local frame = row(options.Text or "Label", options.Height or 32)
            return frame
        end

        function api:Button(options)
            options = options or {}
            local frame = row(options.Title or "Button", 36)
            local button = create("TextButton", {
                BackgroundTransparency = 1,
                Text = "",
                Size = UDim2.fromScale(1, 1)
            }, frame)
            button.MouseButton1Click:Connect(function()
                if type(options.Callback) == "function" then task.spawn(options.Callback) end
            end)
            return {Instance = frame}
        end

        function api:Toggle(options)
            options = options or {}
            local id = options.Id
            local value = options.Default == true
            local frame, label = row(options.Title or "Toggle", 36)
            local switch = create("TextButton", {
                Text = "",
                AutoButtonColor = false,
                BackgroundColor3 = value and Theme.Accent or Theme.Stroke,
                Position = UDim2.new(1, -48, 0.5, -9),
                Size = UDim2.fromOffset(36, 18)
            }, frame)
            corner(switch, 9)
            local knob = create("Frame", {
                BackgroundColor3 = Color3.new(1, 1, 1),
                Position = value and UDim2.fromOffset(20, 2) or UDim2.fromOffset(2, 2),
                Size = UDim2.fromOffset(14, 14),
                BorderSizePixel = 0
            }, switch)
            corner(knob, 7)
            local function set(newValue, silent)
                value = newValue == true
                tween(switch, 0.15, {BackgroundColor3 = value and Theme.Accent or Theme.Stroke})
                tween(knob, 0.15, {Position = value and UDim2.fromOffset(20, 2) or UDim2.fromOffset(2, 2)})
                window.Values[id or options.Title or ""] = value
                if not silent and type(options.Callback) == "function" then task.spawn(options.Callback, value) end
            end
            switch.MouseButton1Click:Connect(function() set(not value) end)
            if id then window:_register(id, value, set) end
            return {Instance = frame, Get = function() return value end, Set = set}
        end

        function api:Slider(options)
            options = options or {}
            local id = options.Id
            local min = tonumber(options.Min) or 0
            local max = tonumber(options.Max) or 100
            if max < min then max = min end
            local value = math.clamp(tonumber(options.Default) or min, min, max)
            local frame = row((options.Title or "Slider") .. "  [" .. tostring(value) .. "]", 54)
            local label = frame:FindFirstChildOfClass("Frame"):FindFirstChildOfClass("TextLabel")
            local bar = create("TextButton", {
                Text = "",
                AutoButtonColor = false,
                BackgroundColor3 = Theme.Stroke,
                Position = UDim2.new(0, 10, 1, -17),
                Size = UDim2.new(1, -20, 0, 6)
            }, frame)
            corner(bar, 4)
            local fill = create("Frame", {
                BackgroundColor3 = Theme.Accent,
                BorderSizePixel = 0,
                Size = UDim2.new(max == min and 0 or (value - min) / (max - min), 0, 1, 0)
            }, bar)
            corner(fill, 4)
            local function set(newValue, silent)
                value = math.clamp(tonumber(newValue) or min, min, max)
                local ratio = max == min and 0 or (value - min) / (max - min)
                fill.Size = UDim2.new(ratio, 0, 1, 0)
                label.Text = (options.Title or "Slider") .. "  [" .. tostring(value) .. "]"
                window.Values[id or options.Title or ""] = value
                if not silent and type(options.Callback) == "function" then task.spawn(options.Callback, value) end
            end
            local function update(input)
                local ratio = math.clamp((input.Position.X - bar.AbsolutePosition.X) / math.max(bar.AbsoluteSize.X, 1), 0, 1)
                set(math.floor((min + (max - min) * ratio) * 100 + 0.5) / 100)
            end
            bar.MouseButton1Down:Connect(update)
            bar.MouseButton1Up:Connect(function() end)
            bar.MouseMoved:Connect(function(x)
                if UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1) then
                    update({Position = Vector2.new(x, 0)})
                end
            end)
            if id then window:_register(id, value, set) end
            return {Instance = frame, Get = function() return value end, Set = set}
        end

        function api:Dropdown(options)
            options = options or {}
            local id = options.Id
            local items = options.Options or {}
            local value = options.Default
            local frame = row(options.Title or "Dropdown", 36)
            local choice = create("TextButton", {
                BackgroundTransparency = 1,
                Text = tostring(value or "Select"),
                TextColor3 = Theme.Muted,
                TextSize = 11,
                Font = Enum.Font.Gotham,
                TextXAlignment = Enum.TextXAlignment.Right,
                Position = UDim2.new(0.4, 0, 0, 0),
                Size = UDim2.new(0.56, -10, 1, 0)
            }, frame)
            local function set(newValue, silent)
                value = newValue
                choice.Text = tostring(value)
                window.Values[id or options.Title or ""] = value
                if not silent and type(options.Callback) == "function" then task.spawn(options.Callback, value) end
            end
            choice.MouseButton1Click:Connect(function()
                if #items == 0 then return end
                local index = table.find(items, value) or 0
                set(items[index % #items + 1])
            end)
            if id then window:_register(id, value, set) end
            return {Instance = frame, Get = function() return value end, Set = set, Refresh = function(newItems) items = newItems or {} end}
        end

        function api:Input(options)
            options = options or {}
            local id = options.Id
            local value = tostring(options.Default or "")
            local frame = row(options.Title or "Input", 38)
            local box = create("TextBox", {
                BackgroundColor3 = Theme.Background,
                BorderSizePixel = 0,
                ClearTextOnFocus = false,
                Text = value,
                PlaceholderText = options.Placeholder or "Enter text",
                PlaceholderColor3 = Theme.Muted,
                TextColor3 = Theme.Text,
                TextSize = 11,
                Font = Enum.Font.Gotham,
                Position = UDim2.new(0.42, 0, 0, 5),
                Size = UDim2.new(0.56, -10, 1, -10)
            }, frame)
            corner(box, 5)
            local function set(newValue, silent)
                value = tostring(newValue or "")
                box.Text = value
                window.Values[id or options.Title or ""] = value
                if not silent and type(options.Callback) == "function" then task.spawn(options.Callback, value) end
            end
            box.FocusLost:Connect(function() set(box.Text) end)
            if id then window:_register(id, value, set) end
            return {Instance = frame, Get = function() return value end, Set = set}
        end

        function api:Keybind(options)
            options = options or {}
            local id = options.Id
            local value = normalizeKey(options.Default) or Enum.KeyCode.Unknown
            local frame = row(options.Title or "Keybind", 36)
            local keyButton = create("TextButton", {
                BackgroundColor3 = Theme.Background,
                Text = value.Name,
                TextColor3 = Theme.Text,
                TextSize = 11,
                Font = Enum.Font.Gotham,
                Position = UDim2.new(1, -94, 0, 5),
                Size = UDim2.fromOffset(84, 26)
            }, frame)
            corner(keyButton, 5)
            local listening = false
            local function set(newValue, silent)
                value = normalizeKey(newValue) or Enum.KeyCode.Unknown
                keyButton.Text = value.Name
                window.Values[id or options.Title or ""] = value.Name
                if not silent and type(options.Callback) == "function" then task.spawn(options.Callback, value) end
            end
            keyButton.MouseButton1Click:Connect(function()
                listening = true
                keyButton.Text = "..."
            end)
            local connection = UserInputService.InputBegan:Connect(function(input)
                if listening and input.UserInputType == Enum.UserInputType.Keyboard then
                    listening = false
                    set(input.KeyCode)
                elseif not listening and input.KeyCode == value and type(options.Callback) == "function" then
                    task.spawn(options.Callback, value)
                end
            end)
            if id then window:_register(id, value.Name, set) end
            return {Instance = frame, Get = function() return value end, Set = set, Disconnect = function() connection:Disconnect() end}
        end

        return api
    end

    return tab
end

return UNiHub
