# ADR 0004: Choose Languages Per Component

- **Status:** Accepted
- **Date:** 2026-11-02

## Context

ADR 0001 favored C# across the system partly to minimize the number of languages the project owner would maintain. The owner regularly works with multiple languages and does not consider a polyglot repository a maintenance problem.

The Unity client, companion domain, provider integrations, and local AI/audio workloads have different ecosystem needs.

## Decision

Use:

- **C# and Unity** for the avatar client.
- **C# and ASP.NET Core** for the companion service and its domain/application layers.
- **Python and FastAPI** for the AI Gateway.
- **Python and FastAPI** for the Speech Gateway.

Keep languages isolated by process boundaries. Services share versioned JSON/OpenAPI contracts, not source-code domain packages.

The Python gateways use type annotations, strict static analysis, explicit provider protocols, and contract tests. Python package and runtime versions will be pinned when implementation begins.

## Why

- Unity uses C# natively.
- C# provides a strongly typed foundation for companion behavior, memory orchestration, and workout-domain rules.
- Python provides direct access to a broad ecosystem for hosted AI SDKs, local models, speech processing, and ML runtimes.
- FastAPI supports asynchronous streaming responses and WebSockets, which match the gateway contracts.
- Network boundaries already exist for provider isolation, so using Python does not leak Python types into the C# applications.

References:

- [FastAPI streaming responses](https://fastapi.tiangolo.com/advanced/custom-response/)
- [FastAPI WebSockets](https://fastapi.tiangolo.com/advanced/websockets/)
- [PyTorch ecosystem](https://pytorch.org/projects/pytorch/)

## Consequences

- Developers need both .NET and Python toolchains.
- Formatting, linting, testing, and dependency management are configured separately per language.
- Cross-service contract tests become more important than shared source libraries.
- The gateway services can adopt Python-native local AI or audio libraries without wrapping them for .NET.
- This decision supersedes ADR 0001's rejection of Python based on avoiding a second language; the remaining C#/Unity decisions still stand.

