# 04 Mirror

**Component:** `Mirror` · **File:** `src/effects/Mirror.tsx`

Two modes: `floor` adds a glossy reflection under the content (like a product on a shiny surface). `split` mirrors the left half onto the right half for a symmetric, kaleidoscope-like image.

**Use it for:** Product shots, logo reveals, titles on a dark stage (floor). Stylized, surreal or music-video looks (split).

## Usage

```tsx
<Mirror mode="floor" reflection={0.3}>
  <Img src={staticFile("product.png")} style={{ width: 700 }} />
</Mirror>
```

## Notes

- Floor mode needs content with a clear bottom edge. Transparent PNGs work best.
- Split mode works on full-frame footage. Point the camera's subject at the center line for the strongest effect.

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

export type MirrorProps = {
  children: React.ReactNode;
  /** "floor" = glossy reflection below, "split" = left half mirrored to the right. */
  mode?: "floor" | "split";
  /** Reflection opacity for floor mode. */
  reflection?: number;
  gap?: number;
};

export const Mirror: React.FC<MirrorProps> = ({
  children,
  mode = "floor",
  reflection = 0.35,
  gap = 8,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const rise = spring({ frame, fps, config: { damping: 200 } });

  if (mode === "split") {
    const open = interpolate(rise, [0, 1], [0, 50]);
    return (
      <AbsoluteFill>
        <AbsoluteFill style={{ clipPath: `inset(0 50% 0 0)` }}>{children}</AbsoluteFill>
        <AbsoluteFill
          style={{
            transform: "scaleX(-1)",
            clipPath: `inset(0 50% 0 ${50 - open}%)`,
          }}
        >
          {children}
        </AbsoluteFill>
      </AbsoluteFill>
    );
  }

  const y = interpolate(rise, [0, 1], [120, 0]);
  return (
    <AbsoluteFill style={{ alignItems: "center", justifyContent: "center" }}>
      <div style={{ transform: `translateY(${y}px)`, opacity: rise }}>
        <div>{children}</div>
        <div
          style={{
            marginTop: gap,
            transform: "scaleY(-1)",
            opacity: reflection,
            maskImage: "linear-gradient(to top, black 0%, transparent 60%)",
            WebkitMaskImage: "linear-gradient(to top, black 0%, transparent 60%)",
            filter: "blur(1px)",
          }}
        >
          {children}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```
