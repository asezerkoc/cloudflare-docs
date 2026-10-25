# 19 Kinetic captions

**Component:** `KineticCaptions` · **File:** `src/effects/KineticCaptions.tsx`

Short-form style captions: a few words at a time, each word pops in exactly when it is spoken, and the current word is highlighted. Thick outline for readability.

**Use it for:** Reels, TikTok and Shorts, talking-head videos, any video that will be watched muted.

## Usage

```tsx
// words come from a transcription with word timestamps (see notes)
<AbsoluteFill>
  <OffthreadVideo src={staticFile("talk.mp4")} />
  <KineticCaptions words={words} perPage={3} highlight="#ffe600" />
</AbsoluteFill>
```

## Notes

- Word timings: use `@remotion/install-whisper-cpp` (`transcribe()` with token-level timestamps) or `@remotion/openai-whisper`, then map each word to `{ text, startMs, endMs }`.
- For vertical 1080x1920 video, use `perPage={2}` or `3` and `fontSize` about 110.
- Load a heavy display font (Montserrat Black, Anton) with `@remotion/google-fonts` for the classic look.

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

export type CaptionWord = {
  text: string;
  /** Start and end time in milliseconds (as from Whisper / @remotion/install-whisper-cpp). */
  startMs: number;
  endMs: number;
};

export type KineticCaptionsProps = {
  words: CaptionWord[];
  /** Max words on screen at once. */
  perPage?: number;
  highlight?: string;
  fontSize?: number;
};

/** TikTok-style captions: words pop in as they are spoken, the current word is highlighted. */
export const KineticCaptions: React.FC<KineticCaptionsProps> = ({
  words,
  perPage = 4,
  highlight = "#ffe600",
  fontSize = 96,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const ms = (frame / fps) * 1000;

  // Find the page that contains the current time.
  const pages: CaptionWord[][] = [];
  for (let i = 0; i < words.length; i += perPage) pages.push(words.slice(i, i + perPage));
  const page = pages.find((p) => ms >= p[0].startMs && ms < p[p.length - 1].endMs + 150);
  if (!page) return null;

  return (
    <AbsoluteFill style={{ justifyContent: "flex-end", alignItems: "center", paddingBottom: 220 }}>
      <div
        style={{
          display: "flex",
          flexWrap: "wrap",
          justifyContent: "center",
          // Gap scales with the font so the enlarged active word never touches its neighbours
          gap: `0 ${fontSize * 0.5}px`,
          maxWidth: 1500,
          fontFamily: "Montserrat, Arial Black, sans-serif",
          fontWeight: 900,
          fontSize,
          textTransform: "uppercase",
          lineHeight: 1.1,
        }}
      >
        {page.map((w, i) => {
          const startFrame = (w.startMs / 1000) * fps;
          const pop = spring({ frame: frame - startFrame, fps, config: { damping: 12, mass: 0.5 } });
          const active = ms >= w.startMs && ms < w.endMs;
          const visible = ms >= w.startMs;
          return (
            <span
              key={i}
              style={{
                display: "inline-block",
                color: active ? highlight : "white",
                opacity: visible ? 1 : 0,
                transform: `scale(${visible ? interpolate(pop, [0, 1], [0.6, active ? 1.06 : 1]) : 0.6}) translateY(${interpolate(pop, [0, 1], [20, 0])}px)`,
                WebkitTextStroke: "6px black",
                paintOrder: "stroke fill",
                textShadow: "0 8px 0 rgba(0,0,0,0.35)",
              }}
            >
              {w.text}
            </span>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```
