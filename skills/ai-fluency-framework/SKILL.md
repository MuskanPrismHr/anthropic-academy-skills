---
name: "ai-fluency-framework"
description: "Reference guide to the AI Fluency Framework (Dakan & Feller): the three ways people work with AI (Automation, Augmentation, Agency), generative AI fundamentals and current limitations, and the 4D competencies — Delegation (deciding what to hand to AI vs. keep human), Description (product/process/performance instructions and prompting technique), Discernment (evaluating AI output, process, and behavior, and the Description-Discernment feedback loop), and Diligence (accountability, transparency, and responsible AI use). Use whenever the user asks about AI fluency, how to divide work between themselves and AI, how to write or troubleshoot a prompt, how to evaluate or push back on an AI's output, or how to think about ethical/responsible use of AI — even if they don't name the framework explicitly."
---

## Purpose

Condensed reference to Anthropic's "AI Fluency: Framework & Foundations" course, based on the AI Fluency Framework developed by Rick Dakan (Ringling College of Art and Design) and Joseph Feller (University College Cork). Use this to help someone reason about how to divide work with AI, communicate with it effectively, evaluate its output critically, or think through the ethics of a specific AI collaboration. Pull the relevant section; don't dump the whole framework on someone with a narrow question.

## Why AI fluency, and three ways of working with AI

AI Fluency is the practical skill, judgment, and values needed to work with AI systems effectively, efficiently, ethically, and safely — not just knowing which buttons to press. Three distinct modes of collaboration, useful for naming which one you're actually in:

- **Automation** — the AI completes a defined task based on your instructions (the classic "ask, get an answer" mode).
- **Augmentation** — you and the AI collaborate as thinking and execution partners, building on each other's contributions in real time.
- **Agency** — you configure the AI to work more independently on your behalf, shaping its knowledge and behavior patterns up front rather than directing each task.

The 4D competencies below apply across all three modes — they don't map one-to-one onto Automation/Augmentation/Agency, they're the underlying skills that make any of the three go well.

## Generative AI fundamentals (what you're actually working with)

Generative AI *creates* new content (text, images, code) rather than only analyzing existing data. Three developments made modern large language models possible: algorithmic/architectural breakthroughs (especially the transformer architecture), vast amounts of digital training data, and large increases in computational power. Models learn in two stages: **pre-training** (finding patterns across enormous amounts of text) and **fine-tuning** (learning to follow instructions and produce helpful, appropriately-behaved responses).

**Current capabilities:** versatility across very different task types without retraining, maintaining conversational context, connecting to external tools.

**Current limitations:** a knowledge cutoff date, hallucinations (fluent but factually wrong output), a finite context window, and real challenges with complex multi-step reasoning.

**The framing that matters:** the most effective use pairs complementary strengths — AI's speed, breadth, and tirelessness with human critical thinking, judgment, creativity, domain expertise, and ethical oversight. Neither replaces the other; the pairing is the point.

## The 4D competencies

### 1. Delegation — deciding what to hand off and what to keep

Delegation is the strategic decision of how to divide work between yourself and AI, based on both your actual goals and a realistic read of AI's current capabilities — not a default of "give it everything" or "trust nothing."

**A working approach for a real project (breaks down like this):**
1. Get a clear vision of the finished result first — what does success actually look like, and what would make this particular project valuable to you. (Worth having a real back-and-forth about this with Claude rather than a one-shot brief — the useful insights tend to show up in the discussion, not the first answer.)
2. Break the project into its major tasks.
3. For each task, ask: what specific skills/knowledge/AI capabilities does it need? Which parts lean on uniquely human strengths (judgment, taste, domain context, accountability)? Which parts play to AI's strengths (breadth, speed, drafting, pattern-finding)? Where would genuine back-and-forth collaboration outperform either party working alone?
4. Write the resulting task-by-task delegation plan down — it's the reference point for how you'll apply Description and Discernment later.

The throughline: your expertise and judgment remain the foundation of the collaboration. Delegation isn't "what can I get out of doing" — it's "where does this actually go best."

### 2. Description — communicating what you want across three dimensions

Description is how you bridge your intention and the AI's capabilities. It has three distinct dimensions, useful because most vague requests are actually missing one of them specifically:

- **Product Description** — what you want, concretely: the output itself, its format, style, length, level of detail.
- **Process Description** — how you want the AI to get there: what method, framework, steps, or reasoning approach to follow.
- **Performance Description** — how you want the AI to behave and engage with you while working: concise or detailed, challenging or supportive, idea-focused or analysis-focused.

