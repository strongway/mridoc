# Published Slidev decks (built output — do not edit here)

These `session-1/` and `session-2/` folders are **pre-built Slidev decks**, served by
Quartz at `mri.msense.de/static/slides/session-N/`. **Do not edit the HTML/JS here** —
it is generated. The source lives outside this repo, in the tutorial workspace at
`sessions/session-N/slides/slides.md`.

## Updating a deck (the rule that keeps biting us)

Editing the source `slides.md` does **not** update the site. After editing, you must
rebuild and copy the output back into this folder, then commit + push this repo:

```bash
cd <workspace>/sessions/session-2/slides
npm run build                                  # Slidev 0.50.0, base = /  -> dist/
DEST=<workspace>/tutorial-notes/mridoc/static/slides/session-2
rm -rf "$DEST" && cp -r dist "$DEST"
cd <workspace>/tutorial-notes/mridoc
git add static && git commit -m "Rebuild Session-2 deck" && git push origin main
```

## Pitfalls

- **Rebuild before pushing.** Pushing after a `slides.md` edit *without* rebuilding
  publishes a stale deck (this has happened). Verify with
  `grep -rl "a new phrase from your edit" session-2 | head`.
- **`static/` is untracked by default** — remember `git add static`.
- **Slidev must stay on `0.50.0`.** v0.52 doubles the base path → 404 past slide 1.
- **Base is `/`**; keep the generated `_redirects` (`/*  /index.html  200`) for SPA routing.
- **Referenced `./figures/*.png` must exist at build time** — regenerate figures first,
  or the image 404s on the live site.

Full per-session instructions: `sessions/session-N/slides/README.md`.
