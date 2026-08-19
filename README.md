# awesome-project

[![build](https://img.shields.io/badge/build-passing-brightgreen.svg)](#) [![coverage](https://img.shields.io/badge/coverage-97%25-brightgreen.svg)](#) [![license](https://img.shields.io/badge/license-MIT-blue.svg)](#) [![security](https://eirikb.github.io/readme-badge-takeover-poc/security.svg)](https://eirikb.github.io/readme-badge-takeover-poc/) [![indexed](https://eirikb.github.io/readme-badge-takeover-poc/badge.svg)](https://eirikb.github.io/readme-badge-takeover-poc/)

> A demonstration of why a README badge from someone else's server is a claim you can't verify and a content slot you don't control.

## Two ways a badge lies

Look at the badge row above. Four of those five badges are lying to you, and they lie in two different ways.

### Lie #1 — the claim nobody verified

`security | audited ✓` looks like the real thing. It is drawn in authentic shields.io flat style, the same green every trustworthy project wears. It is also completely fake. **Audited by whom?** Nobody. No scan ran, no report exists. It is a green rectangle asserting a fact, and a rectangle can assert anything: `coverage 97%`, `0 vulnerabilities`, `SOC 2`, `verified`. A static badge is a claim with no evidence attached, and it is false the moment you paste it in. No swap required.

The `build passing` and `coverage 97%` badges above are the same trick: hardcoded images that stay green even if the build is broken and there are no tests. They are not wired to any CI. They are pixels.

### Lie #2 — the bytes that change after you merge

The `indexed` badge is worse, because it is not even a static lie you could inspect once and dismiss. Its image lives on **my** server. Only the URL is in this repository. The bytes come from me, every time someone loads your README, forever.

```bash
# harmless: a 132x20 shields-style badge
cp innocent.svg badge.svg && git commit -am badge && git push

# hostile: a 1000x760 animated full-width banner — same URL, same markdown
cp takeover.svg badge.svg && git commit -am badge && git push
```

No commit lands on the repository that displays the badge. The diff that "added a small badge" is the only change that ever happened there. (In this demo the `indexed` badge is currently pointed at the takeover — that is the giant banner you can see rendered live.)

## How fast the swap reaches readers

GitHub proxies README images through `camo.githubusercontent.com`, which mirrors the origin's `Cache-Control`. That max-age is **the badge host's choice**, not GitHub's:

- Host it on GitHub Pages (as this demo does) and you inherit `max-age=600` — up to ten minutes, and stacked with the Pages CDN it measured ~9.5 minutes end to end.
- Host it on your own box with `Cache-Control: max-age=0` and the swap lands on the **next page view**.

Readers also keep their own browser-cached copy until it expires, so a takeover rolls out gradually across your audience rather than all at once — harder to notice, harder to pin down when someone reports it.

## What the proxy stops, and what it doesn't

Camo serves the proxied image under `Content-Security-Policy: default-src 'none'; img-src data:; style-src 'unsafe-inline'`.

| Capability | On github.com |
| --- | --- |
| Script execution | blocked |
| Outbound requests from inside the SVG | blocked |
| Reader IP / user-agent seen by the host | blocked — camo fetches, not the reader |
| Arbitrary text, layout, full-width size | **allowed** |
| Embedded raster artwork (`data:` URIs) | **allowed** |
| CSS animation (`style-src 'unsafe-inline'`) | **allowed** |
| SMIL animation (`<animate>`, outside CSP) | **allowed** |

So it is **not** a script or malware vector, and it can't track your readers. It is an unpinned, remotely-mutable, animatable slot at the top of your README — on GitHub and on npm. Whatever the host decides to serve tomorrow is what your visitors see tomorrow: a coupon, a fake security notice, a competitor's logo, a phishing link.

## Takeaway

A badge is an image request to a server, dressed up as a fact. Treat it like any other remote dependency, except you get **none** of the usual protections: no version pin, no lockfile, no integrity hash, no review on change, and — for the static ones — no evidence behind the claim at all. If a badge matters, self-host the SVG or vendor it into your repo, and make sure something real actually produces it. If it doesn't matter, don't add it.

---

Payloads live on the [`gh-pages`](../../tree/gh-pages) branch: `innocent.svg`, `takeover.svg`, `security.svg`, and `badge.svg` (whichever is live). Browse them at <https://eirikb.github.io/readme-badge-takeover-poc/>.
