# stepwise — landing site

Static HTML landing page for **stepwise** — terminal-native tooling for the
AI-agent session lifecycle: **ccr** finds the session, **agx** walks through
what happened, **sift** snapshots what the agent did in each turn.

**Live:** <https://brevity1swos.github.io/stepwise/>

No build step. No framework. Plain HTML + CSS.

## Preview locally

```sh
cd stepwise
python3 -m http.server 8080
# open http://localhost:8080
```

Or with any other static server — `npx serve`, `caddy file-server`, etc.

## Deployment

The site is published with **GitHub Pages**, served from `main` at `/ (root)`.
Pushing to `main` triggers a Pages rebuild; the live site updates within a
minute or two at <https://brevity1swos.github.io/stepwise/>.

To change the Pages config: repo **Settings → Pages** (source =
*Deploy from a branch* → `main` → `/ (root)`). No `.nojekyll` is needed —
there are no underscore-prefixed paths. To attach a custom domain, add a
`CNAME` file and configure it under Settings → Pages.

## Assets

All three tool cards use real demo GIFs:

- `assets/ccr-demo.gif`, `assets/agx-demo.gif`, `assets/sift-demo.gif` — copied
  from each tool repo's `assets/demo.gif`. Regenerate the source with `vhs`
  (each repo has an `assets/demo.tape`), then re-copy.

## Content notes

- Tone: dry, technical, utility-focused.
- No mention of "AI-driven development" philosophy or internal framing — that
  stays out of public-facing copy.
- Each tool's bullets are sourced from the respective repo's `README.md`
  (`ccr/README.md`, `agx/README.md`, `sift/README.md`). Keep ship-status
  claims honest — current gaps are tracked in `docs/gaps.md`.
- Install / status, current as of this revision:
  - **ccr** — `cargo install ccr` (crates.io, v0.2.x). Nicknames + bookmarks,
    live-session detection, scriptable CLI (`list` / `path` / `show` /
    `export --format json` / `stats`). Never modifies session files.
  - **agx** — `cargo install agx-tui` (crates.io, v0.2.x; the binary is `agx`).
  - **sift** — public repo, **not** on crates.io (the `sift` crate name is taken
    by an unrelated project); install from source. Workspace at v0.1.0.
- `rgx` (terminal regex debugger) is referenced only as a "Related" footer
  link — it is not part of the stepwise session-lifecycle suite.

## Editing

- `index.html` — structure and copy.
- `style.css` — typography, spacing, colors. Single file, scoped to the page.
- Keep the page single-file. If it grows past ~400 lines of HTML, consider
  splitting sections into partials via an SSG (Jekyll / Eleventy) — but not
  before.
