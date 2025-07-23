# Epics and User Stories

This document outlines the potential epics and user stories that may have guided the development of AnkiConnect, reverse-engineered from the codebase architecture and existing documentation.

## Epics

### Epic: Programmatic Anki Interaction

**Goal:** Enable external applications to interact with Anki programmatically through a well-defined API.

### Epic: Enhanced Note Editing Experience

**Goal:** Provide a more powerful and integrated note editing experience within Anki, especially when triggered from external sources.

### Epic: Anki Data Management

**Goal:** Allow users and external applications to manage various Anki data entities (decks, notes, cards, models, media, tags) efficiently.

### Epic: Anki State and Statistics Access

**Goal:** Provide external applications with the ability to query the current state of Anki and retrieve various statistics.

## User Stories

### Epic: Programmatic Anki Interaction

-   **Story: Expose HTTP API**
    -   As an external application developer, I want AnkiConnect to expose an HTTP endpoint so I can send requests to Anki.
-   **Story: Handle API Requests**
    -   As an AnkiConnect user, I want the plugin to correctly parse and route API requests to the appropriate Anki functions.
-   **Story: Manage API Permissions**
    -   As an Anki user, I want to control which external applications can access my Anki collection to ensure security.
-   **Story: Cross-Origin Resource Sharing (CORS)**
    -   As a web application developer, I want to be able to make requests to AnkiConnect from different origins to build web-based tools.

### Epic: Enhanced Note Editing Experience

-   **Story: Open Enhanced Editor**
    -   As an external application, I want to be able to open a specific note in an enhanced editor dialog within Anki.
-   **Story: Navigate Note History**
    -   As a user of the enhanced editor, I want to navigate between recently edited notes to quickly switch context.
-   **Story: Preview Cards in Editor**
    -   As a user of the enhanced editor, I want to preview the cards generated from the current note to verify its appearance.
-   **Story: Browse Edited Notes**
    -   As a user of the enhanced editor, I want to open the Anki browser with a list of recently edited notes to perform bulk actions.
-   **Story: Seamless Editor Integration**
    -   As a user, I want the enhanced editor to integrate seamlessly with Anki's existing GUI and react to external changes.

### Epic: Anki Data Management

-   **Story: Manage Decks**
    -   As an external application, I want to create, delete, and retrieve information about Anki decks.
-   **Story: Manage Notes**
    -   As an external application, I want to add, update, delete, and query information about Anki notes.
-   **Story: Manage Cards**
    -   As an external application, I want to find, suspend, unsuspend, and modify Anki cards.
-   **Story: Manage Models (Note Types)**
    -   As an external application, I want to create, update, and query information about Anki note models (templates and fields).
-   **Story: Manage Media Files**
    -   As an external application, I want to store, retrieve, and delete media files associated with notes.
-   **Story: Manage Tags**
    -   As an external application, I want to add, remove, and replace tags on notes.

### Epic: Anki State and Statistics Access

-   **Story: Get Review Statistics**
    -   As an external application, I want to retrieve statistics about cards reviewed today or by day.
-   **Story: Get Collection Statistics**
    -   As an external application, I want to retrieve HTML reports of the entire Anki collection's statistics.
-   **Story: Get Profile Information**
    -   As an external application, I want to retrieve information about Anki profiles and switch between them.
-   **Story: Synchronize Collection**
    -   As an external application, I want to trigger a synchronization of the Anki collection with AnkiWeb.

Sources: `plugin/__init__.py`, `plugin/web.py`, `plugin/edit.py`