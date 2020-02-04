# 泥芯(MudCore) - LPMUD 游戏开发框架

LPMUD游戏开发框架核心代码，仅仅包括核心代码，可以在此基础上开发任何MUD，本框架需配合 FluffOS v2019使用。

> 这是一个像汉芯和红芯一样伟大的项目 ^_^

这个项目的诞生源于我的LPC零基础开发教程（https://bbs.mud.ren），在写教程的过程中，发现很多朋友并不关心底层细节，只想能直接用来开发项目，国内绝大多数项目是《东方故事2》底层的，都是在这类MUD基础上修改，优点是可以快速上手，但不够灵活。能不能把底层独立出来，开发任何类型的MUD都可以使用？在思考后，我开始了这个项目，特色是只提供底层代码和接口，不提供任何游戏性的内容，独立于个人项目之外，只需简单配置即可实现个人MUD开发。本项目不考虑旧版的兼容性，只支持 FluffOS v2019 版。

## 框架特色

框架提供大量常用模拟外部函数和最基本的游戏底层（包括指令系统、聊天频道、表情系统和档案存取功能），可以用来快速开发个人的MUD，框架部分独立维护，更新升级不影响个人项目。个人项目目录结构和功能灵活自主，框架提供的功能可灵活选择使用，只需要继承对应模块即可。

## 框架使用说明

把本框架放在你的MUD项目中，推荐保持默认目录名称 `mudcore`，并做如下配置：

1. 运行时配置文件<config.ini>中定义包含文件目录：

```ini
include directories : /mudcore/include:/include
```

2. 全局包含头文件<globals.h>中做如下配置：

```c
// 定义日志目录
#define LOG_DIR "/log/"

// 定义存档目录
#define DATA_DIR "/data/"

// 引用框架头文件(放在最后)
#include <mudcore.h>
```

3. 主控对象文件继承框架对象：

```
inherit CORE_MASTER_OB;
```

4. 模拟外部函数文件继承框架对象：

```
inherit CORE_SIMUL_EFUN_OB;
```

只需要以上配置，你的MUD基本框架就可以正常启动，并自带登录注册功能。

对新MUD开发，如果不想使用框架提供的登录注册功能，请在 <globals.h> 中定义连线对象`LOGIN_OB` 和 `USER_OB` 指向自己实现功能的文件。

个人开发MUD不建议直接修改 `mudcore` 的任何代码，如果需要增加功能，请通过继承和覆盖的方式实现。

如果需要预加载，请定义 `PRELOAD` 并指定文件列表位置，在对应文件列出预加载对象文件。

如果使用系统自带注册登录系统，但想使用个人的欢迎界面，请定义 `MOTD` 并指定位置，如果需要更多的玩家信息初始化，可以定义 `CHAR_D` 并从 `setup()` 方法中初始化。

如果要使用框架提供的指令系统，请在<globals.h>中做如下示例定义：

```c
// 管理员
#define WIZARD "mudren"

// 管理员指令路径
#define CMD_PATH_WIZ ({"/cmds/wiz/"})

// 玩家指令路径
#define CMD_PATH_STD ({"/cmds/std/"})

// 核心指令 look
#define CMD_LOOK "/cmds/std/look"
```

指令需要实现以下方法：

```c
int main(object me, string arg)
{
    // todo 指令功能

    return 1;
}

int help(object me)
{
    // todo 指令使用说明

    return 1;
}
```

## 框架功能介绍

### 目录结构

目录|说明
-|-
docs|开发者文档，包括框架模拟外部函数说明文档等
include|框架头文件
inherit|框架特性继承文件
system|系统文件目录

#### system系统目录

目录|说明
-|-
system/kernel|系统核心文件，包括master对象和simul_efun对象
system/daemons|系统守护进程
system/object|系统核心对象，包括登录对象、基本玩家对象和 VOID 环境

#### 核心对象

名称|文件路径|核心功能
-|-|-
CORE_MASTER_OB|/system/kernel/master|主控对象，负责接收玩家连接请求、安全验证、日志记录等核心功能
CORE_SIMUL_EFUN_OB|/system/kernel/simul_efun|模拟外部函数对象，负责实现模拟外部函数功能
CORE_LOGIN_OB|/system/object/login|示例玩家登陆对象
CORE_USER_OB|/system/object/user|示例玩家对象
CORE_VOID_OB|/system/object/void|临时环境，示例登录房间

#### 系统头文件(Include Headers)

头文件在 `/include/` 目录中，主要是常量宏定义和函数声明，框架基本提供如下头文件：

