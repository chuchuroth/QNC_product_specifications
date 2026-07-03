










---
---
---
---
---
---

---

### AI与工具集成：核心概念与MCP的重要性

近期，AI领域涌现出许多新概念和技术，尤其是在AI模型与外部工具的集成方面，其中**Prompt、Agent、Function Calling和MCP**是理解这一演进的关键。

**1. Prompt（提示词）**
*   **User Prompt（用户提示词）**：用户发送给AI模型的问题或想说的话。
*   **System Prompt（系统提示词）**：用于描述AI的角色、性格、背景信息、语气等。它包含非用户直接输入的内容，会在每次用户发送User Prompt时自动发送给AI模型，使对话更自然。在网页聊天机器人中，系统提示词通常预设且用户不可随意更改，但可能通过设置（如ChatGPT的“Customize ChatGPT”功能）添加用户偏好。

**2. AI Agent（智能体）**
*   **定义**：AutoGPT首次尝试让AI自行完成任务，并将这种负责在AI模型、工具和最终用户之间传导的程序称为AI Agent。
*   **Agent Tool（智能体工具）**：提供给AI调用的函数或服务，例如列出文件（list files）、读取文件（read files）等。
*   **通信方式**：
    *   **System Prompt集成**：Agent将工具的功能描述和使用方法注册到自身，生成System Prompt告知AI模型可用工具及其调用格式。
    *   **Function Calling（函数调用）**：大型模型厂商（如GPT、Claude、Gemini）推出的新功能。它通过**标准化工具描述（使用JSON对象定义工具名、功能说明、所需参数）**和**AI使用工具时的返回格式**来解决System Prompt方式可能出现的AI返回格式不正确的问题。Function Calling使得AI模型能够更有针对性地理解调用场景，且AI服务器端可以检测并重试错误回复，降低用户端开发难度和Token开销。Cherry Studio支持Function Calling，而Client则使用System Prompt。
    *   **并存状态**：System Prompt和Function Calling两种方式目前并存，因为Function Calling缺乏统一标准且许多开源模型尚不支持。

**3. MCP (Model Context Protocol - 模型上下文协议)**
*   **核心理念**：MCP的全称是Model Context Protocol，由Anthropic公司于2024年11月25日发布。简单来说，**MCP是一个能够让大模型更好地使用各类工具的协议**。
*   **作用**：在MCP出现之前，AI主要通过API接口调用工具，但API存在局限性（不同工具API规范各异，且API关注数据传输而非数据含义）。MCP通过**统一规范工具接口**，解决了这一问题。它将用户的查询、工具的描述和参数以结构化的方式传递给大模型，由大模型决定如何处理。
*   **职责**：MCP本身与AI模型无关，它主要负责**帮助Agent管理工具、资源和提示词模板**。
*   **组件**：
    *   **MCP Server（MCP服务器）**：一个符合MCP协议的程序，通常在本地通过Node或Python启动，不一定是远程服务器。它内部包含了提供给AI调用的“工具”（Tool），这些工具实际上是编程语言中的函数。
    *   **MCP Client（MCP客户端）**：调用MCP Server的Agent。
*   **通信方式**：MCP Server和MCP Client之间可以通过**标准输入/输出（stdio）**进行通信（目前大多数MCP Server采用此模式），也可以通过HTTP部署在网络上进行通信。
*   **价值**：AI的发展路径包括掌握更多信息和控制更多工具。**MCP被认为是实现“控制更多工具”这条路线的关键**，有了MCP，AI从“功能机”进化到“智能机”时代，可以自由接入海量工具。

**4. 整体工作流程**
用户向AI Agent或MCP Client提出请求（User Prompt）。Agent通过MCP协议从MCP Server获取所有工具的信息。Agent将这些工具信息转化为System Prompt或Function Calling格式，并与用户请求一同发送给AI模型。AI模型根据需求决定调用某个工具（例如网页浏览工具），并以特定格式返回调用请求。Agent接收请求后，通过MCP协议调用MCP Server中的相应工具。工具执行操作并将结果返回给Agent，Agent再转发给AI模型。AI模型根据工具结果和自身知识生成最终答案。最后，Agent将结果展示给用户。

