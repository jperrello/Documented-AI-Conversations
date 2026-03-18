# llmstxt-youtube Pipeline Design
**Date:** March 17, 2026
**Time:** 4:14 PM PDT
---

## User

/document I have an idea where we create an lms.txt: llmstxt.org but for youtbube videos it wouldnt be called llms.txt. it would be a feature on coding agents where I can paste a youtube video link and a better understanding of the video would be given to the agent. currently the best option is just giving the transcript, which is flawed. we need a system that gives agents more subtle details about the video, ultimately strengthening the context.

## Claude

Documenting.

This is a compelling idea. The core problem is real — transcripts lose a massive amount of signal. A transcript is just words, but a coding tutorial video carries:

- **Visual context** — what's on screen (code, terminal output, diagrams, browser tabs, file trees)
- **Temporal structure** — the order things are built, what gets deleted/refactored, the "arc" of the explanation
- **Implicit intent** — when someone hovers over code, scrolls to a specific section, highlights text, or switches between files, that's signal a transcript completely drops
- **Error/debug cycles** — the most valuable part of many coding videos is watching someone hit an error, diagnose it, and fix it. Transcripts flatten this into an incoherent sequence of words

The llms.txt analogy is sharp. llms.txt gives agents a structured, machine-readable summary of a website. Your idea would give agents a structured, machine-readable summary of a video — not just what was *said*, but what was *shown* and *done*.

Some initial thoughts on what this could look like:

**What the output format might contain:**
- Timestamped segments with semantic labels (setup, implementation, debugging, explanation)
- Extracted code blocks tied to timestamps
- Screen content descriptions (what file is open, what terminal shows)
- Dependency/tool list derived from what's visible
- A structured summary that preserves the build order and decision points

**Why "just use the transcript" fails:**
- Speaker says "this" or "here" referencing something on screen — meaningless without the visual
- Code is typed live but never fully dictated
- Error messages appear on screen but aren't read aloud
- File paths, configs, and terminal output are visual-only

What's your thinking on the implementation side? Are you imagining this as a pipeline that processes videos ahead of time, or something that runs on-demand when an agent gets a YouTube link?

## User

theres this thing scarnon: https://github.com/ryan-allen/scarnon i wonder if it uses some stack we can benefit from. probably not as it is just a skill but potentially the concept may help. spawn research agents to see what march 2026 landscape is

## Claude

Two research agents deployed — one on scarnon specifically, one scanning the broader March 2026 landscape of video-to-context tools.

```tool
[Tool: Agent] scarnon-research — Deep dive into https://github.com/ryan-allen/scarnon — clone or fetch the repo contents, understand what it does...
```

```tool
[Tool: Agent] landscape-research — Research the March 2026 landscape of tools/projects that convert YouTube videos into structured context for AI agents...
```

**Scarnon summary:** It's a Claude Code skill (no actual code — just a 563-line SKILL.md that Claude interprets). It uses yt-dlp to search YouTube, download SRT subtitles, then fans out to Haiku subagents to extract structured summaries from each transcript. The synthesis step produces cross-video comparisons with timestamped links.

**Key finding: It's transcript-only.** No frame extraction, no visual analysis, no OCR, no screen content. Exactly the limitation your idea addresses.

**What's useful from scarnon's approach:**
- The fan-out pattern — parallel Haiku subagents each process one video's transcript, keeping raw SRT files out of the main context window
- yt-dlp as the YouTube interface (no API key needed for search, metadata, or subtitle download)
- Structured extraction into ~2KB summaries per video with timestamped references

**Landscape results — the gap is confirmed. No "llms.txt for video" exists.**

The landscape breaks into tiers:

**Tier 1: Transcript-only (where everyone is today)**
- Scarnon, youtube-transcript MCP servers (4+ implementations), YouTube2Book, TubeSage
- All rely on YouTube auto-captions. ~20% of videos have no captions. Zero visual understanding.

**Tier 2: Frame analysis exists but is fragmented**
- **video-analyzer (byjlw)** — OpenCV keyframe extraction + Whisper + Llama 3.2 Vision. Produces natural-language frame descriptions merged with transcript.
- **Creu-Labs/video-analyzer** — Chained OpenAI vision analysis where each frame gets context from the previous 5 frames (describes only changes).
- **AKS (CVPR 2025)** — Adaptive keyframe sampling that maximizes visual info within a fixed token budget.

