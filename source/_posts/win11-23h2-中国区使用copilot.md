---
title: Windows11 23h2 中国区使用copilot
date: 2023-11-20 10:10:50
tags:
- Windows
- AI
- 电脑人工智能管家
- copilot
---

**！需要魔法！需要魔法！需要魔法**

## 首先Windows更新到 23h2

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9cd1309d54974705ad258898cbb703d4~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2879&h=1799&s=496091&e=png&b=f1f6f9)

更新后任务栏就有一个copilot的图标了，如果没有的话可以修改注册表和在Windows设置里去打开（以上两个步骤是用于让Windows显示copilot图标的，有可能需要一个或两个都需要或两个都不需要，看自身情况）

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9807510c5f8641fc9c663a41b1aca766~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=793&h=95&s=104209&e=png&b=0b4a74)

### 这时候点开copilot还是显示不在服务区内的


## 然后GitHub上下载ViVe工具
链接如下：

[GitHub - thebookisclosed/ViVe: C# library and console app for using new feature control APIs available in Windows 10 version 2004 and newer](https://github.com/thebookisclosed/ViVe)

看描述就是个c#写的库或控制台，用于使用Windows10 2004及更新的版本的的新功能和一些控制api，去右下角下载releases版就行了。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42ed87424b0544dbbbb8d6da74d3be53~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2143&h=1581&s=342185&e=png&b=ffffff)

### 下载下来是这样的，除了第一个aishell文件是我记录命令的txt，其余都是应该有的文件。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/581db666cf7a4ea896eb204e96efb8f1~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1485&h=853&s=66585&e=png&b=ffffff)

## 使用ViVe工具
管理员权限打开cmd，去到ViVe工具的目录，输入以下命令，执行完后出现successfully 类似字样说明执行成功了，然后重启电脑就行。
```shell
ViVeTool.exe /enable /id:44774629,44850061,44776738,42105254,41655236
```
### 注意
- 如果不是管理员身份打开的，命令会出现 “拒绝访问” 的错误
- 如果管理员打开的cmd无法使用cd命令，可添加盘符参数，比如我ViVe工具放在D:\\Program Files\ViVe，管理员权限命令行默认都是C:\\windows\System32，那么要用cd命令切换到D盘如下：
```shell
cd /d D:\\Program Files\ViVe
```

## 重启电脑后
重启电脑后基本就可以，开启魔法，打开任务栏的copilot，刷新一下就能使用了。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/61e9b152e8cc41d4a48e9b05f087aba1~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=719&h=1799&s=175124&e=png&b=f6f6f6)

其实这个聊天/作图什么的我也还没用，但是他能操作我的计算机，帮我管理文件，打开app之类的，这个才是我想要的，现在他能做啥我也还不是很清楚，使用感受以后再更新。

## by the way
整个过程，该开魔法的就开魔法，要重启的自己注意一下要不要关魔法，这些细节自己都注意一下，争取一次成功。然后顺便提一下之前了解到的一个GitHub项目

[GitHub - KillianLucas/open-interpreter: OpenAI's Code Interpreter in your terminal, running locally](https://github.com/KillianLucas/open-interpreter)

这个做的其实也挺符合我想要的，但是这个应该也是要key，那么自然就不得不考虑自身的经济情况了。还有就是界面的问题，看个人喜好了。我主张就是 白嫖 > 官方 > 掌控程度。（收费的不在考虑范围呢哈哈哈）当然还是要尊重开源的，大部分该给的链接都已经给到了，侵删。最后非常感谢他们对人类科技进步做出的贡献。
