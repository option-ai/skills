---
name: visual-plan
description: >-
  Turn ordinary text plans into a rich, interactive, self-contained HTML
  document — with diagrams, file maps, annotated code, collapsible sections,
  and open questions — instead of a flat Markdown plan rendered in the terminal.
metadata:
  visibility: exported
---

# Visual Plan (Self-Authored HTML)

This skill turns the implementation plan you would normally dump into chat as
Markdown into a **single self-contained `.html` file** the user opens in their
browser. No MCP server, no hosted service, no dependencies — just one HTML file
you write to disk and open. The goal is a scannable, navigable review surface
that's far easier to approve than a wall of terminal Markdown.

`/visual-plan` is the entry point. When the user already has a text plan (pasted,
in a file, or earlier in the chat), use it as source material and render it as
HTML rather than starting over.

## When To Use

Build a visual plan whenever the plan is better as a reviewable artifact than a
chat paragraph: multi-file, ambiguous, risky, architecture-heavy, data-heavy, or
UI-heavy work — plus modest work like a single UI surface with states, a small
workflow, a before/after change, or a component/API/data-shape decision that
needs alignment before code.

Skip it for truly trivial, unambiguous work — typos, one-line fixes, a single
well-specified function — and just make the change. Never pad a plan with filler
and never ship a single-step plan.

## Plan Discipline

- **Research before you draft.** Read the real files, actions, schema, and
  patterns first; name actual files, symbols, and data shapes instead of
  inventing them. Lead with reuse: for each step, name what it reuses (existing
  actions, schema, components, helpers) before what it adds. Delegate wide
  exploration to a sub-agent when useful.
- **Decide the hard-to-reverse bets first.** For non-trivial backend, data, or
  API work, call out decisions expensive to undo once data or callers depend on
  them — wire format, public ids, data-model shape, auth and ownership
  boundaries — and get those right in the plan. Then scope the smallest first cut
  that proves the approach, stating what's in and what's explicitly deferred.
- **Keep examples at the right altitude.** When the idea is a broad framework or
  operating-model change, don't collapse it into the first concrete example.
  Separate the core abstraction from motivating examples, and label examples as
  examples unless they are the whole requested scope.
