# Test results

All data is fictional. I set the expected result of every checkpoint before running the tests.

## Method

`OT Eval` and `OT Draft Eval` call the real workflows (`OT Contract Signed`, `OT Daily Check`, `OT Stage Update`, `OT Drafts`) through their "When called by eval" entry points. In test mode the workflows run the same code but skip every write to the real tables.

1. Six fictional contracts go through `OT Contract Signed`, which builds their plans.
2. Each project has a list of dated stage updates (`demo_stage_updates.csv`). For every simulated date, only the updates made up to that date are applied.
3. `OT Daily Check` runs once per simulated date: 2, 3, 4, 9 and 16 November 2026. The 3 and 4 November dates test the stall threshold (exactly 5 and 6 business days without a change).
4. `OT Stage Update` receives five form submissions, three of which should be rejected.
5. The results are compared with `eval_checkpoints.csv`.

## Rules: 44 checkpoints

| Run | Checkpoints | Passed | False flags | Missed flags | Note |
| --- | --- | --- | --- | --- | --- |
| 2026-11-09 19:46 | 39 | 35 | 0 | 0 | T36 to T39 "no result": bug in the test (see below) |
| 2026-11-09 19:47 | 39 | 39 | 0 | 0 | after the fix |
| 2026-11-09 20:12 | 44 | **44** | **0** | **0** | second round: 5 checkpoints for the reminder repeat rule |

| Group | Checkpoints | Examples |
| --- | --- | --- |
| Plan from template | T01 to T08 | 8 stages in order; weekend skipped; API + webhook add 6 days; SSO adds 1 day to technical access; CSV-only switches the owner to us and renames stage 4; warning when the plan ends after the target |
| Health and flags | T09 to T23 | on plan stays green; stuck at technical access goes yellow, then red on day 6 without a change (not on day 5); client blocking goes yellow, then red; late training is yellow while the launch is still reachable; red once it is not; no false "overdue" on the handover after a late launch |
| Projection and metrics | T24 to T34 | projected launch; time to first value 16, 24 and 36 days, empty when not reached; stages on time 100% and 17%; 3 projects at risk on 9 and 16 November |
| Reminder repeat rule | T40 to T44 | reminder sent 2 business days ago is held; after 3 it is sent again; a reminder for another stage does not hold this one; Friday to Monday counts as 1 business day; a second run on the same day sends nothing new. Health stays the same when a reminder is held |
| Stage update form | T35 to T39 | closing a stage while an earlier one is open, a block without a note and a future date are rejected; a valid update resets the stall clock; closing Launch creates the handover note |

"False flags" counts flags the rules raised but the checkpoint did not expect. "Missed flags" counts expected flags the rules did not raise. Both are counted on the 15 health-and-flag checkpoints.

### Bug in the first run

The four stage-update checkpoints after T35 came back as "no result". The `Compare with expected` node had "execute once" switched on, so it read only the first of the five results from `OT Stage Update`. The stage-update workflow itself had returned the correct five results. I switched the setting off and ran the eval again.

## AI drafts

`OT Draft Eval` freezes the six projects on 9 November 2026. The five projects that have not launched get both a kickoff email and a weekly status, which makes 10 drafts written by Claude Haiku. Hand-written bad drafts ("canaries") check that the code check can fail.

| Run | AI drafts passing the code checks | Canaries caught | What I found |
| --- | --- | --- | --- |
| 1, 19:53 | 10 of 10 | 4 of 4 | Reading the drafts, 3 were misleading: "Everything is on schedule" for a project at risk, "the target launch is still 6 November" when the projection was 9 November, and a request to send a file "by 3 November" on 9 November. |
| 2, 19:58 | 9 of 10 | 6 of 6 | After the fix. The one failure was a false alarm in my check: the draft said "we cannot promise that earlier date", which is the honest wording. |
| 3, 20:01 | **10 of 10** | **6 of 6** | I read all 10 again: none misleading. At-risk projects now state the projected launch, and the CSV-only kickoff says the 13 November target cannot be promised. |

Changes after run 1:
- The weekly-status brief now comes from the daily check: current due date (moved when the previous stage finished late), business days late, projected launch and health. Later stages are listed without their original dates.
- New code checks: a deadline that has already passed, "on track" or "on schedule" when health is not green, and a missing realistic launch date when the launch is at risk.
- Two new canaries for these cases (C5, C6).

Change after run 2: the promise check ignores a denied promise ("cannot promise", "can not guarantee").

| Canary | Bad draft | Must be flagged as |
| --- | --- | --- |
| C1 | Asks for API keys by an invented date | date not in the plan |
| C2 | Offers to set up the client's technical access on their behalf | offers to do the client task |
| C3 | Says an open stage is complete | says an open stage is done |
| C4 | Offers to clean data free of charge | free work or discount |
| C5 | Says "everything is on track" for a red project | says on track while health is red |
| C6 | Asks for a file by a date that has passed | asks for something by a date that has passed |

## Review form: 6 checks

`OT Draft Review` was run in test mode with five fictional drafts and a set of form answers.

| Check | Result |
| --- | --- |
| Only drafts with status `pending review` are on the page | passed |
| Draft text is escaped, so HTML in a subject cannot change the page | passed |
| A draft that passed the checks can be approved without a note | passed |
| A draft that failed the checks is kept pending when approved without a note | passed |
| Rejected without a note is kept pending | passed |
| Edit with a note is saved | passed |

The form page itself was not clicked through by a person.

## Reminder repeat rule on the real tables

I ran `OT Daily Check` twice in a row with the simulated date 20 November on a test project. The first run logged "Kickoff was due on 2026-11-11 and is 7 business day(s) late"; the second run logged "Held (reminded on 2026-11-20): Kickoff".

## Limits

- 39 checkpoints test the rules as written, on six scenarios I designed. They do not show that 5 business days is the right stall threshold for a real team.
- The code checks catch specific patterns. Run 1 showed that a draft can pass every check and still mislead, so a person reviews every draft.
- 10 drafts per run is a small sample, and each run gives different wording.
