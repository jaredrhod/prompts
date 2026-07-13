# The Cinematic Camera

Paste this into the same Claude Code agent that built your visualizer from The Visualizer prompt. It adds the show-off button: press the spacebar and a scripted, movie-style flythrough renders over your live scene while it listens, thinks, and speaks. This is the flyover from my demo video, as a framework your agent designs around YOUR scene.

---

I want you to add a cinematic presentation camera to the visualizer you built me from The Visualizer prompt. It is the show-off button: when I press Space, a roughly 30 second scripted flythrough renders OVER the live simulation, so whatever the scene is doing is what the lens sees, nothing pre-rendered. Pressing Space again bails out cleanly. If my visualizer was not built from that prompt, read my scene code first and adapt; everything below works on any canvas scene.

## Design the shots around MY scene. Ask me first.

Before writing code, study the scene and propose a small pool of shot types that fit it: a long glide along its main feature, a slow push into a rich detail, a corner-to-corner diagonal, a vertical descent, whatever the scene offers. Get my OK on the pool before you build.

Each press of Space deals a random hand from the pool (random subjects, directions, durations) so no two runs look alike, and every run ends on the same finale: a pull-back that lands on the full wide scene. Two rules in the dealer: never cut from a subject straight to the same subject, and never let the finale's landing subject be the shot right before the finale.

## Camera grammar (the difference between cinema and a screensaver)

- Constant-speed drift within every shot. The camera never sits still, and cuts land mid-motion. Per-shot ease-in and ease-out reads as the camera pausing at every cut, which kills it.
- The only deceleration in the whole run is the finale easing in to land.
- One apparent speed across the whole deck: derive each shot's duration from its travel distance and zoom, or random durations will make some pans visibly sprint.
- A slight perspective tilt during the run sells the flyover. Arrive already tilted from the first frame of the run (easing the tilt in after a cut reads as the world warping) and flatten it in sync with the final pull-back, so the landing and the un-tilt are one motion.

## Architecture

- During the run, re-render the world through the camera transform every frame instead of scaling a prebaked image, and cull to what is in frame. That keeps close-ups infinitely crisp and cheap.
- Keep the scene's real activity flowing through whatever the lens is on, so macro shots are never dead air.
- Hide the HUD chrome during the run and bring it back on landing. Screen-space effects (grain, vignette, sheen) stay in screen space, not world space.
- Canvas shadows do not scale with a zoom transform. If the scene uses them, scale the shadow parameters by the zoom factor or they shrink as you dive in.

## Verify before you call it done

1. A full run over the mock loop: cuts land mid-motion, the camera never stops, the finale lands on the full wide scene.
2. A second Space mid-run bails out cleanly back to the normal view.
3. Runs look different press to press, and the finale never repeats the previous shot's subject.
4. Extend the shot harness: a URL parameter that freezes a flythrough at a given time, so you can screenshot mid-run frames with headless Chrome and EYEBALL your own work.
5. Fullscreen still holds 60fps through the heaviest wide shot.

Then show me the button.
