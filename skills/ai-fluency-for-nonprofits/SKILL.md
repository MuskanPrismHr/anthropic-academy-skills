---
name: "ai-fluency-for-nonprofits"
description: "Reference guide to applying the AI Fluency Framework (4Ds) at a mission-driven nonprofit with limited staff time and sensitive stakeholder data. Covers funder/policy research with the Description-Discernment loop, grant and donor writing that preserves organizational voice and authenticity, matching tool sensitivity to donor/beneficiary PII and sanitizing data before sharing it with AI, validating AI-assisted data analysis against known-answer data before trusting it on new data, deciding what workflow tasks to automate versus keep human (and testing automations on real examples before relying on them), and drafting an organization-wide AI policy that keeps a human in the loop without creating quiet dependency. Use whenever the user works at or advises a nonprofit, NGO, or other mission-driven organization asking how to use AI for grants, donor communications, program data, or team-wide AI policy."
---

## Purpose

Nonprofits face two constraints most for-profit AI-fluency advice ignores: almost no spare staff time to double-check AI output, and routine handling of sensitive donor and beneficiary data. This skill applies the 4D framework (Delegation, Description, Discernment, Diligence) to that reality - use it whenever AI use touches funder relationships, program data with real people's information in it, or a small team's day-to-day workload.

The framework splits into two loops that map onto different decisions: Delegation-Diligence handles the higher-level "should we use AI here, and are we accountable for it" question; Description-Discernment handles the day-to-day "how do we get a good result from this specific interaction" question.

## Researching funders, policy, and compliance

Context-rich prompting matters more here than almost anywhere else: state who you are, who you serve, and what you specifically need to know, rather than asking a broad question. A funder-research prompt should include your mission and the specific program seeking funding, your organization's characteristics (budget, geography, populations served), the funding parameters you need, and what "good fit" means beyond topic overlap (values alignment, giving history, accessibility).

Discernment isn't optional on the output: check whether suggested funders actually fund organizations of your size and type, verify current deadlines and eligibility (funder information goes stale), and flag anything that reads as outdated. Treat the first response as a starting point, not an answer - use what it gets wrong or misses to write a sharper follow-up. AI accelerates orientation to a funding landscape; it doesn't replace the judgment call on whether a prospect actually fits your mission.

## Writing grant proposals and donor communications

Upload examples of your own successful past writing so AI matches your organization's actual voice, rather than starting from a generic template. Then feed it what only you know: your track record, specific partnerships, staff expertise, and real community relationships - the details that make a proposal credible are exactly the ones AI can't invent.

Give specific revision instructions rather than vague ones: "add these details about our relationship with the Riverside coalition" gets you somewhere; "make this more compelling" doesn't. You're the one deciding what stays in the final draft - AI accelerates the writing, but the substance of what the organization is claiming has to be something you can stand behind.

## Handling donor and beneficiary data safely

Different AI tools have different data-handling terms - a free consumer tool and a paid account with a strict data-retention agreement are not interchangeable, and the more sensitive the data, the more that distinction matters. Apply Problem Awareness and Platform Awareness before starting: often you can get the benefit you need without sharing anything sensitive at all, by breaking the task into smaller pieces or working with a sanitized version of the data.

Before sharing anything containing donor or beneficiary information, ask whether the actual goal requires the identifying details at all - for most pattern-level analysis, it doesn't. Replace names with generic identifiers, strip contact information, generalize or drop location detail, and consider whether exact dollar amounts need to be exact or could be ranges. If removing identifying information limits what AI can help with, that's a signal to provide more non-identifying context rather than to put the PII back in.

## Validating AI for data analysis

Don't hand AI a new analytical task on faith - test it first against past data where you already know the correct answer. If it can reproduce known results with the right guidance, that's the evidence you need to trust it on similar future analysis; if it can't, that's where the capability gap is. Each round of testing should teach you something specific about what AI does well unprompted and what additional context it needs every time - document that so the next person on your team doesn't have to relearn it.

## Deciding what to automate

The operative question is "should AI do this," not "can AI do this." Start by analyzing your actual workload - what's being asked repeatedly, what patterns show up - before deciding what to hand to AI. Routine, well-documented requests are good automation candidates; complaints, high-stakes requests, or anything requiring real judgment should stay human regardless of whether AI could technically attempt it.

Test any automation against real examples (actual emails or requests you've received) before trusting it, since gaps in your instructions only show up against real inputs. Practice all three forms of Diligence as you go: Creation Diligence (being deliberate about what you automate, not automating by default), Deployment Diligence (reviewing outputs before they reach anyone, especially early on), and Transparency Diligence (being clear with recipients that AI was involved).

## Writing an organization-wide AI policy

Being "the human in the loop" at a mission-driven organization means more than oversight - it means AI serves decisions your organization makes about its mission, not the reverse. A working AI policy needs to cover scope (what's in vs. out of bounds), accountability (what happens when AI gets something wrong, how you monitor over time), transparency (what stakeholders/funders/those you serve need to know about AI involvement, and how you disclose it in grants and reports), and values alignment (how AI use connects back to mission, and when you'll choose not to use AI even if it would be more efficient).

A practical check against creeping dependency: regularly ask "can we explain what the AI is doing in this process?" If yes, that's healthy augmentation. If the answer is no, rework the process until someone on the team can explain it end to end - that's a stronger safeguard than avoiding AI use altogether. The goal is for AI to absorb the noise so staff time goes to the parts of the work that are actually human: the personalized outreach, the relationship, the judgment call.

## Quick reference

- Researching funders or policy → give AI your mission, size, and geography up front; verify deadlines and eligibility yourself before acting on any suggestion.
- Writing a grant or donor piece → upload past successful writing for voice, supply the org-specific facts AI can't know, and give specific (not vague) revision instructions.
- About to share data with donor or beneficiary details → ask if the identifying details are actually needed for the goal; sanitize (generic identifiers, no contact info, ranges instead of exact figures) before sharing if not.
- Considering AI for data analysis → test it first on data where you already know the answer; don't trust it on new data until it's passed that test.
- Deciding whether to automate a workflow → ask "should AI do this," test on real examples, and review outputs before they go out, especially early on.
- Writing a team AI policy → cover scope, accountability, transparency, and values alignment, and check periodically whether staff can still explain what the AI in any given process is doing.

## Acknowledgements

This skill synthesizes concepts from Anthropic Academy's "AI Fluency for Nonprofits" course, itself based on the AI Fluency Framework by Rick Dakan, Joseph Feller, and Anthropic, released under CC BY-NC-SA 4.0.
