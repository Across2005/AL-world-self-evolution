# AL-world · AI Learning Art World

<p align="center">
  <img src="https://img.shields.io/badge/version-3.0.0-blue" alt="version">
  <img src="https://img.shields.io/badge/node-%3E%3D16-green" alt="node">
  <img src="https://img.shields.io/badge/python-%3E%3D3.8-blue" alt="python">
  <img src="https://img.shields.io/badge/license-MIT-green" alt="license">
</p>

> **具有自进化能力的 AI 绘画增强 Skill** —— 向量化语义检索 + 智能分类存储 + 自进化闭环。

AL-world 是一个面向 AI 绘画的知识库系统。它把「汉文化服饰元素」「自然人体绘画参考」「泛用绘画提示词」三大知识库统一管理，通过视觉模型（Kimi / StepFun / Qwen-VL）自动分析图片，提取可复用的生图提示词，并**智能分类、自动归档、自进化索引**。生图时自动调用索引匹配最相关的知识条目。

---

## ✨ 核心特性

| 特性 | 方向 | 说明 |
|------|------|------|
| 向量化语义检索 | 方向一 | 自然语言搜索替代关键词匹配，用向量找到最相关内容 |
| 三路融合分类 | 方向一 | 向量相似度 + 标签匹配 + 文本相似度 综合打分 |
| 新建分类自动注册 | 方向二 | 每次新建分类 → 自动更新 `category-map` + `tag-index` + `vector-index` |
| 自进化闭环 | 方向二 | 标签同义词表随新条目自动扩展，下次检索立刻识别新分类 |
| 评分反馈 | 方向二 | 用户可对提示词质量打分（`vote_score` / `quality` 标记） |
| 视觉模型桥接 | —— | 接入 Kimi / StepFun / Qwen-VL 分析图片 → 提取 prompt → 自动归类 |

---

## 📦 目录结构

```
AL-world/
├── SKILL.md                   # Skill 入口与说明
├── README.md                  # 本文件
├── knowledge/                 # 三大知识库（已清洗，仅含可复用提示词）
│   ├── hanfu-culture/         # 汉文化服饰元素库（形制 / 纹样 / 配色）
│   │   ├── index.json
│   │   └── prompts.json
│   ├── figure-drawing/        # 人体绘画参考库（解剖 / 姿态 / 光照）
│   │   ├── index.json
│   │   └── prompts.json
│   └── art-prompts/           # 泛用绘画提示词库（风格 / 构图 / 氛围）
│       ├── index.json
│       └── prompts.json
├── services/                  # 核心引擎
│   ├── config.json            # 视觉模型 + 阈值配置
│   ├── vision-bridge.js       # 视觉模型桥接（Node.js）
│   ├── vector-engine.py       # 向量检索 + 索引自进化（Python）
│   ├── knowledge-pipeline.py  # 知识流水线 + 自进化闭环（Python）
│   ├── classify.py            # 共享分类引擎（Python）
│   └── package.json
├── scripts/
│   ├── al-init.sh             # 首次部署配置向导（含视觉模型 API 提示）
│   ├── al-analyze.sh          # 单张图片分析全流程
│   ├── al-batch.sh            # 批量分析
│   └── al-serve.sh            # 状态检查
├── index/                     # 全局索引（自进化）
│   ├── category-map.json      # 类别映射
│   ├── tag-index.json         # 标签倒排索引
│   ├── vector-index.json      # 向量索引
│   └── synonyms.json          # 自进化同义词表
└── docs/
    ├── VISION.md              # 视觉模型集成指南
    └── FUTURE.md              # 未来演进路线图
```

---

## 🚀 快速开始

### 1. 首次部署（配置视觉模型）

> **重要**：此 skill 需要配置一个视觉模型（Kimi / StepFun / Qwen-VL）才能发挥自进化功能。
> 运行初始化向导时会被询问「是否现在配置视觉模型」。您可以提供 Kimi 多模态 API 或其它视觉模型 API，向导会帮您写入环境变量。

```bash
# 交互式配置向导（写入 ~/.bashrc / ~/.zshrc）
bash scripts/al-init.sh

# 手动配置（二选一即可）
export KIMI_API_KEY="sk-xxxx"
export STEPFUN_API_KEY="sk-xxxx"
```

### 2. 重建向量索引

```bash
python services/vector-engine.py rebuild
```

### 3. 分析一张图片

```bash
bash scripts/al-analyze.sh /path/to/image.jpg kimi
```

流程：`vision-bridge.js` 调用视觉模型 → 结构化分析 → `knowledge-pipeline.py` 提取精华提示词 → 三路融合分类 → 请示用户保存到匹配 / 新建 / 指定分类 → 自动注册到全部索引。

