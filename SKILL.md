---
name: shelem-score-calculation-skill
description: >
  Scores Shelem (شلم) hands and running game totals. Use this skill when the
  user reports a Shelem bid, Mok (مک), sweep, shelem or sar-shelem/سرشلم
  contract, defending or declarer points, Hakem, or asks for the score,
  winner, or 1165 threshold — even if they never say "score a hand." Expand
  spoken bids (20→120, 40→140) then run scripts/calculate_score.py. Never
  compute a score yourself.
license: MIT
compatibility: Requires Python 3.9+
metadata:
  author: Shelem Scorekeeping contributors
  version: "1.0"
---

# Shelem score calculation

Score one completed hand at a time. The engine is stateless: keep the running
totals yourself and pass the scores from **before** each hand.

**Never compute a score, delta, total, or winner yourself.** Quote only what
the script (or an equivalent `calculate_shelem_score` tool) returned.

## Workflow

1. Keep game state: `team1_score` and `team2_score` (start at 0; may go negative).
2. When a hand is complete, normalize speech (below), then write a JSON object.
3. Score it:

   ```bash
   python scripts/calculate_score.py hand.json
   ```

   JSON on stdin is also valid: `python scripts/calculate_score.py < hand.json`.
4. If stdout is an `error`, ask the user for the missing or contradictory fact.
   Do not change numbers to make the call succeed.
5. Announce only the returned totals, deltas, `scoring_rule`, and `winner`.
6. Store the new totals as the next hand's opening scores.
7. If the user corrects an earlier hand, re-score **every hand from that point**.

If a `calculate_shelem_score` tool is already available, call it with the same
fields instead of the script. Do not mix a tool result with your own arithmetic.

Reply in the user's language. For the reply shape, see [assets/reply-template.md](assets/reply-template.md).

## Normalize before scoring

| Speech | Engine field |
| --- | --- |
| "20", "۳۰", "40", "50" as a bid | `bid` 120, 130, 140, 150 — not 20/30/40/50 |
| Mok / مک | `outcome: "mok"` (declarer captured exactly the bid; this is a **made** bid) |
| Sweep, or "shelem" **after a numeric bid** | `outcome: "sweep"` (`D = 0`), `contract` omitted / `normal` |
| "شلم خوندم" / "shelem" as the bid itself | `bid: 165` (`contract` omitted or `"shelem"`) |
| سرشلم / sar-shelem | `bid: 165`, `contract: "sar-shelem"` only if the user said so |
| Hakem / حاکم / "we bid" | `hakem_team` or `bidding_team`: `1` or `2` |

Legal bids are only 100, 105, …, 165. Bid 165 **is** Shelem; do not send
`contract: "normal"`. Send `sar-shelem` only when the user said so.

## JSON fields

Required: `team1_score`, `team2_score`, `bid`, and at least one of
`hakem_team` / `bidding_team`. Also required: `defending_points`,
`declarer_points`, or `outcome` (`mok` or `sweep`).

Optional: `contract`, `bid_successful` (cross-check only), the other team field
(must match if both are sent), the other point field (must sum to 165).

Example:

```json
{
  "team1_score": 0,
  "team2_score": 0,
  "bidding_team": 1,
  "bid": 140,
  "defending_points": 25
}
```

More examples: [assets/hand.example.json](assets/hand.example.json).

## Gotchas

- **"Shelem" has two meanings.** After a numeric bid it means a sweep (defenders
  took 0; award `2 × bid`). Alone, or as a bid of 165, it is the Shelem
  contract (+330 / −330). There is no normal 165. A failed Shelem is always
  −330, even when defenders captured 85 or more. Never send
  `contract: "shelem"` when `bid < 165`.
- **Mok is success.** Do not treat it as a failed bid.
- **Defenders cannot capture 165.** The Hakem buries at least 5 points before
  play, so `defending_points` is 0–160. If the user reports 165 for the
  defenders, ask; do not send it to the script.
- **The 5–40 / 45–80 band table is wrong.** Compare the Hakem's captured points
  with the bid. Sweep (`D = 0`) doubles a normal award; `D >= 85` on a failed
  normal bid doubles the loss.
- **1000-point rule** applies only to a **defending** team already at 1000+:
  if the bid is made, that team is awarded 0 (its cumulative score is not reset).
- **Do not invent a tie-break.** If the script errors because a team is already
  at 1165, stop.
- **Ask; do not guess.** Missing Hakem or missing points is a question for
  the user.

## When to read more

- Script error, or the user asks *why* a score is what it is → read
  [references/scoring-rules.md](references/scoring-rules.md).
- `python scripts/calculate_score.py --help` documents flags and exit codes.
- `python scripts/calculate_score.py --self-check` must print `"ok": true`
  before you trust a modified script.
