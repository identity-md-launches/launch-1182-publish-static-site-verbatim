# DESIGN.md — The Kiln · ZTO

This documents the design system as implemented in the mirrored page `dist/index.html`. Every
value below is read from that file (line numbers refer to it). The page is published verbatim,
so this document describes the upstream author's decisions; it does not propose changes. A
future contributor adding a sibling page should reuse the tokens and classes named here.

## Overview

The Kiln is a single-page dapp front end for a Uniswap v4 pool on the ZTO token, aimed at
ZTO traders and Pepeolithic NFT holders. The visual character is a dark cave at night: near-
black brown backgrounds, ochre and amber text, a torch-lit hero photograph and rising ember
particles. Headings use a distressed display face (Rubik Dirt); body copy is a clean
grotesk; every number, label and button is monospace. Hierarchy comes from face and colour,
not weight: headings are weight 400 and the ochre-hi colour, body is chalk, secondary text is
chalk-dim.

System-wide rules: one centred content column (`.wrap`), section-level grids of "slabs"
(soft panels) and "cards" (brighter, bordered action panels), a four-corner organic radius
on slabs and buttons, and mono uppercase micro-labels above every block. The torch, embers,
sparks and reveal animations are page-wide atmosphere and are all disabled under
`prefers-reduced-motion`.

The hero's full-bleed photograph and the five-stat strip are the landing page's arrangement,
not a rule for every page.

## Colors

Defined as CSS custom properties on `:root` (`dist/index.html:19-24`). Hex notation is the
canonical format; alpha variants are written inline as `rgba()`.

| Token | Value | Job |
| --- | --- | --- |
| `--rock-0` | `#0d0805` | Page background (`body`), nav background at 86% alpha, dark end of inputs and chips |
| `--rock-1` | `#17100a` | Defined, not referenced by any rule |
| `--rock-2` | `#241810` | Defined, not referenced (toast uses the close `rgba(36,24,16,.96)`) |
| `--rock-3` | `#3a2717` | Defined, not referenced (slab gradient uses `rgba(58,39,23,.72)`, the same colour) |
| `--chalk` | `#efe6d6` | Primary text, h3, input text |
| `--chalk-dim` | `#b9aa90` | Secondary text (`.dim`, `.fine`, nav links, stat captions, kv keys, footer) |
| `--ochre` | `#d9a441` | Micro-labels (`.label`), table headers, "max" button |
| `--ochre-hi` | `#f2c462` | Headings, links, logo, big numbers, primary button fill, selected tier row |
| `--amber` | `#f5a623` | Only the arrow in the logo (`.logo b`) and the favicon |
| `--ember` | `#ff8a3d` | Accent word in the h1, slab titles in the "What this is" section, rule numerals, hot pill |
| `--red` | `#b5482a` | Selected chip / selected inventory button background |
| `--red-hi` | `#d4633d` | Danger-ish primary button (`.btn.red`, used for "Sell to the Kiln"), live dot, selected tile ring |
| `--char` | `#15100c` | Text on the ochre-hi primary button |
| `--moss` | `#a5d78a` | Defined, not referenced |
| `--slab` | `20px 28px 22px 30px / 28px 20px 30px 22px` | Organic border radius for slabs and cards |

Literal colours used without a token: `#fff3e6` (text on red backgrounds), `#ff8a65` (error
text `.err`, line 117), `#8a6420` and `#6d2412` (3px bottom "shadow" edges of buttons, lines
57 and 60), `#1a110a` (hero and tile placeholder background), `#ffb65a` / `#ffc46a` (ember and
spark particles).

Borders are ochre at low alpha: `rgba(217,164,65,.14)` for dividers (nav, table rows,
footer), `.17` for slabs, `.2` for inputs, `.35`–`.45` for interactive chips and cards.

Contrast ratios computed from the solid source values (WCAG 2.x formula, see
`artifacts/validation.md` for method and limits):

