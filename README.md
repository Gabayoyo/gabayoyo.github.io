# gabayoyo.github.io

Personal site for Aaron Gabayoyo: a short index of compiler and systems projects.

Served by GitHub Pages from the repository root. The entire site is one
self-contained `index.html` — no build step, no package manager, no framework.
Open the file in a browser and it works; push it and it deploys.

## Contents

- `index.html` — the entire site (markup, styles, and script inline)
- `references/` — style presets and section templates consulted while designing the page

## Previewing locally

Open `index.html` directly, or serve the directory to match how Pages resolves paths:

```sh
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deploying

Pages is configured under **Settings → Pages**, source set to the `main` branch
and `/ (root)`. Any commit to `main` that changes `index.html` redeploys the site;
there is nothing to run or build. Changes to `README.md` do not affect the site.

## Editing

Each project is one `<article class="card">` holding a heading, a description and
an optional status badge, so adding or reordering entries needs no tooling.

The five main cards split that description into a summary and a details paragraph,
divided by a rule under the summary: the summary is one sentence stating what the
project is, and everything below the rule is the depth, where the mechanism, the
comparison or the test coverage belongs. A reader scanning the column reads only the
summaries, so nothing important to that scan may live below the rule. Their heading
carries no rule of its own, so the card has one internal line and it marks the
boundary that matters.

The two cards in Other repositories keep a single short paragraph and take the rule
from the other end: a hairline under the heading, splitting title from content, since
a one-line summary above a one-line detail would be divided by a rule separating
nothing. Either way, one line per card.

Two cards currently carry `<span class="wip">In progress</span>` —
BiggerBrotherWebApp and riscv-pipeline-simulator. The badge sets
`pointer-events: none`, because it sits over the card-wide link overlay and would
otherwise swallow clicks meant for the repository. There is no separate "Repository" link: the heading anchor stretches
over the whole card via `::after`, so clicking anywhere in it opens the
repository. Hovering or focusing a card gives three cues and no others: a green
bar draws down its left edge, the title darkens and underlines, and the card lifts
with its border tinted to the accent. The fill never changes.

Descriptions are deliberately short — roughly 20–90 words per card, ~360 total.
Each one aims to say what the project is, the one mechanism worth knowing, and any
concrete result, test coverage or CI it carries. Detailed walkthroughs belong in
each repository's own README, not here; the card is for someone scanning seven
projects in half a minute.

Testing and CI appear only where they are part of what the project is — mlir-match
states that its lit suite is what checks the verifier and both lowerings agree;
MLIR-MRacle, being a testing harness, says what it ships to fuzz with. Do not add
test or CI coverage as a credential. A line like "CI runs under ASan and UBSan on
every push" is verifiable but reads as padding, and it weakens the sentence around
it. If a claim cannot be written as a natural part of the description, leave it out.
Everything stated on the cards should still be checkable against the repositories.

Repo titles use `--title-green` rather than the accent, darkening to
`--title-green-hover`. They carry no underline: the accent bar, the card's lift and
the title's colour change all signal that the whole card is the link. The section
headings carry no underline either — only the 3px rule that fades out to their
right. The card gradient's darkest stop is `#e8f1ea`, and the
title has to clear 4.5:1 against it — a lighter green does not, which is why these
two tokens are darker than the accent used elsewhere. The bar occupies
`var(--card-edge)` of the card's left padding, so shrinking one without the other
will push the bar over the text.

The cards are flat. There is no gradient or tint on them, and adding one back is
not straightforward: an earlier attempt shifted 14 luminance points against only 9
of chroma, which reads as a grey shadow rather than colour, and an angled one
resolves its stops against card height so short cards came out darker than tall
ones. The card's colour now comes from three flat devices instead — the accent bar
down the left edge (always visible, saturating and stretching on hover), the green
title, and the soft hairline under the title separating it from the description.

## Design tokens

Palette, type scale and spacing are custom properties in the `:root` block at the
top of the inline stylesheet. Type is set by `--font-heading` (Poppins),
`--font-body` (Nunito Sans) and `--font-mono` (JetBrains Mono). Monospace is used
for inline `code` only, where the content is a technical identifier; UI text such
as the contact links and status badges uses the body font.

## Background

`.backdrop` is `position: fixed` and paints three full-bleed `linear-gradient`
bands over the paper colour, with a faint grain layer on `.backdrop::after`. The
masthead carries its own two, and closes with a 2px `border-image` gradient rule
rather than an inset shadow or a blurred drop shadow — both of those read as a
smudge under the header at this thickness. The rule beside each section heading is
a 3px gradient that fades to fully transparent at its right end. It
holds at any viewport size because nothing in it is a positioned element, a
repeating 1px pattern, or a large promoted layer.

