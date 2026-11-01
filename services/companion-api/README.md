# Companion API

This folder will contain the C#/.NET solution for the companion service.

```text
src/
├── GetRip.Domain/
├── GetRip.Application/
├── GetRip.Infrastructure/
└── GetRip.Api/
tests/
├── GetRip.UnitTests/
└── GetRip.IntegrationTests/
```

This service contains personality orchestration, workout behavior, memory selection, and conversation-flow orchestration. It calls the provider-neutral AI Gateway for generation and the Speech Gateway for STT/TTS. It contains no provider SDK or provider API key.



