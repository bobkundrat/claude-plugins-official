# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Claude Code Plugins Official Marketplace** — a curated directory of plugins and extensions for Claude Code. It contains both internal Anthropic-developed plugins (`/plugins`) and third-party partner plugins (`/external_plugins`). Plugins are configuration-driven (Markdown with YAML frontmatter + JSON manifests), not compiled code.

## Commands

### Validate frontmatter (CI check)
```bash
cd .github/scripts && bun install yaml && bun validate-frontmatter.ts
```

Validate specific files:
```bash
bun .github/scripts/validate-frontmatter.ts path/to/file.md
```

Validation runs automatically in CI on PRs that touch `**/agents/*.md`, `**/skills/*/SKILL.md`, or `**/commands/*.md`.

### Runtime
This project uses **Bun** (not Node.js) for all TypeScript execution.

## Architecture

### Plugin Structure

Every plugin follows this layout:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json      # Required: { name, description, author }
├── .mcp.json            # Optional: MCP server configuration
├── commands/            # Slash commands (user-invoked via /command-name)
│   └── *.md             # Frontmatter requires: description
├── agents/              # Agent definitions (AI-invoked autonomously)
│   └── *.md             # Frontmatter requires: name, description
├── skills/              # Skill definitions (model-invoked contextual guidance)
│   └── skill-name/
│       └── SKILL.md     # Frontmatter requires: description (or when_to_use)
└── README.md
```

### Extensibility Types

- **Commands** — User-triggered via slash commands. Frontmatter fields: `description`, `argument-hint`, `allowed-tools`, `model`.
- **Agents** — Autonomously invoked by the model. Frontmatter fields: `name`, `description`, `model`, `color`.
- **Skills** — Contextual guidance the model incorporates. Frontmatter fields: `name`, `description`/`when_to_use`, `version`.
- **MCP Servers** — External service integrations configured in `.mcp.json` with `type` (http/stdio/sse), `url`, and optional `headers`.

### Master Catalog

`.claude-plugin/marketplace.json` is the master registry of all plugins (50+). It defines plugin metadata, LSP server configurations, MCP server definitions, and categories. This file drives plugin discovery and installation in Claude Code.

### Frontmatter Validation

`.github/scripts/validate-frontmatter.ts` parses YAML frontmatter and enforces required fields per file type. It includes special handling for YAML special characters (glob patterns like `**/*.{ts,tsx}`) by auto-quoting values containing `{}[]*&#!|>%@` before parsing.

### Contribution Model

- Internal plugins (`/plugins`): Anthropic team members only.
- External plugins (`/external_plugins`): Submitted via the [plugin directory submission form](https://clau.de/plugin-directory-submission). A GitHub Actions workflow (`close-external-prs.yml`) automatically closes PRs from non-team members.
- Reference implementation: `plugins/example-plugin/`.
