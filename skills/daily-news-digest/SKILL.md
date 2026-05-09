---
description: Generate a daily news digest in Markdown. Reads sources from news-sources.md in the current directory, selects 1-5 items per the user's selection criteria, deduplicates against the past 7 days of digests, and writes a dated digest file. Use when the user asks for "today's digest", "news brief", "daily curated news", or similar.
argument-hint: "[YYYY-MM-DD]"
allowed-tools: Read Write Glob Grep WebFetch
---

You are generating a **daily news digest**. Follow the rules below exactly. Do not invent facts; summarize only what each source actually says.

## Inputs

1. **Sources file**: `news-sources.md` in the current working directory.
   - The first section of this file contains the user's **selection criteria** (what to pick, what to skip).
   - Following sections list source URLs. Lines starting with `✅` are active; `❌` are skipped.
   - If `news-sources.md` is missing, instruct the user to copy `news-sources.example.md` to `news-sources.md` and edit. Then stop.

2. **Output directory**: `./digests/` relative to the current working directory. Create it if missing.

3. **Date**: today's date from `<env>`. If `$ARGUMENTS` is provided (format `YYYY-MM-DD`), use it as the run date.

## Procedure

1. **Read sources**: parse `news-sources.md` for selection criteria + active source URLs.
2. **Fetch in parallel**: WebFetch each active URL. Filter to entries published within the last 48 hours.
3. **Dynamic discovery (optional)**: if any source is a discussion forum (Reddit, HN, etc.) and its replies contain links to other primary sources (YouTube videos, blog posts, etc.) with strong engagement signals (e.g. 50+ upvotes, 10+ comments), add those links as candidates.
4. **Dedup**: read up to the past 7 days of `digests/*.md`. Exclude any URL that has already been featured.
5. **Select 1-5 items**: rank candidates by the user's selection criteria. Prefer fewer high-value items over a long list. If 0 items meet the bar, write a digest stating "no items today" rather than skipping the file.
6. **Summarize each item**: read the article content (don't summarize from the URL alone). Write 4-6 sentences in the user's language. Include: background → what's new/important → how to use it.
7. **Write the file**: `digests/<date>-digest.md` using the format in the next section. If the file already exists, append a new `## Update HH:MM` section instead of overwriting (one-file-per-day rule).
8. **Report briefly**: tell the user how many items were included and the output path. Do not paste the full digest into the chat unless asked.

## Output format

```markdown
---
date: YYYY-MM-DD
type: daily-digest
---

# Daily Digest - YYYY-MM-DD

## TL;DR
1. (item 1 headline, ≤50 chars)
2. (item 2 headline, ≤50 chars)
3. (item 3, optional)

## Items

### 1. (title in user's language)
- **Original title**: ... (only if translated)
- **Source**: Outlet name / YYYY-MM-DD
- **Summary**: 4-6 sentences.
- **Why it matters**: 1-2 sentences with a concrete take-away.
- **URL**: https://...

### 2. ...

## Failed sources
- (URL: short reason; or "none")

<!-- DAILY_DIGEST_DONE: <TL1> | <TL2> | <TL3> | <absolute_or_repo_url_to_this_file> -->
```

## Constraints

- **Never invent facts.** Summarize only what the source explicitly states.
- **Read articles before summarizing.** A summary built from the URL alone is unacceptable.
- **TL;DR lines must be short** — they will be forwarded to push notifications and SMS-style channels.
- The `<!-- DAILY_DIGEST_DONE: ... -->` marker is **machine-parsed** by scheduler scripts. Do not remove, reformat, or change the field separator (` | `).
- Respect robots.txt. Avoid hitting the same domain in tight bursts.
- If a source returns an error, list it under `## Failed sources` and continue.

## Pairing with a scheduler

This skill produces a Markdown file only. To deliver the digest to Discord / Slack / email, wrap the skill in a scheduler that invokes Claude Code headlessly and parses the `DAILY_DIGEST_DONE` marker:

```bash
claude --print --dangerously-skip-permissions "/daily-news-digest:daily-news-digest"
# → digests/YYYY-MM-DD-digest.md is written.
# → Your wrapper grep's DAILY_DIGEST_DONE and posts the TL;DR via webhook.
```

A complete reference wrapper (Discord webhook, git push, failure counter, Windows Task Scheduler registration) lives in the plugin's `README.md`.
