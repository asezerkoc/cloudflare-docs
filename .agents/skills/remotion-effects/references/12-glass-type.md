# 12 Glass type

**Component:** `GlassType` · **File:** `src/effects/GlassType.tsx`

Glassmorphism title: a frosted, blurred panel with translucent lettering, a thin bright edge and a specular highlight that sweeps across the letters.

**Use it for:** Modern tech or product titles, iOS-style UI looks, titles over colorful footage or gradients.

## Usage

```tsx
<GlassType
  text="Introducing Orbit"
  fontSize={180}
  background={<OffthreadVideo src={staticFile("colorful-bg.mp4")} muted />}
/>
```

## Notes

- Glass only reads over a busy, colorful background. Over a flat color it just looks gray.
- `backdrop-filter` is expensive. If a render is slow, render with `--concurrency` set lower or pre-blur the background.
- Set `panel={false}` for lettering only, without the frosted card.

## Code

```tsx
import React from "react";
import {
  AbsoluteFill,
  interpolate,
  spring,
  useCurrentFrame,
  useVideoConfig,
} from "remotion";

export type GlassTypeProps = {
  text: string;
  /** Something colorful behind the glass. Glass only reads over a busy background. */
  background: React.ReactNode;
  fontSize?: number;
  /** Show a frosted panel behind the text. */
  panel?: boolean;
};

/** Frosted-glass panel + translucent lettering with a moving specular highlight. */
export const GlassType: React.FC<GlassTypeProps> = ({
  text,
  background,
  fontSize = 220,
  panel = true,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const enter = spring({ frame, fps, config: { damping: 200 } });
  // Highlight sweeps across the letters, -50% -> 150%
  const sweep = interpolate(frame, [10, 50], [-50, 150], {
    extrapolateLeft: "clamp",
    extrapolateRight: "clamp",
  });

  return (
    <AbsoluteFill>
      <AbsoluteFill>{background}</AbsoluteFill>
      <AbsoluteFill style={{ alignItems: "center", justifyContent: "center" }}>
        <div
          style={{
            padding: panel ? "60px 100px" : 0,
            borderRadius: 48,
            background: panel ? "rgba(255,255,255,0.12)" : undefined,
            backdropFilter: panel ? "blur(30px) saturate(160%)" : undefined,
            border: panel ? "1.5px solid rgba(255,255,255,0.35)" : undefined,
            boxShadow: panel
              ? "inset 0 1px 0 rgba(255,255,255,0.5), 0 30px 60px rgba(0,0,0,0.25)"
              : undefined,
            transform: `translateY(${interpolate(enter, [0, 1], [60, 0])}px) scale(${interpolate(enter, [0, 1], [0.94, 1])})`,
            opacity: enter,
          }}
        >
          <div
            style={{
              fontFamily: "Helvetica Neue, Arial, sans-serif",
              fontWeight: 800,
              fontSize,
              letterSpacing: -fontSize * 0.04,
              lineHeight: 1,
              color: "transparent",
              backgroundImage: `linear-gradient(110deg, rgba(255,255,255,0.55) ${sweep - 20}%, rgba(255,255,255,1) ${sweep}%, rgba(255,255,255,0.55) ${sweep + 20}%)`,
              backgroundClip: "text",
              WebkitBackgroundClip: "text",
              WebkitTextStroke: "1.5px rgba(255,255,255,0.7)",
              filter: "drop-shadow(0 8px 24px rgba(0,0,0,0.2))",
            }}
          >
            {text}
          </div>
        </div>
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```
