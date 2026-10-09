# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **steamworks-api-simplified** — a documentation-only repository that serves as a comprehensive, human-friendly guide to the Steam Web API. Created by [yilmaz.games](https://yilmaz.games), it's the "Steam API guide we wish existed." There is no application code, build system, or test suite.

The repo also contains `steamworks-api-specialist/SKILL.md`, a condensed AI-agent skill file derived from the main guide.

## Repository Structure

- `README.md` — The main guide (English). This is the primary artifact of the project.
- `translations/README.{lang}.md` — Translated versions (currently Turkish and German)
- `steamworks-api-specialist/SKILL.md` — AI-agent-consumable version of the guide
- `.github/CONTRIBUTING.md` — Contribution guidelines
- `.github/ISSUE_TEMPLATE/` — Issue templates (bug report, improvement, translation)
- `LICENSE` — CC0 1.0 (public domain)

## Editing Conventions

- **Follow the existing format** when adding or modifying endpoint documentation: endpoint URL, parameters table, response example JSON, then gotchas/notes.
- **Keep code blocks, URLs, API endpoints, and JSON examples in English** across all translations. Only translate prose, headers, and table headers.
- Translations use ISO 639-1 codes: `translations/README.{code}.md` (e.g., `README.fr.md`).
- When updating the English README, corresponding sections in translations may need updating too.
- The SKILL.md file should stay in sync with README.md content — if endpoints or gotchas change in the README, reflect those changes in SKILL.md.
- SKILL.md is also installed as a Claude Code skill from the maintainers' private `yilmaz-games/skills` repo, cloned at `~/.claude/skills`. After SKILL.md changes here, copy it to `~/.claude/skills/steamworks-api-specialist/SKILL.md` through a branch and PR in that repo, and check whether the older `steam-api` skill there needs the same change.

## Content Rules

- One change per PR.
- All example endpoints and code samples must be tested/working.
- No promotional content beyond the existing okey.gg/yilmaz.games references.
- The guide targets indie game developers who may be new to web APIs — keep explanations practical, not encyclopedic.
