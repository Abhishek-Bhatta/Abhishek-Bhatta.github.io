---
title: "Eliminating Cold Starts"
date: 2026-04-03
---

The internal post-mortem automation eventually worked but that wasn't the problem I'd been trying to solve. Customer-facing post-mortems were a different kind of challenge. When an incident happened, we owed our customers a written explanation — what broke, why, and what we'd done to prevent it. We needed to convey what we had learnt from the post-mortem and how we planned to evolve our product and our processes. The language had to be precise yet readable. More importantly, we wanted them to trust that they were in good hands.

## A simpler problem

Technical writing ability on the team varied, as it does with any global team that has a diverse background. Engineers from a customer facing background ended up delivering post-mortems with fewer review cycles and rewrites. Of course, everyone was improving steadily but our incident rate demanded faster growth. That's challenging, especially when writing and communication is a skill that develops with experience and time. As the incident rate grew, delivery slowed down and bottlenecks appeared in both drafting and reviewing. I'd learnt a bunch from my previous automation attempts and I was eager to try and optimize this.

The customer facing post-mortem had a few things working in my favor: it didn't need a lot of tool calls. I wasn't assembling a timeline from Slack channels and Freshdesk tickets. I already had the internal post-mortem. I just needed to read that, pull in one or two customer-facing tickets, and translate the content for a different audience. There were also several published post-mortems that I could use to extract the overall writing style and good vs bad examples from.

## Reducing activation energy

I skipped the freeform-first approach and went straight to phase-by-phase generation — one paragraph, correct it, move on. The feedback loop was the same: generate, refine in Confluence, feed the diff back to Claude, ask it to generate a new prompt, update the project instructions.

Four iterations. That was enough to get the output to a consistent floor — not polished, not as good as a deliverable document, but good enough that anyone could start from it. Starting a customer facing post-mortem had much less ceremony and didn't feel like a monumental task anymore. When I shared it with the team, the impact wasn't that the output got dramatically better. It was that the variance dropped. First drafts arrived faster and review cycles shortened. 

The automation didn't make anyone a better writer — it gave everyone a starting point that wasn't a blank page. 

## Where the pattern broke

I tried the same approach on internal ticket replies, like status updates. Give Claude the ticket, generate a draft, correct it, feed the diff back.

Much to my surprise, it didn't work. Every ticket had its own story. Every ticket involved a different set of stakeholders. The history of each ticket was unique and so were the decisions at different points in time. There was no template to converge on, so the prompt never stabilised. I spent significant time iterating on a skill that kept needing to be rewritten for each ticket. I eventually abandoned it.

The post-mortem automation worked because there was an underlying structure to converge on. Ticket replies didn't have one. That distinction mattered more than I'd initially assumed. Many months later, I realized an important lesson — if a decision is unique every time, make it easier for a user to decide. In short, give them the _context_ they need but don't automate the outcome. We'll explore this in a future post.