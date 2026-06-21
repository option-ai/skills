# HTML plan template

A copy-pasteable, **dark-mode-first** interactive plan with: header aligned to the
content column, a **floating dock** of controls, **comment mode** (chrome appears
only when you opt in), point-and-click + **general** annotations, **keyboard
shortcuts** with a help overlay, **Mermaid** diagrams (theme-aware), a real
**file-tree renderer**, inlined Tabler icons, and inlined brand logos.

Everything is inlined except **Mermaid**, which loads from a pinned CDN. The page
works from `file://`; diagrams render whenever the network is reachable (and after
deploying to Cloudflare). For a strictly offline file, vendor Mermaid's minified
bundle in place of the CDN import.

Keep **verbatim**: the `:root` tokens, the pre-paint theme script, both icon
sprites, the Mermaid module script, and the app script. They are the palette,
logos, diagram, and feedback contract. Per block you want commentable, add a
stable `data-anchor-id` + `data-anchor-label`.

## What this revision fixes

- **File maps render as a tree**, not monospace text (folder/file icons, guide
  lines, new/edit pills). **Diagrams use Mermaid.**
- **General comments** (dock button / `g`) attach to the whole plan, no anchor.
- **Header and body share one centered fixed-width column.**
- **Highlight boxes only in comment mode** — reading is clean.
- **Keyboard shortcuts for everything** (`?` shows them).
- **Controls live in a floating dock**, not the header.

## The template

