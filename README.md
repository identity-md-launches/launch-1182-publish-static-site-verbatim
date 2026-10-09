# The Kiln · ZTO — verbatim static mirror

This repository publishes a byte-for-byte mirror of an existing static site. Nothing in
`dist/` was written, reformatted, minified, re-encoded or otherwise changed by this
repository: every file is exactly what the source served, verified against the source's own
`MANIFEST.json`.

- **Source:** `https://7130de0a-c74f-4dc3-9a8a-79db37963f97-00-1lk87v2ravyz1.reed.replit.dev/zto/kiln-site/`
- **Export:** `dist/` (the folder a static host or gateway serves)
- **Integrity record:** `dist/MANIFEST.json` (upstream, sha256 per path) and
  `dist/MANIFEST-PUBLISHED.txt` (sha256 of what is actually in `dist/`)
- **Design documentation:** `DESIGN.md` (tokens, typography, components and responsive
  behaviour read from `dist/index.html`)
- **Validation record:** `artifacts/validation.md` (Better Interface review, browser checks,
  findings and limitations)

## What is in `dist/`

| Path | Role |
| --- | --- |
| `index.html` | The single page: inline CSS, inline JavaScript, no build step |
| `MANIFEST.json` | Upstream sha256 map of every mirrored file (except itself) |
| `MANIFEST-PUBLISHED.txt` | sha256 of every file placed in `dist/`, including `MANIFEST.json` |
| `img/index.json` | Upstream list of image files |
| `img/fire.jpg` | Hero background (735×679 progressive JPEG) |
| `img/og.jpg` | Open Graph preview image (1200×630 JPEG), referenced by the `og:image` meta tag |
| `fonts/index.json` | Upstream list of font files |
| `fonts/rubik-dirt.woff2` | Display face (headings, logo, big numbers) |
| `fonts/space-grotesk.woff2` | Body face, variable weight 300–700 |
| `fonts/space-mono.woff2`, `fonts/space-mono-bold.woff2` | Labels, buttons, numbers (400 and 700) |

All asset URLs inside `index.html` are relative (`fonts/…`, `img/…`), so the export works at a
gateway subpath or an ENS name. There is no framework, no bundler, no lockfile and no
`node_modules`: the page is hand-written HTML and the publisher serves `dist/` as-is.

The page's scripts, meta tags, fonts and external RPC calls are intentional and were left
untouched. At run time the page reads the Kiln contract over public Ethereum JSON-RPC
endpoints and talks to an injected wallet (`window.ethereum`) for transactions. Two
relative links (`../cave-site/`) point at a sibling site that is not part of this mirror.

## Install

Nothing to install. A clone of this repository is the complete site.

```sh
git clone <this repository>
```

## Preview locally

Serve `dist/` with any static file server. Opening `index.html` from the file system also
works for layout, but browsers block `fetch` to the RPC endpoints from `file://` pages, so use a
server to see live chain data.

```sh
python3 -m http.server 8000 --directory dist
# then open http://localhost:8000/
```

With no internet the page still renders completely (hero, copy, swap and Kiln cards, tables,
footer); the live numbers show `—` and the inventory panel shows the fetch error text, which is
the page's own behaviour.

## Rebuild (re-mirror)

There is no build. To refresh the mirror from the source, download the same file set again
and re-verify it. The commands below are the exact procedure used to produce this `dist/`.

```sh
BASE="https://7130de0a-c74f-4dc3-9a8a-79db37963f97-00-1lk87v2ravyz1.reed.replit.dev/zto/kiln-site"
mkdir -p dist/img dist/fonts
for f in index.html MANIFEST.json img/index.json fonts/index.json; do
  curl -sSfL --retry 3 -o "dist/$f" "$BASE/$f"
done
# every file named in img/index.json -> dist/img/, every file in fonts/index.json -> dist/fonts/
for f in $(jq -r '.[]' dist/img/index.json);   do curl -sSfL --retry 3 -o "dist/img/$f"   "$BASE/img/$f";   done
for f in $(jq -r '.[]' dist/fonts/index.json); do curl -sSfL --retry 3 -o "dist/fonts/$f" "$BASE/fonts/$f"; done

# verify every file against the upstream manifest; stop on any mismatch
cd dist
jq -r 'to_entries[] | "\(.value)  \(.key)"' MANIFEST.json | sha256sum -c -
# record what was published
for f in index.html MANIFEST.json img/index.json fonts/index.json $(jq -r '.[]' img/index.json | sed 's#^#img/#') $(jq -r '.[]' fonts/index.json | sed 's#^#fonts/#'); do
  sha256sum "$f"
done > MANIFEST-PUBLISHED.txt
```

If `sha256sum -c` reports anything other than `OK` for every line, do not publish and do not
substitute content: report the mismatching path.

## Verify the committed export

```sh
cd dist && jq -r 'to_entries[] | "\(.value)  \(.key)"' MANIFEST.json | sha256sum -c -
cd dist && sha256sum -c MANIFEST-PUBLISHED.txt
```

## Publish

Upload the contents of `dist/` unchanged to any static host, IPFS gateway or ENS content
hash. Serve `index.html` at the directory root. No server-side rewrites are needed: the page
is a single document with hash anchors (`#trade`, `#kiln`, `#why`, `#how`, `#proof`).

## Validation results (this mirror, 2026-10-09)

| Check | Command / tool | Result |
| --- | --- | --- |
| Download | `curl -sSfL --retry 3` for all 10 files | all 200, no failures |
| Integrity vs upstream | `sha256sum -c` against `MANIFEST.json` (9 entries) | 9/9 `OK`; re-run after writing `MANIFEST-PUBLISHED.txt`, still 9/9 |
| Production build | none exists; the site is hand-written HTML | not applicable |
| Typecheck | none exists; no TypeScript, no package manifest | not applicable |
| Local asset references | grep of `url(`/`src`/`href` in `index.html` vs files in `dist/` | 5 referenced local paths + `og.jpg`, all present |
| Rendered export, desktop 1280×720 | IMD browser tool at `http://127.0.0.1:8899/dist/index.html` | page title, hero image and all 4 font faces loaded (`document.fonts` all `loaded`); no horizontal overflow |
| Rendered export, 390×844 | same | nav links collapse, grids go single column, hero fits (no clipping, stats below CTAs); no horizontal overflow |
| Rendered export, 320×640 | same | no horizontal overflow (`scrollWidth` 320); hero content clips at this height, see `artifacts/validation.md` finding L1 |
| Console / network | browser console and request log | only failures are 3 POSTs to the public RPC hosts (`ERR_INTERNET_DISCONNECTED`, the sandbox browser has no internet); all local resources 200 |
| Interactions | flip direction, Connect wallet (no injected wallet), nav anchor `#how`, keyboard Tab | tokens swap ETH↔ZTO; toast with MetaMask / Coinbase / Rainbow deep links appears; `#how` scrolls into view; default focus ring visible on the focused hero button |
| Better Interface review | six domains, see `artifacts/validation.md` | 7 findings recorded; none applied because the brief forbids any change to the mirrored files |

Limitations: the browser has no internet, so live chain data, the wallet flow and the
external links were not exercised end to end. Native 200% zoom, a real screen reader and a
physical device were not tested. See `artifacts/validation.md` for the full record.
