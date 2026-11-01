# ADR 0002: Provider-Neutral AI Gateway

- **Status:** Accepted
- **Date:** 2026-11-02

## Context

The companion initially uses OpenAI, but it should be possible to add or switch LLM APIs without changing the Unity client, companion domain, or application use cases. Provider APIs differ in SDKs, message formats, streaming events, tool calls, errors, conversation state, and usage reporting.

## Decision

Create a separately deployable AI Gateway with one versioned, provider-neutral streaming contract.

The companion requests a capability alias such as `companion.default`. Gateway configuration maps that alias to a provider and model. The companion never sends a provider name or vendor model ID during normal operation.

Each provider is an internal gateway module implementing `ILanguageModelProvider`. The gateway normalizes requests, streamed events, errors, completion status, and token usage.

The gateway owns all provider SDKs and credentials. The companion owns prompts, personality, selected memory, workout behavior, and interpretation of the generated reply.

## Why a Service Boundary

- A stable network contract prevents provider SDK types from leaking into the application.
- Provider modules and credentials can be deployed or configured independently.
- A local model can later be exposed through the same contract.
- Contract tests can enforce consistent behavior across providers.
- Provider failures and rate limits can be handled in one place.

## Contract Principles

- Version the contract from its first implementation.
- Optimize for the product's required capabilities, not the union of every vendor feature.
- Use semantic event names such as `text.delta` rather than vendor event names.
- Return normalized errors while preserving provider details only in gateway diagnostics.
- Use capability aliases instead of provider or model names in companion requests.
- Do not promise seamless fallback until privacy, cost, latency, and behavior differences are explicitly handled.
- Permit provider-specific extensions only in an optional namespaced field; companion core logic must not depend on them.

## Consequences

- Local development runs Unity plus two backend processes.
- The gateway adds a small network and operational cost.
- Provider changes no longer require companion or Unity code changes when they fit the existing contract.
- A genuinely new capability may require a new version of the gateway contract rather than hiding important differences.
- The gateway is an infrastructure boundary, not a place for character or workout business logic.

