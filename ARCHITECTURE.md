# ORION — System Architecture

## 1. Overview

ORION is a local-first AI assistant and knowledge engineering system designed to acquire, organize, analyze, and use knowledge through a structured pipeline.

The architecture separates language-model interaction, knowledge management, learning planning, source discovery, document processing, and assistant functionality into distinct components.

The Python package and database retain the historical name JARVIS for compatibility. The Python package is located in `jarvis/`, and the database filename is `jarvis.sqlite3`.

The primary architectural goal is to build a system that can distinguish between:

- Known information.
- Missing information.
- Information supported by sources.
- Information that requires further verification.
- Information that remains unknown.

ORION is built around explicit data structures, traceable information sources, and independently testable components rather than relying exclusively on information stored inside an LLM.

## 2. Architectural Principles

### 2.1 Local-First Design

ORION is designed to operate in a locally controlled environment.

Core data, knowledge storage, document processing, and model interaction are intended to remain under the user's control.

External services are used where required for specific capabilities, such as source discovery.

### 2.2 Modular Architecture

Each major subsystem has a defined responsibility.

Components are independently testable and maintain clear boundaries. Changes to one subsystem should not require unnecessary modifications to unrelated components.

### 2.3 Traceable Knowledge

Knowledge is not intended to be stored as isolated statements without context.

Where applicable, information retains relationships to:

- Its source.
- Its original document.
- Its associated learning task.
- Its processing history.
- Its validation status.

### 2.4 Separation of Knowledge and Language Models

The LLM is responsible for language understanding, reasoning assistance, and generation.

It is not the authoritative storage layer for ORION's knowledge.

Persistent information belongs in external storage, while retrieval and verification mechanisms provide relevant context to the model.

### 2.5 Verifiability

ORION distinguishes between generating an answer and establishing that the answer is supported by evidence.

The presence of a source does not automatically establish the reliability of its contents.

## 3. High-Level Architecture

The system is organized into logical layers.

| Layer | Responsibility | Status |
|---|---|---|
| User Interface | User interaction and presentation | Implemented |
| Assistant Layer | Request handling and coordination | Implemented |
| Knowledge Processing | Knowledge management and gap analysis | L1–L2 implemented |
| Learning Pipeline | Learning plans, tasks, and workflow coordination | L3 implemented |
| Source Discovery | External source discovery and ranking | L4 implemented |
| Document Processing | Document ingestion, extraction, versioning, and chunking | L5 implemented |
| Storage Layer | Persistent knowledge and document data | Implemented components |
| Model Runtime | Local LLM inference | Used in development environment |
| Verification Layer | Evidence evaluation and answer validation | Further implementation required |

These statuses distinguish implemented components from capabilities that still require integration, evaluation, or additional development.

## 4. Knowledge Acquisition Pipeline

The central development architecture is a five-stage pipeline.

```text
L1 → L2 → L3 → L4 → L5
```

Each stage has an independent responsibility.

### 4.1 L1 — Knowledge Base

**Purpose:** Structured knowledge management.

The Knowledge Base provides the foundation for storing and organizing information.

Its responsibilities include:

- Knowledge storage.
- Claims and structured information.
- Source relationships.
- Provenance tracking.
- Validation history.

The Knowledge Base provides persistent information that can be used by other ORION components.

### 4.2 L2 — Knowledge Gap Analyzer

**Purpose:** Identify missing knowledge.

The Knowledge Gap Analyzer evaluates knowledge in relation to a learning goal.

Its responsibilities include:

- Identifying missing information.
- Tracking knowledge gaps.
- Maintaining the lifecycle of identified gaps.
- Providing information that guides learning-plan creation.

The objective is to identify what must be learned rather than simply generating an unrestricted list of topics.

### 4.3 L3 — Learning Planner

**Purpose:** Convert learning goals and knowledge gaps into structured plans.

The Learning Planner generates learning plans and tasks.

Its capabilities include:

- Structured learning plans.
- Task dependencies.
- Plan versioning.
- Explainable planning.

Plans provide a structured path from a learning goal to the knowledge required to achieve it.

### 4.4 L4 — Source Discovery Engine

**Purpose:** Find candidate sources for learning tasks.

The Source Discovery Engine searches for information relevant to specific learning tasks.

Implemented provider integrations include:

- Brave Search.
- Crossref.
- GitHub.

Additional capabilities include:

- Provider-specific result parsing.
- URL normalization.
- Task-scoped ranking.
- Deduplication.
- SSRF and redirect protections (implemented; code-level verification pending).