文件|说明
-|-
ansi.h|定义ANSI 色彩/游标控制码
dbase.h|数据增删改查功能（CORE_DBASE）函数声明
function.h|驱动提供，配合 functionp() 使用
localtime.h|驱动提供，配合 localtime() 使用
mudcore.h|框架全局头文件，需要在项目的 <globals.h> 中引用
mysql.h|定义数据库配置
origin.h|驱动提供，配合 origin() 使用
parser_error.h|驱动提供，定义 parser error
runtime_config.h|驱动提供，配合 get_config() 使用
socket_err.h|驱动提供，配合 socket_error() 使用
socket.h|定义 socket 类型，配合 socket_create() 使用
type.h|驱动提供，配合 typeof() 使用

#### 继承特征(Inherit Objects)

继承文件在 `/inherit/` 目录，实现了核心的功能片段，方便直接使用，具体参考 `/docs/inherit/` 目录。基本提供如下继承文件：

名称|文件|说明
-|-|-
CORE_CLEAN_UP|/inherit/clean_up.c|自动清理接口，实现 clean_up() 方法的垃圾回收功能
CORE_COMMAND|/inherit/command.c|角色指令系统接口，实现生物对象特征功能
CORE_DBASE|/inherit/dbase.c|数据存取功能接口，实现对象参数的增删改查功能
CORE_MESSAGE|/inherit/message.c|玩家信息处理功能接口，实现分页显示
CORE_MOVE|/inherit/move.c|对象移动接口，由角色、物品对象继承，方便移动
CORE_NAME|/inherit/name.c|ID和名称接口，让对象可以被看见(查找)和命名
CORE_NPC|/inherit/npc.c|NPC角色接口，配合环境实现自动回归功能
CORE_OBSAVE|/inherit/obsave.c|系统数据存取接口，配合 DBASE_D 使用
CORE_ROOM|/inherit/room.c|游戏环境标准接口，实现房间核心功能
CORE_SAVE|/inherit/save.c|对象数据存取接口，主要是玩家角色使用存档和读档
CORE_USER|/inherit/user.c|玩家角色接口，玩家对象需继承本接口


#### 守护进程(Daemons)

守护进程是独立出来的系统服务，每个进程有自己的API可以调用，具体参考 `/docs/dameons/` 目录。

名称|文件路径|核心功能
-|-|-
CORE_CHANNEL_D|/system/daemons/channel_d.c|负责处理玩家聊天功能和频道管理
CORE_CHINESE_D|/system/daemons/chinese_d.c|负责处理中、英文对照，以及一些与中文相关的功能。
CORE_DBASE_D|/system/daemons/dbase_d.c|负责系统数据存档处理，默认存储位置 /data/dbase_d.o
CORE_EMOTE_D|/system/daemons/emote_d.c|负责管理游戏表情动作和相应功能
CORE_LOGIN_D|/system/daemons/login_d.c|框架示例登录管理功能，可参考
CORE_TIME_D|/system/daemons/time_d.c|负责控制游戏系统时间和计划任务
CORE_VIRTUAL_D|/system/daemons/virtual_d.c|负责虚拟对象处理

#### 系统函数(Simul-Efuns)

除驱动自带的 efun 外，框架提供了部分模拟外部函数，具体参考 `/docs/simul_efun/` 目录。

simul-efun|简介
-|-
print_r|数组打印功能，格式化输出，开发调试用
atoi|string 类型数字转 int 类型数字
itoc|int 类型数字转 ASCII 字符，如 itoc(65) 返回值为 A
bitCheck|检查数值指定为是否为1
bitSet|返回数值指定位为1的值
bitClear|返回数值指定位为0的值
chinese_number|返回中文数字
chinese|返回指定字符串的中文名称
is_chinese|判断指定字符串是否为中文
is_english|判断指定字符串是否为英文
check_control|判断指定字符串是否包含控制字符
check_space|判断指定字符串是否包含空格
check_return|判断指定字符串是否包含换行符
remove_ansi|清理 ansi 颜色代码
ansi|转换 $HIR$ 等为 ansi 颜色代码
color_cat|彩色输出内容到屏幕
color_len|统计字符串中颜色占用的长度
update_file|以数组的形式返回文件每一行内容 并跳过 `#` 注释行
log_file|记录日志到指定文件
assure_file|建立文件目录
cat|输出文件内容到屏幕
pronoun|返回中文人称代词
msg|智能区域消息
log_time|方便日志记录的 ctime 时间格式
getcid|返回复制对象的ID
debug|随机颜色输出内容
