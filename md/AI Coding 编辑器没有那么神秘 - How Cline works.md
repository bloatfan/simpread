> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.nazha.co](https://www.nazha.co/posts/how-cline-works)

> 2025/01/24

2025/01/24

This post was created without the involvement of AI.

没想到，AI 在编程领域掀起了半变天。从 [v0](https://v0.dev/) 、[bolt.new](https://bolt.new/) 再到各类结合 Agant 的编程工具 [Cursor](https://www.cursor.com/)、[Windsurf](https://codeium.com/windsurf)，AI Coding 已经具备 idea MVP 的巨大潜力。从传统的 AI 辅助编码，到如今的直接项目生成的背后，到底是怎么实现的？

本文会从开源产品 [Cline](https://github.com/cline/cline) 开始，从中窥探现阶段 AI Coding 产品的一些实现思路。同时了解更深层的原理，也能更好地使用 AI 编辑器。

> 各个 AI Coding 编辑器的最终实现可能不太一致。另外，本文也不会深入到 Tool Use 的实现细节。

Cline 整体的流程我画了一张草图：

![](https://pub-388464a2ac764e37ba36c5ea17d573ee.r2.dev/how-cline-works.png)

Cline 的核心是依赖系统提示词以及大语言模型的指令遵循能力。在编程任务启动的时，收集系统提示词、用户自定义提示、用户的输入、以及所在项目的环境信息（哪些文件、打开的 Tab 等等），提交给 LLM。LLM 按照指令输出解决方案和操作。Cline 通过解析返回的操作指令（比如 `<execute_command />`、`<read_file />`），调用编写好的 Tool Use 能力进行处理，并将处理结果交给 LLM 处理。Cline 会多次调用 LLM 来完成单次任务。

系统提示词[#](https://www.nazha.co/posts/how-cline-works#%E7%B3%BB%E7%BB%9F%E6%8F%90%E7%A4%BA%E8%AF%8D)
--------------------------------------------------------------------------------------------------

Cline 的 [系统提示词](https://github.com/cline/cline/blob/main/src/core/prompts/system.ts) 是类 [v0](https://github.com/sharkqwy/v0prompt) 提示词，使用 Markdown 和 XML 编写。其中详细定义了 LLM Tool Use 规则和用法示例：

[MCP](https://www.anthropic.com/news/model-context-protocol) 服务器也会注入系统提示词中。

用户指令也可以通过 `.clinerules` 注入到系统系统提示词中。

> 从这我们可以大胆推测，Cursor 和 WindSurf 注入 .cursorrules 也是类似的

由此可以看出，Cline 核心是依赖 LLM 的指令遵循能力，因此模型的 [temperature](https://platform.openai.com/docs/api-reference/chat/create) 被设置成了 `0`。

第一轮输入[#](https://www.nazha.co/posts/how-cline-works#%E7%AC%AC%E4%B8%80%E8%BD%AE%E8%BE%93%E5%85%A5)
--------------------------------------------------------------------------------------------------

用户存在多种输入，分别有：

1.  直接输入的文案，用 `<task />` 包含
2.  通过 `@` 输入的文件目录、文件和 url

在 Cline 中，`@` 没有什么太多的科技含量，对于文件目录，列出文件目录结构；文件，读取文件内容；url，则直接 puppeteer 读取内容。然后把这些内容和用户输入，输出给 LLM。

一个示例输入如下：

用户输入还包含一类信息，就是项目环境信息，有当前的工作目录的文件列表、vscode 打开的 tab 等等。

一个简单的任务给 LLM 的输入如下：

> 从这里可以看到，其他 AI Coding 编辑器（比如 Cursor）**有可能**对代码库进行[嵌入](https://docs.cursor.com/context/codebase-indexing)，但 cline 比较粗暴直接。

第一轮返回[#](https://www.nazha.co/posts/how-cline-works#%E7%AC%AC%E4%B8%80%E8%BD%AE%E8%BF%94%E5%9B%9E)
--------------------------------------------------------------------------------------------------

LLM 按照指令要求返回（temperature 被设置为 0），一般包含 `<thinking />` 和操作两个部分。比如：

在这个例子，Cline 通过解析 LLM 输出的指令，调用各类系统操作，包括但不限于：

1.  执行指令
2.  读写文件
3.  搜索内容
4.  MCP 操作

同时，Cline 会收集各类操作操作状态信息。

第二轮输入[#](https://www.nazha.co/posts/how-cline-works#%E7%AC%AC%E4%BA%8C%E8%BD%AE%E8%BE%93%E5%85%A5)
--------------------------------------------------------------------------------------------------

接下来，Cline 会把上一步用户的操作行为，操作输出状态和结果，包含之前的系统 prompt、用户输入，再次输出给 LLM，请求 LLM 给出下一步执行的指导。如此往复。

可以看到，处理单个任务需要往复循环调用多次 LLM，直至任务结束。另外一点是，Cline 基本上就是把所有内容都一股脑塞给 LLM，一次任务的 Token 的使用量非常高。

另外的一个问题就是很容易触发 LLM 的上下文窗口限制，Cline 的处理策略就是暴力截断。

> 这估计也是其他 AI Coding 编辑器的处理方式。之前在使用 windsurf 的时候，就很好奇为什么不受 LLM 上下文窗口的限制。但是，在后面的问答中往往又重复了之前的回答。