# Validation — does this playbook work?

Tested on 2 November 2026 on macOS (arm64), Python 3.15, ffmpeg 8, Node 24.

The design in [README.md](README.md) was built and run, not only reviewed. This file records what
passed, what broke, and what changed because of it. The ebook errata is in
[errata-faceless-course.md](errata-faceless-course.md).

## 1. What was built

| Part | Location | State |
| --- | --- | --- |
| Six agents, personas, titles | OpenBot dev profile | Installed |
| Seven routines, Asia/Jakarta | OpenBot dev profile | Active |
| Brand files, five | `~/OpenBot/Channels/pip-and-poppy/brand/` | Written |
| Season slate, eight episodes | `publish/slate-2026-W45.md` | Written |
| Policy source register | `compliance/sources.md` | Written |
| Renderer, Python + ffmpeg | `tools/render_episode.py` | Runs |
| Uploader, Node | `tools/upload_youtube.mjs` | Dry run passes; real upload blocked |
| Operating instructions | `RUNBOOK.md` | Written |

## 2. Test results

| # | Claim from the playbook | Method | Result | Evidence |
| --- | --- | --- | --- | --- |
| T1 | Six agents can be created with a persona and a title | Ran the seeder against the dev profile database | **Pass** | Six rows in `projection_agents`; each `~/OpenBot/Agents/<id>/AGENTS.md` starts with its own name |
| T2 | The seeder is safe to run twice | Ran it a second time | **Pass** | `created: []`, `updated: [all six]`, no duplicate rows |
| T3 | Seven routines can hold the weekly cadence | Created from the same script | **Pass** | Seven active rows in `projection_agent_routines`, timezone Asia/Jakarta |
| T4 | Personas have no unresolved placeholder | Grepped every generated `AGENTS.md` for `{{CHANNEL_ROOT}}` | **Pass** | 0 matches in six files |
| T5 | One episode renders to the master spec | Rendered `scripts/EP-000/episode.json` | **Pass** | 7.35 s, 1920x1080, 30 fps, H.264, AAC 192 kbps, 213,855 bytes |
| T6 | Loudness lands at −14 LUFS and −1 dBTP | Measured the master with `loudnorm` | **Pass after a fix** | −14.27 LUFS integrated, −1.22 dBTP |
| T7 | Captions are produced and timed | Read `assets/EP-000/captions/EP-000.srt` | **Pass** | Four cues, first at 00:00:00,000, last ends 00:00:07,280 |
| T8 | QC writes the measured values, not claims | Read `renders/EP-000/qc.md` | **Pass after a fix** | Width 1920, height 1080, duration 7.349, loudness and true peak both present |
| T9 | Uploader builds a correct job | `upload --dry-run` | **Pass** | Prints file, bytes, title, privacy, kids flag, thumbnail, captions |
| T10 | Uploader refuses a public upload | Meta with `privacyStatus: public` | **Pass** | Error, exit non-zero |
| T11 | Uploader refuses a false kids flag | Meta with `madeForKids: false` | **Pass** | Error, exit non-zero |
| T12 | Uploader refuses to run without a token | No `tools/token.json` | **Pass** | `Run ... auth first` |
| T13 | A real upload works end to end | Needs an OAuth desktop client | **Blocked** | No `tools/oauth-client.json`; see section 4 |
| T14 | The folder contract is enough for handoffs | Ran the pipeline across `scripts/`, `assets/`, `renders/`, `publish/` | **Pass after a fix** | Each artifact landed in its owner's folder |
| T15 | Personas refer only to sections that exist | Compared `README section N` references against the headings | **Pass** | Sections 6 and 7 both exist |
| T16 | The installed persona matches its source | Compared each `AGENTS.md` with `agents.md` after placeholder substitution | **Fail, then pass** | Forge had the old caption line; re-seeded, then 6/6 in sync |
| T17 | An old episode spec still renders after the art change | Rendered the original EP-000 spec | **Pass** | Same 7.35 s, −14.27 LUFS, −1.22 dBTP |
| T18 | The art kit covers three characters, five expressions, six poses, six backgrounds | `artkit.py sheet`, read every sheet | **Pass after a fix** | See F10 |
| T19 | The speaker's mouth moves only while the voice plays | Pixel difference of the mouth crop at four timestamps | **Pass** | Open vs closed 156,126; same state 3,762; closed vs tail 4,524 |
| T20 | A cast entry with an unknown value fails loudly | `artkit.character` raises for an unknown character, expression, or pose | **Pass** | `ValueError` names the allowed set |
| T21 | Scout turns a slate row into a sourced research card | Ran Scout's persona on EP-001, then spot-checked the card | **Pass** | Seven fields, 21 distinct links. CDC page reads five steps and 20 seconds, as cited. R1 views 85,015,521 on the card against 85,015,551 on the page |
| T22 | Scout reports a conflict instead of hiding it | Read the unclear-items list | **Pass** | Nine items. Four were real defects of ours: see F12, F13, F16 |
| T23 | Bard's script meets the mechanical rules | `check_shotlist.py` on EP-001 | **Pass** | 39 scenes, 195 s, no failures |
| T24 | The checker can fail | Negative test with five planted faults | **Fail, then pass** | First run caught 4 of 5; the avoided word in a voice line slipped through. After the fix: 7 failures reported, real EP-001 still 0 |
| T25 | The render matches the planned duration | Rendered EP-001 and read `qc.md` | **Fail, then pass** | 107.2 s against 195 s planned; after F14, 195.08 s |
| T26 | A child can see the skill in the picture | Read a contact sheet of the render | **Fail** | No water, soap, or rubbing hands in any frame. See F16 |
| T27 | The song is sung over a music bed | Listened for music: none was produced | **Fail** | The voice reads the lyrics as speech. See F17 |
| T28 | The stage cartoon (bang-motion) builds and runs | `build.py`, then 16 snapshots | **Pass** | No JS error; characters, goo, expressions, and background objects move |
| T29 | The close-up stage shows the hands | Contact sheet of the sink stage | **Fail, then pass** | Hands were off-screen: position counted twice (F19). After the fix: hands, soap, water, foam visible |
| T30 | The new render is more alive than the old | `frame_qa.py` on both masters | **Pass** | Held frames 99.4% to 16.6%; frozen time 144 s to 0 s; dead seconds 188 of 195 to 2 of 36; mean change 0.087% to 4.48% of pixels |
| T31 | Limbs stay attached at the joints | Two 8 fps strips of consecutive frames (Poppy rubbing, Pip hands up) | **Pass** | No detached limb. Two defects noted in F22 |
| T32 | Loudness is in range | `ebur128` on the final mux | **Pass after a fix** | First pass -16.0 LUFS; after gain and limiter -14.3 LUFS, -1.0 dBTP |
| T33 | The Higgsfield key is accepted | Read-only call: real key on a fake request id, and a deliberately wrong key | **Pass** | 404 with the real key, 401 with the wrong one. No credit spent. The key is never printed |
| T34 | The two Higgsfield pilot clips run | Submitted Genjutsu Light motion transfer and Seedance 2.5 image-to-video, 720p | **Blocked** | Both answered `403 not_enough_credits`. Uploads worked; nothing was charged. The account needs credit before any clip can run |
| T35 | The rub is visible | Pixel change in the hand region, 21 samples 0.1 s apart | **Pass** | 12.75% of pixels per step around the hands, against 2.64% on an empty wall |
| T36 | One loudness step serves both render paths | `finalize_audio.py` on three masters | **Pass after a fix** | -16.0 to -14.3, in range unchanged, -14.4 to -14.2; encoded once |
| T37 | A storyboard of the whole episode exists before it is built | 19 planned panels and 9 real frames, one sheet per chapter | **Pass** | `storyboard/sheet-ch0.png` to `sheet-ch4.png` in the channel folder |
| T38 | The renovated stage renders and is richer without leaving the flat style | Snapshots, then a full export; before and after at six timestamps | **Pass** | Rug, perspective tiles, cabinet doors, plant, pump label, finger lines, cuffs, thumb. No gradient was added |
| T39 | The renovated render is still alive and in range | `frame_qa.py` and `finalize_audio.py` on the new master | **Pass** | Dead seconds 2 of 36 to 0 of 36; held frames 15.1% to 10.0%; mean change 4.51% to 5.41%; -14.3 LUFS, -1.3 dBTP |
| T40 | Chapters 0 and 4 build from the shared stage and run | `build_chapter.py` for both, snapshots of key seconds | **Pass** | No JS error; walk-in, hops, tail wave, towel reach and pat, three sink inserts, wave and pull-back all visible |
| T41 | The new art asked for by the storyboard exists | Contact sheets | **Pass** | Walk cycle, towel swing, pat with vibration marks at Pip's hands |
| T42 | Chapters 0, 1, 4 join without a visible break | Frames either side of both joins | **Pass** | Rinse close-up leads to the cheer; Momo-and-Poppy shot leads to the wide shot |
| T43 | The joined preview is alive and in range | `frame_qa.py` on the 56.97 s file | **Pass** | 0 frozen seconds, 1 of 56 dead, no black frame, no flash; -14.2 LUFS, -1.3 dBTP |
| T44 | Chapters 2 and 3 build and run with the new hand art | Snapshots of every slot, then a full export | **Pass after two fixes** | Backs of hands with nails, thumb sparkle, fingertip nails; see F29 |
| T45 | The song chapters are alive | `frame_qa.py` | **Pass** | Chapter 2: 0 of 45 dead seconds, 6.1% held; chapter 3: 0 of 40, 8.1% held; no frozen, black, or flash |
| T46 | The music has a steady 96 BPM pulse locked to the 5 s slots | Waveform and spectrogram of the synthesized song | **Pass, structure only** | Regular beats, a stronger hit at each bar. **Nobody has listened to it**; see F31 |
| T47 | The subtitles are clean | Parsed the SRT | **Pass** | 55 cues, 0 overlaps, shortest 0.96 s, fastest 15.1 characters a second |
| T48 | The episode assembles and is in range | `assemble_episode.py`, `frame_qa.py` | **Pass** | 142.67 s (2:22.7), 1080p30, 3 of 142 seconds dead (6, 49, 135), no frozen, black, or flash; -14.3 LUFS, -1.3 dBTP |
| T49 | The soft subtitle track is off by default in the MP4 | `ffprobe` on four ways of muxing | **Fail** | The MP4 muxer marked it default every time. The MKV can be off; see F30 |
| T50 | The burned-in preview is readable and inside the safe area | Six frames of the preview | **Pass with a note** | The pill sits at y 868 to 964; in close-ups it covers a cuff |

