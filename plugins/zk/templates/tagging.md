---
when: when setting tags on a new note, or normalizing tags while filing a note (staging to filed-doc)
---
# tagging

Factory kind-tag scheme, seeded by `/zk:init` (adjusted there to the layout and
tag habits agreed in the interview). `<step>.place` below refers to the
directories configured in `.zettel.json`.

- Staging note in `fleeting.place`: tags exactly `[fleeting]`.
- Staging note in `literature.place`: tags exactly `[literature]`.
- Filed atomic note (filed-atomic) in `permanent.place/<top>/`: tags exactly
  `[zettelkasten, <top>]`.
- Filed document (filed-doc) in `reference.place/<top>/`: tags exactly
  `[<kind>, <top>]`, where `<kind>` is the note's staging kind (`fleeting` or
  `literature`) — its frontmatter `type` value at filing time. A later
  `type: reference` rewrite (tool notes confirmed at filing) does not change
  the tags.
- Never rewrite one kind into another (e.g. a filed-doc that entered as a
  literature note keeps `literature`, it never becomes `zettelkasten`).
- `<top>` must match the topic directory the note actually lives in.

Examples (factory default places):

```yaml
tags: [fleeting]                  # inbox/idea.md (staging)
tags: [literature]                # literature/paper.md (staging)
tags: [zettelkasten, security]    # zettelkasten/security/xss-basics.md (filed-atomic)
tags: [literature, security]      # reference/security/paper.md (filed-doc)
```
