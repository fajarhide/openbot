# Cost to make the channel

Computed on 3 November 2026 for Pip and Poppy. Every number has its source or is marked as an assumption.

## The short answer

- **One-time capital: USD 0.** Every tool is free or already owned: ffmpeg, Node, Python, Chrome for the render,
  the stage and character rig, the music synthesizer, and the YouTube upload API (free quota).
- **Running cost: about USD 6 a month.** That is the voice licence (ElevenLabs Starter), the only paid item the
  pipeline needs. Art and music are made by code and cost nothing.
- **The real cost is time and audience, not cash.** A month of eight episodes asks about 8 hours of your attention,
  and YouTube pays nothing until the channel reaches the Partner Programme threshold.

## What an episode costs

| Item | Per episode | Source |
| --- | --- | --- |
| Voice (ElevenLabs, 1 credit per character) | 1,002 characters; 3 takes use 3,006 credits, 10% of one Starter month | Measured from `shotlist.md`; plan from elevenlabs.io/pricing, read today |
| Music | USD 0 | `tools/music_synth.py`, original composition, no samples |
| Art and animation | USD 0 | `motion/` stage and rig |
| Render | about 39 minutes of machine time for 143 s of video, no cash | Observed: about 10 minutes per 36.5 s chapter on this laptop. Extrapolated |
| Upload | USD 0 | YouTube Data API, about 2,050 of 10,000 daily units |
| Your time | about 1 hour: read the script (G3), watch the master (G4), publish in Studio (G6) | Estimate, not measured |
| Agent (LLM) usage | not measured | Depends on your provider plan |

## Monthly cost at 8 episodes

| Setup | Per month | 6 months | 12 months | Views a month to break even at RPM 0.5 / 1.0 / 1.5 |
| --- | --- | --- | --- | --- |
| Voice only, ElevenLabs Starter | $6 | $36 | $72 | 12,000 / 6,000 / 4,000 |
| Voice, Creator (more credits) | $22 | $132 | $264 | 44,000 / 22,000 / 14,667 |
| Starter + 2 Genjutsu Light shots per episode | $32 | $190 | $379 | 63,200 / 31,600 / 21,067 |
| Starter + 2 Seedance 2.5 shots per episode | $43 | $258 | $516 | 85,952 / 42,976 / 28,651 |

The first row needs the Starter plan (USD 6 a month, commercial licence included). A Starter first month is
advertised at USD 1. You already have an ElevenLabs key; I did not read it, so I cannot tell your plan. If it is
Starter or higher, the voice line is already paid.

Optional AI shots use the Higgsfield prices in VALIDATION F23: Genjutsu Light is billed on output plus source
seconds (USD 0.10 a second at 720p), Seedance 2.5 on generated seconds (USD 0.4622 a second at 720p). Two shots an
episode is an upper estimate of a choice, not a need. The account has no credit today.

## What the money buys, and what it does not

The RPM column is the README assumption (USD 0.50 to 1.50) and is **not verified for made-for-kids content**, which
earns from contextual ads only.

Ads pay nothing until the channel is accepted into the Partner Programme: 1,000 subscribers and 4,000 public
watch hours (8,000 from 1 February 2027). At the 55% average view duration that the scorecard targets, one
143 s episode is watched for about 78 s per view, so:

| Threshold | Views needed |
| --- | --- |
| 4,000 hours (to 31 January 2027) | about 183,500 |
| 8,000 hours (from 1 February 2027) | about 367,001 |

At a running cost of USD 6 a month, the cash at risk over the 6 to 12 months it may take to get there is USD 36 to
72. The loss limit in README section 8 (USD 500 over six months) is not the constraint for the base setup. It
becomes the constraint only with AI shots on every episode.

## Not counted

Your hourly rate, a second device or a better microphone, a trademark or legal review, music distribution (the
README assumed about USD 20 a year, not re-checked), taxes, and any real voice actor. A licensed human voice would
replace the voice line and is the largest swing: price it before choosing it.
