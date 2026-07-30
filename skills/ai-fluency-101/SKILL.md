---
name: "ai-fluency-101"
description: "Reference guide to working with your AI assistant effectively: when to use turn-by-turn chat vs handing off multi-step work vs building software in a codebase, how to write good prompts, Projects, Artifacts, Skills, Connectors, Enterprise Search, and Research, plus other product surfaces (agentic coding tools, Slack integration, design prototyping, Microsoft 365 add-ins, browser extension). Use this whenever the user asks how to use the assistant better, which mode or tool fits a task, how to write an effective prompt, what a specific feature does, or wants help choosing between chat, hand-off, and coding modes — even if they don't name these features explicitly."
---

## Purpose

This skill is a condensed reference to Anthropic's "Claude 101" course. Use it to answer questions about how Claude works and to help the user pick the right tool, mode, or approach for a task. Don't dump the whole thing on the user — pull the relevant section and answer their actual question.

## The three shapes of work

The single most useful mental model: notice what kind of work is in front of you, and let that decide the tool.

| You're about to... | Shape | Where it lives |
|---|---|---|
| Ask, brainstorm, draft, or think something through, turn by turn | Working with Claude, turn by turn | Chat |
| Hand off a multi-step task that ends in a finished deliverable, spans multiple tools, or runs on a schedule | Handing work off | Cowork |
| Write, test, run, and ship code in a codebase | Building software | Code tab (Local or Cloud) |

