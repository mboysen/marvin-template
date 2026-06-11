# Changelog

All notable changes to marvin-template are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/).

## [1.1.0] - 2026-04-18

### Added
- **Software engineering with gstack** — MARVIN now routes all software
  development work (build, fix, debug, review, ship) through gstack's
  specialized workflow skills. Auto-installs on first use.
- **`/new-project` command** — Scaffold new software projects with gstack
  pre-configured. Supports Next.js, React, Python, Go, and Node.js.
- **Versioning and changelog** — MARVIN template now uses semantic versioning.
  `/sync` shows what version you're on and what's new.

## [1.0.0] - 2026-04-18

Initial stable release. Everything that shipped across releases 1-6:

### Added
- Session management (`/start`, `/end`, `/update`)
- Weekly reports (`/report`)
- Git workflow (`/commit`)
- Skill discovery and installation (`/skills`)
- Template sync (`/sync`)
- Content agent, events agent, research agent
- Integration framework (Google Workspace, Slack, Notion, Linear,
  Atlassian, MS 365, Telegram, Parallel Search)
- Team mode with shared context
- Architectural patterns library (daemon, mobile access, parallel
  worktrees, content pipeline, meeting processing)
- Onboarding wizard