**5. 常见MCP客户端/主机及特点**
*   **Client**：VS Code的一个插件，编程能力强。它通过**系统提示词**对接AI模型。支持在MCP市场（如MCP.smpmarket.com, Smithy.ai）查找和自动安装MCP Server。但也支持通过JSON文件手动配置MCP Server，这提供了更高的确定性。
*   **Cherry Studio**：一个完全开源免费的AI客户端。它使用**Function Calling功能**对接AI。需要手动安装`uv`（Python运行环境）和`bn`（类似Node.js的JS运行环境）以支持MCP Server运行。
*   **Cursor**：另一款AI驱动的编程工具。它在处理复杂代码时可能遇到上下文长度限制（约10K Token，相当于400-600行代码），导致难以处理复杂项目中的bug或跨文件依赖。
*   **Cloud Code**：改变编程方式的AI工具。它能分析代码库，识别代码风格和项目结构，并能处理复杂重构任务，保持上下文，提升代码质量。能够实现“言出法随，想法和实现同步”的快感。
*   **Augment**：被认为是比Cursor更强大、更适合复杂项目的AI编程工具。
    *   **超长上下文**：支持高达**200K Token的上下文长度**，相当于8000到1万行标准Python代码，使其能更好地理解复杂项目代码库、识别文件依赖关系和调用模式。
    *   **功能**：能从GitHub、Gitlab等集成工具中提取详细上下文，提升代码感知和自动补全，并实现跨多个文件的功能请求。
    *   **应用示例**：可用于分析开源项目架构、特点、代码质量、耦合度、接口设计和可维护性。能分析函数调用关系并绘制流程图。支持连接多种工具，包括MCPs（例如Context7，用于获取开源项目最新文档）。能够全自动化地编写和优化代码，甚至开发复杂的3D空战游戏。

**6. 常用MCP工具/Server类型示例**
*   **网页抓取**：Fetch（用于抓取网页内容）。
*   **天气查询**：OpenWeatherMap（提供天气预报、气象预警）。
*   **地图/定位服务**：Baidu Map MCP Server（适合中国用户，规划行程、查询交通路线）。
*   **浏览器自动化**：PLRIMCP Server（操作浏览器页面，如点击、填写）。
*   **社交媒体通信**：Server Slack（通过Slack发送机器人通知）。
*   **联网搜索**：Brief Search，DuckDuckGo MCP Server（让大模型获得联网搜索能力）。
*   **知识库与记忆体**：知识图谱MCP（Knowledge Graph MCP，存储实体和关系）。
*   **文件系统操作**：File System MCP（操作本地文件和文件夹的读写）。
*   **命令行执行**：Desktop Commander（允许大模型在本地电脑上执行命令行）。
*   **思维/推理**：Sequential Thinking（帮助大模型拆解问题，将其转换为推理模型）。
*   **云平台服务**：Cloudflare MCP Server（操作Cloudflare的KV存储、R2数据桶、D1数据库等）。
*   **数据库连接**：PostgreSQL MCP（只读连接PostgreSQL数据库）。
*   **版本控制**：GitHub MCP（本地Git仓库操作，如列出仓库、创建分支、提交更改）。
*   **浏览器内容读取**：Brow2（让AI直接读取浏览器内显示的内容）。
*   **视频总结**：MCP YouTube（获取YouTube视频字幕并进行总结）。
*   **文档获取**：Context7（获取开源项目或库的最新文档）。

**7. 工具启动方式**
*   **UVX**：`uvx`是`uv run`命令的缩写，用于运行Python程序，可以自动配置依赖和执行环境。
*   **NPX**：与UVX类似，用于自动下载和安装Node程序，是Node.js的一部分。
*   **直接使用`node`或`bun`** 等。

**8. 注意事项和挑战**
*   **名称误导**：MCP Server的“Server”一词可能具有误导性，因为它通常是本地程序而非远程服务器。
*   **自动安装问题**：虽然某些MCP客户端支持AI自动安装MCP Server，但用户可能对安装路径、版本有要求，且自动安装可能失败，因此手动配置更通用和确定性高。
*   **首次启动超时**：UVX/NPX首次执行程序时需要下载依赖，可能因默认超时时间（如1分钟）导致加载失败，建议先在终端手动执行一次。
*   **AI协作**：在与AI（如Cloud Code）协作处理复杂任务时，建议分步执行：先让AI分析需求，再确认执行，而不是直接让AI“先斩后奏”，这样更稳妥高效。
*   **持续化**：对于需要持久化状态的任务，不应仅依赖内存存储，而应考虑写入文件或使用队列等方式。

MCP的出现，极大地扩展了AI的能力边界，让AI能够从“信息掌握”走向“工具控制”，实现更复杂、更贴近实际的应用。


---

# 人工智能核心概念与技术体系

在人工智能领域，一些核心概念和技术正在共同构建一个更加完整的AI体系，其中包括提示词（Prompt）、AI Agent、工具（Tool）、函数调用（Function Calling）和模型上下文协议（MCP）。

---

## 提示词 (Prompt)

- **用户提示词 (User Prompt)**：  
  用户发送给AI模型的消息，通常是提出的问题或想说的话。

- **系统提示词 (System Prompt)**：  
  用于描述AI的角色、性格、背景信息、语气等。它包含非用户直接说出的内容，每次用户发送用户提示词时，系统会自动将系统提示词一同发送给AI模型，使对话更自然。  
  在网页聊天机器人中，系统提示词常是预设的，但用户可通过设置（如 ChatGPT 的“Customize ChatGPT”功能）来影响其部分内容。

---

## AI Agent（智能体）

