# ADR 0003: Separate Provider-Neutral Speech Gateway

- **Status:** Accepted
- **Date:** 2026-11-02

## Context

The companion needs both text chat and voice chat. Voice requires speech-to-text and text-to-speech capabilities, whose providers, streaming formats, voice IDs, audio encodings, timing data, SDKs, and credentials can differ from the selected LLM provider.

Putting STT and TTS inside the AI Gateway would mix language generation with audio conversion and make independent provider selection harder.

## Decision

Create a provider-neutral Speech Gateway separate from the AI Gateway.

Expose three conceptual streaming operations across the two gateways:

1. `GenerateTextStream` on the AI Gateway.
2. `TranscribeAudioStream` on the Speech Gateway.
3. `SynthesizeSpeechStream` on the Speech Gateway.

The companion service orchestrates the voice pipeline:

```text
audio → STT → transcript → companion/LLM → response text → TTS → audio
```

The Speech Gateway selects providers using semantic capability aliases such as `transcription.default` and `waifu.voice.default`. Provider-specific model and voice IDs remain in gateway configuration.

STT modules implement the typed STT provider protocol. TTS modules implement the typed TTS provider protocol. One provider package may implement both protocols.

## Contract Principles

- STT accepts audio metadata and ordered audio chunks, then returns partial/final transcript events.
- TTS accepts normalized text and voice options, then returns audio chunks and optional word/phoneme/viseme timing.
- Audio format and sample-rate negotiation must be explicit.
- Provider-specific timing or voice metadata may not be required by companion core logic.
- Provider SDK objects and errors are translated at the gateway boundary.
- Credentials exist only in the Speech Gateway.

## Realtime Speech-to-Speech

Direct speech-to-speech models may reduce latency but often combine STT, LLM state, and TTS in one provider-specific session. They are not the initial contract.

If needed later, add a distinct `realtime.conversation` capability rather than pretending it behaves exactly like the transparent STT → LLM → TTS pipeline.

## Consequences

- Text mode runs without the Speech Gateway.
- Voice mode adds one backend process and two service calls around LLM generation.
- STT, LLM, and TTS providers can be selected independently.
- Transcripts remain available for UI, memory review, debugging, and accessibility.
- The companion can begin TTS phrase-by-phrase while later response text is still being generated.
- Every speech provider module must pass shared STT or TTS contract tests.

