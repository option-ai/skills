# Option Skills

[![skills.sh](https://skills.sh/b/option-ai/skills)](https://skills.sh/option-ai/skills)

[Agent Skills](https://code.claude.com/docs/en/skills) by Option, for Claude Code
(and other agents). Browse them, or install one with:

```bash
npx skills add option-ai/skills
```

Or as a Claude Code plugin marketplace:

```
/plugin marketplace add option-ai/skills
/plugin install visual-plan
```

You can also copy any `skills/<name>/` folder straight into `~/.claude/skills/`.

---

## Skills

### `visual-plan`

Turns an ordinary text plan into a **rich, interactive, self-contained HTML
document** — instead of a flat wall of Markdown in the terminal.

**Live demo:** https://visual-plan-benchy-capture.abdulhdr1.workers.dev

The agent writes one `.html` file (CSS + JS inlined, no build step) with:

- **Dark-first** Claude-style theme with a light toggle
- **Diagrams** via Mermaid + a CSS brand-logo kit for architecture/network maps
- **Rendered file trees** (collapsible folders, status pills), annotated code & diffs
- **Anchored commenting** — select any text or a Mermaid node → "Comment this
  section"; comments round-trip back to the agent for targeted edits
- **Open-question answer chips** (recommended pre-selected) that export as decisions
- A floating control dock, scroll-spy TOC, reading-progress bar, copy-code buttons,
  image lightbox, print/PDF stylesheet, and a versioned feedback timeline
- **One-command Cloudflare deploy** to a shareable URL (temporary account, no signup)

Use it: `/visual-plan <describe the change, or paste an existing plan>`. The agent
researches the codebase, writes the HTML plan, deploys it to Cloudflare, and hands
you the link. Review it, comment inline, copy the feedback back into chat, iterate.

---

## Repo layout

```
.claude-plugin/marketplace.json   Claude Code marketplace (lists every skill as a plugin)
skills.sh.json                    skills.sh directory metadata
skills/
  visual-plan/
    SKILL.md                      the skill instructions
    references/html-template.md   the copy-pasteable HTML template (source of truth)
  <next-skill>/                   drop new skills here
```

Adding a skill later: create `skills/<name>/SKILL.md`, then add an entry to
`marketplace.json` (`plugins[]`) and `skills.sh.json` (`groupings[].skills`).

## License

MIT
