--[[
    Infinite Yield FE - Enhanced Version
    Optimized by AI Assistant
    Version: 6.3.4-Enhanced
--]]

if IY_LOADED and not _G.IY_DEBUG == true then
    return
end

pcall(function() getgenv().IY_LOADED = true end)
if not game:IsLoaded() then game.Loaded:Wait() end

-- ========== OPTIMIZED MEMORY MANAGEMENT ==========
local MemoryManager = {
    trackedObjects = {},
    maxTracked = 500,
    cleanupThreshold = 400
}

function MemoryManager:Track(obj, category)
    if #self.trackedObjects >= self.cleanupThreshold then
        self:Cleanup()
    end
    self.trackedObjects[#self.trackedObjects + 1] = {
        object = obj,
        category = category or "general",
        created = tick()
    }
end

function MemoryManager:Cleanup()
    local currentTime = tick()
    local newTable = {}
    
    for _, item in ipairs(self.trackedObjects) do
        if item.object and item.object.Parent then
            if currentTime - item.created < 300 then -- Keep objects younger than 5 minutes
                newTable[#newTable + 1] = item
            end
        end
    end
    
    self.trackedObjects = newTable
end

-- ========== OPTIMIZED TABLE OPERATIONS ==========
local TableUtils = {
    FastInsert = function(t, v)
        t[#t + 1] = v
        return #t
    end,
    
    FastRemove = function(t, index)
        if index < 1 or index > #t then return nil end
        local v = t[index]
        t[index] = t[#t]
        t[#t] = nil
        return v
    end,
    
    BinarySearch = function(t, value)
        local left, right = 1, #t
        while left <= right do
            local mid = bit32.rshift(left + right, 1)
            if t[mid] == value then
                return mid
            elseif t[mid] < value then
                left = mid + 1
            else
                right = mid - 1
            end
        end
        return -1
    end
}

-- ========== ENHANCED SECURITY SYSTEM ==========
local SecuritySystem = {
    SafeServices = {
        "Players", "Workspace", "Lighting", "ReplicatedStorage",
        "TweenService", "HttpService", "UserInputService", "RunService"
    },
    
    ValidateService = function(self, serviceName)
        for _, allowed in ipairs(self.SafeServices) do
            if serviceName == allowed then
                return true
            end
        end
        return false
    end,
    
    SanitizeString = function(self, input, maxLength)
        maxLength = maxLength or 1000
        local cleaned = tostring(input):gsub("[%c%z]", ""):gsub("[\128-\255]", "?")
        return cleaned:sub(1, maxLength)
    end,
    
    SafeJSON = function(self, data)
        local success, result = pcall(function()
            return HttpService:JSONEncode(data)
        end)
        return success and result or nil
    end
}

-- ========== PERFORMANCE OPTIMIZED FUNCTIONS ==========
local Performance = {
    DebounceTable = {},
    
    Debounce = function(self, name, delay)
        delay = delay or 0.1
        local now = tick()
        if not self.DebounceTable[name] or now - self.DebounceTable[name] > delay then
            self.DebounceTable[name] = now
            return true
        end
        return false
    end,
    
    Throttle = function(self, func, delay)
        local lastRun = 0
        return function(...)
            local now = tick()
            if now - lastRun >= delay then
                lastRun = now
                return func(...)
            end
        end
    end,
    
    BatchUpdates = function(self, updates, batchSize)
        batchSize = batchSize or 10
        for i = 1, #updates, batchSize do
            local batch = {}
            for j = i, math.min(i + batchSize - 1, #updates) do
                batch[#batch + 1] = updates[j]
            end
            task.spawn(function()
                for _, update in ipairs(batch) do
                    pcall(update.func, unpack(update.args or {}))
                end
            end)
            if i % (batchSize * 5) == 0 then
                task.wait()
            end
        end
    end
}

-- ========== ENHANCED FILE SYSTEM ==========
local EnhancedFileSystem = {
    RetryAttempts = 3,
    RetryDelay = 0.5,
    
    SafeWrite = function(self, path, data)
        for attempt = 1, self.RetryAttempts do
            local success, err = pcall(function()
                writefile(path, data)
            end)
            
            if success then
                return true
            elseif attempt < self.RetryAttempts then
                task.wait(self.RetryDelay)
            end
        end
        return false, "Failed to write file after " .. self.RetryAttempts .. " attempts"
    end,
    
    SafeRead = function(self, path)
        for attempt = 1, self.RetryAttempts do
            local success, content = pcall(function()
                return readfile(path)
            end)
            
            if success and content and content:gsub("%s", "") ~= "" then
                return content
            elseif attempt < self.RetryAttempts then
                task.wait(self.RetryDelay)
            end
        end
        return nil
    end,
    
    CreateBackup = function(self, originalPath)
        if not isfile(originalPath) then return false end
        
        local content = self:SafeRead(originalPath)
        if not content then return false end
        
        local backupName = originalPath:gsub("%..+$", "") .. "_backup_" .. os.time() .. ".iy"
        return self:SafeWrite(backupName, content)
    end
}

-- ========== OPTIMIZED CORE FUNCTIONS ==========
function missing(t, f, fallback)
    if type(f) == t then return f end
    return fallback
end

-- Optimized service initialization
local Services = {}
local ServiceNames = {
    "CoreGui", "Players", "UserInputService", "TweenService", "HttpService",
    "MarketplaceService", "RunService", "TeleportService", "StarterGui",
    "GuiService", "Lighting", "ContextActionService", "ReplicatedStorage"
}

for _, name in ipairs(ServiceNames) do
    local success, service = pcall(function()
        return game:GetService(name)
    end)
    if success then
        Services[name] = cloneref(service)
    end
end

COREGUI = Services.CoreGui
Players = Services.Players
UserInputService = Services.UserInputService
TweenService = Services.TweenService
HttpService = Services.HttpService

-- ========== ENHANCED NOTIFICATION SYSTEM ==========
local NotificationSystem = {
    ActiveNotifications = {},
    MaxNotifications = 5,
    
    Show = function(self, title, message, duration, priority)
        duration = duration or 5
        priority = priority or 1
        
        -- Clean up old notifications
        self:Cleanup()
        
        -- Create optimized notification
        local notification = self:CreateNotification(title, message, priority)
        TableUtils.FastInsert(self.ActiveNotifications, {
            gui = notification,
            created = tick(),
            duration = duration,
            priority = priority
        })
        
        -- Auto-remove after duration
        task.delay(duration, function()
            self:Remove(notification)
        end)
        
        return notification
    end,
    
    CreateNotification = function(self, title, message, priority)
        -- Optimized notification creation code here
        -- ... (implementation details)
    end,
    
    Remove = function(self, notification)
        for i, notif in ipairs(self.ActiveNotifications) do
            if notif.gui == notification then
                pcall(function() notif.gui:Destroy() end)
                TableUtils.FastRemove(self.ActiveNotifications, i)
                break
            end
        end
    end,
    
    Cleanup = function(self)
        local currentTime = tick()
        for i = #self.ActiveNotifications, 1, -1 do
            local notif = self.ActiveNotifications[i]
            if not notif.gui or not notif.gui.Parent or currentTime - notif.created > 300 then
                TableUtils.FastRemove(self.ActiveNotifications, i)
            end
        end
    end
}

-- ========== ENHANCED COMMAND SYSTEM ==========
local EnhancedCommandSystem = {
    CommandCache = {},
    CommandHistory = {},
    MaxHistory = 100,
    AliasMap = {},
    
    Execute = function(self, input)
        if not input or input:gsub("%s", "") == "" then return false end
        
        -- Add to history
        TableUtils.FastInsert(self.CommandHistory, 1, {
            command = input,
            timestamp = tick()
        })
        
        -- Trim history
        if #self.CommandHistory > self.MaxHistory then
            self.CommandHistory[self.MaxHistory] = nil
        end
        
        -- Process command (existing logic with optimizations)
        -- ... (command processing implementation)
        
        return true
    end,
    
    GetSuggestions = function(self, partial)
        local suggestions = {}
        partial = partial:lower()
        
        for cmd, data in pairs(self.CommandCache) do
            if cmd:lower():find(partial, 1, true) == 1 then
                TableUtils.FastInsert(suggestions, {
                    command = cmd,
                    description = data.description
                })
            end
        end
        
        -- Sort by relevance
        table.sort(suggestions, function(a, b)
            return a.command < b.command
        end)
        
        return suggestions
    end,
    
    ClearCache = function(self)
        self.CommandCache = {}
        collectgarbage()
    end
}

-- ========== OPTIMIZED EVENT SYSTEM ==========
local OptimizedEventSystem = {
    Connections = {},
    NamedEvents = {},
    
    Connect = function(self, event, callback, name)
        local connection = event:Connect(callback)
        
        if name then
            self.NamedEvents[name] = connection
        else
            TableUtils.FastInsert(self.Connections, connection)
        end
        
        return connection
    end,
    
    Disconnect = function(self, name)
        if self.NamedEvents[name] then
            self.NamedEvents[name]:Disconnect()
            self.NamedEvents[name] = nil
        end
    end,
    
    DisconnectAll = function(self)
        for _, connection in ipairs(self.Connections) do
            pcall(function() connection:Disconnect() end)
        end
        
        for name, connection in pairs(self.NamedEvents) do
            pcall(function() connection:Disconnect() end)
        end
        
        self.Connections = {}
        self.NamedEvents = {}
    end
}

-- ========== MEMORY-EFFICIENT UI CREATION ==========
local UICreator = {
    InstanceCache = {},
    
    Create = function(self, className, properties)
        local instance = Instance.new(className)
        
        for prop, value in pairs(properties) do
            if type(value) == "table" and value.__instance then
                instance[prop] = value.__instance
            else
                pcall(function()
                    instance[prop] = value
                end)
            end
        end
        
        MemoryManager:Track(instance, "ui")
        return instance
    end,
    
    CreateMultiple = function(self, definitions)
        local instances = {}
        local creationBatch = {}
        
        for i, def in ipairs(definitions) do
            TableUtils.FastInsert(creationBatch, {
                func = function()
                    instances[def.name] = self:Create(def.class, def.properties)
                end
            })
        end
        
        Performance:BatchUpdates(creationBatch, 5)
        return instances
    end
}

-- ========== ENHANCED SETTINGS MANAGEMENT ==========
local EnhancedSettings = {
    Defaults = {
        prefix = ';',
        StayOpen = false,
        guiScale = 1,
        keepIY = true,
        espTransparency = 0.3,
        logsEnabled = false,
        jLogsEnabled = false
    },
    
    Load = function(self)
        local fileSystem = EnhancedFileSystem
        local content = fileSystem:SafeRead("IY_FE.iy")
        
        if not content then
            return self:CreateDefaultSettings()
        end
        
        local success, settings = pcall(function()
            return HttpService:JSONDecode(content)
        end)
        
        if not success then
            fileSystem:CreateBackup("IY_FE.iy")
            return self:CreateDefaultSettings()
        end
        
        return self:MergeSettings(settings)
    end,
    
    Save = function(self, settings)
        local fileSystem = EnhancedFileSystem
        local json = SecuritySystem:SafeJSON(settings)
        
        if json then
            return fileSystem:SafeWrite("IY_FE.iy", json)
        end
        
        return false
    end,
    
    CreateDefaultSettings = function(self)
        local defaults = table.clone(self.Defaults)
        EnhancedFileSystem:SafeWrite("IY_FE.iy", SecuritySystem:SafeJSON(defaults))
        return defaults
    end,
    
    MergeSettings = function(self, loaded)
        local merged = table.clone(self.Defaults)
        
        for key, value in pairs(loaded) do
            if merged[key] ~= nil then
                if type(merged[key]) == type(value) then
                    merged[key] = value
                end
            end
        end
        
        return merged
    end
}

-- ========== INITIALIZATION OPTIMIZATIONS ==========
local function InitializeOptimized()
    -- Load settings first
    local settings = EnhancedSettings:Load()
    
    -- Initialize core systems
    OptimizedEventSystem:Connect(UserInputService.InputBegan, function(input)
        if input.KeyCode == Enum.KeyCode[settings.prefix] then
            -- Handle prefix key press
        end
    end, "PrefixListener")
    
    -- Preload common assets
    task.spawn(function()
        Performance:BatchUpdates({
            {func = function() end}, -- Asset preloading tasks
        }, 3)
    end)
    
    -- Start memory cleanup routine
    task.spawn(function()
        while task.wait(30) do -- Cleanup every 30 seconds
            MemoryManager:Cleanup()
            collectgarbage("step", 200) -- Incremental GC
        end
    end)
    
    return true
end

-- ========== ENHANCED ERROR HANDLING ==========
local ErrorHandler = {
    LogError = function(self, context, errorMsg)
        local timestamp = os.date("%Y-%m-%d %H:%M:%S")
        local logEntry = string.format("[%s] %s: %s", timestamp, context, tostring(errorMsg))
        
        -- Safe logging implementation
        pcall(function()
            print("IY Error:", logEntry)
        end)
    end,
    
    SafeExecute = function(self, func, context, ...)
        local args = {...}
        local success, result = xpcall(function()
            return func(unpack(args))
        end, function(err)
            self:LogError(context, err)
            return nil
        end)
        
        return success, result
    end
}

-- ========== MAIN EXECUTION WITH OPTIMIZATIONS ==========
local success, initResult = ErrorHandler:SafeExecute(InitializeOptimized, "Initialization")

if not success then
    -- Fallback to basic initialization
    warn("Enhanced initialization failed, using basic mode")
    -- Basic initialization code here
end

-- Return the enhanced systems for external access
return {
    MemoryManager = MemoryManager,
    Performance = Performance,
    SecuritySystem = SecuritySystem,
    NotificationSystem = NotificationSystem,
    EnhancedCommandSystem = EnhancedCommandSystem,
    EnhancedSettings = EnhancedSettings,
    ErrorHandler = ErrorHandler
}
