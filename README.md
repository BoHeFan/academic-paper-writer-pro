# Academic Paper Writer Pro

[![License](https://img.shields.io/github/license/BoHeFan/academic-paper-writer-pro?style=for-the-badge)](https://github.com/BoHeFan/academic-paper-writer-pro/blob/main/LICENSE)
[![GitHub Stars](https://img.shields.io/github/stars/BoHeFan/academic-paper-writer-pro?style=for-the-badge&logo=github&color=yellow)](https://github.com/BoHeFan/academic-paper-writer-pro/stargazers)

**强化版学术论文排版助手，集成降AIGC率功能。**

基于 [academic-paper-writer](https://github.com/TFboy1/academic-paper-writer) 增强开发，新增 **Anti-AIGC 管道**，支持中英文论文降AI率处理。

---

## 与原版对比 (What's New)

| 功能 | 原版 | Pro版 |
|------|------|-------|
| PDF OCR识别 | ✅ | ✅ |
| Word重排版 | ✅ | ✅ |
| Markdown直转 | ✅ | ✅ |
| 内容智能生成 | ✅ | ✅ |
| **中文降AI率** | ❌ | ✅ **新增** |
| **英文LaTeX降AI率** | ❌ | ✅ **新增** |
| **技术术语保护** | ❌ | ✅ **新增** |

### 新增功能详解

#### 1. Pipeline E — Anti-AIGC 管道

全新的降AI率处理管道，支持：

- **中文文本重写**：
  - 冗余扩展：管理 → 开展...的管理工作
  - 词汇替换：采用 → 运用，基于 → 鉴于
  - 句式微调：使用"把"字句、条件句式转换
  - 括号整合：ORM（对象关系映射） → 对象关系映射即ORM

- **英文LaTeX重写**：
  - 词汇规范化：leverage → use，delve into → investigate
  - 结构自然化：移除机械连接词，减少列表格式
  - 排版规范：禁用强调格式，保持LaTeX纯净

- **技术术语保护**：
  - 自动识别并保护技术术语、代码片段、库名、配置项等
  - 确保核心逻辑与原文一致

#### 2. 完整工作流支持

`
Pipeline D (内容生成) → Pipeline E (降AI率) → Pipeline C (排版)
`

一键完成：AI生成初稿 → 降低AI检测率 → 学术格式排版

---

## 安装说明 (Installation)

### 方式一：一键安装（推荐）

`ash
npx skills add https://github.com/BoHeFan/academic-paper-writer-pro --skill academic-paper-writer-pro
`

### 方式二：手动克隆

`ash
git clone https://github.com/BoHeFan/academic-paper-writer-pro.git
`

---

## 使用指南 (Usage Guide)

### 场景一：论文降AI率处理

**中文论文**：
`
请使用 academic-paper-writer-pro 的降AI率功能处理这篇中文论文
`

**英文LaTeX论文**：
`
将这篇英文LaTeX论文去AI化
`

### 场景二：完整工作流（生成+降AI+排版）

`
帮我写一篇论文，写完后进行降AI率处理，然后按IEEE格式排版
`

### 场景三：专业学术排版

`
请使用 academic-paper-writer-pro 技能，把这篇 Word 论文草稿按 IEEE 格式重新排版
`

---

## 管道说明 (Pipeline Overview)

| 管道 | 输入 | 功能 | 触发条件 |
|------|------|------|----------|
| Pipeline A | PDF | OCR识别 + 排版 | 提供.pdf文件 |
| Pipeline B | DOC/DOCX | 重排版 | 提供.doc/.docx文件 |
| Pipeline C | MD | MD直转排版 | 提供.md文件 |
| Pipeline D | 代码/数据 | 内容生成 | 无文件，仅有项目目录 |
| **Pipeline E** | **任意文本** | **降AI率处理** | **用户指定"降AI率"** |

---

## 降AI率功能详解 (Anti-AIGC Details)

### 中文处理规则

| 规则类型 | 原文 | 修改后 |
|----------|------|--------|
| 动词扩展 | 管理 | 开展...的管理工作 |
| 词汇替换 | 采用 | 运用 |
| 词汇替换 | 基于 | 鉴于 |
| 句式调整 | 若...则... | 如果...就... |
| 括号整合 | ORM（对象关系映射） | 对象关系映射即ORM |

### 英文处理规则

| 规则类型 | 原文 | 替换为 |
|----------|------|--------|
| 词汇规范 | leverage | use, employ |
| 词汇规范 | delve into | investigate |
| 词汇规范 | tapestry | context |
| 连接词移除 | First and foremost | 直接开始论述 |
| 连接词移除 | It is worth noting that | 直接陈述事实 |

### 技术术语保护

以下内容**绝对禁止修改**：
- 技术术语（Django, RESTful API, MySQL, JWT等）
- 代码片段（views.py, settings.py等）
- 库名（Boto3, djangorestframework-simplejwt等）
- 配置项（DATABASES, CEPH_STORAGE等）
- API路径（/accounts/api/token/refresh/等）

---

## 项目结构 (Project Structure)

`
academic-paper-writer-pro/
├── SKILL.md                    # 主入口（含Pipeline E定义）
├── README.md                   # 本文件
├── LICENSE                     # MIT许可证
├── TROUBLESHOOTING.md          # 故障排查指南
├── anti_aigc/
│   └── SKILL.md               # 降AI率管道核心实现
├── content_generation/
│   └── SKILL.md               # 内容生成管道
├── docx/
│   └── SKILL.md               # Word文档操作
├── ocr_kb/
│   └── SKILL.md               # OCR管道
├── pdf/
│   └── SKILL.md               # PDF操作
├── templates/                  # 格式模板
└── resources/                  # 资源目录
    ├── pages/                  # PDF切图
    ├── figures/                # 配图
    ├── md/                     # Markdown文件
    ├── scripts/                # 辅助脚本
    └── anti_aigc/              # 降AI率处理
        ├── input/              # 待处理原文
        ├── output/             # 处理后文本
        └── logs/               # 修改日志
`

---

## 致谢 (Credits)

本项目基于 [academic-paper-writer](https://github.com/TFboy1/academic-paper-writer) 进行增强开发。

感谢原作者 [TFboy1](https://github.com/TFboy1) 提供的优秀基础框架。

### 增强内容

- **Pipeline E — Anti-AIGC 管道**：全新的降AI率处理能力
- **中文处理**：冗余扩展、词汇替换、句式微调、括号整合
- **英文处理**：词汇规范化、结构自然化、排版规范
- **技术术语保护**：自动识别并保护关键技术信息

---

## License

MIT License
