# Week 1 evidence

Keep this short. Bullet points, a small table and one diagram or code trace are enough. This is your checkpoint record, not an extra report. Replace the prompts as you work.

Name: Anton Stenström
Exact Unity version: 6000.4.26f1
Checkpoint commit / repository: http://github.com/DevAStenstrom/Salvage-Sprint
Current stage and anything unfinished: 4

## Initial prediction

Three observations the rival needs, including one it might not have:

Seeing a pickup across a gap proves / does not prove reachability because:
because the rival does not know if this is the only path or another one exist and might be discovered later

One question about NPC behaviour:
how do you make an enemy NPC feel fair? 

## One complete trace

Use one actual situation. Include the observation data, derived facts, selected task and reason, movement request/result, and how feedback affects a later decision. Identify the responsible code locations. A tiny state diagram can accompany this trace.

## Scenario record

| Scenario | Predicted result                                                                                    | Observed result and short evidence / remaining issue                                                           |
|---|-----------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------|
| Normal loop and restart | the ai tries to find the closest scrap and bring it home                                            | the ai tries to find the closest scrap and bring it home                                                       |
| Carrying cargo | the ai decides to bring cargo home before looking at other pieces to collect                        | the ai decides to bring cargo home before looking at other pieces to collect                                   |
| Hidden / remembered pickup | the ai does not pick the scrap up unless it has seen it or remembered it was there                  | the ai once it sees the scrap compare which is closest the scrap it was heading to or the newly discovered one |
| Target taken | if the target is taken the ai continues searching for other targets                                 |if the target is taken the ai continues searching for other targets|
| Unreachable target | if the ai cant reach the target it tries to find other targets                                      |if the ai cant reach the target it tries to find other targets|
| No usable pickup | if there is no scrap to collect it tries looking for it and gives up after a certain amount of time |if there is no scrap to collect it tries looking for it and gives up after a certain amount of time|

Supplied test result, with any added tests identified:
Live walkthrough checked by / recording link if used:

## One correction and one independent check

The original weakness and why it happened:
What I changed and why it addresses the cause:
My counterexample or experiment: prediction, observed result, and one limitation:

## Collaboration and help

People I discussed or playtested with: none, was sick
Code, sources or substantive tools/AI assistance I used, and how I verified them:
If unfinished: what works, exact reproduction steps, and the help I need:
