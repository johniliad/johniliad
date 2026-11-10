# Publishing `github.iliadis.net`

This repository holds **one** artifact published in **two** places:

| Surface | What it shows | Source |
| :-- | :-- | :-- |
| `github.com/johniliad` | Profile README | `README.md` |
| `github.iliadis.net` | Identity-bridge site | `index.html` + `assets/` |

Both are served from the same branch, so they can never drift apart.

---

## 1. Files in this repository

```
README.md                              profile README (rendered by GitHub)
index.html                             identity-bridge page (rendered by Pages)
CNAME                                  github.iliadis.net
.nojekyll                              serve files verbatim (no Jekyll build)
robots.txt                             crawl policy + sitemap pointer
sitemap.xml                            single-URL sitemap
.well-known/security.txt                RFC 9116 disclosure contact (see below)
.gitignore                             keeps local scratch out of history
DEPLOY.md                              this file
assets/img/john-iladis.webp            portrait used in both surfaces
assets/img/github-identity-mark.svg    favicon / brand mark
assets/img/github-identity-social-card.png   1200x630 Open Graph card
assets/img/suite-hero.jpg              Tools4U suite banner
assets/img/threatpulse-hero.jpg        ThreatPulse banner
assets/img/cyberpulse-card.webp        CyberPulse card art
assets/img/quickrisk-card.webp         QuickRisk card art
assets/img/eu-cyber-incident-card.webp EU Cyber Incident card art
assets/img/brig-card.webp              BRIG card art
_tmp/                                  local scratch (git-ignored, never published)
```

`_tmp/` is deliberately kept out of the deployed surface: it is listed in `.gitignore`, is not
linked from either page, and is not part of any commit.

`.gitignore` encodes that boundary exhaustively rather than case by case: scratch directories,
stray root-level captures and exports, editor and language-server state, cloud-sync conflict
copies, reconnaissance output (DNS, HAR, packet captures), design sources, office documents and
anything resembling a credential are all excluded, and every block states why. Two checks keep
the file honest before a push — `git ls-files` lists exactly what will be published, and
`git ls-files | Where-Object { git check-ignore -q --no-index -- $_; $LASTEXITCODE -eq 0 }` must
print nothing at all, because a tracked path matching an ignore rule means the rule is too broad.
The `--stdin -v` form of that second check cannot be trusted here: PowerShell appends a carriage
return to every piped line, so `--stdin` receives `.gitignore\r`, which no rule matches, and `-v`
prints negation matches as well — its output cannot separate a real hit from that noise.

### Repository visibility — this repository must stay public

GitHub renders a profile README only when the repository named exactly after the account
(`johniliad/johniliad`) is **public**. A private one makes the README disappear from
`https://github.com/johniliad` without any error message.

Publishing Pages from a private repository needs a paid plan, and even then the site itself is
world-readable unless the account is on Enterprise Cloud with private Pages. So a private
repository would hide nothing that `github.iliadis.net` does not already serve — while breaking
`ProfilePage.isBasedOn` → `codeRepository`, which now points at this repository as the page's
source for every visitor and crawler.

Measured state (2026-11-10): `private: false`, `visibility: public`, `has_pages: true`. The 18
tracked files are HTML, Markdown, one plain-text RFC 9116 file, SVG and raster images — no
scripts, no credentials, no environment files. `_tmp/` (the only place secrets could plausibly
accumulate) has never appeared in any commit on any branch.

### The disclosure contact: `/.well-known/security.txt`

RFC 9116 puts a machine-readable vulnerability contact at a fixed path, and this repository serves
one at `https://github.iliadis.net/.well-known/security.txt`. Pages publishes it because
`.nojekyll` is present: Jekyll silently drops dot-directories, whereas the flag stops Jekyll
running at all, and `.txt` is served as `text/plain; charset=utf-8` — the media type §3 requires.

Four field decisions, each deliberate:

