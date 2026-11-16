# Feature completion matrix

Status as of 2026-11-06, checked against `master` at [3ef2647](https://github.com/l9rins/launchup/commit/3ef2647). All 16 system tests were run on a local copy of that commit the same day, the AI ones with a Gemini key: 14 pass, 2 fail.

Legend: ✅ Done · 🟡 Partial · ⛔ Not built · 🔬 Research task (not code)

Live system: <https://launchup-enhanced.vercel.app>. Demo accounts are in the [README](../README.md#demo-accounts).

## Core workflows

| Feature | Role | Status | Where | Known limitations |
|---|---|---|---|---|
| Register and sign in | Startup | ✅ | `/register`, `/login` | `/forgot-password` link has no page. A confirmation email is sent when `SMTP_*` is set; Render's free tier blocks SMTP, so it is not sent from the live system |
| Failed sign-ins shown in Recent Activity | Manager | ✅ | `/admin` | Stored as a date only, so every entry shows 12:00 AM. Sign-in has no rate limit, so the feed can be flooded |
| Manager sign-in | Manager | ✅ | `/manager-login` | |
| Apply with a startup (wizard) | Startup | ✅ | `/startups` → "Apply with a startup" | No loading indicator on submit (see revisions). The wizard's "Startup name" field is ignored |
| Capsule proposal upload with OCR (typed PDF, photo, scan) | Startup | ✅ | wizard, Project details step | OCR accuracy (CER) not yet measured. If Gemini refuses, the upload is queued and the form fills when it is parsed; the Overview page's upload is not queued |
| Edit a waitlisted application | Startup | ✅ | application pop-up, `/apply?startupId=` | Pop-up shows "NaN% complete" |
| Review applications: qualify, waitlist, reject | Manager | ✅ | `/applications` | Admissions routes are Manager-only since b5d99ed; a founder gets 403 |
| AI application summary with leniency flag; approval of a flagged summary needs acknowledgement | Manager | ✅ | `/applications` | |
| Assign or change a mentor, with load shown | Manager | 🟡 | `/applications`, `/admin/startups` | Assigning works; the mentor list shows name and email but no load (KL-17) |
| Create, edit, delete startups | Manager | ✅ | `/admin/startups` | |
| Readiness assessments (answer per dimension) | Startup | ✅ | `/startups/[id]/assessment` | |
| Rate a readiness level | Mentor, Manager | ✅ | `/startups/[id]/assessment`, `/startups/[id]/readiness-level` | A Manager may override a Mentor's rating (adviser-approved). Last rating wins; no record of who rated; the rate route has no role check |
| Tier badge, radar chart, gap analysis | All | ✅ | `/startups/[id]/readiness-level` | Gap shows the points short of the next tier. The dashboard labels dimensions Team, Product, Traction, Funding instead of the SRS names (KL-18) |
| Filter startups by readiness tier | All | ✅ | `/startups` | Client-side filter |
| Alert Managers to invalid tier or weight config | Manager | ✅ | `/admin` Recent Activity | Invalid tier rows are skipped |
| RNA, RNS, Roadblocks, Initiatives: AI generate, edit, Kanban | Mentor, Startup | ✅ | `/startups/[id]/rna`, `rns`, `roadblocks`, `initiatives` | Gemini free tier allows 20 requests/day for the whole app; one next-steps generation uses 4 of them |
| Queue refused AI generation and retry it | Mentor, Startup | ✅ | same pages, queued-jobs panel | Retries at 2, 5, 15, 30, 60 min, up to 6 attempts. No row lock: two servers on one database can run a job twice. A queued baseline-arm request replays with the full pipeline |
| RNA validation and fact check shown to Mentors | Mentor | ✅ | `/startups/[id]/rna` cards | Flags items that contradict the scores or profile; it does not check every claim |
| Refine an item by chat with AI | Mentor, Startup | ✅ | same pages | Gemini 503 returns a readable error after two retries |
| Overview: General, Capsule Proposal, Progress report, Members, Elevate | All | ✅ | `/startups/[id]/overview/*` | Progress report is limited to the owner, assigned mentor or a Manager (403 otherwise) |
| Show founders their mentor | Startup | ✅ | `/startups` cards | |
| Cohorts and analytics | Manager | ✅ | `/cohorts`, `/analytics` | |
| User management | Manager | ✅ | `/admin/users` | |
| Assessment type configuration | Manager | 🟡 | `/admin/assessments` | "Create assessment type" posts to a GET-only route |
| Tier configuration | Manager | ✅ | `/admin/tiers` | |
| Bias audits and score normalization review | Manager | ✅ | `/admin/ai/bias-audits` | A Manager can request a re-evaluation. Audits written before 3ef2647 used a missing baseline and inflate corrected scores; don't cite them. An override keeps the status Flagged and records no reason |
| OCR document review | Manager | ✅ | `/admin/ocr-documents` | Documents are not linked to their startup, so they are pruned after 30 days |
| Account profile and appearance | All | ✅ | `/account/*` | |

## Reviewer revisions (adviser Leah V. Barbaso; Sir Busico, mentor-side respondent)

| Revision | Status | Evidence | Gap |
|---|---|---|---|
| Rating a level updates right away, no refresh | ✅ Done | `frontend/src/lib/readiness-refresh.ts` re-scores and invalidates every cached view; used by both rating pages. Branches `fix/readiness-rating-refresh` ([958bbf7](https://github.com/l9rins/launchup/commit/958bbf7)), `fix/rating-refresh-all-views` ([ee48b09](https://github.com/l9rins/launchup/commit/ee48b09)) | None on the reachable paths |
| Manager can override a mentor's rating | ✅ Approved by the adviser, Leah V. Barbaso; works as built | `POST /readinesslevel/startup/:id/rate` (`readinesslevel.controller.ts:95`) overwrites one row per dimension; `canRateReadiness` allows Mentor and Manager | Last write wins, and who rated is not stored. The route has no role check, so any signed-in user can call it |
| AI usage limit shown, e.g. "50/60" | ✅ Done | "N/20 AI requests left today · Resets at …" beside Generate on RNA, RNS, Roadblocks, Initiatives. `backend/src/ai/ai-quota.service.ts`, `frontend/src/lib/components/shared/ai-quota-note.svelte` ([a9d4971](https://github.com/l9rins/launchup/commit/a9d4971)) | Counts requests, not tokens, because Gemini's free-tier limit is per request. One shared count for all users. `AI_DAILY_REQUEST_LIMIT` changes the 20 |
| Loading animation when adding or submitting a startup | 🟡 Partial | Manager "Create Startup" shows a spinner (`admin/startups/+page.svelte:354`). OCR upload shows a spinner | The founder's "Submit Application" (`lib/components/startup/Application.svelte:351`) never sets `submitting`, so it shows nothing while the request runs. `/apply` edit mode shows "Updating..." text only |

## Capstone objectives

| Objective | Status | Note |
|---|---|---|
| 1a Prompt templates constrained to DB fields | 🟡 | Grounding instruction appended; toggle `AI_GROUNDING_ENABLED` |
| 1b RAG grounding | ✅ | Measured: 0.22 MAE vs baseline 0.69 on level placement |
| 1c Output validation | ✅ scope-limited | Checks retrieval confidence and length, and flags RNA items that contradict the scores or profile (b9a358e). It does not verify every claim |
| 2a Tier classification | ✅ | |
| 2b Weighted scoring by sector and business model | ✅ | |
| 2c Gap analysis | ✅ | |
| 3a Handwriting OCR | ✅ | Accuracy measurement moved to 3c |
| 3b Sketch / canvas recognition | ⛔ | Descoped; the inherited heuristic was measured unreliable. Recorded as an SRS deviation |
| 3c OCR accuracy and usability study | 🔬 | Harness built; needs transcribed samples |
| 4a Bias measurement vs expert ratings | 🔬 | Needs expert-rated profiles |
| 4b Adversarial prompting | 🟡 | On the application summary, not the scoring path. Adversarial re-scoring was skipped and is recorded as an SRS deviation |
| 4c Score normalization | ✅ | Skipped, and the audit marked unverified, when no usable baseline exists |

Full detail: [`TODO_CHECKLIST.md`](../TODO_CHECKLIST.md) §0.

## Known limitations

- Gemini free tier: 20 requests/day per model for the whole app. Generation fails with a readable message once spent.
- Render free tier sleeps after 15 minutes; the first request can take about a minute.
- The rating route checks login but not role. Admissions routes were fixed in b5d99ed.
- The registration email needs SMTP, which Render's free tier blocks.
- Run the AI job queue on one server per database (`AI_JOB_QUEUE_ENABLED=false` on the others).
- The raw JWT is still passed to the browser for some calls.
- RAG corpus: only the 9 TRL rows come from a public standard; the rest are authored. Provenance is stored per row.
- The system scores six dimensions; the proposal names five. Investment (IRL) was added.
