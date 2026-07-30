---
name: "cowork-fundamentals"
description: "Reference guide to Anthropic's Cowork mode: what makes it different from turn-by-turn chat and from agentic coding tools, setting up a working folder and connectors, the permissions model, the three patterns of work that suit it (multi-step, file-based, multi-tool), how to write a good delegation prompt and answer clarifying questions, scheduled tasks and running Cowork in the cloud, global instructions, projects, skills, plugins, Claude in Chrome, Claude for Microsoft 365, working safely, validating skills with evals, and sharing plugins across a team. Use whenever the user asks how to use Cowork effectively, how to set up a folder/connector/project/skill/plugin, how to write a delegation prompt, whether a task is a good fit for Cowork versus chat, or how to work safely and share workflows with a team — even if they don't name these features explicitly."
---

## Purpose

Condensed reference to Anthropic's "Introduction to Claude Cowork" course. Use it to help the user set up Cowork, decide whether a task fits it, write a good delegation prompt, or configure standing context (global instructions, projects, skills, plugins). Pull the relevant section rather than repeating everything.

## What Cowork actually is

Cowork is a mode of Claude, inside the desktop app (or in the cloud, in beta on eligible plans), built for the work itself, not just an answer. The core mental model: **Cowork is about delegating, not chatting.** Chat is still where you think out loud, draft, brainstorm, and ask — Cowork adds the ability to hand Claude an entire piece of work (context-gathering, analysis, file production, tool use) and have it come back completed.

**Chat vs. Cowork vs. Code — three different shapes of work:**
- **Chat** — turn-by-turn dialogue. Asking questions, exploring ideas, drafting something you'll polish yourself, anything that fits a single response. You stitch multi-tool steps together yourself.
- **Cowork** — a workspace. Point Claude at a folder and your tools, describe an outcome, and it plans, executes, and delivers. Best for multi-step work across files and tools, producing real deliverables (docs, decks, sheets), and work you want to set in motion and check on later.
- **Code** — a full development environment inside a codebase, with terminal and git access. Developer work, not document work.

The biggest mistake new users make: treating Cowork like Chat — typing a question, seeing what comes back, iterating turn by turn. That works, but the value is in reaching for it when you'd normally do the work yourself, not when you'd normally just ask about it.

**How Cowork works in your environment:**
- On your files — reads what's in the folder you point it at, writes finished outputs back.
- In your apps — pulls context from connected email, calendar, messaging, drive, CRM.
- In your browser — Claude in Chrome reads and acts on pages for tools with no connector.
- With your tools — takes action, doesn't just describe what to do.

## Setting up Cowork

Install from claude.com/download (Mac/Windows), sign in, choose Cowork (requires a paid plan and a recent app version).

**Pointing at a folder** is the single most important setup choice per task — it's where Claude has read *and* write access (the core difference from Chat, where Claude can read uploads but can't save anything back). Choose the smallest folder that holds what the task needs, not a catch-all like Documents or Downloads — you can always add another folder later. Cloud-connector behavior varies: many defaults (Google Drive, M365) are read-and-search only; check each connector's description for what it can actually do. A cloud session reaches your connectors, not your local folders — start from the desktop app when the files live on your machine.

**Connectors** link Claude to the apps where work already happens — set up once in Customize, toggled per task. Common first connectors: email/calendar (Outlook/M365 or Gmail), messaging (Slack/Teams), cloud storage (SharePoint/OneDrive/Drive/Box), and CRM/project tools (Notion, Salesforce, HubSpot, Asana, Linear). Once on, reference them naturally in prompts ("check what the team said in Slack about the launch").

**Permissions model** — two modes:
- **Ask before acting** (default) — Claude pauses for approval before actions that touch the outside world: sending, posting, sharing.
- **Act without asking** — no pause for those actions; faster but riskier, use only for trusted tools/tasks.

In *both* modes, Claude always asks before permanently deleting a file — no exceptions. You also control which connectors/MCPs Claude can reach and how often each asks, plus web access and which sites Claude in Chrome can visit.

## The three patterns of work that suit Cowork

