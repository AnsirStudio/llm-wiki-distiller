# llm-wiki-distiller

> Build and maintain an **LLM-maintained personal markdown knowledge base** — not a neutral wiki, but *your* slice of world knowledge: a memo you won't forget, a reusable knowledge asset, compiled for a future model to reason with.
> In one line: **writing (quick / distill) + maintenance (review) + lookup (search)** — plus an **integrate** mode for parallel batch runs — covering the whole lifecycle, with **step-by-step resource loading** so the agent only reads the rules it actually needs.

🌐 中文版 → [`README_CN.md`](./README_CN.md)

---

## What it is

`llm-wiki-distiller` is a **standalone, shareable skill** (an operating manual). It teaches an agent to take scattered material — notes, web clippings, articles, chats, social media, books, research, screenshots, GitHub projects, conversations — and **pre-compile** it into structured, interlinked, traceable markdown, settling into **two layers**:

- **`raw/`** — the source layer: original material, attachments, and the unprocessed inbox. Single source of truth, kept close to its original form.
- **`wiki/`** — the LLM-maintained distilled layer: the slice of world knowledge *you* chose to keep, compiled for a future model to reuse. **Not** a neutral encyclopedia and **not** a copy of the raw text.

The point isn't to re-write generic encyclopedia knowledge a model already has. It's to keep **your viewpoint, your sourcing path, information past the training cutoff, and reusable personal context.**

> **First principle — this is not a neutral encyclopedia, it's "yours".** It serves two purposes: ① a **memo** (things you're afraid to forget, easy to look up); ② a **knowledge asset** (your accumulation, reusable by you and by models acting for you).

> **Division of labor (after Karpathy).** You do **four things**: curate sources, direct the analysis, ask good questions, and think about what it all means. **Everything else is the AI's** — summarizing, cross-referencing, filing, bookkeeping, keeping pages consistent. You don't hand-maintain the wiki and you don't sign off on every entry; correctness is anchored by **traceable sources** (every fact links back to where it came from) and **periodic lint** (scheduled health-checks for contradictions, staleness, broken links, orphans), not by line-by-line human review. People abandon wikis when maintenance outgrows the value — here the AI carries that cost, so it stays near zero.

### Design at a glance

- **Two layers** — `raw/` (source) → `wiki/` (compiled knowledge). Quotes aren't a separate layer; they're a `wiki/quotes/` page type (`type: quote`).
- **One root index.** A single `index.md` (plus `log.md` / `pending.md`) is the navigation.
- **Rules vs. format are split.** `references/` holds the *method* (how to decide, what goes where); `schemas/` holds the *format* (YAML frontmatter, controlled vocabulary, body structure).
- **Progressive disclosure is built in.** `SKILL.md` only routes; each mode reads its references and schemas **step by step**, loading a rule file only at the step that needs it. The agent never front-loads the whole manual.
- **Controlled-vocabulary YAML, no per-entry review chore.** Pages carry Obsidian/Dataview-friendly frontmatter (`status`, `confidence`, `depth`, `volatility`…) with **no** review-status field — AI writes are usable on arrival; trust rests on traceable sources and periodic lint, not line-by-line sign-off.
- **Parallel-safe batching.** Distill has a `capture-only` variant (each sub-agent claims one item, writes only its own summary/raw/proposal) plus an `integrate` mode (one serial writer merges all proposals into shared pages) — so a batch of N items can be captured in parallel with zero write conflicts.

## What it does / does not do

- ✅ **Quick** — any write that doesn't need raw inbox material: add, edit, archive, or delete a `wiki/` entry, or jot a small knowledge point straight in.
- ✅ **Distill (write)** — process `raw/inbox/` (and other raw material), route each piece through raw normalization → wiki candidates → quotes, and write traceable pages.
- ✅ **Review (maintain)** — sweep the library: contradictions, lint, staleness, broken links, orphans, dedup, `scrap`/`dropzone` clustering, distill-quality sampling, library metrics — cursor-driven and sliced, never a full-library read; each run emits a dated report under `review/` (the latest report *is* the cursor, and rollups aggregate report frontmatter into monthly/quarterly views).
- ✅ **Search (ask)** — answer "did I save X" / "what's in the library about Y" by reading the compiled pages and citing them; never writes, never touches `log.md`, never fabricates.
- ❌ **No orchestration awareness** — it ships no `AGENTS.md`/`CLAUDE.md` and doesn't know who dispatches it; when to parallelize and how to dispatch sub-agents is the caller's concern.

## The mode gate (always the first step)

Every invocation **picks a mode first**, from the prompt's semantics — the user never has to name the mode. There are **four gate modes** — quick, distill, review, search — covering the spectrum from a quick one-off addition, to a hands-off batch run, to maintaining the library, to answering questions about it (`integrate` is only ever invoked explicitly by an orchestrator, never guessed from user wording). Selection logic lives in `SKILL.md`; before the gate, a lightweight **Step 0 root check** confirms the working directory really is a knowledge base (needs ≥2 of: `raw/`, `wiki/`, root files) before doing anything.

## Progressive disclosure (the core design)

`SKILL.md` is deliberately thin: core idea + root check + mode gate + a handful of shared hard rules + a resource map. It **routes only**. The detailed method lives in `modes/`, `references/`, and `schemas/`, and is loaded **lazily**:

```
SKILL.md  →  modes/<mode>.md  →  (step by step) references/* then schemas/*
```

