# 11 Spotlight

**Component:** `Spotlight` · **File:** `src/effects/Spotlight.tsx`

Darkens the whole frame except a soft circle of light that glides between keyframed positions and sizes.

**Use it for:** Pointing at part of a UI, chart or photo, guiding attention in tutorials, dramatic reveals.

## Usage

```tsx
<Spotlight
  stops={[
    { at: 0, x: 25, y: 40, r: 160 },
    { at: 45, x: 70, y: 60, r: 220 },
    { at: 90, x: 50, y: 50, r: 900 },
  ]}
>
  <Img src={staticFile("dashboard.png")} />
</Spotlight>
```

## Notes

- `x` and `y` are percentages of the frame. `r` is in pixels.
- End with a large `r` to bring the whole frame back into light.
- Lower `dim` (0.5-0.6) when the background still needs to be readable.

## Code

```tsx
import React from "react";
import {
  AbsoluteFill,
  Easing,
  interpolate,
  useCurrentFrame,
} from "remotion";

export type SpotlightStop = {
  /** Frame at which the light arrives here. */
  at: number;
  /** Center, in % of the frame. */
  x: number;
  y: number;
  /** Radius in px. */
  r: number;
};

export type SpotlightProps = {
  children: React.ReactNode;
  stops: SpotlightStop[];
  /** Darkness outside the light, 0-1. */
  dim?: number;
  /** Soft edge size in px. */
  feather?: number;
};

/** Darkens everything except a moving circle of light. */
export const Spotlight: React.FC<SpotlightProps> = ({
  children,
  stops,
  dim = 0.78,
  feather = 60,
}) => {
  const frame = useCurrentFrame();
  const at = stops.map((s) => s.at);
  const opts = {
    extrapolateLeft: "clamp" as const,
    extrapolateRight: "clamp" as const,
    easing: Easing.inOut(Easing.cubic),
  };
  const pick = (key: "x" | "y" | "r") =>
    stops.length === 1
      ? stops[0][key]
      : interpolate(frame, at, stops.map((s) => s[key]), opts);
  const x = pick("x");
  const y = pick("y");
  const r = pick("r");
  const fadeIn = interpolate(frame, [0, 12], [0, 1], { extrapolateRight: "clamp" });

  return (
    <AbsoluteFill>
      {children}
      <AbsoluteFill
        style={{
          opacity: fadeIn,
          background: `radial-gradient(circle at ${x}% ${y}%, transparent ${r}px, rgba(0,0,0,${dim}) ${r + feather}px)`,
        }}
      />
    </AbsoluteFill>
  );
};
```
