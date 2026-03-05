# FVMLite ExternLua Modding 指南

本指南旨在为希望为 FVMLite 项目制作 Mod 的开发者提供全面的信息。它将介绍项目结构、Mod 的创建流程以及可用的全局变量和 API。

## 1. 项目结构概览

`ExternLua` 目录是 Lua 脚本项目的根目录。以下是其主要子目录和文件的功能：

```
ExternLua/
├── X64/            # 可能包含 x64 平台相关的二进制文件或库
├── X86/            # 可能包含 x86 平台相关的二进制文件或库
├── libs/           # Lua 库目录
│   ├── json.lua    # JSON 处理库 (dkjson 兼容)
│   ├── stable.lua  # 通用工具库
│   └── tl.lua      # Teal 语言运行时库 (如果使用 Teal 语言编写 Mod)
├── mods/           # Mod 存放目录。每个 Mod 都在其自己的子目录中。
├── system/         # 核心系统逻辑和工具
│   ├── core/       # 核心系统子模块
│   │   ├── maps/   # 地图相关资源或逻辑
│   │   └── maps.lua# 地图处理逻辑
│   ├── event_system.lua  # 游戏事件系统
│   ├── global_var.lua    # 全局变量定义（可能包含其他全局配置或数据）
│   ├── mapManager.lua    # 地图管理逻辑
│   ├── mod_loader.lua    # Mod 加载器逻辑
│   ├── plant_base.lua    # 植物基类定义和相关功能
│   └── system_events.lua # 系统事件的定义或处理
├── main.lua        # Lua 系统的主要入口点，负责初始化和启动
└── version.txt     # 项目或 Lua 系统的版本信息
```

## 2. 如何制作一个 Mod

制作一个 Mod 的基本步骤如下：

### 2.1. 创建 Mod 目录

在 `ExternLua/mods/` 目录下为你的 Mod 创建一个新的文件夹。这个文件夹的名称将作为你的 Mod 的唯一标识符。

**示例**: 如果你的 Mod 叫做 `MyAwesomeMod`，则创建 `ExternLua/mods/MyAwesomeMod/`。

### 2.2. 创建 `manifest.lua`

在你的 Mod 目录 (`ExternLua/mods/MyAwesomeMod/`) 中创建一个名为 `manifest.lua` 的文件。这个文件必须返回一个 Lua 表，其中包含 Mod 的元数据。

```lua
-- ExternLua/mods/MyAwesomeMod/manifest.lua
return {
    name = "MyAwesomeMod",         -- [必需] Mod 的唯一名称，建议与 Mod 目录名一致。
    version = "1.0.0",             -- [必需] Mod 的版本号。
    description = "这是一个非常棒的 Mod，它做了一些很酷的事情！", -- [可选] Mod 的简短描述。
    author = "你的名字/你的团队",      -- [可选] Mod 的作者信息。
    dependencies = {               -- [可选] 如果你的 Mod 依赖于其他 Mod，在此处列出它们的 `name`。
        -- "AnotherModName",
        -- "SomeLibraryMod"
    },
    -- ... 其他自定义元数据
}
```

### 2.3. 创建 `main.lua`

在你的 Mod 目录 (`ExternLua/mods/MyAwesomeMod/`) 中创建一个名为 `main.lua` 的文件。这是你的 Mod 的主要逻辑入口点，它应该导出一个 Lua 表，其中至少包含一个 `init` 函数。

```lua
-- ExternLua/mods/MyAwesomeMod/main.lua
local MyAwesomeMod = {}

--- Mod 的初始化函数。在 Mod 加载时被调用。
--- @param eventSystem table 游戏事件系统对象。
--- @param modName string 当前 Mod 的名称。
function MyAwesomeMod.init(eventSystem, modName)
    game.logMessage(modName .. ": 初始化中...")

    -- 在这里注册事件监听器
    eventSystem:register("OnGameStart", function()
        game.logMessage(modName .. ": 游戏已启动！")
        -- 执行游戏启动时的 Mod 逻辑
    end, 0, modName) -- 优先级0，绑定到当前modName

    -- 注册新的植物类（通过 PlantClassManager）
    -- PlantClassManager:registerClass("MyCustomPlant", {
    --     new = function(self) return setmetatable({name="MyCustomPlant"}, {__index = self}) end,
    --     -- ... 其他植物类方法和属性
    -- }, modName)

    -- 发送 Mod 间消息
    -- eventSystem:sendMessage(modName, "MyCustomEventType", { key = "value" })

    -- 订阅 Mod 间消息
    -- eventSystem:subscribe("AnotherModEventType", function(sender, data)
    --     game.logMessage(modName .. ": 收到来自 " .. sender .. " 的消息：" .. data.key)
    -- end, modName)

    game.logMessage(modName .. ": 初始化完成！")
end

--- Mod 的卸载函数。在 Mod 被卸载或热重载时被调用。
--- 在此函数中执行清理工作，例如取消注册事件监听器、释放资源等。
--- @param eventSystem table 游戏事件系统对象。
--- @param modName string 当前 Mod 的名称。
function MyAwesomeMod.unload(eventSystem, modName)
    game.logMessage(modName .. ": 正在卸载...")

    -- 移除此 Mod 注册的所有事件监听器和订阅
    eventSystem:unregisterByMod(modName)

    game.logMessage(modName .. ": 卸载完成。")
end

return MyAwesomeMod
```

