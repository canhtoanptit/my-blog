---
title: 'Building This Blog in an Afternoon: Astro, Claude, and Cloudflare'
description: 'How three tools — Astro for content, Claude as my assistant, and Cloudflare for hosting and DNS — turned a weekend project into an afternoon one.'
pubDate: 'Jul 23 2026'
heroImage: '../../assets/blog-placeholder-1.jpg'
---

I've wanted a personal blog for years. Not a Medium account, not a LinkedIn feed — my own domain, my own words, no algorithm deciding who sees them. What always stopped me wasn't the writing. It was the yak-shaving: pick a framework, wrestle with a theme, fight a build pipeline, wire up DNS, renew a certificate that expired at the worst possible moment.

In 2026 that friction is essentially gone. I put this whole thing together in an afternoon using three tools that each do one job well: **Astro** for the content, **Claude** as my assistant, and **Cloudflare** for hosting and DNS. Here's how the pieces fit together.

## Astro: the content layer

A blog is mostly text, and text should be simple. Astro leans into that. Posts are just Markdown files in a folder:

```
src/content/blog/
  building-this-blog-astro-claude-cloudflare.md
```

Each file has a bit of frontmatter — title, description, publish date, a hero image — and Astro's content collections give me type-safe validation on all of it. If I forget a field or fat-finger a date, the build tells me before anything ships.

The part that still feels like a cheat code: Astro renders to **static HTML by default**. No client-side framework booting up just to display a paragraph. The pages are fast because there's almost nothing to send. For a blog, that's exactly the right trade — the content is the product, and the fastest component is the one that isn't there.

## Claude: the assistant that did the plumbing

This is the piece that genuinely changed since the last time I tried this. I didn't hand-write the scaffolding, the layout tweaks, or the config. I described what I wanted to **Claude** — in plain English — and it did the work in the repo alongside me.

"Set up the git remote with this SSH key." Done. "Move my résumé to the about page and build a homepage like the popular dev blogs." Done, with the professional info laid out in a card grid and the homepage modeled on the layout every good engineering blog seems to converge on: a short intro, a list of recent writing, a row of links. "Remove all the starter posts." Gone.

What used to be an afternoon of Stack Overflow tabs became a conversation. I stayed the architect — deciding what I wanted and why — while the assistant handled the mechanical parts I've done a hundred times and don't need to prove I can do again. That's the right division of labor, and it's the first time AI tooling has actually felt like that to me rather than a novelty.

## Cloudflare: hosting, domain, and DNS in a few clicks

The last mile — the part that historically ate an entire evening — was the easy one.

**Hosting.** Cloudflare Pages connects straight to the GitHub repo. Every push to `main` triggers a build, and the static output lands on Cloudflare's edge network. Global CDN, free TLS, atomic deploys — none of which I had to configure. Push, wait a moment, it's live.

**Domain.** I bought the domain directly from the **Cloudflare Registrar**. At-cost pricing, no upsells, and — crucially — it's already inside the same dashboard as everything else. No copying nameservers between two vendors and waiting a day to see if they took.

**DNS and routing.** Because the registrar and the host are the same company, connecting them is a few clicks. In the Pages project I added `blog.canhtoan.dev` as a custom domain; Cloudflare created the DNS record and provisioned the certificate for me. That's it. The thing I used to dread — DNS propagation roulette — was a non-event.

```js
// astro.config.mjs — the one line that has to match the domain
export default defineConfig({
  site: 'https://blog.canhtoan.dev',
});
```

## Why this combination works

Each tool respects a boundary. Astro owns the content and the build. Claude owns the tedious, well-understood implementation work. Cloudflare owns delivery and the network. None of them try to be a platform that does everything and does it badly.

The result is a stack I can reason about end to end, that costs almost nothing to run, and that I can extend one small piece at a time. More importantly, the setup got out of the way fast enough that I still had energy left to do the actual point of a blog: write.

So — first real post, shipped. If you're sitting on a domain you bought in a burst of optimism and never used, this is your sign. The hard part was never the tools. In 2026, it's *really* not the tools.
