# Enhancement: @-reference files in AI chat

## Parent feature

`[AI chat panel](feature-ai-chat-panel.md)`

## What

Users can type `@` in the chat input to trigger an inline file picker that lets them explicitly include a specific file from the repository as context for their message. The referenced file is included in full in the next AI request, alongside the active document. The file reference appears in the message bubble as a chip showing the filename.

## Why

The current context model automatically includes the active document and a file tree listing, but users have no way to pull in content from a specific file they know is relevant. When asking the AI to compare two documents, or to answer a question that requires reading a specific file, users have to navigate to that file first or describe its contents themselves. @-referencing closes this gap and makes the AI substantially more useful for cross-document work.

## User stories

- Eric can type `@` in the chat input and see a filterable list of files in the repository
- Eric can select a file from the list and see it appear as a chip in his message
- Eric can @-reference multiple files in a single message
- Eric can remove a @-referenced file before sending by clicking the chip's dismiss button
- The AI receives the full content of @-referenced files alongside the active document

## Design changes

*(Added by design specs stage)*

### Input area changes

The chat input grows an @-mention trigger. Typing `@` opens a floating autocomplete popup above the input, showing repository files filtered by the characters typed after `@`. Selecting a file closes the popup and inserts a `@filename` chip inline in the input. Chips are visually distinct from plain text (pill shape, light blue background). The send button remains in the same position.

### Message bubble changes

Sent messages show @-referenced file chips below the message text, each displaying the filename. AI responses don't change visually.

## Technical changes

### Affected files

- `src/components/AiChatPanel.tsx` — add @-mention trigger logic, chip rendering, file picker popup
- `src/stores/aiChat.ts` — extend `sendMessage` to include referenced file paths, extend `ChatMessage` type to carry attachments
- `src-tauri/src/commands/ai.rs` — extend `ai_chat` command to accept `referenced_file_paths: Vec<String>` and include those files' contents in the context builder call
- `src-tauri/src/context.rs` — extend `ContextBuilder` to accept and inline referenced file contents

### Changes

*(Added by tech specs stage)*

The `ChatMessage` type gains an optional `attachments: Vec<string>` field (file paths). `sendMessage` collects the chip state from the input component and passes it as a new `referenced_file_paths` parameter to the `ai_chat` Tauri command. The context builder inlines the full content of each referenced file in a new `## Referenced files` section of the system prompt, between the active document and the repository structure listing.

## Task list

*(Added by task decomposition stage)*
