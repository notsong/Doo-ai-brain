# Doo-ai-brain

**Doo's Personal AI Engineering Brain**

A long-term technical knowledge base designed to be read by both humans and AI agents.

## Purpose

This repository serves as my external AI brain — a persistent, version-controlled knowledge base that any AI agent (Claude Code, Hermes, ChatGPT, etc.) can read to understand my projects, technical decisions, experiments, and engineering patterns.

## Repository Structure

```
Doo-ai-brain/
├── README.md                 # This file
├── AI_RULES.md               # Rules for AI agents reading this repo
├── profile/                  # My engineering profile and working style
├── status/                   # Current status of all active projects
├── projects/                 # Per-project knowledge and experience
├── knowledge/                # Technology-specific knowledge
├── experiments/              # Standalone experiment records
├── decisions/                # Architecture decision records
└── sync/                    # Sync logs
```

## How AI Agents Use This

1. **On startup**: Read `AI_RULES.md` first, then `status/current_status.md`
2. **Before working on a project**: Read `projects/<project>/overview.md` and related docs
3. **When stuck**: Search `knowledge/` for relevant technical patterns
4. **After important work**: Update relevant files and sync back

## How I Use This

1. Complete significant task
2. Run `/brain-sync` in Claude Code
3. Agent reviews session, identifies long-term value
4. Agent updates relevant Markdown files
5. Git commit + push

## Principles

- **Tool-agnostic**: Pure Markdown, readable by any AI or human
- **Signal over noise**: Record decisions, experiments, failures — not trivial edits
- **Version controlled**: Full history of how my thinking evolves
- **Bidirectional**: Both desktop and laptop are contributors
