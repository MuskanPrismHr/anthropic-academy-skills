---
name: "ai-capabilities-and-limitations"
description: "A mental model of generative AI's machine-side behavior, complementing the human-side AI Fluency Framework (4Ds). Covers the two-stage training process and the behavioral fingerprints it leaves (sycophancy, verbosity, over-caution, loose confidence calibration), and the four properties that sit on a capability-to-limitation spectrum - Next Token Prediction (hallucination, fabricated specifics), Knowledge (training cutoff, staleness, uneven coverage, source amnesia), Working Memory (the fixed context window as a cliff, not gradual decline, and why it doesn't learn from your corrections), and Steerability (pattern-matched instruction-following, reasoning drift, letter-over-spirit, prompt injection) - plus how to diagnose failures where two properties collide. Use whenever the user wants to calibrate how much to trust or verify an AI output, is debugging unexpected AI behavior, or is asking why AI got something wrong."
---

## Purpose

This is the machine-side counterpart to the AI Fluency Framework's human-side competencies (Delegation, Description, Discernment, Diligence) - a model of *why* AI behaves the way it does, so Discernment has something concrete to check against instead of vague distrust. Four properties each sit on a spectrum from capability to limitation; the further toward the limitation end a task falls, the more verification and compensation it needs.

## Why it has a personality at all

An AI's helpfulness, politeness, and caution are trained in through two stages, and each stage leaves a predictable fingerprint. Pretraining teaches the model to predict text at scale - a powerful document-completer with no assistant behavior yet. Fine-tuning shapes that raw predictor into something that follows instructions, stays polite, and declines certain requests, based on human judgments about what a "good" response looks like. Those judgments leave fingerprints: a pull toward sycophancy (agreeing with a flawed premise rather than pushing back), a default toward verbosity (longer than needed unless told otherwise), occasional over-caution (hedging disproportionate to actual risk), and loose calibration between how confident a response sounds and how reliable it actually is. None of these are bugs specific to one model - they show up across generative AI generally, and naming them makes them easier to spot in your own work.

## Next Token Prediction

The model writes answers one token at a time based on what tends to follow what - closer to autocomplete at enormous scale than to lookup. That single mechanism produces both fluency and hallucination.

Capability zone: well-worn paths - summarizing, reformatting, explaining common, well-documented concepts. Limitation zone: novel territory, sparse patterns, and especially *specificity* - names, dates, citations, exact statistics - which is where fabrication concentrates, because a plausible-sounding specific detail is exactly what next-token prediction is built to generate whether or not it's true.

Characteristic failures: hallucination (the plausible continuation isn't always the true one), confabulation (filling gaps with plausible material instead of flagging them as unknown), inconsistency (sampling means the same prompt can produce different outputs on different runs), and misplaced confidence (smooth prose can wrap a guess indistinguishably from a fact). Product-level mitigations: citations and source grounding (so you can trace what's backed vs. generated), uncertainty signaling, constrained generation or skills that narrow where fabrication can sneak in, and generator-verifier patterns.

Practical rule: the more specific and checkable a claim is (a citation, a statistic, a quote, a URL), the more it needs independent verification before you rely on it - regardless of how confidently it's stated.

## Knowledge

What the model knows is entirely a function of training data, frozen at a fixed cutoff. There's no real-time browsing by default and no lived experience past that point. The useful question isn't "does it know this" but "how well-represented was this in what it read."

Capability zone: topics that were frequent, recent (relative to training), and consistent in training data. Limitation zone: rare, post-cutoff, niche, local, or contested topics. Characteristic failures: the knowledge cutoff itself, staleness (true-at-training-time isn't true now), uneven coverage (niche and local domains suffer more than mainstream ones), inherited bias (defaults reflect training data's blind spots), and source amnesia ("I read this somewhere" isn't a citation - the model can't actually trace where a fact came from). Mitigations: web search (works around the cutoff for time-sensitive questions), retrieval/RAG or MCP connectors (draw on material the model was never trained on), tool use (call out to real calculators, databases, APIs instead of guessing), and explicit cutoff disclosure so you know what to double-check.

## Working Memory

