# 13 Checklist

**Component:** `Checklist` · **File:** `src/effects/Checklist.tsx`

List items slide in one by one; each checkbox pops, fills with the accent color and a check mark draws itself. Items can be struck through.

**Use it for:** Steps, requirements, "what you get" lists, recaps, to-do style storytelling.

## Usage

```tsx
<Checklist
  title="Before you launch"
  items={["Write the docs", "Ship the feature", "Tell everyone"]}
  every={20}
  strike={false}
/>
```

## Notes

- `every` controls the pace. 20-30 frames per item at 30 fps gives time to read.
- Keep items short (under about 35 characters) so they stay on one line at the default size.

## Code

```tsx
import React from "react";
import {
  AbsoluteFill,
  Easing,
  interpolate,
  spring,
  useCurrentFrame,
  useVideoConfig,
} from "remotion";

export type ChecklistProps = {
  items: string[];
  title?: string;
  /** Frame the first item appears. */
  startAt?: number;
  /** Frames between items. */
  every?: number;
  /** Strike through items after they are checked. */
  strike?: boolean;
  accent?: string;
};

/** Items appear one by one and get ticked with a hand-drawn check. */
export const Checklist: React.FC<ChecklistProps> = ({
  items,
  title,
  startAt = 10,
  every = 20,
  strike = false,
  accent = "#22c55e",
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill
      style={{
        justifyContent: "center",
        padding: "0 260px",
        gap: 34,
        fontFamily: "Inter, Helvetica, Arial, sans-serif",
        color: "white",
      }}
    >
      {title ? (
        <div style={{ fontSize: 72, fontWeight: 800, marginBottom: 20 }}>{title}</div>
      ) : null}
      {items.map((item, i) => {
        const t = startAt + i * every;
        const appear = spring({ frame: frame - t, fps, config: { damping: 200 } });
        const tick = interpolate(frame, [t + 8, t + 18], [1, 0], {
          extrapolateLeft: "clamp",
          extrapolateRight: "clamp",
          easing: Easing.out(Easing.cubic),
        });
        const boxPop = spring({ frame: frame - (t + 6), fps, config: { damping: 10 } });
        const line = interpolate(frame, [t + 18, t + 28], [0, 100], {
          extrapolateLeft: "clamp",
          extrapolateRight: "clamp",
        });
        return (
          <div
            key={i}
            style={{
              display: "flex",
              alignItems: "center",
              gap: 36,
              opacity: appear,
              transform: `translateX(${interpolate(appear, [0, 1], [-60, 0])}px)`,
            }}
          >
            <svg
              width={72}
              height={72}
              viewBox="0 0 72 72"
              style={{ transform: `scale(${interpolate(boxPop, [0, 1], [1, 1.08])})` }}
            >
              <rect
                x={4}
                y={4}
                width={64}
                height={64}
                rx={16}
                fill={tick < 1 ? accent : "transparent"}
                stroke={tick < 1 ? accent : "rgba(255,255,255,0.6)"}
                strokeWidth={5}
              />
              <path
                d="M18 37 L31 50 L55 23"
                fill="none"
                stroke="white"
                strokeWidth={8}
                strokeLinecap="round"
                strokeLinejoin="round"
                pathLength={1}
                strokeDasharray={1}
                strokeDashoffset={tick}
              />
            </svg>
            <div style={{ fontSize: 56, position: "relative" }}>
              {item}
              {strike ? (
                <div
                  style={{
                    position: "absolute",
                    left: 0,
                    top: "52%",
                    height: 5,
                    width: `${line}%`,
                    background: "rgba(255,255,255,0.8)",
                  }}
                />
              ) : null}
            </div>
          </div>
        );
      })}
    </AbsoluteFill>
  );
};
```
