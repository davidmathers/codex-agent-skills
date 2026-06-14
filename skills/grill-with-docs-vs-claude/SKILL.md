---
name: grill-with-docs-vs-claude
description: "Two-act plan hardening with living documentation for Codex. ACT 1: Codex interviews the user about a plan, challenges it against GLOSSARY.md and ADRs, sharpens fuzzy terms, stress-tests scenarios, cross-references code, and updates GLOSSARY.md plus ADRs inline. ACT 2: Claude Code CLI adversarially reviews PLAN.md in a read-only plan-mode session with VERDICT: APPROVED/REVISE; Codex revises and re-submits to the same Claude session until approval or MAX_ROUNDS, then asks for user sign-off before implementation. Use when the user says /grill-with-docs-vs-claude, asks to grill a plan against docs then have Claude review, or is about to build something high-stakes in a project with established terminology/ADRs and wants alignment, documentation, and cross-model sanity checking. Builds on Matt Pocock's grill-with-docs and Chase AI's grill-with-docs-codex (MIT). Not for reviewing already-written code or trivial changes."
---

# Grill with Docs vs Claude

Two acts. Act 1 aligns intent and keeps living docs honest; Act 2 has Claude Code attack the resulting plan.

- **Act 1** is a Codex-hosted grill-with-docs session. Challenge the user against `GLOSSARY.md` and ADRs, then update them inline.
- **Act 2** is a bounded Claude Code adversarial review loop. Claude reviews only; Codex remains responsible for plan revisions and final arbitration.

The user participates by answering the grill and signing off the converged plan.

## ACT 1 - Grill With Docs (user <-> Codex)

Interview the user relentlessly about every aspect of the plan until you reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one by one. For each question, provide your recommended answer.

Ask questions one at a time, waiting for feedback on each question before continuing.

If a question can be answered by exploring the codebase, explore the codebase instead.

## Domain Awareness

During codebase exploration, also look for existing documentation:

```text
/
|-- GLOSSARY.md
|-- docs/
|   `-- adr/
|       |-- 0001-event-sourced-orders.md
|       `-- 0002-postgres-for-write-model.md
`-- src/
```

Use one root `GLOSSARY.md` for the project. Do not use or create `CONTEXT.md` or `CONTEXT-MAP.md`.

Create files lazily, only when there is something to write. If no `GLOSSARY.md` exists, create one when the first term is resolved. If no `docs/adr/` exists, create it when the first ADR is needed.

## During The Session

### Challenge Against The Glossary

When the user uses a term that conflicts with existing language in `GLOSSARY.md`, call it out immediately.

Example: "Your glossary defines 'cancellation' as X, but you seem to mean Y. Which is it?"

### Sharpen Fuzzy Language

When the user uses vague or overloaded terms, propose a precise canonical term.

Example: "You're saying 'account'. Do you mean the Customer or the User? Those are different things."

### Discuss Concrete Scenarios

When domain relationships are being discussed, stress-test them with specific scenarios. Invent scenarios that probe edge cases and force the user to be precise about boundaries between concepts.

### Cross-reference With Code

When the user states how something works, check whether the code agrees. If you find a contradiction, surface it.

Example: "Your code cancels entire Orders, but you just said partial cancellation is possible. Which is right?"

### Update GLOSSARY.md Inline

When a term is resolved, update `GLOSSARY.md` immediately. Do not batch these updates. Use the format in [GLOSSARY-FORMAT.md](./GLOSSARY-FORMAT.md).

`GLOSSARY.md` must be totally devoid of implementation details. Do not treat `GLOSSARY.md` as a spec, a scratch pad, or a repository for implementation decisions. It is a glossary and nothing else.

### Offer ADRs Sparingly

Only offer to create an ADR when all three are true:

1. **Hard to reverse**: the cost of changing the decision later is meaningful.
2. **Surprising without context**: a future reader will wonder why the project works this way.
3. **The result of a real trade-off**: there were genuine alternatives and one was chosen for specific reasons.

If any of the three is missing, skip the ADR. Use the format in [ADR-FORMAT.md](./ADR-FORMAT.md).

## Handoff To Act 2

When the decision tree is resolved, `GLOSSARY.md`/ADRs are updated, and the user is aligned, write the agreed plan to `PLAN.md` using canonical terms from `GLOSSARY.md`:

```markdown
# Plan: <task>
_Locked via grill-with-docs-vs-claude by Codex + <user>. Terms per GLOSSARY.md._

## Goal
<one paragraph, in the project's ubiquitous language>

## Approach
<numbered, concrete steps>

## Key decisions & tradeoffs
<the contestable choices the grill resolved; link any ADRs created>

## Risks / open questions
<anything still open>

