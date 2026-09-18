# Thumb Hold

An exploration spun out of Snoozie Buddy (18 Sep 2026). Not a product, not linked to the
Flutter app or the Banyan web study. One static file, no build. The only network call is
anonymous study analytics (see Instrumentation); `?track=off` removes it.

## The idea

If both thumbs have to stay on the screen, the phone can't be used for anything else.
Could that be a way to break the scroll-dopamine loop, instead of yet another thing that
competes for attention inside it?

## The honest tension

Snoozie's stated goal is *phone down, screen away*. A hold mechanic is the opposite posture:
phone in hand, screen lit. And it restrains rather than changes what someone wants — every
other app is still one swipe away the moment they let go.

So the working hypothesis is narrower than "keep them engaged":

> A **short** hold (60–90s), paced to slow breathing, with the screen dimming as you stay,
> works as a *bridge* — it ends the scrolling session and makes putting the phone down the
> natural next move, rather than something you have to decide to do.

Closest precedent: *Pause* by ustwo (finger follows a slow blob; lifting breaks it).
*Forest* / *Hold* use the same "can't do anything else" logic but via leaving the phone alone.

## What the prototype does

`index.html` — open on a phone.

- The start screen asks nothing: title, one line, **Begin**. Length, lift rule, thumbs and visual
  sit behind an "Options" link with defaults of 90s / Wait / two thumbs / Living light.
- Two thumb pads (or one — accessibility and one-handed use). Both must be down to count.
- While held: orb breathes (4s in / 6s out), ring fills, screen darkens up to ~78%.
- Lifting: light returns immediately, 2s grace, then one of three rules —
  **Wait** (progress pauses), **Slip back** (decays at 2x), **Start over** (resets).
- Two visuals. **Living light** (default): a canvas blob whose outline never quite repeats, light
  streaming from each held thumb into it (lift one and that side drains), thumb movement stirs it,
  colour drifts amber → rose → dusk blue and motion slows as you stay, and the "Breathe in/out"
  words fade after two breaths. **Orb**: the original plain circle, kept as the comparison.
- Music (generated with Web Audio, no files; `?sound=off` or Options to silence; logged per
  session). Two movements at once: each breath rises and falls — soft bell notes climb through
  the inhale and descend through the exhale over a pad that swells and settles — and the whole
  hold drifts toward sleep: notes get lower, fewer and quieter, the pad's upper voices drop away,
  the tone darkens, until only a low hum is left, which fades out over ~5s at the end. Stops within a
  fraction of a second when any thumb lifts, and eases back in when all are down again. iPhones mute it when the ring/silent switch is on silent. Spoken guidance via the
  device's built-in voice was tried and removed: too robotic.
- The breath keeps its rhythm through a brief slip (under the 2s grace) and restarts on a fresh
  inhale after a longer break, so the cue never flips mid-breath.
- No numeric timer by design — the ring is the only progress cue, so the hold isn't clock-watched.
- `?together=1` — the cheapest probe of a "partners" version: two people, one phone, one thumb
  each on the existing pads. Copy and ending change ("Now leave the phone here. Look up."), and
  sessions are logged as `mode: together` so they never mix into the solo numbers. A real group
  mode (more fingers on one phone, then linked phones) waits until the solo question is answered.
- Completion: short vibration, black screen, "Now put the phone down", text fades after 6s.
- Leaving the tab mid-session is logged as abandonment — that's the behaviour under study.
- On the next visit (within 24h) it asks what you did after the last hold — put the phone down,
  went back to scrolling, something else — and records how long you were gone.
- **The sky** (the only game layer). Answering the check-in adds one mark for that night: a star
  if the phone went down, a cloud otherwise — any honest answer counts, so lying gains nothing.
  One mark per night (a night runs to 4am); a later "put it down" upgrades a cloud, nothing ever
  downgrades or is lost. The first 7 stars draw the Ladle, the next 23 the Banyan, both announced
  up front. The sky shows for ~4s after the check-in only — never after a hold, never on demand,
  no streaks, counts or notifications. `?sky=off` is the comparison arm (logged per session);
  `?sky=peek` shows it on load for testing. If put-down rates match with it off, delete it.
- "Past sessions" shows a local log (last 50): goal, held time, lifts, wall-clock time, visual,
  finished or left, the after-answer and time away. Stored in `localStorage` only.

URL params preselect options: `?t=60&lift=decay&thumbs=1&viz=orb`. A non-listed `t` (e.g. `?t=8`)
overrides the picker for quick testing. On desktop, holding **Space** counts as holding.

### Instrumentation

The local log only lives on each tester's phone, so the hosted version also sends anonymous events
to PostHog (EU cloud). Events only: autocapture and session replay are off, no cookies (the
anonymous device id sits in `localStorage`), and the start screen says usage is recorded.
`?track=off` disables it; with the placeholder key left in `PH_KEY` nothing loads at all.

| Event | When | Properties |
| --- | --- | --- |
| `$pageview` | app opened | (PostHog defaults) |
| `hold_started` | Begin tapped | duration, lift, thumbs, viz, sound, hour |
| `lift` | thumbs come off mid-hold | at (seconds of progress), n, duration, lift |
| `hold_ended` | finished, or left after touching | the same row as the local log: held, lifts, wall, completed, ... |
| `hold_never_touched` | left without ever holding | duration, wall |
| `after_answered` | next-visit check-in answered | answer, gap, plus the last hold's completed/duration/lift/viz/held |

`mode` (solo/together) and `sky` (on/off) ride along on every event. Opened → started → ended is
the funnel; break `hold_ended.completed` down by `lift` for question 2, chart `lift.at` for
question 1, and `after_answered.answer` is the first kill criterion.

### Run it

```
python3 -m http.server 4188
```

Then open `http://<your-laptop-ip>:4188` on a phone on the same wifi. Screen wake-lock needs
HTTPS or localhost, so over plain LAN the phone may auto-dim on long holds — deploy to any
static host for a proper test. Vibration doesn't exist on iOS Safari.

## Questions this should answer

1. **Does 90 seconds feel calming or like a chore?** Where does it tip — 60s? 3 min?
2. **Which lift rule feels right?** Suspicion: "Start over" is punishing, "Wait" has no stakes,
   "Slip back" is the interesting middle. Watch lifts-per-session across the three.
3. **What do people do in the 10 seconds after it ends?** This is the whole point and the
   prototype can't see it. Needs asking, or observing.
4. **Two thumbs vs one** — does two-handed grip matter, or is it just awkward in bed?
5. **Does a notification banner break it?** (It will. How much does that matter?)

## What would kill the idea

- People complete the hold and go straight back to scrolling → it's a delay, not a bridge.
- Most sessions abandoned before 30s → the mechanic itself is the friction.
- It only feels good with stakes attached ("Start over") → then it's a game, and games are
  the thing we're trying to get people away from at bedtime.

## If it survives

Fold it into Banyan's evening exit as the "one meaningful act of care": hold Mani with both
thumbs while Nandan settles, release = goodnight. The daytime use (riding out a scroll urge)
is a different product — keep it separate.
