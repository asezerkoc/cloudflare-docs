# 08 Triptych

**Component:** `Triptych` · **File:** `src/effects/Triptych.tsx`

Three panels side by side, revealed one after another with alternating vertical wipes (down, up, down). Each panel also settles from a slight zoom.

**Use it for:** Before, during and after shots, three features or three people, comparing options, rhythmic montage cuts.

## Usage

```tsx
<Triptych
  panels={[
    <OffthreadVideo key="a" src={staticFile("clip-a.mp4")} muted style={{ width: "100%", height: "100%", objectFit: "cover" }} />,
    <OffthreadVideo key="b" src={staticFile("clip-b.mp4")} muted style={{ width: "100%", height: "100%", objectFit: "cover" }} />,
    <OffthreadVideo key="c" src={staticFile("clip-c.mp4")} muted style={{ width: "100%", height: "100%", objectFit: "cover" }} />,
  ]}
  stagger={10}
/>
```

## Notes

- Each panel is about 1/3 of the frame width, so frame the content for a tall crop. Use `objectFit: "cover"` and adjust `objectPosition`.
- Time `stagger` to the beat of the music (at 30 fps and 120 BPM, one beat is 15 frames).

## Code

```tsx
import React from "react";
import {
  AbsoluteFill,
  Easing,
  interpolate,
  useCurrentFrame,
} from "remotion";

export type TriptychProps = {
  panels: [React.ReactNode, React.ReactNode, React.ReactNode];
  /** Frames between each panel reveal. */
  stagger?: number;
  /** Frames each wipe takes. */
  wipe?: number;
  gap?: number;
  background?: string;
};

/** Three panels side by side, each revealed with a vertical wipe. */
export const Triptych: React.FC<TriptychProps> = ({
  panels,
  stagger = 10,
  wipe = 18,
  gap = 16,
  background = "#0b0b0b",
}) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ background, flexDirection: "row", gap, padding: gap }}>
      {panels.map((panel, i) => {
        const p = interpolate(frame, [i * stagger, i * stagger + wipe], [0, 1], {
          extrapolateLeft: "clamp",
          extrapolateRight: "clamp",
          easing: Easing.out(Easing.cubic),
        });
        // Alternate wipe direction: down, up, down
        const clip =
          i % 2 === 0
            ? `inset(0 0 ${(1 - p) * 100}% 0)`
            : `inset(${(1 - p) * 100}% 0 0 0)`;
        const zoom = interpolate(frame, [i * stagger, i * stagger + 90], [1.15, 1], {
          extrapolateLeft: "clamp",
          extrapolateRight: "clamp",
        });
        return (
          <div
            key={i}
            style={{
              flex: 1,
              position: "relative",
              overflow: "hidden",
              clipPath: clip,
            }}
          >
            <AbsoluteFill style={{ transform: `scale(${zoom})` }}>{panel}</AbsoluteFill>
          </div>
        );
      })}
    </AbsoluteFill>
  );
};
```
