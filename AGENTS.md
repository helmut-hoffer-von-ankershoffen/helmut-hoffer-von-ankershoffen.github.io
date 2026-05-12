# AGENTS.md — helmut.hoffer-von-ankershoffen.me

Operator guide for AI agents (and humans) editing this repo. Match Helmut's tone: terse, factual, no decoration.

## What this is

Personal bio site at `https://helmut.hoffer-von-ankershoffen.me`. Single page (one in EN, one in DE), no JS, no build step. Hosted on **GitHub Pages**; pushing to `main` deploys within ~1 min via the `pages-build-deployment` workflow.

The CNAME pin is `helmut.hoffer-von-ankershoffen.me` (apex of a sub-domain). The `www` apex of `helmguild.com` is unrelated — different repo.

## File layout

```
/                                  # EN (default)
  index.html                       # the page
  CNAME                            # pin to helmut.hoffer-von-ankershoffen.me
  README.md                        # one-liner only
  AGENTS.md                        # this file
  assets/
    css/
      me_11.css                    # ACTIVE stylesheet — edit this one
      me.css, me_2.css … me_10.css # historical variants — do NOT edit
    images/
      background.jpg               # 2880×1900 hero bg (B&W, faded into black left)
      helmut-hoffer-von-ankershoffen-ne-oertel-head.jpg  # 1405×1405 avatar
      helmut-hoffer-von-ankershoffen-ne-oertel-card.jpg  # social card (legacy)
      helmut-hoffer-von-ankershoffen-ne-oertel-social-card*.jpg  # OG/Twitter
      signature.png                # Helmut's signature mark
      profile_en.png, avatar-half.png  # legacy assets
  favicon*, apple-touch-icon.png, android-chrome-*.png, mstile-*.png
  site.webmanifest, browserconfig.xml, robots.txt, sitemap.xml
  40x.html, 50x.html               # error pages

/de/                               # DE mirror — same single-page structure
  index.html
```

The CSS is named after a legacy WordPress theme (`me_*.css`, body class `page-template-page-fullsingle-me`). Versioned variants exist for historical reasons; only **`me_11.css`** is actually loaded by `index.html` (and `de/index.html` via `../assets/css/me_11.css`). Don't bump the version unless the change is non-trivial — just edit `me_11.css` in place.

## Bilingual rule (load-bearing)

**Every visitor-facing edit must land in both `index.html` and `de/index.html`.** A single-language change is incomplete and wrong.

This includes:
- Bio paragraph copy
- `<meta name="description">`, `og:description`, `twitter:description`
- `og:title`, `twitter:title` (usually identical, but check)
- Page title text
- JSON-LD blocks
- `og:image` / `twitter:image` references (the DE page uses `_de.jpg` social cards, the EN page does not)

Each page declares its alternate explicitly:

```html
<!-- in /index.html -->
<link rel="alternate" hreflang="de" href="https://helmut.hoffer-von-ankershoffen.me/de/"/>
<!-- in /de/index.html -->
<link rel="alternate" hreflang="en" href="https://helmut.hoffer-von-ankershoffen.me"/>
```

## Bio copy — sync rules

When Helmut rewrites a bio one-liner (in chat, in email, on another surface), it overrides everything older. Sync to **all** copies in both languages immediately, including meta/OG/Twitter blocks. Don't preserve older phrasings "for variety" — they read as drift.

Snowflake role on public surfaces: **"Managing streaming at Snowflake Inc."** (or DE: "Managing streaming bei Snowflake Inc."). Lowercase `streaming` — it's a discipline, not a product name. Team scope and product framing are confidential and must not appear here.

DE Ironman verb: **"trete beim Ironman an"** (compete in), not "finishe" (false-friend). Same applies to other Anglicism traps — favour idiomatic German.

## Portrait pipeline (load-bearing — read before regenerating images)

The hero `background.jpg` and the avatar `head.jpg` are derived from a single source photo. Two traps caught in May 2026:

1. **Source of truth**: `sites/helmguild.com/helmut-hoffer-von-ankershoffen/assets/portrait.jpg` (800×800 colour, transparent acetate glasses). **Do NOT use** `state/content-source/helmut-snowflake-profile.jpg` — that's the older dark-rim photo and produced a wrong "transparent-glasses" replacement on the first attempt.
2. **Output dimensions are load-bearing**: keep `head.jpg` at **1405×1405** and `background.jpg` at **2880×1900**. The CSS positions `background.jpg` with `background-size: cover`, but the aspect ratio matters — non-square `background.jpg` won't compose right.

### Pipeline

```
1. open(<source>.jpg).convert("RGB")
2. ImageOps.mirror(...)           # head 'comes from the right', gaze toward the text column
3. ImageOps.grayscale(...)        # true B&W via luminance, then back to RGB for JPEG
4a. head.jpg  (1405×1405)         # Lanczos-resize to 1405 height, centre-crop with a
                                  # ~5% upward shift so the eyes sit on the upper third.
4b. background.jpg (2880×1900)    # Lanczos-resize to 1900×1900 (square portrait at full
                                  # canvas height); paste onto a black 2880×1900 canvas
                                  # at paste_x = 1320 (~46% from the left); apply a
                                  # 520-px cosine-eased horizontal alpha mask so the
                                  # left edge of the portrait fades smoothly into black.
5. JPEG quality=88, optimize=True, progressive=True
```

The `paste_x` and `fade_w` values are tuned so the bio text column (which sits over the left ~45% of the canvas) doesn't collide with the face. If you change the text layout, re-tune these together.

CSS reference: `assets/css/me_11.css` line 210 sets `body.page-template-page-fullsingle-me { background-image: url("../images/background.jpg"); ... }`. The `<img>` at `index.html:71` references `head.jpg`.

### Cache after asset changes

GitHub Pages serves with `cache-control: max-age=600`. After pushing image changes:
- Verify deploy: `gh run watch <id>` on the `pages-build-deployment` run.
- Verify the live MD5 matches local: `curl -s "https://helmut.hoffer-von-ankershoffen.me/assets/images/<file>?v=$(date +%s)" | md5`.
- Tell Helmut to hard-refresh (cmd+shift+R) — visitor browser caches will linger up to 10 min otherwise.

## Commit identity

Per workspace policy, commits to this repo must be authored as Helmut, not the OpenClaw VM identity. Use the `-c` flag per-invocation rather than editing config:

```sh
git -c user.email=helmuthva@gmail.com -c user.name='Helmut Hoffer von Ankershoffen' commit -m "..."
```

## What lives elsewhere

- The **bio paragraph copy on `helmguild.com/helmut-hoffer-von-ankershoffen/`** is a separate, longer profile (founder narrative, career glance, etc.) — different repo, different audience, and intentionally not in lockstep with this single-page bio.
- The **canonical transparent-glasses portrait** is in the `helmguild.com` repo, not here. This repo only stores the derived B&W composites.
- **AMMP, helmguild thesis, Pepe Arturo** content all lives in `helmguild.com` — never link or quote from there here unless Helmut explicitly asks for crossover.
