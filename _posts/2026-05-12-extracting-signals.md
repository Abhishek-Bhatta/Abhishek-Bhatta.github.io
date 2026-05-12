---
title: "Extracting Signals"
date: 2026-05-12
---

As an organization, we needed to understand our incidents properly. That meant uncovering how they happened, how long it took to detect, the time we took to resolve, and a bevy of other things. The process was to read the internal postmortem, extract a handful of fields and update a Jira ticket. That ticket fed into a data pipeline. A BI tool turned it into dashboards. 

## The First Attempt

Our team had a large backlog of metrics updates leading up to the 2025 holiday season. There were several postmortems that hadn't been processed, mostly because they were empty. The internal postmortem automation from the first post was still gaining gradual adoption. So, I initially designed a Claude skill to bypass the document entirely if it was empty and gather raw context from Slack channels and tickets instead.

Predictably, it produced inconsistent output. The inferences were hit or miss. The same incident would produce different metrics depending on what tools were called. I didn't understand why yet.

## Debugging Hallucinations

I had added human-in-loop verification early; when the skill made an inference, I'd approve or correct it before it wrote anything to Jira. Because I knew these incidents well, I could spot any inconsistencies. Whenever Claude hallucinated, I took a Socratic approach and ask Claude to explain its results. I began to realize that it was either looking at the wrong context or it missed fetching the right context entirely. If I pointed it to a Slack thread it had missed, it subsequently produced the right inferences.

The pattern was clear: good context and correct tool calls mattered. Without it, Claude would make things up. I needed the LLM to admit it didn't know, and then I needed to teach it where to usually look for the right data. I ended up adding a few instructions to the Claude skill to stop if it couldn't find a source for inference. I later learnt that this was called graceful degradation.

As a neat trick, I also externalized the Jira issue schema into a reference file to be able to update the ticket in a single tool call without needing to fetch metadata every time.

## More Hallucinations?

Despite these improvements, I still found strange results as I continued using the skill. It was still confidently incorrect in several runs. It turned out that there was ambiguity in the business logic itself. 

I found that Claude was still very eager to produce correct results. When it found conflicting business logic, it didn't know which rule to prioritize. For example, there may have been a PR that contributed to an issue, but perhaps a separate system was misconfigured when both needed to be changed together. The root cause here isn't a simple code change. Arguably, it's a gap in tooling instead. However, these were not encoded into the skill, so Claude was confident but wrong. I added more graceful degradation: when the rules conflicted, stop and ask rather than guess forward. This also made me think of adding confidence scoring which we'll touch on in a later post.

Eventually, we realized that many incidents genuinely didn't fit a clean category and that measuring a complex process isn't as trivial as we'd thought. There were issues like conflicting definitions, edge cases nobody had thought through, and even dual root causes. This was a clear sign that we'd outgrown our existing process and that we needed to evolve. This automation gave us the data to prove that.

## Production Readiness

My final battle was with Claude's web UI; it was unreliable. I saw network errors mid-session, forgotten turns, and retries that duplicated work. When a turn including a comment posting tool call failed and I used the retry button, it posted a duplicate comment. I added idempotency checks. When sessions broke partway through, the skill had lost track of what it had done. I added a simple JSON state file in the chat to track progress across interruptions. Simple tweaks that would help guide the LLM's behavior to see how they fared.

## What the Process Taught Me

The skill defined a new standard: gather context first, make all inferences, get human approval, then write to Jira. In that order, always. Context gathering and updating business systems needed human checkpoints.

The unexpected side effect was that running the process faster made the ambiguities in our metrics definitions visible. When I was doing this by hand, edge cases got resolved in the moment and forgotten. When a skill had to make the same judgment repeatedly, the inconsistencies surfaced. I ended up in stakeholder conversations about what we actually wanted to measure. These conversations led to new fields and aligned definitions across teams.

We didn't just automate the process. It forced us to externalize tribal knowledge and standardize it across the team. We were, in effect, building employee handbooks that we should've had long ago.
