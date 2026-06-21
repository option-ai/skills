# visual-plan

A self-authored [Agent Skill](https://code.claude.com/docs/en/skills) that turns
an ordinary text plan into a **rich, interactive, self-contained HTML document** —
instead of a flat wall of Markdown in the terminal.

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

## Install

**Via [skills.sh](https://skills.sh):**
```bash
npx skills add option-ai/visual-plan-skill
```

**Via Claude Code plugins:**
```
/plugin marketplace add option-ai/visual-plan-skill
/plugin install visual-plan
```

**Manually:** copy `skills/visual-plan/` into your `~/.claude/skills/` directory
(or `.claude/skills/` in a project).

## Use

```
/visual-plan <describe the change, or paste an existing plan>
```

The agent researches the codebase, writes the HTML plan, deploys it to Cloudflare,
and hands you the link. Review it, comment inline, copy the feedback back into chat,
and iterate.

## Layout

```
.claude-plugin/        plugin + marketplace manifests (Claude Code)
skills/visual-plan/
  SKILL.md             the skill instructions
  references/
    html-template.md   the copy-pasteable HTML template (source of truth)
skills.sh.json         skills.sh directory metadata
```

## License

MIT
