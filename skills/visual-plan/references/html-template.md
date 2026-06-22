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
  /* ===== Palette (foglamp-inspired: neutral grayscale surfaces, layered shadows,
           clay accent used sparingly — foglamp's own Anthropic vendor color) ===== */
  :root { --maxw: 1080px; --pad: 22px; --panel: 360px; }
  :root[data-theme="dark"] {
    --bg: #1c1c1c; --bg-elev: #232323; --card: #2e2e2e; --card-2: #3d3d3d;
    --line: rgba(255,255,255,.09); --line-strong: rgba(255,255,255,.16); --fg: #fafafa; --muted: #9e9e9e;
    --accent: #d97757; --accent-soft: rgba(217,119,87,.14); --accent-line: rgba(217,119,87,.42); --on-accent: #1c1c1c;
    --add: #86d39a; --add-bg: rgba(34,197,94,.13); --add-line: rgba(34,197,94,.3);
    --del: #f1808e; --del-bg: rgba(244,63,94,.13); --del-line: rgba(244,63,94,.34);
    --warn: #e3b341; --warn-bg: rgba(245,158,11,.12); --warn-line: rgba(245,158,11,.34);
    --card-shadow: inset 0 1px 0 0 rgba(255,255,255,.03), inset 0 0 0 1px rgba(255,255,255,.04), 0 0 0 1px rgba(0,0,0,.12), 0 2px 2px rgba(0,0,0,.1), 0 4px 4px rgba(0,0,0,.1);
    --shadow: 0 12px 40px rgba(0,0,0,.55);
  }
  :root[data-theme="light"] {
    --bg: #fbfbfa; --bg-elev: #ffffff; --card: #ffffff; --card-2: #f3f3f1;
    --line: rgba(0,0,0,.10); --line-strong: rgba(0,0,0,.16); --fg: #232323; --muted: #777777;
    --accent: #c15f3c; --accent-soft: rgba(193,95,60,.10); --accent-line: rgba(193,95,60,.4); --on-accent: #ffffff;
    --add: #2f7d3a; --add-bg: rgba(34,197,94,.12); --add-line: rgba(34,197,94,.35);
    --del: #c0392b; --del-bg: rgba(244,63,94,.1); --del-line: rgba(244,63,94,.35);
    --warn: #97751f; --warn-bg: rgba(245,158,11,.13); --warn-line: rgba(245,158,11,.4);
    --card-shadow: 0 0 0 1px rgba(0,0,0,.06), 0 1px 2px -1px rgba(0,0,0,.08), 0 2px 4px rgba(0,0,0,.04);
    --shadow: 0 12px 40px rgba(40,40,40,.16);
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
  .version-pop .vrow .vlabel { color: var(--muted); font-size: 12px; }
  .version-pop .vrow .vd { color: var(--muted); font-size: 11.5px; margin-left: auto; }
  .version-pop a.vrow { cursor: pointer; }
  .version-pop a.vrow .vn::after { content: " ↗"; color: var(--muted); font-weight: 400; }
  .version-pop .vrow.current { box-shadow: inset 0 0 0 1.5px var(--accent-line); }

  /* ===== Subtle badges (foglamp-style: small, low-contrast, dot/ghost — never loud pills) ===== */
  .tag { display: inline-flex; align-items: center; gap: 4px; font-size: 11px; line-height: 1.6; border-radius: 999px;
    padding: 0 8px; background: var(--card-2); color: var(--muted); border: 1px solid var(--line); }
  .tag::before { content: ""; width: 5px; height: 5px; border-radius: 50%; background: currentColor; opacity: .9; }
  .tag.ok { color: var(--add); } .tag.warn { color: var(--warn); } .tag.info { color: var(--accent); } .tag.risk { color: var(--del); }

  /* ===== Blocks ===== */
  .card { background: var(--card); border-radius: 16px; padding: 16px; box-shadow: var(--card-shadow); }
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
  /* foglamp callout: faint tint + 1px colored border; body stays readable, tone via border/tint (default = warn) */
  .callout { border: 1px solid var(--warn-line); background: var(--warn-bg); color: var(--fg); padding: 11px 15px; border-radius: 10px; }
  table { border-collapse: collapse; width: 100%; font-size: 14px; }
  th, td { border: 1px solid var(--line); padding: 7px 11px; text-align: left; }
  th { background: var(--card-2); }

  /* ===== Open questions — answer chips, recommended pre-selected ===== */
  .qa { margin: 16px 0; }
  .qa + .qa { padding-top: 16px; border-top: 1px solid var(--line); }
  .qa .q-text { font-weight: 600; margin-bottom: 9px; }
  .qa .q-opts { display: flex; flex-wrap: wrap; gap: 8px; }
  /* foglamp toggle/chip: transparent by default, MUTED-gray when selected (not a solid fill) */
  .qopt { font: inherit; font-size: 13px; cursor: pointer; background: transparent; color: var(--fg);
    box-shadow: var(--card-shadow); border: 0; border-radius: 14px; padding: 8px 13px; display: inline-flex; align-items: center; gap: 8px; transition: background .12s; }
  .qopt:hover { background: var(--card-2); }
  .qopt.sel { background: var(--card-2); color: var(--fg); box-shadow: 0 0 0 1.5px var(--accent-line), var(--card-shadow); }
  .qopt.sel .rec-tag { color: var(--accent); }
  .qopt .rec-tag { font-size: 10.5px; color: var(--muted); }
  .qa .q-other { margin-top: 9px; }
  .qa .q-other input { width: 100%; max-width: 380px; background: var(--card); color: var(--fg);
    border: 1px solid var(--line); border-radius: 7px; padding: 7px 10px; font: inherit; font-size: 13px; }
  .qa .q-other input:focus { outline: none; border-color: var(--accent); }

  /* ===== Mermaid — fixed-size viewport with pan/zoom (long diagrams stay readable) ===== */
  .mermaid { position: relative; height: clamp(280px, 56vh, 560px); margin: 14px 0; border-radius: 16px;
    background: var(--card); box-shadow: var(--card-shadow); overflow: hidden; color: transparent; }
  .mermaid.done { color: inherit; }
  .mermaid > svg { position: absolute; top: 0; left: 0; transform-origin: 0 0; max-width: none !important; cursor: grab; user-select: none; }
  .mermaid.grab > svg { cursor: grabbing; }
  .mermaid.fs { position: fixed; inset: 3vh 3vw; height: auto; width: auto; z-index: 95; }
  .mz-bar { position: absolute; bottom: 9px; right: 9px; z-index: 3; display: flex; gap: 2px; padding: 3px;
    background: color-mix(in srgb, var(--bg-elev) 88%, transparent); backdrop-filter: blur(8px); border-radius: 11px; box-shadow: var(--card-shadow); }
  .mz-bar button { min-width: 27px; height: 27px; padding: 0 7px; display: grid; place-items: center; border: 0; background: transparent;
    color: var(--muted); cursor: pointer; border-radius: 8px; font: inherit; font-size: 14px; }
  .mz-bar button:hover { background: var(--card-2); color: var(--fg); }
  .mz-hint { position: absolute; bottom: 12px; left: 13px; z-index: 3; font-size: 11px; color: var(--muted); pointer-events: none; }
  .mermaid-fallback { height: auto; overflow: auto; color: var(--fg) !important; padding: 14px; text-align: left; }
  .mermaid-fallback > svg, .mermaid-fallback .mz-bar, .mermaid-fallback .mz-hint { display: none; }
  .mermaid-note { font-size: 12px; color: var(--warn); margin-bottom: 8px; }
  .mermaid-fallback pre { background: var(--bg-elev); border: 1px solid var(--line); border-radius: 8px; padding: 10px; margin: 0; white-space: pre; overflow: auto; font-size: 12.5px; color: var(--fg); }

  /* ===== Brand-logo diagram kit (for product/network maps) ===== */
  .diagram { border-radius: 16px; background: var(--card); box-shadow: var(--card-shadow); padding: 18px; margin: 14px 0; }
  .dlayer { display: grid; gap: 10px; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); margin-bottom: 10px; }
  .dnode { background: var(--bg-elev); box-shadow: inset 0 0 0 1px var(--line); border-radius: 12px; padding: 11px 13px; }
  .dnode .t { font-weight: 600; display: flex; align-items: center; gap: 8px; }
  .dnode .s { color: var(--muted); font-size: 12.5px; margin-top: 3px; }
  .dbar { background: var(--bg-elev); box-shadow: inset 0 0 0 1px var(--line); border-radius: 12px; padding: 12px 14px; text-align: center; margin-bottom: 10px; }
  .dbar .t { font-weight: 600; } .dbar .s { color: var(--muted); font-size: 12.5px; margin-top: 2px; }
  .dbar.ours { box-shadow: inset 0 0 0 1.5px var(--accent-line); } .dbar.ours .t { color: var(--accent); }
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
  .callout.ok { border-color: var(--add-line); background: var(--add-bg); }
  .callout.info { border-color: var(--accent-line); background: var(--accent-soft); }
  .callout.risk { border-color: var(--del-line); background: var(--del-bg); }
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
  <symbol id="i-expand" viewBox="0 0 24 24"><path d="M4 8v-2a2 2 0 0 1 2 -2h2"/><path d="M4 16v2a2 2 0 0 0 2 2h2"/><path d="M16 4h2a2 2 0 0 1 2 2v2"/><path d="M16 20h2a2 2 0 0 0 2 -2v-2"/></symbol>
  <symbol id="i-cpu" viewBox="0 0 24 24"><path d="M5 5m0 1a1 1 0 0 1 1 -1h12a1 1 0 0 1 1 1v12a1 1 0 0 1 -1 1h-12a1 1 0 0 1 -1 -1z"/><path d="M9 9h6v6h-6z"/><path d="M3 10h2M3 14h2M10 3v2M14 3v2M21 10h-2M21 14h-2M10 21v-2M14 21v-2"/></symbol>
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
  <symbol id="b-cloudflare" viewBox="0 0 24 24"><path d="M16.5088 16.8447c.1475-.5068.0908-.9707-.1553-1.3154-.2246-.3164-.6045-.499-1.0615-.5205l-8.6592-.1123a.1559.1559 0 0 1-.1333-.0713c-.0283-.042-.0351-.0986-.021-.1553.0278-.084.1123-.1484.2036-.1562l8.7359-.1123c1.0351-.0489 2.1601-.8868 2.5537-1.9136l.499-1.3013c.0215-.0561.0293-.1128.0147-.168-.5625-2.5463-2.835-4.4453-5.5499-4.4453-2.5039 0-4.6284 1.6177-5.3876 3.8614-.4927-.3658-1.1187-.5625-1.794-.499-1.2026.119-2.1665 1.083-2.2861 2.2856-.0283.31-.0069.6128.0635.894C1.5683 13.171 0 14.7754 0 16.752c0 .1748.0142.3515.0352.5273.0141.083.0844.1475.1689.1475h15.9814c.0909 0 .1758-.0645.2032-.1553l.12-.4268zm2.7568-5.5634c-.0771 0-.1611 0-.2383.0112-.0566 0-.1054.0415-.127.0976l-.3378 1.1744c-.1475.5068-.0918.9707.1543 1.3164.2256.3164.6055.498 1.0625.5195l1.8437.1133c.0557 0 .1055.0263.1329.0703.0283.043.0351.1074.0214.1562-.0283.084-.1132.1485-.204.1553l-1.921.1123c-1.041.0488-2.1582.8867-2.5527 1.914l-.1406.3585c-.0283.0713.0215.1416.0986.1416h6.5977c.0771 0 .1474-.0489.169-.126.1122-.4082.1757-.837.1757-1.2803 0-2.6025-2.125-4.727-4.7344-4.727"/></symbol>
  <symbol id="b-vercel" viewBox="0 0 24 24"><path d="M24 22.525H0l12-21.05 12 21.05z"/></symbol>
  <symbol id="b-openai" viewBox="0 0 24 24"><path d="M22.2819 9.8211a5.9847 5.9847 0 0 0-.5157-4.9108 6.0462 6.0462 0 0 0-6.5098-2.9A6.0651 6.0651 0 0 0 4.9807 4.1818a5.9847 5.9847 0 0 0-3.9977 2.9 6.0462 6.0462 0 0 0 .7427 7.0966 5.98 5.98 0 0 0 .511 4.9107 6.051 6.051 0 0 0 6.5146 2.9001A5.9847 5.9847 0 0 0 13.2599 24a6.0557 6.0557 0 0 0 5.7718-4.2058 5.9894 5.9894 0 0 0 3.9977-2.9001 6.0557 6.0557 0 0 0-.7475-7.0729zm-9.022 12.6081a4.4755 4.4755 0 0 1-2.8764-1.0408l.1419-.0804 4.7783-2.7582a.7948.7948 0 0 0 .3927-.6813v-6.7369l2.02 1.1686a.071.071 0 0 1 .038.052v5.5826a4.504 4.504 0 0 1-4.4945 4.4944zm-9.6607-4.1254a4.4708 4.4708 0 0 1-.5346-3.0137l.142.0852 4.783 2.7582a.7712.7712 0 0 0 .7806 0l5.8428-3.3685v2.3324a.0804.0804 0 0 1-.0332.0615L9.74 19.9502a4.4992 4.4992 0 0 1-6.1408-1.6464zM2.3408 7.8956a4.485 4.485 0 0 1 2.3655-1.9728V11.6a.7664.7664 0 0 0 .3879.6765l5.8144 3.3543-2.0201 1.1685a.0757.0757 0 0 1-.071 0l-4.8303-2.7865A4.504 4.504 0 0 1 2.3408 7.872zm16.5963 3.8558L13.1038 8.364 15.1192 7.2a.0757.0757 0 0 1 .071 0l4.8303 2.7913a4.4944 4.4944 0 0 1-.6765 8.1042v-5.6772a.79.79 0 0 0-.407-.667zm2.0107-3.0231l-.142-.0852-4.7735-2.7818a.7759.7759 0 0 0-.7854 0L9.409 9.2297V6.8974a.0662.0662 0 0 1 .0284-.0615l4.8303-2.7866a4.4992 4.4992 0 0 1 6.6802 4.66zM8.3065 12.863l-2.02-1.1638a.0804.0804 0 0 1-.038-.0567V6.0742a4.4992 4.4992 0 0 1 7.3757-3.4537l-.142.0805L8.704 5.459a.7948.7948 0 0 0-.3927.6813zm1.0976-2.3654l2.602-1.4998 2.6069 1.4998v2.9994l-2.5974 1.4997-2.6067-1.4997Z"/></symbol>
  <symbol id="b-anthropic" viewBox="0 0 24 24"><path d="M17.3041 3.541h-3.6718l6.696 16.918H24Zm-10.6082 0L0 20.459h3.7442l1.3693-3.5527h7.0052l1.3693 3.5528h3.7442L10.5363 3.5409Zm-.3712 10.2232 2.2914-5.9456 2.2914 5.9456Z"/></symbol>
  <symbol id="b-gemini" viewBox="0 0 24 24"><path d="M11.04 19.32Q12 21.51 12 24q0-2.49.93-4.68.96-2.19 2.58-3.81t3.81-2.55Q21.51 12 24 12q-2.49 0-4.68-.93a12.3 12.3 0 0 1-3.81-2.58 12.3 12.3 0 0 1-2.58-3.81Q12 2.49 12 0q0 2.49-.96 4.68-.93 2.19-2.55 3.81a12.3 12.3 0 0 1-3.81 2.58Q2.49 12 0 12q2.49 0 4.68.96 2.19.93 3.81 2.55t2.55 3.81"/></symbol>
  <symbol id="b-perplexity" viewBox="0 0 24 24"><path d="M22.3977 7.0896h-2.3106V.0676l-7.5094 6.3542V.1577h-1.1554v6.1966L4.4904 0v7.0896H1.6023v10.3976h2.8882V24l6.932-6.3591v6.2005h1.1554v-6.0469l6.9318 6.1807v-6.4879h2.8882V7.0896zm-3.4657-4.531v4.531h-5.355l5.355-4.531zm-13.2862.0676 4.8691 4.4634H5.6458V2.6262zM2.7576 16.332V8.245h7.8476l-6.1149 6.1147v1.9723H2.7576zm2.8882 5.0404v-3.8852h.0001v-2.6488l5.7763-5.7764v7.0111l-5.7764 5.2993zm12.7086.0248-5.7766-5.1509V9.0618l5.7766 5.7766v6.5588zm2.8882-5.0652h-1.733v-1.9723L13.3948 8.245h7.8478v8.087z"/></symbol>
  <symbol id="b-supabase" viewBox="0 0 24 24"><path d="M11.9 1.036c-.015-.986-1.26-1.41-1.874-.637L.764 12.05C-.33 13.427.65 15.455 2.409 15.455h9.579l.113 7.51c.014.985 1.259 1.408 1.873.636l9.262-11.653c1.093-1.375.113-3.403-1.645-3.403h-9.642z"/></symbol>
  <symbol id="b-googlecloud" viewBox="0 0 24 24"><path d="M12.19 2.38a9.344 9.344 0 0 0-9.234 6.893c.053-.02-.055.013 0 0-3.875 2.551-3.922 8.11-.247 10.941l.006-.007-.007.03a6.717 6.717 0 0 0 4.077 1.356h5.173l.03.03h5.192c6.687.053 9.376-8.605 3.835-12.35a9.365 9.365 0 0 0-2.821-4.552l-.043.043.006-.05A9.344 9.344 0 0 0 12.19 2.38zm-.358 4.146c1.244-.04 2.518.368 3.486 1.15a5.186 5.186 0 0 1 1.862 4.078v.518c3.53-.07 3.53 5.262 0 5.193h-5.193l-.008.009v-.04H6.785a2.59 2.59 0 0 1-1.067-.23h.001a2.597 2.597 0 1 1 3.437-3.437l3.013-3.012A6.747 6.747 0 0 0 8.11 8.24c.018-.01.04-.026.054-.023a5.186 5.186 0 0 1 3.67-1.69z"/></symbol>
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
          <!-- Each prior version is a LINK only if it has its own deployed URL. The current
               version and any version with no separate URL are plain <div> (not links) so
               clicking never jumps to "#". A version with one label = a changelog entry. -->
          <div class="version-pop" id="versionPop" hidden>
            <div class="vh">Plan versions</div>
            <div class="vrow current"><span class="vn">v3</span> current · <span class="vlabel">what changed</span> <span class="vd">2026-06-21</span></div>
            <a class="vrow" href="https://prior-v2-deploy.example.workers.dev"><span class="vn">v2</span> · <span class="vlabel">label</span> <span class="vd">2026-06-20</span></a>
            <div class="vrow"><span class="vn">v1</span> · <span class="vlabel">label (no separate deploy)</span> <span class="vd">2026-06-19</span></div>
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
  // Fixed-viewport pan/zoom so long diagrams stay readable (drag to pan, wheel/buttons to zoom, fullscreen).
  function setupPanZoom(el) {
    const svg = el.querySelector('svg'); if (!svg) return;
    svg.style.maxWidth = 'none';
    const vb = svg.viewBox && svg.viewBox.baseVal;
    const natW = (vb && vb.width) || svg.getBoundingClientRect().width || 800;
    const natH = (vb && vb.height) || svg.getBoundingClientRect().height || 400;
    svg.setAttribute('width', natW); svg.setAttribute('height', natH); svg.style.height = natH + 'px';
    const st = { s: 1, x: 0, y: 0 };
    const apply = () => { svg.style.transform = `translate(${st.x}px,${st.y}px) scale(${st.s})`; };
    const fit = () => { const vw = el.clientWidth, vh = el.clientHeight; st.s = Math.min(vw / natW, vh / natH) * 0.95; st.x = (vw - natW * st.s) / 2; st.y = (vh - natH * st.s) / 2; apply(); };
    const zoom = (f, cx, cy) => { const ns = Math.max(0.15, Math.min(8, st.s * f)); cx = cx ?? el.clientWidth / 2; cy = cy ?? el.clientHeight / 2; st.x = cx - (cx - st.x) * (ns / st.s); st.y = cy - (cy - st.y) * (ns / st.s); st.s = ns; apply(); };
    let drag = null;
    svg.addEventListener('mousedown', e => { drag = { x: e.clientX, y: e.clientY, ox: st.x, oy: st.y }; el.classList.add('grab'); e.preventDefault(); });
    window.addEventListener('mousemove', e => { if (!drag) return; st.x = drag.ox + (e.clientX - drag.x); st.y = drag.oy + (e.clientY - drag.y); apply(); });
    window.addEventListener('mouseup', () => { drag = null; el.classList.remove('grab'); });
    el.addEventListener('wheel', e => { e.preventDefault(); const r = el.getBoundingClientRect(); zoom(e.deltaY < 0 ? 1.12 : 0.89, e.clientX - r.left, e.clientY - r.top); }, { passive: false });
    const bar = document.createElement('div'); bar.className = 'mz-bar';
    bar.innerHTML = '<button data-z="out" title="Zoom out">−</button><button data-z="in" title="Zoom in">+</button><button data-z="fit" title="Fit" style="font-size:11px">Fit</button><button data-z="fs" title="Fullscreen"><svg class="icon"><use href="#i-expand"/></svg></button>';
    const hint = document.createElement('div'); hint.className = 'mz-hint'; hint.textContent = 'drag to pan · scroll to zoom';
    el.appendChild(bar); el.appendChild(hint);
    bar.querySelector('[data-z=in]').onclick = () => zoom(1.25);
    bar.querySelector('[data-z=out]').onclick = () => zoom(0.8);
    bar.querySelector('[data-z=fit]').onclick = fit;
    bar.querySelector('[data-z=fs]').onclick = () => { el.classList.toggle('fs'); setTimeout(fit, 60); };
    fit();
  }
  document.addEventListener('keydown', e => { if (e.key === 'Escape') document.querySelectorAll('.mermaid.fs').forEach(m => m.classList.remove('fs')); });
  try {
    const mermaid = (await import('https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs')).default;
    let seq = 0;
    async function draw() {
      const dark = document.documentElement.getAttribute('data-theme') !== 'light';
      mermaid.initialize({
        startOnLoad: false, securityLevel: 'loose', suppressErrorRendering: true,
        theme: dark ? 'dark' : 'default',
        themeVariables: {
          primaryColor: dark ? '#2e2e2e' : '#ffffff', primaryTextColor: dark ? '#fafafa' : '#232323',
          primaryBorderColor: dark ? '#555' : '#bbb', lineColor: dark ? '#9e9e9e' : '#777',
          fontFamily: 'Inter,-apple-system,BlinkMacSystemFont,Segoe UI,Roboto,sans-serif'
        }
      });
      for (let i = 0; i < blocks.length; i++) {
        const el = blocks[i];
        try {
          const { svg } = await mermaid.render('vpm-' + (seq++), src[i]);
          el.classList.remove('mermaid-fallback'); el.classList.add('done'); el.innerHTML = svg;
          setupPanZoom(el);
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
    // never let a placeholder / dead link ("#", unset) navigate or jump to top
    vPop.querySelectorAll('a.vrow').forEach(function (a) {
      var href = a.getAttribute('href') || '';
      if (!href || href === '#' || /URL$/.test(href) || /example\./.test(href)) {
        var div = document.createElement('div'); div.className = a.className; div.innerHTML = a.innerHTML; a.replaceWith(div);
      } else { a.target = '_blank'; a.rel = 'noopener'; }
    });
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
  In the sprite: socials (`b-whatsapp`/`telegram`/`signal`/`discord`/`slack`/
  `instagram`/`messenger`/`x`/`linkedin`) + cloud/AI (`b-cloudflare` `#F38020`,
  `b-vercel` cc, `b-openai` cc, `b-anthropic` `#D97757`, `b-gemini` `#1BA1E3`,
  `b-perplexity` `#20808D`, `b-supabase` `#3FCF8E`, `b-googlecloud` `#4285F4`,
  `b-google` `#4285F4`, `b-github` cc). **Never use one brand's logo for another**
  (Gemini ≠ Google "G"). For a node with no real logo (Cron, KV cache, export…),
  use `#i-cpu` or no icon — never approximate. Add a logo only by copying its exact
  Simple Icons path.
- **Mermaid diagrams get a fixed viewport + pan/zoom automatically** (drag, scroll,
  −/+/Fit/fullscreen) — author the `<pre class="mermaid">` and the plumbing handles
  sizing; never hand-size diagrams.
- **Versions:** a timeline row links only if that version has its own deployed URL;
  current/undeployed versions are plain `<div>` (no `href="#"`).
- **Reuse the dock, compose/pin, shortcut, and annotation plumbing verbatim** —
  change content, palette tokens, and diagram/tree contents only.
