---
name: code-brief
description: Use this skill when the user runs /code-brief, asks to "analyze this codebase", "explain this project", "generate a project overview", "document this repo", "create a codebase summary", "I want to understand this code", or says "give me a brief/overview/intro for this project". Produces a structured HTML or Markdown document saved to the project root describing what the project does, how it's built, and how to navigate it.
---

# /code-brief — Codebase Overview Generator

Analyze the codebase in the current working directory and produce a structured document explaining it. Save the output as `OVERVIEW.html` or `OVERVIEW.md` in the project root.

---

## Phase 1: Ask the User

Ask both questions in a single AskUserQuestion call so the user fills them out at once.

**Question 1 — Output Format** (header: "Format"):
- HTML (recommended) — styled page with collapsible sections, Mermaid diagrams rendered in-browser
- Markdown — GitHub-compatible text; Mermaid in fenced code blocks

**Question 2 — Depth** (header: "Depth"):
- Overview (recommended) — tech stack, annotated directory tree, entry points, quick-start commands
- Non-technical — plain English: what it does, key features, who it's for, how it works
- Deep-dive — everything in Overview plus architecture diagram, module breakdown, data/request flow, design patterns, external dependencies

Record the answers before continuing.

---

## Phase 2: Analyze the Codebase

Work through these steps in order. Each step builds on the previous.

### Step 2.1 — Read high-signal root files

Check for each file below and read it if it exists. These give the highest-signal understanding per token spent.

| File(s) | What to extract |
|---|---|
| `README.md` / `README.rst` / `README.txt` | Project name, purpose, setup steps, feature list |
| `package.json` | name, description, scripts (start/build/test/dev), dependencies |
| `pyproject.toml` / `setup.py` / `setup.cfg` | name, version, dependencies, entry points |
| `Cargo.toml` | name, description, workspace members |
| `go.mod` | module name, Go version |
| `composer.json` | name, description, require |
| `Gemfile` / `*.gemspec` | gem name, dependencies |
| `pubspec.yaml` | name, description, dependencies |
| `pom.xml` / `build.gradle` | artifactId, groupId, dependencies |
| `docker-compose.yml` / `Dockerfile` | services, base images, exposed ports |
| `.env.example` | required environment variables |
| `Makefile` | common task targets |
| `CLAUDE.md` | project-specific context already written for AI |

Do not fabricate information from files that do not exist. Record what was found and what was absent.

### Step 2.2 — Scan directory structure (depth 1–2)

List all directories and files at depth 1 and depth 2. Ignore these directories entirely: `node_modules/`, `.git/`, `__pycache__/`, `.venv/`, `venv/`, `dist/`, `build/`, `target/`, `vendor/`, `.next/`, `.nuxt/`, `coverage/`.

For each directory, form a working hypothesis about its role: source code, tests, config, docs, migrations, static assets, generated output, etc. These hypotheses become the annotations in the directory tree section.

### Step 2.3 — Identify tech stack and entry points

Based on manifest files and directory scan, determine:

- **Primary language(s)**: infer from file extensions if no manifest confirms it
- **Framework(s)**: look for framework-specific markers (`next.config.js` → Next.js, `manage.py` → Django, `tokio` in Cargo.toml → async Rust, etc.)
- **Database/storage**: ORM configs, migration directories, connection strings in `.env.example`
- **Infrastructure/deployment**: Dockerfile, `vercel.json`, `fly.toml`, `.github/workflows/`, Terraform files
- **Entry points**: `main.py`, `index.ts`, `src/main.rs`, `cmd/`, `app.py`, `server.js`, `__main__.py`, scripts in `package.json`

### Step 2.4 — Deep source reading (deep-dive mode only)

Only perform this step if the user selected Deep-dive.

Read 4–8 key source files to understand internal architecture. Prioritize in this order:
1. Entry point file(s) from Step 2.3
2. Core domain/business logic (`src/core/`, `src/lib/`, `internal/`, `pkg/`)
3. Main router or controller (`routes.`, `router.`, `controllers/`)
4. Data model definitions (`models/`, `schema.`, `types.`, `entities/`)
5. Configuration/initialization (`config.`, `settings.`, `app.`, bootstrap patterns)

For each file read, note: what it does, what it imports from, and what depends on it.

---

## Phase 3: Generate Output Content

Produce the content for the chosen depth mode. Then format it (Phase 4).

### Overview mode

