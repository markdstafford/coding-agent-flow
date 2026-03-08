# Episteme

## What

Episteme is a document authoring application for teams working on collaborative documentation. Teams use it to draft, review, discuss, and approve documents—whether product descriptions, technical designs, standard operating procedures, or wiki-style knowledge bases. The application guides documents through the full lifecycle: authoring, gathering feedback through comments, resolving discussions, and obtaining formal sign-offs.

What makes Episteme different is that AI participates as a collaborator throughout this entire workflow. When documents are stored as structured, interconnected content, AI understands how they relate to each other, follows your team's templates and conventions, and has full context of existing decisions. This means AI can help draft new content that aligns with what you've already written, show relevant related documents during review, and assist with discussions by understanding your team's prior decisions and patterns.

Teams get both a structured workflow for moving documents from draft to approved, and AI assistance that's deeply aware of their full document collection and how everything connects.

## Why

Writing good documentation requires deep thinking about critical decisions—what are we building and why, how should this system work, what trade-offs matter. But authors spend mental energy on mechanical tasks: which template to use, what process to follow, correct grammar and structure, connecting related documents. This cognitive load pulls focus away from the actual problem.

Episteme delegates all the mechanical work to AI—templates, process, grammar, interconnections—so humans can spend their mind space entirely on critical thinking. For a product manager, that means focusing on what to build and why. For an engineer, it's thinking through system design and trade-offs. AI lifts all boats by handling everything else, making all documents better while humans just think about the problem.

## Personas

1. **Patricia: Product Manager** - Drafts product descriptions and requirements, coordinates review cycles, manages sign-off process
2. **Eric: Engineer** - Writes technical designs and architecture documents, reviews others' work, implements based on approved docs
3. **Raquel: Reviewer** - Team member who provides feedback and comments on documents across different areas
4. **Aaron: Approver** - Senior team member or stakeholder who provides final sign-off on documents before they're considered accepted
5. **Olivia: Operations Lead** - Creates and maintains standard operating procedures, process documentation, and team runbooks

## Narratives

### Creating a new document

Patricia opens Episteme and sees her team's document repository in the left sidebar—product descriptions, technical designs, and architecture decisions organized by project. She clicks "Create new document" and an AI chat panel appears, asking what she wants to create.

"I need to write a PD for our new notification system," Patricia types. The AI immediately understands this means a product description template, determines the right location in the repository structure, and begins guiding her through the process. "Let's start with what you're building. In 1-2 sentences, what is the notification system?" Patricia responds with her core idea, and the AI expands it into a full What section, presenting it for approval. Patricia reads the draft and notices it's too technical—she asks the AI to rewrite it focusing on user outcomes instead of implementation. The revised version captures her intent perfectly, and she approves it.

The AI moves to Why, then Personas, then Narratives, asking targeted questions at each step. Patricia never touches the markdown document itself; she's only thinking about critical decisions—what to build, why it matters, who it's for. The AI handles template structure, formatting, and building the actual document.

After 30 minutes of focused conversation about the problem space, Patricia clicks "Publish draft." Before publishing, the AI runs a review pass, checking the document against the repository. It flags that the existing alert system isn't mentioned: "I noticed we have an alert system documented in `alert-system-tech-design.md`. Should I add a note about how this notification system relates to it?" Patricia clarifies the relationship, and the AI adds a brief explanation to the Why section. With the review complete, the AI creates a branch, commits the document, and pushes to the remote repository. Raquel and Eric are automatically notified that a new product description is ready for review.

### Reviewing and commenting

Eric receives a notification that Patricia's notification system product description is ready for review. He opens the document in Episteme and sees the full markdown in the main pane with an AI chat panel on the side. He clicks "Summarize" and the AI provides a three-paragraph overview of the PD's key points.

As Eric reads through the document, he has questions. He asks the AI, "What's the expected latency for these notifications?" The AI explains the performance considerations mentioned in the Goals section, referencing specific targets. Eric's question is answered without creating any comment. Later, he notices the personas section doesn't include the ops team who will need to configure notifications. He tells the AI, "The ops team should be a persona here—they'll be configuring notification rules." The AI recognizes this is a real issue and automatically creates a GitHub discussion attached to the Personas section with Eric's concern.

Eric continues through the document, chatting with the AI about various aspects. Most of his questions get answered immediately from document context. Three genuine issues surface, and the AI creates discussions for each at the appropriate location. When Eric finishes his review, Patricia receives a notification that comments have been added to her draft.

### Resolving comments and approval

