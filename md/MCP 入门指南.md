> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.nazha.co](https://www.nazha.co/posts/what-is-mcp-and-how-to-use)

> 2025/02/08

2025/02/08

This post was created without the involvement of AI.

MCP，全称 Model Context Protocol，由 [Anthropic 在 2024 年 11 月](https://www.anthropic.com/news/model-context-protocol)推出的，社区共建的开放协议。目的是提供一个通用的开放标准，用来连接 [[LLM]] 和外部数据、行为。

近段时间，LLM 表现出强大的学习能力和规划能力，能够处理更加复杂、抽象的任务。这种强大的能力，让我们看到了 LLM 在 AGI（通用人工智能）中的巨大潜力。但同时，LLM 也不是万能的，它缺失了很多能力。LLM 可以作为 [[智能体]] 的大脑，外部工具就是智能体的手和脚，协助智能体执行决策。

![](https://pub-388464a2ac764e37ba36c5ea17d573ee.r2.dev/a-baisc-agent.png)

> 一个典型的 Agent 的设计，LLM 充当大脑模块，通过多模态输入，处理信息，然后做出决策和规划行动。

MCP 就是想要通过一个开放的协议，为外部工具（或数据源）提供统一和 LLM 交互的统一集成。

> MCP 就是手脚连接身体的 “关节”。

MCP 的[系统架构](https://modelcontextprotocol.io/introduction)如下：

![](https://pub-388464a2ac764e37ba36c5ea17d573ee.r2.dev/mcp-system-ar.png)

主要有以下几个概念：

*   **MCP Host**：LLM 的宿主应用，比如 [Cursor](https://docs.cursor.com/advanced/model-context-protocol)、Cline 等等，**是处理一个或多个 MCP Server 的应用程序**。
*   **MCP Client**：Host 内部专门用于与 MCP Server 建立和维持一对一连接的模块。它负责按照 MCP 协议的规范发送请求、接收响应和处理数据。简单来说，MCP Client 是 Host 内部处理 RPC 通信的 “代理”，专注于与一个 MCP Server 进行标准化的数据、工具或 prompt 的交换。
*   **MCP Server**：提供外部能力或数据的工具，比如实时获取天气、浏览网页等等能力

MCP Client 更多是一个[底层技术术语](https://github.com/modelcontextprotocol/specification/discussions/135)，是关于 MCP Server 连接到 MCP Host 的底层细节，不用过于区分 MCP Host 和 MCP Client。

MCP 的可能性[#](https://www.nazha.co/posts/what-is-mcp-and-how-to-use#mcp-%E7%9A%84%E5%8F%AF%E8%83%BD%E6%80%A7)
-----------------------------------------------------------------------------------------------------------

MCP 当前的发展还处在非常早期的阶段，尤其是其他 LLM 大厂，比如 OpenAI、Google 会不会跟进规范也不确定，社区目前也并不活跃。但它仍然存在一定潜力：

*   提供了简化 Agent 开发的方案
*   相对而言，MCP 不依赖任何特定的 LLM
*   MCP Server 可被扩展，可被复制，可被重用
*   MCP 协议中增加了安全性的控制

MCP Server 的下载安装[#](https://www.nazha.co/posts/what-is-mcp-and-how-to-use#mcp-server-%E7%9A%84%E4%B8%8B%E8%BD%BD%E5%AE%89%E8%A3%85)
-----------------------------------------------------------------------------------------------------------------------------------

目前有[很多客户端](https://modelcontextprotocol.io/clients)已经支持 MCP Servers 的使用，比如 Cline、[Cursor](https://docs.cursor.com/advanced/model-context-protocol)，可以直接在这些客户端中使用[社区开发好的 Servers](https://github.com/modelcontextprotocol/servers/tree/main?tab=readme-ov-file)。下面是一些收集 MCP Servers 的网站：

*   [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers/tree/main?tab=readme-ov-file) 官方社区维护
*   [Smithery](https://smithery.ai/)
*   [PluseMCP](https://www.nazha.co/posts/pulsemcp)

由于 MCP Servers 通常是由官方提供的 [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk) 或 [Python SDK](https://github.com/modelcontextprotocol/python-sdk) 构建的，因此需要在电脑上安装：

1.  [node.js](https://nodejs.org/zh-cn/download)
2.  [uv](https://github.com/astral-sh/uv)

比如，我们需要安装 [fetch](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch) 这个 Server 来支持 Agent 提取 web 内容。配置如下：

在 Cursor 中，可以这样配置：

![](https://pub-388464a2ac764e37ba36c5ea17d573ee.r2.dev/configure-mcp-in-cursor.png)

Type 设置为 `command`，表示本地命令行操作；`uvx mcp-server-fetch` 表示启动 `mcp-server-fetch` 服务。

Cline 参照的是 Claude 用法，直接把配置拷贝即可。

![](https://pub-388464a2ac764e37ba36c5ea17d573ee.r2.dev/configure-mcp-in-cline.png)

MCP 标准协议中存在一个握手过程，用来验证 Server 是否正常（图中绿色的小点代表握手成功）。

这样，你的 Chatbot 就拥有了爬取网页的能力。那如何唤起这个能力呢？

MCP Server 的使用[#](https://www.nazha.co/posts/what-is-mcp-and-how-to-use#mcp-server-%E7%9A%84%E4%BD%BF%E7%94%A8)
---------------------------------------------------------------------------------------------------------------

在使用中，通常有两种方式来触发 MCP Server 的调用（都是通过 prompt 调用，也叫作 prompt tool use）。

1.  通过 Server 的名称或描述，显式调用

比如：fetch this post [https://www.nazha.co/posts/how-cline-works，](https://www.nazha.co/posts/how-cline-works%EF%BC%8C) and summarize it. 其中，fetch 就是关键词。

![](https://pub-388464a2ac764e37ba36c5ea17d573ee.r2.dev/how-to-trigger-mcp-server.png)

2.  让 Agent 自己确定使用哪些工具

只描述需求，让 LLM 自己来确定是否要使用到某些工具。

常见的有用、好玩的 Server[#](https://www.nazha.co/posts/what-is-mcp-and-how-to-use#%E5%B8%B8%E8%A7%81%E7%9A%84%E6%9C%89%E7%94%A8%E5%A5%BD%E7%8E%A9%E7%9A%84-server)
----------------------------------------------------------------------------------------------------------------------------------------------------------

下面推荐介绍一些好用、好玩的 Server ：

1.  [Brave Search](https://github.com/modelcontextprotocol/servers/tree/main/src/brave-search) - 让 Agent 加上搜索的翅膀

用来进行网络搜索，即 `brave_web_search` 和 `brave_local_search`。借助 [Brave](https://brave.com/) 提供的能力。配置如下：

在使用前，需要申请 [Brave Api Key](https://brave.com/search/api/)。

2.  [fetch](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch)

用来读取网页内容。也可以配合 Brave Search 来使用。配置如下：

比如这个例子：search Sam Altman，and get the latest post of Sam Altman

![](https://pub-388464a2ac764e37ba36c5ea17d573ee.r2.dev/mcp-brave-search-fetch-exmaple.png)

Agent 先通过 Brave Search 搜索 Sam Altman，然后用 fetch 读取其博客的内容，分析结果得到其最新的博客文章是：[Reflections](https://blog.samaltman.com/reflections)，就这样实现了 LLM 的连网能力。

3.  [puppeteer](https://github.com/modelcontextprotocol/servers/tree/main/src/puppeteer) - 增强 Agent 的自动化

Puppeteer 可以支持浏览器自动化，当前 `@modelcontextprotocol/server-puppeteer` 列出的一些能力有：

*   `puppeteer_navigate` - 页面跳转
*   `puppeteer_screenshot` - 截屏
*   `puppeteer_click` - 元素点击
*   `puppeteer_hover` - 元素 Hover
*   `puppeteer_fill` - 文本填空
*   `puppeteer_select` - 选中选择框的元素
*   `puppeteer_evaluate` - 在 Console 中执行 JavaScript 代码

依托这个 MCP Server，可能可以做一些自动化的操作，比如 UI 自动测试、爬虫一类的事情。

配置如下：

4.  [mcp-obsidian](https://github.com/smithery-ai/mcp-obsidian)

这个 Server 可以用来读取和搜索 Obsidian Vault 中的内容。

5.  其他服务集成

各类服务也可以接入 MCP 提供能力，比如：

*   如果想要接入 [[RAG]]，[Needle](https://needle-ai.com/) 这个 SaSS 服务支持了 MCP。
*   支持图像能力，可以接入 [Replicate](https://replicate.com/) 这个服务，使用 [mcp-server-replicate](https://github.com/gerred/mcp-server-replicate?tab=readme-ov-file) 这个 Server。

MCP 的背后 - 跟 LLM 的集成[#](https://www.nazha.co/posts/what-is-mcp-and-how-to-use#mcp-%E7%9A%84%E8%83%8C%E5%90%8E---%E8%B7%9F-llm-%E7%9A%84%E9%9B%86%E6%88%90)
---------------------------------------------------------------------------------------------------------------------------------------------------------

LLM 为什么能够使用这些工具，其背后是一个典型的 [Agent 实现方式](https://www.nazha.co/posts/how-cline-works)。它是一个多步的操作：

1.  给 LLM 发送初始 prompt
2.  等待 LLM 给出 MCP Server 使用的响应
3.  客户端调用 MCP Server 的工具使用函数
4.  调用结果返回，并把结果再次提供给 LLM

![](https://pub-388464a2ac764e37ba36c5ea17d573ee.r2.dev/how-to-integration-to-llm.png)

### LLM 访问可使用的工具[#](https://www.nazha.co/posts/what-is-mcp-and-how-to-use#llm-%E8%AE%BF%E9%97%AE%E5%8F%AF%E4%BD%BF%E7%94%A8%E7%9A%84%E5%B7%A5%E5%85%B7)

目前让 LLM 识别有哪些可用的工具的思路有以下两种：

*   通过支持 Tool Use 的官方接口，比如 Anthropic 的 [Tool Use 接口](https://docs.anthropic.com/en/docs/build-with-claude/tool-use) 和 OpenAI 的 [Function Calling 接口](https://platform.openai.com/docs/guides/function-calling)。

以 Anthropic 的接口为例：

以 OpenAI 的接口为例：

*   **通过系统 Prompt，依赖 LLM 的指令遵循**

方案一不具备通用性，受限于 LLM 服务商提供的能力。为了让其他 LLM 也拥有 Tool Use 的能力，可以通过上下文学习的方式微调 LLM。通常来说，LLM 有很强的指令遵循的能力，在训练过程中也涌现出工具使用的能力。

比如下面的一个系统 system prompt 的例子（源 prompt 参考 cline [system prompt](https://gist.github.com/maoxiaoke/cd960ac88e11b08cbb4fa697439ebc68)）

LLM 的输出结果可能是：

在 Cline 的 system prompt 中，约定了可使用的 MCP Server 及其能力。通过解析 LLM 返回约定指令，调用 MCP Server。

### MCP 工具调用[#](https://www.nazha.co/posts/what-is-mcp-and-how-to-use#mcp-%E5%B7%A5%E5%85%B7%E8%B0%83%E7%94%A8)

在获取 LLM 调用指令之后，即可调用 MCP Server 来获取结果返回：

获取到 MCP Server 返回结果后，再次作为上下文提交给 LLM 处理。

这样 Agent 的一轮交互就结束了。

> 更多有关 LLM 集成的工程化考量，推荐大家阅读 [MCP Client Development Guide: Building Robust and Flexible LLM Integrations](https://github.com/cyanheads/model-context-protocol-resources/blob/main/guides/mcp-client-development-guide.md)

MCP Server 的能力[#](https://www.nazha.co/posts/what-is-mcp-and-how-to-use#mcp-server-%E7%9A%84%E8%83%BD%E5%8A%9B)
---------------------------------------------------------------------------------------------------------------

MCP Server 目前支持三类可被复用的能力：

1.  Tools

最常见的能力，向 LLM 提供可执行的功能，比如爬取网页内容、获取天气信息等，可以用来让 LLM 跟外界系统进行交互。

2.  Resources

允许 LLM 公开读取 Server 的数据或内容的能力。比如， [puppeteer](https://github.com/modelcontextprotocol/servers/tree/main/src/puppeteer) 这个 Server 就提供了对外读取浏览器控制台输出的能力。一般来说，也可以由 Tools 替代，但语义上对 LLM 更友好。

3.  Prompts

这个能力支持提示词的共享。

MCP Server 的开发也比较简单，[官网](https://modelcontextprotocol.io/introduction) 也提供了示例参考。

小结[#](https://www.nazha.co/posts/what-is-mcp-and-how-to-use#%E5%B0%8F%E7%BB%93)
-------------------------------------------------------------------------------

MCP 任重而道远。