# Tutorial

This is a walkthrough for someone using the **zk** plugin for the first time: what to install, how to initialize a vault, and how the day-to-day commands behave — what each one asks you, what it writes, and what the resulting notes look like.

## 1. Prerequisites and installation

You need:

- **Claude Code**, with plugin support.
- An **Obsidian vault** (existing or new) — any layout is fine; `/zk:init` adapts to it rather than imposing one.
- **qmd** (optional) — a command-line semantic search tool. If it isn't installed, or you choose not to enable it, retrieval falls back to plain-text search (Grep) instead of semantic search. This fallback is expected behavior, not an error, and the plugin never nags you to install qmd.

To install the plugin:

```
/plugin marketplace add wiasliaw/zettel-skill
/plugin install zk@zettel-skill
```

Then, from the root of your Obsidian vault, run `/zk:init` to set it up. Everything below assumes that has been done.

## 2. Initializing a vault: `/zk:init`

`/zk:init` must be run with your vault root as the current working directory — it starts by showing you `pwd` and asking you to confirm. It then inventories what already exists (an existing `.zettel.json`, the directories it points to, `.zettel/`, `.qmd/`) so it knows whether this is a fresh setup or a gap-filling re-run.

### The interview

`/zk:init` asks you about five things, each tied to something observable rather than guessed:

1. **Layout for each stage.** For each of the four stages — fleeting, literature, permanent, reference — you confirm where its notes live (`place`) and, for fleeting/literature/permanent, which template file it's built from (`templateFile`). Reference has no template: reference notes are never created directly, only produced by filing during `/zk:permanent`. The factory defaults are `inbox`, `literature`, `zettelkasten`, `reference`, and `_template/*.md`, but your existing vault layout takes priority — nothing is assumed.

2. **Tag scheme.** You're shown the factory kind-tag scheme (below) and can adjust it, as long as the result still satisfies three invariants: a staging note's tags must uniquely identify which stage it's from; a filed note's tags must identify both its stage and its topic; and no flow may ever rewrite one kind of tag into another.

3. **Whether to enable qmd.** The interview runs `command -v qmd` first. If it isn't found, retrieval is set to the Grep fallback and the qmd-specific questions are skipped entirely. If it is found, you're asked whether you want to enable it; declining also falls back to Grep.

   If you do enable it, a further sub-interview covers: probing your environment (embed model, compute backend) so you have context for later decisions; the index scope (which directories qmd should index — see below); the CLI argument strings for each of qmd's four operations (`query`, `search`, `update`, `embed` — left empty unless you have preferences); and a short context description of the vault and, optionally, of specific topic partitions. None of this touches disk yet — it only feeds the plan in the next step.

4. **What happens to your existing notes.** Nothing is moved or rewritten. `/zk:init` only explains the two-layer model so you understand where your existing notes will end up landing conceptually: notes in a staging directory (fleeting/literature) are picked up later by the lifecycle commands; notes already in a filed directory (permanent/reference) become part of the indexed, linkable slip-box as-is.

5. **Initial topic partitions.** Optional — you can leave this empty and let `/zk:permanent` create topic directories organically the first time it files something.

### The scaffold plan confirmation gate

Before writing anything, `/zk:init` presents the complete plan in the conversation: every directory it's about to create, the full content of `.zettel.json`, the full text of the tagging convention, and (if qmd is enabled) the qmd settings and `.qmd/index.yml` content. Nothing is written until you approve this plan; declining means nothing is written at all.

### What gets written

Once approved:

- The stage directories you agreed on, plus `.zettel/conventions/`, `.zettel/proposals/`, and `.zettel/logs/`.
- Note templates, seeded from the plugin's factory templates into each stage's `templateFile` path. An existing file at that path is never overwritten — it's listed in the closing report instead.
- The tagging convention, written to `.zettel/conventions/tagging.md`. This is the one built-in exception to "conventions only ever arrive via approved proposals" — factory seeding doesn't count as a promotion.
- `.zettel.json` itself. Here is the factory-default shape (every value is what you agreed to in the interview; `version` is filled with whatever version of the plugin you have installed):

  ```json
  {
    "version": "0.2.1",
    "fleeting":   { "place": "inbox",        "templateFile": "_template/fleeting.md" },
    "literature": { "place": "literature",   "templateFile": "_template/literature.md" },
    "permanent":  { "place": "zettelkasten", "templateFile": "_template/permanent.md" },
    "reference":  { "place": "reference" },
    "qmd":        { "enabled": true, "query": "", "search": "", "update": "", "embed": "" }
  }
  ```

  Note that `reference` has no `templateFile`, and that `qmd.enabled` means "qmd is installed *and* you chose to use it" — this single field is what every later command checks to decide between semantic retrieval and the Grep fallback.

