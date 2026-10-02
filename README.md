# AI Change Impact Analyzer

Static analysis finds what a change touches; AI explains why it matters. Point it at a Python repo on GitHub, pick a commit or pull request, and get an impact map, risk level, test plan and a downloadable report.

```
GitHub repo → commit/PR → change detection → AST + dependency graph → impact map → AI risk analysis → tests → report
```

## Run

```bash
pip install -r requirements.txt
export ANTHROPIC_API_KEY=sk-ant-...        # optional: AI-written explanations (rule-based without it)
export GITHUB_TOKEN=ghp_...                # optional: private repos, higher API limits, PR comments
uvicorn app:app --reload                   # http://127.0.0.1:8000
```
Requires `git`. Repos are cached in your temp folder; history lives in `history.db` (SQLite, override with `AICA_DB`).

## Features

- **Repo, commit and PR selection** (PRs via GitHub API + `pull/N/head`; a PR is analyzed against its merge-base).
- **Change detection** down to changed functions/classes (diff hunks mapped onto AST line ranges).
- **Dependency analysis**: reverse import graph, 3 hops; affected functions, classes, **HTTP routes** (Flask/FastAPI-style decorators) and tests.
- **Impact map**: interactive SVG, Changed → Dependent → Potentially affected → Tests.
- **Risk**: Low/Medium/High from a transparent score, plus explicit risk factors. Claude adds per-component explanations, consequences and recommendations; it never changes the level.
- **Tests**: must/should-run list and a ready `pytest` command (function-level when possible).
- **History** (SQLite), **reports** (standalone HTML, print to PDF, Markdown), validation, timeouts, and clear error messages.

## API

| Endpoint | Purpose |
|---|---|
| `GET /api/commits?repo=` · `GET /api/pulls?repo=` | List changes to pick from |
| `POST /api/analyze` `{repo, sha \| pr}` | Run and store an analysis |
| `GET /api/history` · `/api/history/{id}` · `DELETE` | Past analyses |
| `GET /api/history/{id}/report?format=html\|md&download=true` | Report |
| `POST /api/webhook/github` | PR webhook (HMAC-verified) |

## Automatic PR analysis

**Webhook**: set `GITHUB_WEBHOOK_SECRET` (and `GITHUB_TOKEN` to post the report as a PR comment). In the GitHub repo add a webhook to `https://<your-host>/api/webhook/github`, content type `application/json`, event *Pull requests*. Unsigned requests are rejected.

**Actions**: copy `.github/workflows/impact-analysis.yml`, set the repo variable `ANALYZER_URL`; the report appears in each PR's job summary.

## Scoring

Affected file: hops (35/20/10) + 20 if it references a changed symbol + fan-in (≤15) + change size (≤15). Changed file: 20 + fan-in (≤30) + churn (≤20) + 10 for public symbols. High ≥ 65, Medium ≥ 40. Overall risk is the top score, raised by risk factors (e.g. changed API, no covering tests, 6+ Medium/High components).

## Limitations

Import-level static analysis: dynamic imports/reflection are invisible; symbol matching is by name; deleted files are flagged but importers aren't traced; API detection covers `@x.get("/path")`-style decorators; no auth on the dashboard itself (run it locally or behind your own auth). Next: call-graph precision, coverage-based test selection, graph export.
