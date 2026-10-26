# 16 Chart grow

**Component:** `ChartGrow` · **File:** `src/effects/ChartGrow.tsx`

Bar chart where each bar springs up in turn and its label counts up to the real value with it.

**Use it for:** Growth numbers, results, comparisons, any statistic in an explainer or pitch video.

## Usage

```tsx
<ChartGrow
  title="Monthly active users"
  unit="k"
  data={[
    { label: "Jan", value: 12 },
    { label: "Feb", value: 19 },
    { label: "Mar", value: 31 },
    { label: "Apr", value: 54, color: "#22c55e" },
  ]}
/>
```

## Notes

- Highlight the bar that matters with its own `color`.
- The count-up never overshoots, even though the bar bounces. That keeps the numbers honest.
- Use 3-7 bars. For more, use a line chart.

## Code

```tsx
import React from "react";
import {
  AbsoluteFill,
  spring,
  useCurrentFrame,
  useVideoConfig,
} from "remotion";

export type ChartDatum = { label: string; value: number; color?: string };

export type ChartGrowProps = {
  data: ChartDatum[];
  title?: string;
  /** Frames between bars starting to grow. */
  stagger?: number;
  /** Suffix after the counted-up number, e.g. "%" or "k". */
  unit?: string;
  color?: string;
};

/** Bar chart where bars spring up and the value labels count up with them. */
export const ChartGrow: React.FC<ChartGrowProps> = ({
  data,
  title,
  stagger = 6,
  unit = "",
  color = "#f6821f",
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const max = Math.max(...data.map((d) => d.value));
  const chartHeight = 620;

  return (
    <AbsoluteFill
      style={{
        padding: "100px 160px",
        fontFamily: "Inter, Helvetica, Arial, sans-serif",
        color: "white",
        justifyContent: "flex-end",
      }}
    >
      {title ? (
        <div style={{ fontSize: 64, fontWeight: 800, marginBottom: 60 }}>{title}</div>
      ) : null}
      <div
        style={{
          display: "flex",
          alignItems: "flex-end",
          gap: 40,
          height: chartHeight,
          borderBottom: "4px solid rgba(255,255,255,0.4)",
        }}
      >
        {data.map((d, i) => {
          const grow = spring({
            frame: frame - 10 - i * stagger,
            fps,
            config: { damping: 15, mass: 0.9 },
          });
          // Count-up follows the bar but never overshoots the real value.
          const shown = Math.round(d.value * Math.min(grow, 1));
          return (
            <div
              key={i}
              style={{ flex: 1, display: "flex", flexDirection: "column", alignItems: "center", gap: 14 }}
            >
              <div style={{ fontSize: 44, fontWeight: 700, opacity: Math.min(grow * 2, 1) }}>
                {shown}
                {unit}
              </div>
              <div
                style={{
                  width: "100%",
                  height: (d.value / max) * (chartHeight - 80) * grow,
                  background: d.color ?? color,
                  borderRadius: "14px 14px 0 0",
                }}
              />
            </div>
          );
        })}
      </div>
      <div style={{ display: "flex", gap: 40, marginTop: 20 }}>
        {data.map((d, i) => (
          <div key={i} style={{ flex: 1, textAlign: "center", fontSize: 32, opacity: 0.8 }}>
            {d.label}
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```
