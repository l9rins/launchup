# Feature completion matrix

Status as of 2026-11-02, checked against the code on `master`.

Legend: ✅ Done · 🟡 Partial · ⛔ Not built · 🔬 Research task (not code)

Live system: <https://launchup-enhanced.vercel.app>. Demo accounts are in the [README](../README.md#demo-accounts).

## Core workflows

| Feature | Role | Status | Where | Known limitations |
|---|---|---|---|---|
| Register and sign in | Startup | ✅ | `/register`, `/login` | `/forgot-password` link has no page |
| Manager sign-in | Manager | ✅ | `/manager-login` | |
| Apply with a startup (wizard) | Startup | ✅ | `/startups` → "Apply with a startup" | No loading indicator on submit (see revisions). The wizard's "Startup name" field is ignored |
| Capsule proposal upload with OCR (typed PDF, photo, scan) | Startup | ✅ | wizard, Project details step | OCR accuracy (CER) not yet measured |
| Edit a waitlisted application | Startup | ✅ | application pop-up, `/apply?startupId=` | |
| Review applications: qualify, waitlist, reject | Manager | ✅ | `/applications` | Admissions routes check login but not the Manager role |
| AI application summary with leniency flag; approval of a flagged summary needs acknowledgement | Manager | ✅ | `/applications` | Gate is enforced in the backend; the audit row assumes the caller is a Manager |
| Assign or change a mentor, with load shown | Manager | ✅ | `/applications`, `/admin/startups` | Same missing role check as above |
| Create, edit, delete startups | Manager | ✅ | `/admin/startups` | |
| Readiness assessments (answer per dimension) | Startup | ✅ | `/startups/[id]/assessment` | |
| Rate a readiness level | Mentor, Manager | ✅ | `/startups/[id]/assessment`, `/startups/[id]/readiness-level` | Last rating wins; no record of who rated (see revisions) |
| Tier badge, radar chart, gap analysis | All | ✅ | `/startups/[id]/readiness-level` | |
| RNA, RNS, Roadblocks, Initiatives: AI generate, edit, Kanban | Mentor, Startup | ✅ | `/startups/[id]/rna`, `rns`, `roadblocks`, `initiatives` | Gemini free tier allows 20 requests/day for the whole app |
| Refine an item by chat with AI | Mentor, Startup | ✅ | same pages | Gemini 503 returns a readable error after two retries |
| Overview: General, Capsule Proposal, Progress report, Members, Elevate | All | ✅ | `/startups/[id]/overview/*` | |
| Show founders their mentor | Startup | ✅ | `/startups` cards | |
| Cohorts and analytics | Manager | ✅ | `/cohorts`, `/analytics` | |
| User management | Manager | ✅ | `/admin/users` | |
| Assessment type configuration | Manager | 🟡 | `/admin/assessments` | "Create assessment type" posts to a GET-only route |
| Tier configuration | Manager | ✅ | `/admin/tiers` | |
| Bias audits and score normalization review | Manager | ✅ | `/admin/ai/bias-audits` | |
| OCR document review | Manager | ✅ | `/admin/ocr-documents` | |
| Account profile and appearance | All | ✅ | `/account/*` | |

## Adviser revisions (Sir Busico, Ma'am Leah)

| Revision | Status | Evidence | Gap |
|---|---|---|---|
| Rating a level updates right away, no refresh | ✅ Done | `frontend/src/lib/readiness-refresh.ts` re-scores and invalidates every cached view; used by both rating pages. Branches `fix/readiness-rating-refresh` ([958bbf7](https://github.com/l9rins/launchup/commit/958bbf7)), `fix/rating-refresh-all-views` ([ee48b09](https://github.com/l9rins/launchup/commit/ee48b09)) | None on the reachable paths |
| Manager can override a mentor's rating | 🟡 Open decision, pending Ma'am Leah | `POST /readinesslevel/startup/:id/rate` (`readinesslevel.controller.ts:95`) overwrites one row per dimension; `canRateReadiness` allows Mentor and Manager | Last write wins, and who rated is not stored. The route has no role check, so any signed-in user can call it |
| AI usage limit shown, e.g. "50/60" | ✅ Done | "N/20 AI requests left today · Resets at …" beside Generate on RNA, RNS, Roadblocks, Initiatives. `backend/src/ai/ai-quota.service.ts`, `frontend/src/lib/components/shared/ai-quota-note.svelte` ([a9d4971](https://github.com/l9rins/launchup/commit/a9d4971)) | Counts requests, not tokens, because Gemini's free-tier limit is per request. One shared count for all users. `AI_DAILY_REQUEST_LIMIT` changes the 20 |
| Loading animation when adding or submitting a startup | 🟡 Partial | Manager "Create Startup" shows a spinner (`admin/startups/+page.svelte:354`). OCR upload shows a spinner | The founder's "Submit Application" (`lib/components/startup/Application.svelte:351`) never sets `submitting`, so it shows nothing while the request runs. `/apply` edit mode shows "Updating..." text only |

## Capstone objectives

| Objective | Status | Note |
|---|---|---|
| 1a Prompt templates constrained to DB fields | 🟡 | Grounding instruction appended; toggle `AI_GROUNDING_ENABLED` |
| 1b RAG grounding | ✅ | Measured: 0.22 MAE vs baseline 0.69 on level placement |
| 1c Output validation | ✅ scope-limited | Checks retrieval confidence and length, not groundedness |
| 2a Tier classification | ✅ | |
| 2b Weighted scoring by sector and business model | ✅ | |
| 2c Gap analysis | ✅ | |
| 3a Handwriting OCR | ✅ | Accuracy measurement moved to 3c |
| 3b Sketch / canvas recognition | ⛔ | Descoped; the inherited heuristic was measured unreliable |
| 3c OCR accuracy and usability study | 🔬 | Harness built; needs transcribed samples |
| 4a Bias measurement vs expert ratings | 🔬 | Needs expert-rated profiles |
| 4b Adversarial prompting | 🟡 | On the application summary, not the scoring path |
| 4c Score normalization | ✅ | |

Full detail: [`TODO_CHECKLIST.md`](../TODO_CHECKLIST.md) §0.

## Known limitations

- Gemini free tier: 20 requests/day per model for the whole app. Generation fails with a readable message once spent.
- Render free tier sleeps after 15 minutes; the first request can take about a minute.
- Several Manager-only startup routes (approve, waitlist, appoint/change mentor, mark complete) and the rating route check login but not role. The UI hides them; the API does not refuse them.
- The raw JWT is still passed to the browser for some calls.
- RAG corpus: only the 9 TRL rows come from a public standard; the rest are authored. Provenance is stored per row.
- The system scores six dimensions; the proposal names five. Investment (IRL) was added.
