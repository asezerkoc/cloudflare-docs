---
name: remotion
description: Builds, edits, previews, and renders programmatic videos with Remotion (React-based video framework). Load when asked to make a video, animation, motion graphic, intro/outro, social media clip, captioned video, or data-driven video with code, or when working with a Remotion project (`remotion`, `@remotion/*` packages, `<Composition>`, `useCurrentFrame`).
---

# Remotion

Remotion turns React components into videos. Every frame is a render of your component tree at a given frame number; the renderer screenshots each frame in headless Chromium and encodes the result with FFmpeg into MP4, WebM, GIF, or image sequences.

Use this skill to scaffold a project, write scenes, preview them, and render final files.

## Workflow

1. **Clarify the brief** — duration, aspect ratio (16:9 `1920x1080`, 9:16 `1080x1920`, 1:1 `1080x1080`), fps (default `30`), text content, brand colors, fonts, music, and output format. Pick sensible defaults and state them instead of blocking on questions.
2. **Plan scenes** — write a short scene list with frame ranges before coding (for example, `0–60 intro`, `60–180 feature`, `180–240 outro`). Frames = seconds × fps.
3. **Scaffold or locate the project** (see below). Keep Remotion projects out of `src/content/` and other docs folders in this repository; use a scratch directory or a separate repository unless the user asks otherwise.
4. **Write compositions and scenes** following the rules in this file.
5. **Type-check and preview** — run `npx tsc --noEmit`, then render a few stills to check layout before rendering the full video.
6. **Render** the final file and report its path, duration, resolution, and size.

## Project setup

New project (interactive template picker; use `--yes` style flags or pick the blank/Hello World template):

```sh
npx create-video@latest
```

Add to an existing React/TypeScript project:

```sh
npm i remotion @remotion/cli
```

All `remotion` and `@remotion/*` packages must be pinned to the **exact same version**. Mismatched versions cause runtime errors. Upgrade with `npx remotion upgrade`.

Minimal structure:

```txt
src/
├── index.ts      # registerRoot(RemotionRoot)
├── Root.tsx      # <Composition> declarations
└── MyVideo.tsx   # scene components
public/           # assets referenced with staticFile()
```

```tsx title="src/index.ts"
import { registerRoot } from "remotion";
import { RemotionRoot } from "./Root";

registerRoot(RemotionRoot);
```

```tsx title="src/Root.tsx"
import { Composition } from "remotion";
import { z } from "zod";
import { MyVideo, myVideoSchema } from "./MyVideo";

export const RemotionRoot: React.FC = () => (
	<Composition
		id="MyVideo"
		component={MyVideo}
		durationInFrames={240}
		fps={30}
		width={1920}
		height={1080}
		schema={myVideoSchema}
		defaultProps={{ title: "Hello", accent: "#f6821f" }}
	/>
);
```

Use `calculateMetadata` on `<Composition>` when duration or size depends on props or fetched data (for example, matching the length of an audio file).

## Core API

| API                                                | Purpose                                                                |
| -------------------------------------------------- | ---------------------------------------------------------------------- |
| `useCurrentFrame()`                                | Current frame number (relative to the enclosing `<Sequence>`)          |
| `useVideoConfig()`                                 | `{ fps, width, height, durationInFrames, id }`                         |
| `interpolate(frame, input, output, opts)`          | Map frame ranges to values; use `extrapolateLeft/Right: "clamp"`       |
| `spring({ frame, fps, config, delay })`            | Physics-based 0→1 animation; `config: { damping: 200 }` for no bounce  |
| `Easing`                                           | Easing functions for `interpolate` (`Easing.bezier`, `Easing.inOut`)   |
| `<AbsoluteFill>`                                   | Full-size absolutely positioned layer                                  |
| `<Sequence from durationInFrames>`                 | Time-shift children; resets `useCurrentFrame()` to 0 inside            |
| `<Series>` / `<Series.Sequence>`                   | Back-to-back scenes without computing offsets                          |
| `<Loop>`, `<Freeze>`                               | Repeat or hold content                                                 |
| `<Img>`, `<OffthreadVideo>`, `<Audio>`, `<IFrame>` | Media that waits for loading before a frame is captured                |
| `staticFile("logo.png")`                           | Reference files in `public/`                                           |
| `delayRender()` / `continueRender(handle)`         | Block rendering until async work (fetch, font load) finishes           |
| `random("seed")`                                   | Deterministic pseudo-random numbers                                    |
| `interpolateColors(frame, input, colors)`          | Animate between colors                                                 |

Useful packages (install at the same version as `remotion`):

- `@remotion/transitions` — `<TransitionSeries>` with `fade()`, `slide()`, `wipe()`, `linearTiming()`, `springTiming()`.
- `@remotion/google-fonts` — `loadFont()` from `@remotion/google-fonts/Inter` and similar.
- `@remotion/media-utils` — audio data and visualizations (`useAudioData`, `visualizeAudio`).
- `@remotion/captions` — caption data types and TikTok-style page grouping; pair with `@remotion/install-whisper-cpp` for transcription.
- `@remotion/shapes`, `@remotion/paths`, `@remotion/noise`, `@remotion/three`, `@remotion/lottie` — shapes, SVG path animation, noise, 3D, and Lottie.
- `@remotion/zod-types` — `zColor()` and other schema helpers for editable props in Studio.
- `@remotion/renderer` — Node.js rendering API (`bundle`, `selectComposition`, `renderMedia`, `renderStill`).
- `@remotion/lambda` / `@remotion/cloudrun` — distributed cloud rendering.
- `@remotion/player` — embed a playable preview in a React web app.

