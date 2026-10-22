# ResumeFit 2.0 — Evidence-First AI Resume Tailor

**A real, locally runnable UCLA AI Startup capstone demo.** This is an **upgrade of the original ResumeFit-MVP** rather than a throwaway template: it retains the selected blue-and-white UI, authentication, credits, history and optional Stripe test-mode billing. Version 2.1 adds a multi-step **verifying agent** and a per-user **Fact Bank** (see below).

> **Status (handoff):** the app runs locally end to end, including live OpenAI calls. **Not yet done: public deployment and a real Stripe test-mode round trip.** See [Handoff / remaining work](#handoff--remaining-work).

## Get running on a Mac using Cursor (5–10 minutes)

1. **Unzip** `ResumeFit-2.0.zip`. In Cursor, **File → Open Folder → `ResumeFit-2.0`**. Make sure you opened this new folder, not the original `ResumeFit-MVP`.
2. Open **Terminal → New Terminal** within Cursor. Run these commands in order:

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   python -m pip install -r requirements.txt
   python -m uvicorn server:app --reload --host 127.0.0.1 --port 8001
   ```

3. Open **http://127.0.0.1:8001/**. You can also launch `start_mac.command` from Terminal; it sets up the environment and opens the browser.

**Why port 8001?** The original ResumeFit MVP may still be running on 8000. Both can run at once without confusion. If Cursor shows an error saying the port is in use, stop the other server using `Control + C`, or choose another unused port.

### One-message Cursor Agent prompt

> Inspect README.md and requirements.txt. Run this **ResumeFit-2.0** FastAPI app locally on my Mac using port 8001, install dependencies in a local .venv, verify the /health endpoint, and give me the URL. Keep the existing blue-and-white interface and do not rewrite or replace any feature. The old ResumeFit-MVP may still be running on port 8000.

## Interactive demo — no keys required

1. Click **Try the interactive demo** under the headline. This loads a *realistic fictional resume*, a junior data analyst job description and rule-based results produced by the running backend.
2. Click any **JD Requirement chip** to inspect the original resume statement supporting the requirement, or see an explicit *not evidenced* warning.
3. Click **Show source proof** next to a bullet for original supporting text.
4. Click **Accept** next to a proposed bullet. Optionally click **Edit** to customize the text. Choose **Apply accepted edits** to update the working resume.
5. Click **Export Word .docx** to download a clean, single-column resume. Click **Copy tailored resume** or **Analysis .txt** as alternatives.
6. Try **dragging in a PDF / DOCX / TXT resume** (or click the upload zone); the extracted text fills the left input automatically. You can also upload a job description file.
7. Try **History** by registering a throwaway demo account and running an analysis, then reopen or delete it. In rule-based mode, runs do not use paid AI credits.

> **Transparent mode labeling:** The built-in guided demo is **not** GPT-generated. It uses safe deterministic rules so it runs without external secrets. The app labels it *Rule-based · Interactive demo*. Do not claim this demo proves real LLM generation. For a genuine real AI analysis, enable the API as described below.

## Enable real OpenAI-powered tailoring

1. Obtain your **own** API key from the OpenAI developer platform. A ChatGPT subscription does **not** automatically include API usage; you need an API-enabled account with available billing/credits.
2. Copy `.env.example` to `.env`, and paste the key **only into the server-side** `OPENAI_API_KEY` variable. Never include the key in GitHub, browser JavaScript, demos, or screenshots.
3. Optional: change `OPENAI_MODEL` to a model your API account can access. Default is `gpt-4.1-mini`.
4. Restart the server, refresh the page. Register/login to use one of the 3 free AI credits, then upload/paste a real resume and JD and click **Tailor Resume**. A **successful live AI analysis** uses one credit.
5. Look for the badge **Real AI · Live analysis**. This distinction is important for an honest capstone demonstration.

**Model grounding:** The system prompt requests evidence-backed JSON containing 3–5 proposed bullet rewrites, change explanations, JD requirements, source quotes and missing qualifications. The backend checks that the before/evidence excerpts are present in the original resume, and rejects new numeric claims and several named technical tools. These checks do *not* guarantee zero hallucinations. Review each suggestion, especially manually edited ones, before use. Missing qualifications mean *not documented in the resume*, not *the user cannot do them*. There is no fake ATS pass-rate percentage.

**Live-API status:** the v2.1 agent has been run against the real OpenAI API (`gpt-4.1-mini`) locally, both through `eval/run_eval.py` and through the browser UI. Results depend on your key and model access; re-run the eval after changing the model or prompts.

## 2.1 — Agentic tailoring (what changed)

With `OPENAI_API_KEY` set, `/api/tailor` no longer sends one prompt. `agent.py` runs a bounded multi-step loop:

1. **Plan** – the JD becomes 3–8 requirements tagged must-have / nice-to-have (plus a deterministic tool that detects named technologies).
2. **Retrieve (RAG)** – BM25 search over the resume's facts *and the user's saved Fact Bank* finds candidate evidence per requirement.
3. **Map** – the model must cite fact IDs; evidence text is looked up by ID in code, so it cannot be paraphrased or invented.
4. **Draft** – only facts that serve must-have requirements are rewritten.
5. **Verify + repair** – each rewrite is split into claims; every "supported" claim needs a quote that is a real substring of the source bullet. Rule checks (new numbers, new tools, mostly-new wording) run in code. Failures go back to the drafter with feedback for up to 2 repair rounds, then are **dropped** rather than shown.
6. **Coach** – summary and truthful next steps for gaps.

The UI shows a "How the agent worked" trace, per-claim proof, rejected rewrites, and a **Fact Bank** page (`/api/facts`) that persists verified statements across applications. Credits are charged only when the result is grounded (a verified rewrite or an evidenced requirement); provider errors return 502/504 and never charge.

Run `python eval/run_eval.py` (needs a key) to compare the single-prompt baseline against the agent on high / partial / low-match cases. Tests: `pip install -r requirements-dev.txt && python -m unittest discover -s tests -v` (the agent tests use a scripted LLM, no key needed).

## Features in version 2.0

| Functional today, no API key | Requires your OpenAI API key | Requires external service setup |
|---|---|---|
| Blue/white responsive workspace, sample preview, guided interactive demo | Real JD-specific LLM rewriting | Public deployment on Render or equivalent |
| PDF, DOCX, TXT extraction in memory | Live AI requirement mapping and nuanced explanations | Stripe **test** Checkout and signed webhook validation |
| Manual text input + JD document upload | AI analysis credits for logged-in users | Production-grade data persistence, protections, payment readiness |
| Before / After + Why + Original Source Proof | | |
| Accept / Edit / Apply edits + Word, TXT and clipboard export | | |
| Signup/login, session cookies, history and deletion | | |

### Upload constraints and user privacy

- Upload limit **5 MB** each, up to **20 PDF pages**, and up to **20,000 extracted characters**. Supports PDF, DOCX and plain TXT.
- The server extracts text in memory and does **not** permanently save the raw uploaded binary.
- *Text* submitted for analysis by a signed-in user can be stored in local SQLite as part of run history. Data is not shared publicly; you should not use real confidential resumes in a public classroom demo.
- Text-only PDF extraction cannot read scanned/image-only PDFs. The UI displays a clear message suggesting manual paste; no OCR is claimed.
- Word export recreates a clean **single-column textual document**, **not** the formatting of the original uploaded PDF or DOCX.
- New users receive three real-AI credits; the default demo mode is free. Stripe live billing is not enabled by default.
- This is a capstone MVP. Before inviting real paying customers: implement email verification, abuse/rate limiting, HTTPS/privacy policy/retention, persistent managed storage, app monitoring and backups, and test Stripe checkout with real webhook delivery in test mode. Do not enable live payment processing until those checks are complete.

## Tests

```bash
python -m unittest discover -s tests -v
```

26 tests cover: demo evidence, no made-up numeric claims/tools in accepted AI output, password hashing, Stripe signature validation, sample endpoint, PDF/DOCX upload, invalid/oversized files, demo analysis, Word export round-trip, **and (v2.1)** the agent loop with a scripted LLM (verification, repair, dropping unverifiable rewrites, citation integrity, retrieval, provider errors), credit-charging rules, and Fact Bank isolation. Install test dependencies first: `pip install -r requirements-dev.txt`.

## Technical architecture

```
Browser (responsive HTML + CSS + JavaScript)
  ├─ POST /api/upload — FastAPI, PDF/DOCX/TXT parsing (pypdf, python-docx)
  ├─ POST /api/tailor — demo engine OR authenticated OpenAI chat-completions API
  ├─ POST /api/export/docx — generate clean Word document from approved edits
  ├─ /api/auth/* — salted PBKDF2 passwords, HttpOnly cookie sessions
  ├─ /api/history/* — per-user persisted run versions, deletion
  └─ /api/billing/* — optional test-mode Stripe Checkout and signed webhook
                 SQLite (local dev, requires persistent storage when hosted)
```

## Deployment

**Live Website:** https://resumefit-xrcw.onrender.com

ResumeFit is deployed on Render as a publicly accessible FastAPI web application.

### Technology Stack
- **Hosting:** Render
- **Backend:** Python, FastAPI, Uvicorn
- **Frontend:** HTML, CSS, JavaScript
- **AI:** OpenAI API (`gpt-4.1-mini`)
- **Database:** SQLite with a 1 GB Render Persistent Disk
- **Payments:** Stripe Sandbox Checkout and Webhooks

### Deployment Configuration

**Build Command:**
`pip install -r requirements.txt`

**Start Command:**
`uvicorn server:app --host 0.0.0.0 --port $PORT`

**Environment Variables:**
- `OPENAI_API_KEY`
- `OPENAI_MODEL`
- `APP_BASE_URL`
- `COOKIE_SECURE=true`
- `DATABASE_PATH=/var/data/resumefit.db`
- `STRIPE_SECRET_KEY`
- `STRIPE_WEBHOOK_SECRET`

All API keys and secrets are configured securely through Render environment variables and are not committed to GitHub.

### Deployment Testing

The following functionality has been successfully tested:

- Public HTTPS website access
- User registration and authentication
- Live AI-powered resume tailoring
- Persistent user history and credits after redeployment
- Stripe Sandbox Checkout ($4.99 for 5 AI credits)
- Successful Stripe Webhook delivery (`checkout.session.completed`, HTTP 200)
- Automatic credit update from 2 to 7 credits after test payment

**Stripe Webhook Endpoint:**
`https://resumefit-xrcw.onrender.com/api/billing/webhook`


## Files

- `server.py` — API, upload parsing, business logic, authentication, database, billing.
- `static/index.html`, `styles.css`, `app.js` — production-style UI and review controls.
- `agent.py` — the tailoring agent (plan → retrieve → map → draft → verify/repair → coach).
- `eval/run_eval.py`, `eval/results.md` — baseline-vs-agent evaluation harness and its last output.
- `tests/` — 26 passing automated tests (no API key needed).
- `.env.example` — environment variable template (no real secrets).
- `DEMO_SCRIPT.md` — the recommended 3-minute live capstone demo.
- `MARKET_RESEARCH.md` — sourced notes on competing products and differentiation.
- `CAPSTONE_PRESENTATION.md` — original capstone presentation outline.

**Brand principle:** *Tailor your resume, not your facts.*

## Handoff / remaining work

Done locally and verified: auth, upload/export, agent with live OpenAI, Fact Bank, credit rules (no charge on errors or ungrounded results), basic rate limiting, 26 automated tests, baseline-vs-agent eval (`eval/results.md`).

## Implementation Status & Future Improvements

### Completed Implementation

ResumeFit has been successfully implemented and deployed as a publicly accessible AI-powered resume tailoring application.

**Core Application Features**
- User registration, login, and session management.
- Resume upload, AI-powered analysis, and Word export.
- Multi-step AI agent for job requirement extraction, evidence retrieval, resume rewriting, and verification.
- Fact Bank, analysis history, and credit management.
- Automated testing and baseline-versus-agent evaluation.

**Deployment & Infrastructure**
- Deployed the FastAPI application on Render with a public HTTPS URL.
- Configured server-side OpenAI API integration and environment variables.
- Implemented persistent SQLite storage using a 1 GB Render Persistent Disk.
- Verified that user accounts, analysis history, and credits remain available after redeployment.

**Stripe Payment Integration**
- Integrated Stripe Sandbox Checkout for simulated credit purchases.
- Configured a secure webhook for `checkout.session.completed`.
- Successfully tested a $4.99 simulated purchase of 5 AI credits.
- Verified that the webhook returned HTTP 200 OK and automatically updated the user's balance from 2 to 7 credits.
- All payment testing was performed in Sandbox mode without processing real payments.

**Live Application:** https://resumefit-xrcw.onrender.com

### Testing & Evaluation

The deployed application has been manually tested for user authentication, AI-powered resume tailoring, result storage, payment processing, and database persistence.

The AI agent is designed to ground rewritten resume bullets in existing user-provided evidence rather than inventing qualifications. The repository also includes automated tests and an evaluation framework comparing the baseline model with the multi-step agent.

Additional validation is recommended for unsuccessful or canceled payments, duplicate webhook events, and the complete user journey under different input conditions.

### Limitations & Future Improvements

Although the core application is functional, several improvements could enhance its reliability, scalability, and user experience:

1. **AI Performance:** Optimize multi-step LLM calls to reduce response latency and API costs.
2. **Retrieval Quality:** Explore embedding-based semantic retrieval to complement the existing BM25 approach.
3. **Security:** Add email verification, stronger rate limiting, and additional protections for user data.
4. **Scalability:** Consider migrating from SQLite to a managed database for larger-scale deployment.
5. **Payment Reliability:** Expand testing for failed transactions, webhook retries, and duplicate event handling.
6. **User Experience:** Improve resume formatting customization and provide more detailed feedback on job requirement coverage.

### Conclusion

ResumeFit demonstrates an end-to-end AI product that combines practical business value, evidence-grounded LLM reasoning, web application development, persistent data storage, and simulated payment processing.

The deployed MVP validates the feasibility of AI-assisted resume tailoring while providing a foundation for future improvements in AI accuracy, system reliability, and production readiness.

**Security Note:** API keys and payment credentials are stored as server-side environment variables and must never be committed to the public repository. Stripe remains in Sandbox mode; additional security and operational safeguards are required before enabling real payments.:** `.env` is git-ignored; never commit it. Any API key that has been pasted into chats, screenshots or terminals should be rotated before the public deployment.
