# Design Rationale

This document explains the reasoning behind zk's behaviors — not just what the plugin does, but why it is built this way. It assumes you already know the six commands (`init`, `fleeting`, `literature`, `permanent`, `query`, `adapt`) and the three subagents (`zk-research`, `zk-atomizer`, `zk-adversarial-review`); if you don't, start with the tutorial instead.

## 1. Two-tier note model

**The principle.** Every note lives in one of two tiers. *Staging* notes are drafts-in-progress: fleeting notes and literature notes, stored flat (no topic subfolders), excluded from semantic search indexing, and never the target of a wikilink. *Filed* notes are the finished product: permanent notes (atomic notes) and reference notes, organized into topic folders, indexed, and freely linkable. The only transition between tiers is archiving — staging notes get atomized and archived into the filed tier; there is no path back into staging.

**Why.** An idea you haven't finished thinking through, or a source you haven't finished digesting, shouldn't show up in search results or accumulate incoming links. Filed notes are meant to be citable — the moment something is linkable, other notes start depending on its wording. Keeping staging flat and unindexed means half-formed material can't quietly become load-bearing infrastructure for the rest of the slip-box.

**Where it shows up.** The fleeting and literature workflows only write into their respective staging locations; permanent is the only command that writes into the filed tier, and it only reads from staging as input. Directory names themselves are not fixed — each is a configurable location the user decides during initialization — but the two-tier boundary and the one-way transition are not configurable.

## 2. Atomic notes and title as API

**The principle.** A permanent note is atomic: it carries exactly one claim. Its heading is not a label but a citable, argument-bearing sentence — precise enough that a wikilink pointing at it, or a single search hit landing on it, tells you exactly what it asserts without needing to read the body. This is "title as API": other notes and future searches interact with the claim through its title, so the title has to carry the full weight of the claim on its own.

**Why.** A note titled "Attention residue" tells you a topic; a note titled "Attention residue drags down performance on a new task after a switch, and the drag scales with how unfinished the old task was" tells you a claim. Only the second is something another note can meaningfully cite, agree with, or dispute. If titles stay vague, links between notes stop being arguments and become vague topic associations instead.

**Where it shows up.** The body's job is to carry all the material behind that claim — mechanisms, causal chains, reasoning, quantitative detail, boundary conditions — in proportion to how much the source material actually supports. Simplifying the prose for clarity is fine; shrinking the claim itself, or inflating it beyond what the source says, is not. Everything written into an atomic note has to trace back to its source note, with one narrow exception: a synthesis sentence that draws a connection to another already-filed note is allowed, but only when it carries an inline wikilink to that note and the connection has been verified by reading that note's full text first.

## 3. Linking discipline

**The principle.** Wikilinks are unidirectional and inline — woven into the sentence where a note actually invokes another note's claim — and there is no "Related notes" section anywhere. A search or recall hit is never itself justification for a link; it's only a candidate. Before a link is created, its target has to be read in full, and the note being written has to actually discuss that target's concept in its own prose.

**Why.** A link is supposed to represent an argumentative relationship — this note relies on, extends, or contradicts that one — not a similarity score. If links were created from recall hits alone, the slip-box would fill up with connections that mean "these two notes came up in the same search" rather than "these two ideas actually relate." That destroys the value of following links to reason through an argument.

**Where it shows up.** Staging notes are never link targets — they aren't finished claims yet, so there's nothing stable to link to. A high similarity score or a shared source between two notes explicitly does not, on its own, constitute grounds for a link.

## 4. Adversarial review

**The principle.** Every claim written into a note — whether freshly drafted or freshly atomized — gets checked by a dedicated review pass that defaults to disbelief: it tries to disprove each claim before accepting it as faithful to its source. Each claim ends up classified into exactly one of three buckets: faithful, factually wrong, or unsubstantiated. "Unsubstantiated" is not a euphemism for "probably wrong" — it's the honest outcome when the review can find neither support nor contradiction, and it is never allowed to serve as grounds for weakening a claim, narrowing a title, or deleting a passage.

**Why.** A reviewer that defaults to trust rubber-stamps plausible-sounding errors. A reviewer that treats "no evidence found" as "wrong" over-deletes material that just hasn't been checked against the right source yet. The three-way split keeps those two failure modes from swallowing each other.

**Where it shows up.** The review agent never writes to any file — it can only report. For factual errors, it proposes a specific correction plus the anchoring text the correction is based on; for unsubstantiated claims, it reports without proposing a fix. Whoever dispatched the review decides, claim by claim, whether to apply a suggested correction — the review agent doesn't act on its own findings. And once a review is done, nothing about it is left behind in the note itself: no review metadata, no flags, no trace in the frontmatter or body.

## 5. The memory loop

**The principle.** A convention is something the user does — a habit of the person operating the plugin, standing outside the plugin's own rules — not a rule the plugin invented for itself. Conventions don't get written directly; they arrive through exactly one channel: a proposal, drafted when a command notices a signal that looks like a user habit worth remembering. There is no shortcut that lets a proposal skip the queue and become a convention on its own. `/zk:adapt` is where proposals get resolved — one at a time, in conversation, with the user deciding each one; the command never decides on the user's behalf. A rejected proposal is simply deleted; deletion is the archival — there's no separate archive of rejected proposals sitting around.

**Why.** If the plugin could silently adopt behavior patterns it noticed, users would lose track of why the plugin started doing something differently. Routing every adoption through an explicit, reviewable proposal — and requiring a human decision for each one — keeps the plugin's behavior explainable and keeps the user in control of what "the way I like things done" actually means for their vault.

