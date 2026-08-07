Title: Executive Summary and Remaining Content in TieredRAG
Date: 2026-08-07
Category: GenAI
Tags: GenAI, LLM, RAG, TieredRAG, document-processing, retrieval, token-optimization
Slug: executive-summary-and-remaining-content-in-tieredrag

TieredRAG splits a document into two parts before any retrieval happens. The first part — the executive summary — is always processed. The second part — the remaining content — is only touched when the first part isn't enough. Here's what each part contains and why the split works.

## Executive Summary (Priority Content)

The executive summary is not just the summary page of a document. In TieredRAG, it refers to all high-signal sections extracted from the full document — roughly the top 20% by information density.

**What it includes:**

- Abstract or introduction
- Executive summary or overview
- Key findings and conclusions
- Important tables and figures
- Critical paragraphs flagged by structure or position

**Why process this first** — These sections are written to communicate the most important information in the least space. For most queries, the answer lives here. Processing only this portion keeps token usage low and response time fast.

**What happens after** — The LLM evaluates this content and produces a response with a confidence score. If confidence meets the threshold (typically ≥ 90%), the answer is returned immediately. The rest of the document is never touched.

## Remaining Content (Temporary Memory)

Everything outside the executive summary goes into temporary memory — the other 80% of the document.

**What it includes:**

- Supporting arguments and detailed explanations
- Appendices and references
- Methodology sections
- Data tables and raw figures
- Background context

**How it's accessed** — It is not loaded into the LLM context upfront. It sits in a secondary store. Only when the confidence check fails does the system query this memory, retrieve the most relevant chunks, and feed them into a second evaluation pass.

**Why not just load everything** — Putting 100 pages into context adds noise, inflates token cost, and slows the response. The remaining content is detailed but rarely needed for most queries. Keeping it in memory and retrieving selectively gives you coverage without the cost.

## The Split in Practice

| | Executive Summary | Remaining Content |
|---|---|---|
| Size | ~20% of document | ~80% of document |
| Loaded upfront | Always | Only on fallback |
| Purpose | Fast, high-confidence answers | Fallback detail retrieval |
| Token cost | Low | On-demand only |

> The executive summary is the fast path. The remaining content is the safety net. Most queries never need the net — and that's exactly the point.