**Project name and purpose**
Use the name from the manifest. Write a one-line purpose from the README description, or infer it from the project structure if no README exists.

**Tech stack table**
Four columns: Category | Technology | Version (if known) | Notes.
Rows: Language, Framework, Database, Infrastructure, Testing, CI/CD. Omit rows where nothing applies.

**Annotated directory tree**
ASCII tree using box-drawing characters. Annotate each top-level directory with a short phrase. Annotate key individual files (entry points, config files, schema files). Do not annotate every file.

```
src/
├── api/          # HTTP handlers and route definitions
├── core/         # Domain logic, framework-independent
├── db/           # Database models and migrations
│   ├── models.py
│   └── migrations/
└── main.py       # Application entry point
```

**Entry points**
List each entry point: file path, how to invoke it, what it starts or does.

**Quick start**
Commands to install dependencies and run the project, pulled from README, Makefile, or `package.json` scripts. Infer standard patterns where needed (e.g. `npm install && npm run dev`). If genuinely unknown, say so rather than inventing commands.

---

### Non-technical mode

Write entirely in plain English. No code blocks. No technical terms without immediate plain-language explanation. Target reading level: non-developer stakeholder.

**What is this?**
One to two paragraphs: what the software does, what problem it solves, why someone would want it.

**Key features**
Bullet list of 5–8 user-facing capabilities. Write each as a benefit or action ("Lets you...", "Automatically...", "Supports..."), not as a technical feature name.

**Who is it for?**
One paragraph describing the intended user or audience.

**How it works**
One paragraph, no jargon. Analogies welcome.

---

### Deep-dive mode

Include all Overview sections, then add:

**Architecture diagram**
Mermaid `graph TD` or `graph LR` diagram showing major components and their relationships. Include: entry point(s), service/application layer, data layer, external services. Cap at 10–15 nodes — clarity beats completeness.

**Key modules and their responsibilities**
Table: Module/File | Responsibility | Key dependencies. One row per significant module from Step 2.4.

**Data or request flow**
Mermaid `sequenceDiagram` or `flowchart LR` tracing a representative request or data transformation from input to output. Choose the most characteristic flow for this type of application.

**Notable design patterns**
Bullet list of patterns observed (dependency injection, repository pattern, event-driven, CQRS, middleware chain, etc.). One sentence of evidence per pattern cited. Do not list patterns not evidenced by the source read.

**External dependencies**
Table: Dependency | Purpose | Notes. Production dependencies only, not dev/test tools.

---

## Phase 4: Format and Save

### Markdown

Write standard GitHub-flavored Markdown. Use `#` for project name, `##` for sections, `###` for subsections. Mermaid diagrams use triple-backtick fenced blocks with `mermaid` language tag. Save as `OVERVIEW.md` in the project root.

### HTML

Produce a single self-contained HTML file. All CSS inline. Mermaid.js loaded from CDN. Save as `OVERVIEW.html` in the project root, then run `open OVERVIEW.html`.

