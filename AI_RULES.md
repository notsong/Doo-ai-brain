# AI Rules for Doo-ai-brain

> **Read this first.** These rules tell any AI agent how to use this knowledge base.

## About Me (Doo)

<!-- TODO: Fill in your background, primary tech stack, and long-term goals -->

- **Role**: AI/Computer Vision Engineer
- **Primary Languages**: Python
- **Primary Frameworks**: PyTorch, OpenCV, ONNX
- **Domains**: Computer Vision, Deep Learning, Model Deployment
- **Long-term Focus**: [To be filled]

## How to Read This Repository

### Mandatory Reading (Before Any Task)

1. `AI_RULES.md` (this file) — understand the rules
2. `profile/engineer_profile.md` — know my background and preferences
3. `status/current_status.md` — know what's currently active

### Task-Specific Reading

- Before working on a project → read `projects/<project>/overview.md`
- Before making architecture decisions → read `decisions/architecture_decisions.md`
- When encountering specific tech → check `knowledge/<technology>.md`

## What Should Be Recorded

Record information that has **long-term value across sessions**:

| Category | Examples |
|----------|----------|
| **Technical Decisions** | Why chose X over Y, architecture tradeoffs |
| **Project Experience** | Key findings, non-obvious pitfalls, solutions |
| **Experiment Results** | What was tested, results, conclusions |
| **Failed Approaches** | What didn't work and WHY (saves time later) |
| **Solutions** | How a difficult problem was solved |
| **Engineering Patterns** | Reusable patterns discovered during work |
| **Environment Setup** | Non-trivial config that took time to figure out |

## What Should NOT Be Recorded

Do NOT record:

- Routine code changes (variable renames, formatting)
- Temporary debugging sessions with no lasting insight
- Information already well-documented in the project's own repo
- Trivial or obvious facts
- Session logs or chat transcripts

## Golden Rule

> **"If I join a new project 6 months from now, what do I need to know?"**

If information helps answer that question, record it. If not, skip it.

## Syncing Protocol

### Claude Code: `/brain-sync`

1. Review the current session/task
2. Identify information with long-term value
3. Update or create relevant Markdown files
4. Update `sync/sync_log.md`
5. Run: `git add -A && git commit -m "sync: <summary>" && git push`

### Before Starting Work

```bash
cd <path-to>/Doo-ai-brain
git pull
```

Then instruct the AI agent to read relevant files.

## Hermes (Future)

Hermes is my long-term AI technical brain. It will:
- Read this repository for project context
- Contribute knowledge and experiment summaries
- Help design technical solutions based on accumulated experience

When Hermes is active, update `status/current_status.md` to reflect its role.
