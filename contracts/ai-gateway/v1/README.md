# AI Gateway Contract v1

This contract will define provider-neutral communication between the companion service and the AI Gateway.

The first implementation should cover:

- A streaming generate operation.
- Ordered input messages with stable roles and text content.
- A capability alias rather than a vendor model name.
- Normalized events: `stream.started`, `text.delta`, `message.completed`, `usage`, and `error`.
- Cancellation and correlation identifiers.
- A health/capability description operation.

Provider SDK objects and vendor event names must not appear in this contract.

