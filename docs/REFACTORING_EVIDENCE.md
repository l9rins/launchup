# Refactoring evidence

Significant changes to the inherited LaunchUp codebase, with the commit or pull request that made each one.

- PR numbers `#7`–`#69` were opened on [johannn07/launchup](https://github.com/johannn07/launchup/pulls?q=is%3Apr). The same merge commits are in this repository, so each one also links to its commit here.
- Later work was merged as branches; those link to the merge commit.

## Architecture and AI pipeline

| Change | Before | After | Evidence |
|---|---|---|---|
| Configurable AI pipeline with per-run provenance | Model and prompt behaviour hardcoded; `temperature`/`maxOutputTokens` passed where the SDK dropped them | `AiConfigService` resolves model and five enhancement flags from env; every generation writes an `ai_generation_runs` row with its exact config | PR #7, [661f27e](https://github.com/l9rins/launchup/commit/661f27e) |
| RAG corpus | `verifiedFrameworks` and `businessModels` hardcoded to `[]` | 54 rubric rows + 10 framework rows, each with provenance; deterministic and semantic retrieval | PR #13, [c1b978d](https://github.com/l9rins/launchup/commit/c1b978d) |
| Output validation layer | None | `OutputValidatorService` wired into RNA, RNS, roadblock generation | PR #18, [f3a2c24](https://github.com/l9rins/launchup/commit/f3a2c24) |
| Weighted scoring | Fixed weights keyed per tier, which made the composite non-monotonic | `WeightProfileService` cascade by sector and business model, falling back to `DEFAULT_WEIGHTS` | PR #19, [682657c](https://github.com/l9rins/launchup/commit/682657c) |
| Adversarial application summary | Single positive-first summary | Field-ordered response: unmet criteria and risks before any summary | PR #25, [70a66c4](https://github.com/l9rins/launchup/commit/70a66c4) |
| Leniency flag threshold | `flagged = criticalCount === 0`, measured to fire 0/10 | `flagged = ratio < 0.75`, validated on a held-out run | PR #26, [67f6071](https://github.com/l9rins/launchup/commit/67f6071) |
| AI error handling | Gemini 429/503 surfaced as a 500 | Readable 503, retry on 503, daily quota told apart from per-minute limit | [8eb4bc6](https://github.com/l9rins/launchup/commit/8eb4bc6), [cd6c3b1](https://github.com/l9rins/launchup/commit/cd6c3b1) |
| AI quota indicator | Users hit the limit with no warning | Backend counts every Gemini call; pages show requests left and reset time | [a9d4971](https://github.com/l9rins/launchup/commit/a9d4971) |

## Security and roles

| Change | Before | After | Evidence |
|---|---|---|---|
| Auth guards | 11 controllers had no guard; JWT secret had a fallback | Guards added; secret required | PR #11, [97690b7](https://github.com/l9rins/launchup/commit/97690b7) |
| Three-role model per SRS §2.3 | Admin role plus a cookie-driven "Manager as Mentor" pseudo-role | Startup, Mentor, Manager. Existing Admin accounts converted to Manager on boot; separate founder and Manager logins | PR #49, [0d6b5f6](https://github.com/l9rins/launchup/commit/0d6b5f6) |
| Startup read access (IDOR) | Any user could read any startup | Read endpoints scoped to owner, assigned mentor, or Manager | PR #50, [1400349](https://github.com/l9rins/launchup/commit/1400349) |

## Deployment

| Change | Before | After | Evidence |
|---|---|---|---|
| First deployment | Never deployed (the old live URLs belong to the previous team) | Render backend, Vercel frontend, Neon DB | PRs #35–#36, [9e92a44](https://github.com/l9rins/launchup/commit/9e92a44), [967e693](https://github.com/l9rins/launchup/commit/967e693) |
| Same-origin API proxy | Browser tried to send a Vercel cookie to Render, which can never work | `/api/[...path]` swaps the cookie for a Bearer header; 19 callers moved | PR #37, [b93e213](https://github.com/l9rins/launchup/commit/b93e213) |

## Frontend fixes

| Change | Before | After | Evidence |
|---|---|---|---|
| Svelte 5 reactivity | 11 state variables declared without `$state`, so dialogs and lists never re-rendered | All converted; exposed and fixed a missing `populate` | [2870e9e](https://github.com/l9rins/launchup/commit/2870e9e) |
| Rating refresh | A rating needed a page reload; header tier read an arbitrary row | Re-score and invalidate all cached views; tier reads the newest evaluation | [958bbf7](https://github.com/l9rins/launchup/commit/958bbf7), [ee48b09](https://github.com/l9rins/launchup/commit/ee48b09) |
| Application submit | Submit threw "Invalid URL" server-side and saved nothing | Server action calls the API directly and shows failures | [f7eb460](https://github.com/l9rins/launchup/commit/f7eb460) |
| Design system pass | Mixed widths, radii and accents per page | One 72rem frame, one accent, one radius scale across landing, auth and app | [9125433](https://github.com/l9rins/launchup/commit/9125433), [03d3ed7](https://github.com/l9rins/launchup/commit/03d3ed7), [05ab8d5](https://github.com/l9rins/launchup/commit/05ab8d5) |

## How to show a comparison

```bash
git show --stat 0d6b5f6        # files touched by the Admin role removal
git diff 0d6b5f6^1 0d6b5f6     # full before/after
```

On GitHub: `https://github.com/l9rins/launchup/compare/<before>...<after>`.