**Six foundational prompting techniques** (the practical toolkit for Product/Process Description):
1. **Give context** — be specific about what you want, why, and relevant background — don't make the model guess your situation.
2. **Show examples** — demonstrate the output style/format you're actually looking for; showing beats describing.
3. **Specify constraints** — format, length, and other concrete requirements, stated explicitly rather than implied.
4. **Break complex tasks into steps** — guide multi-step reasoning explicitly instead of asking for the whole leap at once.
5. **Ask the AI to think first** — give it room to reason through its approach before committing to an answer.
6. **Define the AI's role or tone** — specify how you want it to communicate, which shapes Performance as much as Product.

**The "secret weapon":** ask the AI itself to help you improve your prompt. Prompting is iterative and genuinely collaborative — expect to refine based on results rather than getting it right in one shot. The patterns that reliably work: a clear task overview, explicit format specs, explicit constraints, and relevant background — in roughly that order of impact.

### 3. Discernment — evaluating what comes back, across the same three dimensions

Discernment is the thoughtful, critical evaluation of AI outputs, processes, and behavior — the other half of the conversation Description started. It mirrors Description's three dimensions:

- **Product Discernment** — is the output itself actually good? Accurate, well-formed, fit for purpose?
- **Process Discernment** — how did the AI approach the task? Did it consider multiple angles, or lock onto one interpretation too early? Did it skip a step it should have shown?
- **Performance Discernment** — was the AI's engagement actually useful — clear, appropriately challenging or supportive, building on what you said rather than ignoring it?

Discernment isn't paranoid double-checking of everything forever — it's calibrated critical evaluation, proportionate to the stakes of what you're using the output for.

### The Description-Discernment loop

In practice these two competencies run as a loop, not a single pass:

1. **Describe** what you need (Product), how to approach it (Process), and how you want the AI to engage (Performance).
2. **Discern** what comes back on all three dimensions — the output, the approach, the behavior.
3. **Refine** — give specific feedback on what worked and what didn't, adjust the description, iterate until it's actually right.
4. **Integrate your own expertise** — add the perspective, taste, or domain knowledge that only you bring, and make the final call on what to keep, change, or discard. You stay accountable for the final output, full stop — that responsibility never transfers to the AI.

Run this loop per task rather than trying to nail an entire project in one giant prompt. The best outcomes tend to come from genuine iterative collaboration, not a single perfect instruction.

### 4. Diligence — the ethical and safety layer underneath everything else

Where Delegation, Description, and Discernment are mostly about effectiveness and efficiency, Diligence is specifically about doing this *responsibly*: accountability, transparency, and taking ownership of AI-assisted work rather than treating the AI as a scapegoat or a black box.

**Building your own working policy (useful structure for a personal or team AI-use guideline):**
- Set clear standards for when and how you'll use AI in different contexts.
- Draw explicit boundaries around sensitive or confidential information — what never goes into a prompt.
- Define how you'll maintain quality control on AI-assisted work (this is where Discernment plugs in directly).
- Name the ethical issues most relevant to your specific field or role, rather than generic ones.
- Set decision-making criteria for when an ethical question actually comes up, not after the fact.
- Consider who else is affected by a given AI interaction, not just you.
- Decide how and when you'll disclose AI involvement in different contexts, with a rough template for attribution/transparency statements and a sense of when a more detailed disclosure is warranted versus a light one.

## Putting it together

The four competencies aren't sequential lessons you finish and move past — they're the same four questions you run on every substantial AI collaboration: what should I hand off (Delegation), how do I ask for it well (Description), is what came back actually good (Discernment), and am I doing this responsibly (Diligence). Fluency here is a practiced skill, not a one-time unlock — expect it to sharpen with repeated, reflective use rather than mastery after a single course. The framework is intentionally durable: it's built to keep making sense as underlying AI capabilities keep changing, because it's about the collaboration pattern, not any one model's quirks.

## Practical exercises worth reusing

- **Personal AI fluency plan** — rate yourself (novice/developing/confident) on each of the 4Ds, pick one or two to actively develop, and set concrete practice activities and a timeline — genuinely useful as a periodic self-check, not just a one-time onboarding exercise.
- **Personal prompt library** — for 5-10 tasks you do repeatedly with AI, build a reusable template prompt for each (with placeholders for the variable parts), plus notes on which Description techniques and Discernment checks work best for that specific task type.
- **Puzzle-based practice** (riddles, cooperative crosswords, word association, twenty questions, collaborative storytelling) — because puzzles have tight, checkable answers unlike open-ended writing tasks, they're an unusually good low-stakes way to drill precise Description and sharp Discernment (product/process/performance) without the stakes of a real deliverable.
- **A personal AI policy** — write your own Diligence guidelines using the structure above, with Claude as a thinking partner to pressure-test the ethical edge cases specific to your field.

