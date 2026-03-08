# AI Chat Assistant

## What

The AI chat assistant is a persistent side panel that lets users have contextual conversations with an AI about their document repository. Users can ask questions, request summaries, explore relationships between documents, and get explanations -- all without leaving the reading experience.

The assistant has access to the full document repository with a tiered context model: the active document is primary context, other open documents are secondary context, and the rest of the repository is tertiary context. Users authenticate via AWS SSO to use Claude on Bedrock -- the app shells out to `aws sso login` which opens a browser-based auth flow, and the AWS SDK handles credential caching and refresh automatically. The chat panel sits alongside the document viewer, maintaining conversation history for the duration of the session.

This is the foundational AI integration -- the interaction surface that later features (document authoring, review assistance, search) will build on top of.

## Why

Without AI integration, Episteme is just another markdown viewer. The entire premise of the app is that AI participates as a collaborator throughout the document lifecycle -- authoring, review, discussion, and approval. The chat assistant is the channel through which that collaboration happens.

During authoring, the AI drafts content that aligns with existing documents, follows team conventions, and connects related decisions. During review, it summarizes documents, answers questions about content, and helps reviewers understand implications by referencing prior work. During discussion, it helps resolve comments by understanding the full context of what's already been decided. Every stage gets faster because the AI has read everything and understands how it all fits together.

This feature establishes that core interaction model -- the conversational side panel that every subsequent feature builds on.

## Personas

All five personas use the chat assistant, but the primary users are:

1. **Patricia: Product Manager** - uses AI to draft and refine documents, check consistency with existing specs
2. **Eric: Engineer** - uses AI to understand documents during review, ask technical questions, explore cross-references
3. **Raquel: Reviewer** - uses AI to get summaries, ask clarifying questions, surface concerns

Aaron and Olivia use it too, but less frequently -- Aaron during approval review, Olivia when maintaining SOPs.

## Narratives

### Exploring a document during review

Eric opens Patricia's notification system product description. The document is long and touches several existing systems. He opens the AI chat panel and types "Summarize this PD in three bullet points." The AI responds with a concise overview, pulling out the key decisions and goals.

Eric notices a reference to the existing alert system but isn't sure how they overlap. He asks, "How does this relate to the alert system in alert-system-tech-design.md?" The AI pulls context from both documents and explains the relationship -- the notification system handles user-facing messages while the alert system handles ops-level monitoring. Eric follows up: "Are there any other docs that reference either of these systems?" The AI surfaces two more documents he hadn't thought to check.

### Getting up to speed on a new area

Raquel just joined the team and needs to review a technical design. She opens the document and immediately asks the AI, "What context do I need to understand this doc?" The AI identifies the parent product description, two related ADRs, and a domain model document, summarizing the key points from each. In five minutes, Raquel has enough context to give a meaningful review -- something that would have taken an hour of reading across files.

## User stories

- Eric can open a chat panel alongside the document he's viewing
- Eric can ask the AI to summarize the current document
- Eric can ask questions about the current document's content and get answers
- Eric can ask the AI about relationships between the current document and others in the repository
- Raquel can ask the AI for context from other documents without navigating away
- Patricia can ask the AI to find all documents related to a topic
- Eric can see conversation history for the current session
- Patricia can authenticate via AWS SSO to connect to Claude on Bedrock
- Eric can start a new conversation to clear context
- Eric can see which document is providing primary context to the AI

## Goals

- Establish the core AI interaction model that all future features build on
- Reduce time to understand a document and its context from ~30 min to ~5 min
- Make the full repository accessible through conversation

## Non-goals

