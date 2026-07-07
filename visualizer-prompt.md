# The Visualizer (Face in the Code)

Paste this whole prompt into Claude Code on a Mac. Your agent builds a Matrix-style digital-rain visual that idles like a screensaver until your voice agent speaks, then a face surfaces inside the falling code and breathes with the live voice. Built to pair with The Voice Line prompt, but it ships with its own test driver so you can build and enjoy it standalone.

---

I want you to build me a stream visualizer. Matrix-style digital rain that reacts to a voice assistant through a small file-based signal bus. Build it exactly to this spec. It is a proven design.

## The look

A fullscreen field of falling glyphs, phosphor green on near-black, like the film: bright glyph at the head of each column, fading trail behind it. It idles like a screensaver. When the voice speaks, a face surfaces INSIDE the code: the glyphs themselves brighten through a portrait's luminance mask, so the face is made of code, not layered on top of it. The face breathes with the live voice envelope and dissolves back into rain when the voice stops.

## Project layout

Create the project at `~/voice-visualizer/` with a uv-managed Python 3.12 environment. One main file `visualizer.py` (pygame, 60fps), an `assets/` folder, and `tools/test-drive.py`.

Assets:
- `assets/face.png` a grayscale portrait on a black background. The loader must crop to content and normalize, so ANY portrait works: swap the file, restart, new face. If I do not provide one, generate a placeholder and tell me how to swap it.
- `assets/VT323-Regular.ttf` (free on Google Fonts) for UI lettering and the boot intro.
- Rain glyphs come from a system font with katakana coverage (Arial Unicode on macOS), and mirror the glyphs horizontally like the film.

## The signal bus (the contract)

Watch these files in the voice project folder at `~/voice-line/` (make the path a constant at the top):

- `.voice_state` plain text: idle, listening, thinking, or speaking
- `.voice_waveform` JSON `{"ts": <unix float>, "samples": [64 floats]}` written about 15 times per second while the voice plays. Treat it as live only if ts is within 0.4 seconds. Peak-normalize the samples on read.
- `.voice_alert` if this file exists, show the alert state

CRITICAL stomp-tolerance rule: a live waveform means the voice is speaking, no matter what the state file says. Trust the waveform over the state. This protects the show from any stray process overwriting the state file mid-speech.

## The states

- **idle** the rain falls at its screensaver pace.
- **listening** an amber ribbon renders along the bottom, driven by MY live mic level (open the mic locally in the visualizer for this, the bus does not carry input audio).
- **thinking** the rain accelerates to about 2.6x, occasional glitch bursts tear across columns, and a PROCESSING label descrambles character by character in the corner.
- **speaking** the face surfaces. Map the portrait's luminance onto a glyph grid noticeably FINER than the rain grid, that contrast is the whole look. Modulate face brightness with the voice envelope so it breathes as it talks. Dissolve in over a few hundred ms, dissolve back to rain when speech ends.
- **alert** the code bleeds red with a dark vignette, for anything else on the machine that wants attention.

## Details that make it feel right

- A short boot intro on launch: a few typed-out terminal lines in VT323, any key skips it.
- Keys: F toggles fullscreen, ESC or Q quits.
- A tiny state tag in a corner so I always know what state it thinks it is in.
- Prerender glyph surfaces at a handful of brightness levels and cache them. Never render fonts per frame, that is the difference between 60fps and a slideshow.
- A dev hook: if a screenshots env var is set, save a frame every ~3 seconds so you can verify your own work without a window manager.

## The test driver

`tools/test-drive.py` fakes a full voice turn on the bus: idle, then listening, then thinking, then speaking with a synthetic waveform at 15Hz, then alert, then back to idle. Give it a `--once` flag. Warn in its help text never to run it while the real voice line is up, two writers on one bus means chaos.

## Verify before you call it done

1. Launch the visualizer and the test driver together and watch it walk through all five states.
2. The face is clearly recognizable during speaking and clearly gone during idle.
3. Fullscreen holds 60fps.
4. Kill the test driver mid-speaking and confirm the visualizer recovers to idle on its own within a second (the staleness rule).

Then show me the launch command.
