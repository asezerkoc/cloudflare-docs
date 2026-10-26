# 02 Frame break

**Component:** `FrameBreak` · **File:** `src/effects/FrameBreak.tsx`

A photo sits inside a bordered frame, then the subject pops out past the border toward the viewer. The background stays clipped while the clip on the subject layer opens.

**Use it for:** Introducing a person or product, thumbnail-style hooks, "stepping out of the screen" moments.

## Usage

```tsx
<FrameBreak
  background={<Img src={staticFile("photo.jpg")} style={{ width: "100%", height: "100%", objectFit: "cover" }} />}
  subject={<Img src={staticFile("photo-cutout.png")} style={{ width: "100%", height: "100%", objectFit: "cover" }} />}
  breakAt={20}
/>
```

## Notes

- The subject must be a cutout (transparent PNG, or WebM with alpha) of the same photo, aligned pixel for pixel with `background`. Use `objectFit: "cover"` on both.
- The clip stays open at the bottom (`0px`) so the subject still looks grounded in the frame.
- Cut the subject out ahead of time with a background-removal tool. Remotion cannot segment images by itself.

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

export type FrameBreakProps = {
  /** Fills the frame and stays clipped inside it. */
  background: React.ReactNode;
  /** Cut-out subject (transparent PNG/WebM) that pops out of the frame. */
  subject: React.ReactNode;
  frameWidth?: number;
  frameHeight?: number;
  /** Frame at which the subject breaks out. */
  breakAt?: number;
  borderColor?: string;
};

export const FrameBreak: React.FC<FrameBreakProps> = ({
  background,
  subject,
  frameWidth = 900,
  frameHeight = 560,
  breakAt = 20,
  borderColor = "#ffffff",
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const enter = spring({ frame, fps, config: { damping: 200 } });
  const pop = spring({ frame: frame - breakAt, fps, config: { damping: 12 } });

  const subjectScale = interpolate(pop, [0, 1], [1, 1.25]);
  const subjectY = interpolate(pop, [0, 1], [0, -90]);

  const frameStyle: React.CSSProperties = {
    position: "absolute",
    width: frameWidth,
    height: frameHeight,
    left: "50%",
    top: "50%",
    transform: `translate(-50%, -50%) scale(${interpolate(enter, [0, 1], [0.85, 1])}) rotate(-2deg)`,
  };

  return (
    <AbsoluteFill style={{ opacity: enter }}>
      {/* Layer 1: framed background, clipped */}
      <div
        style={{
          ...frameStyle,
          overflow: "hidden",
          border: `14px solid ${borderColor}`,
          boxShadow: "0 40px 80px rgba(0,0,0,0.45)",
        }}
      >
        {background}
      </div>
      {/* Layer 2: the subject. Before the break it is clipped to the
          frame, after the break the clip opens so it spills over the border. */}
      <div
        style={{
          ...frameStyle,
          clipPath:
            pop < 0.01
              ? "inset(0)"
              : `inset(${-400 * pop}px ${-400 * pop}px 0px ${-400 * pop}px)`,
        }}
      >
        <div
          style={{
            position: "absolute",
            inset: 0,
            transform: `translateY(${subjectY}px) scale(${subjectScale})`,
            transformOrigin: "50% 100%",
            filter: `drop-shadow(0 30px 30px rgba(0,0,0,${0.5 * pop}))`,
          }}
        >
          {subject}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```
