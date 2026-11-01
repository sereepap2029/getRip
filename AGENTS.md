# Project Working Agreement

These rules apply to code written by people or AI assistants in this repository.

## Priorities

1. Keep the code readable enough for the project owner to work alongside an AI assistant.
2. Keep LLM and speech provider secrets and SDKs inside their gateway provider modules.
3. Keep domain and application logic independent of Unity, ASP.NET Core, LLM vendors, and storage details.
4. Prefer small vertical features and explicit interfaces over large frameworks or clever abstractions.
5. Store authored character knowledge, skills, and memory as Markdown or plain text inside this repository.

## Dependency Rules

- `GetRip.Domain` depends on no other project layer.
- `GetRip.Application` may depend on `GetRip.Domain` and defines ports needed by use cases.
- `GetRip.Infrastructure` implements companion application ports and may use file-system or AI Gateway client libraries, but no provider SDK.
- `GetRip.Api` is the composition root and may reference Application and Infrastructure.
- The Unity client communicates with the API through contracts; it does not reference service projects.
- The companion service calls only the provider-neutral AI Gateway contract; it never references OpenAI or another provider SDK.
- Each AI Gateway provider module implements the typed language-model provider protocol and contains all vendor-specific translation.
- The companion service calls only the provider-neutral Speech Gateway contracts for transcription and synthesis.
- Speech provider modules implement the typed STT or TTS provider protocols; a provider may implement both.
- Unity components should not contain workout rules, prompt construction, persistence, or direct LLM calls.

## Coding Rules

- Use C# nullable reference types in Unity and the companion service.
- Use type-annotated Python with strict static analysis in the AI and Speech Gateways.
- Do not share source packages across C# and Python boundaries; share only versioned contracts and generated clients when useful.
- Prefer dependency injection through constructors in service code.
- Keep Unity `MonoBehaviour` classes thin; delegate work to plain C# classes where practical.
- Add tests for domain rules and application use cases.
- Never commit API keys, tokens, personal health data, generated build output, or local Unity state.
- Record a new architecture decision in `docs/decisions/` before introducing a major framework or changing a repository boundary.

## User-Run Build and Test Rule

- AI assistants must not compile the project, start the application, run unit or integration tests, or invoke Unity builds.
- When verification is needed, provide the project owner with the exact working directory and exact terminal command to run manually.
- Briefly explain what the command checks and what successful output should look like.
- Ask the owner to paste the command output back when it is needed for diagnosis or the next change.
- Writing or updating test code is allowed; executing it is reserved for the project owner.
- Do not claim that a build or test passes unless the owner has run the command and supplied a successful result.

## Scope Control

- Begin with exactly three backend deployables when voice is enabled: the companion service, AI Gateway, and Speech Gateway. Text-only development needs only the first two.
- Add abstractions only at real external boundaries or when a second implementation exists.
- Do not add databases, message brokers, containers, cloud deployment, or authentication until a concrete requirement calls for them.

