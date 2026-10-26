# CameraTD

Hand-controlled morphing point clouds over a live webcam feed, built in TouchDesigner.

Open your right hand and a model appears inside a spinning wireframe cage. Open your left hand and the particles flow out like smoke; close it and they swim back together, morphing into the next model.

## Requirements

- TouchDesigner 2025.33230 or newer
- A webcam
- The [MediaPipe TouchDesigner plugin](https://github.com/torinmb/mediapipe-touchdesigner) v0.5.3

## Setup

1. Download `release.zip` from the plugin's [v0.5.3 release](https://github.com/torinmb/mediapipe-touchdesigner/releases/tag/v0.5.3).
2. Unzip it into a `mediapipe` folder next to `Camera-TD.toe`, so this file exists:
   `mediapipe/toxes/MediaPipe.tox`
3. Open `Camera-TD.toe`. The final output is `/project1/OUT`.

## Controls

| Input | Action | Result |
|---|---|---|
| Right hand | Open | Cage and model grow in |
| Right hand | Close | Both disappear |
| Left hand | Open | Particles flow out in a swirling cloud |
| Left hand | Close | Particles flow back and morph to the next model |
| Keyboard `1` | Hold / release | Explode / morph (works without hand tracking) |

If the hands come out swapped, open `hand_open_callbacks` and exchange the values of `SHOW_HAND` and `EXPLODE_HAND`.

## How it works

- **Point clouds as textures.** Each model is loaded with a Point File In TOP, once for XYZ position and once for RGB colour. Two Fit TOPs resample every model to 800 x 800 = 640,000 points, so any two models can morph point for point.
- **Morphing.** Switch TOPs with input blending pick between models; a Count CHOP steps the index each time the explode signal is released.
- **Fluid explosion.** `flow_sim` is a GLSL feedback simulation: each frame particles are advected along a curl-noise flow field, with a spring pulling them home (loose while exploding, firm when released) and a leash limiting how far they can go.
- **Hand tracking.** `hand_open` measures how open each hand is (mean fingertip-to-wrist distance divided by palm length) from the MediaPipe landmarks.
- **Rendering.** The positions and colours instance point sprites on a Geometry COMP, rendered over the camera feed with a light bloom.

## Tuning

The `settings` CHOP holds the main controls:

| Channel | Default | Effect |
|---|---|---|
| `spin` | 45 | Rotation speed, degrees per second |
| `tumble` | 18 | How far the cage tips back and forth |
| `spread` | 1.2 | How far the cloud reaches from the model |
| `flow_speed` | 0.5 | How fast particles stream |
| `swirl_size` | 0.8 | Swirl scale: lower = bigger, lazier eddies |
| `return_speed` | 3.0 | How quickly particles flow back on release |

If the particles ever end up scrambled, pulse **Reset** on `flow_feedback`.

## Adding models

Put a `.ply` point cloud (ideally with `red`/`green`/`blue` colour) in `pointclouds/`, then in the Textport (Alt+T):

```python
op('/project1/models').module.add_model('C:/path/to/model.ply')
```

This copies an existing model's chain, points it at the new file, wires it into the switches and widens the morph loop. `list_models()` prints what's loaded. Use the model's `ptransform_<name>` node to rotate or scale it.

Meshes (`.fbx`, `.obj`, `.glb`) need converting to a point cloud first, for example with a Sprinkle POP and Lookup Texture POP in TouchDesigner, or by exporting points from Blender or CloudCompare.

The sample models (`galaxy.ply`, `torusknot.ply`, `helix.ply`) are generated. The walking character model is not included in this repo; the project will show a missing-file warning for `pointclouds/Strut Walking.ply` until you add your own model there or remove that input.

## Credits

- Effect based on wxll's tutorial [morphing particles and pointclouds in touchdesigner using TOPs](https://youtu.be/sE716KGZw0w).
- Hand tracking by Torin Blankensmith's [MediaPipe TouchDesigner](https://github.com/torinmb/mediapipe-touchdesigner).
