# Week 1 evidence

Keep this short. Bullet points, a small table and one diagram or code trace are enough. This is your checkpoint record, not an extra report. Replace the prompts as you work.

Name: Anton Stenström
Exact Unity version: 6000.3.26f1
Checkpoint commit / repository: http://github.com/DevAStenstrom/Salvage-Sprint (exact commit hash submitted in Omniway)
Current stage and anything unfinished: Stage 5. The Stage 2–4 rules are in place, all six scenarios run and all tests pass. Still to do: the walkthrough recording.

## Initial prediction

Three observations the rival needs, including one it might not have:
which scrap is nearest & if the rival is already holding scrap & if it can reach the scrap (this one the rival might not have until it tries)
One question about NPC behaviour:
how do you make an enemy NPC feel fair? for example in this game how do you make the speed justifiable and what is considered to hard?

## One complete trace

Scene: 05_Unreachable (F6), run 2026-11-27. Pickup 0 is on a platform with no route to it; pickup 1 is reachable. The quoted lines are from the Console (decision logging in `RivalBrain`). Candidate flags: V = visible, R = remembered, B = blocked, d = distance.

1. **Observation** (`PickupSensor.Capture`): `carrying=False candidates: 0(d=6,0 V--) 1(d=8,0 V--)`. Both pickups are inside the sensing radius (10) and visible.
2. **Facts** (`DecisionPolicy.Infer`): `NeedsDelivery=False UsableTarget=True Retained=False`. No cargo and no current target, so the nearest eligible candidate is selected: pickup 0.
3. **Decision** (`DecisionPolicy.Choose`): `[48,5] Collect target=0: Nearest eligible known pickup`.
4. **Action** (`RivalBrain.Update` → `Navigator.Request`): a route is requested to pickup 0 at (17.00, 0.40, 7.00).
5. **Feedback** (`Navigator.Poll` → `NavigationFeedback.Classify`): `[Navigator] Navigation feedback rejected the route; key=0; ... pathStatus=PathPartial`. Travel becomes Failed. `RivalBrain` logs `[48,6] FEEDBACK: Route failed for pickup 0; ignore for 6 seconds` and stores the block in `failedUntil`.
6. **Later decision**: the next observation is `0(d=6,0 VRB) 1(d=8,0 V--)`. Pickup 0 is now blocked, so `IsEligible` rejects it: `[48,6] Collect target=1: Nearest eligible known pickup`, with `travel before decision: Failed`.
7. After that: `[50,9] Deliver` (cargo held), `[53,3] Explore` (pickup 0 still blocked, `0(d=6,1 V-B)`), and at `[54,7]` the block has expired, so it tries `Collect target=0` again and fails again.

State transitions seen in the runs:

- Explore → Collect when an eligible pickup is selected (F4 at 12,9: pickup 0 comes into view).
- Collect → Deliver after collection gives cargo (F2 at 1,2).
- Deliver → Collect or Explore after delivery (F2 at 2,4 goes to Collect 11; F4 at 11,2 goes to Explore).
- Collect → another Collect target or Explore when the current target becomes unusable (F6 at 48,6 goes to Collect 1; F5 at 6,7 goes to Explore).

## Scenario record

All runs 2026-11-27 with the player left still. I ran scenario 1; scenarios 2–6 were run in my editor by Claude (see Collaboration and help).

