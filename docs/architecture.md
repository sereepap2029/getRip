# Architecture

## Goal

Create a maintainable AI workout companion that can speak through an animated 3D character while keeping conversation, workout, and memory logic testable outside Unity.

## System Shape

```text
User
  │
  ▼
Unity Avatar Client
  │  HTTP + streamed events
  ▼
ASP.NET Core Companion API  ◄── reads ── character/, memory/, skills/
  │
  ├── Application use cases
  ├── Domain rules
  └── Infrastructure adapters
          ├── provider-neutral text stream ──► AI Gateway
          │                                    └── LLM provider module
          └── provider-neutral audio stream ─► Speech Gateway
                                               ├── STT provider module
                                               └── TTS provider module
```

The companion begins as a **C# modular monolith**, while LLM and speech-provider access use separate, intentionally narrow Python services. This creates stable external-provider boundaries without turning every application module into a microservice.

## Responsibilities

### Unity avatar client

The client owns:

- VRM model loading and rendering.
- Animation, facial expression, gaze, and lip-sync presentation.
- Chat and settings UI.
- Microphone and speaker device interaction when voice is added.
- Translating API response events into avatar actions.

The client does not own prompts, long-term memory, workout policy, or provider API keys.

Suggested Unity code areas under `Assets/GetRip/Scripts/`:

- `Bootstrap/` — dependency wiring and startup.
- `Api/` — typed companion-service client and streaming transport.
- `Avatar/` — animation, expression, gaze, and lip-sync adapters.
- `UI/` — views and presentation logic.
- `Audio/` — microphone capture and speech playback.

Use a small state machine for visible avatar states such as idle, listening, thinking, speaking, celebrating, and concerned. API messages request semantic actions; they should not name animation clips directly.

### Companion service

The service follows dependency inversion:

```text
GetRip.Api ───────────────┐
                         ▼
GetRip.Infrastructure ─► GetRip.Application ─► GetRip.Domain
```

- `GetRip.Domain` — entities, value objects, and rules that need no framework.
- `GetRip.Application` — use cases and interfaces such as `ITextGenerator`, `IMemoryStore`, and `IClock`.
- `GetRip.Infrastructure` — AI Gateway client and text-file implementations of those interfaces.
- `GetRip.Api` — endpoints, streaming transport, configuration, and dependency wiring.

This is Clean Architecture used as a boundary rule, not as a reason to create an interface for every class.

### AI Gateway service

The AI Gateway is the only process that knows which LLM provider is in use. It owns:

- Provider API keys and SDK dependencies.
- Provider modules implementing a common `ILanguageModelProvider` interface.
- Mapping normalized requests to provider-specific requests.
- Mapping provider events, errors, finish reasons, and usage into normalized responses.
- Routing capability aliases to a configured provider and model.
- Streaming, timeouts, retries, rate-limit handling, and provider health reporting.

The gateway does **not** own the waifu personality, workout rules, user memory, or UI behavior. Those remain in the companion service so changing providers cannot change the product architecture.

The first module is OpenAI. Later modules can support other hosted APIs or local models without changing the Unity client or companion application use cases.

Internally, the gateway is split into:

- `core/` — normalized request/event types and the language-model provider protocol.
- `api/` — FastAPI endpoints, routing, configuration, and dependency wiring.
- `providers/openai/` — OpenAI-specific SDK and translation code.

Each future provider is another module under `Providers/`; it must pass the same contract tests.

### Speech Gateway service

Voice is a pipeline composed by the companion; it is not a second kind of LLM request.

The Speech Gateway owns two independent provider-neutral capabilities:

- **STT — speech to text:** accepts streamed audio and emits partial and final transcripts.
- **TTS — text to speech:** accepts text and emits streamed audio plus optional timing metadata.

The gateway owns speech-provider SDKs, voice identifiers, audio-format translation, credentials, timeouts, and normalized errors. A provider module may support only STT, only TTS, or both.

Internally, the gateway is split into:

- `core/` — normalized audio, transcript, synthesis, timing, and provider protocols.
- `api/` — FastAPI endpoints, capability routing, configuration, and dependency wiring.
- `providers/` — replaceable STT and TTS adapters.

Use semantic aliases such as `transcription.default` and `waifu.voice.default`. The companion must not depend on provider voice IDs.

