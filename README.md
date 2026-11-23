# PLE Agent Harness — plan, verify, then deliver

An agent harness for the Kaggle competition **Google — Agent Harness on
Physical-Engineering's Last Exam** ($400k). It receives an engineering task
over WebSocket, drives an LLM with tools (bash, read_file, write_file,
edit_file, plus the task's own MCP servers for documents, spreadsheets, email
and CAD), and produces checked deliverable files.

Built on the competition's `local-model` starter, with four additions:

1. **Plan phase** — the first model call is a read-only planner (10% of the
   time budget) that writes `plan.md`: exact deliverable paths and formats,
   method, standards and clauses to check, and verification steps.
2. **Deterministic deliverable gate** — before the agent may finish, every
   planned deliverable is checked in the container (exists, non-empty, opens
   in its claimed format). A missing deliverable forces another tool round
   instead of an empty `done`.
3. **Verification pass** — a fresh-conversation verifier with read-only tools
   re-opens every deliverable, independently recomputes the key numbers with
   a fresh script, and checks the units. It replies `PASS` or a numbered gap
   list; up to 2 fix rounds, only while at least 25% of the time remains.
4. **Final-message check + cost log** — the closing summary must list
   deliverable paths, results with units, and assumptions/limitations, and
   every turn emits a `PLEBENCH_LOG {...}` JSON line to stderr (tokens, wall
   time, tool/MCP counts) for score-vs-cost analysis.

## Status

- `kit validate` passes. Smoke-tested without Docker against a mock
  evaluator (stub model endpoint + mock task container): 10/10 assertions
  pass — the loop reaches `done`, the verifier recomputes independently and
  replies `PASS`, and the final-message check fires as designed.
- Real dev-set evaluation (29 tasks) is not run yet: it needs a Docker
  machine (~16 GB RAM, 6 CPU per slot).

## How to run

Prerequisites: the PLE starter kit (`google-research/ple`), `uv`, Docker, a
Hugging Face token for the gated dataset
(`google/physical-engineerings-last-exam`), and a model API key.

```bash
uv run python -m kit validate ../my-harness
uv run python eval/run.py ../my-harness --tasks <subset>
```

Development model: Gemini through its OpenAI-compatible endpoint
(`https://generativelanguage.googleapis.com/v1beta/openai/`). For Stage 2,
set `model_api` in `submission.yaml` to a sponsored model such as
`google-cloud/gemini-3.8-flash` — switching after local generation does not
disqualify, per the official competition rules.

## Files

| File | Purpose |
|---|---|
| `serve.py` | The harness: WebSocket server + agent loop |
| `taskbridge.py` | Session, MCP client and WebSocket plumbing |
| `submission.yaml` | Model, install and serve configuration |
| `host/` | Optional auth gate for a self-hosted model |

## License

Apache License 2.0 — see `LICENSE`.