## Out of scope
<bounds>
```

Initialize `PLAN-REVIEW-LOG.md`:

```markdown
# Plan Review Log: <task>
Act 1 (grill-with-docs-vs-claude) complete. Plan locked; GLOSSARY.md/ADRs updated. MAX_ROUNDS=<n>.
```

## ACT 2 - Review (Codex <-> Claude Code CLI)

Hand the locked plan to Claude Code for adversarial review. Codex revises the plan, decides which Claude findings are worth acting on, and keeps the log.

### Prerequisites

- `claude --version` works and Claude Code is authenticated.
- Use non-interactive print mode (`claude -p`) so the review can be captured.
- Keep Claude read-only by using `--permission-mode plan`, a read-only tool allowlist, and write-tool disallows.
- On auth, model, tool, or CLI errors, stop and surface the error. Do not silently retry with broader permissions.

### Tunables

| Var | Default | Meaning |
|-----|---------|---------|
| `MAX_ROUNDS` | `5` | Hard cap. Loop always terminates here. |
| `PLAN_FILE` | `PLAN.md` | The locked plan from Act 1. |
| `LOG_FILE` | `PLAN-REVIEW-LOG.md` | Append-only argument transcript. |

If invoked with arguments such as `rounds=3`, use them. Echo resolved values before Act 2 starts.

### Review Prompt

Use this prompt each round, adjusting only the round-specific wording:

```text
You are an adversarial reviewer for an implementation plan. Be skeptical and specific; your job is to find what breaks, not to be agreeable.

Read PLAN.md, GLOSSARY.md, ADRs, and any repo files you need. You are reviewing only and must not modify files.

Identify concrete flaws: security holes, race conditions, missing edge cases, schema conflicts, domain-language mismatches, wrong assumptions, observability gaps, and simpler alternatives. For each flaw, give a one-line fix.

End with exactly one line:
VERDICT: APPROVED
or:
VERDICT: REVISE
```

### Round 1 - Fresh Claude Session

Generate and reuse an explicit session id:

```bash
cat > /tmp/grill-claude-review-prompt.txt <<'PROMPT'
You are an adversarial reviewer for an implementation plan. Be skeptical and specific; your job is to find what breaks, not to be agreeable.

Read PLAN.md, GLOSSARY.md, ADRs, and any repo files you need. You are reviewing only and must not modify files.

Identify concrete flaws: security holes, race conditions, missing edge cases, schema conflicts, domain-language mismatches, wrong assumptions, observability gaps, and simpler alternatives. For each flaw, give a one-line fix.

End with exactly one line:
VERDICT: APPROVED
or:
VERDICT: REVISE
PROMPT

CLAUDE_SESSION_ID="$(uuidgen | tr '[:upper:]' '[:lower:]')"
CLAUDE_REVIEW_TOOLS="Read,Grep,Glob,LS"
CLAUDE_WRITE_DISALLOWS="Edit,MultiEdit,Write,NotebookEdit,Bash"

claude -p \
  --session-id "$CLAUDE_SESSION_ID" \
  --permission-mode plan \
  --tools "$CLAUDE_REVIEW_TOOLS" \
  --disallowedTools "$CLAUDE_WRITE_DISALLOWS" \
  --output-format text \
  "$(cat /tmp/grill-claude-review-prompt.txt)" \
  > /tmp/claude-verdict.txt
```

No `/tmp/claude-verdict.txt`, empty output, or missing final verdict means the run failed. Stop and tell the user.

### Rounds 2..MAX - Resume Same Claude Session

After Codex revises `PLAN.md`, resume the same Claude session:

```bash
claude -p \
  --resume "$CLAUDE_SESSION_ID" \
  --permission-mode plan \
  --tools "$CLAUDE_REVIEW_TOOLS" \
  --disallowedTools "$CLAUDE_WRITE_DISALLOWS" \
  --output-format text \
  "I revised the plan. Re-review PLAN.md, GLOSSARY.md, ADRs, and relevant repo files. Check prior findings and flag anything new. Do not modify files. End with VERDICT: APPROVED or VERDICT: REVISE." \
  > /tmp/claude-verdict.txt
```

### Each Round

1. Read `/tmp/claude-verdict.txt`; append `## Round <n> - Claude` plus the critique to `LOG_FILE`.
2. Last line `VERDICT: APPROVED`: proceed to Resolution.
3. Last line `VERDICT: REVISE`: Codex decides what is worth acting on, revises `PLAN_FILE`, and appends `### Codex response` with what changed, what was rejected, and why.
4. Round greater than `MAX_ROUNDS`: proceed to Deadlock.

## Resolution

### Approved

Present the final plan, a three-bullet summary of what the two acts improved, and the round count. Ask for user sign-off before implementing. Do not write implementation code during either act.

### Deadlock

If the cap is hit without approval, list unresolved points and Codex's counter-position. Hand the decision to the user. Do not fake convergence.

## Hard Rules

- Act 1 precedes Act 2.
- `GLOSSARY.md` stays a glossary only. No implementation details.
- Do not use or create `CONTEXT.md` or `CONTEXT-MAP.md`.
- Claude is read-only every round. Never let Claude edit files or run Bash.
- The loop always terminates at `MAX_ROUNDS`.
- Codex is final arbiter on `VERDICT: REVISE`; reject findings only with logged reasons.
- Code only after user sign-off. `PLAN-REVIEW-LOG.md` is part of the deliverable.

## What Not To Do

- Do not use this for reviewing already-written code.
- Do not skip Act 1.
- Do not let Claude modify files.
- Do not batch glossary updates until the end.
