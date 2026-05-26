---
name: build-to-image-spec
description: |
  Implement a UI screen, page, or component to match a reference image — mockup, screenshot, or photographic render — with visual fidelity that survives the target's aspect ratio, while keeping every rendered value and every affordance honest: real data and working features only, never mockup values transcribed into code, placeholders fabricated to fill the layout, or stubbed controls that look real but do nothing.
  TRIGGER when: user hands you a design image, mockup, screenshot, photographic reference, Figma frame, or exported comp and asks to build, implement, match, recreate, or "make it look like" it.
  SKIP: originating a visual direction with no reference image (use the frontend-design skill); pure copy or content edits; backend-only work with no screen to match.
---

# Build to image spec

You've been handed a reference image — a mockup, screenshot, or photographic
render of a screen — and asked to build it. Two things are true at once, and
they do not conflict:

- **The image is the fidelity spec.** Visual quality, polish, density, mood,
  color, type, edge treatment, finish: match the bar it sets. If the live
  screen looks dimmer, blurrier, more cluttered, or less crafted than the
  reference, the implementation is short of the spec.
- **The image's content is not the spec.** Every value, label, panel, and
  affordance the reference shows is illustrative. What ships must be *real*
  and *working*: every value traceable to a live source, every control
  hooked to a producer that exists. Mockup content never silently becomes
  product, and a stubbed affordance never ships to fill the layout.

The slogan: **the fidelity is the goal; the honesty is the job.**

**Photographic and polished references teach the quality bar, not a feature
inventory.** A glossy render tells you what "good" looks like — the color,
the materials, the light, the texture, the typographic care, the finish.
It does not tell you what features to ship. Read the level of polish into
the implementation; do not read in the features that have no producer yet.
That distinction is the entire skill: copy the fidelity, not the content.

A reference image overrides design-direction work. It has already chosen
the palette, type, layout, and finish — do not also pick a visual anchor
or reach for an "unexpected" direction. Match what you were handed. The
content discipline from the frontend-design skill still applies in full.

## 1. Read the image before you build

Don't build from a glance. Inventory the reference first:

- **Save it to disk** at a known path so you can measure it and re-view it.
- **Get its dimensions and scale.** `sips -g pixelWidth -g pixelHeight ref.png`
  (or `magick identify ref.png`). A mockup exported at 2× — the common Figma
  default — is twice its logical size: a 2880-px-wide image is a **1440-px
  CSS viewport**. Confirm the scale; never assume the pixel width is the
  viewport.
- **Decide both viewports.** The **ref viewport** is the image's logical
  size. The **target viewport** is the shape the screen actually ships at:
  the responsive width users will land on, the card the panel renders
  inside, the modal the content lives in. They are usually different.
  Write both down before you build — they drive the screenshot loop.
- **Inventory the regions** — rail, header, panels, cards, columns, footer —
  and the component type of each.
- **Extract values, don't eyeball them.** Sample exact colors from the image
  (`magick ref.png -crop 1x1+X+Y txt:-`); measure spacing and type sizes in
  pixels. Then map them to the codebase's existing design tokens.

## 2. The loop

Building blind and declaring it done is the failure mode. Work the loop:

1. **Build** the screen. Use the codebase's existing components and design
   tokens wherever they already fit — the image specifies *fidelity and
   structure*, the codebase still owns *implementation*. Author one-off
   styles only for genuinely new structure, never to bypass the design
   system.
2. **Run the app and reach the real screen** — real auth, real seed data,
   real routing. Screenshotting a static HTML stub defeats the honesty
   check; the point is to see what real data does to the layout.
3. **Screenshot at both viewports.** Shoot the live screen at the ref
   viewport (`playwright-cli resize <ref-w> <ref-h>` → `screenshot
   --filename=live-ref.png`), then resize to the target viewport and shoot
   `live-target.png`.
4. **Ref-shape compare side-by-side.** Scale the reference to the
   screenshot's width and append:
   - `magick ref.png -resize <live-width>x scaled-ref.png`
   - `magick scaled-ref.png live-ref.png +append compare-ref.png`

   Read `compare-ref.png`. Side-by-side, **not** a pixel diff: honest
   content legitimately diverges from the mockup, so a diff lights up
   everywhere and tells you nothing. Your eye separates *fidelity* drift
   (a bug) from *content* drift (correct).
