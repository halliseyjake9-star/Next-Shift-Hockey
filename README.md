# Next Shift Hockey

Public site plus a private portal page per player. Static, no backend, no login system. Data lives in `data/players/*.json`; the build script turns it into HTML.

## Layout

```
index.html          public homepage
pricing.html        Foundation 149 / Performance 299 / Elite 599
apply.html          application form (set ACTION_URL before launch)
assets/site.css     shared styles
data/rubric.json    12 metrics, 5 categories, written anchors
data/players/*.json one file per player — source of truth
templates/portal.html
scripts/build.js    JSON -> p/<token>/index.html
p/                  generated output, committed
```

## Build

```bash
node scripts/build.js
```

Node 18+. No dependencies. Exits non-zero if anything is wrong, so nothing half-scored ever ships.

The build fails on:

- a metric that isn't in the rubric
- a score outside 1–5 that isn't `"NS"`
- a 1 or a 5 with no evidence note
- an `NS` with no note explaining what couldn't be seen
- **scorer drift** — two scorers in the same window more than 1 apart on any metric

Drift is the one that matters most. If two evaluators land 4 and 2 on the same player, the rubric anchor is ambiguous. Fix the anchor in `rubric.json`. Never average the two scores — that's how a delta stops meaning anything.

Incomplete records print as warnings and still build, so you can publish a baseline before every field is filled.

## Adding a player

1. Copy `data/players/nsh-0001.json` to a new `nsh-000N.json`.
2. Clear `portal_token` — the build mints one and writes it back.
3. Fill the profile. Leave `scoring_events` empty until you've actually scored them.
4. `node scripts/build.js`, commit, push.
5. Send the family `https://nextshifthockey.com/p/<token>/`.

The token is the access control. Unguessable, `noindex`, not linked from anywhere. It is not real security — good enough for evaluation scores, not for anything you'd regret leaking. Real auth is a later problem, once there are enough clients to justify it.

## Scoring events

Scores are an array, never a flat object. Every event is stamped with who scored it and when:

```json
{ "window": "2026-10", "scorer_id": "jh", "scored_on": "2026-10-08",
  "source": "film", "games_viewed": 3,
  "metrics": { "scanning": 2 },
  "evidence": { "scanning": "No shoulder check on 11 of 14 tracked receptions." } }
```

This is what lets you separate player improvement from evaluator drift later. It can't be retrofitted once there's data in the system.

## Deploy

1. `git init && git add -A && git commit -m "scaffold"`
2. Push to a new GitHub repo.
3. Settings → Pages → deploy from `main`, root.
4. Buy `nextshifthockey.com` (Cloudflare Registrar, at cost). Add a `CNAME` file containing `nextshifthockey.com`, point DNS at GitHub Pages, enable Enforce HTTPS.

## Before you take money

- Put the real prices in the service agreement — the compensation section is still blank.
- Set `ACTION_URL` in `apply.html` to a Formspree or Tally endpoint.
- Stripe payment links for the three tiers, linked from `pricing.html`.
- Written compliance clearance from any active NCAA coach before they advise a prospect for pay.
- Massachusetts counsel on the agreement, the minor consent, and the media release.

## Calibration before the first client

Both evaluators score the same three players off the same film, independently. Compare metric by metric. Anywhere you're more than one apart, rewrite the anchor until you aren't. Two hours. Do it before either of you scores anyone who is paying.
