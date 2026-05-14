# Blog: Thinking with Claude

A personal blog documenting the journey of building software with AI tools — what works,
what breaks, and the meta-layer of coordination that makes it sustainable.

## Repository layout

```
drafts/       gitignored — raw voice transcriptions and sensitive input, NEVER commit
_drafts/      Jekyll drafts — gitignored, local-only working copies, NEVER commit
_drafts_backup/  gitignored, NEVER commit
_context/     gitignored — research notes, chat logs, sensitive background, NEVER commit
_posts/       published posts — the ONLY content that goes to GitHub
```

## Git rules — read before touching git

This is a **public repository**. Only `_posts/`, `_config.yml`, `Gemfile`, `index.md`,
`CLAUDE.md`, `docker-compose.yml`, and `.gitignore` should ever be committed.

**Never stage or commit:**
- Anything in `drafts/`, `_drafts/`, `_drafts_backup/`, `_context/`
- `cleanup-response.md`, `session-handoff.md`, `pdf-preview.sh`, `.codex`, `.env`
- Any file containing real names, internal project names, company names, or private URLs

Before every commit: run `git status` and `git diff --cached`. If anything outside
`_posts/` or the site config files is staged, stop and ask before proceeding.

## Generating a post from a transcript

When given a voice transcript or rough notes from `drafts/`:

**Sanitization first — before anything else:**
- Strip all personal names (colleagues, clients, anyone mentioned in passing)
- Strip internal project names, company names, and product names unless they are publicly
  known and essential to the point
- Strip any credentials, file paths to private systems, URLs to private resources
- Strip off-topic tangents that clearly aren't meant for publication
- Flag anything ambiguous — ask rather than guess

**Then extract:**
- The core insight or realization the post is trying to convey
- Specific moments, decisions, or turning points (these are the emotional core)
- Concrete details: dates, commit messages, verbatim prompts, tool output — these make
  the posts credible and specific. Keep them if not sensitive.
- The "before" and "after" — what changed and why it mattered

**Post structure (follow the existing _drafts/ posts as the model):**
- Lead with the problem or situation, not the conclusion
- Use section headers to mark stages of the narrative
- Keep sections short — 2–4 paragraphs max
- The final section earns the insight; don't state it up front
- No bullet-point summaries at the end. Let the narrative close.

## Voice and tone

- First person, direct, no hedging
- Technical but not jargon-heavy — assume the reader codes but isn't expert in this stack
- Honest about failures and wrong turns; that's the point of the blog
- No "in conclusion" or "in summary" — just end when the thought is complete
- Short sentences preferred over long ones

## Context files

Each post in `_drafts/` or `_posts/` may have a corresponding `_context/` file with
raw research, session logs, commit references, and background. Read the context file
before drafting or editing a post — it contains the specific evidence the post draws on.

## Front matter

Every post in `_posts/` needs:
```yaml
---
title: "Title Here"
date: YYYY-MM-DD
---
```

`_drafts/` posts don't need a date in the front matter (Jekyll ignores it anyway).

## Moving a post to publish

When asked to publish a draft:
1. Move from `_drafts/FILENAME.md` to `_posts/YYYY-MM-DD-FILENAME.md`
2. Add front matter with the correct date
3. Do a final read for any remaining sensitive content before confirming