Use this exact template structure — fill in `[Project Name]`, `[subtitle]`, badge content, and section bodies:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[Project Name] — Overview</title>
  <script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      font-size: 15px; line-height: 1.6; color: #1a1a2e; background: #f8f9fc;
    }
    .page-header {
      background: linear-gradient(135deg, #1a1a2e 0%, #16213e 60%, #0f3460 100%);
      color: #fff; padding: 3rem 2rem 2rem; text-align: center;
    }
    .page-header h1 { font-size: 2.4rem; font-weight: 700; letter-spacing: -0.02em; }
    .page-header .subtitle { font-size: 1.1rem; opacity: 0.8; margin-top: 0.5rem; }
    .badge-row { display: flex; gap: 0.5rem; justify-content: center; flex-wrap: wrap; margin-top: 1rem; }
    .badge {
      background: rgba(255,255,255,0.15); border: 1px solid rgba(255,255,255,0.2);
      border-radius: 999px; padding: 0.2rem 0.75rem; font-size: 0.8rem; font-weight: 500;
    }
    main { max-width: 900px; margin: 2rem auto; padding: 0 1.5rem; }
    .section {
      background: #fff; border-radius: 12px; border: 1px solid #e8eaf0;
      margin-bottom: 1.5rem; overflow: hidden;
    }
    .section-header {
      display: flex; align-items: center; justify-content: space-between;
      padding: 1rem 1.5rem; cursor: pointer; user-select: none;
      border-bottom: 1px solid #e8eaf0;
    }
    .section-header h2 { font-size: 1rem; font-weight: 600; color: #1a1a2e; }
    .section-header .toggle { font-size: 0.75rem; color: #888; }
    .section-body { padding: 1.5rem; }
    .section-body.collapsed { display: none; }
    table { width: 100%; border-collapse: collapse; font-size: 0.9rem; }
    th {
      background: #f4f5f8; text-align: left; padding: 0.6rem 1rem;
      font-weight: 600; font-size: 0.8rem; text-transform: uppercase;
      letter-spacing: 0.05em; color: #555;
    }
    td { padding: 0.6rem 1rem; border-top: 1px solid #eee; vertical-align: top; }
    tr:hover td { background: #fafbff; }
    pre.tree {
      font-family: "SFMono-Regular", Consolas, monospace; font-size: 0.85rem;
      line-height: 1.7; background: #f4f5f8; border-radius: 8px;
      padding: 1rem 1.25rem; overflow-x: auto;
    }
    code {
      font-family: "SFMono-Regular", Consolas, monospace; font-size: 0.85em;
      background: #f0f1f5; border-radius: 4px; padding: 0.1em 0.35em;
    }
    pre:not(.tree) {
      background: #1a1a2e; color: #e8eaf0; border-radius: 8px;
      padding: 1rem 1.25rem; overflow-x: auto; line-height: 1.6;
    }
    pre:not(.tree) code { background: none; padding: 0; font-size: 0.85rem; }
    .mermaid { text-align: center; padding: 1rem 0; }
    ul, ol { padding-left: 1.5rem; }
    li { margin: 0.3rem 0; }
    p + p { margin-top: 0.75rem; }
    footer { text-align: center; color: #aaa; font-size: 0.8rem; padding: 2rem 0 3rem; }
  </style>
</head>
<body>
<header class="page-header">
  <h1>[Project Name]</h1>
  <p class="subtitle">[One-line purpose]</p>
  <div class="badge-row">
    <span class="badge">[Language]</span>
    <span class="badge">[Framework]</span>
    <!-- add one badge per major stack element -->
  </div>
</header>
<main>
  <!-- Repeat this block for each section -->
  <div class="section">
    <div class="section-header" onclick="toggle(this)">
      <h2>Section Title</h2>
      <span class="toggle">▾</span>
    </div>
    <div class="section-body">
      <!-- section content -->
    </div>
  </div>
</main>
<footer>Generated by /code-brief · [Date]</footer>
<script>
  mermaid.initialize({ startOnLoad: true, theme: 'neutral', securityLevel: 'loose' });
  function toggle(header) {
    const body = header.nextElementSibling;
    const isCollapsed = body.classList.toggle('collapsed');
    header.querySelector('.toggle').textContent = isCollapsed ? '▸' : '▾';
  }
</script>
</body>
</html>
```

Place Mermaid diagrams inside `<div class="mermaid">` elements. Directory trees inside `<pre class="tree">`. Shell commands inside `<pre><code>` blocks.

---

## Phase 5: Edge Cases

Handle these silently — do not ask the user additional questions.

| Situation | Action |
|---|---|
| No README found | Continue; note "No README found — description inferred from project structure" in output |
| No manifest files found | Identify dominant file extensions; leave stack table sparse with a note |
| Monorepo detected (`packages/`, `apps/`, `pnpm-workspace.yaml`, `turbo.json`, `lerna.json`) | List each package in the tree; for deep-dive, read the 1–2 most significant packages only |
| Very large codebase (>100 top-level items after filtering) | Show depth-1 tree only; note the limitation |
| Credential files visible (`.env`, `secrets.yml`, `credentials.json`) | Note their existence in the tree; do not read or include their contents |
| `OVERVIEW.html` or `OVERVIEW.md` already exists | Overwrite silently |
| Empty or near-empty directory | Produce a minimal output noting no project structure was found; do not fabricate content |

---

## Anti-Patterns

- Do NOT read `.env` files or any file that may contain secrets — note their existence only
- Do NOT fabricate features, commands, or dependencies not evidenced by files you read
- Do NOT include every file in the directory tree — annotate only files that clarify structure
- Do NOT list design patterns (in deep-dive) unless you observed evidence for them in source files
- Do NOT ask for confirmation before overwriting an existing OVERVIEW file
