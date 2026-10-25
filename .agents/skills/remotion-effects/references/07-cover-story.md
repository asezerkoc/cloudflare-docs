# 07 Cover story

**Component:** `CoverStory` · **File:** `src/effects/CoverStory.tsx`

Turns a photo into a magazine cover: a giant masthead drops in behind the subject, cover lines slide in from the left and the photo slowly pushes in.

**Use it for:** Profiles and interviews, "featured" moments, personal-brand intros, YouTube openers.

## Usage

```tsx
<CoverStory
  background={<Img src={staticFile("portrait.jpg")} style={{ width: "100%", height: "100%", objectFit: "cover" }} />}
  subject={<Img src={staticFile("portrait-cutout.png")} style={{ width: "100%", height: "100%", objectFit: "cover" }} />}
  masthead="VOGUE"
  issue="OCTOBER 2026"
  headlines={["The founder rewriting the rules", "Inside the studio", "12 lessons from year one"]}
/>
```

## Notes

- The masthead sits behind the subject only when you pass `subject` (a cutout aligned with `background`). Without it, the title sits on top, which is still fine.
- Do not use the real name or logo of an existing magazine for anything published. A made-up masthead avoids trademark problems.
- The first headline is the large lead line. Keep it under about 40 characters.

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

export type CoverStoryProps = {
  /** Full background photo. */
  background: React.ReactNode;
  /** Same photo with the person cut out (transparent PNG). Sits on top of the masthead. */
  subject?: React.ReactNode;
  masthead: string;
  issue?: string;
  headlines: string[];
  accent?: string;
};

/** Magazine cover: masthead behind the subject, cover lines slide in. */
export const CoverStory: React.FC<CoverStoryProps> = ({
  background,
  subject,
  masthead,
  issue = "ISSUE 01",
  headlines,
  accent = "#ff2d55",
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const push = interpolate(frame, [0, 6 * fps], [1.08, 1], {
    extrapolateRight: "clamp",
  });
  const mast = spring({ frame: frame - 5, fps, config: { damping: 200 } });

  return (
    <AbsoluteFill style={{ fontFamily: "Georgia, 'Times New Roman', serif" }}>
      <AbsoluteFill style={{ transform: `scale(${push})` }}>{background}</AbsoluteFill>
      {/* Masthead: sits between background and subject */}
      <AbsoluteFill style={{ alignItems: "center", paddingTop: 30 }}>
        <div
          style={{
            fontSize: 300,
            fontWeight: 900,
            letterSpacing: -12,
            lineHeight: 1,
            color: "white",
            textTransform: "uppercase",
            transform: `translateY(${interpolate(mast, [0, 1], [-200, 0])}px)`,
            opacity: mast,
          }}
        >
          {masthead}
        </div>
      </AbsoluteFill>
      {subject ? (
        <AbsoluteFill style={{ transform: `scale(${push})` }}>{subject}</AbsoluteFill>
      ) : null}
      {/* Cover lines */}
      <AbsoluteFill
        style={{
          justifyContent: "flex-end",
          padding: 80,
          gap: 18,
          color: "white",
          textShadow: "0 4px 20px rgba(0,0,0,0.5)",
        }}
      >
        <div style={{ fontSize: 28, letterSpacing: 6, color: accent, fontFamily: "sans-serif" }}>
          {issue}
        </div>
        {headlines.map((line, i) => {
          const s = spring({ frame: frame - 20 - i * 8, fps, config: { damping: 200 } });
          return (
            <div
              key={i}
              style={{
                fontSize: i === 0 ? 84 : 48,
                fontWeight: i === 0 ? 800 : 500,
                maxWidth: 900,
                transform: `translateX(${interpolate(s, [0, 1], [-80, 0])}px)`,
                opacity: s,
                borderLeft: i === 0 ? `10px solid ${accent}` : undefined,
                paddingLeft: i === 0 ? 24 : 0,
              }}
            >
              {line}
            </div>
          );
        })}
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```
