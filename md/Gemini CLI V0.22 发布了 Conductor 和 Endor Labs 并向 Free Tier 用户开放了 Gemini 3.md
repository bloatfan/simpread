> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/aLb1WR7Fqq8rTXG7QYoacg)

前言

小伙伴们大家好，我是小溪，见字如面。2025 年 12 月 22 日，Google 发布了 Gemini CLI V0.22.0 版本，向所有免费用户开放了 Gemini 3 系列模型

Gemini CLI 更新地址：https://github.com/google-gemini/gemini-cli/discussions/15488

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D0)

只要是 Free Tier 用户，Gemini CLI 更新到 V0.22.0 及以上版本，开启 Preview Features 就可以使用 Gemini 3 系列模型了，目前除了 Gemini 3 Pro 还开放了 Gemini 3 Flash。

当前使用版本

0.22.4

更新配置

想在最新版本 Gemini CLI 中使用 Gemini 3 系列模型，只需要执行两步操作。

第一步更新 Gemini CLI 到最新版本

```
$ brew upgrade gemini-cli
```

更新完成后，在命令行终端输入命令 gemini -v 查看版本号，只要版本在 0.22.0 版本及以上版本即可

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D1)

第二步开启 Preview Features，在交互式命令中输入 /settings，开启【Preview Features】后保存配置

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D2)

输入 /model 切换到【Auto（Gemini 3）】

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D3)

扩展插件更新

除了 Gemin CLI 的更新，这次还带来了 Conductor 和 Endor Labs 2 个重量级扩展插件的更新

Conductor

按照官方的说法，Conductor 不是简单地编写代码而是将 Gemini CLI 转变为一个主动的项目经理，遵循严格的协议来指定、规划和实施软件功能和 bug 修复，确保每个任务都有一个一致的、高质量的生命周期: Context -> Spec & Plan -> Implement。

Conductor 背后的理念很简单：就是让人能够掌控 AI 生成代码的方向和范围。通过将上下文视作代码之外的受控工件，将代码库转化为单一真实可信的数据源，它以深度持久的项目认知驱动着每次编码交互。

Github 地址：https://github.com/gemini-cli-extensions/conductor

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D4)

安装设置

使用 Conductor 前需要先安装扩展插件，在命令行终端输入如下指令进行安装

```
$ gemini extensions install https://github.com/gemini-cli-extensions/conductor
```

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D5)

在命令行终端输入 gemini extensions list 查看安装列表，看到 Conductor 信息表示安装成功

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D6)

通过扩展信息可以看到 Conductor 扩展默认只提供了一个上下文配置

自定义命令

在交互式命令中输入 /conductor 可以看到 Conductor 提供的自定义命令列表

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D7)

Conductor 提供了 5 个自定义命令，其中前 3 个是核心命令：

*   /conductor:setup：项目初始化、需求背景、技术栈和约束，每个项目运行一次
    
*   /conductor:newTrack：开启一个新的 Track（工作单元），可以是新特性或 bug 修复，生成 spec.md 和 plan.md
    
*   /conductor:implement：根据制定的规范和计划好执行定义的任务
    
*   /conductor:status：显示任务进度状态
    
*   /conductor:revert：根据 Track 回滚任务
    

步骤一：初始化项目约束

可以与 Gemini CLI 充分讨论需求后再执行 / conductor:setup，也可以直接通过 / conductor:setup 进行需求、约束描述

```
/conductor:setup 使用React+Vite+TailWindcss创建一个todo list应用，包含任务的创建、删除、任务检索，使用zustand状态管理，非必要不要引入其他三方库
```

当产品需求存在疑问时，Gemini 会询问我们问题以定义项目的愿景和功能，我们需要根据问题给出对应回答

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D8)

回答一个问题后，Gemini 会根据选择继续询问其他需求问题

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D9)

没有疑问后，Gemini 会为我们起草产品指南，包含 项目背景、目标受众、核心目标 和 主要功能 等，接下来我们可以根据 Gemini 提示选择批准或者继续和 Gemini 讨论调整产品指南

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D10)

经过漫长的问答环节，最终 Gemini 生成的 Conductor 文档结构如下

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D11)

```
conductor/
├── product.md           # 产品定义和目标
├── product-guidelines.md# 产品指南
├── tech-stack.md        # 技术栈说明
├── workflow.md          # 工作流偏好
└── code_styleguides/    # 代码风格指南
```

步骤二：创建新的 Track

创建新功能或者修复问题时，先用 /conductor:newTrack 生成规范和计划，这一步很重要 Conductor 会根据你的描述和项目上下文，自动生成详细的需求文档和实现步骤。

直接输入交互式命令

```
/conductor:newTrack 添加任务完成进度展示功能
```

同样根据 Gemini 提问回答问题，这个过程需要花费一定时间来完成，要确保 AI 理解了你的意图

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D12)

执行完成后，生成的 Track 目录结构如下：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D13)

Conductor 的 Track 会生成两个关键文件：

*   spec.md：详细的需求规范
    
*   plan.md：可执行的任务列表
    

步骤三：实施计划

在交互式命令中输入 /conductor:implement mvp_todo_20260102 开始实施计划，只有一个任务时也可以直接输入 /conductor:implement

Conductor 会按照 plan.md 里的任务顺序逐个完成，每完成一个就打勾标记。中途可以暂停，下次继续时它会从上次的位置继续执行。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2026/01/18/640%3Fwx_fmt%3Dpng%26watermark%3D1%23imgIndex%3D14)

每个计划实施完成后 Gemini 都会进行自测及开发者验证

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbD4UVEvib1ibPjRXnr3HWzQibhrgtrUsMyNfJbY9l7d5ibadOOiccQld0qPHA/640?wx_fmt=png&watermark=1#imgIndex=15)

