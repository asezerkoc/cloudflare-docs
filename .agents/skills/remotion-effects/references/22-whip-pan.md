# 22 Whip pan

**Component:** `WhipPan` · **File:** `src/effects/WhipPan.tsx`

Fast camera whip between two scenes: both slide sideways together with heavy horizontal motion blur that peaks in the middle of the move.

**Use it for:** Fast-paced edits, travel and vlog transitions, "meanwhile..." cuts, linking two locations.

## Usage

```tsx
<WhipPan
  at={60}
  duration={10}
  direction="left"
  from={<OffthreadVideo src={staticFile("city.mp4")} />}
  to={<Sequence from={55}><OffthreadVideo src={staticFile("beach.mp4")} /></Sequence>}
/>
```

## Notes

- 8-12 frames at 30 fps feels like a real whip. Longer looks like a slide.
- Wrap `to` in a `<Sequence>` starting at the start of the whip so the clip begins playing from its first frame.
- Match direction to the camera movement in the footage when possible.

## Code

```tsx
import React, { useId } from "react";
import {
  AbsoluteFill,
  Easing,
  interpolate,
  useCurrentFrame,
  useVideoConfig,
} from "remotion";

export type WhipPanProps = {
  from: React.ReactNode;
  to: React.ReactNode;
  /** Frame at the middle of the whip. */
  at: number;
  /** Total frames of the whip. 8-12 feels right at 30 fps. */
  duration?: number;
  direction?: "left" | "right";
};

/** Fast camera whip between two scenes with horizontal motion blur. */
export const WhipPan: React.FC<WhipPanProps> = ({
  from,
  to,
  at,
  duration = 10,
  direction = "left",
}) => {
  const frame = useCurrentFrame();
  const { width } = useVideoConfig();
  const id = useId().replace(/:/g, "");
  const sign = direction === "left" ? -1 : 1;

  const p = interpolate(frame, [at - duration / 2, at + duration / 2], [0, 1], {
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
    easing: Easing.inOut(Easing.cubic),
  });
  // Blur peaks in the middle of the move.
  const blur = Math.sin(p * Math.PI) * 80;
  const offset = p * width * sign;

  return (
    <AbsoluteFill style={{ overflow: "hidden", background: "black" }}>
      <svg width="0" height="0" style={{ position: "absolute" }}>
        <filter id={`whip-${id}`} x="-20%" y="0" width="140%" height="100%">
          <feGaussianBlur stdDeviation={`${blur} 0`} />
        </filter>
      </svg>
      <AbsoluteFill style={{ filter: blur > 0.5 ? `url(#whip-${id})` : undefined }}>
        {p < 1 ? (
          <AbsoluteFill style={{ transform: `translateX(${offset}px)` }}>{from}</AbsoluteFill>
        ) : null}
        {p > 0 ? (
          <AbsoluteFill style={{ transform: `translateX(${offset - width * sign}px)` }}>
            {to}
          </AbsoluteFill>
        ) : null}
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```
