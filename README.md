<div align="center">

# 小红书舆情获取 Skill

> 使用 opencli 工具获取小红书舆情数据，分析用户反馈、市场热度与竞品动态

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/Node.js-18%2B-green.svg)](https://nodejs.org)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-green)](https://agentskills.io)

<br>

评论才是舆情金矿 · 模板自动匹配 · 适配小红书反爬机制

[效果示例](#效果示例) · [安装](#安装) · [项目结构](#项目结构) · [使用说明](#使用说明) · [**English**](README_EN.md)

</div>

---

## 效果示例

### 舆情报告 / 调研报告

根据监测对象自动匹配模板（详见 [templates/](templates/)）：

- **舆情报告** — 单一品牌/产品、话题/事件
- **调研报告** — 竞品对比、选购分析

```markdown
# [监测对象] 小红书舆情报告

> 采集时间：YYYY-MM-DD
> 时间范围：XXXX

## 一、数据概况
- 帖子数量：45
- 评论数量：230
- 独立用户数：180
- 高赞评论（≥50赞）数：12

## 二、舆情分析
### 3.1 情绪分布
| 情绪 | 占比 | 代表观点 |
|------|------|----------|
| 负面 | 45% | 质疑真实性、吐槽成本 |
| 中性 | 30% | 问技术问题、讨论方案 |
| 正面 | 25% | 实际应用案例分享 |
```

### 评论分析方法示例

| 评论原文 | 帖子背景 | 真实情绪 |
|----------|----------|----------|
| "今天躺床上动了动手指，赚了-1145" | 赚钱帖 | **自嘲讽刺** |
| "看似28w获奖，实则砸了280w宣传" | 赚钱帖 | **质疑营销** |
| "我们做了打卡bot和持仓新闻bot" | 机会讨论帖 | **正面分享** |

---

## 安装

### 前置要求

```bash
# 安装 opencli
npm install -g @jackwener/opencli

# 验证安装
opencli --version
opencli xiaohongshu --help
```

### Claude Code

```bash
# 安装到当前项目（在 git 仓库根目录执行）
mkdir -p .claude/skills
git clone https://github.com/yuweiwan/xiaohongshu-sentiment-skill .claude/skills/xiaohongshu-sentiment

# 或安装到全局（所有项目都能用）
git clone https://github.com/yuweiwan/xiaohongshu-sentiment-skill ~/.claude/skills/xiaohongshu-sentiment
```

### OpenClaw

```bash
git clone https://github.com/yuweiwan/xiaohongshu-sentiment-skill ~/.openclaw/workspace/skills/xiaohongshu-sentiment
```

---

## 项目结构

```
xiaohongshu-sentiment-skill/
├── README.md                      # 本文件
├── README_EN.md                   # English version
├── LICENSE
├── skill.md                       # Skill 入口（原则驱动，无表格）
├── templates/
│   ├── 舆情报告.md                 # 品牌/产品口碑模板
│   └── 调研报告.md                 # 竞品对比/市场调研模板
└── references/
    └── xiaohongshu-patterns.md    # 平台经验（反爬机制、错误码）
```

---

## 使用说明

### 核心原则

**评论才是舆情金矿。** 只爬帖子不爬评论等于只看到表面。分析评论时必须先读懂帖子背景——同一句话在赞美帖和批评帖中含义完全相反。

### 流程概览

```
1. 前置检查 → 验证 opencli 可用性
2. 需求确认 → 监测对象、爬取目的、时间范围
3. 数据采集 → 搜索 → 去重排序 → 获取详情+评论（随机1-3秒间隔）
4. 评论分析 → 先读帖再读评、识别阴阳怪气、去噪
5. 输出报告 → 自动匹配模板（可自行增减章节）
```

### 反爬机制

- **请求频率**：每次操作间隔随机 1-3 秒，禁止并发
- **URL要求**：必须使用搜索结果中的完整URL（含xsec_token），禁止 bare note ID
- **错误处理**：SECURITY_BLOCK → 换用完整URL重试；NOTE_NOT_FOUND → 跳过；连续3篇失败 → 降低频率或换方法

### 采集命令

```bash
# 搜索帖子
opencli xiaohongshu search "<关键词>" --limit <数量> --format json

# 获取帖子详情
opencli xiaohongshu note "<完整URL>" --format json

# 获取评论
opencli xiaohongshu comments "<完整URL>" --limit <数量> --format json
```

### 评论分析三原则

**先读帖再读评。** 先获取标题、正文、标签了解背景，再看评论结合上下文判断。

**识别阴阳怪气。** 夸张语气可能是讽刺；"赚了-XXX"是自嘲而非真负面；模仿炫富是在讽刺原帖；正话反说需结合整体语境判断。

**去噪。** 软广/营销号（频繁推产品带购买链接）单独标注不计入舆情；无具体观点的纯情绪杠精忽略。

### 报告模板

根据监测对象语义自动匹配：

- 含"vs"、"对比"、"竞品"、"选购" → `templates/调研报告.md`
- 单一品牌/产品、话题/事件 → `templates/舆情报告.md`

模板仅为参考，可根据调研到的有价值内容自行增减章节或创造新模板。

---

## 平台经验

详见 [references/xiaohongshu-patterns.md](references/xiaohongshu-patterns.md)，包含已验证的平台特征、有效模式与已知陷阱。

---

<div align="center">

MIT License © [yuweiwan](https://github.com/yuweiwan)

</div>
