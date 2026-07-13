# The Voice Line

Paste this whole prompt into Claude Code on a Mac. Your agent builds a voice conversation system: hold a key, talk to your agent out loud, release, and it answers through your speakers in a real voice. Type instead whenever you want, same conversation. Everything runs local, $0 on a Claude subscription, and your agent will offer you a choice: the free local voice, or any voice you like from ElevenLabs if you want the premium sound.

---

I want you to build me a voice conversation system for talking to you out loud at my desk. Build it exactly to this spec. It is a proven design, so where this spec is specific, follow it precisely. The gotchas listed at the bottom are hard-won lessons. Read them before writing any code.

## What it does

I hold a key anywhere in macOS, my mic opens, I talk. I release the key, and what I said gets transcribed locally, sent to a live Claude Code session (you, with all your tools and my project context), and your reply is spoken through my speakers in a natural voice, sentence by sentence as you generate it. First audio should land about 1 to 2 seconds after I release the key on warm turns.

## Architecture (half-duplex)

```
mic -> ears (sounddevice capture)
    -> local whisper server on port 2022 (/v1/audio/transcriptions)
    -> warm Claude Agent SDK session (streaming, one client per session)
    -> mouth (sentence-chunked TTS via local Kokoro on port 8880, cancellable playback)
    -> speakers
```

Half-duplex means the mic is gated while the mouth is speaking so the system never hears itself. Do not attempt barge-in on open speakers.

## Project layout

Create the project at `~/voice-line/` with a uv-managed Python 3.12 environment:

- `main.py` entry point, the turn loop, hold-to-talk wiring
- `ears.py` mic capture and transcription
- `brain.py` the warm Claude Agent SDK session
- `mouth.py` TTS queue and playback
- `ptt.py` the global hold-to-talk key listener
- `ducking.py` optional Spotify ducking
- `signals.py` the visualizer signal bus (spec below)
- `run-voice-line.sh` launcher

Python packages: sounddevice, webrtcvad, numpy, pynput, claude-agent-sdk, httpx. Pin setuptools below 81 because webrtcvad needs pkg_resources. You also need the ffmpeg binary available.

## Services this rides on

Two local servers must exist. Check for them and set them up if missing:

1. A whisper.cpp server on port 2022 exposing the OpenAI-style route `/v1/audio/transcriptions` with a small English model and Metal acceleration. Note: the `/inference` route is not the one you want, it 404s on the OpenAI client path.
2. A Kokoro TTS server on port 8880 exposing `/v1/audio/speech` (kokoro-fastapi works well). Request `response_format: pcm`, which returns raw int16 at 24kHz mono. Pick a voice you like, `bm_lewis` is a good default.

## Hold-to-talk (the default mode)

- Global key listener via pynput. Holding the chosen key opens the mic, releasing closes it with a 0.18 second tail so my last word survives.
- Taps shorter than 250ms are ignored.
- CRITICAL: macOS fires key-repeat on_press events continuously while a key is held. Filter with a held-state flag or every repeat becomes a fresh press. This bug will kill every reply before it speaks if you skip it.
- Pressing the key while the assistant is talking interrupts playback immediately. This makes it speaker-safe with no headphones.
- The mic is fully closed between holds so room audio and music never leak into the transcriber.
- Needs Input Monitoring permission for the hosting terminal, and mic permission comes from launching in Terminal, not launchd. Never run this as a background daemon. Nobody wants a 24/7 open mic.

Also build a legacy open-mic mode behind an `--open-mic` flag: webrtcvad endpointing, discard any utterance with less than 240ms of actual speech, and strip bracketed non-speech markers like [SIGHS] and [BLANK_AUDIO] that whisper emits.

## The brain

- One warm ClaudeSDKClient per voice session, created at launch. Warm turns are fast; the first turn pays a prompt-cache toll of several seconds, so hide it behind a spoken greeting by firing a warmup query at startup.
- Set the session cwd to the project folder whose CLAUDE.md defines your identity, so the voice session is the same assistant as my terminal sessions.
- Use the system prompt preset with an appended spoken-discipline block: short conversational sentences, no markdown, no code blocks, no lists read aloud, write for the ear. TTS performs punctuation, so dull text is dull audio. After the first sentence, ship sentences in two-sentence breaths, since lone short sentences sound flat.
- Stream partial messages. Chunk into sentences and hand each completed sentence to the mouth immediately.
- CRITICAL: flush the sentence buffer when a content block stops. If you only flush on sentence-ending punctuation, pre-tool filler like "On it, checking now" sits silent through the whole tool run and then plays glued to the answer.
- Quit phrases end the session: "goodbye", "end voice mode", "hang up". Ctrl-C also works.