### 2.4. 添加 Mod 逻辑

在 `main.lua` 或你的 Mod 目录下通过 `require` 引入的其他 Lua 文件中编写你的 Mod 逻辑。你可以利用 `eventSystem` 监听游戏事件，使用 `game` 对象与游戏引擎交互，注册新的游戏内容（如植物类），或通过 `eventSystem` 实现 Mod 之间的通信。

### 2.5. 管理 Mod 启用/禁用 (可选)

你可以通过在 `ExternLua/mods/config.json` 文件中配置来启用或禁用特定的 Mod。如果此文件不存在，你需要手动创建它。

```json
// ExternLua/mods/config.json
{
    "enabled": {
        "MyAwesomeMod": true,   // 将 Mod "MyAwesomeMod" 设置为启用
        "AnotherDisabledMod": false // 将 Mod "AnotherDisabledMod" 设置为禁用
    }
}
```
**注意**: 如果 `config.json` 中没有明确指定你的 Mod 状态，它将默认被视为启用。

## 3. 全局变量和 API 参考

以下是你在制作 Mod 时可以访问的全局变量和核心 API：

### 3.1. `_PLANT_CLASSES` (Table)

*   **描述**: 全局植物类注册表。这是一个 Lua 表，键是植物类的名称（字符串），值是植物类的 Lua 表定义。它主要由底层游戏引擎（非 Lua 部分）访问和使用。
*   **用途**: Mod 通常不直接操作此表，而是通过 `PlantClassManager` 间接注册植物类。

### 3.2. `PlantClassManager` (Object)

*   **描述**: 封装了植物类注册、查询和管理逻辑的对象。
*   **重要方法**:
    *   `PlantClassManager:registerClass(className, classTable, modName)`:
        *   `className` (string): 植物类的唯一名称。
        *   `classTable` (table): 包含植物类逻辑的 Lua 表（例如，`new`、`update` 方法，属性定义等）。
        *   `modName` (string): 注册此植物类的 Mod 的名称。
        *   **用途**: Mod 应使用此方法向游戏系统注册新的植物类。

### 3.3. `game` (Object)

*   **描述**: 游戏引擎与 Lua 脚本交互的核心接口对象。提供了许多与游戏核心功能相关的 API。
*   **重要方法**:
    *   `game.getPlatform()` (returns string): 获取当前运行平台（例如 `'windows'`, `'android'`, `'linux'`）。
    *   `game.getExternalStoragePath()` (returns string): 获取外部存储路径（主要用于 Android 平台）。
    *   `game.fileExists(path)` (returns boolean): 检查指定路径的文件是否存在。
    *   `game.dirExists(path)` (returns boolean): 检查指定路径的目录是否存在。
    *   `game.logMessage(message)`: 向游戏日志输出一条消息，用于调试和信息输出。
    *   `game.registerPlantClass(className, classTable)`: （内部方法）直接向游戏引擎注册植物类。Mod 通常通过 `PlantClassManager` 间接调用。
    *   `game.createLuaPlant(luaClassName, ident)` (returns plantObject or nil): 创建一个由 Lua 定义的植物实例。
        *   `luaClassName` (string): 植物类的名称（已通过 `PlantClassManager` 注册）。
        *   `ident` (string): 植物实例的唯一标识符。
    *   `game.createPlant(ident)` (returns plantObject or nil): 创建一个普通的（非 Lua 定义的）植物实例。
    *   `game.registerMap(ident, mapData)`: 注册一个地图。
        *   `ident` (string): 地图的唯一标识符。
        *   `mapData` (string): JSON 格式的地图数据字符串。
    *   `game.getPlantProperty(selfPtr, propName)` (returns any): 获取植物实例 `selfPtr` 的属性 `propName`。
    *   `game.setPlantProperty(selfPtr, propName, value)` (returns boolean): 设置植物实例 `selfPtr` 的属性 `propName` 为 `value`。

