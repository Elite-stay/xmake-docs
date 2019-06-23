
在自定义脚本、插件脚本、任务脚本、平台扩展、模板扩展等脚本代码中使用，也就是在类似下面的代码块中，可以使用这些模块接口：

```lua
on_run(function (target)
    print("hello xmake!")
end)
```

<p class="warn">
为了保证外层的描述域尽可能简洁、安全，一般不建议在这个域使用接口和模块操作api，因此大部分模块接口只能脚本域使用，来实现复杂功能。</br>
当然少部分只读的内置接口还是可以在描述域使用的，具体见下表：
</p>

| 接口                                            | 描述                                         | 可使用域                   | 支持版本 |
| ----------------------------------------------- | -------------------------------------------- | -------------------------- | -------- |
| [val](#val)                                     | 获取内置变量的值                             | 脚本域                     | >= 2.1.5 |
| [import](#import)                               | 导入扩展摸块                                 | 脚本域                     | >= 2.0.1 |
| [inherit](#inherit)                             | 导入并继承基类模块                           | 脚本域                     | >= 2.0.1 |
| [ifelse](#ifelse)                               | 类似三元条件判断                             | 描述域、脚本域             | >= 2.0.1 |
| [try-catch-finally](#try-catch-finally)         | 异常捕获                                     | 脚本域                     | >= 2.0.1 |
| [pairs](#pairs)                                 | 用于遍历字典                                 | 描述域、脚本域             | >= 2.0.1 |
| [ipairs](#ipairs)                               | 用于遍历数组                                 | 描述域、脚本域             | >= 2.0.1 |
| [print](#print)                                 | 换行打印终端日志                             | 描述域、脚本域             | >= 2.0.1 |
| [printf](#printf)                               | 无换行打印终端日志                           | 脚本域                     | >= 2.0.1 |
| [cprint](#cprint)                               | 换行彩色打印终端日志                         | 脚本域                     | >= 2.0.1 |
| [cprintf](#cprintf)                             | 无换行彩色打印终端日志                       | 脚本域                     | >= 2.0.1 |
| [format](#format)                               | 格式化字符串                                 | 描述域、脚本域             | >= 2.0.1 |
| [vformat](#vformat)                             | 格式化字符串，支持内置变量转义               | 脚本域                     | >= 2.0.1 |
| [raise](#raise)                                 | 抛出异常中断程序                             | 脚本域                     | >= 2.0.1 |
| [os](#os)                                       | 系统操作模块                                 | 部分只读操作描述域、脚本域 | >= 2.0.1 |
| [io](#io)                                       | 文件操作模块                                 | 脚本域                     | >= 2.0.1 |
| [path](#path)                                   | 路径操作模块                                 | 描述域、脚本域             | >= 2.0.1 |
| [table](#table)                                 | 数组和字典操作模块                           | 描述域、脚本域             | >= 2.0.1 |
| [string](#string)                               | 字符串操作模块                               | 描述域、脚本域             | >= 2.0.1 |
| [process](#process)                             | 进程操作模块                                 | 脚本域                     | >= 2.0.1 |
| [coroutine](#coroutine)                         | 协程操作模块                                 | 脚本域                     | >= 2.0.1 |
| [find_packages](#find_packages)                 | 查找依赖包                                   | 脚本域                     | >= 2.2.5 |

在描述域使用接口调用的实例如下，一般仅用于条件控制：

```lua
-- 扫描当前xmake.lua目录下的所有子目录，以每个目录的名字定义一个task任务
for _, taskname in ipairs(os.dirs("*"), path.basename) do
    task(taskname)
        on_run(function ()
        end)
end
```

上面所说的脚本域、描述域主要是指：

```lua
-- 描述域
target("test")
    
    -- 描述域
    set_kind("static")
    add_files("src/*.c")

    on_run(function (target)
        -- 脚本域
    end)

-- 描述域
```

### val

#### 获取内置变量的值

[内置变量](#内置变量)可以通过此接口直接获取，而不需要再加`$()`的包裹，使用更加简单，例如：

```lua
print(val("host"))
print(val("env PATH"))
local s = val("shell echo hello")
```

而用[vformat](#vformat)就比较繁琐了：

```lua
local s = vformat("$(shell echo hello)")
```

不过`vformat`支持字符串参数格式化，更加强大， 所以应用场景不同。

### import

#### 导入扩展摸块

import的主要用于导入xmake的扩展类库以及一些自定义的类库模块，一般用于：

* 自定义脚本([on_build](#targeton_build), [on_run](#targeton_run) ..)
* 插件开发
* 模板开发
* 平台扩展
* 自定义任务task

导入机制如下：

1. 优先从当前脚本目录下导入
2. 再从扩展类库中导入

导入的语法规则：

基于`.`的类库路径规则，例如：

导入core核心扩展模块

```lua
import("core.base.option")
import("core.project")
import("core.base.task") -- 2.1.5 以前是 core.project.task
import("core")

function main()
    
    -- 获取参数选项
    print(option.get("version"))

    -- 运行任务和插件
    task.run("hello")
    project.task.run("hello")
    core.base.task.run("hello")
end
```

导入当前目录下的自定义模块：

目录结构：

```
plugin
  - xmake.lua
  - main.lua
  - modules
    - hello1.lua
    - hello2.lua
```

在main.lua中导入modules

```lua
import("modules.hello1")
import("modules.hello2")
```

导入后就可以直接使用里面的所有公有接口，私有接口用`_`前缀标示，表明不会被导出，不会被外部调用到。。

除了当前目录，我们还可以导入其他指定目录里面的类库，例如：

```lua
import("hello3", {rootdir = "/home/xxx/modules"})
```

为了防止命名冲突，导入后还可以指定的别名：

```lua
import("core.platform.platform", {alias = "p"})

function main()
 
    -- 这样我们就可以使用p来调用platform模块的plats接口，获取所有xmake支持的平台列表了
    utils.dump(p.plats())
end
```

import不仅可以导入类库，还支持导入的同时作为继承导入，实现模块间的继承关系

```lua
import("xxx.xxx", {inherit = true})
```

这样导入的不是这个模块的引用，而是导入的这个模块的所有公有接口本身，这样就会跟当前模块的接口进行合并，实现模块间的继承。

2.1.5版本新增两个新属性：`import("xxx.xxx", {try = true, anonymous = true})`

try为true，则导入的模块不存在的话，仅仅返回nil，并不会抛异常后中断xmake.
anonymous为true，则导入的模块不会引入当前作用域，仅仅在import接口返回导入的对象引用。

### inherit

#### 导入并继承基类模块

这个等价于[import](#import)接口的`inherit`模式，也就是：

```lua
import("xxx.xxx", {inherit = true})
```

用`inherit`接口的话，会更简洁些：

```lu
inherit("xxx.xxx")
```

使用实例，可以参看xmake的tools目录下的脚本：[clang.lua](#https://github.com/xmake-io/xmake/blob/master/xmake/tools/clang.lua)

这个就是clang工具模块继承了gcc的部分实现。

### ifelse

#### 类似三元条件判断

由于lua没有内置的三元运算符，通过封装`ifelse`接口，实现更加简洁的条件选择：

```lua
local ok = ifelse(a == 0, "ok", "no")
```

### try-catch-finally

#### 异常捕获

lua原生并没有提供try-catch的语法来捕获异常处理，但是提供了`pcall/xpcall`等接口，可在保护模式下执行lua函数。

因此，可以通过封装这两个接口，来实现try-catch块的捕获机制。

我们可以先来看下，封装后的try-catch使用方式：

```lua
try
{
    -- try 代码块
    function ()
        error("error message")
    end,

    -- catch 代码块
    catch 
    {
        -- 发生异常后，被执行
        function (errors)
            print(errors)
        end
    }
}
```

上面的代码中，在try块内部认为引发了一个异常，并且抛出错误消息，在catch中进行了捕获，并且将错误消息进行输出显示。

而finally的处理，这个的作用是对于`try{}`代码块，不管是否执行成功，都会执行到finally块中

也就说，其实上面的实现，完整的支持语法是：`try-catch-finally`模式，其中catch和finally都是可选的，根据自己的实际需求提供

例如：

```lua
try
{
    -- try 代码块
    function ()
        error("error message")
    end,

    -- catch 代码块
    catch 
    {
        -- 发生异常后，被执行
        function (errors)
            print(errors)
        end
    },

    -- finally 代码块
    finally 
    {
        -- 最后都会执行到这里
        function (ok, errors)
            -- 如果try{}中存在异常，ok为true，errors为错误信息，否则为false，errors为try中的返回值
        end
    }
}

```

或者只有finally块：

```lua
try
{
    -- try 代码块
    function ()
        return "info"
    end,

    -- finally 代码块
    finally 
    {
        -- 由于此try代码没发生异常，因此ok为true，errors为返回值: "info"
        function (ok, errors)
        end
    }
}
```

处理可以在finally中获取try里面的正常返回值，其实在仅有try的情况下，也是可以获取返回值的：

```lua
-- 如果没发生异常，result 为返回值："xxxx"，否则为nil
local result = try
{
    function ()
        return "xxxx"
    end
}
```

在xmake的自定义脚本、插件开发中，也是完全基于此异常捕获机制

这样使得扩展脚本的开发非常的精简可读，省去了繁琐的`if err ~= nil then`返回值判断，在发生错误时，xmake会直接抛出异常进行中断，然后高亮提示详细的错误信息。

例如：

```lua
target("test")
    set_kind("binary")
    add_files("src/*.c")

    -- 在编译完ios程序后，对目标程序进行ldid签名
    after_build(function (target))
        os.run("ldid -S %s", target:targetfile())
    end
```

只需要一行`os.run`就行了，也不需要返回值判断是否运行成功，因为运行失败后，xmake会自动抛异常，中断程序并且提示错误

如果你想在运行失败后，不直接中断xmake，继续往下运行，可以自己加个try快就行了：

```lua
target("test")
    set_kind("binary")
    add_files("src/*.c")

    after_build(function (target))
        try
        {
            function ()
                os.run("ldid -S %s", target:targetfile())
            end
        }
    end
```

如果还想捕获出错信息，可以再加个catch:

```lua
target("test")
    set_kind("binary")
    add_files("src/*.c")

    after_build(function (target))
        try
        {
            function ()
                os.run("ldid -S %s", target:targetfile())
            end,
            catch 
            {
                function (errors)
                    print(errors)
                end
            }
        }
    end
```

不过一般情况下，在xmake中写自定义脚本，是不需要手动加try-catch的，直接调用各种api，出错后让xmake默认的处理程序接管，直接中断就行了。。

### pairs

#### 用于遍历字典

这个是lua原生的内置api，在xmake中，在原有的行为上对其进行了一些扩展，来简化一些日常的lua遍历代码。

先看下默认的原生写法：

```lua
local t = {a = "a", b = "b", c = "c", d = "d", e = "e", f = "f"}

for key, val in pairs(t) do
    print("%s: %s", key, val)
end
```

这对于通常的遍历操作就足够了，但是如果我们相对其中每个遍历出来的元素，获取其大写，我们可以这么写：

```lua
for key, val in pairs(t, function (v) return v:upper() end) do
     print("%s: %s", key, val)
end
```

甚至传入一些参数到第二个`function`中，例如：

```lua
for key, val in pairs(t, function (v, a, b) return v:upper() .. a .. b end, "a", "b") do
     print("%s: %s", key, val)
end
```

### ipairs

#### 用于遍历数组

这个是lua原生的内置api，在xmake中，在原有的行为上对其进行了一些扩展，来简化一些日常的lua遍历代码。

先看下默认的原生写法：

```lua
for idx, val in ipairs({"a", "b", "c", "d", "e", "f"}) do
     print("%d %s", idx, val)
end
```

扩展写法类似[pairs](#pairs)接口，例如：

```lua
for idx, val in ipairs({"a", "b", "c", "d", "e", "f"}, function (v) return v:upper() end) do
     print("%d %s", idx, val)
end

for idx, val in ipairs({"a", "b", "c", "d", "e", "f"}, function (v, a, b) return v:upper() .. a .. b end, "a", "b") do
     print("%d %s", idx, val)
end
```

这样可以简化`for`块代码的逻辑，例如我要遍历指定目录，获取其中的文件名，但不包括路径，就可以通过这种扩展方式，简化写法：

```lua
for _, filename in ipairs(os.dirs("*"), path.filename) do
    -- ...
end
```

### print

#### 换行打印终端日志

此接口也是lua的原生接口，xmake在原有行为不变的基础上也进行了扩展，同时支持：格式化输出、多变量输出。

先看下原生支持的方式：

```lua
print("hello xmake!")
print("hello", "xmake!", 123)
```

并且同时还支持扩展的格式化写法：

```lua
print("hello %s!", "xmake")
print("hello xmake! %d", 123)
```

xmake会同时支持这两种写法，内部会去自动智能检测，选择输出行为。

### printf

#### 无换行打印终端日志

类似[print](#print)接口，唯一的区别就是不换行。

### cprint

#### 换行彩色打印终端日志

行为类似[print](#print)，区别就是此接口还支持彩色终端输出，并且支持`emoji`字符输出。

例如：

```lua
    cprint('${bright}hello xmake')
    cprint('${red}hello xmake')
    cprint('${bright green}hello ${clear}xmake')
    cprint('${blue onyellow underline}hello xmake${clear}')
    cprint('${red}hello ${magenta}xmake')
    cprint('${cyan}hello ${dim yellow}xmake')
```

显示结果如下：

![cprint_colors](https://tboox.org/static/img/xmake/cprint_colors.png)

跟颜色相关的描述，都放置在 `${  }` 里面，可以同时设置多个不同的属性，例如：

```
    ${bright red underline onyellow}
```

表示：高亮红色，背景黄色，并且带下滑线

所有这些描述，都会影响后面一整行字符，如果只想显示部分颜色的文字，可以在结束位置，插入`${clear}`清楚前面颜色描述

例如：

```
    ${red}hello ${clear}xmake
```

这样的话，仅仅hello是显示红色，其他还是正常默认黑色显示。

其他颜色属于，我这里就不一一介绍，直接贴上xmake代码里面的属性列表吧：

```lua
    colors.keys = 
    {
        -- 属性
        reset       = 0 -- 重置属性
    ,   clear       = 0 -- 清楚属性
    ,   default     = 0 -- 默认属性
    ,   bright      = 1 -- 高亮
    ,   dim         = 2 -- 暗色
    ,   underline   = 4 -- 下划线
    ,   blink       = 5 -- 闪烁
    ,   reverse     = 7 -- 反转颜色
    ,   hidden      = 8 -- 隐藏文字

        -- 前景色 
    ,   black       = 30
    ,   red         = 31
    ,   green       = 32
    ,   yellow      = 33
    ,   blue        = 34
    ,   magenta     = 35 
    ,   cyan        = 36
    ,   white       = 37

        -- 背景色 
    ,   onblack     = 40
    ,   onred       = 41
    ,   ongreen     = 42
    ,   onyellow    = 43
    ,   onblue      = 44
    ,   onmagenta   = 45
    ,   oncyan      = 46
    ,   onwhite     = 47
```

除了可以色彩高亮显示外，如果你的终端是在macosx下，lion以上的系统，xmake还可以支持emoji表情的显示哦，对于不支持系统，会
忽略显示，例如：

```lua
    cprint("hello xmake${beer}")
    cprint("hello${ok_hand} xmake")
```

上面两行代码，我打印了一个homebrew里面经典的啤酒符号，下面那行打印了一个ok的手势符号，是不是很炫哈。。

![cprint_emoji](https://tboox.org/static/img/xmake/cprint_emoji.png)

所有的emoji表情，以及xmake里面对应的key，都可以通过[emoji符号](http://www.emoji-cheat-sheet.com/)里面找到。。

2.1.7版本支持24位真彩色输出，如果终端支持的话：

```lua
import("core.base.colors")
if colors.truecolor() then
    cprint("${255;0;0}hello")
    cprint("${on;255;0;0}hello${clear} xmake")
    cprint("${bright 255;0;0 underline}hello")
    cprint("${bright on;255;0;0 0;255;0}hello${clear} xmake")
end
```

xmake对于truecolor的检测支持，是通过`$COLORTERM`环境变量来实现的，如果你的终端支持truecolor，可以手动设置此环境变量，来告诉xmake启用truecolor支持。

可以通过下面的命令来启用和测试：

```bash
$ export COLORTERM=truecolor
$ xmake --version
```

2.1.7版本可通过`COLORTERM=nocolor`来禁用色彩输出。

### cprintf

#### 无换行彩色打印终端日志

此接口类似[cprint](#cprint)，区别就是不换行输出。

### format

#### 格式化字符串

如果只是想格式化字符串，不进行输出，可以使用这个接口，此接口跟[string.format](#string-format)接口等价，只是个接口名简化版。

```lua
local s = format("hello %s", xmake)
```

### vformat

#### 格式化字符串，支持内置变量转义

此接口跟[format](#format)接口类似，只是增加对内置变量的获取和转义支持。

```lua
local s = vformat("hello %s $(mode) $(arch) $(env PATH)", xmake)
```

### raise

#### 抛出异常中断程序

如果想在自定义脚本、插件任务中中断xmake运行，可以使用这个接口抛出异常，如果上层没有显示调用[try-catch](#try-catch-finally)捕获的话，xmake就会中断执行，并且显示出错信息。

```lua
if (errors) raise(errors)
```

如果在try块中抛出异常，就会在catch和finally中进行errors信息捕获，具体见：[try-catch](#try-catch-finally)

### find_packages

#### 查找依赖包

此接口是对[lib.detect.find_package](#detect-find_package)接口的封装，提供多个依赖包的查找支持，例如：

```lua
target("test")
    set_kind("binary")
    add_files("src/*.c")
    on_load(function (target)
        target:add(find_packages("openssl", "zlib"))
    end)
```

### os

系统操作模块，属于内置模块，无需使用[import](#import)导入，可直接脚本域调用其接口。

此模块也是lua的原生模块，xmake在其基础上进行了扩展，提供更多实用的接口。

<p class="tip">
os模块里面只有部分readonly接口（例如：`os.getenv`, `os.arch`）是可以在描述域中使用，其他接口只能在脚本域中使用，例如：`os.cp`, `os.rm`等
</p>

| 接口                                            | 描述                                         | 支持版本 |
| ----------------------------------------------- | -------------------------------------------- | -------- |
| [os.cp](#os-cp)                                 | 复制文件或目录                               | >= 2.0.1 |
| [os.mv](#os-mv)                                 | 移动重命名文件或目录                         | >= 2.0.1 |
| [os.rm](#os-rm)                                 | 删除文件或目录树                             | >= 2.0.1 |
| [os.trycp](#os-trycp)                           | 尝试复制文件或目录                           | >= 2.1.6 |
| [os.trymv](#os-trymv)                           | 尝试移动重命名文件或目录                     | >= 2.1.6 |
| [os.tryrm](#os-tryrm)                           | 尝试删除文件或目录树                         | >= 2.1.6 |
| [os.cd](#os-cd)                                 | 进入指定目录                                 | >= 2.0.1 |
| [os.rmdir](#os-rmdir)                           | 删除目录树                                   | >= 2.0.1 |
| [os.mkdir](#os-mkdir)                           | 创建指定目录                                 | >= 2.0.1 |
| [os.isdir](#os-isdir)                           | 判断目录是否存在                             | >= 2.0.1 |
| [os.isfile](#os-isfile)                         | 判断文件是否存在                             | >= 2.0.1 |
| [os.exists](#os-exists)                         | 判断文件或目录是否存在                       | >= 2.0.1 |
| [os.dirs](#os-dirs)                             | 遍历获取指定目录下的所有目录                 | >= 2.0.1 |
| [os.files](#os-files)                           | 遍历获取指定目录下的所有文件                 | >= 2.0.1 |
| [os.filedirs](#os-filedirs)                     | 遍历获取指定目录下的所有文件或目录           | >= 2.0.1 |
| [os.run](#os-run)                               | 安静运行程序                                 | >= 2.0.1 |
| [os.runv](#os-runv)                             | 安静运行程序，带参数列表                     | >= 2.1.5 |
| [os.exec](#os-exec)                             | 回显运行程序                                 | >= 2.0.1 |
| [os.execv](#os-execv)                           | 回显运行程序，带参数列表                     | >= 2.1.5 |
| [os.iorun](#os-iorun)                           | 运行并获取程序输出内容                       | >= 2.0.1 |
| [os.iorunv](#os-iorunv)                         | 运行并获取程序输出内容，带参数列表           | >= 2.1.5 |
| [os.getenv](#os-getenv)                         | 获取环境变量                                 | >= 2.0.1 |
| [os.setenv](#os-setenv)                         | 设置环境变量                                 | >= 2.0.1 |
| [os.tmpdir](#os-tmpdir)                         | 获取临时目录路径                             | >= 2.0.1 |
| [os.tmpfile](#os-tmpfile)                       | 获取临时文件路径                             | >= 2.0.1 |
| [os.curdir](#os-curdir)                         | 获取当前目录路径                             | >= 2.0.1 |
| [os.filesize](#os-filesize)                     | 获取文件大小                                 | >= 2.1.9 |
| [os.scriptdir](#os-scriptdir)                   | 获取脚本目录路径                             | >= 2.0.1 |
| [os.programdir](#os-programdir)                 | 获取xmake安装主程序脚本目录                  | >= 2.1.5 |
| [os.projectdir](#os-projectdir)                 | 获取工程主目录                               | >= 2.1.5 |
| [os.arch](#os-arch)                             | 获取当前系统架构                             | >= 2.0.1 |
| [os.host](#os-host)                             | 获取当前主机系统                             | >= 2.0.1 |

#### os.cp

- 复制文件或目录

行为和shell中的`cp`命令类似，支持路径通配符匹配（使用的是lua模式匹配），支持多文件复制，以及内置变量支持。

例如：

```lua
os.cp("$(scriptdir)/*.h", "$(projectdir)/src/test/**.h", "$(buildir)/inc")
```

上面的代码将：当前`xmake.lua`目录下的所有头文件、工程源码test目录下的头文件全部复制到`$(buildir)`输出目录中。

其中`$(scriptdir)`, `$(projectdir)` 这些变量是xmake的内置变量，具体详情见：[内置变量](#内置变量)的相关文档。

而`*.h`和`**.h`中的匹配模式，跟[add_files](#targetadd_files)中的类似，前者是单级目录匹配，后者是递归多级目录匹配。

此接口同时支持目录的`递归复制`，例如：

```lua
-- 递归复制当前目录到临时目录
os.cp("$(curdir)/test/", "$(tmpdir)/test")
```

<p class="tip">
尽量使用`os.cp`接口，而不是`os.run("cp ..")`，这样更能保证平台一致性，实现跨平台构建描述。
</p>

#### os.mv

- 移动重命名文件或目录

跟[os.cp](#os-cp)的使用类似，同样支持多文件移动操作和模式匹配，例如：

```lua
-- 移动多个文件到临时目录
os.mv("$(buildir)/test1", "$(buildir)/test2", "$(tmpdir)")

-- 文件移动不支持批量操作，也就是文件重命名
os.mv("$(buildir)/libtest.a", "$(buildir)/libdemo.a")
```

#### os.rm

- 删除文件或目录树

支持递归删除目录，批量删除操作，以及模式匹配和内置变量，例如：

```lua
os.rm("$(buildir)/inc/**.h", "$(buildir)/lib/")
```

#### os.trycp

- 尝试复制文件或目录

跟[os.cp](#os-cp)类似，唯一的区别就是，此接口操作失败不会抛出异常中断xmake，而是通过返回值标示是否执行成功。

```lua
if os.trycp("file", "dest/file") then
end
```

#### os.trymv

- 尝试移动文件或目录

跟[os.mv](#os-mv)类似，唯一的区别就是，此接口操作失败不会抛出异常中断xmake，而是通过返回值标示是否执行成功。

```lua
if os.trymv("file", "dest/file") then
end
```

#### os.tryrm

- 尝试删除文件或目录

跟[os.rm](#os-rm)类似，唯一的区别就是，此接口操作失败不会抛出异常中断xmake，而是通过返回值标示是否执行成功。

```lua
if os.tryrm("file") then
end
```

#### os.cd

- 进入指定目录

这个操作用于目录切换，同样也支持内置变量，但是不支持模式匹配和多目录处理，例如：

```lua
-- 进入临时目录
os.cd("$(tmpdir)")
```

如果要离开进入之前的目录，有多种方式：

```lua
-- 进入上级目录
os.cd("..")

-- 进入先前的目录，相当于：cd -
os.cd("-")

-- 进入目录前保存之前的目录，用于之后跨级直接切回
local oldir = os.cd("./src")
...
os.cd(oldir)
```

#### os.rmdir

- 仅删除目录

如果不是目录就无法删除。

#### os.mkdir

- 创建目录

支持批量创建和内置变量，例如：

```lua
os.mkdir("$(tmpdir)/test", "$(buildir)/inc")
```

#### os.isdir

- 判断是否为目录

如果目录不存在，则返回false

```lua
if os.isdir("src") then
    -- ...
end
```

#### os.isfile

- 判断是否为文件

如果文件不存在，则返回false

```lua
if os.isfile("$(buildir)/libxxx.a") then
    -- ...
end
```

#### os.exists

- 判断文件或目录是否存在

如果文件或目录不存在，则返回false

```lua
-- 判断目录存在
if os.exists("$(buildir)") then
    -- ...
end

-- 判断文件存在
if os.exists("$(buildir)/libxxx.a") then
    -- ...
end
```

#### os.dirs

- 遍历获取指定目录下的所有目录

支持[add_files](#targetadd_files)中的模式匹配，支持递归和非递归模式遍历，返回的结果是一个table数组，如果获取不到，返回空数组，例如：

```lua
-- 递归遍历获取所有子目录
for _, dir in ipairs(os.dirs("$(buildir)/inc/**")) do
    print(dir)
end
```

#### os.files

- 遍历获取指定目录下的所有文件

支持[add_files](#targetadd_files)中的模式匹配，支持递归和非递归模式遍历，返回的结果是一个table数组，如果获取不到，返回空数组，例如：

```lua
-- 非递归遍历获取所有子文件
for _, filepath in ipairs(os.files("$(buildir)/inc/*.h")) do
    print(filepath)
end
```

#### os.filedirs

- 遍历获取指定目录下的所有文件和目录

支持[add_files](#targetadd_files)中的模式匹配，支持递归和非递归模式遍历，返回的结果是一个table数组，如果获取不到，返回空数组，例如：

```lua
-- 递归遍历获取所有子文件和目录
for _, filedir in ipairs(os.filedirs("$(buildir)/**")) do
    print(filedir)
end
```

#### os.run

- 安静运行原生shell命令

用于执行第三方的shell命令，但不会回显输出，仅仅在出错后，高亮输出错误信息。

此接口支持参数格式化、内置变量，例如：

```lua
-- 格式化参数传入
os.run("echo hello %s!", "xmake")

-- 列举构建目录文件
os.run("ls -l $(buildir)")
```

<p class="warn">
使用此接口执行shell命令，容易使构建跨平台性降低，对于`os.run("cp ..")`这种尽量使用`os.cp`代替。<br>
如果必须使用此接口运行shell程序，请自行使用[config.plat](#config-plat)接口判断平台支持。
</p>

更加高级的进程运行和控制，见[process](#process)模块接口。

#### os.runv

- 安静运行原生shell命令，带参数列表

跟[os.run](#os-run)类似，只是传递参数的方式是通过参数列表传递，而不是字符串命令，例如：

```lua
os.runv("echo", {"hello", "xmake!"})
```

#### os.exec

- 回显运行原生shell命令

与[os.run](#os-run)接口类似，唯一的不同是，此接口执行shell程序时，是带回显输出的，一般调试的时候用的比较多

#### os.execv

- 回显运行原生shell命令，带参数列表

跟[os.execv](#os-execv)类似，只是传递参数的方式是通过参数列表传递，而不是字符串命令，例如：

```lua
os.execv("echo", {"hello", "xmake!"})
```

#### os.iorun

- 安静运行原生shell命令并获取输出内容

与[os.run](#os-run)接口类似，唯一的不同是，此接口执行shell程序后，会获取shell程序的执行结果，相当于重定向输出。

可同时获取`stdout`, `stderr`中的内容，例如：

```lua
local outdata, errdata = os.iorun("echo hello xmake!")
```

#### os.iorunv

- 安静运行原生shell命令并获取输出内容，带参数列表

跟[os.iorunv](#os-iorunv)类似，只是传递参数的方式是通过参数列表传递，而不是字符串命令，例如：

```lua
local result, errors = os.iorunv("echo", {"hello", "xmake!"})
```

#### os.getenv

- 获取系统环境变量

```lua
print(os.getenv("PATH"))
```

#### os.setenv

- 设置系统环境变量

```lua
os.setenv("HOME", "/tmp/")
```

#### os.tmpdir

- 获取临时目录

跟[$(tmpdir)](#var-tmpdir)结果一致，只不过是直接获取返回一个变量，可以用后续字符串维护。

```lua
print(path.join(os.tmpdir(), "file.txt"))
```

等价于：

```lua
print("$(tmpdir)/file.txt"))
```

#### os.tmpfile

- 获取临时文件路径

用于获取生成一个临时文件路径，仅仅是个路径，文件需要自己创建。

#### os.curdir

- 获取当前目录路径

跟[$(curdir)](#var-curdir)结果一致，只不过是直接获取返回一个变量，可以用后续字符串维护。

用法参考：[os.tmpdir](#os-tmpdir)。

#### os.filesize

- 获取文件大小

```lua
print(os.filesize("/tmp/a"))
```

#### os.scriptdir

- 获取当前描述脚本的路径

跟[$(scriptdir)](#var-scriptdir)结果一致，只不过是直接获取返回一个变量，可以用后续字符串维护。

用法参考：[os.tmpdir](#os-tmpdir)。

#### os.programdir

- 获取xmake安装主程序脚本目录

跟[$(programdir)](#var-programdir)结果一致，只不过是直接获取返回一个变量，可以用后续字符串维护。

#### os.projectdir

- 获取工程主目录

跟[$(projectdir)](#var-projectdir)结果一致，只不过是直接获取返回一个变量，可以用后续字符串维护。

#### os.arch

- 获取当前系统架构

也就是当前主机系统的默认架构，例如我在`linux x86_64`上执行xmake进行构建，那么返回值是：`x86_64`

#### os.host

- 获取当前主机的操作系统

跟[$(host)](#var-host)结果一致，例如我在`linux x86_64`上执行xmake进行构建，那么返回值是：`linux`

### io

io操作模块，扩展了lua内置的io模块，提供更多易用的接口。

| 接口                                            | 描述                                         | 支持版本 |
| ----------------------------------------------- | -------------------------------------------- | -------- |
| [io.open](#io-open)                             | 打开文件用于读写                             | >= 2.0.1 |
| [io.load](#io-load)                             | 从指定路径文件反序列化加载所有table内容      | >= 2.0.1 |
| [io.save](#io-save)                             | 序列化保存所有table内容到指定路径文件        | >= 2.0.1 |
| [io.readfile](#io.readfile)                     | 从指定路径文件读取所有内容                   | >= 2.1.3 |
| [io.writefile](#io.writefile)                   | 写入所有内容到指定路径文件                   | >= 2.1.3 |
| [io.gsub](#io-gsub)                             | 全文替换指定路径文件的内容                   | >= 2.0.1 |
| [io.tail](#io-tail)                             | 读取和显示文件的尾部内容                     | >= 2.0.1 |
| [io.cat](#io-cat)                               | 读取和显示文件的所有内容                     | >= 2.0.1 |
| [io.print](#io-print)                           | 带换行格式化输出内容到文件                   | >= 2.0.1 |
| [io.printf](#io-printf)                         | 无换行格式化输出内容到文件                   | >= 2.0.1 |

#### io.open

- 打开文件用于读写

这个是属于lua的原生接口，详细使用可以参看lua的官方文档：[The Complete I/O Model](https://www.lua.org/pil/21.2.html)

如果要读取文件所有内容，可以这么写：

```lua
local file = io.open("$(tmpdir)/file.txt", "r")
if file then
    local data = file:read("*all")
    file:close()
end
```

或者可以使用[io.readfile](#io.readfile)更加快速地读取。

如果要写文件，可以这么操作：

```lua
-- 打开文件：w 为写模式, a 为追加写模式
local file = io.open("xxx.txt", "w")
if file then

    -- 用原生的lua接口写入数据到文件，不支持格式化，无换行，不支持内置变量
    file:write("hello xmake\n")

    -- 用xmake扩展的接口写入数据到文件，支持格式化，无换行，不支持内置变量
    file:writef("hello %s\n", "xmake")

    -- 使用xmake扩展的格式化传参写入一行，带换行符，并且支持内置变量
    file:print("hello %s and $(buildir)", "xmake")

    -- 使用xmake扩展的格式化传参写入一行，无换行符，并且支持内置变量
    file:printf("hello %s and $(buildir) \n", "xmake")

    -- 关闭文件
    file:close()
end
```

#### io.load

-  从指定路径文件反序列化加载所有table内容

可以从文件中加载序列化好的table内容，一般与[io.save](#io-save)配合使用，例如：

```lua
-- 加载序列化文件的内容到table
local data = io.load("xxx.txt")
if data then

    -- 在终端中dump打印整个table中内容，格式化输出
    utils.dump(data)
end
```

#### io.save

- 序列化保存所有table内容到指定路径文件 

可以序列化存储table内容到指定文件，一般与[io.load](#io-load)配合使用，例如：

```lua
io.save("xxx.txt", {a = "a", b = "b", c = "c"})
```

存储结果为：

```
{
    ["b"] = "b"
,   ["a"] = "a"
,   ["c"] = "c"
}
```

#### io.readfile

- 从指定路径文件读取所有内容

可在不打开文件的情况下，直接读取整个文件的内容，更加的方便，例如：

```lua
local data = io.readfile("xxx.txt")
```

#### io.writefile

- 写入所有内容到指定路径文件

可在不打开文件的情况下，直接写入整个文件的内容，更加的方便，例如：

```lua
io.writefile("xxx.txt", "all data")
```

#### io.gsub

- 全文替换指定路径文件的内容

类似[string.gsub](#string-gsub)接口，全文模式匹配替换内容，不过这里是直接操作文件，例如：

```lua
-- 移除文件所有的空白字符
io.gsub("xxx.txt", "%s+", "")
```

#### io.tail

- 读取和显示文件的尾部内容

读取文件尾部指定行数的数据，并显示，类似`cat xxx.txt | tail -n 10`命令，例如：

```lua
-- 显示文件最后10行内容
io.tail("xxx.txt", 10)
```

#### io.cat

- 读取和显示文件的所有内容

读取文件的所有内容并显示，类似`cat xxx.txt`命令，例如：

```lua
io.cat("xxx.txt")
```

#### io.print

- 带换行格式化输出内容到文件

直接格式化传参输出一行字符串到文件，并且带换行，例如：

```lua
io.print("xxx.txt", "hello %s!", "xmake")
```

#### io.printf

- 无换行格式化输出内容到文件

直接格式化传参输出一行字符串到文件，不带换行，例如：

```lua
io.printf("xxx.txt", "hello %s!\n", "xmake")
```

### path

路径操作模块，实现跨平台的路径操作，这是xmake的一个自定义的模块。

| 接口                                            | 描述                                         | 支持版本 |
| ----------------------------------------------- | -------------------------------------------- | -------- |
| [path.join](#path-join)                         | 拼接路径                                     | >= 2.0.1 |
| [path.translate](#path-translate)               | 转换路径到当前平台的路径风格                 | >= 2.0.1 |
| [path.basename](#path-basename)                 | 获取路径最后不带后缀的文件名                 | >= 2.0.1 |
| [path.filename](#path-filename)                 | 获取路径最后带后缀的文件名                   | >= 2.0.1 |
| [path.extension](#path-extension)               | 获取路径的后缀名                             | >= 2.0.1 |
| [path.directory](#path-directory)               | 获取路径最后的目录名                         | >= 2.0.1 |
| [path.relative](#path-relative)                 | 转换成相对路径                               | >= 2.0.1 |
| [path.absolute](#path-absolute)                 | 转换成绝对路径                               | >= 2.0.1 |
| [path.is_absolute](#path-is_absolute)           | 判断是否为绝对路径                           | >= 2.0.1 |
| [path.splitenv](#path-splitenv)                 | 分割环境变量中的路径                         | >= 2.2.7 |

#### path.join

- 拼接路径

将多个路径项进行追加拼接，由于`windows/unix`风格的路径差异，使用api来追加路径更加跨平台，例如：

```lua
print(path.join("$(tmpdir)", "dir1", "dir2", "file.txt"))
```

上述拼接在unix上相当于：`$(tmpdir)/dir1/dir2/file.txt`，而在windows上相当于：`$(tmpdir)\\dir1\\dir2\\file.txt`

如果觉得这样很繁琐，不够清晰简洁，可以使用：[path.translate](path-translate)方式，格式化转换路径字符串到当前平台支持的格式。

#### path.translate

- 转换路径到当前平台的路径风格

格式化转化指定路径字符串到当前平台支持的路径风格，同时支持`windows/unix`格式的路径字符串参数传入，甚至混合传入，例如：

```lua
print(path.translate("$(tmpdir)/dir/file.txt"))
print(path.translate("$(tmpdir)\\dir\\file.txt"))
print(path.translate("$(tmpdir)\\dir/dir2//file.txt"))
```

上面这三种不同格式的路径字符串，经过`translate`规范化后，就会变成当前平台支持的格式，并且会去掉冗余的路径分隔符。

#### path.basename

- 获取路径最后不带后缀的文件名

```lua
print(path.basename("$(tmpdir)/dir/file.txt"))
```

显示结果为：`file`

#### path.filename

- 获取路径最后带后缀的文件名

```lua
print(path.filename("$(tmpdir)/dir/file.txt"))
```

显示结果为：`file.txt`

#### path.extension

- 获取路径的后缀名

```lua
print(path.extensione("$(tmpdir)/dir/file.txt"))
```

显示结果为：`.txt`

#### path.directory

- 获取路径最后的目录名

```lua
print(path.directory("$(tmpdir)/dir/file.txt"))
```

显示结果为：`dir`

#### path.relative

- 转换成相对路径

```lua
print(path.relative("$(tmpdir)/dir/file.txt", "$(tmpdir)"))
```

显示结果为：`dir/file.txt`

第二个参数是指定相对的根目录，如果不指定，则默认相对当前目录：

```lua
os.cd("$(tmpdir)")
print(path.relative("$(tmpdir)/dir/file.txt"))
```

这样结果是一样的。

#### path.absolute

- 转换成绝对路径

```lua
print(path.absolute("dir/file.txt", "$(tmpdir)"))
```

显示结果为：`$(tmpdir)/dir/file.txt`

第二个参数是指定相对的根目录，如果不指定，则默认相对当前目录：

```lua
os.cd("$(tmpdir)")
print(path.absolute("dir/file.txt"))
```

这样结果是一样的。

#### path.is_absolute

- 判断是否为绝对路径

```lua
if path.is_absolute("/tmp/file.txt") then
    -- 如果是绝对路径
end
```

###### path.splitenv

- 分割环境变量中的路径

```lua
local pathes = path.splitenv(vformat("$(env PATH)"))

-- for windows 
local pathes = path.splitenv("C:\\Windows;C:\\Windows\\System32")
-- got { "C:\\Windows", "C:\\Windows\\System32" }

-- for *nix 
local pathes = path.splitenv("/usr/bin:/usr/local/bin")
-- got { "/usr/bin", "/usr/local/bin" }
```

结果为一个包含了输入字符串中路径的数组。


### table

table属于lua原生提供的模块，对于原生接口使用可以参考：[lua官方文档](https://www.lua.org/manual/5.1/manual.html#5.5)

xmake中对其进行了扩展，增加了一些扩展接口：

| 接口                                            | 描述                                         | 支持版本 |
| ----------------------------------------------- | -------------------------------------------- | -------- |
| [table.join](#table-join)                       | 合并多个table并返回                          | >= 2.0.1 |
| [table.join2](#table-join2)                     | 合并多个table到第一个table                   | >= 2.0.1 |
| [table.unique](#table-unique)                   | 对table中的内容进行去重                      | >= 2.0.1 |
| [table.slice](#table-slice)                     | 获取table的切片                              | >= 2.0.1 |

#### table.join

- 合并多个table并返回

可以将多个table里面的元素进行合并后，返回到一个新的table中，例如：

```lua
local newtable = table.join({1, 2, 3}, {4, 5, 6}, {7, 8, 9})
```

结果为：`{1, 2, 3, 4, 5, 6, 7, 8, 9}`

并且它也支持字典的合并：

```lua
local newtable = table.join({a = "a", b = "b"}, {c = "c"}, {d = "d"})
```

结果为：`{a = "a", b = "b", c = "c", d = "d"}`

#### table.join2

- 合并多个table到第一个table

类似[table.join](#table.join)，唯一的区别是，合并的结果放置在第一个参数中，例如：

```lua
local t = {0, 9}
table.join2(t, {1, 2, 3})
```

结果为：`t = {0, 9, 1, 2, 3}`

#### table.unique

- 对table中的内容进行去重

去重table的元素，一般用于数组table，例如：

```lua
local newtable = table.unique({1, 1, 2, 3, 4, 4, 5})
```

结果为：`{1, 2, 3, 4, 5}`

#### table.slice

- 获取table的切片

用于提取数组table的部分元素，例如：

```lua
-- 提取第4个元素后面的所有元素，结果：{4, 5, 6, 7, 8, 9}
table.slice({1, 2, 3, 4, 5, 6, 7, 8, 9}, 4)

-- 提取第4-8个元素，结果：{4, 5, 6, 7, 8}
table.slice({1, 2, 3, 4, 5, 6, 7, 8, 9}, 4, 8)

-- 提取第4-8个元素，间隔步长为2，结果：{4, 6, 8}
table.slice({1, 2, 3, 4, 5, 6, 7, 8, 9}, 4, 8, 2)
```

### string

字符串模块为lua原生自带的模块，具体使用见：[lua官方手册](https://www.lua.org/manual/5.1/manual.html#5.4)

xmake中对其进行了扩展，增加了一些扩展接口：

| 接口                                            | 描述                                         | 支持版本 |
| ----------------------------------------------- | -------------------------------------------- | -------- |
| [string.startswith](#string-startswith)         | 判断字符串开头是否匹配                       | >= 1.0.1 |
| [string.endswith](#string-endswith)             | 判断字符串结尾是否匹配                       | >= 1.0.1 |
| [string.split](#string-split)                   | 分割字符串                                   | >= 1.0.1 |
| [string.trim](#string-trim)                     | 去掉字符串左右空白字符                       | >= 1.0.1 |
| [string.ltrim](#string-ltrim)                   | 去掉字符串左边空白字符                       | >= 1.0.1 |
| [string.rtrim](#string-rtrim)                   | 去掉字符串右边空白字符                       | >= 1.0.1 |

#### string.startswith

- 判断字符串开头是否匹配

```lua
local s = "hello xmake"
if s:startswith("hello") then
    print("match")
end
```

#### string.endswith

- 判断字符串结尾是否匹配

```lua
local s = "hello xmake"
if s:endswith("xmake") then
    print("match")
end
```

#### string.split

- 分割字符串

v2.2.7版本对这个接口做了改进，以下是对2.2.7之后版本的使用说明。

按模式匹配分割字符串，忽略空串，例如：

```lua
("1\n\n2\n3"):split('\n') => 1, 2, 3
("abc123123xyz123abc"):split('123') => abc, xyz, abc
("abc123123xyz123abc"):split('[123]+') => abc, xyz, abc
```

按纯文本匹配分割字符串，忽略空串（省去了模式匹配，会提升稍许性能），例如：

```lua
("1\n\n2\n3"):split('\n', {plain = true}) => 1, 2, 3
("abc123123xyz123abc"):split('123', {plain = true}) => abc, xyz, abc
```

按模式匹配分割字符串，严格匹配，不忽略空串，例如：

```lua
("1\n\n2\n3"):split('\n', {strict = true}) => 1, , 2, 3
("abc123123xyz123abc"):split('123', {strict = true}) => abc, , xyz, abc
("abc123123xyz123abc"):split('[123]+', {strict = true}) => abc, xyz, abc
```

按纯文本匹配分割字符串，严格匹配，不忽略空串（省去了模式匹配，会提升稍许性能），例如：

```lua
("1\n\n2\n3"):split('\n', {plain = true, strict = true}) => 1, , 2, 3
("abc123123xyz123abc"):split('123', {plain = true, strict = true}) => abc, , xyz, abc
```

限制分割块数

```lua
("1\n\n2\n3"):split('\n', {limit = 2}) => 1, 2\n3
("1.2.3.4.5"):split('%.', {limit = 3}) => 1, 2, 3.4.5
```

#### string.trim

- 去掉字符串左右空白字符

```lua
string.trim("    hello xmake!    ")
```

结果为："hello xmake!"

#### string.ltrim

- 去掉字符串左边空白字符

```lua
string.ltrim("    hello xmake!    ")
```

结果为："hello xmake!    "

#### string.rtrim

- 去掉字符串右边空白字符

```lua
string.rtrim("    hello xmake!    ")
```

结果为："    hello xmake!"

### process

这个是xmake扩展的进程控制模块，用于更加灵活的控制进程，比起：[os.run](#os-run)系列灵活性更高，也更底层。

| 接口                                            | 描述                                         | 支持版本 |
| ----------------------------------------------- | -------------------------------------------- | -------- |
| [process.open](#process-open)                   | 打开进程                                     | >= 2.0.1 |
| [process.wait](#process-wait)                   | 等待进程结束                                 | >= 2.0.1 |
| [process.close](#process-close)                 | 关闭进程对象                                 | >= 2.0.1 |
| [process.waitlist](#process-waitlist)           | 同时等待多个进程                             | >= 2.0.1 |

#### process.open

- 打开进程

通过路径创建运行一个指定程序，并且返回对应的进程对象：

```lua
-- 打开进程，后面两个参数指定需要捕获的stdout, stderr文件路径
local proc = process.open("echo hello xmake!", outfile, errfile)
if proc then

    -- 等待进程执行完成
    --
    -- 参数二为等待超时，-1为永久等待，0为尝试获取进程状态
    -- 返回值waitok为等待状态：1为等待进程正常结束，0为进程还在运行中，-1位等待失败
    -- 返回值status为，等待进程结束后，进程返回的状态码
    local waitok, status = process.wait(proc, -1)

    -- 释放进程对象
    process.close(proc)
end
```

#### process.wait

- 等待进程结束

具体使用见：[process.open](#process-open)

#### process.close

- 关闭进程对象

具体使用见：[process.open](#process-open)

#### process.waitlist

- 同时等待多个进程

```lua
-- 第二个参数是等待超时，返回进程状态列表
for _, procinfo in ipairs(process.waitlist(procs, -1)) do
    
    -- 每个进程的：进程对象、进程pid、进程结束状态码
    local proc      = procinfo[1]
    local procid    = procinfo[2]
    local status    = procinfo[3]

end
```

### coroutine

协程模块是lua原生自带的模块，具使用见：[lua官方手册](https://www.lua.org/manual/5.1/manual.html#5.2)
