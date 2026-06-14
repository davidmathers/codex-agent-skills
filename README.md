# Codex Agent Skills

Agent Skills specifically for OpenAI Codex

## Skills

- [`grill-with-docs-vs-claude`](./skills/grill-with-docs-vs-claude): a Codex planning skill that uses Claude Code CLI for adversarial review.

## Install

List available skills from this repo:

```bash
npx skills add davidmathers/codex-agent-skills --list
```

Install Grill with Docs vs Claude into Codex:

```bash
npx skills add davidmathers/codex-agent-skills --skill grill-with-docs-vs-claude --agent codex
```

## Requirements

`grill-with-docs-vs-claude` requires Claude Code CLI to be installed, authenticated, and available as `claude` on `PATH`, because Act 2 shells out to Claude for cross-model plan review.