### 3.4. `eventSystem` (Object)

*   **描述**: 游戏事件系统对象。允许 Mod 注册事件监听器、触发事件、以及进行 Mod 之间的消息通信。此对象通常作为参数传递给 Mod 的 `init` 和 `unload` 函数。
*   **重要方法**:
    *   `eventSystem:register(eventName, callback, priority, modName)`: 注册一个事件监听器。
        *   `eventName` (string): 要监听的事件名称（例如 `"OnGameStart"`, `"OnPlantCreated"`）。
        *   `callback` (function): 处理事件的回调函数。参数取决于事件类型。
        *   `priority` (number, defaults to `0`): 监听器的优先级。数字越大，越早被调用。
        *   `modName` (string): 注册此监听器的 Mod 的名称。
        *   **用途**: 监听游戏内部发生的各种事件。
    *   `eventSystem:trigger(eventName, ...)` (returns boolean): 触发一个事件，并传递可选参数。
        *   `eventName` (string): 要触发的事件名称。
        *   `...`: 传递给监听器的任意参数。
        *   **返回值**: 如果任何监听器返回 `false`，则返回 `false` (表示事件被阻止)，否则返回 `true`。
        *   **用途**: Mod 可以触发自定义事件供其他 Mod 监听。
    *   `eventSystem:sendMessage(sender, msgType, data)`: 发送一个 Mod 间消息。
        *   `sender` (string): 发送消息的 Mod 名称。
        *   `msgType` (string): 消息的类型（自定义字符串）。
        *   `data` (any): 消息附带的任意数据。
        *   **用途**: 实现 Mod 之间的异步通信。
    *   `eventSystem:subscribe(msgType, callback, modName)`: 订阅特定类型的 Mod 间消息。
        *   `msgType` (string): 要订阅的消息类型。
        *   `callback` (function): 收到消息时调用的回调函数。参数通常是 `(sender, data)`。
        *   `modName` (string): 订阅此消息的 Mod 的名称。
        *   **用途**: 接收其他 Mod 发送的特定类型消息。
    *   `eventSystem:unsubscribe(msgType, modName)`: 取消 Mod 对特定消息类型的订阅。
        *   `msgType` (string): 要取消订阅的消息类型。
        *   `modName` (string): 尝试取消订阅的 Mod 名称。
        *   **用途**: 清理不再需要的消息订阅。
    *   `eventSystem:processMessages()`: 处理消息队列中所有待处理的 Mod 间消息。此函数应由游戏引擎在适当的时机（例如，每帧）调用。
    *   `eventSystem:unregisterByMod(modName)`: 移除 Mod `modName` 注册的所有事件监听器和消息订阅。
        *   `modName` (string): 要清理的 Mod 的名称。
        *   **用途**: 在 Mod 卸载或热重载时进行清理。

### 3.5. `_VERSION` (String)

*   **描述**: Lua 内置的全局变量，表示当前 Lua 解释器的版本字符串。
*   **用途**: 检查 Lua 版本。

### 3.6. `json` (Table)

*   **描述**: Lua JSON 库，用于 JSON 数据的编码和解码。
*   **重要方法**:
    *   `json.decode(jsonString)` (returns table or nil): 将 JSON 字符串解析为 Lua 表。
    *   `json.encode(luaTable)` (returns string): 将 Lua 表编码为 JSON 字符串。
*   **用途**: Mod 处理配置数据、网络通信数据等 JSON 格式的数据。

### 3.7. `tl` (Table)

*   **描述**: Teal 语言相关的库，如果项目使用 Teal 编写，会用到。
*   **重要方法**:
    *   `tl.version()` (returns string): 获取 Teal 版本。
*   **用途**: 如果你的 Mod 使用 Teal 语言编写，可能会用到。

### 3.8. `jit` (Table)

*   **描述**: 仅在 LuaJIT 环境下可用的全局对象，提供了 JIT 编译器相关的信息和控制。
*   **重要属性**:
    *   `jit.version` (string): 获取 LuaJIT 版本。
*   **用途**: 检查 LuaJIT 状态，或进行高级优化。

---

希望这份指南能帮助你开始为 FVMLite 制作 Mod！祝你编码愉快！
