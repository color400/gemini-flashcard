# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a single-file, client-side English vocabulary flashcard app (`index.html`). No build step, no package manager, no server — open the file directly in a browser to run it.

## Running the App

```bash
open index.html
# or serve it locally:
python3 -m http.server 8080
```

## Architecture

Everything lives in `index.html` — HTML structure, CSS (inline `<style>`), and JavaScript (`<script>`). There are no external files beyond CDN dependencies.

**CDN dependencies:**
- Tailwind CSS (via CDN `<script>`) — utility classes for all layout and styling
- Font Awesome 6.0 (via CDN `<link>`) — icons throughout the UI

**JavaScript architecture (all global, no modules):**

- `vocabularyDatabase` — in-memory array that is the single source of truth for all words. Words are never persisted; refreshing the page resets to the 5 hardcoded defaults.
- Game state lives in module-level `let` variables: `reviewPile`, `currentCard`, `isFlipped`, `isReverseMode`, `currentStreak`, etc.
- `reviewPile` is a working copy of `vocabularyDatabase` (shuffled). Cards the user marks "not known" are pushed back onto the end; cards marked "known" are removed. Session ends when `reviewPile` is empty.
- `processWithGemini()` — calls the Gemini API (`gemini-2.5-flash-preview-09-2025`) with structured JSON output schema. The API key is injected at runtime (left as empty string `""`); this app is designed to run in an environment that auto-provides the key. Retry logic: 5 attempts with exponential backoff (1s, 2s, 4s, 8s, 16s).
- Supports two input modes: image upload (base64 encoded inline) and plain text. Both are processed through the same `processWithGemini()` path.

**UI states** managed by toggling `hidden`/`flex` Tailwind classes on four mutually exclusive panels:
1. `#game-area` — active flashcard session
2. `#result-screen` — session completion summary
3. `#empty-state` — shown when `vocabularyDatabase` is empty
4. `#loading-screen` — AI processing overlay

**Keyboard shortcuts:** Arrow Up/Down flips card; Arrow Left = "don't know"; Arrow Right = "know"; Enter on result screen restarts.

**Two quiz modes** toggled by `isReverseMode`: English→Chinese (default) or Chinese→English. The same card data renders differently based on this flag in `renderCardContent()`.
