# Claude Agent SDK 深度攻略：从零搭建自己的 AI 代理

2026年，AI Agent 是绝对的主角。但市面上大多数 Agent 教程，要么是用 LangChain 套个壳，要么是 AutoGPT 那种玩具级别的东西，根本没法用在生产环境。

今天介绍的 Claude Agent SDK，完全不一样。它就是 Claude Code 的内核。你用过 Claude Code 吗？那个能帮你读文件、写代码、执行命令的 AI 编程神器，它背后的引擎，现在开放给所有开发者了。

这意味着什么？你可以用同样的技术，打造属于自己的 AI Agent。不是玩具，是真正能干活的生产级工具。

本期视频，我会从零开始，手把手带你搭建三个实用的 AI Agent。全程代码开源，学完就能用。话不多说，我们直接开始。

## 什么是 Claude Agent SDK

先搞清楚一个概念：Claude Agent SDK 到底是什么？

简单来说，它是 Anthropic 把 Claude Code 的核心能力打包成了一个开发库。你用 Python 或者 TypeScript，就能调用 Claude Code 同款的能力：

1. 工具调用：让 AI 自己决定什么时候用什么工具
2. Agent 循环：AI 可以多轮思考、多次行动，直到完成任务
3. 上下文管理：自动处理长对话、大文件等复杂场景

以前你想实现这些功能，要么自己造轮子，要么用一堆第三方库拼凑。现在，一个 SDK 全搞定。

更重要的是，这不是实验性的东西。Claude Code 每天有无数人在用，Agent SDK 就是它的同款引擎，稳定性和可靠性都经过了验证。

## 环境准备

在开始写代码之前，我们先把环境配置好。

首先是 Python 版本，需要 3.10 或更高。打开终端，输入 python --version 检查一下。

然后是 Node.js，需要 18 或更高版本。这是因为 Agent SDK 底层依赖 Claude Code 的 NPM 包。

接下来安装 Agent SDK：

```bash
pip install claude-agent-sdk
```

最后配置 API Key。你需要一个 Anthropic 的 API Key，设置到环境变量里：

```bash
export ANTHROPIC_API_KEY="你的API密钥"
```

如果你用的是 AWS Bedrock 或者 Google Vertex，也可以用。设置对应的环境变量就行：

```bash
# AWS Bedrock
export CLAUDE_CODE_USE_BEDROCK=1

# Google Vertex
export CLAUDE_CODE_USE_VERTEX=1
```

环境准备好了，我们开始写第一个 Agent。

## 案例一：代码审查 Agent

我们从最简单的开始。做一个代码审查 Agent，给它一段代码，它帮你找 bug、找安全漏洞、提改进建议。

```python
from claude_agent_sdk import Agent

# 创建 Agent 实例
agent = Agent()

# 定义任务
task = """
请审查以下 Python 代码，找出潜在的 bug、安全漏洞和可以改进的地方：

def get_user(user_id):
    query = f"SELECT * FROM users WHERE id = {user_id}"
    result = db.execute(query)
    return result[0]
"""

# 执行任务
response = agent.run(task)
print(response)
```

就这么简单。运行一下，Agent 会告诉你这段代码有 SQL 注入风险，没有处理空结果的情况，还会给出修复建议。

这个例子虽然简单，但已经展示了 Agent SDK 的核心用法：创建 Agent，给它任务，让它自己思考并返回结果。

## 案例二：带搜索能力的助手

第一个例子里，Agent 只能分析我们给它的内容。但很多时候，我们需要 Agent 自己去搜索信息。

Agent SDK 内置了 Web 搜索工具。我们来做一个能搜索的助手：

```python
from claude_agent_sdk import Agent, WebSearchTool

# 创建带搜索能力的 Agent
agent = Agent(
    tools=[WebSearchTool()]
)

# 让它搜索最新信息
task = "Claude 4 Opus 相比 Claude 3.5 有哪些提升？请搜索最新资料总结一下"

response = agent.run(task)
print(response)
```

这次 Agent 会自己决定要不要搜索，搜索什么关键词，然后整合信息给你答案。

你也可以加一个兜底工具，比如当搜索失败时使用本地知识库：

