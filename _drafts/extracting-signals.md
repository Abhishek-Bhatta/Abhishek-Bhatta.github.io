---
title: "Extracting Signals"
---

As an organization, we needed to understand our incidents properly — not just that they happened, but how long they took to detect, how long to resolve, which services were affected and so on. The process was: read the internal postmortem, extract a handful of fields, update a Jira ticket. That ticket fed into a data pipeline. A BI tool turned it into dashboards. 

## The First Attempt

Our team had a large backlog of metrics updates — postmortems that had never been processed. So I initially designed a Claude skill to bypass the document entirely if it was empty and gather raw context from Slack channels and tickets instead. Broad scope, fast results.

Predictably, it produced inconsistent output. The inferences were hit or miss. The same incident would get different field values depending on what tools were called. I didn't understand why yet.

## Debugging Hallucinations

I had added human-in-loop verification early — when the skill made an inference, I'd approve or correct it before it wrote anything to Jira. Because I knew these incidents well, I could spot any inconsistencies. I decided to be Socratic and ask Claude to explain its results. I began to realize it was looking at the wrong context or had not fetched the right context at all. If I pointed it to a Slack thread it had missed, it subsequently produced the right inferences.

The pattern was clear: good context and correct tool calls mattered. Without it, any model invented plausible-sounding values that were entirely incorrect. I needed the model to admit it didn't know, but then I needed to teach it where to usually look for the right data. I ended up adding a few instructions to the Claude skill.

I eventually added rules around the tool calls themselves, where to look vs where not to look, degrade gracefully if unable to find the right context, and so on. As a neat trick, I also externalized the Jira issue schema into a mapping reference to be able to update the ticket in a single tool call without needing to fetch metadata every time.

## More Hallucinations?

I had added inference rules into the skill. For example, if the postmortem mentioned a PR that had an unintended side effect, record the primary root cause as a code change. If the reasoning was unclear, flag it for human review. Despite our fix from earlier and supposedly clear "logic", it still wasn't reliable enough for production use by the rest of the team. It turned out that there was ambiguity in the business logic itself. 

Sometimes, an incident genuinely didn't fit a clean category — conflicting definitions, edge cases nobody had thought through, dual root causes, and plenty of other interesting observations. I added graceful degradation: when the rules conflicted, stop and ask rather than guess forward. I quickly understood that there was another type of hallucination. We no longer invented new inferences, but sometimes inferences were correctly mapped but wrong in terms of business logic. For example, there may have been a PR that contributed to an issue, but perhaps a separate system was misconfigured when both needed to be changed together. This isn't a simple code change, it's a gap in tooling instead. However, these were not encoded into the skill, so Claude was confident, but wrong. This made me think of adding confidence scoring (we'll touch on this in a later post).

## Production Readiness

My final battle was with Claude's web UI. It was unreliable — network errors mid-session, forgotten turns, tool calls that reset unexpectedly. When a comment posting tool call failed and I reapproved it, it posted twice. I added idempotency checks. When sessions broke partway through, the skill had lost track of what it had done. I added a simple JSON state file in the chat to track progress across interruptions. None of this was sophisticated. It was just making the skill production ready.

## What the Process Taught Me

The skill defined a new standard: gather context first, make all inferences, get human approval, then write to Jira. In that order, always. Context gathering and updating business systems needed human checkpoints.

The unexpected side effect was that running the process faster made the ambiguities in our metrics definitions visible. When I was doing this by hand, edge cases got resolved in the moment and forgotten. When a skill had to make the same judgment repeatedly, the inconsistencies surfaced. I ended up in stakeholder conversations about what we actually wanted to measure — conversations that led to new fields and aligned definitions across teams.

We didn't just automate the process. It forced us to externalize tribal knowledge and standardize it across the team. We were, in effect, building employee handbooks that we should've had long ago.