- If you enabled qmd: `.qmd/index.yml`, describing which folders qmd indexes (always including your `permanent.place` and `reference.place`, optionally more, but never the two staging places — staging notes are never indexed), followed by an initial `qmd update && qmd embed` to build the index and a `qmd doctor` check to confirm it's healthy.

### Re-running `/zk:init`

Running `/zk:init` again on an already-initialized vault only fills gaps and migrates — it updates `.zettel.json`'s `version` field and adds anything missing, but never overwrites an existing file and never touches note content. This is also how you toggle qmd on or off later: just re-run `/zk:init` and answer the qmd question differently.

## 3. The daily workflow

Two commands are parallel entry points for getting new material into the vault — `/zk:fleeting` for a raw idea, `/zk:literature` for a source you're reading — and both feed into `/zk:permanent`, which is the only path from draft into the permanent, linkable slip-box. `/zk:query` reads that slip-box without changing anything. `/zk:adapt` is a separate loop for reviewing behavioral suggestions the plugin has noticed about how you work.

Every write command opens with the same two checks: it confirms `.zettel.json` exists (otherwise it stops and tells you to run `/zk:init` — it never creates a config on your behalf), and it re-indexes qmd if enabled (a no-op if qmd is disabled or the index is already current). It also scans your conventions and pending proposals at the start of every run, and — for fleeting and literature — loads or creates that note's **devlog**, a per-note work journal kept under `.zettel/logs/` (more on this in section 4). This is what makes fleeting/literature sessions resumable across interruptions: pick the note back up later and the command picks up where it left off.

### `/zk:fleeting` — develop a raw idea

**Role:** a Socratic thinking partner. **Writes to:** your `fleeting.place`. **Gate:** draft-then-review.

Call it with either a path to an existing fleeting note, or with raw text:

```
/zk:fleeting spaced repetition feels like it works because it forces retrieval, not because of repetition itself
```

```
/zk:fleeting inbox/spaced-repetition.md
```

It lets you dump the idea first, then interrogates it using Paul and Elder's six question types — clarification, assumptions, evidence, viewpoints, implications, and "why does this matter" — but not mechanically or in fixed order: each round it picks whichever thread currently looks weakest, and its default posture is to challenge rather than agree. The conversation ends either when the idea has converged into a single clear claim, or when a contradiction surfaces that needs resolving.

When you're ready, it shows you the full revised text and waits for your confirmation before writing anything (draft-then-review) — decline and nothing is saved. On confirmation: an existing note is updated in place (with its `updated` timestamp bumped); raw text becomes a new note built from your `fleeting.templateFile`, with `version` read live from `.zettel.json`, tags set per your tagging convention, and a uniqueness check on the filename across the whole vault.

The closing report tells you what got written, what the idea converged to, and the devlog status, plus a note of any relevant conventions or pending proposals it noticed along the way.

### `/zk:literature` — read a source closely

**Role:** close-reading a source, then a QA pass, then a draft, then an immediate adversarial review of that draft. **Writes to:** your `literature.place`. **Gates:** a filing confirmation gate, then draft-then-review for the write-up (skippable only if you explicitly say to write directly).

Call it with a source — a URL or a bibliographic reference — or with the path to an existing literature note:

```
/zk:literature https://example.com/some-paper
```

```
/zk:literature literature/some-paper.md
```

Every literature note holds exactly one source, recorded in its `source` field, which can never be empty — it's the sole anchor the close-reading step works from. If you ask it to add a second source to a note that already has one, it doesn't merge them; it creates a second, independent note instead, and any atomic notes later split out of the two get cross-linked once they exist.

For a genuinely new source, unless your instruction already made the intent to file explicit, it first shows you the filing plan (filename, title, the source text itself, and any known reading focus) and waits for confirmation before creating anything.

Once the note exists, the close-reading step dispatches a read-only research subagent anchored strictly to that `source`. If you've stated a reading focus (either in the note's Intent section or in conversation), the agent gives full coverage within that focus and a structural summary outside it; without a stated focus, it fully covers the whole source. Critically, if the agent can't retrieve the source's full text, it aborts and reports why — it never falls back to secondhand summaries or its own trained knowledge to paper over a source it couldn't read.

