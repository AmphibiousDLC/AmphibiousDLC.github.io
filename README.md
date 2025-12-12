<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Salamander Game: Pond Avoidance</title>
<style>
  body {
    background-color: #f0f0f0;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    height: 100vh;
    margin: 0;
    transition: background-color 0.1s ease;
  }
  #gameContainer {
    display: none; 
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100%;
  }
  canvas {
    border: solid 2px #333;
    background-color: white; 
    margin-bottom: 10px;
    width: 90vw; 
    height: 60vh; 
    max-width: 800px;
    max-height: 500px;
  }
  .controls {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-areas: ". up ." "left down right" ". spawn .";
    gap: 10px;
    justify-items: center;
    width: 200px; 
  }
  button {
    width: 60px;
    height: 60px;
    font-size: 20px;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    color: white;
  }
  #up, #down, #left, #right { background-color: #4CAF50; }
  #left, #right { font-size: 24px; } 

  #spawn { 
    grid-area: spawn; 
    width: 100%; 
    height: 40px;
    font-size: 14px;
    background-color: #00BFA5; 
    margin-top: 10px;
  }
  #discoButton {
    background-color: purple;
    width: 200px;
    height: 40px;
    margin-top: 10px;
  }

  /* --- Start Screen Styling --- */
  #startScreen {
    text-align: center;
    padding: 20px;
  }
  #gameTitle {
    font-size: 48px;
    font-weight: bold;
    color: blue;
    text-shadow: -2px -2px 0 #00ff00, 2px -2px 0 #00ff00, -2px 2px 0 #00ff00, 2px 2px 0 #00ff00;
    margin-bottom: 30px;
  }
  #startButton {
    /* Made the button size adjust to fit the text and animation effects */
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 15px 40px;
    font-size: 24px;
    cursor: pointer;
    border: 3px solid black; 
    border-radius: 10px;
    color: white;
    animation: rainbow-flash-bg 1s infinite alternate; 
  }
  @keyframes rainbow-flash-bg {
    0% { background-color: red; }
    20% { background-color: orange; }
    40% { background-color: yellow; }
    60% { background-color: green; }
    80% { background-color: blue; }
    100% { background-color: purple; }
  }
</style>
</head>
<body>

<div id="startScreen">
    <div id="gameTitle">Salamander Friends</div>
    <button id="startButton" onclick="startGame()">Start Game</button>
</div>

<div id="gameContainer">
    <canvas id="gameCanvas" width="800" height="500"></canvas>

    <div class="controls">
      <button id="up" onmousedown="move('up', true)" onmouseup="move('up', false)" ontouchstart="move('up', true)" ontouchend="move('up', false)">↑</button>
      <button id="left" onmousedown="move('left', true)" onmouseup="move('left', false)" ontouchstart="move('left', true)" ontouchend="move('left', false)">←</button>
      <button id="down" onmousedown="move('down', true)" onmouseup="move('down', false)" ontouchstart="move('down', true)" ontouchend="move('down', false)">↓</button>
      <button id="right" onmousedown="move('right', true)" onmouseup="move('right', false)" ontouchstart="move('right', true)" ontouchend="move('right', false)">→</button>
      <button id="spawn" onclick="spawnNewSalamander()">Spawn Clone</button>
    </div>
    <button id="discoButton" onclick="toggleDiscoMode()">DISCO MODE</button>
</div>


