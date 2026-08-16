---
title: 'Building a "Today I Learned" Site — and Using It to Learn How AI Systems Work'
description: 'TIL started as a place to save what I read. It turned into the best way I have found to actually learn AI-system engineering: BYOK, hybrid retrieval, a digest pipeline, a chat agent — and the evals that told me I had built the search wrong.'
pubDate: 'Aug 16 2026'
heroImage: '../../assets/blog-placeholder-2.jpg'
---

I read something interesting almost every day and forget most of it by the weekend. A link, a paper, a clever blog post — read once, closed, gone. So I built a small tool to fix that: paste a URL, and an LLM turns it into something I will actually remember.

I called it **TIL** — Today I Learned. But that was only half the reason I built it. The other half was that I wanted a real, honest playground for learning how to engineer an AI system — not a to-do-list demo, but something with retrieval, agents, a background pipeline, and a way to know whether any of it actually works. A tool small enough to finish and real enough to teach me something.

This post is about both halves: the little site, and the experiment hiding inside it.

## The core loop

The whole product is one gesture. You paste a link, and TIL:

1. fetches and extracts the article to clean markdown,
2. asks an LLM for a **~150-word summary**, the **single most interesting takeaway**, **3–6 tags**, and **one follow-up question worth exploring**,
3. stores it and indexes it for search.

Over time that becomes a searchable feed of what you have learned. Everything runs as one full-stack Cloudflare Worker — a React SPA and a Hono API behind the same deploy — with SQLite (Cloudflare D1) underneath. It is single-user and self-hosted on purpose: it runs in *my* Cloudflare account, with *my* LLM key, and no third party holds my data or my key.

That "follow-up question" field is my favourite. Most read-later tools stop at *what did this say?* Asking the model for *what should I look at next?* turns a passive archive into something that nudges you forward.

## The part that makes it an experiment

A summarizer is a weekend toy. What made this worth building was treating it as an AI *system* — the boring, load-bearing parts that separate a demo from something you would trust with your own API key.

**Every external capability sits behind a small interface.** `LLMClient`, `Extractor`, `Embedder`, `VectorStore` — the app layer never imports the LLM SDK directly. That one discipline paid for itself immediately: it let me flip the entire stack between two modes with a single environment variable.

```
TIL_STACK=local   Readability + Ollama embeddings + cosine search in SQLite
TIL_STACK=cloud   Workers AI + Vectorize
```

Same embedding model (`bge-m3`) on both sides, so local search behaves like production. I can develop the whole capture → digest → search flow offline, then deploy the same code against Cloudflare's services.

**The key never touches the browser.** It is bring-your-own-key — OpenAI, Anthropic, or Groq — but every call is routed through my own Cloudflare AI Gateway, which gives me caching, rate limiting, and cost visibility for free. The Worker holds the key; the frontend never sees it. For a system that can spend real money, that boundary is the whole game.

## Retrieval, and the humbling part

Search is where "learn how AI systems work" stopped being a slogan.

TIL indexes every entry two ways at ingest time: a **vector** in Vectorize for semantic queries ("what have I saved about consensus algorithms?") and a **full-text row** in SQLite FTS5 for keyword and exact matches ("that arXiv paper on speculative decoding"). The chat agent's search tool queries both and fuses the results with reciprocal-rank fusion. Textbook hybrid retrieval. I was pleased with it.

Then I built an eval harness and measured it. The result, on a golden set of ranked queries:

```
nDCG@8    FTS-only     0.593
          hybrid       0.674
          vector-only  0.816
```

Hybrid beat keyword search everywhere — and **lost to plain vector search on every single slice**. The clever fused thing I was proud of was worse than just using the vectors.

Digging in, the cause was not the tuning knob I expected (the RRF `k` barely moved the needle). It was two quiet bugs: the keyword leg *never abstained* — it ORed every token, including stopwords, so it always returned a full pool of confidently-wrong candidates — and when ranks tied, the merge broke the tie **alphabetically by entry id**, cheerfully interleaving junk ahead of correct vector hits. The fusion was actively dragging down its own best leg.

The fix was an abstaining keyword leg (filter stopwords, return nothing on an empty query), a principled tiebreak, and — the piece that actually closed the gap — down-weighting keyword votes when the keyword leg had clearly found a *topic* (dozens of loose hits) rather than pinned a *document* (a handful). Re-measured:

```
nDCG@8    hybrid       0.828   ≥   vector-only  0.806
```

Now hybrid is at least as good as its best leg on every slice, and better where it counts — on the identifier-style queries (error codes, CVEs) that live in the article text but never make it into the embedding.

I keep coming back to this, because it is the whole thesis of the project. I had 500-plus unit tests, all green. They proved the wiring was correct. They said nothing about whether the search was *good*. Every real quality failure I have found on this project was found by **looking** — reading transcripts, staring at rankings — not by a passing test. The evals just turned "this feels off" into a number I could move.

## A pipeline and an agent

Two more pieces round out the "system" part.

The **weekly digest** is a scheduled Cloudflare Workflow that goes out to Hacker News, Lobsters, arXiv, and my own RSS feeds, ranks the candidates by recency, popularity, and cross-source corroboration, and has an LLM write up the most interesting ones. It is a durable pipeline — plan, fetch, rank, synthesize — not an autonomous agent wandering the web. The distinction matters: I can reason about every step, and a failure at step three does not take the whole run down with it.

The **chat agent** is where the safety posture gets real. You can ask "what did I read most this month?" and it answers using three tools — hybrid search, single-entry lookup, and reading statistics — all strictly **read-only**, with arguments clamped and results size-capped before they reach the model, and every answer cites the entries it came from. A couple of implementation truths I did not expect going in: chat is WebSocket-only, and because browsers cannot set auth headers on a WebSocket handshake, the app mints a 60-second signed ticket that is accepted *only* on a chat upgrade — so the real token never ends up in a URL or an access log.

And throughout, article text is treated as **untrusted data**. The extraction and digest path has no tools at all, and every prompt that handles fetched content is told, explicitly, not to follow instructions found inside it. When your input is "whatever was on the internet at that URL," prompt injection is not a hypothetical.

## What the experiment taught me

The tool does what I wanted: I paste links, and a month later I can actually find what I read and why it mattered. That alone would have been worth it.

But the lasting value was the shape of the thing. Building a *small* AI system end to end — with real seams, a background pipeline, an agent with tools, and above all a way to measure quality — taught me more than any tutorial, precisely because it let me be wrong in a way I could catch. The hybrid-search story is the one I will remember: I built something reasonable, believed it was good, and only a boring evaluation harness told me the truth.

If you want to learn how these systems really work, don't build the demo that always succeeds. Build the small real thing, then build the ruler that tells you it doesn't — yet.

## Links

- **Code:** [github.com/canhtoanptit/til](https://github.com/canhtoanptit/til)
- **Live instance:** [til.canhtoanptit.workers.dev](https://til.canhtoanptit.workers.dev/)