You can then ask questions; answers stay anchored to the source and the research material, and anything drawn from outside that is explicitly labeled as such. When you signal you're ready ("write it up" or equivalent), the conversation gets synthesized into the note body, filling or updating its Intent section to reflect what you actually focused on. By default you see the draft and confirm before it's saved; only an explicit "write directly" (or equivalent) skips that and writes immediately.

The moment a draft is written, in the same invocation, an adversarial review subagent runs against it — not deferred to the end, and not skipped. That subagent works from a default posture of distrust: it breaks the note into individual claims, tries to disprove each one, and only calls a claim faithful if that fails. Every claim ends up classified as exactly one of faithful, factually wrong, or unsubstantiated. It never edits anything itself — it only reports, with a suggested fix and its supporting quote for anything factually wrong. You then decide, claim by claim, which suggestions to actually apply; the note's `updated` field is set once, at the point you apply them. If a review turns up an unsubstantiated point that seems worth reading more about, it may suggest spinning up a new literature note to cover it — but only as a suggestion you approve, never automatically.

You can also ask for a review at any point mid-conversation ("review this") without writing a draft — it reviews the note's current full text and reports in the conversation only.

The closing report always includes the review's full detail — total claim count, the breakdown by category, the text and reasoning for every non-faithful claim — in full, not folded into a one-line summary, even when `/zk:literature` was invoked as part of a larger request.

### `/zk:permanent` — split and file

**Role:** fully automatic. **Touches:** new permanent notes, the source note's location, the index, its own devlog. **Gate:** none — zero confirmation, by design.

```
/zk:permanent inbox/spaced-repetition.md
```

The input must be an existing staging note — something currently in your `fleeting.place` or `literature.place`. (A reference-layer path is only ever used to detect and resume an interrupted run, never as new material to split.)

Once started, it runs to completion without stopping to ask you anything, in this order:

1. **Split.** A subagent reads the source note in full and, entirely on its own judgment, splits it into one-concept-per-file atomic notes, picks a topic ("top tag") for each — preferring an existing topic directory, creating a new one only when nothing fits — and adds inline `[[wikilink]]` links between the new notes where one atomic note's prose actually references another's concept.
2. **File the source note.** It's moved into `reference.place/<topic>/` — never deleted — where `<topic>` is whichever topic the majority of that batch's atomic notes landed in. Its tags are normalized to the filed scheme (kind plus topic) without ever changing what kind it is.
3. **Delete the devlog.** Once filing succeeds, the source note's journal entry under `.zettel/logs/` is deleted — that's the one and only devlog action `/zk:permanent` ever performs.
4. **Reindex.** If qmd is enabled, it runs `qmd update && qmd embed` so the new atomic notes and the relocated source note become searchable; if qmd is disabled, this step is skipped, since there's no index to maintain.
5. **Verify.** A second, adversarial-review subagent checks every new atomic note's faithfulness against the source note (now in its new location) and verifies every inline link actually points somewhere the linking note's prose supports — again, report-only; you decide which of its suggested fixes to apply.

If every concept in the source note turns out to have too little material to split into its own note, nothing above happens at all — the source note and its devlog are left untouched, and the report just explains, concept by concept, why each was skipped (you might address that by returning to `/zk:fleeting` or `/zk:literature` to develop it further).

Because this runs unattended, it's built to survive interruption: if it's stopped partway through (say, after splitting but before filing, or after filing but before deleting the log), a later invocation can detect and resume the exact right step rather than redoing work or duplicating notes. It never re-splits a source note that's already been split.

The closing report always presents two things side by side, each complete on its own: the filing result (every new note's filename and topic, whether a new topic directory was created, every inline link and why, where the source note ended up, any skipped concepts, and the devlog/reindex status), and the verification report (claim totals by category, the reasoning behind every non-faithful claim, every link's pass/fail, and — for every suggested fix — whether it was actually applied).

### `/zk:query` — ask your slip-box

**Role:** read-only. Never writes a file, never updates the index, never touches a devlog.

```
/zk:query what have I written about desirable difficulties in learning?
```

It searches only the filed layer — `permanent.place` and `reference.place` — using qmd's semantic search if enabled or a Grep-based fallback otherwise, always adding an exact-term Grep pass on top to catch literal tokens (identifiers, proper nouns) that semantic search tends to miss. Before answering, it reads the full text of whichever 2–5 candidates the answer will actually depend on — a similarity score or hit count alone is never treated as grounds to cite something.