5. **Target-shape inspect.** Open `live-target.png` on its own. Does the
   polish, density, and hierarchy that read in the ref-shape compare still
   read here? If the ref-shape view passes but the target-shape view feels
   cramped, hollow, or off, the spirit didn't survive the resize — rework
   spacing, type scale, or composition until it does.
6. **Fix** every fidelity, structure, spacing, or treatment difference
   either view surfaces. Expect and ignore content differences — those
   are correct.
7. **Repeat** until both views clear the bar. Done is defined by the two
   views, not by "it looks plausible." A screen that's perfect at the ref
   shape and broken at the target shape is broken.

## 3. Honest data and features

Every string, number, label, and *control* on the live screen must name
real information or hook to a working producer. The reference's content is
a placeholder — never transcribe it into code, never add an affordance
that matches the ref but doesn't actually do anything.

Where the reference shows a value, panel, or control you have no real
source for:

- **Cut it from this slice.** A panel with no real data does not ship; a
  button with no handler does not ship; a counter with no producer does
  not ship. Render an honest-empty state, a `—`, or leave the region out
  of the composition.
- **Honesty outranks visual fidelity.** A blank list where the mockup
  shows five rows, a `—` where it shows "$4,200", three real items where
  it shows eight, a missing "Recent activity" panel where the ref had
  one — all correct if that is the true state.
- **Never stub.** A control that looks real but does nothing is the
  worst outcome: it lies twice, about the affordance and about the
  state behind it. If it can't ship working with real data, it doesn't
  ship.

Every cut is recorded in `omissions.md` (see §5).

## 4. One frame is not the whole screen

The reference is a single state at a single size. Derive the rest
deliberately: loading, empty, error, hover/focus, and responsive behavior
above and below the reference viewport. The **empty state** especially —
honest data will reach it, and the reference never shows it.

## 5. The omissions audit

`omissions.md` is the deliverable that proves the screen is honest. Not a
planning file, not a TODO list — an audit of every element from the
reference that did not make it into the live screen, with the reason. The
discipline is: a clean audit with three honest entries beats a screen with
three stubbed affordances.

One row per omission:

- **What** — the element from the ref (e.g., "Recent activity panel,"
  "Streak counter," "Eight-row activity list," "$4,200 'projected refund'
  card").
- **Why** — one of:
  - `no-producer` — nothing in the system produces this value; not in
    scope for this slice.
  - `missing-impl` — a producer should exist; flagged for a follow-up to
    build it.
  - `data-divergence` — the producer exists and emitted a different value
    (the honest one). The element is present; only the content differs.
  - `aspect-cut` — the element fit the ref viewport but not the target
    viewport; deliberately removed so the spirit survives.
- **Disposition** — named follow-up, accepted as-is, or re-evaluate when
  the target viewport changes.

When the reference's content genuinely all maps to real producers, the
file still exists with a single line stating that. The audit is the
record that the question was asked.

## Don't

- Don't transcribe mockup content — names, dollar amounts, counts, dates,
  panels, affordances — into the code.
- Don't stub controls that look real but don't work. No dead buttons, no
  fake counters, no chrome that pretends a system did something.
- Don't silently invent a backend producer to make a number appear. That
  is an omission entry or a separate slice, not a fix.
- Don't fork one-off styles when an existing component or token already
  matches the image.
- Don't accept "close enough" on fidelity. Iterate the loop until both
  views clear the bar.
- Don't skip the target-shape view because the ref-shape view passes. A
  screen that polishes at the ref viewport and breaks at the target
  viewport is broken.
- Don't skip the project's own gates (lint, typecheck, tests). A matching
  screenshot is not a passing build.

## Many screens

For a set of screens, build the shared substrate first — the shell, layout,
navigation, shared components — and verify it against one screen at both
viewports before building the rest on top. Each individual screen still
runs its own two-shape loop and keeps its own `omissions.md`.
