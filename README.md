# second-me-wiki-distiller

> Build and maintain an **LLM-maintained personal markdown knowledge base** — not a neutral wiki, but the seed of a **second me**: a memo you won't forget, a reusable knowledge asset, and a representation a future model can use to reason in your knowledge, taste, and way of thinking.
> In one line: **writing (quick / distill) + maintenance (review) + lookup (search)** — four modes covering the whole lifecycle, with **step-by-step resource loading** so the agent only reads the rules it actually needs.

🌐 中文版 → [`README_CN.md`](./README_CN.md)

---

## What it is

`second-me-wiki-distiller` is a **standalone, shareable skill** (an operating manual). It teaches an agent to take scattered material — notes, web clippings, articles, chats, social media, books, research, screenshots, GitHub projects, conversations — and **pre-compile** it into structured, interlinked, traceable markdown, settling into **three layers**:

- **`raw/`** — the source layer: original material, attachments, and the unprocessed inbox. Single source of truth, kept close to its original form.
- **`wiki/`** — the LLM-maintained distilled layer: the slice of world knowledge *you* chose to keep, compiled for a future model to reuse. **Not** a neutral encyclopedia and **not** a copy of the raw text.
- **`self/`** — a slow-changing representation of you. This is the **second-me seed**: who you are, what you value, how you think.

The point isn't to re-write generic encyclopedia knowledge a model already has. It's to keep **your viewpoint, your sourcing path, information past the training cutoff, and reusable personal context.**

> **First principle — this is not a neutral encyclopedia, it's "yours".** It serves three purposes: ① a **memo** (things you're afraid to forget, easy to look up); ② a **knowledge asset** (your accumulation, reusable); ③ a **second-me** (once accumulated enough, a model can use your knowledge + style + thinking to make rough judgments on your behalf).

> **Division of labor (after Karpathy).** You do **four things**: curate sources, direct the analysis, ask good questions, and think about what it all means. **Everything else is the AI's** — summarizing, cross-referencing, filing, bookkeeping, keeping pages consistent. You don't hand-maintain the wiki and you don't sign off on every entry; correctness is anchored by **traceable sources** (every fact links back to where it came from) and **periodic lint** (scheduled health-checks for contradictions, staleness, broken links, orphans), not by line-by-line human review. People abandon wikis when maintenance outgrows the value — here the AI carries that cost, so it stays near zero.

### Design at a glance

- **Three layers** — `raw/` (source) → `wiki/` (compiled knowledge) → `self/` (the second-me seed). Quotes aren't a separate layer; they're a `wiki/quotes/` page type (`type: quote`).
- **One root index.** A single `index.md` (plus `log.md` / `nextstep.md`) is the navigation.
- **Rules vs. format are split.** `references/` holds the *method* (how to decide, what goes where); `schemas/` holds the *format* (YAML frontmatter, controlled vocabulary, body structure).
- **Progressive disclosure is built in.** `SKILL.md` only routes; each mode reads its references and schemas **step by step**, loading a rule file only at the step that needs it. The agent never front-loads the whole manual.
- **Controlled-vocabulary YAML, no per-entry review chore.** Pages carry Obsidian/Dataview-friendly frontmatter (`status`, `confidence`, `depth`, `volatility`…). `wiki/` and root pages carry **no** review-status field — AI writes are usable on arrival; trust rests on traceable sources and periodic lint, not line-by-line sign-off. Only `self/` pages keep a lightweight `verified` checkbox (default `false`, the user ticks it `true`), since facts about *you* are the costly ones to get wrong.

## What it does / does not do

- ✅ **Quick** — any write that doesn't need raw inbox material: add, edit, archive, or delete a `wiki/`/`self/` entry, or jot a small knowledge point straight in.
- ✅ **Distill (write)** — process `raw/inbox/` (and other raw material), route each piece through raw normalization → wiki candidates → self signals → quotes, and write traceable pages.
- ✅ **Review (maintain)** — sweep the library: contradictions, lint, staleness, broken links, orphans, dedup, `scrap`/`dropzone` clustering, and **self up-leveling** (aggregate latent attention signals from the log into stable conclusions).
- ✅ **Search (ask)** — answer "did I save X" / "what's in the library about Y" by reading the compiled pages and citing them; never writes, never touches `log.md`, never fabricates.
- ❌ **No orchestration awareness** — it ships no `AGENTS.md`/`CLAUDE.md` and doesn't know who dispatches it; multi-agent batching is a higher layer's concern.

## The mode gate (always the first step)

Every invocation **picks a mode first**, from the prompt's semantics — the user never has to name the mode. There are **four modes** — quick, distill, review, search — covering the spectrum from a quick one-off addition, to a hands-off batch run, to maintaining the library, to answering questions about it. Selection logic lives in `SKILL.md`; before the gate, a lightweight **Step 0 root check** confirms the working directory really is a knowledge base (needs ≥2 of: `raw/`, `wiki/`, `self/`, root files) before doing anything.

## Progressive disclosure (the core design)

`SKILL.md` is deliberately thin: core idea + root check + mode gate + a handful of shared hard rules + a resource map. It **routes only**. The detailed method lives in `modes/`, `references/`, and `schemas/`, and is loaded **lazily**:

```
SKILL.md  →  modes/<mode>.md  →  (step by step) references/* then schemas/*
```

Each mode is a numbered sequence, and each step states exactly which rule file to read *at that step* — e.g. distill reads `references/raw.md` only when claiming material, `references/wiki.md` only when there's a wiki candidate, and a concrete `schemas/*.md` only at the moment of writing. The agent never reads the whole manual up front, which keeps context lean and decisions local.

