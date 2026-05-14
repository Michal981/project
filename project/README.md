<!DOCTYPE html>
<html lang="cs">
<head>
<meta charset="UTF-8">
<title>Escape the Virus</title>

<style>
body {
    margin: 0;
    background: #0b1b2b;
    color: white;
    font-family: Arial;
    text-align: center;
}

canvas {
    background: #111;
    display: none;
    margin: 20px auto;
    border: 2px solid #1e90ff;
}

.menu {
    margin-top: 100px;
}

button {
    padding: 15px 30px;
    margin: 10px;
    font-size: 18px;
    background: #1e90ff;
    border: none;
    color: white;
    cursor: pointer;
}

button:hover {
    background: #63b3ff;
}
</style>
</head>

<body>

<h1>🦠 ESCAPE THE VIRUS</h1>

<div class="wrapper">

    <!-- MENU -->
    <div id="menu" class="menu">
        <p style="font-size:24px;">
            🎮 Mezerník = skok | P = pauza
        </p>

        <button onclick="startGame()">Play</button>
        <button onclick="openSkins()">Změna skinu</button>
    </div>

    <!-- SKINS -->
    <div id="skins" class="menu" style="display:none;">
        <h2>Vyber skin</h2>

        <button onclick="setSkin('#00ffcc')">Tyrkysový</button>
        <button onclick="setSkin('#ffcc00')">Žlutý</button>
        <button onclick="setSkin('#ff66ff')">Růžový</button>

        <br><br>

        <button onclick="backToMenu()">Zpět</button>
    </div>

    <!-- GAME -->
    <canvas id="game" width="800" height="300"></canvas>

</div>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

// Skin hráče
let skin = "#00ffcc";

// Hráč
let player = {
    x: 50,
    y: 200,
    w: 30,
    h: 30,
    dy: 0,
    jumping: false
};

// Herní proměnné
let gravity = 0.6;
let obstacles = [];
let frame = 0;
let score = 0;

// Rychlost hry
let speed = 5;

// Stav hry
let paused = false;
let running = false;
let gameOver = false;

// Start hry
function startGame() {

    document.getElementById("menu").style.display = "none";
    document.getElementById("skins").style.display = "none";

    canvas.style.display = "block";

    running = true;
}

// Otevření skinů
function openSkins() {
    document.getElementById("menu").style.display = "none";
    document.getElementById("skins").style.display = "block";
}

// Zpět do menu
function backToMenu() {
    document.getElementById("skins").style.display = "none";
    document.getElementById("menu").style.display = "block";
}

// Nastavení skinu
function setSkin(color) {
    skin = color;
}

// Ovládání
document.addEventListener("keydown", e => {

    if (!running) return;

    // Skok
    if (e.code === "Space" && !player.jumping) {
        player.dy = -12;
        player.jumping = true;
    }

    // Pauza
    if (e.key.toLowerCase() === "p") {
        paused = !paused;
    }
});

// Spawn překážky
function spawnObstacle() {

    obstacles.push({
        x: canvas.width,
        y: 220,
        w: 20,
        h: 30
    });
}

// Update hry
function update() {

    if (!running || paused || gameOver) return;

    frame++;
    score++;

    // Spawn překážek
    if (frame % 60 === 0) {
        spawnObstacle();
    }

    // POSTUPNÉ ZRYCHLOVÁNÍ
    if (frame % 300 === 0) {
        speed += 0.5;
    }

    // Gravitace
    player.y += player.dy;
    player.dy += gravity;

    // Zem
    if (player.y > 200) {
        player.y = 200;
        player.jumping = false;
    }

    // Pohyb překážek
    obstacles.forEach(o => {

        o.x -= speed;

        // Kolize
        if (
            player.x < o.x + o.w &&
            player.x + player.w > o.x &&
            player.y < o.y + o.h &&
            player.y + player.h > o.y
        ) {
            gameOver = true;
            running = false;
        }
    });

    // Mazání překážek
    obstacles = obstacles.filter(o => o.x > -20);
}

// Vykreslení
function draw() {

    if (!running && !gameOver) return;

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Zem
    ctx.fillStyle = "#444";
    ctx.fillRect(0, 250, canvas.width, 5);

    // Hráč
    ctx.fillStyle = skin;
    ctx.fillRect(player.x, player.y, player.w, player.h);

    // Překážky
    ctx.fillStyle = "red";

    obstacles.forEach(o => {
        ctx.fillRect(o.x, o.y, o.w, o.h);
    });

    // Score
    ctx.fillStyle = "white";
    ctx.font = "16px Arial";
    ctx.fillText("Skóre: " + score, 10, 20);

    // Rychlost
    ctx.fillText("Rychlost: " + speed.toFixed(1), 10, 45);

    // GAME OVER SCREEN
    if (gameOver) {

        ctx.fillStyle = "rgba(0,0,0,0.8)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        ctx.fillStyle = "red";
        ctx.font = "60px Arial";
        ctx.fillText("GAME OVER", 170, 120);

        ctx.fillStyle = "white";
        ctx.font = "40px Arial";
        ctx.fillText("Final Score: " + score, 220, 190);

        ctx.font = "30px Arial";
        ctx.fillText("Final Speed: " + speed.toFixed(1), 230, 240);

        ctx.font = "22px Arial";
        ctx.fillText("Obnov stránku pro restart", 250, 285);
    }

    // Pauza
    if (paused) {
        ctx.fillStyle = "white";
        ctx.font = "30px Arial";
        ctx.fillText("PAUZA", 340, 150);
    }
}

// Herní loop
function loop() {
    update();
    draw();
    requestAnimationFrame(loop);
}

loop();
</script>

</body>
</html>