发现问题可以直接通过自然语言与 Gemini 对话修复

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbDibE3xfsiadgRn6lxcWY9ibu0SaPnw7MFT5qlImUTkUCWSyosXg6dwxhtw/640?wx_fmt=png&watermark=1#imgIndex=16)

功能符合预期也需要同步 Gemini，Gemini 才会继续执行下一步开发任务

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbDDDBV2T99VHslJJaEfyYUiaqCVtUVvvMvvFAcMendWfyKnSE9VYks6oQ/640?wx_fmt=png&watermark=1#imgIndex=17)

Track 都执行完成后会提示归档或移除操作用于标记 Track 完成，通常选择归档即可

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbDayImibxV5xUmBC7cdhmAg2ichYHwl5oprfpY7tNXnibLPUicCfGUnnojLQ/640?wx_fmt=png&watermark=1#imgIndex=18)

归档完成后，Conductor 会将 tracks 中的任务移动到 archive 目录中

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbDTXt07wHpLzrxTFANEicVTxj3DEhWIicUxiagYwNapyQeic3tL21D70DejQ/640?wx_fmt=png&watermark=1#imgIndex=19)

第一个 Track 完成后效果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbD3nQAHy1icAeRrW6dTf8icWDibAngrXB4e9wol2VcGEgdmKe71lA5kribfQ/640?wx_fmt=png&watermark=1#imgIndex=20)

任务操作

Conductor 提供了任务状态查看命令，直接输入交互式命令 /conductor:status 即可查看所有任务执行状态（过程有点慢，通过自然语言实时分析的）

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbD3vNlWBNRWHBz3HKaBEf28jzia6js1djkTAzyTUiajDeSvmqicAYIibA1tA/640?wx_fmt=png&watermark=1#imgIndex=21)

使用 Conductor 有个好处就是不用担心代码版本管理问题，任务变更或者执行完成后 Conductor 会自行创建版本控制和检查点，根据这些 commit 我们可以手动进行代码版本管理

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbDzZ6xxQZp2LvBmSbeibj6es0r1GUBeibMKAm0TllcpibicvA4A2vibnrRnlw/640?wx_fmt=png&watermark=1#imgIndex=22)

不过 Conductor 提供了更优的回滚操作，Conductor 的回滚是根据 Track 来的，而不是按 git commit，这意味着你可以精确地撤销某个功能的实现，而不用担心误伤其他改动。使用也很方便，在交互式命令中直接使用 /conductor:revert 命令

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbDkX8uWiayroVjaib2FogO6DVkLia6E8TxZPbpZlLQ3u2wJE0TXhp0ic70Kg/640?wx_fmt=png&watermark=1#imgIndex=23)

Endor Labs

Endor Labs 集成了 Endor Labs MCP 服务器和 Gemini CLI，通过自然语言交互直接从终端启用高级代码分析和漏洞扫描功能，我们只需要负责提问，代码相关问题交给 Endor Labs 就可以了。

Github 地址：https://github.com/endorlabs/gemini-extension

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbD3uhCia11DYjT28aN2FdlMA1wLL8icwdELbwLJtBDugUEJ4BaKxJCpU0Q/640?wx_fmt=png&watermark=1#imgIndex=24)

使用 Endor Labs 前也需要先进行安装，在命令行终端输入如下指令安装

```
$ gemini extensions install https://github.com/endorlabs/gemini-extension
```

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbDVPIURzCXicXWFPgkZMWQL9vTsicnVG1ictbfwjqzJ6uYgrrkX1jGrEA6g/640?wx_fmt=png&watermark=1#imgIndex=25)

在命令行终端输入 gemini extensions list 查看安装列表，看到 Endor Labs 信息表示安装成功

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbDQTicaH3cFNohlibUmXj7LpzbqTiaBzIrSQicBduErR4SVOxsvvXPfb8Q7g/640?wx_fmt=png&watermark=1#imgIndex=26)

可以看到 Endor Labs 扩展提供了一个上下文配置和一个 endor-labs MCP 服务。Endor Labs 的使用相对要简单很多，直接使用自然语言提出问题即可

```
扫描我的项目查找是否存在安全漏洞
```

Endor Labs 会调用 endor-labs MCP 服务

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbDz6oP1YozjEEEPZibBzhmDIk0r2J5hC5LVPicL0YNx5YDhm3QTYqLTAtQ/640?wx_fmt=png&watermark=1#imgIndex=27)

首次使用 endor-labs MCP 服务会打开浏览器提示进行授权，授权完成后即可正常使用

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbDTb3r2BlAILribT4ApsBNe84CeTsBfA3BuwvhOa1YHRsicKfLQTGqxicmQ/640?wx_fmt=png&watermark=1#imgIndex=28)

扫描完成后即可看到 Endor Lab 的分析报告

![](https://mmbiz.qpic.cn/mmbiz_png/GtKFnezjrYiaY4ekewy4IcqmtRLtm1nbDuMFYMNiclibL9Tib8p7oTE15cmfgZjW1u6IWPLBT9NIbyEibYh81nHSQ8A/640?wx_fmt=png&watermark=1#imgIndex=29)

总结

在复杂项目上，Conductor 的这种先规划后执行的模式是非常有用的，它强制我们在动手之前要想清楚做什么，涉及开发过程的方方面面的问题，而不是边做边改，每个任务开发完成后都会要求我们进行 MVP 测试，该机制从一定程度上避免了 AI 生成代码屎山的可能。但 Conductor 也有缺点，这种上下文驱动方式意味着每次操作都要读取和分析项目上下文，对 Token 的消耗会明显增加。因为 Conductor 每完成一个任务都要求开发者进行自测反馈，在一定程度上拉长了研发周期。

点击关注，及时接收最新消息