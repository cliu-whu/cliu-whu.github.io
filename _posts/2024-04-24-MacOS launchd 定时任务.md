---
layout: post
title: "MacOS launchd 定时任务"
categories: Unix_Linux
---

> macOS 使用 launchd 进程来管理守护进程和代理，而您还可以用它来运行 shell 脚本。您不与 launchd 直接交互，而是使用 launchctl 工具来载入或卸载 launchd 守护进程和代理。

# 1 简介
首先编写自动化任务脚本，随后在特定目录下添加一个运行该脚本的plist文件，可以通过终端中的launchctl命令与launchd服务交互从而立刻载入该plist文件。<br />自动化脚本方式：

1. Shell脚本，用于自动化命令行命令，与app交互有限
2. 快捷指令，用于自动化第一方app操作，不能与第三方app交互
3. AppleScript脚本，用于自动化第一方与部分第三方app交互
4. Automator，可以录制操作并重现，是模拟鼠标键盘操作，如果窗口被遮挡会点击不上，目测锁屏后不能用（未测试）

这些方式可以相互调用，推荐一二结合或一三结合。<br />plist文件必须存放于指定目录，如下：<br />**~/Library/LaunchAgents **       			Per-user agents provided by the user.<br />**/Library/LaunchAgents **         			Per-user agents provided by the administrator.<br />**/Library/LaunchDaemons**        		System wide daemons provided by the administrator.<br />**/System/Library/LaunchAgents ** 		OS X Per-user agents.<br />**/System/Library/LaunchDaemons**  	OS X System wide daemons.<br />其中LaunchDaemons在用户登录前就会加载，LaunchAgents 登录后就才会加载。/System/Library/LaunchAgents、 /System/Library/LaunchDaemons为操作系统定义的，用户（管理员）不可以修改，往其中移入文件会发现被禁止。<br />/Library/LaunchAgents、/Library/LaunchDaemons是全局的，每个用户都可加载，~/Library/LaunchAgents只有单个用户可加载。
# 2 自动化脚本
Shell脚本可参考Linux，主要介绍快捷指令和AppleScript脚本。
## 2.1 快捷指令
官方的介绍：<br />[Mac 上的“快捷指令”介绍](https://support.apple.com/zh-cn/guide/shortcuts-mac/apdf22b0444c/5.0/mac/12.0)<br />快捷指令的编写非常简单，非常容易上手，以发邮件为例：<br />![截屏2022-10-29 11.20.59.jpeg](/assets/png/2024-04-24-MacOS launchd 定时任务-1.jpeg)

可以在快捷指令中运行，或终端中通过`shortcuts`命令运行。
## 2.2 AppleScript脚本
官方的介绍：<br />[在 Mac 上使用 AppleScript 和“终端”自动执行任务](https://support.apple.com/zh-cn/guide/terminal/trml1003/mac)<br />也可以参考本人文档：<br />[AppleScript介绍](/Unix_Linux/2024/04/24/AppleScript介绍.html)<br />AppleScript脚本采用Mac上的脚本编辑器编写，许多第三方app留有接口，将任意支持的app拖移到脚本编辑器app图标上可以打开该app的词典，提供语法参考，例如微信：<br />![截屏2022-10-29 11.37.28.png](/assets/png/2024-04-24-MacOS launchd 定时任务-2.png)<br />以下是发送邮件的脚本示例：
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
在命令行通过`osascript`命令可运行脚本。
# 3 plist文件
plist文件采用xml格式，例：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>osascript.mail</string>
    <key>ProgramArguments</key>
    <array>
        <string>osascript</string>
        <string>/Users/cliu/Library/Mobile Documents/com~apple~ScriptEditor2/Documents/mail.scpt</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Minute</key>
        <integer>0</integer>
        <key>Hour</key>
        <integer>12</integer>
    </dict>
    <key>StandardOutPath</key>
    <string>/Users/cliu/Documents/log/myjob.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/cliu/Documents/log/myjob.log</string>
</dict>
</plist>
```
该进程名称为osascript.mail，执行的操作是每天12时0分执行桌面上的mail.scpt，输出与错误信息输出到myjob.log中。plist文件编写规则：在key后面跟上它的值即可，其中`<key>Label</key>`表示key为Label，`<string>osascript.mail</string>`表示Label的值是字符串 'osascript.mail'。以下列举了一些key：<br />**Label**						进程的名称<br />**ProgramArguments**			使用到程序和参数，第一行为程序，后续为参数<br />**StartCalendarInterval**		执行的时间<br />**StandardOutPath**			标准输出文件<br />**StandardErrorPath**			错误信息输出文件<br />这个网站比较详细：[Launchd，如何在Mac上运行服务](https://yishanhe.net/dive-into-launchd/)<br />更多可参考官方手册，通过终端执行`man launchd.plist`查阅，值的类型定义（integer、string等）通过`man plist`查阅。
# 4 launchctl命令
以~/Library/LaunchAgents下osascript.mai.plist为例：
```shell
加载osascript.mai.plist：
launchctl load /Users/cliu/Library/LaunchAgents/osascript.mail.plist
查看该代理进程情况：
launchctl list | grep 'script.mail'
卸载osascript.mai.plist：
launchctl unload /Users/cliu/Library/LaunchAgents/osascript.mail.plist
立刻执行（忽略定时）：
launchctl start osascript.mail
```
更多可参考官方手册，通过终端执行`man launchctl`查阅。
# 5 注意

1. plist文件推荐放在LaunchAgents下，经过我测试放在LaunchDaemons下只能运行Shell脚本，运行快捷指令或AppleScript脚本会报错，即使通过Shell脚本调用它们也不行。
2. plist文件中字符串不要加引号，如果字符串是路径且包含空格，不要使用转义符，原封不动复制过来即可。
3. 从快捷指令运行Shell脚本时，如果脚本出错则后续指令不会执行，有时候需要输出错误信息并继续执行，则应该将错误输出重定向到标准输出中，并忽略错误继续执行，即`command 2>&1 || true`。
4. 当屏幕锁定时，从快捷指令无法运行Shell脚本，高级设置中勾选了允许运行脚本仍报错显示不能运行脚本，因此建议从Shell脚本中运行快捷指令。
5. AppleScript有些第三方app不提供词典，或提供的语法没有用（如微信），可以借助System Events操作。
