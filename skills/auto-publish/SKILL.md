---
name: auto-publish
description: Publish finished videos as short clips through Synoclips (clips.synofex.com) to connected YouTube, Instagram, TikTok and Facebook accounts. Takes a video file from this computer (mp4, mov, mkv or webm, for example the long video made with long-video-generation-using-free-clips) or a YouTube, Twitch or Kick link, makes vertical clips, reviews them against the brand rules, builds a checked posting plan from the content calendar, and after the user approves the exact list, schedules or posts each clip and logs the result. Use this skill whenever the user wants to publish, post, schedule or auto-post a video or its clips, mentions Synoclips or clips.synofex.com, asks to "publish my video", "schedule my clips", "post the shorts", "auto publish", or asks what is scheduled or why a post failed.
---

# Auto-publish (Synoclips)

Takes a finished video to published short clips with as little manual work as possible, while the user keeps the final say. "Auto" means Claude prepares everything (clips, titles, hashtags, times, checks); posting happens only after the user approves the exact list of posts.

## Read this first

- **Approval is required for every batch.** Before calling `schedule_clip` or `publish_clip_now`, show the plan (which clip, which accounts, when) and wait for a clear yes in chat. An approval covers only the list shown; a new or changed list needs a new yes. The same applies to `cancel_scheduled_post`, `change_post_time` and `retry_post`.
- **Ways in.** A local file (mp4, mov, mkv, webm) goes in through `get_video_upload_link`, which has two purposes:
  - `purpose: "clips"` (the default) makes clips from the file with the settings you pass;
  - `purpose: "post"` sends a finished video, such as a ready-made Short or Reel, unchanged to the Schedule page, the same as uploading it on the website's Schedule page. Nothing is cut and no clips or captions are made.

  A YouTube, Twitch or Kick video goes in through `create_clips_from_link`, which always makes clips.
- **Pick the purpose before uploading.** Use `"post"` when the file is already the video to publish (vertical, final edit, its own captions and music). Clip mode on a short finished video can come back with "nothing clip-worthy was found" and no clips, and it can burn its own captions over the design. Use `"clips"` when the file is a long video to cut into shorts.
- **Uploading sends the user's file to their Synoclips account.** Before running the upload script, say which file (name and size), which purpose, and for `"clips"` which clip settings, and wait for a yes. Read the script the tool returns before running it; run only that script, with only that file. If the user asks only to upload, stop after the upload: do not retitle, add hashtags or schedule.
- **Gate for videos from long-video-generation-using-free-clips:** do not clip or post one until its `project.json` has `review.human_checked: true` (facts, licences and the full watch-through done).
- Creating clips and some edits use the user's Synoclips credits. Say so before starting and report the credits left.

## Tools

The Synoclips connector provides the tools (names end in the part after the last `__`). The main ones:

| Step | Tool |
|---|---|
| Check setup | `list_accounts`, `get_credits_and_plan` |
| Bring the video in | `get_video_upload_link` (local file; `purpose` `"clips"` or `"post"`), `create_clips_from_link` (link), `get_video_status`, `list_videos`, `get_clip_options` |
| Review and fix | `list_clips`, `get_clip`, `get_clip_caption_text`, `get_clip_transcript`, `rename_or_retitle_clip`, `set_clip_hashtags`, `edit_clip` |
| Publish | `schedule_clip`, `publish_clip_now` (approval first) |
| Follow up | `list_scheduled_posts`, `list_posts`, `change_post_time`, `cancel_scheduled_post`, `retry_post`, `get_analytics` |

If the tools are not available, say the Synoclips connector is not connected, and give the manual path (clips.synofex.com, then the same steps by hand). Full details and limits: `references/synoclips-tools.md`.

## Workflow

1. **Setup check.** Call `list_accounts` and `get_credits_and_plan`. Report which accounts are connected (platform and account id only), credits left, and whether auto-posting and scheduling are in the plan. If a wanted platform is missing, give the connect link from the response (accounts are connected on the website, not here).
2. **Source video.** For a video from the long-video skill, first confirm `review.human_checked` is true and its licences are `ok`. Then:
   - **Local file to cut into clips** (`out/video.mp4` or any long mp4, mov, mkv, webm): call `get_video_upload_link` with `purpose: "clips"` and the clip settings from step 3. It returns a private link that works for one hour and, for Claude Code, a script that uploads a file from this computer. Ask before uploading (file name, size, purpose, settings), read the script, then run it with that file. If the script cannot run (no network, a blocked command), give the user the link to open and drop the file on the page. Then find the new video with `list_videos` (newest first) and follow it with `get_video_status`.
   - **Finished video to post as it is** (a Short, Reel or TikTok that is already edited): call `get_video_upload_link` with only `purpose: "post"`. Clip settings do not apply. Ask before uploading (file name, size, purpose), read the script, then run it with that file; the script ends with "uploaded, ready to post". The result is a `list_videos` entry with `kind: "post"` and stage "Ready to schedule", holding one clip `v1` (kind `upload`) that covers the whole file. Its title is the file name, and its hook, description and hashtags are empty. Skip steps 3 and 4's clip review: set the title, description and hashtags with `rename_or_retitle_clip` and `set_clip_hashtags` (only when the user asked for them), then go to step 5 with that `video_id` and `clip_id: "v1"`.
   - If `get_video_upload_link` does not list `purpose`, the session loaded an older copy of the tool: reconnect Synoclips (`/mcp` → Reconnect) or start a new session. The server may still accept `purpose`, but say that you are relying on it before calling.
   - **Link** (YouTube, Twitch or Kick): call `create_clips_from_link` with the link and the settings from step 3.
   Never claim a file was uploaded until `list_videos` shows it.