## 3. Findings and the revision each one caused

### F1. Single-pass loudness normalization undershoots — fixed

The first render applied `loudnorm` once and measured −15.30 LUFS, 1.3 LU off target. A single pass
on short mono speech is a known weakness: it normalizes on a guess of the signal.

**Revision.** The renderer now measures the joined audio, feeds the measured values into a second
pass, then re-measures the master and corrects up to twice with a limiter at 0.84. The QC file
reports the measured value. Playbook step 9 and the Forge persona now say "loudness measured", not
"loudness at target".

### F2. The planned voice tool does not work on this machine — documented, not fixed

`say --data-format=LEF32@22050` fails with `Opening output file failed: fmt?`. Plain `say -o x.aiff`
works. More important: the macOS system voice has no licence for commercial YouTube use.

**Revision.** The renderer uses plain `say` and labels the voice as a prototype. The voice guide
already required a licensed voice; the runbook now states that `say` is for pipeline tests only and
must be replaced before the first upload. This is a hard gate, not a note.

### F3. Burned-in captions were the wrong instruction — fixed

The playbook told Forge to burn the captions and also write an SRT. The test showed two problems.
The master already carries one line of scene text, so burned-in captions duplicate it. And a burned
caption cannot be translated, corrected, or turned off, which matters for a channel that will dub
later.

