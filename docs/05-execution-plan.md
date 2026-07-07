# 90-Day Execution Plan

## Priority order (and why)

1. **System 1 — JobOps** (weeks 1–5): produces interviews directly; every later week benefits from it running.
2. **System 2 — MarketMind** (weeks 6–8): the strongest portfolio artifact for AI/Data Engineer roles.
3. **System 4 — Cortex-lite** (weeks 9–10): cheap to build on System 1's platform; locks in daily time savings.
4. **System 5 — StudioForge + publish** (weeks 11–13): the video factory takes the stretch slot (avatar + educational line); System 3 (Atelier) moves to backlog — see the verdict table in [README.md](README.md).

**Standing rule from week 2:** apply to jobs *through the system* every weekday (target: 5 quality applications/day industry, plus academic postings as they appear). The job search is the product's production traffic; building without applying is failure.

## Weekly milestones

### Phase 1 — JobOps (Systems 1 platform)

| Week | Milestone | Definition of done |
|---|---|---|
| 1 | Platform skeleton + capture→extract | Compose stack up (api/worker/web/pg); `POST /jobs` with URL → extracted `JobPosting` JSON visible in a basic list UI; bullet DB seeded from your real resumes |
| 2 | Scoring + visa + tracking | USCIS H-1B data loaded & joined; fit score + ATS keyword gap on job detail page; applications table + kanban; **start daily applications (manually tailored still)** |
| 3 | Resume studio | Tailor → diff → approve → Typst-rendered PDF for all 5 versions (SWE/AI/Data/Teaching/Academic CV); anti-hallucination verifier on |
| 4 | Materials generation | Cover letter, recruiter message, short answers; teaching-track: philosophy/research/diversity statements adapted from your master versions; follow-up scheduler live |
| 5 | Hardening + analytics | Gmail alert scan + reply detection; funnel & response-rate-by-version charts; backup cron; tag `v1.0`, write README + demo GIF |

### Phase 2 — MarketMind

| Week | Milestone | Definition of done |
|---|---|---|
| 6 | Data pipeline | Bars/news/filings ingesting for 30 tickers into Timescale + pgvector + S3; ticker overview page |
| 7 | RAG assistant + evals | `/ask` with cited structured answers; 50-question golden set; eval run in CI |
| 8 | Trading engine (paper) | Momentum strategy + vectorbt backtest page; risk manager + kill switch; Alpaca paper orders flowing; tag `v1.0`, record 3-min demo video |

### Phase 3 — Cortex-lite

| Week | Milestone | Definition of done |
|---|---|---|
| 9 | Email + KB agents | Email classify/summarize/link running every 30 min; KB ingested; `/kb/ask` with citations |
| 10 | Briefing + Today | 07:30 daily briefing arriving; Today page is your default tab; reply drafting to Gmail drafts |

### Phase 4 — Publish & stretch

| Week | Milestone | Definition of done |
|---|---|---|
| 11 | StudioForge MVP part 1 + portfolio packaging | Voice clone + avatar set up; text → script → cloned VO → avatar render → assembled video → **first real video uploaded (private) and published after review**. In parallel: public repos get architecture READMEs, demo video, resume/LinkedIn updated |
| 12 | StudioForge MVP part 2 | Production board, packager (titles/thumbnails/metadata), analytics sync, Shorts auto-cut; 2 more videos published; blog post #1 (financial RAG evals) |
| 13 | Retro + steady state | Response-rate analysis → resume improvements; Operating Manual routines fully adopted (incl. 1 video/week at ~2 h); manga-line go/no-go decided per doc 07 §11; backlog groomed (Atelier, MarketMind advanced) |

**Weekly rhythm:** Mon–Thu build (2–4 h/day) + apply daily via JobOps; Fri: 1 h maintenance (see Operating Manual) + weekly retro note; interviews always preempt build time — that's the system working, not a schedule slip.

## GitHub repository structure

Three repos. JobOps/Cortex stays **private** (it contains your pipeline data and personal corpus); the portfolio repos are public.

```
jobops/                      # PRIVATE — Systems 1 + 4 (shared platform)
├── apps/
│   ├── api/                 # FastAPI: routers/, services/, models/, db/
│   ├── worker/              # jobs/: ingest_job.py, tailor.py, email_scan.py, briefing.py ...
│   └── web/                 # Next.js dashboard
├── packages/
│   ├── llm/                 # model gateway, models.yaml, retry/logging
│   └── schemas/             # Pydantic + zod-generated types
├── prompts/                 # ONE FILE PER AGENT, versioned (see Operating Manual)
│   ├── jd-extractor.md  resume-tailor.md  materials-cover-letter.md ...
├── evals/
│   ├── fixtures/            # golden inputs/outputs per agent
│   └── run_evals.py
├── resume-data/             # bullets.yaml, statements/, templates/*.typ
├── infra/                   # docker-compose.yml, Caddyfile, backup.sh
├── migrations/
├── docs/                    # these architecture docs, ADRs/
└── .github/workflows/       # ci.yml (tests + evals), nightly.yml

marketmind/                  # PUBLIC — System 2
├── apps/{api,web}/ 
├── services/{ingester,analyst,trader}/
├── strategies/              # momentum.py, base.py — plugin interface
├── prompts/  evals/  infra/  migrations/  docs/  notebooks/
└── README.md                # architecture + demo video + DISCLAIMER

studioforge/                 # System 5 — pipeline code PUBLIC; media/voice assets PRIVATE (S3 only)
├── apps/{api,web}/          # production board UI shares the jobops dashboard component kit
├── pipeline/
│   ├── script/  tts/  avatar/  manga/   # asset plugins behind one interface
│   └── assemble/            # Remotion compositions + ffmpeg timeline
├── prompts/  evals/         # incl. labeled manga chapter for panel/OCR evals
├── infra/                   # compose + per-render GPU job templates (Modal/RunPod)
└── docs/                    # incl. rights-policy.md (doc 07 §11 distilled)

atelier/                     # PUBLIC — System 3 (backlog)
├── apps/{storefront,brain}/ # Next.js on Vercel; FastAPI assistant/embeddings
├── prompts/  evals/  seed/  docs/
```

## Documentation structure (every repo, same shape)

```
README.md            # what/why, architecture diagram, quickstart, demo link
docs/
├── architecture.md  # from these blueprints, kept current
├── adr/             # 0001-postgres-only.md, 0002-no-llm-in-order-path.md, ...
├── runbook.md       # start/stop, backup/restore, common failures
├── prompts.md       # prompt-change protocol + eval gate
└── api.md           # generated from OpenAPI
```

ADRs matter doubly here: they are the maintenance memory that lets a cheaper model (or future-you) change the system safely, and they're interview talking points.
