---
name: ig-audit
description: >-
  Post-mortem on what the user has already posted - which reels actually
  worked, why, and what to stop making. Use when the user pastes their
  Instagram insights or past posts, or has Metricool connected, and asks
  "what's working", "why did this flop", "read my analytics", "audit my
  content", or wants to know what to do more of.
---

# ig-audit

The only honest source of what works for an account is that account. Every
rule in every Instagram guide, including the ones in this pack, is a prior.
The user's own last 30 posts are the evidence.

## Input

Ask for whichever the user has:

- Insights per post: views, reach, interactions, watch time, saves, shares,
  follows, and the non-follower share of reach. Screenshots are fine.
- Or the retention graph for their best and worst recent reels. This one
  screenshot is worth more than the rest combined.
- Or just the posts and their view counts, which is enough for a first pass.

Also read `~/.claude/instagram/log.md` if it exists, since it records which
hook formula each post used.

## If Metricool is connected, pull it instead of asking

Check the tool list for the Metricool connector: tools ending in
`getBrandSettings` and `getAnalyticsDataByMetrics`. They may be listed as
deferred, in which case load them through tool search first. If they are not
there, use the paste flow above and mention once that connecting Metricool
skips it.

1. **`getBrandSettings`** for the brand id and timezone. If it says no network
   is connected, or the brand has no Instagram, relay the connection link it
   returns and fall back to the paste flow. Do not retry.
2. **`getAnalyticsAvailableMetrics`** with `network: instagram` and
   `connector: reels`, then `posts`. Field ids get deprecated ("Do not use this
   field"), so check the list rather than trusting the ids below blind.
3. **`getAnalyticsDataByMetrics`**, last 90 days, one call per connector:

| need | reels | posts |
| --- | --- | --- |
| date, text, url | `IGRE02` `IGRE03` `IGRE06` | `IGPO02` `IGPO03` `IGPO06` `IGPO07` (type) |
| views | `IGRE23` | `IGPO28` |
| reach | `IGRE11` | `IGPO14` |
| sends | `IGRE21` shares | `IGPO27` shares |
| saves | `IGRE12` | `IGPO15` |
| hold at 3s | `IGRE28` reel view rate | - |
| watch time | `IGRE24` average, `IGRE27` retention | - |
| follows | - | `IGPO29` |

Say plainly what this does **not** give you: follows per reel and the
non-follower share of reach. Ask for those two from the app's own insights
screen for the top five and bottom five only, or mark them `n/a`. Never fill
the gap with an estimate.

`IGRE28` is Metricool's three-second view rate, not the retention graph. Label
it as that in the output. It is close enough to grade the hook and not close
enough to read the middle of the video.

This is read only. The audit never schedules, edits or deletes anything in
Metricool.

## What to actually measure

Raw views is the least useful number on the page, because it is mostly a
function of how many people already follow the account. Compute these instead
and show the working:

| metric | how | what it tells you |
| --- | --- | --- |
| **Outlier multiple** | views / the account's own median views | whether this was a real hit or a normal day |
| **Non-follower reach** | % of reach from people who do not follow | whether it travelled at all |
| **Hold at 3s** | viewers still there at 3s / viewers who started | whether the hook worked. This is the hook's grade. |
| **Average watch time** | straight from insights | whether the middle worked |
| **Sends per reach** | shares / reach | the strongest single signal you can earn. A send is a person putting their name on it. |
| **Follows per reach** | follows / reach | whether the profile converted the attention |

Rank by outlier multiple and sends per reach, not views. A reel with 4,000
views and 90 sends beat the one with 60,000 views and 11.

## Then find the pattern

With the top five and bottom five side by side, look for what separates them,
and be willing to conclude something the user will not like:

- **Hold at 3 seconds.** If the top and bottom differ here, it is the hook and
  nothing else, and everything downstream is a distraction.
- **Hook formula.** Which ids from `ig-reel/hooks.json` are in the top five?
- **Format.** Reel, carousel, single image.
- **Length.** Group into under 15s, 15 to 30s, 30 to 60s, over 60s.
- **Theme.**
- **Whether the user replied to comments in the first hour.**
- **Day and time.** Check this **last** and only if the others show nothing.
  It is almost never the cause and it is where people want it to be.

State the finding as a claim with the evidence attached, and say how confident
it is. With 30 posts you can see a pattern. With 6 you cannot, and saying so is
better than inventing one.

## The distinction that saves people months

**A reel that gets views and no follows is not a failed reel, it is a profile
problem.** A reel that gets no views is a hook problem. Separate the two before
recommending anything. If non-follower reach is high and follows per reach is
low, stop rewriting hooks and go to `/ig-profile`.

## Output

```
AUDIT  ·  31 posts  ·  Jun 12 - Sep 5  ·  median views 4,100

TOP 5 BY OUTLIER MULTIPLE
  18.2x  #3  Nobody Tells You   74,600 views  62% non-follower  hold@3s 71%  128 sends
   6.4x  #1  Cost Confession    26,300 views  48% non-follower  hold@3s 64%   71 sends
  ...

BOTTOM 5
   0.3x  #11 Numbered            1,200 views  9% non-follower   hold@3s 31%    2 sends
  ...

WHAT THE DATA SAYS
1. Hold at 3 seconds is the whole story. Top five average 66%, bottom five 33%.
   Everything else you are worried about is downstream of the first two seconds.
2. The posts where you were the one who looked bad: mean 8.1x vs 0.9x for
   everything else. n=5. Strongest signal here and it is not close.
3. Tool listicles get views and nothing else. High reach, no sends, no follows.
   Three of your bottom five.
4. Day of week shows nothing. Your Tuesday and Friday means are inside the
   noise. Stop optimising it.

STOP: listicles.
DO MORE: the ones with a cost you paid and a number attached.
```

Then hand the conclusions to `/ig-plan` so next week is built on the user's own
evidence rather than on defaults, and to `/ig-viral` so the swipe file gets
filtered to the formulas that work for this account specifically.