* **`Contact` is a web URI** (`https://www.iliadis.net/contact/`), not a `mailto:`. §2.5.3 accepts
  a web page with contact information, that page is the single channel already published in the
  footers of `index.html` and `README.md`, and a form does not hand a harvestable address to every
  crawler (§5.8). When a security mailbox exists, it can be prepended: the first `Contact` is the
  preferred one, the later one the fallback.
* **`Canonical` names the file, not the site.** §2.5.2 defines it as the URI where the *file* is
  located, and adds that a file retrieved from a URI absent from every `Canonical` field SHOULD
  NOT be trusted. `Canonical: https://github.iliadis.net/` would therefore invalidate the file a
  scanner had just fetched; the canonical URI is the well-known path itself.
* **`Expires` is set 364 days ahead.** §2.5.5 makes the field mandatory, forbids repeating it and
  recommends less than a year; the date is the renewal trigger in §9. An expired file is stale,
  not merely old — parsers discard it (§5.3), which is worse than never having published one.
* **No `Policy`, `Encryption`, `Acknowledgments` or `Hiring`**, because none of them would point at
  something that exists, and an empty promise is worse than an absent field. The file is unsigned:
  §2.3 only *recommends* an OpenPGP cleartext signature, no key is published for this domain, and
  an `Encryption` field must carry a key URI, never a key.

Scope is per host (§3.1): the file speaks for `github.iliadis.net` only — not for `www.iliadis.net`,
not for `github.com/johniliad`, not for the applications the page describes. There is no
`/security.txt` copy at the root: §3 prefers the well-known path, and Pages cannot issue the
redirect §3 describes for the legacy location, so a second copy would only invite drift.

---

## 2. First push

The repository is **already initialised and committed** in this folder (`main`, working tree clean,
remote `origin` pointing at `https://github.com/johniliad/johniliad.git`). The remote is *not*
empty, though, so read the reconciliation note below before pushing — a plain `git push` will be
rejected.

```powershell
cd "C:\Users\john\OneDrive\LLM apps\readme-profile-github"
git push -u origin main
```

If `origin` should use SSH instead of HTTPS:

```powershell
git remote set-url origin git@github.com:johniliad/johniliad.git
git push -u origin main
```

If the repository on GitHub already has commits (a README created in the browser, for example):

```powershell
git fetch origin
git reset --soft origin/main                    # keep the working tree, replay it on top
git add -A
git commit -m "Profile README and github.iliadis.net identity bridge"
git push origin main
```

> **Measured state of `origin/main` (2026-11-10):** it holds a *separate, unrelated* history of 17
> commits — README edits made in the browser plus several `CNAME` changes — whose tree contains only
> `README.md` (14 lines) and `CNAME` = `git.iliadis.net`. The two histories share no ancestor, so a
> plain `git push` is rejected as non-fast-forward; replay this folder's tree on top of it with the
> `fetch` / `reset --soft` sequence above. That push also replaces the `CNAME`, so the Pages custom
> domain moves from `git.iliadis.net` to `github.iliadis.net` and the legacy README stops being
> served — see section 5 for what is left at `git.iliadis.net` afterwards.

The profile repository must be **public** for Pages to serve it.

---

## 3. Enable GitHub Pages (once)

1. Open `https://github.com/johniliad/johniliad/settings/pages`.
2. **Source**: *Deploy from a branch* → Branch `main` → Folder `/ (root)` → **Save**.
3. Wait for the first build (1–2 minutes). The site appears at
   `https://johniliad.github.io/johniliad/` immediately.
4. **Custom domain**: enter `github.iliadis.net` → **Save**. GitHub writes `CNAME` itself;
   the file in this repository already contains the same value, so nothing changes.
