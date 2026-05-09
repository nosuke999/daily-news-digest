# news-sources.md (example)

> Copy this file to `news-sources.md` in the directory where you run the skill, and edit. Lines starting with `✅` are active sources; `❌` are skipped.

## Selection criteria (rewrite this for your domain)

I want a daily brief on **front-end web development with AI coding tools** (Claude Code, Cursor, Copilot, etc.).

**Adopt** items that:
- Teach a concrete technique (a new flag, hook, skill, prompt pattern, workflow)
- Come from official sources (Anthropic, Vercel, Vue/React core teams)
- Have strong community engagement (Reddit 50+ upvotes, HN front page)

**Skip** items that:
- Are introductory "I tried X" posts
- Are pure marketing / affiliate content
- Don't teach the reader how to do something better

## Tier 1 — Official sources

- ✅ https://www.anthropic.com/news (RSS not stable, scrape page)
- ✅ https://www.anthropic.com/engineering
- ✅ https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
- ❌ https://www.youtube.com/@anthropic-ai (covered via dynamic discovery)

## Tier 2 — Community

- ✅ https://www.reddit.com/r/ClaudeCode/top/?t=day
- ✅ https://hn.algolia.com/?dateRange=last24h&query=claude%20code

## Tier 3 — JP / regional (use only when Tier 1-2 is light)

- ❌ https://zenn.dev/topics/claude-code
- ❌ https://qiita.com/tags/claude-code

## Dynamic discovery

YouTube videos linked from Reddit/HN comments with **50+ upvotes or 10+ comments** are valid candidates even if they aren't listed above.

## Notes

- 7-day dedup window: items already featured in `digests/*.md` from the past 7 days are skipped.
- Output file: `digests/YYYY-MM-DD-digest.md` (created on first run).
- Update this file whenever your interests shift; the skill always reads the current version.
