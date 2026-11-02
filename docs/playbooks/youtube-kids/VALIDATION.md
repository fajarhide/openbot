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

## 4. What is still unproven

| Item | Why | How to close it |
| --- | --- | --- |
| Real upload, including the kids attestation and the caption track | No Google Cloud OAuth desktop client | Create the client, run `upload_youtube.mjs auth`, then upload `EP-000` as a private draft |
| A real voice | Needs a licensed voice, not `say` | Buy or record the voice; record one line; re-render |
| Real art | The renderer draws placeholder shapes | Replace the scene builder before EP-001 |
| Retention, click-through, revenue | Needs published videos and 7 days of data | Follow the day-7 scorecard |
| YouTube Kids acceptance | Needs a live channel | Read the Studio notifications after the first upload |

Nothing in section 3 is a design failure. F1, F4, F5, and F6 were implementation faults that the
first real run exposed. F3 and F7 were design faults in the playbook, and both are now corrected.

## 5. Verdict

The playbook is applicable for a two-person-per-episode, two-episodes-a-week preschool channel,
with two conditions:

1. Replace the prototype voice before the first upload. This is a licence condition, not a polish
   item.
2. Decide the OAuth token question before the first month ends. Otherwise the upload routine stops
   silently.

The automation boundary is right: everything up to a private draft is automatic, and publishing stays
with a person. That boundary is what makes the made-for-kids attestation defensible.
