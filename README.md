# build-to-image-spec

A skill for Claude Code, Codex, and similar agent harnesses: implement a UI screen, page, or component to match a reference image (mockup, screenshot, Figma export, photographic render) with visual fidelity that survives the target's aspect ratio, while keeping every rendered value and affordance honest.

The slogan: **the fidelity is the goal; the honesty is the job.**

## The two truths

- **The image is the fidelity spec.** Color, type, density, polish, finish: match the bar it sets. If the live screen looks dimmer or less crafted than the reference, the implementation is short of the spec.
- **The image's content is not the spec.** Every value, label, panel, and control it shows is illustrative. What ships must be real and working: every value traceable to a live source, every control hooked to a producer that exists. No mockup values transcribed into code, no placeholders fabricated to fill the layout, no stubbed controls that look real but do nothing.

## How it works

1. **Read the image before building.** Measure it, confirm the export scale (a 2x Figma export is half its pixel width in CSS px), sample exact colors instead of eyeballing them, and write down both viewports: the ref viewport (the image's logical size) and the target viewport (the shape the screen actually ships at). They are usually different.
2. **The two-viewport screenshot loop.** Build, run the real app with real data, screenshot the live screen at both viewports, and compare against the scaled reference side-by-side, not as a pixel diff (honest content legitimately diverges from the mockup, so a diff lights up everywhere and tells you nothing). Iterate until both views clear the bar: a screen that is perfect at the ref shape and broken at the target shape is broken.
3. **The omissions audit.** Every element from the reference that didn't make it into the live screen is recorded in `omissions.md` with a reason (`no-producer`, `missing-impl`, `data-divergence`, `aspect-cut`) and a disposition. A clean audit with three honest entries beats a screen with three stubbed affordances.

The loop shells out to `playwright-cli` for screenshots and ImageMagick for measuring and compositing; any equivalent tools work.

Companion to [frontend-design](https://github.com/Ilm-Alan/frontend-design): that skill originates a visual direction when there is no reference; this one matches a reference you were handed.

## Install

### Claude Code

```bash
git clone https://github.com/Ilm-Alan/build-to-image-spec.git ~/.claude/skills/build-to-image-spec
```

### Codex

```bash
git clone https://github.com/Ilm-Alan/build-to-image-spec.git ~/.codex/skills/build-to-image-spec
```

### Gemini CLI

Enable experimental skills in `~/.gemini/settings.json`:

```json
{ "experimental": { "skills": true } }
```

Then clone into `~/.gemini/skills/build-to-image-spec`.

## License

MIT
