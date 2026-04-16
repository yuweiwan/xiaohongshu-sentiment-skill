<div align="center">

# 小红书舆情获取 Skill

> 使用 opencli 工具获取小红书舆情数据，分析用户反馈、市场热度与竞品动态

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/Node.js-18%2B-green.svg)](https://nodejs.org)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-green)](https://agentskills.io)

<br>

支持关键词搜索、帖子详情采集、评论分析  
包含完整的评论分析方法（识别阴阳怪气、去噪、情绪归类）  
适配小红书反爬机制，包含请求频率控制和错误处理

[效果示例](#效果示例) · [安装](#安装) · [项目结构](#项目结构) · [使用说明](#使用说明) · [**English**](README_EN.md)

</div>

---

## 效果示例

### 舆情报告示例

```markdown
# [监测对象] 小红书舆情报告

> 采集时间：2026-04-16
> 时间范围：近30天
> 爬取目的：A. 广泛调研用户反馈

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

### 3.2 核心话题
- 话题1：营销过度，用户信任透支
- 话题2：Token成本过高，难以规模化
- 话题3：跨境电商是真实应用场景
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
├── skill.md                       # Skill 入口（含完整使用流程）
└── references/
    └── xiaohongshu-patterns.md     # 平台经验文档（反爬机制、错误码）
```

---

## 使用说明

### 流程概览

```
1. 前置检查 → 验证 opencli 工具可用性
2. 需求确认 → 明确监测对象、爬取目的、时间范围
3. 数据采集 → 搜索帖子 → 获取详情+评论（间隔2-3秒）
4. 舆情分析 → 评论分析方法（阴阳怪气识别、去噪、情绪归类）
5. 输出报告 → 标准报告结构
```

### 关键参数

| 参数 | 推荐值 | 说明 |
|------|--------|------|
| 搜索数量（快速验证） | 3-5 个 | 预检测试用 |
| 搜索数量（初步摸底） | 10-20 个 | 单次搜索 |
| 搜索数量（深度调研） | 30-50 个 | 分2-3批，间隔3-5秒 |
| 请求延迟（搜索） | 1-2 秒 | 避免风控 |
| 请求延迟（详情/评论） | 2-3 秒 | 评论数据敏感 |

### 采集命令

```bash
# 搜索帖子（搜索结果已包含完整URL，可直接用于后续调用）
opencli xiaohongshu search "<关键词>" --limit <数量> --format json

# 获取帖子详情 - 必须使用完整URL（含xsec_token）
opencli xiaohongshu note "<完整URL>" --format json

# 获取评论
opencli xiaohongshu comments "<完整URL>" --limit <数量> --format json
```

### 评论分析方法

**第一步：读帖子再读评论**
- 先获取帖子标题、正文、标签，了解帖子背景
- 再看评论，结合上下文判断真实情绪
- 同一句话在赞美帖和批评帖中含义可能完全相反

**第二步：识别阴阳怪气**

| 评论特征 | 识别方式 | 处理 |
|----------|----------|------|
| 夸张语气 | 过度感叹、反讽句式 | 可能是讽刺 |
| 负数/亏损 | "赚了-XXX"、"亏了XXX" | 大概率自嘲，不是真负面 |
| 模仿炫富 | 模仿原帖夸张数字 | 讽刺原帖 |
| 正话反说 | 表面夸实际贬 | 需结合上下文 |

**第三步：去噪**

| 类型 | 特征 | 处理 |
|------|------|------|
| 软广/营销号 | 频繁推荐产品、带购买链接 | 单独标注，不计入舆情 |
| 纯炫富帖 | 晒收入、晒成果 | 需判断是否真实 |
| 杠精/引战 | 无具体观点只有情绪 | 忽略 |

**第四步：情绪归类**

| 真实情绪 | 典型表达 |
|-----------|----------|
| 正面 | 真诚感谢、具体认可、提出建设性意见 |
| 负面 | 具体批评、理性质疑、有理有据反对 |
| 中性 | 问技术问题、讨论成本、客观陈述 |

---

## 错误处理

| 错误码 | 含义 | 处理方式 |
|--------|------|----------|
| `SECURITY_BLOCK` | 小红书风控拦截 | 换用搜索结果中的完整 URL 重试 |
| `EMPTY_RESULT` | 页面加载无内容 | 可能需要登录或帖子已受限 |
| `NOTE_NOT_FOUND` | 帖子不存在或已被删除 | 跳过该帖子 |
| 连续 3 篇获取失败 | 策略问题 | 降低频率、换用 URL 或切换方法 |

---

## 平台经验

详见 [references/xiaohongshu-patterns.md](references/xiaohongshu-patterns.md)，包含：

- **xsec_token 验证**：每个帖子 URL 都包含身份验证 token，丢失或过期会导致 `SECURITY_BLOCK`
- **请求频率**：同一 IP 并发请求易触发风控，禁止并发
- **延迟策略**：搜索 1-2 秒，帖子详情/评论 2-3 秒
- **分批去重**：热门关键词搜索结果可能有重复帖子，按 url 去重后分批采集

---

<div align="center">

MIT License © [yuweiwan](https://github.com/yuweiwan)

</div>
