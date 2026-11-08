# Shelem scoring rules

Read this file when `scripts/calculate_score.py` returns an error, when the
user asks why a score is what it is, or when a 165 / Shelem / 1000-point /
winner question is in play.

`scripts/calculate_score.py` is the scoring authority. Do not score a hand
from this file. If this file and the script disagree, the script and
`python scripts/calculate_score.py --self-check` win.

The script scores one hand. It does not remember earlier hands or invent a
winner the rules do not define. All arithmetic is integer arithmetic.

## Constants

| Name | Value |
| --- | ---: |
| Points in one hand | 165 |
| Legal bids | 100, 105, …, 165 |
| Max defending points (`D`) | 160 |
| Hakem buried minimum | 5 |
| Winning threshold | 1165 (not a cap) |
| 1000-point rule | defending team at 1000+ scores 0 if the bid is made |
| Heavy failure | defending points ≥ 85 |

Negative cumulative scores are allowed. A zero award does not reset a total.

## Declarer award

`B` is the bid. `D` is defending points. `C = 165 - D`.

| `scoring_rule` | Contract | Condition | Hakem |
| --- | --- | --- | ---: |
| `sweep_double` | normal | `D = 0` | `+2 × B` |
| `bid_made` | normal | `D > 0` and `C >= B` | `+B` |
| `bid_failed` | normal | `C < B` and `D < 85` | `−B` |
| `heavy_failure` | normal | `C < B` and `D >= 85` | `−2 × B` |
| `shelem_made` | shelem or sar-shelem | `D = 0` | +330 |
| `shelem_failed` | shelem or sar-shelem | `D` from 5 through 160 | −330 |

Success is only `sweep_double`, `bid_made`, and `shelem_made`. Mok
(`C = B`) is `bid_made`, not a failure.

Compare captured points with the bid. Do **not** use the old 5–40 / 45–80
band table: a bid of 135 with `D = 35` (`C = 130`) is a failure (−135).
Mok at 120 has `D = 45` and is a made bid.

## Contract

- Bid below 165: omit `contract` (it becomes `normal`). `shelem` /
  `sar-shelem` on any other bid is rejected.
- Bid 165 **is** Shelem. There is no normal 165. Omit `contract` or set
  `shelem`; both score +330 on `D = 0` and −330 on any captured defense.
  `contract: "normal"` on a bid of 165 is rejected.
- `sar-shelem` uses the Shelem table and is set only when the user said
  so. The engine does not check that a Shelem bid came first.

## Resolving `D`

1. `outcome: "mok"` → `D = 165 - bid`
2. `outcome: "sweep"` → `D = 0`
3. Else `defending_points`, or `165 - declarer_points`
4. If more than one of these is present they must agree, and the two point
   fields must sum to 165
5. `D` must be a multiple of 5 from 0 through 160. The Hakem buries at
   least 5 points before play, so the defenders cannot capture 165 and
   the Hakem cannot capture 0. `D = 165` (or `declarer_points = 0`) is
   rejected.

## 1000-point rule

Uses scores from **before** the hand, separately per team.

- A Hakem at 1000+ is scored by the ordinary Hakem rules.
- A defender at 1000+ scores 0 if the bid is made, whatever they captured.
  If the bid fails they keep the points they captured.
- `thousand_point_rule_applied` is true only when this exception removes a
  nonzero award. A sweep already awards 0, so the flag stays false.

## Winner

After adding awards: if exactly one new total is 1165 or more, that team
wins. No tie-break exists. A hand that starts with a team already at 1165
is rejected.

## Hands the script must reproduce

| Hand | Hakem | Defender |
| --- | ---: | ---: |
| Normal bid 140, D = 25 | +140 | +25 |
| Normal bid 135, D = 35 | −135 | +35 |
| Normal bid 140, D = 10 | +140 | +10 |
| Normal bid 120, D = 90 | −240 | +90 |
| Mok at 140 | +140 | +25 |
| Sweep at bid 130 | +260 | 0 |
| Shelem, D = 0 | +330 | 0 |
| Shelem, D = 5 through 160 | −330 | +D |
| Bid 165 with no contract, same D | same as Shelem | same as Shelem |
| sar-shelem, same D | same as Shelem | same as Shelem |
| Defenders on 1100, opposing bid 140, D = 10 | Hakem +140 | 0 |
| Defenders on 1100, opposing bid 140, D = 35 | Hakem −140 | +35 |
| Hakem on 1100, bid 140, D = 25 | +140 | +25 |
| Both on 1000, team 1 Mok at 120 | +120 → 1120 | +0 → 1000, no winner |

The last row is the spoken-"20" regression: expand to 120 first. Team 2
captured 45 and was already on 1000, so those 45 points are withheld.
