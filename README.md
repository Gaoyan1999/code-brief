# code-brief

Generate a beautiful HTML introduction for any codebase.

## What it does

Point `code-brief` at a codebase and it analyzes the structure, purpose, and key components, then produces a self-contained HTML file that introduces the project — architecture overview, file tree, key modules, and more.

## Usage

```bash
code-brief <path-to-codebase> [options]
```

Output: a single `brief.html` file ready to open in any browser.

## Claude Code and Codex

This repo is designed to support both Claude Code and Codex.

- Claude Code: keep the `.claude/` setup and expose the skill as `/code-brief` if your Claude environment is configured that way.
- Codex: install or symlink [skills/code-brief/SKILL.md](/Users/daniel/tools/code-brief/skills/code-brief/SKILL.md:1) into `~/.codex/skills/code-brief`, then restart Codex.

Example:

```bash
mkdir -p ~/.codex/skills
ln -s /Users/daniel/tools/code-brief/skills/code-brief ~/.codex/skills/code-brief
```

After restart, invoke it by mentioning `code-brief` in your prompt. Some Codex clients may also expose `/code-brief`, but that depends on the client, not just the skill file.

## Why

Reading an unfamiliar codebase is slow. `code-brief` gives you a structured, readable entry point in seconds — useful for onboarding, code reviews, and documentation.
