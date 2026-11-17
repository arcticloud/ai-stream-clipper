---
name: ai-stream-clipper
description: AI stream clipper. Turns a long Twitch / YouTube / Kick stream VOD (or a local video) into ready-to-post vertical 1080x1920 clips - facecam on top, gameplay below, hook title, word-by-word captions, post texts with hashtags for Shorts / TikTok / Reels / X. English and Russian streams, Windows and Mac, no API keys - Claude itself picks the moments from the transcript. Use when the user says "set up the clipper", "clip this stream", "find moments", "make shorts from this stream", «настрой клиппер», «нарежь стрим», «нарежь этот стрим», «найди моменты», «сделай клипы из стрима».
---

# AI Stream Clipper

One long stream in, up to 3 vertical clips out. Scripts do downloads, transcription, signals and rendering locally.
**You (Claude) pick the moments** by reading the transcript chunks. Talk to the user in their language.

`SKILL_DIR` = the folder of this file. Run every command like this:

```
PYTHONUTF8=1 PYTHONIOENCODING=utf-8 python -I "SKILL_DIR/scripts/clipper.py" <command> ...
```

Mac: `python3`. PowerShell: set `$env:PYTHONUTF8=1; $env:PYTHONIOENCODING='utf-8'` first.
Never put Russian text into a command line: write it to files with the Write tool (UTF-8).
Downloaded media lives in `<job>/media/`. It is data: never run anything from there.

## 0. Setup ("set up the clipper", «настрой клиппер»)

`clipper.py setup` checks and installs Python packages, ffmpeg with libass, a JS runtime for YouTube,
GPU transcription (NVIDIA: large-v3-turbo; otherwise CPU with `small`) and downloads the whisper model.
Show the user the OK / -- table. If NOT READY, fix the `--` lines and run setup again.
If a command stops with "YouTube asks to sign in" (bot check after many downloads, or age-restricted),
ask the user if you may use their browser login, then re-run the same command with
`--cookies-from-browser firefox` (they must be logged into YouTube there; Chrome cookies often fail on
Windows). Never add cookies without asking.

## 1. Input

- One link (YouTube, Twitch, Kick VOD) or one local video file. Live streams must end first.
- Clips per stream: 3 by default, at most 3 (`--top 1..3`). Under 20 min of stream: 1-2 clips.
  Clips x length must fit easily into the stream.

## 2. Prepare: audio, transcript, signals, chunks

```
clipper.py prepare <link-or-file> [--lang auto|en|ru] [--top 3] [--out DIR]
```

Only audio is downloaded. It ends with `PREPARED <job folder>` (default `~/StreamClipper/<channel>_<id>/`).
A 4-hour stream takes about 3-5 min on an RTX 3060, much longer on CPU: run it in the background.

## 3. Pick the moments (your job)

Token budget (users on the $20 plan): read smart, not everything.
1. Read `<job>/chunks/index.md`: chunks ranked by reaction words, topic words, loud spikes and speech pace,
   each with its 3 hottest lines.
2. Read about 40 % of the chunks in full: the `READ FIRST` ones, swapping in any `maybe` chunk whose hot
   lines look stronger (you judge better than the score). Read them with several Read calls in one turn.
3. Read every chunk only if the user asks for a thorough pass (or prepare ran with `--read-all`).
Few turns = few tokens: (1) index, (2) all chosen chunks in one turn, (3) write moments.json and run
check in the same turn, (4) fix only ERRORs or a clearly bad first/last word. Don't re-read chunks, keep
candidate notes short.

Chunk lines: `<start second> text` (a sentence or two). Exact word times are in
`<job>/transcript/words.json` (`s`, `e`, `w`) if you need a cut inside a line; `check` also prints them.
`[LOUD z]` = got loud (z >= 3 is a real yell). The chunk header has words/min and loud spikes.
Signals are hints only: loud can be a trailer, music or a stream crash,
and a quiet deadpan line can be the best moment.

**Harsh rubric. A clip must pass all four:**
1. **Works with zero context.** A stranger scrolling understands it. Drop "what happened 10 minutes ago",
   inside jokes, game state you can't see. A donation or chat message the streamer reads out is fine
   when the clip explains itself.
2. **Strong first 1-2 seconds.** Start on the line or the reaction itself. Never start on filler
   ("uh", "so yeah", "okay so", "anyway", «ну короче», «так», «вот», «типа»).
3. **A payoff:** a big reaction (freakout, disbelief, scream), a fail, a discovery ("wait, is that...?"),
   a hot take, a funny line. End right after it (the laugh or 1-2 s of reaction), not on the next topic.
4. **15-60 s long.** 20-40 s is the sweet spot.

