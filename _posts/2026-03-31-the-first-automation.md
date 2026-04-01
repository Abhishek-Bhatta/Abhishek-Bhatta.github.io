---
title: "The First Automation"
date: 2026-03-31
---

Our incident post-mortem process had always been challenging to automate. At Workato's scale, things break in non-trivial ways, and the post-mortem was how we made sense of it. The goal was always to automate the toil — copying incident details from Slack channels and Jira tickets into a document by hand felt unnecessary. But the real constraint was depth and quality. We wanted to compile as many learnings from an incident that we could. We wanted timeline entries detailed enough to extract reliable metrics: not just that an incident happened, but how it evolved, where time was lost, what ideas we missed and how to set a foundation for a better future. I'd tried to automate it once before, and we briefly had an intern build a custom solution, but we deprecated it soon after the intern left. Nobody had made it work.

## What I had tried

Back in 2023, I'd tried to automate parts of document generation. I discovered that Confluence's API was borderline unusable. Parsing the document format it expected required a library and a lot of patience. Even as an expert Workato recipe builder, doing this well in a short time span was challenging. My first attempt got as far as filling in the basics — channel name, ticket links, date. Anything that required reading details and making sense of it, I gave up on. So, for a long time, RCA documents were filled mostly by hand.

I'd used LLMs a year earlier and was mostly unimpressed. My early experiments ended up in generated content that I threw away, but I was eager to try and figure out some solution that could reduce toil.

## The first iteration

Our organization unlocked Claude for everyone sometime last year, and I felt that it was a good chance to try a content generation use case in earnest. I created a Claude project and wrote a simple prompt: read the template, read the Slack incident channel, pull in any relevant Jira and Freshdesk tickets, populate the document. Ask me before writing anything.

The first session was a long freeform conversation. Claude fetched things, generated text, asked questions, looped back. Eventually it produced something, but it didn't look like any RCA we'd written before. The format was off. The framing was off. The first try wasn't promising.

I also made a separate mistake early on. Once the project was set up, I decided to overhaul the RCA template — there were things I'd wanted to fix for years. What I didn't know was that other teams had automations built on top of the existing structure. I broke those. I confused people who'd been using the old format. I reverted within a day. I'd never expected to cause an incident myself.

## An unexpected roadblock

The incident timeline needed to show when customers first reported the issue, and that information lived in Freshdesk, not Slack. Much to my dismay, Freshdesk doesn't provide a native MCP server. But I'd been moving fast and I wanted to carry that momentum.

Since I work at Workato and we'd launched an MCP server offering earlier that year, I decided to poke around. Our platform can expose API collections as MCP servers. I didn't want to build a full connector from scratch, so I looked for an OpenAPI spec — Freshdesk doesn't publish an official one, but I found an unofficial spec in about ten minutes. I imported it, cleaned it up slightly, and the MCP server was running in fifteen minutes. That went way better than I'd expected and I was working on RCA generation again in half an hour.

One catch: our production Claude instance is locked down — adding custom MCP servers requires an approval process. So once I finished my RCA and wanted to share my Claude project with the team, I submitted an internal ticket. Getting the MCP approved and deployed took two days since people are distributed across the globe. Once in production, the whole team and the support team could use Claude with the Freshdesk MCP to speed up their work. Workato's RBAC meant we could gate access by team to make sure we prevented any unauthorized access.

That gap stuck with me. Fifteen minutes to build. Two days to deploy. I'd never experienced this kind of speed before, and I was determined to keep going.

## Refining the output

I found that Claude was way too interested in exploring unrelated channels and context so I added approval gates for tool calls — before Claude explored a Slack channel or fetched a ticket, I had to confirm it was relevant. This stopped the wandering. I also found that reviewing a single, large document was painful. Instead, I broke generation into phases and reviewed the output one section at a time, corrected it, and moved on. Only after the full document was approved did Claude write anything to Confluence. 

The process felt slower but it was more reliable now. This reliability was the foundation. I wasn't undoing mistakes made three sections ago. The key idea I came back to was that "Slow is smooth, smooth is fast."

## The feedback loop

At the end of each session, I was looking at a gap: the document Claude had generated versus the document I'd actually wanted. That gap contained information — specific corrections, things I'd rephrased, structural changes I'd made. I started feeding that diff back to Claude and asking it to analyse what had changed and why, then use those differences to write an updated prompt. I'd put the updated prompt back into the project instructions. Two or three iterations, and the output was consistent enough to share with the team.

The prompt loop was the thing worth keeping. Every completed RCA made the next one slightly better, without me having to work out what to change. But that presented me with a new problem, could I automate that too? Spoiler alert: it wasn't easy.