| Pair | Ratio |
| --- | --- |
| chalk on rock-0 (body) | 16.09 |
| chalk-dim on rock-0 (secondary) | 8.74 |
| ochre on rock-0 (labels) | 8.86 |
| ochre-hi on rock-0 (headings, links) | 12.19 |
| ember on rock-0 | 8.50 |
| char on ochre-hi (primary button) | 11.56 |
| `#fff3e6` on red-hi (`.btn.red`) | 3.40 |
| `#fff3e6` on red (selected chip / tile button) | 4.90 |

Text over the slab and card gradients and over the hero photograph sits on translucent or
image backgrounds; those ratios were not measured.

There is one theme. Do not add a light theme or new accent colours; pick from the table above.

## Typography

Four local font files, all loaded with `@font-face` and `font-display: swap`
(`dist/index.html:14-17`); all four were confirmed loaded in the browser.

| Family | File | Weights supplied | Role |
| --- | --- | --- | --- |
| Rubik Dirt | `fonts/rubik-dirt.woff2` | 400 only | h1, h2, logo, stat numbers, tile numbers, `.why .slab b`, rule numerals |
| Space Grotesk | `fonts/space-grotesk.woff2` | variable 300–700 | body, `.lede`, `.big`, h3 (700), hero `b` (500) |
| Space Mono | `fonts/space-mono.woff2` | 400 | labels, nav links, captions, tables, kv lists, inputs, `.fine`, `.err`, toast |
| Space Mono | `fonts/space-mono-bold.woff2` | 700 | buttons, chips, pills, tile captions, `.tok` |

Fallback stacks: `"Space Grotesk", system-ui, sans-serif`; `"Space Mono", ui-monospace,
monospace`; `"Rubik Dirt", "Space Grotesk", sans-serif`.

Scale and roles:

- Body: `17px / 1.55` Space Grotesk (`body`, line 27). `.lede` is `1.16rem`, max measure
  `56ch`. `.why .big` is `clamp(1.25rem, 2.4vw, 1.7rem) / 1.35`, max `46ch`.
- h1 (hero only): `clamp(2.8rem, 10.4vw, 8.4rem) / .92`, ochre-hi, with a hard text shadow;
  one `span` in ember.
- h2: `clamp(2rem, 5.2vw, 3.5rem) / 1.02`, ochre-hi, letter-spacing `.01em`.
- h3: `700 1.1rem / 1.25` Space Grotesk, chalk.
- `.label` (micro-label): `400 11px / 1.4` Space Mono, uppercase, letter-spacing `.18em`,
  ochre. Nav links and stat captions use the same style at `.14em` in chalk-dim.
- Numbers: Rubik Dirt `30px` in the hero stats, `2.4rem` in tiles; kv values are
  Space Mono 13px with `overflow-wrap: anywhere` so addresses and pool ids wrap.
- Fine print: `.fine` `10px / 1.5` Space Mono; errors `.err` 12px.

Headings are weight 400 by design (`h1, h2, h3 { font-weight: 400 }`, line 34); only h3 and
mono buttons are bold. No OpenType features or variable axes beyond weight are used.

## Layout

- Content column: `.wrap { width: min(1120px, calc(100% - 32px)); margin: 0 auto }`
  (line 31). Every nav, hero, section and footer block uses it.
- Sections: `section { padding: 84px 0 12px }`; the "What this is" section overrides to
  `padding-top: 40px`. Each section opens with `.head` (a 12px-gap grid of label, h2, lede)
  followed by a `.grid`.
- Grids: `.grid { gap: 16px }` with `.g2` (two equal columns) and `.g3` (three). The
  "What this is" grid is four columns. `.tiles` is `repeat(auto-fill, minmax(118px, 1fr))`
  with 10px gap for NFT tiles. `.kv` is a two-column key/value grid (`1fr auto`, gap
  `6px 14px`). `.rule` is `28px 1fr`.