Discovery results are associated with their corresponding learning tasks.

A discovered source is a candidate for further processing, not automatically verified evidence.

### 4.5 L5 — Document Ingestion

**Purpose:** Convert discovered documents into structured, persistent data.

The ingestion service accepts discovery results associated with an actual task and a specific learning-plan version.

It uses the existing `DocumentStoreService`.

Implemented capabilities include:

- SHA-256 identification of original documents.
- Deduplication of unchanged document content.
- Version creation when document content changes.
- Separate storage of original documents, extracted text, and document chunks.

The ingestion stage preserves the relationship between acquired documents and the learning workflow that requested them.

## 5. Data Flow

The intended knowledge acquisition workflow is:

1. A learning goal is evaluated against available knowledge.
2. The Knowledge Base provides relevant knowledge signals.
3. The Knowledge Gap Analyzer identifies missing information.
4. The Learning Planner creates a structured plan.
5. Learning tasks are passed to the Source Discovery Engine.
6. Candidate sources are discovered and associated with their tasks.
7. Discovery results are selected and passed to Document Ingestion. The ingestion service accepts results associated with a real task and a specific plan version. The complete automatic handoff across L1–L5 has not yet been validated.
8. Documents are identified, extracted, deduplicated, and versioned.
9. Extracted text and chunks are stored for subsequent knowledge processing.

The full L1–L5 workflow remains under integration and validation.

A tested integration path currently connects goal evaluation, Knowledge Base signals, and learning-plan formation. This does not establish complete end-to-end validation of all five stages.

## 6. Storage Architecture

### 6.1 SQLite

SQLite is used as the local database technology.

It provides persistent storage for structured application data.

The database filename `jarvis.sqlite3` retains its historical name for compatibility, as described in the Overview.

### 6.2 Document Storage

Document storage is handled through `DocumentStoreService`.

The document-processing architecture distinguishes between:

- Original document files.
- Extracted text.
- Document chunks.
- Document identity and version information.

SHA-256 identification supports recognition of unchanged document content.

When content changes, a new document version is created rather than treating the changed document as identical to the previous one.

### 6.3 Persistent Knowledge

The Knowledge Base is separate from the LLM's internal model weights.

This allows knowledge to be stored, updated, traced, and evaluated independently of model replacement or adaptation.

## 7. LLM and Model Runtime

ORION uses a local model runtime through Ollama.

The recorded development environment includes:

| Component | Configuration |
|---|---|
| Runtime | Ollama 0.40.14 |
| Installed model | `qwen3.5:9b` |
| Database | SQLite |
| Python | 3.12.16 |

The installed model is part of the current development environment.

The exact responsibilities of LLM inference across individual pipeline stages have not yet been fully documented.

### 7.1 Intended Model Responsibilities

The architecture separates model-assisted operations from deterministic application logic.

The LLM may assist with:

- Understanding user goals.
- Interpreting knowledge requirements.
- Explaining knowledge gaps.
- Generating learning plans.
- Analyzing retrieved information.
- Producing natural-language responses.

Structured storage, document identification, deduplication, versioning, and other application-level operations remain controlled by their respective services.

### 7.2 Model Adaptation

Model specialization is a later development objective.

LoRA is considered a potential method for adapting model behavior to ORION's requirements.

Full model training is not a current implementation requirement.

The priority is to establish a reliable architecture around an existing model before introducing additional training complexity.

## 8. Retrieval and RAG Architecture

Retrieval-Augmented Generation is a planned core capability.

Its purpose is to provide the LLM with relevant information from ORION's external knowledge and document storage.

**Current status:**

- Document ingestion and chunk storage are implemented.
- Retrieval over ingested document chunks has not yet been fully implemented and evaluated.
- Evidence-grounded answer generation remains a development objective.

The intended retrieval workflow is:

1. Receive a user question or learning task.
2. Identify the information required.
3. Retrieve relevant knowledge and document content.
4. Preserve source and provenance information.
5. Provide the retrieved context to the LLM.
6. Generate a response grounded in the available information.
7. Evaluate whether the available evidence supports the response.

RAG is an active development objective, not a completed capability.

## 9. Verification and Evidence Evaluation

A central architectural objective is to make ORION capable of evaluating the reliability of information.

**Current status:**

- Knowledge storage includes source relationships and validation history.
- Source discovery provides candidate sources.
- Complete independent evidence evaluation has not yet been established.
- A fully validated verification pipeline remains under development.

