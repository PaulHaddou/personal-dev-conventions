# dev-conventions

Living documentation of my personal dev conventions, written to be readable by humans and LLMs alike.

The goal is to turn conventions that are currently implicit (in my head, or scattered across past projects) into explicit, versioned documents. This makes it possible to:

- Onboard a new project faster, by starting from a known baseline instead of re-deciding everything.
- Give an LLM (Claude, or any other coding assistant) a clear reference to follow, instead of guessing my preferences from context.
- Iterate on the conventions themselves over time, with a history of *why* something changed.

## Scope

These conventions currently cover the **frontend** (Next.js, App Router). Backend conventions (Fastify) live separately for now, a link will be added here once that convention exists.

## Structure

Each file below covers one topic, independently of the others:

| File | Topic | Status |
|---|---|---|
| `architecture.md` | Application architecture (Feature-Sliced Design, adapted for Next.js) | ✅ Draft |
| `folder-structure.md` | Folder structure | ⏳ Not started |
| `component-structure.md` | Typical component structure | ⏳ Not started |
| `state.md` | State management | ⏳ Not started |
| `data-fetching.md` | Data fetching | ⏳ Not started |
| `styling.md` | Styling | ⏳ Not started |
| `testing.md` | Testing | ⏳ Not started |
| `naming.md` | Naming | ⏳ Not started |

## How to use this with an LLM

Point your assistant to the relevant file(s) as context before starting a task, e.g. "follow `architecture.md` and `naming.md` for this feature." Each document is meant to be self-contained enough to be dropped into a prompt or system context without extra explanation.

## Status

This repo is under active construction. Structure, file names, and conventions below are expected to change as they get used on real projects, nothing here is final yet.