```html
<!doctype html>
<html lang="en" data-theme="dark">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="color-scheme" content="dark light">
<title>PLAN TITLE — Visual Plan</title>
<script>
  (function () {
    try { document.documentElement.setAttribute('data-theme', localStorage.getItem('vp-theme') || 'dark'); }
    catch (e) { document.documentElement.setAttribute('data-theme', 'dark'); }
  })();
</script>
<style>
  /* ===== Claude Code palette ===== */
  :root { --maxw: 1080px; --pad: 22px; --panel: 360px; }
  :root[data-theme="dark"] {
    --bg: #1a1916; --bg-elev: #211f1c; --card: #28251f; --card-2: #2f2b25;
    --line: #3a352e; --line-strong: #4a443b; --fg: #ece8df; --muted: #9c9588;
    --accent: #d97757; --accent-soft: #36281f; --accent-line: #6b4733; --on-accent: #1a1916;
    --add: #84c08a; --add-bg: #1d2a1d; --del: #e08a82; --del-bg: #2c1c1a;
    --warn: #d9b266; --warn-bg: #2a2415; --shadow: 0 12px 40px rgba(0,0,0,.5);
  }
  :root[data-theme="light"] {
    --bg: #f7f5ef; --bg-elev: #ffffff; --card: #ffffff; --card-2: #f3f1e8;
    --line: #e7e3d7; --line-strong: #d8d3c4; --fg: #2c2a24; --muted: #6f6a5d;
    --accent: #c15f3c; --accent-soft: #f7e9e1; --accent-line: #e3b9a6; --on-accent: #ffffff;
    --add: #2f7d3a; --add-bg: #e9f3e7; --del: #c0392b; --del-bg: #fbeae7;
    --warn: #97751f; --warn-bg: #f7eecb; --shadow: 0 12px 40px rgba(60,50,40,.18);
  }
  * { box-sizing: border-box; }
  html { scroll-behavior: smooth; }
  body { margin: 0; background: var(--bg); color: var(--fg); transition: padding-right .22s ease;
    font: 16px/1.65 -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; }
  code, pre { font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace; }
  a { color: var(--accent); }
  h2 { font-size: 20px; margin: 0 0 12px; padding-bottom: 6px; border-bottom: 1px solid var(--line); }
  .icon { width: 1.1em; height: 1.1em; fill: none; stroke: currentColor; stroke-width: 2;
    stroke-linecap: round; stroke-linejoin: round; vertical-align: -.16em; }
  .brand { width: 18px; height: 18px; fill: currentColor; flex: none; }

  /* ===== Layout — plan centered in the window; TOC floats left (absolute) ===== */
  :root { --body: 740px; }
  .wrap { max-width: var(--body); margin: 0 auto; padding: 30px var(--pad) 130px; }
  /* TOC is taken out of flow so the plan stays centered in the viewport */
  nav.toc { position: fixed; top: 96px; width: 176px; left: calc(50vw - var(--body) / 2 - 176px - 28px);
    font-size: 13px; max-height: calc(100vh - 140px); overflow: auto; }
  nav.toc a { display: block; padding: 5px 0 5px 10px; color: var(--muted); text-decoration: none; border-left: 2px solid transparent; }
  nav.toc a:hover { color: var(--accent); border-left-color: var(--accent); }
  @media (max-width: 1180px) { nav.toc { display: none; } }
  main { min-width: 0; }
  main > section { margin-bottom: 34px; scroll-margin-top: 24px; }

  /* ===== Plan header: metadata row on top, title, full-width description (no divider) ===== */
  .plan-head { margin: 4px 0 32px; }
  .plan-head .meta { display: flex; flex-wrap: wrap; align-items: center; gap: 6px 14px; font-size: 12.5px; color: var(--muted); margin-bottom: 13px; }
  .plan-head .meta .m-item { display: inline-flex; align-items: center; gap: 5px; }
  .plan-head .meta .m-item .icon { width: 14px; height: 14px; opacity: .8; }
  .plan-head .meta a { color: var(--muted); text-decoration: none; }
  .plan-head .meta a:hover { color: var(--accent); }
  .plan-head h1 { margin: 0 0 10px; font-size: 26px; letter-spacing: -.012em; }
  .plan-head .outcome { margin: 0; color: var(--fg); font-size: 15px; max-width: none; }
  /* version badge + timeline popover */
  .version { position: relative; }
  .version > .vbtn { display: inline-flex; align-items: center; gap: 4px; cursor: pointer; font: inherit; font-size: 12px;
    color: var(--accent); background: var(--accent-soft); border: 0; border-radius: 999px; padding: 1px 9px; }
  .version > .vbtn:hover { filter: brightness(1.08); }
  .version-pop { position: absolute; top: 130%; left: 0; z-index: 40; min-width: 230px; background: var(--bg-elev);
    border: 1px solid var(--line); border-radius: 11px; box-shadow: var(--shadow); padding: 6px; }
  .version-pop[hidden] { display: none; }
  .version-pop .vh { font-size: 11px; text-transform: uppercase; letter-spacing: .05em; color: var(--muted); padding: 5px 9px; }
  .version-pop .vrow { display: flex; align-items: baseline; gap: 8px; padding: 6px 9px; border-radius: 7px; text-decoration: none; color: var(--fg); font-size: 13px; }
  .version-pop .vrow:hover { background: var(--card-2); }
  .version-pop .vrow .vn { font-weight: 600; color: var(--accent); }
  .version-pop .vrow .vd { color: var(--muted); font-size: 11.5px; margin-left: auto; }
  .version-pop .vrow.current { background: var(--accent-soft); }

  /* ===== Subtle badges (foglamp-style: small, low-contrast, dot/ghost — never loud pills) ===== */
  .tag { display: inline-flex; align-items: center; gap: 4px; font-size: 11px; line-height: 1.6; border-radius: 999px;
    padding: 0 8px; background: var(--card-2); color: var(--muted); border: 1px solid var(--line); }
  .tag::before { content: ""; width: 5px; height: 5px; border-radius: 50%; background: currentColor; opacity: .9; }
  .tag.ok { color: var(--add); } .tag.warn { color: var(--warn); } .tag.info { color: var(--accent); } .tag.risk { color: var(--del); }

  /* ===== Blocks ===== */
  .card { background: var(--card); border: 1px solid var(--line); border-radius: 12px; padding: 16px; }
  /* JS sets --cols to a BALANCED count so the last row is never an orphan (4 -> 2x2) */
  .grid { display: grid; gap: 12px; grid-template-columns: repeat(var(--cols, 2), minmax(0, 1fr)); margin: 14px 0; }
  @media (max-width: 620px) { .grid { grid-template-columns: 1fr; } }
  .step { display: flex; gap: 13px; align-items: flex-start; margin: 12px 0; }
  .step .num { flex: none; width: 26px; height: 26px; border-radius: 8px; background: var(--accent-soft);
    color: var(--accent); display: grid; place-items: center; font-weight: 700; font-size: 13px; border: 1px solid var(--accent-line); }
  .files { font-size: 13px; color: var(--muted); margin-top: 5px; }
  .files code { background: var(--card-2); border: 1px solid var(--line); color: var(--fg); padding: 1px 6px; border-radius: 5px; font-size: 12.5px; }
  details { border: 1px solid var(--line); border-radius: 10px; padding: 0 15px; margin: 11px 0; background: var(--bg-elev); }
  details[open] { padding-bottom: 13px; }
  summary { cursor: pointer; padding: 13px 0; font-weight: 600; list-style: none; display: flex; align-items: center; gap: 9px; }
  summary::-webkit-details-marker { display: none; }
  summary .chev { color: var(--muted); transition: transform .15s; }
  details[open] summary .chev { transform: rotate(90deg); }
  pre:not(.mermaid) { background: var(--bg-elev); border: 1px solid var(--line); border-radius: 10px; padding: 13px; overflow: auto; font-size: 13px; line-height: 1.55; }
  .filelabel { font-size: 12px; color: var(--muted); margin: 10px 0 -4px; display: flex; align-items: center; gap: 6px; }
  .diff .add { background: var(--add-bg); color: var(--add); display: block; margin: 0 -13px; padding: 0 13px; }
  .diff .del { background: var(--del-bg); color: var(--del); display: block; margin: 0 -13px; padding: 0 13px; }
  /* squared off (no rounding) — cleaner, less "AI card" */
  .callout { border-left: 3px solid var(--warn); background: var(--warn-bg); padding: 11px 15px; border-radius: 0; }
  table { border-collapse: collapse; width: 100%; font-size: 14px; }
  th, td { border: 1px solid var(--line); padding: 7px 11px; text-align: left; }
  th { background: var(--card-2); }

  /* ===== Open questions — answer chips, recommended pre-selected ===== */
  .qa { margin: 16px 0; }
  .qa + .qa { padding-top: 16px; border-top: 1px solid var(--line); }
  .qa .q-text { font-weight: 600; margin-bottom: 9px; }
  .qa .q-opts { display: flex; flex-wrap: wrap; gap: 8px; }
  .qopt { font: inherit; font-size: 13px; cursor: pointer; background: var(--card); color: var(--fg);
    border: 1px solid var(--line-strong); border-radius: 7px; padding: 7px 12px; display: inline-flex; align-items: center; gap: 7px; }
  .qopt:hover { border-color: var(--accent); }
  .qopt.sel { background: var(--accent); color: var(--on-accent); border-color: var(--accent); }
  .qopt .rec-tag { font-size: 10px; opacity: .7; }
  .qa .q-other { margin-top: 9px; }
  .qa .q-other input { width: 100%; max-width: 380px; background: var(--card); color: var(--fg);
    border: 1px solid var(--line); border-radius: 7px; padding: 7px 10px; font: inherit; font-size: 13px; }
  .qa .q-other input:focus { outline: none; border-color: var(--accent); }

  /* ===== Mermaid ===== */
  /* color:transparent hides the raw source until JS renders the SVG (or a fallback); no flash */
  .mermaid { background: var(--bg-elev); border: 1px solid var(--line); border-radius: 12px; padding: 16px; margin: 14px 0; text-align: center; overflow: auto; color: transparent; min-height: 40px; }
  .mermaid.done { color: inherit; }
  .mermaid svg { max-width: 100%; height: auto; }
  .mermaid-fallback { text-align: left; }
  .mermaid-note { font-size: 12px; color: var(--warn); margin-bottom: 8px; }
  .mermaid-fallback pre { background: var(--card); border: 1px solid var(--line); border-radius: 8px; padding: 10px; margin: 0; white-space: pre; overflow: auto; font-size: 12.5px; color: var(--fg); }

  /* ===== Brand-logo diagram kit (for product/network maps) ===== */
  .diagram { border: 1px solid var(--line); border-radius: 12px; background: var(--bg-elev); padding: 18px; margin: 14px 0; }
  .dlayer { display: grid; gap: 10px; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); margin-bottom: 10px; }
  .dnode { background: var(--card); border: 1px solid var(--line-strong); border-radius: 10px; padding: 11px 13px; }
  .dnode .t { font-weight: 600; display: flex; align-items: center; gap: 8px; }
  .dnode .s { color: var(--muted); font-size: 12.5px; margin-top: 3px; }
  .dbar { background: var(--card); border: 1px solid var(--line-strong); border-radius: 10px; padding: 12px 14px; text-align: center; margin-bottom: 10px; }
  .dbar .t { font-weight: 600; } .dbar .s { color: var(--muted); font-size: 12.5px; margin-top: 2px; }
  .dbar.ours { border-color: var(--accent); box-shadow: inset 0 0 0 1px var(--accent-soft); } .dbar.ours .t { color: var(--accent); }
  .dflow { text-align: center; color: var(--muted); margin: -2px 0 6px; }

  /* ===== File tree ===== */
  .filetree, .filetree ul { list-style: none; margin: 0; padding: 0; }
  .filetree { font-size: 13.5px; border: 1px solid var(--line); border-radius: 12px; background: var(--bg-elev); padding: 14px 16px; }
  .filetree ul { margin-left: 9px; padding-left: 15px; border-left: 1px solid var(--line); }
  .filetree li { padding: 3px 0; }
  .filetree li > span { display: inline-flex; align-items: center; gap: 6px; border-radius: 5px; padding: 1px 4px; margin: -1px -4px; }
  .filetree li:has(> ul) > span:hover, .filetree li:not(:has(> ul)) > span:hover { background: var(--card-2); }
  .tree-i { color: var(--muted); flex: none; }
  .filetree li:has(> ul) > span > .tree-i { color: var(--accent); }
  .tree-chev { width: 14px; height: 14px; color: var(--muted); transition: transform .12s; flex: none; }
  .filetree li.collapsed > span > .tree-chev { transform: rotate(-90deg); }
  .filetree li.collapsed > ul { display: none; }
  .tpill { display: inline-flex; align-items: center; gap: 4px; font-size: 10.5px; line-height: 1.7; padding: 0 7px; border-radius: 999px; background: var(--card-2); color: var(--muted); }
  .tpill::before { content: ""; width: 5px; height: 5px; border-radius: 50%; background: currentColor; }
  .tpill.new { color: var(--add); } .tpill.edit { color: var(--warn); }

  /* ===== Dock — sizes to its buttons; expands upward into one surface ===== */
  .dock { position: fixed; bottom: 16px; left: 50%; transform: translateX(-50%) translateZ(0); z-index: 45;
    width: fit-content; max-width: 94vw; border: 1px solid var(--line); border-radius: 16px; overflow: hidden;
    background: color-mix(in srgb, var(--bg-elev) 97%, transparent); backdrop-filter: blur(12px); box-shadow: var(--shadow); }
  .dock-row { display: flex; align-items: center; justify-content: center; gap: 2px; padding: 6px; }
  .dock-row button { display: inline-flex; align-items: center; gap: 6px; cursor: pointer; font: inherit; font-size: 13px;
    background: transparent; color: var(--fg); border: 0; border-radius: 10px; padding: 8px 10px; }
  .dock-row button:hover { background: var(--card-2); color: var(--accent); }
  .dock-row button.active { background: var(--accent); color: var(--on-accent); }
  .dock-row .sep { width: 1px; height: 22px; background: var(--line); margin: 0 3px; }
  .dock-row .cnt { font-weight: 700; font-size: 12px; }
  /* persistent shortcut hints, always visible on the dock */
  .dock-row kbd { background: var(--card-2); border: 1px solid var(--line-strong); border-radius: 5px;
    padding: 0 5px; font: inherit; font-size: 11px; color: var(--muted); line-height: 1.5; }
  .dock-row button.active kbd { background: rgba(255,255,255,.22); color: var(--on-accent); border-color: transparent; }

  /* Expanding region: fills the dock's width without driving it (width:0 + min-width:100%),
     so only the HEIGHT animates — the button row never reflows (no flicker). */
  .dock-panel { width: 0; min-width: 100%; box-sizing: border-box; max-height: 0; opacity: 0; overflow: hidden;
    transition: max-height .28s cubic-bezier(.2,.7,.2,1), opacity .18s ease; }
  .dock.open .dock-panel { max-height: 56vh; opacity: 1; }
  .dock.open .dock-row { border-top: 1px solid var(--line); }
  .dp-head { padding: 9px 13px; display: flex; align-items: center; gap: 6px; }
  .dp-head h3 { margin: 0; font-size: 13px; flex: 1; color: var(--muted); font-weight: 600; }
  /* Borderless chat: just messages, no boxes */
  .chat { overflow: auto; padding: 4px 14px 12px; display: flex; flex-direction: column; gap: 15px; max-height: 46vh; }
  .chat .msg { font-size: 13px; }
  .chat .msg .lbl { font-size: 11px; font-weight: 700; color: var(--accent); cursor: pointer; display: inline-block; margin-bottom: 3px; }
  .chat .msg.general .lbl { color: var(--muted); }
  .chat .msg .quote { font-size: 11.5px; color: var(--muted); border-left: 2px solid var(--line); padding-left: 8px; margin: 4px 0; }
  .chat .msg .body { color: var(--fg); overflow-wrap: anywhere; }
  .chat .msg .del-c { float: right; color: var(--muted); cursor: pointer; background: none; border: 0; font-size: 15px; line-height: 1; opacity: 0; }
  .chat .msg:hover .del-c { opacity: .7; }
  .chat .empty { color: var(--muted); font-size: 12.5px; }
  /* Compose section (C) — lives inside the dock, expands it; not a floating window */
  .compose { padding: 12px 13px 13px; }
  .compose .target { display: inline-flex; align-items: center; gap: 6px; max-width: 100%; font-size: 12px; color: var(--muted);
    background: var(--card-2); border: 1px solid var(--line); border-radius: 8px; padding: 3px 5px 3px 9px; margin-bottom: 9px; }
  .compose .target span { overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
  .compose .target.pinned { color: var(--accent); border-color: var(--accent-line); }
  .compose .target .tclear { background: none; border: 0; color: inherit; cursor: pointer; font-size: 15px; line-height: 1; padding: 0 3px; }
  .compose textarea { width: 100%; min-height: 64px; resize: vertical; background: var(--card); color: var(--fg);
    border: 1px solid var(--line); border-radius: 9px; padding: 9px; font: inherit; font-size: 13px; }
  .compose .row { display: flex; gap: 8px; align-items: center; justify-content: flex-end; margin-top: 9px; }
  .compose .chint { margin-right: auto; font-size: 11px; color: var(--muted); }

  /* copy confirmation toast */
  .vp-toast { position: fixed; bottom: 88px; left: 50%; transform: translateX(-50%); z-index: 80;
    background: var(--accent); color: var(--on-accent); padding: 8px 15px; border-radius: 11px; font-size: 13px;
    box-shadow: var(--shadow); animation: vp-pop .18s ease; }
  @keyframes vp-pop { from { opacity: 0; transform: translate(-50%, 6px) scale(.96); } to { opacity: 1; transform: translate(-50%, 0) scale(1); } }

  .btn { display: inline-flex; align-items: center; gap: 6px; cursor: pointer; white-space: nowrap; font: inherit; font-size: 13px;
    background: var(--card); color: var(--fg); border: 1px solid var(--line); border-radius: 8px; padding: 6px 11px; }
  .btn:hover { border-color: var(--accent); color: var(--accent); }
  .btn.accent { background: var(--accent); color: var(--on-accent); border-color: var(--accent); }
  .btn.icon-only { padding: 7px; }
  .btn.xs { padding: 4px 8px; font-size: 12px; }
  .btn.xs.icon-only { padding: 5px; }

  /* ===== Annotations — highlight sits OUTSIDE the block (offset gap) ===== */
  [data-anchor-id] { position: relative; border-radius: 8px; }
  /* While composing, blocks become pinnable: highlight on hover, gap via outline-offset */
  body.assigning [data-anchor-id] { cursor: pointer; }
  body.assigning [data-anchor-id]:hover { outline: 1.5px solid var(--accent-line); outline-offset: 5px; }
  body.assigning .has-anno { outline: 1.5px solid var(--accent); outline-offset: 5px; }
  .anno-badge { display: inline-flex; align-items: center; gap: 3px; margin-left: 8px; background: var(--accent-soft);
    color: var(--accent); border-radius: 999px; padding: 0 8px; font-size: 11.5px; font-weight: 600; vertical-align: middle; }
  .anno-badge .icon { width: 12px; height: 12px; }
  .anno-flash { animation: annoflash 1.3s ease; }
  @keyframes annoflash { 0%,100% { background: transparent; } 28% { background: var(--accent-soft); } }
  /* fine-grained text marks (CSS Custom Highlight API) */
  ::highlight(vp-mark) { background-color: color-mix(in srgb, var(--accent) 38%, transparent); color: var(--fg);
    text-decoration: underline; text-decoration-color: var(--accent); }

  /* ===== Keyboard help overlay ===== */
  .kbd-help { position: fixed; inset: 0; z-index: 70; display: grid; place-items: center; background: rgba(0,0,0,.45); }
  .kbd-help[hidden] { display: none; }
  .kbd-card { width: 380px; max-width: calc(100vw - 32px); background: var(--bg-elev); border: 1px solid var(--line); border-radius: 14px; box-shadow: var(--shadow); }
  .kbd-card .ph { padding: 13px 16px; border-bottom: 1px solid var(--line); display: flex; align-items: center; }
  .kbd-card .ph h3 { margin: 0; font-size: 15px; flex: 1; }
  .kbd-card table { font-size: 13.5px; }
  .kbd-card td { border: 0; padding: 8px 16px; }
  .kbd-card kbd { background: var(--card-2); border: 1px solid var(--line-strong); border-bottom-width: 2px; border-radius: 6px; padding: 1px 7px; font: inherit; font-size: 12px; }

  @media (max-width: 860px) {
    .dock-row { flex-wrap: wrap; justify-content: center; }
  }

  /* ===== QOL ===== */
  /* reading-progress bar + scroll-spy TOC */
  .vp-progress { position: fixed; top: 0; left: 0; height: 2px; width: 0; background: var(--accent); z-index: 70; transition: width .1s linear; }
  nav.toc a.active { color: var(--accent); border-left-color: var(--accent); font-weight: 600; }
  /* callout tone variants (default is warn) */
  .callout.ok { border-left-color: var(--add); background: var(--add-bg); }
  .callout.info { border-left-color: var(--accent); background: var(--accent-soft); }
  .callout.risk { border-left-color: var(--del); background: var(--del-bg); }
  /* copy-code button (JS wraps pre in .code-wrap) */
  .code-wrap { position: relative; }
  .code-wrap > .copy-code { position: absolute; top: 8px; right: 8px; opacity: 0; transition: opacity .12s;
    display: inline-flex; align-items: center; gap: 5px; font: inherit; font-size: 12px; cursor: pointer;
    background: var(--card-2); color: var(--muted); border: 1px solid var(--line); border-radius: 7px; padding: 3px 8px; }
  .code-wrap:hover > .copy-code { opacity: 1; }
  .code-wrap > .copy-code:hover { color: var(--accent); border-color: var(--accent); }
  /* images: zoomable, with lightbox */
  main img { max-width: 100%; height: auto; border: 1px solid var(--line); border-radius: 8px; cursor: zoom-in; }
  .vp-lightbox { position: fixed; inset: 0; z-index: 90; display: none; flex-direction: column; align-items: center; justify-content: center;
    background: rgba(0,0,0,.86); padding: 4vh 4vw; }
  .vp-lightbox.open { display: flex; }
  .vp-lightbox img { max-width: 92vw; max-height: 84vh; border-radius: 8px; cursor: zoom-out; }
  .vp-lightbox .cap { color: #eee; font-size: 13px; margin-top: 12px; max-width: 70ch; text-align: center; }
  /* chat: timestamps + per-message actions (edit/delete) */
  .chat .msg .meta { font-size: 10.5px; color: var(--muted); margin-top: 2px; }
  .chat .msg .acts { float: right; display: inline-flex; gap: 8px; opacity: 0; transition: opacity .12s; }
  .chat .msg:hover .acts { opacity: .75; }
  .chat .msg .acts button { background: none; border: 0; color: var(--muted); cursor: pointer; font-size: 13px; line-height: 1; padding: 0; }
  .chat .msg .acts button:hover { color: var(--accent); }
  .chat .msg .edit-ta { width: 100%; min-height: 52px; resize: vertical; background: var(--card); color: var(--fg);
    border: 1px solid var(--accent); border-radius: 7px; padding: 7px; font: inherit; font-size: 13px; margin-top: 4px; }
  .chat .msg .edit-row { display: flex; gap: 7px; justify-content: flex-end; margin-top: 6px; }
  /* earlier / resolved feedback, tucked behind a disclosure so the active list stays clean */
  .chat details.earlier { margin-top: 4px; border-top: 1px solid var(--line); padding-top: 10px; }
  .chat details.earlier > summary { cursor: pointer; font-size: 12px; color: var(--muted); list-style: none; }
  .chat details.earlier > summary::-webkit-details-marker { display: none; }
  .chat details.earlier > summary::before { content: "▸ "; }
  .chat details.earlier[open] > summary::before { content: "▾ "; }
  .chat details.earlier > div { margin-top: 10px; display: flex; flex-direction: column; gap: 14px; }
  .chat .msg.resolved { opacity: .55; }
  /* selection popover: "Comment this section" appears on any text selection / node click */
  .sel-pop { position: fixed; z-index: 65; display: none; }
  .sel-pop.show { display: block; }
  .sel-pop button { display: inline-flex; align-items: center; gap: 6px; font: inherit; font-size: 12.5px; cursor: pointer;
    background: var(--accent); color: var(--on-accent); border: 0; border-radius: 9px; padding: 6px 11px; box-shadow: var(--shadow); white-space: nowrap; }
  .sel-pop button:hover { filter: brightness(1.06); }
  /* honor reduced-motion */
  @media (prefers-reduced-motion: reduce) {
    html { scroll-behavior: auto; }
    * { animation-duration: .001ms !important; transition-duration: .001ms !important; }
  }
  /* print / save-as-PDF: clean static document */
  @media print {
    :root { --body: 100%; }
    .dock, .vp-progress, .vp-lightbox, .kbd-help, .vp-toast, nav.toc, .copy-code { display: none !important; }
    .wrap { max-width: 100%; padding: 0; }
    .anno-badge, .has-anno { box-shadow: none !important; outline: none !important; }
    details:not([open]) > *:not(summary) { display: revert; }  /* expand all */
    details { border: 0; padding: 0; }
    a[href^="http"]::after { content: " (" attr(href) ")"; font-size: .85em; color: #555; }
    section, .card, .callout, .diagram, pre { break-inside: avoid; }
    body { background: #fff; color: #111; }
  }
</style>
</head>
<body>

<div class="vp-progress" id="vpProgress"></div>

<!-- ===== Tabler UI icon sprite (MIT, stroke) ===== -->
<svg width="0" height="0" style="position:absolute" aria-hidden="true">
  <symbol id="i-moon" viewBox="0 0 24 24"><path d="M12 3c.132 0 .263 0 .393 0a7.5 7.5 0 0 0 7.92 12.446a9 9 0 1 1 -8.313 -12.454z"/></symbol>
  <symbol id="i-sun" viewBox="0 0 24 24"><path d="M12 12m-4 0a4 4 0 1 0 8 0a4 4 0 1 0 -8 0"/><path d="M3 12h1m8 -9v1m8 8h1m-9 8v1m-6.4 -15.4l.7 .7m12.1 -.7l-.7 .7m0 11.4l.7 .7m-12.1 -.7l-.7 .7"/></symbol>
  <symbol id="i-message" viewBox="0 0 24 24"><path d="M12 20l-3 -3h-2a3 3 0 0 1 -3 -3v-6a3 3 0 0 1 3 -3h10a3 3 0 0 1 3 3v6a3 3 0 0 1 -3 3h-2l-3 3"/><path d="M8 9l8 0"/><path d="M8 13l6 0"/></symbol>
  <symbol id="i-list" viewBox="0 0 24 24"><path d="M9 6l11 0"/><path d="M9 12l11 0"/><path d="M9 18l11 0"/><path d="M5 6l0 .01"/><path d="M5 12l0 .01"/><path d="M5 18l0 .01"/></symbol>
  <symbol id="i-keyboard" viewBox="0 0 24 24"><path d="M2 6m0 2a2 2 0 0 1 2 -2h16a2 2 0 0 1 2 2v8a2 2 0 0 1 -2 2h-16a2 2 0 0 1 -2 -2z"/><path d="M6 10l0 .01"/><path d="M10 10l0 .01"/><path d="M14 10l0 .01"/><path d="M18 10l0 .01"/><path d="M6 14l0 .01"/><path d="M18 14l0 .01"/><path d="M10 14l4 .01"/></symbol>
  <symbol id="i-copy" viewBox="0 0 24 24"><path d="M8 8m0 2a2 2 0 0 1 2 -2h8a2 2 0 0 1 2 2v8a2 2 0 0 1 -2 2h-8a2 2 0 0 1 -2 -2z"/><path d="M16 8v-2a2 2 0 0 0 -2 -2h-8a2 2 0 0 0 -2 2v8a2 2 0 0 0 2 2h2"/></symbol>
  <symbol id="i-download" viewBox="0 0 24 24"><path d="M4 17v2a2 2 0 0 0 2 2h12a2 2 0 0 0 2 -2v-2"/><path d="M7 11l5 5l5 -5"/><path d="M12 4l0 12"/></symbol>
  <symbol id="i-chev" viewBox="0 0 24 24"><path d="M9 6l6 6l-6 6"/></symbol>
  <symbol id="i-x" viewBox="0 0 24 24"><path d="M18 6l-12 12"/><path d="M6 6l12 12"/></symbol>
  <symbol id="i-folder" viewBox="0 0 24 24"><path d="M5 4h4l3 3h7a2 2 0 0 1 2 2v8a2 2 0 0 1 -2 2h-14a2 2 0 0 1 -2 -2v-11a2 2 0 0 1 2 -2"/></symbol>
  <symbol id="i-file" viewBox="0 0 24 24"><path d="M14 3v4a1 1 0 0 0 1 1h4"/><path d="M17 21h-10a2 2 0 0 1 -2 -2v-14a2 2 0 0 1 2 -2h7l5 5v11a2 2 0 0 1 -2 2z"/></symbol>
  <symbol id="i-branch" viewBox="0 0 24 24"><path d="M7 18m-2 0a2 2 0 1 0 4 0a2 2 0 1 0 -4 0"/><path d="M7 6m-2 0a2 2 0 1 0 4 0a2 2 0 1 0 -4 0"/><path d="M17 6m-2 0a2 2 0 1 0 4 0a2 2 0 1 0 -4 0"/><path d="M7 8v8"/><path d="M9 18h6a2 2 0 0 0 2 -2v-5"/><path d="M14 14l3 -3l3 3"/></symbol>
  <symbol id="i-calendar" viewBox="0 0 24 24"><path d="M4 7a2 2 0 0 1 2 -2h12a2 2 0 0 1 2 2v12a2 2 0 0 1 -2 2h-12a2 2 0 0 1 -2 -2z"/><path d="M16 3v4"/><path d="M8 3v4"/><path d="M4 11h16"/></symbol>
  <symbol id="i-history" viewBox="0 0 24 24"><path d="M12 8l0 4l2 2"/><path d="M3.05 11a9 9 0 1 1 .5 4m-.5 5v-5h5"/></symbol>
  <symbol id="i-chevron-down" viewBox="0 0 24 24"><path d="M6 9l6 6l6 -6"/></symbol>
</svg>

<!-- ===== Brand logo sprite (Simple Icons, MIT, fill). Add more by copying exact SI paths. ===== -->
<svg width="0" height="0" style="position:absolute" aria-hidden="true">
  <symbol id="b-whatsapp" viewBox="0 0 24 24"><path d="M17.472 14.382c-.297-.149-1.758-.867-2.03-.967-.273-.099-.471-.148-.67.15-.197.297-.767.966-.94 1.164-.173.199-.347.223-.644.075-.297-.15-1.255-.463-2.39-1.475-.883-.788-1.48-1.761-1.653-2.059-.173-.297-.018-.458.13-.606.134-.133.298-.347.446-.52.149-.174.198-.298.298-.497.099-.198.05-.371-.025-.52-.075-.149-.669-1.612-.916-2.207-.242-.579-.487-.5-.669-.51-.173-.008-.371-.01-.57-.01-.198 0-.52.074-.792.372-.272.297-1.04 1.016-1.04 2.479 0 1.462 1.065 2.875 1.213 3.074.149.198 2.096 3.2 5.077 4.487.71.306 1.263.489 1.694.625.712.227 1.36.195 1.871.118.571-.085 1.758-.719 2.006-1.413.248-.694.248-1.289.173-1.413-.074-.124-.272-.198-.57-.347m-5.421 7.403h-.004a9.87 9.87 0 01-5.031-1.378l-.361-.214-3.741.982.998-3.648-.235-.374a9.86 9.86 0 01-1.51-5.26c.001-5.45 4.436-9.884 9.888-9.884 2.64 0 5.122 1.03 6.988 2.898a9.825 9.825 0 012.893 6.994c-.003 5.45-4.437 9.885-9.885 9.885M20.52 3.449C18.24 1.245 15.24 0 12.045 0 5.463 0 .104 5.359.101 11.945c0 2.096.547 4.142 1.588 5.945L0 24l6.335-1.652a11.882 11.882 0 005.71 1.454h.005c6.585 0 11.946-5.359 11.949-11.945a11.821 11.821 0 00-3.479-8.408"/></symbol>
  <symbol id="b-telegram" viewBox="0 0 24 24"><path d="M11.944 0A12 12 0 0 0 0 12a12 12 0 0 0 12 12 12 12 0 0 0 12-12A12 12 0 0 0 12 0a12 12 0 0 0-.056 0zm4.962 7.224c.1-.002.321.023.465.14a.506.506 0 0 1 .171.325c.016.093.036.306.02.472-.18 1.898-.962 6.502-1.36 8.627-.168.9-.499 1.201-.82 1.23-.696.065-1.225-.46-1.9-.902-1.056-.693-1.653-1.124-2.678-1.8-1.185-.78-.417-1.21.258-1.91.177-.184 3.247-2.977 3.307-3.23.007-.032.014-.15-.056-.212s-.174-.041-.249-.024c-.106.024-1.793 1.14-5.061 3.345-.48.33-.913.49-1.302.48-.428-.008-1.252-.241-1.865-.44-.752-.245-1.349-.374-1.297-.789.027-.216.325-.437.893-.663 3.498-1.524 5.83-2.529 6.998-3.014 3.332-1.386 4.025-1.627 4.476-1.635z"/></symbol>
  <symbol id="b-signal" viewBox="0 0 24 24"><path d="M12 0a12 12 0 0 0-12 12 12 12 0 0 0 1.78 6.29L.05 23.2a.62.62 0 0 0 .76.76l4.9-1.73A12 12 0 0 0 12 24a12 12 0 0 0 12-12A12 12 0 0 0 12 0Zm0 2.4A9.6 9.6 0 0 1 21.6 12 9.6 9.6 0 0 1 12 21.6a9.6 9.6 0 0 1-4.9-1.34.6.6 0 0 0-.5-.05l-3.2 1.13 1.13-3.2a.6.6 0 0 0-.05-.5A9.6 9.6 0 0 1 2.4 12 9.6 9.6 0 0 1 12 2.4Z"/></symbol>
  <symbol id="b-discord" viewBox="0 0 24 24"><path d="M20.317 4.3698a19.7913 19.7913 0 00-4.8851-1.5152.0741.0741 0 00-.0785.0371c-.211.3753-.4447.8648-.6083 1.2495-1.8447-.2762-3.68-.2762-5.4868 0-.1636-.3933-.4058-.8742-.6177-1.2495a.077.077 0 00-.0785-.037 19.7363 19.7363 0 00-4.8852 1.515.0699.0699 0 00-.0321.0277C.5334 9.0458-.319 13.5799.0992 18.0578a.0824.0824 0 00.0312.0561c2.0528 1.5076 4.0413 2.4228 5.9929 3.0294a.0777.0777 0 00.0842-.0276c.4616-.6304.8731-1.2952 1.226-1.9942a.076.076 0 00-.0416-.1057c-.6528-.2476-1.2743-.5495-1.8722-.8923a.077.077 0 01-.0076-.1277c.1258-.0943.2517-.1923.3718-.2914a.0743.0743 0 01.0776-.0105c3.9278 1.7933 8.18 1.7933 12.0614 0a.0739.0739 0 01.0785.0095c.1202.099.246.1981.3728.2924a.077.077 0 01-.0066.1276 12.2986 12.2986 0 01-1.873.8914.0766.0766 0 00-.0407.1067c.3604.698.7719 1.3628 1.225 1.9932a.076.076 0 00.0842.0286c1.961-.6067 3.9495-1.5219 6.0023-3.0294a.077.077 0 00.0313-.0552c.5004-5.177-.8382-9.6739-3.5485-13.6604a.061.061 0 00-.0312-.0286zM8.02 15.3312c-1.1825 0-2.1569-1.0857-2.1569-2.419 0-1.3332.9555-2.4189 2.157-2.4189 1.2108 0 2.1757 1.0952 2.1568 2.419 0 1.3332-.9555 2.4189-2.1569 2.4189zm7.9748 0c-1.1825 0-2.1569-1.0857-2.1569-2.419 0-1.3332.9554-2.4189 2.1569-2.4189 1.2108 0 2.1757 1.0952 2.1568 2.419 0 1.3332-.946 2.4189-2.1568 2.4189Z"/></symbol>
  <symbol id="b-slack" viewBox="0 0 24 24"><path d="M5.042 15.165a2.528 2.528 0 0 1-2.52 2.523A2.528 2.528 0 0 1 0 15.165a2.527 2.527 0 0 1 2.522-2.52h2.52v2.52zM6.313 15.165a2.527 2.527 0 0 1 2.521-2.52 2.527 2.527 0 0 1 2.521 2.52v6.313A2.528 2.528 0 0 1 8.834 24a2.528 2.528 0 0 1-2.521-2.522v-6.313zM8.834 5.042a2.528 2.528 0 0 1-2.521-2.52A2.528 2.528 0 0 1 8.834 0a2.528 2.528 0 0 1 2.521 2.522v2.52H8.834zM8.834 6.313a2.528 2.528 0 0 1 2.521 2.521 2.528 2.528 0 0 1-2.521 2.521H2.522A2.528 2.528 0 0 1 0 8.834a2.528 2.528 0 0 1 2.522-2.521h6.312zM18.956 8.834a2.528 2.528 0 0 1 2.522-2.521A2.528 2.528 0 0 1 24 8.834a2.528 2.528 0 0 1-2.522 2.521h-2.522V8.834zM17.688 8.834a2.528 2.528 0 0 1-2.523 2.521 2.527 2.527 0 0 1-2.52-2.521V2.522A2.527 2.527 0 0 1 15.165 0a2.528 2.528 0 0 1 2.523 2.522v6.312zM15.165 18.956a2.528 2.528 0 0 1 2.523 2.522A2.528 2.528 0 0 1 15.165 24a2.527 2.527 0 0 1-2.52-2.522v-2.522h2.52zM15.165 17.688a2.527 2.527 0 0 1-2.52-2.523 2.526 2.526 0 0 1 2.52-2.52h6.313A2.527 2.527 0 0 1 24 15.165a2.528 2.528 0 0 1-2.522 2.523h-6.313z"/></symbol>
  <symbol id="b-instagram" viewBox="0 0 24 24"><path d="M12 0C8.74 0 8.333.015 7.053.072 5.775.132 4.905.333 4.14.63c-.789.306-1.459.717-2.126 1.384S.935 3.35.63 4.14C.333 4.905.131 5.775.072 7.053.012 8.333 0 8.74 0 12s.015 3.667.072 4.947c.06 1.277.261 2.148.558 2.913.306.788.717 1.459 1.384 2.126.667.666 1.336 1.079 2.126 1.384.766.296 1.636.499 2.913.558C8.333 23.988 8.74 24 12 24s3.667-.015 4.947-.072c1.277-.06 2.148-.262 2.913-.558.788-.306 1.459-.718 2.126-1.384.666-.667 1.079-1.335 1.384-2.126.296-.765.499-1.636.558-2.913.06-1.28.072-1.687.072-4.947s-.015-3.667-.072-4.947c-.06-1.277-.262-2.149-.558-2.913-.306-.789-.718-1.459-1.384-2.126C21.319 1.347 20.651.935 19.86.63c-.765-.297-1.636-.499-2.913-.558C15.667.012 15.26 0 12 0zm0 2.16c3.203 0 3.585.016 4.85.071 1.17.055 1.805.249 2.227.415.562.217.96.477 1.382.896.419.42.679.819.896 1.381.164.422.36 1.057.413 2.227.057 1.266.07 1.646.07 4.85s-.015 3.585-.074 4.85c-.061 1.17-.256 1.805-.421 2.227-.224.562-.479.96-.899 1.382-.419.419-.824.679-1.38.896-.42.164-1.065.36-2.235.413-1.274.057-1.649.07-4.859.07-3.211 0-3.586-.015-4.859-.074-1.171-.061-1.816-.256-2.236-.421-.569-.224-.96-.479-1.379-.899-.421-.419-.69-.824-.9-1.38-.165-.42-.359-1.065-.42-2.235-.045-1.26-.061-1.649-.061-4.844 0-3.196.016-3.586.061-4.861.061-1.17.255-1.814.42-2.234.21-.57.479-.96.9-1.381.419-.419.81-.689 1.379-.898.42-.166 1.051-.361 2.221-.421 1.275-.045 1.65-.06 4.859-.06l.045.03zm0 3.678c-3.405 0-6.162 2.76-6.162 6.162 0 3.405 2.76 6.162 6.162 6.162 3.405 0 6.162-2.76 6.162-6.162 0-3.405-2.76-6.162-6.162-6.162zM12 16c-2.21 0-4-1.79-4-4s1.79-4 4-4 4 1.79 4 4-1.79 4-4 4zm7.846-10.405c0 .795-.646 1.44-1.44 1.44-.795 0-1.44-.646-1.44-1.44 0-.794.646-1.439 1.44-1.439.793-.001 1.44.645 1.44 1.439z"/></symbol>
  <symbol id="b-messenger" viewBox="0 0 24 24"><path d="M.001 11.639C.001 4.949 5.241 0 12 0s11.999 4.95 11.999 11.639c0 6.689-5.24 11.638-11.999 11.638-1.214 0-2.378-.159-3.473-.461a.961.961 0 0 0-.641.046l-2.381 1.05a.96.96 0 0 1-1.347-.849l-.065-2.134a.957.957 0 0 0-.322-.68C1.879 17.954.001 14.99.001 11.639zm8.32-2.19l-3.525 5.593c-.337.535.318 1.142.821.761l3.788-2.875a.722.722 0 0 1 .867-.002l2.803 2.104a1.8 1.8 0 0 0 2.601-.48l3.525-5.593c.338-.535-.317-1.142-.82-.761l-3.789 2.875a.722.722 0 0 1-.867.002l-2.803-2.104a1.8 1.8 0 0 0-2.601.48z"/></symbol>
  <symbol id="b-google" viewBox="0 0 24 24"><path d="M12.48 10.92v3.28h7.84c-.24 1.84-.853 3.187-1.787 4.133-1.147 1.147-2.933 2.4-6.053 2.4-4.827 0-8.6-3.893-8.6-8.72s3.773-8.72 8.6-8.72c2.6 0 4.507 1.027 5.907 2.347l2.307-2.307C18.747 1.44 16.133 0 12.48 0 5.867 0 .307 5.387.307 12s5.56 12 12.173 12c3.573 0 6.267-1.173 8.373-3.36 2.16-2.16 2.84-5.213 2.84-7.667 0-.76-.053-1.467-.173-2.053H12.48z"/></symbol>
  <symbol id="b-github" viewBox="0 0 24 24"><path d="M12 .297c-6.63 0-12 5.373-12 12 0 5.303 3.438 9.8 8.205 11.385.6.113.82-.258.82-.577 0-.285-.01-1.04-.015-2.04-3.338.724-4.042-1.61-4.042-1.61C4.422 18.07 3.633 17.7 3.633 17.7c-1.087-.744.084-.729.084-.729 1.205.084 1.838 1.236 1.838 1.236 1.07 1.835 2.809 1.305 3.495.998.108-.776.417-1.305.76-1.605-2.665-.3-5.466-1.332-5.466-5.93 0-1.31.465-2.38 1.235-3.22-.135-.303-.54-1.523.105-3.176 0 0 1.005-.322 3.3 1.23.96-.267 1.98-.399 3-.405 1.02.006 2.04.138 3 .405 2.28-1.552 3.285-1.23 3.285-1.23.645 1.653.24 2.873.12 3.176.765.84 1.23 1.91 1.23 3.22 0 4.61-2.805 5.625-5.475 5.92.42.36.81 1.096.81 2.22 0 1.606-.015 2.896-.015 3.286 0 .315.21.69.825.57C20.565 22.092 24 17.592 24 12.297c0-6.627-5.373-12-12-12"/></symbol>
  <symbol id="b-linkedin" viewBox="0 0 24 24"><path d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433a2.062 2.062 0 01-2.063-2.065 2.064 2.064 0 112.063 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z"/></symbol>
  <symbol id="b-x" viewBox="0 0 24 24"><path d="M18.901 1.153h3.68l-8.04 9.19L24 22.846h-7.406l-5.8-7.584-6.638 7.584H.474l8.6-9.83L0 1.154h7.594l5.243 6.932ZM17.61 20.644h2.039L6.486 3.24H4.298Z"/></symbol>
</svg>

<div class="wrap">
  <nav class="toc">
    <a href="#overview">Overview</a>
    <a href="#arch">Architecture</a>
    <a href="#flow">Flow</a>
    <a href="#steps">Steps</a>
    <a href="#files">File map</a>
    <a href="#risks">Risks</a>
    <a href="#questions">Open questions</a>
  </nav>

  <main>
    <!-- Header: metadata row (repo · branch · date · version) on top, title, full-width outcome.
         Agent fills repo URL, branch, date, version number, and the version-pop rows. -->
    <div class="plan-head">
      <div class="meta">
        <span class="m-item"><svg class="icon"><use href="#b-github"/></svg> <a href="https://github.com/owner/repo">owner/repo</a></span>
        <span class="m-item"><svg class="icon"><use href="#i-branch"/></svg> main</span>
        <span class="m-item"><svg class="icon"><use href="#i-calendar"/></svg> 2026-06-21</span>
        <span class="version">
          <button class="vbtn" id="vBtn" type="button"><svg class="icon"><use href="#i-history"/></svg> v3 <svg class="icon"><use href="#i-chevron-down"/></svg></button>
          <div class="version-pop" id="versionPop" hidden>
            <div class="vh">Plan versions</div>
            <a class="vrow current" href="#"><span class="vn">v3</span> current <span class="vd">2026-06-21</span></a>
            <a class="vrow" href="VERSION_2_URL"><span class="vn">v2</span> <span class="vd">2026-06-20</span></a>
            <a class="vrow" href="VERSION_1_URL"><span class="vn">v1</span> <span class="vd">2026-06-19</span></a>
          </div>
        </span>
      </div>
      <h1>PLAN TITLE</h1>
      <p class="outcome">One-line outcome: what is true when this is done — full width.</p>
    </div>

    <section id="overview" data-anchor-id="overview" data-anchor-label="Overview">
      <h2>Overview</h2>
      <p>Objective, scope, non-goals. Lead with one concrete snapshot if abstract.</p>
    </section>

    <!-- Brand-logo diagram kit: neutral nodes, accent only on "our code". -->
    <section id="arch" data-anchor-id="arch" data-anchor-label="Architecture">
      <h2>Architecture</h2>
      <div class="diagram">
        <div class="dlayer">
          <div class="dnode"><div class="t"><svg class="brand" style="color:#25D366"><use href="#b-whatsapp"/></svg> WhatsApp</div><div class="s">device link</div></div>
          <div class="dnode"><div class="t"><svg class="brand" style="color:#26A5E4"><use href="#b-telegram"/></svg> Telegram</div><div class="s">MTProto</div></div>
          <div class="dnode"><div class="t"><svg class="brand" style="color:#3A76F0"><use href="#b-signal"/></svg> Signal</div><div class="s">linked device</div></div>
          <div class="dnode"><div class="t"><svg class="brand" style="color:#5865F2"><use href="#b-discord"/></svg> Discord</div><div class="s">token / OAuth</div></div>
        </div>
        <div class="dflow">↓</div>
        <div class="dbar"><div class="t">Bridge layer</div><div class="s">one process per network</div></div>
        <div class="dbar ours"><div class="t">Orchestration API (our code)</div><div class="s">signup · provisioning</div></div>
      </div>
    </section>

    <!-- Mermaid for flow / sequence / state / ER diagrams. -->
    <section id="flow" data-anchor-id="flow" data-anchor-label="Onboarding flow">
      <h2>Flow</h2>
      <pre class="mermaid">
flowchart LR
  A[User] --> B{Has account?}
  B -- no --> C[Signup]
  B -- yes --> D[Connect network]
  C --> D
  D --> E[Bridge login]
  E --> F[(Unified inbox)]
      </pre>
    </section>

    <section id="steps" data-anchor-id="steps" data-anchor-label="Steps">
      <h2>Steps</h2>
      <div class="step" data-anchor-id="step-1" data-anchor-label="Step 1">
        <div class="num">1</div>
        <div><strong>Step title.</strong> What it reuses, then what it adds.
          <div class="files">Touches <code>actions/foo.ts</code></div></div>
      </div>
      <div class="filelabel"><svg class="icon"><use href="#i-copy"/></svg> actions/foo.ts</div>
      <pre class="diff"><code><span class="del">- old line</span><span class="add">+ new line</span>
  context line</code></pre>
    </section>

    <!-- Rendered file tree: wrap each label in <span>; mark folders by nesting <ul>. -->
    <section id="files" data-anchor-id="files" data-anchor-label="File map">
      <h2>File map</h2>
      <ul class="filetree">
        <li><span>app</span>
          <ul>
            <li><span>actions</span><ul><li data-new><span>foo.ts</span></li></ul></li>
            <li data-edit><span>components/Bar.tsx</span></li>
          </ul>
        </li>
        <li><span>install.sh</span></li>
      </ul>
    </section>

    <!-- Callout tones: default (warn), .ok, .info, .risk. Images get a free lightbox. -->
    <section id="risks" data-anchor-id="risks" data-anchor-label="Risks">
      <h2>Risks</h2>
      <div class="callout risk"><strong>Account bans.</strong> Unofficial APIs can flag accounts → mitigation.</div>
      <div class="callout ok" style="margin-top:10px"><strong>Shipped.</strong> v1 is live behind a flag.</div>
    </section>

    <!-- Open questions as answerable chips: one option per real choice, mark the
         recommended one [data-rec] (it's pre-selected). Answers export as Decisions. -->
    <section id="questions" data-anchor-id="questions" data-anchor-label="Open questions">
      <h2>Open questions</h2>
      <div class="qa" data-qid="wire-format" data-q="Which wire format for the public API?">
        <div class="q-text">Which wire format should the public API use?</div>
        <div class="q-opts">
          <button class="qopt" data-rec type="button">JSON <span class="rec-tag">recommended</span></button>
          <button class="qopt" type="button">Protobuf</button>
          <button class="qopt" type="button">MessagePack</button>
        </div>
        <div class="q-other"><input type="text" placeholder="Other / notes…"></div>
      </div>
    </section>
  </main>
</div>

<!-- ===== Dock — single surface that expands upward ===== -->
<div class="dock" id="dock" aria-label="Plan controls">
  <!-- Expanding region inside the dock: the compose box (C) OR the feedback chat (F) -->
  <div class="dock-panel">
    <div id="dpCompose" hidden>
      <div class="compose">
        <div class="target" id="cTarget"><svg class="icon"><use href="#i-file"/></svg> <span>Whole plan</span>
          <button class="tclear" type="button" title="Unpin" hidden>&times;</button></div>
        <textarea id="genText" placeholder="Type a comment… then click a section or select text to pin it (optional)"></textarea>
        <div class="row"><span class="chint">⌘/Ctrl+↵ to save · click/select to pin</span>
          <button class="btn xs" id="cCancel" type="button">Cancel</button>
          <button class="btn xs accent" id="cSave" type="button">Save</button></div>
      </div>
    </div>
    <div id="dpFeedback" hidden>
      <div class="dp-head"><h3>Feedback <span class="cnt" id="annoCount2"></span></h3>
        <button class="btn xs icon-only" id="dlFb" title="Download"><svg class="icon"><use href="#i-download"/></svg></button>
        <button class="btn xs icon-only" id="dpClose" title="Close"><svg class="icon"><use href="#i-x"/></svg></button>
      </div>
      <div class="chat" id="annoList"></div>
    </div>
  </div>

  <div class="dock-row">
    <button id="cmtMode" title="Comment — type, then pin to a place or save for the whole plan"><svg class="icon"><use href="#i-message"/></svg> Comment <kbd>c</kbd></button>
    <button id="fbPanel" title="Feedback"><svg class="icon"><use href="#i-list"/></svg> <span class="cnt" id="annoCount"></span> <kbd>f</kbd></button>
    <button id="copyDock" title="Copy feedback (⌘/Ctrl+C)"><svg class="icon"><use href="#i-copy"/></svg></button>
    <span class="sep"></span>
    <button id="themeBtn" title="Theme"><svg class="icon"><use href="#i-moon"/></svg> <kbd>t</kbd></button>
    <button id="kbdBtn" title="All shortcuts"><svg class="icon"><use href="#i-keyboard"/></svg> <kbd>?</kbd></button>
  </div>
</div>

<!-- ===== Selection popover ("Comment this section") ===== -->
<div class="sel-pop" id="selPop"><button type="button"><svg class="icon"><use href="#i-message"/></svg> Comment this section</button></div>

<!-- ===== Image lightbox ===== -->
<div class="vp-lightbox" id="vpLightbox"><img alt=""><div class="cap"></div></div>

<!-- ===== Keyboard help ===== -->
<div class="kbd-help" id="kbdHelp" hidden>
  <div class="kbd-card">
    <div class="ph"><h3>Keyboard shortcuts</h3><button class="btn icon-only" id="kbdClose"><svg class="icon"><use href="#i-x"/></svg></button></div>
    <table>
      <tr><td><kbd>c</kbd></td><td>New comment — type, then pin or save</td></tr>
      <tr><td colspan="2" style="color:var(--muted);padding-top:0;font-size:12px">while composing: click a section or select text to pin it</td></tr>
      <tr><td><kbd>f</kbd></td><td>Toggle feedback chat</td></tr>
      <tr><td><kbd>t</kbd></td><td>Toggle theme</td></tr>
      <tr><td><kbd>⌘/Ctrl</kbd> <kbd>C</kbd></td><td>Copy all feedback</td></tr>
      <tr><td><kbd>?</kbd></td><td>This help</td></tr>
      <tr><td><kbd>Esc</kbd></td><td>Close composer / dock / overlay</td></tr>
      <tr><td><kbd>⌘/Ctrl</kbd> <kbd>↵</kbd></td><td>Save the open comment</td></tr>
    </table>
  </div>
</div>

<!-- ===== Mermaid (theme-aware, per-diagram render with graceful fallback). Pinned CDN. ===== -->
<script type="module">
  const blocks = [...document.querySelectorAll('.mermaid')];
  const src = blocks.map(e => e.textContent);
  // On any failure (bad syntax or no network) show the source cleanly — never Mermaid's error graphic.
  function fallback(el, code) {
    el.classList.add('mermaid-fallback', 'done'); el.innerHTML = '';
    const note = document.createElement('div'); note.className = 'mermaid-note'; note.textContent = 'Diagram preview unavailable — showing source';
    const pre = document.createElement('pre'); pre.textContent = (code || '').trim();
    el.appendChild(note); el.appendChild(pre);
  }
  try {
    const mermaid = (await import('https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs')).default;
    let seq = 0;
    async function draw() {
      const dark = document.documentElement.getAttribute('data-theme') !== 'light';
      mermaid.initialize({
        startOnLoad: false, securityLevel: 'loose', suppressErrorRendering: true,
        theme: dark ? 'dark' : 'default',
        themeVariables: {
          primaryColor: dark ? '#28251f' : '#ffffff', primaryTextColor: dark ? '#ece8df' : '#2c2a24',
          primaryBorderColor: '#c15f3c', lineColor: dark ? '#9c9588' : '#6f6a5d',
          fontFamily: '-apple-system,BlinkMacSystemFont,Segoe UI,Roboto,sans-serif'
        }
      });
      for (let i = 0; i < blocks.length; i++) {
        const el = blocks[i];
        try {
          const { svg } = await mermaid.render('vpm-' + (seq++), src[i]);
          el.classList.remove('mermaid-fallback'); el.classList.add('done'); el.innerHTML = svg;
        } catch (err) { fallback(el, src[i]); }
      }
    }
    await draw();
    document.addEventListener('vp-theme', draw);
  } catch (e) {
    blocks.forEach((el, i) => fallback(el, src[i]));
  }
</script>

<!-- ===== App: theme, comment mode, annotations, shortcuts ===== -->
<script>
(function () {
  var PLAN = document.title.replace(/ — Visual Plan$/, '');
  var PLAN_VERSION = parseInt(((document.getElementById('vBtn') || {}).textContent || '').replace(/[^\d]/g, ''), 10) || 1;
  var KEY = 'vp-comments::' + PLAN;
  var comments = [];
  try { comments = JSON.parse(localStorage.getItem(KEY)) || []; } catch (e) {}
  var save = function () { try { localStorage.setItem(KEY, JSON.stringify(comments)); } catch (e) {} };
  var AKEY = 'vp-answers::' + PLAN;
  var answers = {};
  try { answers = JSON.parse(localStorage.getItem(AKEY)) || {}; } catch (e) {}
  var saveAnswers = function () { try { localStorage.setItem(AKEY, JSON.stringify(answers)); } catch (e) {} };
  var sel = function (id) { return document.querySelector('[data-anchor-id="' + (window.CSS && CSS.escape ? CSS.escape(id) : id) + '"]'); };
  var esc = function (s) { return (s || '').replace(/[&<>]/g, function (m) { return { '&': '&amp;', '<': '&lt;', '>': '&gt;' }[m]; }); };

  /* ---- Open questions: answer chips, recommended pre-selected, persisted, exported ---- */
  document.querySelectorAll('.qa').forEach(function (qa) {
    var qid = qa.getAttribute('data-qid') || (qa.querySelector('.q-text') || {}).textContent || '';
    var qtext = qa.getAttribute('data-q') || (qa.querySelector('.q-text') || {}).textContent || qid;
    var opts = [].slice.call(qa.querySelectorAll('.qopt'));
    var other = qa.querySelector('.q-other input');
    opts.forEach(function (o) { if (!o.getAttribute('data-val')) o.setAttribute('data-val', (o.childNodes[0] && o.childNodes[0].textContent || o.textContent).trim()); });
    function pick(val, isOther) {
      opts.forEach(function (o) { o.classList.toggle('sel', !isOther && o.getAttribute('data-val') === val); });
      if (other && !isOther) other.value = '';
      answers[qid] = { question: qtext, answer: val };
      saveAnswers();
    }
    opts.forEach(function (o) { o.onclick = function () { pick(o.getAttribute('data-val'), false); }; });
    if (other) other.oninput = function () {
      var v = other.value.trim();
      if (v) { opts.forEach(function (o) { o.classList.remove('sel'); }); answers[qid] = { question: qtext, answer: v }; saveAnswers(); }
    };
    var stored = answers[qid];
    if (stored && stored.answer) {
      var match = opts.filter(function (o) { return o.getAttribute('data-val') === stored.answer; })[0];
      if (match) pick(stored.answer, false);
      else if (other) { other.value = stored.answer; }
    } else {
      var rec = qa.querySelector('.qopt[data-rec]');
      if (rec) pick(rec.getAttribute('data-val'), false);
    }
  });

  /* ---- File tree: icons, pills, collapsible folders, click-to-copy path ---- */
  document.querySelectorAll('.filetree li').forEach(function (li) {
    var span = li.querySelector(':scope > span'); if (!span) return;
    var folder = !!li.querySelector(':scope > ul');
    if (folder) span.insertAdjacentHTML('afterbegin', '<svg class="icon tree-chev"><use href="#i-chevron-down"/></svg>');
    span.insertAdjacentHTML('afterbegin', '<svg class="icon tree-i"><use href="#' + (folder ? 'i-folder' : 'i-file') + '"/></svg>');
    if (li.hasAttribute('data-new')) span.insertAdjacentHTML('beforeend', ' <span class="tpill new">new</span>');
    if (li.hasAttribute('data-edit')) span.insertAdjacentHTML('beforeend', ' <span class="tpill edit">edit</span>');
    if (folder) {
      span.onclick = function () { li.classList.toggle('collapsed'); };
    } else {
      // build a path from ancestor folder labels and copy it on click
      var parts = [], node = li;
      while (node && node.tagName === 'LI') {
        var ps = node.querySelector(':scope > span');
        if (ps) { var t = ps.cloneNode(true); t.querySelectorAll('.tpill,.icon').forEach(function (n) { n.remove(); }); parts.unshift(t.textContent.trim()); }
        node = node.parentElement ? node.parentElement.closest('li') : null;
      }
      var path = parts.join('/');
      span.title = 'Click to copy path: ' + path;
      span.style.cursor = 'copy';
      span.onclick = function () { if (navigator.clipboard) navigator.clipboard.writeText(path).then(function () { toast('Copied ' + path); }); };
    }
  });

  /* ---- Auto-balance card grids so the last row is never an orphan (4 -> 2x2) ---- */
  document.querySelectorAll('.grid').forEach(function (g) {
    var n = g.children.length, cols;
    if (n <= 1) cols = 1; else if (n <= 3) cols = n; else if (n === 4) cols = 2;
    else { cols = Math.ceil(Math.sqrt(n)); if (n % cols !== 0 && n % 3 === 0) cols = 3; if (cols > 4) cols = 4; }
    g.style.setProperty('--cols', cols);
  });

  /* ---- Version timeline popover ---- */
  var vBtn = document.getElementById('vBtn'), vPop = document.getElementById('versionPop');
  if (vBtn && vPop) {
    vBtn.onclick = function (e) { e.stopPropagation(); vPop.hidden = !vPop.hidden; };
    document.addEventListener('click', function (e) { if (!e.target.closest('.version')) vPop.hidden = true; });
  }

  /* ---- Reading progress + scroll-spy TOC ---- */
  var progress = document.getElementById('vpProgress');
  var spySections = [].slice.call(document.querySelectorAll('main > section[id]'));
  var tocLinks = {};
  document.querySelectorAll('nav.toc a[href^="#"]').forEach(function (a) { tocLinks[a.getAttribute('href').slice(1)] = a; });
  function onScroll() {
    var h = document.documentElement, max = h.scrollHeight - h.clientHeight;
    if (progress) progress.style.width = (max > 0 ? (h.scrollTop / max) * 100 : 0) + '%';
  }
  window.addEventListener('scroll', onScroll, { passive: true }); onScroll();
  if ('IntersectionObserver' in window && spySections.length) {
    var io = new IntersectionObserver(function (entries) {
      entries.forEach(function (e) {
        var a = tocLinks[e.target.id]; if (!a) return;
        if (e.isIntersecting) { Object.keys(tocLinks).forEach(function (k) { tocLinks[k].classList.remove('active'); }); a.classList.add('active'); }
      });
    }, { rootMargin: '-15% 0px -70% 0px' });
    spySections.forEach(function (s) { io.observe(s); });
  }

  /* ---- Copy-code buttons on code/diff blocks ---- */
  document.querySelectorAll('main pre:not(.mermaid)').forEach(function (pre) {
    var wrap = document.createElement('div'); wrap.className = 'code-wrap';
    pre.parentNode.insertBefore(wrap, pre); wrap.appendChild(pre);
    var b = document.createElement('button'); b.className = 'copy-code'; b.type = 'button';
    b.innerHTML = '<svg class="icon"><use href="#i-copy"/></svg> Copy';
    b.onclick = function () {
      var text = (pre.innerText || pre.textContent || '').replace(/^\s*Copy\s*/, '');
      var done = function () { b.innerHTML = 'Copied ✓'; setTimeout(function () { b.innerHTML = '<svg class="icon"><use href="#i-copy"/></svg> Copy'; }, 1200); };
      if (navigator.clipboard) navigator.clipboard.writeText(text).then(done, done); else done();
    };
    wrap.appendChild(b);
  });

  /* ---- Image lightbox ---- */
  var lb = document.getElementById('vpLightbox'), lbImg = lb && lb.querySelector('img'), lbCap = lb && lb.querySelector('.cap');
  document.querySelectorAll('main img').forEach(function (img) {
    img.addEventListener('click', function () {
      if (!lb) return;
      lbImg.src = img.currentSrc || img.src; lbImg.alt = img.alt || '';
      lbCap.textContent = img.getAttribute('data-caption') || img.alt || '';
      lb.classList.add('open');
    });
  });
  if (lb) lb.addEventListener('click', function () { lb.classList.remove('open'); });

  /* ---- Theme ---- */
  var themeBtn = document.getElementById('themeBtn');
  function syncTheme() {
    var dark = document.documentElement.getAttribute('data-theme') !== 'light';
    themeBtn.querySelector('use').setAttribute('href', dark ? '#i-moon' : '#i-sun');
  }
  function toggleTheme() {
    var next = document.documentElement.getAttribute('data-theme') === 'light' ? 'dark' : 'light';
    document.documentElement.setAttribute('data-theme', next);
    try { localStorage.setItem('vp-theme', next); } catch (e) {}
    syncTheme(); document.dispatchEvent(new Event('vp-theme'));
  }
  themeBtn.onclick = toggleTheme; syncTheme();

  /* ---- Collapsible chevrons ---- */
  document.querySelectorAll('summary').forEach(function (s) {
    var c = document.createElement('span'); c.className = 'chev';
    c.innerHTML = '<svg class="icon"><use href="#i-chev"/></svg>'; s.prepend(c);
  });

  /* ---- Text-range helpers for fine-grained marks (CSS Custom Highlight API) ---- */
  var UI_SKIP = '.chev,.anno-badge,.tpill';
  function cleanTextNodes(block) {
    var nodes = [], w = document.createTreeWalker(block, NodeFilter.SHOW_TEXT, {
      acceptNode: function (n) { return (n.parentElement && n.parentElement.closest(UI_SKIP)) ? NodeFilter.FILTER_REJECT : NodeFilter.FILTER_ACCEPT; }
    });
    var n; while ((n = w.nextNode())) nodes.push(n);
    return nodes;
  }
  /* Clean-text offset of a boundary (container,offset) within block — robust to
     selections that start/end at element boundaries (not just text nodes). */
  function cleanOffset(block, container, offset) {
    var r = document.createRange();
    r.setStart(block, 0);
    try { r.setEnd(container, offset); } catch (e) { return null; }
    var frag = r.cloneContents();
    frag.querySelectorAll(UI_SKIP).forEach(function (n) { n.remove(); });
    return frag.textContent.length;
  }
  function offsetsOf(block, range) {
    var start = cleanOffset(block, range.startContainer, range.startOffset);
    var end = cleanOffset(block, range.endContainer, range.endOffset);
    return (start != null && end != null && end > start) ? { start: start, end: end } : null;
  }
  function rangeFromOffsets(block, s, e) {
    var nodes = cleanTextNodes(block), pos = 0, r = document.createRange(), st = 0;
    for (var i = 0; i < nodes.length; i++) {
      var node = nodes[i], len = node.textContent.length;
      if (st === 0 && s >= pos && s <= pos + len) { r.setStart(node, s - pos); st = 1; }
      if (st === 1 && e <= pos + len) { r.setEnd(node, e - pos); st = 2; break; }
      pos += len;
    }
    return st === 2 ? r : null;
  }
  var HL = !!(window.CSS && CSS.highlights && typeof Highlight !== 'undefined');
  function refreshHighlights(pendingRange) {
    if (!HL) return;
    var ranges = [];
    comments.forEach(function (c) {
      if (c.type !== 'text' || (c.resolved || (c.version != null && c.version < PLAN_VERSION))) return;
      var b = sel(c.anchorId); if (!b) return;
      var r = rangeFromOffsets(b, c.start, c.end); if (r) { c._range = r; ranges.push(r); }
    });
    if (pendingRange) ranges.push(pendingRange);
    CSS.highlights.delete('vp-mark');
    if (ranges.length) { var h = new Highlight(); ranges.forEach(function (r) { h.add(r); }); CSS.highlights.set('vp-mark', h); }
  }

  /* ---- Dock controller: compose (C) and feedback chat (F) both expand the dock ---- */
  var dock = document.getElementById('dock');
  var dpFeedback = document.getElementById('dpFeedback');
  var dpCompose = document.getElementById('dpCompose');
  var fbBtn = document.getElementById('fbPanel');
  var cmtBtn = document.getElementById('cmtMode');
  var genText = document.getElementById('genText');
  var pending = null;
  function composing() { return dock.classList.contains('open') && !dpCompose.hidden; }
  function showSection(which) {
    dpCompose.hidden = which !== 'compose';
    dpFeedback.hidden = which !== 'feedback';
    dock.classList.add('open');
    cmtBtn.classList.toggle('active', which === 'compose');
    fbBtn.classList.toggle('active', which === 'feedback');
    document.body.classList.toggle('assigning', which === 'compose');
  }
  function closeDock() {
    dock.classList.remove('open');
    cmtBtn.classList.remove('active'); fbBtn.classList.remove('active');
    document.body.classList.remove('assigning'); refreshHighlights(null);
  }
  function openFeedback() { showSection('feedback'); render(); }
  function toggleFeedback() {
    if (dock.classList.contains('open') && !dpFeedback.hidden) closeDock(); else openFeedback();
  }
  function openComposer(preset) {
    if (composing() && !preset) { closeDock(); return; }
    pending = null; genText.value = '';
    showSection('compose');
    setTarget(preset && preset.type ? preset : null);
    setTimeout(function () { genText.focus(); }, 60);
  }
  function setTarget(t) {
    pending = t;
    var chip = document.getElementById('cTarget'), label = chip.querySelector('span'), clr = chip.querySelector('.tclear'), ic = chip.querySelector('use');
    chip.classList.toggle('pinned', !!t); clr.hidden = !t;
    if (!t) { label.textContent = 'Whole plan'; ic.setAttribute('href', '#i-file'); refreshHighlights(null); }
    else if (t.type === 'block') { label.textContent = t.anchorLabel; ic.setAttribute('href', '#i-message'); refreshHighlights(null); }
    else if (t.type === 'node') { label.textContent = t.anchorLabel + ' · “' + (t.quote || '').slice(0, 40) + '”'; ic.setAttribute('href', '#i-message'); refreshHighlights(null); }
    else { label.textContent = '“' + t.quote.slice(0, 48) + (t.quote.length > 48 ? '…' : '') + '”'; ic.setAttribute('href', '#i-message'); refreshHighlights(t._range); }
  }
  function saveComposer() {
    var b = genText.value.trim(); if (!b) { genText.focus(); return; }
    var c = { body: b, ts: new Date().toISOString() };
    if (!pending) { c.type = 'general'; c.anchorId = '__general__'; c.anchorLabel = 'General'; c.quote = ''; }
    else if (pending.type === 'block') { c.type = 'block'; c.anchorId = pending.anchorId; c.anchorLabel = pending.anchorLabel; c.quote = ''; }
    else if (pending.type === 'node') { c.type = 'node'; c.anchorId = pending.anchorId; c.anchorLabel = pending.anchorLabel; c.quote = pending.quote; }
    else { c.type = 'text'; c.anchorId = pending.anchorId; c.anchorLabel = pending.anchorLabel; c.quote = pending.quote; c.start = pending.start; c.end = pending.end; }
    c.version = PLAN_VERSION;
    comments.push(c); save(); pending = null; genText.value = ''; openFeedback();
  }
  cmtBtn.onclick = function () { openComposer(); };
  fbBtn.onclick = toggleFeedback;
  document.getElementById('dpClose').onclick = closeDock;
  document.getElementById('cCancel').onclick = closeDock;
  document.getElementById('cSave').onclick = saveComposer;
  document.querySelector('#cTarget .tclear').onclick = function () { setTarget(null); genText.focus(); };
  genText.onkeydown = function (e) { if ((e.metaKey || e.ctrlKey) && e.key === 'Enter') saveComposer(); };

  /* ---- Build a text target from the current selection (used by compose-mode + sel-pop) ---- */
  function selectionTarget() {
    var s = window.getSelection ? window.getSelection() : null;
    if (!s || s.isCollapsed || !s.rangeCount) return null;
    var range = s.getRangeAt(0), host = range.commonAncestorContainer;
    host = (host.nodeType === 1 ? host : host.parentElement);
    var block = host && host.closest('[data-anchor-id]');
    if (!block) return null;
    var offs = offsetsOf(block, range), quote = String(s).replace(/\s+/g, ' ').trim();
    if (!offs || !quote) return null;
    return { type: 'text', anchorId: block.getAttribute('data-anchor-id'), anchorLabel: block.getAttribute('data-anchor-label'),
      quote: quote.slice(0, 160), start: offs.start, end: offs.end, _range: rangeFromOffsets(block, offs.start, offs.end), _rect: range.getBoundingClientRect() };
  }

  /* ---- Selection popover: select any text -> "Comment this section" ---- */
  var selPop = document.getElementById('selPop'), selTarget = null;
  function showSelPop(t) {
    selTarget = t; var r = t._rect;
    selPop.style.left = Math.max(8, Math.min(r.left + r.width / 2 - 95, window.innerWidth - 210)) + 'px';
    selPop.style.top = Math.max(8, r.top - 46) + 'px';
    selPop.classList.add('show');
  }
  function hideSelPop() { selPop.classList.remove('show'); selTarget = null; }
  selPop.querySelector('button').onmousedown = function (e) { e.preventDefault(); };
  selPop.querySelector('button').onclick = function (e) {
    e.stopPropagation(); var t = selTarget; hideSelPop();
    if (window.getSelection) window.getSelection().removeAllRanges();
    if (t) openComposer(t);
  };
  window.addEventListener('scroll', hideSelPop, { passive: true });

  /* assign a target while composing; otherwise offer the sel-pop */
  document.addEventListener('mouseup', function (e) {
    if (e.target.closest('.dock,.kbd-help,.sel-pop')) return;
    if (composing()) {
      var t = selectionTarget();
      if (t) { setTarget(t); window.getSelection().removeAllRanges(); genText.focus(); toast('Pinned to selection'); return; }
      if (e.target.closest('a, button, summary, input, textarea')) return;
      var blk = e.target.closest('[data-anchor-id]');
      if (blk) { setTarget({ type: 'block', anchorId: blk.getAttribute('data-anchor-id'), anchorLabel: blk.getAttribute('data-anchor-label') }); genText.focus(); toast('Pinned to ' + blk.getAttribute('data-anchor-label')); }
      return;
    }
    setTimeout(function () { var st = selectionTarget(); if (st) showSelPop(st); else hideSelPop(); }, 0);
  });

  /* ---- Click a Mermaid node -> comment on it; click a marked span -> open its comment ---- */
  document.addEventListener('click', function (e) {
    if (composing() || e.target.closest('.dock,.kbd-help,.vp-lightbox,.sel-pop,.version')) return;
    var node = e.target.closest('.mermaid g.node, .mermaid .node, .mermaid .nodeLabel');
    if (node) {
      var dblock = node.closest('[data-anchor-id]'); if (dblock) {
        var ntext = (node.textContent || '').replace(/\s+/g, ' ').trim().slice(0, 80);
        var nr = node.getBoundingClientRect();
        showSelPop({ type: 'node', anchorId: dblock.getAttribute('data-anchor-id'), anchorLabel: dblock.getAttribute('data-anchor-label'),
          quote: ntext, _rect: nr });
        return;
      }
    }
    var x = e.clientX, y = e.clientY, hit = null;
    comments.forEach(function (c) {
      if (hit || c.type !== 'text' || c.resolved || (c.version != null && c.version < PLAN_VERSION)) return;
      var b = sel(c.anchorId); if (!b) return;
      var r = c._range || rangeFromOffsets(b, c.start, c.end); if (!r) return;
      var rects = r.getClientRects();
      for (var i = 0; i < rects.length; i++) { var box = rects[i]; if (x >= box.left && x <= box.right && y >= box.top && y <= box.bottom) { hit = c; break; } }
    });
    if (hit) { openFeedback(); flashChat(hit); }
  });

  /* ---- Render markers + chat ---- */
  function timeAgo(ts) {
    if (!ts) return '';
    var diff = (Date.now() - new Date(ts).getTime()) / 1000;
    if (diff < 45) return 'just now';
    if (diff < 3600) return Math.round(diff / 60) + 'm ago';
    if (diff < 86400) return Math.round(diff / 3600) + 'h ago';
    return Math.round(diff / 86400) + 'd ago';
  }
  function flashChat(c) {
    if (c.resolved) return;
    var node = document.querySelector('#annoList .msg[data-ci="' + comments.indexOf(c) + '"]');
    if (node) { node.scrollIntoView({ behavior: 'smooth', block: 'center' }); node.classList.add('anno-flash'); setTimeout(function () { node.classList.remove('anno-flash'); }, 1300); }
  }
  /* a comment is "earlier" once it's resolved or was made against an older plan version */
  function isEarlier(c) { return !!c.resolved || (c.version != null && c.version < PLAN_VERSION); }
  function msgLabel(c) {
    if (c.type === 'general' || c.anchorId === '__general__') return 'Whole plan';
    return c.anchorLabel + (c.type === 'text' ? ' · text' : c.type === 'node' ? ' · node' : '');
  }
  function buildMsg(c, listEl, earlier) {
    var gen = c.type === 'general' || c.anchorId === '__general__';
    var d = document.createElement('div'); d.className = 'msg' + (gen ? ' general' : '') + (earlier ? ' resolved' : '');
    d.setAttribute('data-ci', comments.indexOf(c));
    var acts = earlier
      ? '<button class="restore-c" title="Restore">&#8634;</button><button class="del-c" title="Delete">&times;</button>'
      : '<button class="edit-c" title="Edit">&#9998;</button><button class="resolve-c" title="Mark resolved">&#10003;</button><button class="del-c" title="Delete">&times;</button>';
    d.innerHTML = '<span class="acts">' + acts + '</span>' +
      '<div class="lbl">' + esc(msgLabel(c)) + '</div>' +
      (c.quote ? '<div class="quote">“' + esc(c.quote) + '”</div>' : '') +
      '<div class="body">' + esc(c.body) + '</div>' +
      '<div class="meta">' + esc(timeAgo(c.ts)) + (c.version && c.version < PLAN_VERSION ? ' · v' + c.version : '') + (c.edited ? ' · edited' : '') + '</div>';
    if (!gen) d.querySelector('.lbl').onclick = function () { jumpTo(c); };
    d.querySelector('.del-c').onclick = function () { var ix = comments.indexOf(c); if (ix >= 0) { comments.splice(ix, 1); save(); render(); } };
    if (earlier) d.querySelector('.restore-c').onclick = function () { c.resolved = false; c.version = PLAN_VERSION; save(); render(); };
    else {
      d.querySelector('.edit-c').onclick = function () { startEdit(d, c); };
      d.querySelector('.resolve-c').onclick = function () { c.resolved = true; save(); render(); };
    }
    listEl.appendChild(d);
  }
  function render() {
    document.querySelectorAll('.anno-badge').forEach(function (n) { n.remove(); });
    document.querySelectorAll('.has-anno').forEach(function (n) { n.classList.remove('has-anno'); });
    var active = comments.filter(function (c) { return !isEarlier(c); });
    var earlier = comments.filter(isEarlier);
    var byId = {};
    active.forEach(function (c) { if (c.anchorId && c.anchorId !== '__general__') (byId[c.anchorId] = byId[c.anchorId] || []).push(c); });
    Object.keys(byId).forEach(function (id) {
      var el = sel(id); if (!el) return;
      el.classList.add('has-anno');
      var badge = document.createElement('span'); badge.className = 'anno-badge';
      badge.innerHTML = '<svg class="icon"><use href="#i-message"/></svg>' + byId[id].length;
      (el.querySelector('h2, summary, strong') || el).appendChild(badge);
    });
    document.getElementById('annoCount').textContent = active.length || '';
    var c2 = document.getElementById('annoCount2'); if (c2) c2.textContent = active.length ? '(' + active.length + ')' : '';
    var list = document.getElementById('annoList'); list.innerHTML = '';
    if (!active.length && !earlier.length) { list.innerHTML = '<p class="empty">No comments yet. Press <b>c</b> or select text, then pin it — or just save for the whole plan.</p>'; refreshHighlights(null); return; }
    if (!active.length) list.insertAdjacentHTML('beforeend', '<p class="empty">No open comments — see earlier below.</p>');
    active.forEach(function (c) { buildMsg(c, list, false); });
    if (earlier.length) {
      var det = document.createElement('details'); det.className = 'earlier';
      det.innerHTML = '<summary>Earlier / resolved (' + earlier.length + ')</summary>';
      var sub = document.createElement('div'); det.appendChild(sub);
      earlier.forEach(function (c) { buildMsg(c, sub, true); });
      list.appendChild(det);
    }
    refreshHighlights(null);
  }
  function startEdit(node, c) {
    var bodyEl = node.querySelector('.body'); if (!bodyEl || node.querySelector('.edit-ta')) return;
    bodyEl.style.display = 'none';
    var ta = document.createElement('textarea'); ta.className = 'edit-ta'; ta.value = c.body;
    var row = document.createElement('div'); row.className = 'edit-row';
    row.innerHTML = '<button class="btn xs" data-x type="button">Cancel</button><button class="btn xs accent" data-ok type="button">Save</button>';
    bodyEl.insertAdjacentElement('afterend', ta); ta.insertAdjacentElement('afterend', row); ta.focus();
    function fin() { ta.remove(); row.remove(); bodyEl.style.display = ''; }
    row.querySelector('[data-x]').onclick = fin;
    row.querySelector('[data-ok]').onclick = function () { var v = ta.value.trim(); if (v) { c.body = v; c.edited = true; save(); render(); } else fin(); };
    ta.onkeydown = function (e) { if ((e.metaKey || e.ctrlKey) && e.key === 'Enter') row.querySelector('[data-ok]').click(); if (e.key === 'Escape') { e.stopPropagation(); fin(); } };
  }
  function jumpTo(c) {
    var el = sel(c.anchorId); if (!el) return;
    var det = el.closest('details'); if (det) det.open = true;
    if (c.type === 'text') {
      var r = rangeFromOffsets(el, c.start, c.end);
      if (r) { var rect = r.getBoundingClientRect(); window.scrollTo({ top: window.scrollY + rect.top - 140, behavior: 'smooth' }); }
      else el.scrollIntoView({ behavior: 'smooth', block: 'center' });
    } else el.scrollIntoView({ behavior: 'smooth', block: 'center' });
    el.classList.add('anno-flash'); setTimeout(function () { el.classList.remove('anno-flash'); }, 1350);
  }

  /* ---- Export / copy ---- */
  function toast(msg) {
    var t = document.createElement('div'); t.className = 'vp-toast'; t.textContent = msg;
    document.body.appendChild(t); setTimeout(function () { t.remove(); }, 1500);
  }
  function hasFeedback() { return comments.length > 0 || Object.keys(answers).length > 0; }
  function payload() {
    var out = '# Plan feedback: ' + PLAN + ' (v' + PLAN_VERSION + ')\n';
    var ids = Object.keys(answers).filter(function (k) { return answers[k] && answers[k].answer; });
    if (ids.length) {
      out += '\n## Decisions (Open Questions)\n' + ids.map(function (k) {
        return '- ' + (answers[k].question || k) + ' → ' + answers[k].answer;
      }).join('\n') + '\n';
    }
    var active = comments.filter(function (c) { return !isEarlier(c); });  // drop resolved / older-version
    if (active.length) {
      out += '\n## Comments\n' + active.map(function (c, i) {
        var head = c.anchorId === '__general__' ? '[general]' : '[anchor-id: ' + c.anchorId + '] (' + c.anchorLabel + ')' + (c.type === 'node' ? ' [node]' : '');
        return (i + 1) + '. ' + head + (c.quote ? '\n   quote: "' + c.quote + '"' : '') + '\n   comment: ' + c.body;
      }).join('\n\n') + '\n';
    }
    return out;
  }
  function copyFeedback() {
    if (!hasFeedback()) { toast('Nothing to copy yet'); return; }
    var t = payload();
    var done = function () { toast('Feedback copied ✓'); };
    if (navigator.clipboard) navigator.clipboard.writeText(t).then(done, done);
    else { var a = document.createElement('textarea'); a.value = t; document.body.appendChild(a); a.select(); try { document.execCommand('copy'); } catch (e) {} a.remove(); done(); }
  }
  document.getElementById('copyDock').onclick = copyFeedback;
  document.getElementById('dlFb').onclick = function () {
    if (!hasFeedback()) { toast('Nothing to download yet'); return; }
    var blob = new Blob([payload()], { type: 'text/markdown' });
    var a = document.createElement('a'); a.href = URL.createObjectURL(blob); a.download = 'plan-feedback.md'; a.click(); URL.revokeObjectURL(a.href);
  };

  /* ---- Keyboard shortcuts + help ---- */
  var help = document.getElementById('kbdHelp');
  function toggleHelp() { help.hidden = !help.hidden; }
  document.getElementById('kbdBtn').onclick = toggleHelp;
  document.getElementById('kbdClose').onclick = toggleHelp;
  document.addEventListener('keydown', function (e) {
    // ⌘/Ctrl+C copies all feedback — but only when nothing is selected, so normal copy still works
    if ((e.metaKey || e.ctrlKey) && (e.key === 'c' || e.key === 'C')) {
      if (e.target.matches('textarea, input')) return;
      var s = window.getSelection ? String(window.getSelection()) : '';
      if (!s) { copyFeedback(); e.preventDefault(); }
      return;
    }
    if (e.target.matches('textarea, input')) { if (e.key === 'Escape') closeDock(); return; }
    if (e.metaKey || e.ctrlKey || e.altKey) return;
    switch (e.key) {
      case 'c': openComposer(); break;
      case 'f': toggleFeedback(); break;
      case 't': toggleTheme(); break;
      case '?': toggleHelp(); break;
      case 'Escape': help.hidden = true; closeDock(); break;
      default: return;
    }
    e.preventDefault();
  });

  render();
})();
</script>
</body>
</html>
```

