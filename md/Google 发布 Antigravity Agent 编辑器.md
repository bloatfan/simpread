> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/PI-NHBextrwQFzIYGlYnKw)

前言

2025 年 11 月 19 日随着 Google 发布 Gemini 3 Pro 模型一并上线了 Antigravity Agent IDE，在 Antigravity 中已默认支持使用 Gemini 3 Pro 相关模型，对 AI 编辑器感兴趣的小伙伴可以看往期内容：

*   [盘点放弃 Cursor 期间发布的新特性，我再次心动了](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247493624&idx=1&sn=7da2b04af474497f84cd3012a4ae1969&scene=21#wechat_redirect)
    
*   [抢先体验字节 AI 编辑器 Trae](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247486531&idx=1&sn=c394cf2369eb9c97d0b4a058542dbbd9&scene=21#wechat_redirect)
    
*   [当下最有可能取代 Cursor 的 AI 编辑器 Kiro 初体验](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247491843&idx=1&sn=9042479ff0f8e9699e6dae60176665c8&scene=21#wechat_redirect)
    
*   [腾讯首个全栈 AI 编辑器 CodeBuddy](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247492083&idx=1&sn=93eb18056cb7bddcb1cacdb875e33f3b&scene=21#wechat_redirect)
    
*   [初识 Verdent AI](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247493909&idx=1&sn=acf8b16b73f1bd03377bb2c8922a3d3a&scene=21#wechat_redirect)
    

当前使用版本

Antigravity Version: 1.11.2

优势

*   支持 macOS、Windows、Linux 多平台
    
*   免费预览阶段，可以免费使用
    

限制

*   macOS 最低支持 12(Monterey)
    
*   Windows 最低支持 Windows 10（64 位）
    
*   Linux 最低支持 Ubuntu 20、Debian 10、Fedora 36、RHEL 8 等
    
*   需要使用 Google 账号登录授权
    
*   需要科学上网环境
    
*   Gemini 3 Pro（high）模型每天有使用额度限制，每 5 小时刷新一次
    
*   有使用区域限制
    

简介

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D0)

官网地址：https://antigravity.google/

安装配置

官方下载地址：https://antigravity.google/download

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D1)

根据安装限制提示，选择自己系统对应的版本下载安装包，安装成功后打开 Antigravity 界面如下：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D2)

点击【Next】继续，可以选择【Start fresh】继续，也可以选择从 VS Code 或者 Cursor 导入插件（据说当前版本导入有问题）

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D3)

点击【Next】继续，选择一个编辑器主题，选择自己喜欢的即可

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D4)

点击【Next】继续，这一步为配置 Antigravity Agent 使用方式

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D5)

提供了 4 种模式：

*   Agent-driven development：代理驱动开发，以 AI 代理为核心，开发者提出高层意图，代理负责拆分任务、制定计划、执行代码、运行测试等一系列工作
    
*   Agent-assisted development（推荐）：代理辅助开发，AI 代理起辅助作用，帮助开发者完成部分开发任务，开发者仍占据主导地位
    
*   Review-driven development：审核驱动的开发，侧重于对开发过程和结果的审核，开发者对权限和每一个关键变更进行把关，保证开发方向和结果的准确性
    
*   Custom configuration：自定义配置
    

我这里使用默认推荐的配置，点击【Next】继续，进入编辑器配置

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D6)

点击【Next】继续进入账号授权配置，点击【Sign in with Google】

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D7)

登录成功后，同意协议完成配置，配置完成即可进入编辑器界面

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D8)

基本使用

编辑器界面

Antigravity IDE 是基于 VS Code 魔改版（懂得都懂），使用过 VS Code 的小伙伴可以无缝衔接。当前 Antigravity 也提供了 Agent Manager、Open Brower、Antigravity Settings 等个性化功能

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D9)

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D10)

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D11)

在编辑器右下角，点击【**Antigravity-Settings**】可以快速开关 **Agent** 和 **Tab** 设置

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D12)

快捷键

Antigravity IDE 提供了一系列快捷键操作，这里以 macOS 为例展示：

*   Cmd+L：打开 Agent 对话框
    
*   Cmd+I：打开内联对话框
    
*   Cmd+E：打开 Agent 管理
    
