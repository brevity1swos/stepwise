# stepwise — landing site

Static HTML landing page introducing the `rgx` / `agx` / `sift` suite.

No build step. No framework. Plain HTML + CSS.

## Preview locally

```sh
cd stepwise
python3 -m http.server 8080
# open http://localhost:8080
```

Or with any other static server — `npx serve`, `caddy file-server`, etc.

## Assets

- `assets/rgx-demo.gif` — copied from `../regex_101/assets/demo.gif`. Regenerate the source with `vhs ../regex_101/assets/demo.tape`, then re-copy.
- `assets/agx-demo.gif` — copied from `../agx/assets/demo.gif`. Regenerate from the agx repo's `assets/demo.tape`, then re-copy.
- sift has no GIF yet; the tool card renders a code-preview block instead.

## Going public

The site is kept private for now. When ready to publish:

1. Create a new public repository `brevity1swos/stepwise` on GitHub.
2. Push this directory to `main`.
3. In repo **Settings → Pages**, set source to **Deploy from a branch** → `main` → `/ (root)`.
4. Wait for the first build; site will be live at `https://brevity1swos.github.io/stepwise/`.
5. (Optional) Add a `CNAME` file and configure a custom domain.

No `.nojekyll` needed — there are no underscore-prefixed paths.

## Content notes

- Tone: dry, technical, utility-focused.
- No mention of "AI-driven development" philosophy or internal framing — that stays out of public-facing copy until the user decides otherwise.
- Each tool's bullets are sourced from the respective repo's `README.md` (see `rgx/README.md`, `agx/README.md`, `sift/README.md`).
- Install commands: `rgx` is on crates.io; `agx` and `sift` repos are **not yet public** — install commands on the landing page are placeholders until the repos flip public.

## Editing

- `index.html` — structure and copy.
- `style.css` — typography, spacing, colors. Single file, scoped to the page.
- Keep the page single-file. If it grows past ~400 lines of HTML, consider splitting sections into partials via SSG (Jekyll / Eleventy) — but not before.
