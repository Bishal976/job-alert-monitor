# Job Alert Monitor

An automated lead-monitoring CLI for freelance/contract frontend work. It polls Reddit
and Hacker News for relevant posts, filters out noise, scores the remaining leads with
an LLM, and emails you the strong matches with a ready-to-send outreach draft.

Hosted for free on GitHub Actions — no server to manage.

## How it works

1. **Sources**:
   - Hacker News — newest 20 stories (official Firebase API), every 30 minutes
   - Hacker News "Who is Hiring" thread — top 50 comments, checked once daily
   - Reddit — combined `.rss` feed across 13 subreddits (`SUBREDDITS` in
     `src/config.ts`), every 30 minutes

2. **Keyword match** — a post must mention at least one of the keywords in
   `KEYWORDS` (`src/config.ts`), e.g. React, Next.js, TypeScript, frontend, AI
   integration, or AI-coding-tool terms (cursor, claude code, vibe coded, etc.).

3. **Dedup** — seen post/comment IDs are tracked per-source in `seen_posts.json`
   (FIFO-pruned at 1000 entries, written once per run, committed back to the repo
   by the GitHub Actions workflow so it persists across runs).

4. **Intent pre-filter** (Reddit only, before any AI call) — rejects freelancer
   self-ads, showcase/weekly threads, and (on r/webdev) help-request posts with no
   buying intent. Keyed off the per-post subreddit, not the feed URL, since all 13
   subreddits are fetched as one combined feed.

5. **AI scoring + drafting** (Groq, `llama-3.3-70b-versatile`) — scores each
   remaining post 1-10 for fit against a hardcoded persona/profile, gated on real
   buying intent (not a showcase, not unpaid, not the author's own services). If
   the score is ≥ 6, it also drafts a 2-3 paragraph outreach reply.

6. **Per-cycle AI call cap** — at most 5 AI calls per cycle. Anything over the cap
   isn't dropped — it's collected into a single digest email for manual review.

7. **Email** (Gmail SMTP) — three possible email types per cycle:
   - `[LEAD x/10] {title}` — AI-scored match with reasoning + drafted reply
   - `[LEAD - NO AI] {title}` — AI call failed, but the lead wasn't dropped
   - `[LEADS DIGEST - REVIEW MANUALLY]` — leads skipped due to the AI call cap

Each source runs independently — if one fails (network error, feed down, etc.) the
others still run and the error is logged.

## Hosting (GitHub Actions)

Two scheduled workflows in `.github/workflows/` run the bot as a one-shot process
(no long-lived daemon, no node-cron):

- `main-cycle.yml` — Reddit + HN scan, every 30 minutes (`*/30 * * * *`)
- `hn-hiring.yml` — HN "Who is Hiring" thread check, daily at 10:00 AM IST (`30 4 * * *` UTC)

Both commit `seen_posts.json` back to `main` after each run so dedup state survives
between runs (commit message includes `[skip ci]`, so it doesn't re-trigger anything).

GitHub Actions minutes are unlimited on **public** repos — this repo is public for
that reason (no secrets are in the code; they live in repo Secrets).

Required repo secrets (Settings → Secrets and variables → Actions):

- `GMAIL_USER`, `GMAIL_APP_PASSWORD`, `GROQ_API_KEY`, `DEBUG_FILTERING`

You can trigger either workflow manually via the Actions tab or `gh workflow run main-cycle.yml`.

## Local development

```bash
npm install
cp .env.example .env   # fill in GMAIL_USER, GMAIL_APP_PASSWORD, GROQ_API_KEY
npx ts-node src/index.ts main       # one scan cycle (Reddit + HN)
npx ts-node src/index.ts hn-hiring  # one HN "Who is Hiring" check
```

`npm run build && npm start` compiles and runs the same way against `dist/`.
Startup exits with an error if `GMAIL_USER`, `GMAIL_APP_PASSWORD`, or `GROQ_API_KEY`
are missing.

## Configuration

- `KEYWORDS`, `SUBREDDITS`, `REDDIT_FEED_URL` — `src/config.ts`
- AI persona, gate-check rules, score threshold, budget guidance,
  `MAX_AI_CALLS_PER_CYCLE` — `src/ai-filter.ts`
- Reddit intent-filter patterns — `passesIntentFilter()` in `src/reddit.ts`
- Cron schedules — `.github/workflows/main-cycle.yml` and `.github/workflows/hn-hiring.yml`

## Dedup

Seen post/comment IDs are stored in `seen_posts.json`, tracked in git (not
gitignored) so the GitHub Actions workflows can persist it across runs. Delete its
contents (or the relevant source's array) to re-alert on everything currently in
the feeds.
