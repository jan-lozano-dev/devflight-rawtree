# DevFlight: observability for AI-assisted development, on RawTree

Datathon FME 2026 (UPC), Tinybird side challenge: *Best agentic use of RawTree*. DevFlight is a local platform that collects telemetry from the AI coding agents a team uses (for example Codex and Claude Code) and from a small math copilot, stores it in [RawTree](https://rawtree.com), and shows it in a dashboard with a conversational **AI Analyst**. We built it to watch ourselves build the Qualcomm main challenge.

## How it works

- **RawTree** keeps the analytic events.
- **FastAPI** applies permissions and builds the SQL. Keys for RawTree and the model provider stay in the backend.
- **React** shows the dashboard and the analyst chat. The compiled frontend is included; Node is only needed to change it.
- **SQLite** stores identities, analyst jobs, private views and a durable outbox, so events are accepted locally before they are written remotely.

## What it does

- **Global dashboard:** activity, errors, emitters, sessions, reported token use and source status, over every compatible table of the authorised RawTree source.
- **Ingestion:** OTLP/HTTP JSON (logs, traces, metrics), flat events and the JavaScript SDK envelope, with retry-safe IDs.
- **AI Analyst:** a conversation that chooses structured RawTree queries, keeps its progress, shows the SQL and evidence, and can turn a finding into a saved follow-up view that is checked by a verification query.
- **Experiments tab:** reads the `eval.grade` events of the Qualcomm project and breaks accuracy down by language, topic and level. It only calls two runs comparable when dataset, split, suite version and hardware match.
- **Explorer:** a virtual table over all events, filters, grouping, CSV export and private views.

## Quick start

Python 3.11 or later.

```sh
python3 -m venv .venv
.venv/bin/python -m pip install ".[platform]"
cp .env.example .env            # then fill in the RawTree and Helmcode keys
.venv/bin/python start_platform.py start
```

On Windows, use `py -3 -m venv .venv`, `.venv\Scripts\python.exe -m pip install ".[platform]"` and `devflight.cmd start`.

Open <http://127.0.0.1:8788>. The first start writes the admin credential to `.devflight/platform/admin.token`; paste it into the form and do not share the file. `status`, `restart` and `stop` manage the services. The app is served in Spanish.

To capture telemetry from a coding agent, see [docs/TELEMETRY_QUICKSTART.md](docs/TELEMETRY_QUICKSTART.md). Exporters send `Authorization: Bearer <credential>` to `POST /api/events`, `/v1/logs`, `/v1/traces` or `/v1/metrics`.

## Tests

```sh
python3 -m unittest discover -s tests -q      # 85 tests, 2 skipped; passes on Linux, Python 3.12
npm --prefix frontend ci && npm --prefix frontend run build
```

Tests with real RawTree and Helmcode accounts are not automated; `python scripts/platform_demo.py --live` runs a real investigation once the keys are set.

## Limits to know

- Rows are events, not necessarily model requests. Token counts are the ones each provider reports; a missing value stays `null` and is never turned into zero cost.
- A shared API key does not identify all of its users: events without an origin ID show as unknown.
- Delivery is "at least once with deduplication", not exactly once. Local capacity is capped (512 MiB by default).
- The AI Analyst needs a provider that serves the configured model (`HELMCODE_MODEL`); *Check connection* only reads the provider's catalogue.

More detail in [docs/reference.md](docs/reference.md).

## Repository

| Path | What |
|---|---|
| `devflight/` | Python package: collector, normalisation, RawTree client, analytics, AI Analyst, server |
| `frontend/`, `devflight/web/` | React and TypeScript source, and its compiled build |
| `sdk/` | JavaScript SDK |
| `copilot/`, `board/`, `predict.py`, `eval.py` | A small instrumented math copilot that emits the events DevFlight shows |
| `scripts/`, `tests/` | Demo and verification scripts, and the unit tests |
| `docs/` | Documentation ([index](docs/README.md)) |

## License

MIT, see [LICENSE](LICENSE).
