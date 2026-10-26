---
name: remotion-effects
description: Library of 22 tested Remotion (React video) effects and transitions — orbit, frame break, green screen, mirror, UI world, paper cut, cover story, triptych, doodle boil, screen wall, spotlight, glass type, checklist, cinemascope, headline wall, chart grow, lower third, zoom-through cut, kinetic captions, pull-back reveal, freeze frame, whip pan. Load when asked to make a video with Remotion, or when a request names one of these effects (in any language, e.g. "orbit efekti", "yeşil ekran", "altyazı efekti").
---

# Remotion effects

A catalog of 22 motion-graphics effects as self-contained Remotion components. Each reference file has a description, when to use it, a usage example, notes and the full component source. All components were type-checked and rendered with Remotion 4.0.538 at 1920x1080, 30 fps.

## How to use this skill

1. Match the request to an effect in the table below. Users may use the number ("efekt 14"), the English name or a description ("siyah sinema bantları" → Cinemascope).
2. Read **only** the matching reference file(s) in `references/`.
3. Copy the component code into the user's Remotion project as `src/effects/<Component>.tsx`, unchanged unless the user asks for changes.
4. Use it in a composition as shown in the reference's Usage section. Replace placeholder assets with the user's files from `public/` via `staticFile()`.
5. Render a still at a frame where the effect is visible and look at it before saying it works (see "Verify" below).

Several effects combine well. For example, Cinemascope + LowerThird, GreenScreen + UiWorld, or KineticCaptions on top of anything. Nest them: each component fills its parent with `AbsoluteFill`.

## Effects

| #   | Effect           | Component        | Type       | Reference                                                              |
| --- | ---------------- | ---------------- | ---------- | ---------------------------------------------------------------------- |
| 01  | Orbit            | `Orbit`          | Layout     | [01-orbit.md](references/01-orbit.md)                                   |
| 02  | Frame break      | `FrameBreak`     | Reveal     | [02-frame-break.md](references/02-frame-break.md)                       |
| 03  | Green screen     | `GreenScreen`    | Compositing| [03-green-screen.md](references/03-green-screen.md)                     |
| 04  | Mirror           | `Mirror`         | Look       | [04-mirror.md](references/04-mirror.md)                                 |
| 05  | UI world         | `UiWorld`        | 3D scene   | [05-ui-world.md](references/05-ui-world.md)                             |
| 06  | Paper cut        | `PaperCut`       | Scene      | [06-paper-cut.md](references/06-paper-cut.md)                           |
| 07  | Cover story      | `CoverStory`     | Layout     | [07-cover-story.md](references/07-cover-story.md)                       |
| 08  | Triptych         | `Triptych`       | Layout     | [08-triptych.md](references/08-triptych.md)                             |
| 09  | Doodle boil      | `DoodleBoil`     | Overlay    | [09-doodle-boil.md](references/09-doodle-boil.md)                       |
| 10  | Screen wall      | `ScreenWall`     | Scene      | [10-screen-wall.md](references/10-screen-wall.md)                       |
| 11  | Spotlight        | `Spotlight`      | Overlay    | [11-spotlight.md](references/11-spotlight.md)                           |
| 12  | Glass type       | `GlassType`      | Title      | [12-glass-type.md](references/12-glass-type.md)                         |
| 13  | Checklist        | `Checklist`      | Infographic| [13-checklist.md](references/13-checklist.md)                           |
| 14  | Cinemascope      | `Cinemascope`    | Look       | [14-cinemascope.md](references/14-cinemascope.md)                       |
| 15  | Headline wall    | `HeadlineWall`   | Scene      | [15-headline-wall.md](references/15-headline-wall.md)                   |
| 16  | Chart grow       | `ChartGrow`      | Infographic| [16-chart-grow.md](references/16-chart-grow.md)                         |
| 17  | Lower third      | `LowerThird`     | Title      | [17-lower-third.md](references/17-lower-third.md)                       |
| 18  | Zoom-through cut | `ZoomThroughCut` | Transition | [18-zoom-through-cut.md](references/18-zoom-through-cut.md)             |
| 19  | Kinetic captions | `KineticCaptions`| Title      | [19-kinetic-captions.md](references/19-kinetic-captions.md)             |
| 20  | Pull-back reveal | `PullBackReveal` | Camera     | [20-pull-back-reveal.md](references/20-pull-back-reveal.md)             |
| 21  | Freeze frame     | `FreezeFrame`    | Moment     | [21-freeze-frame.md](references/21-freeze-frame.md)                     |
| 22  | Whip pan         | `WhipPan`        | Transition | [22-whip-pan.md](references/22-whip-pan.md)                             |

## Project setup

If the user has no Remotion project yet:

```sh
npx create-video@latest --blank my-video
cd my-video
npm run dev
```

The components only import from `remotion` and `react`. No other packages are needed. Put user media in `public/` and reference it with `staticFile("name.mp4")`.

Register a composition in `src/Root.tsx`:

```tsx
<Composition
	id="Intro"
	component={Intro}
	durationInFrames={150}
	fps={30}
	width={1920}
	height={1080}
/>
```

## Conventions all components follow

- **Deterministic.** Everything is driven by `useCurrentFrame()`. No `Math.random()`, no CSS transitions or `@keyframes`, no `setTimeout`. Use `random("seed")` from `remotion` for randomness. Breaking this causes flicker, because Remotion renders frames in parallel.
- **Timing in frames.** Props like `stagger`, `breakAt` and `cutAt` are frame numbers relative to the component's own start. Wrap a component in `<Sequence from={...}>` to place it on the timeline.
- **Motion.** `spring({ frame, fps, config: { damping: 200 } })` for smooth settles, lower damping (9-15) for bouncy pops, and `interpolate` with `Easing` for linear-timed moves. Always clamp with `extrapolateLeft/Right: "clamp"` unless overshoot is wanted.
- **Media.** Use `<OffthreadVideo>` for video and `<Img>` for images, so Remotion waits for them to load before rendering a frame.
- **SVG filter ids** come from `useId()`, so the same effect can be used several times on one frame.
- **Resolution.** Sizes assume 1920x1080. For vertical 1080x1920, scale font sizes and radii and recheck the layout with a still.

## Verify

Render a still at a frame where the effect is in progress, then view the image:

```sh
npx remotion still src/index.ts Intro out/check.png --frame=45
```

For transitions, check a frame before, during and after the cut. In a sandbox without Remotion's downloaded browser, pass `--browser-executable` pointing to an installed Chrome or Chromium headless shell.

Render the final video with:

```sh
npx remotion render src/index.ts Intro out/intro.mp4
```

## Content rules

- Do not reproduce real brands, magazine mastheads or news outlets with made-up content (Cover story, Headline wall). Use invented names or accurately quoted real material.
- Effects that need a cutout (Frame break, Cover story) need a transparent PNG prepared outside Remotion. Ask the user for it instead of faking it.
