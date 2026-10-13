# valentina-mkrtu.github.io

Technical notes on AI task handoffs and web-game effects.

Blog: https://valentina-mkrtu.github.io/

## PixiJS rain splash at a known impact point

This recipe creates a small splash when game logic reports a raindrop hitting a surface. It uses a known impact position; it does not infer arbitrary scene geometry or simulate water. AI generated this draft. The authoring values are proposed starting points and have not been visually benchmarked.

1. Create a one-shot effect named `rain-splash` for `pixi-ui-2d`. Use a point emitter, no continuous emission, and a burst of six particles. Start with lifetimes of 0.12–0.25 seconds and small horizontal velocity variations. Give the particles a brief upward movement followed by downward acceleration in the host's coordinate convention.
2. Use a pale blue-gray billboard with alpha blending. Fade opacity over its lifetime and keep the splash lower than the foreground characters. The [pixi js particle editor integration guide](https://nixiefx.com/pixijs-particle-effects/) explains the export bundle and texture-provider workflow.
3. Validate, inspect the `pixi2d` report, and serve the complete export with the game. Preload any referenced texture before spawning. Convert each impact point into the VFX layer's local coordinates, then create one instance for that event.
4. Advance the owning renderer once per frame in seconds. Remove completed instances and destroy the renderer when the scene leaves. Cap the number of simultaneous splashes when rain density rises.

Check one impact, repeated impacts, a moving camera, and scene re-entry. Record the actual runtime versions and observed behavior. A splash that repeats every frame while a drop rests on the ground indicates a trigger problem.

Unsupported for this recipe: Three.js lit shading and mesh-surface emission. PixiJS draw ordering is not GPU depth testing. Editor bloom is not an exported postprocessing pipeline. The six-particle choice is not a mobile performance guarantee. Keep runtime validation pending until an actual exported asset has been integrated.
