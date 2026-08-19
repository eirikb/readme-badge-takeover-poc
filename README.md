# ms-365-mcp-server

[![npm version](https://img.shields.io/npm/v/@softeria/ms-365-mcp-server.svg)](https://www.npmjs.com/package/@softeria/ms-365-mcp-server) [![build status](https://github.com/softeria/ms-365-mcp-server/actions/workflows/build.yml/badge.svg)](https://github.com/softeria/ms-365-mcp-server/actions/workflows/build.yml) [![license](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/softeria/ms-365-mcp-server/blob/main/LICENSE) [![indexed](https://eirikb.github.io/readme-badge-takeover-poc/badge.svg)](https://eirikb.github.io/readme-badge-takeover-poc/)

Microsoft 365 MCP Server

A Model Context Protocol (MCP) server for interacting with Microsoft 365 and Microsoft Office services through the Graph
API.

---

## What this repo actually is

A proof of concept for the risk discussed in softeria/ms-365-mcp-server#641.

The badge line above is a copy of the real one, with a fourth badge appended exactly as a
directory-listing service would ask you to add it. That fourth badge is served from this repo's
own `gh-pages` branch, so **I** control what it renders, not this README.

Nothing about the markdown above changes between the harmless and the hostile version. Only the
bytes at the far end of the URL change.

### The swap

```bash
# harmless: a 132x20 shields-style badge
git checkout gh-pages && cp innocent.svg badge.svg && git commit -am "badge" && git push

# hostile: an 880x420 animated full-width banner, same URL
git checkout gh-pages && cp takeover.svg badge.svg && git commit -am "badge" && git push
```

GitHub proxies README images through `camo.githubusercontent.com`, which serves
`cache-control: max-age=300`. So a swap reaches every reader within five minutes, with no commit
to the repo displaying it.

### What the proxy does and does not stop

Camo's CSP on the proxied response is `default-src 'none'; img-src data:; style-src 'unsafe-inline'`.

| | |
| --- | --- |
| Script execution | blocked |
| Outbound requests from inside the SVG | blocked |
| Reader IP / user-agent exposed to the host | blocked (camo fetches, not the reader) |
| Arbitrary text and layout | **allowed** |
| Embedded raster artwork (`data:` URIs) | **allowed** |
| CSS animation (`style-src 'unsafe-inline'`) | **allowed** |
| SMIL animation (`<animate>`, outside CSP entirely) | **allowed** |
| Arbitrary size (width clamps to the column, height does not) | **allowed** |

So it is not a malware vector. It is an unpinned, remotely mutable, animatable content slot at the
top of your README, on GitHub and on npm.

Files live on the `gh-pages` branch: `innocent.svg`, `takeover.svg`, `badge.svg` (whichever is live).
