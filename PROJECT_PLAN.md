# Scheme Saathi — Project Plan

An AI agent that tells Indians which government schemes they qualify for, in Tamil or English, by voice or text.

Goals: (1) real public deployment, (2) a portfolio project explainable line by line in interviews.

> Status: planning. Last updated 2026-11-05. Costs and library versions below are estimates; re-verify at each milestone.

---

## 1. Working agreement (how we build)

1. Build in small milestones. Each one produces something that runs.
2. For every milestone:
   - a. Concept explained in plain language first (what it is, why we need it).
   - b. Smallest piece of code, explained line by line.
   - c. A small task for the learner to write or change.
   - d. 2-3 check questions. We do not continue until they are answered.
3. Never dump a whole file. Go function by function.
4. On errors: learner reads the traceback and finds the cause first; mentor guides, does not just fix.
5. Short explanations, with everyday Indian analogies where they help.
6. End of every session:
   - What we built
   - Concepts learned (one line each)
   - One interview question, with what a good answer includes
   - Next milestone
7. If asked to "just give me the code": remind of this agreement once, then help write it.
8. Prefer free or cheap tools; state the cost of anything suggested, including cost at real-user traffic.
9. Accuracy matters: real people will rely on this.

## 2. Design principles

- **The LLM NEVER decides eligibility.** The rule-based checker is the final judge. The LLM only (a) extracts the user's profile from their words and (b) explains results.
- **Every scheme has an official source link and a "last verified" date.** Every answer shows both, plus a note to confirm on the official site.
- **Privacy by default.** Do not store sensitive data (caste, income, gender, etc.) unless there is a clear reason. Designed in from M3 onward. User profiles live only in memory during the chat.
- **Never invent eligibility rules.** If a rule is unclear in the official source, say so (`rule_status: "unclear"`) instead of guessing.

## 3. Learner profile

- Fresh CS graduate, basic Python. New to LLMs, APIs, RAG, agents, deployment.
- Past habit to avoid: copy-pasting code without understanding it.
- Libraries are welcome everywhere they save time (including LangChain), except the M2 eligibility checker, which stays plain Python so it is auditable and testable.

## 4. Environment (checked 2026-11-05)

| Item | Value |
|---|---|
| OS | Windows 11 |
| Python | 3.14.3 installed; **plan to use 3.12 for this project** (some libraries lag on the newest Python) |
| pip | 26.1.2 |
| Git | 2.55.0, user.name `siddharth` |
| Node | v24 (not needed until, at most, M8) |
| Editor | VS Code |
| Project folder | `D:\schemeproject` (empty at start) |

## 5. Tech stack

| Layer | Choice | Introduced | Notes |
|---|---|---|---|
| Language | Python 3.12 | M0 | Install alongside 3.14 via `winget install Python.Python.3.12`; use `py -3.12` |
| Environment | `venv` | M0 | One isolated library box per project |
| Version control | Git + GitHub | M0 | History, backup, portfolio |
| Secrets | `.env` + `python-dotenv` | M0 | Never commit keys; `.gitignore` the `.env` |
| Data format | JSON | M1 | Scheme database lives in the Git repo |
| Data validation | Pydantic | M1 | Rejects malformed scheme/profile data |
| Eligibility checker | Plain Python | M2 | No AI, no LangChain |
| Testing | pytest | M2 | |
| LLM API | Claude Haiku or Gemini Flash (decide in M3) | M3 | Check current pricing and data-retention/training terms first |
| LLM framework | LangChain (structured output, loaders, splitters) | M3-M4 | First make one plain API call to see what is underneath |
| Agent framework | LangGraph | M5 | Follow-up question loop |
| Embeddings | `sentence-transformers` (multilingual, local) | M4 | Free |
| Vector DB | ChromaDB (local) | M4 | |
| Tamil | LLM translation; compare a dedicated Indian-language model | M6 | |
| Speech-to-text | Whisper | M7 | Local or API; decide by hardware and cost |
| Text-to-speech | Edge-TTS or gTTS | M7 | Free at demo scale; check caveats (unofficial APIs, Tamil voice quality) |
| Frontend | Streamlit | M8 | Python-only; chat box, mic button, language switch |
| Backend | FastAPI (`/chat`, `/feedback`) | M8 | Separate from frontend so rate limiting/auth is possible. Fallback: start Streamlit-only, split later |
| Feedback DB | SQLite (built into Python) | M9 | Stores question text, scheme ID, user's note — never the profile |
| Hosting | Hugging Face Spaces or Streamlit Community Cloud | M9 | Free tiers; apps may sleep when idle; verify limits |
| Scheduled jobs | GitHub Actions | M10 | |
| HTTP / scraping | `requests` (plus a parser if needed) | M10 | |

