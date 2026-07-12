# prize-hunt

`prize-hunt` helps find current prize competitions, hackathons, AI/data challenges, grants, and open source bounties, then ranks them against a GitHub profile or manual tech-stack description.

## Files

| File | Purpose |
| --- | --- |
| `SKILL.md` | Codex skill entrypoint and source of truth |
| `prize-hunt.md` | Legacy slash-command wrapper for environments that pass `$ARGUMENTS` |
| `github-profile.md` | Helper instructions for GitHub username analysis |
| `agents/openai.yaml` | Optional UI metadata for Codex skill lists |

## Improvements In This Version

- Adds a standard `SKILL.md` so Codex can discover and invoke the skill.
- Makes live web search, browsing, or official HTTP/API fetching mandatory for current contest data.
- Prioritizes mainstream official sources such as Devpost, Kaggle, Challenge.gov, MLH, Tianchi, DataFountain, and Heywhale.
- Pushes obscure, unverifiable, closed, or weak-prize contests into watchlist/excluded sections instead of the main recommendations.
- Adds transparent source coverage, confidence labels, official-link verification, and a 100-point scoring model.

## Usage

```text
Use $prize-hunt to find current competitions for my GitHub profile: <username>
Use $prize-hunt to find prize competitions for "Python, FastAPI, React, ML recommendation systems"
```

The default report filename is `prize-hunt-report-YYYY-MM-DD.md`.
