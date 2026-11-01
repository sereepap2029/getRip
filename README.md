# GetRip — AI Workout Waifu

GetRip is a personal AI companion project. Its purpose is to motivate me to work out consistently, build more muscle, improve my health and appearance, and feel more confident.

The companion will initially use the OpenAI API for conversation and a 3D anime-style avatar for its visible character. A provider-neutral AI Gateway keeps the rest of the application independent from any specific LLM vendor.

## Chosen Foundation

- **Languages:** C# for Unity and companion domain; Python for AI and speech gateways
- **3D client:** Unity
- **Avatar format:** VRM 1.0 through UniVRM
- **First avatar creation tool:** VRoid Studio
- **Companion service:** ASP.NET Core
- **AI Gateway:** Python/FastAPI with replaceable provider modules
- **Speech Gateway:** Python/FastAPI with provider-neutral streaming STT and TTS modules
- **First LLM provider:** OpenAI Responses API with streaming
- **Project knowledge:** Markdown and plain-text files stored in this repository
- **Architecture:** A companion modular monolith plus provider-neutral AI and Speech Gateway services

Languages are selected per boundary rather than standardized across the repository. C# fits Unity and the structured companion domain. Python gives the provider gateways direct access to the broader AI, audio, and local-model ecosystem. Versioned network contracts keep the runtimes independent.

The project uses the **OpenAI API**, not the ChatGPT consumer application as an API. Provider API keys belong only in their gateway service and must never be placed in Unity, the companion service, or source code.

## Repository Map

```text
getRip/
├── apps/avatar-client/          Unity application and 3D presentation
├── services/companion-api/     ASP.NET Core service and business logic
├── services/ai-gateway/        Provider-neutral LLM gateway and adapters
├── services/speech-gateway/    Provider-neutral STT and TTS gateway
├── contracts/                  Versioned client/service message contracts
├── character/                  Personality, voice, boundaries, and prompts
├── memory/                     Plain-text companion memory
├── skills/                     Plain-text skill definitions
├── docs/                       Architecture and decision records
└── tools/                      Developer utilities
```

Read [docs/architecture.md](docs/architecture.md) before adding application code. Technology choices are recorded in [ADR 0001](docs/decisions/0001-foundation.md), the LLM boundary in [ADR 0002](docs/decisions/0002-provider-neutral-ai-gateway.md), the voice pipeline in [ADR 0003](docs/decisions/0003-speech-gateway.md), and the polyglot language strategy in [ADR 0004](docs/decisions/0004-polyglot-services.md).

## Current Status

The repository has its architecture and folder boundaries, but no product features have been implemented yet.

## Development Verification

The project owner runs all compilation, application startup, Unity builds, and automated tests manually. AI assistants may prepare code and tests, but must provide the exact working directory and terminal command instead of executing verification themselves.