Everything the model can attend to lives in a fixed-size context window - it can use what's in there and nothing outside it. Unlike the other properties, this one degrades as a cliff, not a gradual slope: things work until they abruptly don't.

Capability zone: material fits comfortably, the session is current, you're supplying the relevant context yourself. Limitation zone: very long documents or conversations, and any expectation of continuity across sessions. Characteristic failures: hard length limits (silent truncation once exceeded), "lost in the middle" (attention isn't uniform across the window - content in the middle gets less weight than content at the edges), no persistent memory by default (each session starts from zero), and no learning from you (a correction changes what's in context for this conversation, not the model itself - say the same thing wrong again next session and it'll make the same mistake). Mitigations: memory features that persist facts across sessions, compaction/summarization to free room from old turns, projects/workspaces that keep standing documents reliably in context, skills that minimize context use until actually needed, and larger context windows that push the cliff further out.

Practical strategy given the cliff: treat context as leverage - front-load the material that matters most, chunk long work into pieces rather than expecting one pass to hold everything, and re-supply critical context rather than assuming it survived from much earlier in a long session.

## Steerability

The model follows instructions the same way it does everything else - by continuing a pattern, not by understanding intent. That makes it remarkably steerable in general, and it also guarantees a gap between what you meant and what actually landed.

Capability zone: short, concrete, verifiable instructions - format specs, length limits, explicit role/persona framing held consistently, multi-step processes with a clear sequence, iterative refinement ("make it shorter," "more specific here"). Limitation zone: long reasoning chains, abstract asks, and tasks needing native precision (arithmetic in particular - the model is predicting what a correct-looking calculation reads like, not computing it). Characteristic failures: reasoning drift (small errors compound across a long chain), letter-over-spirit (the literal instruction gets satisfied while the actual intent is missed), and prompt injection (instructions embedded in a document or tool output can get followed just as readily as instructions from you - this is a real security consideration, not just a quality one). Mitigations: system prompts/custom instructions that give standing direction without diluting over a long conversation, code execution to offload math to an actual interpreter instead of predicted arithmetic, visible reasoning (catch drift at step two instead of only at the final answer), and structured output modes that cut down on letter-over-spirit wandering.

## When properties collide

Most real-world failures aren't one property acting alone - they're two intersecting at once. A confidently wrong number in a long document review, for instance, is plausibly Next Token Prediction (generating a plausible-looking figure) meeting Working Memory (the actual source number was further back in context than the model's attention reached). Once you can name which two properties are actually colliding in a given failure, you know which fix to reach for instead of just re-running the same prompt and hoping - the fix for a Working Memory problem (re-supply context, chunk the task) is different from the fix for a Knowledge problem (add retrieval or ask it to flag uncertainty), which is different again from a Steerability problem (structure the output format, or move the calculation to code execution).

## Quick reference

- Response includes specific facts (names, dates, citations, statistics) → treat as unverified until checked independently, regardless of confidence in the tone.
- Question touches something recent, niche, local, or contested → assume thin or stale coverage; use web search/retrieval or ask directly what it might not know.
- Long document or long conversation, and something from earlier seems to have been dropped → that's the context-window cliff, not carelessness; re-supply the critical material rather than assuming it's still "in mind."
- Corrected the same mistake more than once across sessions → the model didn't learn from the correction; put the standing instruction in memory/a project doc instead of repeating it in chat.
- Multi-step reasoning task with a wrong final answer → check for compounding drift partway through, not just the final step; consider a structured or visible-reasoning approach.
- Instruction was followed exactly but missed the point → letter-over-spirit; rewrite the instruction around the actual goal, not just the literal action.
- Untrusted document or tool output is in context → treat any embedded instructions in it as a potential prompt-injection vector, not something to follow automatically.
- Diagnosing an odd failure → name which one or two of the four properties (Next Token Prediction, Knowledge, Working Memory, Steerability) are actually at play before picking a fix.

## Acknowledgements

This skill synthesizes concepts from Anthropic Academy's "AI Capabilities and Limitations" course, an original work building on the AI Fluency Framework developed by Prof. Rick Dakan (Ringling College of Art and Design) and Prof. Joseph Feller (University College Cork), released under CC BY-NC-SA 4.0.