**Revision.** Playbook step 9 and the Forge persona now say: write the SRT, upload it with the
video, do not burn it in. The uploader now calls `captions.insert`. That call needs the
`youtube.force-ssl` scope and costs 400 quota units, so the scope list and the quota table changed.
Google has an open report that it rejects some SRT files, so the uploader treats a failure as a
warning and the runbook says to add the track in Studio.

### F4. The renderer wrote its output to the wrong folder — fixed

The first version resolved the channel root as `spec_path.parent.parent`, so a spec in
`scripts/EP-000/` put `renders/` and `assets/` inside `scripts/`. The design assumes one root above
both.

**Revision.** Three parents. This is the kind of fault that only a real run finds. It also proved
the value of the folder contract: the wrong path was obvious at once because it broke the contract.

### F5. QC reported an empty measurement — fixed

`ffprobe` was told to read stream `a:0` when it wanted video fields, so `qc.md` showed
`Resolution: 1920x` with a blank height. A QC file that shows a blank value can pass a careless
review.

**Revision.** The probe names the stream explicitly. A blank value is now impossible for width,
height, and duration.

### F6. Two faults in the team seeder — fixed

`avatarHue: 200` is rejected: the contract allows only ten fixed hues. And agent ids were keyed by
display name while routines looked them up by slug, so the first run created zero routines.

