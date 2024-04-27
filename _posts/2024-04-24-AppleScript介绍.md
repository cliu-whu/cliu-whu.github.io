---
layout: post
title: "AppleScript介绍"
categories: Unix-like
---

> AppleScript 是一种功能强大的、通用的脚本语言，它内嵌在 OS X 中。可以使用 AppleScript 创建快捷、自动执行重复任务，甚至制作可以节省您很多时间的自定应用程序。AppleScript 可让您控制可脚本控制的应用程序。

# 1 简介
利用AppleScript，可实现Mac打开MacOS的自动化操作，与Shell、快捷指令、Automator相比，有不可替代的优势：

- AppleScript可操作第三方软件，是Shell、快捷指令不能实现的；
- AppleScript操作可后台运行，而Automator是模拟鼠标操作，不可后台运行。

用Mac自带应用“脚本编辑器”即可编写、编译、运行AppleScript脚本，或于终端执行osascript命令可运行脚本。
# 2 基本逻辑
需要了解三个基本概念：命令（command）、对象（object）和属性（property）。

- 命令：可以理解为动词，表示执行什么操作。
- 对象：可以理解为名次，表示操作的对象，具体化表现为窗口、按钮、文本框等，对象有从属关系，例如多个按钮从属于一个窗口。
- 属性：对象的一些属性，比如名称（name）。

AppleScript内在逻辑即是对象及其属性进行一些操作，对象的指定通过名称或序号（index）实现，如`window "微信 (聊天)"`或`window 1`；部分对象也可以用复数指定所有，如`windows`；对象的从属对象或属性通过`of`衔接以指定，如`button 1 of window 1`。需要注意的是，这些对象从属于当前目标（target），指定`window "微信 (聊天)"`的前提条件是当前目标为`application "WeChat"`（后面会提到如何指定当前目标）。<br />更多查看官方英文手册：[Introduction to AppleScript Language Guide](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptLangGuide/introduction/ASLR_intro.html#//apple_ref/doc/uid/TP40000983-CH208-SW1)
# 3 基本语法
想要了解语法可以查阅词典，将对应的应用移到“脚本编辑器”即可，或于“脚本编辑器”菜单的文件—打开词典...查看。AppleScript基本命令查看官方英文手册：[Commands Reference](https://developer.apple.com/library/archive/documentation/AppleScript/Conceptual/AppleScriptLangGuide/reference/ASLR_cmds.html#//apple_ref/doc/uid/TP40000983-CH216-SW59)。以下介绍几种常用的语法：

- 告诉，意为指定某个对象为当前目标
```shell
tell <对象>
	命令...
end tell
或者单行：tell <对象> to 命令...
```

- 设置变量以及对象属性的值，通过`as`可转类型，如`text`、`integer`、`real`、`boolean`等
```shell
set <变量> to <值> [ as <类型> ]
```

- 获取变量、属性、对象等的值
```shell
get <说明符>
```

- 读取文件，用文件的指定方式有`alias`、`file`、`POSIX file`等，推荐终端路径`POSIX file`
```shell
read <文件>
read POSIX file <文件的终端路径>
```

- 写入文件
```shell
write <数据> to <文件>
```

- 激活，即打开应用并置于前端
```shell
activate <应用>
```

- 运行Shell脚本，第一种属于后台，可获取脚本输出结果；第二种属于前台（会打开终端窗口），输出结果在终端窗口里
```shell
do shell script <脚本命令>
或者：tell application "Terminal" to do script <脚本命令>
```

- 点击，相当于鼠标点击，但不同于Automator，它不会控制鼠标移动
```shell
click <对象>
```

- 键入，相当于键盘输入
```shell
keystroke <按键>
```

- 条件语句
```shell
if <条件> then
	命令...
else
	命令...
end if
```

- 循环语句，更多见[https://help.apple.com/applescript/mac/10.9/?lang=zh-cn#apscrpt2712](https://help.apple.com/applescript/mac/10.9/?lang=zh-cn#apscrpt2712)
```shell
repeat with x from (5) to (25)
  命令...
end repeat
```

# 4 一些例子
### 4.1 用“邮件”应用发送一封邮件

1. 告诉`application "Mail"`；
2. 制作`outgoing message of application "Mail"`（`tell`指定了当前目标所以不写`of application "Mail"`）；
3. 设为变量`theMessage`并指定了属性；
4. 最后发送。

```shell
set theSubject to "邮件标题"
set recipientName to "test"
set recipientAddress to "123456@qq.com"
set theContent to "我是正文"

tell application "Mail"
	set theMessage to make new outgoing message with properties {subject:theSubject, content:theContent, visible:false}
	tell theMessage
		make new to recipient with properties {name:recipientName, address:recipientAddress}
	end tell
	send theMessage
end tell
```

### 4.2 用微信发消息
微信给了词典，但是并没有任何用处，所以需要使用到系统自带的“System Events”这个应用。这个应用功能非常强大，可以实现操作任意第三方app，可以说AppleScript是离不开它的。例如：

1. 告诉`application "WeChat"`，将`window 1`设置为可视防止“System Events”找不到它；
2. 从文件读取要发送的信息，注意`as «class utf8»`保证了中文不是乱码；
3. 告诉`process "WeChat" of application "System Events"`，脚本中分为了两次告诉，是一个意思；
4. 将其设置为最前面，是为了满足后面键入操作需要；
5. 点击`radio button 1 of window 1`，保证在“聊天”界面而不是“通讯录”等；
6. 循环变量`x`从`1`到`2`，告诉`splitter group 1 of window "微信 (聊天)"`，选择第`x`个聊天对象；
7. 告诉`text area 1 of scroll area 2 of splitter group 1`，即为聊天文本框；
8. 判断聊天文本框的名称是否为指定值，是则从剪切板粘贴内容于此并键入回车键（发送），否则不执行操作；
9. 回到第6步，从而达到遍历前两个聊天对象的目的，遍历的范围是循环处指定。

```shell
tell application "WeChat" to set visible of window 1 to true
set mfile to POSIX file "/Users/XXX/message.txt"
set message to read mfile as «class utf8»
set friend to "文件传输助手"
tell application "System Events"
	tell process "WeChat"
		set frontmost to true
		click radio button 1 of window 1
		repeat with x from (1) to (2)
			tell splitter group 1 of window "微信 (聊天)"
				select row x of table 1 of scroll area 1
				tell text area 1 of scroll area 2 of splitter group 1
					if name = friend then
						set value to message
						keystroke return
					end if
				end tell
			end tell
		end repeat
	end tell
end tell
```
不过遗憾的是，在Mac屏幕锁定之后“System Events”里面的`process`找不到`windows`，因此这个脚本在锁定屏幕后无法使用，也就是说“System Events”功能受限。
# 5 一些全局量

- `pi`：圆周率
- `version`：版本
- `space, tab, return, linefeed, and quote`：
- `me, my`：当前脚本，当前脚本的
- `it, its`：当前目标，当前目标的
- `the clipboard`：剪切板内容
