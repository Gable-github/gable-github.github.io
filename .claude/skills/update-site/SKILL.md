---
name: update-site
description: Make and ship changes to gabryelsoh.com, the Gatsby v3 personal portfolio in this repo. Covers where each piece of content lives, how to verify changes that an ordinary build check cannot see, the deploy procedure, and the traps that silently break the site. Use for any edit here (copy, jobs, projects, resume PDF, styling) and always before deploying.
---

# Updating gabryelsoh.com

Gatsby v3 static site, served from GitHub Pages at `gabryelsoh.com`.
Repo: `Gable-github/gable-github.github.io`.

## Read this first — three traps

### 1. Pushing to `main` does NOT update the live site

The site is served from the **`gh-pages`** branch, built from `public/`. Nothing a
visitor sees changes when you push source to `main`.

Publishing requires `npm run deploy` (= `gatsby build && gh-pages -d public`).

Push and deploy are independent. A change is only done when **both** have happened.
Do not tell the user a change is live until you have fetched the live URL and
confirmed it.

### 2. `static/CNAME` is load-bearing — never delete it

`gh-pages` defaults to `remove: "."`, meaning it **wipes the branch** before copying
`public/`. The custom domain depends on a `CNAME` file surviving in the deployed
output, and `gatsby build` only emits one if it exists in `static/`.

`static/CNAME` contains `gabryelsoh.com` with **no trailing newline**. If it is ever
removed, the next deploy silently drops the custom domain.

Confirm after every deploy:

```bash
git fetch origin gh-pages --quiet && git show origin/gh-pages:CNAME
```

### 3. You cannot verify page copy by grepping `public/index.html`

The homepage is **client-rendered**: `public/index.html` has an essentially empty
`<body>` (just a skip-link). Grepping it for site text returns nothing for new _and_
old strings alike, so a real failure and a real success look identical. Use the
method under "Verifying" instead.

## Where content lives

| What                               | Where                                                             |
| ---------------------------------- | ----------------------------------------------------------------- |
| Hero, About, Contact copy          | `src/components/sections/{hero,about,contact}.js` — hardcoded JSX |
| Work experience                    | `content/jobs/<Company>/index.md`                                 |
| Projects list                      | `content/projects/*.md`                                           |
| Featured projects (homepage cards) | `content/featured/*/index.md`                                     |
| Blog posts                         | `content/posts/*/index.md`                                        |
| Resume PDF                         | `static/gabryel_resume.pdf`                                       |
| Email, socials, nav links, colors  | `src/config.js`                                                   |

**The resume filename is load-bearing.** `/gabryel_resume.pdf` is linked from
`src/components/nav.js`, `src/components/menu.js`, and
`src/components/sections/hero.js`. Overwrite the file in place; renaming it 404s all
three links. Re-check with:

```bash
grep -rn "resume" src/ --include="*.js"
```

## Procedure

1. Node 18 required. If deps are missing: `npm install --legacy-peer-deps`.
2. Branch first — the repo uses feature branches merged to `main`.
3. Make edits.
4. `npx gatsby clean && npm run build` — see below for why `clean` matters.
5. Verify locally (next section).
6. Commit. Husky runs lint-staged (prettier + eslint) automatically.
7. Push the branch, and merge to `main` if that is what the user wants.
8. `npm run deploy` — this is the step that makes it live.
9. Verify live, then report.

## Verifying

**Always run `npx gatsby clean` before building.** `gatsby build` does not clear
`public/`, so chunks from previous builds linger. Stale chunks both ship dead code
and make grep results ambiguous — an old chunk containing the string you just
removed will look like your edit failed.

After a clean build, any hit in `public/` is live:

```bash
# hardcoded copy (hero / about / contact)
grep -rl "new string you added" public/*.js
```

Markdown-driven content lands in static-query JSON, not the JS bundle. The filename
hashes change whenever a GraphQL query changes, so discover them rather than
hardcoding:

```bash
for f in public/page-data/sq/d/*.json; do
  echo -n "$(basename $f): "
  python3 -c "import json,io;print(list(json.load(io.open('$f',encoding='utf-8'),strict=False).get('data',{}).keys()))"
done
```

Then read the relevant one (`jobs`, `projects`, `featured`, `site`). Use
`strict=False` — the rendered HTML contains literal newlines that trip strict JSON
parsing.

**Check both directions**: that new strings are present _and_ that the strings they
replaced are gone. Stale copy surviving a "successful" edit is the failure mode this
catches.

If you must work without cleaning, find which chunk is actually live by pulling the
referenced filename out of `public/index.html` and grepping only that file.

### Live verification

```bash
# resume PDF — compare against the file you deployed
curl -sL "https://gabryelsoh.com/gabryel_resume.pdf?cb=$(date +%s)" | md5 -q

# page copy — resolve the live chunk from the live HTML, then grep it
CB=$(date +%s)
CH=$(curl -sL "https://gabryelsoh.com/?cb=$CB" | grep -o '[0-9a-f]\{40\}-[0-9a-f]\{20\}\.js' | head -1)
curl -sL "https://gabryelsoh.com/$CH" | grep -c "new string"
```

GitHub Pages takes roughly 15–60s to propagate. Poll in a backgrounded `until` loop
rather than concluding the deploy failed on the first miss. Cache-bust with a query
param.

## Keeping the resume in sync

Source of truth:
`~/Documents/Job application documents/Resumes/Current/GabryelSoh_Resume.pdf`

Check the MD5 against `static/gabryel_resume.pdf` before assuming it needs updating.

When syncing, hunt specifically for **forward-looking claims that have gone stale** —
they never announce themselves:

- `Incoming <Company>` job titles, and placeholder bullets for roles not yet started
- Availability windows in `contact.js` (and the resume's own header line)
- Class year in `hero.js` — "junior" / "senior" drift against the graduation date
- Any "joining X next" phrasing in `hero.js` / `about.js`

Grep for the old strings across `src/` and `content/` rather than trusting a
section-by-section read.

## House style

The copy is deliberately casual and first-person ("I also like hooping, even though
my knees are lowk bad"). It is not filler to be cleaned up. Keep edits surgical —
fix tense and facts, preserve the voice. Do not rewrite sections that are merely
informal.

Prefer showing the user a flagged uncertainty over silently changing a personal
fact (job locations, titles, dates). The resume is the source of truth for facts,
but the user is the source of truth for things the resume abbreviates.

## Environment gotchas

- **zsh eats unquoted glob flags.** `grep -rn "x" src/ --include=*.js` fails with
  `no matches found`. Quote it: `--include="*.js"`. A silently-failed grep reads as
  "no results", which is worse than an error.
- macOS has no `timeout` command by default.
- Builds take ~10–20s; `npm run deploy` prints `Published` on success.