<script>
  const canvas = document.getElementById("gameCanvas");
  const ctx = canvas.getContext("2d");
  canvas.width = 800;
  canvas.height = 500;
  const characterWidth = 100; const characterHeight = 25; const speed = 3; const cloneSpeed = 1;
  const player = { x: 0, y: 0, angle: 0, color: "#444444", spotColor: "#FFD700", dx: 0, dy: 0, spinSpeed: 0 };
  const spawnedSalamanders = [];
  const keys = { up: false, down: false, left: false, right: false };
  let isDiscoMode = false; let discoIntervalId = null;

  // Define pond properties globally so we can check against them
  const pond = {
      centerX: canvas.width * 0.2,
      centerY: canvas.height * 0.8,
      radiusX: canvas.width * 0.18,
      radiusY: canvas.height * 0.15
  };

  // Helper function to check if a salamander's center point is inside the elliptical pond
  function isInsidePond(salamanderX, salamanderY) {
      // Ellipse equation check: ((x-h)^2 / a^2) + ((y-k)^2 / b^2) <= 1
      const h = pond.centerX;
      const k = pond.centerY;
      const a = pond.radiusX;
      const b = pond.radiusY;
      const result = (Math.pow(salamanderX - h, 2) / Math.pow(a, 2)) + (Math.pow(salamanderY - k, 2) / Math.pow(b, 2));
      return result <= 1;
  }

  function drawEnvironment() {
      // Draw Grass background (Duller Green Shade)
      ctx.fillStyle = "#8FBC8F"; 
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // Draw Pond
      ctx.fillStyle = "#00BFFF"; 
      ctx.beginPath();
      ctx.ellipse(pond.centerX, pond.centerY, pond.radiusX, pond.radiusY, 0, 0, 2 * Math.PI);
      ctx.fill();
      ctx.strokeStyle = "#008CBA";
      ctx.lineWidth = 3;
      ctx.stroke();

      // Draw Lily Pad
      const lilyPadX = pond.centerX + pond.radiusX * 0.5;
      const lilyPadY = pond.centerY - pond.radiusY * 0.3;
      ctx.fillStyle = "#32CD32"; 
      ctx.beginPath();
      ctx.arc(lilyPadX, lilyPadY, 15, 0.2 * Math.PI, 1.8 * Math.PI);
      ctx.lineTo(lilyPadX + 15 * Math.cos(0.2 * Math.PI), lilyPadY + 15 * Math.sin(0.2 * Math.PI));
      ctx.closePath();
      ctx.fill();
  }
  

  function startGame() {
      document.getElementById("startScreen").style.display = "none";
      document.getElementById("gameContainer").style.display = "flex";
      player.x = canvas.width / 2;
      player.y = canvas.height / 2;
      requestAnimationFrame(gameLoop);
  }

  function move(direction, isPressed) { keys[direction] = isPressed; event.preventDefault(); }
  function getRandomColor() { const letters = '0123456789ABCDEF'; let color = '#'; for (let i = 0; i < 6; i++) { color += letters[Math.floor(Math.random() * 16)]; } return color; }
  function flashRainbowBackground() { document.body.style.backgroundColor = getRandomColor(); }
  function toggleDiscoMode() {
      isDiscoMode = !isDiscoMode; const button = document.getElementById("discoButton");
      if (isDiscoMode) { button.textContent = "STOP DISCO"; button.style.backgroundColor = "red"; discoIntervalId = setInterval(flashRainbowBackground, 100); [player, ...spawnedSalamanders].forEach(s => s.spinSpeed = 0.1); } 
      else { button.textContent = "DISCO MODE"; button.style.backgroundColor = "purple"; clearInterval(discoIntervalId); document.body.style.backgroundColor = "#f0f0f0"; [player, ...spawnedSalamanders].forEach(s => s.spinSpeed = 0); }
  }
  function spawnNewSalamander() {
    // Ensure spawned salamanders don't start in the pond
    let newX, newY;
    do {
        newX = Math.random() * (canvas.width - characterWidth) + characterWidth / 2;
        newY = Math.random() * (canvas.height - characterHeight) + characterHeight / 2;
    } while (isInsidePond(newX, newY));

    spawnedSalamanders.push({ x: newX, y: newY, angle: 0, color: getRandomColor(), spotColor: null, dx: (Math.random() - 0.5) * cloneSpeed * 2, dy: (Math.random() - 0.5) * cloneSpeed * 2, spinSpeed: 0 });
  }
  function changeCloneDirections() {
      if (!isDiscoMode) { for (const salamander of spawnedSalamanders) { salamander.dx = (Math.random() - 0.5) * cloneSpeed * 2; salamander.dy = (Math.random() - 0.5) * cloneSpeed * 2; } }
  }
  setInterval(changeCloneDirections, 3000);

  function drawSpots(drawX, drawY, spotColor) {
      if (!spotColor) return; ctx.fillStyle = spotColor;
      const spots = [{x: drawX + characterWidth * 0.15, y: drawY + characterHeight * 0.5}, {x: drawX + characterWidth * 0.35, y: drawY + characterHeight * 0.2}, {x: drawX + characterWidth * 0.35, y: drawY + characterHeight * 0.8}, {x: drawX + characterWidth * 0.5, y: drawY + characterHeight * 0.4}, {x: drawX + characterWidth * 0.6, y: drawY + characterHeight * 0.75}, {x: drawX + characterWidth * 0.8, y: drawY + characterHeight * 0.5},];
      spots.forEach(spot => { ctx.beginPath(); ctx.arc(spot.x, spot.y, 3, 0, 2 * Math.PI); ctx.fill(); });
  }

  function drawSalamander(x, y, drawAngle, color, spotColor) {
    const legColor = color; ctx.save(); ctx.translate(x, y); ctx.rotate(drawAngle); 
    const drawX = -characterWidth / 2; const drawY = -characterHeight / 2;
    ctx.fillStyle = color;
    ctx.beginPath();
    ctx.moveTo(drawX, drawY + characterHeight * 0.5); ctx.lineTo(drawX + characterWidth * 0.4, drawY); ctx.lineTo(drawX + characterWidth * 0.95, drawY); ctx.lineTo(drawX + characterWidth, drawY + characterHeight * 0.5); ctx.lineTo(drawX + characterWidth * 0.95, drawY + characterHeight); ctx.lineTo(drawX + characterWidth * 0.4, drawY + characterHeight);
    ctx.closePath(); ctx.fill();
    ctx.fillStyle = legColor;
    ctx.fillRect(drawX + characterWidth * 0.42, drawY + characterHeight - 5, 5, 8); ctx.fillRect(drawX + characterWidth * 0.42, drawY + 5 - 8, 5, 8); ctx.fillRect(drawX + characterWidth * 0.7, drawY + characterHeight - 5, 5, 8); ctx.fillRect(drawX + characterWidth * 0.7, drawY + 5 - 8, 5, 8); 
    drawSpots(drawX, drawY, spotColor);
    ctx.fillStyle = "black";
    ctx.beginPath(); ctx.arc(drawX + characterWidth * 0.85, drawY + characterHeight * 0.25, 2.5, 0, 2 * Math.PI); ctx.fill();
    ctx.beginPath(); ctx.arc(drawX + characterWidth * 0.85, drawY + characterHeight * 0.75, 2.5, 0, 2 * Math.PI); ctx.fill();
    ctx.restore();
  }

  function gameLoop() {
    drawEnvironment(); 
    
    // --- Update Player ---
    let originalPlayerX = player.x;
    let originalPlayerY = player.y;

    if (!isDiscoMode) { 
        player.dx = 0; player.dy = 0;
        if (keys.up) player.dy -= speed; if (keys.down) player.dy += speed; if (keys.left) player.dx -= speed; if (keys.right) player.dx += speed;
        if (player.dx !== 0 || player.dy !== 0) { player.angle = Math.atan2(player.dy, player.dx); }
    } else { player.angle += player.spinSpeed; }
    player.x += player.dx; player.y += player.dy;

    // Check player collision with pond
    if (isInsidePond(player.x, player.y)) {
        // If they move into the pond, revert to the previous position
        player.x = originalPlayerX;
        player.y = originalPlayerY;
    }

    player.x = Math.max(characterWidth / 2, Math.min(canvas.width - characterWidth / 2, player.x));
    player.y = Math.max(characterHeight / 2, Math.min(canvas.height - characterHeight / 2, player.y));
    drawSalamander(player.x, player.y, player.angle, player.color, player.spotColor);

    // --- Update and Draw Spawned Salamanders ---
    for (const salamander of spawnedSalamanders) {
        if (!isDiscoMode) {
            let originalCloneX = salamander.x;
            let originalCloneY = salamander.y;

            salamander.x += salamander.dx; 
            salamander.y += salamander.dy;

            // Check clone collision with pond
            if (isInsidePond(salamander.x, salamander.y)) {
                // Bounce them off the pond edge by reversing direction
                salamander.x = originalCloneX;
                salamander.y = originalCloneY;
                salamander.dx *= -1;
                salamander.dy *= -1;
            }

            // Wall collision (already had this)
            if (salamander.x <= characterWidth / 2 || salamander.x >= canvas.width - characterWidth / 2) { salamander.dx *= -1; }
            if (salamander.y <= characterHeight / 2 || salamander.y >= canvas.height - characterHeight / 2) { salamander.dy *= -1; }

            salamander.angle = Math.atan2(salamander.dy, salamander.dx);
        } else { salamander.angle += salamander.spinSpeed; }
        drawSalamander(salamander.x, salamander.y, salamander.angle, salamander.color, salamander.spotColor);
    }
    requestAnimationFrame(gameLoop);
  }
</script>

</body>
</html>
