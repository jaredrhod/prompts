# The Visualizer

Paste this whole prompt into Claude Code on a Mac. Your agent builds a fullscreen visual that reacts to your voice agent as it listens, thinks, and speaks. You pick the scene: mine is Matrix-style digital rain where a face surfaces inside the falling code, but the same engine can drive anything you can describe. Built to pair with The Voice Line prompt, and it ships with its own test driver so you can build and enjoy it standalone.

---

I want you to build me a stream visualizer: a fullscreen visual that reacts to a voice assistant through a small file-based signal bus. The engine spec below is proven. The scene that runs on it is mine to choose.

## The scene is my choice. Ask me first.

Before you design or write anything, ask me what I want on screen. Give me a few examples to spark ideas: Matrix-style digital rain where a face surfaces inside the code when the voice speaks, a truck dashboard where a CB radio needle dances with the voice, a starfield that warps to light speed while it thinks, a neon city skyline that pulses like an equalizer, a fireplace that flares as it talks. Anything I can describe, you can build with this engine.

Once I pick, propose how MY scene will express each of the five states below, and get my OK on the design before you write code.

## The five states (design these around my scene)

- **idle** the scene at rest, like a screensaver. Alive but calm, something I can leave on screen all day.
- **listening** a clearly visible "I hear you" element, driven by MY live mic level (open the mic locally in the visualizer for this, the bus does not carry input audio).
- **thinking** the scene visibly working: more speed, more energy, a burst of activity, plus some kind of processing indicator that fits the theme.
- **speaking** the centerpiece. The scene's main element comes alive and moves with the live voice envelope, so the visual breathes with every word. This is the star of the show, make it unmistakable.
- **alert** an unmistakable emergency treatment (a red bleed, a warning flare, whatever fits the scene), for anything else on the machine that wants attention.

## Project layout

Create the project at `~/voice-visualizer/` with a uv-managed Python 3.12 environment. One main file `visualizer.py` (pygame, 60fps), an `assets/` folder for whatever the scene needs, and `tools/test-drive.py`.

If my scene uses a portrait or image (like my face-in-the-code), the loader must crop to content and normalize so ANY swapped image works. If I do not provide assets, generate placeholders and tell me how to swap them.

## The signal bus (the contract)

Watch these files in the voice project folder at `~/voice-line/` (make the path a constant at the top):

- `.voice_state` plain text: idle, listening, thinking, or speaking
- `.voice_waveform` JSON `{"ts": <unix float>, "samples": [64 floats]}` written about 15 times per second while the voice plays. Treat it as live only if ts is within 0.4 seconds. Peak-normalize the samples on read.
- `.voice_alert` if this file exists, show the alert state

CRITICAL stomp-tolerance rule: a live waveform means the voice is speaking, no matter what the state file says. Trust the waveform over the state. This protects the show from any stray process overwriting the state file mid-speech.

## Details that make it feel right

- A short boot intro on launch that fits the theme (mine types out terminal lines), any key skips it.
- Keys: F toggles fullscreen, ESC or Q quits.
- A tiny state tag in a corner so I always know what state it thinks it is in.
- Prerender and cache anything expensive (glyph surfaces, sprites, gradients). Never render fonts or rebuild big assets per frame, that is the difference between 60fps and a slideshow.
- A dev hook: if a screenshots env var is set, save a frame every ~3 seconds so you can verify your own work without a window manager.

## The test driver

`tools/test-drive.py` fakes a full voice turn on the bus: idle, then listening, then thinking, then speaking with a synthetic waveform at 15Hz, then alert, then back to idle. Give it a `--once` flag. Warn in its help text never to run it while the real voice line is up, two writers on one bus means chaos.

## Verify before you call it done

1. Launch the visualizer and the test driver together and watch it walk through all five states.
2. The speaking centerpiece clearly reacts to the voice envelope and is clearly gone during idle.
3. Fullscreen holds 60fps.
4. Kill the test driver mid-speaking and confirm the visualizer recovers to idle on its own within a second (the staleness rule).

Then show me the launch command.
