<div align="center">

# Xiaohongshu Sentiment Skill

> Use opencli to collect and analyze Xiaohongshu (Little Red Book) sentiment data, including user feedback, market trends, and competitor insights

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Node.js](https://img.shields.io/badge/Node.js-18%2B-green.svg)](https://nodejs.org)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blueviolet)](https://claude.ai/code)
[![AgentSkills](https://img.shields.io/badge/AgentSkills-Standard-green)](https://agentskills.io)

<br>

Comments are the real gold mine · Auto template matching · Adapted for Xiaohongshu anti-scraping

[Demo](#demo) · [Install](#install) · [Project Structure](#project-structure) · [Usage](#usage) · [**中文**](README.md)

</div>

---

## Demo

### Sentiment Report / Research Report

Templates auto-match based on monitoring object (see [templates/](templates/)):

- **Sentiment Report** — single brand/product, topic/event
- **Research Report** — competitor comparison, purchase guides

```markdown
# [Target] Xiaohongshu Sentiment Report

> Collected: YYYY-MM-DD
> Time Range: XXXX

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
├── skill.md                       # Skill entry (principle-driven, no tables)
├── templates/
│   ├── 舆情报告.md                 # Brand/product sentiment template
│   └── 调研报告.md                 # Competitor/market research template
└── references/
    └── xiaohongshu-patterns.md    # Platform patterns (anti-scraping, error codes)
```

---

## Usage

### Core Principle

**Comments are the real gold mine.** Getting posts without comments is just scratching the surface. Always read the post context first — the same phrase can mean opposite things in a praising post vs a criticizing post.

⚠️ **CRITICAL: Strict Rate Limiting** ⚠️

Every operation must be spaced 2-3 seconds apart. No concurrency allowed. Violating this rule will immediately trigger risk control, disconnect the Chrome debugger, and cause all subsequent operations to fail. This is a mandatory requirement with no exceptions.

### Workflow

```
1. Pre-check → Verify opencli availability
2. Confirm requirements → Define target, purpose, time range
3. Search keywords → Get post list (mandatory 2-3s wait between searches)
4. Deduplicate & sort by likes → Select high-value posts
5. Get post details sequentially → Wait 2-3s after each post
6. Get comments sequentially → Wait 2-3s after each post
7. Comment analysis → Read post first, detect sarcasm, filter noise
8. Output report → Auto-matched template (flexible, add/remove sections as needed)
```

### Pre-flight Checklist

Before starting collection, you MUST confirm:

- [ ] Confirm every operation will be spaced 2-3 seconds apart
- [ ] Confirm no concurrency / background tasks will be used
- [ ] Confirm scripts implement `time.sleep(random.uniform(2, 3))`
- [ ] If running commands manually, wait 3 seconds after each command

**Consequences of rate limit violations**: Chrome debugger disconnect → All subsequent opencli commands fail → 10-15 minute cooldown required → May need Chrome restart or re-login.

### Anti-Scraping

- **Request frequency**: Mandatory 2-3s interval between operations, no concurrency; searches also require 5-10s intervals
- **URL requirement**: Must use full URL from search results (with xsec_token), bare note ID is forbidden; note that `search_result` URLs cannot be used directly for note/comments commands
- **Error handling**: SECURITY_BLOCK → retry with full URL or stop and wait 10-15 minutes; NOTE_NOT_FOUND → skip; 3 consecutive failures → reduce frequency or change method

### Collection Commands

```bash
# Search posts
opencli xiaohongshu search "<keyword>" --limit <count> --format json

# Get post details
opencli xiaohongshu note "<full URL>" --format json

# Get comments
opencli xiaohongshu comments "<full URL>" --limit <count> --format json
```

### Comment Analysis: 3 Principles

**Read post before comments.** Get title, content, tags first to understand context, then read comments with that context in mind.

**Detect sarcasm.** Exaggerated tone may be irony; "made -XXX" is self-deprecation not real negative; mock flexing is mocking the original post; sarcasm needs full context.

**Filter noise.** Soft ads/marketing accounts (frequent product promotion with purchase links) are tagged separately and excluded from sentiment; emotional trolls with no specific point are ignored.

### Report Templates

Auto-matched by monitoring object semantics:

- Contains "vs", "comparison", "competitor", "purchase guide" → `templates/调研报告.md`
- Single brand/product, topic/event → `templates/舆情报告.md`

Templates are reference structures only — adapt, extend, or create new templates based on what you actually find.

---

## Platform Patterns

See [references/xiaohongshu-patterns.md](references/xiaohongshu-patterns.md) for verified platform characteristics, effective patterns, and known pitfalls.

---

<div align="center">

MIT License © [yuweiwan](https://github.com/yuweiwan)

</div>
