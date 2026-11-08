# Reply after a scored hand

Quote numbers from the script JSON only. Adapt the language to the user.
Do not recompute totals.

```markdown
[One sentence: who was Hakem, bid, captured points, scoring_rule]

This hand: Team 1 {team1_delta:+d}, Team 2 {team2_delta:+d}
Totals: Team 1 {team1_score}, Team 2 {team2_score}
```

If `game_finished` is true, add: Team {winner} wins (reached 1165).
If `thousand_point_rule_applied` is true, mention that the defenders' captured
points were withheld.
If `contract` is `shelem` or `sar-shelem`, say Shelem / sar-shelem, not "165".
If the hand was a normal-bid sweep, keep the numeric bid; do not call it a
Shelem contract.

On error: ask for `field` from the JSON. Do not propose a guessed value.
