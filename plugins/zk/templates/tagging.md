---
when: when setting tags on a new note, or normalizing tags while filing a note (staging to filed)
---
# tagging

Factory kind-tag scheme, seeded by `/zk:init` (adjusted there to the layout and
tag habits agreed in the interview). `<step>.place` below refers to the
directories configured in `.zettel.json`.

- Staging note in `fleeting.place`: tags exactly `[fleeting]`.
- Staging note in `literature.place`: tags exactly `[literature]`.
- Filed permanent note in `permanent.place/<top>/`: tags exactly `[zettelkasten, <top>]`.
- Filed source note in `reference.place/<top>/`: tags exactly `[<kind>, <top>]`,
  where `<kind>` is the note's frontmatter `type` value (`fleeting` or `literature`).
- Never rewrite one kind into another (e.g. a filed literature source note keeps
  `literature`, it never becomes `zettelkasten`).
- `<top>` must match the topic directory the note actually lives in.

Examples (factory default places):

```yaml
tags: [fleeting]                  # inbox/idea.md (staging)
tags: [literature]                # literature/paper.md (staging)
tags: [zettelkasten, security]    # zettelkasten/security/xss-basics.md (filed permanent)
tags: [literature, security]      # reference/security/paper.md (filed source note)
```