The system is intended to distinguish between:

- Confirmed information.
- Refuted information.
- Unverified information.
- Unknown information.

These distinctions should not be determined solely by the LLM's confidence or the fluency of its response.

### 9.1 Evidence-Based Evaluation

The planned verification process considers:

- Whether a claim has supporting sources.
- Whether the source is relevant to the claim.
- Whether the available evidence actually supports the statement.
- Whether contradictory evidence exists.
- Whether the available information is sufficient to reach a conclusion.

### 9.2 Separation of Generation and Verification

Answer generation and evidence evaluation remain logically separate architectural responsibilities.

The LLM may propose an answer, but the verification process must evaluate the supporting information independently.

A response should not be treated as verified merely because the model generated it confidently.

## 10. Assistant Layer

The assistant layer coordinates interaction between the user and ORION's internal components.

The Python package retains the historical `jarvis/` name, as explained in the Overview.

Existing project modules include:

- `jarvis/assistant.py`
- `jarvis/server.py`

The project also contains a frontend implemented with HTML, CSS, and JavaScript.

The assistant layer handles application interaction and coordinates available functionality.

The exact internal routing and orchestration responsibilities of every component require further documentation.

## 11. Voice Architecture

The project contains separate speech and text-to-speech modules:

- `jarvis/speech.py`
- `jarvis/tts.py`

Voice input and TTS are implemented capabilities.

Further code-level verification is required to establish their complete behavior, reliability, and platform compatibility.

Voice functionality is part of the assistant interface rather than the knowledge storage architecture.

## 12. Security Boundaries

Security is an architectural requirement, particularly when ORION interacts with external sources or performs remote system actions.

Relevant areas include:

- URL validation.
- SSRF protections.
- Redirect handling.
- Authorization checks.
- Restrictions on remote actions.

These protections are implemented, but their complete security properties have not yet been independently established.

Further code-level verification and testing are required.

## 13. Technology Stack

| Technology | Purpose |
|---|---|
| Python | Main application language |
| SQLite | Local persistent database |
| Ollama | Local model runtime |
| Qwen 3.5 9B | Installed development model |
| pytest | Automated testing |
| HTML / CSS / JavaScript | Frontend |
| Brave Search | Source discovery |
| Crossref | Academic source discovery |
| GitHub | Repository and source discovery integration |

## 14. Current Implementation Status

| Component | Status |
|---|---|
| L1 — Knowledge Base | Implemented as an individual component |
| L2 — Knowledge Gap Analyzer | Implemented as an individual component |
| L3 — Learning Planner | Implemented as an individual component |
| L4 — Source Discovery Engine | Implemented as an individual component |
| L5 — Document Ingestion | Implemented as an individual component |
| SQLite storage | Existing implementation |
| DocumentStoreService | Existing implementation |
| Local Ollama runtime | Used in the development environment |
| Voice input and TTS | Implemented; further verification required |
| Security protections | Implemented; further verification required |
| Full L1–L5 integration | In progress |
| Retrieval over document chunks | Not yet fully implemented and evaluated |
| Evidence verification | Further implementation and evaluation required |
| Cross-platform validation | In progress |

## 15. Development Roadmap

1. Complete end-to-end integration of L1–L5.
2. Implement and evaluate retrieval over ingested document chunks.
3. Establish a reproducible database initialization procedure.
4. Improve source reliability assessment.
5. Define and document LLM responsibilities within individual stages.
6. Develop a structured evidence verification process.
7. Complete security and authorization testing.
8. Validate Windows-specific functionality on Windows.
9. Add sanitized examples demonstrating the complete learning and ingestion workflow.
10. Evaluate model specialization through LoRA after the core architecture becomes reliable.

## 16. Architectural Direction

ORION is being developed as a modular, local-first knowledge engineering system.

Its long-term direction combines:

- Local language-model inference.
- Structured external knowledge.
- Knowledge-gap analysis.
- Explainable learning planning.
- Automated source discovery.
- Versioned document ingestion.
- Retrieval-Augmented Generation.
- Independent evidence evaluation.

The central objective is not simply to create an assistant that generates answers.

It is to create a system that can acquire information, preserve its origins, identify missing knowledge, retrieve relevant evidence, and distinguish supported conclusions from uncertainty.

ORION remains under active development. Its architecture separates implemented components, tested integration paths, and planned capabilities so that future development can proceed without confusing design objectives with verified functionality.
