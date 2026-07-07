# Published Slidev decks (built output — do not edit here)

`session-1/` and `session-2/` are **pre-built Slidev decks**, served by Quartz at
`mri.msense.de/static/slides/session-N/`. **Do not edit the HTML/JS here** — it is
generated. The source lives outside this repo, in the tutorial workspace at
`sessions/session-N/slides/slides.md`.

> [!important] This is `quartz/static/`, the folder Quartz actually deploys.
> Quartz's static emitter (`quartz/plugins/emitters/static.ts`) copies **`quartz/static/`**
> to the site. A `static/` folder at the **repo root is silently ignored** — decks copied
> there will never appear online. (That mistake once made the live slides look "stuck".)

## Updating a deck

Editing the source `slides.md` does **not** update the site. Rebuild and copy the output
into this folder, then commit + push:

```bash
cd <workspace>/sessions/session-2/slides
# --base MUST be the served subpath, or the deck renders blank (assets 404 at site root)
npx slidev build --base /static/slides/session-2/ --out dist slides.md
DEST=<workspace>/tutorial-notes/mridoc/quartz/static/slides/session-2
rm -rf "$DEST" && cp -r dist "$DEST"
cd <workspace>/tutorial-notes/mridoc
git add quartz/static && git commit -m "Rebuild Session-2 deck" && git push origin main
```

Pushing to `main` triggers the GitHub Pages deploy (`.github/workflows/deploy.yml`,
`npx quartz build`). Verify after it finishes, e.g.
`curl -s https://mri.msense.de/static/slides/session-2/index.html | grep -o 'index-[A-Za-z0-9_-]*\.js'`
should show the new bundle hash.

## Pitfalls

- **Build with `--base /static/slides/session-N/`** — the deck's served subpath. Default
  base `/` makes the HTML reference `/assets/...`, which 404s at the site root → **blank deck**.
- **Deploy dir is `quartz/static/`**, not repo-root `static/` (see above).
- **Rebuild before pushing** — editing `slides.md` alone publishes nothing new.
- **Stay on Slidev `0.50.0`** — v0.52 doubles the base path → 404 past slide 1.
- Keep the generated `_redirects` (`/*  /index.html  200`) for SPA routing.
- **Referenced `./figures/*.png` must exist at build time** — regenerate figures first,
  or the image 404s on the live site.
- **Self-contained `.html` viewers** (e.g. nilearn `view_img`) go in `quartz/static/media/`,
  **not** `content/media/` — a `.html` under `content/` is emitted as an extension-less
  page and served as `application/octet-stream`, which browsers download.
