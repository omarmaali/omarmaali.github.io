---
layout: post
title: "The four guardrails I couldn’t prove were tested"
date: 2026-08-16
reading_time: "6 min read"
summary: >-
  I had six guardrails and a README claiming each one had a test. Switching them
  off one at a time turned out to be the only way to find out whether that was
  true. The first answer I got was wrong, which is the more useful half of the
  story.
---

I finished an agent project with six guardrails in it. The README said each one had a
test. I believed that when I wrote it, because I had written the tests myself and I
could point at them.

Then I tried to prove it, and found I couldn't.

## Reading a test tells you almost nothing

Here is the shape of the problem. A guardrail is a line, or a handful of lines, whose
entire job is to stop something bad. When it is working, nothing happens. That is the
point of it. So a test for a guardrail passes in exactly the same way whether the
guardrail is doing its job or has been quietly disconnected from the code path six
commits ago.

A green suite tells you the code you have works. It cannot tell you that a test would
go red if you deleted a particular line. And for a guardrail, that is the only property
that matters, because the realistic failure mode isn't someone deciding to remove your
safety check. It's someone refactoring the function it lives in, and nothing anywhere
noticing.

You can't get at this by reading. I tried. Reading a test suite for coverage of a
specific line is exactly the kind of task where you convince yourself faster than you
verify.

## Switching them off

The only honest check is mechanical: turn each guardrail off, one at a time, re-run the
whole suite, and see what fails. If nothing fails, no test was covering it.

This is mutation testing, narrowed to a specific target. General mutation testing tools
mutate everything — flip a comparison operator, swap a constant — and drown you in
results, most of which are about code you don't care very much about. For guardrails you
want the opposite: a small, deliberate list of switches, each one corresponding to a
protection you have made a promise about.

So the tool needs three ways to disable a thing, because guardrails are written three
different ways:

- an environment variable, when the guardrail already has a kill switch
- a config key, when it is threshold-driven
- a find-and-replace in the source, when it is just a line of code

That last one edits files on disk, so it refuses to run on a dirty working tree,
snapshots the target file's SHA-256 before writing, restores in a `finally`, and re-reads
the file afterwards to confirm the hash matches. I did not want a crashed test run to
leave someone's repository quietly modified.

## 62%

I pointed it at the agent project. Nine guardrails once I counted properly. Five killed,
which means a test caught the removal. Four survived.

62%.

I nearly published that number. It is a good number to publish, in the sense that it is
embarrassing and specific and makes the tool look useful. A tidy story: *I thought my
code was well tested, it was 62% tested, here is the tool that told me.*

I spent five minutes checking the survivors first.

Three of the four were false. The tests were setting `settings.max_steps = 4` directly on
the settings object, so switching the environment variable never reached them. The
guardrail was tested. The switch just wasn't the thing the test was looking at. Given a
handle that edits the check itself, all nine were killed. The honest score is 9 of 9.

Then I pointed it at the other project, a retrieval assistant with four guardrails. Two
more false survivors, same cause. And one real one.

## The real one

The retrieval assistant's system prompt says: *You answer ONLY from the numbered sources
given in the CONTEXT block.* I replaced that with "Answer however you like" and re-ran
the entire suite.

Everything stayed green.

That gap is real and it is not fixable by adding a unit test, which is why it is worth
saying out loud rather than patching over. What a prompt instructs is validated by
evaluation runs against a real model, not by pytest. A unit test can check that the
prompt string contains the sentence, and that test would pass while telling you nothing
about whether the model obeys it. The honest thing is to document the boundary: the
deterministic guardrails are covered by the suite, and the prompt-level behaviour is
covered by the eval runs, and those are different kinds of evidence with different
strengths.

## What four false alarms are worth

Four false survivors across two projects, and every one of them was a properly tested
guardrail that my tooling reported as unprotected.

If I had shipped the 62% and moved on, I would have published a number that was wrong in
the direction that flattered the tool. The finding that a measurement tool is loudest
exactly where it is least reliable is not a footnote to the project. It is the most
useful thing the project produced.

So the tool now says so itself. When a config or environment mutation survives, it prints
a caveat next to the result, in the terminal and in the generated report: **a patch
survivor is evidence, a config survivor is only a question.** If you disabled the
guardrail by editing the line and the suite stayed green, you have a genuine gap. If you
disabled it by flipping a config value and the suite stayed green, the first thing to
check is whether your tests read that config at all.

## What it still doesn't prove

A killed mutation proves that *some* test fails when the guardrail is off. It says
nothing at all about whether the guardrail is correct. You can have a thoroughly tested
guardrail that checks the wrong thing, and this technique will give it full marks.

It also won't survive a refactor of the line it targets, because the patch mutator is a
find-and-replace rather than an AST transform. And it needs each guardrail to be
disableable from outside the code, which turns out to be diagnostic in itself: if you
can't switch a protection off with an environment variable, a config key or a one-line
patch, it's usually because it's woven into a function that is doing three other things
at the same time.

## If you want to try this without a tool

You don't need one. Pick the guardrail in your system that would hurt most if it silently
stopped working. Comment it out. Run your tests.

If they pass, you have learned something today. If they fail, check *which* test failed
and whether it failed for the reason you expected, because the second most common outcome
after "nothing failed" is "something failed for an unrelated reason and I would have
counted it".

Either way it takes about two minutes, and you will know something about your codebase
that reading it could not have told you.