1. **Multi-step** — the task requires several steps (gather, compare, draft, format) that you'd hand off as one arc. E.g., triaging a week of feedback emails into themes with example quotes; pulling figures from several reports into one dashboard.
2. **File-based** — the objective is a real artifact (Word doc, spreadsheet, deck, PDF) built from real inputs already in your folder, edited and saved back in place. Chat can produce a new document; Cowork works on the files you already have.
3. **Multi-tool** — the work spans systems you already use (Gmail, Slack, M365, calendar, CRM) via connectors, with Cowork planning across them and running the whole sequence as one delegation instead of you stitching prompts together.

When a task shows one or more of these shapes, it's a good candidate to hand off.

## Delegating a task well

**A good Cowork prompt does three things:**
1. **Names the deliverable** — format and length specifics ("a one-page brief," "a slide for the QBR") save a regenerate.
2. **Names the inputs** — which folder, channels, date range, application.
3. **Names nuance** — the expertise/context Claude can't guess ("account for the 3 new locations we opened in Q3," "lead with the recommendation," "flag anything we can't verify").

**Clarifying questions** — Claude asks before committing, to close context gaps up front rather than mid-task. Answer specifically; most are quick-pick options, but you can answer in your own words if none fit.

**Steering mid-task** — watch the plan/progress panel as it works. If it's heading the wrong direction (wrong source, format, tone), interrupt immediately rather than waiting for it to finish and regenerating — the cost of a redirect is low, and Cowork picks up from where it was.

**Reviewing the finished deliverable** — treat it like a draft from someone you trust but don't yet fully know:
- Does it meet the actual objective, or something subtly different?
- Are the facts accurate? Ask Claude which docs it pulled from, then verify yourself.
- Does anything sound made up? A specific date/name/quote you can't trace to a source is a red flag.

If the draft is mostly right, say what to change rather than starting over — Claude edits faster than it regenerates. If it's wrong in a load-bearing way, the prompt was missing the load-bearing context — supply it and ask for adjustments.

## Scheduled tasks and Cowork in the cloud

**Scheduled tasks** run on a cadence you set. Two ways to create one: type `/schedule` and describe the task + cadence from scratch, or do the task once, confirm the output, then `/schedule` to turn that exact process into a recurring one.

Where it runs depends on where it was created and stays there:
- **Desktop-created** — runs when your computer is on; if it's off/asleep at the scheduled time, Claude catches up when you're back and flags the delay.
- **Cloud-created** — runs in the cloud at its set time regardless of whether your computer is on, working through cloud connectors rather than local folders.

Good scheduling candidates: a Friday review of what shipped, a monthly metrics roll-up, a morning briefing drawing on calendar + email + notes.

**Cowork in the cloud** (beta, eligible plans) lets you start a task from your phone or any browser, and check on or pick up the deliverable later. Two things to know: a cloud session reaches connectors, not local folders (start from desktop if the task needs local files); and a session stays on the surface where it started — start a new one to change where it runs.

## Four building blocks that compound: global instructions, projects, skills, plugins

Each gives Cowork a different kind of durable knowledge, and they're independent but compounding:

| Block | What Cowork learns | What it unlocks |
|---|---|---|
| Global instructions | Who you are and how you work | Every task starts calibrated to your role, formats, preferences |
| Projects | The context of one stream of work | Cowork works like someone already on the team inside that project |
| Skills | How a specific process should be done | Cowork runs matching tasks your team's way — templates, standards, steps |
| Plugins | The expertise of a role or field | Cowork goes from generalist to specialist for that function |

You won't set up all four on day one — global instructions and projects come first, skills emerge once you notice you're re-explaining the same workflow, plugins once a process is worth sharing.

### Global instructions

A standing brief, written once in Settings → Cowork → Global instructions, referenced in every session (chat and scheduled tasks alike). Include: who you are and what you do; shorthand/acronyms you use; how you like output delivered (format, length, tone). Don't overthink the first draft — a useful starter shape: *"I'm a [role] at [company], working on [1-2 streams]. Common shorthand: [...]. Most deliverables are [docs/decks/briefs]. I want updates concise — lead with the recommendation, background in one paragraph."* Refine it over time as you notice repeated corrections.

### Projects

A workspace scoped to one stream of work (a customer, a recurring deliverable, a launch). Contains:
- **Instructions** — scoped to just this project.
- **Scheduled tasks** — recurring runs that belong to the project, run with its context.
- **Context** — folders/links every conversation in the project can draw on.
- **Memory** — what Claude learns automatically from conversations inside the project (the one thing you don't write yourself — this is what makes a project different from a bare folder: outside a project, each session starts fresh apart from global instructions; inside one, every conversation adds to what's known).

