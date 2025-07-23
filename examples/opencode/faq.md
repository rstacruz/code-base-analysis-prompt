---
createdAt: 2025-07-23T00:00:00Z
---

# Frequently Asked Questions

This document provides answers to frequently asked questions about the OpenCode codebase, based on the generated codebase analysis documents.

### Overall architecture

> Q: What is the overall architecture of the OpenCode application?

- Monorepo project with distinct packages for CLI, TUI, web interface, and SDKs.
- Core logic in `packages/opencode`.
- Modular architecture with functionalities encapsulated in distinct modules.
- Communication between components (e.g., Go-based TUI and TypeScript server) via a generated SDK.
- See [Architecture section in the main documentation](index.md#architecture).

### Modules and features

> Q: What are the main modules in the `packages/opencode/src` directory?

- **Core/Foundation:** App, Global, ID, Util.
- **CLI/User Interface:** CLI, CLI Commands, Format, IDE, Installation.
- **System/Infrastructure:** Bun, Bus, Config, File, Flag, LSP, MCP, Permission, Provider, Server, Session, Share, Snapshot, Storage, Tool, Trace.
- **Security/Authentication:** Auth.
- See [Modules section in the main documentation](index.md#modules).

### Configuration

> Q: How does OpenCode manage application configuration?

- The `Config` module loads, parses, and manages configuration.
- Supports hierarchical loading from `opencode.jsonc` and `opencode.json`.
- Merges global and project-specific configurations.
- Handles environment variable interpolation and file content inclusion.
- Uses Zod schemas for type safety and validation.
- See [Config Module](config.md).

### User sessions

> Q: How does OpenCode handle user sessions and AI interactions?

- The `Session` module manages creation, persistence, and lifecycle of sessions.
- Integrates with language models via the `Provider` module.
- Supports tool execution, message queuing, session summarization, and state reversion.
- Publishes session and message events via the `Bus` module.
- See [Session Module](session.md).

### Provider module

> Q: What is the role of the `Provider` module?

- Manages and interacts with various language model providers (e.g., Anthropic, OpenAI).
- Loads provider configurations, authenticates, retrieves models, and exposes tools.
- Dynamically loads provider SDKs.
- See [Provider Module](provider.md).

### File system and Git

> Q: How does OpenCode interact with the file system and Git?

- The `File` module provides utilities for Git-related file status and content reading.
- Tracks changes (added, deleted, modified) and reads file content (raw or Git patch).
- The `Snapshot` module creates and restores snapshots of the working directory using Git commits.
- See [File Module](file.md) and [Snapshot Module](snapshot.md).

### Inter-process communication and real-time updates

> Q: How does OpenCode handle inter-process communication and real-time updates?

- The `Bus` module implements a simple event bus system for decoupled communication.
- Allows components to publish and subscribe to events.
- The `Server` module provides an SSE (`/event`) endpoint to stream all bus events to clients for real-time updates.
- See [Bus Module](bus.md) and [Server Module](server.md).

### Development and extensibility

> Q: How can I contribute to OpenCode development?

- The project uses `bun` for dependency management and scripting.
- Key directories include `packages/opencode`, `packages/tui`, `packages/function`, `packages/web`, `sdks/github`, `sdks/vscode`, `infra`, `docs`, and `scripts`.
- Follow the development workflow for installation, running, type checking, and testing.
- Adhere to the specified code style guidelines.
- See [Development Guide](development.md).

### IDE integration and code intelligence

> Q: How does OpenCode integrate with IDEs and provide code intelligence?

- The `IDE` module provides functionalities for interacting with IDEs, primarily for extension installation.
- The `LSP` module provides Language Server Protocol client functionalities for diagnostics, hover information, and symbol lookup.
- See [IDE Module](ide.md) and [LSP Module](lsp.md).

### Security and permissions

> Q: How does OpenCode manage authentication and user permissions?

- The `Auth` module manages authentication information for various providers (OAuth, API keys).
- The `Permission` module manages user permissions for actions, allowing asking for, approving, or rejecting permissions.
- It remembers user choices for future interactions.
- See [Auth Module](auth.md) and [Permission Module](permission.md).
