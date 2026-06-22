# Prize Hunt — 全网搜寻可获奖金的竞赛并匹配你的技术栈

用户输入: $ARGUMENTS

---

## 你的任务

根据用户的 GitHub 用户名或技术栈描述，搜索全网可获奖金的竞赛/黑客松/Bounty，生成一份按匹配度排序的竞赛推荐报告。

**输入规则:**
- 如果 `$ARGUMENTS` 看起来像 GitHub 用户名（纯英文、无空格、无中文），走 **GitHub 分析**路径
- 如果 `$ARGUMENTS` 包含中文描述、技术栈关键词、或用户明确描述了经验，走 **手动描述**路径
- 如果 `$ARGUMENTS` 为空，直接询问用户提供 GitHub 用户名或技术栈描述

---

## Phase 1: 技术栈建模

### 路径 A — GitHub 用户名

**先读取同目录下的 `github-profile.md`，然后按其指令执行**，将 `$ARGUMENTS`（用户名）作为输入，获取完整技术栈画像。

`github-profile.md` 会完成：仓库分析 → 语言分布 → 依赖提取 → 领域标签 → 成熟度评估，并输出结构化画像。将其输出作为本阶段结果。

### 路径 B — 手动描述

解析 `$ARGUMENTS` 中的技术栈信息，提取：语言、框架、项目经验、擅长领域。描述不够详细时追问。

### 输出

将路径 A 或路径 B 的结果向用户展示，确认后再继续。等待用户确认。

---

## Phase 2: 竞赛搜索（并行执行）

> **关键：用 Agent 工具并行执行多组搜索，大幅提速。**

按以下分组，并行派发搜索 Agent（每组一个 Agent）：

### Agent 1: 国际黑客松平台
搜索关键词（逐一搜索）：
- `site:devpost.com hackathon open 2026`
- `site:mlh.io hackathon 2026`
- `hackathon 2026 prize money registration`
- `devpost hackathon open now prize`

### Agent 2: AI/ML 竞赛
搜索关键词：
- `site:kaggle.com competition prize 2026`
- `AI challenge 2026 submit project prize`
- `machine learning competition 2026 registration`
- `data science bounty 2026`

### Agent 3: 开源 Bounty & 企业竞赛
搜索关键词：
- `open source bounty program 2026`
- `github sponsors bounty 2026`
- `developer competition 2026 enterprise`
- `tech giant hackathon 2026 prize` (Microsoft, Google, Meta 等)

### Agent 4: 中文竞赛平台
搜索关键词：
- `天池大赛 2026 报名`
- `黑客马拉松 2026 奖金 报名`
- `AI竞赛 2026 大模型 比赛`
- `编程竞赛 2026 奖金`
- `企业赛 2026 技术大赛 报名`

### Agent 5: Web3 & 区块链（条件触发）
仅当 Phase 1 发现 Web3/区块链相关技术栈时触发：
- `web3 hackathon bounty 2026`
- `ethereum hackathon 2026 prize`
- `solana hackathon 2026`
- `blockchain bounty program 2026`

### Agent 6: 领域补充搜索（条件触发）
根据 Phase 1 的领域标签追加，例如：
- 前端 → `UI/UX hackathon 2026`、`frontend challenge 2026`
- 移动 → `mobile app competition 2026`
- 游戏 → `game jam 2026 prize`
- 安全 → `CTF competition 2026 prize`

### 每个 Agent 的输出格式

每个搜索 Agent 返回结构化列表，每条包含：
- 比赛名称
- 主办方
- 主题领域
- 奖金金额（注明原始币种，附 USD/CNY 换算）
- 截止日期（`YYYY-MM-DD` 格式，已过期的标注）
- 报名/详情链接
- 参赛形式（个人/团队/均可）
- 数据来源（哪个搜索结果）

### 过滤规则

**排除**：已截止超过 1 周的、仅限特定地区/学校的、无任何奖金或奖励的、链接已失效的

---

## Phase 3: 去重 & 匹配分析

### 去重

合并所有 Agent 的结果，按比赛名称 + 主办方去重。同一条目保留信息最完整的版本。

### 匹配评分

对每个竞赛评估：

| 维度 | 权重 | 评分标准 |
|------|------|---------|
| **领域匹配度** (0-5) | ×3 | 竞赛主题与用户技术栈的重合程度 |
| **项目复用度** (0-5) | ×3 | 是否有现成 GitHub 项目可直接提交或小幅改造 |
| **奖金吸引力** (0-5) | ×1 | 奖金金额相对投入时间的性价比 |
| **时间充裕度** (0-5) | ×2 | 截止日期是否给予足够开发时间 |
| **竞争激烈度** (0-5) | ×1 | 小众精品 > 大众赛事（5=小众，1=顶级大赛） |

**综合评分 = 领域匹配×3 + 项目复用×3 + 奖金×1 + 时间充裕×2 + 竞争×1**（满分 45）

**优先级标签**：
- 🥇 **高优先** — 项目复用度 ≥ 3（现有项目可参赛）
- 🥈 **推荐** — 领域匹配 ≥ 3 且时间充裕 ≥ 3
- 🥉 **可关注** — 其余有价值项

---

## Phase 4: Idea 生成（条件触发）

当 top 3 竞赛的项目复用度 < 3 时，为每个竞赛生成 1-2 个参赛 idea：

每个 idea 包含：
- **一句话描述**
- **技术路线**（匹配用户技术栈中的哪些能力）
- **MVP 范围**（最小可提交版本）
- **预估工时**（小时）
- **需额外学习的内容**（如有）
- **成功概率**（低/中/高，附简要理由）

---

## Phase 5: 输出报告

用 Bash 获取今天日期：`date +%Y-%m-%d`（记为 TODAY）

报告写入：`prize-hunt-report-${TODAY}.md`

报告结构：

```markdown
---
date: ${TODAY}
tags: [prize-hunt, 竞赛, hackathon]
---

# 🏆 Prize Hunt Report — ${TODAY}

## 📊 技术栈画像
（Phase 1 摘要）

## 🎯 竞赛推荐

### 🥇 高优先 — 现有项目可参赛
| 竞赛 | 奖金 | 截止日期 | 匹配项目 | 改造建议 | 评分 |
|------|------|----------|----------|----------|------|
| ... | ... | ... | ... | ... | ... |

### 🥈 推荐 — 技术匹配，需新建项目
（同上表格 + Phase 4 idea 摘要）

### 🥉 可关注
（简要列表）

## 💡 参赛 Ideas（详细版）
（Phase 4 生成的完整 idea，如有）

## 📋 行动计划
1. 近期（1-2 周内截止）→ 立即行动
2. 中期（1-2 月）→ 规划准备
3. 远期（3 月+）→ 持续关注

## 🔗 汇总链接
（所有竞赛的报名/详情链接，方便一键访问）
```

---

## 重要注意事项

- **并行搜索**：Phase 2 的 Agent 必须并行派发，不要串行等待
- **日期用实际值**：`YYYY-MM-DD` 格式，不用相对时间
- **奖金注明币种**：原始币种 + 换算
- **链接可访问性**：无法访问的链接在报告中注明
- **搜索覆盖度**：至少 5 组搜索 Agent，不要只搜一两组就停
- **可操作性**：用户看完报告就知道下一步该做什么
- **透明度**：向用户说明搜索了什么、找到了多少、过滤了什么
