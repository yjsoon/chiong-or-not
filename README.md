# Chiong or not? 🏃‍♂️🚌

One page, one question: **you standing in Aperia Mall, bus 13 coming — should you run or can lepak?**

Live arrival data, your walking speed, and a straight answer. No timetable guessing.

## Why bus 13

From Aperia Mall to Yio Chu Kang, **13 is the only direct bus**. Checked every stop within
walking distance of the mall against every Yio Chu Kang stop, in route order — only one match.

| | |
|---|---|
| **Board** | Stop `07371` — "Aft Kallang Rd", Lavender St |
| **Alight** | Stop `55509` — Yio Chu Kang Int (= Yio Chu Kang MRT) |
| **Ride** | 30 stops, roughly 45 min |
| **Route** | Yio Chu Kang Int ⇄ Upp East Coast Ter |

⚠️ **Important hor.** Board at the stop **across the road**, not the one outside the mall.
The stop literally named "Aperia" (`07379`) goes the *opposite* way, into town. Don't anyhow whack.

## How it decides

The real question is not "when is the bus" — it is **"how much does missing it cost"**.
So the answer is headway-aware: if the next bus is 3 min behind, no point sprinting.

With `eta` = seconds until arrival, `walk`/`run` = your configured times, `BUFFER` = 60s cushion:

| Situation | Verdict |
|---|---|
| `eta ≥ walk + 60` | **STEADY LAH** |
| `run + 60 ≤ eta < walk + 60` | **CHIONG AH!!!** |
| ↳ but next bus < 5 min behind | **RELAK LA BROTHER** |
| `eta < run + 60` | can't make it → decide again on the next bus |
| Bus at stop, and you're basically there | **IT IS THERE NOW** |
| No GPS on that bus | **AGAK AGAK ONLY** — never tells you to sprint |

That last row matters. Some entries in the feed are timetable estimates with no bus actually
being tracked (`monitored: 0`). Sprinting for a guess is a mug's game, so the app won't suggest it.

## Walk time

No GPS — you're indoors, where position error is 50–100 m, which is ±1.2 min of noise on a
decision decided by about 1 min. So: two sliders and three one-tap presets, saved to your browser.

- **Already outside, steady** — 2 min walk / 1.5 min run
- **Inside mall lepak-ing** — 4 / 2.5 (default)
- **Basement carpark, blur** — 6 / 4

Defaults cover ~165 m plus the Lavender St pedestrian crossing, which is the biggest wildcard
in the whole trip.

## Data

[`arrivelah`](https://github.com/cheeaun/arrivelah) by cheeaun — a free public proxy over
LTA DataMall. **No API key needed.** Polled every 15 s (its cache TTL, so faster is pointless).
The countdown ticks locally between polls, so it stays smooth and doesn't care whether your
phone clock agrees with the server.

Free service, so don't hammer it. Say thank you. 🙏

## Running it

It's one static HTML file. No build, no npm, no nothing.

```
start index.html
```

## Testing the verdicts

Waiting around for every situation to happen naturally = damn sian. So there's a mock hook:

```
index.html?mock=steady     # plenty of time
index.html?mock=chiong     # RUN
index.html?mock=relak      # tight, but next bus close behind
index.html?mock=gg         # missed it, nothing after
index.html?mock=there      # bus at the stop right now
index.html?mock=agak       # timetable guess, no GPS
index.html?mock=nobus      # no 13 running
index.html?mock=offline    # API unreachable
```

Fine-grained overrides also can, on live or mock data:

```
index.html?eta=180&eta2=900&monitored=0
```

## Deploy

```
gh repo create chiong-or-not --public --source=. --push
gh api -X POST repos/<you>/chiong-or-not/pages -f 'source[branch]=main' -f 'source[path]=/'
```

Needs to be on https for a phone to be happy about it. No secrets in here — the API takes no key.