- 负责在 AI 模型、工具和最终用户之间进行转化和协调的程序。
- 例如，**AutoGPT** 是一个本地运行的小程序，通过注册文件管理函数（如 list files、read files）作为工具，并生成系统提示词告知AI模型这些工具及其用法，从而让AI能够调用这些工具来完成任务。
- 许多 AI Agent 在 AI 模型返回格式不正确时，会自动进行重试以处理“不听话”的情况。

---

## Agent Tool（智能体工具）

- 提供给 AI 模型调用的函数或服务。
- 这些工具可以帮助 AI 执行实际操作，而不仅仅是回答问题。
- 例如，管理文件的函数、网页浏览工具等。

---

## 函数调用（Function Calling）

- 这是大型模型厂商（如 ChatGPT、Claude、Gemini）推出的新功能，旨在统一工具描述和AI返回的格式规范。
- 它通过标准化工具描述（例如，每个工具都用 JSON 对象定义名称、说明和参数）并固定 AI 使用工具时的返回格式，解决了AI模型可能返回不正确格式的问题。
- 相比于在系统提示词中随意编写工具描述，函数调用使得模型训练更具针对性。即使 AI 生成错误回复，服务前端也能检测并重试，用户无感知。

### 优点：
- 降低了用户端的开发难度
- 节省了用户端重试带来的 token 开销

### 现状：
- 当前与系统提示词方式并存
- 函数调用尚未有统一标准，且许多开源模型不支持

---

## 模型上下文协议（Model Context Protocol, MCP）

- **全称**：Model Context Protocol，由 Anthropic 公司于 2024 年 11 月 25 日发布。
- **核心作用**：让大型模型能够更好地使用各类外部工具。大模型本身只会问答，不会使用外部工具，MCP 的出现赋予了它们这种能力。
- MCP 是一个专门用来规范 AI Agent 和工具服务之间如何交互的通信协议。

### MCP 架构组成：

- **MCP Server（MCP服务器）**：  
  运行 MCP 协议的工具服务。可本地运行（Node/Python）或通过 HTTP 部署。  
  提供给 AI 调用的“工具”（函数），也可提供数据（资源）或提示词模板。

- **MCP Client（MCP客户端）**：  
  调用 MCP Server 的 AI Agent。常见的 MCP Host 包括：Cloud Desktop、Cursor、Cline、Cherry Studio、Augment 等。

- **MCP Tool（MCP工具）**：  
  MCP 中的“工具”通常指编程语言中的函数，例如查询天气、发送信息等。

- **通信方式**：  
  MCP Server 和 MCP Client 之间可通过标准输入输出（`stdio`）或 HTTP 通信。

### 优势：

- 统一所有工具接口，解决传统 API 规范不一的问题（如同 AI 的 USB-C 接口）。
- 不仅关注数据传输，还能将用户的查询、工具描述和参数结构化传递给大模型，让模型理解并自主决策处理。

---

## AI 编程工具与 MCP 的结合应用

### Cline

- VS Code 插件，支持连接多种 API 和模型，支持 MCP。
- 可通过 MCP 服务操作 Obsidian 笔记、提取网页内容（如 Tavily MCP）、并保存为笔记。

### Cherry Studio

- 开源免费 AI 客户端，强调对 Function Calling 的支持。
- 支持配置启动各种 MCP Server：

  - Fetch MCP Server：从网页中获取内容  
  - 百度地图 MCP Server：规划行程、查询交通  
  - Playwright MCP Server：自动化浏览器操作  
  - Slack MCP Server：发送机器人通知  
  - DuckDuckGo MCP Server：联网搜索  
  - 知识图谱 MCP Server：存储/检索结构化知识  
  - 文件系统 MCP Server：读写本地文件  
  - Desktop Commander：本地执行命令  
  - Sequential Thinking：将问题拆解为推理模型  
  - Cloudflare MCP Server：操作 KV 存储、R2 数据桶、D1 数据库  
  - PostgreSQL MCP Server：访问 PostgreSQL 数据库（只读）  
  - GitHub/Git MCP Server：版本控制、管理仓库  
  - YouTube MCP Server：获取视频摘要或字幕  

### Cloud Code

- 强大的 AI 编程工具，可自动化处理复杂编程需求。
- 使用 `init` 指令扫描理解项目结构，分析需求后按步骤执行任务。

### Augment

- 比 Cursor 更强大的 AI 编程工具，适用于大型复杂项目。

#### 特点：

- **超长上下文**：支持高达 200K token（约 8,000-10,000 行 Python 代码）
- **高级分析生成**：分析项目架构、代码质量、模块化、耦合度、接口设计与可维护性，生成调用图、依赖图
- **结合 MCP 工具**：  
  集成 Context 7 MCP，可自动获取文档，结合 AI 自动搜索、编写、优化、测试代码，无需人工干预

---

## 总结

MCP 是 AI 从“功能机”迈向“智能机”时代的关键。这些概念和工具共同协作，使 AI 能更好地理解世界、获取信息，并操作各种外部工具，完成更复杂与多样化的任务。



---
---
---
---
---