**Revision.** Forge uses hue 215, and ids are keyed by slug. Both are written up in the runbook so a
person who rebuilds the team does not repeat them.

### F7. A weekly routine cannot hold a YouTube token forever — changed the operating model

Google issues a refresh token that expires after **7 days** when the OAuth consent screen is in
Testing with an external user type. A weekly upload routine would fail with `invalid_grant` every
week. This is not in the playbook, and it is the single most likely reason the automation stops
working after the first month.

**Revision.** The runbook gives two options: re-authorise weekly (fine while evaluating), or publish
the OAuth app while it is still unverified, which keeps the token and shows a warning screen. The
100-user cap is irrelevant for one channel. The playbook no longer implies that one authorisation is
permanent.

### F8. Quota is real but not tight — quantified

One episode costs about 2,050 units: 1,600 for `videos.insert`, 400 for `captions.insert`, 50 for
`thumbnails.set`. The default daily quota is 10,000 units, so four episodes a day is the ceiling.
Two a week uses 20% of one day's quota in a week.

**Revision.** The quota table and the runbook state the per-episode cost, so a backfill of ten
episodes is planned, not discovered.

### F9. An installed persona does not follow a source edit — fixed, and now checked

`agents.md` is the source, and each `AGENTS.md` is a copy with the channel path substituted. When
the caption rule changed, the installed Forge persona kept the old instruction: "burn the captions".
An agent would have done the thing that had just been corrected.

**Revision.** Re-ran the seeder; all six now match. The runbook says to re-seed after any persona
edit, and the check is now mechanical: replace the placeholder, compare the text, count the matches.

### F10. The first art kit failed its own layout checks — fixed by looking at the output

Three faults, none of which a script could see. The review sheet cut off Pip's head. The character
feet sat behind the caption pill, so the cast looked as if it stood in the text. And the first caption
bar was a dark band with no margin from the bottom edge.

**Revision.** Feet moved above the pill, the pill moved inside the title-safe area (y 868 to 964, limit
972), and the sheet layout was recomputed. The lesson for Forge: open the PNG, do not trust the exit
code. The Forge persona now says to read the kit sheets before writing a scene.