## The weak-signal mechanism

A design choice specific to a self-centric base. **Strong signals about the person go into `self/` directly. But there's also a stream of *weak* signals — what you keep choosing to distill is itself faint evidence about your attention and interests.** Writing each into `self/` would flood it and duplicate the log, so they're left **latent in `log.md`** (which carries lightweight `topic` tags). **Review mode** later aggregates them over `topic × time-window` into conclusions that graduate into the identity layer — turning a pile of routine actions into a real observation, with no duplication and no bloat.

## Core rules (summary)

1. **Trace, never fabricate** — every fact links to a source; if missing, leave it blank, don't pad with `unknown`. One fact, one source library-wide.
2. **Light verify on `self/` only** — `self/` pages carry a `verified` checkbox (AI keeps it `false`; only the user ticks `true`). `wiki/`/root pages have no review gate — they're trusted on write and kept honest by sources + lint.
3. **Search before write** — dedup by source / similar title / alias / URL / core sentence.
4. **Absolute dates** — never persist "yesterday / today / recently" without the `YYYY-MM-DD`.
5. **Keep contradictions** — on conflict, keep both claims, sources, and dates; mark it, never silently overwrite.
6. **Never delete, only iterate** — archive / supersede / deprecate, and keep history; confirm before any physical delete.
7. **Don't over-infer** — weak-evidence personality / preference calls get `confidence: low` or wait in `nextstep.md`; don't infer protected traits, medical/legal/financial status from thin evidence.
8. **Wiki is compiled memory, not a raw store** — keep the raw path, don't move full original text into `wiki/`.
9. **Calibrate depth on the timeline** — knowledge before `2025-01-01` is written light unless it matters to you; newer knowledge (filling the post-training gap) earns more detail, judged by *usefulness*, not completeness.

## The structure it scaffolds (idempotent, checks where first)

Fills in whatever's missing, skips what already exists; "already exists" is judged by ≥2 signals (`raw/`, `wiki/`, `self/`, or ≥2 of the root files) — not by bumping into one coincidentally-named file. If the bar isn't met and the user hasn't pointed to an existing library, it stops and asks rather than guessing a location.

```
<project-root>/
├── index.md  log.md  nextstep.md           # single root navigation + append-only log + open items
├── raw/    # single source of truth, kept close to original
│   ├── inbox/        # unprocessed entry  (inbox/clipping/ for clipper auto-saves)
│   ├── attachment/   # binaries (PDF/img/video/PPT/audio), each referenced by a raw .md pointer
│   └── articles/ books/ chats/ design/ ideas/ research/ videos/ work/ socialmedia/ dropzone/
├── wiki/   # concept/ entity/ topic/ summary/ method/ tip/ github/ comparison/ quotes/ scrap/
└── self/   # observations.md identity.md preferences.md now.md profile.md  (no separate index — the five files are the entry)
```

> Internal links use Obsidian style with a folder prefix: `[[wiki/concept/example]]`, `[[self/identity]]`. Non-markdown material never stands alone — it goes into `raw/attachment/` with a minimal markdown pointer file that wiki pages reference.

## How to use

Drop material into `raw/inbox/`, then tell the agent what you want ("distill these", "review the library", "jot down this person's profile link", "did I save anything about X"). It runs the root check, picks a mode from your wording, loads only the rules that step needs, claims items out of inbox when needed, routes each through the layers, writes the pages, and updates `index.md` / `log.md` / `nextstep.md` — lookup questions are answered straight from the library with no file changes. Full method in `SKILL.md`.

> One more thing: this skill keeps evolving with the author's own day-to-day use. If a step / prompt / guardrail doesn't fit how you work, fork it and adjust — future updates mostly follow the author's usage and won't necessarily fit everyone.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Entry (deliberately thin): core idea + Step 0 root check + mode gate + shared hard rules + resource map. Routes only. |
| `modes/{quick,distill,review,search}.md` | Per-mode step-by-step behavior, each step loading references/schemas lazily. |
| `references/raw.md` | raw/ method: categories, attachments, pointer files, dedup. |
| `references/wiki.md` | wiki/ method: page-type decision, boundaries, depth, GitHub deep-read, conflicts. |
| `references/self.md` | **Core** — self five-piece, evidence levels, extraction dimensions, second-me profile, sensitivity. |
| `schemas/common.md` | Shared YAML frontmatter + controlled vocabulary + date rules. |
| `schemas/raw.md` | raw pointer format, source metadata, attachment references. |
| `schemas/wiki.md` | Per-type frontmatter + body structure for each wiki page type. |
| `schemas/self.md` | Schema for the self five-piece. |
| `schemas/root.md` | `index.md` / `log.md` / `nextstep.md` format. |

## Lineage & Credits

Its architecture draws on:

- **Andrej Karpathy's "LLM Wiki"** (gist): <https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f> — pre-compiling knowledge into interlinked markdown, with immutable raw sources + an LLM-maintained layer + a rules schema.
- The **honcho project** (Plastic Labs' open-source personal memory layer): <https://github.com/plastic-labs/honcho> — background-distilling personal atomic facts, keeping contradictions, superseding old with new, abstaining when evidence is thin.

## Independence

This skill is **self-contained and usable on its own** — it depends on no external orchestration file and on no other skill. Copy the whole `second-me-wiki-distiller/` directory to reuse or share it elsewhere.