## Notes for the agent

- **Mermaid** is the default for flow/sequence/state/ER/dependency diagrams; keep
  the `.diagram` brand-logo kit only for product/network maps where logos matter.
  Author Mermaid as `<pre class="mermaid">…</pre>`; it re-themes on toggle. **Write
  valid, simple Mermaid** — bad syntax falls back to showing the source, not a
  rendered diagram, so it pays to keep it clean:
  - **Always quote node labels** that contain spaces, punctuation, parentheses,
    slashes, `.`, `:`, backticks, or code — `A["benchy run"]`, not `A[benchy run]`;
    `B["Parse → filter"]`, not `B[Parse (filter)]`.
  - Stick to common diagram types (`flowchart TD/LR`, `sequenceDiagram`,
    `stateDiagram-v2`, `erDiagram`) and basic edges (`-->`, `-->|label|`).
  - One statement per line; no trailing stray characters; don't put raw `<`/`>` or
    unescaped quotes inside labels. Prefer a few small diagrams over one huge one.
  - When a relationship is mostly boxes/layers with product logos, use the CSS
    `.diagram` kit instead — it never has a syntax failure mode.
- **File trees:** nested `<ul class="filetree">`, each label wrapped in `<span>`;
  a folder is any `<li>` that contains a nested `<ul>`. Mark files with
  `data-new`/`data-edit` for pills. Never ship a file map as a `<pre>` text block.
