# Opportunity Radar

A daily 8am ET email digest of VC fellowships, accelerators, grants, and VC job
openings — delivered to xiaomax0730@gmail.com.

## How it runs

A **Claude Code cloud routine** (scheduled agent) fires on a cron in Anthropic's
cloud. It is not a local cron job and it does not touch this machine. Each morning
it spins up a fresh session, researches, composes an HTML email, and sends it
through the Resend API.

- **Routine ID:** `trig_01Pj4QSjr4z76fT55NbEcjLM`
- **Manage / read run logs:** https://claude.ai/code/routines/trig_01Pj4QSjr4z76fT55NbEcjLM
- **Schedule:** `0 12 * * *` UTC = 8:00am ET (see DST caveat below)
- **Model:** `claude-sonnet-5`

## Why the prompt carries everything

Cloud routines cannot read local files or local environment variables. So the
roster and the Resend key both live inside the routine's prompt, not in this
directory. **This directory is the source of truth you edit; the routine is the
deployed copy.**

## Why there are no hardcoded deadlines

The obvious design — a YAML of programs and their deadlines — fails badly here.
Fellowship aggregator sites publish contradictory and frequently fabricated dates
for SEO. During research, one listed the Thiel Fellowship with a hard December 31
deadline; Thiel reviews on a rolling basis. Baked-in dates also rot silently: the
digest keeps looking authoritative long after it stopped being correct.

So `programs.yaml` stores only durable facts — the canonical URL and the typical
cycle — and the agent re-verifies live status against the official source each
morning. The prompt contains an explicit blocklist of aggregator domains that may
be used to *discover* a program but never to *source a date*, and requires every
item to be marked `OPEN` / `CLOSED` / `OPENS SOON` / `UNCONFIRMED`.

## Files

| File | Purpose |
|---|---|
| `programs.yaml` | The tracked roster. Edit this to add or drop a program. |
| `routine_prompt.md` | The deployed prompt, with `__RESEND_API_KEY__` as a placeholder. |
| `README.md` | This file. |

The real Resend key is **only** in the routine config on the Claude account — it is
deliberately not written to disk here.

## Changing what's tracked

1. Edit `programs.yaml`.
2. Mirror the change into the roster tables in `routine_prompt.md`.
3. Push the updated prompt to the routine — ask Claude Code:
   *"update the Opportunity Radar routine prompt from routine_prompt.md"*
   (substituting the real Resend key for the placeholder).

## Changing the schedule

Cron is fixed UTC and does **not** follow US daylight saving. `0 12 * * *` is
8:00am ET from March to November, but 7:00am ET once the US falls back in
November. To hold 8am ET through the winter, change it to `0 13 * * *` then.

## When the email doesn't arrive

Open the run at https://claude.ai/code/routines and read the log. The prompt
requires the agent to print either `SENT` with a message id or `FAILED` with the
HTTP status and response body, so the failure mode should be visible. Most likely
causes: expired/revoked Resend key (`401`), or a `from`-address rejection (`422`).
