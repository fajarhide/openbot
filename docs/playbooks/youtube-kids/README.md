# YouTube Kids Channel Team — OpenBot Playbook

Status: built and tested on the dev profile. See [VALIDATION.md](VALIDATION.md) for the evidence.
Date: 2026-11-02.
Scope: preschool songs and early learning, English, ages 2–5. The team produces and uploads a
private draft. A person publishes.

Personas for the six agents are in [agents.md](agents.md). Operating commands are in
`~/OpenBot/Channels/pip-and-poppy/RUNBOOK.md`. Corrections for the source course are in
[errata-faceless-course.md](errata-faceless-course.md).

## 1. What the team must produce

For each episode:

1. A research card that names the search demand and the learning goal.
2. Original lyrics, a script, and a shot list.
3. A voice-over, a music bed, and a rendered 1080p master.
4. A thumbnail and the metadata (title, description, tags, chapters, playlist).
5. A compliance record that says pass or fail against YouTube's kids and monetization policies.
6. A published-episode report after 7 and 28 days.

The team stops after a private draft. Publishing the video, setting the audience, and the
monetization application stay with the person, because those are the actions that can lose the
channel.

## 2. The business constraint, before the roster

Three YouTube rules decide the design. Read them before you judge the team size.

| Rule | What it means for this channel |
| --- | --- |
| **Made for kids** content gets contextual ads only. Comments, end screens, cards, memberships, notifications, and personalized ads are off. | RPM is lower than general entertainment. Retention tools such as end screens are unavailable, so the episode itself must hold the viewer. [Ad policy](https://support.google.com/youtube/answer/9713557?hl=en), [features](https://support.google.com/youtube/answer/9632097?hl=en) |
| **Inauthentic content** is not monetizable. Since 15 July 2025 the policy names "AI-generated content made with generic or unoriginal templates" and "characters in the same situation over and over". | A template factory fails review. Every episode needs a distinct storyline and a real learning goal. [Policy](https://support.google.com/youtube/answer/1311392?hl=en) |
| **Kids quality principles** apply at channel level. Low-quality made-for-kids content gets limited or no ads; a channel that focuses on it can be suspended from YPP. | One bad batch can cost the whole channel, not one video. [Quality principles](https://support.google.com/youtube/answer/10774223) |

### The threshold window

| Route | Requirement | Status |
| --- | --- | --- |
| Ads and Premium, current | 1,000 subscribers + 4,000 public watch hours in 365 days, or 10M Shorts views in 90 days | 31 January 2027 |
| Ads and Premium, new applicants | 1,000 subscribers + 8,000 watch hours in 365 days, or 20M Shorts views in 90 days | From 1 February 2027 |
| Fan funding and shopping | 500 subscribers + 3 public uploads in 90 days + 3,000 watch hours or 3M Shorts views | Unchanged |

Source: [YPP eligibility](https://support.google.com/youtube/answer/72851?hl=en), [YouTube blog, 10 Aug 2026](https://blog.youtube/news-and-events/youtube-partner-program-updates-2027-new-opportunities-earn/).
Channels already in YPP keep the old terms. From 1 February 2027, Shorts ad and subscription
revenue needs 10M qualified Shorts views in 90 days; below that, a channel stays in YPP and earns on
long-form until it crosses the line again.

**Plan for 8,000 hours.** A new channel that starts now cannot realistically reach 4,000 hours by
31 January 2027. Build the plan on the new number and treat the old one as a bonus. 8,000 hours is
480,000 minutes. At a 3-minute average view duration that is 160,000 views, or roughly 1,300 views
per video across 120 videos. Two long-form uploads a week reaches 120 videos in about 14 months.

### Revenue model

Ads are the base, not the plan. A made-for-kids channel in the US, UK, Canada, and Australia
commonly earns about USD 0.50–2.00 per 1,000 views. A channel at 500,000 views a month earns about
USD 250–1,000. Treat that as an estimate and replace it with your own Studio RPM after 30 days.

Layers that do not depend on the kids ad limit:

- **Music distribution.** Release each song to Spotify, Apple Music, and Amazon through a
  distributor. Nursery songs repeat, so streaming royalties accumulate.
- **YouTube Premium share.** Premium watch time pays more per view than ads and is not limited by
  the made-for-kids ad rules.
- **Licensing and localization.** Dubbed versions, app content, and character licensing come later,
  but the brand bible must allow them from episode 1.
- **Merchandise.** Only after 10,000 subscribers and a character children recognize.

Do not plan on: memberships, Super Chat, branded content inside YouTube Kids (not allowed), or
keyword-stuffed compilations.

## 3. Roster

Six agents. One orchestrator, five specialists. Fewer roles means the Director does everything and
becomes the bottleneck. More roles means coordination cost with no new output.

| Agent | Role | Main artifact | Routine |
| --- | --- | --- | --- |
| **Atlas** | Channel Director. Owns the brand bible, the slate, delegation, and the release gate. | `publish/EP-xxx/checklist.md`, `brand/brand-bible.md` | Monday slate, daily stand-up |
| **Scout** | Audience and trend researcher. Finds demand, parent concerns, and competitor patterns. Never copies. | `research/EP-xxx/research.md`, weekly trend note | Monday trend scan |
| **Bard** | Curriculum and song writer. Turns one learning goal into lyrics, script, and shot list. | `scripts/EP-xxx/` | On assignment |
| **Forge** | Audio and video producer. Voice-over, music bed, render, captions, QC stills. | `renders/EP-xxx/` | Thursday render QA |
| **Quill** | Metadata and thumbnail officer. Titles, descriptions, tags, chapters, thumbnails, DSP metadata. | `publish/EP-xxx/`, `dsp/EP-xxx/` | On assignment |
| **Verity** | Compliance and monetization auditor. Blocks a release. Keeps the compliance log. | `compliance/EP-xxx.md`, `compliance/log.csv` | Friday sweep, monthly YPP check |

## 4. How OpenBot runs the team

### Agents and personas

Create six agents. Each agent has its own workspace at `~/OpenBot/Agents/<agent-id>/`. Put the
matching persona file at the workspace root as `AGENTS.md`. OpenBot reads that file as the agent's
standing instructions, so the role survives restarts and provider switches.

Set the profile title and description to match the persona heading. That text is also part of the
standing remit, and the Director uses it to pick the right teammate.

Suggested models:

| Agent | Model | Reason |
| --- | --- | --- |
| Atlas, Verity | Strong reasoning model (Claude) | Judgment, policy reading, and a release decision |
| Bard, Quill | Strong writing model | Lyrics, titles, and parent-facing text |
| Scout | Model with browser tools | Search, competitor pages, and Studio reports |
| Forge | Model with shell tools | ffmpeg, render scripts, and deterministic checks |

### One channel folder, six owners

All agents read and write one shared folder outside the workspaces. Agents never write outside
their own subfolders, so two agents cannot overwrite one file.

```
~/OpenBot/Channels/<slug>/
  brand/            brand-bible.md, characters.md, voice-guide.md, music-policy.md, lexicon.md
  research/         EP-014/research.md, weekly/2026-W45.md
  scripts/          EP-014/learning.md, lyrics.md, script.md, shotlist.md, review.md
  assets/           EP-014/vo/*.wav, music/*.wav, art/*.png, captions/*.srt
  renders/          EP-014/master-1080p.mp4, thumbnail.png, qc.md
  publish/          EP-014/title.txt, description.txt, tags.txt, chapters.txt, checklist.md
  dsp/              EP-014/isrc.txt, credits.txt, artwork.png
  compliance/       EP-014.md, log.csv, monthly/2026-11.md
  analytics/        EP-014-d7.md, EP-014-d28.md, monthly/2026-11.md
```

Episode IDs are `EP-001` upward. One folder per episode, in every stage, same ID.

### Delegation

The Director delegates with `openbot.send_message` and ends its turn. Rules that keep this reliable:

- One message per agent per episode. It carries the goal, the episode ID, the paths, the learning
  goal, and the done test. The teammate cannot see the Director's chat.
- The request says what the agent must not do. For example: "Do not invent a source. Do not use a
  known character. Do not upload."
- Replies carry artifact paths, not pasted text. The Director reads the files.
- A missing part gets one focused follow-up. The Director does not redo the work with the same tools.
- If Verity fails an episode, the Director sends the failure list back to the owner of the failed
  item and names the re-check time.

### Routines (Asia/Jakarta)

| When | Agent | Instruction |
| --- | --- | --- |
| Daily 07:00 | Atlas | List the episodes in flight, the owner of each open step, and the one decision needed today. Do not publish. |
| Monday 08:00 | Scout | Write `research/weekly/<iso-week>.md`: search demand, five competitor videos with view and retention signals, three episode ideas with a distinct storyline. |
| Monday 08:30 | Atlas | Pick the week's two episodes, write `publish/slate-<iso-week>.md`, and assign them. |
| Thursday 15:00 | Forge | Render QC: check every `renders/EP-*/qc.md` from this week against the render spec. Report failures only. |
| Friday 16:00 | Verity | Audit every episode queued for upload against the compliance gate. Write pass or fail with the failing item. |
| 1st of month 09:00 | Verity | YPP readiness check: watch hours, subscribers, policy warnings, strikes, and the current threshold. |
| 1st of month 09:30 | Scout | Write `analytics/monthly/<month>.md`: what held viewers, what lost them, and what to change next month. |

### Human gates

| Gate | Who | Check | If it fails |
| --- | --- | --- | --- |
| G1 Concept | Director | Distinct storyline, one learning goal, fits the brand | Send back to Scout |
| G2 Research | Director | Demand shown, sources linked, no copied script or art | Send back to Scout |
| G3 Script | **Person** | Reads the lyrics aloud, approves the learning goal | Rewrite with Bard |
| G4 Master | **Person** | Watches the whole video on a phone, checks audio and captions | Fix with Forge |
| G5 Package | Verity | Compliance gate is pass, metadata matches the video | Hold the release |
| G6 Upload | **Person** | Audience set to made for kids, monetization on, schedule set | — |

The person spends about 30 minutes per episode. That is the cost of keeping the channel.

## 5. Pipeline

| # | Step | Owner | Output | Done test |
| --- | --- | --- | --- | --- |
| 1 | Trend scan | Scout | `research/weekly/<week>.md` | Three ideas, each with a different storyline and a linked demand signal |
| 2 | Slate | Atlas | `publish/slate-<week>.md` | Two episodes assigned with an ID and a due date |
| 3 | Research card | Scout | `research/EP-xxx/research.md` | Learning goal, target phrase, three reference videos, parent concern, do-not-copy list |
| 4 | Learning design | Bard | `scripts/EP-xxx/learning.md` | One measurable goal, vocabulary list of 8–12 words, one repeatable refrain |
| 5 | Lyrics and script | Bard | `lyrics.md`, `script.md` | Original lyrics, 180–270 second script, no sentence over 12 words, no scary beat |
| 6 | Shot list | Bard | `shotlist.md` | Every scene has a duration, a character action, and a caption line |
| 7 | Script review | Person | `scripts/EP-xxx/review.md` | Approved, or a written change list |
| 8 | Voice and music | Forge | `assets/EP-xxx/vo/`, `music/` | No clipped words, loudness −14 LUFS integrated, music at least 18 dB under the voice |
| 9 | Render | Forge | `renders/EP-xxx/master-1080p.mp4` | 1920×1080, 30 fps, H.264, AAC 192 kbps, no black frame over 0.5 s, loudness measured at −14 LUFS, captions as an SRT beside the master, not burned in |
| 10 | Thumbnail and metadata | Quill | `publish/EP-xxx/` | Title under 60 characters, no keyword stuffing, thumbnail text at most 4 words |
| 11 | Compliance | Verity | `compliance/EP-xxx.md` | Every item in section 7 is checked, with one evidence line each |
| 12 | Release | Person | Private draft upload | Made for kids, captions uploaded, schedule set, compliance record linked in the description file. A person publishes from Studio. |

After release: Scout writes the day-7 and day-28 reports, and Atlas uses them for the next slate.

## 6. Episode quality bar

These numbers are the bar the Director enforces. They are targets for a new channel, not promises.

| Signal | Target | Read at |
| --- | --- | --- |
| Thumbnail click-through | 4–8% | Day 7 |
| Average view duration | 55% or more | Day 7 |
| Viewers still watching at 30 seconds | 80% or more | Day 7 |
| Returning viewers | Rising month over month | Day 28 |
| Subscribers per 1,000 views | 3 or more | Day 28 |

Quality rules that beat any metric:

- One learning goal per episode. A second goal goes into the next episode.
- The refrain repeats at least three times, with the same melody and the same words.
- The last 10 seconds recap the learning goal in one sentence.
- Every episode has a problem, an attempt, and a resolution. No episode is a slideshow.
- No character from another studio. No "surprise egg" format. No fake learning claim.

### The 7-day scorecard

Every episode gets a card at day 7 and day 28, in `analytics/EP-xxx-d7.md` and `analytics/EP-xxx-d28.md`.
Use the same fields in the same order, so two episodes can be compared.

| Field | Where it comes from |
| --- | --- |
| Views at 24 hours, 72 hours, 7 days | Studio, Content tab |
| Impressions and click-through rate | Studio, Reach |
| Average view duration and percentage | Studio, Engagement |
| Viewers still watching at 30 seconds | Retention curve, first point after the intro |
| Saves and shares per 1,000 views | Studio, Engagement |
| Subscribers gained | Studio, Audience |
| Top three traffic sources | Studio, Reach |

Decision rule, written down at every check:

- After 10 episodes: keep the format when average view duration is 55% or more.
- After 20 episodes: change the hook or the length when 30-second retention is under 70%.
- After 30 episodes: stop this format when no episode passed 1,000 views. Write the reason in
  `analytics/decision-<date>.md` and pick a new format, not a new coat of paint.
- Never change two variables in the same week. Then you cannot tell which one worked.


## 7. Compliance gate

Verity checks every line and writes the evidence next to it. One failed line holds the release.

**Audience and law**

- [ ] The video is set as made for kids. The content targets children, so this is not optional.
- [ ] The channel does not claim a 13+ audience to unlock personalized ads.
- [ ] No personal data is collected: no forms, no email capture, no third-party tracking links.

**YouTube Kids and quality**

- [ ] No branded content, endorsement, or product placement. These are removed from YouTube Kids.
- [ ] No product focus, unboxing, or "buy this" call to action.
- [ ] The title and thumbnail describe the content. No keyword stuffing, no misleading promise.
- [ ] The storyline is distinct from the last four episodes, not a template with new colors.
- [ ] The video matches one high-quality principle: self-care, curiosity, creativity, life skills, or
      the wider world.
- [ ] No low-quality marker: deceptive education, jumbled narrative, sensational framing, or a known
      children's character in a new situation.

**Advertiser-friendly**

- [ ] No scary image, threat, injury, or distress, including a character that appears young.
- [ ] No negative behavior shown as fun: no cheating, bullying, waste, or unsafe imitation.
- [ ] No medical, health, or developmental claim.
- [ ] Music, art, voice, and fonts are licensed for commercial use and the license file is linked.

**Monetization and originality**

- [ ] The lyrics are original. Public-domain melody is fine; a recording from another channel is not.
- [ ] The script was written for this episode. AI tools may draft, but a person edited and approved it.
- [ ] Synthetic media disclosure is set where the content could be mistaken for real. Stylized
      animation usually does not need it; a realistic human or animal does.
- [ ] The channel keeps the evidence that a person reviewed the script and the master.

**Release hygiene**

- [ ] The description states the learning goal in one sentence.
- [ ] The episode sits in the right playlist and series.
- [ ] The compliance record path is in `publish/EP-xxx/checklist.md`.
- [ ] No upload can happen without a `PASS` record and a person's approval. The uploader writes a
      private draft only; a person publishes.

### Policy numbers must come from the source

A number that describes a platform rule has an expiry date: a watch-hour threshold, a Shorts view
threshold, the chapter rule, the audience setting, the AI disclosure rule. Verity does not accept it
from a course, a video, or an old script.

- Check it on the official help page on the day the episode is written.
- Record the URL, the value, and the date in `compliance/sources.md`.
- Remove a number that cannot be checked. Do not soften it to "around" or "about".
- Recheck the file during the monthly routine and update the values.

Section 2 holds the values that were current on 2 November 2026. They change for new applicants on
1 February 2027, so a script written this year can be wrong next year.

## 8. Tooling and cost

| Need | Option | Monthly cost |
| --- | --- | --- |
| Render | Remotion (React to MP4), or ffmpeg with a PNG sequence | 0 |
| Voice | Licensed TTS with a commercial clause, or a hired voice actor | 5–25 |
| Music | Production library with a YouTube clause, or an original composer | 15–30 |
| Captions | Whisper locally, corrected by Forge | 0 |
| Art | Licensed illustration set, or a designer per series | 20–60 |
| Thumbnails | One Figma template, exported by script | 0 |
| Distribution | Music distributor for Spotify and Apple | 20/year |
| **Total** | | **40–115** |

Compute cost is the variable. One 4-minute episode is roughly 20–40 minutes of rendering and 30–60
minutes of agent time. Render on this computer, not in a cloud account, until the format is stable.

### Upload through the API

The uploader uses the YouTube Data API with a desktop OAuth client in the channel owner's Google
Cloud project. One episode costs about 2,050 quota units: 1,600 for the video, 400 for the caption
track, 50 for the thumbnail. The default daily quota is 10,000 units, so four episodes a day is the
ceiling.

Two rules keep it safe:

- The uploader can only write `privacyStatus: private` and `selfDeclaredMadeForKids: true`. It
  refuses any other value. A person publishes from Studio.
- The OAuth consent screen has a publishing status. While it is in **Testing**, Google expires the
  refresh token after **seven days**, so a weekly routine breaks silently after the first week.
  Either re-authorise weekly, or publish the app while it is still unverified. Publishing shows a
  warning screen and caps the app at 100 users, which is irrelevant for one channel.

### Break-even and loss limit

Write both numbers before episode 1. They decide when to stop, not when to feel optimistic.

Break-even through ads only, at a fixed cost of USD 60 a month:

| RPM | Views per month to break even |
| --- | --- |
| USD 0.50 | 120,000 |
| USD 1.00 | 60,000 |
| USD 1.50 | 40,000 |

Formula: `monthly cost ÷ RPM × 1,000`. Premium watch time and music distribution shorten the
break-even; count them as separate lines, not as a reason to lower the target.

Loss limit, example: USD 500 across 6 months, covering tools and art. Review it at video 10, 20, and
30. At video 30, either the scorecard in section 6 passes or the format changes. Do not extend the
limit without writing down why.

One more rule from the same source: never buy a channel, never buy subscribers, and never join an
engagement pod. Those put the monetization review at risk for a gain that does not last.

## 9. Rollout

| Week | Work | Exit test |
| --- | --- | --- |
| 1 | Brand bible, characters, voice guide, music policy | A stranger can describe the channel in one sentence |
| 2 | Create six agents, load personas, create the channel folder | One test episode passes the pipeline with placeholders |
| 3 | Episode 1 and 2, human review, upload | Both pass the compliance gate with no waiver |
| 4–8 | Two episodes a week, day-7 and day-28 reports | Retention at 30 seconds above 70%, on the way to the 80% bar |
| 9–12 | Fix the weakest signal, add dubbing plan, start DSP releases | A repeatable cost per episode and a rising returning-viewer line |

Do not raise the cadence above two long-form episodes a week until the compliance gate passes five
episodes in a row without a waiver. Volume is how channels fail the inauthentic-content review.

## 10. Risks

| Risk | Signal | Control |
| --- | --- | --- |
| Template production | Episodes differ only by color and noun | Scout must name a distinct storyline; Verity fails a repeat |
| Scary or distressing scene | A character in distress, even animated | Bard's hard stop, Verity's gate, person watches the master |
| Copyright claim | A melody, character, or font from another owner | License file per asset; no known characters |
| Wrong audience setting | Content for toddlers set as 13+ | Person sets it at upload; Verity checks the screenshot |
| Channel-level YPP suspension | A batch of low-quality kids videos | Two-episode cadence, human review, compliance log |
| Voice or likeness problem | A cloned voice without consent | Licensed TTS only, or a contracted actor with a release |
| Automation error | A bad render or wrong file uploaded | The uploader refuses a public status and a false kids flag; a person publishes |
| Upload stops after one week | OAuth refresh token expires in Testing mode | Publish the OAuth app, or re-authorise; a failed upload is logged, not silent |

## 11. Build state and next step

Steps 1–5 below are done, and [VALIDATION.md](VALIDATION.md) records the evidence:

1. Six agents created with the titles from section 3. ✅
2. Each persona written as `AGENTS.md` in its workspace, placeholder replaced. ✅
3. `~/OpenBot/Channels/pip-and-poppy/` created with the folder contract and five brand files. ✅
4. Seven routines created in section 4. ✅
5. `EP-000` rendered through the whole pipeline. ✅

What remains before episode 1 can ship:

1. Buy or record a voice with a commercial YouTube licence, and replace the prototype voice.
2. Create the Google Cloud OAuth desktop client and run the one-time authorisation.
3. Decide the OAuth token option in `RUNBOOK.md` section 2, because a Testing-mode token expires in
   seven days.
4. Replace the placeholder scene builder with the real art system.

Do not raise the frequency above two long-form episodes a week. Volume is how a kids channel fails
the inauthentic-content review.
