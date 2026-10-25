
# STT Server — Architecture Decision Records

## ADR-001: Use FastAPI

Status: Accepted

Context:
The service must expose a reusable HTTP API to other applications.

Decision:
Use FastAPI for REST endpoints, request validation, and OpenAPI
documentation.

Consequences:
The application can expose a stable API without coupling the
inference engine to HTTP handling.

## ADR-002: Use uv

Status: Accepted

Context:
The project needs reproducible dependency management and a
consistent Python development workflow.

Decision:
Use uv and commit the generated uv.lock file.

Consequences:
Dependencies will be installed using the locked project
configuration after it has been established.

## ADR-003: Use faster-whisper Initially

Status: Accepted for MVP

Context:
The project requires local speech recognition without paid
inference APIs.

Decision:
Use faster-whisper as the initial inference implementation.

Consequences:
The implementation must isolate faster-whisper behind an
engine contract so it can be replaced later.

## ADR-004: Use MVC-Inspired Layering and SOLID

Status: Accepted

Context:
The project should be modular, testable, and maintainable.

Decision:
Separate controllers, application services, domain models,
API schemas, and inference engines.

Consequences:
The initial structure contains more modules than a minimal
FastAPI application, but responsibilities remain explicit.

## ADR-005: Start With Offline REST Transcription

Status: Accepted

Context:
Real-time transcription introduces additional complexity
in audio transport, buffering, and partial results.

Decision:
Implement file transcription before streaming.

Consequences:
The first usable version will be easier to test and debug.
Streaming will be introduced after the REST contract stabilizes.

## ADR-006: Select a Hardware-Conscious Model

Status: Accepted for Initial Experiment

Context:
The development machine has 4 GB of NVIDIA VRAM and
16 GB of system RAM.

Decision:
Start benchmarking the multilingual Whisper base model.
Evaluate small after the baseline works.

Consequences:
Model selection will be validated through measurements rather
than assuming that a model fits every workload.

## ADR-007: Keep Inference Local

Status: Accepted

Context:
The service should not require a paid cloud inference provider.

Decision:
Run the speech recognition model locally by default.

Consequences:
Model files must be downloaded or provisioned locally.
Runtime inference must not depend on a paid inference API.
