# 09 Doodle boil

**Component:** `DoodleBoil` · **File:** `src/effects/DoodleBoil.tsx`

Hand-drawn "line boil": strokes draw themselves on, then keep jittering every few frames like frame-by-frame animation. A turbulence displacement filter gets a new seed every `hold` frames.

**Use it for:** Hand-drawn annotations, circles and arrows over footage, sketchbook or whiteboard explainers, playful titles.

## Usage

```tsx
<AbsoluteFill>
  <OffthreadVideo src={staticFile("footage.mp4")} />
  <DoodleBoil hold={3} amount={6}>
    <circle cx={1200} cy={420} r={150} fill="none" stroke="#ff3b30" strokeWidth={10} pathLength={1} />
    <path d="M800 800 Q950 650 1080 520" fill="none" stroke="#ff3b30" strokeWidth={10} strokeLinecap="round" pathLength={1} />
  </DoodleBoil>
</AbsoluteFill>
```

## Notes

- Every stroke needs `pathLength={1}` for the draw-on to work. Set `drawOn={0}` to skip the draw-on.
- `hold` 2-3 looks like classic animation ("on twos"). Using 1 jitters every frame and looks noisy.
- SVG `<text>` also boils, which is useful for handwritten-style titles.

## Code

```tsx
import React, { useId } from "react";
import { AbsoluteFill, interpolate, useCurrentFrame } from "remotion";

export type DoodleBoilProps = {
  /** SVG children (paths, text...) drawn in a 1920x1080 viewBox. */
  children: React.ReactNode;
  /** Change the wobble every N frames. 2-3 feels hand-animated ("on twos"). */
  hold?: number;
  /** Wobble amount in px. */
  amount?: number;
  /** Draw-on duration in frames (strokes need pathLength={1}). 0 = off. */
  drawOn?: number;
};

/**
 * Line boil: strokes jitter every few frames like hand-drawn animation.
 * feTurbulence gets a new seed every `hold` frames and displaces the artwork.
 */
export const DoodleBoil: React.FC<DoodleBoilProps> = ({
  children,
  hold = 3,
  amount = 6,
  drawOn = 30,
}) => {
  const frame = useCurrentFrame();
  const id = useId().replace(/:/g, "");
  const seed = Math.floor(frame / hold) % 10;
  const draw =
    drawOn > 0
      ? interpolate(frame, [0, drawOn], [1, 0], { extrapolateRight: "clamp" })
      : 0;

  return (
    <AbsoluteFill>
      <svg viewBox="0 0 1920 1080" width="100%" height="100%">
        <defs>
          <filter id={`boil-${id}`}>
            <feTurbulence type="turbulence" baseFrequency="0.02" numOctaves="2" seed={seed} />
            <feDisplacementMap in="SourceGraphic" scale={amount} />
          </filter>
        </defs>
        <g
          filter={`url(#boil-${id})`}
          style={{ strokeDasharray: 1, strokeDashoffset: draw }}
        >
          {children}
        </g>
      </svg>
    </AbsoluteFill>
  );
};
```
