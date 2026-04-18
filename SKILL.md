---
name: academic-paper-writer-pro
description: 强化版学术论文排版助手，集成降AIGC率功能。支持 PDF/DOC/DOCX/MD 多种输入格式，自动选择 OCR 管道、重排版管道、MD 直转管道或 Anti-AIGC 管道。包含环境清理确认、断点恢复、智能配图裁剪、逐单元增量生成 DOCX、双单元质量核查、中间状态保存和 BibTeX 参考文献管理。新增中英文降AI率处理能力。所有中间文件放 resources/，最终产物放 outputs/。
---

# 学术论文专家（主路由）

> [!IMPORTANT]
> 本文件是**主路由入口 + Pipeline B/C/E 定义 + 排版格式规范库**。Pipeline A（OCR 管道）的详细规范请参见 \ocr_kb/SKILL.md\。Pipeline E（降AI率管道）的详细规范请参见 \nti_aigc/SKILL.md\。
> 所有管道的 DOCX 生成均使用 \docx/SKILL.md\ 中定义的方法（docx-js 创建新文档 / unpack-edit-pack 编辑现有文档）。

## 0. 目录规范 (Directory Convention)

> [!IMPORTANT]
> 所有中间文件和生成产物必须严格遵守以下目录结构，**禁止在项目根目录下放置任何生成文件**（包括 \.md\、\.py\、\.docx\ 等）。

\\\
项目根目录/
├── resources/
│   ├── pages/              # Pipeline A: 切分出的单页 PNG 图片
│   ├── figures/            # 所有管道: 配图（裁剪的或提取的）
│   ├── md/                 # 所有管道: 提取/拆分结果
│   │   ├── page_N.md       # Pipeline A 按页
│   │   └── section_N.md    # Pipeline B/C 按章节
│   ├── scripts/            # 所有 Python 辅助脚本（可跨任务复用，清理时不删除）
│   ├── anti_aigc/          # Pipeline E: 降AI率处理
│   │   ├── input/          # 待处理原文
│   │   ├── output/         # 处理后文本
│   │   └── logs/           # 修改日志
│   ├── compiled_paper.md   # 最终汇总的完整 Markdown
│   ├── config.json         # 任务配置（源文件、格式、管道类型），整个任务期间不变
│   └── checkpoint.json     # 运行时进度（当前单元、计数器、失败项），随处理进度更新
├── outputs/                # 最终交付的 .docx / .bib 文件 + 核查点中间版本
├── ocr_kb/                 # Pipeline A: OCR 工作流 Skill
│   └── SKILL.md
├── docx/                   # Word 文档操作 Skill（DOCX 生成的技术实现基础）
│   └── SKILL.md
├── pdf/                    # PDF 操作 Skill
│   └── SKILL.md
├── content_generation/     # Pipeline D: 论文内容智能生成 Skill
│   └── SKILL.md
├── anti_aigc/              # Pipeline E: 降AI率处理 Skill
│   └── SKILL.md
├── SKILL.md                # 本文件（主入口路由 + Pipeline B/C/E + 排版规范库）
└── .*                      # 用户提供的原始文件 (.pdf / .doc / .docx / .md / 项目代码目录)
\\\

---

## 1. 新任务启动协议 (Step 0: Pre-flight)

> [!CAUTION]
> 以下四项检查必须在执行**任何**写作或排版动作之前**全部完成**，不可跳过。

### 1.0 技能版本检查与自动更新 (Version Check & Auto Update)

在每次启动处理时，必须首先对比用户本地的 skill 版本和现在的最新版本：
- 运行相应的命令（如 \git fetch origin && git status -uno\）检测本地仓库状态。
- **如果本地版本与最新版本不一致**：应当主动提示并帮助用户自动更新：
  > "检测到 \cademic-paper-writer-pro\ 技能存在最新版本，是否需要帮您自动更新？回复 **更新** 进行升级，或 **跳过** 继续当前任务。"
- 若用户选择更新，自动执行更新命令（如 \git pull\ 或 \
px skills add https://github.com/tfboy1/academic-paper-writer-pro --skill academic-paper-writer-pro\），待更新完成后再进入下一步。

### 1.1 环境清理确认

检测 \esources/\ 下的 \pages/\、\md/\、\igures/\ 以及 \esources/checkpoint.json\ 是否存在旧文件。