## Scene example

```tsx title="src/MyVideo.tsx"
import {
	AbsoluteFill,
	Sequence,
	interpolate,
	spring,
	useCurrentFrame,
	useVideoConfig,
} from "remotion";
import { z } from "zod";

export const myVideoSchema = z.object({
	title: z.string(),
	accent: z.string(),
});

const Title: React.FC<{ text: string; accent: string }> = ({ text, accent }) => {
	const frame = useCurrentFrame();
	const { fps } = useVideoConfig();

	const enter = spring({ frame, fps, config: { damping: 200 } });
	const opacity = interpolate(frame, [0, 20], [0, 1], {
		extrapolateLeft: "clamp",
		extrapolateRight: "clamp",
	});

	return (
		<AbsoluteFill style={{ justifyContent: "center", alignItems: "center" }}>
			<h1
				style={{
					fontSize: 140,
					color: accent,
					opacity,
					transform: `translateY(${interpolate(enter, [0, 1], [80, 0])}px)`,
				}}
			>
				{text}
			</h1>
		</AbsoluteFill>
	);
};

export const MyVideo: React.FC<z.infer<typeof myVideoSchema>> = ({
	title,
	accent,
}) => (
	<AbsoluteFill style={{ backgroundColor: "#0b0b0f" }}>
		<Sequence durationInFrames={120}>
			<Title text={title} accent={accent} />
		</Sequence>
		<Sequence from={120}>
			<Title text="Thanks for watching" accent="#ffffff" />
		</Sequence>
	</AbsoluteFill>
);
```

## Rules for correct videos

Rendering is parallel and frame-by-frame, so every frame must be a pure function of the frame number and props.

- **Drive all motion from `useCurrentFrame()`.** Do not use CSS `transition`, CSS `@keyframes`, `requestAnimationFrame`, `setTimeout`, or animation libraries that run on wall-clock time. They will flicker or freeze in renders.
- **No `Math.random()`.** Use `random("some-seed")` so every render process gets the same value.
- **Always clamp `interpolate`** unless you want values to keep growing past the range.
- **Use Remotion media components** (`<Img>`, `<OffthreadVideo>`, `<Audio>`) instead of raw `<img>`/`<video>`/`<audio>`, so the renderer waits for assets.
- **Wrap async work in `delayRender()`/`continueRender()`**, including fetches and custom font loading. Unresolved handles time out after 30 seconds by default.
- **Reference local assets with `staticFile()`**, not relative imports of large media.
- **Think in frames.** Convert seconds with `seconds * fps`; read `fps` from `useVideoConfig()` instead of hardcoding it inside scenes.
- **Keep text inside safe areas** — leave margins of about 5% of width/height, larger for 9:16 social formats where UI overlays the edges.
- **Load fonts explicitly** with `@remotion/google-fonts` or `@font-face` plus `delayRender`; system fonts differ between machines.

## Preview and render commands

```sh
npx remotion studio                                   # Live preview in the browser
npx remotion compositions                              # List composition IDs
npx remotion still MyVideo out/frame.png --frame=60    # Render a single frame
npx remotion render MyVideo out/video.mp4              # Render the full video
npx remotion render MyVideo out/video.mp4 --props='{"title":"Merhaba"}'
npx remotion render MyVideo out/clip.gif --codec=gif --every-nth-frame=2
```

Common render flags: `--codec` (`h264`, `h265`, `vp8`, `vp9`, `prores`, `gif`), `--crf`, `--concurrency`, `--frames=0-89`, `--scale`, `--props` (JSON string or path to a JSON file), `--browser-executable`.

### Cloud or sandboxed environments

Remotion downloads its own Chrome Headless Shell on first render. If downloads are blocked but a Chromium is already installed (for example, Playwright's browsers under `/opt/pw-browsers/`), point Remotion at it:

```sh
npx remotion render MyVideo out/video.mp4 \
  --browser-executable="$(ls -d /opt/pw-browsers/chromium_headless_shell-*/chrome-linux/headless_shell | head -n1)"
```

Check that the path exists before using it. Verify the result with `ffprobe out/video.mp4` (bundled with Remotion as `npx remotion ffprobe`) and inspect a few stills visually before reporting success.

## Licensing

Remotion is free for individuals and small companies, but larger companies need a paid company license. When a user plans commercial use, point them to the current terms at https://www.remotion.dev/license instead of stating thresholds from memory.

## References

- Documentation: https://www.remotion.dev/docs
- API reference: https://www.remotion.dev/docs/api
- CLI reference: https://www.remotion.dev/docs/cli
- Templates: https://www.remotion.dev/templates
