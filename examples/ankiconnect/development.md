# Development Guide

This document provides guidance for developers who want to contribute to AnkiConnect.

## Tech Stack

AnkiConnect is primarily developed in **Python**, leveraging the Anki desktop application's API (`aqt` and `anki` modules). It uses standard Python libraries for web server functionalities and utility operations.

## Development Environment Setup

To set up your development environment, you will need:

1.  **Anki Desktop**: AnkiConnect is an add-on for Anki. You'll need a working installation of Anki Desktop (version 2.1.x or later, specifically 2.1.50+ is recommended for full feature compatibility).
2.  **Python**: Ensure you have a Python environment compatible with your Anki installation. Anki typically bundles its own Python environment, so direct management might not be necessary for basic development.
3.  **Git**: For version control.

**Steps:**

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/FooSoft/anki-connect.git
    cd anki-connect
    ```
2.  **Link the plugin to Anki (Development Mode):**
    Anki add-ons are typically located in a specific directory. You can symlink your cloned repository into Anki's add-ons folder for easier development. The `link.sh` script in the project root can assist with this:
    ```bash
    ./link.sh
    ```
    This script will attempt to find your Anki add-ons folder and create a symbolic link. You may need to adjust it for your specific operating system and Anki installation path.

## Repository Structure

-   `plugin/`: Contains the core Python source code for the AnkiConnect add-on.
    -   `__init__.py`: Main entry point and API handler.
    -   `web.py`: Implements the HTTP web server.
    -   `edit.py`: Contains GUI extensions, particularly the enhanced note editor.
    -   `util.py`: Utility functions and helpers.
    -   `config.json`: Default configuration for the add-on.
    -   `config.md`: Documentation for the configuration.
-   `tests/`: Unit and integration tests for the plugin.
-   `docs/`: Project documentation, including architecture analysis.
-   `link.sh`: Script to symlink the plugin for development.
-   `package.sh`: Script to package the add-on for distribution.

## Development Workflow

1.  **Make changes**: Modify the Python source files in the `plugin/` directory.
2.  **Restart Anki**: After making changes to the Python code, you typically need to restart Anki for the changes to take effect.
3.  **Run tests**: Before committing, ensure all tests pass.

## Code Quality and Testing

-   **Testing Framework**: The project uses `pytest` for testing.
-   **Running Tests**: Navigate to the project root and run:
    ```bash
    pytest
    ```
-   **Code Style**: Adhere to PEP 8 guidelines for Python code. Use a linter (e.g., `flake8` or `ruff`) to check for style and common errors.

## Build System

The `package.sh` script is used to create a distributable `.ankiaddon` package. This script bundles the necessary files into a zip archive with the correct extension.

```bash
./package.sh
```

Sources: `link.sh`, `package.sh`, `plugin/__init__.py`, `plugin/web.py`, `plugin/edit.py`, `plugin/util.py`, `tests/`