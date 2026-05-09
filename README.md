# daily-news-digest

A Claude Code plugin that generates a daily news digest in Markdown — curated from sources you define, deduplicated against the past 7 days, in any language. Pair it with a scheduler (cron / Windows Task Scheduler) for hands-off operation.

> **Status**: 0.1.0 (early). Tested on Windows + PowerShell. Bug reports and PRs welcome.

## What it does

- Reads source URLs and selection criteria from `news-sources.md` (you edit this).
- Fetches each source, picks 1–5 items per day per **your criteria**, summarizes each in 4–6 sentences.
- Deduplicates against the last 7 days of digests.
- Writes `digests/YYYY-MM-DD-digest.md`.
- Embeds a `<!-- DAILY_DIGEST_DONE: ... -->` marker for scheduler scripts to parse.

## Install

```text
/plugin marketplace add nosuke999/daily-news-digest
/plugin install daily-news-digest@nosuke999/daily-news-digest
```

(Substitute the GitHub user/repo with whichever fork you prefer.)

## Quick start

```bash
# 1. Configure your sources
cp news-sources.example.md news-sources.md
# edit news-sources.md to list your sources and selection criteria

# 2. Run once interactively
claude
# then:
/daily-news-digest:daily-news-digest

# 3. Check the output
ls digests/
```

The skill writes `digests/YYYY-MM-DD-digest.md`. If you run it twice on the same day, the second invocation appends a `## Update HH:MM` section instead of overwriting.

## Scheduler integration (Discord webhook + git push)

The skill itself just produces Markdown. To get push notifications and source-controlled history, wrap it in a small script:

### Windows Task Scheduler (PowerShell)

`run.ps1`:

```powershell
$ErrorActionPreference = "Stop"
$ProjectRoot = $PSScriptRoot
Set-Location $ProjectRoot

$today = Get-Date -Format "yyyy-MM-dd"
$logDir = Join-Path $PSScriptRoot "logs"
if (-not (Test-Path $logDir)) { New-Item -ItemType Directory -Path $logDir | Out-Null }

# Load .env
Get-Content (Join-Path $PSScriptRoot ".env") | ForEach-Object {
    if ($_ -match '^\s*([^#=]+)=(.*)$') {
        [Environment]::SetEnvironmentVariable($Matches[1].Trim(), $Matches[2].Trim(), "Process")
    }
}

# Run skill headlessly
& claude --print --dangerously-skip-permissions "/daily-news-digest:daily-news-digest" 2>&1 |
    Out-File (Join-Path $logDir "$today.log") -Encoding utf8

# Parse marker
$digest = Get-Content (Join-Path $ProjectRoot "digests/$today-digest.md") -Raw -Encoding utf8
if ($digest -match '<!--\s*DAILY_DIGEST_DONE:\s*(.+?)\s*\|\s*(.+?)\s*\|\s*(.+?)\s*\|\s*(\S+)\s*-->') {
    $body = "**Daily digest ($today)**`n1. $($Matches[1])`n2. $($Matches[2])`n3. $($Matches[3])`n`n$($Matches[4])"
    $payload = @{ embeds = @(@{ title = "Daily digest"; description = $body }) } | ConvertTo-Json -Depth 5
    Invoke-RestMethod -Uri $env:DISCORD_WEBHOOK_URL -Method Post -ContentType "application/json; charset=utf-8" -Body $payload | Out-Null
}

# Optional: commit to git
& git add "digests/$today-digest.md" 2>&1 | Out-Null
& git commit -m "feat: digest $today" 2>&1 | Out-Null
& git push 2>&1 | Out-Null
```

Register with `schtasks`:

```powershell
schtasks /Create /TN "Daily News Digest" /TR "pwsh -NoProfile -ExecutionPolicy Bypass -File `"$PWD\run.ps1`"" /SC DAILY /ST 07:00 /F
```

### cron (macOS / Linux)

```bash
0 7 * * * cd ~/daily-news-digest && claude --print --dangerously-skip-permissions "/daily-news-digest:daily-news-digest" > logs/$(date +\%F).log 2>&1
```

Add a post-run script that parses `DAILY_DIGEST_DONE` and posts to your channel of choice.

## Configuration

`news-sources.md` is the **single source of configuration**. It contains:

1. **Selection criteria**: a paragraph in your own words describing what to adopt and what to skip. The skill uses this verbatim — make it specific.
2. **Source list**: URLs marked `✅` (active) or `❌` (skipped).
3. **Optional dynamic discovery rules**: e.g. "YouTube videos with 50+ upvotes from Reddit comments are valid candidates."

See `news-sources.example.md` for a starter for AI-coding-tools coverage.

## Output format

Each digest file is plain Markdown with YAML frontmatter, a TL;DR, and 1–5 items. The trailing `<!-- DAILY_DIGEST_DONE: TL1 | TL2 | TL3 | URL -->` line is parsed by scheduler wrappers. Do not edit it manually.

## What this plugin doesn't do

- **No webhook delivery** — pair with a scheduler script (above) or invoke from any orchestrator.
- **No persistent storage** — each run reads `digests/` for the dedup window; that's it.
- **No PII filtering** — your selection criteria controls what gets summarized.

## Roadmap

- [ ] 0.2.0: built-in webhook plugins (Discord / Slack) so the skill can deliver natively without a wrapper script.
- [ ] 0.3.0: multi-language fallback in summarization (output language detection from `news-sources.md`).
- [ ] 0.4.0: pluggable selection scorer (LLM-judge with persisted preference data).

Open issues on GitHub for feature requests.

## License

MIT — see [LICENSE](./LICENSE). Copyright © 2026 subhive.
