# code-brief

Generate a structured, beautiful HTML (or Markdown) overview of any codebase — architecture, tech stack, key modules, and more — in one command.

## What it does

Point `/code-brief` at a project and it reads the source, then produces a self-contained `OVERVIEW.html` with collapsible sections, a live Mermaid architecture diagram, a clickable tech-stack grid, and a key-features card layout. A Markdown fallback is also available for GitHub rendering.

Useful for onboarding, code reviews, and documentation — readable in seconds, no manual writing required.

## Install for Claude Code

Skills live in `~/.claude/skills/`. Clone this repo and symlink the skill directory:

```bash
git clone https://github.com/Gaoyan1999/code-brief.git ~/tools/code-brief
mkdir -p ~/.claude/skills
ln -s ~/tools/code-brief/skills/code-brief ~/.claude/skills/code-brief
```

Restart Claude Code, then run:

```
/code-brief
```

## Install for Codex

Skills live in `~/.codex/skills/`. Use the same clone and symlink it there instead:

```bash
git clone https://github.com/Gaoyan1999/code-brief.git ~/tools/code-brief
mkdir -p ~/.codex/skills
ln -s ~/tools/code-brief/skills/code-brief ~/.codex/skills/code-brief
```

Restart Codex, then invoke it by mentioning `code-brief` in your prompt. Some Codex clients also expose `/code-brief` as a slash command.
