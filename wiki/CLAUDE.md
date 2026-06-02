# Concept Wiki — maintainer guide

This file is the **schema** for the EconML concept wiki. Claude Code auto-loads it whenever it works with files under `wiki/`. It turns a session into a disciplined wiki maintainer rather than a generic chatbot. Read it before ingesting sources, answering queries, or linting.

The wiki is a persistent, interlinked knowledge base of the **causal-inference / ML concepts** implemented in EconML. Its purpose: compile the knowledge that is otherwise scattered across `doc/spec/*.rst`, docstrings, and papers into one queryable artifact, so answers compound over time instead of being re-derived on every question.

## The three layers

1. **Raw sources (read-only).** The EconML codebase (`econml/`), the user guide (`doc/spec/*.rst`), the embedded `doc/Causal-Inference-User-Guide-v4-022520.pdf`, and the cited academic papers. The wiki **reads from these but never modifies them.** They are the source of truth.
2. **The wiki (Claude-owned).** Everything under `wiki/`. Claude creates pages, updates them as understanding deepens, maintains cross-references, and keeps the index/log current. The user reads it; Claude writes it.
3. **The schema.** This file. Co-evolve it with the user as conventions settle.

## Directory layout

```
wiki/
├── CLAUDE.md       # this schema
├── index.md        # catalog of every page, grouped by category (content-oriented)
├── log.md          # append-only chronological record (ingest | query | lint)
├── overview.md     # landing page / map of the domain
├── glossary.md     # term → 1-2 line definition, each linking to its concept page
├── concepts/       # cross-cutting theory (method-agnostic building blocks)
├── methods/        # estimator families (combine concepts; map to code)
├── architecture/   # how concepts map onto the class hierarchy & code
└── guides/         # task-oriented synthesis (choosing a method, inference, validation…)
```

## Page conventions

Every content page (everything except `index.md` and `log.md`) starts with YAML frontmatter:

```yaml
---
title: Double Machine Learning
type: method            # concept | method | architecture | guide
tags: [dml, orthogonal-ml, residualization]
related: [orthogonal-ml, cross-fitting, doubly-robust-learner]
source_files:           # raw sources this page synthesizes (paths, not line numbers)
  - econml/dml/dml.py
  - doc/spec/estimation/dml.rst
refs:                   # academic citations, see glossary/overview for the master list
  - "Chernozhukov et al. 2016, arXiv:1608.00060"
---
```

Body rules:
- **Link liberally** with `[[kebab-case-name]]` Obsidian-style wikilinks. The link target is another page's filename without the `.md` extension or directory (Obsidian resolves by basename). Every name in `related:` should also appear as a `[[link]]` somewhere in the body. A `[[link]]` to a page that doesn't exist yet is fine — it marks something worth writing.
- **Cite code by file path + symbol**, e.g. `econml/_ortho_learner.py::_OrthoLearner`, never by line number (line numbers rot; symbol names survive refactors).
- **Method pages** follow the structure used in `doc/spec/estimation/*.rst`: *What it is → When to use it → Formal methodology (the moment/loss) → How it maps to code → Key references*.
- **Concept pages** stay method-agnostic: define the idea once; let method pages reference it. Don't restate a concept's full theory inside a method page — link to the concept page instead.
- **Math**: use inline `$...$` / block `$$...$$` LaTeX (Obsidian renders it). Keep notation consistent with [[api-vocabulary]] (Y, T, X, W, Z, θ(X), etc.).
- **Naming**: kebab-case filenames; singular nouns for concepts (`propensity-score`, not `propensity-scores`).
- Synthesize and cross-link; **do not copy `.rst` prose verbatim.**

## Operations

### Ingest / Document
To add or deepen a concept:
1. Read the relevant raw sources (the right `doc/spec` file + the implementing code).
2. Write or update the page following the conventions above.
3. Add/refresh its entry in `index.md`.
4. Update `related:` and back-links on neighboring pages so the new page isn't an orphan.
5. Append a `log.md` entry.

### Query
When the user asks a question:
1. Read `index.md` first to locate relevant pages, then drill into them (don't re-derive from raw code unless the wiki is missing the answer).
2. Answer with citations to source files / pages.
3. **File good answers back into the wiki.** A novel synthesis, comparison, or connection should become a new page (usually under `guides/`) or extend an existing one — so explorations compound instead of vanishing into chat history. Log it.

### Lint
Periodic health check:
- Orphan pages (no inbound `[[links]]`); broken wikilinks (target file missing).
- Concepts mentioned across pages but lacking their own page.
- Stale claims: a page that contradicts the current code/`doc/spec` (the code is authoritative — flag and fix the page).
- Missing cross-references between obviously related pages.
- Report findings and proposed fixes; suggest new questions/sources worth pursuing.

## index.md vs log.md

- **`index.md`** is *content-oriented*: a catalog of what exists. One line per page (`- [[name]] — one-line summary`), grouped by category. Updated on every ingest. Read it first when answering queries.
- **`log.md`** is *chronological*: an append-only timeline. Each entry starts with a consistent prefix so it's greppable: `## [YYYY-MM-DD] ingest|query|lint | short title`. Quick recent history: `grep "^## \[" wiki/log.md | tail -5`.

## Authority

When the wiki and the code disagree, **the code wins** — update the wiki. The `doc/spec/*.rst` user guide is the next-best authority for theory and intent. Papers ground the math but EconML's implementation may differ in details (e.g. its MSE-variant forest criterion) — note such divergences explicitly on the page.