---

## 📖 常用命令

| 命令 | 用途 |
|------|------|
| `bash scripts/al-init.sh` | 首次部署向导，询问是否配置视觉模型 API |
| `python services/vector-engine.py rebuild` | 重建所有向量索引 |
| `python services/vector-engine.py status` | 查看索引状态（条目数 / 维度 / 各库分布） |
| `python services/vector-engine.py search --query "唐代齐胸襦裙金色刺绣"` | 语义搜索 |
| `python services/classify.py search "赛博朋克霓虹夜景"` | 交互式 / 命令行搜索知识库 |
| `python services/classify.py list` | 列出全部已有条目 |
| `bash scripts/al-analyze.sh <图片> [kimi\|stepfun\|qwen]` | 单张图片分析全流程 |
| `bash scripts/al-batch.sh <目录> [模型] [上限]` | 批量分析 |
| `python services/knowledge-pipeline.py --input <结果.json> --auto` | 自动分类保存（跳过交互） |
| `python services/knowledge-pipeline.py --feedback "hanfu-culture:prompts.json:ming_mamianqun +1"` | 给提示词打分（+1 / -1） |

---

## 🔄 自进化机制

### 新建分类时，索引如何自动进化

当用户在分类对话框选择「新建分类」后，系统会一步完成以下注册（无需人工维护）：

1. **写入** `prompts.json` → 新条目
2. **注册** `category-map.json` → 添加 `{kb}-{sub}` 映射
3. **注册** `tag-index.json` → 新标签指向新条目
4. **注册** `vector-index.json` → 向量化后加入语义搜索库
5. **扩展** `synonyms.json` → 新标签加入同义词表

> 这意味着：**每一次新建分类后，下一次检索立刻就能匹配到这个新分类。**

### 反馈闭环

```
生图结果 → 用户投票 (👍 +1 / 👎 -1)
    ↓
knowledge-pipeline.py --feedback "kb:prompts.json:prompt_id +1"
    ↓
修改 prompts.json 中 vote_score / quality(high|medium|low)
    ↓
（规划中）下次向量检索时，高质量条目权重提升
```

> 注：当前版本已实现 `vote_score` 与 `quality` 的记录与持久化；检索阶段的「高质量加权」为规划特性，见 `docs/FUTURE.md`。

---

## 🧠 架构流程图

```
[用户]                    [AL-world]
  │                           │
  ├─ 提供图片 ───────────────→ vision-bridge.js
  │  ← 结构化分析结果 ────────┤
  ├─ 确认分类（新建/已有）───→│ knowledge-pipeline.py
  │                           ├─ 保存到 prompts.json
  │                           ├─ vector-engine.py add-entry
  │                           │   ├─ category-map.json +
  │                           │   ├─ tag-index.json +
  │                           │   ├─ vector-index.json +
  │                           │   └─ synonyms.json +
  │  ← "索引已进化" ──────────┤
  ├─ 下次生图 "唐代襦裙" ────→│ 自动语义搜索
  │                           │ vector-index + tag-index 命中新分类
  │  ← 最相关的提示词 ───────┤
  └─ 生图结果 👍 ────────────→│ knowledge-pipeline --feedback
                               └─ vote_score + 1 → quality: high
```

---

## 📊 知识库现状

| 知识库 | 提示词模板 | 分类维度 |
|--------|-----------|---------|
| `hanfu-culture` | 9 | 朝代形制 / 纹样 / 配色 / 公式 |
| `figure-drawing` | 8 | 胸部形态 / 拉伸姿态 / 站姿 / 解剖 |
| `art-prompts` | 6 | 风格 / 构图 / 光照 / 氛围 / 技法 |

> 原始知识清洗自本地 `D:\art\` 下的三个旧知识库，已去除非提示词内容，仅保留可复用的生图参数模板。

---

## 🛠 技术栈

- **Node.js**：`vision-bridge.js` 调用视觉模型 HTTP API，输出结构化 JSON。
- **Python 3**：向量引擎、分类引擎、知识流水线。
- **轻量级向量检索**：内置 `LightweightEmbedding`（128 维哈希 + 统计特征），无需外部依赖即可运行；预留接口可无缝切换至 `text2vec` / `bge` 等真实 Embedding 模型。

---

## 🗺 路线图

详见 [`docs/FUTURE.md`](docs/FUTURE.md)：

- **v3.0（当前）**：向量化检索 + 自进化分类 + 反馈闭环
- **v3.5**：多 Agent 联邦（风格 / 形制 / 解剖 / 质量 多模型协作）
- **v4.0**：动态知识图谱 + 实时生图提示词注入

---

## 📄 License

[MIT](LICENSE)
