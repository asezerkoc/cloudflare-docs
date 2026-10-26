# 21 Freeze frame

**Component:** `FreezeFrame` · **File:** `src/effects/FreezeFrame.tsx`

The footage stops dead with a white flash, punches in, desaturates, and a tilted label slams on screen. This is the classic "Yep, that's me. You're probably wondering how I got here." moment.

**Use it for:** Introducing a character, comedic beats, narrator interruptions, highlighting a key moment in sports or gameplay.

## Usage

```tsx
<FreezeFrame freezeAt={45} label="Yep. That's me." sublabel="Three weeks before launch">
  <OffthreadVideo src={staticFile("clip.mp4")} />
</FreezeFrame>
```

## Notes

- This uses Remotion's `<Freeze>`, so any child driven by `useCurrentFrame` (OffthreadVideo, Video, animations) freezes correctly.
- Add a record-scratch sound with `<Audio>` in a `<Sequence from={freezeAt}>` for the full effect.
- Remove the `grayscale` filter to keep color.

## Code

```tsx
import React from "react";
import {
  AbsoluteFill,
  Freeze,
  interpolate,
  spring,
  useCurrentFrame,
  useVideoConfig,
} from "remotion";

export type FreezeFrameProps = {
  /** Footage or animated scene. Must be driven by useCurrentFrame (OffthreadVideo, Video, etc). */
  children: React.ReactNode;
  /** Frame to freeze on. */
  freezeAt: number;
  /** Label that slams in, e.g. "Yep. That's me." */
  label?: string;
  /** Optional sub label. */
  sublabel?: string;
  accent?: string;
};

/** Footage stops dead with a flash, desaturates, punches in and a label slams on. */
export const FreezeFrame: React.FC<FreezeFrameProps> = ({
  children,
  freezeAt,
  label,
  sublabel,
  accent = "#ffe600",
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const frozen = frame >= freezeAt;
  const t = frame - freezeAt;
  const clamp = { extrapolateLeft: "clamp" as const, extrapolateRight: "clamp" as const };

  const punch = spring({ frame: t, fps, config: { damping: 14 } });
  const slam = spring({ frame: t - 4, fps, config: { damping: 9, mass: 0.6 } });
  const flash = interpolate(t, [0, 1, 6], [0, 1, 0], clamp);

  return (
    <AbsoluteFill style={{ overflow: "hidden" }}>
      <AbsoluteFill
        style={{
          transform: frozen ? `scale(${interpolate(punch, [0, 1], [1, 1.08])})` : undefined,
          filter: frozen
            ? `grayscale(${punch * 0.85}) contrast(${1 + punch * 0.2})`
            : undefined,
        }}
      >
        <Freeze frame={freezeAt} active={frozen}>
          {children}
        </Freeze>
      </AbsoluteFill>
      {frozen && label ? (
        <AbsoluteFill style={{ justifyContent: "flex-end", padding: 100 }}>
          <div
            style={{
              alignSelf: "flex-start",
              transform: `rotate(-4deg) scale(${interpolate(slam, [0, 1], [2.2, 1])})`,
              opacity: Math.min(slam * 3, 1),
              fontFamily: "Arial Black, Impact, sans-serif",
            }}
          >
            <div
              style={{
                background: accent,
                color: "black",
                fontSize: 110,
                padding: "6px 30px",
                textTransform: "uppercase",
              }}
            >
              {label}
            </div>
            {sublabel ? (
              <div
                style={{
                  background: "black",
                  color: "white",
                  fontSize: 44,
                  padding: "8px 30px",
                  display: "inline-block",
                }}
              >
                {sublabel}
              </div>
            ) : null}
          </div>
        </AbsoluteFill>
      ) : null}
      <AbsoluteFill style={{ background: "white", opacity: flash }} />
    </AbsoluteFill>
  );
};
```
