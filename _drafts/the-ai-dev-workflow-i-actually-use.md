---
layout: post
title: "The AI dev workflow I actually use after 15 years as an architect"
reading_time: "7 min read"
summary: >-
  Not a tool list. The filter — which AI development tools survived real enterprise
  work, which I deleted after two weeks, and the one rule I use to decide.
---

<!--
  DRAFT — not published. Files in _drafts/ are ignored by the build.
  To publish: move to _posts/ and rename with a date prefix, e.g.
  _posts/2026-09-16-the-ai-dev-workflow-i-actually-use.md

  Outline carried over from Growth/Backlog_LinkedIn_DevToolkit_Article.md.
  Publish here first, then post an excerpt on LinkedIn pointing back.
-->

## Hook

Two or three lines: what the workflow looked like eighteen months ago versus now. Concrete,
not abstract. Resist the urge to open with a statistic.

## The keep/delete rule

One sentence, quotable, stated before any tool is named. Everything after this is an
application of it. Draft candidates worth trying:

- A tool earns its place if removing it would cost me more than learning it did.
- If I can't tell whether it helped, it didn't.

## What stayed

One short story each, with the concrete win. Do not let these all resolve into a neat
closing line — vary the shape.

- The code assistant, and what it is actually good at versus what it is sold as
- An AI review pass before the human review, aimed at the failure classes generated code
  actually has: invented APIs, requirement drift, swallowed errors, over-engineering
- Spec-driven scaffolding, which is where most of the real gain lives
- Test generation, with the caveat that generated tests are the ones most likely to be
  unable to fail
- Commit and PR summaries, the cheapest win on the list

## What I deleted

The differentiator. Everyone writes the first list; almost nobody writes this one. Be
specific about the reason: noise, hallucinated APIs, slower than typing, or the security
and IP position at an enterprise client.

## What changes in an enterprise

The paragraph most posts skip. What happens when the code isn't yours to paste into a
chatbot. Tool tiers by data class, what never goes into a prompt, and the incident rule
for a pasted secret. Keep it generic — no internal tooling, no client names, no employer
specifics.

## The evidence I trust

Tie back to the mutation-testing work: a pass over two of my own AI-built projects found
four guardrails whose tests could never fail. That is the argument for the review
discipline, and it is mine rather than a vendor's.

## Close

Point at the playbook repository. No "thoughts?" ending.
