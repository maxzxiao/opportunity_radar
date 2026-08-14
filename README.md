# Opportunity Radar

A daily 8am ET email digest of VC fellowships, accelerators, grants, and VC job
openings — delivered to xiaomax0730@gmail.com.

## How it runs

A **Claude Code cloud routine** (scheduled agent) fires on a cron in Anthropic's
cloud. It is not a local cron job and it does not touch this machine. Each morning
it spins up a fresh session, researches, composes an HTML email, and sends it
through the Resend API.

- **Repo:** https://github.com/maxzxiao/opportunity_radar (`main` + `claude/discoveries`)
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

## How discovery works

The roster is the reliable backbone, but it's fixed — the interesting programs are
the ones nobody's roster has yet. Two mechanisms handle that.

**Weekday rotation.** Each run is a fresh session with no recollection of yesterday,
so an instruction like "vary your search angle" is worthless — the agent has nothing
to vary against. Instead the discovery angle is keyed to the weekday: Monday hunts
fund-run scout programs, Tuesday hard tech grants, Wednesday AI research residencies,
Thursday newly-opened accelerator batches, Friday foundation open calls, Saturday
international programs, Sunday a "just announced this week" sweep. Deterministic, and
it produces real coverage variety instead of the illusion of it.

**The memory branch.** Discoveries accumulate in `memory/discovered.md` on a
long-lived `claude/discoveries` branch. Each run checks it out, reads what's already
known, and skips anything unchanged — so the discovery section stays genuinely new
rather than repeating itself into irrelevance.

This branch is not an arbitrary choice. Anthropic-hosted cloud sessions can only push
to their **current working branch**, and only to branches prefixed `claude/`. Pushing
to `main` is rejected outright. Keeping memory on a single `claude/` branch that the
agent checks out at the start means read and write both work with no merge step, no
PR, and no auto-merge workflow.

## Files

| File | Purpose |
|---|---|
| `programs.yaml` | The tracked roster. Edit this to add or drop a program. |
| `routine_prompt.md` | The deployed prompt, with `__RESEND_API_KEY__` as a placeholder. |
| `memory/discovered.md` | Accumulated discoveries. The routine only reads/writes the `claude/discoveries` copy. |
| `.gitignore` | Excludes `send.py` — it's generated with the real key substituted in. |
| `README.md` | This file. |

## Promoting a discovery

When something keeps appearing and looks like a keeper, the run log will say
*"Suggest promoting X to the tracked roster."* Move it into `programs.yaml`, mirror it
into the roster tables in `routine_prompt.md`, and ask Claude Code to push the updated
prompt. Promotion is deliberately manual — it's the one point where a human decides
what's worth watching every day.

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