Three earlier approaches are worth not repeating:

- A `radial-gradient` dot grid aliases into visible streaks once it lands on
  fractional device pixels.
- Positioned blotch elements need a document-height container, and that container
  combined with `will-change: transform` and a scale is promoted to a GPU layer
  larger than the maximum texture size — the browser then truncates and tiles it
  and the colour visibly cuts off on wide screens.
- A hairline column in the backdrop is, by definition, a line.

Animating `background-position` gives the same slow drift as scrolling, without
any of those problems. The
grain is not decoration: it dithers the wash, which would otherwise band in 8-bit
colour because its entire tint range is only a handful of levels.

## Design constraints

These are not preferences — each one replaced something that measurably looked
wrong, so keep them unless you are changing the thing they protect:

- **Keep the card gradient horizontal.** An angled gradient resolves its stops
  against card height too, so short cards compress the same tint and read darker.
- **No `will-change`, and nothing larger than a texture budget.** Promoting the
  backdrop produced a layer past the GPU texture limit, which the browser then
  truncated and tiled — visibly cutting the colour off on wide screens.
- **No repeating 1px patterns.** A dotted `radial-gradient` grid aliases into
  streaks at fractional device pixels.
- **Grain is load-bearing.** The backdrop wash spans only a few levels of
  luminance and bands without it.
- **Three fonts, three jobs.** Poppins for headings, Nunito Sans for text,
  JetBrains Mono for inline `code` only. Adding a fourth face for flavour is what
  makes a page look generated.
- **Text blocks trim their line boxes.** The name, the tagline and the intro trim
  their first and last line boxes to cap and baseline, so the band's padding and
  the centred prose measure the glyphs rather than the leading each line height
  carries. Without the trim the name sits lower than the padding implies; that is
  the fallback on browsers without `text-box-trim`, not a layout bug to chase with
  padding. Do not re-add a padding compensation alongside it, since the two would
  then fight and overshoot in the other direction.
- **Interactive targets clear 44px.** Card titles look like 31px text but their
  real target is the whole card, via the `::after` overlay. The skip link and the
  contact links are padded to reach it directly.

  That padding has a consequence worth understanding before changing it: a 44px
  target around ~15px text needs roughly 15px of padding a side, so the two contact
  links' text necessarily sits ~46px apart. Shrinking the padding makes the pair
  look tighter but drops the target to ~35px, which is too small to tap reliably.
  The compromise here is a `--accent-wash` fill on hover, which shows that the
  space between the two links belongs to the target rather than being a gap.

## Content notes

The masthead is an identity strip rather than a hero: the name with a short tagline
under it, and the contact links opposite. It is deliberately shallow, so the page opens
with a name and gets on with the work.

The intro paragraph follows the band in the page column, centred and set a step larger
than body text, with no section heading over it: it carries the opening of the page that
a heading would otherwise announce. Its measure is capped so the line length stays
readable rather than running the full card width.

Availability is the last line of that paragraph rather than a separate block, so the two
can never drift apart at any width. It is not a badge, banner or coloured chip, and not
above the work. The projects are the argument; availability is a detail a reader finds.
The wording covers both internships and graduate roles in ordinary language rather than
naming a scheme, and pairs the fact with an invitation ("happy to talk compilers") so it
reads as a standing offer rather than a request.

The intro states a specialisation with adjacent interests, and frames tools as experience with
them rather than presenting them as an inventory. It carries no skills list: the projects are the
evidence for what the author can do, and naming tools in the abstract would undercut work that
already demonstrates them. Interests sit as plain likes rather than defended as connected, since
a one-line justification for each reads as a plea.

Do not add emphasis to that paragraph, including its availability line. It was
previously bolded inside the intro and read as a claim about the page rather than a
statement about the reader's options.

The footer is a full-bleed deep sage end-cap, deliberately outside `.page` so it runs
the full window width. It carries the name, contact links and a back-to-top link on a
single centred row, with no closing copy and no rule above it.

Type scales are nested so they never collide: section heading > primary card title >
secondary card title, at every width. Check that ordering if you change any of the
three clamps - they cross over easily, because each mixes a `rem` floor with a `vw`
slope and a `rem` ceiling.

## Accessibility

The page is readable with JavaScript disabled — the scroll fade is scoped to a
`js` class on the root element, so without scripting every card renders at full
opacity. `prefers-reduced-motion` disables the fade, the hover transitions and the
backdrop drift. Text on the darkest background the page can produce stays above
the WCAG AA 4.5:1 threshold.
