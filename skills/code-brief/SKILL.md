---
name: code-brief
description: Generate a structured HTML or Markdown overview of a codebase.
---

# /code-brief — Codebase Overview Generator

Analyze the codebase in the current working directory and produce a structured document explaining it. Save the output as `OVERVIEW.html` or `OVERVIEW.md` in the project root.

This skill is intended to work in both Claude Code and Codex.
- In Claude Code, `/code-brief` may be exposed as a slash command.
- In Codex, use the installed skill by naming `code-brief` in the prompt. Some Codex clients may also expose it as `/code-brief`, but do not assume slash-command support.

---

## Phase 1: Ask the User

Collect both answers before continuing.

- In Claude Code, prefer a single `AskUserQuestion` call so the user can answer both at once.
- In Codex or any environment without structured question UI, ask a concise single message covering both choices, or proceed with defaults if the user already implied them.

**Question 1 — Output Format** (header: "Format"):
- HTML (recommended) — styled page with collapsible sections, Mermaid diagrams rendered in-browser
- Markdown — GitHub-compatible text; Mermaid in fenced code blocks

**Question 2 — Depth** (header: "Depth"):
- Quick Read + Deep Dive (recommended) — add repository map, entry points, module breakdown, data/request flow, design patterns, external dependencies
- Quick Read only — plain-English introduction, tech stack, architecture, quick-start

Default behavior if the user does not specify:
- Output Format: HTML
- Include Quick Read cards only

If the user has not already implied the depth they want, ask one concise follow-up: `Also include Deep Dive cards?`

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

- **System type**: classify the project as one of these before deciding how to explain architecture:
  - `Application` — user-facing product or service with clear business workflows
  - `Framework / Library` — reusable APIs, runtime, compiler, SDK, or developer tooling
  - `Platform / Infrastructure` — control plane, data plane, networking, orchestration, storage, or runtime substrate
  - `Pipeline / Data system` — ingestion, transformation, scheduling, batch/stream processing, model or analytics flow
- **Primary language(s)**: infer from file extensions if no manifest confirms it
- **Framework(s)**: look for framework-specific markers (`next.config.js` → Next.js, `manage.py` → Django, `tokio` in Cargo.toml → async Rust, etc.)
- **Database/storage**: ORM configs, migration directories, connection strings in `.env.example`
- **Infrastructure/deployment**: Dockerfile, `vercel.json`, `fly.toml`, `.github/workflows/`, Terraform files
- **Entry points**: `main.py`, `index.ts`, `src/main.rs`, `cmd/`, `app.py`, `server.js`, `__main__.py`, scripts in `package.json`

### Step 2.4 — Deep source reading (Deep Dive only)

Only perform this step if the user selected Quick Read + Deep Dive.

Read 4–8 key source files to understand internal architecture. Prioritize in this order:
1. Entry point file(s) from Step 2.3
2. Core domain/business logic (`src/core/`, `src/lib/`, `internal/`, `pkg/`)
3. Main router or controller (`routes.`, `router.`, `controllers/`)
4. Data model definitions (`models/`, `schema.`, `types.`, `entities/`)
5. Configuration/initialization (`config.`, `settings.`, `app.`, bootstrap patterns)

For each file read, note: what it does, what it imports from, and what depends on it.

---

## Phase 3: Generate Output Content

Produce the content as individual cards. Do not group them under larger "Quick Read" or "Deep Dive" section headers. Generate cards in this order:

1. What is this?
2. Key features
3. Who is it for?
4. How it works
5. Tech stack
6. Architecture
7. Quick start
8. Repository map
9. Entry points
10. Key modules
11. Data flow
12. Design patterns
13. External dependencies

Generate cards 1–7 for every run. Generate cards 8–13 only if the user selected Quick Read + Deep Dive.
Not every card must be generated. If the repository does not contain enough evidence to support a card, omit that card rather than inventing content.

Use the name from the manifest for the project title. Write a one-line purpose from the README description, or infer it from the project structure if no README exists.

