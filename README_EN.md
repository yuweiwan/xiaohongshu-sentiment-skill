<div align="center">

# Xiaohongshu Sentiment Skill

> Use opencli to collect and analyze Xiaohongshu (Little Red Book) sentiment data, including user feedback, market trends, and competitor insights

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/Node.js-18%2B-green.svg)](https://nodejs.org)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-green)](https://agentskills.io)

<br>

Keyword search, post details, and comment analysis
Complete comment analysis methodology (sarcasm detection, noise filtering, sentiment classification)
Adapted for Xiaohongshu's anti-scraping mechanisms with rate limiting and error handling

[Demo](#demo) · [Install](#install) · [Project Structure](#project-structure) · [Usage](#usage) · [**中文**](README.md)

</div>

---

## Demo

### Sentiment Report Example

```markdown
# [Target] Xiaohongshu Sentiment Report

> Collected: 2026-04-16
> Time Range: Last 30 days
> Purpose: A. Broad user feedback research

## 1. Data Overview
- Posts: 45
- Comments: 230
- Unique Users: 180
- High-likes Comments (≥50): 12

## 2. Sentiment Analysis
### 2.1 Sentiment Distribution
| Sentiment | % | Representative Opinion |
|-----------|---|------------------------|
| Negative | 45% | Questioning authenticity, complaining about costs |
| Neutral | 30% | Technical questions, solution discussions |
| Positive | 25% | Sharing real use cases |

### 2.2 Core Topics
- Topic 1: Excessive marketing, user trust erosion
- Topic 2: High token costs, hard to scale
- Topic 3: Cross-border e-commerce is a real application scenario
```

### Comment Analysis Example

| Comment | Post Context | Real Sentiment |
|---------|--------------|----------------|
| "Lying in bed tapping my fingers, made -1145 today" | Money-making post | **Sarcastic self-deprecation** |
| "Looks like 280k was spent on marketing for 28k prize" | Money-making post | **Questioning marketing** |
| "We built a check-in bot and portfolio news bot" | Opportunity discussion | **Positive sharing** |

---

## Install

### Prerequisites

```bash
# Install opencli
npm install -g @jackwener/opencli

# Verify installation
opencli --version
opencli xiaohongshu --help
```

### Claude Code

```bash
# Install to current project (run in git repo root)
mkdir -p .claude/skills
git clone https://github.com/yuweiwan/xiaohongshu-sentiment-skill .claude/skills/xiaohongshu-sentiment

# Or install globally (available for all projects)
git clone https://github.com/yuweiwan/xiaohongshu-sentiment-skill ~/.claude/skills/xiaohongshu-sentiment
```

### OpenClaw

```bash
git clone https://github.com/yuweiwan/xiaohongshu-sentiment-skill ~/.openclaw/workspace/skills/xiaohongshu-sentiment
```

---

## Project Structure

```
xiaohongshu-sentiment-skill/
├── README.md                      # Chinese version
├── README_EN.md                   # This file
├── LICENSE
├── skill.md                       # Skill entry (complete usage workflow)
└── references/
    └── xiaohongshu-patterns.md     # Platform patterns (anti-scraping, error codes)
```

---

## Usage

### Workflow

```
1. Pre-check → Verify opencli availability
2. Confirm requirements → Define target, purpose, time range
3. Data collection → Search posts → Get details+comments (2-3s interval)
4. Sentiment analysis → Comment analysis (sarcasm detection, noise filtering, sentiment)
5. Output report → Standard report structure
```

### Key Parameters

| Parameter | Recommended | Description |
|-----------|-------------|-------------|
| Search count (quick check) | 3-5 | Pre-check |
| Search count (initial survey) | 10-20 | Single search |
| Search count (deep research) | 30-50 | 2-3 batches, 3-5s interval |
| Request delay (search) | 1-2s | Avoid rate limiting |
| Request delay (details/comments) | 2-3s | Comment data is sensitive |

### Collection Commands

```bash
# Search posts (results contain full URLs with xsec_token)
opencli xiaohongshu search "<keyword>" --limit <count> --format json

# Get post details - must use full URL (with xsec_token)
opencli xiaohongshu note "<full URL>" --format json

# Get comments
opencli xiaohongshu comments "<full URL>" --limit <count> --format json
```

### Comment Analysis Method

**Step 1: Read post before comments**
- Get post title, content, tags first to understand context
- Then read comments, judge real sentiment with context
- Same phrase in a praising post vs criticizing post can mean opposite things

**Step 2: Detect sarcasm**

| Comment Feature | Detection | Action |
|-----------------|-----------|--------|
| Exaggerated tone | Overly dramatic, sarcastic sentences | Probably ironic |
| Negative numbers/losses | "Made -XXX", "lost XXX" | Likely self-deprecation, not real negative |
| Mock flexing | Imitating original post's exaggerated numbers | Mocking the original post |
| Saying the opposite | Praising on surface, criticizing underneath | Need context |

**Step 3: Noise filtering**

| Type | Feature | Action |
|------|---------|--------|
| Soft ads/marketing | Frequent product promotion, purchase links | Tag separately, exclude from sentiment |
| Pure flex posts | Showing income, achievements | Verify authenticity |
| Trolls/provocateurs | No specific point, only emotions | Ignore |

**Step 4: Sentiment classification**

| Real Sentiment | Typical Expression |
|----------------|-------------------|
| Positive | Genuine thanks, specific approval, constructive suggestions |
| Negative | Specific criticism, rational questioning, reasoned objection |
| Neutral | Technical questions, cost discussions, objective statements |

---

## Error Handling

| Error Code | Meaning | Solution |
|------------|---------|----------|
| `SECURITY_BLOCK` | Xiaohongshu rate limiting | Retry with full URL from search results |
| `EMPTY_RESULT` | Page loaded with no content | May require login or post is restricted |
| `NOTE_NOT_FOUND` | Post doesn't exist or deleted | Skip this post |
| 3 consecutive failures | Strategy issue | Reduce frequency, use different URL, or change method |

---

## Platform Patterns

See [references/xiaohongshu-patterns.md](references/xiaohongshu-patterns.md), includes:

- **xsec_token verification**: Every post URL contains an auth token, losing or expiring it causes `SECURITY_BLOCK`
- **Request frequency**: Same IP concurrent requests trigger rate limiting, no concurrency allowed
- **Delay strategy**: Search 1-2s, post details/comments 2-3s
- **Batch deduplication**: Hot keyword search results may have duplicate posts, deduplicate by URL before batching

---

<div align="center">

MIT License © [yuweiwan](https://github.com/yuweiwan)

</div>
