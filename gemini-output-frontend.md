```html
<!DOCTYPE html>
<html>
<head>
<title>Tetris</title>
<style>
body {
margin: 0;
background-color: #222;
color: #eee;
}
canvas {
background-color: #000;
border: 4px solid #eee;
}
</style>
</head>
<body>
<canvas id="tetris" width="300" height="600"></canvas>
<script>
const canvas = document.getElementById('tetris');
const ctx = canvas.getContext('2d');

const ROWS = 20;
const COLS = 10;
const BLOCK_SIZE = 30;

const COLORS = [
'cyan', 'blue', 'orange', 'yellow', 'green', 'purple', 'red'
];

let board = [];
let currentPiece = {};
let nextPiece = {};
let score = 0;
let gameOver = false;

// Create empty board
function createBoard() {
for (let r = 0; r < ROWS; r++) {
board[r] = [];
for (let c = 0; c < COLS; c++) {
board[r][c] = 0;
}
}
}

// Draw a block
function drawBlock(x, y, color) {
ctx.fillStyle = color;
ctx.fillRect(x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
ctx.strokeStyle = 'black';
ctx.strokeRect(x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);
}

// Draw the board
function drawBoard() {
for (let r = 0; r < ROWS; r++) {
for (let c = 0; c < COLS; c++) {
if (board[r][c] > 0) {
drawBlock(c, r, COLORS[board[r][c] - 1]);
}
}
}
}

// Draw the current piece
function drawPiece() {
currentPiece.shape.forEach((row, r) => {
row.forEach((value, c) => {
if (value > 0) {
drawBlock(currentPiece.x + c, currentPiece.y + r, COLORS[currentPiece.color - 1]);
}
});
});
}

// Create a new piece
function newPiece() {
const shapes = [
[ // I-shape
[0, 0, 0, 0],
[1, 1, 1, 1],
[0, 0, 0, 0],
[0, 0, 0, 0]
],
[ // J-shape
[1, 0, 0],
[1, 1, 1],
[0, 0, 0]
],
[ // L-shape
[0, 0, 1],
[1, 1, 1],
[0, 0, 0]
],
[ // O-shape
[1, 1],
[1, 1]
],
[ // S-shape
[0, 1, 1],
[1, 1, 0],
[0, 0, 0]
],
[ // T-shape
[0, 1, 0],
[1, 1, 1],
[0, 0, 0]
],
[ // Z-shape
[1, 1, 0],
[0, 1, 1],
[0, 0, 0]
]
];

const randomShape = shapes[Math.floor(Math.random() * shapes.length)];
const randomColor = Math.floor(Math.random() * COLORS.length) + 1;

currentPiece = {
x: 3,
y: 0,
shape: randomShape,
color: randomColor
};

nextPiece = {
x: 3,
y: 0,
shape: shapes[Math.floor(Math.random() * shapes.length)],
color: Math.floor(Math.random() * COLORS.length) + 1
};
}

// Move the piece down
function movePieceDown() {
if (!collision(0, 1)) {
currentPiece.y++;
} else {
freezePiece();
}
}

// Move the piece left or right
function movePieceHorizontal(direction) {
if (!collision(direction, 0)) {
currentPiece.x += direction;
}
}

// Rotate the piece
function rotatePiece() {
let newShape = [];
for (let c = 0; c < currentPiece.shape[0].length; c++) {
newShape[c] = [];
for (let r = currentPiece.shape.length - 1; r >= 0; r--) {
newShape[c][r] = currentPiece.shape[r][c];
}
}

if (!collision(0, 0, newShape)) {
currentPiece.shape = newShape;
}
}

// Check for collisions
function collision(xOffset, yOffset, newShape) {
if (newShape) {
return newShape.some((row, r) => {
return row.some((value, c) => {
if (value > 0) {
let newX = currentPiece.x + c + xOffset;
let newY = currentPiece.y + r + yOffset;
return newX < 0 || newX >= COLS || newY >= ROWS || (newY >= 0 && board[newY][newX] > 0);
}
});
});
} else {
return currentPiece.shape.some((row, r) => {
return row.some((value, c) => {
if (value > 0) {
let newX = currentPiece.x + c + xOffset;
let newY = currentPiece.y + r + yOffset;
return newX < 0 || newX >= COLS || newY >= ROWS || (newY >= 0 && board[newY][newX] > 0);
}
});
});
}
}

// Freeze the piece on the board
function freezePiece() {
currentPiece.shape.forEach((row, r) => {
row.forEach((value, c) => {
if (value > 0) {
board[currentPiece.y + r][currentPiece.x + c] = currentPiece.color;
}
});
});

checkLines();
currentPiece = nextPiece;
nextPiece = {
x: 3,
y: 0,
shape: [
[0, 0, 0, 0],
[1, 1, 1, 1],
[0, 0, 0, 0],
[0, 0, 0, 0]
],
color: Math.floor(Math.random() * COLORS.length) + 1
};

if (collision(0, 0)) {
gameOver = true;
}
}

// Check for complete lines and remove them
function checkLines() {
let linesCleared = 0;
for (let r = 0; r < ROWS; r++) {
if (board[r].every(value => value > 0)) {
board.splice(r, 1);
board.unshift(new Array(COLS).fill(0));
linesCleared++;
}
}

if (linesCleared > 0) {
score += linesCleared * 100;
}
}

// Game loop
function update() {
if (!gameOver) {
movePieceDown();
drawBoard();
drawPiece();
} else {
ctx.font = '30px Arial';
ctx.fillText('Game Over!', 30, 300);
ctx.fillText('Score: ' + score, 30, 350);
}
requestAnimationFrame(update);
}

// Event listeners
document.addEventListener('keydown', (event) => {
switch (event.key) {
case 'ArrowLeft':
movePieceHorizontal(-1);
break;
case 'ArrowRight':
movePieceHorizontal(1);
break;
case 'ArrowDown':
movePieceDown();
break;
case 'ArrowUp':
rotatePiece();
break;
}
});

// Start the game
createBoard();
newPiece();
update();
</script>
</body>
</html>
```

