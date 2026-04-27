---
name: code-brief
description: Generate a structured HTML or Markdown overview of a codebase.
---

# /code-brief — Codebase Overview Generator

Analyze the codebase in the current working directory and produce a structured document explaining it. Save the output as `OVERVIEW.html` or `OVERVIEW.md` in the project root.

---

## Phase 1: Ask the User

Ask both questions in a single AskUserQuestion call so the user fills them out at once.

**Question 1 — Output Format** (header: "Format"):
- HTML (recommended) — styled page with collapsible sections, Mermaid diagrams rendered in-browser
- Markdown — GitHub-compatible text; Mermaid in fenced code blocks

**Question 2 — Sections** (header: "Sections", multiSelect: true):
- Overview (recommended) — tech stack, annotated directory tree, entry points, quick-start commands
- Feature Introduction — plain English: what it does, key features, who it's for, how it works
- Deep-dive — architecture diagram, module breakdown, data/request flow, design patterns, external dependencies

The user may select any combination. Generate all selected sections in the output, in the order listed above.

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

### Step 2.4 — Deep source reading (Deep-dive section only)

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

Produce the content for each section the user selected, in order: Overview → Feature Introduction → Deep-dive. Then format it (Phase 4).

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

### Feature Introduction section

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

### Deep-dive section

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

Read `template.html` (in the same directory as this skill file) to get the exact template structure. Fill in `[Project Name]`, `[subtitle]`, badge content, sidebar links, and section bodies. Only include sidebar links for sections the user selected.

**Section and subsection IDs** — use these exact values so sidebar anchor links work:

| Section | Section ID | Subsection | Subsection ID |
|---|---|---|---|
| Overview | `overview` | Tech Stack | `overview-tech-stack` |
| | | Directory Tree | `overview-directory-tree` |
| | | Entry Points | `overview-entry-points` |
| | | Quick Start | `overview-quick-start` |
| Feature Introduction | `feature-introduction` | What is this? | `feature-what-is-this` |
| | | Key Features | `feature-key-features` |
| | | Who is it for? | `feature-who-is-it-for` |
| | | How it works | `feature-how-it-works` |
| Deep-dive | `deep-dive` | Architecture | `deep-architecture` |
| | | Key Modules | `deep-key-modules` |
| | | Data Flow | `deep-data-flow` |
| | | Design Patterns | `deep-design-patterns` |
| | | Dependencies | `deep-dependencies` |

Each subsection heading must be `<h3 id="[subsection-id]">Title</h3>`. The sidebar must include nested `<ul>` sub-links for each subsection within a selected section (see commented examples in `template.html`).

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