3. **Clip settings.** Choose settings that fit the source (the same inputs for both tools):
   - faceless or stock-footage video: `layout: "blur"` (keeps the full frame); talking head: `layout: "face"`;
   - `quality: 1080`, `enhance_audio: true`; `music: "off"` if the video already has music;
   - `mode: "ai"` by default, or `mode: "custom"` with `count`, `min_length`, `max_length` if the user wants a set number;
   - `caption_style` from `get_clip_options` if the brand has one.
   Clips take a few minutes. Check `get_video_status`; if it is still running, tell the user and check again later instead of calling it in a tight loop.
4. **Review clips.** `list_clips` (`sort: "best"`), then for each candidate `get_clip` and `get_clip_caption_text`:
   - the hook and title must be true to the clip (no promises the clip does not deliver);
   - check caption text against the brand voice: save it to a file and run `brand-voice/scripts/voice_tools.py check FILE --rules rules.json` if that skill is installed;
   - every number shown or said must come from the source video's facts;
   - fix with `rename_or_retitle_clip` (title ≤ 100, hook ≤ 60, description ≤ 2,000 characters) and `set_clip_hashtags` (≤ 30; a few relevant ones are better). Use `edit_clip` for trims or caption word fixes (may use credits).
   Drop weak or misleading clips rather than polishing them.
5. **Build the plan.** Save the chosen clips (video_id, clip_id, title, hook, description, hashtags, score) to `clips.json`, then:
   ```bash
   python scripts/publish_plan.py build --clips clips.json --accounts ACCOUNT_ID[,ACCOUNT_ID] --calendar calendar.csv --tz Asia/Karachi --rules rules.json
   ```
   Use the `calendar.csv` exported by social-media-calendar when it exists; otherwise `--start YYYY-MM-DD --times 09:00,18:00 --per-day 2`. Fix every FAIL (edit `plan.json`, then `publish_plan.py check plan.json`).
6. **Ask for approval.** Show the `plan.md` table: number, time with time zone, account, clip title, hook. Mention credits, the synthetic-voice disclosure if the video uses one, and anything you could not verify. Wait for a clear yes; accept edits ("drop 3", "move 2 to Friday") and re-check.
7. **Publish.** For each approved item call `schedule_clip` (or `publish_clip_now` only if the user asked for "now"), with exactly the approved clip, accounts and time. After each call, log it:
   ```bash
   python scripts/publish_plan.py log plan.json --item N --status scheduled --post-id ID
   ```
   If a call fails, log `failed` with the reason and continue with the rest; do not retry without asking.
8. **Confirm and follow up.** `list_scheduled_posts` to show what is queued. Later, `list_posts` (`status: "failed"` for problems, with reasons) and `get_analytics` for views; hand the numbers to the analytics skill.

## Rules

- Never post, schedule, reschedule, cancel or retry without the user's explicit approval for those exact items in this conversation.
- Post only content the user has the rights to; stock footage licences from the long-video project must be `ok`.
- Disclose a synthetic voice or AI-generated visuals where the platform asks (for example YouTube's "altered or synthetic content" setting, which the user sets in YouTube Studio or the platform app).
- Do not post the same clip twice to the same account; space posts on one account (the plan warns under 4 hours).
- No engagement bait, fake urgency or misleading hooks; titles must match the clip.
- Account ids, video ids and post ids are fine to show; do not paste access tokens or personal data into files or chat.
- Report results faithfully: say which posts were scheduled, which failed and why.

## If something fails

- **No connected account for a platform:** give the connect link from `list_accounts`; continue with the connected ones if the user agrees.
- **Not enough credits or a feature missing from the plan:** report it from `get_credits_and_plan` and give the plans link; do not start a run that will fail.
- **Clip creation failed:** read `get_video_status`; common causes are private videos, region locks or very long live recordings.
- **Upload link expired** (after one hour) or the upload stopped: call `get_video_upload_link` again for a new link; check `list_videos` first so the same file is not uploaded twice.
- **"Done, but nothing clip-worthy was found"** on a short finished video: it went in with `purpose: "clips"`. Check `list_videos` for an existing `kind: "post"` entry of the same file first. If there is none, ask, then upload it again with `purpose: "post"`.
- **`get_video_upload_link` is missing:** the session connected before the tool existed; reconnect Synoclips (`/mcp` → Reconnect) or start a new session. Until then, use a YouTube link.
- **Post failed:** `list_posts` with `status: "failed"` gives the reason (for example a missing platform permission); fix, then ask before `retry_post`.

## Files

- `scripts/publish_plan.py`: `build` (plan from clips and calendar), `check` (re-check an edited plan), `log` (record each result in `publish_log.csv`). It never posts.
- `references/synoclips-tools.md`: every connector tool, its inputs, limits and when it needs approval.
