# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

German-language collection of biology study games, built like the sibling repo `bi_minis`: the root `index.html` is the tile overview (with QR-code buttons) and lists every game in a hand-maintained `projects` array (`{folder, name, created, hidden?}`, newest first; `hidden: true` hides the tile). Each game is a single static file in its own folder:

- `evolutionsfaktoren/index.html` – "Evolutionsfaktoren – Interaktives Glossar & Memory" (described in detail below).
- `wer-gehoert-zu-wem/index.html` – "Wer gehört zu wem?": a one-player pairing game. Data is the `matchPairs` array (`{id, type, title, description, item1, item2}`, each item `{name, icon}`). Icons are inline SVGs built with the `icon(bg, body)` helper; all gradients and clip paths live once in the shared hidden `<svg><defs>` at the top of `<body>` (so IDs stay unique while icons are rendered in the grid and in the modals). Reuse those defs for new icons instead of adding `<defs>` inside an icon.

To add a game: create `<folder>/index.html`, add a home link to `../index.html`, and add an entry to `projects` in the root `index.html`.

There is no build, package manager, linter, or test suite. Open the files in a browser. Tailwind and Font Awesome (fa-solid only) are vendored in `vendor/` and referenced from the games as `../vendor/...`, so everything works offline. `README.md` is just a title (UTF-16 encoded).

## Architecture (`evolutionsfaktoren/index.html`)

All logic lives in the inline `<script>` at the bottom of `evolutionsfaktoren/index.html`, using global state and `onclick="..."` handlers in the markup (no modules or framework).

- **`rawDataset`**: the single source of truth, an array of `{ id, front, back }` glossary pairs (term and explanation). Both modes read from it. To add or change content, edit this array. Keep `id` values unique, because the memory game matches cards by `pairId`.
- **`switchMode(mode)`**: toggles between the two modes by swapping Tailwind class strings on the tab buttons and the `hidden` class on `#mode-cards` / `#mode-memory`. Active and inactive tab styles are hardcoded strings here, so a tab restyle must be made in both the markup and this function. The memory game starts lazily on the first switch to it.
- **Flashcard mode**: it keeps a copy of the data in `flashcardDeck`, indexed by `currentCardIndex`. The card flips with a CSS 3D transform (the `.rotate-y-180` class on `#card-inner`; the helper classes `perspective-1000`, `backface-hidden` and `transform-style-3d` are in the `<style>` block). Navigating while flipped un-flips first and re-renders after a 200ms timeout.
- **Memory mode**: this is a two-player, hot-seat variant that differs from standard memory.
  - There are two separate grids: term cards (left, `termCards`) and explanation cards (right, `explCards`). Each is shuffled independently. Both grids get identical CSS grid geometry (`gridColumns()`), so the cards are the same size on both sides.
  - A turn selects exactly one card from each side (`handleMemoryCardClick` blocks picking the same side twice). A clicked card flips face-up on the board at once, so the player can look for its partner.
  - `fitText()` shrinks the font by binary search until the text fits its box with no scrolling or overflow. It is used for the small board faces and for the enlarged clones.
  - `HOLD_MS` (500ms) after the second flip, `startReview()` clones both cards as face-up `position:fixed` elements. They animate from their board rects to the screen centre over the `#review-layer` overlay, and size, font size and padding transition together. The Ja/Nein buttons live in that overlay.
  - `handleUserDecision` then runs `flyToPlayer` (correct "Ja": the clones fly to the player's score chip, the pair is marked matched, the score goes up, and the turn stays) or `flyBack` (any other result: the clones return face-up to the board, rest `HOLD_MS`, flip back to the cover, then the player switches).
  - `isProcessingTurn` locks input during the whole review and the fly animations.
  - Board elements have the id `mem-card-${side}-${index}`. `renderMemoryGrids()` rebuilds both grids from state, so matched cards render as empty dashed placeholders to keep the layout stable.
  - The game ends via `showWinnerModal` when `matchedPairsCount === totalPairs`. The number of pairs comes from the `#pair-select` dropdown.

UI text and comments are German and English mixed. Keep user-facing strings in German.
