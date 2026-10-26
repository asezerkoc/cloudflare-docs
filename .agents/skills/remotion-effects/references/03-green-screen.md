# 03 Green screen

**Component:** `GreenScreen` · **File:** `src/effects/GreenScreen.tsx`

Chroma key: removes the green background from footage and composites it over any background. Uses an SVG color-matrix filter (GPU in Chrome) with optional despill.

**Use it for:** Talking-head footage shot on green, placing a presenter over slides, UI or b-roll.

## Usage

```tsx
<GreenScreen background={<OffthreadVideo src={staticFile("broll.mp4")} muted />}>
  <OffthreadVideo src={staticFile("presenter-green.mp4")} />
</GreenScreen>
```

## Notes

- `strength` 1.6-2.4 covers most footage. Raise it if green remains, lower it if skin or light colors become see-through.
- Despill sets green to the average of red and blue. This removes the green fringe but can mute real greens in clothing. Set `despill={false}` if the subject wears green.
- If the user can export footage with alpha (ProRes 4444 or WebM VP9), prefer that over keying.

## Code

```tsx
import React, { useId } from "react";
import { AbsoluteFill } from "remotion";

export type GreenScreenProps = {
  /** Footage shot on green: <OffthreadVideo>, <Video> or <Img>. */
  children: React.ReactNode;
  /** What shows through where the green was. */
  background: React.ReactNode;
  /** Higher = more aggressive key. 2 is a good start. */
  strength?: number;
  /** Pulls green tint off edges and hair. */
  despill?: boolean;
};

/**
 * Chroma key with an SVG filter. Runs on the GPU in Chrome, so it is fast to
 * render and works on any element (video, image, even HTML).
 *
 * alpha = 1 + r + b - strength * g  -> pure green goes to 0, everything else stays opaque.
 */
export const GreenScreen: React.FC<GreenScreenProps> = ({
  children,
  background,
  strength = 2,
  despill = true,
}) => {
  const id = useId().replace(/:/g, "");
  return (
    <AbsoluteFill>
      <svg width="0" height="0" style={{ position: "absolute" }}>
        <filter id={`key-${id}`} colorInterpolationFilters="sRGB">
          {/* Despill: clamp green to the average of red and blue */}
          {despill ? (
            <feColorMatrix
              type="matrix"
              values="1 0 0 0 0
                      0.5 0 0.5 0 0
                      0 0 1 0 0
                      0 0 0 1 0"
              result="despilled"
            />
          ) : null}
          <feColorMatrix
            in="SourceGraphic"
            type="matrix"
            values={`0 0 0 0 0
                     0 0 0 0 0
                     0 0 0 0 0
                     1 ${-strength} 1 0 1`}
            result="mask"
          />
          <feComposite
            in={despill ? "despilled" : "SourceGraphic"}
            in2="mask"
            operator="in"
          />
        </filter>
      </svg>
      <AbsoluteFill>{background}</AbsoluteFill>
      <AbsoluteFill style={{ filter: `url(#key-${id})` }}>{children}</AbsoluteFill>
    </AbsoluteFill>
  );
};
```
