# AI News Aggregator

A personalized daily AI news digest. It scrapes YouTube channels and the OpenAI and Anthropic blogs, summarizes each item with an LLM, ranks everything against a user profile, and emails the top picks as a formatted HTML digest.

<!-- Add a screenshot of a real digest email here -->
<!-- ![Sample digest email](docs/sample-email.png) -->

## What it does

- **Scrapes** new videos (with transcripts) from YouTube channels via RSS, plus OpenAI and Anthropic news, research, and engineering posts
- **Converts** full article pages to clean markdown using Docling
- **Summarizes** every item into a short title and summary using an LLM
- **Curates** the summaries against a configurable profile (interests, expertise level, preferences) and scores each one from 0 to 10
- **Emails** the top N as an HTML digest with a short personalized intro
- **Runs end to end** with one command

## Architecture

```mermaid
flowchart LR
    A[YouTube RSS] --> S[Scrapers]
    B[OpenAI RSS] --> S
    C[Anthropic RSS] --> S
    S --> DB[(Postgres)]
    DB --> P[Backfill: transcripts and markdown]
    P --> D[Digest agent]
    D --> DB
    DB --> K[Curator agent]
    U[User profile] --> K
    K --> E[Email agent]
    E --> M[HTML email via Gmail SMTP]
```

## Tech stack

| Area | Tools |
| --- | --- |
| Language | Python 3.13, managed with `uv` |
| Scraping | feedparser, youtube-transcript-api, Docling, BeautifulSoup |
| Storage | PostgreSQL (Neon), SQLAlchemy |
| LLM | Groq (`openai/gpt-oss-120b`) through the OpenAI-compatible SDK |
| Validation | Pydantic (typed scraper models and structured LLM output) |
| Email | Gmail SMTP, Markdown to HTML |

## Project structure

```
app/
  agent/       digest, curator, and email agents
  database/    SQLAlchemy models, connection, repository
  profiles/    user profile that drives curation
  scrapers/    YouTube, OpenAI, Anthropic
  services/    backfill jobs, digest/curation/email orchestration, SMTP
  runner.py        scrape and persist
  daily_runner.py  full pipeline
main.py            CLI entry point
```

## Getting started

**Prerequisites:** Python 3.13+, [uv](https://docs.astral.sh/uv/), a Postgres database (a free Neon project works), a free [Groq](https://console.groq.com) API key, and a Gmail account with an [app password](https://myaccount.google.com/apppasswords).

```bash
git clone https://github.com/dishha-r/ai-news-aggregator.git
cd ai-news-aggregator
uv sync
```

Create a `.env` file:

```
DATABASE_URL=postgresql://user:password@host/dbname?sslmode=require
GROQ_API_KEY=your_groq_key
MY_EMAIL=you@gmail.com
APP_PASSWORD=your_gmail_app_password

# Optional: proxy for YouTube transcript requests
PROXY_USERNAME=
PROXY_PASSWORD=
```

Create the tables, then run the pipeline:

```bash
uv run python app/database/create_tables.py
uv run python main.py              # last 24 hours, top 10
uv run python main.py 168 5        # last 7 days, top 5
```

To personalize the ranking, edit `app/profiles/user_profile.py`. To change which YouTube channels are tracked, edit `app/config.py`.

## Design decisions

- **Provider-agnostic LLM layer.** The agents talk to an OpenAI-compatible endpoint, currently served by Groq's free tier. Changing providers means changing the base URL, API key, and model name, with no changes to the agent logic.
- **Batched curation.** Ranking all digests in one structured-output call failed once the database reached about 50 items, because the model could not return that much valid JSON in one response. The curator now ranks in batches of 15, then merges and re-ranks by score.
- **Idempotent ingestion.** Videos and articles use their natural IDs (video ID, GUID) as primary keys, and the repository skips anything already stored, so the pipeline can be re-run safely.
- **No endless retries.** Videos whose transcripts are unavailable are marked with a sentinel value so they are not fetched again on every run.
- **Repository pattern.** All database access goes through one class, which keeps the agents and scrapers free of SQL.
- **Typed boundaries.** Scrapers return Pydantic models, and LLM output is validated against Pydantic schemas before it reaches the database.

## Roadmap

- [ ] Scheduled daily runs in the cloud
- [ ] More sources (research blogs, newsletters)
- [ ] Tests for scrapers and repository methods
- [ ] Feedback loop: learn from which digest links get clicked

## Data sources

OpenAI news comes from its official RSS feed. Anthropic does not publish official feeds, so the project uses the community-maintained [Olshansk/rss-feeds](https://github.com/Olshansk/rss-feeds).