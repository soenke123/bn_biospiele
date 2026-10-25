# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

German-language biology study app ("Evolutionsfaktoren – Interaktives Glossar & Memory"). The whole app is a single static file, `index.html` (~700 lines: markup, a small `<style>` block, and one inline `<script>`). `README.md` is just a title (UTF-16 encoded).

There is no build, package manager, linter, or test suite. To run it, open `index.html` in a browser. Tailwind (`cdn.tailwindcss.com`) and Font Awesome are loaded from CDNs, so a network connection is needed for styling and icons.

## Architecture

All logic lives in the inline `<script>` at the bottom of `index.html`, using global state and `onclick="..."` handlers in the markup (no modules or framework).

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