The answer cites the notes it actually read with `[[note]]` links. It then reports a **serendipity** find: starting from the outbound links inside the notes it read in full, it follows whichever of those targets weren't already part of the search results, reads up to five of them, and reports anything with a genuine, non-obvious connection to your query — writing "None" explicitly if it finds nothing, rather than omitting the section. Finally, it reports any **gaps**: places your query touched but that your notes don't yet fully cover, framed as a possible next thing to write, not as "your query is unanswerable."

### `/zk:adapt` — decide on proposals

**Role:** a conversation, one proposal at a time. **Touches:** `.zettel/conventions/` and `.zettel/proposals/` only.

```
/zk:adapt
```

Along the way, other commands sometimes notice something worth remembering about how you work — a correction you made twice, a preference you stated — and write it up as a **proposal** file. `/zk:adapt` is where you act on those: it reads every pending proposal in full (including any feedback already recorded in its User review section) and walks through them one at a time, never batching several together and never deciding on your behalf. For each, you choose exactly one of:

- **Reject** — the proposal file is deleted; deletion is the archive, there's no separate "rejected" folder.
- **Promote** — it becomes (or updates) a convention file under `.zettel/conventions/`, one file per topic, each carrying a one-line trigger condition in its frontmatter.

Right before actually writing or deleting anything, it re-reads the proposal and the relevant convention one more time, in case either changed since you last saw it — it never applies a decision made against a stale version.

Promoted conventions take effect starting with the next write command's opening scan — not retroactively on anything already written.

## 4. Vault artifacts and mental model

### Staging vs. filed

Every note lives in one of two layers, and the directories for each are whatever you configured in `.zettel.json` — there's no assumption that they're named `inbox`, `literature`, `zettelkasten`, or `reference`.

- **Staging** (`fleeting.place`, `literature.place`): flat, unpartitioned storage. Staging notes are never added to the qmd index and are never link targets — nothing else in your vault is allowed to `[[link]]` to a staging note.
- **Filed** (`permanent.place/<topic>/`, `reference.place/<topic>/`): partitioned by topic. Filed notes are indexed (when qmd is enabled) and are valid link targets.

Filing — staging becoming filed — is the only direction notes ever move. Nothing goes back.

### Inside `.zettel/`

- **`conventions/`** — your long-term preferences, one file per topic, each with a one-line trigger condition. Every write command reads these at the start of every run.
- **`proposals/`** — suggestions awaiting your decision in `/zk:adapt`. Their existence is only ever mentioned, never acted on automatically.
- **`logs/`** — one devlog per staging note, mirroring its path (e.g., a note at `inbox/foo.md` has its journal at `.zettel/logs/inbox/foo.md`). The devlog records what happened across sessions on that note — the request that started it, key events, and a running status — so `/zk:fleeting` and `/zk:literature` can resume exactly where they left off after an interruption. It's deleted the moment `/zk:permanent` successfully files that note; filed notes never have a devlog.

### Templates are the schema

The frontmatter fields a note is expected to have are defined by the template file at its stage's `templateFile` path — not by anything hardcoded in the plugin. You can edit these template files yourself to add or change fields, and the plugin will read your version. Whatever fields you add beyond the built-in ones are treated as opaque and preserved: no command will ever strip or rewrite a frontmatter field it doesn't recognize (useful, for instance, if you use custom properties for Obsidian Bases).

### What the plugin never touches

The plugin never runs a git command of any kind — version control of your vault is entirely up to you, and it always treats your current working tree as the latest truth. It also never modifies its own installed files; every behavior described in this document comes from the plugin as installed, and any change to that behavior only happens through a plugin update.

## 5. FAQ

**What happens if I never turn on qmd?** Nothing breaks. Retrieval falls back to Grep-based plain-text search across your filed notes, plus exact-term matching on top. It's slower and less semantic, but it's a fully supported mode, not a degraded error state — and the plugin never prompts you to install qmd.

**I renamed or moved a staging note manually — what happens to its devlog?** The devlog is found purely by mirroring the note's path; renaming or moving the note outside of a zk command breaks that mirror. The next time you run a lifecycle command on the note at its new path, it won't find a devlog there and will start a fresh one — the note's session history up to that point is not recovered. This is a known, accepted limitation, not a bug to report.

**Is it safe to run `/zk:init` again on a vault I already initialized?** Yes. A re-run only fills in whatever's missing and updates the recorded plugin version — it never overwrites an existing file and never touches your note content. This is also the supported way to turn qmd on or off after the fact.