| Scenario | Predicted result | Observed result and short evidence / remaining issue |
|---|---|---|
| Normal loop and restart (F2) | the ai tries to find the closest scrap and bring it home | First decision `Collect target=10`, the nearest of the visible pickups 7, 10 and 11. Then Deliver at 1,2, Collect 11 at 2,4, Deliver at 3,9. Rival reached 5/5 with 97 s left, the RIVAL WINS box showed, and restart reset score and timer. No warnings. |
| Carrying cargo (F3) | the ai decides to bring cargo home before looking at other pieces to collect | First decision was `Deliver`, `carrying=True`, although pickup 9 was visible at d=2,2. Delivered in about 2 s (11,6 to 13,7), then `Collect target=10`. Pickup 10 was beside its route to the depot and was not collected until after the delivery. |
| Hidden / remembered pickup (F4) | the ai does not pick the scrap up unless it has seen it or remembered it was there | First decision `Collect target=1` with candidates `0(d=5,0 ---) 1(d=8,0 V--)`: the nearer pickup 0 was hidden and unknown, so it was ignored. On the way the debug panel showed "Retain current eligible target". After delivering, pickup 0 was hidden again, so it chose Explore at 11,2, then saw it (`0(d=6,5 V--)`), chose `Collect target=0` at 12,9 and had the cargo by 15,1. Remaining issue: memory holds only the current target, so pickup 0, seen at 8,8 while delivering, was forgotten. |
| Target taken (F5) | if the target is taken the ai continues searching for other targets | `Collect target=0` at 5,7. Console: "SCENARIO: player took the rival's target remotely". Next decision at 6,7 was `Explore`, candidates none, so the taken target was released at once. Then `Collect target=1` at 7,3, Deliver at 10,0, Explore, and pickup 2 found at 24,1 and picked up by 26,6. |
| Unreachable target (F6) | if the ai cant reach the target it tries to find other targets | See the trace. Pickup 0 failed with PathPartial and was blocked for 6 s; the rival collected and delivered pickup 1 instead. It retried pickup 0 at 54,7 and 72,8 and failed both times. Remaining issue: it keeps retrying a route that can never work. |
| No usable pickup (F7) | if there is no scrap to collect it tries looking for it and gives up after a certain amount of time | Task stayed Explore, Target -1, Visible 0, reason "No eligible known pickup: visit the next survey point". 7 route requests around the survey points, nothing in the Console. My prediction was wrong on one point: it never gave up. The match ended as a DRAW 0–0 when the 20 s timer ran out. |

Supplied test result, with any added tests identified: 22 passed, 0 failed (EditMode, 2026-11-27). That is the 17 supplied tests plus 5 teacher extras (3 `ReferencePolicyTests`, 2 `NavigationRegressionTests`). I added no tests.
Live walkthrough checked by / recording link if used: to be added.

## One correction and one independent check

The original weakness and why it happened: the teaching baseline (`week1-before-fixes`) only checked that a pickup was available and nearer, so the rival could target a pickup it had never seen. My repository already contained the teacher's fix, so I reproduced the weakness instead: on 2026-11-27 the line `&& (candidate.Visible || candidate.Remembered)` was temporarily removed from `IsEligible` in `DecisionPolicy.cs`.

- **How it showed up in the tests**: 19 passed, 3 failed: `Stage2_HiddenUnknownPickupIsNotAUsableTarget`, `Stage2_VisibleAlternativeBeatsNearerHiddenPickup` ("Expected: 2 But was: 1") and `MatchingCurrentIdDoesNotMakeAnUnknownHiddenPickupKnown`.
- **How it showed up in the game (F4)**: the first decision was `[7,5] Collect target=0` with candidates `0(d=5,0 ---) 1(d=8,0 V--)`. The rival drove around the wall to a pickup that was hidden and that it had never seen, and the reason text still called it a "known pickup".

What I changed and why it addresses the cause: the rule was put back, so a candidate must be available, and visible or remembered, and not blocked, before distances are compared. The cause was that distance was the only test and knowledge was never checked; the rule removes unknown pickups before the nearest one is picked. After restoring it, 22 tests pass and the same F4 start gives `[7,4] Collect target=1` with the same candidates `0(d=5,0 ---) 1(d=8,0 V--)`. The rule is the teacher's reference rule, not my own invention, and the file is identical to the reference again.

My counterexample or experiment: prediction, observed result, and one limitation:

- **Prediction** (written before the F6 run): after a failed route the rival will not choose pickup 0 again until 6 seconds have passed, and it will then retry and fail again every time, because the block is only a timer.
- **Observed**: first failure at 48,6 and the retry at 54,7, which is 6,1 s later, and it failed again at 54,8. The next retry was at 72,8, which is 18 s later and not 6. The rival had gone exploring out of sensing range and only chose pickup 0 again when it came back within 10 units and saw it (`0(d=9,9 V--)`). So 6 seconds is the minimum wait, not a schedule.
- **Limitation**: one run only, timestamps are rounded to 0,1 s, and decisions are made every 0,15 s, so the 6,1 s is approximate.

## Collaboration and help

People I discussed or playtested with: none, was sick
Code, sources or substantive tools/AI assistance I used, and how I verified them:
I used Claude to just explain what needed to be done it was a bit unclear to me

If unfinished: what works, exact reproduction steps, and the help I need: the six scenarios and all tests work. Only the walkthrough recording is left.
