---
name: "ai-fluency-for-builders"
description: "Reference guide to applying the AI Fluency Framework (4Ds) to building software with AI, for engineers, designers, and product people who own the full arc from customer problem to shipped solution. Covers the Builder's Toolkit (six capabilities from empathy to shipping, mapped to Automation/Augmentation/Agency delegation modes), writing acceptance tests before code, the Description Chain that translates a user need into a precise AI instruction, the Five Lenses of Discernment for evaluating AI-generated code (functional integrity, production readiness, problem fit, experience quality, responsible impact), the four UX principles AI gets wrong by default (clarity, hierarchy, accessibility, feedback), and what full ownership means at ship time. Use whenever the user is building a product/feature with AI and asking how to scope delegation, write an AI-executable spec, evaluate AI-generated code or UI before shipping, or what they own once it's live."
---

## Purpose

Builders (engineers, designers, product people) own more of the arc than most AI-fluency advice accounts for - not just "did the code run" but whether the right thing got built, whether it holds up under real use, and whether you can stand behind it after it ships. This skill applies the 4D framework (Delegation, Description, Discernment, Diligence) to that full arc, with the core insight that as AI accelerates implementation, your value shifts toward the parts of the toolkit AI can't do: framing the problem, exercising judgment, and deciding what "done" actually means.

## The Builder's Toolkit and where AI fits

Before writing code, a build passes through six capabilities, and AI's usefulness varies sharply across them: Empathy (understanding who you're building for - AI can surface data and personas, but can't feel the gap), Design, Architecture, Implementation (where AI is strongest), Judgment, and Shipping. Map each capability to a delegation mode rather than asking "should I use AI" as a single yes/no: Automation (AI does it, you check), Augmentation (you and AI work it together), or Agency (AI operates with latitude inside boundaries you set). Delegating implementation is usually safe. Delegating judgment usually is not - most AI failures in a build trace back to a description, discernment, or diligence failure made earlier, not to a coding mistake.

## Write acceptance tests before any code exists

Decomposing the problem means producing a plan before implementation starts, not during it. Write acceptance tests as five to seven concrete pass/fail statements - "a user with a basic smartphone can find the current status in under 30 seconds without creating an account" beats "the system is easy to use." A test a stranger could actually adjudicate is what gives you and AI a shared definition of done; a vague one just defers the disagreement to later, when it's more expensive to fix.

## The Description Chain

Prompt engineering is one link in a longer chain: user voice → product requirement → technical spec → AI instruction → tests. The builder is the translator at every step - AI cannot hear what the user did not say, and each translation involves a judgment call the previous step didn't make for you. When code works but the product doesn't, that's a description failure, and the fix is to trace back through the chain to find which link actually broke rather than patching the symptom where it surfaced.

Practical discipline at each link: in the product requirement, make every adjective a decision you can defend - if you write "fast," say how fast, in what condition. In the technical spec, name the pieces and how they talk to each other, and use AI in augmentation mode to propose an approach you then push back on against your actual constraints. Tests are the most precise form of description available - a test suite that passes while a real user is unsatisfied means you described the wrong intent, not that the code is broken.

## The Five Lenses of Discernment (for code)

"Working" stops being the bar once AI can produce a working version in minutes. Evaluate AI-generated code across five lenses, each harder to check mechanically than the last:

1. **Functional Integrity** - does it work? Produces correct output for real inputs, not just test data. Common AI failure: code that passes unit tests but breaks on real data the prompt never covered.
2. **Production Readiness** - does it work well? Handles concurrency, load, and the real infrastructure stack. Common AI failure: smooth in development, broken under load or behind a real deployment.
3. **Problem Fit** - is it the right thing? Solves the user's actual need, not just the literal spec. Common AI failure: a technically complete feature that addresses the prompt but misses the underlying need.
4. **Experience Quality** - is it good? Users can complete the core task without help or instruction. Common AI failure: generic UI patterns that technically work but leave users confused or uninvested.
5. **Responsible Impact** - is it responsible? Transparent about AI's role, doesn't present generated content as verified fact. Common AI failure: AI-generated content presented as authoritative, or a demographic left out by default.

Lens 1 is easy to test mechanically - run it and see. By lens 5, you're making judgment calls AI can't make for you; that's the taste AI doesn't have, and building it is a builder skill, not a one-time checklist.

## The four UX principles AI gets wrong by default

"Make it look good" is a wish, not a spec - describe experience with the same precision you'd use for a function. Four principles matter most when specifying UI for AI to build:

- **Clarity** - every element should instantly communicate its purpose; users shouldn't have to guess what a button does or a field means.
- **Hierarchy** - visual weight should match information priority; the most important thing should look the most important, not just appear first in a flat list.
- **Accessibility** - roughly one in five people has a disability (visual, cognitive, or neurological), and AI generates for the median user by default. Small text, color-only signals, and unexplained jargon quietly exclude people unless accessibility is specified up front and then audited in what comes back - it is not something AI gets right unprompted.
- **Feedback** - when something breaks, users need to know what happened, what to do next, and how to get help; a raw error code answers none of those questions.

A design critique and an AI-executable description are different artifacts - "this feels confusing" is a critique; "the primary action needs the highest visual weight on the screen, and the error state needs to name what went wrong and offer a next step" is a description AI can act on. Learning to translate between the two is the actual skill.

## Diligence: full ownership at ship time

In the builder model, Diligence means you own the outcome, not the output - "AI wrote it" explains nothing and excuses nothing once something is live. Shipping surfaces its own technical vocabulary AI will not proactively raise unless you ask about it directly: migrations, versioning, rate limits, feature flags. Before deploying, work through five questions honestly: do you understand what your code actually does, not just what it's supposed to do; do your acceptance tests still pass, including edge cases you added after first contact with real use; who does this build not serve well (access is a design decision, not an afterthought); could the output be misread or misused, and have you been transparent about AI's role; and how will you know if it's working after it ships.

Tests are what make post-launch iteration safe - the test-first habit up front is precisely why you can keep changing things confidently later without fear of silent regressions. Prototype freely, but ship selectively: cheap-to-generate code only creates value when paired with an honest evaluation of whether it's actually good enough, and being willing to veto or deprecate something you built when the evidence says it isn't working is an underrated builder skill in its own right.

## Quick reference

- Starting any AI-assisted build → write a reusable brief (what you're building, your role, your constraints) so you're not re-establishing context every session.
- Deciding what to delegate → map the six toolkit capabilities to Automation/Augmentation/Agency individually; don't treat delegation as one yes/no decision for the whole project.
- About to write a prompt for a feature → check the full Description Chain first (user need → requirement → spec → instruction → tests); a bug that "shouldn't be possible" is usually a broken link upstream, not a coding error.
- Reviewing AI-generated code before merging → run it through all five lenses, not just "does it run" - problem fit and responsible impact are where AI-built products most often fail unnoticed.
- Reviewing AI-generated UI → check clarity, hierarchy, accessibility, and feedback explicitly; AI defaults to the median user and will not get accessibility right without being told and then audited.
- About to deploy → answer the five diligence questions (understanding, testing including edge cases, access, responsibility, feedback loop) before shipping, not after.
- Something is technically complete but feels off → that's very likely a Problem Fit or Experience Quality gap (lenses 3-4), not a Functional Integrity bug (lens 1) - look upstream in the Description Chain rather than just re-running the same prompt.

## Acknowledgements

This skill synthesizes concepts from Anthropic Academy's "AI Fluency for Builders" course, developed in partnership with CodePath, building on the AI Fluency Framework by Prof. Rick Dakan (Ringling College of Art and Design) and Prof. Joseph Feller (University College Cork), released under CC BY-NC-SA 4.0.
