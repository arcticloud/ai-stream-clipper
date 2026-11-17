# AI Stream Clipper: turn streams into vertical clips

[Русская версия](README.ru.md)

Give it a stream VOD link, get up to 3 ready-to-post vertical clips: the streamer's face on top, gameplay
below, a hook title, word-by-word captions, and a post caption with hashtags for Shorts, TikTok, Reels and X.
Works on Windows and Mac, with English and Russian streams. No API keys: Claude Code itself picks the
moments, everything else runs on your computer for free.

## What you need

- Claude Code (Claude subscription, from $20/month).
- Windows 10/11 or a Mac. Python 3.9+ ([python.org](https://www.python.org/downloads/); on Windows tick
  "Add Python to PATH" during install).
- An NVIDIA graphics card makes transcription 5-10x faster. Without one it still works, just slower.
- 5 GB of free disk space.

## Install (once)

1. Put this folder into your Claude skills folder as `ai-stream-clipper`:
   - Windows: `%USERPROFILE%\.claude\skills\ai-stream-clipper\`
   - Mac: `~/.claude/skills/ai-stream-clipper/`

   With git: `git clone https://github.com/arcticloud/ai-stream-clipper $HOME/.claude/skills/ai-stream-clipper`
   (or download the ZIP from GitHub and unpack it there).
2. Open Claude Code in any folder and type: **set up the clipper**
3. Claude checks your computer, installs what's missing (ffmpeg, yt-dlp, speech-to-text) and says READY.
   The first time takes 5-15 minutes because the transcription model downloads.

## How to use it

- **clip this stream https://www.youtube.com/watch?v=...** (YouTube, Twitch, Kick, or a file on your computer)

One stream at a time, up to 3 clips from it. A 4-hour stream takes about 10-15 minutes from link to
finished clips on an RTX 3060. Without an NVIDIA card, transcription takes longer.
Clips land in the `StreamClipper` folder in your home folder: the `.mp4`, the post text `post.md` and a
`report.md`.

YouTube says "confirm you're not a bot" (happens after many downloads) or the stream is age-restricted?
Log into YouTube in Firefox and tell Claude: "use my Firefox cookies".

## Free, no subscription (Ollama)

You can run Claude Code on a free local model: install [Ollama](https://ollama.com), download a model that
fits your memory (16 GB: Qwen 3.5 9B, 32 GB or more: Qwen 3.5 27B), type `ollama launch claude` in a terminal,
then use the same prompts. Its picks are rougher and slower than Claude's.

## Important

Only clip streamers who allow clipping (many run their own clipping programs), and watch every clip
before you post it.

## License

MIT, see `LICENSE`. Fonts and models: `THIRD_PARTY_NOTICES.md`.

---

The full version (several streams at once, a stream finder, chat signals, up to 8 clips, a list of open clipping
campaigns and ready prompts, updates) is in my Skool community, link in the video description.
