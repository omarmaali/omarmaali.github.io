---
layout: post
title: "My graph-RAG hypothesis failed. Here’s the line of code that explains it."
reading_time: "8 min read"
summary: >-
  I spent a dissertation testing whether routing a query through a knowledge graph
  before falling back to vector search would cut token cost without costing accuracy.
  It didn't. Working out why was worth more than the result would have been.
---

<!--
  DRAFT — not published. Files in _drafts/ are ignored by the build.
  To publish: move to _posts/ and rename with a date prefix, e.g.
  _posts/2026-09-02-why-my-graph-rag-hypothesis-failed.md

  Everything below is an outline with the real numbers already in place.
  Write it in your own voice; the material is all here.
-->

## The hypothesis

State it plainly and early, before the reader knows it fails. A knowledge graph should
let you answer a multi-hop question by traversing relationships instead of stuffing
retrieved text into the context window. If that works, you pay fewer tokens for the same
answer.

Setup: 50-question HotpotQA multi-hop subset, sentence-transformers embedder, gpt-4o-mini,
`compose_facts=4`, `max_snippets=1`. A vector-only arm and a graph-routed arm, everything
else held constant.

## The result

| | Vector-only | Graph-routed |
|---|---|---|
| Exact match | 0.000 | 0.000 |
| Mean answer F1 | 0.088 | 0.055 |
| Tokens per question | 100.9 | 95.3 |

A 5.5% token saving for a 37% relative drop in F1. A `compose_facts` sweep at 2, 4, 6
and 8 confirms it is systematic rather than a bad run: cheaper at every setting, less
accurate at every setting. That is a bad trade and there is no way to write it up as
anything else.

## Cause one: the part that was supposed to win was switched off

This is the section that carries the post. `ingest/hotpot_official.py` calls
`graph.add(...)`, a method that does not exist on `GraphStore`. A bare `except` swallowed
the `AttributeError`, so the networkx view was never built and `best_path_facts` — the
multi-hop path scoring, the entire source of the hypothesised advantage — was inactive
for the whole benchmark. "Graph-routed" retrieval had quietly degraded into the
string-matching cascade underneath it.

Worth being explicit about the decision here: I documented this rather than fixing it and
re-running, so the published numbers stay reproducible against the published code. That
is a defensible choice and also an uncomfortable one. Say why.

## Cause two: the fairness constraint made the advantage unexpressable

`max_snippets=1` is what makes the A/B fair, and it also caps supporting-fact recall at
0.50 by construction, because HotpotQA questions need two supporting sentences. An
experiment can be fair and uninformative at the same time.

## Cause three: it was never the fallback

Instrumenting the routing layer on its own: the graph answered 50 of 50 questions and
never once used its vector fallback, at 0.165 supporting-fact recall. So the accuracy gap
isn't the fallback and isn't context volume. It is *which single sentence the router
chose*. Under a grounded-only prompt, a topically-related but not load-bearing sentence
becomes "I don't know" rather than a wrong answer — which is the right failure mode, and
still a failure.

## What I would actually take from this

The point isn't that graph RAG doesn't work. The point is that the system was
instrumented well enough that a null result was diagnosable down to a specific line,
instead of being written up as "graph RAG didn't work for us" and quietly believed by
everyone who read it.

Close on the three follow-ups, in priority order: fix the ingest call and re-run both
arms, which is the first real test of the hypothesis; raise the budget to two snippets in
both arms; add paired significance tests.