### F11. The playbook's "no slideshow" rule cannot be met with still frames — partly fixed

README section 6 says no episode is a slideshow. A still image per scene is a slideshow, however good
the art is.

**Revision.** The renderer now alternates the speaker's mouth between open and closed every 0.2 s while
the voice line plays, and closes it after. Measured on EP-000: open versus closed differ by about
156,000 grey levels in the mouth crop, the same state differs by about 3,700, and the tail frame after
speech matches a closed mouth.

**Still open.** Only the mouth moves. Characters do not walk, and there is no camera move. A person
must judge in one viewing whether that holds a 3-minute episode for a 3-year-old. If it does not, the
next step is a two-pose body bob and a slow push-in, not a different tool.

### F12. Three personas cited a document the agents cannot read — fixed

Scout's report quoted its own instruction: "README section 6", and noted that no such README exists in
the channel folder. The same was true for Atlas, and for Verity's compliance gate ("README section 7").
An agent reads only its own folder and the channel root. Verity would have been told to audit against a
file it could not open, and a fail-closed rule cannot work like that.

**Revision.** The gate and the scorecard are now copied into `compliance/gate.md` and
`analytics/scorecard.md` by `tools/sync_playbook.py`, and the three personas point there.

### F13. The numbers disagreed between documents — fixed

Script length was 150–210 s in a persona, 180–270 s in the runbook, and 3:00–4:30 in the brand bible.
Song length was 90–120 s in a persona and 60–90 s in the bible. Bard reported each conflict and picked a
value that satisfied them all, which shows the persona was usable, but the sources should not need that.

**Revision.** One value each: script 180–270 s, song 60–90 s, in the playbook, the personas, and the
runbook.

### F14. The renderer ignored the shot list's timing — fixed

The shot list gave every scene 5 s, 195 s in total. The renderer measured the voice line and added 0.6 s,
so the episode came out at 107 s: shorter than the 180 s minimum. Nothing flagged it.

**Revision.** A scene may carry `seconds`, and the renderer takes the longer of that and voice plus 0.6 s.
`check_shotlist.py` writes it. EP-001 now renders at 195.08 s.

### F15. The first checker did not scan the voice lines — fixed

A planted "Hurry up, come on" in a voice line passed. The checker read `lyrics.md`, `script.md` and
`learning.md`, but the render uses the shot list's voice line, which can differ from them. Found only by
feeding the checker a bad input on purpose.

**Revision.** The checker scans the shot list's voice lines and captions too. Rule for the team: a check
that has never failed is not yet a check.

### F16. EP-001 cannot teach the skill it is about — open, with a stop rule

The render shows the same bathroom in more than 30 of 39 scenes. Pip points at a sink from two steps
away. There is no water, no soap, no foam, and no hands being rubbed. "Rub it slow" is a boy with his
arms out. Bard listed all of this under "Kit gaps", as I had told it to continue and record the gap.
Every mechanical check passed, so this is a failure that only a viewer sees.

**Revision.** Two stop rules: Bard stops when the kit cannot draw the learning-goal action, and Atlas
does not release a script to render while that action sits under "Kit gaps". The playbook's own bar was
"a problem, an attempt, a resolution, not a slideshow". EP-001 meets none of the three visually.

**Still open.** The kit needs props (soap, foam, a water stream, a towel) and a close-up of two hands
rubbing, with a two-frame rub loop. That is the next build, and no episode about hand washing ships
before it.

### F17. A songs channel has no way to make a song — open, needs a decision

The playbook assigns Forge "a music bed", but no tool in the pipeline produces one. `say` reads the lyrics
as speech. EP-001's 90 s "song" is a narrator reciting rhyming lines over silence. This is the first time
the gap was visible, because the earlier test had no song.

**Not decided here.** The choices are a composer, a production-library track with a YouTube monetisation
clause, or a generated track under a plan whose terms allow monetisation and whose Content ID position is
known. Each has a licence risk that Verity must check. Pick one before episode 1.