- Hero: `height: min(calc(100vh - 56px), 860px); min-height: 620px; overflow: hidden`,
  content vertically centred with `padding-bottom: 90px`, stats absolutely positioned
  26px from the bottom (lines 67-84).
- Nav: sticky, 56px tall, blurred `rgba(13,8,5,.86)` background, z-index 50.
- Breakpoints (lines 145 and 162), a single `max-width: 900px` query:
  `.g2`, `.g3` and the footer collapse to one column; the "What this is" grid goes to two
  columns; `nav .links` is hidden (the logo and the wallet button remain; there is no menu
  button, section navigation on narrow screens is by scrolling and the hero CTAs); hero
  `padding-bottom` grows to 150px so the stats strip clears the CTAs.
- Observed in the browser: no horizontal overflow at 1280, 390 or 320 CSS px. At 390×844 the
  hero fits. At 320×640 the hero's centred content is taller than the hero box, so the top
  micro-label is hidden under the nav and the stats strip overlaps the CTA area (hero
  `overflow: hidden`). Widths between 390 and 900 were not checked.
- `body { overflow-x: hidden }` guards against the ember particles, which can extend a few
  pixels past the viewport inside the hero.

## Elevation & Depth

Three tonal layers plus shadows:

1. Page: `--rock-0` with a faint amber radial glow at the top (`body` background-image) and a
   fixed pointer-following glow (`.glow`, line 45) at z-index 0.
2. Slab (`.slab`, line 86): a 160° gradient from `rgba(58,39,23,.72)` to `rgba(30,20,12,.82)`,
   1px ochre border at `.17`, inset highlight `inset 0 1px 0 rgba(255,220,160,.06)` and drop
   shadow `0 14px 40px rgba(0,0,0,.35)`. Informational panels.
3. Card (`.card`, line 100): a brighter gradient `rgba(44,28,16,.93)` to `rgba(16,10,6,.95)`,
   1px ochre-hi border at `.45`, ring `0 0 0 1px rgba(0,0,0,.4)` and `0 18px 50px rgba(0,0,0,.5)`.
   Action panels (swap, sell, buy).

Buttons carry a 3px solid bottom edge plus a soft drop shadow (`.btn`, line 57) and press
down 2px on `:active`. Inputs (`.field`) sit one step darker than their card. The toast is a
fixed bottom-centre panel at z-index 90; spark particles are z-index 95; the sticky nav is 50.

## Shapes

- Organic four-corner radius `var(--slab)` on slabs and cards; the button variant is
  `14px 18px 14px 20px / 18px 14px 20px 14px` (`.btn`, `.tile2`).
- Regular radii: 14px for inputs and the toast, 12px for chips, 10px for inventory buttons,
  8px for pills, 6px for the tiny "OS" badge, 50% for the flip button and the live dot.
- 1px borders throughout; no 2px borders except the selected-tile ring (`box-shadow`).
- Tiles are square (`aspect-ratio: 1`) with the image clipped by `overflow: hidden`.

## Components

All components are plain HTML with class names in the inline stylesheet; there is no
component library or build. Line numbers are in `dist/index.html`.

- **Micro-label `.label`** (line 33): uppercase mono caption that heads every section and
  panel. Put one as the first child of a `.head`, `.slab` or `.card .top`.
- **Button `.btn`** (lines 57-62): primary is ochre-hi fill with char text. Variants:
  `.ghost` (translucent dark fill, 1px chalk ring), `.red` (red-hi fill, for selling to the
  Kiln), `.sm` (12px, tighter padding). Works as `<a>` or `<button>`. States: hover brightens
  8%, active scales to .96 and drops 1px, `[disabled]` is 55% opacity with `not-allowed`
  cursor. Clicking any `.btn` also fires decorative sparks unless reduced motion is set.