*   Shift+Cmd+L：打开 Agent 对话框，开启一个新的对话
    
*   ⌥+Enter：Agent 命令行请求权限时允许权限的快捷键，尝试没有效果
    
*   ⌥+Backspace：Agent 命令行请求权限时拒绝权限的快捷键
    
*   Cmd+Enter：命令行请求权限时允许权限的快捷键
    
*   Cmd+Backspace：命令行请求权限时拒绝权限的快捷键
    
*   Tab：代码补全建议
    
*   Escape：取消建议
    

Chat 对话

Antigravity 的聊天窗口还是比较简单的，和传统的对话窗口一样包含 上下文添加、对话模式选择 和 模型切换

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D13)

上下文内容可以上传 图片、工作区上下文 和 工作流，Mentions 和 @ 快捷键操作一致，添加 Workflows 也可以使用 / 快捷键

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D14)

工作区上下文支持多种类型：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D15)

*   Code Context Items：代码块
    

*   Files：文件
    
*   Directories：目录
    
*   MCP servers：MCP 服务
    
*   Rules：提示词规则
    
*   Conversations：对话
    
*   Terminal：命令行终端
    

添加代码块上下文只需选中代码块点击【Chat】或者使用快捷键【Cmd+L】即可将代码块添加到对话上下文

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D16)

使用快捷键 / 可快速添加 Workflows

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D17)

Antigravity 目前支持 Planning 和 Fast 2 种对话模式

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D18)

*   Planning：智能体可以在执行任务前进行规划，适用于深度研究、复杂任务或协作工作
    
*   Fast：智能体将直接执行任务，适用于可以更快完成的简单任务
    

模型目前支持 Gemini 3 Pro (High)、Gemini 3 Pro (Low)、Claude Sonnet 4.5、Claude Sonnet 4.5 (Thinking)、GPT-OSS 120B (Medium)

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D19)

Antigravity 同样支持内联对话，内联对话适合单文件的代码编辑，在文件编辑器中直接使用 Ctrl/Cmd+I 快捷键即可唤起内联对话窗口

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D20)

目前内联对话只能选择 Gemini 2.5 Flash 和 Gemini 3 Pro(Low)

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D21)

在对话框输入任意提示词即可进行聊天对话

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D22)

Agent Manager

Agent Manager 是 Antigravity 的亮点核心，在 Antigravity 里，我们可以一次打开多个工作区，每个工作区可以同时开启多个任务，例如：

*   Agent A 收集整理数据
    
*   Agent B 开发功能
    
*   Agent C 输出项目文档
    

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D23)

多个 Agent 在不同的窗口工作，可以在 Agent Manager 中查看任务状态，也可以随时切换到编辑器查看详细

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D24)

创建多任务管理也比较简单，点击【Start conversation】开启一个新会话，选择工作区后，输入提示词开始任务，如果需要同时执行多个任务可以重复刚刚到操作

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D25)

Antigravity 提供了后台任务能力，如果有任务希望在后台执行可以工作区可以选择【Playground】

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D26)

Antigravity 的后台任务其实就是创建了一个临时工作区，所有后台任务在临时工作区内完成，点击 Agent Manager 右上角的【Open Editor】可以打开临时工作区

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D27)

临时工作区存放在 ~/.gemini/antigravity/playground 目录下

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D28)

Open Browser

Open Browser 是 Antigravity 提供的另一个实用功能，基于最新的 Gemini 3 模型和 Computer Use 提供了对 Chrome 浏览器的操控，可自行打开浏览器预览、获取日志、网络请求、截图等操作。有点类似 Cursor 中的 Browser，但是 Antigravity 的 Open Browser 不支持浏览器内预览及页面标签选择定位，相当于直接 MCP 集成到浏览器中，与网站进行交互和可视化操作

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D29)

Antigravity 使用 Open Browser 打开浏览器时会提示安装 Browser Setup 浏览器扩展，点击【Install extension】进行安装

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D30)

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D31)

安装成功后，不要关闭浏览器，要求 Antigravity 执行浏览器操作，Antigravity 就会直接在 Chrome 打开对应链接获取信息，AI 操作浏览器过程中页面无法操作，这个和 Comet 浏览器很像，对 Comet 浏览器还不了解的小伙伴可以看往期内容：[Perplexity AI Agent 原生浏览器 Comet](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247492675&idx=1&sn=1fbe020cab2ee64f74b1b7de7d9cc623&scene=21#wechat_redirect)

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D32)