Patricia receives a notification that Eric and Raquel have completed their reviews. She opens her draft and sees three discussion threads flagged in the document. She clicks on Eric's comment about the ops persona and a chat panel opens beneath it. "Add Olivia as an ops lead persona—she'll handle notification rule configuration and monitoring," Patricia types. The AI drafts a new persona and adds it to the Personas section, highlighting the changes in yellow so Patricia can see exactly what was added.

Patricia works through the remaining two comments, chatting with the AI to resolve each one. For Raquel's concern about notification frequency limits, the AI updates the Goals section with specific rate-limiting targets. Patricia reviews each change as it's highlighted, ensuring the resolutions match her intent. When all discussions are resolved, she clicks "Send for approval."

The AI creates a pull request attempting to merge her branch to main. Aaron receives a notification requesting his approval on the notification system PD. He opens the approval view and sees a summary of the document's key points, followed by a summary of the review process—what issues were raised and how Patricia resolved them. Aaron reads through the resolution summaries, satisfied that the concerns were addressed properly. He clicks "Approve," and the pull request merges to main. The notification system product description is now official, visible to the entire team in the repository.

## High-level requirements

### Platform decisions

- **Initial release**: Desktop app for macOS
- **Phase 2**: Windows desktop app
- **Phase 3**: Web application (modern browsers only)
- **Phase 4**: Mobile apps (iOS/Android)
- **Phase 5**: Terminal/CLI application
- **Deployment**: Local desktop apps with cloud backend for document sync and collaboration
- **Browser support** (web phase): Chrome, Firefox, Safari, Edge - latest versions only

### Architecture decisions

- **Application architecture**: Local-first embedded architecture (frontend + backend in single app)
- **Frontend framework**: React 18+ with TypeScript
- **Desktop framework**: Tauri (see ADR-001)
- **Backend logic** (embedded): Rust (Tauri backend)
- **Document editor**: TipTap (see ADR-002)
- **Real-time collaboration** (Phase 2): Y.js CRDT with WebSocket sync server (Hocuspocus or y-websocket)
- **Git integration**: Shell out to git CLI (requires git in PATH)
- **AI integration**: Claude on Bedrock via AWS SSO (add other providers later)
- **State management**: Zustand (see ADR-003)
- **Styling approach**: Tailwind CSS (see ADR-004)

### Data decisions

- **Primary storage**: Markdown files in git repository
- **Document metadata**: Front matter in markdown files (doc IDs, reviewers, approval status, etc.)
- **Search/indexing** (future feature): Local search index built on-demand (library TBD)
- **Local cache**:
  - Document summaries cached for hover previews
  - Other caching decisions deferred
- **Sensitive data storage**: Delegated to AWS CLI for AI credentials (SSO token cache); OS keychain reserved for future needs
- **Git credentials**: Handled by git CLI
- **User settings/preferences**: Local JSON files in app data directory (added to .gitignore)
- **Real-time collaboration state** (Phase 2): Y.js document state (in-memory with optional persistence - decide later)

### Security decisions

- **Authentication**: GitHub OAuth (see ADR-005)
- **Authorization**: GitHub repository permissions and PR approval workflows (managed by GitHub, not app-level)
- **AI credentials**: AWS SSO via standard AWS credential chain (profile in `~/.aws/config`, `aws sso login` for auth, no app-level key storage)
- **Data protection**: Plain text markdown in git (no encryption at rest needed)
- **Input validation**:
  - Zod for settings/preferences validation (see ADR-006)
  - File path validation to prevent path traversal
- **Markdown sanitization**: DOMPurify or safe markdown parser for XSS prevention when rendering
- **Compliance**: Not applicable for initial release

### Operations decisions

- **Error tracking/monitoring**: Deferred - will add later
- **Logging**: Deferred - will add later
- **Updates/deployment**: Deferred - will add later
- **Testing frameworks**: Vitest for unit/integration tests, Playwright for E2E tests (see ADR-007)
- **CI/CD**: GitHub Actions for automated builds, tests, and multi-platform compilation
- **Backup/recovery**: Git handles document versioning; app-specific backup deferred

## Design guidance

### Typography

**Primary font stack**: System font stack with Helvetica Neue fallback
```
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Helvetica Neue", Arial, sans-serif
```

**Why system fonts**: Top document apps (GitHub, Notion, Linear) use system fonts for native feel, excellent rendering quality, no font loading delays, and perfect readability.