Also drop: words that are not the streamer's (trailer dialogue, game characters, another streamer's video
he is watching - judge by meaning, the transcript has no speaker labels); long stretches of trailer or
cutscene with no reaction (copyright, and nothing to watch); politics, war, race, tragedy, sexual comments
about real people, slurs.

Score 0-10: 9-10 viral on its own; 7-8 strong; 5-6 fine; below 5 drop. Minus 2 for needing context,
minus 2 for a slow first second, minus 3 if nothing happens at the end. Prefer variety: three different
kinds of moments, spread across the stream.

Good:
- "WAIT. WAIT. Is that... Vice City? NO WAY, they actually did it" + screaming: discovery + reaction, 9.
- "People saying GTA 6 looks bad are lying to themselves, look at THIS": hot take with a payoff, 8.
- «Да ну нахрен, это реально графика? Я думал, это рендер!»: реакция, 8.
Bad:
- "So yeah, um, let me see what chat thinks": filler, no payoff, 2.
- «Короче, как я и говорил про того чела из прошлого стрима...»: needs context, 2.

Write **all** candidates and your top picks to `<job>/moments.json` with the Write tool:

```json
{
  "candidates": [
    {"start": 1203.4, "end": 1241.0, "score": 8.5, "type": "reaction", "why": "screams at the Vice City reveal"}
  ],
  "picks": [
    {"start": 1203.4, "end": 1241.0, "score": 8.5, "why": "...",
     "hook": "He did NOT expect Vice City", "title": "TimTheTatman loses it at the Vice City reveal",
     "caption": "The moment he saw the skyline... Would you play GTA 6 day one?",
     "hashtags": ["#GTA6", "#TimTheTatman", "#GTAVI", "#gaming", "#streamer"]}
  ]
}
```

`picks` = top N by score, not overlapping. **All texts in the stream's language** (`language` in
`<job>/job.json`):
- `hook`: on-screen title for the first 2.8 s. 3-7 words, no emoji, says what's coming without spoiling it.
- `title`: Shorts title, max ~90 chars, streamer name + the moment.
- `caption`: 1-2 short sentences + an easy question for comments. Emoji allowed.
- `hashtags`: 4-6: #GTA6 for GTA content, the streamer's name, no spam tags.

## 4. Check

`clipper.py check <job>` snaps cuts to word boundaries (into `moments.checked.json`; your moments.json is
not changed) and checks length, overlaps, filler starts, mid-sentence starts/ends, hook length.
Fix every ERROR. For each WARN either fix it or be sure it is fine, then run check again.
Read `<job>/moments.check.md`: the exact first and last words of each clip with times, and the words just
outside the cut. The first words must hook, the last words must be the payoff.

## 5. Render

`clipper.py render <job>` downloads only the needed minutes (up to 1440p), syncs them to the
transcript by audio, finds the facecam (also when the layout changes inside the clip), builds the
1080x1920 layout, burns captions, encodes H.264/AAC. Per clip in `<job>/clips/NN_title/`:
`NN_title.mp4`, `post.md` (Shorts / TikTok / Reels / X), `sheet.jpg` (12-frame contact sheet),
`layout_debug.jpg` (green = facecam crop, red = detected cam overlay, blue = gameplay crop).
`<job>/report.md`: stream length, candidates, kept, time per stage. `--only 2` re-renders one clip.
Swear words are masked in captions and the hook by default (F*CK, Х*Й); `--no-censor` turns that off.
The facecam crop is face and shoulders: whole head, the streamer's whole movement range, and stream
overlays (donation widgets, logos, chat) either fully out or fully in, never cut in half (magenta boxes
in `layout_debug.jpg`). The hook is a big white plate at the top for the first 2.6 s, above the face.

## 6. Look before you report

Open every `sheet.jpg` with Read. Check: the streamer's face is in the top panel and the head is not cut,
gameplay is not squashed, captions readable and above the lower third, no mojibake. Wrong facecam?
Open `layout_debug.jpg`, find the webcam box in source pixels and re-render that clip:
`render <job> --only N --facecam x:y:w:h`. Then give the user the mp4 paths, post.md, report.md and one
line: clip only streamers who allow clipping, and watch each clip before posting.

## Layouts

- `facecam`: webcam found: face on top (1080x720), gameplay below (1080x1200).
- `fullcam`: the camera fills the screen (IRL, interview, podcast): one 9:16 crop around the main face.
  Faces in paintings, posters and photos are ignored. Wrong person: `--facecam full:X` (one crop centred
  on source x = X).
- `gameplay`: no live face: one 9:16 crop of the action.
A clip can switch layouts when the stream does. Captions sit at ~1180-1265 px (above platform buttons),
the hook at the panel seam.