### Data storage summary

- **Schemes:** JSON files in the repo (versioned, reviewable, ~20 schemes).
- **Search index:** ChromaDB, rebuilt from the JSON/source text.
- **User profiles:** not stored at all.
- **Feedback:** SQLite, minimal fields only.

### "Upload" summary

- **User uploads:** only microphone recording or an audio file for voice (M7). We do NOT collect documents like Aadhaar.
- **Deployment:** push code to GitHub, connect to the host; API keys set in the host's secret settings, never in the repo.

## 6. Architecture (target)

```
User (text / voice, Tamil / English)
        |
   Streamlit UI  ──►  FastAPI backend
                           |
        ┌──────────────────┼───────────────────┐
        ▼                  ▼                   ▼
  LLM: extract        Rule-based           RAG: retrieve scheme
  profile from        eligibility          text + sources
  free text           checker (FINAL       (ChromaDB)
  (Pydantic)          JUDGE, plain Python)
        |                  |                   |
        └──────► LLM: explain results in user's language ◄──┘
                           |
        Answer + official link + last-verified date + "confirm on official site"
```

Agent loop (M5): if profile fields needed by the checker are missing, ask a follow-up question instead of guessing.

## 7. Milestones

Legend: `[ ]` not started, `[~]` in progress, `[x]` done. **M = Milestone.**

### M0 — Setup `[~]`
- Install Python 3.12; create venv in `D:\schemeproject`
- Git init, `.gitignore`, GitHub repo, first commit and push
- `.env` + `.env.example`, load with `python-dotenv`
- Project structure (planned): `data/`, `src/`, `tests/`, `docs/`, `PROJECT_PLAN.md`, `README.md`
- **Concepts:** terminal, venv, pip, Git commit/push/branch, `.gitignore`, secrets management, project layout
- **Check questions asked:** why each project gets its own venv; what goes wrong if an API key is pushed to GitHub
- **Open questions to the learner:** GitHub username; Python 3.12 OK; any LLM API key yet; Git/terminal experience
- **Cost:** free

