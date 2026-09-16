# hannahhokkim.site

Quarto website, deployed to GitHub Pages. Edit Markdown, push, done.

```
_quarto.yml      site config: nav, theme, fonts
styles.scss      all the styling (palette + type at the top)
index.qmd        home
about.qmd        about
research.qmd     research program + the three-strand figure
publications.qmd full publication list
cv.qmd           CV page (PDF lives in files/)
files/           downloadable PDFs
images/          images
.github/workflows/publish.yml   renders + deploys on every push to main
CNAME            the custom domain — do not delete
```

---

## Step 0 — move this folder somewhere sensible

It currently sits inside `Documents/ChatGPT/Jobs`, which is itself a git repo. Drag the whole
`hannahhokkim-site` folder somewhere standalone first — `~/Documents/hannahhokkim-site` is fine.
Nothing inside it cares where it lives.

## Step 0.5 — one cleanup command

I built this in a sandbox that isn't allowed to delete files inside your folders, so git left some
junk behind: a `.git/stale-locks/` folder and some `tmp_obj_*` files under `.git/objects/`. They're
harmless, but git will complain until they're gone. Once the folder is moved (Step 0), run:

```bash
rm -rf .git/stale-locks
git gc --prune=now
git status
```

`git status` should print "nothing to commit, working tree clean". If any git command ever says
*"Another git process seems to be running"*, it's a leftover lock — `rm -f .git/*.lock
.git/index.lock` and try again.

## Step 1 — install Quarto (for local preview)

Download from <https://quarto.org/docs/get-started/>. If you have RStudio, you already have it.

Then, in a Terminal inside the folder:

```bash
quarto preview
```

That opens the site in your browser and live-reloads as you save. This is the whole editing loop.
You do **not** need Quarto installed for the site to publish — GitHub does the rendering — but
previewing locally is much nicer than pushing and waiting.

## Step 2 — create the GitHub repo

Make a GitHub account if you don't have one. Then create a **new, empty, public** repository named
`hannahhokkim-site`. Don't add a README or .gitignore — this folder already has them.

Public is required for GitHub Pages on a free plan. For an academic site that is normal, and
frankly on-brand for your open science work. If you'd rather keep it private, Cloudflare Pages
hosts private repos free and the rest of these steps are nearly identical — tell me and I'll
rewrite this section.

## Step 3 — push

The repo is already initialised with one commit, so you only need to connect it and push. In
Terminal, inside the folder:

```bash
git gc --prune=now          # clears temp files left by the sandbox I built this in
git remote add origin https://github.com/YOUR-USERNAME/hannahhokkim-site.git
git push -u origin main
```

GitHub will ask you to sign in. Use a personal access token, not your password — GitHub walks you
through it, or install the `gh` CLI and run `gh auth login` first, which handles it for you.

If you'd rather start the history yourself, `rm -rf .git` first and then `git init -b main`,
`git add .`, `git commit -m "initial commit"` before the two commands above.

## Step 4 — turn on Pages

In the repo on github.com: **Settings → Pages**.

- **Source:** "GitHub Actions" (not "Deploy from a branch")

That's it. Go to the **Actions** tab and you should see "publish site" running. It takes about a
minute. When it's green, your site is live at `https://YOUR-USERNAME.github.io/hannahhokkim-site/`.

**Check that it looks right before touching DNS.** Nothing about your live site has changed yet.

## Step 5 — point the domain at GitHub

Still in **Settings → Pages**, under "Custom domain", enter `hannahhokkim.site` and save.

Then in Hostinger: **hPanel → Domains → DNS / Nameservers → DNS records**.

Delete the existing `A` record(s) for `@` that point at Hostinger, and add these four:

| Type | Name | Points to        |
|------|------|------------------|
| A    | @    | 185.199.108.153  |
| A    | @    | 185.199.109.153  |
| A    | @    | 185.199.110.153  |
| A    | @    | 185.199.111.153  |

Optionally also add these four `AAAA` records on `@` for IPv6:
`2606:50c0:8000::153`, `2606:50c0:8001::153`, `2606:50c0:8002::153`, `2606:50c0:8003::153`

And one `CNAME` so the www version works:

| Type  | Name | Points to                |
|-------|------|--------------------------|
| CNAME | www  | YOUR-USERNAME.github.io  |

DNS takes anywhere from a few minutes to a few hours. Once GitHub sees it, go back to
**Settings → Pages** and tick **Enforce HTTPS**. The certificate is issued automatically and free.

## Step 6 — after it's live

Leave the old builder site alone for a week in case you want to look something up. Then you can
delete it, and drop the Hostinger hosting plan down to domain-only registration — the hosting
itself is no longer doing anything.

---

## Editing from here on

1. Open a `.qmd` file in any text editor. It's Markdown.
2. `quarto preview` to watch it as you type.
3. When happy:

```bash
git add .
git commit -m "what changed"
git push
```

Ninety seconds later it's live. Every version is kept, so nothing you do is unrecoverable.

### Common edits

- **Add a publication:** add a bullet to `publications.qmd`. Copy the shape of the one above it.
- **Change colors or fonts:** the variables at the top of `styles.scss` — `$pine`, `$paper`,
  `$ink`, and the `$font-family-*` lines. Everything else derives from them.
- **New CV:** replace `files/hannah-hok-kim-cv.pdf`, keeping the filename.
- **Add a page:** create `newpage.qmd` with a `title:` at the top, then add it to the `navbar`
  list in `_quarto.yml`.

### If a push doesn't show up

Check the **Actions** tab. A red X means the render failed, and the log says which file and line.
Nothing breaks on the live site when a build fails — the previous version stays up.

## Notes on what I set up

- The research page's three-strand figure is hand-built HTML and CSS living in `research.qmd`
  and `styles.scss`, not an image. It stays sharp at any size and reflows to a vertical timeline
  on phones. `images/research-program.png` is your original figure, kept as a spare.
- The site is light-mode only, matching your old one. A dark mode is a small addition if you want.
- `about.qmd` carries your existing text verbatim, including three places where the old site had
  link text but no working link ("view their 3D scans of objects here", the Cluny exhibit, and the
  Bernward Column scan). Those need real URLs — they were dead on the old site too.
- `_site/` is the rendered output. It's gitignored; open `_site/index.html` to preview without
  Quarto installed.