```python
from claude_agent_sdk import Agent, WebSearchTool, Tool

# 自定义一个兜底工具
class LocalKnowledgeTool(Tool):
    name = "local_knowledge"
    description = "当网络搜索不可用时，查询本地知识库"

    def run(self, query: str) -> str:
        # 这里接入你的本地知识库
        return "从本地知识库返回的结果..."

agent = Agent(
    tools=[WebSearchTool(), LocalKnowledgeTool()]
)
```

这样，Agent 会优先搜索网络，搜不到就查本地知识库。

## 案例三：全能文件处理 Agent

最后一个案例，我们做一个真正有用的东西：一个能读写文件、执行命令的全能 Agent。

```python
from claude_agent_sdk import Agent, FileReadTool, FileWriteTool, BashTool

# 创建全能 Agent
agent = Agent(
    tools=[
        FileReadTool(),
        FileWriteTool(),
        BashTool()
    ]
)

# 给它一个复杂任务
task = """
1. 读取当前目录下所有的 Python 文件
2. 分析每个文件的代码质量
3. 生成一份 Markdown 格式的代码审查报告
4. 保存到 code_review_report.md
"""

response = agent.run(task)
print(response)
```

这个 Agent 会自己读取文件、分析代码、生成报告、保存文件。整个过程全自动。

这就是 Agent 的魅力。你不需要一步一步告诉它怎么做，只需要告诉它你要什么结果，它自己想办法完成。

## 进阶：安全钩子

当 Agent 能执行命令、写文件的时候，安全就变得很重要。Agent SDK 提供了 Hooks 机制，让你可以在关键操作前进行拦截和审核。

```python
from claude_agent_sdk import Agent, BashTool, Hook

# 定义安全钩子
class SafetyHook(Hook):
    def before_tool_call(self, tool_name: str, args: dict) -> bool:
        # 禁止执行危险命令
        if tool_name == "bash":
            dangerous_commands = ["rm -rf", "sudo", "chmod 777"]
            for cmd in dangerous_commands:
                if cmd in args.get("command", ""):
                    print(f"拦截危险命令: {args['command']}")
                    return False
        return True

agent = Agent(
    tools=[BashTool()],
    hooks=[SafetyHook()]
)
```

有了这个钩子，Agent 就不会执行 rm -rf 这种危险命令了。在生产环境里，这种安全措施是必须的。

## 进阶：多 Agent 协作

一个 Agent 能力有限，但多个 Agent 协作，就能处理更复杂的任务。

比如做一个研究系统：一个 Agent 负责搜索资料，一个负责分析总结，一个负责写报告。

```python
from claude_agent_sdk import Agent, WebSearchTool, FileWriteTool

# 研究 Agent：负责搜索
researcher = Agent(
    name="researcher",
    tools=[WebSearchTool()],
    system_prompt="你是研究员，负责搜索和收集信息"
)

# 分析 Agent：负责分析
analyst = Agent(
    name="analyst",
    system_prompt="你是分析师，负责分析数据并得出结论"
)

# 写作 Agent：负责输出
writer = Agent(
    name="writer",
    tools=[FileWriteTool()],
    system_prompt="你是技术写作专家，负责撰写清晰的报告"
)

# 协调工作流
research_result = researcher.run("搜索 Claude Agent SDK 的最新特性")
analysis = analyst.run(f"分析以下内容：{research_result}")
writer.run(f"根据以下分析撰写报告并保存：{analysis}")
```

这种多 Agent 架构，在处理复杂任务时特别有用。每个 Agent 专注自己擅长的事情，协作完成整体目标。

## 总结

Claude Agent SDK 把 Claude Code 的核心能力开放给了所有开发者。你可以用它构建各种 AI Agent：

- 代码审查助手
- 智能搜索助手
- 自动化文件处理
- 多 Agent 协作系统

它不是玩具，是生产级的解决方案。Claude Code 每天服务无数开发者，用的就是同一套引擎。

如果你之前尝试过其他 Agent 框架但觉得不够用，强烈建议试试 Claude Agent SDK。这可能是目前最实用的 Agent 开发工具。

代码我会放在 GitHub 上，链接在评论区置顶。有问题欢迎留言讨论。
