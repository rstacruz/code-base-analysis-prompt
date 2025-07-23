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

- **Story: Expose HTTP API**
  - As an external application developer, I want AnkiConnect to expose an HTTP endpoint so I can send requests to Anki.
  - **Main entry points:** `plugin/web.py:WebServer.listen`, `plugin/web.py:WebServer.__init__`
- **Story: Handle API Requests**
  - As an AnkiConnect user, I want the plugin to correctly parse and route API requests to the appropriate Anki functions.
  - **Main entry points:** `plugin/__init__.py:AnkiConnect.handler`, `plugin/web.py:WebClient.parseRequest`
- **Story: Manage API Permissions**
  - As an Anki user, I want to control which external applications can access my Anki collection to ensure security.
  - **Main entry points:** `plugin/__init__.py:AnkiConnect.requestPermission`, `plugin/web.py:WebServer.allowOrigin`
- **Story: Cross-Origin Resource Sharing (CORS)**
  - As a web application developer, I want to be able to make requests to AnkiConnect from different origins to build web-based tools.
  - **Main entry points:** `plugin/web.py:WebServer.handlerWrapper`, `plugin/web.py:WebServer.allowOrigin`

### Epic: Enhanced Note Editing Experience

- **Story: Open Enhanced Editor**
  - As an external application, I want to be able to open a specific note in an enhanced editor dialog within Anki.
  - **Main entry points:** `plugin/__init__.py:AnkiConnect.guiEditNote`, `plugin/edit.py:Edit.open_dialog_and_show_note_with_id`
- **Story: Navigate Note History**
  - As a user of the enhanced editor, I want to navigate between recently edited notes to quickly switch context.
  - **Main entry points:** `plugin/edit.py:History.append`, `plugin/edit.py:History.get_note_to_left_of`, `plugin/edit.py:History.get_note_to_right_of`, `plugin/edit.py:Edit.show_previous`, `plugin/edit.py:Edit.show_next`
- **Story: Preview Cards in Editor**
  - As a user of the enhanced editor, I want to preview the cards generated from the current note to verify its appearance.
  - **Main entry points:** `plugin/edit.py:Edit.show_preview`, `plugin/edit.py:DecentPreviewer`
- **Story: Browse Edited Notes**
  - As a user of the enhanced editor, I want to open the Anki browser with a list of recently edited notes to perform bulk actions.
  - **Main entry points:** `plugin/edit.py:Edit.show_browser`, `plugin/edit.py:trigger_search_for_dialog_history_notes`
- **Story: Seamless Editor Integration**
  - As a user, I want the enhanced editor to integrate seamlessly with Anki's existing GUI and react to external changes.
  - **Main entry points:** `plugin/edit.py:Edit.register_with_anki`, `plugin/edit.py:Edit.on_operation_did_execute`

### Epic: Anki Data Management

- **Story: Manage Decks**
  - As an external application, I want to create, delete, and retrieve information about Anki decks.
  - **Main entry points:** `plugin/__init__.py:AnkiConnect.deckNames`, `plugin/__init__.py:AnkiConnect.deckNamesAndIds`, `plugin/__init__.py:AnkiConnect.createDeck`, `plugin/__init__.py:AnkiConnect.deleteDecks`, `plugin/__init__.py:AnkiConnect.getDecks`, `plugin/__init__.py:AnkiConnect.changeDeck`, `plugin/__init__.py:AnkiConnect.getDeckConfig`, `plugin/__init__.py:AnkiConnect.saveDeckConfig`, `plugin/__init__.py:AnkiConnect.setDeckConfigId`, `plugin/__init__.py:AnkiConnect.cloneDeckConfigId`, `plugin/__init__.py:AnkiConnect.removeDeckConfigId`, `plugin/__init__.py:AnkiConnect.deckNameFromId`
- **Story: Manage Notes**
  - As an external application, I want to add, update, delete, and query information about Anki notes.
  - **Main entry points:** `plugin/__init__.py:AnkiConnect.addNote`, `plugin/__init__.py:AnkiConnect.updateNoteFields`, `plugin/__init__.py:AnkiConnect.updateNote`, `plugin/__init__.py:AnkiConnect.updateNoteModel`, `plugin/__init__.py:AnkiConnect.deleteNotes`, `plugin/__init__.py:AnkiConnect.findNotes`, `plugin/__init__.py:AnkiConnect.notesInfo`, `plugin/__init__.py:AnkiConnect.notesModTime`, `plugin/__init__.py:AnkiConnect.removeEmptyNotes`, `plugin/__init__.py:AnkiConnect.canAddNote`, `plugin/__init__.py:AnkiConnect.canAddNoteWithErrorDetail`
