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

## Phase 2: 竞赛搜索（多层降级策略）

> **核心原则：不依赖单一数据获取方式。按优先级逐层尝试，每层获取到数据就跳过下一层。**

并行派发 Agent 执行以下各层。每层内各 Agent 并行，层与层之间也并行（最终合并去重）。

---

### Layer 1: API/JSON 端点直取（最可靠）

这些平台有可直接返回 JSON 的端点，用 `curl` 即可获取，无需 JS 渲染。

#### Agent 1: Devpost 黑客松
```bash
curl -s "https://devpost.com/api/hackathons?status[]=open&per_page=50&page=1" | head -500
```
如果返回 JSON，提取 `hackathons` 数组中的 `title`、`prize_amount`、`submission_period_dates`、`url`。

#### Agent 2: MLH 赛季活动
```bash
curl -s "https://mlh.io/seasons/2026/events" -o /dev/null -w "%{http_code}"
curl -s "https://raw.githubusercontent.com/MLH/mlh-policies/main/seasons/2026.json" 2>/dev/null
```
备选：搜索 GitHub 上 `MLH` 官方 repo 的 events 数据文件。

#### Agent 3: Kaggle 竞赛
```bash
curl -s "https://www.kaggle.com/api/v1/competitions/list?search=&page=1" 2>/dev/null
```
如果返回 401，改用：
```bash
curl -s "https://www.kaggle.com/competitions?listOption=active" | grep -oP '"competitionName":"[^"]*"|"deadline":"[^"]*"|"rewardUrl":"[^"]*"' | head -100
```
备选：搜索 GitHub 上的 `kaggle-competition` awesome list。

#### Agent 4: 中文竞赛平台 API
```bash
# 天池
curl -s "https://tianchi.aliyun.com/api/competition/list?status=open&pageNo=1&pageSize=20" 2>/dev/null

# DataFountain
curl -s "https://www.datafountain.cn/api/competitions?status=open&page=1" 2>/dev/null
```
如果 API 不可用，尝试抓取页面中的 `<script>` 标签内嵌 JSON（很多 SSR 页面会内嵌数据）。

---

### Layer 2: GitHub 搜索（高度可靠）

GitHub 搜索不受 JS 渲染影响，且能找到大量竞赛相关信息。

#### Agent 5: GitHub 竞赛追踪仓库
```bash
# 搜索 awesome-hackathon 类列表
gh search repos "hackathon 2026" --sort stars --limit 10 --json name,description,url

# 搜索竞赛奖金相关 repo
gh search repos "prize competition developer 2026" --sort updated --limit 10 --json name,description,url

# 搜索中文竞赛整理
gh search repos "竞赛 黑客松 2026" --sort stars --limit 5 --json name,description,url
```

对找到的高价值 repo，读取其 README 获取竞赛列表。

#### Agent 6: GitHub Issue/讨论中的 Bounty
```bash
# 搜索带有 bounty 标签的 issue
gh search issues "bounty label:bounty" --sort created --limit 20 --json title,repository,url,createdAt

# 搜索 open source reward
gh search issues "reward open source" --sort created --limit 10 --json title,repository,url
```

---

### Layer 3: WebSearch 补充搜索（如果可用）

> **仅在 Layer 1/2 获取数据不足时执行。先测试 WebSearch 是否可用（搜索一个简单关键词），不可用则跳过整层。**

#### Agent 7: 英文通用搜索
- `hackathon 2026 prize open registration`
- `developer competition 2026 prize money`
- `AI challenge 2026 submit project`
- `open source bounty program 2026`

#### Agent 8: 中文通用搜索
- `黑客马拉松 2026 报名 奖金`
- `天池大赛 2026 报名`
- `AI竞赛 2026 大模型 比赛`
- `编程竞赛 2026 奖金`

#### Agent 9: 领域补充搜索（条件触发）
根据 Phase 1 领域标签追加：
- ML/AI → `data science competition 2026`、`kaggle prize 2026`
- Web3 → `web3 hackathon bounty 2026`
- 前端 → `UI/UX hackathon 2026`
- 游戏 → `game jam 2026 prize`
- 安全 → `CTF competition 2026 prize`

---

### Layer 4: 已知常青竞赛数据库（兜底）

> **当以上所有层都获取不到足够数据时，使用此层。这些是每年/每季举办的知名竞赛，基于已知信息生成，用户自行验证链接有效性。**

根据 Phase 1 的领域标签，从以下列表中筛选相关项输出：

**AI/ML 类：**
- Kaggle Seasonal Competitions（常年有，奖金 $10K-$100K+）
- ARC Prize（AI 推理能力，$1M+ 奖池）
- NeurIPS/ICML/CVPR Workshop Challenges（随顶会周期）
- AIcrowd Challenges（常年有）

**黑客松类：**
- MLH Season（每年 9 月-次年 8 月，全球 100+ 场）
- Devpost Featured Hackathons（常年有线上赛）
- ETHGlobal 系列（Web3，每年多场）
- HackMIT / PennApps / HackGT 等高校赛（秋季为主）

**开源 Bounty 类：**
- Google Summer of Code（每年 2 月申请）
- GitHub Sponsors / IssueHunt（常年有赏金 issue）
- Gitcoin Grants（Web3，季度性）

**中文平台类：**
- 天池大赛（阿里，常年有）
- DataFountain（常年有）
- 和鲸 Heywhale（常年有）
- 华为/百度/腾讯 各自的 AI 竞赛（不定期）

**企业赛类：**
- Microsoft Imagine Cup（年度）
- Google Solution Challenge（年度）
- Hack the North（加拿大，年度）

**使用此层时必须在报告中标注：「以下为已知常青竞赛，具体时间和奖金请访问官网确认」**

---

### 每个 Agent 的统一输出格式

每个 Agent 返回结构化列表，每条包含：
- 比赛名称
- 主办方
- 主题领域
- 奖金金额（注明原始币种，附 USD/CNY 换算）
- 截止日期（`YYYY-MM-DD` 格式，已过期的标注）
- 报名/详情链接
- 参赛形式（个人/团队/均可）
- 数据来源（具体哪个 Layer/Agent/URL）
- **置信度**（高=API直取 / 中=页面解析 / 低=已知常青）

### 过滤规则

**排除**：已截止超过 1 周的、仅限特定地区/学校的、无任何奖金或奖励的、链接已失效的

### 搜索完成后

向用户透明报告：
- 各 Layer 的执行状态（成功/跳过/失败）
- 每个 Layer 获取到多少条有效竞赛
- 总共去重后多少条进入匹配分析

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

- **降级策略**：Layer 1→4 逐层尝试，不是所有层都必须成功。只要有数据进入 Phase 3 即可
- **并行搜索**：每层内的 Agent 并行派发，不同层之间也可并行
- **工具可用性检测**：执行 Layer 3 前先测试 WebSearch 是否可用，不可用则跳过
- **curl 优先**：能用 `curl` 获取的数据不要用 WebFetch，curl 不受域名验证限制
- **日期用实际值**：`YYYY-MM-DD` 格式，不用相对时间
- **奖金注明币种**：原始币种 + 换算
- **链接可访问性**：无法访问的链接在报告中注明
- **置信度标注**：区分 API 直取（高）、页面解析（中）、已知常青（低）
- **可操作性**：用户看完报告就知道下一步该做什么
- **透明度**：向用户报告每层搜索的状态和获取数量