- **Open questions as answerable chips.** Each question is a `<div class="qa"
  data-qid="…" data-q="…">` with a `.q-text`, a `.q-opts` row of `<button
  class="qopt">` (one per real choice), and an optional `.q-other` write-in input.
  Mark the recommended option `data-rec` — it's **pre-selected by default** so the
  user approves by exception. Selections persist and export under "Decisions". Don't
  fall back to plain "Recommended default: …" prose.
- **Header metadata + version:** fill `.meta` from git — repo URL + `owner/repo`,
  current branch, today's date — and set the version `vN` in the `#vBtn`. Fill the
  `#versionPop` rows with each prior version (`vN` · date · its deployed URL); the
  newest is `current`. Bump `vN` on each materially-new revision.
- **Commenting is automatic:** selecting any text shows "Comment this section";
  clicking a Mermaid node comments on that node. Don't add your own comment buttons.
- **Badges are subtle by design** (`.tag`, `.tpill`, `.anno-badge`, version pill) —
  small, low-contrast, dot/ghost style. Don't restyle them into loud filled pills.
- **Card grids auto-balance:** put cards in `.grid`; JS picks the column count so
  the last row isn't an orphan (4 → 2×2). Use `.grid` (not `.dlayer`) for card sets.
- **Callouts carry a tone:** default is warn; add `.ok` (green), `.info` (accent),
  or `.risk` (red). Squared corners — don't round them.
