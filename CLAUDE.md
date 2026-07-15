# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A single-file, dependency-free Tic-Tac-Toe game: [index.html](index.html). No build step, no package.json, no server-side code — everything (HTML, CSS, JS) lives inline in that one file so it can be opened directly in a browser or double-clicked from Google Drive.

The repo also contains a second, unrelated single-file app: [dictionary.html](dictionary.html), a mobile-first English dictionary web app (with [dictionary-icon.png](dictionary-icon.png) as its iOS home-screen icon). It follows the same convention — all HTML/CSS/JS inline, no build step. It fetches definitions from the public `api.dictionaryapi.dev` API, adds Spanish translations of the word and each definition (Google's unofficial `translate.googleapis.com` gtx endpoint, with `api.mymemory.translated.net` as fallback), plays pronunciation audio (recorded audio from the API, `speechSynthesis` en-US fallback), and persists user-created word categories in `localStorage` under the key `mydictionary.v1`. Translations are stored as `_esWord`/`_es` fields mutated onto the cached API entry objects, so saved words keep their Spanish offline.

## Running / testing

There is no build or test tooling. To preview changes, serve the directory and open it in a browser, e.g.:

```
python3 -m http.server 8756
```

then visit `http://localhost:8756/index.html`. There is no automated test suite — verify changes manually by playing through the UI (win, draw, mode switch, difficulty switch, play again, reset score).

## Architecture

Everything is in `index.html`, organized top to bottom as: `<style>` block, static HTML skeleton, then a single `<script>` block. Key pieces of the script:

- `state` — single mutable object holding board array, current player, game mode (`pvp`/`ai`), AI difficulty, and scores. All rendering derives from this object; there is no other source of truth.
- `render(winInfo)` — the only function that touches the DOM based on `state`. Any state change should be followed by a call to `render` (directly or via `finishTurn`).
- `finishTurn()` — central turn-resolution function: checks for a win/draw, updates scores, flips `state.current`, calls `render`, and if it's now the AI's turn, schedules `runAiMove` after a short delay (so the AI "thinking" pause is real, not simulated in the status text alone).
- AI logic: `getAiMove` picks between a random move and `bestAiMove` based on `state.difficulty` (Easy: 80% random, Medium: 40% random, Hard: 0% random). `bestAiMove`/`minimax` is a standard unbeatable minimax search — Hard mode is intentionally undefeatable, so a full optimal-play game should always end in a draw.
- Mode/difficulty switching (`setMode`, `setDifficulty`) both call `resetScore()`, so switching modes or difficulty mid-session clears the board and scores rather than trying to reconcile in-progress state.

When making changes, keep the single-file structure — do not split this into separate HTML/CSS/JS files or introduce a build step unless explicitly asked.