**Explanation:**

1. **HTML Structure:**
- A simple HTML document with a canvas element (`id="tetris"`) for drawing the game.
- Basic styling for the canvas and body.

2. **JavaScript:**
- **Constants:** Define game parameters like rows, columns, block size, and colors.
- **Variables:** Initialize game state variables: `board`, `currentPiece`, `nextPiece`, `score`, `gameOver`.
- **`createBoard()`:** Creates an empty 2D array to represent the game board.
- **`drawBlock()`:** Draws a single block on the canvas.
- **`drawBoard()`:** Iterates through the board array and draws filled blocks for non-zero values.
- **`drawPiece()`:** Draws the current piece based on its shape and color.
- **`newPiece()`:** Generates a new piece with a random shape and color. It also initializes the `nextPiece` for display.
- **`movePieceDown()`:** Moves the current piece down one row unless it collides.
- **`movePieceHorizontal()`:** Moves the piece left or right based on the direction.
- **`rotatePiece()`:** Rotates the piece by 90 degrees counter-clockwise, ensuring no collision after rotation.
- **`collision()`:** Checks for collisions with the board boundaries or existing blocks.
- **`freezePiece()`:** Places the current piece on the board, checks for completed lines, and generates a new piece.
- **`checkLines()`:** Checks for full rows, removes them, and updates the score.
- **`update()`:** The main game loop that handles updating the game state, drawing, and event handling.
- **Event Listeners:** Responds to arrow key presses for movement and rotation.
- **Game Initialization:** Calls `createBoard()`, `newPiece()`, and `update()` to start the game.

**To run this code:**

1. Save the code as an HTML file (e.g., `tetris.html`).
2. Open the file in a web browser.

You can now play a basic version of Tetris using your arrow keys!