Good project candidates: a client/account, a recurring deliverable (monthly report, weekly update), a launch/initiative. **Three ways to start one:** from scratch, from an existing folder (becomes the working directory), or migrated from a Chat project (one-way — changes in Cowork don't sync back to Chat).

### Skills

A reusable playbook — a folder that teaches Claude how to do a specific kind of recurring work the way you want it done. Claude loads it automatically when a task matches (no need to invoke by name, though you can be explicit). Four kinds of files a skill can bundle, used in whatever combination the process actually needs:
- **Instructions (SKILL.md)** — what the skill does, when to use it, how to do it — written like a runbook for a new colleague.
- **Assets** — logos, templates, slide masters, fonts: raw materials for real-looking output.
- **References** — examples of good output, style guides, clause libraries — how Claude learns what "good" looks like.
- **Scripts** — small pieces of code for the parts of the process that must run identically every time (a variance calc, a chart formatter).

**Building one:** start a Cowork conversation and say "I want to build a skill for [the recurring process you keep re-explaining]. Walk me through what you need to know." Claude interviews you (what it should do, when it triggers, what good output looks like, what resources to draw on) and produces an installable skill folder. Iterate by giving Claude a correction and asking it to update the skill in place. Skills work identically inside a project — a skill built for variance analysis fires whenever variance analysis is the task, project or not.

### Plugins

A packaged set of skills built around a job — skills plus the connectors and subagents they depend on. (A subagent here is a purpose-built helper a skill can spin up for one part of the work in its own context, e.g., a research subagent for a research step.) Two shapes:
- **End-to-end process** — sequential skills packaged as one pipeline (e.g., a monthly-close plugin: pull actuals → build variance table → draft board memo).
- **A function's toolkit** — a team's most-used, independent skills bundled together (e.g., a finance plugin: variance analysis, modeling, investment-memo drafting, quarterly reports).

Anthropic publishes plugins for common roles (finance, legal, sales, marketing, support, PM, etc.) in the marketplace — find them at Customize → Plugins, install, approve the connectors it uses. **Customize an installed plugin** via Customize → Plugins → [name] → Customize, a Cowork task where you point Claude at your own examples/templates to adapt it ("update the /nda-triage skill so tone/format match these three red-lined NDAs"). **Build your own** from scratch by working with Cowork if nothing existing fits — most teams start with one skill, then add more until it's plugin-worthy. Try `/setup-cowork` for a short interview that recommends a plugin for your role. Check your org's Directory before building — an admin may have already published something.

## Claude in Chrome (extending beyond the desktop)

The bridge for any browser-based tool with no connector: internal dashboards, vendor/customer portals, web apps behind a login. Claude reads and acts on pages directly, then hands results back into the same Cowork conversation to build the deliverable — one delegation, multiple surfaces (e.g., pull yellow/red accounts from a dashboard in Chrome, then pull Drive/Slack context and build a one-page summary in Cowork).

**Watch-outs:** you must be signed in yourself — Claude can't authenticate for you, it works in your existing session. Be deliberate about what you grant access to: on the open web, Claude sees what you see, including anything you're logged into — narrow scope and review actions for sensitive sites.

## Claude for Microsoft 365

Claude lives inside Word, Excel, PowerPoint, and Outlook as an add-in, working on the file/space you have open — and one conversation can carry context across apps (Outlook brief → Word memo → Excel model → PowerPoint deck → back to Outlook for scheduling).

**Rule of thumb:** reach for **Cowork** when work pulls from many sources and ends in a deliverable (building a brief from 20 files, a report from Salesforce + 3 Slack channels, a scheduled workflow). Reach for **Claude in M365** when you're editing in place inside the Office file itself and carrying context between apps. Most real work uses both — Cowork builds the first draft, M365 refines it in place.

## Working safely

Guardrails are constant regardless of mode: Claude always asks before permanently deleting; in "Ask before acting" (default), it also asks before sending/sharing/posting. What you add on top:

- **Use a dedicated working folder, not a catch-all.** Pointing Claude at Documents/Downloads/Desktop is like letting a new colleague rummage through everything you own.
- **Back up anything irreplaceable before starting** — old deliverables, unre-issuable contracts — somewhere Cowork can't reach.
- **Test new workflows on copies first**, especially before pointing a scheduled task at live data.
- **Be specific about destructive verbs** — "cut the section" and "update the file" are ambiguous between "remove from view" and "delete." Say what you mean: "remove from the draft but keep the file."
- **Name the bounds in the prompt** — "only the 3 most recently updated files," "don't message anyone, draft only" — this narrows scope and gives you a clear line for spotting drift.
- **Use scheduled tasks for drafts initially** until you trust the task's behavior, rather than letting it send/post unsupervised.
- **In the moment:** read the plan once it's made (does it make sense, right sources, right order); watch for unexpected patterns (files/sites you didn't mention, scope creep — "something feels off" is a real signal, stop the task); approve confirmation prompts deliberately (most mistakes happen from clicking through a confirmation that wasn't quite the intended action, not from a safeguard failing).

**When Cowork isn't the right tool:** regulated workflows needing an audit trail (Cowork activity isn't captured in audit logs/Compliance API/data exports); anything you wouldn't trust a smart, quick colleague to do unsupervised (sending a legal doc to a counterparty, posting a public announcement, a customer-facing change — Claude prepares, you ship); highly sensitive personal data outside an IT-approved boundary.

