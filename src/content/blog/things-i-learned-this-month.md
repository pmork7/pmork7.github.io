---
title: "Things I Learned This Month"
date: 2026-03-10
description: "A handful of small things that clicked for me this month."
tags: ["learning", "dev"]
---

A few things that clicked for me this month.

## Postgres `EXPLAIN ANALYZE` is underrated

I've been using Postgres for years but only recently started leaning hard on `EXPLAIN ANALYZE`. Seeing the actual row estimates vs. real counts makes slow query debugging so much faster. If you're not using it, start.

## Read the source, not just the docs

Docs lie — or at least, they go stale. I spent two hours debugging a library behavior that was clearly documented in the source (with a comment explaining *why* it worked that way) but not in the official docs. Reading source is slower upfront but faster overall.

## Small feedback loops change everything

I switched my test runner to watch mode by default for a project I'm working on. Running tests on save rather than manually feels like a different workflow entirely. The constant signal keeps me honest.

---

That's it for this month. Nothing groundbreaking, but small things compound.