**Tier 3: Code extraction from video (partially solved)**
- **Polarize** — Chrome extension using Gemini to extract code from video frames.
- **Tube-Code** — Gemini pulls code from transcripts.
- **Pixelcode AI** — Screenshot-based OCR to code.
- **ACE (2016 academic paper)** — Consolidates code across video frames with statistical correction. Foundational work.
- None produce a structured document. All point tools.

**Tier 4: Commercial APIs that could power this**
- **Twelve Labs** — Auto-indexes: thumbnails + transcripts + OCR + logo detection with timecodes. Pegasus model returns developer-defined JSON schemas.
- **Google Gemini Video API** — Accepts YouTube URLs directly. Configurable FPS, natively multimodal. Structured JSON output.
- **VideoDB** — "Video as data" paradigm. Indexes video streams into structured records.

**What this means for your idea:** The pieces exist but nobody has assembled them into a standard. Transcript extraction (yt-dlp), keyframe extraction (OpenCV, AKS), frame-to-description (vision LLMs), code extraction from frames (Polarize, OCR), structured JSON output (Twelve Labs, Gemini) — all available. What's missing is the format spec and a pipeline that produces it. That's your contribution.

Scarnon's architecture pattern (Claude Code skill + yt-dlp + fan-out subagents) is the right chassis. You'd extend it with a vision layer between "download video" and "produce output."

## User

maybe we are thinking to specifically about agents. youtube has been around since 2026. there has to be frameworks, libraries, countless programs about pulling more informative information about videos. im sure theres academic research in the space of extracting video elements in a macine readable way. take a video like https://www.youtube.com/watch?v=tXqnRMU1fTs this video is a John Oliver discussion about legal immegration. i imagine someone may link to it and want to create quizzes, or figures, or potentially another video from this video. what do you see when i paste that url? we need to think what does this pipeline consist of. we probably still want the transcript with timestamps. we probably want the description of the video, the title, any figures shown on screen refeerences made, etc, it is hard for me to think of a ful picture. we sont need a full picture but maybe now we can think of all the chunks needed to create something like this and maybe we will find the glue along the way?

## Claude

Two agents deployed:

1. **video-extraction-research** — surveying the full academic and tooling landscape: scene detection, figure extraction, speaker diarization, video OCR, ed-tech quiz generation, video-to-document pipelines. Decades of work across journalism, education, film studies, archival.

2. **video-metadata** — running yt-dlp against that John Oliver video to see exactly what's freely available right now without any vision processing.

```tool
[Tool: Agent] video-extraction-research — Research the broader field of video information extraction — NOT just for AI agents, but for any purpose...
```

```tool
[Tool: Agent] video-metadata — Use yt-dlp to extract ALL available metadata and information from https://www.youtube.com/watch?v=tXqnRMU1fTs...
```

**Video metadata results — what's already freely available from that John Oliver video:**
- Full transcript (2,008 lines of SRT, messy but complete)
- 4 chapters: `[Untitled Intro] → FAMILY → EMPLOYMENT → GOOD LUCK`
- 42 thumbnail URLs at various sizes
- Viewer retention heatmap (100 data points — shows where people rewatch/drop off)
- 79 metadata fields including engagement stats, technical specs, upload date

**What a human sees that the agent completely misses:**
- On-screen graphics, charts, statistics, infographics (Last Week Tonight is heavy on these)
- News clips playing within the show — the transcript flattens Oliver's commentary and embedded source clips into one undifferentiated word stream
- Speaker identity — no way to tell when Oliver is talking vs. quoting a clip
- Visual gags and comedy bits tied to what's on screen
- Any quote, citation, or source shown as text overlay

**The retention heatmap is an underappreciated signal.** It tells you where viewers found the video most valuable. Spikes in retention = moments worth extracting more carefully. That's a free prioritization signal for which frames to analyze with vision.

**Landscape research confirms: every piece of this pipeline has mature tooling. Nobody has assembled them.**

The extractable chunks, mapped to proven tools:

| Chunk | Tool/Library | Maturity |
|-------|-------------|----------|
| Transcript + timestamps | yt-dlp (SRT), WhisperX (fallback) | Battle-tested |
| Chapters | yt-dlp (pre-parsed from description) | Solid |
| Shot/scene boundaries | PySceneDetect, ffmpeg scene filter | 20+ years of research |
| On-screen text (OCR) | PaddleOCR (0.93 confidence), EasyOCR, Tesseract | Production-grade |
| Speaker diarization | pyannote.audio + WhisperX | State of the art |
| Face detection | OpenCV, dlib | Decades old |
| Keyframe extraction | ffmpeg, OpenCV | Trivial |
| Figure/chart extraction | Lecture2Notes (open source) | Works for lectures, nothing for general video |
| Thumbnails | yt-dlp (42 URLs free) | Free |
| Retention heatmap | yt-dlp (100 data points) | Free |
| Description + metadata | yt-dlp (79 fields) | Free |
| Video understanding | Gemini API (accepts YouTube URLs natively), Twelve Labs Pegasus | Commercial but powerful |

