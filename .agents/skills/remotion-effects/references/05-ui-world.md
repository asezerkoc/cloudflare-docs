# 05 UI world

**Component:** `UiWorld` · **File:** `src/effects/UiWorld.tsx`

App screens float at different depths in 3D space and the camera dollies forward through them, with a slow orbit. Cards fade out as they pass the camera.

**Use it for:** SaaS and app promos, feature tours, "inside the product" openers.

## Usage

```tsx
<UiWorld
  cards={[
    { node: <Img src={staticFile("screen-dashboard.png")} style={{ width: 800 }} />, x: -500, y: -100, z: -400, rotateY: 20 },
    { node: <Img src={staticFile("screen-analytics.png")} style={{ width: 800 }} />, x: 450, y: 80, z: -900, rotateY: -20 },
    { node: <Img src={staticFile("screen-settings.png")} style={{ width: 800 }} />, x: -300, y: 150, z: -1500, rotateY: 15 },
  ]}
  travel={1600}
/>
```

## Notes

- Space cards about 500-700px apart in `z` and alternate `x` left and right so they do not overlap on screen.
- `travel` should roughly equal the distance to the last card you want to fly past.
- Real screenshots look much better than drawn placeholders.

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

export type UiWorldCard = {
  node: React.ReactNode;
  /** Position in the 3D world, px. z is depth (negative = further away). */
  x: number;
  y: number;
  z: number;
  rotateY?: number;
};

export type UiWorldProps = {
  cards: UiWorldCard[];
  /** How far the camera travels forward over the composition, px. */
  travel?: number;
  perspective?: number;
  background?: string;
};

/** UI screens floating in 3D space; the camera dollies through them. */
export const UiWorld: React.FC<UiWorldProps> = ({
  cards,
  travel = 1600,
  perspective = 1200,
  background = "radial-gradient(circle at 50% 40%, #1e2a4a, #05070d)",
}) => {
  const frame = useCurrentFrame();
  const { durationInFrames } = useVideoConfig();

  const progress = interpolate(frame, [0, durationInFrames], [0, 1], {
    easing: Easing.inOut(Easing.cubic),
  });
  const cameraZ = progress * travel;
  const cameraRotateY = interpolate(progress, [0, 1], [-14, 10]);
  const cameraRotateX = interpolate(progress, [0, 1], [8, 2]);

  return (
    <AbsoluteFill style={{ background, perspective, overflow: "hidden" }}>
      <AbsoluteFill
        style={{
          transformStyle: "preserve-3d",
          transform: `translateZ(${cameraZ}px) rotateX(${cameraRotateX}deg) rotateY(${cameraRotateY}deg)`,
        }}
      >
        {cards.map((card, i) => {
          // Fade cards as they pass the camera.
          const distance = card.z + cameraZ;
          const opacity = interpolate(distance, [-3000, -2000, 150, 500], [0, 1, 1, 0], {
            extrapolateLeft: "clamp",
            extrapolateRight: "clamp",
          });
          return (
            <div
              key={i}
              style={{
                position: "absolute",
                left: "50%",
                top: "50%",
                transform: `translate(-50%, -50%) translate3d(${card.x}px, ${card.y}px, ${card.z}px) rotateY(${card.rotateY ?? 0}deg)`,
                opacity,
                boxShadow: "0 30px 80px rgba(0,0,0,0.5)",
                borderRadius: 20,
                overflow: "hidden",
              }}
            >
              {card.node}
            </div>
          );
        })}
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```
