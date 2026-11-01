# AI Gateway

This Python/FastAPI service presents one stable LLM contract to the companion and hides all provider-specific APIs.

```text
src/getrip_ai_gateway/
├── core/
├── api/
└── providers/
    └── openai/
tests/
├── unit/
├── contract/
└── integration/
```

`core/` defines normalized requests, streamed events, errors, usage data, and a typed language-model provider protocol. It contains no provider SDK references.

Provider modules translate between those abstractions and one vendor API. The initial module will use OpenAI. Every added provider must pass the same contract tests.

Provider API keys are supplied only to this service through secure runtime configuration.

FastAPI is selected because it supports asynchronous streaming responses and WebSockets, while Python keeps future local-model and ML integrations close to their native ecosystem.