**Chat (turn by turn):** reach for this when the answer changes what you ask next (you couldn't have written the whole request up front), when you want to stay in the loop on every turn (drafting, editing, thinking out loud), or when it's just quick. Desktop app extras: quick entry (double-tap Option on Mac), screenshots/window sharing, dictation, desktop connectors.

**Cowork (hand off):** reach for this when the task has several steps you'd normally do in sequence, the output is a real deliverable (doc, spreadsheet, deck, PDF) saved to a folder rather than pasted into chat, the work spans multiple tools, or it should run on a schedule / while you do something else. Claude still asks clarifying questions and shows its plan up front, and stops for approval before sending emails, sharing files, or other consequential actions. Capabilities: local folder access (reads and saves back to the same folder — the concrete difference from Chat), scheduled tasks, subagents (splits big jobs across parallel background workers), Projects, browser use (Claude in Chrome), computer use (research preview, Pro/Max), plugins (bundles of skills/connectors/agents for a role). Available on Pro, Max, Team, Enterprise.

**Claude Code (build software):** works directly in a codebase — reads, writes, tests, runs commands. Visual diffs, built-in terminal, git-backed rollback. Choose Local (a folder on your machine) or Cloud (a connected GitHub repo, sessions persist even if you close the app). Control how autonomous it is: Manually approve, Accept edits, or Plan mode. Lives in the Code tab, Pro/Max/Team/Enterprise.

## Writing effective prompts

Treat prompts like briefing a coworker: natural, concise, conversational. Three elements to include:

1. **Setting the stage** — your role, objectives, relevant context Claude should know.
2. **Defining the task** — the specific action: write, analyze, build, something else.
3. **Specifying rules** — tone, style, format, examples to match.

Example that uses all three: *"I'm the marketing lead at an indie streaming startup, preparing an investor pitch deck for Series A. Research the current state of the independent film streaming market and identify key trends, competitor positioning, and growth opportunities. Use current web research with citations, structured as a professional report up to 5 pages, with an executive summary, market analysis, competitive landscape, and growth opportunities."*

This is adapted from the 4D Framework for AI Fluency (Dakan & Feller): **Delegation** (what should a human vs. Claude do), **Description** (communicating clearly what you want), **Discernment** (critically evaluating outputs), **Diligence** (using AI responsibly, staying accountable).

**Iteration mindset:** the first response is a starting point, not the final answer. Give specific feedback ("cut the first two paragraphs, make the conclusion more action-oriented") rather than vague notes. If a conversation goes off the rails, it's often faster to start a fresh chat with a clearer prompt than to fight to redirect it. The pencil icon on a sent message lets you edit and resubmit instead of adding a new turn.

**Personalization:** Memory automatically retains role, preferences, and working style across chats (editable in Settings). Styles let you set a default tone/format (concise, formal, explanatory, or custom) applied everywhere.

## Troubleshooting common problems

| Problem | Likely cause | Fix |
|---|---|---|
| Response is too generic | Not enough context about the specific situation | Add audience, role, constraints — specifics instead of a generic ask |
| Too long / too short | Claude guessed at length | State it explicitly: "two-paragraph summary," "under 100 words," "comprehensive, length isn't a concern" |
| Wrong format | Claude understood *what* but not *how to present it* | Show an example, or describe the structure explicitly |
| Confident-sounding but wrong facts | Claude can generate plausible but incorrect specifics, especially on niche topics | For high-stakes info, verify independently; ask for citations/confidence; enable web search |
| Wrong tone | Claude defaults to helpful/professional | Describe the tone directly, or provide a writing sample to match |

**Simple evals for your own workflows:** gather 5–10 examples of a task you already do, write prompts that would produce something similar, run them, and compare against your originals — does Claude capture the key info, is the tone right, what's missing. Use this to build intuition for where Claude is strong versus where it needs more guidance from you.

## Projects

Self-contained workspaces with their own memory, chat history, knowledge base, and instructions — for ongoing work, not one-off questions.

**Use a project when** you have reference material you'll reuse repeatedly, consistent requirements for how Claude should respond, or a team that needs to share the same context.

**Setup:**
1. Projects (sidebar) → **+ New Project** → name it descriptively, add a description (for humans, not read by Claude directly), set visibility.
2. **Instructions** — tell Claude how to behave in every chat in this project: context, process, tone, specific requirements. Can encode small automations ("when I upload a meeting transcript, summarize using this template").
3. **Knowledge base** — upload reference docs, background material, examples, specs. Name files descriptively ("Q4-2024-Brand-Guidelines.pdf", not "document1.pdf") — Claude uses filenames and proximity to understand relationships between documents.

Large knowledge bases auto-scale via RAG (retrieval-augmented generation): once you approach the context limit, Claude switches to searching your files and pulling only what's relevant, expanding effective capacity up to 10x.

**Collaboration** (Team/Enterprise): share at **Can view**, **Can edit**, or **Owner** level; share with specific people or the whole org.

**Skills vs. Projects:** projects store knowledge (the *what*), skills define process (the *how*). A skill can pull from knowledge stored in a project.

## Artifacts

Standalone, interactive outputs rendered in a dedicated window next to the conversation — not buried in chat text. Claude creates one automatically when content is significant/self-contained (typically 15+ lines), likely to be edited or reused, and stands on its own outside the conversation. If it doesn't happen automatically, just ask: "create this as an artifact."

**Types:** documents (markdown/plain text), code snippets, HTML pages, SVG images, Mermaid diagrams, React components (interactive, with real logic). Word/Excel/PowerPoint/PDF files are a separate file-creation capability, not artifacts — they come back as downloadable files.

**Sharing:** copy/download for personal use; share within org (Team/Enterprise, requires team auth); or publish publicly (free/Pro/Max) — only the selected version goes public, others can view and "remix" it, not indexed by search engines, unpublish any time.

**Tips:** be specific about what you want (not "build a budget tracker" but the full feature list); describe the intended end user; iterate one change at a time rather than everything at once.

## Skills

Folders of instructions (and optionally scripts/resources) Claude loads dynamically for specialized, repeatable tasks. Feature preview on Pro/Max/Team/Enterprise, requires Code execution and file creation enabled in Settings → Capabilities.

**Anthropic Skills** — built-in, cover Excel/Word/PowerPoint/PDF creation, invoked automatically, no setup needed.
**Custom Skills** — created by you or your org for domain-specific workflows (brand guidelines, a QBR methodology, a compliance checklist) so Claude follows the same steps every time.

**Creating one through conversation** (no code needed): tell Claude what you want the skill to do → answer its interview questions about the workflow and what good output looks like → upload templates/examples if you have them → save. It then appears in your skills list and Claude invokes it automatically when relevant going forward; ask Claude to edit it any time to iterate.

**Security:** only install custom skills from trusted sources; review contents before use if it came from somewhere external, since skills can include executable code.

**Skills vs. Projects** (same table as above): projects = knowledge Claude references; skills = process Claude executes.

## Connectors

MCP (Model Context Protocol) is the open standard behind connectors — think "USB-C for AI": one consistent interface so Claude can plug into many different tools.

**Web connectors** (Google Drive, Notion, Slack, Asana, Stripe, etc.) link Claude to cloud services. **Desktop extensions** run locally through the Claude Desktop app for local files and native apps (e.g. Figma, browser control).

**Setup (web):** claude.ai/directory or **+ → Connectors** in a chat → Connect → authenticate with the service's own login → review and grant the specific permissions requested → test with a simple request.

**Security model:** access is scoped to what the connector needs; Claude only sees what *you* have access to in that tool (connecting your email doesn't expose your CEO's inbox); connections are revocable any time from Claude's settings or the third-party service.

## Enterprise Search

Team/Enterprise only, admin-enabled. Adds "Ask {Org Name}" to the sidebar — effectively a pre-built, org-wide project with your company's connected knowledge already loaded and custom instructions tuned by Anthropic. Good for "what happened while I was out," policy/process questions, cross-source research, onboarding questions, and project status synthesis across SharePoint/Slack/Gmail/Drive etc., always with citations.

**Setup:** an Owner connects a Documents source and a Chat source (email recommended) once for the org; individual users then authenticate their own accounts to each connected service. It only surfaces what the individual user already has permission to see in the source tool — it doesn't grant new access.

## Research

An agentic, multi-step investigation mode — not a single search. Claude plans its approach (using Thinking), runs many searches that build on each other (sometimes across hundreds of sources), and compiles a cited report. Takes minutes rather than seconds because of the depth.

**When to reach for what:**
- **Research** — comprehensive/comparative reports, multi-source synthesis, verifiable citations (e.g. "compare these three vendors on price, timeline, and support, with sources").
- **Quick web search** — a single fact, one or two sources, speed matters more than depth.
- **Thinking** — deep reasoning on a self-contained problem (math, code, logic) that doesn't need external information.
- **Enterprise Search** — the answer lives in the org's own docs/Slack/email, not the public web.

**Enable it:** **+ → Research** (web search must also be on). **Prompt tips:** be specific about the goal, specify the sections/structure you want, include constraints (budget, timeline, geography), and if unsure how to frame it, ask Claude to help write the Research prompt first. With Google Workspace or other integrations connected, Research can pull in your email/calendar/docs alongside web sources.

## Other ways to work with Claude

| Surface | Best for | Where it runs |
|---|---|---|
| Claude Code | Building features, debugging, navigating an unfamiliar codebase, automating lint/merge/release-note grunt work | Terminal, IDE, or browser |
| @Claude | Drafting replies, summarizing threads, meeting prep pulling from workspace history, handing a bug report straight to a Claude Code session | Slack |
| Claude Design | Turning a brief/sketch/screenshot into a working UI prototype, exploring variations, staying consistent with your design system | Web |
| Claude for Excel | Tracing formulas across tabs, updating assumptions while preserving dependencies, debugging #REF!/#VALUE!/circular refs, building pivots/charts | Excel sidebar |
| Claude for PowerPoint | Outline → first-draft deck, tightening copy, restructuring sections, consistent formatting | PowerPoint sidebar |
| Claude for Word | Outline → structured draft in your template, revising in place, working through tracked changes/comments | Word sidebar |
| Claude for Outlook (beta, separate install) | Triaging mail, drafting replies with thread/calendar context, summarizing long chains | Outlook sidebar |
| Claude in Chrome (public beta) | Summarizing pages while browsing, drafting from webmail, filling repetitive forms, multi-step browser workflows, pulling context from internal tools/CRMs/dashboards | Chrome sidebar |

Claude in Chrome asks permission before high-risk actions (purchases, sharing personal data) and blocks certain site categories (financial services, adult content) by default — recommended for low-risk tasks on trusted sites while in beta.

## Quick answers to "which one do I use?"

- Multi-step, ends in a real file, spans tools, or runs on a schedule → **Cowork**.
- Back-and-forth thinking, drafting, or a quick question → **Chat**.
- Touches actual source code → **Claude Code** (or @Claude/Slack if the trigger is a bug report in a thread).
- Needs comprehensive multi-source synthesis with citations → **Research**; needs a fact fast → plain web search; needs org-internal knowledge → **Enterprise Search**; needs Claude to reason through a self-contained problem → **Thinking**.
- A repeatable process/workflow you want Claude to run the same way every time → build a **Skill**.
- Ongoing reference material and standing instructions for a workstream → set up a **Project**.

