# Site Improvement Plan — saikeo.github.io

_Analysis date: 2026-11-08 · Target: Jekyll blog hosted on GitHub Pages_

This document reviews the repository as it currently stands and proposes
concrete improvements. It focuses on the **site engineering** (build config,
templates, SEO, accessibility, performance, and repo hygiene), not on the
technical subject matter of the posts.

Items are grouped by theme and each is tagged with a rough priority:

- **P1** — broken / user-visible now, fix first
- **P2** — quality, SEO, maintainability
- **P3** — nice-to-have / polish

---

## 1. Broken or non-functional build configuration (P1)

### 1.1 Pagination is referenced but never configured
`index.html` loops over `paginator.posts`:

```liquid
{% for post in paginator.posts %}
```

But `_config.yml` does **not** load `jekyll-paginate` and defines no
`paginate:` / `paginate_path:` settings. On GitHub Pages `paginator` is
therefore `nil`, so **the loop renders nothing**. Meanwhile `index.md` (layout
`home`) also exists and lists posts a second way.

**Action:** pick one home page.
- Recommended: delete `index.html`, keep `index.md` + `_layouts/home.html`
  (it already renders the full post list and works without a plugin).
- Or, if you want paging, add `jekyll-paginate` to `plugins:` and set
  `paginate: 10` / `paginate_path: "/page:num/"`, then remove `index.md`.

Having two files that both compile to `/index.html` also causes a Jekyll
"output overwrites" conflict — only one wins, non-deterministically.

### 1.2 `site.url` / `site.baseurl` are not set
`atom.xml` builds every link from `{{ site.url }}`:

```xml
<link>{{ site.url }}/{{ post.url }}</link>
```

With no `url:` in `_config.yml`, these resolve to empty strings, producing
feed links like `//2026/09/22/...`. **Action:** add to `_config.yml`:

```yaml
url: "https://saikeo.github.io"
baseurl: ""
```

### 1.3 Redundant, broken hand-rolled feed
`jekyll-feed` is already in `plugins:` and generates a correct `/feed.xml`.
The hand-written `atom.xml` duplicates it, is mislabelled (`rss version="2.0"`
in a file called atom), and references fields that don't exist:

- `site.author.name`, `site.author.email`, `site.author.url` — but `author`
  in `_config.yml` is the **string** `"Saikeo"`, not a map, so `.name` etc.
  are all empty.

**Action:** delete `atom.xml`. The footer/social `rss` link already points to
`/feed.xml`, which jekyll-feed provides. Confirm `jekyll-feed` picks up author
by converting `author:` to a mapping (see 2.1).

### 1.4 Google Analytics uses a dead property
`_includes/head.html` **and** `_layouts/default.html` both embed
`UA-144762480-1`. Universal Analytics (the `UA-` property type) stopped
processing data on **1 July 2023**, so no analytics are being collected. The
snippet is also duplicated (loaded twice per page).

**Action:** either remove the tracking entirely, or migrate to GA4
(`G-XXXXXXXXXX`) / a privacy-friendly alternative (Plausible, GoatCounter).
Keep it in **one** place — `head.html` — and drive it from a config value, e.g.
`plainwhite.analytics_id`, which `default.html` already checks for but which is
never set.

---

## 2. SEO & metadata (P2)

### 2.1 Author metadata is a string, not a map
`jekyll-seo-tag` and `jekyll-feed` both expect `author` to optionally be a
mapping. **Action:**

```yaml
author:
  name: Saikeo Kavhanxay
  email: kavhanxay@hotmail.com   # or a single consistent address — see 4.3
```

### 2.2 No sitemap
`plainwhite.sitemap: true` is set, but the `plainwhite` theme is commented out
(`# theme: plainwhite`), so that flag does nothing. **Action:** add
`jekyll-sitemap` to `plugins:` to emit `/sitemap.xml`, and reference it from
`robots.txt`.

### 2.3 `robots.txt` blocks a non-existent page
```
Disallow: /contact.html
```
There is no `contact.html` in the repo. **Action:** remove that line and add
`Sitemap: https://saikeo.github.io/sitemap.xml`.

### 2.4 Social meta / canonical
With `url` set (1.2) and `jekyll-seo-tag` already installed, add per-post
`image:` consistently and a site-wide fallback so link previews (Open Graph /
Twitter cards) render. Several posts set `image:`, most don't.

---

## 3. Templates, accessibility & performance (P2)

### 3.1 Social links render even when empty
`_layouts/default.html` always prints Facebook, GitHub, LinkedIn and Twitter
icons. `social_links.github` and `facebook` are blank in config, so those
anchors link to `https://github.com/` and `https://facebook.com/` — dead-ends.

**Action:** wrap each in a guard:

```liquid
{% if site.plainwhite.social_links.github %}
  <a href="https://github.com/{{ site.plainwhite.social_links.github }}">…</a>
{% endif %}
```

Also add `aria-label`s to the icon links (they contain no text, so screen
readers announce nothing).

### 3.2 Missing `alt` text on images
Many in-post images use empty alt (`![](…)`) or decorative `<img>` with no
`alt`. **Action:** add descriptive alt text; it helps accessibility and SEO.

### 3.3 All images are hotlinked from third parties (reliability + privacy)
Post bodies and every `thumbnail:` pull from imgur, random blogs, vendor CDNs,
Microsoft avatar URLs, pngtree, etc. Problems:
- **Link rot** — these will break over time (several likely already 404).
- **Mixed content** — at least one is `http://` (`awsmp-logos.s3…`,
  `i.imgur.com/ydjR8W9.png`), which browsers block on an HTTPS site.
