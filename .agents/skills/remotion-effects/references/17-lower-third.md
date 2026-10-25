# 17 Lower third

**Component:** `LowerThird` · **File:** `src/effects/LowerThird.tsx`

Name and title strap in the bottom-left corner: an accent bar grows up, the text slides out from behind it, holds, then animates out in reverse.

**Use it for:** Introducing a speaker in interviews, vlogs, events, news-style videos.

## Usage

```tsx
<AbsoluteFill>
  <OffthreadVideo src={staticFile("interview.mp4")} />
  <Sequence from={30} durationInFrames={150}>
    <LowerThird name="Jane Doe" title="Head of Product, Acme" />
  </Sequence>
</AbsoluteFill>
```

## Notes

- Wrap it in a `<Sequence>`. The strap animates out 20 frames before the sequence ends unless you set `hold`.
- Keep it on screen for 3-5 seconds (90-150 frames at 30 fps).

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

export type LowerThirdProps = {
  name: string;
  title?: string;
  accent?: string;
  /** Frames to hold on screen before animating out. Defaults to fill the composition. */
  hold?: number;
};

/** Name + title strap: accent bar wipes in, text slides out from behind it, then reverses out. */
export const LowerThird: React.FC<LowerThirdProps> = ({
  name,
  title,
  accent = "#f6821f",
  hold,
}) => {
  const frame = useCurrentFrame();
  const { durationInFrames } = useVideoConfig();
  const outAt = hold ?? durationInFrames - 20;
  const ease = Easing.out(Easing.cubic);
  const clamp = { extrapolateLeft: "clamp" as const, extrapolateRight: "clamp" as const };

  const inBar = interpolate(frame, [0, 12], [0, 1], { ...clamp, easing: ease });
  const inText = interpolate(frame, [8, 22], [0, 1], { ...clamp, easing: ease });
  const out = interpolate(frame, [outAt, outAt + 14], [0, 1], {
    ...clamp,
    easing: Easing.in(Easing.cubic),
  });

  const textX = interpolate(inText, [0, 1], [-100, 0]) - out * 100;
  const textClip = `inset(0 ${Math.max(0, (1 - inText + out) * 100)}% 0 0)`;

  return (
    <AbsoluteFill style={{ justifyContent: "flex-end", padding: "0 0 120px 120px" }}>
      <div style={{ display: "flex", alignItems: "stretch", gap: 24 }}>
        <div
          style={{
            width: 12,
            background: accent,
            transform: `scaleY(${inBar * (1 - out)})`,
            transformOrigin: "bottom",
          }}
        />
        <div style={{ clipPath: textClip }}>
          <div
            style={{
              transform: `translateX(${textX}%)`,
              fontFamily: "Inter, Helvetica, Arial, sans-serif",
              color: "white",
            }}
          >
            <div
              style={{
                fontSize: 64,
                fontWeight: 800,
                background: "rgba(0,0,0,0.65)",
                padding: "10px 24px",
              }}
            >
              {name}
            </div>
            {title ? (
              <div
                style={{
                  fontSize: 34,
                  fontWeight: 500,
                  background: accent,
                  padding: "8px 24px",
                  display: "inline-block",
                }}
              >
                {title}
              </div>
            ) : null}
          </div>
        </div>
      </div>
    </AbsoluteFill>
  );
};
```
