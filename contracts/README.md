# Contracts

Versioned JSON examples or JSON Schema shared across process boundaries live here.

- `companion-api/` defines the Unity client ↔ companion service contract.
- `ai-gateway/` defines the companion service ↔ AI Gateway contract.
- `speech-gateway/` defines the companion service ↔ Speech Gateway STT/TTS contracts.

Contracts describe transport data only. They do not contain domain behavior, provider SDK types, secrets, prompts, or Unity animation clip names.

