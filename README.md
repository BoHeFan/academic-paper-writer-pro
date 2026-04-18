# Academic Paper Writer Pro

一个专业的 AI Agent Skill，用于辅助学术论文的研究、撰写、排版，**并新增降AI率功能**。

本 Skill 强制执行结构化的工作流程，利用精准的 \.docx\ 和 \.pdf\ 处理能力，
确保您的文稿严格符合各类学术格式要求（如 IEEE、ACM、Springer、NeurIPS、MLA、APA 及各类高校模板）。

**新特性**：集成 Anti-AIGC 管道，支持中英文论文降AI率处理，有效降低AI检测系统的识别率。

## 1. 环境准备 (Prerequisites)

在使用本 Skill 之前，您需要一个支持文件操作和命令行工具的 Agentic 环境。我们支持以下两种主流环境：

### 选项 A: OpenCode (推荐)

一个专为开发者工作流优化的开源 Agentic 框架。

- **安装指南**: [OpenCode 官方文档](https://github.com/code-yeongyu/oh-my-opencode)
- **快速安装**:
  - **桌面版**: [https://opencode.ai/download](https://opencode.ai/download)
  - **命令行版**:
    \\\
    npm install -g opencode
    \\\

### 选项 B: Claude Code

Anthropic 官方推出的 Agentic CLI 工具。

- **安装指南**: [Claude Code 官方文档](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code)
- **注意**: 请确保您的环境中已安装 \git\ 和 \
pm\。

---

## 2. 安装说明 (Installation)

考虑到不同用户的使用环境，我们提供了 **一键自动化安装** 和 **手动配置** 两种方式。

> **?? 官方 Skill 主页**: [https://skills.sh/tfboy1/academic-paper-writer-pro/academic-paper-writer-pro](https://skills.sh/tfboy1/academic-paper-writer-pro/academic-paper-writer-pro)

### 选项一：一键自动化安装 (推荐)

如果您使用的是兼容的 Agentic 框架（如 Claude Code 或 OpenCode），只需在您的工作目录下运行以下一条命令，系统将自动拉取代码仓库并配置依赖，免去手动操作：

\\\
npx skills add https://github.com/tfboy1/academic-paper-writer-pro --skill academic-paper-writer-pro
\\\

### 选项二：手动克隆与配置

如果受限于网络或框架环境无法使用一键安装，请按以下步骤手动导入本 Skill：

#### 1. 克隆仓库

导航至您的 Agent 工作区或 Skills 目录，并克隆本仓库：

\\\
# 克隆到您的 skills 目录
git clone https://github.com/tfboy1/academic-paper-writer-pro.git ./agent/skills/academic-paper-writer-pro
\\\

#### 2. 加载 Skill

- **对于 OpenCode**: Agent 会自动检测配置路径下的 Skills。您可能需要重启会话，或显式要求 Agent "加载 academic-paper-writer-pro skill"。
- **对于 Claude Code**: 您可以通过在上下文窗口中提供此目录，或挂载该目录，指示 Claude 将其作为工具集使用。

---

## 3. 使用指南 (Usage Guide)

安装完成后，您可以直接使用自然语言控制整个写作、排版与降AI率流程。

### 场景一：纯小白一键自动写论文 (Auto Thesis Writing) 

如果您没有任何撰写基础，手中仅有项目代码或一份简单的立项任务书，您可以让 AI 从零开始为您**"先创作，后排版"**，全自动生成上万字的毕业设计或学术论文：

1. **投喂资料与启动**：只需启动您的 Agent，并将您的项目代码文件夹的绝对路径告诉大模型，或者将其拖入当前工作目录。
2. **下达小白"魔法指令"**：
   > "阅读这个项目代码里的数据库和源码逻辑，帮我写一篇不少于15000字的毕业设计论文。要求：写完后直接使用 \cademic-paper-writer-pro\ 技能进行全自动排版，并最终导出成学校要求的 Word 文档！"

3. **喝杯咖啡等待极客魔法**：Agent 将接管所有的脑力与体力劳动。
4. **一键提取与提交**：前往本目录下的 \outputs/\ 文件夹直接提取装订成册的 Word 成品！

---

### 场景二：已有草稿的专业学术排版 (Professional Typesetting)

如果您已经靠自己完成了论文的中期草稿（支持 Markdown、Word、PDF 等格式文件），只需借助本技能完成最终的底层无损学术排版。

#### 使用指令示例：

> "请使用 \cademic-paper-writer-pro\ 技能，把这篇 Word 论文草稿按 IEEE 格式重新排版。"
> "使用 \cademic-paper-writer-pro\ 技能，将这个 Markdown 转换为 Springer LNCS 格式的 Word 文档。"

---

### 场景三：论文降AI率处理 (Anti-AIGC Processing) 

**新功能**：如果您担心AI辅助生成的内容被检测系统识别，可以使用降AI率管道进行处理。

#### 中文论文降AI率

> "请使用 \cademic-paper-writer-pro\ 的降AI率功能处理这篇中文论文"
> "对这段中文技术文档进行降AI率重写，保持技术准确性"

**处理策略**：
- 冗余扩展：将简洁动词替换为更长的、带有动作过程描述的短语
- 词汇替换：系统性替换常用词汇（如"采用"→"运用"，"基于"→"鉴于"）
- 句式微调：使用"把"字句、条件句式转换等
- 括号内容整合：将括号内容自然融入句子

#### 英文LaTeX论文降AI率

> "将这篇英文LaTeX论文去AI化"
> "使用降AI率管道处理这段英文LaTeX代码"

**处理策略**：
- 词汇规范化：避免过度滥用的复杂词汇（leverage, delve into, tapestry）
- 结构自然化：移除机械连接词，减少列表格式
- 排版规范：禁用强调格式，保持LaTeX纯净

#### 完整工作流示例

> "帮我写一篇论文，写完后进行降AI率处理，然后按IEEE格式排版"

这将依次执行：
1. Pipeline D：内容生成
2. Pipeline E：降AI率处理
3. Pipeline C：MD直转排版

---

## 4. 管道说明 (Pipeline Overview)

| 管道 | 输入 | 功能 | 触发条件 |
|------|------|------|----------|
| Pipeline A | PDF | OCR识别 + 排版 | 提供.pdf文件 |
| Pipeline B | DOC/DOCX | 重排版 | 提供.doc/.docx文件 |
| Pipeline C | MD | MD直转排版 | 提供.md文件 |
| Pipeline D | 代码/数据 | 内容生成 | 无文件，仅有项目目录 |
| **Pipeline E** | **任意文本** | **降AI率处理** | **用户指定"降AI率"** |

---

## 5. 降AI率功能详细说明 (Anti-AIGC Details)

### 5.1 中文处理规则

| 规则类型 | 示例 |
|----------|------|
| 动词扩展 | "管理" → "开展...的管理工作" |
| 词汇替换 | "采用" → "运用"，"基于" → "鉴于" |
| 句式调整 | "若...则..." → "如果...就..." |
| 括号整合 | "ORM（对象关系映射）" → "对象关系映射即ORM" |

### 5.2 英文处理规则

| 规则类型 | 原文 | 替换为 |
|----------|------|--------|
| 词汇规范 | leverage | use, employ |
| 词汇规范 | delve into | investigate |
| 连接词移除 | First and foremost | 直接开始论述 |
| 连接词移除 | It is worth noting that | 直接陈述事实 |

### 5.3 技术术语保护

以下内容**绝对禁止修改**：
- 技术术语（Django, RESTful API, MySQL等）
- 代码片段（views.py, settings.py等）
- 库名（Boto3等）
- 配置项（DATABASES等）
- API路径

---

## 6. 资源库 (Resources)

本仓库提供了一些内置资源以帮助您快速上手：

- ?? **\	emplates/\**: 包含 IEEE, ACM, APA 等主流学术会议/期刊的官方模板下载链接。
- ?? **\nti_aigc/\**: 降AI率处理管道的核心实现。
- ? **\TROUBLESHOOTING.md\**: 常见问题排查指南。

---

## 致谢 (Credits & Acknowledgments)

本项目基于 [academic-paper-writer](https://github.com/TFboy1/academic-paper-writer) 进行增强开发，新增降AI率功能。

核心增强功能：
- **Pipeline E — Anti-AIGC 管道**：支持中英文论文降AI率处理
- **中文处理**：冗余扩展、词汇替换、句式微调
- **英文处理**：词汇规范化、结构自然化、排版规范

---

## License

MIT License
