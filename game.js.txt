/*
 * 2048 - plain JavaScript version.
 *
 * How it works (good for explaining in an interview):
 *  - The game state is a 4x4 array of numbers (0 = empty cell).
 *  - Every move is done by "sliding" each row to the left.
 *    Other directions reuse the same function by rotating/reversing the grid.
 *  - After each valid move a new tile (2 or 4) appears in a random empty cell.
 *  - The board is re-drawn from the state array after every move.
 */

const SIZE = 4;
let grid;            // 2D array: grid[row][col]
let score = 0;
let best = Number(localStorage.getItem("best2048")) || 0;
let won = false;     // so the "You win" message shows only once

const boardEl = document.getElementById("board");
const scoreEl = document.getElementById("score");
const bestEl = document.getElementById("best");
const messageEl = document.getElementById("message");
const messageText = document.getElementById("message-text");

/* ---------- Setup ---------- */
function newGame() {
  grid = Array.from({ length: SIZE }, () => Array(SIZE).fill(0));
  score = 0;
  won = false;
  messageEl.classList.add("hidden");
  addRandomTile();
  addRandomTile();
  draw();
}

/* Put a 2 (90%) or a 4 (10%) in a random empty cell. Returns the cell. */
function addRandomTile() {
  const empty = [];
  for (let r = 0; r < SIZE; r++)
    for (let c = 0; c < SIZE; c++)
      if (grid[r][c] === 0) empty.push([r, c]);
  if (empty.length === 0) return null;
  const [r, c] = empty[Math.floor(Math.random() * empty.length)];
  grid[r][c] = Math.random() < 0.9 ? 2 : 4;
  return [r, c];
}

/* ---------- Core logic ---------- */

/* Slide one row to the left and merge equal neighbours once. */
function slideRow(row) {
  const tiles = row.filter(n => n !== 0);   // 1) remove empty cells
  const result = [];
  for (let i = 0; i < tiles.length; i++) {
    if (tiles[i] === tiles[i + 1]) {         // 2) merge equal pair
      const merged = tiles[i] * 2;
      result.push(merged);
      score += merged;                       // merged value is added to score
      i++;                                   // skip the tile we just merged
    } else {
      result.push(tiles[i]);
    }
  }
  while (result.length < SIZE) result.push(0); // 3) pad with zeros
  return result;
}

/* Helpers used to reuse slideRow for every direction. */
const reverseRows = g => g.map(row => [...row].reverse());
const transpose = g => g[0].map((_, c) => g.map(row => row[c]));

/* Returns true if the move changed the board. */
function move(direction) {
  const before = JSON.stringify(grid);
  if (direction === "left") {
    grid = grid.map(slideRow);
  } else if (direction === "right") {
    grid = reverseRows(reverseRows(grid).map(slideRow));
  } else if (direction === "up") {
    grid = transpose(transpose(grid).map(slideRow));
  } else if (direction === "down") {
    grid = transpose(reverseRows(reverseRows(transpose(grid)).map(slideRow)));
  }
  return JSON.stringify(grid) !== before;
}

/* The game is over when the board is full and no neighbours are equal. */
function hasMoves() {
  for (let r = 0; r < SIZE; r++) {
    for (let c = 0; c < SIZE; c++) {
      if (grid[r][c] === 0) return true;
      if (c + 1 < SIZE && grid[r][c] === grid[r][c + 1]) return true;
      if (r + 1 < SIZE && grid[r][c] === grid[r + 1][c]) return true;
    }
  }
  return false;
}

function hasTile(value) {
  return grid.some(row => row.includes(value));
}

/* One full turn: move, spawn a tile, redraw, check win/lose. */
function play(direction) {
  if (!move(direction)) return;       // nothing moved -> ignore the key
  const spawned = addRandomTile();
  if (score > best) {
    best = score;
    localStorage.setItem("best2048", best); // saved in the browser
  }
  draw(spawned);

  if (!won && hasTile(2048)) {
    won = true;
    showMessage("You win! 🎉");
  } else if (!hasMoves()) {
    showMessage("Game over");
  }
}

/* ---------- Drawing ---------- */
function draw(spawned) {
  boardEl.innerHTML = "";
  for (let r = 0; r < SIZE; r++) {
    for (let c = 0; c < SIZE; c++) {
      const value = grid[r][c];
      const tile = document.createElement("div");
      tile.className = "tile";
      if (value) {
        tile.textContent = value;
        tile.classList.add(value > 2048 ? "vbig" : "v" + value);
      }
      if (spawned && spawned[0] === r && spawned[1] === c) tile.classList.add("pop");
      boardEl.appendChild(tile);
    }
  }
  scoreEl.textContent = score;
  bestEl.textContent = best;
}

function showMessage(text) {
  messageText.textContent = text;
  messageEl.classList.remove("hidden");
}

/* ---------- Input: keyboard ---------- */
const keyMap = {
  ArrowLeft: "left", ArrowRight: "right", ArrowUp: "up", ArrowDown: "down",
  a: "left", d: "right", w: "up", s: "down"
};
document.addEventListener("keydown", e => {
  const dir = keyMap[e.key];
  if (dir) {
    e.preventDefault();               // stop the page from scrolling
    play(dir);
  }
});

/* ---------- Input: touch swipe ---------- */
let startX, startY;
boardEl.addEventListener("touchstart", e => {
  startX = e.touches[0].clientX;
  startY = e.touches[0].clientY;
});
boardEl.addEventListener("touchend", e => {
  const dx = e.changedTouches[0].clientX - startX;
  const dy = e.changedTouches[0].clientY - startY;
  if (Math.max(Math.abs(dx), Math.abs(dy)) < 30) return; // too short = tap
  if (Math.abs(dx) > Math.abs(dy)) play(dx > 0 ? "right" : "left");
  else play(dy > 0 ? "down" : "up");
});

/* ---------- Buttons ---------- */
document.getElementById("new-game").addEventListener("click", newGame);
document.getElementById("retry").addEventListener("click", newGame);

newGame();
