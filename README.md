# AI Job-Search Engine

A job-application engine that reads job descriptions, scores them against my CV with a **local LLM**, tailors a CV and cover letter per posting, and fills ATS application forms in a browser — running **100% on local hardware with zero cloud API costs**.

Built on the open-source [career-ops](https://www.npmjs.com/package/@santifer/career-ops) framework, wired to a self-hosted inference server (see the companion `local-ai-stack` repo) with custom patches for streaming, thinking-control, and token budgets.

## What it does

```
Daily cron (09:00)
  ├─ Discovery: portal JSON APIs (Greenhouse / Lever / Ashby / Workable) + web search
  ├─ Dedupe vs scan-history.tsv
  ├─ Evaluation: local LLM scores each JD in 7 blocks
  │    match matrix · level strategy · comp analysis · customization plan
  │    interview stories · posting legitimacy · risk summary
  ├─ Tailoring: per-JD CV (HTML → PDF) + cover letter
  ├─ Fact gate: every employer/title/tool/metric claim cross-checked against cv.md
  └─ Digest: apply/skip table with WHO column (agent fills public ATS forms,
       human handles LinkedIn Easy Apply / CAPTCHAs)
```

## Highlights

- **Truth enforcement.** A fact-gate validator blocks any generated CV or letter that claims a tool, employer, title, or metric not present in the source `cv.md` (or an explicit allowlist). The LLM cannot hallucinate its way into an interview.
- **Local engine wiring.** OpenAI-compatible endpoint on a home NPU server; custom patches for SSE streaming (avoids header timeouts on long generations), `enable_thinking: false` control (stops reasoning tokens eating the output budget), and raised `max_tokens`.
- **ATS browser automation.** Fills Lever/Greenhouse/Ashby/Workable/Manatal forms via CDP: shadow-DOM file uploads, numeric-masked salary fields, quick-apply modal propagation. CAPTCHA solves stay human.
- **Legitimacy screening.** Flags citizenship requirements, phantom aggregator listings (cited employer whose careers domain is NXDOMAIN), stale postings, and salary-structure ambiguity before wasting an application.

## Layout

| Path | Purpose |
|---|---|
| `openai-eval.mjs` | 7-block JD evaluation against `cv.md` |
| `openai-tailor.mjs` | Per-JD tailored CV (HTML) |
| `generate-cover-letter.mjs` | Cover letter PDF renderer with fact gate |
| `generate-pdf.mjs` | CV PDF renderer + `verify-cv-facts.mjs` gate |
| `config/profile.yml` | Candidate profile, comp floors, visa status |
| `modes/` | Agent-driven workflow playbooks (auto-pipeline, cover, oferta) |
| `portals.yml` | Portal search queries + verified careers URLs |
| `data/` | Scan history, PDF index, tracker |
| `reports/032-*` | Sample evaluation report (real Dubai posting) |
| `output/` | Sample tailored CV + cover letter PDFs |

## Engine config

`.env` points everything at the local server:

```
OPENAI_BASE_URL=http://127.0.0.1:8731/v1
OPENAI_MODEL=halogen-qwen3.8-flash-next
OPENAI_API_KEY=<any-ascii-string>
```

## Sample artifacts

Included in `reports/` and `output/`: a full evaluation (score, match matrix, legitimacy check) plus the tailored CV and cover letter produced for a real Senior AI Developer posting in Dubai.

---
*Framework: [career-ops](https://www.npmjs.com/package/@santifer/career-ops) by @santifer — this repo is my deployment with local-engine wiring, patches, and pipeline automation.*
