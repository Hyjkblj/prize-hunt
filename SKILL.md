---
name: prize-hunt
description: Find and recommend current prize competitions, hackathons, developer challenges, data/AI contests, grants, and open source bounty programs with official web verification and fit scoring. Use when the user asks to discover contests with prizes, match competitions to a GitHub profile or tech stack, compare hackathons, find Kaggle/AI/data challenges, search bounties, or generate a prize-hunt report.
---

# Prize Hunt

Use this skill to discover active or upcoming opportunities with real rewards, verify them from current web sources, and rank them against the user's GitHub profile, project history, or stated tech stack.

## Non-negotiables

- Treat contest data as time-sensitive. Always use available internet search, browsing, or direct HTTP/API fetches before recommending anything.
- Prefer official pages and mainstream platforms. Use aggregator pages only to discover candidates, then verify each candidate on the official page.
- Do not rely on memory for dates, prizes, status, or eligibility. If a field cannot be verified, mark it `Unknown` and lower confidence.
- Do not claim that web search is unavailable until at least one available search/browse mechanism and direct official HTTP fetches have been attempted.
- Keep obscure or niche contests out of the main recommendations unless they are unusually well matched, clearly active, and backed by a reputable sponsor.
- Use absolute dates in `YYYY-MM-DD` format. Never use only "today", "soon", "next month", or similar relative dates.

## Input Handling

- If the input looks like a GitHub username, analyze it with `github-profile.md`, then use the resulting profile for matching.
- If the input is a free-form description, extract languages, frameworks, domains, reusable projects, time budget, region, language preference, and team preference.
- If the input is empty or too vague, ask for either a GitHub username or a short stack/project description.
- If useful constraints are missing, make reasonable defaults: remote/global preferred, individual or small team acceptable, cash/prize/grant preferred over swag.

## Current Date

Record the current date before filtering deadlines:

- Linux/macOS shell: `date +%F`
- PowerShell: `Get-Date -Format yyyy-MM-dd`
- Dedicated time/date tool: use its returned date and timezone

Use that date in the report metadata and in deadline filtering.

## Search Workflow

Run the search in this order. Parallelize independent searches when the environment supports it.

### 1. Mainstream Official Sources

Search or browse at least six relevant sources from Tier A/B before using broad web results.

Tier A, preferred sources:

- Devpost open hackathons: `devpost.com/hackathons`
- Kaggle active competitions: `kaggle.com/competitions`
- Challenge.gov open prize challenges: `challenge.gov`
- MLH events for student-friendly hackathons: `mlh.io`
- Microsoft Imagine Cup, Google Solution Challenge, and similar official annual developer contests when currently open
- Tianchi, DataFountain, Heywhale, and other official Chinese data/AI competition platforms when Chinese-language participation is acceptable

Tier B, useful but more domain-specific:

- AIcrowd, DrivenData, Zindi, EvalAI, CodaLab, Codabench
- HackerEarth challenges and Devfolio hackathons
- ETHGlobal, DoraHacks, Gitcoin, and ecosystem-specific bounty boards, only when Web3/open-source funding fits the user
- GitHub issues or discussions with official `bounty`, `reward`, or funded labels, only when the repository is active and the bounty terms are clear

Tier C, fallback and watchlist:

- Seasonal conferences and workshop challenges such as NeurIPS, ICML, CVPR, ACL, KDD, SIGGRAPH, DEF CON, or domain-specific calls
- Evergreen programs such as Google Summer of Code, only when the current cycle is open or announced
- Curated lists and newsletters, only as discovery surfaces that lead back to official pages

### 2. Search Query Patterns

Use official-domain searches first, then domain-specific searches based on the profile:

- `site:devpost.com hackathon prize open registration`
- `site:kaggle.com/competitions active competition prize`
- `site:challenge.gov open prize challenge submissions`
- `site:mlh.io events hackathon`
- `site:tianchi.aliyun.com 竞赛 报名 奖金`
- `site:datafountain.cn 竞赛 报名 奖金`
- `site:heywhale.com 竞赛 奖金`
- `AI challenge prize open submissions`
- `developer competition prize open registration`
- `open source bounty issue reward`

Add profile-specific terms:

- ML/AI/data: `machine learning competition prize`, `data science challenge open`
- Web/full-stack: `developer hackathon API challenge prize`, `startup hackathon prize`
- Web3: `web3 hackathon bounty prize`, `ETHGlobal hackathon`
- Security: `CTF prize open registration`, `bug bounty challenge`
- Game: `game jam prize open registration`
- Hardware/IoT: `IoT challenge prize open submissions`