### F18. The agent test used subagents, not OpenBot agents — a limit on T21 to T23

The personas ran through OpenCode subagents with the installed `AGENTS.md` as their instructions. That
tests what the persona says and how a model follows it. It does not test OpenBot's routine scheduler,
threads, or message delivery between agents.

### F19. The close-up stage drew its hands off-screen — fixed

Every hand pair was placed twice: once by the group transform and once by its joint. At the first contact
sheet the sink was empty. Found only by looking at the frames.

**Revision.** A pair is placed by its joint alone, and its scale is a joint property. The basin was also
shrunk, because at the first size it read as a bath.

### F20. Loudness undershoots on a short voice-over — fixed, with one shared step

`loudnorm` in one pass landed at -16.0 LUFS on the 36 s chapter, the same fault as F1. A gain step fixed it by
hand, and the two render paths still had no common step.

**Revision.** `tools/finalize_audio.py in.mp4 out.mp4` searches the gain by analysis only (a gain and a limiter,
measured with `ebur128`), then encodes once. Tested on three masters: -16.0 reached -14.3 LUFS in three
measurements; a file already in range was copied unchanged; the old artkit master moved from -14.4 to -14.2.
The first version re-encoded the audio on every step and compared floats badly: it re-processed a file that was
already at -14.3 and, in one intermediate step, reached +0.1 dBFS. Both were fixed before the tool was used.

### F21. "Dead seconds" measured area, not life — partly fixed

The first motion render had 10 of 36 seconds flagged as dead. Most were a small movement (a pair of rubbing
hands at a distance), which the pixel-share measure under-counts. The fix was a slow camera breath on both
stages, a small sway on idle hands, and two camera pushes on the quietest beats: 10 dead seconds became 2
(seconds 7 and 8, where the hands wait at the sink). Read the number with that limit in mind.

### F22. Two visual defects — fixed, and a third found on the way

Poppy's rubbing was hard to see, and the goo drips read as small sticks.

**Revision.**
- The goo is now a lumpy blob with a highlight and a tapered drop that stretches.
- The drip lost its counter-rotation: with hands raised, the drop pointed up and read as a thumb. Each drip now
  cancels the rotation of arm, forearm, and hand, so it always hangs down (a third defect, found on the contact
  sheet, not by a metric).
- Poppy's hands moved from the hips to the chest, with opposite-phase forearms, a slight squash of the body, and
  two curved vibration marks on each side. The first marks were straight rays and read as whiskers.
- The camera is closer for the rub.

**Measured.** Over 21.0 to 22.0 s, sampled every 0.1 s, the share of pixels that changed per step: 12.75% around
Poppy's hands, 4.40% on Pip (breath and blink), 2.64% on an empty wall (camera breath). The whole-video numbers
barely moved (mean change 4.48% to 4.51%, held frames 16.6% to 15.1%), as expected from a local fix.

**Still open.** Seconds 7 and 8 stay flagged as dead: the hands wait at the sink for the next line. A slow push
on that beat would fix it; it is not in this render.

### F23. The Higgsfield catalogue has a model the public docs do not list — noted

The API lists 91 models. "Genjutsu Light" (motion transfer and object swap, no restyle) is in the catalogue and
not in the public docs. Prices come from the estimate endpoint, which does not charge:

| Model | Price | Limits |
| --- | --- | --- |
| Seedance 2.5 image-to-video | $0.2056 / s at 480p, $0.4622 at 720p, $1.1372 at 1080p | 4 to 30 s per clip, optional last frame, audio can be off |
| Genjutsu Light, motion transfer | $0.05 / s at 480p, $0.10 at 720p, $0.20 at 1080p, on output plus input seconds | Source trimmed to 15 s |
| Genjutsu, restyle | $0.318 / s at 480p, $0.681 at 720p, $1.632 at 1080p, on input seconds | Source 4 to 30 s; styles are presets |

