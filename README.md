# awesome-project

[![build](https://img.shields.io/badge/build-passing-brightgreen.svg)](#) [![coverage](https://img.shields.io/badge/coverage-97%25-brightgreen.svg)](#) [![license](https://img.shields.io/badge/license-MIT-blue.svg)](#) [![security](https://eirikb.github.io/readme-badge-takeover-poc/security.svg)](https://eirikb.github.io/readme-badge-takeover-poc/) [![indexed](https://eirikb.github.io/readme-badge-takeover-poc/badge.svg)](https://eirikb.github.io/readme-badge-takeover-poc/)

> A demonstration of why a README image from someone else's server is a claim you can't verify, a content slot you don't control, and — taken to its limit — a whole fake page you can be made to read and click.

## A badge is an image request, dressed up as a fact

Everything below is one idea escalating. A badge, a banner, and a "README" are all just `<img>` tags. If the bytes come from someone else's server, they own what you see.

### Lie #1 — the claim nobody verified

`security | audited ✓` in the row above looks like the real thing. It is drawn in authentic shields.io flat style, the same green every trustworthy project wears, and it is completely fake. **Audited by whom?** Nobody. A static badge is a claim with no evidence attached, false the moment you paste it in. `build passing` and `coverage 97%` are the same trick: hardcoded images wired to no CI.

### Lie #2 — the bytes that change after you merge

The `indexed` badge's image lives on **my** server. Only the URL is in this repository, so the bytes arrive from me on every README view, forever.

```bash
cp innocent.svg badge.svg   && git commit -am badge && git push   # a tidy 132x20 badge
cp takeover.svg badge.svg   && git commit -am badge && git push   # a 1000x760 animated banner, same URL
```

No commit lands on the repo that displays it. Measured end to end on GitHub Pages: ~9.5 minutes for the swap to reach readers, and that delay is the host's `Cache-Control` to choose — `max-age=0` lands on the next page view.

### Lie #3 — the entire README, faked

This is Lie #2 with the payload dressed as GitHub itself. The image below is not a screenshot and not this file. It is an SVG that redraws GitHub's rendered-README chrome — the `README.md` header bar, an underlined `<h1>`, mini badges, a code block with a copy icon, a blue "Security advisory" alert, and a list of blue links. A reader scrolling a repo cannot tell it from the real documentation panel.

[![It looks exactly like a README](https://eirikb.github.io/readme-badge-takeover-poc/readme.svg)](https://eirikb.github.io/readme-badge-takeover-poc/#gotcha)

**About the links** — the interesting part:

- **Inside the SVG, links are inert on GitHub.** An `<img>`-embedded SVG renders in the browser's *secure static mode*: no scripts, no external fetches, no interactivity. The `<a>` elements inside get no clicks. (Declarative CSS/SMIL animation still runs — that is why the banner moves.)
- **The whole panel is one giant link.** The image above is wrapped in a single markdown link. Click *anywhere* on that fake README — the install command, the "Download the signed patch" button, a resource link — and you go where I point, not where the text says. Try it: it sends you to the reveal page.
- **On a standalone view, the internal links are live.** Open `readme.svg` directly, or embed it via `<object>`/`<iframe>` off-GitHub, and every link in it is individually clickable to its own destination. Secure static mode is a GitHub-`<img>` protection, not an SVG one.

So the ceiling is: a full-screen forgery of the page a developer trusts most, animated, that navigates on click, and updates whenever the host likes — with a repo diff that says nothing but "add badge".

## What the proxy stops, and what it doesn't

GitHub proxies README images through `camo.githubusercontent.com` under `Content-Security-Policy: default-src 'none'; img-src data:; style-src 'unsafe-inline'`.

| Capability | On github.com |
| --- | --- |
| Script execution | blocked |
| Outbound requests from inside the SVG | blocked |
| Reader IP / user-agent seen by the host | blocked — camo fetches, not the reader |
| Interactive links *inside* the SVG | blocked (secure static mode) |
| Wrapping the whole image in one markdown link | **allowed** |
| Arbitrary text, layout, full-width size | **allowed** |
| Embedded raster artwork (`data:` URIs) | **allowed** |
| CSS + SMIL animation | **allowed** |

Not a script or malware vector, and it can't track your readers. It *is* an unpinned, remotely-mutable, animatable, click-through slot the size of the whole README — on GitHub and on npm.

## Takeaway

Treat every third-party README image like a remote dependency, except you get none of the usual protections: no version pin, no lockfile, no integrity hash, no review on change, and for the static ones no evidence behind the claim. If an image matters, self-host it or vendor it into your repo. If it doesn't, don't add it — and be twice as suspicious of the ones that look the most official.

---

Payloads live on the [`gh-pages`](../../tree/gh-pages) branch: `innocent.svg`, `takeover.svg`, `security.svg`, `readme.svg`, and `badge.svg` (whichever is live). Browse them at <https://eirikb.github.io/readme-badge-takeover-poc/>.