Write the plain-English introduction cards entirely in plain English. No code blocks. No technical terms without immediate plain-language explanation. Target reading level: non-developer stakeholder.

**What is this?**
One to two paragraphs: what the software does, what problem it solves, why someone would want it.

**Key features**
5–8 user-facing capabilities. For HTML output, render as a `feature-grid` of `feature-card` elements. Each card has three parts:
- `feature-icon`: a single relevant emoji
- `feature-title`: a short bold action phrase (4–7 words, e.g. "Runs fully autonomously")
- `feature-desc`: one concise sentence (≤15 words) saying what it does or why it matters

Do not write long explanations. Keep descriptions scannable.

Example card:
```html
<div class="feature-card">
  <span class="feature-icon">🤖</span>
  <span class="feature-title">Lets agents shop autonomously</span>
  <span class="feature-desc">Agents authenticate with an API key and buy with no human in the loop.</span>
</div>
```

Wrap all cards in `<div class="feature-grid">`.

For Markdown output, fall back to a bullet list: `- **Short title** — one concise sentence.`

**Who is it for?**
One paragraph describing the intended user or audience.

**How it works**
One paragraph, no jargon. Analogies welcome.

**Tech stack**
Render this section as a table with four columns: `Category | Technology | Version | Notes`.
Keep technologies of the same category adjacent to each other. Sort primarily by category, then by the importance or prominence evidenced in the repository.
Use one row per stack item. If multiple tightly-coupled tools are conventionally presented together, they may share one row in the `Technology` column, with versions combined in the `Version` column.
Reuse the major technologies from this table in the top banner row as linked `logo + label` badges.
For the banner badges only, point links to the official technology websites and include logos when a trustworthy logo URL and official site can be identified from repo evidence or well-known official sources. If not, omit that badge rather than guessing.
Include only technologies that are actually evidenced by the repository.

**Architecture**
Draw a layered high-level diagram that optimizes for flow clarity, not component count. Start by asking:
- What is the main input? User action, HTTP request, developer API call, scheduled job, event, or message
- How does that input move from one layer to the next?
- What communication style connects the layers? HTTPS, REST, GraphQL, function call, event bus, queue, reactive update, compiler pass, etc.
- What is the final destination? Rendered UI, service response, persisted data, emitted artifact, external side effect

Structure the diagram vertically from top to bottom around 3–5 layers. Do not lay the primary architecture left-to-right unless the user explicitly asks for that. Label each layer clearly and place 1–4 core components inside it when evidence exists. Prefer stable responsibilities over implementation detail.

Choose the layer model that best fits the project type:
- **Application**: `Entry / Presentation -> Communication -> Service / Domain -> Data / Infrastructure`
  - Typical web example: user or browser -> web app -> `HTTPS / REST` -> backend service groups -> MySQL / MongoDB / Redis
  - The communication layer may be collapsed into a labeled connector if it is simple
- **Framework / Library**: `Consumer -> Public Interface -> Runtime Coordination -> Core Mechanism -> Platform Target`
  - Example shape for Vue-like systems: developer code or user events -> public API -> scheduler / component runtime -> reactivity / renderer / compiler -> DOM / SSR / custom target
- **Platform / Infrastructure**: `Operator / Client -> API / Control Plane -> Workers / Data Plane -> Storage / Network / Runtime`
- **Pipeline / Data system**: `Sources -> Ingestion -> Processing / Orchestration -> Storage / Serving -> Outputs / Consumers`

Inside each layer, show simple module boxes with short names only, such as `Web App`, `API Gateway`, `Payment Service`, `HTML Renderer`, `Slide Generator`, `MySQL`, or `Redis`. Do not turn the diagram into prose. Avoid bullets, long descriptions, user stories, or business workflow narration inside the boxes.

Keep it concise. A simple project may only need 3 boxes. Do not mention files, functions, classes, or low-level implementation details here. The goal is to make the main flow legible at a glance and the main code or functional modules easy to identify.