Inputs must be public URLs. `POST /files/generate-upload-url` hosts a file and returns one. Outputs are
deleted after 7 days.

### F24. An AI render of a whole episode does not fit the loss limit — decision

For a 195 s episode at 720p, before any retry: Seedance 2.5 about $90, Genjutsu Light about $39 (output plus
input seconds), at 480p about $40 and $20. The loss limit in README section 8 is USD 500 over six months, so
Seedance at 720p spends it in about 5 episodes and Genjutsu Light in about 13. The stage cartoon costs no
credit. Use the AI path for chosen shots, not for whole episodes, until a pilot shows it is worth the money.

### F25. The Higgsfield account has no credit — blocked

The key is valid and uploads work, but a submit returns `403 not_enough_credits`. The pilot costs about USD 3.91
at 720p (A about 1.60, B about 2.31). Higgsfield says added funds expire after one year, so add only what the
pilot and one retry need. `tools/higgsfield.py` is ready: `run <endpoint> <body.json> --out <file>`, with
`--dry-run` to upload and read the estimate without any charge.

### F26. The first storyboard of the whole episode — four findings, none seen before building

No storyboard existed before chapter 1 was built. The shot list was a text table for the old format, and the
rundown covered chapter 1 only. Drawing the rest with the real rig showed:

- **Length.** At voice pace for spoken scenes and 5 s for each of the 16 sung scenes, the episode is about 2:21,
  below the 3:00 floor in the brand bible.
- **Repetition.** Eight song scenes share one close-up (four hands, foam, basin). The call and answer should cut
  back to the two children.
- **Missing art.** The back of a hand with nails, a thumb pose, nail scrubbing, and a towel pat.
- **A moved location.** The hello scenes were planned in a hallway; the stage has one room.

A caption that said a dollop landed on each hand, while the panel showed one pair, was caught on the second look
at the sheet. The five decisions are listed in `storyboard/README.md`.

### F27. A prettier storyboard than the build would hide the gap — decision, option A

A sample sheet, drawn by an image model, looked richer than our renders: soft gradients, eye highlights, glossy
surfaces. It also changed the dress and the hair of Poppy between panels, gave Momo a different face in each, and
labelled redrawn panels as real frames. Matching it would have meant leaving the brand (flat, two dots and a
mouth). The choice was to take its layout and the room detail, and keep the flat style.

**Done.** The sheets use the sample layout (dark pill, divider, title, scene badge, status chips). The stage got
wainscot, baseboard, window sill, perspective floor tiles, a rug, cabinet doors, a plant, a pump label, finger
lines, a thumb, cuffs, a collar, and a dress hem. Chapter 1 was re-rendered and re-measured (T38, T39). A panel
that pointed Pip's arm at Poppy under a caption about Momo was caught on the sheet and redrawn.

**Not matched, on purpose.** Gradients and gloss. Adding them is a brand decision, not a polish step.

### F28. Chapters 0 and 4: what went wrong, what was avoided, and what is approximate

- **Speaker map.** `make_chapter_audio.py` read every line as the narrator, because the pattern still had the
  column bar in it. Found from its own printout; fixed.
- **Avoided by design, not by a failing test.** A joint set with `tl.set` is not drawn until a tween updates it
  (the F19 class of bug). The shared `put()` helper writes the joint and the DOM in one step. It has not been
  exercised on a case where it failed.
- **Approximate.** The three sink inserts of S38 are cut at fractions of the line length (20, 42, 60 and 80
  percent), not at the real word times. The voice is a placeholder; retime when the voice is licensed.
- **Livelier chapter 1 than the short ones.** Held frames are 10% in chapter 1 and 31% in chapters 0 and 4, and
  each short chapter has one dead second (chapter 0 second 6, between Poppy's hops and Momo's wave; chapter 4
  second 3, between the pat and the first insert). A slow push on those two beats would close it.