### M1 — Scheme data `[ ]`
- Collect ~20 real schemes (central + Tamil Nadu) into JSON
- Per scheme: id, name, level (central/state), benefit summary, eligibility rules (age, income, state, caste category, gender, occupation), required documents (if officially listed), official source link, last-verified date, per-rule `rule_status` (clear / unclear)
- Pydantic model to validate every scheme file
- **Concepts:** JSON, Pydantic, reading official sources, data provenance, handling unclear rules honestly
- **Cost:** free (learner's time reading official sites)

### M1.5 — Scheme sources review `[ ]` (added to roadmap)
- Why added: agree how unclear/ambiguous rules are recorded *before* any rule is trusted. Supports the "never guess" principle.
- Review each scheme's rules against its official source; mark unclear ones; record verification date

### M2 — Rule-based eligibility checker `[ ]`
- Pure Python, no AI, no LangChain
- Input: user profile; output: matching schemes with reasons per rule, and "cannot determine" where profile data is missing or rule is unclear
- Pytest suite: boundary values (e.g. exactly age 18), missing fields, unclear rules
- **Concepts:** functions, dicts, rule engines, edge cases, testing, why tests matter when people rely on you

### M3 — LLM profile extraction `[ ]`
- Start with one plain API call (~10 lines), then move to LangChain structured output
- Free text → structured profile (Pydantic), no sensitive data stored
- Choose provider: pricing, data-retention/training terms (free tiers may use inputs for training — matters because users describe income/caste)
- **Concepts:** LLMs and their limits, APIs, tokens, cost, prompts, structured output, hallucination, privacy
- **Cost:** low (fractions of a rupee per chat; confirm with live pricing)

### M4 — RAG `[ ]`
- Embed scheme documents (multilingual local model), store in ChromaDB, retrieve relevant ones, answer "why do I qualify" with sources
- LangChain loaders, splitters, Chroma integration
- **Concepts:** embeddings, vector DBs, similarity search, chunking, grounding and citations, evaluating retrieval

### M5 — Agent loop `[ ]`
- LangGraph flow: detect missing profile fields → ask follow-up → re-run checker
- Guardrails so the agent stays on-task
- **Concepts:** tool calling, state, loops, graphs, guardrails

### M6 — Tamil support `[ ]`
- Tamil input and output; translation quality testing; Tamil embeddings
- Rule checker stays language-independent
- **Concepts:** Unicode/Tamil text, multilingual models, translation evaluation

### M7 — Voice `[ ]`
- Speech-to-text (Whisper) in, text-to-speech out (Tamil + English)
- **Concepts:** audio formats, ASR/TTS, noise, latency
- **Check:** hardware suitability, Tamil voice quality, TTS service terms

### M8 — UI and backend `[ ]`
- FastAPI backend (`/chat`, `/feedback`); Streamlit frontend with chat, mic, language toggle
- **Concepts:** REST APIs, routes, request/response models, session state, frontend/backend separation

### M9 — Deploy and safety `[ ]`
Roadmap change: split into two sessions.
- **M9a Deploy:** host on free tier, secrets via host settings, README with demo GIF
- **M9b Safety:** API spending cap, rate limiting, "this was wrong" feedback button (SQLite), logging without sensitive data
- **Concepts:** hosting, env vars, rate limiting, cost control, privacy-safe logging, documentation
- **Cost:** free hosting; API spend capped by us

### M10 — Maintenance `[ ]`
- Scheduled routine (GitHub Actions) to re-check scheme source pages, detect changes, flag rules for human review (e.g. after each budget)
- Update last-verified dates; version rule changes
- **Concepts:** scheduled jobs, change detection, keeping data fresh

## 8. Cost summary (estimates — verify live)

| Item | Estimate |
|---|---|
| Hosting (HF Spaces / Streamlit Cloud) | Free tier |
| Embeddings, ChromaDB, Whisper (local), SQLite | Free |
| LLM API | Low; roughly low hundreds of rupees/month at ~1,000 conversations on a cheap model |
| Safety net | Spending cap on API key (M9b) so spikes cannot become surprise bills |

## 9. Risks and cautions

- **Python 3.14 too new** for some libraries → use 3.12.
- **LangChain changes its API often**; many tutorials are outdated → check current docs before each LangChain milestone, read tracebacks together.
- **LangChain abstraction layer** makes debugging harder → plain API call first in M3.
- **Free LLM tiers may train on inputs** → affects provider choice in M3.
- **Free hosts sleep idle apps**; free-tier limits can change → re-verify in M9.
- **Whisper/Docker/hardware fit** unverified → check at M7 / M9.
- **Wrong eligibility info harms real people** → rule checker is final judge, every answer shows source + last-verified date + "confirm on the official site".
- **Scheme rules change** (budgets, GOs) → M10 maintenance routine.

## 10. Skills that run through every milestone

- Reading tracebacks and debugging
- Cost awareness (what each API call costs)
- Responsible AI: accuracy, privacy, honesty about uncertainty
- Explaining your work (one interview question per session)

## 11. Interview pitch (target)

"I built an LLM-powered agent with a RAG pipeline, structured extraction, rule-based safety checks, multilingual voice support, and a deployed FastAPI service with rate limiting and privacy-by-design."

## 12. Session log

| Date | Milestone | Built | Interview question |
|---|---|---|---|
| 2026-11-05 | M0 (started) | Planning, stack, this document | — |

## 13. Glossary

- **Milestone (M):** one small chunk of the project that ends with something runnable.
- **venv:** isolated folder of libraries for one project.
- **LLM:** large language model (e.g. Claude, Gemini).
- **RAG:** retrieval-augmented generation — fetch relevant documents first, then have the LLM answer from them.
- **Embedding:** text turned into numbers that capture meaning, so similar texts are close together.
- **Vector DB:** database that finds the nearest embeddings.
- **Agent:** an LLM-driven loop that can call tools and decide the next step.
- **Pydantic:** library that defines the exact shape of data and rejects bad data.
- **Hallucination:** the LLM stating something plausible but false.
- **Rate limiting:** capping how often one user can call the service.
