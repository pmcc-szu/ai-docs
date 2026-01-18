# Claude Code Skills 使用教程

## 什么是 Skills？

Skills 是 Claude Code 的自定义技能系统，可以把常用的复杂指令封装成简单的斜杠命令。比如输入 `/帮我写作`，Claude 就会自动按照预设的规则帮你撰写文案。

## 目录结构

所有 Skills 存放在项目根目录的 `.claude/skills/` 文件夹下：

```
.claude/
└── skills/
    ├── 帮我写作/
    │   ├── SKILL.md          # 核心配置文件（必须）
    │   └── references/       # 参考资料（可选）
    │       └── 160 Obsidian.md
    ├── 字幕转markdown/
    │   ├── SKILL.md
    │   └── scripts/          # 脚本文件（可选）
    │       └── screenshot.py
    └── 来点选题/
        └── SKILL.md
```

## SKILL.md 文件格式

每个 Skill 必须包含一个 `SKILL.md` 文件，格式如下：

```markdown
---
name: 技能名称
description: 简短描述这个技能做什么
---

这里写具体的指令内容...
```

### 头部信息（Frontmatter）

| 字段 | 说明 |
|------|------|
| `name` | 技能名称，会显示在命令列表中 |
| `description` | 简短描述，帮助用户了解这个技能的用途 |

### 正文内容

正文就是你希望 Claude 执行的完整指令，可以包含：

- 任务描述
- 具体要求和规则
- 输出格式要求
- 后续处理步骤

## 如何使用 Skills

在 Claude Code 对话中，直接输入斜杠命令即可：

```
/帮我写作
/字幕转markdown
/来点选题
```

## 实战示例

### 示例1：简单的选题助手

```markdown
---
name: 来点选题
description: 当用户视频创作选题枯竭的时候，为用户提供选题思路
---
我是视频作者，你是我的"灵感助理"，以下是我往期的数据较好的往期视频，请提供更多的类似10-15个灵感。用中文输出

把Gemini3生成的应用，免费部署成自己的网站
Obsidian邪修用法，免费云同步，AI，手机端，进阶技巧
...（更多往期选题）
```

### 示例2：带参考资料的写作助手

```markdown
---
name: 帮我写作
description: 结合用户提供的材料进行文案写作
---

结合我发你的材料，撰写一篇深度文案，输出到项目根目录
如果是软件/AI/计算机相关的材料，请先阅读参考 references/ 里面所有的范文，学习我的行文风格。

其他要求如下：
- 整体行文要流畅、易于口播
- 节奏感强：多用短句，逻辑清晰
...
```

### 示例3：带脚本的自动化流程

```markdown
---
name: srt字幕转markdown笔记
description: 把srt字幕文件转换成markdown笔记
---
你是专业的字幕文本处理助手，任务是将 SRT 字幕完整转换为 Markdown 笔记...

**后续处理**：
markdown生成完毕以后，调用
python scripts/screenshot.py
对视频进行截图
```

## 进阶技巧

### 1. 添加参考资料

在 Skill 文件夹中创建 `references/` 目录，放入范文或参考材料：

```
帮我写作/
├── SKILL.md
└── references/
    ├── 范文1.md
    └── 范文2.md
```

在 SKILL.md 中引用：
```
请先阅读参考 references/ 里面所有的范文，学习我的行文风格。
```

### 2. 添加自动化脚本

可以在 Skill 中调用 Python 脚本或其他工具：

```
字幕转markdown/
├── SKILL.md
└── scripts/
    └── screenshot.py
```

### 3. 调用 MCP 工具

如果配置了 MCP 服务，可以在 Skill 中直接调用：

```markdown
用 create_repository MCP 工具创建一个仓库，然后用 create_or_update_file MCP 工具上传文件。
```

## 创建新 Skill 的步骤

1. 在 `.claude/skills/` 下创建新文件夹
2. 创建 `SKILL.md` 文件
3. 填写 frontmatter（name 和 description）
4. 编写详细的指令内容
5. 根据需要添加 references 或 scripts

## 常见问题

**Q: Skill 不生效怎么办？**
A: 检查 SKILL.md 的 frontmatter 格式是否正确，`---` 必须在文件开头。

**Q: 可以在 Skill 中引用其他文件吗？**
A: 可以，把文件放在 Skill 文件夹下，然后在指令中说明让 Claude 读取。

**Q: 如何查看所有可用的 Skills？**
A: 在 Claude Code 中输入 `/` 可以看到所有可用的斜杠命令。
