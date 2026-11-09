# Data and privacy

All companies, people and dates in this repository are fictional.

## How data gets in today

Two n8n forms fill two data tables:

- `Contract signed form` (workflow `OT Contract Signed`) creates one row in `OT Projects` and eight rows in `OT Stages`.
- `Stage update form` (workflow `OT Stage Update`) changes the status of one stage.

Both forms require an n8n login, because they change project data.

## Data tables

| Table | One row per | Main fields |
| --- | --- | --- |
| `OT Projects` | signed contract | `project_id`, `client`, `plan`, `tech_contact`, `integrations`, `signed_date`, `target_launch`, `planned_launch`, `plan_warnings` |
| `OT Stages` | stage of a project | `project_id`, `seq`, `stage_key`, `stage_name`, `owner` (`us` or `client`), `business_days`, `planned_start`, `due_date`, `status`, `status_since`, `done_on`, `note` |
| `OT Daily Log` | project per daily check, plus one `ALL` row | `health`, `flags`, `reminders`, `current_stage`, `projected_launch`, `ttfv_days`, `on_time_pct` |
| `OT Drafts` | email draft or handover note | `kind`, `subject`, `body`, `check_passed`, `check_notes`, `review_status` (always starts as `pending review`) |
| `OT Eval Results` / `OT Eval Runs` | test checkpoint / test run | expected, actual, passed |

`status_since` is the date of the last status change. The stall rule counts business days from the later of this date and the day the previous stage was finished.

## How data would get in for real

| | A. Pull from a project tool and CRM | B. Webhook from the CRM |
| --- | --- | --- |
| Trigger | When a deal is marked "closed won" or once a day | The CRM calls `https://<instance>/webhook/contract-signed` when a deal closes |
| Authentication | API key or OAuth2 stored in n8n credentials | `X-API-Key` header checked by the Webhook node; requests without it get 401 |
| Writing | Upsert into `OT Projects` by the deal ID, then build the plan | Validate fields, reply 400 with the errors or 200 with the project ID |
| Stage updates | Read task status from the project tool | The project tool posts status changes to a second webhook |

The rules do not change with the source: they only read `OT Projects` and `OT Stages`.

## What goes to the AI

The model receives the stage table (stage names, owner as "our team" or "your team", dates, status and the blocker note), the next stage and the launch dates. It does **not** receive the client name, the technical contact or any email address: the name is added by code after the reply. Nothing is sent to a client; every draft waits for review.

## Limits for a real setup

- Contact names and emails are personal data. A real deployment needs access rules on the tables, retention settings for the execution history and a data processing agreement with the AI provider.
- Business days ignore public holidays. A real setup needs a holiday calendar per country.
- Reminders are written to the daily log, not sent. Sending them needs an approval step and a channel (email or chat).
