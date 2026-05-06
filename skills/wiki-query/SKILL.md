---
name: wiki-query
description: "Answer questions using the Obsidian wiki vault. Reads hot cache first, then discovers pages via folder/Glob/tag, then reads the most relevant pages. Synthesizes answers with citations. Files good answers back as wiki pages. Supports quick, standard, and deep modes. Triggers on: what do you know about, query:, what is, explain, summarize, find in wiki, search the wiki, based on the wiki, wiki query quick, wiki query deep."
allowed-tools: Read Glob Grep
---

# wiki-query: Query the Wiki

The wiki has already done the synthesis work. Read strategically, answer precisely, and file good answers back so the knowledge compounds.

---

## Query Modes

Three depths. Choose based on the question complexity.

| Mode | Trigger | Reads | Token cost | Best for |
|------|---------|-------|------------|---------|
| **Quick** | `query quick: ...` or simple factual Q | hot.md + scoped Glob | ~1,500 | "What is X?", date lookups, quick facts |
| **Standard** | default (no flag) | hot.md + Glob/Grep + 3-5 pages | ~3,000 | Most questions |
| **Deep** | `query deep: ...` or "thorough", "comprehensive" | Full Glob walk + optional web | ~8,000+ | "Compare A vs B across everything", synthesis, gap analysis |

> [!note] No master index
> This skill does **not** read or maintain `wiki/index.md`. A central hub page is an Obsidian graph-view anti-pattern. Discovery uses the folder hierarchy (`wiki/concepts/`, `wiki/entities/`, etc.), Glob (`wiki/**/*.md`), Grep over frontmatter `tags:`, and per-folder `_index.md` sub-indexes when present.

---

## Quick Mode

Use when the answer is likely in the hot cache or a single targeted page.

1. Read `wiki/hot.md`. If it answers the question, respond immediately.
2. If not, infer the right folder from the question (entity → `wiki/entities/`, concept → `wiki/concepts/`, etc.) and Glob that folder for filenames matching the topic.
3. If a folder has an `_index.md`, read it for the scoped catalog.
4. If you can answer from the hot cache plus one targeted file, do so.
5. If not, say "Not in quick cache. Run as standard query?"

Do not Glob the entire `wiki/` tree in quick mode.

---

## Standard Query Workflow

1. **Read** `wiki/hot.md` first. It may already have the answer or directly relevant context.
2. **Discover** candidate pages:
   - Glob the relevant folders (`wiki/concepts/*.md`, `wiki/entities/*.md`, `wiki/sources/*.md`, etc.).
   - Grep frontmatter `tags:` or page bodies for the topic keywords.
   - Read any `_index.md` files in scoped folders (these are sub-indexes, not a master hub).
3. **Read** the 3-5 most relevant pages. Follow wikilinks to depth-2 for key entities. No deeper.
4. **Synthesize** the answer in chat. Cite sources with wikilinks: `(Source: [[Page Name]])`.
5. **Offer to file** the answer: "This analysis seems worth keeping. Should I save it as `wiki/questions/answer-name.md`?"
6. If the question reveals a **gap**: say "I don't have enough on X. Want to find a source?"

---

## Deep Mode

Use for synthesis questions, comparisons, or "tell me everything about X."

1. Read `wiki/hot.md`.
2. Glob `wiki/**/*.md` to enumerate every page. Filter by folder + `tags:` + filename keyword.
3. Read every relevant page. No skipping.
4. If wiki coverage is thin, offer to supplement with web search.
5. Synthesize a comprehensive answer with full citations.
6. Always file the result back as a wiki page. Deep answers are too valuable to lose.

---

## Token Discipline

Read the minimum needed:

| Start with | Cost (approx) | When to stop |
|------------|---------------|--------------|
| hot.md | ~500 tokens | If it has the answer |
| Glob + Grep on a folder | ~100 tokens | Once you can identify 3-5 candidate pages |
| folder `_index.md` (if present) | ~500 tokens | If it covers the question scope |
| 3-5 wiki pages | ~300 tokens each | Usually sufficient |
| 10+ wiki pages | expensive | Only for synthesis across the entire wiki |

If hot.md has the answer, respond without reading further.

---

## Discovery Reference

There is no master `wiki/index.md` to scan. To map the vault:

```bash
# All pages in a domain
Glob: wiki/concepts/*.md
Glob: wiki/entities/*.md
Glob: wiki/sources/*.md
Glob: wiki/questions/*.md

# Tag-based filter across the whole vault
Grep: 'tags:.*\b<topic-tag>\b' in wiki/**/*.md (frontmatter)

# Title keyword across the whole vault
Glob: wiki/**/*<keyword>*.md
```

Combine these to land on 3-5 candidate pages, then Read them.

---

## Domain Sub-Index Format

Each domain folder has a `_index.md` for focused lookups:

```markdown
---
type: meta
title: "Entities Index"
updated: YYYY-MM-DD
---
# Entities

## People
- [[Person Name]]: role, org

## Organizations
- [[Org Name]]: what they do

## Products
- [[Product Name]]: category
```

Use sub-indexes when the question is scoped to one domain. Avoid reading the full master index for narrow queries.

---

## Filing Answers Back

Good answers compound into the wiki. Don't let insights disappear into chat history.

When filing an answer:

```yaml
---
type: question
title: "Short descriptive title"
question: "The exact query as asked."
answer_quality: solid
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: [question, <domain>]
related:
  - "[[Page referenced in answer]]"
sources:
  - "[[wiki/sources/relevant-source.md]]"
status: developing
---
```

Then write the answer as the page body. Include citations. Link every mentioned concept or entity.

After filing, append an entry to `wiki/log.md`. Do not maintain a master `wiki/index.md`. If `wiki/questions/_index.md` exists in the vault, you may add a one-line entry there.

---

## Gap Handling

If the question cannot be answered from the wiki:

1. Say clearly: "I don't have enough in the wiki to answer this well."
2. Identify the specific gap: "I have nothing on [subtopic]."
3. Suggest: "Want to find a source on this? I can help you search or process one."
4. Do not fabricate. Do not answer from training data if the question is about the specific domain in this wiki.