## Communication Contract

Process-boundary payloads live in `contracts/` as versioned JSON examples or JSON Schema. Keeping contracts outside the runtimes avoids coupling Unity's supported .NET profile to the server's .NET version or coupling the companion to an LLM SDK.

A companion reply should eventually carry semantic information similar to:

```json
{
  "text": "Nice work. One more controlled set.",
  "emotion": "encouraging",
  "avatarAction": "celebrate_small",
  "conversationId": "..."
}
```

The exact contract will be designed with the first conversational feature. Text should stream as it arrives; an emotion or avatar action can be sent as structured metadata.

### AI Gateway contract

The companion calls a versioned gateway operation with provider-neutral data. It selects a capability alias such as `companion.default`, not a vendor or model name:

```json
{
  "capability": "companion.default",
  "messages": [
    { "role": "user", "content": "Motivate me for today's workout." }
  ],
  "stream": true
}
```

The streamed result uses stable event types such as `text.delta`, `message.completed`, `usage`, and `error`. Provider-only metadata may be recorded inside the gateway for diagnostics but must not leak into the normal contract.

Capability aliases are resolved by gateway configuration, for example:

```text
companion.default ──► openai / configured-model
companion.fast    ──► local / configured-model
```

This allows a provider or model to change without rebuilding the main program. Routing should be explicit configuration; automatic fallback can be added later only after its cost and behavior rules are defined.

### Text and voice flows

Text uses one generation pipeline:

```text
Unity text ─► Companion ─► AI Gateway ─► streamed text/events ─► Unity
```

Voice composes three provider-neutral operations:

```text
Unity microphone audio
  └─► Companion ─► Speech Gateway STT ─► transcript
                    Companion ─► AI Gateway ─► response text
                    Companion ─► Speech Gateway TTS ─► audio chunks/timing
  ◄────────────────────────────────────────────────────────────── Unity playback
```

The companion can show partial transcripts immediately. As complete response phrases become available, it can send them to TTS so audio playback begins before the entire LLM response finishes.

TTS timing events may include word, phoneme, or viseme timing when a provider supports them. Unity uses these for lip-sync; when unavailable, the client may derive approximate mouth movement from the audio signal.

Direct realtime speech-to-speech is deliberately not the base contract. It may later be introduced as a separate `realtime.conversation` capability for providers that support it, without removing the transparent STT → LLM → TTS path.

## OpenAI Provider

The initial provider module uses the OpenAI Responses API and converts its typed streaming events into gateway events. OpenAI conversation identifiers remain inside that module or gateway-owned state; they never become companion domain objects.

The gateway reads provider API keys from environment or secret configuration. It never returns keys to callers and never stores them in repository files.

OpenAI supports streaming typed response events and conversation state through the Responses API. The official documentation also warns against embedding API keys in client applications:

- [Streaming API responses](https://developers.openai.com/api/docs/guides/streaming-responses)
- [Conversation state](https://developers.openai.com/api/docs/guides/conversation-state)
- [API key safety](https://help.openai.com/en/articles/5112595-best-practices-for-api-key-safety)

## Character, Skills, and Memory

- `character/` holds the authored identity, tone, relationship boundaries, and behavioral guidance.
- `skills/` holds focused capability instructions, one folder or text file per skill.
- `memory/` holds human-readable facts and summaries. Sensitive local memory belongs under `memory/private/` and is ignored by Git.

These text files are the source of truth. Infrastructure decides how to load and select relevant content; domain logic must not read files directly.

## Testing Strategy

- Domain unit tests cover workout and behavior rules without a network or Unity.
- Application unit tests use fake ports for the AI Gateway, memory, and time.
- Companion integration tests cover text-file adapters and both companion-facing boundaries.
- Gateway contract tests run every provider module against the same expected normalized behavior.
- Speech Gateway contract tests separately verify STT and TTS modules.
- Provider integration tests are opt-in and use their provider's test credentials.
- Unity tests cover message-to-avatar-state mapping and critical presentation behavior.
- Live LLM calls are never part of the normal test suite.

## Deliberately Deferred

No decision has been made yet about voice providers, workout sensing, a database, cloud hosting, mobile deployment, or the final character design. Those choices should follow real requirements rather than shape the foundation prematurely.

