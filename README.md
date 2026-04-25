# SilverWorker Wiki

This repository contains the planning documents, specifications, session history, and PR reviews for the **SilverWorker** project.

It is deployed as a GitHub Pages site for human-readable access.

## Structure

| Directory | Contents |
|-----------|----------|
| `planning/` | MVP specs (spec_01~10), tech stack, DB schema, implementation plan |
| `history/` | Session history, setup logs, review fixes |
| `PR_Review/` | Claude & Gemini reviewer feedback per PR |
| `PROGRESS.md` | Current development status dashboard |
| `AGENTS.md` | Master orchestration document for AI agents |

## For Collaborators

This repo is **separate** from the code repository (`SilverWorkerNow/`). If you are setting up the project:

```bash
# Clone the code repo
git clone https://github.com/dudxo13/SilverWorkerNow.git

# Clone this wiki repo into the General/ directory
git clone https://github.com/dudxo13/SilverWorker-Wiki.git General/
```

AI agents (Claude Code, OpenCode, Gemini CLI) use the `AGENTS.md` and `PROGRESS.md` in this directory for context.

## Access the Wiki

**URL**: https://dudxo13.github.io/SilverWorker-Wiki/

The wiki is auto-deployed on every push to `master` or `main`.

## What's NOT Here

- **Source code**: In `SilverWorkerNow/` repo
- **Graphify knowledge graph**: AI-only, stays in the parent `SilverWorker/` directory
- **CI scripts / Flutter config**: In `SilverWorkerNow/` repo (see `scripts/` and `config/` there)

---

*Last updated: 2026-04-25*