- **Story: Manage Cards**
  - As an external application, I want to find, suspend, unsuspend, and modify Anki cards.
  - **Main entry points:** `plugin/__init__.py:AnkiConnect.findCards`, `plugin/__init__.py:AnkiConnect.cardsInfo`, `plugin/__init__.py:AnkiConnect.cardsModTime`, `plugin/__init__.py:AnkiConnect.suspend`, `plugin/__init__.py:AnkiConnect.unsuspend`, `plugin/__init__.py:AnkiConnect.areSuspended`, `plugin/__init__.py:AnkiConnect.setEaseFactors`, `plugin/__init__.py:AnkiConnect.getEaseFactors`, `plugin/__init__.py:AnkiConnect.setSpecificValueOfCard`, `plugin/__init__.py:AnkiConnect.setDueDate`, `plugin/__init__.py:AnkiConnect.forgetCards`, `plugin/__init__.py:AnkiConnect.relearnCards`, `plugin/__init__.py:AnkiConnect.answerCards`, `plugin/__init__.py:AnkiConnect.cardsToNotes`
- **Story: Manage Models (Note Types)**
  - As an external application, I want to create, update, and query information about Anki note models (templates and fields).
  - **Main entry points:** `plugin/__init__.py:AnkiConnect.modelNames`, `plugin/__init__.py:AnkiConnect.modelNamesAndIds`, `plugin/__init__.py:AnkiConnect.createModel`, `plugin/__init__.py:AnkiConnect.updateModelTemplates`, `plugin/__init__.py:AnkiConnect.updateModelStyling`, `plugin/__init__.py:AnkiConnect.modelNameFromId`, `plugin/__init__.py:AnkiConnect.modelFieldNames`, `plugin/__init__.py:AnkiConnect.modelFieldDescriptions`, `plugin/__init__.py:AnkiConnect.modelFieldFonts`, `plugin/__init__.py:AnkiConnect.modelFieldsOnTemplates`, `plugin/__init__.py:AnkiConnect.modelTemplates`, `plugin/__init__.py:AnkiConnect.modelStyling`, `plugin/__init__.py:AnkiConnect.findAndReplaceInModels`, `plugin/__init__.py:AnkiConnect.modelTemplateRename`, `plugin/__init__.py:AnkiConnect.modelTemplateReposition`, `plugin/__init__.py:AnkiConnect.modelTemplateAdd`, `plugin/__init__.py:AnkiConnect.modelTemplateRemove`, `plugin/__init__.py:AnkiConnect.modelFieldRename`, `plugin/__init__.py:AnkiConnect.modelFieldReposition`, `plugin/__init__.py:AnkiConnect.modelFieldAdd`, `plugin/__init__.py:AnkiConnect.modelFieldRemove`, `plugin/__init__.py:AnkiConnect.modelFieldSetFont`, `plugin/__init__.py:AnkiConnect.modelFieldSetFontSize`, `plugin/__init__.py:AnkiConnect.modelFieldSetDescription`
- **Story: Manage Media Files**
  - As an external application, I want to store, retrieve, and delete media files associated with notes.
  - **Main entry points:** `plugin/__init__.py:AnkiConnect.storeMediaFile`, `plugin/__init__.py:AnkiConnect.retrieveMediaFile`, `plugin/__init__.py:AnkiConnect.deleteMediaFile`, `plugin/__init__.py:AnkiConnect.getMediaFilesNames`, `plugin/__init__.py:AnkiConnect.getMediaDirPath`
- **Story: Manage Tags**
  - As an external application, I want to add, remove, and replace tags on notes.
  - **Main entry points:** `plugin/__init__.py:AnkiConnect.addTags`, `plugin/__init__.py:AnkiConnect.removeTags`, `plugin/__init__.py:AnkiConnect.getTags`, `plugin/__init__.py:AnkiConnect.clearUnusedTags`, `plugin/__init__.py:AnkiConnect.replaceTags`, `plugin/__init__.py:AnkiConnect.replaceTagsInAllNotes`, `plugin/__init__.py:AnkiConnect.getNoteTags`

### Epic: Anki State and Statistics Access

- **Story: Get Review Statistics**
  - As an external application, I want to retrieve statistics about cards reviewed today or by day.
  - **Main entry points:** `plugin/__init__.py:AnkiConnect.getNumCardsReviewedToday`, `plugin/__init__.py:AnkiConnect.getNumCardsReviewedByDay`, `plugin/__init__.py:AnkiConnect.cardReviews`, `plugin/__init__.py:AnkiConnect.getReviewsOfCards`, `plugin/__init__.py:AnkiConnect.getLatestReviewID`, `plugin/__init__.py:AnkiConnect.insertReviews`
- **Story: Get Collection Statistics**
  - As an external application, I want to retrieve HTML reports of the entire Anki collection's statistics.
  - **Main entry points:** `plugin/__init__.py:AnkiConnect.getCollectionStatsHTML`
- **Story: Get Profile Information**
  - As an external application, I want to retrieve information about Anki profiles and switch between them.
  - **Main entry points:** `plugin/__init__.py:AnkiConnect.getProfiles`, `plugin/__init__.py:AnkiConnect.getActiveProfile`, `plugin/__init__.py:AnkiConnect.loadProfile`
- **Story: Synchronize Collection**
  - As an external application, I want to trigger a synchronization of the Anki collection with AnkiWeb.
  - **Main entry points:** `plugin/__init__.py:AnkiConnect.sync`

Sources: `plugin/__init__.py`, `plugin/web.py`, `plugin/edit.py`, `plugin/util.py`

