# 2048 - Browser Game

A clone of the classic 2048 puzzle game, built with plain HTML, CSS and JavaScript. No frameworks, no libraries, no build step.

**Play it live:** https://YOUR-USERNAME.github.io/YOUR-REPO-NAME/

## Features

- 4x4 board with the original merge rules
- Keyboard controls (arrow keys or WASD) and swipe controls on mobile
- Score and best score (the best score is saved in the browser with `localStorage`)
- Win and game over screens
- Responsive layout that works on phones and desktops

## How to run locally

1. Download or clone this repository.
2. Open `index.html` in any modern browser.

## How it works

- The game state is a 4x4 array of numbers (`0` means an empty cell).
- A single function, `slideRow`, slides and merges one row to the left. The other three directions reuse it by reversing or transposing the grid.
- After every valid move, a new tile (2 or 4) appears in a random empty cell and the board is redrawn from the state array.
- The game ends when the board is full and no neighbouring tiles are equal.

## Project structure

```
index.html   page structure
style.css    layout, colors and animations
game.js      game logic, rendering and input handling
```

## What I learned

- Structuring game logic separately from rendering
- Handling keyboard and touch events
- Using CSS Grid and `localStorage`
- Publishing a static site with GitHub Pages

## Possible improvements

- Slide animations between cells
- Undo button
- Different board sizes
