# Frequently Asked Questions

This document provides answers to frequently asked questions about the AnkiConnect architecture and functionality.

### Overall Architecture

> Q: What is the overall architecture of AnkiConnect?

-   AnkiConnect is an Anki plugin that exposes a local web server.
-   It allows external applications to interact with Anki programmatically via a JSON-RPC style API over HTTP.
-   Key components include the Web Server, API Handler, and GUI Extensions.
-   See [Architecture section in the main documentation](architecture.md#architecture).

### API Module

> Q: What is the role of the API module?

-   It is the central part of AnkiConnect.
-   Responsible for handling the logic of all available API actions.
-   Receives requests from the web server, interacts with the Anki collection, and returns results.
-   See [API Module documentation](api.md).

> Q: How are API methods identified and versioned?

-   The `AnkiConnect` class uses a decorator-based system (`@util.api()`) to identify API methods.
-   Methods are dynamically looked up based on `action` and `version` parameters in the request.
-   See [API Module Architecture](api.md#architecture).

### Web Server Module

> Q: What does the Web Server module do?

-   Handles all HTTP communication for AnkiConnect.
-   Listens for incoming requests, parses them, and passes them to the API handler.
-   Manages Cross-Origin Resource Sharing (CORS).
-   See [Web Server Module documentation](web-server.md).

> Q: How does AnkiConnect handle CORS?

-   It checks the `Origin` header of incoming requests against a configurable whitelist.
-   If allowed, it adds necessary `Access-Control-Allow-Origin` headers to the response.
-   It also handles pre-flight `OPTIONS` requests.
-   See [Web Server Module Features: CORS Support](web-server.md#cors-support).

### GUI Module

> Q: What custom GUI components does AnkiConnect provide?

-   The primary component is a feature-rich note editor dialog (`Edit` class).
-   It extends Anki's built-in `EditCurrent` dialog with additional functionality.
-   See [GUI Module documentation](gui.md).

> Q: What are the key features of the enhanced note editor?

-   **Preview Button**: Renders and displays cards for the current note.
-   **Navigation History**: Allows users to move between recently edited notes.
-   **Browse Button**: Opens the Anki browser with the history of edited notes.
-   See [GUI Module Features: Enhanced Note Editor](gui.md#enhanced-note-editor).

### Utility Module

> Q: What is the purpose of the Utility module?

-   Provides various helper functions and utilities used across the plugin.
-   Includes API decorators, media type definitions, configuration settings retrieval, and data downloading.
-   See [Utility Module documentation](util.md).

### Development and Contribution

> Q: How can I set up a development environment for AnkiConnect?

-   Clone the repository and use the `link.sh` script to symlink the plugin to Anki's add-ons folder.
-   Requires Anki Desktop and a compatible Python environment.
-   See [Development Guide: Development Environment Setup](development.md#development-environment-setup).

> Q: How do I run tests for AnkiConnect?

-   The project uses `pytest`.
-   Navigate to the project root and run `pytest`.
-   See [Development Guide: Code Quality and Testing](development.md#code-quality-and-testing).

Sources: `notes/architecture/architecture.md`, `notes/architecture/api.md`, `notes/architecture/web-server.md`, `notes/architecture/gui.md`, `notes/architecture/util.md`, `notes/architecture/development.md`