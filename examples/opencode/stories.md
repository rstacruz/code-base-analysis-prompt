---
createdAt: 2025-07-23T00:00:00Z
---

# Epics and User Stories

This document outlines the potential epics and user stories that may have guided the development of the OpenCode application, reverse-engineered from the codebase architecture and module functionalities.

## Epics

### Epic: Application Core & Management

**Goal:** Enable users to manage the application's fundamental operations, configuration, and command-line interactions.

### Epic: AI Interaction & Language Models

**Goal:** Provide robust capabilities for interacting with AI language models and utilizing their associated tools.

### Epic: File System & Version Control

**Goal:** Enable users to interact with the file system, track changes, and manage project versions.

### Epic: Inter-process Communication & Events

**Goal:** Facilitate seamless communication and real-time updates between different components of the application.

### Epic: Development & Extensibility

**Goal:** Support developers with integrated tools and a flexible architecture for extending the application.

### Epic: Authentication & Permissions

**Goal:** Secure access to resources and manage user consent for sensitive actions.

## User Stories

### Epic: Application Core & Management

- **Story: Application Lifecycle**
  - As a user, I want to start and stop the OpenCode application.
- **Story: Configuration Management**
  - As a user, I want to configure application settings, including themes, keybinds, and experimental features.
  - As a user, I want the application to load configuration from global and project-specific files.
- **Story: Installation & Updates**
  - As a user, I want to see information about the current OpenCode installation, including its version and available updates.
  - As a user, I want to update the OpenCode application to the latest version.
- **Story: Command-Line Interface**
  - As a user, I want to execute various commands via the command-line interface (e.g., listing models).
  - As a user, I want clear and formatted error messages when CLI commands fail.
- **Story: Global Application State**
  - As a developer, I want the application to manage its global paths and cache effectively.

### Epic: AI Interaction & Language Models

- **Story: Conversational AI Sessions**
  - As a user, I want to chat with AI models in a conversational session.
  - As a user, I want to create, list, and manage multiple chat sessions.
  - As a user, I want to summarize long conversations with the AI to maintain context.
  - As a user, I want to revert my session to a previous state.
- **Story: Model Selection & Usage**
  - As a user, I want to select different AI models and providers for my tasks.
  - As a user, I want to use a smaller, more efficient model for tasks like summarization.
  - As a developer, I want to easily integrate new language model providers and their SDKs.
- **Story: Tool Utilization**
  - As a user, I want the AI to use various tools (e.g., read files, execute bash commands) to assist with my tasks.
  - As a user, I want to manage connections to external Model Context Protocol (MCP) servers that provide additional tools.

### Epic: File System & Version Control

- **Story: File Status & Content**
  - As a user, I want to view the Git status of files in my project (added, modified, deleted).
  - As a user, I want to read the content of files, including seeing Git diffs for modified files.
- **Story: File Search**
  - As a user, I want to search for text within files in my project.
  - As a user, I want to find files by name.
- **Story: Project Snapshots**
  - As a user, I want to create snapshots of my project's working directory to save its state.
  - As a user, I want to revert my project to a previous snapshot.
- **Story: Persistent Storage**
  - As a user, I want the application to persist my session data, messages, and other relevant information.
  - As a developer, I want a reliable way to store and retrieve JSON data.

### Epic: Inter-process Communication & Events

- **Story: Real-time Updates**
  - As a user, I want real-time updates and notifications from the application (e.g., when a file is edited, or a session is updated).
- **Story: Session Sharing**
  - As a user, I want to share my conversation sessions with others via a unique URL.
  - As a user, I want to unshare a previously shared session.
- **Story: Decoupled Communication**
  - As a developer, I want a decoupled way for components to communicate via an event bus.

### Epic: Development & Extensibility

- **Story: Code Formatting**
  - As a developer, I want the application to automatically format my code files based on their extension.
- **Story: IDE Integration**
  - As a developer, I want the application to integrate with my IDE (e.g., VS Code) for enhanced features like extension installation.
- **Story: Code Intelligence (LSP)**
  - As a developer, I want to interact with Language Server Protocol (LSP) servers for code intelligence (diagnostics, hover information, symbol lookup).
- **Story: Bun Runtime Management**
  - As a developer, I want to execute Bun commands and manage Bun packages within the application.
- **Story: Utility Functions**
  - As a developer, I want to use common utility functions for logging, error handling, filesystem operations, and asynchronous programming patterns.
- **Story: Network Tracing**
  - As a developer, I want to trace network requests for debugging purposes.

### Epic: Authentication & Permissions

- **Story: Secure Authentication**
  - As a user, I want my authentication credentials for various providers (e.g., OAuth, API keys) to be securely stored.
- **Story: User Consent Management**
  - As a user, I want the application to ask for my explicit permission before performing sensitive actions.
  - As a user, I want the application to remember my permission choices for future interactions.
