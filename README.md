# Content Creator Skills for Claude

<p align="center">
  <a href="https://clips.synofex.com/r/5cb59f4d">
    <img src="assets/synoclips-banner.svg" alt="Sponsored by Syno Clips: turn live streams and videos into viral clips" width="100%">
  </a>
</p>

<p align="center"><b>Sponsored by <a href="https://clips.synofex.com/r/5cb59f4d">Syno Clips</a></b>: turn long videos into short clips and publish them to YouTube, Instagram, TikTok and Facebook. It powers the auto-publish skill.</p>

Ten Claude skills that cover the full content workflow: **research → plan → write → design → produce video → publish → measure**. Each skill is a folder with instructions for Claude (`SKILL.md`), reference guides, and Python scripts that do the mechanical work (API calls, checks, rendering, exports).

All skills share three principles:

- **Evidence over guesses.** Numbers come from APIs, exports or sources you can open; missing data stays blank and is never invented.
- **Honest output.** No fake testimonials, statistics, thumbnails or claims; AI voices and paid partnerships are disclosed.
- **You stay in control.** Nothing is posted, scheduled or sent without your approval, and a person reviews before publishing.

---

## Contents

1. [Folder structure](#1-folder-structure)
2. [Setup](#2-setup)
3. [How the skills work together](#3-how-the-skills-work-together)
4. [The skills](#4-the-skills)
   - [4.1 viral-content-research](#41-viral-content-research)
   - [4.2 content-strategy](#42-content-strategy)
   - [4.3 brand-voice](#43-brand-voice)
   - [4.4 short-form-video-scripts](#44-short-form-video-scripts)
   - [4.5 long-video-generation-using-free-clips](#45-long-video-generation-using-free-clips)
   - [4.6 thumbnail-design](#46-thumbnail-design)
   - [4.7 social-media-content](#47-social-media-content)
   - [4.8 social-media-calendar](#48-social-media-calendar)
   - [4.9 analytics](#49-analytics)
   - [4.10 auto-publish](#410-auto-publish)
5. [Troubleshooting](#5-troubleshooting)
6. [Language support](#6-language-support)
7. [Sponsor](#7-sponsor)

---

## 1. Folder structure

```text
Skills/
├── README.md                     ← this file
├── assets/                       ← images used in this README
└── skills/                       ← the 10 skills
    ├── analytics/
    ├── auto-publish/
    ├── brand-voice/
    ├── content-strategy/
    ├── long-video-generation-using-free-clips/
    ├── short-form-video-scripts/
    ├── social-media-calendar/
    ├── social-media-content/
    ├── thumbnail-design/
    └── viral-content-research/
```

Every skill folder has the same layout:

```text
<skill-name>/
├── SKILL.md        ← what Claude reads: when to use the skill, workflow, rules
├── references/     ← detailed guides Claude opens when a step needs them
└── scripts/        ← Python tools (run them yourself or let Claude run them)
```

---

## 2. Setup

### 2.1 Install the skills into Claude

Open PowerShell in this folder and copy each skill into your Claude skills folder:

```powershell
Copy-Item -Recurse -Force ".\skills\*" "$env:USERPROFILE\.claude\skills\"
```

Claude loads them automatically; restart the app if a skill does not appear. The skills start on their own when your request matches (for example "make a thumbnail"); you do not need to name them.

### 2.2 Software

| Software | Needed by | Install |
|---|---|---|
| Python 3.9+ | all skills | python.org |
| Pillow | thumbnail-design | `pip install pillow` |
| FFmpeg 5+ (`ffmpeg`, `ffprobe`) | long-video | `winget install ffmpeg` or ffmpeg.org |
| tzdata (Windows) | social-media-calendar, analytics (time zones such as `Asia/Karachi`) | `pip install tzdata` |
| piper-tts (optional) | long-video free voiceover | `pip install piper-tts` |
| rembg (optional, large) | thumbnail cutouts | `pip install "rembg[cpu]"` |
| faster-whisper (optional, large) | long-video word-timed subtitles | `pip install faster-whisper` |

Every other script uses the Python standard library only.

### 2.3 API keys

Keys are read from environment variables only. Never paste keys into chat or files. Set each one once in PowerShell, then restart the Claude app:

```powershell
setx YOUTUBE_API_KEY "your-key"
```

| Variable | Cost | Used by | Where to get it |
|---|---|---|---|
| `YOUTUBE_API_KEY` | free (10,000 units/day) | viral-content-research, content-strategy, short-form-video-scripts | Google Cloud Console → new project → enable "YouTube Data API v3" → Credentials → API key |
| `PIXABAY_API_KEY` | free | long-video, thumbnail-design | pixabay.com/api/docs (log in) |
| `PEXELS_API_KEY` | free | long-video, thumbnail-design | pexels.com/api (new keys are sometimes paused; Pixabay alone is enough) |
| `ELEVENLABS_API_KEY` | paid, optional | long-video voiceover | elevenlabs.io |
| `REMOVEBG_API_KEY` | paid, optional | thumbnail cutouts | remove.bg |
| `OPENAI_API_KEY`, `THUMB_IMAGE_MODEL` | paid, optional | AI thumbnail backgrounds | platform.openai.com |
| Synoclips connector | your Synoclips plan | auto-publish | Create a [Syno Clips account](https://clips.synofex.com/r/5cb59f4d), then add the Synoclips connector in the Claude app; connect YouTube, Instagram, TikTok and Facebook accounts on the Synoclips website (Settings → Integrations). No key is stored on this computer. |

Without keys the skills still work: research falls back to web search (labelled "unverified") and video uses your own footage.

### 2.4 Piper voices (free voiceover)

Replace `<you>` with your Windows user name. Use forward slashes in these paths.

```powershell
python -m piper.download_voices en_US-lessac-medium --data-dir C:/Users/<you>/piper-voices
python -m piper.download_voices ur_PK-fasih-medium ur_PK-aegis_female-medium --data-dir C:/Users/<you>/piper-voices
setx PIPER_CMD "{python} -X utf8 -m piper --data-dir C:/Users/<you>/piper-voices -m {voice} -f {out}"
```

`-X utf8` is required on Windows for Urdu (and other non-Latin) text. Urdu voices read **Urdu script**, not Roman Urdu. Check each voice's licence before commercial use.

---

## 3. How the skills work together

```text
viral-content-research ─┐
                        ├─► content-strategy ─► social-media-calendar ─────────────┐
brand-voice ────────────┘          │                                               │
        │                          ▼                                               ▼
        ├────────► short-form-video-scripts ─┐                               auto-publish ──► analytics
        ├────────► long-video-generation ────┼─► thumbnail-design                  ▲
        └────────► social-media-content ─────┘                                     │
                   (human review, then upload and publish via Synoclips) ──────────┘
```

A typical cycle:

1. **Research** what is outperforming (viral-content-research) and where the gap is (content-strategy).
2. **Define the voice** once (brand-voice); every later text is checked against it.
3. **Create**: short scripts, long videos, thumbnails and posts from the same angle and facts.
4. **Schedule** on a calendar the team can actually produce (social-media-calendar).
5. **Publish** clips through Synoclips after you approve the exact list (auto-publish).
6. **Measure** with real exports (analytics), then feed what worked back into the strategy.

---

## 4. The skills

### 4.1 viral-content-research

#### What It Does

Find what is outperforming its own channel's normal results in a niche, explain why, and turn the patterns into content ideas.

#### When to Use

Ask Claude things like: "What is going viral in my niche?", "Which videos about X are outperforming?", "Give me video ideas that will actually perform."

#### What You Get

A short report (snapshot, top outliers, patterns, hook bank, ideas, audience language, timing, method) plus raw `.csv`/`.json` data and comment exports.

#### Requirements

Python; `YOUTUBE_API_KEY` for verified numbers (optional).

#### Files

| File | Purpose |
|---|---|
| `scripts/yt_research.py` | YouTube API: `search`, `trending`, `stats`, `comments`; computes outlier scores. |
| `references/analysis-framework.md` | Hook types, title formulas, validation, control groups, saturation. |
| `references/web-research.md` | Web query patterns and confidence labels. |
| `references/report-template.md` | Report structure. |

#### How It Works

**Key metrics:**

| Field | Meaning |
|---|---|
| `outlier_x` | Views ÷ channel's median views on recent uploads (median floored at `--min-baseline`, default 1,000). 3x notable, 5x+ real outlier. |
| `baseline_floored` / `*` | The floor was used: small channel, weaker evidence. |
| `views_per_day` | Velocity: rising vs old hits. |
| `views_per_sub`, `like_rate_pct`, `comment_rate_pct` | Fallback and tie-breaker signals. |

#### Commands

```text
yt_research.py search -q QUERY [-q ...] [--days 30] [--region PK] [--lang en] [--max 30] [--order viewCount|relevance|date|rating]
                      [--duration any|short|medium|long] [--baseline-top 15] [--min-baseline 1000] [--limit 25] [--out BASE]
yt_research.py trending [--region US] [--category ID] [--max 50] [--out BASE]
yt_research.py stats VIDEO [VIDEO ...] [--out BASE]
yt_research.py comments VIDEO [--max 50] [--out BASE]
```

Quota: each `search` query costs 100 units; other calls about 1 unit.

#### Rules

A pattern needs at least 3 videos from 3 channels; numbers are labelled verified / page-visible / unverified; quotes stay under 15 words; ideas transform a pattern, never copy a video.

---

### 4.2 content-strategy

#### What It Does

A complete, defensible strategy: personas, niche and competitor analysis, content gaps, 3 to 5 pillars, funnel, platform roles, formats, cadence, KPIs and a calendar. Every decision cites evidence or is marked as an assumption.

#### When to Use

Ask Claude things like: "Plan my content", "What should I post?", "Build a social media strategy", "Where is the gap in my niche?"

#### What You Get

A strategy document, an evidence ledger (`evidence.csv`: id, source, claim, metric, date, confidence), demand keywords, competitor stats, a gap matrix and a calendar.

#### Requirements

Python; `YOUTUBE_API_KEY`; optional Google Trends CSV exports and SEO connectors (Semrush, Ubersuggest).

#### Files

| File | Purpose |
|---|---|
| `scripts/research_helper.py` | `suggest` (autocomplete keywords), `trends-csv`, `gap` (demand vs supply), `calendar`. |
| `scripts/yt_competitors.py` | Competitor channels: cadence, median views, outlier share, short-form share, durations, title patterns. |
| `scripts/yt_research.py` | Shared YouTube helper. |
| `references/data-sources.md` | Which source answers which question; ledger format; confidence labels. |
| `references/frameworks.md` | Personas, pillar rubric, gap matrix, funnel, KPIs, cadence logic. |
| `references/deliverable-template.md` | Strategy document and calendar columns. |

#### How It Works

**Gap matrix classes:** `gap` (no title contains all the keyword's words), `thin` (a few do), `contested` (many do). Matching ignores function words in English and Roman Urdu / Hinglish and merges spelling variants (word lists `STOP_ROMAN` and `VARIANTS` in `research_helper.py`).

#### Commands

```text
research_helper.py suggest SEED [SEED ...] [--source google|youtube] [--hl en] [--gl PK] [--expand questions,modifiers,alpha] [--out BASE]
research_helper.py trends-csv FILE [FILE ...] [--out BASE]
research_helper.py gap --demand FILE --supply CSV [CSV ...] [--thin 2] [--out BASE]
research_helper.py calendar --example | --plan PLAN.json [--out BASE]
yt_competitors.py CHANNEL [CHANNEL ...] [--uploads 30] [--out BASE]      (@handle, URL or UC id; ~4 units each)
```

#### Rules

Cadence comes from capacity first; KPIs have a baseline, target and source; autocomplete and Trends are relative signals, not search volume; general benchmarks are labelled low confidence.

---

### 4.3 brand-voice

#### What It Does

Turn "how we sound" into a short voice guide plus measurable habits, then rewrite and check every draft against it.

#### When to Use

Ask Claude things like: "Build my brand voice", "Rewrite this in my style", "Does this sound like us?", "Audit my last 20 posts for consistency."

#### What You Get

A 1 to 2 page voice guide (traits as "X, but not Y", tone dials, language rules, vocabulary, format habits, platform shifts, situation tones, before/after examples, checklist), `profile.json` (measured habits) and `rules.json` (hard rules for automatic checks).

#### Requirements

Python only. Best with at least 10 real posts (1,000+ words); fewer gives a rough sketch.

#### Files

| File | Purpose |
|---|---|
| `scripts/voice_tools.py` | `profile` (measure samples), `check` (draft vs profile and rules), `cliches` (stock phrases list). |
| `references/voice-guide-template.md` | Guide template, tone-dial scale, situation tones, rewrite checklist. |
| `references/extraction-method.md` | Collecting samples, interview questions, reading the numbers, audit scoring. |

#### How It Works

**`rules.json` example:**

```json
{"banned": ["guaranteed", "zero bill"], "preferred": {"utilize": "use"}, "must_include": [],
 "max_sentence_words": 20, "max_emoji": 2, "max_hashtags": 5}
```

#### Commands

```text
voice_tools.py profile FILE [FILE ...] [--out profile.json]        (posts separated by a line with ---)
voice_tools.py check DRAFT.txt [--profile profile.json] [--rules rules.json] [--strict] [--json out.json]
voice_tools.py cliches
```

#### Rules

The voice is based on real samples, never on imitating a named person or competitor; no fabricated testimonials; sensitive situations (apologies, health, money, grief) get a plain, careful tone; voice never overrides facts or required disclosures.

---

### 4.4 short-form-video-scripts

#### What It Does

Shootable scripts for YouTube Shorts, Instagram Reels and TikTok, with 3-second hooks, scene-by-scene production notes and timing checked by a script.

#### When to Use

Ask Claude things like: "Write a 30-second reel script about X", "Give me 10 hooks", "Turn this article into a Short", "Make this video better."

#### What You Get

The script document, `script.json`, `voiceover.txt`, `captions.srt`, `shotlist.csv`/`.md` and a small test plan.

#### Requirements

Python; optional `YOUTUBE_API_KEY` and Trends CSV for topic validation. Production is free by default (own voice or editor text to speech, own or free-licensed footage, free editor); ElevenLabs, HeyGen and Veo are optional paid upgrades.

#### Files

| File | Purpose |
|---|---|
| `scripts/script_tools.py` | `budget`, `check`, `srt`, `shotlist`. |
| `scripts/yt_research.py`, `scripts/research_helper.py` | Shared research helpers. |
| `references/hooks-and-structures.md` | Hook types with templates and examples, word budgets, structures, pattern interrupts, CTAs. |
| `references/platforms.md` | Shorts vs Reels vs TikTok and adaptation rules. |
| `references/production-stack.md` | Free workflow and paid hand-offs. |
| `references/rewrite-playbook.md` | Diagnosing and improving an existing script or video. |
| `references/output-templates.md` | Document layout and JSON schema. |

#### How It Works

**`script.json` scene fields:** `voiceover`, `on_screen_text`, `visual`, `broll`, `camera`, `sfx`, optional `seconds`, `cta` (true for the call-to-action scene).

**`check` flags:** over/under length, slow hooks, greeting openers, scenes over ~5 s without a visual change, long sentences, too much on-screen text, missing CTA.

#### Commands

```text
script_tools.py budget --seconds 15 30 60 90 [--wpm 165]
script_tools.py check SCRIPT.json [--target 30] [--wpm N] [--json report.json] [--strict]
script_tools.py srt SCRIPT.json --out captions.srt [--max-words 4] [--upper] [--audio-seconds S]
script_tools.py shotlist SCRIPT.json --out BASE [--md] [--audio-seconds S]
```

#### Rules

Hooks are heuristics, not guarantees; no invented stats or results; realistic AI voices/avatars disclosed; only assets you have rights to.

---

### 4.5 long-video-generation-using-free-clips

#### What It Does

Turn a topic into a finished 1080p YouTube video (5 to 30+ minutes) from free stock footage, voiceover and automated FFmpeg editing, with per-clip licence tracking and originality checks.

> **Monetization risk:** YouTube demonetizes mass-produced or "inauthentic" content. Stock clips plus a synthetic voice and no original angle is exactly that pattern. Each video needs an original angle, sourced facts and a full human review.

#### When to Use

Ask Claude things like: "Make a 10-minute faceless video about X", "Turn this script into a video with free clips and a voiceover."

#### What You Get

| File | Content |
|---|---|
| `out/video.mp4` | Final 1920x1080 video, music ducked, loudness normalized. |
| `out/preview.mp4` | Fast 640x360 draft. |
| `out/subtitles.srt` | Captions on the final timeline. |
| `out/chapters.txt`, `out/description.txt`, `out/credits.txt` | YouTube packaging (description kept under 5,000 bytes). |
| `out/thumbnail.jpg` | Simple base thumbnail. |
| `build/review.html` | One frame per shot next to its narration, for review. |
| `project.json` | The whole plan: scenes, media and licences, audio durations, YouTube fields, review status. |

#### Requirements

Python, FFmpeg, `PIXABAY_API_KEY` and/or `PEXELS_API_KEY`; Piper (free) or ElevenLabs (paid) or your own recordings; optional faster-whisper.

#### Files

| File | Purpose |
|---|---|
| `scripts/project.py` | `init`, `validate`, `estimate`, `status`. |
| `scripts/voiceover.py` | Per-scene narration (piper, elevenlabs, own, silent) with exact durations. |
| `scripts/stock_search.py` | Searches Pexels, Pixabay, Commons, Internet Archive, Unsplash; ranks by topic relevance. |
| `scripts/fetch_clips.py` | `pick`, `review`, `reject`, `add-local`, `list`. |
| `scripts/subtitles.py` | SRT from script timing or Whisper. |
| `scripts/build_video.py` | Shots → scenes (transitions, slow zoom on photos, on-screen text) → audio mix → MP4. |
| `scripts/package_youtube.py` | `credits` (chapters, credits, description, checks) and `thumbnail`. |
| `scripts/common.py` | Shared helpers. |
| `references/youtube-policy-and-originality.md` | Policy, originality gates, sensitive topics, disclosure. |
| `references/sources-and-licenses.md` | Terms and attribution per footage source. |
| `references/scripting-and-packaging.md` | Research, script rules, scenes, `project.json` schema, titles, descriptions. |
| `references/voiceover.md` | Voices, pacing, pronunciation, Whisper. |
| `references/editing-and-render.md` | Timing, transitions, ducking, loudness, troubleshooting. |

#### How It Works

```powershell
python scripts/project.py init myvideo --title "How to cut your electricity bill" --minutes 8
# edit myvideo/project.json: angle, sources, scenes (narration, 2-3 filmable keywords, on_screen_text)
python scripts/project.py validate myvideo
python scripts/voiceover.py myvideo
python scripts/stock_search.py myvideo --sources pixabay
python scripts/fetch_clips.py pick myvideo --prefer mixed
python scripts/fetch_clips.py review myvideo            # open build/review.html, look at every shot
python scripts/fetch_clips.py reject myvideo s08 63028  # remove a wrong shot (blocked from re-picking)
python scripts/fetch_clips.py pick myvideo --scene s08 --fill
python scripts/build_video.py myvideo --draft
python scripts/subtitles.py myvideo
python scripts/build_video.py myvideo
python scripts/package_youtube.py credits myvideo
```

Finally, watch the whole video, check facts and licences, then set `review.human_checked` to `true` in `project.json`.

**How footage is chosen:** each result's own tags/alt text/title is compared with the scene keyword. A tag holding the whole phrase scores high (earlier tags count more); sharing only one word scores half. Relevance outranks resolution, so an on-topic photo beats an off-topic video. `pick` skips matches under 0.3 unless `--allow-offtopic`. Tags can still mislead, so always run `review`.

#### Commands

```text
project.py init DIR --title ".." [--minutes 12] [--language en] [--force] | validate DIR [--strict] | estimate DIR | status DIR
voiceover.py DIR [--engine piper|elevenlabs|own|silent] [--scene ID] [--force] [--dry-run]
stock_search.py DIR [--scene ID] [--sources pexels,pixabay,commons,archive,unsplash] [--kinds video,image] [--keep 12] [--max-requests 120]
fetch_clips.py pick DIR [--prefer video|image|mixed] [--scene IDS] [--redo | --fill] [--allow-offtopic] [--dry-run]
fetch_clips.py review DIR | reject DIR SCENE ID [ID ...] | list DIR
fetch_clips.py add-local DIR SCENE FILE --license ".." [--creator ..] [--page-url ..] [--confirmed]
subtitles.py DIR [--mode script|whisper] [--model small]
build_video.py DIR [--draft] [--only s01,s02] [--burn-subs] [--transition fade,dissolve] [--no-text] [--no-music]
package_youtube.py credits DIR | thumbnail DIR --text "3 WORDS" [--source FILE]
```

#### Rules

Respect each source's API terms and download only used files; every file is recorded with its licence (anything not `ok` blocks publishing); each shot must show what the narration says; synthetic voices are not presented as real people; no public figures or implied endorsements.

---

### 4.6 thumbnail-design

#### What It Does

Clear, honest, on-brand YouTube thumbnails (1280x720) and Shorts covers (1080x1920), from concept to checked export.

#### When to Use

Ask Claude things like: "Make a thumbnail for this video", "Give me thumbnail concepts", "Redesign my thumbnail."

#### What You Get

3 to 5 concepts with a recommendation, final JPG/PNG under 2 MB, variants and a contact sheet, a feed-size preview, credits (`attribution.json`) and an A/B test plan.

#### Requirements

Python, Pillow, `PIXABAY_API_KEY` or `PEXELS_API_KEY`; optional rembg / remove.bg for cutouts and OpenAI for AI backgrounds.

#### Files

| File | Purpose |
|---|---|
| `scripts/thumb_compose.py` | `render` a JSON spec, `variants`, `resize`, `sheet`. |
| `scripts/thumb_check.py` | Format, size, brightness, contrast, text size at feed size, margins, duration-badge corner, overlap, off-canvas images. |
| `scripts/thumb_assets.py` | `search` (with credits), `cutout`, `ai-image`, `prompt`, `upscale`, `competitors` (study only). |
| `references/design-principles.md` | Composition, text, colour, faces, concept families, testing, mistakes. |
| `references/workflow-and-concepts.md` | Concept template, spec format, prompts, brand kit. |
| `references/tools-and-apis.md`, `references/specs-and-policy.md` | Tools, limits, YouTube specs and policy. |

#### How It Works

**Spec example:**

```json
{
  "size": "youtube",
  "output": "out/thumb.jpg",
  "background": {"image": "assets/ac.jpg", "focus": [0.3, 0.5], "darken": 0.15, "vignette": 0.35},
  "layers": [
    {"type": "shape", "shape": "rect", "xy": [0.0, 0.0, 0.47, 1.0], "fill": "#0B3D91"},
    {"type": "text", "text": "26°", "box": [0.03, 0.35, 0.42, 0.5], "color": "#FFE600",
     "stroke": {"color": "#000000", "width": 10}, "shadow": true}
  ]
}
```

Positions are fractions of the canvas. **For image layers, `x`/`y` are margins from the anchor edge, not positions.** A photo without transparency shows as a rectangle; cut it out or use it as the `background`.

#### Commands

```text
thumb_compose.py render SPEC.json [--out PATH]
thumb_compose.py variants SPEC.json --texts "A|B|C" [--themes dark,bright,warm,cool] [--flip] [--max 6]
thumb_compose.py resize IMAGE --to youtube|youtube_hd|shorts|WxH | sheet IMG [IMG ...] --out sheet.jpg
thumb_check.py IMAGE [--preview DIR] [--json out.json]
thumb_assets.py search "query" --out DIR [--source auto|pexels|pixabay] [--count 6]
thumb_assets.py cutout IMAGE [--engine rembg|removebg] | upscale IMAGE --factor 2 | prompt --subject ".."
```

#### Rules

The image shows something the video delivers (no fake UI, banners, before/after or false promises); never copy another creator's thumbnail; no celebrities or AI faces of real people; 1 to 4 large words added with `thumb_compose.py`; keep key content out of the bottom-right corner.

---

### 4.7 social-media-content

#### What It Does

Platform-native posts (Instagram captions and carousels, LinkedIn, X threads, Facebook, Threads, TikTok and Shorts captions, Pinterest, Bluesky) in the brand's voice, checked against each platform's limits.

#### When to Use

Ask Claude things like: "Write a post about X", "Make a caption", "Turn this video into 5 posts", "Write a week of content."

#### What You Get

Posts grouped by idea or platform (2 to 3 hook options for important posts), alt text and visual briefs, `pack.json` and `posts.csv` ready for a scheduler, plus open questions (placeholders like `[X%]` for numbers you must supply).

#### Requirements

Python only.

#### Files

| File | Purpose |
|---|---|
| `scripts/content_tools.py` | `limits`, `check` (pack), `csv`, `thread` (split long text), `text` (check one caption). |
| `references/platform-playbook.md` | Formats, structure, lengths, hooks and CTAs per platform; language notes. |
| `references/content-formats.md` | Content types, hook patterns, templates, repurposing matrix, reply drafting. |
| `references/compliance.md` | Disclosure, claims, rights, community rules, privacy, pre-publish check. |

#### How It Works

**Pack format:**

```json
{"campaign": "Launch", "posts": [
  {"id": "p1", "platform": "instagram", "format": "carousel", "text": "caption", "hashtags": ["#a"],
   "media": [{"file": "slide1.png", "alt": "what the image shows"}], "cta": "Save this", "link": "", "notes": ""}]}
```

#### Commands

```text
content_tools.py limits [--platform instagram]
content_tools.py check PACK.json [--json report.json]
content_tools.py csv PACK.json --out posts.csv
content_tools.py thread TEXT.txt --platform x [--limit 280]
content_tools.py text "caption" --platform instagram
```

#### Rules

One post, one idea, one action; no invented facts, testimonials or engagement; native per platform, not copy-paste; disclose paid partnerships and affiliate links; no individual medical, legal or financial advice; nothing is posted without your approval.

---

### 4.8 social-media-calendar

#### What It Does

A dated calendar the team can actually produce: cadence per platform, pillar balance, campaigns and key dates, production and approval schedule, time-zone aware times, and exports.

#### When to Use

Ask Claude things like: "Make a calendar for next month", "When should I post?", "Plan a 2-week launch campaign", "Reschedule everything after Friday by 3 days."

#### What You Get

`plan.json`, and exports `calendar.csv` (spreadsheet or scheduler import), `calendar.ics` (any calendar app, UTC times) and `calendar.md` (readable), plus a check report.

#### Requirements

Python; `tzdata` on Windows for named time zones.

#### Files

| File | Purpose |
|---|---|
| `scripts/calendar_tools.py` | `skeleton`, `check`, `export`, `shift`. |
| `references/planning-framework.md` | Capacity and cadence, pillar mix, days and times, campaigns, production, approvals, risk, reviews. |
| `references/templates.md` | Plan JSON, item fields, campaign timeline, production offsets, weekly and monthly reviews. |

#### How It Works

**Plan format:**

```json
{"brand": "Brand", "timezone": "Asia/Karachi", "start": "2026-12-01", "weeks": 4,
 "platforms": {"instagram": {"per_week": 3, "days": ["Mon","Wed","Fri"], "times": ["20:00"]}},
 "pillars": {"Educate": 40, "Proof": 20, "Story": 20, "Offer": 10, "Community": 10},
 "blackout": ["2026-12-25"], "events": [{"date": "2026-12-15", "name": "Launch", "note": ""}],
 "items": []}
```

#### Commands

```text
calendar_tools.py skeleton PLAN.json [--out PLAN.json]
calendar_tools.py check PLAN.json [--json report.json]
calendar_tools.py export PLAN.json --out DIR
calendar_tools.py shift PLAN.json --days N [--from YYYY-MM-DD] [--platform NAME]
```

`check` flags cadence mismatches, pillar drift, gaps over 5 days, duplicate titles, blackout hits, bad dates, overdue items and missing fields.

#### Rules

Capacity first (recommend the lowest cadence the team can keep for 8 weeks); days and times are assumptions until analytics confirm them; moving dates (religious, school, sales) come from a current source; sensitive dates are flagged for human review; keep a buffer of 3 to 5 approved posts.

---

### 4.9 analytics

#### What It Does

Turn platform exports into decisions: what to repeat, stop and test next, with honest limits on what the data shows.

#### When to Use

Ask Claude things like: "Check my analytics", "What is working?", "Why did my reach drop?", "Did thumbnail A beat B?"

#### What You Get

Cleaned data, summaries by format/pillar/weekday/platform, top and bottom posts, trends, best time slots (with sample sizes), A/B test results and a report with next actions.

#### Requirements

Python; CSV exports from each platform's own analytics (screenshots work, read carefully).

#### Files

| File | Purpose |
|---|---|
| `scripts/analytics_tools.py` | `normalize`, `merge`, `summary`, `top`, `trend`, `times`, `abtest`, `report`. |
| `references/data-sources.md` | Where to export data per platform, API facts and gaps, UTM setup, data hygiene. |
| `references/kpi-and-interpretation.md` | KPIs by goal, metric definitions, sample sizes, A/B test design, report structure. |

#### How It Works

**Data handling:**

- **Dates:** slash dates are read day-first or month-first from the whole column. If every date fits both (like 03/04/2026), `normalize` stops and asks for `--date-format`.
- **Durations:** `4:07` or `0:04:07` become seconds; YouTube's "Average view duration" maps automatically.
- **CTR:** clicks ÷ impressions, or YouTube's "Impressions click-through rate (%)" when there is no clicks column.
- Missing fields stay blank, never zero. Medians are used so one viral post does not distort results.

#### Commands

```text
analytics_tools.py normalize EXPORT.csv --platform NAME [--map field=Column ...] [--tz Asia/Karachi] [--date-format FMT] --out clean.csv
analytics_tools.py merge A.csv B.csv --out all.csv
analytics_tools.py summary clean.csv [--by platform|format|pillar|weekday] [--min-n 3]
analytics_tools.py top clean.csv [--metric er|views|reach|saves|shares|ctr] [--n 5] [--bottom] [--platform P]
analytics_tools.py trend clean.csv [--metric views] [--period 4]
analytics_tools.py times clean.csv [--metric er] [--min-n 3]
analytics_tools.py abtest --a-clicks N --a-impressions N --b-clicks N --b-impressions N
analytics_tools.py report clean.csv --out report.md
```

#### Rules

Start from the goal and its few KPIs; under about 5 posts per group differences are anecdotes; correlation is not cause, so propose a test; compare engagement rates only within one platform; never use competitor numbers as your own; exports are private data.

---

### 4.10 auto-publish

#### What It Does

Publish finished videos as short clips through **[Syno Clips](https://clips.synofex.com/r/5cb59f4d)** to your connected YouTube, Instagram, TikTok and Facebook accounts. Claude prepares everything automatically (clips, titles, hashtags, times, checks); posting happens only after you approve the exact list.

#### When to Use

Ask Claude things like: "Publish my video", "Make shorts from this YouTube link and schedule them", "Post the clips on my calendar slots", "What is scheduled?", "Why did my post fail?"

#### What You Get

Clips made in Synoclips, a checked posting plan (`plan.md` / `plan.json`), scheduled or published posts after your yes, a `publish_log.csv` of every result, and follow-up on failures and views.

#### Requirements

The Synoclips connector in the Claude app, a Synoclips plan with auto-posting and scheduling (ask "show my Synoclips credits and plan"), and accounts connected on the Synoclips website. Python for `publish_plan.py`.

No account yet? [Sign up for Syno Clips](https://clips.synofex.com/r/5cb59f4d).

#### Files

| File | Purpose |
|---|---|
| `SKILL.md` | Workflow, approval rules and failure handling. |
| `scripts/publish_plan.py` | `build` a posting plan from chosen clips and calendar slots, `check` an edited plan, `log` each result. Never posts. |
| `references/synoclips-tools.md` | Every connector tool, its inputs, limits and which ones need approval. |

#### How It Works

1. Setup check: connected accounts, credits, plan features.
2. Video in: upload the local file (after your OK) or give a YouTube, Twitch or Kick link.
3. Clips are made with settings that fit the video: `blur` layout for stock footage, `face` for a talking head, 1080p, audio clean-up.
4. Review clips: honest hooks and titles, brand-voice check, facts from the source video; fix titles, hooks and hashtags.
5. Build the plan from `calendar.csv` (social-media-calendar) or fixed times; fix every FAIL.
6. **You approve** the table (time, account, clip, title, hook).
7. Each approved item is scheduled (or posted now if you asked) and logged.
8. Follow up: scheduled list, failed posts with reasons, views for the analytics skill.

**Two ways in:** a **video file from this computer** (mp4, mov, mkv, webm) through `get_video_upload_link`, or a **YouTube, Twitch or Kick link** through `create_clips_from_link`. For a file, Claude asks first (file name, size, settings), then runs the upload script the tool returns, or gives you a private upload page (valid one hour) to drop the file on. For a video made with long-video-generation, finish the human review (`review.human_checked: true`) first.

**Plan checks:** past times, more than 120 days ahead, blackout dates, the same clip twice, missing accounts, title over 100 / hook over 60 / description over 2,000 characters, more than 30 hashtags, banned brand words, empty titles, posts on one account less than 4 hours apart, and scores under your minimum.

**Related:** for continuously watching a channel and clipping every new upload, the separate `synoclips-autopilot` plugin covers that workflow.

#### Commands

```text
publish_plan.py build --clips clips.json --accounts ID[,ID] (--calendar calendar.csv | --start YYYY-MM-DD --times 09:00,18:00)
                      [--tz Asia/Karachi] [--per-day 2] [--platforms youtube,instagram,tiktok,facebook] [--min-score 0]
                      [--max 10] [--min-gap-hours 4] [--blackout DATES] [--rules rules.json] [--out plan]
publish_plan.py check plan.json [--rules rules.json] [--min-gap-hours 4]
publish_plan.py log plan.json --item N --status scheduled|posted|failed|cancelled [--post-id ID] [--note TEXT]
```

#### Rules

Nothing is posted, scheduled, rescheduled, cancelled or retried without your explicit yes for those exact items; only content you have rights to; synthetic voices disclosed where the platform asks; no misleading hooks; failures reported as they are.

---

## 5. Troubleshooting

| Problem | Skill | Fix |
|---|---|---|
| A skill does not start | all | Check the folder is in `%USERPROFILE%\.claude\skills\` and contains `SKILL.md`; restart the app. |
| "Set YOUTUBE_API_KEY" | research skills | `setx YOUTUBE_API_KEY "..."`, restart the app. |
| Quota exceeded | research skills | Wait for the daily reset; plan fewer `search` queries (100 units each). |
| "Unknown timezone Asia/Karachi" | calendar, analytics | `pip install tzdata`. |
| Piper "file not found" or crash on Urdu | long-video | Set `PIPER_CMD` exactly as in section 2.4 (includes `-X utf8`). |
| Many off-topic video clips | long-video | Plainer keywords, `--kinds image` for that scene, then `review` → `reject` → `pick --fill`. |
| Description too long | long-video | `credits` shortens stock credits automatically; post `out/credits.txt` as a pinned comment. |
| Grey box around a photo | thumbnail | Cut the subject out or use the photo as the background. |
| "mostly off the canvas" | thumbnail | Image-layer `x`/`y` are margins; set them near 0. |
| Dates look wrong | analytics | Pass `--date-format "%d/%m/%Y %H:%M"` (day first) or `"%m/%d/%Y %H:%M"` (month first). |
| Network blocked | any script | Run the printed command on your own machine; scripts also read exported files. |
| "Synoclips connector is not connected" | auto-publish | Add the Synoclips connector in the Claude app and sign in. |
| A platform is missing | auto-publish | Connect the account on clips.synofex.com → Settings → Integrations. |
| Claude cannot upload a local video file | auto-publish | Reconnect the Synoclips connector in the Claude app so it loads its latest tools (or start a new chat). Meanwhile, use a YouTube link. |
| Upload link expired | auto-publish | Links last one hour; ask Claude for a new one (it checks your video list first so nothing is uploaded twice). |
| A post failed | auto-publish | Ask "why did my post fail?"; Claude reads the reason and asks before retrying. |

---

## 6. Language support

All documentation is in English. The skills also create and analyse content in **Urdu, Roman Urdu and Hinglish**:

- **brand-voice** measures the language mix of your posts and keeps spellings consistent.
- **content-strategy** understands Roman Urdu and Hinglish search phrases when it looks for content gaps.
- **short-form-video-scripts** flags weak greeting openers in English, Roman Urdu and Hindi.
- **long-video-generation-using-free-clips** can narrate with Urdu voices (written in Urdu script) and produce right-to-left subtitles.

---

## 7. Sponsor

These skills are sponsored by **[Syno Clips](https://clips.synofex.com/r/5cb59f4d)**: AI clipping for live streams and videos, with captions, framing and scores, and publishing to four platforms. The auto-publish skill uses it to turn your finished videos into Shorts, Reels and TikToks.

<p align="center"><a href="https://clips.synofex.com/r/5cb59f4d"><b>Try Syno Clips →</b></a></p>