## The mouth

- A queue of sentences, synthesized one at a time, played back to back, cancellable mid-stream (interrupt = clear queue + stop playback now).
- **The voice is my choice. Before you build the mouth, ask me:** free local voice (Kokoro, the default, $0 forever), or a higher quality ElevenLabs voice using my own ElevenLabs API key. If I pick ElevenLabs, walk me through creating the key, point me at their voice library so I can pick ANY voice I like (the voice id is one setting in the code), store the key in an ELEVENLABS_API_KEY env var (never hardcode it), and keep Kokoro wired in as the automatic fallback so if ElevenLabs is down or the key runs out of credits the voice degrades instead of going mute.
- Kokoro path: POST the sentence, get raw PCM int16 24kHz mono, play through sounddevice.
- ElevenLabs path, hard-won audio doctrine: fetch mp3_44100_128 and decode locally with ffmpeg (raw PCM at 44.1k needs their Pro tier, and the mp3 decode hides inside network wait). Use the turbo model with stability 0.5 and similarity 0.75. Do not use the multilingual model for English and do not set style above 0, both make delivery slow and dull. Their website voice previews are mastered demo clips, raw API output never matches them, so master locally with an ffmpeg chain: presence boost around 3.2kHz, a little low shelf around 140Hz, gentle compression, a limiter.
- While audio is playing, feed the signal bus (below) with each PCM block.

## Typed input (first-class, not a side channel)

Typing in the voice terminal is a real turn: a background reader feeds typed lines into the exact same handler as speech, so the reply is spoken aloud, typing while it talks interrupts playback, and the quit phrases work typed. Two hard-won rules. First, race the typed queue against the key listener with asyncio.wait FIRST_COMPLETED and keep unfinished futures alive across iterations. Second, take the terminal raw (cbreak, kernel echo off, restore on exit) and run your own tiny line editor, because canonical mode cannot host paste-aware input: assemble bracketed pastes invisibly into ONE message no matter their shape, scrub gutter glyphs and hard wraps out of pasted text, and echo a long paste as a character count instead of the text.

## Spotify ducking (optional but great)

While the assistant speaks, if Spotify is playing above volume 30, drop it to max(30, current x 0.6) via AppleScript. Restore with a 1.2 second debounce so back-to-back sentence chunks do not yo-yo the volume. Never launch Spotify if it is not running.

## The signal bus (for the visualizer)

Write these files in the project root so a separate visualizer can watch them. My visualizer prompt builds a browser scene that reads this bus through its own small read-only server, but the contract is just files, so anything can watch them. Your only job is to write them, all writes wrapped in try/except, the bus must never crash the voice line. (One file you never write: `.voice_alert`. That one belongs to any OTHER process on the machine that wants the visualizer's attention.)

- `.voice_state` plain text, one of: idle, listening, thinking, speaking
- `.voice_waveform` JSON `{"ts": <unix float>, "samples": [64 floats]}`, written at most 15 times per second while audio plays. Downsample each PCM block to 64 points, raw int16 magnitudes are fine.
- `.voice_loading_pid` exists while an optional thinking sound is playing

State transitions: key press writes listening, key release writes thinking, first audio block writes speaking, playback end writes idle. CRITICAL self-heal rule: every waveform write also re-writes state to speaking. This only runs while audio is audibly playing, and it means any stray process that stomps the state file gets corrected within about 70ms. This one rule fixed a bug that took a whole evening to find.

## Verify before you call it done

1. Both local servers respond.
2. A full turn end to end: hold, speak, release, hear the reply.
3. Interrupt works mid-reply.
4. A tool-using turn speaks filler within a couple of seconds, then the answer.
5. Nothing plays twice and the mic never hears the speakers.

Then show me the launch command and a one-page cheat sheet of the controls.
