# GitHub Profile

Use this helper when Prize Hunt receives a GitHub username. The goal is not to judge the user; it is to extract a practical competition-matching profile.

## Input

The input should be a GitHub username. If the value contains spaces, Chinese text, URLs mixed with prose, or multiple names, ask a short clarification or extract the most likely username and state the assumption.

## Data Collection

Prefer `gh` when authenticated:

```bash
gh api users/{username}/repos --paginate --jq '.[] | {name, html_url, fork, archived, language, stargazers_count, topics, updated_at, description}'
```

If `gh` is unavailable, use GitHub's public REST API with `curl`:

```bash
curl -L "https://api.github.com/users/{username}/repos?per_page=100&sort=updated"
```

If both fail, report the failure and ask for a manual stack/project description.

## Repository Selection

Analyze up to 12 repositories:

- Prefer non-fork, non-archived repositories.
- Prioritize recent updates, stars, clear descriptions, and complete READMEs.
- Keep forks, tutorials, and toy repos only as weak signals.
- If the account has fewer than three useful repositories, mark confidence as low.

For each selected repository, collect:

- Language breakdown: `gh api repos/{username}/{repo}/languages`
- Topics, description, stars, last update, and repository URL
- README summary: `gh api repos/{username}/{repo}/readme -q '.content' | base64 -d`
- Dependency manifests when present:
  - `package.json`
  - `requirements.txt`
  - `pyproject.toml`
  - `go.mod`
  - `Cargo.toml`
  - `pom.xml`
  - `build.gradle`
  - `Gemfile`
  - `composer.json`
  - `Dockerfile`
  - `.github/workflows/*`

Use `gh api repos/{username}/{repo}/contents/{file}` or direct raw GitHub URLs. Ignore missing files.

## Profile Dimensions

Produce a concise structured profile:

- Primary languages: weight by repo count, stars, and recent activity.
- Frameworks and libraries: group as core, common, occasional.
- Domains: web frontend, web backend, full-stack, ML/AI, data science, data engineering, DevOps/cloud, Web3, security, game, mobile, IoT, developer tooling, or other.
- Reusable projects: projects that could be adapted for a competition with modest effort.
- Maturity: production-ready, prototype, learning/toy, or mixed.
- Constraints inferred from repos: likely solo/team fit, deployment experience, data/model experience, UI/product polish.
- Confidence: high, medium, or low.

## Output Format

Return this structure for Prize Hunt to consume:

```markdown
## GitHub Profile - {username}

### Summary
One paragraph describing the user's likely strongest competition angles.

### Primary Stack
- Languages:
- Frameworks/libraries:
- Tools/platforms:

### Domain Tags
- ...

### Reusable Projects
| Project | Evidence | Reuse Angle | Confidence |
| --- | --- | --- | --- |

### Maturity And Fit
- Maturity:
- Best contest types:
- Weaker contest types:
- Confidence:

### Source Repositories
| Repo | Stars | Updated | Signal |
| --- | ---: | --- | --- |
```

Keep the profile factual. Do not overstate expertise from a single small repository, and do not treat GitHub language percentages as a complete skill inventory.
