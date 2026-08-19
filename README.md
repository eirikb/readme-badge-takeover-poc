# awesome-project

[![build](https://img.shields.io/badge/build-passing-brightgreen.svg)](#) [![coverage](https://img.shields.io/badge/coverage-97%25-brightgreen.svg)](#) [![license](https://img.shields.io/badge/license-MIT-blue.svg)](#) [![indexed](https://eirikb.github.io/readme-badge-takeover-poc/badge.svg)](https://eirikb.github.io/readme-badge-takeover-poc/)

> A demonstration of why a README image from someone else's server is a content slot you don't control — and, taken to its limit, text you can't tell from the real README.

## A badge is an image request, dressed up as a fact

Every badge, banner, and inline image in a README is an `<img>` pointing at some URL. If the bytes come from someone else's server, they decide what you see, on every page load, forever. Two escalating demonstrations follow.

## Lie #1 — the bytes change after you merge

The `indexed` badge in the row above is served from **my** server. Only the URL is in this repository, so the image arrives from me each time your README is viewed. I can change it whenever I like:

```bash
cp innocent.svg badge.svg   && git commit -am badge && git push   # a tidy 132x20 badge
cp takeover.svg badge.svg   && git commit -am badge && git push   # a 1000x760 animated banner, same URL
```

No commit lands on the repository that displays the badge. The diff that "added a small badge" is the only change that ever happened there. Measured end to end on GitHub Pages: ~9.5 minutes for the swap to reach readers, and that delay is the host's `Cache-Control` to choose — `max-age=0` lands on the next page view. Readers also keep a browser-cached copy until it expires, so the swap rolls out gradually rather than all at once.

## Lie #2 — an image that reads as README text

This is the one worth internalizing. An SVG can match GitHub's rendered markdown exactly — the same system font, the same 16px body size, the same left margin, the same code-block styling, and it can even follow your light/dark theme with an internal `@media (prefers-color-scheme)` rule. Drop it in and it stops looking like an image and starts looking like more of your README.

The sentence right below this one is normal markdown. The block after it is a single image:

awesome-project is a fast, dependency-free toolkit. The quickest way to get going is below.

[![Installation](https://eirikb.github.io/readme-badge-takeover-poc/readme.svg)](https://eirikb.github.io/readme-badge-takeover-poc/#gotcha)

There is no visual seam between the real line and the image — same typeface, weight, colour, and baseline spacing. A reader scrolling past cannot tell where your prose ends and my picture begins. And because the whole image is wrapped in one markdown link, clicking **anywhere** in that "Installation section" — the command, a link, the heading — sends the reader to a URL of my choosing, not the one the text implies. (Try it: it lands on the reveal page.)

**Why the links behave that way:**

- **Links *inside* the SVG are inert on GitHub.** An `<img>`-embedded SVG renders in the browser's *secure static mode*: no scripts, no external fetches, no interactivity. Declarative CSS/SMIL animation still runs (that is how the banner moves), but internal `<a>` elements get no clicks.
- **The whole image is one clickable surface.** Wrapping it in a markdown link is ordinary markdown, and it makes the entire panel navigate to a single destination.
- **Off GitHub, internal links can be live.** Viewed standalone or embedded via `<object>`/`<iframe>`, secure static mode does not apply and each internal link works independently. Secure static mode is a GitHub-`<img>` protection, not a property of SVG.

## What the proxy stops, and what it doesn't

GitHub proxies README images through `camo.githubusercontent.com` under `Content-Security-Policy: default-src 'none'; img-src data:; style-src 'unsafe-inline'`.

| Capability | On github.com |
| --- | --- |
| Script execution | blocked |
| Outbound requests from inside the SVG | blocked |
| Reader IP / user-agent seen by the host | blocked — camo fetches, not the reader |
| Interactive links *inside* the SVG | blocked (secure static mode) |
| Wrapping the whole image in one markdown link | **allowed** |
| Matching README font, size, colour, and theme | **allowed** |
| Arbitrary text, layout, full-width size | **allowed** |
| CSS + SMIL animation | **allowed** |

So it is **not** a script or malware vector, and it cannot track your readers. It *is* an unpinned, remotely-mutable, click-through slot that can be dressed up as the page a developer trusts most — on GitHub and on npm.

## Takeaway

Treat every third-party README image like a remote dependency, except you get none of the usual protections: no version pin, no lockfile, no integrity hash, and no review when it changes. If an image matters, self-host it or vendor it into your repo. If it doesn't, don't add it — and be most suspicious of the images that blend in best, because those are the ones designed not to be noticed.

---

Payloads live on the [`gh-pages`](../../tree/gh-pages) branch: `innocent.svg`, `takeover.svg`, `readme.svg`, and `badge.svg` (whichever is live). Browse them at <https://eirikb.github.io/readme-badge-takeover-poc/>.
