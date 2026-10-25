# 14 Cinemascope

**Component:** `Cinemascope` · **File:** `src/effects/Cinemascope.tsx`

Black letterbox bars slide in to a widescreen ratio (2.39:1 by default), with a slow push-in, a warm film grade and a vignette.

**Use it for:** Making phone or 16:9 footage feel cinematic, trailers, dramatic moments, "chapter" openers.

## Usage

```tsx
<Cinemascope ratio={2.39}>
  <OffthreadVideo src={staticFile("footage.mp4")} />
</Cinemascope>
```

## Notes

- 2.39 is the classic anamorphic look. 1.85 is a subtler widescreen.
- Set `grade={false}` when the footage is already graded.
- Keep important content (faces, text) in the middle band.

## Code

```tsx
import React from "react";
import {
  AbsoluteFill,
  Easing,
  interpolate,
  useCurrentFrame,
  useVideoConfig,
} from "remotion";

export type CinemascopeProps = {
  children: React.ReactNode;
  /** Target aspect ratio. 2.39 = anamorphic scope, 1.85 = flat. */
  ratio?: number;
  /** Frames for the bars to slide in. */
  duration?: number;
  /** Teal/orange-ish film grade and vignette. */
  grade?: boolean;
};

/** Letterbox bars slide in to a widescreen ratio with a slow push and film grade. */
export const Cinemascope: React.FC<CinemascopeProps> = ({
  children,
  ratio = 2.39,
  duration = 24,
  grade = true,
}) => {
  const frame = useCurrentFrame();
  const { width, height, durationInFrames } = useVideoConfig();
  const barTarget = Math.max(0, (height - width / ratio) / 2);
  const bar = interpolate(frame, [0, duration], [0, barTarget], {
    extrapolateRight: "clamp",
    easing: Easing.inOut(Easing.cubic),
  });
  const push = interpolate(frame, [0, durationInFrames], [1, 1.06]);

  return (
    <AbsoluteFill style={{ background: "black", overflow: "hidden" }}>
      <AbsoluteFill
        style={{
          transform: `scale(${push})`,
          filter: grade ? "contrast(1.08) saturate(0.9) sepia(0.12) hue-rotate(-8deg)" : undefined,
        }}
      >
        {children}
      </AbsoluteFill>
      {grade ? (
        <AbsoluteFill
          style={{
            background: "radial-gradient(ellipse at center, transparent 55%, rgba(0,0,0,0.55) 100%)",
          }}
        />
      ) : null}
      <div style={{ position: "absolute", left: 0, right: 0, top: 0, height: bar, background: "black" }} />
      <div style={{ position: "absolute", left: 0, right: 0, bottom: 0, height: bar, background: "black" }} />
    </AbsoluteFill>
  );
};
```
