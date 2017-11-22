

### 类命名

类名应该以单个或多个单词命名，每个单词首字母大写。例如：

~~~
HelloAction
MapAction
~~~

如果一个类是基础类，建议使用 `Base` 做最后一个单词。例如：

~~~
ActionBase
InstanceBase
~~~


### 函数与函数命名

函数与函数的命名基本规则是：

-   第一个词为“动词”或者“介词”，例如 `set`, `get`, `on`, `after` 等。
-   完整的函数名类似 `setPosition`, `getOpacity`, `afterUserSignIn`。

根据函数的不同用途，采用了不同的命名规则：

-   如果是作为基础函数来使用，那么函数名应该参考 Lua 标准库，使用全小写命名。例如：

    ~~~lua
    cc.class()
    cc.printf()
    string.ucfirst()
    ~~~

-   其他情况，函数应该由单个或多个单词命名，除第一个单词外的其他单词首字母大写。例如：

    ~~~lua
    countAction()
    echoAction()
    ~~~

-   如果是供模块或类内部使用的函数或函数，应该在命名前添加 `_` 字符。例如：

    ~~~lua
    _openSession()
    _newRedis()
    ~~~

-   如果是事件处理函数，建议使用 `on` 开头。例如：

    ~~~lua
    onConnected()
    onDisconnected()
    ~~~

-   对外部函数的本地引用，应该使用 `模块名_函数名` 的形式。例如：

    ~~~lua
    local string_format = string.format
    local os_time = os.time
    ~~~

除了以上规则，还建议采用以下的惯例：


-   立即改变对象状态的函数

    命名规范：

    -   **动词** + [名词]
    -   如果单个动词可以明确表示意义，就不需要跟名词。

    示例：

    ~~~lua
    node:move(...)   -- 立即移动到指定位置
    node:rotate(...) -- 立即旋转到指定角度
    node:show()      -- 显示对象
    node:align(...)  -- 对齐对象
    ~~~

-   持续改变对象状态的函数

    命名规范：

    -   **动词** + [名词] + [介词]
    -   如果单个动词可以明确表示意义，就不需要跟名词。
    -   介词通常选择 `to`, `by` 等：
        -   `to` 表示忽略对象的当前状态，最终达到指定状态
        -   `by` 表示以当前状态为基础，改变一定程度，最终状态由当前状态和改变程度决定

    示例：

    ~~~lua
    node:moveTo(...) -- 移动到指定位置
    node:moveBy(...) -- 在当前位置基础上移动一定距离
    ~~~

-   在对象上执行操作

    命名规范：

    -   **动词** + [名词] + [副词 | 介词]
    -   如果单个动词可以明确表示意义，就不需要跟名词。

    示例：

    ~~~lua
    node:add(...)                  -- 添加子对象
    node:addTo(...)                -- 将对象添加到父对象
    node:playAnimationOnce(...)    -- 在对象上播放一次动画
    node:playAnimationForever(...) -- 在对象上持续播放动画
    ~~~


### 变量命名

变量命名应该简洁明了，除第一个单词外的其他单词首字母大写。例如：

~~~lua
local username
local sessionId
~~~

如果是类的内部成员变量，应该在命名前添加 `_` 字符。例如：

~~~lua
self._session
self._count
~~~

如果是用于 `for` 等语法中的占位符变量，可以直接使用 `_` 作为变量名。例如：

~~~lua
for _, v in ipairs(arr) do
    print(v)
end
~~~


### 常量命名

使用全大写，单词之间以 "_" 下划线分隔。例如：

~~~lua
local DEFAULT_DELAY = 1
Constants.DEFAULT_ACTION = "index"
~~~


### 事件命名

事件命名规则与常量相同，采用全大写，单词之间以 "_" 下划线分隔。例如：

~~~lua
local Bear = cc.class("Bear")

Bear.EVENT = table.readonly({
    WALK = "WALK",
    RUN  = "RUN",
})
~~~

使用函数：

~~~lua
local bear = Bear:new()
bear:bind(Bear.EVENT.WALK, cc.handler(self, self.onWalk))
~~~

在定义类的 `EVENT` 事件列表时，使用了 `table.readonly()` 函数：

-   `table.readonly()` 可以将一个 `table` 设置为只读，确保事件列表不会在运行时被修改。
-   其次在运行时如果访问了列表中没有定义的值，也会抛出错误，方便排除 bug。

<br />


## 参数

如果输入参数有重要性优先级，则按照优先级排列。例如：

~~~lua
display.newSprite(filename, x, y)
~~~

如果有要操作的目标，则目标应该作为第一个参数。例如：

~~~lua
transition.move(target, ...)
~~~

<br />


## 返回值

返回值按照函数和函数的功能设计，采用以下规则：

-   对于不涉及具体逻辑的功能性函数或函数，应该只返回一个值。
-   如果有多个值需要返回，应该包装为 `table`。
-   如果函数执行中发生错误，而这个错误是可以恢复或者需要返回给调用者的，应该返回 `nil` 和错误信息字符串。
-   如果是类的函数，并且调用者不需要返回值，可以返回 `self`，以实现链式调用：

    ~~~lua
    obj:doSomething():again()
    ~~~

<br />


## 错误处理

对于错误处理，不同上下文需要不同的机制。基本上按照错误的严重程度来划分：

-   必须打断执行的，使用 `cc.throw()` 函数直接抛出错误信息。
-   允许继续执行的，使用 `cc.printerror()` 函数输出错误信息，然后继续执行。由于 `cc.printerror()` 会打印调用栈，所以在日志里可以看到这个问题发生的具体位置。
-   如果只是需要提醒开发者注意的意外情况，而非错误，应该使用 `cc.printwarn()` 输出警告信息。
-   如果只是单纯的调试信息，使用 `cc.printinfo()` 和 `cc.printdebug()` 输出。两者区别在于会被不同的 `cc.DEBUG` 设定所过滤。
-   如果一个函数或函数在内部发生错误时需要把具体的错误信息返回给调用者，那么应该使用如下形式：

    ~~~lua
    function test(arg)
        if type(arg) == "string" then
            return arg .. " hello"
        else
            return nil, "invalid parameter"
        end
    end

    local result, err = test(arg)
    if not result then
        -- 如果第一个返回值为 nil，则表示发生了错误
        cc.printerror(err)
    else
        ...
    end
    ~~~

<br />


## 定义模块

模块是指一个单独的 `.lua` 文件，并且可以用 `require()` 或者 `cc.import()` 函数来载入。

模块必须定义为一个 `local` `table`，并且在 `.lua` 文件最后 `return` 这个 `table`。例如：

~~~lua
-- external references
local string_format = string.format

-- declare a module
local _M = {}

-- private function references
local _concat

-- exports API
function _M.say(name)
    print(_concat("hello", name))
end

-- private

_concat = function(str1, str2)
    return string_format("%s, %s", str1, str2)
end

-- return the module
return _M
~~~

规范：

-   在源代码最前面添加外部引用
-   以 `_M` 作为定义模块的 `table`
-   所有需要导出的接口，定义为 `_M` 的 `function`
-   如果是仅用于模块内部的变量和函数，全部定义为 `local`
-   内部函数以前引用的形式定义，然后在 `--private` 后实现函数

<br />


## 定义类

类都是定义在模块中。例如：

~~~lua
local gbc = cc.import("#gbc")
local HelloAction = cc.class("HelloAction", gbc.ActionBase)

function HelloAction:sayAction(args)
    ...
end

return HelloAction
~~~

模块中的类与模块定义遵循同样的规范。

<br />
