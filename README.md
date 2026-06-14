# Obsidian 医学科研 Wiki — LLM 驱动的文献管理框架

丢论文进去，LLM 自动入库、交叉引用、追踪阅读进度、发现研究空白。

## 安装

### 1. 确认前置工具

本框架依赖两个工具，请先安装：

| 工具 | 安装方式 | 验证 |
|------|---------|------|
| [Obsidian](https://obsidian.md/) | 官网下载 macOS/Windows/Linux 客户端 | 打开任意 vault，设置 → 核心插件 → 确保 **Bases** 已启用 |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) | 终端运行 `npm install -g @anthropic-ai/claude-code` | 终端运行 `claude --version` |

### 2. 获取框架

```bash
cd ~/Documents/Obsidian                           # 或其他存放 vault 的目录
git clone https://github.com/ENIGMA-K/med-research-wiki.git
```

### 3. 在 Obsidian 中打开

1. 启动 Obsidian
2. 点击「打开其他 Vault」→「打开本地文件夹」
3. 选择刚 clone 的 `med-research-wiki` 文件夹
4. 进入设置 → 核心插件 → 确认 **Bases** 已启用
5. 在右侧边栏打开 `wiki/synthesis/文献索引.base`，确认能正常显示（此时为空表）

### 4. 在 Claude Code 中打开

```bash
cd ~/Documents/Obsidian/med-research-wiki
claude
```

Claude Code 会自动加载 `CLAUDE.md` 和 `.claude/` 下的所有配置。输入 `/ingest` 测试，看到命令被识别即表示安装成功。

### 5. 开始使用

```bash
# 1. 把第一篇论文 PDF 拖进 raw/artical/
# 2. 对 Claude Code 说：
/ingest
```

## 前置工具

只需要两个：

| 工具 | 用途 | 安装 |
|------|------|------|
| Obsidian | 浏览 wiki 知识库、查看 BASE 数据库 | [obsidian.md](https://obsidian.md/) |
| Claude Code | LLM 引擎，执行所有工作流 | `npm install -g @anthropic-ai/claude-code` |

不需要安装任何 Obsidian 第三方插件。本框架只用 Obsidian 核心插件 Bases，其他功能由 Claude Code 的 `.claude/` 配置驱动。

## 目录结构

```
├── CLAUDE.md              # LLM 工作流说明书（13 个工作流，修改行为改这里）
├── README.md              # 本文档
├── .claude/               # Claude Code 扩展配置
│   ├── commands/   (9个)   #   /ingest /blast /pub-research /gap /lint /query /export /logit /refac
│   ├── skills/     (3个)   #   方法论审视者 / PICO 解构者 / 论文写作顾问
│   └── agents/     (4个)   #   Blast引擎 / 学术爬虫 / Gap扫描器 / Lint检查器
├── raw/                    # 丢原始文件
│   ├── artical/            #   论文 PDF
│   ├── data/               #   原始数据（Excel/CSV）
│   └── assets/             #   图片、图表
├── wiki/                   # LLM 自动维护的结构化知识
│   ├── entities/           #   作者、机构、期刊、疾病
│   ├── concepts/           #   方法、术语、框架
│   ├── summaries/          #   每篇论文一页摘要
│   └── synthesis/          #   文献综述、空白分析、论文大纲
│       ├── 文献索引.base    #   论文主索引（Obsidian BASE）
│       ├── 期刊数据库.base  #   期刊 CiteScore/h-index 缓存
│       └── 术语表.base      #   中英医学术语库
├── work/                   # 个人草稿区（LLM 不主动修改）
│   ├── tables/             #   基线表、统计结果
│   ├── manuscripts/        #   论文草稿
│   ├── code/               #   分析脚本
│   ├── notes/              #   个人笔记
│   └── log/                #   每日工作日志
└── index.md                # Wiki 页面目录索引
```

## 使用指南

### 读一篇论文

**有 PDF**：拖进 `raw/artical/`，对 Claude Code 说 `/ingest`。LLM 自动：读 PDF → 提取研究问题/方法/人群/发现 → 评估方法学质量 → 创建作者实体页 → 提取中英术语 → 写摘要页 → 更新索引。

**只有 DOI / PMID**：直接说，LLM 拉取元数据入库（`pdf_status: to-fetch`），后续补 PDF 后状态自动变 `downloaded`。

录入后在 Obsidian 中打开 `wiki/synthesis/文献索引.base`，四个视图：

| 视图 | 看什么 |
|------|--------|
| 全部文献 | 所有已录入论文，按年份降序 |
| 待录入 | `status: stub` 的占位页 |
| 待获取全文 | `pdf_status: to-fetch`，提醒找 PDF |
| 预印本/会议 | `source_type: preprint` 或 `conference` |

### 追踪阅读进度

每篇论文有 5 级阅读状态。在 Claude Code 中直接说：

| 你说 | 效果 |
|------|------|
| 「**已速览** Zhang 2023」 | `reading_status` → `skimmed` |
| 「**已读** Ferreira 2018」 | `reading_status` → `read` |
| 「**已分析** 那篇 T1 mapping」 | `reading_status` → `analyzed` |

`文献索引.base` 按「阅读进度」列排序，一眼看到哪些读了、哪些还没读。

### 找文献（还没种子论文）

`/pub-research <研究问题>` — 同时搜 PubMed、Semantic Scholar、Europe PMC、arXiv、百度学术、知网、万方、维普。产出 `wiki/synthesis/文献调研_主题_日期.md`，含 Top 20 核心文献、主题分布、**交叉验证**专节（共识/争议/中英差异/待验证）。

问题太宽时，先说「**PICO 分析**」激活 Skill，LLM 帮你拆成 P-I-C-O 再搜。

### 追引用链（已有种子论文）

`/blast <DOI> [--depth N] [--threshold T]` — 从种子沿参考文献向后追。默认 depth=1, threshold=0.8。产出 `raw/blast_种子_d1.md`。重要性按期刊 CiteScore×0.6 + h-index×0.4 排序，期刊缓存于 `wiki/synthesis/期刊数据库.base`。

### 找研究空白

`/gap [--scope <方向|论文>]` — 五维扫描：人群/方法/对照/结局/时间空白。产出 `wiki/synthesis/Gap分析_范围_日期.md`，每个空白标注优先级（1-5 ★），可直接作为课题选题参考。

### 维护与导出

| 操作 | 说明 |
|------|------|
| `/lint` | 健康检查：矛盾、孤页、过期 stub、缺失页面 |
| 说「**刷新期刊**」 | 更新期刊 CiteScore/h-index 缓存 |
| `/export <页面> [--format bibtex\|ris]` | 导出参考文献到 `work/manuscripts/` |
| `/logit` | 生成每日工作日志到 `work/log/` |

### Skill 怎么用

Skill 不单独执行，而是改变其他操作的执行深度。说出触发词后，后续对话自带对应视角：

| Skill | 触发词 | 效果 |
|-------|--------|------|
| 方法论审视者 | 「审视这篇」 | 入库时自动从 8 维度评估方法学，标注严重问题 |
| PICO 解构者 | 「PICO 分析」 | 检索前先拆成 P-I-C-O，构建精准检索式 |
| 论文写作顾问 | 「写作建议」 | 每个发现提示可嵌入论文哪个章节，自动写段落草稿 |

### 快速参考

```
┌─ 发现文献 ────────────────────────────────┐
│ /pub-research <问题>   多库检索摸底       │
│ /blast <DOI> [--depth] 追引用链          │
│ /gap [--scope 方向]    找研究空白        │
├─ 管理文献 ────────────────────────────────┤
│ /ingest                录入论文          │
│ "已读 XX"              更新阅读进度       │
│ /lint                  健康检查          │
├─ 产出 ────────────────────────────────────┤
│ /export <页面>         导出 BibTeX/RIS   │
│ /logit                 每日工作日志       │
├─ 思维增强（Skill） ────────────────────────┤
│ "审视这篇"             方法学评估         │
│ "PICO 分析"            结构化检索         │
│ "写作建议"             论文素材整理       │
└──────────────────────────────────────────┘
```

## 数据源

Semantic Scholar / Europe PMC / PubMed / arXiv, medRxiv, bioRxiv / 百度学术 / 知网, 万方, 维普

> ⚠️ 知网/万方/维普仅能获取题录，无法获取摘要和全文。

## 自定义

1. **换领域**：编辑 `CLAUDE.md` 的「领域标签体系」节
2. **改行为**：编辑 `.claude/commands/` 下的对应文件
3. **加功能**：参考已有格式在 `.claude/` 中添加新文件

## 许可证

MIT