**Markdown**: render as ASCII (Unicode box-drawing chars ┌─┐ │ └─┘ ▼ ►) inside a fenced code block, under ~20 lines, with layers stacked vertically.
**HTML**: render as a top-to-bottom layout inside `<div class="architecture">`. Use one `<div class="arch-layer">` per layer, put module boxes inside it with `<div class="arch-node">Short Module Name</div>`, and separate layers with a simple `<div class="arch-arrow">↓</div>` or a short connector label such as `HTTPS / REST` above the arrow. Do not place paragraph text or bullet lists inside architecture boxes.

**Quick start**
Commands to install dependencies and run the project, pulled from README, Makefile, or `package.json` scripts. Infer standard patterns where needed (e.g. `npm install && npm run dev`). If genuinely unknown, say so rather than inventing commands.

**Repository map**
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

**Key modules**
Table: Module/File | Responsibility | Key dependencies. One row per significant module from Step 2.4.

**Data flow**
Mermaid `sequenceDiagram` or `flowchart LR` tracing one representative end-to-end flow from input to output. Choose the most characteristic flow for the detected system type:
- Application: user action or request through presentation, communication, service, and data layers
- Framework / Library: developer call, state change, or runtime event through public API, coordination layer, and core mechanism
- Platform / Infrastructure: operator or client request through control plane into workers, storage, or runtime substrate
- Pipeline / Data system: source event or batch input through ingestion, transform stages, and output sink

The flow should reinforce the Architecture card. Use layer-oriented labels, not file names, unless a specific named service or subsystem is the clearest unit.

**Design patterns**
Bullet list of patterns observed (dependency injection, repository pattern, event-driven, CQRS, middleware chain, etc.). One sentence of evidence per pattern cited. Do not list patterns not evidenced by the source read.

**External dependencies**
Table: Dependency | Purpose | Notes. Production dependencies only, not dev/test tools.

---

## Phase 4: Format and Save

### Markdown

Write standard GitHub-flavored Markdown. Use `#` for project name and `##` for each card title. Mermaid diagrams use triple-backtick fenced blocks with `mermaid` language tag. Save as `OVERVIEW.md` in the project root.

### HTML

Produce a single self-contained HTML file. All CSS inline. Mermaid.js loaded from CDN. Save as `OVERVIEW.html` in the project root.

After writing `OVERVIEW.html`, open it automatically in the system's default browser. If the environment truly cannot open a browser, stop after saving the file.

Read `template.html` (in the same directory as this skill file) to get the exact template structure. Fill in `[Project Name]`, `[subtitle]`, badge content, sidebar links, and section bodies. Only include sidebar links for cards that were generated.
In the top banner, render the major stack items as linked badges that include both a logo and a text label.

**Card IDs** — use these exact values so sidebar anchor links work:

| Card | ID |
|---|---|
| What is this? | `what-is-this` |
| Key features | `key-features` |
| Who is it for? | `who-is-it-for` |
| How it works | `how-it-works` |
| Tech stack | `tech-stack` |
| Architecture | `architecture` |
| Quick start | `quick-start` |
| Repository map | `repository-map` |
| Entry points | `entry-points` |
| Key modules | `key-modules` |
| Data flow | `data-flow` |
| Design patterns | `design-patterns` |
| External dependencies | `dependencies` |

Each card must be one `<div class="section" id="[card-id]">` block with a matching sidebar link. Do not nest cards under larger section headings. Use the card title as the `<h2>` inside the section header.

Place Mermaid diagrams inside `<div class="mermaid">` elements. Architecture diagrams inside `<div class="architecture">` (HTML div-based boxes, not ASCII). Directory trees inside `<pre class="tree">`. Shell commands inside `<pre><code>` blocks.
For the `Tech stack` card in both HTML and Markdown, render a normal table with `Category`, `Technology`, `Version`, and `Notes` columns. Keep same-category rows grouped together.

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
