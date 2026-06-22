# prize-hunt

全网搜寻可获奖金的竞赛/黑客松/Bounty，并匹配用户技术栈生成推荐报告。

## 包含文件

| 文件 | 说明 |
|------|------|
| `prize-hunt.md` | 主 skill：竞赛搜索、匹配评分、报告生成 |
| `github-profile.md` | 子 skill：分析 GitHub 用户技术栈画像 |

## 前置条件

- **Claude Code** 或支持 `$ARGUMENTS` 的 Claude skill 运行环境
- **gh CLI** — 已安装并认证（`gh auth login`）
- **WebSearch 工具** — 用于搜索竞赛信息

## 安装

将整个 `prize-hunt/` 文件夹复制到以下任一位置：

```
# 项目级（仅当前项目可用）
<your-project>/.claude/skills/prize-hunt/

# 用户级（所有项目可用）
~/.claude/skills/prize-hunt/
```

## 使用

```
/prize-hunt <github用户名>
/prize-hunt <技术栈描述，如 "Python ML 做过推荐系统">
/prize-hunt
```

- 传 GitHub 用户名 → 自动分析仓库构建技术栈画像
- 传技术栈描述 → 直接使用描述匹配竞赛
- 不传参数 → 交互式询问

## 输出

在当前目录生成 `prize-hunt-report-YYYY-MM-DD.md`，包含：
- 技术栈画像摘要
- 按匹配度排序的竞赛推荐（高优先/推荐/可关注）
- 参赛 idea（如需新建项目）
- 行动计划与汇总链接
