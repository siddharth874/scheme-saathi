# Scheme Saathi

An AI agent that helps people in India find which government schemes they may qualify for, in Tamil or English, by text or voice.

> **Status: early development.** Nothing here is ready for real use yet. Always confirm scheme details on the official website.

## How it will work

- A rule-based checker decides eligibility. The AI never does.
- The AI only reads what the user says and explains the result.
- Every scheme shows its official source link and the date it was last verified.
- Sensitive details (income, caste, gender) are not stored.

## Setup

```powershell
python3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

Copy `.env.example` to `.env` and fill in your own keys. Never commit `.env`.

## Plan

See [PROJECT_PLAN.md](PROJECT_PLAN.md) for the full roadmap.