**Where it shows up.** Every write-capable command scans existing conventions before doing its work and follows whichever ones apply; it also surfaces the existence of pending proposals in its report, without letting a pending proposal change or block what it's about to do.

## 6. Devlog

**The principle.** Every staging note carries its own devlog — a work journal that mirrors its path, one note to one devlog. If a session gets interrupted mid-thought, the next session loads the log, sees exactly where things stopped, and resumes from there instead of starting over. The log is append-only: nothing written into it is ever deleted or rewritten, except a single status summary at the top of the file, which is the one part that stays rewritable as things progress. When a note is successfully archived into the filed tier, its devlog is deleted at that moment — deletion is the archival, the same as with rejected proposals.

**Why.** The devlog exists to make interrupted work resumable and to keep an honest record of what actually happened during drafting, without cluttering the finished note with process details a reader doesn't need. Once a note is filed, the reasoning trail that got it there is no longer relevant to anyone reading the slip-box — only the finished claim matters, so the process record doesn't follow the note into the filed tier.

**Where it shows up.** Devlogs exist only for staging notes; filed notes never have one. The two lifecycle commands that produce staging notes are the only ones that read or write devlogs — permanent's only devlog action is deleting the one belonging to the note it just archived.

## 7. Pure markdown, no scripts

**The principle.** The plugin ships no hooks, no executable code, and never runs git commands on the user's behalf. Every mechanical discipline — numbering, formatting, index maintenance, file naming — is carried entirely in prose instructions that the model follows through self-discipline, not through code that enforces it automatically.

**Why.** A plugin that runs scripts or hooks has behavior you can't fully read by looking at its files — you'd have to trace what the code does. Keeping everything as instructions the model reads and follows means the plugin's entire behavior is auditable just by reading markdown, and the plugin can move with the vault to any machine without carrying an execution environment. Version control of the vault's contents is left entirely to the user; the plugin doesn't touch git, because the vault's history is the user's business, not the plugin's.

**Where it shows up.** Code blocks throughout the plugin's own files exist only to show CLI invocations or data formats as examples — never as embedded logic meant to run.

## 8. Plugin read-only, vault autonomy

**The principle.** The plugin's own installed files are read-only at runtime — no command or agent ever modifies the plugin's own files; changes to the plugin only happen through its own development process, never through anything a user's session does. On the vault side, state is limited to exactly four categories: the config file, the template files that define note schemas, a hidden directory holding conventions, proposals, and logs, and the notes themselves.

**Why.** Separating "what the plugin is" from "what the vault contains" is what makes the plugin installable into an arbitrary existing vault without disturbing what's already there. If the plugin could rewrite itself based on how a particular vault used it, its behavior would drift unpredictably across installations.

**Where it shows up.** Pre-existing notes in a vault are never moved or rewritten by initialization or by any other command; frontmatter fields the plugin doesn't recognize are left untouched rather than stripped, since they may be doing work for other tools (such as custom properties read by other Obsidian features). Directory layout for every stage is decided entirely through user interview during initialization, never assumed from a default. The config schema itself stays minimal — a field is only added once there's an actual behavior that consumes it.

## 9. Retrieval design

**The principle.** Semantic search, when available, is the main funnel for finding candidate notes, supplemented by exact-term text search that catches literal tokens — identifiers, proper nouns — that semantic matching tends to miss. Whether semantic search is available and permitted is a single configuration flag, and it is the only signal deciding which path retrieval takes. When that flag says semantic search is off, falling back to plain-text search (Grep fallback) is a legitimate, expected path — not an error, and nothing to apologize for in a report. When the flag says semantic search is on, though, a failure in the semantic search tool has to stop the operation and get reported; it must never be silently papered over by quietly falling back to text search, because a tool that's supposed to work but doesn't is a bug worth surfacing, not one worth hiding.

**Why.** Making the flag the sole routing signal keeps retrieval behavior predictable — there's exactly one place to look to know why a search took the path it took. The read-the-full-text rule underneath both paths — any hit is a candidate, not evidence; nothing gets cited or linked until its full text has actually been read — exists because similarity scores and match counts describe how a search engine ranked something, not whether it actually says what you need it to say.

**Where it shows up.** Index maintenance happens at exactly two points: an idempotent reindex when a lifecycle command starts working on a staging note, and an incremental reindex when permanent finishes archiving. No other command touches the index. When semantic search is disabled entirely, there's no index to maintain, so both maintenance points are skipped.

## 10. Agent boundaries

**The principle.** The three subagents each have one job and a hard boundary around it. The research agent reads a source in full and reports back — read-only, no writes — and if it can't actually obtain the source's full text, it aborts rather than substituting a summary, a review, or anything from its own training knowledge; using secondhand material in place of the real source is the one line it will not cross. The atomizer works fully autonomously from start to finish — it makes every judgment call itself (where to split a note, which topic tag to use, when to create a new topic folder, how to resolve a naming collision) and never stops mid-task to ask a question. The adversarial review agent is read-only and reports only — see the discussion above for what happens to its findings.

**Why.** Each of these boundaries protects a different thing. Research's boundary protects the slip-box from secondhand distortion — a paraphrase of a paraphrase drifts from what the source actually said. The atomizer's autonomy exists because atomizing is a high-volume, mechanical-enough decision space that stopping to ask about every split or every tag choice would make the whole pipeline unusable; the guardrail against that autonomy going wrong is the adversarial review that follows it. Review's read-only boundary keeps the decision to actually change a note in the hands of whichever command dispatched the review, rather than letting an agent designed to assume the worst also be the one deciding what to do about it.