### F29. Two defects in the new hand art, seen on a closer crop — fixed

The thumb sparkle was drawn with a thin dark stroke inside the hand group, and the outline filter widened it into
black wedges. The nails of the right hands were hidden under the foam.

**Revision.** The sparkle is now a white stroke, so the filter gives it an ink border. The foam is reduced to 55%,
50%, and 25% across S25 to S27. Thumbnails had hidden both faults; a crop at full size showed them.

### F30. Subtitles: two limits of this toolchain, one decision

This ffmpeg has no `subtitles` or `drawtext` filter (no libass or freetype), so burned-in text is drawn as PNG
overlays from SVG. And its MP4 muxer marks the first subtitle track as default whatever `-disposition` says, in
four variants tried.

**Decision.** The MP4 master carries no subtitle track. `EP-001.en.srt` is the file to upload. An MKV carries a
soft track that is off by default, and a third copy has the text drawn in, for review only.

### F31. The music is built, not heard — a gate for a person

`tools/music_synth.py` writes an original piece with numpy only (ukulele, glockenspiel, bass, kick, clap; 96 BPM;
C major), with the tune and the chords written for this episode. The first spectrogram showed sharp
high-frequency ticks on every beat, so the attack noise and the clap were softened. That change was judged from
the picture, not by ear. A person must listen to the music, and to the voice, before anything is published (G4).
The "sung" lines are chanted, because the voice is a macOS prototype (Junior and Kathy for the children) and
cannot sing.

### F32. The episode is built and is 2:22.7 — still below the 3:00 floor

The decision on length (a verse on drying, or a lower floor for ages 2 to 3) is still open. The chapters and the
assembly do not depend on it.

## 4. What is still unproven

| Item | Why | How to close it |
| --- | --- | --- |
| Real upload, including the kids attestation and the caption track | No Google Cloud OAuth desktop client | Create the client, run `upload_youtube.mjs auth`, then upload `EP-000` as a private draft |
| A real voice | Needs a licensed voice, not `say` | Buy or record the voice; record one line; re-render |
| Art that shows the skill | Props and a close-up now exist in the stage cartoon, for chapter 1 only | Build chapters 2 to 4 of EP-001; fix F22; a person watches it once |
| AI shots from Higgsfield | Quality, identity across clips, and licence terms are unknown | Pilot of two 8 s clips, see F23 and F24 |
| A song | The music is synthesized and original, but unheard (F31); the voice is a chant, not singing | A person listens; choose a licensed voice; decide on a singing voice or a composer |
| Agent hand-offs inside OpenBot | The test used subagents (F18) | Run one episode with Atlas messaging the others in the dev profile |
| Scout's other 14 claims | Only two were spot-checked | Verity re-reads the card before the episode is approved |
| Retention, click-through, revenue | Needs published videos and 7 days of data | Follow the day-7 scorecard |
| YouTube Kids acceptance | Needs a live channel | Read the Studio notifications after the first upload |

F1, F4, F5, F6, F14 and F15 were implementation faults that a real run exposed. F3, F7, F12 and F13 were
design faults in the playbook, and all four are corrected. F16 and F17 are open design gaps: the pipeline
produces a video, but not yet one that a child could learn from or sing along with.

## 5. Verdict

The playbook is applicable for a two-person-per-episode, two-episodes-a-week preschool channel,
with four conditions. The first two are legal, the last two are about the product:

1. Replace the prototype voice before the first upload. This is a licence condition, not a polish
   item.
2. Decide the OAuth token question before the first month ends. Otherwise the upload routine stops
   silently.
3. Build the props and the hands close-up before any episode about an action. EP-001 proved that a
   passing shot list can still produce a video that shows nothing.
4. Choose a music source before episode 1. A songs channel without a song is not the channel described.

The automation boundary is right: everything up to a private draft is automatic, and publishing stays
with a person. That boundary is what makes the made-for-kids attestation defensible.