**Type scale** (using Tailwind's built-in scale):
- **Headings**: `text-4xl` (h1), `text-3xl` (h2), `text-2xl` (h3), `text-xl` (h4)
- **Body**: `text-base` (16px)
- **Small**: `text-sm` (14px)
- **Code/mono**: `font-mono` (system monospace stack)

**Line height**: Use Tailwind's `leading-relaxed` (1.625) for body text, `leading-tight` for headings

**Font weights**:
- Regular (400) for body
- Medium (500) for emphasis
- Semibold (600) for subheadings
- Bold (700) for main headings

### Colors

**Approach**: Use Tailwind's built-in color palette with semantic naming

**Light mode palette**:
- **Background**: `bg-white`, `bg-gray-50` (secondary)
- **Text**: `text-gray-900` (primary), `text-gray-600` (secondary)
- **Borders**: `border-gray-200`
- **Accent**: `bg-blue-600` (primary actions), `bg-blue-50` (hover states)

**Dark mode palette**:
- **Background**: `bg-gray-900`, `bg-gray-800` (secondary)
- **Text**: `text-gray-100` (primary), `text-gray-400` (secondary)
- **Borders**: `border-gray-700`
- **Accent**: `bg-blue-500` (primary actions), `bg-blue-900` (hover states)

**Theme switching**: Use Tailwind's `dark:` variant classes for automatic dark mode support

### Markdown rendering

**Use @tailwindcss/typography plugin** (`prose` classes):
- Automatically styles markdown with excellent defaults
- Supports dark mode with `prose-invert`
- Customizable for brand consistency

**Example**:
```html
<div className="prose dark:prose-invert max-w-none">
  {/* rendered markdown */}
</div>
```

### Spacing

Use Tailwind's built-in spacing scale (4px increments):
- **Tight**: `space-y-2` (8px)
- **Normal**: `space-y-4` (16px)
- **Comfortable**: `space-y-6` (24px)
- **Spacious**: `space-y-8` (32px)

**Document layout**:
- Sidebar: `w-64` (256px)
- Main content: `max-w-4xl` (896px) for optimal readability
- Padding: `p-6` (24px) for panels, `p-8` (32px) for main content

### Icons

**Library**: Lucide React (34.9M weekly downloads, clear industry leader)
- Clean, consistent design
- Excellent React integration
- Comprehensive icon set
- Works perfectly with Tailwind sizing classes

**Usage**: `<Icon className="w-5 h-5" />` for standard UI, `w-4 h-4` for inline text

### Component patterns

**Buttons**:
- Primary: `bg-blue-600 hover:bg-blue-700 text-white`
- Secondary: `bg-gray-100 hover:bg-gray-200 text-gray-900`
- Ghost: `hover:bg-gray-100 text-gray-700`

**Panels/Cards**:
- Light: `bg-white border border-gray-200 rounded-lg shadow-sm`
- Dark: `dark:bg-gray-800 dark:border-gray-700`

**Focus states**: Use Tailwind's `focus-visible:ring-2 ring-blue-500` for keyboard navigation accessibility

### Accessibility

- **Color contrast**: Maintain WCAG AA standards (4.5:1 for normal text, 3:1 for large text)
- **Focus indicators**: Always visible for keyboard navigation
- **Semantic HTML**: Use proper heading hierarchy (h1 → h2 → h3)
- **ARIA labels**: For icon-only buttons and interactive elements
- **Keyboard navigation**: All features accessible via keyboard

## Related features

### Initial release features

1. **Baseline desktop app** - Hello world Tauri application with React frontend
2. **Open folder** - Select and open a local folder containing markdown files
3. **Sidebar file browser** - Display folder structure and file list in sidebar
4. **Markdown rendering** - View and render markdown files with TipTap

### Future features

- **Document authoring with AI** - Create new documents through AI-guided conversation following templates
- **AI review pass** - Automatic review of documents before publishing to catch obvious issues
- **Document review workflow** - Comment system with AI-assisted review conversations
- **GitHub integration** - Branch/PR creation, GitHub discussions for comments, approval workflows
- **Approval workflow** - Sign-off process integrated with GitHub PRs
- **AI chat assistant** - Contextual AI help during authoring, review, and approval
- **Templates** - Pre-defined document templates (product descriptions, tech specs, SOPs, ADRs)
- **Document linking and backlinks** - Automatic detection and management of cross-references
- **Document search** - Full-text search across document collection
- **Real-time collaboration** - Multiple users editing simultaneously with Y.js CRDT
- **Document summaries** - AI-generated summaries for hover previews
- **Notifications** - Updates when documents are ready for review or approved
- **Multi-repository support** - Work with multiple document repositories
- **Web application** - Browser-based version of the app
- **Mobile apps** - iOS and Android versions
- **Terminal/CLI** - Command-line interface for power users
