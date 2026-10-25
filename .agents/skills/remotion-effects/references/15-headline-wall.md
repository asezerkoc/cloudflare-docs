# 15 Headline wall

**Component:** `HeadlineWall` · **File:** `src/effects/HeadlineWall.tsx`

News cards pile onto a tilted wall that drifts upward. Then one headline lifts forward while all the others dim.

**Use it for:** "Everyone is talking about it" moments, press coverage montages, setting up a topic or controversy.

## Usage

```tsx
<HeadlineWall
  hero={4}
  heroAt={45}
  headlines={[
    { source: "Tech Wire", title: "Startup hits one million users" },
    { source: "The Daily", title: "Why everyone is switching" },
    // ...about 12 items
  ]}
/>
```

## Notes

- Use about 12 headlines for a 3-column wall so the screen is full.
- Do not show real outlet names with made-up headlines. Either quote real coverage accurately or use invented outlet names.
- Pick a `hero` that is visible in the middle of the wall after the tilt and drift (indexes 3-8 with 3 columns).

## Code

```tsx
import React from "react";
import {
  AbsoluteFill,
  interpolate,
  random,
  spring,
  useCurrentFrame,
  useVideoConfig,
} from "remotion";

export type Headline = { source: string; title: string };

export type HeadlineWallProps = {
  headlines: Headline[];
  /** Index of the headline that comes forward. -1 = none. */
  hero?: number;
  /** Frame the hero headline comes forward. */
  heroAt?: number;
  cols?: number;
};

/** Screen fills with news headlines, then one comes forward while the rest dim. */
export const HeadlineWall: React.FC<HeadlineWallProps> = ({
  headlines,
  hero = 0,
  heroAt = 45,
  cols = 3,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const focus = spring({ frame: frame - heroAt, fps, config: { damping: 200 } });
  const drift = interpolate(frame, [0, 10 * fps], [0, -120]);

  return (
    <AbsoluteFill style={{ background: "#ececec", overflow: "hidden" }}>
      <div
        style={{
          display: "grid",
          gridTemplateColumns: `repeat(${cols}, 1fr)`,
          gap: 28,
          padding: 40,
          transform: `translateY(${drift}px) rotate(-4deg) scale(1.12)`,
        }}
      >
        {headlines.map((h, i) => {
          const enter = spring({
            frame: frame - Math.floor(random(`h-${i}`) * 25),
            fps,
            config: { damping: 18 },
          });
          const isHero = i === hero;
          const dim = isHero ? 1 : interpolate(focus, [0, 1], [1, 0.25]);
          const lift = isHero ? interpolate(focus, [0, 1], [1, 1.35]) : 1;
          return (
            <div
              key={i}
              style={{
                background: "white",
                padding: "26px 30px",
                borderRadius: 6,
                boxShadow: isHero
                  ? `0 ${30 * focus}px ${60 * focus}px rgba(0,0,0,0.35)`
                  : "0 2px 8px rgba(0,0,0,0.12)",
                fontFamily: "Georgia, serif",
                opacity: enter * dim,
                transform: `translateY(${interpolate(enter, [0, 1], [40, 0])}px) scale(${lift})`,
                zIndex: isHero ? 2 : 1,
                position: "relative",
              }}
            >
              <div
                style={{
                  fontFamily: "Arial, sans-serif",
                  fontSize: 18,
                  fontWeight: 700,
                  letterSpacing: 2,
                  textTransform: "uppercase",
                  color: "#c00",
                  marginBottom: 10,
                }}
              >
                {h.source}
              </div>
              <div style={{ fontSize: 36, fontWeight: 700, lineHeight: 1.15, color: "#111" }}>
                {h.title}
              </div>
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```
