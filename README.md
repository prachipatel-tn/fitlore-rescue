# FitLore Rescue submission

This repository contains the runnable source archive, project report, and recording guide.

1. Download `FitLore-Rescue-Source.zip` and extract it into a new folder.
2. Open the extracted folder in a terminal and follow the setup instructions below.
3. The archive includes the Python app, browser interface, tests, licensed images, and live evaluation evidence. No credentials or runtime databases are included.

[Native Google Doc report](https://docs.google.com/document/d/1unQyGjH-shQhOp4D55r3fNvyWaWiowYcRQ2m696kLkY/edit)

The report and recording guide are also available at the repository root. The demonstration video is still to be recorded.

# FitLore Rescue

A local shoot-recovery agent that investigates an unavailable hero jacket, checks replacement garments, and pauses for human approval before updating affected looks.

Built on FitLore's fashion-production concept and visual identity. Week 3 adds a new model-selected tool loop, durable LangGraph checkpoints, bounded recovery, and an idempotent approval transaction. This is a focused companion prototype, not a migration of the Week 2 database.

## Run

Python 3.12 recommended.

```sh
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
cp .env.example .env
# Add your Nebius key and model to .env. Never commit .env.
FITLORE_ALLOW_MODEL=true .venv/bin/python -m uvicorn app.main:app --host 127.0.0.1 --port 8033
```

Open http://127.0.0.1:8033. Select **Rehearsal · no AI** for deterministic offline testing. Live mode requires Nebius and uses a durable 20-attempt ceiling per runtime directory. Attempts count even when requests fail; they do not represent dollars. The application does not automatically reset this allowance.

Alternatively, set `FITLORE_CREDENTIALS_FILE` to an existing dotenv file; only NEBIUS_API_KEY, NEBIUS_MODEL and NEBIUS_BASE_URL are read from it. Credentials remain server-side.

## Agent architecture

```mermaid
flowchart TD
    User[Stylist request] --> Decide[Model selects one tool]
    Decide --> Tools[Read brief / inspect impact / search wardrobe / check replacement]
    Tools --> Decide
    Decide --> Proposal[Validated proposal]
    Proposal --> Approval[Persistent human approval interrupt]
    Approval -->|Approve| Recheck[Recheck revision and constraints]
    Recheck --> Commit[Atomic idempotent plan update]
    Approval -->|Reject| Unchanged[Keep plan unchanged]
    Decide -->|No safe option / failure / limit| Handoff[Stylist handoff]
    DB[(SQLite checkpoints and workspace)] --- Decide
    DB --- Approval
    DB --- Commit
```

The model outputs a schema-constrained action. Python dispatches only allowlisted tools; the model cannot execute code or directly write the project. `check_replacement` enforces category, availability, color, recorded size, sleeve coverage, ready time, and replacement cost. A proposal requires successful reads and a successful candidate check. Approval uses LangGraph `interrupt()` and `Command(resume=...)`.

`app/agent.py` holds the model prompt, tools and graph. `app/store.py` holds the durable workspace, event log, model attempt ledger and transaction. `app/fixtures.py` defines fictional data. `app/main.py` exposes the local UI and API. `web/` contains the interface and licensed category photography inherited from FitLore.

## Validation

```sh
.venv/bin/python -m pytest -q
```

Automated tests use an explicitly deterministic decision driver. They validate control flow and integrity; they are not live model quality evaluations. Live evidence is exported separately in `output/submission/live-evidence.json`.

Two real Nebius runs were inspected: transient inventory failure recovered to an approved plan; no qualifying garment led to handoff with no write. The model made unnecessary reads and initially selected invalid candidates in the no-match case. Constraint checks rejected them. This small sample is not a general success-rate claim.

## Scope

- Fictional local inventory, eight garments, two looks, two downstream review tasks. Each new investigation gets an independent scenario copy.
- Single operator on localhost; no multi-user authentication or remote deployment.
- No retailer search, real stock verification, bookings, payments, messages, vision analysis or physical fit guarantee.
- Approval changes look assignments only. Stylist fit and photographer approval remain open.
- Memory means durable run state and audit history, not learned preferences or cross-project retrieval.
- The fixed rescue workflow is not a general styling assistant. The request text guides investigation within that scope.
- A maximum of ten model-selected steps per run and two inventory failures prevents endless loops. Model/network failures hand off without a project update.

## References

- [LangGraph interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts)
- [LangGraph persistence](https://docs.langchain.com/oss/python/langgraph/persistence)
- Photography attribution: `web/images/credits.json` and `/static/credits.html`.

## Submission

The local report and recording guide are in `output/submission/`. Import the report into Google Docs and verify reviewer access. The final video must show a live application run and stay under five minutes. Publish the sanitized source package to the intended GitHub repository; no credentials or runtime databases belong in that repository.