Check Point 检查点

Antigravity 提供了 Check Point 功能，鼠标放在每次对话上都会浮现一个【Undo changes up to this point】的按钮

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D33)

点击【Undo changes up to this point】会弹出确认窗口

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D34)

点击【Confirm】确认后会回到当前对话开始前的状态

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D35)

Rules

点击对话框侧边栏的下拉菜单，选择【Customizations】可以进入 Rules 和 Workflows 的配置界面

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D36)

Rules 支持 Global（全局）和 Workspace（项目）2 种配置

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D37)

点击【Global】可以添加全局提示词规则，创建完成后可以看到路径为：～/.gemini/GEMINI.md，居然和 Gemini CLI 是共用的

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D38)

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D39)

点击【Workspce】可以添加项目提示词规则，输入提示词规则名称

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D40)

配置规则触发时机，看着和 Cursor 操作一致，对 Cursor 配置感兴趣的小伙伴可以看往期内容：[【Cursor】Cursor 中的 Project Rules 是什么？](https://mp.weixin.qq.com/s?__biz=MzU5Njg3ODUzOA==&mid=2247488086&idx=1&sn=62b32f6d070501fd9647c6bb5988f02e&scene=21#wechat_redirect)

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D41)

触发时机类型：

*   Manual：手动
    
*   Always On：始终开启
    
*   Model Decision：模型决策
    
*   Glob：全局
    

最后填写 描述 和 提示词 内容后保存，保存后项目提示词规则会存放到 工作区 /.agent/rules 目录下

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D42)

创建后的所有提示词规则都会在【Rules】下展示

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D43)

匹配到规则后 Agent 就会应用对应的规则内容

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D44)

也可以让 AI 帮我们创建 Rules

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26from%3Dappmsg%26watermark%3D1%23imgIndex%3D45)

创建完成后的提示词内容如下

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D46)

Workflows

在 Customizations 配置界面选择【Workflows】进入工作流配置界面，Antigravity 的工作流其实就相当于 Claude Code 中的自定义命令，工作流配置也支持 Global（全局）和 Workspace（项目）2 种配置

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D47)

点击【Global】，输入工作流名称和提示词内容后保存，可以看到全局工作流保存在 ~/.gemini/antigravity/global_workflows  目录下

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D48)

创建的所有工作流都会在【Workflows】下展示

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D49)

在对话窗口直接输入 / 快捷键可以选择工作流

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D50)

执行工作流后效果如下：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D51)

MCP 服务

点击对话框侧边栏的下拉菜单，选择【MCP Servers】可以进入 MCP 服务配置界面，Antigravity 默认提供了 MCP 服务市场，只不过目前数量有限

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D52)

进入 MCP 服务详情，点击【Install】可以安装 MCP

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D53)

安装完成后，可以在 MCP 详情 禁用 和 卸载 MCP

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D54)

点击【Manage MCP Servers】可以对已安装的 MCP 进行统一管理

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D55)

如果 MCP 市场没有所需的 MCP 服务，可以点击【View raw config】打开 MCP JSON 配置文件进行配置

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D56)

配置文件路径为： ~/.gemini/antigravity/mcp_config.json

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D57)

产品定价

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D58)

常见问题

Antigravity 授权成功 IDE 没有成功回调

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D59)

尝试配置科学上网环境，开启 Tun 重试登录

Antigravity IDE 授权一直 loading

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D60)

在网络正常的情况下授权都很快，出现这个问题的小伙伴可以看下这个帖子：https://linux.do/t/topic/1185785，也可以查看 Antigraviity 支持的区域，查询自己登录账号的区域以及更改区域

支持区域：https://antigravity.google/docs/faq，

账号区域查询、更改地址：https://policies.google.com/country-association-form

Model quota limit exceeded

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D61)

如果使用的是 Gemini 3 Pro（High）使用达到一定次数后会出现这个问题，这是到达模型使用次数了，5 个小时后再次重试即可

Model provider overload

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%23imgIndex%3D62)

当前访问人数较多，提供商服务过载了，重试即可

点击关注，及时接收最新消息