- **若存在旧文件**：列出文件清单，明确询问用户：
  > "检测到上次任务的中间文件（N 个页面/章节、M 个 Markdown、K 张配图）。是否删除以避免干扰？回复 **删除** 清空，或 **保留** 继续上次任务。"
- **若不存在旧文件**：跳过此步。
- **用户选择"删除"时**：删除 \esources/pages/*\、\esources/md/*\、\esources/figures/*\、\esources/checkpoint.json\、\esources/config.json\。\esources/scripts/\ **不删除**（脚本可复用）。
- **用户选择"保留"时**：读取 \esources/checkpoint.json\，进入断点恢复流程（见 §1.3）。

### 1.2 源文件确认与管道选择

扫描项目根目录下的所有支持格式文件（\.pdf\、\.doc\、\.docx\、\.md\）：

- **只有 1 个**：自动选定，根据扩展名决定管道。
- **有多个**：列出所有文件名、类型及大小，要求用户明确指定目标文件。
- **没有**：报错，要求用户放入源文件。

**管道自动选择规则**：

| 扩展名 | 管道 | 进度单位 | 核心 Skill |
|--------|------|----------|-----------|
| \.pdf\ | **Pipeline A — OCR 管道** | 页 (page) | \ocr_kb/SKILL.md\ |
| \.doc\ / \.docx\ | **Pipeline B — 重排版管道** | 章节 (section) | \docx/SKILL.md\ + 本文件 §3.2 |
| \.md\ | **Pipeline C — MD 直转管道** | 章节 (section) | \docx/SKILL.md\ + 本文件 §3.3 |
| 无 / 目录 | **Pipeline D — 内容生成管道** | 章节 (section) | \content_generation/SKILL.md\ |
| 用户指定降AI率 | **Pipeline E — Anti-AIGC 管道** | 段落 (paragraph) | \nti_aigc/SKILL.md\ |

### 1.3 断点恢复检测

检查 \esources/checkpoint.json\ 是否存在且合法：

- **存在且 \status == "suspended"\**：
  向用户展示上次进度（已完成单元数 / 总单元数，管道类型），询问：
  > "检测到上次任务（Pipeline X，已完成 N/M 个单元）。是否从断点继续？回复 **继续** 从第 N+1 个单元开始，或 **重新开始** 清空所有中间文件。"
- **不存在或 \status == "completed"\**：正常启动新任务。

---

## 2. 文件完整性检查与资源准备

### 2.1 文件完整性检查

要求用户把需要排版的文稿和格式要求放入本目录下：
- **草稿 (Draft)**：用户的内容文件（\.pdf\、\.doc\、\.docx\ 或 \.md\）。
- **格式要求 (Style Guide/Template)**：\.docx\ 模板或 \.pdf\ 指南（如 IEEE 模板）。
- **参考文献 (References)**：询问用户是否有 \.bib\ 文件。如果有，**必须优先使用**。

### 2.2 格式规范解析（所有管道必执行）

> [!IMPORTANT]
> 在拆分内容或提取文字之前，**必须先完成格式规范解析**，确定所有排版参数。

1. **如果用户提供了 \.docx\ 模板**（\config.template_file\ 非空）：
  - 使用 \docx/SKILL.md\ 的 unpack 方法打开模板
  - 提取样式定义（标题层级、正文字体/字号、行距、页边距、页眉页脚、分栏设置）
  - 将提取到的参数记录到 \config.json\ 的 \ormat_params\ 字段

2. **如果用户指定了格式名称**（如"IEEE"、"APA"）：
  - 从本文件 **§6 排版格式规范库** 中读取对应的预设参数
  - 将参数记录到 \config.json\ 的 \ormat_params\ 字段

3. **如果两者都有**：模板优先，预设参数作为兜底。

### 2.3 资源目录准备

确保 \esources/\（含子目录 \pages/\、\md/\、\igures/\、\scripts/\、\nti_aigc/\）和 \outputs/\ 存在，不存在则自动创建。

### 2.4 任务配置持久化

创建或读取 \esources/config.json\（详细格式见 \ocr_kb/SKILL.md\ §0.5），记录源文件路径、文件类型、管道类型、格式规范和排版参数，使后续续作无需重复指定。

---

## 3. 核心处理流程（按管道分发）

### 3.1 Pipeline A — OCR 管道（PDF 输入）

> [!IMPORTANT]
> Pipeline A 的**全部详细规范**定义在 \ocr_kb/SKILL.md\ 中。本节仅做高层描述。

1. **切图**：运行 \esources/scripts/split_pdf.py\ 将 PDF 分割为单页 PNG。
2. **逐页循环**：按 \ocr_kb/SKILL.md\ 的规范逐页提取 + 逐页追加 DOCX。
3. **核查与悬挂**：每 2 页核查，每 4 页悬挂。
4. **汇总定稿**：合并所有 \page_N.md\ → \compiled_paper.md\ → 最终 DOCX。

**DOCX 生成方式**：使用 \docx/SKILL.md\ 中的 docx-js 创建新文档，按 §6 格式规范库中的参数配置样式。

---

### 3.2 Pipeline B — 重排版管道（.doc / .docx 输入）

适用场景：用户已有 Word 文档，需要按新的格式规范（如 IEEE）无损排版。为了绝对保留原文原生复杂公式（OMML）、图表和批注，本管道**严禁**使用 Markdown 中转提取方案，必须采用底层 XML 外观平移（Unpack-Edit-Pack）。

#### 第一步：格式转换（仅 .doc）
如果输入是 \.doc\（旧格式），先转换为 \.docx\：
\\\ash
python docx/scripts/office/soffice.py --headless --convert-to docx
\\\

#### 第二步：解包 (Unpack) 与提取规范
1. 按 §2.2 解析目标格式参数。
2. 将输入 DOCX 解压到原生 XML 目录：
\\\ash
python docx/scripts/office/unpack.py  resources/unpacked/
\\\

#### 第三步：XML 级别样式重构 (Edit XML)
此步骤不触碰主体内容逻辑。编写局部脚本直接从底层"换肤"：
1. **替换 Styles (\word/styles.xml\)**：针对格式规范（如 IEEE），将 \styles.xml\ 内的全部核心样式强制替换为预设样式名称及字体。
2. **强制排定 \<w:sectPr>\**：在 \word/document.xml\ 内，定位节区配置块，强制挂载双栏设定及要求的纸张尺寸。
3. **重新绑定 \<w:pStyle>\**：遍历所有 \<w:p>\，通过文字特征或旧样式的权重判定其所属层级，将其挂载为 IEEE 标准的 \Heading1\、\Normal\。

#### 第四步：原貌打包定稿 (Pack)
对重构后的外观进行无缝打包验证：
\\\ash
python docx/scripts/office/pack.py resources/unpacked/ outputs/_final_.docx --original
\\\

---

### 3.3 Pipeline C — MD 直转管道（.md 输入）

> [!CAUTION]
> **绝对禁止使用 Pandoc 进行 Markdown 到 Word 的一键直转！**
> Pandoc 会丢失所有针对中国学位论文或顶会要求的高度定制化格式（如三线表渲染、首行缩进强绑定等）。必须且只能严格调用 **\docx/SKILL.md\ 中定义的 \docx-js\ (Node引擎)** 或底层的 \unpack -> edit XML -> pack\ 脚本。

适用场景：用户已有 Markdown 格式的论文，需要转为带格式的 Word 文档。

详细操作规范见原项目 \SKILL.md\ §3.3。

---

### 3.4 Pipeline D — 内容生成管道（项目记录/代码输入）

适用场景：用户没有完整的论文草稿，只有代码、实验数据或笔记，希望基于现有项目素材智能生成符合学术格式的论文初稿。

详情操作规范见 \content_generation/SKILL.md\。

---

### 3.5 Pipeline E — Anti-AIGC 管道（降AI率处理）

> [!IMPORTANT]
> Pipeline E 的**全部详细规范**定义在 \nti_aigc/SKILL.md\ 中。本节仅做高层描述。

适用场景：用户已有AI辅助生成的论文初稿，需要降低AI检测率，使文本更接近人类写作风格。

#### 3.5.1 管道入口条件

- 用户明确指定"降AI率"、"去AI化"、"降低AI检测"等需求
- 或用户提供的文档被检测为高AI生成概率

#### 3.5.2 语言自动检测

| 语言 | 处理策略 | 输出格式 |
|------|----------|----------|
| 中文 | 冗余扩展 + 词汇替换 + 句式微调 | Markdown / DOCX |
| 英文 | 词汇规范化 + 结构自然化 + 排版规范 | LaTeX / Markdown |

#### 3.5.3 处理流程

1. **读取原文**：从用户指定文件或 \esources/anti_aigc/input/\ 读取待处理文本
2. **语言检测**：自动判断中文/英文
3. **分段处理**：按段落逐段应用相应规则
4. **生成修改后文本**：输出到 \esources/anti_aigc/output/\
5. **记录修改日志**：在 \esources/anti_aigc/logs/\ 中记录主要修改点
6. **质量检查**：确保技术术语未被错误修改，核心逻辑一致

#### 3.5.4 与其他管道衔接

- **前置**：Pipeline D 生成初稿后 → Pipeline E 降AI率 → Pipeline C 排版
- **独立**：用户已有文档 → Pipeline E 处理 → 输出优化版本

#### 3.5.5 调用示例

\\\
用户指令：
"请使用 anti_aigc 管道处理这篇论文，降低AI检测率"
"对这段中文进行降AI率重写"
"将这篇英文LaTeX论文去AI化"
\\\

---

### 3.6 所有管道的通用规则

以下规则不分管道，**强制执行**：

| 规则 | 说明 |
|------|------|
| **逐单元处理** | 不论是页还是章节，每完成一个单元就追加到 DOCX 并更新 checkpoint |
| **每 2 单元核查** | 对照源文件确认无信息丢失或格式错乱 |
| **每 4 单元悬挂** | 保存中间版本，通知用户发送"继续"刷新上下文 |
| **needs_review 清零门禁** | 最终汇总前，\
eeds_review\ 必须为空 |
| **中间版本保留** | 核查点版本不删除，以便回退 |
| **全局编号** | figure / table / equation 三个计数器全局递增，跨单元不重置 |

---

## 4. 参考文献管理

- **优先**：解析用户提供的 \.bib\ 文件。
- **补充**：如果用户未提供 \.bib\，**不要自行编造或搜索文献**。标记所有缺失的引用（列出引用标记名），通知用户手动补充。
- **输出**：在 \outputs/references.bib\ 中生成参考文献文件。

---

## 5. 通用排版细则 (Cross-Pipeline Formatting Details)

> [!IMPORTANT]
> 以下细则适用于所有管道，具体参数受 §6 格式规范库中的预设值或用户模板覆盖。DOCX 的技术实现统一参考 \docx/SKILL.md\。

详细排版参数见原项目 \SKILL.md\ §5。

---

## 6. 排版格式规范库 (Format Style Library)

> [!IMPORTANT]
> 格式预设以独立 \.md\ 文件存放在 \	emplates/\ 目录下，每个文件包含完整的排版参数表。当用户指定格式名称时，读取对应文件。如果用户提供了 \.docx\ 模板，模板中的设置**覆盖**预设。

### 6.1 内置格式预设

| 格式名称 | 文件 | 适用场景 |
|----------|------|----------|
| IEEE | \	emplates/ieee.md\ | IEEE 会议论文、期刊论文 |
| APA 第7版 | \	emplates/apa7.md\ | 心理学、社会科学论文 |
| 中国学位论文 | \	emplates/chinese_thesis.md\ | 国内高校本硕博学位论文 |
| ACM | \	emplates/acm.md\ | ACM 计算机学会会议、期刊论文 |
| Springer LNCS | \	emplates/springer_lncs.md\ | 计算机科学讲义等学术专著 |
| NeurIPS | \	emplates/nips.md\ | NeurIPS 及机器学习相关会议 |
| MLA | \	emplates/mla.md\ | 人文学科、语言文学领域论文 |

---

## 7. 完成后总结输出

排版完成后，**必须**输出以下结构化完成报告：

\\\
【完成报告】
源文件：
输入类型：
处理管道：Pipeline
格式规范：
总单元数：
提取公式数：
裁剪/嵌入配图数：
提取/转换表格数：
核查点版本数：
最终文件：outputs/<最终文件名>.docx
遗留问题：
\\\

---

## 8. 学术诚信门禁系统 (Academic Integrity Gate)

详细规范见原项目 \SKILL.md\ §9。

---

## 9. 多视角自审评议面板 (Multi-Perspective Self-Review Panel)

详细规范见原项目 \SKILL.md\ §10。

---

## 10. 反泄漏与上下文腐蚀防护 (Anti-Leakage & Context-Rot Prevention)

详细规范见原项目 \SKILL.md\ §11。

---

## 11. 写作质量自检清单 (Writing Quality Check) — 中文学术特化

详细规范见原项目 \SKILL.md\ §12。

---

## 12. 修订分数轨迹追踪 (Score Trajectory Tracking)

详细规范见原项目 \SKILL.md\ §13。
