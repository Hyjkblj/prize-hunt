# GitHub Profile — 分析 GitHub 用户的技术栈画像

用户输入: $ARGUMENTS

---

## 你的任务

根据 GitHub 用户名，分析其公开仓库，输出一份技术栈画像。

**输入规则:**
- `$ARGUMENTS` 应为 GitHub 用户名（纯英文、无空格）
- 如果为空，询问用户提供
- 如果包含多个用户名，逐一分析并对比

---

## Step 1: 获取仓库列表

```bash
gh api users/{username}/repos --paginate -q '.[] | {name, language, stargazers_count, topics, updated_at, description}' 2>&1
```

如果失败（用户不存在、API 限流等），提示用户检查用户名或稍后重试。

按 `stargazers_count` 降序排列，取 top 10 作为重点分析对象。

---

## Step 2: 深入分析重点仓库

对 top 10 中的每个仓库：

### 2.1 语言分布
```bash
gh api repos/{username}/{repo}/languages
```

### 2.2 依赖文件（按优先级尝试，存在即读取）
```
package.json → Node.js/前端
requirements.txt / pyproject.toml → Python
go.mod → Go
Cargo.toml → Rust
pom.xml / build.gradle → Java/Kotlin
Gemfile → Ruby
composer.json → PHP
```

通过 `gh api repos/{username}/{repo}/contents/{file} -q '.content' | base64 -d` 读取。

### 2.3 README 摘要
```bash
gh api repos/{username}/{repo}/readme -q '.content' | base64 -d | head -100
```

提取项目用途、技术亮点。

---

## Step 3: 构建画像

综合所有数据，生成以下维度的分析：

### 主力语言
按以下权重排序：
- 出现频率（多少个 repo 使用）× 0.4
- 该语言 repo 的总 star 数 × 0.3
- 最近活跃度（近 6 个月有更新）× 0.3

输出 top 3-5 语言及占比估算。

### 框架/库
从依赖文件中提取，按出现频次排序。区分：
- **核心依赖**（出现在 ≥ 3 个 repo）
- **常用工具**（出现在 2 个 repo）
- **偶尔使用**（仅 1 个 repo）

### 领域标签
基于项目描述、README、Topics 综合判断，从以下标签中选择（可多选）：
- Web 前端 / Web 后端 / 全栈
- 移动开发（iOS/Android/跨平台）
- ML/AI / 数据科学 / 数据工程
- DevOps / 基础设施 / 云原生
- 区块链 / Web3
- 游戏开发
- 工具链 / CLI / 开发者工具
- 安全 / 密码学
- 嵌入式 / IoT
- 其他（注明）

### 项目成熟度评估
- **生产级**：有 CI/CD、完善文档、活跃维护、star ≥ 50
- **半成品**：有基本功能但文档不全或维护不活跃
- **实验/学习**：fork 练习、课程项目、toy repo

给出整体成熟度评级（初级/中级/高级）。

---

## 输出格式

向用户展示以下画像摘要：

```
## 📊 GitHub 技术栈画像 — {username}

### 🗣️ 主力语言
1. Python (60%) — 6 个 repo，总 star 120
2. TypeScript (25%) — 3 个 repo，总 star 45
3. Go (15%) — 1 个 repo，star 12

### 📦 核心框架/库
- **核心**: React, FastAPI, Pandas
- **常用**: Docker, PostgreSQL, Tailwind
- **偶尔**: Redis, gRPC

### 🏷️ 领域标签
Web 后端, ML/AI, 工具链

### 📈 成熟度
中级 — 有 2 个生产级项目，其余为半成品/实验

### 🔗 重点项目
| 项目 | Stars | 语言 | 用途 |
|------|-------|------|------|
| [repo-a](https://github.com/{username}/repo-a) | 80 | Python | ML 推理服务 |
| [repo-b](https://github.com/{username}/repo-b) | 45 | TypeScript | 管理后台 |
```

---

## 注意事项

- 如果 top 10 中有明显的 fork/课程项目，在画像中标注但降低权重
- 如果仓库总数 < 3，提醒用户数据有限，画像可能不够准确
- 语言分布仅基于 GitHub Linguist 统计，不等于实际能力水平
- 画像用于后续匹配，保持客观描述，不加主观评价