- **Make it stand alone.** A reader who never saw the chat should understand the
  plan. If the user pasted an existing plan, rewrite it as a clean standalone
  proposal — no revision language ("unlike the previous version", "this revision
  changes…").
- **Planning is read-only.** Make no source edits while building the plan. Start
  editing only after the user approves the direction.
- **Clarify vs. assume.** Don't ask how to build it — explore and present the
  approach and options in the plan. Ask a clarifying question only when an
  ambiguity would change the design and you can't resolve it from the code; batch
  2–4 high-leverage questions via the normal ask-user flow. Otherwise state the
  assumption explicitly and proceed, and keep anything unresolved in the plan's
  Open Questions section.
- **The plan is the approval gate.** After surfacing the HTML file, ask the user
  to review and approve before you write code, and name which files/areas the
  work touches. Presenting the plan and requesting sign-off *is* the approval
  step — don't tack on a separate "does this look good?" question.

## Core Workflow

1. Gather context: inspect the codebase, delegate wide exploration when useful,
   and ask native clarifying questions only when truly needed. If a source plan
   exists, use its exact text — don't invent source text.
2. Write a single self-contained HTML file to disk. Default location:
   `/tmp/visual-plan-<slug>.html` (or a repo-local path like
   `plans/<slug>.html` if the user wants it checked in). Follow the **HTML
   Authoring Contract** below.
3. Open it for the user. On macOS run `open <path>`; otherwise print the
   `file://` path. Always print the path in chat so a text-only host still has a
   click target.
4. Ask the user to review and approve. Iterate by editing the same HTML file in
   place (don't regenerate from scratch and lose sections).
5. For high-stakes plans (architecture, backend, data, multi-file, risky), run a
   quick adversarial self-review of the *written plan* (not a re-research):
   spawn one skeptical reviewer to find weak/missing/wrong parts — unanchored
   steps, implicit hard-to-reverse decisions, option-menus that should commit,
   obvious missing decisions, padding. Apply clear-cut fixes; route genuine
   judgment calls back to the user in Open Questions.

## HTML Authoring Contract

Produce ONE file with CSS, JS, SVG icons, and brand logos all inlined and no web
fonts (use the system font stack). The **only** permitted external dependency is
**Mermaid**, loaded from a pinned CDN for diagram rendering (see the Diagrams
bullet). Everything else stays inlined; the page must open and fully function
(theme toggle, commenting, annotations, file trees) from a `file://` URL — and
the diagrams render too whenever the network is reachable (local file with
internet, or after deploying to Cloudflare). If the user asks for a strictly
offline file, vendor Mermaid by inlining its minified bundle instead of the CDN.

Required structure and behavior:

- **Document head:** `<!doctype html>`, `<meta charset>`, `<meta name="viewport">`,
  `<meta name="color-scheme" content="dark light">`, and a descriptive `<title>`.
- **Dark-mode first, with a toggle.** The page loads in dark mode by default
  (don't defer to `prefers-color-scheme` for the default — dark is the default).
  A header toggle (sun/moon Tabler icon) flips between dark and light; persist the
  choice to `localStorage` and apply it before first paint to avoid a flash.
- **Claude Code color palette.** Use the Anthropic/Claude palette via CSS
  variables — clay-coral accent, warm near-black surfaces in dark, cream paper in
  light (exact tokens are defined in `references/html-template.md`; reuse them
  verbatim). Do not invent a generic blue/gray scheme. **Dark is the polished
  default** — it must look intentional, not an inverted afterthought. Do NOT fill
  large surfaces (diagram nodes, cards) with the soft accent tint; that reads as
  washed-out. Reserve `--accent`/`--accent-soft` for small emphasis (pills,
  borders, badges, "our code" highlights) and build diagrams from elevated
  neutral surfaces with thin accent borders.
- **Tabler icons, inlined.** Use [Tabler](https://tabler.io/icons) icons for all
  UI affordances (theme toggle, comment/annotation, copy, collapse chevrons,
  section markers). Inline each icon's SVG (Tabler is MIT-licensed `currentColor`
  stroke SVG) — never link a CDN or icon font. Define them once as `<symbol>`s in
  a hidden `<svg>` sprite and reference with `<use>`.
- **Brand logos for named products/services.** Whenever the plan names a
  recognizable product, network, or service (WhatsApp, Telegram, Signal, Discord,
  Slack, Instagram, Messenger, Google, GitHub, LinkedIn, X, …) — in diagram nodes,
  integration lists, coverage tables, or option cards — render its **brand logo**
  next to the name, not a bare text box. Use the inlined brand-logo sprite in
  `references/html-template.md` (Simple Icons paths, MIT, `fill="currentColor"`
  tinted with the brand color). Reuse the sprite's logos; if a needed logo is
  missing, add it by copying the exact Simple Icons path — never hand-draw or
  approximate a logo (a wrong logo is worse than none; fall back to a neutral
  badge + name only when no real logo is available).
- **No header bar — a title row instead.** Don't render a sticky header. At the
  top of the body put a title row: the plan name (h1) and one-line outcome on the
  **left**, and the **repository and date on the right** (you fill the real repo
  and today's date), aligned with space-between. Stacks on very narrow screens.
- **The plan is centered in the window; the TOC floats left, out of flow.** The
  body column is centered in the viewport (`margin: 0 auto`). The table-of-contents
  nav is positioned **absolutely/fixed to the left** so it does not occupy layout
  space or push the plan off-center — the plan stays centered, the TOC sits in the
  left margin and hides on narrow screens.
- **Floating dock that expands (one surface, not a second window).** All controls
  live in a floating pill at the bottom: comment, feedback, **copy-feedback**,
  theme, and keyboard-shortcuts. The dock **sizes to its buttons** (`width:
  fit-content`) — no dead/empty width. Opening the compose box or the feedback
  chat **expands the dock itself upward with an animation** — both live *inside*
  the dock; never render a separate floating panel/window detached above it.
- **Collapsible sections** (`<details>`/`<summary>`, styled with a Tabler chevron)
  so reviewers can scan headings then expand. Keep the most important sections
  open by default.
- **Steps as discrete, numbered cards**, each naming the real files/symbols it
  touches and what it reuses before what it adds.
- **Diagrams via Mermaid — and it must be valid.** Render flow, sequence, state,
  ER, and dependency diagrams with Mermaid (`<pre class="mermaid">…</pre>`, pinned
  CDN ESM build, theme-wired). Bad Mermaid syntax is the #1 failure mode, so write
  it carefully: **quote every node label containing spaces, punctuation,
  parentheses, slashes, or code** (`A["benchy run"]` not `A[benchy run]`); stick to
  common diagram types and basic edges; one statement per line; no raw `<`/`>` in
  labels. The template renders each diagram individually and **falls back to showing
  the source on a parse error** (never Mermaid's bomb graphic), but aim to render
  cleanly. Keep the inlined CSS brand-logo kit (`.diagram`/`.dnode`/`.dbar`) for
  **architecture/network maps where brand logos matter** (it has no syntax failure
  mode). Never leave a relationship as plain prose when a diagram makes it legible;
  build nodes from elevated neutral surfaces, not flat accent-tinted boxes.
- **Code / diffs** in styled `<pre><code>` blocks with a subtle file-path label.
  Show added/removed lines with color, not just `+`/`-`.
- **File map / tree as a rendered tree, never plain monospace text.** Use the
  template's file-tree renderer: a nested `<ul class="filetree">` with folder/file
  Tabler icons, indent guide lines, and `data-new`/`data-edit` status pills — so it
  reads like an IDE explorer, not a code block.
- **Open Questions as answerable chips.** Near the bottom, render each open
  question as selectable **answer buttons** (`.qa` / `.qopt`), one option per real
  choice, with the recommended option marked `data-rec` and **pre-selected by
  default** so the user approves by exception (plus an optional write-in). The
  user's picks persist and round-trip to the agent under a "Decisions" section in
  the copied feedback. Don't use plain "Recommended default: …" prose.
- **Annotation / feedback feature (required on every plan).** The reviewer must be
  able to comment on any part of the plan, and each comment must be anchored so
  the agent knows exactly what it points at. Implement per the **Annotation
  Contract** below; the working implementation lives in
  `references/html-template.md` — reuse it, don't reinvent it.

### Annotation Contract

- **Anchorable blocks.** Give every meaningful block (sections, step cards,
  diagrams, code blocks, table rows, open questions) a stable
  `data-anchor-id="kebab-id"` and a human-readable `data-anchor-label`. The id
  must be stable across edits so feedback maps back reliably — derive it from the
  content, not a random/incrementing counter.
- **One comment flow: type → optionally pin → save.** There is a single `c`
  command (dock "Comment" button or the `c` key) — no separate general-comment
  command. Pressing it **expands the dock into a compose box** (same surface as the
  feedback chat, not a floating window) with a **target chip** defaulting to "Whole
  plan". The reviewer types, then optionally **pins a target**, then saves:
  - **Click a section** to pin the comment to that block (chip shows its label).
  - **Select text** inside the plan to pin to that exact span — the selection is
    captured as a quote and **marked/highlighted** in the doc (fine-grained, not
    just whole-section).
  - **Save without pinning** → the comment applies to the **whole plan** (stored as
    `anchor-id: __general__`, exported `[general]`).
  While the composer is open the body is in "assigning" state: blocks highlight on
  hover with an `outline` + `outline-offset` gap; nothing highlights while just
  reading. The chip's ✕ unpins back to whole-plan.
- **Fine-grained text marks.** Text-anchored comments store the containing block's
  `anchor-id` plus character offsets + the quote, and are visually marked with the
  **CSS Custom Highlight API** (`::highlight(vp-mark)`), reconstructed on load and
  degrading gracefully where unsupported. Block-anchored comments get a count badge
  on the block. Clicking a chat entry scrolls to (and flashes) its block or span.
- **Feedback is a borderless chat that expands the dock.** Opening feedback (`f`)
  grows the dock upward (animated) into a compact, scrollable list of all comments
  — whole-plan, block, and text. Render entries as plain stacked **chat messages
  with no border or card box around them** (small accent label, optional quoted
  line, body, delete on hover). It never shifts or covers the document.
- **Copy from the dock + ⌘/Ctrl+C.** A copy-feedback button sits directly on the
  dock (no need to open the chat first), and `⌘/Ctrl+C` copies all feedback when
  nothing is text-selected (so normal copy still works). Confirm with a small toast.
- **Keyboard shortcuts for everything, and visibly shown.** Each dock button
  carries its shortcut as an always-visible `kbd` chip (not just a hover tooltip),
  and a `?` overlay lists them all. At minimum: `c` new comment, `f` feedback chat,
  `t` theme, `⌘/Ctrl+C` copy feedback, `?` shortcuts overlay, `Esc` close
  composer/overlay/dock, `⌘/Ctrl+Enter` save the open comment.
- **Animate opening.** The dock feedback chat and the composer animate in
  (grow/fade), not snap.
- **Persistence.** Store comments in `localStorage` keyed by the plan title so they
  survive reload. No backend, no network.
- **Round-trip to the agent.** The dock copy button (and `⌘/Ctrl+C`) serializes a
  compact, agent-readable block with two sections: **Decisions** (the chosen Open
  Questions answers — `question → answer`) and **Comments** (each as `[anchor-id: …]
  (label)` for block/text targets, text targets also carrying the `quote` of the
  marked span, or `[general]` for whole-plan notes, plus the body). The user pastes
  it back; the agent applies the decisions and uses the anchor id (and quote for
  text) to locate each comment for targeted edits. Offer a "Download feedback"
  fallback saving the same payload as a `.md` file.
- When the user pastes feedback back, treat the `anchor-id` (and quoted span for
  text comments) as the source of truth for *where* each comment belongs, address
  each one with a targeted edit, and preserve every other section.

Quality bar:

- Real content over chrome. Don't add tabs, canvases, or animations that don't
  earn their place. Visual where visuals help; structured where structure helps.
- **Avoid the over-styled "AI" look.** Don't wrap every block in a rounded,
  accent-tinted, colored-border card. Callouts are squared (no rounding); reserve
  tints and rounding, keep borders restrained, and let typography and spacing carry
  the hierarchy.
- Self-contained and robust: opening the file twice, or from any directory, must
  work identically.
- Keep the prose at the quality of a serious technical plan — outcome-first,
  grounded in real code, no marketing tone.

See `references/html-template.md` for a copy-pasteable starting skeleton and
`references/document-quality.md` for the written-plan quality bar.

## Iterating On Feedback

When the user comments, edit the existing HTML file in place with targeted edits
— preserve every existing section and only change what the feedback touches. Re-open
or tell the user to refresh. When scope shifts, update the HTML (the document is
the source of truth, not the chat) and keep it standalone.

## Deploying The Plan To Cloudflare (Shareable Link)

By default the plan is a local file. When the user wants a **shareable link** —
to send the plan to a teammate, review it on another device, or get a public URL
— deploy the single HTML file to Cloudflare Workers using a **temporary account**,
which needs no Cloudflare signup, login, OAuth, or API token up front. See
<https://blog.cloudflare.com/temporary-accounts/>.

Offer this only when sharing is wanted; for solo local review, the `file://` path
is enough. Deploying publishes the plan to the public internet at a hard-to-guess
URL — confirm with the user first if the plan covers private or unreleased work.

Steps:

1. Put the plan HTML in its own directory as `index.html`, e.g.
   `/tmp/visual-plan-<slug>/index.html` (a Worker assets deploy serves a
   directory, so move/copy the file there and name it `index.html`).
2. Write a minimal `wrangler.jsonc` next to it that serves the directory as
   static assets — no Worker script needed:

   ```jsonc
   {
     "name": "visual-plan-<slug>",
     "compatibility_date": "2025-01-01",
     "assets": { "directory": "." }
   }
   ```

3. Deploy with the temporary-account flag (run from that directory; uses `npx` so
   no global install is required):

   ```bash
   npx wrangler@latest deploy --temporary
   ```

   If you run `wrangler deploy` without auth and without the flag, Wrangler prints
   a notice about `--temporary`; just rerun with the flag. The first run
   provisions a throwaway account, mints a token Wrangler caches locally, and
   prints both a **live preview URL** (the deployed plan) and a **claim URL**.
4. Give the user **both** URLs in chat: the live plan URL to open/share, and the
   claim URL with a note that the deployment stays live for **60 minutes** and
   they can click claim to make the account/Worker permanently theirs (otherwise
   it auto-deletes). Quote the 60-minute window so the expiry is explicit.
5. To push edits within the window, update `index.html` and rerun
   `npx wrangler@latest deploy --temporary` from the same directory — it reuses
   the cached temporary credentials and redeploys to the same URL.

If Wrangler errors (network, version, or a deploy failure), report what happened
and fall back to handing over the local `file://` path; don't loop on retries.

## Notes

- By default this skill writes plain local files — nothing leaves the machine.
  The single HTML file *is* a shareable artifact: the user can send it or check it
  in. When the user wants a hosted link, deploy it to Cloudflare with a temporary
  account (see **Deploying The Plan To Cloudflare**).
- If the user explicitly wants the plan checked into source control, write it
  under `plans/<slug>.html` instead of `/tmp`.