## Validating skills before relying on or sharing them

A skill or plugin is a small product other people will use — worth a test drive first, the same way you'd sanity-check a template or model before handing it off. **Evals** (evaluations) are just a try-out: a realistic prompt goes in, you look at the output, you tell Claude what to fix — no code required.

**How skill-creator runs it:** produces 2+ realistic prompts, and for each one, a pair of outputs — one *with* your skill, one *without* — so you can see what difference the skill actually makes, not just whether the output is okay in isolation. For each pair, ask: is this the version I'd send? If not, what specifically is missing or off ("the tone is too formal," "it skipped the executive summary" — specific, actionable feedback, not "this isn't quite right").

**Iterate:** submit feedback, Claude revises the skill, re-run the same prompts, change one thing at a time so you can tell what moved the needle. Most skills are ready after one or two rounds. The bar isn't perfect evals — it's that the cases you care about pass meaningfully better than the no-skill baseline, and you've explicitly named the cases you don't yet handle.

## Sharing plugins across a team

Once a skill has earned its place (run through evals, holds up beyond one person's use case), bundle it into a plugin and hand it to whoever owns your org's private marketplace (team lead, enablement/ops owner, or IT). They choose how it lands:
- **Available** — appears in the company Directory, install if wanted.
- **Installed by default** — already present when Cowork opens; can be turned off.
- **Required** — installed and stays on (e.g., mandatory compliance checks).
- **Hidden** — in the marketplace but not shown in the Directory (staging/restricted rollouts).

Teammates see it in their Directory labeled as coming from the company, alongside public Anthropic plugins — they can use/disable it (unless required) but can't edit it; updates flow from the maintainer.

**Habits that keep a shared plugin healthy:**
- **One named owner** who reviews changes, re-runs evals after edits, decides on updates/retirement.
- **Evals before every publish** — treat the eval loop as the actual gate.
- **Specific names** — "sales-customer-renewal-prep," not "meeting-prep," which will collide with other teams' plugins.
- **A regular review rhythm** (quarterly is reasonable) — retire what nobody uses, improve what people flag.

## Quick reference: what to reach for

- Quick question, brainstorm, draft you'll polish yourself → **Chat**.
- Multi-step, multi-tool, or ends in a real file/deliverable → **Cowork**, with a prompt naming deliverable + inputs + nuance.
- Recurring cadence (weekly rollup, morning briefing) → a **scheduled task**.
- Need to kick something off or check on it away from your desk → **Cowork in the cloud**.
- A tool with no connector but a browser interface → **Claude in Chrome**, then hand results to Cowork.
- Editing a file you already have open in Word/Excel/PowerPoint/Outlook → **Claude for Microsoft 365**.
- Repeated correction to your global preferences → add it to **global instructions**.
- A recurring stream of work (client, deliverable, launch) → set up a **project**.
- Re-explaining the same process more than once → build a **skill**.
- A skill (or a few) worth handing to the whole team → bundle into a **plugin**, validate with **evals**, then distribute via the org marketplace.

