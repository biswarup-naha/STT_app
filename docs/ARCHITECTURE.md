
# Speech-To-Text Server — Architecture

## 1. Overview

STT Server is a modular, local-first speech recognition API
built with Python, FastAPI, uv, and faster-whisper.

The architecture follows MVC-inspired separation of concerns
and applies SOLID principles where they improve maintainability.

The initial application supports offline audio transcription
through REST. Real-time transcription is a later extension.

## 2. Architectural Style

The application uses:

- MVC-inspired separation between controllers, models, and routes.
- A service layer for application orchestration.
- An inference-engine abstraction for dependency inversion.
- Dependency injection for assembling application components.
- A modular monolith deployment initially.

A modular monolith keeps all components in one deployable
application while preserving clear internal boundaries.

Separate microservices are not required for the initial MVP.

## 3. Component Responsibilities

### Controllers

Location: src/controllers/

Responsibilities:

- Define HTTP endpoints.
- Receive and validate HTTP requests.
- Invoke application services.
- Return appropriate HTTP responses.

Controllers must not load models or contain inference logic.

### Services

Location: src/services/

Responsibilities:

- Coordinate transcription requests.
- Apply application-level rules.
- Call the configured inference engine.
- Convert engine results into application results.

Services must not depend directly on FastAPI request objects
or a concrete inference library.

### Domain Models

Location: src/models/

Responsibilities:

- Represent transcription results and domain concepts.
- Remain independent of HTTP and inference frameworks.

### API Schemas

Location: src/schemas/

Responsibilities:

- Validate incoming request parameters.
- Define public response structures.
- Control serialization of transcription results.

### Inference Engines

Location: src/engines/

Responsibilities:

- Define the transcription engine contract.
- Load and manage the selected STT model.
- Process audio and return recognized text and metadata.
- Isolate inference-library-specific behavior.

### Configuration

Location: src/config.py

Responsibilities:

- Read configuration from environment variables.
- Define model, device, compute type, and upload limits.
- Avoid embedding deployment-specific settings in code.

### Dependency Wiring

Location: src/dependencies.py

Responsibilities:

- Construct application services and inference engines.
- Share managed engine instances across requests.
- Keep dependency construction out of controllers.

## 4. Request Flow

1. A client submits an audio file to FastAPI.
2. The controller validates request-level parameters.
3. The transcription service receives the transcription request.
4. The configured engine processes the audio.
5. The engine returns a transcription result.
6. The service returns the application result.
7. The controller serializes the response using API schemas.

## 5. SOLID Design Rules

Single Responsibility:
Each module has one primary reason to change.

Open/Closed:
New inference engines can be introduced behind the same contract.

Liskov Substitution:
Every engine implementation must honor the shared contract.

Interface Segregation:
Components depend only on the operations they need.

Dependency Inversion:
Application services depend on an engine abstraction,
not directly on faster-whisper.

## 6. Model Lifecycle

The inference engine should be initialized through application
dependency wiring and loaded once per application worker.

The model must not be loaded for every HTTP request.

Startup and shutdown behavior will be implemented and tested
in a later development milestone.

## 7. Hardware Strategy

Initial hardware:

- NVIDIA GPU with 4 GB VRAM.
- System RAM: 16 GB.

Begin with the multilingual Whisper base model.

Evaluate the small model after measuring memory usage,
latency, and transcription quality.

Support CPU execution as a fallback.

GPU support depends on compatible NVIDIA drivers,
CUDA libraries, and CTranslate2 requirements.

## 8. Testing Strategy

Unit tests:

- Test application services with a fake engine.
- Test input validation and result handling.
- Avoid downloading real models in unit tests.

Integration tests:

- Test HTTP endpoints using FastAPI test clients.
- Test invalid uploads and response contracts.

Model smoke tests:

- Run real inference against a small audio sample.
- Keep these tests separate from fast unit tests.

## 9. Deployment

Local development uses uv.

Docker deployment will be added after local functionality
and tests are established.

The first deployment target is a single application worker.
GPU container support will be validated separately.

## 10. Future Extensions

- WebSocket-based transcription.
- Real-time audio chunk processing.
- WebRTC integration.
- Alternative local inference engines.
- Performance measurements and model benchmarking.
