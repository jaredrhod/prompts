# Prompts

Prompts you paste into Claude Code. Your agent builds the thing.

These are the actual builds I run every day, written up as prompts your own agent can follow. Every one of them carries the hard-won lessons from my real build, so your agent gets it right the first time instead of losing the same evenings I lost. Free, no email, no catch.

The pretty version with copy buttons lives at [jaredrhod.com/prompts](https://jaredrhod.com/prompts). This repo is the same content in raw markdown, plus the license.

## How to use

1. Open Claude Code on your computer (Mac, Windows, or Linux).
2. Copy the whole prompt file and paste it in.
3. Let it build. Your agent installs what it needs, writes the code, and verifies it works before handing it to you.

## The prompts

- **[The Voice Line](voice-line-prompt.md):** hold a key, talk to your agent out loud, release, and it answers through your speakers in a real voice. Type instead whenever you want, same conversation. Everything runs local, $0 on a Claude subscription, and your agent will offer you a choice: the free local voice, or any voice you like from ElevenLabs if you want the premium sound.
- **[The Voice Line: Windows Edition](voice-line-prompt-windows.md):** the same proven build, written native for Windows and hardened with the real fixes from a full Windows build contributed by joatsaint (input handling, the silent CPU-instead-of-GPU trap, dead-air-free speech, running the servers as services). His original write-up is preserved verbatim in [windows-voice-line.md](windows-voice-line.md). The OS toggle on jaredrhod.com/prompts serves the right edition automatically.
- **[The Visualizer](visualizer-prompt.md):** a fullscreen browser scene that reacts to your voice agent as it listens, thinks, and speaks. You pick the scene: your agent asks what you want on screen before it builds anything. Mine is a living circuit board where the voice is the current running through it, and you can watch it run below. Pairs with The Voice Line out of the box (they share a signal-bus contract), and it ships with a mock mode so you can build and enjoy it standalone.
- **[The Cinematic Camera](cinematic-camera-prompt.md):** the add-on for the visualizer, and the flyover you see in the demo. Press the spacebar and a scripted, movie-style flythrough renders over your live scene, with shots your agent designs around YOUR scene, nothing pre-rendered. Paste it into the same agent that built your visualizer.

[![Watch the demo: a living circuit board visualizer running live](https://img.youtube.com/vi/6Tb41ORADgs/maxresdefault.jpg)](https://www.youtube.com/watch?v=6Tb41ORADgs)

Build the voice line and the visualizer and you get the full pairing: hold the key, talk, and your scene comes alive while your agent answers. Add the camera and you get the movie.

## Support

Free to use, and always will be. If this helped you out, you can buy me a coffee:

[![Support me on Ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/jaredrhod)

## License

[CC BY-NC-SA 4.0](LICENSE): free to use, share, and build on with credit. Not for resale.