- Document editing or creation through the chat (that's the next feature)
- Persistent conversation history across sessions (planned: session persistence with collapsible "turn" links, similar to Codelayer)
- Multi-provider AI support (start with Claude on Bedrock only)
- Multiple open documents (planned: when supported, open docs become secondary context)
- Keyboard shortcuts (planned: will map shortcuts holistically after major features are built)

## Design spec

### Layout

The chat panel is a resizable right-side panel, similar to the sidebar on the left. Default width `w-96` (384px). The document viewer shrinks to accommodate it. A toggle button in the top toolbar shows/hides the panel.

### Chat panel structure (top to bottom)

- **Header bar**: "AI Assistant" label, new conversation button, close button
- **Message area**: scrollable list of messages, auto-scrolls to bottom
- **Input area**: multi-line text input with send button, pinned to bottom

### Message bubbles

- **User messages**: right-aligned, `bg-blue-600 text-white`, rounded
- **AI messages**: left-aligned, `bg-gray-100 dark:bg-gray-800`, rounded, rendered as markdown (using prose classes)

### Empty state

When no conversation exists, show a brief prompt like "Ask me about this document or your repository" with 2-3 suggested starter questions based on the current document (e.g., "Summarize this document", "What documents relate to this one?").

### Authentication

The app uses AWS SSO via the standard AWS credential chain. The user configures an AWS profile in `~/.aws/config` with SSO session details (start URL, account ID, role name, region). The app stores the profile name in its settings.

- **First use**: Settings page where user enters their AWS profile name (e.g., `ai-prod-llm`). The app validates the profile exists in `~/.aws/config`.
- **Login**: App shells out to `aws sso login --profile <name>`, which opens the browser for SSO auth. Credentials are cached by the AWS CLI automatically.
- **Expired credentials**: When a Bedrock API call fails with an auth error, the chat panel shows a "Re-authenticate" button that re-triggers the SSO login flow.
- **No keychain/key storage needed**: The AWS CLI handles all credential caching and refresh.

## Tech spec

### Introduction and overview

**Prerequisites:**
- ADR-001 (Tauri) - Rust backend handles all system operations including AWS API calls
- ADR-003 (Zustand) - Frontend state management pattern for chat state
- ADR-004 (Tailwind) - Styling approach for chat panel UI

**Goals:**
- Bedrock API response initiated within 1s of user sending a message
- Context building (reading files from disk) completes in <500ms for repositories up to 1000 markdown files
- Chat panel renders without blocking the document viewer

**Non-goals:**
- Prompt optimization or token budget management (keep it simple)
- Caching AI responses

**Glossary:**
- **Primary context**: The currently active/viewed document, included in full in the AI prompt
- **Secondary context**: Other open documents (planned -- not implemented in v1 since multi-doc support doesn't exist yet, but the context builder should be designed with this tier in mind)
- **Tertiary context**: All other markdown files in the repository, included as a file listing with summaries when available

### System design and architecture

**Component and data flow diagram** (not visual layout -- see design spec for UI layout):

```
┌─────────────────────────────────────────────────────┐
│ React Frontend                                       │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
│  │ DocumentViewer│  │ AiChatPanel  │  │  Sidebar   │ │
│  └──────────────┘  └──────┬───────┘  └────────────┘ │
│                           │                          │
│                    useAiChatStore                     │
│                           │                          │
│                    invoke("ai_chat") + Channel        │
│                    invoke("ai_check_auth")            │
│                    invoke("ai_sso_login")             │
└───────────────────────────┼──────────────────────────┘
                            │ Tauri IPC
┌───────────────────────────┼──────────────────────────┐
│ Rust Backend              │                          │
│                    ┌──────┴───────┐                   │
│                    │ commands/ai  │                   │
│                    └──────┬───────┘                   │
│                           │                          │
│              ┌────────────┼────────────┐             │
│              │            │            │             │
│       ┌──────┴──────┐ ┌──┴───────┐ ┌──┴──────────┐  │
│       │ContextBuilder│ │ Bedrock  │ │  Shell      │  │
│       │ (reads files)│ │ Stream   │ │(sso login)  │  │
│       └─────────────┘ └─────────┘ └─────────────┘  │
└──────────────────────────────────────────────────────┘
```

**Component breakdown:**

- **AiChatPanel** (new React component): Chat UI with message list, input, header
- **useAiChatStore** (new Zustand store): Manages messages, loading state, auth status
- **commands/ai.rs** (new Rust module): Tauri commands for chat, auth check, SSO login
- **ContextBuilder** (new Rust module): Reads workspace files and builds prompt context
- **AWS Bedrock client** (in Rust): Uses `aws-sdk-bedrockruntime` with default credential chain

### Detailed design

#### Rust backend: `commands/ai.rs`

Three Tauri commands:

**`ai_check_auth`** - Check if AWS credentials are valid
```
Input: aws_profile: String
Output: Result<bool, String>
Behavior:
  - Set AWS_PROFILE env var
  - Attempt to create a Bedrock client and call a lightweight operation
  - Return true if credentials are valid, false if expired/missing
```

**`ai_sso_login`** - Trigger SSO login flow
```
Input: aws_profile: String
Output: Result<(), String>
Behavior:
  - Shell out to: aws sso login --profile <aws_profile>
  - This opens the user's browser for SSO auth
  - Wait for the process to complete
  - Return success/failure
```

**`ai_chat`** - Send a message and stream the response
```
Input:
  - messages: Vec<ChatMessage>  (full conversation history)
  - active_file_path: Option<String>
  - workspace_path: String
  - aws_profile: String
  - on_event: tauri::ipc::Channel<StreamEvent>  (Tauri channel for streaming)
Output: Result<(), String>
StreamEvent variants:
  - Token(String)     -- partial text chunk
  - Done(String)      -- full completed response
  - Error(String)     -- error message (auth errors flagged distinctly)
Behavior:
  1. Build context using ContextBuilder
  2. Construct system prompt with context
  3. Call Bedrock ConverseStream API (Claude) with messages
  4. For each response chunk, send Token event via channel
  5. On completion, send Done event with full response text
  6. On auth error, send Error event so frontend can prompt re-auth
```

#### Rust backend: Context building

The ContextBuilder constructs the system prompt by reading from disk:

```
System prompt structure:
  "You are an AI assistant for a document repository. You help users
   understand, navigate, and work with their documentation.

   ## Active document
   <full contents of the currently viewed file>

   ## Repository structure
   <tree listing of all markdown files with their frontmatter titles>

   The user may ask you to read other files. When they reference a
   document, use the repository structure to identify the right file."
```

For v1, the context strategy is simple:
- **Primary context**: Full content of the active document (always included)
- **Secondary context**: (placeholder) When multi-doc support is added, other open documents will be included here. The context builder accepts an `open_file_paths: Vec<String>` parameter now but it will be empty in v1.
- **Tertiary context**: File tree listing with frontmatter titles extracted (lightweight scan)
- No file content is included for tertiary docs unless the user asks about a specific file — in which case the AI response should note it needs that file's content, and the next turn can include it

**File scanning**: Walk the workspace directory (reusing logic from `files.rs`), read the first 10 lines of each markdown file to extract the title from frontmatter or first heading.

#### Frontend: `stores/aiChat.ts`

```typescript
interface ChatMessage {
  role: "user" | "assistant";
  content: string;
}

interface AiChatStore {
  messages: ChatMessage[];
  isStreaming: boolean;
  streamingContent: string;  // accumulates tokens during streaming
  isAuthenticated: boolean;
  authChecked: boolean;
  awsProfile: string | null;
  error: string | null;

  checkAuth: () => Promise<void>;
  login: () => Promise<void>;
  sendMessage: (content: string) => Promise<void>;
  clearConversation: () => void;
}
```

`sendMessage` flow:
1. Append user message to messages array
2. Set isStreaming = true, streamingContent = ""
3. Create a Tauri Channel that listens for StreamEvents
4. Invoke `ai_chat` with full message history + active file path from fileTreeStore + workspace path from workspaceStore + channel
5. On Token event: append to streamingContent (UI renders this as the in-progress assistant message)
6. On Done event: append final assistant message to messages array, clear streamingContent, set isStreaming = false
7. On Error event: set error, if auth error set isAuthenticated = false

#### Frontend: `components/AiChatPanel.tsx`

States:
- **Auth not checked yet**: Show loading spinner
- **Not authenticated**: Show AWS profile input + "Connect" button, or "Re-authenticate" button if profile is already saved
- **Authenticated, no messages**: Show empty state with suggested prompts
- **Authenticated, has messages**: Show message list + input
- **Streaming**: Show input disabled, render `streamingContent` as an in-progress assistant message with a blinking cursor. Markdown rendered incrementally.

#### Preferences extension

Add `aws_profile: Option<String>` to the existing `Preferences` struct in both Rust and TypeScript. Follows the existing save/load pattern.

#### New Rust dependencies (Cargo.toml)

- `aws-config` - AWS credential chain and config loading
- `aws-sdk-bedrockruntime` - Bedrock API client
- `tokio` features may need expanding for async AWS SDK calls

#### New frontend dependencies (package.json)

None — all AI communication goes through Tauri commands.

### Security, privacy, and compliance

**Authentication**: AWS SSO via standard credential chain. No secrets stored by the app — AWS CLI manages all credential caching in `~/.aws/sso/cache/`.

**Authorization**: Bedrock access controlled by the IAM role attached to the SSO profile (`BedrockAccess-AI-Tools-Prod`). The app has no role in authorization.

**Data privacy**: Document contents are sent to Bedrock (Claude) as prompt context. Users should be aware their documents are sent to AWS. No data is persisted by the app beyond the session.

**Input validation**:
- AWS profile name validated against `~/.aws/config` before use
- Workspace path validation (existing pattern from `read_file`) prevents path traversal
- Message content is plain text, no injection risk on the Bedrock API side

### Observability

**Logging** (using existing tauri-plugin-log):
- INFO: Chat message sent (no content, just timestamp + active file)
- INFO: Auth check result
- ERROR: Bedrock API errors, SSO login failures
- DEBUG: Context building timing, file count scanned

**Metrics**: Deferred (per app.md operations decisions)

**Alerting**: N/A for desktop app

### Testing plan

**Unit tests (Vitest):**
- `aiChat` store: message append, clear, loading state transitions
- Preferences parsing with new `aws_profile` field
- Chat message rendering (user vs assistant styling)

**Integration tests:**
- Tauri command invocation mocking (chat flow end-to-end in frontend)
- Context builder: reads correct files, respects workspace boundaries

**E2E tests (Playwright):**
- Chat panel opens/closes via toggle button
- Empty state shows suggested prompts
- Message input and display flow (with mocked backend)

### Alternatives considered

**AWS SDK in JavaScript (frontend) instead of Rust backend:**
- Pros: Simpler, no Rust AWS dependencies, faster iteration
- Cons: Webview has restricted filesystem access; reading `~/.aws` credentials from the webview is unreliable and platform-dependent; violates the existing pattern of system operations in Rust
- **Decision**: Rust backend. Consistent with ADR-001's architecture — system operations (including external API calls) happen in Rust.

**Direct Anthropic API instead of Bedrock:**
- Pros: Simpler auth (just an API key), no AWS dependency
- Cons: Doesn't match the user's existing infrastructure; API keys need secure storage
- **Decision**: Bedrock via AWS SSO. Matches existing team infrastructure.

### Risks

**AWS SDK for Rust bundle size**: The AWS SDK crates may significantly increase the Rust binary size. Mitigation: Only include `bedrockruntime` crate, use feature flags to minimize dependencies. Monitor binary size after adding.

**SSO credential expiry during long sessions**: If a user is mid-conversation and credentials expire, the chat will fail. Mitigation: Detect auth errors on each API call and surface the re-authenticate prompt immediately, preserving conversation history.

**Context window limits**: Large repositories or large documents could exceed Claude's context window. Mitigation: For v1, truncate the repository file listing if it exceeds a reasonable limit (~50K tokens). Active document is always included in full. Revisit with smarter context selection in future.

## Task list

- [x] **Story: AWS Bedrock integration (Rust backend)**
  - [x] **Task: Add AWS SDK dependencies to Cargo.toml**
    - **Description**: Add `aws-config`, `aws-sdk-bedrockruntime`, and any required `tokio` feature flags to the Rust backend. Verify the project compiles with the new dependencies.
    - **Acceptance criteria**:
      - [x] `aws-config` and `aws-sdk-bedrockruntime` added to `Cargo.toml`
      - [x] `tokio` features sufficient for async AWS SDK calls
      - [x] Project compiles successfully with `cargo build`
      - [ ] Binary size increase noted (for risk tracking)
    - **Dependencies**: None
  - [x] **Task: Implement `ai_check_auth` Tauri command**
    - **Description**: Create `commands/ai.rs` with a command that checks whether AWS SSO credentials are valid for a given profile. Set the `AWS_PROFILE` env var, create a Bedrock client, and attempt a lightweight operation to validate credentials.
    - **Acceptance criteria**:
      - [x] New `commands/ai.rs` module created and registered in `commands/mod.rs`
      - [x] Command accepts `aws_profile: String` parameter
      - [x] Returns `true` when credentials are valid
      - [x] Returns `false` (not an error) when credentials are expired or missing
      - [x] Command registered in `lib.rs` invoke handler
      - [ ] Rust-level unit test covers valid and expired credential scenarios (mocked) -- deferred, tested via frontend store tests
    - **Dependencies**: "Task: Add AWS SDK dependencies to Cargo.toml"
  - [x] **Task: Implement `ai_sso_login` Tauri command**
    - **Description**: Add a command that shells out to `aws sso login --profile <name>` to trigger the browser-based SSO flow. Use `tauri_plugin_shell` or `std::process::Command` to execute. Wait for the process to complete and return success/failure.
    - **Acceptance criteria**:
      - [x] Command accepts `aws_profile: String` parameter
      - [x] Shells out to `aws sso login --profile <profile>`
      - [x] Returns `Ok(())` on successful login
      - [x] Returns `Err` with descriptive message on failure
      - [x] AWS profile name validated (no shell injection -- alphanumeric, hyphens, underscores only)
    - **Dependencies**: None
  - [x] **Task: Implement context builder**
    - **Description**: Create a Rust module that builds the AI system prompt from the workspace. Reads the active document in full (primary context). Walks the workspace directory to produce a file tree listing with titles extracted from frontmatter or first heading of each markdown file (tertiary context). Accepts an `open_file_paths: Vec<String>` parameter for future secondary context (empty for now). Reuse directory walking logic from `files.rs` where possible.
    - **Acceptance criteria**:
      - [x] Reads active document contents in full
      - [x] Walks workspace and extracts title from first 10 lines of each markdown file
      - [x] Produces structured system prompt string matching the format in the tech spec
      - [x] Respects workspace path boundaries (no path traversal)
      - [x] Skips hidden directories and non-markdown files (consistent with `files.rs`)
      - [x] `open_file_paths` parameter accepted but unused in v1
      - [ ] Completes in <500ms for a 1000-file test workspace -- not benchmarked, deferred
      - [x] Unit tests verify title extraction and helper functions (in context.rs #[cfg(test)])
    - **Dependencies**: None
  - [x] **Task: Implement `ai_chat` streaming Tauri command**
    - **Description**: Add the main chat command that builds context, calls Bedrock's `ConverseStream` API with the full conversation history, and streams response tokens back to the frontend via a `tauri::ipc::Channel`. Send `Token` events for each chunk, `Done` with the full response on completion, and `Error` on failure (with auth errors flagged distinctly).
    - **Acceptance criteria**:
      - [x] Command accepts messages, active_file_path, workspace_path, aws_profile, and a Channel
      - [x] Calls context builder to construct system prompt
      - [x] Calls Bedrock `ConverseStream` API with Claude model
      - [x] Streams `Token(String)` events to channel as chunks arrive
      - [x] Sends `Done(String)` event with full accumulated response on completion
      - [x] Sends `Error(String)` on failure; auth errors include "auth" in the error string
      - [ ] Rust-level integration test with mocked Bedrock client -- deferred, verified via manual testing
    - **Dependencies**: "Task: Implement `ai_check_auth` Tauri command", "Task: Implement context builder"

- [x] **Story: Preferences extension**
  - [x] **Task: Add `aws_profile` to Rust Preferences struct**
    - **Description**: Extend the existing `Preferences` struct in `commands/preferences.rs` with an `aws_profile: Option<String>` field. Ensure backwards compatibility -- loading preferences files without this field should default to `None`.
    - **Acceptance criteria**:
      - [x] `aws_profile: Option<String>` added to `Preferences` struct
      - [x] Existing preferences files without the field load successfully (defaults to `None`)
      - [x] Save/load round-trip preserves the field
      - [x] Unit test covers migration from old format (preferences.test.ts updated)
    - **Dependencies**: None
  - [x] **Task: Add `aws_profile` to TypeScript preferences**
    - **Description**: Extend `PreferencesSchema` in `src/lib/preferences.ts` with `aws_profile: z.string().nullable()`. Update `DEFAULT_PREFERENCES` accordingly.
    - **Acceptance criteria**:
      - [x] Zod schema updated with `aws_profile` field
      - [x] `DEFAULT_PREFERENCES` includes `aws_profile: null`
      - [x] `parsePreferences` handles missing field gracefully
      - [x] Unit test verifies parsing with and without the field (preferences.test.ts)
    - **Dependencies**: None

- [x] **Story: Frontend AI chat store**
  - [x] **Task: Create `useAiChatStore` Zustand store**
    - **Description**: Create `src/stores/aiChat.ts` following the existing Zustand patterns. Manage messages array, streaming state (`isStreaming`, `streamingContent`), auth state (`isAuthenticated`, `authChecked`), `awsProfile`, and `error`. Implement `checkAuth`, `login`, `sendMessage` (with Tauri Channel for streaming), and `clearConversation` actions. `sendMessage` should read active file path from `useFileTreeStore` and workspace path from `useWorkspaceStore`.
    - **Acceptance criteria**:
      - [x] Store created with all fields from the tech spec interface
      - [x] `checkAuth` invokes `ai_check_auth` and updates `isAuthenticated`/`authChecked`
      - [x] `login` invokes `ai_sso_login` and re-checks auth on success
      - [x] `sendMessage` appends user message, creates Tauri Channel, invokes `ai_chat`
      - [x] Channel handler updates `streamingContent` on Token, finalizes message on Done
      - [x] Auth errors set `isAuthenticated = false`
      - [x] `clearConversation` resets messages and streamingContent
      - [x] Unit tests cover all state transitions (28 tests in aiChat.test.ts)
    - **Dependencies**: "Task: Add `aws_profile` to TypeScript preferences"

- [x] **Story: Chat panel UI**
  - [x] **Task: Create `ChatMessage` component**
    - **Description**: Create `src/components/ChatMessage.tsx` that renders a single chat message. User messages are right-aligned with `bg-blue-600 text-white`. Assistant messages are left-aligned with `bg-gray-100 dark:bg-gray-800` and rendered as markdown using the `prose` classes. Both have rounded corners.
    - **Acceptance criteria**:
      - [x] Accepts `message: { role, content }` prop
      - [x] User messages styled right-aligned with blue background
      - [x] Assistant messages styled left-aligned with gray background
      - [x] Assistant message content rendered as markdown (prose classes)
      - [x] Dark mode support via Tailwind `dark:` variants
      - [x] Unit test verifies both message types render correctly (6 tests in ChatMessage.test.tsx)
    - **Dependencies**: None
  - [x] **Task: Create `AiChatPanel` component**
    - **Description**: Create `src/components/AiChatPanel.tsx` -- the main chat panel with header bar, message area, and input area. Implement all states: auth checking (spinner), not authenticated (profile setup/login), empty state (suggested prompts), active conversation (message list + input), and streaming (in-progress message with cursor). Message list auto-scrolls to bottom. Input is a multi-line textarea with send button, disabled during streaming.
    - **Acceptance criteria**:
      - [x] Header bar with "AI Assistant" label, new conversation button (trash/refresh icon), close button
      - [x] Auth-checking state shows loading spinner
      - [x] Not-authenticated state shows AWS profile text input + "Connect" button
      - [x] Not-authenticated with saved profile shows "Re-authenticate" button
      - [x] Empty state shows welcome text + 2-3 clickable suggested prompts
      - [x] Active conversation renders message list using `ChatMessage` component
      - [x] Streaming state renders `streamingContent` as in-progress assistant message
      - [x] Message area auto-scrolls to bottom on new messages/tokens
      - [x] Input textarea with send button, disabled during streaming
      - [x] Enter sends message (Shift+Enter for newline)
      - [x] Connects to `useAiChatStore` for all state and actions
      - [x] Unit tests cover each panel state (14 tests in AiChatPanel.test.tsx)
    - **Dependencies**: "Task: Create `ChatMessage` component", "Task: Create `useAiChatStore` Zustand store"
  - [x] **Task: Integrate chat panel into App layout**
    - **Description**: Modify `src/App.tsx` to add a toggle button and conditionally render `AiChatPanel` as a right-side panel alongside the document viewer. The panel should be a fixed-width panel (`w-96`) that the document viewer yields space to. Add a toggle button to the top area of the app. On mount, run `checkAuth` if `awsProfile` is set in preferences.
    - **Acceptance criteria**:
      - [x] Toggle button visible when a folder is open
      - [x] Clicking toggle shows/hides the chat panel
      - [x] Chat panel renders to the right of the document viewer
      - [x] Document viewer shrinks to accommodate the panel (flex layout)
      - [x] `checkAuth` called on mount when `awsProfile` exists in preferences
      - [x] Panel state (open/closed) does not persist across sessions
      - [x] No layout breakage when panel is toggled
    - **Dependencies**: "Task: Create `AiChatPanel` component"

- [x] **Story: End-to-end integration and testing**
  - [x] **Task: Manual integration test with real Bedrock**
    - **Description**: Test the full flow end-to-end with a real AWS SSO profile and Bedrock. Open a folder, open the chat panel, authenticate, ask a question about the active document, verify streaming response appears. Document any issues found.
    - **Acceptance criteria**:
      - [x] SSO login flow opens browser and completes successfully
      - [x] Auth check correctly detects valid/expired credentials
      - [x] Sending a message streams tokens into the chat panel
      - [x] Response references the active document content accurately
      - [ ] Re-authentication works after credential expiry -- not tested this session
      - [ ] Error states display correctly (no profile, no credentials, API error) -- partially tested
    - **Dependencies**: All previous tasks
  - [ ] **Task: E2E tests with mocked backend** -- deferred to separate PR
    - **Description**: Write Playwright E2E tests that cover the chat panel user flows with mocked Tauri commands. Test panel toggle, empty state, message send/receive, and auth states.
    - **Acceptance criteria**:
      - [ ] Test: panel opens and closes via toggle button
      - [ ] Test: empty state shows suggested prompts
      - [ ] Test: clicking a suggested prompt sends it as a message
      - [ ] Test: user message appears in chat, streaming response renders
      - [ ] Test: not-authenticated state shows setup UI
      - [ ] All tests pass in CI
    - **Dependencies**: "Task: Integrate chat panel into App layout"