Each mode is a numbered sequence, and each step states exactly which rule file to read *at that step* — e.g. distill reads `references/raw.md` only when claiming material, `references/wiki.md` only when there's a wiki candidate, and a concrete `schemas/*.md` only at the moment of writing. The agent never reads the whole manual up front, which keeps context lean and decisions local.

## Core rules (summary)

1. **Trace, never fabricate** — every fact links to a source; if missing, leave it blank, don't pad with `unknown`. One fact, one source library-wide.
2. **Search before write** — dedup by source / similar title / alias / URL / core sentence.
3. **Absolute dates** — never persist "yesterday / today / recently" without the `YYYY-MM-DD`.
4. **Keep contradictions** — on conflict, keep both claims, sources, and dates; mark it, never silently overwrite.
5. **Never delete, only iterate** — archive / supersede / deprecate, and keep history; confirm before any physical delete.
6. **Don't over-infer** — weak-evidence calls get `confidence: low` or wait in `pending.md`; don't infer protected traits, medical/legal/financial status from thin evidence.
7. **Wiki is compiled memory, not a raw store** — keep the raw path, don't move full original text into `wiki/`.
8. **No per-page edit logs** — change history belongs to git and `log.md`; a page only gets a timeline when the chronology *is* the content (product evolution, a topic unfolding).
9. **Calibrate depth on the timeline** — knowledge before `2025-01-01` is written light unless it matters to you; newer knowledge (filling the post-training gap) earns more detail, judged by *usefulness*, not completeness.

## The structure it scaffolds (idempotent, checks where first)

Fills in whatever's missing, skips what already exists; "already exists" is judged by ≥2 signals (`raw/`, `wiki/`, or ≥2 of the root files) — not by bumping into one coincidentally-named file. If the bar isn't met and the user hasn't pointed to an existing library, it stops and asks rather than guessing a location.

```
<project-root>/
├── index.md  log.md  pending.md           # single root navigation + append-only log + open items
├── _staging/  # capture→integrate proposals (fixed dir; contents transient — consumed by integrate)
├── raw/    # single source of truth, kept close to original
│   ├── inbox/        # unprocessed entry  (inbox/clipping/ for clipper auto-saves)
│   ├── attachment/   # binaries (PDF/img/video/PPT/audio), each referenced by a raw .md pointer
│   └── articles/ books/ chats/ design/ ideas/ research/ videos/ work/ socialmedia/ dropzone/
├── wiki/   # concept/ entity/ topic/ summary/ method/ tip/ github/ comparison/ quotes/ scrap/
└── review/ # dated review reports, review/YYYY-MM/YYYY-MM-DD-HHMM.md — the latest one doubles as the review cursor
```

> Internal links use Obsidian style: bare names `[[example]]` in page bodies, folder-prefixed `[[wiki/concept/example]]` in `log.md`. Non-markdown material never stands alone — it goes into `raw/attachment/` with a minimal markdown pointer file that wiki pages reference.

## How to use

Drop material into `raw/inbox/`, then tell the agent what you want ("distill these", "review the library", "jot down this person's profile link", "did I save anything about X"). It runs the root check, picks a mode from your wording, loads only the rules that step needs, claims items out of inbox when needed, routes each through the layers, writes the pages, and updates `index.md` / `log.md` / `pending.md` — lookup questions are answered straight from the library with no file changes. Full method in `SKILL.md`.

> One more thing: this skill keeps evolving with the author's own day-to-day use. If a step / prompt / guardrail doesn't fit how you work, fork it and adjust — future updates mostly follow the author's usage and won't necessarily fit everyone.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Entry (deliberately thin): core idea + Step 0 root check + mode gate + shared hard rules + resource map. Routes only. |
| `modes/{quick,distill,review,search}.md` | Per-mode step-by-step behavior, each step loading references/schemas lazily. |
| `modes/integrate.md` | Orchestrator-only: merge all `capture-only` proposals into shared pages — single serial writer. |
| `references/raw.md` | raw/ method: categories, attachments, pointer files, dedup. |
| `references/wiki.md` | wiki/ method: page-type decision, boundaries, depth, GitHub deep-read, conflicts. |
| `schemas/common.md` | Shared YAML frontmatter + controlled vocabulary + date rules. |
| `schemas/raw.md` | raw pointer format, source metadata, attachment references. |
| `schemas/wiki.md` | Per-type frontmatter + body structure for each wiki page type. |
| `schemas/root.md` | `index.md` / `log.md` / `pending.md` format. |
| `schemas/proposal.md` | The capture→integrate contract (`_staging/<id>.md`). |
| `schemas/report.md` | Review report format — dated reports whose frontmatter doubles as the review cursor and feeds rollups. |

## Lineage & Credits

Its architecture draws on:

- **Andrej Karpathy's "LLM Wiki"** (gist): <https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f> — pre-compiling knowledge into interlinked markdown, with immutable raw sources + an LLM-maintained layer + a rules schema.
- The **honcho project** (Plastic Labs' open-source personal memory layer): <https://github.com/plastic-labs/honcho> — background-distilling personal atomic facts, keeping contradictions, superseding old with new, abstaining when evidence is thin.

## Independence

This skill is **self-contained and usable on its own** — it depends on no external orchestration file and on no other skill. Copy the whole `llm-wiki-distiller/` directory to reuse or share it elsewhere.
