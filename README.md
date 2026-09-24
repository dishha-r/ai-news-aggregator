# AI News Aggregator

A personal project to scrape AI-related news and content, generate summarized digests, 
and deliver them automatically — built step-by-step while following a YouTube tutorial 
(referencing datalumina/ai-news-aggregator for structure).

## Features (planned)
- [ ] Scrape RSS feeds and YouTube video transcripts for AI-related content
- [ ] Store articles/transcripts in a database
- [ ] Generate AI-powered summaries/digests using OpenAI
- [ ] Email digests automatically on a schedule

## Tech Stack
- Python 3.12+
- uv (package management)
- PostgreSQL (via SQLAlchemy)
- OpenAI API
- feedparser, BeautifulSoup, youtube-transcript-api

## Setup
```bash
uv sync
cp .env.example .env  # add your API keys and DB credentials
```

## Status
🚧 Work in progress — building along with a tutorial, one step at a time.