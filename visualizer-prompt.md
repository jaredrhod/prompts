# The Visualizer

Paste this whole prompt into Claude Code on a Mac. Your agent builds a fullscreen visual that reacts to your voice agent as it listens, thinks, and speaks. You pick the scene: mine is a living circuit board where the voice is the current running through it, but the same engine can drive anything you can describe. Built to pair with The Voice Line prompt, and it ships with a mock mode so you can build and enjoy it standalone.

---

I want you to build me a stream visualizer: a fullscreen browser scene that reacts to a voice assistant through a small local server. The engine spec below is proven. The scene that runs on it is mine to choose.

## The scene is my choice. Ask me first.

Before you design or write anything, ask me what I want on screen. Give me a few examples to spark ideas: a living circuit board where signals race the traces toward a glowing chip, Matrix-style digital rain where a face surfaces inside the code, a truck dashboard where a CB radio needle dances with the voice, a starfield that warps to light speed while it thinks, a neon city skyline that pulses like an equalizer, a fireplace that flares as it talks. Anything I can describe, you can build with this engine.

Once I pick, propose how MY scene will express each of the five states below, and get my OK on the design before you write code.

## The five states (design these around my scene)

- **idle** the scene at rest, like a screensaver. Alive but calm, motion at maybe 20%, something I can leave on screen all day.
- **listening** a clearly visible "I hear you" treatment: steady, clearly busier than idle, and reading as inbound attention rather than output. Driven by the bus state, the visualizer never opens a mic.
- **thinking** the scene visibly working: more speed, more energy, a burst of activity, plus some kind of processing indicator that fits the theme.
- **speaking** the centerpiece. The scene's main element comes alive and moves with the live voice level, so the visual breathes with every word. This is the star of the show, make it unmistakable.
- **alert** an unmistakable emergency treatment (a red bleed, a warning flare, whatever fits the scene), for anything else on the machine that wants attention.

## Project layout

Create the project at `~/voice-visualizer/`:

- `index.html` the entire scene. One self-contained file: plain canvas 2D and vanilla JS, no frameworks, no CDN, no build step, works offline. Prefer drawing everything procedurally so nothing external is required.
- `server.py` a tiny Python 3 server, standard library only, no packages, no environment to manage. The python3 already on the Mac is fine.
- `Visualizer.command` a double-click launcher.
- `assets/` only if the scene truly needs an image. If my scene uses a portrait or image, the loader must crop to content and normalize so ANY swapped image works. If I do not provide assets, generate placeholders and tell me how to swap them.

## The server (the only bridge to the voice line)

`server.py` binds 127.0.0.1 on port 8777 and does exactly two jobs: serve the page, and serve `/state` as JSON, `{"state": "idle|listening|thinking|speaking", "level": 0.0 to 1.0, "alert": true|false}`, by reading the voice line's signal bus files in `~/voice-line/` (make the paths constants at the top):

- `.voice_state` plain text: idle, listening, thinking, or speaking
- `.voice_waveform` JSON `{"ts": <unix float>, "samples": [64 floats]}`. Compute level from the mean absolute sample, scaled to 0 to 1. Treat it as live only if ts is within about 2 seconds.
- `.voice_alert` alert is true while this file exists

The server is strictly READ-ONLY on the bus. It never writes those files. CRITICAL stomp-tolerance rule: a live waveform means the voice is speaking no matter what the state file says, so `/state` reports speaking whenever the waveform is fresh. This protects the show from any stray process overwriting the state file mid-speech.

The page polls `/state` about 10 times a second and smooths everything client-side (about 90ms on level). If `/state` stops answering or goes stale, ease the scene back to idle on its own. Add an optional samples passthrough if my scene wants a real oscilloscope.

## The launcher

`Visualizer.command`: if port 8777 is not answering, start `server.py` in the background (log to /tmp), then open a Chrome kiosk at the localhost address with a fresh throwaway `--user-data-dir` profile so no tabs or extensions ride along. If Chrome is missing, open the default browser and tell me to go fullscreen. Cmd+Q closes the kiosk; the server stays warm.

## Details that make it feel right

- ONE continuously eased energy value drives the whole scene (fast attack around half a second, slower release around one second). Every state change rides that curve: glow, speed, color, everything. A hard step, or two competing systems, reads as a jump cut and kills the illusion.
- Glow energy and motion energy are separate axes. Speaking wants full glow at a calm cruise; thinking wants full speed. "Alive" and "fast" are different things.
- Bake anything expensive once to an offscreen canvas at load and blit it per frame; animate only the moving layer. Cap devicePixelRatio around 1.25 on big displays. That is the difference between 60fps and a slideshow.
- A short boot intro on launch that fits the theme, any key skips it.
- A tiny state tag in a corner so I always know what state it thinks it is in, and an FPS meter on the F key.

## The mock harness (never fake data on the real bus)

Two writers on one bus means chaos, so the test path never touches it:

- `server.py --mock` on port 8778 serves the same page, but its `/state` walks a scripted loop: idle, listening, thinking, speaking with a synthetic breathing level, alert, back to idle. This is also how I enjoy the scene standalone without the voice line running.
- `?mockstate=speaking` (any state) in the URL makes the page simulate that state locally without touching `/state` at all.
- `?shot=speaking&t=1200` (any state, any ms) renders the scene deterministically, advances the simulation t milliseconds, and freezes. That is your self-verification hook: screenshot it with headless Chrome and EYEBALL your own frames as you iterate. Do not guess at visuals. Gotcha: headless Chrome fires a late initial resize that can clear the canvas after your synchronous render, so re-render on resize or your screenshots come back black.

## Verify before you call it done

1. Run the mock server and watch the full loop: all five states, in and out, no jump cuts anywhere.
2. The speaking centerpiece clearly rides the level and is clearly gone during idle.
3. The fullscreen kiosk holds 60fps (check the F meter).
4. Kill the mock server mid-speaking and confirm the scene eases back to idle on its own within a couple of seconds (the staleness rule).
5. After any server edit, verify you are not talking to a stale instance: an old process can hold the port and keep answering with old code. Find the PID with lsof on the port and kill exactly that, never trust a pattern pkill.

Then show me the double-click launcher.
