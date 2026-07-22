---
title: 'Debugging My Workday'
description: 'What two years of working from home taught me about focus, boundaries, and treating my own day like a system worth fixing.'
pubDate: 'Jul 22 2026'
heroImage: '../../assets/blog-placeholder-2.jpg'
---

For most of my career I treated my workday as a given — a fixed backdrop that the "real" work happened in front of. Then I started working from home full time, the backdrop disappeared, and I had to build the whole thing myself. Commute, colleagues, the ambient pressure of an office: gone. What was left was me, a laptop, and a suspicious amount of freedom.

It turns out a workday is just a system. And like any system, mine had bugs.

## The bug report

The symptoms were familiar to anyone who's done this:

- I'd sit down at 9am and look up to find it was 1pm and I'd shipped nothing but Slack replies.
- I'd finish a deep-focus block and have no idea whether I'd been working for forty minutes or three hours.
- "Just one more thing before dinner" reliably became two hours after dinner.

Classic production incident: the system works, but nobody can tell you *why* it works on the good days and falls over on the bad ones. So I did what I'd do with any flaky service — I started adding observability before I tried to fix anything.

## Instrumenting the day

I didn't buy an app. I opened a plain text file and wrote down, roughly hourly, what I was actually doing. No judgment, just logging. After two weeks the patterns were embarrassingly obvious:

- My best code happened between 9 and 11:30am. Every meeting scheduled in that window was, in effect, a load spike hitting my most valuable server.
- Context switches were the real cost. It wasn't the 5-minute Slack reply; it was the 20 minutes afterward spent reloading the mental stack I'd just dropped.
- I never actually stopped working. I just got worse at it as the day went on, and mistook "still at the desk" for "still productive."

You can't fix what you can't see. Once the log made the cost of interruptions visible, the fixes stopped being abstract self-help advice and started being obvious engineering decisions.

## The fixes that stuck

**Protect the peak, batch the rest.** I blocked 9–11:30 as no-meeting, notifications-off deep work. Everything asynchronous — reviews, replies, planning — got batched into the afternoon when my focus was already degraded anyway. I wasn't working *more* hours, I was matching the work to the capacity I actually had at each hour.

**A hard shutdown ritual.** The office used to give me a physical `SIGTERM` — I'd stand up, walk out, and the day was over. At home there's no such signal, so I built one: at 6pm I write tomorrow's first task in that same text file, close the laptop, and go for a walk. The walk isn't optional. It's the graceful shutdown that stops today's process from leaking into tonight.

**Treat rest as part of the system, not a failure of it.** This was the hard one. For a long time I felt guilty stepping away from the desk at 2pm, even when my brain was clearly cache-cold. But nobody praises a server for running at 100% CPU indefinitely — that's the thing you page someone about. Rest isn't the absence of work. It's the maintenance window that keeps the work possible.

## What this is really about

I dressed all of this up in engineering metaphors because that's how my brain likes to hold ideas. But underneath the observability and the graceful shutdowns is something simpler and less technical: I stopped treating my own attention as infinite and free, and started treating it as the scarce, valuable resource it actually is.

Working from home didn't give me more discipline. It removed all the external scaffolding that used to fake discipline on my behalf, and forced me to build my own. The upside is that a system you build yourself is one you can actually debug.

I still have bad days. The difference now is that a bad day is a bug report, not a verdict — something to look at, understand, and quietly fix in the morning.
