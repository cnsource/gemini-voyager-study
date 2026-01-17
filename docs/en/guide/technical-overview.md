# Technical Overview

This page summarizes the Gemini Voyager tech stack, build workflow, extension architecture, and provides feature-by-feature implementation details for developers.

## Tech Stack

- **Language & UI**: TypeScript + React 19 for modern, component-driven UI.
- **Build System**: Vite 7 with CRXJS for extension packaging.
- **Styling**: Tailwind CSS and class utilities (tailwind-merge).
- **Docs**: VitePress powers the documentation site.
- **Testing**: Vitest is used for unit tests and coverage.

## Development & Build Scripts

Use `bun` or `npm` to run the primary workflows:

- `dev:*`: nodemon-wrapped watch builds for Chrome/Firefox/Safari.
- `build:*`: browser-specific release builds.
- `docs:*`: VitePress dev/build/preview commands.
- `test:*`: Vitest runs, including UI and coverage.

## Entry Points & Initialization Strategy

The content-script entry point is `src/pages/content/index.tsx`, which:

- Detects Gemini/AI Studio/custom domains.
- Staggers initialization for background tabs to avoid burst traffic.
- Starts heavy features (timeline/folders) before lighter ones.

Popup/Options pages handle settings and UI controls, while the service worker is used for background tasks (for example, fetching assets needed during export).

## Feature Implementation Details (Per Feature)

Below is a feature-by-feature breakdown to help you locate implementation modules and understand how each part works.

### 1) Timeline Navigation

- Entry: `startTimeline()` runs during content script initialization.
- Core: `TimelineManager` injects the timeline bar/track/tooltip, syncs with scroll, and manages pointer interactions.
- Data: starred and collapsed states are stored in localStorage; UI preferences and position are stored in chrome.storage.sync.
- UX: supports node levels, collapse/expand, draggable bar, and keyboard shortcuts.

### 2) Folder Organization

- Gemini: `FolderManager` injects sidebar UI, supports drag-and-drop, and tracks conversation references.
- AI Studio: `AIStudioFolderManager` renders a simplified folder UI for prompts/library pages.
- Storage: `FolderStorageAdapter` bridges localStorage and browser.storage.local for Safari compatibility and migration.
- Import/Export: JSON import/export with merge/overwrite strategies.

### 3) Prompt Library

- Entry: `startPromptManager()` enables the prompt panel and trigger button.
- Features: tag filtering, search, copy, import/export, and backup packaging.
- Storage: uses `StorageService` with migration from localStorage to chrome.storage.local.
- Custom websites: when a custom domain is configured, only Prompt Manager is initialized.

### 4) Chat Export

- Entry: `startExportButton()` injects export UI.
- Extraction: collects user/assistant nodes via selector sets, dedupes by text/offset, and reuses timeline star data for highlights.
- Formats: `ConversationExportService` uses a strategy pattern for JSON/Markdown/PDF; Markdown export bundles images into ZIP when needed.

### 5) Deep Research Export

- Entry: `startDeepResearchExport()` detects `deep-research-immersive-panel` and injects a menu button.
- Extraction: `extractThinkingPanels()` parses `thinking-panel` items and browse chips (link references).
- Output: formats content to Markdown and triggers download.

### 6) Mermaid Diagram Rendering

- Entry: `startMermaid()` enables Mermaid rendering in content scripts.
- Detection: validates code blocks with keyword checks and minimum length safeguards.
- UX: toggles between source/diagram and supports fullscreen zoom/pan.

### 7) Quote Reply

- Entry: `startQuoteReply()` injects a floating quote button and styling.
- Logic: listens for selection changes, validates selection inside main content, and inserts Markdown blockquotes into the input box.

### 8) Formula Copy

- Entry: `startFormulaCopy()` initializes a singleton service.
- Logic: event delegation captures clicks on KaTeX/MathJax nodes with `data-math` and copies in selected format.
- Storage: format preference is stored in chrome.storage.sync.

### 9) NanoBanana Watermark Removal

- Entry: `startWatermarkRemover()` on Gemini conversations.
- Engine: `WatermarkEngine` applies reverse alpha blending, detecting 48/96-size watermark configs.
- Integration: MutationObserver processes images and communicates state through custom events to a fetch interceptor.

### 10) Input Collapse

- Entry: `startInputCollapse()` injects CSS and detects the input container.
- Logic: collapses the input when empty, showing a placeholder; expands on interaction.

### 11) Chat Width Adjustment

- Entry: `startChatWidthAdjuster()` reads `geminiChatWidth` and injects CSS to override max-width constraints.
- Compatibility: legacy pixel values are normalized to percentages and migrated.

---

> Core implementations live under `src/pages/content` and `src/features`. Start from the content-script entry point when debugging or extending functionality.
