# super-ollama

Terminal-native, in-process local LLM engine: `llama.cpp` embedded in the CLI (no HTTP server in the main UX), focused on low overhead and stable resource teardown.

## Links

- **Repository:** [github.com/Inference-Foundry/super-ollama](https://github.com/Inference-Foundry/super-ollama)
- **Roadmap (wiki):** [Roadmap](https://github.com/Inference-Foundry/super-ollama/wiki/Roadmap)
- **Work tracking:** issues in the repo; the wiki notes [GitHub Issues with labels by phase](https://github.com/Inference-Foundry/super-ollama/wiki/Roadmap) and the plan file `super-ollama-agent-plan.md` at the repo root as the authoritative breakdown.

## Plan (summary)

**Done (Phase 1):** Removed desktop `app/`; `internal/engine` wrapping server handlers in-process; `cmd/super-ollama` with `ask`, `chat`, `config`, hidden `runner`; config for `default_model` and quieter default logging.

**Later phases (planned):**

| Phase | Theme |
| --- | --- |
| 2 | SQLite + vector store, context injection, `learn` |
| 3 | Screenshot capture, OCR, indexing, `snap`, privacy filters |
| 4 | Email assistant (IMAP/SMTP), keyring |
| 5 | Markdown TODO manager + AI triage |
| 6 | `~/.super-ollama/` prompts/profile layout, install script, polish |

**Non-goals (from roadmap):** No bundled HTTP API in the super-ollama UX (inference stays in-process); no first-party GUI in this repository; no cloud sync or multi-user in scope.

## How to help

Read the repository [contributing guide](https://github.com/Inference-Foundry/super-ollama/blob/main/CONTRIBUTING.md) and [security policy](https://github.com/Inference-Foundry/super-ollama/blob/main/SECURITY.md). Architectural changes expect benchmarks and profiling data.
