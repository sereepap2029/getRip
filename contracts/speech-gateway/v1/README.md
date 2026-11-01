# Speech Gateway Contract v1

This contract will define provider-neutral communication between the companion service and Speech Gateway.

## Transcription stream

Input:

- Capability alias.
- Audio encoding, sample rate, channel count, and language hint when known.
- Ordered audio chunks and cancellation.

Output events:

- `transcription.started`
- `transcript.partial`
- `transcript.final`
- `transcription.completed`
- `error`

## Synthesis stream

Input:

- Capability/voice alias.
- Text or ordered text segments.
- Requested output audio format when needed.

Output events:

- `synthesis.started`
- `audio.chunk`
- Optional `timing.word`, `timing.phoneme`, or `timing.viseme`
- `synthesis.completed`
- `error`

Provider model names, voice IDs, SDK objects, and vendor event names must not appear in the normal contract.

