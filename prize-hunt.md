# Prize Hunt

User input: `$ARGUMENTS`

This is the legacy slash-command entrypoint. Follow `SKILL.md` in this same directory as the source of truth, and treat `$ARGUMENTS` as the user's input.

Required behavior:

1. Build a stack/profile from `$ARGUMENTS`.
   - If it looks like a GitHub username, read and follow `github-profile.md`.
   - Otherwise parse the text as a manual stack/project description.
2. Use current web search, browsing, or direct official HTTP/API fetches before recommending competitions.
3. Search mainstream official sources first: Devpost, Kaggle, Challenge.gov, MLH, Microsoft/Google official contests, Tianchi, DataFountain, Heywhale, and relevant Tier B platforms.
4. Verify every main recommendation on an official page.
5. Exclude obscure, unverifiable, closed, school-only, region-only, or no-prize opportunities from the main list.
6. Rank candidates with the scoring model in `SKILL.md`.
7. Generate `prize-hunt-report-YYYY-MM-DD.md` unless the user only asks for a short answer.

If live web search is unavailable, say so at the top of the report, try direct official source fetches, and keep unverified or evergreen items out of the main ranking.
