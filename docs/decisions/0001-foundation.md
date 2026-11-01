# ADR 0001: C#, Unity, VRM, and ASP.NET Core Foundation

- **Status:** Accepted; the AI integration boundary is amended by [ADR 0002](0002-provider-neutral-ai-gateway.md)
- **Date:** 2026-11-02

## Context

The project needs a conversational AI, a controllable anime-style 3D avatar, and an architecture that the owner can understand and code alongside an AI assistant. The OpenAI API key must remain outside the distributed 3D client.

## Decision

Use:

- C# as the primary programming language.
- Unity as the 3D client runtime.
- VRM 1.0 as the avatar interchange format and UniVRM as the Unity integration.
- VRoid Studio as the easiest first tool for creating or customizing the avatar.
- ASP.NET Core as a separate companion API process.
- The OpenAI Responses API behind a service-side adapter.
- A modular monolith organized with Clean Architecture / Ports and Adapters.
- Markdown and plain text as the source of truth for authored character, skill, and memory content.

## Why

C# provides one language across Unity and the service. Unity natively uses C# and has established tooling around humanoid animation. VRM is a platform-independent humanoid avatar format with standardized expressions, gaze, and humanoid animation support; UniVRM provides its Unity implementation. VRoid Studio can export characters as VRM files.

Keeping OpenAI access in ASP.NET Core prevents the secret from being embedded in the Unity build and keeps AI orchestration independently testable.

References:

- [Unity scripting concepts](https://docs.unity3d.com/current/Documentation/Manual/ScriptingSection.html)
- [VRM features](https://vrm.dev/en/vrm/vrm_features/)
- [UniVRM](https://vrm.dev/en/univrm/)
- [VRoid Studio](https://vroid.com/en/studio)
- [OpenAI Responses streaming](https://developers.openai.com/api/docs/guides/streaming-responses)

## Alternatives Considered

### Godot with C#

Godot is an attractive open-source engine and officially supports C#. It was not chosen for the initial foundation because the Unity + UniVRM path is more direct for VRM avatar work. The client boundary makes a future engine change possible without rewriting the companion service.

### Unreal Engine with C++ or Blueprints

Unreal offers excellent rendering but adds more complexity and a second primary programming model for a character-focused desktop companion. It is unnecessary at this stage.

### Python service

Python has strong AI tooling, but using it here would make the owner learn and maintain two main languages immediately. The official OpenAI documentation includes C# examples, so Python is not required for the API integration.

### Direct OpenAI calls from Unity

Rejected because a distributed client cannot safely contain a secret API key, and it would couple avatar presentation to AI infrastructure.

## Consequences

- Unity and the service can be developed and tested separately.
- The owner primarily learns C# rather than two languages.
- Running locally initially requires two processes.
- API contracts must be kept versioned and compatible.
- Unity-specific code must remain out of domain and application layers.

