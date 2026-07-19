> Community contribution: the Windows port of The Voice Line, built and documented by **joatsaint** (jaredrhod Discord), July 2026. His document is preserved verbatim below.

# Build Your Own Jarvis on Windows — Claude Code Prompt

**Source:** This is a Windows-hardened port of Jared Rhod's "Voice Line" prompt
(jaredrhod.com/prompts), written for macOS. Full credit to Jared for the
original architecture and concept — everything below documents the real,
hard-won differences hit building the exact same thing on Windows, so you
don't have to rediscover them. If you're on a Mac, use Jared's original.

---

## What you're building

Hold a key anywhere, talk, release, hear Claude Code reply out loud — a real
voice-driven conversation with your Claude Code session, not a toy demo.

```
mic -> ears (audio capture)
    -> local Whisper server (speech-to-text)
    -> warm Claude Agent SDK session (streaming)
    -> mouth (sentence-chunked text-to-speech)
    -> speakers
```

Half-duplex: the mic is gated while the mouth is speaking, so the system
never hears and responds to itself.

---

## Paste this to Claude Code to start the build

> I want to build a hold-to-talk voice assistant that lets me talk to a
> Claude Code session out loud and hear it respond, on Windows. Architecture:
> mic capture -> local Whisper server for speech-to-text -> a warm Claude
> Agent SDK session (streaming) -> local Kokoro TTS for speech synthesis ->
> speakers. Half-duplex — mic must be gated while audio is playing back so
> the system doesn't hear itself. I'll walk through setup for the two local
> servers (Whisper, Kokoro) separately. For now, scaffold the project:
> hold-to-talk key capture (global, works even when the terminal isn't
> focused), a threaded approach to reading keyboard/typed input (NOT
> `asyncio.add_reader()` on stdin — that raises `NotImplementedError` on
> Windows because the default `ProactorEventLoop` doesn't support it), and a
> two-stage audio pipeline where synthesis of the next sentence overlaps
> playback of the current one instead of running fully sequentially.

---

## Real Windows-specific differences from Jared's original (found building this)

- **No `termios`/`cbreak`.** Windows consoles don't support POSIX raw mode.
  Use `msvcrt` for character-level input instead, plus
  `ENABLE_VIRTUAL_TERMINAL_INPUT` if you want Windows Terminal's
  bracketed-paste sequences detected too.
- **No macOS Input Monitoring permission gate.** A global key listener
  (e.g. `pynput`) works out of the box on a normal Windows console process —
  just make sure microphone access is allowed for "Desktop apps" under
  Windows Settings > Privacy & security > Microphone.
- **App-volume ducking uses `pycaw`** (Windows Core Audio session API), not
  AppleScript — controls a specific app's own volume slider directly, the
  real Windows equivalent.
- **`NotImplementedError` on startup, the single most likely first bug you
  hit:** Windows' default asyncio event loop (`ProactorEventLoop`, required
  for the Claude Agent SDK's subprocess handling) does not implement
  `add_reader()`. If your input-reading code uses that, it will crash
  immediately. Fix: read keystrokes on a background thread instead, and feed
  them into the asyncio loop via `call_soon_threadsafe`.
- **Long silences between every spoken sentence, if you build TTS naively
  first:** synthesizing and playing each sentence fully sequentially
  produces a real, audible gap at every sentence boundary. Fix: a two-stage
  pipeline — synthesis of the next sentence overlaps playback of the
  current one, via two queues instead of one linear loop.
- **Whichever local Whisper server build you use, verify the actual
  transcription route before assuming it matches documentation.** Different
  builds expose different endpoints — one real build tested here only
  exposed `/inference`, not the OpenAI-compatible `/v1/audio/transcriptions`
  route some docs assume exists. Test the real endpoint with `curl` before
  wiring your code to it.
- **If you use Kokoro-FastAPI for TTS: its own GPU launch script may not
  actually get you the GPU build.** One real, confirmed case: the script ran
  `uv pip install -e ".[gpu]"`, which is `uv`'s pip-compatible legacy mode —
  that mode does NOT apply the project's `[tool.uv.sources]` custom
  PyTorch-CUDA index routing (a `uv`-project-mode-only feature), so it
  silently resolved a plain, unpinned CPU-only torch build instead of the
  pinned CUDA one. Even `uv sync --extra gpu` (the "correct" project-mode
  command) resolved the wrong version in this case, due to the project's
  marker-based dependency conflicts. What actually worked: install the
  exact pinned CUDA wheel directly, bypassing the extras/marker resolution
  entirely — e.g. `uv pip install "torch==<version>+cu126" --index-url
  https://download.pytorch.org/whl/cu126`. **Always verify** with
  `torch.cuda.is_available()` before assuming GPU is active — this bug is
  silent and the app still runs, just 10-14x slower, on CPU.
- **A false-trigger risk if you build always-listening mode:** with
  always-on voice active and a separate voice-to-text feature also engaged
  (e.g. Claude Code's own built-in voice mode via spacebar), audio playing
  in the background (a video, music) can get picked up as user speech and
  trigger a response to dialogue that was never meant for the assistant.
  This is the concrete, real argument for push-to-talk over always-on — see
  activation modes below.

---

## Three activation-mode options — pick one, know the tradeoffs

1. **Always-on background listening.** Pros: zero-latency start, no manual
   step required. Cons: the false-trigger risk above is real, not
   theoretical; also runs continuously in the background using system
   resources.
2. **Push-to-talk (what this build actually uses).** Pros: eliminates false
   triggers entirely, fully predictable — it only hears you when you want
   it to. Cons: requires a manual key press every time you want to talk.
3. **Wake-word activation** ("Hey Jarvis"-style). Not built here, but a real
   third option worth knowing about. Requires an additional always-listening
   wake-word detection model running locally — which reintroduces a
   narrower version of the same false-trigger risk class as always-on, just
   scoped to a smaller listening window. No strong recommendation either
   way — depends on how much you value hands-free convenience vs. the
   false-trigger risk.

---

## Running it as a real Windows service (survives reboots)

Installing a Windows service always requires Administrator privileges — no
way around this. If you want your voice assistant's backend servers to
auto-start on boot and auto-restart if they crash, install them via a tool
like NSSM (Non-Sucking Service Manager) from an elevated PowerShell window.
Two real gotchas hit doing this:

- If your speech-to-text server has an optional format-conversion flag that
  depends on `ffmpeg`, don't assume `ffmpeg` is on PATH for the service
  account — `LocalSystem` doesn't inherit your own user PATH. Either add
  `ffmpeg` explicitly, or drop the conversion flag if your audio is already
  in the right format going in.
- If you're re-running an install command against a service that already
  exists (e.g. from an earlier failed attempt), be aware that some service
  tools only apply command-line-argument changes when *creating* a brand
  new service — re-running the same install line against an existing
  service can silently no-op on updating the actual command/arguments.
  Explicitly force-set the application path and arguments every time your
  install script runs, not just on first creation.

---

*Built and documented 2026-07, adapting Jared Rhod's original "Voice Line"
prompt (jaredrhod.com/prompts) to Windows. Share freely, keep the credit.*
