# Operating Manual

How to run this ecosystem daily, keep it healthy, and keep improving it with any model — including cheaper ones after the model that designed it is gone.

---

## 1. Daily operation (target: ≤ 60 minutes of your attention)

### Morning (10 min) — consume, don't produce
1. Read the **07:30 briefing** (Cortex): calendar, top-5 priorities, important mail, pipeline deltas, deadlines ≤7 days.
2. Open **Today** page; accept/reorder priorities. Do not open raw Gmail first — that's the old workflow.

### Midday application block (30–40 min) — the core loop
1. **Inbox** page: triage new captured jobs (pursue/skip). Skip anything `visa: unlikely` for sponsored roles without a second thought — that's the filter's job.
2. For each "pursue": click **Tailor** → review the diff (you are checking truthfulness, not wordsmithing) → approve → generate cover letter/recruiter message → skim, fix one thing if needed, approve.
3. Submit on the actual portal (always a human act), mark **Applied**. Target 5/day.
4. Clear the **follow-ups due** list — drafts are already prepared; review and send from Gmail.

### Evening (10 min)
1. Mail-triage page: handle `action-needed` threads using prepared drafts.
2. Capture any interesting postings seen during the day (bookmarklet).
3. If interviewing tomorrow: open the prep packet, read it once tonight.

### Weekly (Friday, ~1 h)
Analytics review (response rate by version/family — act on it monthly, observe weekly) · accept/reject automation suggestions · check MarketMind eval scores and ingestion health · `docker compose ps` + backup verification (one runbook command) · 15-min retro note in `docs/retro/`.

---

## 2. Required prompts (the prompt library)

All prompts live in `prompts/*.md` as versioned files with YAML frontmatter — never inline in code:

```markdown
---
id: resume-tailor          version: 7          model_tier: strong
input_schema: TailorInput  output_schema: TailorOutput
eval_fixture: evals/fixtures/resume-tailor/
changelog:
  - v7: forbid adding metrics not present in source bullet
---
You are tailoring a resume ... {rules} ... {output format}
```

**The prompts you interact with directly** (everything else runs automatically):

| Situation | Where | Prompt pattern |
|---|---|---|
| Odd JD (hybrid role) | Job detail → "re-extract with note" | "Treat this as {family}; weight {skills} in scoring." |
| Resume tweak | Resume studio chat | "Swap the Kafka bullet for the Spark one and tighten the summary for {company}." |
| Teaching materials | Materials panel | "Adapt my teaching philosophy for {institution}: emphasize {mission/student body facts}." |
| Own-data recall | Cortex Ask | "What did I claim about {project} in the AI resume? Cite the bullet." |
| Interview prep | Prep packet chat | "Drill me: 5 system-design questions this team would ask, using their stack from the JD." |
| Market question | MarketMind Ask | "Summarize {ticker}'s last 10-Q risk-factor changes vs prior; cite sections." |

**Prompt-change protocol (non-negotiable):** edit file → bump `version` → run `python evals/run_evals.py --agent <id>` → commit only if scores don't regress. This single rule is what keeps quality stable across model swaps and future edits.

---

## 3. Maintenance

| Cadence | Task | Effort |
|---|---|---|
| Daily (automatic) | Backups, Gmail scan, ingestion; failures surface in the briefing's health line | 0 |
| Weekly | Friday checklist above | ~1 h |
| Monthly | Review response-rate analytics → update bullet DB / base resumes; rotate any expiring tokens; `docker compose pull` + redeploy | ~2 h |
| Quarterly | Refresh USCIS H-1B data (annual data, quarterly check); dependency upgrades; re-run full eval suite; prune dead jobs/threads (archive to S3) | ~half day |
| On failure | `docs/runbook.md` first — every incident you solve gets a runbook entry (write it, or have the model write it, same session) | — |

---

## 4. Continuity: how a cheaper model keeps improving this

This is designed into the architecture, not bolted on:

1. **Model gateway + `models.yaml`** — swapping any agent to a cheaper model is a one-line config change. Route by difficulty: extraction/classification → cheapest tier already; keep the strong tier only for `resume-tailor` and `materials-writer`, and test downgrades agent-by-agent.
2. **Evals are the safety net.** A cheaper model is "good enough" for an agent when its eval scores match the incumbent's. The golden fixtures encode the quality bar *outside any model's judgment* — that is the mechanism that makes model succession safe.
3. **The docs are the context.** These six documents + ADRs + runbook are written to be a complete briefing for any future model. New session bootstrap prompt:

   > "You are maintaining my personal AI platform. Read `docs/README.md`, then `docs/0N-*.md` for the system we're working on, then `docs/adr/`. Rules: never edit code without running its evals; never add a database or framework without an ADR; prompts change only via the prompt-change protocol in `docs/06-operating-manual.md` §2. Task: {task}."

4. **Small, typed, boring interfaces.** Each agent is one prompt file + one Pydantic schema + one worker function. A cheaper model can safely improve one agent without understanding the whole system — blast radius is contained by design.
5. **The improvement loop any model can run monthly:**
   - Read the funnel/response analytics → propose 3 concrete changes (bullets, prompts, scoring weights) as a PR with eval results attached.
   - Read `automation_suggestions` → implement the top accepted one.
   - Read `docs/retro/` for recurring friction → file issues.
   Human approves PRs; the model never gets send-email, place-live-order, or auto-submit capabilities regardless of how capable it is — those gates are policy, not model-trust decisions.
6. **MCP surface** (Cortex advanced): `search_kb`, `get_pipeline`, `get_today` exposed as MCP tools means any MCP-capable model — including much cheaper ones — can operate the system conversationally without custom integration.

---

## 5. Reducing manual work: the automation ladder

Every workflow starts at level 2 and earns promotion — this is how the system absorbs more work without ever taking an action you'd regret:

| Level | Meaning | Examples (steady state) |
|---|---|---|
| 0 | Manual | Final submit on job portals; sending any email; anything with money |
| 1 | Model assists on request | Interview drilling, odd-JD handling |
| 2 | Model drafts, you approve | Resumes, letters, statements, replies, trades beyond paper |
| 3 | Auto with review queue | JD extraction, visa verdicts, email classification, sentiment scores |
| 4 | Fully auto | Capture, scoring, tracking, follow-up reminders, briefings, backups, evals |

**Promotion rule:** a workflow moves up one level after ~50 consecutive outputs you approved without edits (the `events` table gives you this number). **Demotion rule:** one materially-wrong output drops it back down. Submission, sending, and real-money trading are permanently capped at their listed levels.

The end state after 90 days: your manual work is *judgment* — approving diffs, talking to humans, performing in interviews — and the system does everything else.
