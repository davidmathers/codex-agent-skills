# Grill with Docs vs Claude

`grill-with-docs-vs-claude` is a Codex skill for hardening implementation plans before code is written. Codex runs the documentation-aware planning interview, updates `GLOSSARY.md` and ADRs as decisions crystallize, writes the locked plan to `PLAN.md`, then asks Claude Code CLI to adversarially review that plan in a bounded read-only loop.

This skill differs from Matt Pocock's original in that his `CONTEXT.md` is renamed here to `GLOSSARY.md`.

The skill requires Claude Code CLI to be installed, authenticated, and available as `claude` on `PATH`, because Act 2 shells out to Claude for the cross-model review. Remember to set Claude's default model to your preference before using.