- **Images get a free lightbox.** Any `<img>` inside `main` is zoomable (click →
  fullscreen, Esc/click to close); add `data-caption="…"` for a caption. Use real
  `<img>` for screenshots rather than describing them.
- **These reader features are automatic — don't re-implement:** scroll-spy TOC +
  reading-progress bar, copy-code buttons on every `pre`, the image lightbox,
  clickable text marks (click a highlight → its comment), comment edit + relative
  timestamps, `prefers-reduced-motion`, and a print/PDF stylesheet (dock hidden,
  details expanded, link URLs shown). Just author content; the plumbing handles it.
- **Stable anchor ids** derived from content; feedback round-trips on them.
  General comments use the reserved `__general__` id and export as `[general]`.
- **Brand logos:** `<svg class="brand" style="color:#BRAND"><use href="#b-NAME"/></svg>`.
  Hexes: WhatsApp `#25D366`, Telegram `#26A5E4`, Signal `#3A76F0`, Discord
  `#5865F2`, Slack `#4A154B`, Instagram `#E4405F`, Messenger `#0084FF`, Google
  `#4285F4`, GitHub `currentColor`, LinkedIn `#0A66C2`, X `currentColor`. Add new
  logos only by copying exact Simple Icons paths — never approximate.
- **Reuse the dock, compose/pin, shortcut, and annotation plumbing verbatim** —
  change content, palette tokens, and diagram/tree contents only.
