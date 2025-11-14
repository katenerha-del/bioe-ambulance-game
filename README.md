# bioe-ambulance-game
bioe_ambulance_game/
│
├─ index.html
├─ style.css
└─ script.js
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Bioethanol Ambulance Rescue</title>
<link rel="stylesheet" href="style.css">
</head>
<body>
<div id="hud">
    Fuel: <span id="fuel">100</span> | Patients Rescued: <span id="rescued">0</span>
</div>
<canvas id="gameCanvas"></canvas>
<script src="script.js"></script>
</body>
</html>
body {
    margin: 0;
    overflow: hidden;
    background: linear-gradient(#87ceeb, #fff);
    font-family: Arial, sans-serif;
}

canvas {
    display: block;
    background: transparent;
}

#hud {
    position: absolute;
    top: 10px;
    left: 10px;
    color: white;
    font-size: 18px;
    font-weight: bold;
    z-index: 10;
    text-shadow: 1px 1px 2px black;
}
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

// Ambulance
const ambulance = {
    x: 100,
    y: canvas.height - 120,
    width: 100,
    height: 50,
    wheelExtend: 0,
    speed: 0,
    fuel: 100,
    rescued: 0
};

// Controls
const keys = {};
document.addEventListener('keydown', e => keys[e.key] = true);
document.addEventListener('keyup', e => keys[e.key] = false);

// Patients
const patients = [];
function spawnPatient() {
    const patient = {
        x: Math.random() * (canvas.width - 50) + 50,
        y: canvas.height - 80,
        size: 20,
        rescued: false
    };
    patients.push(patient);
}
setInterval(spawnPatient, 3000);

// Drones
const drones = [];
function spawnDrone() {
    const drone = {
        x: Math.random() * canvas.width,
        y: 50,
        speedX: (Math.random() - 0.5) * 2,
        speedY: 1
    };
    drones.push(drone);
}
setInterval(spawnDrone, 2000);

// Physics
const gravity = 0.5;

// Game loop
function update() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Fuel decreases when moving
    if ((keys['ArrowRight'] || keys['ArrowLeft']) && ambulance.fuel > 0) {
        ambulance.fuel = Math.max(ambulance.fuel - 0.05, 0);
    }

    // Movement
    if (keys['ArrowRight'] && ambulance.fuel > 0) ambulance.x += 5 + ambulance.wheelExtend/5;
    if (keys['ArrowLeft'] && ambulance.fuel > 0) ambulance.x -= 5 + ambulance.wheelExtend/5;
    if (keys['ArrowUp']) ambulance.wheelExtend = Math.min(ambulance.wheelExtend + 1, 20);
    if (keys['ArrowDown']) ambulance.wheelExtend = Math.max(ambulance.wheelExtend - 1, 0);

    // Draw ambulance
    ctx.fillStyle = 'red';
    ctx.fillRect(ambulance.x, ambulance.y, ambulance.width, ambulance.height);
    ctx.fillStyle = 'black';
    ctx.beginPath();
    ctx.arc(ambulance.x + 20, ambulance.y + ambulance.height, 20 + ambulance.wheelExtend/2, 0, Math.PI*2);
    ctx.arc(ambulance.x + 80, ambulance.y + ambulance.height, 20 + ambulance.wheelExtend/2, 0, Math.PI*2);
    ctx.fill();

    // Draw patients
    ctx.fillStyle = 'yellow';
    patients.forEach(p => {
        if (!p.rescued) ctx.fillRect(p.x, p.y, p.size, p.size);
    });

    // Rescue patients
    patients.forEach(p => {
        if (!p.rescued &&
            ambulance.x < p.x + p.size &&
            ambulance.x + ambulance.width > p.x &&
            ambulance.y < p.y + p.size &&
            ambulance.y + ambulance.height > p.y) {
            p.rescued = true;
            ambulance.rescued += 1;
        }
    });

    // Draw drones
    ctx.fillStyle = 'white';
    drones.forEach(d => {
        d.x += d.speedX;
        d.y += d.speedY;
        if (d.x < 0 || d.x > canvas.width) d.speedX *= -1;
        if (d.y > canvas.height/2) d.speedY *= -1;
        ctx.beginPath();
        ctx.arc(d.x, d.y, 10, 0, Math.PI*2);
        ctx.fill();
    });

    // HUD
    document.getElementById('fuel').textContent = ambulance.fuel.toFixed(0);
    document.getElementById('rescued').textContent = ambulance.rescued;

    requestAnimationFrame(update);
}

update();
