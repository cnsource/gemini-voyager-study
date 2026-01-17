# Technical Overview

This page summarizes the Gemini Voyager tech stack, build workflow, and extension architecture to help developers quickly understand how the project is assembled.

## Product Scope

Gemini Voyager is a Manifest V3 browser extension that augments Gemini/AI Studio with timeline navigation, folders, prompt library, and export tooling via injected content scripts and extension UI pages.

## Tech Stack

- **Language & UI**: TypeScript + React 19 for modern, component-driven UI.
- **Build System**: Vite 7 with CRXJS for extension packaging; per-browser config targets Chrome/Firefox/Safari.
- **Styling**: Tailwind CSS and class utilities (tailwind-merge).
- **Docs**: VitePress powers the documentation site.
- **Testing**: Vitest is used for unit tests and coverage.

## Development & Build Scripts

Use `bun` or `npm` to run the primary workflows:

- `dev:*`: nodemon-wrapped watch builds for Chrome/Firefox/Safari.
- `build:*`: browser-specific release builds.
- `docs:*`: VitePress dev/build/preview commands.
- `test:*`: Vitest runs, including UI and coverage.

## Extension Architecture

Key Manifest V3 components include:

- **Background**: a `service_worker` entry for background logic.
- **Content Scripts**: injected scripts and styles for Gemini and AI Studio domains.
- **Extension Pages**: Popup and Options pages for quick actions and settings.
- **Permissions**: core extension permissions plus host permissions for Gemini/AI Studio, with optional wildcard permissions for custom websites.

## Documentation Output

Technical documentation lives in `docs/` and is published via VitePress with bilingual (中文/English) navigation. Use `docs:dev` locally to preview changes while developing or extending the documentation.
