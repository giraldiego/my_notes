Perfect, that's a clean structure. Let's get started.

## Plan Refinement Based on Your Answers

A few important observations:

- **Filenames don't always match syllabus titles.** For example, the syllabus shows "Environment preparation (Github Codespaces)" but the file is `006 AI Engineering Buildcamp： From RAG to Agents.html`. So we'll need fuzzy matching or content-based matching (reading the `<h1>` from each file) rather than filename matching.
- **The numeric prefix (001, 002...) gives us order**, which is valuable.
- **There's a full-width Unicode colon (`：`) in some filenames** — Python handles this fine, but worth noting for the agent.
- **Right-side panel stays** — good, less work, and it preserves intra-section navigation.

## Workflow with Antigravity

Since you chose option (b), here's how we'll proceed:
- Each turn I give you **one prompt**.
- You paste it into Antigravity.
- You share back with me **what the agent produced** (script output, file contents, or a summary).
- I review, then give you the next prompt.

⚠️ **Important safety rule for the agent**: We always work on **copies** or with **dry-run modes first** before mutating files. I'll bake this into the prompts.

---

## Prompt #1 — Discovery & Analysis (read-only)

This first prompt is **purely investigative**. The agent should not modify any file. Its only job is to help us understand the HTML structure so we can write precise selectors in the next step.

Copy-paste this into Antigravity:

---

> **Task: Discovery phase — analyze HTML structure (READ-ONLY, do not modify any files)**
>
> I have a folder containing HTML pages downloaded with the SingleFile browser extension from an online course platform. The structure is:
>
> ```
> .
> ├── syllabus.html
> ├── Week 1/
> │   ├── 001 Course overview.html
> │   ├── 002 Course platform.html
> │   └── ... (29 files)
> ├── Week 2/
> │   └── ... (2 files)
> └── Week 3/
>     └── ... (14 files)
> ```
>
> My goal (for context, not for this task) is to eventually: remove the left-side navigation sidebar from each lesson page, rewrite all external links to be local, and turn `syllabus.html` into an `index.html` that links to each local lesson file.
>
> **For THIS task, do NOT modify any file. I only want a written analysis report.** Please do the following:
>
> 1. **Pick 3 sample lesson files** from different weeks (e.g., `Week 1/001 Course overview.html`, `Week 1/015 RAG (Retrieval-Augmented Generation).html`, `Week 3/002 Agentic RAG and function calling.html`).
>
> 2. For each sample file, report:
>    - Approximate file size.
>    - The top-level structure of `<body>` (just the first 2-3 levels of nesting, with tag names, IDs, and classes).
>    - The CSS selector or DOM path that identifies the **left sidebar** (the panel with "Home / Inbox / People / announcements / general / intros / questions / submissions / etc.").
>    - The CSS selector or DOM path that identifies the **top header bar** (with "Back" button and breadcrumb title).
>    - The CSS selector or DOM path that identifies the **main lesson content area** (title + body text).
>    - The CSS selector or DOM path that identifies the **right-side lesson list panel** (LESSON 1, LESSON 2…).
>    - Whether assets (CSS, images, fonts) are inlined as data URIs or referenced externally.
>    - Find any `<a href="...">` tags that point to the live website (e.g., URLs starting with `https://` and the course platform's domain) — list 5-10 sample hrefs so I can see the URL patterns.
>
> 3. **Analyze `syllabus.html`** and report:
>    - The CSS selector / DOM path for each "Week" block.
>    - The CSS selector / DOM path for each "Session" block within a week.
>    - The CSS selector / DOM path for each "Section" block (e.g., "Course Overview & Logistics", "Environment Preparation").
>    - The CSS selector / DOM path for each individual **lesson link** within a section, and what the `href` looks like.
>    - The text content pattern of each lesson entry (so we know how to match them to local files later).
>
> 4. **Compare** lesson titles in the syllabus against filenames in the Week folders. Pick 5 lessons from the syllabus and tell me:
>    - The exact title text in the syllabus.
>    - The matching local file path (if you can find one).
>    - Whether the match is exact, partial, or unclear.
>
> 5. Save your full report as `analysis_report.md` in the project root. Do **not** create or modify any other files.
>
> Use Python with `beautifulsoup4` and `lxml` for parsing. If those packages aren't installed, install them in a virtual environment first and tell me what you did.

---

Once the agent finishes, paste back the contents of `analysis_report.md` (or summarize the key findings if it's huge), and I'll craft Prompt #2 based on the actual selectors and patterns the agent discovered.