# Speech Gateway

This Python/FastAPI service exposes provider-neutral streaming speech-to-text and text-to-speech operations.

```text
src/getrip_speech_gateway/
├── core/
├── api/
└── providers/
    ├── stt/
    └── tts/
tests/
├── unit/
├── contract/
└── integration/
```

The core package defines normalized audio chunks, partial/final transcripts, synthesized audio chunks, optional timing data, and typed STT/TTS provider protocols.

Provider modules contain all vendor-specific SDKs and translation. Every module must pass the applicable STT or TTS contract-test suite.

Speech-provider credentials are supplied only to this service through secure runtime configuration.

Python is selected here because future local transcription, synthesis, audio-processing, and ML libraries are commonly Python-native. FastAPI provides asynchronous streaming and WebSocket support at the service boundary.