5. Once DNS is verified, tick **Enforce HTTPS**. The certificate is issued by GitHub
   (Let's Encrypt) within minutes to about an hour.

> Only one custom domain can be attached to one Pages site. This repository's custom domain
> must be `github.iliadis.net`, not `git.iliadis.net` — see section 5.

---

## 4. DNS

Verified on 2026-11-10 (Cloudflare resolver, `1.1.1.1`):

| Record | Value | State |
| :-- | :-- | :-- |
| `github.iliadis.net` | `CNAME johniliad.github.io` | already in place — this site |
| `git.iliadis.net` | `CNAME johniliad.github.io` | unused by this site — see section 5 |
| `iliadis.net` (apex) | unchanged — existing web host | do not touch |

Nothing needs to be added for `github.iliadis.net` to start working: the name already
resolves to GitHub Pages. It currently returns **HTTP 404** only because no Pages site has
`github.iliadis.net` configured as its custom domain yet. Step 3 fixes exactly that.

Optional hardening (recommended): leave the existing DNSSEC and CAA records on `iliadis.net`
as they are. GitHub Pages needs no new CAA exception, because the certificate is issued to
GitHub's own infrastructure rather than to your zone.

### Measured state, 2026-11-10

| Check | Result |
| :-- | :-- |
| `github.iliadis.net` DNS | resolves (`CNAME johniliad.github.io`) |
| `https://github.iliadis.net/` | HTTP 404, `server: GitHub.com`, `strict-transport-security` present — Pages is reachable, the custom domain is simply not claimed yet |
| all 22 outbound links in `index.html` | HTTP 200 (ResearchGate answers 403 to command-line clients; opens normally in a browser) |
| 8 local assets referenced by `index.html` | present on disk |
| 7 local images referenced by `README.md` | present on disk |
| JSON-LD blocks | both parse as valid JSON |
| rendered layout | verified headless at 1300 px and 520 px — no overflow, no missing images |
| page theme | light: white canvas, `#0b1220` ink, `#0f766e` accents, `theme-color` `#ffffff` |
| social card | 1200×630 PNG regenerated on a light ground so the share preview matches the page |

---

## 5. Domain scope: `github.iliadis.net` only

This repository has exactly one custom domain: **`github.iliadis.net`**. Every published
reference agrees — `CNAME`, the canonical link, `og:url`, `twitter:image`, `sitemap.xml`,
`robots.txt` and the README badges. Nothing in this project points at `git.iliadis.net`.

`git.iliadis.net` is **not** part of this project. It used to serve an older, Jekyll-rendered
copy of the README (it still answers with `canonical http://git.iliadis.net/`, has no HTTPS
certificate and shows a broken Markdown table). Do not attach it here:

* Only one custom domain can be attached to one Pages site. Pointing this repository at
  `git.iliadis.net` would take `github.iliadis.net` offline and republish the page on a name
  the rest of the identity graph does not reference.
* The DNS record `CNAME git.iliadis.net → johniliad.github.io` may stay or go; it belongs to
  the old site, not to this one. Once `github.iliadis.net` is live, that older surface is
  redundant — either delete the repository that owns it (its Pages URL then 404s and the DNS
  record becomes inert) or leave it and accept a stale page on a name nothing links to.
* If that name should later redirect instead of dead-ending, publish the one-file redirect
  below in a **separate** repository with its own `CNAME` set to `git.iliadis.net` — never in
  this one.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Moved to github.iliadis.net</title>
  <link rel="canonical" href="https://github.iliadis.net/" />
  <meta name="robots" content="noindex" />
  <meta http-equiv="refresh" content="0; url=https://github.iliadis.net/" />
</head>
<body>
  <p>This identity page has moved to
    <a href="https://github.iliadis.net/">github.iliadis.net</a>.</p>
</body>
</html>
```

A second Pages site needs its own DNS record, so keep `CNAME git.iliadis.net → johniliad.github.io`
(already present) and set that repository's custom domain to `git.iliadis.net`.

---

## 6. Optional: clean the working tree

```powershell
Remove-Item -LiteralPath "_tmp" -Recurse -Force -WhatIf   # inspect first
Remove-Item -LiteralPath "_tmp" -Recurse -Force
```

`_tmp/` holds only scratch: PowerShell verification scripts, DNS dumps, screenshots and reports.
It is already excluded by `.gitignore`, so it never reaches GitHub — delete it whenever you like.

---

## 7. Verification checklist

Run from the repository root after Pages reports *Your site is live*:

```powershell
# 1. DNS and HTTP
Resolve-DnsName github.iliadis.net -Server 1.1.1.1 | Select-Object Name,Type,NameHost
curl.exe -sS -o NUL -w "%{http_code} %{content_type}`n" https://github.iliadis.net/

# 2. HTTPS certificate and redirect
curl.exe -sSI https://github.iliadis.net/ | Select-String -Pattern "HTTP/|location|strict-transport"

# 3. Identity files
curl.exe -sS https://github.iliadis.net/robots.txt
curl.exe -sS https://github.iliadis.net/sitemap.xml
curl.exe -sS https://github.iliadis.net/CNAME
curl.exe -sSI https://github.iliadis.net/.well-known/security.txt | Select-String -Pattern "HTTP/|content-type"
curl.exe -sS https://github.iliadis.net/.well-known/security.txt

# 4. Metadata actually served
curl.exe -sS https://github.iliadis.net/ | Select-String -Pattern "canonical|og:image|application/ld\+json"

# 5. GitHub -> iliadis.net identity link (expect rel="nofollow me" on www.iliadis.net)
curl.exe -sS https://github.com/johniliad | Select-String -Pattern 'rel="nofollow me"'

# 6. Publication surface: nothing stale, nothing private
git ls-files                                              # expect exactly the 18 files in §1
git ls-files | Where-Object { git check-ignore -q --no-index -- $_; $LASTEXITCODE -eq 0 }   # expect no output
git status --short --ignored                               # everything withheld, and by which rule

# 7. security.txt is still fresh: Expires must exist, parse, and be in the future (RFC 9116 §2.5.5)
$txt = (Invoke-WebRequest https://github.iliadis.net/.well-known/security.txt -UseBasicParsing).Content
$exp = ([regex]::Match($txt, '(?m)^Expires:\s*(\S+)')).Groups[1].Value
"Expires: $exp"
[datetimeoffset]::Parse($exp) -gt [datetimeoffset]::UtcNow    # expect True
```

Then, in a browser:

- [ ] `https://github.iliadis.net/` renders the light identity bridge, portrait loaded, no broken images.
- [ ] Open Graph preview (LinkedIn Post Inspector, X Card Validator) shows the 1200×630 card, white background, dark type.
- [ ] Google Rich Results Test on `https://github.iliadis.net/` finds `ProfilePage` + `Person`.
- [ ] `https://github.com/johniliad` renders the new README; every `assets/img/...` image displays.
- [ ] Every "Live app" link in the Projects section opens the application.
- [ ] `view-source:` on the live page contains no reference to `git.iliadis.net`.
- [ ] `https://github.iliadis.net/.well-known/security.txt` opens as plain text — not a 404, not HTML — and [securitytxt.org](https://securitytxt.org/) reports one `Contact`, a future `Expires` and the well-known `Canonical`.
- [ ] The *Identity, stated once* section shows the provenance note, and no graph disclosure is rendered on the page; `view-source:` still shows `"isBasedOn"` and `#source-repository` inside the `<head>` JSON-LD.
- [ ] `https://github.com/johniliad/johniliad` shows **Homepage** `github.iliadis.net` in the right-hand *About* panel.
- [ ] `https://github.com/johniliad` still lists `www.iliadis.net` among the profile links — that is the `rel="me"` half of the loop.
- [ ] The repository is still **public** (`(Invoke-RestMethod https://api.github.com/repos/johniliad/johniliad).private` returns `False`); a private repository silently removes the profile README.

---

## 8. Two-way linking: what GitHub emits, what only you can set

An identity link is worth something only when both ends state it. Half of this is in the files; the other half lives in GitHub settings and cannot be committed.

### What the two surfaces already say (measured 2026-11-10)

| Direction | Mechanism | Where it comes from |
| :-- | :-- | :-- |
| `www.iliadis.net` → GitHub | `<link rel="me" href="https://github.com/johniliad">` + a visible `rel="me"` link | that site's HTML |
| GitHub → `www.iliadis.net` | profile sidebar link rendered as `<a rel="nofollow me" ... href="https://www.iliadis.net">` | GitHub profile *website* field |
| `github.iliadis.net` → GitHub | `<link rel="me">` in `<head>` + *Verified profiles* card | `index.html` |
| `github.iliadis.net` → `www.iliadis.net` | `rel="me"`, `rel="author"`, `Person.@id` reuse, `sameAs` | `index.html` head + JSON-LD |
| `github.iliadis.net` → this repository | `ProfilePage.isBasedOn` → `#source-repository` (`SoftwareSourceCode`) + provenance note | `index.html` |
| `README.md` → bridge and website | badges, identity table, footer | `README.md` |

Because GitHub marks its profile links `me` and `www.iliadis.net` returns `rel="me"` to the profile, the strongest loop available to you is **already closed** — both directions carry a machine-readable claim, not just a hyperlink.

> Do **not** put the repository URL in `Person.sameAs`. `sameAs` asserts "another profile of this same individual"; a code repository is a work, not an identity. It belongs in `isBasedOn` / `codeRepository`, which is where it now is.

### What GitHub will not let you commit

GitHub sanitises rendered README HTML: links from `README.md` come out `rel="nofollow"` and never `rel="me"`. So the GitHub-side half must be set in the UI, once:

| Setting | Where | Value |
| :-- | :-- | :-- |
| Profile **website** | Settings → Public profile | `https://www.iliadis.net/` — already set; this is the link GitHub marks `rel="me"` |
| Profile **social accounts** | Settings → Public profile | LinkedIn, ORCID and Instagram are already listed; GitHub renders four profile links, so `github.iliadis.net` fits only if a slot is freed |
| Repository **Homepage** | repo → *About* → gear | `https://github.iliadis.net/` — currently **empty**, so the repo does not yet link to its own site |
| Repository **Description** | repo → *About* → gear | already set |
| Repository **Topics** | repo → *About* → gear | currently **empty**; suggested: `profile-readme`, `identity-bridge`, `schema-org`, `github-pages`, `cybersecurity` |

The Homepage field is the one real gap: it puts a GitHub-owned link from `github.com/johniliad/johniliad` to `github.iliadis.net` directly under the description, on the page most likely to be reached from search.

Verify it afterwards without a browser:

```powershell
(Invoke-RestMethod https://api.github.com/repos/johniliad/johniliad) | Select-Object homepage, topics
```

### One line on the other site

`www.iliadis.net` already points at the profile with `rel="me"`. For the subdomain to sit *inside* that loop rather than merely pointing into it, the page that owns the identity should also carry:

```html
<link rel="me" href="https://github.iliadis.net/" />
```

That is a change in the `www.iliadis.net` repository, not this one — this repository's half is already published.

---

## 9. Keeping it current

* Content changes: edit `index.html` for the site, `README.md` for the profile. Commit and push — Pages redeploys on every push to `main`.
* New published project: add an `ItemList` entry in the JSON-LD **and** a card in the Projects grid, so the human and machine views stay in step.
* New verified profile: add it to the *Verified profiles* grid, to `sameAs` in the JSON-LD, and to the identity table in `README.md`.
* Changed portrait: replace `assets/img/john-iladis.webp` (keep the 180×179 display size in `README.md` and the `width`/`height` attributes in `index.html` in step with the real file) and regenerate the social card.
* Before the `Expires` date in `.well-known/security.txt`: move it to 364 days from that day, and change `Contact` if the reporting channel changes. An expired file is discarded by parsers, so the date is a maintenance deadline rather than decoration — the checklist in §7 step 7 is the test.

### Rollback

```powershell
git revert HEAD --no-edit
git push origin main
```

Pages redeploys the previous commit within a minute; DNS and the certificate are untouched.

---

## 10. Boundaries

This site states identity, not capability claims. It publishes no metrics that are not already
documented on `iliadis.net`, cites no regulator beyond naming the instruments the applications
implement, and makes no claim of certification, audit or endorsement by any authority.

`.well-known/security.txt` publishes a reporting *channel*, not a permission to test: it invites no
probing, and RFC 9116 §5.5 states that explicitly. Bug-bounty terms, safe-harbour language or a
disclosure policy would each have to exist before any of them could be referenced.