- **Privacy/leakage** — readers' browsers hit all those third-party hosts.

**Action:** download the images you rely on into `assets/img/<post>/` and
reference them locally. At minimum, fix the `http://` ones to `https://`.

### 3.4 Unused include
`_includes/toc.html` (the jekyll-toc helper) is present but never `{% include %}`d
anywhere, even though `2019-12-17-Bug-Bounty-Cheetsheet.md` sets `toc: true`.
**Action:** either wire it into `_layouts/post.html` (render a TOC when
`page.toc`), or delete the include.

### 3.5 Copyright year is hard-coded
`default.html` prints `&copy; 2026`. **Action:** use
`&copy; {{ 'now' | date: "%Y" }}`.

---

## 4. Repository hygiene & content cleanup (P2 / P3)

### 4.1 Stray and duplicate files that should not ship
| File | Issue | Suggested action |
|------|-------|------------------|
| `newabout.md` | No front matter → renders as raw/unstyled; duplicates `about.md` | Delete; fold any wanted text into `about.md` |
| `2019-07-30-Generic-Cheetsheet.md` | A draft post sitting in repo **root**, not `_posts/`; `published: false` | Move to `_drafts/` or delete |
| `2018-01-06-Yet-Another-Sample-Page.md.old` | Leftover sample | Delete |
| `_unpublished/2019-08-14-Win32-Buffer-Overflow-SEH.md` | Older duplicate of a published post | Delete or move to `_drafts/` |
| `favicon1.ico`, `assets/portfolio1.png` | Unused `…1` duplicates | Delete |
| `assets/.DS_Store` | macOS cruft, committed | Delete; add to `.gitignore` |
| `.idea/` (10 files incl. `workspace.xml`) | IDE config committed; `workspace.xml` churns every session (it's the only dirty file right now) | `git rm -r --cached .idea` and add to `.gitignore` |

Suggested `.gitignore` additions:
```
.DS_Store
.idea/
_site/
.sass-cache/
.jekyll-cache/
```

### 4.2 Typos in titles / tags (user-visible)
- `Cheetsheet` → **Cheatsheet** (filename + title of the Linux notes post; the
  Bug Bounty post also has it in the filename).
- `Teraaform` → **Terraform** (title of the CentOS post).
- `Initail` → **Initial**, `Secuirty` → **Security** (Juniper vSRX post).
- `Overfow` → **Overflow**, `Buffer Overfow` tag (ROP post + EIP post tags).
- `2020-02-26-IPSec-Site-Site-With-Overlapping Subnet.md` — **space in
  filename** produces an ugly, fragile URL. Rename with a hyphen.

Note: renaming a published post changes its URL. If SEO/history matters, keep
the old slug via `permalink:` front matter or a redirect.

### 4.3 Inconsistent identity / contact details
- `about.md` says based in **Laos**; `newabout.md` says **Scotland**.
- Config `email: kavhanxay@hotmail.com`; the git context shows
  `phailin.psp@gmail.com`; posts reference the hotmail address.
- Taglines are split across `tagline` / `tagline1` inconsistently.

**Action:** settle on one bio, one location, one contact address, and update
`about.md` + `_config.yml` together. Delete `newabout.md`.

### 4.4 Tag casing / formatting is inconsistent
Tags mix styles: `Buffer Overfow` vs `Buffer Overflow`, `Secuirty`,
`2FA Bypass`, `OpenSUSE_official`, and one post (`2019-07-30-Generic…`) has
malformed front matter `tags:[...]` (no space after the colon, so it won't
parse as YAML). **Action:** normalise a small controlled tag vocabulary and fix
the malformed line.

### 4.5 Placeholder / unprofessional draft content
`_posts/2019-10-18-We-are-lets-drink-corp.md` has the tag `[Drinking,Beer,Fuck]`.
It's `published: false` so it won't appear, but it's still in the public repo.
**Action:** delete it or clean it up before it's ever flipped to published.

---

## 5. Suggested order of work

1. **P1 build fixes** (Section 1): pick one home page, set `url`/`baseurl`,
   delete `atom.xml`, fix/remove analytics. → Site actually builds correctly
   and the feed/home work.
2. **Repo hygiene** (4.1, add `.gitignore`): removes churn (`.idea/workspace.xml`)
   and cruft, making every later diff clean.
3. **SEO + templates** (Sections 2 & 3): author map, sitemap, robots, guarded
   social links, alt text, dynamic year.
4. **Content pass** (4.2–4.5): typos, filename with a space, identity
   consistency, tag normalisation, draft cleanup.
5. **Image localisation** (3.3): ongoing — migrate hotlinked images to
   `assets/img/` as you touch each post.

---

## 6. Quick wins (can be done in minutes)

- [ ] Set `url: "https://saikeo.github.io"` in `_config.yml`
- [ ] Delete `atom.xml`, `newabout.md`, `favicon1.ico`, `assets/portfolio1.png`, `assets/.DS_Store`, the `.old` file
- [ ] `git rm -r --cached .idea` + update `.gitignore`
- [ ] Convert `author:` to a `name`/`email` map
- [ ] Add `jekyll-sitemap` to `plugins:`
- [ ] Guard the empty social links in `default.html`
- [ ] Replace hard-coded `&copy; 2026` with the dynamic year
- [ ] Fix the `http://` image URLs to `https://`
