# Speech Gateway

This service exposes provider-neutral streaming speech-to-text and text-to-speech operations.

```text
src/
├── GetRip.SpeechGateway.Abstractions/
├── GetRip.SpeechGateway.Api/
└── Providers/
    ├── STT/
    └── TTS/
tests/
├── GetRip.SpeechGateway.UnitTests/
├── GetRip.SpeechGateway.ContractTests/
└── GetRip.SpeechGateway.IntegrationTests/
```

The abstractions define normalized audio chunks, partial/final transcripts, synthesized audio chunks, optional timing data, `ISpeechToTextProvider`, and `ITextToSpeechProvider`.

Provider modules contain all vendor-specific SDKs and translation. Every module must pass the applicable STT or TTS contract-test suite.

Speech-provider credentials are supplied only to this service through secure runtime configuration.