### 3. Direct Fetch Fallback

When search tools are limited, fetch official pages directly with `curl -L --compressed` or equivalent. If a page is JavaScript-heavy, use a browser tool if available. If no live verification is possible, be explicit in the report and keep those results out of the main ranking.

## Candidate Evidence

For every candidate, capture:

- Competition name
- Host/sponsor
- Official URL
- Source tier and discovery method
- Status: open, upcoming, rolling, closed, unknown
- Deadline in `YYYY-MM-DD`, or `Unknown`
- Prize/reward and original currency
- Eligibility and region limits
- Remote/onsite/team requirements
- Theme/domain and required deliverable
- Why it matches the user's stack or projects
- Confidence: high, medium, low

High confidence requires an official reachable page with current status and deadline or a clearly open rolling program.

## Filtering Rules

Exclude from main recommendations:

- Deadline is more than seven days in the past.
- No concrete reward, prize, grant, funding, bounty, internship, credits of meaningful value, or judge-recognized award.
- Only local to a region, school, company, or membership group that the user has not indicated they belong to.
- Official page is unavailable and no reliable current source confirms status.
- Rules, eligibility, or submission requirements are too vague to act on.
- The source appears inactive, abandoned, or mainly promotional.

Allow exceptions only when the opportunity is a strong domain fit and clearly marked as a watchlist item.

## Mainstream Bias

The final main recommendation list should normally contain:

- At least 70% Tier A or strong Tier B sources.
- At most 30% niche, ecosystem-specific, or low-publicity sources.
- No more than three watchlist/niche items unless the user explicitly asked for niche opportunities.

If fewer than five good mainstream candidates exist, say so and show the search coverage instead of padding the report.

## Scoring

Score each verified candidate out of 100:

| Dimension | Points | Guidance |
| --- | ---: | --- |
| Stack/domain fit | 25 | Direct overlap with languages, frameworks, domain, and project history |
| Project reuse | 20 | Existing GitHub/project work can be submitted or adapted quickly |
| Time feasibility | 15 | Deadline leaves enough time for the likely deliverable |
| Prize ROI | 15 | Reward relative to required effort and competitiveness |
| Source authority | 15 | Tier A/mainstream official source scores highest |
| Accessibility | 10 | Remote/global/clear eligibility and individual-friendly rules |

Penalties:

- `-40`: deadline already passed or status closed
- `-30`: eligibility likely excludes the user
- `-20`: official page unavailable or key fields unverified
- `-10`: obscure sponsor, unclear judging, or low trust
- `-10`: only weakly related to the user's profile

Labels:

- `High priority`: 80+
- `Recommended`: 65-79
- `Watchlist`: 50-64
- `Exclude`: below 50, unless included only to explain why it was rejected

## Report Output

When asked to produce a report, write `prize-hunt-report-YYYY-MM-DD.md` in the current working directory unless the user gives another path.

Use this structure:

```markdown
---
date: YYYY-MM-DD
skill: prize-hunt
input: "original user input"
---

# Prize Hunt Report - YYYY-MM-DD

## Profile Summary

Brief stack, domains, reusable projects, constraints, and assumptions.

## Search Transparency

| Source/Query | Method | Status | Valid Candidates |
| --- | --- | --- | ---: |

Mention failed sources and whether search/browse was unavailable.

## Top Recommendations

| Rank | Competition | Source | Prize | Deadline | Fit | Score |
| ---: | --- | --- | --- | --- | --- | ---: |

For each top item, include:

- Official link
- Why it fits
- What to submit or build
- First action in the next 24-48 hours
- Confidence and unverified fields, if any

## Watchlist

Include promising but less certain or more niche opportunities.

## Excluded Or Low-Fit Items

Briefly list notable items excluded and why, especially when they looked attractive at first.

## Submission Ideas

For top items without an obvious existing project fit, generate one or two concise ideas with MVP scope, stack, estimated effort, and risk.

## Link Index

List all official links used.
```

## Quality Bar

- Every main recommendation has a direct official link.
- Dates and prizes are verified or explicitly marked unknown.
- The report explains why each item is worth the user's time.
- Search coverage is transparent enough that the user can see whether missing results are due to lack of opportunities, access limitations, or filtering choices.
- The final list favors actionable, reputable opportunities over obscure long-tail contests.
