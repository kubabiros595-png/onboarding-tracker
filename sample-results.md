# Sample results

Fictional examples from n8n test runs on 9 November 2026 (simulated date). Every draft was saved as `pending review`; nothing was sent.

## Weekly status: client-side stage late (P2 Brightmoor Freight, red)

Daily check said: Technical access 7 business days late, no change for 9 business days, reminder to the client's technical contact. Projected launch 2 December, target 11 December.

**Subject:** Weekly status update: technical access is late

> Hi Brightmoor Freight team,
>
> Here is our weekly status update.
>
> Done: Kickoff, held on 23 October 2026.
>
> Late: Technical access. It was due on 29 October 2026 and is now 7 business days late. The stage is in progress. We have asked your IT team for the API keys again.
>
> Next: Technical access, your team's task, due 29 October 2026. Because this date has passed, please send the API keys as soon as you can.
>
> Waiting on your side: the API keys for Technical access. Integration installed is also your team's task, and it comes later in the plan.
>
> Launch: The projected launch is 2 December 2026. The target launch is 11 December 2026. The late Technical access stage could affect both dates.
>
> Please reply in writing to confirm when we can expect the API keys.
>
> Best regards,
> Implementation team

Code check: all checks passed.

## Weekly status: our stage late, launch at risk (P4 Tidewell Solar, red)

**Subject:** Weekly update: Launch stage is late

> Hi Tidewell Solar team,
>
> Here is your weekly status update.
>
> Done so far:
> - Kickoff, 23 September 2026
> - Technical access, 30 September 2026
> - Integration installed, 20 October 2026
> - First test event, 22 October 2026
> - First value, 27 October 2026
> - Training, 4 November 2026
>
> Late: Launch was due 5 November 2026 and is 2 business days late. It is still in progress.
>
> Next: Launch, owned by our team, due 5 November 2026.
>
> Our projected launch date is 9 November 2026. Our target was 6 November 2026, so launch will be later than planned.
>
> Your tasks: none right now. Nothing is waiting on your side.
>
> Please reply by email if you have any questions.
>
> Best regards,
> Implementation team

## Kickoff when the template cannot meet the target (P6 Cobalt Ridge Hotels, CSV import only)

**Subject:** Welcome aboard: your onboarding plan starts now

> Hi Cobalt Ridge Hotels team,
>
> Thank you for signing. Our onboarding starts now.
>
> First, our team holds the kickoff, due 30 October 2026. Next, your team completes Technical access, due 4 November 2026. Please confirm your technical contact by replying to this email.
>
> Your tasks:
> - Technical access: due 4 November 2026
> - First value: due 16 November 2026
>
> Our plan puts launch on 19 November 2026. That is later than your 13 November 2026 target, so we cannot promise that earlier date.
>
> Please reply in writing with your technical contact's name and email.
>
> Best regards,
> Implementation team

In run 2 the code check flagged this draft for "promise or guarantee" because of the words "cannot promise". That was a false alarm in my check, not a problem in the email, so the check now ignores a denied promise.

## The same project before the fix (run 1)

The first version gave the AI the original plan dates instead of the daily-check state. This draft passed every code check, but it is wrong: the project was red, with a projected launch of 19 November against a 13 November target.

> […] After that, Training is due 18 November 2026, Launch is due 19 November 2026, and Handover to CSM is due 23 November 2026.
>
> Everything is on schedule. […]

## Handover note to the CSM (P5 Orchard Lane Insurance)

Built by code, not AI, when Launch was marked done on 26 October 2026:

```
HANDOVER: Orchard Lane Insurance (P5)
State: launched on 2026-10-26 (target 2026-11-20). Plan: Growth. Integrations: api.
Time to first value: 16 days. Stages done late: none.
Open items: Handover to CSM (us, due 2026-11-03).
Notes from the implementation: Training: Client asked for a second session for the support team.
Contacts: technical contact Priya Sand (fictional).
Next step: CSM sends a welcome email within 2 business days, by 2026-10-28, and agrees the first check-in in writing.
```

## Daily check on 9 November 2026

| Project | Current stage | Health | Flags | Projected launch | Target |
| --- | --- | --- | --- | --- | --- |
| P1 Larkspur Dental Group | First value (client) | green | none | 17 Nov | 4 Dec |
| P2 Brightmoor Freight | Technical access (client) | red | overdue, stalled | 2 Dec | 11 Dec |
| P3 Quillfeather Books | First value (client), blocked | yellow | overdue, blocked by client | 12 Nov | 27 Nov |
| P4 Tidewell Solar | Launch (us) | red | overdue, launch at risk | 9 Nov | 6 Nov |
| P5 Orchard Lane Insurance | All stages done | green | none | launched 26 Oct | 20 Nov |
| P6 Cobalt Ridge Hotels | First test import (us) | red | launch at risk | 19 Nov | 13 Nov |

The full table for all five simulated dates is in `daily_check_by_date.csv`.
