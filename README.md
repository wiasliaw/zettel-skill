English | [繁體中文](README-zh.md)

# zettel-skill

A Claude Code plugin marketplace for running a Zettelkasten workflow inside any Obsidian vault.

## What is this

`zettel-skill` is a Claude Code plugin marketplace monorepo — a single repo that can host multiple plugins, each under `plugins/<name>/`. Right now it ships one plugin, **zk**.

`zk` is a generalized Zettelkasten workflow: point it at any Obsidian vault, run `/zk:init` once, and the vault gets its own configuration and templates so the rest of the commands can operate on it. The note lifecycle has two parallel entry points — quick idea capture and source reading — that both feed into a fully automatic splitting-and-filing step, plus a read-only retrieval command for querying what's already filed. A separate memory loop lets the plugin surface behavioral suggestions that you can promote into standing conventions or reject.

## Requirements

- Claude Code
- An Obsidian vault
- [qmd](https://github.com/tobi/qmd) (optional) — if it isn't installed or isn't enabled, retrieval falls back to plain text search (Grep) instead of semantic search. This fallback is expected behavior, not an error.

## Installation

1. In Claude Code, run:
   ```
   /plugin marketplace add wiasliaw/zettel-skill
   ```
2. Then install the plugin:
   ```
   /plugin install zk@zettel-skill
   ```
3. In your Obsidian vault root, run `/zk:init` to configure it.

## Commands

| Command | What it does |
|---|---|
| `/zk:init` | Interviews you about the vault, proposes a scaffold plan, and — once you approve it — writes the step directories, note templates, tagging convention, config file (`.zettel.json`), and qmd setup. |
| `/zk:fleeting <path\|text>` | Acts as a Socratic thinking partner to develop a raw idea into a draft, with your review before it's saved (draft-then-review). |
| `/zk:literature <path\|source>` | Closely reads a source, asks clarifying questions, writes up the note, and runs an adversarial review pass on the finished draft. |
| `/zk:permanent <path>` | Fully automatic: splits a draft into atomic notes, adds inline links between them, files the original source note into your reference archive, updates the retrieval index, verifies the result, and reports what it did. |
| `/zk:query <query>` | Read-only retrieval: answers your question from what's already filed, surfaces one related note via serendipitous discovery, and reports any gaps it noticed in your notes. |
| `/zk:adapt` | Walks through pending behavioral suggestions one at a time so you can promote each into a standing convention or reject it. |

`/zk:init`, `/zk:fleeting`, `/zk:literature`, `/zk:permanent`, and `/zk:adapt` only run when you invoke them explicitly. `/zk:query` is read-only and may also be triggered contextually.

## Documentation

- [Tutorial](docs/en/tutorial.md) — a walkthrough of the full workflow
- [Design](docs/en/design.md) — how the plugin is put together and why
- [Workflow](docs/en/workflow.md) — the day-to-day usage pattern

## License

Apache-2.0
