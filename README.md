# ORION

**ORION is a local-first AI assistant and knowledge engineering system.**

The project combines a personal AI assistant with a structured knowledge acquisition pipeline. Its goal is to move beyond simple LLM conversations toward organized knowledge, evidence tracking, learning plans, source discovery, and document processing.

## Core Architecture

ORION's knowledge pipeline consists of five implemented development components. The five stages have been implemented as individual components; full end-to-end integration and cross-platform validation remain in progress.

| Stage | Component | Responsibility |
|---|---|---|
| L1 | Knowledge Base | Knowledge storage, claims, sources, provenance and validation history |
| L2 | Knowledge Gap Analyzer | Identifies and tracks gaps in existing knowledge |
| L3 | Learning Planner | Creates explainable, versioned learning plans and tasks |
| L4 | Source Discovery Engine | Discovers, ranks and deduplicates relevant sources |
| L5 | Document Ingestion | Processes documents, extracts text, creates chunks and manages versions |

## Implemented Capabilities

### Knowledge Management

- Structured knowledge storage with source provenance.
- Knowledge-gap classification and lifecycle tracking.
- Evidence-aware learning plan generation.
- Versioned learning plans and task dependencies.

### Source Discovery

- Source discovery integrations for Brave Search, Crossref, GitHub and the local knowledge base.
- Source deduplication and task-scoped ranking.
- URL validation and SSRF protections.

### Document Processing

- Document ingestion with SHA-256 identification.
- Duplicate document detection and content versioning.
- Text extraction from supported document formats.
- Structured document chunking with stable identifiers.

### Assistant

- Local assistant backend and browser-based frontend.
- Voice input and text-to-speech components.
- Controlled remote system actions, such as launching and closing allowlisted applications, with authorization checks.

## Technology Stack

- Python 3.11+
- SQLite
- Ollama
- `qwen3.5:9b` (local model)
- HTML, CSS and JavaScript
- pytest

The local macOS environment used for the November 6 verification:

- Python 3.12.16
- Ollama model: `qwen3.5:9b`

## Testing

Latest recorded test run on November 6, 2026:

- 326 passed
- 3 failed
- 1 skipped
- 122 subtests passed

The three failures occurred in Windows-specific system privilege tests because `ctypes.WinDLL` is unavailable in the macOS test environment.

These results are reported as observed; they are not presented as a fully passing cross-platform test suite.

## Run on macOS

**Setup limitation:** The macOS launcher is not yet a clean-install bootstrap script. It requires an existing, isolated data directory containing a readable and writable `jarvis.sqlite3` database.

No verified database initialization command is currently documented. A reproducible database initialization procedure or setup script is still needed before a fresh clone can be started using this guide alone.

Once the data directory and database have been prepared, the launcher can be started as follows:

```bash
export ORION_DATA_DIR="/absolute/path/to/orion-data"
./scripts/run-macos.sh
```

The launcher checks that:

- The data directory exists and is separate from the project's default data directory.
- The database is a readable and writable regular file.
- Ollama is available and responding.
- The required local model `qwen3.5:9b` is installed.

The launcher does not automatically download the model or create the database.

The database filename `jarvis.sqlite3` is a historical name retained for compatibility.

## Current Limitations

- The latest recorded macOS test run contains three platform-specific failures.
- Full end-to-end behavior across every component requires further validation.
- Retrieval quality over ingested chunks has not yet been established as a completed capability.
- Source discovery and document ingestion do not, by themselves, constitute independent verification of every source claim.
- Exact LLM usage by each individual pipeline stage requires further architectural documentation.

## Roadmap

- [ ] Validate complete L1–L5 workflows.
- [ ] Develop and evaluate retrieval over ingested chunks.
- [ ] Improve source reliability assessment.
- [ ] Expand cross-platform automated testing.
- [ ] Document LLM responsibilities and decision boundaries.
- [ ] Add a reproducible database initialization and clean-install procedure.

## Project Status

L1–L5 are implemented as individual development components. Their complete end-to-end integration remains under validation.

See [Progress Report — November 6, 2026](PROGRESS_2026-11-06.md) for the milestone details.
