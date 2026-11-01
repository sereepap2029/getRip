# AI Gateway

This service presents one stable LLM contract to the companion and hides all provider-specific APIs.

```text
src/
├── GetRip.AiGateway.Abstractions/
├── GetRip.AiGateway.Api/
└── Providers/
    └── GetRip.AiGateway.OpenAI/
tests/
├── GetRip.AiGateway.UnitTests/
├── GetRip.AiGateway.ContractTests/
└── GetRip.AiGateway.IntegrationTests/
```

`GetRip.AiGateway.Abstractions` defines normalized requests, streamed events, errors, usage data, and `ILanguageModelProvider`. It contains no provider SDK references.

Provider modules translate between those abstractions and one vendor API. The initial module will use OpenAI. Every added provider must pass the same contract tests.

Provider API keys are supplied only to this service through secure runtime configuration.

