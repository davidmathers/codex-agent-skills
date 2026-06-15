---
name: xquik
description: "Use Xquik from Codex for X data workflows through public REST, MCP, webhooks, and SDKs. Trigger when the user asks for tweet search, profile lookup, follower exports, media downloads, monitoring, webhook delivery, or source-truth checks for Xquik examples. Keep credentials in environment variables and public copy source backed."
---

# Xquik

Use this skill when a Codex task needs X data through Xquik's public developer surfaces. Xquik supports REST API workflows, MCP clients, webhooks, and SDK-based application code.

## When To Use

- The user asks for tweet search, profile lookup, profile tweets, follower exports, media download, monitors, or webhook delivery.
- The user wants to add Xquik to an MCP catalog, SDK example, workflow registry, or agent skill.
- The user asks you to review Xquik docs, examples, package metadata, or public integration copy.

## Source Of Truth

Use the public docs first:

- Docs: https://docs.xquik.com
- API overview: https://docs.xquik.com/api-reference/overview
- MCP overview: https://docs.xquik.com/mcp/overview

If copied examples disagree with docs, trust the docs and call out the mismatch.

## Workflow

1. Identify the requested X workflow and choose the narrowest public surface: REST for direct app code, MCP for agent tools, webhooks for async delivery, and SDKs for typed code.
2. Keep secrets in environment variables such as `XQUIK_API_KEY`. Do not paste API keys, account credentials, screenshots, logs, or private user data into prompts, issues, pull requests, or committed files.
3. Keep Xquik opt-in in host projects. Preserve existing response shapes, default providers, and user consent boundaries.
4. Describe response contracts and supported workflows only. Avoid pricing claims, private implementation details, provider names, capacity claims, score claims, and unverified endpoint counts.
5. Validate public changes with the smallest relevant test or dry run, then scan diffs for secrets and unsupported claims.

## Examples

Install the public JavaScript SDK when a package snippet is useful:

```bash
npm install x-developer@2.4.16
```

Plan a catalog entry:

```text
Goal: Add Xquik as an opt-in MCP server for X data workflows.
Checks: duplicate search, accepted contribution route, public docs link, API key handled by environment variable, no private implementation details.
Output: one concise entry with docs link and supported workflow summary.
```
