<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Animal Battle Arena</title>

<style>
body {
  margin: 0;
  background: #05050f;
  color: white;
  font-family: sans-serif;
  text-align: center;
}

/* Snow */
#snow {
  position: fixed;
  inset: 0;
  z-index: 0;
}

/* Layout */
.container {
  position: relative;
  z-index: 1;
  padding: 20px;
}

/* Players */
.players {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-top: 10px;
}

.player {
  padding: 10px 20px;
  border-radius: 10px;
  background: #111;
  transition: 0.3s;
}

.active {
  background: #00f5ff;
  color: black;
  font-weight: bold;
}

/* Board */
.board {
  display: grid;
  grid-template-columns: repeat(3, 100px);
  gap: 10px;
  justify-content: center;
  margin-top: 20px;
}

.cell {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  background: #111;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 40px;
  cursor: pointer;
  transition: 0.2s;
}

.cell:hover {
  background: #222;
}

/* Animation */
.jump {
  animation: jump 0.5s ease;
}

@keyframes jump {
  0% { transform: scale(1); }
  50% { transform: scale(1.4) translateY(-20px); }
  100% { transform: scale(1); }
}

/* Result */
#result {
  margin-top: 15px;
  font-size: 20px;
  font-weight: bold;
}

button {
  margin-top: 20px;
  padding: 10px 20px;
  cursor: pointer;
}
</style>
</head>

<body>

<canvas id="snow"></canvas>

<div class="container">
  <h1>Animal Arena</h1>

  <div class="players">
    <div id="p1Box" class="player"></div>
    <div id="p2Box" class="player"></div>
  </div>

  <h3 id="status"></h3>

  <div class="board" id="board"></div>

  <div id="result"></div>

  <button onclick="resetGame()">New Game</button>
</div>

<script>
/* THEMES */
const THEMES = [
  { p1: "🐭", p2: "🐱", n1: "Rat", n2: "Cat" },
  { p1: "🦁", p2: "🦌", n1: "Lion", n2: "Deer" },
  { p1: "🐶", p2: "🐼", n1: "Dog", n2: "Panda" },
];

let themeIndex = 0;
let currentTheme = THEMES[0];

/* PLAYERS */
let player1 = prompt("Enter Player 1 name:", "Player 1");
let player2 = prompt("Enter Player 2 name:", "Player 2");

let score = {p1:0, p2:0};

/* GAME */
let board = Array(9).fill(null);
let cur = "p1";
let over = false;

const WINS = [
  [0,1,2],[3,4,5],[6,7,8],
  [0,3,6],[1,4,7],[2,5,8],
  [0,4,8],[2,4,6]
];

const boardEl = document.getElementById("board");
const statusEl = document.getElementById("status");
const resultEl = document.getElementById("result");
const p1Box = document.getElementById("p1Box");
const p2Box = document.getElementById("p2Box");

/* INIT UI */
function updatePlayersUI(){
  p1Box.textContent = `${player1} (${currentTheme.n1}) : ${score.p1}`;
  p2Box.textContent = `${player2} (${currentTheme.n2}) : ${score.p2}`;

  p1Box.classList.toggle("active", cur==="p1");
  p2Box.classList.toggle("active", cur==="p2");
}

/* BUILD BOARD */
for(let i=0;i<9;i++){
  const cell = document.createElement("div");
  cell.className = "cell";
  cell.dataset.i = i;
  cell.onclick = handleClick;
  boardEl.appendChild(cell);
}

function cells(){
  return document.querySelectorAll(".cell");
}

/* CLICK */
function handleClick(e){
  const i = e.target.dataset.i;
  if(board[i] || over) return;

  board[i] = cur;
  render();

  const win = WINS.find(c => c.every(j => board[j] === cur));

  if(win){
    over = true;
    score[cur]++;
    updatePlayersUI();

    win.forEach(i=>{
      cells()[i].classList.add("jump");
    });

    resultEl.textContent =
      cur==="p1"
      ? `${currentTheme.n1} defeated ${currentTheme.n2} 💥`
      : `${currentTheme.n2} defeated ${currentTheme.n1} 💥`;

  }
  else if(board.every(Boolean)){
    over = true;
    resultEl.textContent = "Draw 🤝";
  }
  else{
    cur = cur==="p1" ? "p2" : "p1";
    updateStatus();
  }
}

/* RENDER */
function render(){
  cells().forEach((cell,i)=>{
    const v = board[i];
    if(v==="p1") cell.textContent = currentTheme.p1;
    else if(v==="p2") cell.textContent = currentTheme.p2;
    else cell.textContent = "";
  });
}

/* STATUS */
function updateStatus(){
  statusEl.textContent =
    cur==="p1"
    ? `${player1}'s Turn`
    : `${player2}'s Turn`;

  updatePlayersUI();
}

/* RESET */
function resetGame(){
  themeIndex = (themeIndex+1) % THEMES.length;
  currentTheme = THEMES[themeIndex];

  board = Array(9).fill(null);
  cur = "p1";
  over = false;
  resultEl.textContent = "";

  render();
  updateStatus();
}

/* INIT */
updateStatus();

/* SNOW */
const canvas = document.getElementById("snow");
const ctx = canvas.getContext("2d");

let snow = [];

function resize(){
  canvas.width = innerWidth;
  canvas.height = innerHeight;
}

function initSnow(){
  snow = Array.from({length:100}, ()=>({
    x: Math.random()*canvas.width,
    y: Math.random()*canvas.height,
    r: Math.random()*4+1,
    d: Math.random()*1
  }));
}

function drawSnow(){
  ctx.clearRect(0,0,canvas.width,canvas.height);
  ctx.fillStyle="white";
  ctx.beginPath();

  snow.forEach(s=>{
    ctx.moveTo(s.x,s.y);
    ctx.arc(s.x,s.y,s.r,0,Math.PI*2);
  });

  ctx.fill();
  moveSnow();
  requestAnimationFrame(drawSnow);
}

function moveSnow(){
  snow.forEach(s=>{
    s.y += s.d;
    if(s.y > canvas.height){
      s.y = 0;
      s.x = Math.random()*canvas.width;
    }
  });
}

resize();
initSnow();
drawSnow();

</script>

</body>
</html>