**The gap is integration, not capability.**

For that John Oliver video, the full extraction would look like:

1. **Metadata pass** (free, instant): title, description, chapters, retention heatmap, thumbnails
2. **Transcript pass** (free, seconds): SRT download, clean up punctuation, align with chapters
3. **Scene detection pass** (local, fast): PySceneDetect identifies shot boundaries — distinguishes Oliver at desk vs. news clips vs. graphics vs. custom segments
4. **OCR pass** (local, moderate): Sample frames at scene boundaries + retention spikes, PaddleOCR extracts on-screen text
5. **Speaker pass** (local, moderate): pyannote.audio labels who's speaking when
6. **Vision pass** (API, expensive): Send keyframes to Gemini/Claude Vision for semantic description of what's shown
7. **Synthesis**: Merge all passes into one structured document

Passes 1-2 are what scarnon does. Passes 3-5 are all mature local tools. Pass 6 is where cost lives. Pass 7 is the format spec — the thing that doesn't exist yet.

**Key realization:** You don't need vision for everything. OCR + scene detection + diarization get you maybe 85% of the missing context using purely local, free tools. Vision LLMs are only needed for the remaining 15% — interpreting complex graphics, describing visual gags, understanding what a chart means. The retention heatmap tells you which moments are worth spending vision API tokens on.

This is starting to look like a layered pipeline where each layer is optional and adds more richness. The "glue" might just be: a schema that defines all possible chunk types, and a pipeline that fills in whichever chunks you can afford to compute.

## User

I see so in a plan we decide to make we should ignore what the urls already give us -- it will be there automatically, but certain things like the heatmap can give more context. I think this is a good use for /ralph lets just have a ralph loop build this whole pipeline I think the glue will come from there. I think the success condition will be hard to measure but we can consistently use the john oliver video as a baseline reference and video to test with. I think our next job should be to go through the ralph pipeline till we get a good plan

## Claude

Invoked /ralph skill. Found existing RALPH_PROMPT.md (streaming/studio pipeline). User chose to create new directory `llmstxt-youtube/` for this ralph.

Went through the full interactive design:

**Q1 Objective:** Pipeline takes YouTube URL → structured document with all extractable layers. Confirmed.

**Q2 Constraints:** Fully local first — no API calls. yt-dlp, ffmpeg, Python libs only.

**Q3 Components (6, dependency order):**
1. fetch — yt-dlp metadata + SRT + chapters + heatmap + thumbnails + video. No deps.
2. transcript — Clean SRT → structured JSON. Deps: fetch.
3. scenes — PySceneDetect + ffmpeg keyframes. Deps: fetch.
4. ocr — PaddleOCR on keyframes. Deps: scenes.
5. diarize — pyannote.audio + WhisperX speaker labels. Deps: fetch.
6. synthesize — Merge all passes → one structured markdown document. Deps: transcript, scenes, ocr, diarize.

**Q4 Pattern:** A) Simple — one task per pass, pick next ready, build, test, move on.

**Q5 Completion:** A) Binary — test passes or fails.

**Q6 Moons:** No — beads only for state tracking.

```tool
[Tool: Bash] bd init, bd create (epic + 6 tasks), bd dep add (8 dependency edges), bd remember (4 memories)
```

```tool
[Tool: Write] llmstxt-youtube/research.md — full spec with tool choices, output formats, test criteria per component
```

```tool
[Tool: Write] llmstxt-youtube/ralph/RALPH_PROMPT.md — autonomous agent prompt
```

```tool
[Tool: Write] llmstxt-youtube/ralph/ralph.sh — loop script
```

Ralph setup complete:
- `eab.1` → fetch (ready now)
- `eab.2` → transcript (blocked by fetch)
- `eab.3` → scenes (blocked by fetch)
- `eab.4` → ocr (blocked by scenes)
- `eab.5` → diarize (blocked by fetch)
- `eab.6` → synthesize (blocked by all others)

To start: `./ralph/ralph.sh` from llmstxt-youtube/
