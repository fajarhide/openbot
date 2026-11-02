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

### F20. Loudness undershoots again on a short voice-over — fixed, and the fix is not shared yet

`loudnorm` in one pass landed at -16.0 LUFS on the 36 s chapter. This is the same fault as F1. The fix this
time was a gain step and a limiter on the final mux (-14.3 LUFS). The motion path has no shared loudness step
with `render_episode.py`, so the same fault will come back.

### F21. "Dead seconds" measured area, not life — partly fixed

The first motion render had 10 of 36 seconds flagged as dead. Most were a small movement (a pair of rubbing
hands at a distance), which the pixel-share measure under-counts. The fix was a slow camera breath on both
stages, a small sway on idle hands, and two camera pushes on the quietest beats: 10 dead seconds became 2
(seconds 7 and 8, where the hands wait at the sink). Read the number with that limit in mind.

### F22. Two visual defects remain — open

Poppy's rubbing is hard to see at that zoom. The goo drips are drawn as short yellow strokes and read as small
sticks held in the hands. Neither is caught by a metric.

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

## 4. What is still unproven

| Item | Why | How to close it |
| --- | --- | --- |
| Real upload, including the kids attestation and the caption track | No Google Cloud OAuth desktop client | Create the client, run `upload_youtube.mjs auth`, then upload `EP-000` as a private draft |
| A real voice | Needs a licensed voice, not `say` | Buy or record the voice; record one line; re-render |
| Art that shows the skill | Props and a close-up now exist in the stage cartoon, for chapter 1 only | Build chapters 2 to 4 of EP-001; fix F22; a person watches it once |
| AI shots from Higgsfield | Quality, identity across clips, and licence terms are unknown | Pilot of two 8 s clips, see F23 and F24 |
| A song | No music source (F17) | Choose composer, library, or licensed generator; record the licence |
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
