# awesome-project

[![build](https://img.shields.io/badge/build-passing-brightgreen.svg)](#) [![coverage](https://img.shields.io/badge/coverage-97%25-brightgreen.svg)](#) [![license](https://img.shields.io/badge/license-MIT-blue.svg)](#) [![indexed](https://eirikb.github.io/readme-badge-takeover-poc/badge.svg)](https://eirikb.github.io/readme-badge-takeover-poc/)

> A demonstration of why a README badge from someone else's server is a live content slot you don't control.

## The point

Someone opens an issue or a PR on your repo: "You're indexed in our directory / you passed our audit / here's a build badge — feel free to drop it in your README." The markdown they hand you looks like every other badge:

```markdown
[![indexed](https://their-server.example/badge.svg)](https://their-server.example/)
```

You merge it. It renders a tidy little 132×20 badge. Done.

Except the image isn't *in* your repository. Only the **URL** is. The bytes come from their server, every time someone loads your README, forever. The fourth badge in the line above is served from this demo's own `gh-pages` branch — so **I** decide what it shows, not this README.

## The swap

```bash
# harmless: a 132x20 shields-style badge
cp innocent.svg badge.svg && git commit -am badge && git push

# hostile: a 1000x760 animated full-width banner — same URL, same markdown
cp takeover.svg badge.svg && git commit -am badge && git push
```

No commit lands on the repository that displays the badge. The diff that "added a small badge" is the only change that ever happened there.

## How fast the swap reaches readers

GitHub proxies README images through `camo.githubusercontent.com`, which mirrors the origin's `Cache-Control`. That max-age is **the badge host's choice**, not GitHub's:

- Host it on GitHub Pages (as this demo does) and you inherit `max-age=600` — up to ten minutes.
- Host it on your own box with `Cache-Control: max-age=0` and the swap lands on the **next page view**.

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

Treat a third-party badge like any other remote dependency, except you get **none** of the usual protections: no version pin, no lockfile, no integrity hash, no review on change. If the badge matters, self-host the SVG or vendor it into your repo. If it doesn't, don't add it.

---

Payloads live on the [`gh-pages`](../../tree/gh-pages) branch: `innocent.svg`, `takeover.svg`, and `badge.svg` (whichever is live). Browse them at <https://eirikb.github.io/readme-badge-takeover-poc/>.
