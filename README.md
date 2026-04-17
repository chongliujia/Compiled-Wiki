# Compiled Wiki

*A knowledge system built with compiler principles*

---

## Overview

This project is a **compiled knowledge system powered by LLMs**.

Instead of retrieving information from raw documents at query time (as in traditional RAG systems), this approach treats knowledge construction as a **compile-time process**.

Raw information is ingested, structured, and incrementally compiled into a persistent, evolving wiki.

---

## Core Idea

Traditional LLM workflows:

```
Question → Retrieve documents → Generate answer
```

This system:

```
Sources → Compile → Structured Knowledge → Query
```

The key shift:

> Knowledge is not reconstructed every time — it is **built once and maintained over time**.

---

## Built with Compiler Principles

This system is inspired by **compiler architecture**, not search engines.

### Mapping

| Compiler Concept                 | This System       |
| -------------------------------- | ----------------- |
| Source code                      | Raw documents     |
| IR (Intermediate Representation) | Structured claims |
| Compiler passes                  | Ingest pipeline   |
| Output binary                    | Wiki pages        |
| Static analysis                  | Lint system       |

---

## Architecture

```
raw/      → source of truth (immutable)
ir/       → structured knowledge (claims, entities)
graph/    → dependency + conflict graph
wiki/     → human-readable output
compiler/ → rules, schema, passes
logs/     → change history
```

---

## Core Concepts

### 1. Claims (IR)

All knowledge is represented as structured claims.

Each claim includes:

* subject / predicate / object
* type (fact, inference, hypothesis)
* source reference
* status (supported, disputed, etc.)

> Claims are the ground truth layer.

---

### 2. Entities (Symbol Table)

Entities act like a compiler symbol table:

* canonical names
* aliases
* attached claims
* relationships

---

### 3. Wiki (Rendered View)

The wiki is **not the source of truth**.

It is a rendered view generated from structured knowledge:

* entity pages
* concept pages
* analyses
* hypotheses

---

### 4. Dependency Graph

Knowledge is modeled as a graph:

* claims → entities
* entities → pages
* pages → analyses

This enables:

* precise updates
* incremental rebuilds
* impact tracking

---

## Compiler Pipeline

### Ingest (Compilation)

1. **Parse**
   Extract candidate claims from sources

2. **Normalize**
   Resolve entities, deduplicate, standardize

3. **Resolve**
   Merge with existing knowledge, detect conflicts

4. **Validate**
   Ensure all claims are well-formed and sourced

5. **Plan**
   Determine which parts of the system need updates

6. **Patch**
   Apply minimal changes (no full rewrites)

7. **Render**
   Update wiki pages from structured data

8. **Lint**
   Check consistency and detect issues

9. **Log**
   Record all changes

---

## Query Flow

1. Read `wiki/index.md`
2. Traverse relevant pages
3. Optionally inspect structured claims
4. Generate answer with citations
5. (Optional) write back new analysis

---

## Why Not RAG?

Standard RAG systems:

* recompute knowledge every time
* lack accumulation
* struggle with synthesis across many documents

This system:

* builds knowledge incrementally
* maintains structure over time
* separates understanding from answering

---

## Design Principles

### 1. Traceability First

Every claim must map to a source.

### 2. No Silent Overwrites

Conflicts are explicit, never merged away.

### 3. Incremental Updates

Only affected parts are updated.

### 4. Structure Over Prose

Prefer structured knowledge over smooth text.

### 5. Wiki is a View

The wiki is derived, not authoritative.

---

## Lint System (Static Analysis)

### Errors

* claim without source
* broken references
* unresolved entities
* untracked contradictions

### Warnings

* orphan pages
* stale summaries
* duplicate entities

---

## Conflict Handling

Contradictions are first-class objects.

They are:

* detected during compilation
* stored explicitly
* rendered in wiki pages

No automatic resolution is applied.

---

## Workflow

### Ingest

Add a source → compile → update wiki

### Query

Read compiled knowledge → answer

### Lint

Continuously validate system health

---

## When to Use

Best for:

* personal research
* deep topic exploration
* long-term knowledge accumulation
* structured thinking

---

## When Not to Use

Not ideal for:

* real-time search
* lightweight Q&A
* rapidly changing data streams

---

## Philosophy

> This is not a note-taking tool.
> This is not a search engine.

> This is a **knowledge compiler** — a system that transforms raw information into a structured, evolving understanding.

---

## Getting Started

1. Add a source to `raw/`
2. Run ingest via LLM agent
3. Review generated wiki updates
4. Iterate and refine schema

---

## Future Work

* richer dependency tracking
* automated conflict resolution workflows
* search layer (hybrid BM25/vector)
* visualization of knowledge graph
* UI over claims and entities

---

## Final Note

The value of this system is not in generating answers.

It is in **building a knowledge base that improves over time, instead of starting from scratch every time**.