- **Chip `.chip` / wallet chip `.wchip`** (lines 63-65): small mono toggles. `.chip.on` is
  red. `.wchip` is the nav "Connect wallet" control and shows the short address and
  Pepeolithic count when connected.
- **Pill `.pill`** (line 160): 10px uppercase status tag; `.hot` turns ember when the wallet
  holds a pass.
- **Slab `.slab`** and **Card `.card`** (lines 86, 100): see Elevation. A card's first row is
  `.top` (flex, space-between) holding a `.label` with an optional live `.dot` and a `.pill`.
- **Live dot `.dot`** (line 102): 8px red-hi circle with a pulse animation; `.off` is
  chalk-dim and static. Indicates whether the last chain read succeeded.
- **Amount field `.field`** (lines 105-110): grid of `input` + `.tok` token name, with an
  optional full-width `.bal` balance line containing a "max" button. Input is 1.5rem mono,
  no border, and `outline: none` (see validation finding A1). Placeholder is the only label.
- **Flip button `.flip button`** (line 112): 36px circle with the ⇅ glyph; swaps the swap
  direction and clears both fields.
- **Fee summary `.fee`** (lines 113-116): stacked rows of `span` + `b`; `.you` row is
  highlighted in ochre-hi.
- **Key/value list `.kv`** (lines 123-125): label in chalk-dim, value right-aligned in
  ochre-hi, values wrap anywhere; used for pool stats and contract addresses.
- **Tier table `table.tiers`** (lines 94-98): 13px mono, ochre uppercase headers, last column
  right-aligned; `tr.me` highlights the connected wallet's tier.
- **NFT tile `.tile2`** (lines 134-144, generated by the `tile()` function at line 511):
  a `button` with a square `.pic` (shows a `?` until the image loads, then the image), a
  `.cap` row with the id and price, and an "OS" link to OpenSea. `.sel` adds a red-hi ring;
  hover lifts 3px. Selecting one sets the sell or buy target.
- **Stats strip `.stats`** (lines 81-84): caption + Rubik Dirt number pairs; numbers get a
  `.pop` animation when a value changes.
- **Rule `.rule`** (lines 146-147): numbered list item with a Rubik Dirt numeral in ember.
- **Toast `#toast`** (lines 151-152): `role="status"`, bottom-centre, shown by adding
  `.show`; auto-hides after 3.2s (longer for wallet and transaction messages).
- **Reveal `.reveal`** (lines 43-44): sections fade and rise in on intersection; a 1.8s
  fallback timer reveals everything so content is never left hidden.

Keyboard: all actions are native `button` or `a` elements and reach focus in DOM order; the
page relies on the browser's default focus ring (no custom `:focus-visible` style), and the
amount inputs remove it.

## Do's and Don'ts

- Start a new page with `nav`, then `section > .wrap > .head (.label, h2, .lede) + .grid`.
  Reuse `.slab` for explanation and `.card` for anything the user acts on.
- Use `.btn` for the one primary action in a card, `.btn.ghost` for secondary actions and
  `.btn.red` only for selling into the fire. Do not invent a third button colour.
- Colour comes from the `:root` tokens; new text is `--chalk`, secondary is `--chalk-dim`,
  emphasis is `--ochre-hi`, a single accent word may be `--ember`.
- Keep headings at weight 400 in Rubik Dirt; do not bold them. Keep all numbers in Space
  Mono or Rubik Dirt so columns stay stable.
- Keep the single 900px breakpoint. Any new multi-column grid needs a rule in the existing
  `@media (max-width: 900px)` block collapsing it.
- Respect `prefers-reduced-motion`: any new motion must check the `still` flag (line 571)
  as the existing sparks, embers and pops do.
- Do not add a light theme, a cookie banner, analytics or a build step; the site is a
  single hand-written document served as-is.
- Recipe for one more page: copy `index.html`, keep the `head` (fonts, tokens, stylesheet)
  and the `nav`/`footer`, replace the sections, keep asset paths relative (`fonts/`, `img/`).
