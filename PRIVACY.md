# Privacy Policy

This is a personal automation tool operated solely by Bishal Kumar for his own
lead-generation workflow. It is not a public-facing service, has no end
users, and does not collect personal data from anyone other than the
operator.

## What this tool processes

- **Public post content** — titles and text of publicly visible posts from
  Reddit and Hacker News.
- **Third-party AI processing** — matched post content is sent to
  [Groq](https://groq.com)'s LLM API for scoring and drafting an outreach
  reply. See Groq's own privacy policy for how they handle data sent to
  their API.
- **Dedup state** — only post/comment IDs (e.g. `t3_xxxxx`, numeric HN IDs)
  are retained in `seen_posts.json`, solely to avoid re-processing the same
  post twice. No post content, author names, or other metadata is stored.
- **Email delivery** — matched leads are emailed to the operator's own
  Gmail account via Gmail SMTP, using a Gmail App Password. No data is sold,
  shared with third parties, or used for any purpose beyond generating these
  alerts.

## Data retention

`seen_posts.json` retains up to ~1000 IDs per source, FIFO-pruned (oldest 200
dropped once the cap is exceeded). That's the only persistent state this tool
keeps.

## Scope

This document describes how this specific tool handles data today. It is not
legal advice and is not a substitute for a formal privacy policy should this
tool ever be turned into a service with external users.

## Contact

Questions about this tool's data handling: singhbishalkumarsingh@gmail.com
