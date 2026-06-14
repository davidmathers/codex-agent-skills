# GLOSSARY.md Format

## Structure

```md
# Project Glossary

One or two sentences describing the bounded project language this glossary covers.

## Language

**Order**:
A one or two sentence description of the term.
_Avoid_: Purchase, transaction

**Invoice**:
A request for payment sent to a customer after delivery.
_Avoid_: Bill, payment request

**Customer**:
A person or organization that places orders.
_Avoid_: Client, buyer, account
```

## Rules

- **Be opinionated.** When multiple words exist for the same concept, pick the best one and list the others under `_Avoid_`.
- **Keep definitions tight.** One or two sentences max. Define what the concept is, not what it does.
- **Only include terms specific to this project's domain.** General programming concepts such as timeouts, error types, and utility patterns do not belong even if the project uses them extensively.
- **Group terms under subheadings** when natural clusters emerge. If all terms belong to a single cohesive area, a flat list is fine.

Before adding a term, ask whether it is a concept unique to the project's domain or a general programming concept. Only the former belongs.

## Location

Use one root `GLOSSARY.md` for the project. If it does not exist, create it lazily when the first term is resolved.

Do not use or create `CONTEXT.md` or `CONTEXT-MAP.md`.
