<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Salamander Game: Secret Codes</title>
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
  #codeArea {
      margin-bottom: 10px;
      text-align: center;
  }
  #codeInput {
      padding: 5px;
      font-size: 14px;
      width: 150px; 
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
    <div id="codeArea">
        Enter Code: <input type="text" id="codeInput" onkeydown="checkCode(event)" placeholder="Midas or Poseidon">
    </div>

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
  // Base dimensions (will be scaled for the player if necessary)
  const BASE_CHAR_WIDTH = 100; 
  const BASE_CHAR_HEIGHT = 25; 
  const speed = 3;
  const cloneSpeed = 1;

  // Player dimension variables (dynamically updated by codes)
  let characterWidth = BASE_CHAR_WIDTH;
  let characterHeight = BASE_CHAR_HEIGHT;
  
  let playerBodyColor = "#444444";
  let playerSpotColor = "#FFD700";
  let playerLegColor = "#444444"; 
  let playerHasHat = false; 
  let playerHasSpidermanSuit = false; 

  const player = { x: 0, y: 0, angle: 0, dx: 0, dy: 0, spinSpeed: 0 };
  const spawnedSalamanders = [];
  const keys = { up: false, down: false, left: false, right: false };
  let isDiscoMode = false; let discoIntervalId = null;


  function checkCode(event) {
      if (event.key === 'Enter') {
          const code = document.getElementById("codeInput").value.toLowerCase();
          // Reset accessories and size
          playerHasHat = false; 
          playerHasSpidermanSuit = false;
          characterWidth = BASE_CHAR_WIDTH;
          characterHeight = BASE_CHAR_HEIGHT;

          switch(code) {
              case 'midas':
                  playerBodyColor = "#FFD700"; playerSpotColor = null; playerLegColor = "#FFD700"; break;
              case 'poseidon':
                  playerBodyColor = "#00BFFF"; playerSpotColor = null; playerLegColor = "#00BFFF"; break;
              case 'spiderman':
                  playerBodyColor = "#D50000"; playerSpotColor = null; playerLegColor = "#0047AB"; playerHasSpidermanSuit = true; break;
              case 'ethan':
                  playerBodyColor = "#FF69B4"; playerSpotColor = null; playerLegColor = "#FF69B4"; break;
              case 'mrpen':
                  playerBodyColor = "#D3D3D3"; playerSpotColor = null; playerLegColor = "#444444"; playerHasHat = true; break;
              case 'sans': // New Code Case: Sans (Blue body, white head)
                  playerBodyColor = "#0000FF"; // Blue body/tail
                  playerSpotColor = null;
                  playerLegColor = "#0000FF";
                  // We handle the white head in the draw function logic
                  break;
              case 'amphibiousdlc': // New Code Case: Huge, dark grey, two yellow stripes
                  playerBodyColor = "#444444"; // Dark grey
                  playerSpotColor = "#FFD700"; // Yellow stripes (using spots logic for stripes)
                  playerLegColor = "#444444";
                  characterWidth = 200; // Make huge
                  characterHeight = 50;
                  break;
              default: break;
          }
          document.getElementById("codeInput").value = '';
      }
  }

  function drawSuitAndHat(drawX, drawY, isPlayerHat = false) {
    if (!isPlayerHat) return; 
    ctx.fillStyle = "#222222"; 
    ctx.beginPath();
    ctx.ellipse(drawX + characterWidth * 0.85, drawY + characterHeight * 0.05, 18 * (characterWidth/BASE_CHAR_WIDTH), 5 * (characterHeight/BASE_CHAR_HEIGHT), 0, 0, 2 * Math.PI);
    ctx.fill();
    ctx.fillRect(drawX + characterWidth * 0.85 - 12 * (characterWidth/BASE_CHAR_WIDTH), drawY + characterHeight * 0.05 - 20 * (characterHeight/BASE_CHAR_HEIGHT), 24 * (characterWidth/BASE_CHAR_WIDTH), 20 * (characterHeight/BASE_CHAR_HEIGHT));
    ctx.fillStyle = "#444444"; 
    ctx.fillRect(drawX + characterWidth * 0.45, drawY, characterWidth * 0.25, characterHeight);
  }
  
  function drawSpidermanSuit(drawX, drawY, isSpiderman = false) {
      if (!isSpiderman) return;
      ctx.fillStyle = "#FFFFFF"; 
      ctx.beginPath();
      ctx.ellipse(drawX + characterWidth * 0.55, drawY + characterHeight * 0.5, 6, 4, 0, 0, 2 * Math.PI);
      ctx.fill();
      ctx.fillRect(drawX + characterWidth * 0.55 - 8, drawY + characterHeight * 0.5 - 1, 16, 2); 
  }


  function drawEnvironment() {
      ctx.fillStyle = "white"; 
      ctx.fillRect(0, 0, canvas.width, canvas.height);
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
    const randomX = Math.random() * (canvas.width - BASE_CHAR_WIDTH) + BASE_CHAR_WIDTH / 2; const randomY = Math.random() * (canvas.height - BASE_CHAR_HEIGHT) + BASE_CHAR_HEIGHT / 2;
    // Clones always use base size
    spawnedSalamanders.push({ x: randomX, y: randomY, angle: 0, color: getRandomColor(), spotColor: null, legColor: null, hasHat: false, hasSpiderman: false, width: BASE_CHAR_WIDTH, height: BASE_CHAR_HEIGHT, dx: (Math.random() - 0.5) * cloneSpeed * 2, dy: (Math.random() - 0.5) * cloneSpeed * 2, spinSpeed: 0 });
  }
  function changeCloneDirections() {
      if (!isDiscoMode) { for (const salamander of spawnedSalamanders) { salamander.dx = (Math.random() - 0.5) * cloneSpeed * 2; salamander.dy = (Math.random() - 0.5) * cloneSpeed * 2; } }
  }
  setInterval(changeCloneDirections, 3000);

  function drawSpots(drawX, drawY, spotColor, currentWidth, currentHeight) {
      if (!spotColor) return; ctx.fillStyle = spotColor;
      // Adjust spot positions based on current size ratio
      const spots = [
          {x: drawX + currentWidth * 0.15, y: drawY + currentHeight * 0.5}, {x: drawX + currentWidth * 0.35, y: drawY + currentHeight * 0.2}, 
          {x: drawX + currentWidth * 0.35, y: drawY + currentHeight * 0.8}, {x: drawX + currentWidth * 0.5, y: drawY + currentHeight * 0.4}, 
          {x: drawX + currentWidth * 0.6, y: drawY + currentHeight * 0.75}, {x: drawX + currentWidth * 0.8, y: drawY + currentHeight * 0.5},
      ];
      spots.forEach(spot => { ctx.beginPath(); ctx.arc(spot.x, spot.y, 3 * (currentWidth/BASE_CHAR_WIDTH), 0, 2 * Math.PI); ctx.fill(); });
  }

  function drawSalamander(x, y, drawAngle, color, spotColor, customLegColor, hasHat, hasSpiderman, currentWidth, currentHeight) {
    ctx.save(); ctx.translate(x, y); ctx.rotate(drawAngle); 
    const drawX = -currentWidth / 2; const drawY = -currentHeight / 2;
    
    ctx.fillStyle = color;
    ctx.beginPath();
    ctx.moveTo(drawX, drawY + currentHeight * 0.5); ctx.lineTo(drawX + currentWidth * 0.4, drawY); ctx.lineTo(drawX + currentWidth * 0.95, drawY); ctx.lineTo(drawX + currentWidth, drawY + currentHeight * 0.5); ctx.lineTo(drawX + currentWidth * 0.95, drawY + currentHeight); ctx.lineTo(drawX + currentWidth * 0.4, drawY + currentHeight);
    ctx.closePath(); ctx.fill();

    // Specific Sans code head override (White head section)
    if (color === "#0000FF") {
        ctx.fillStyle = "#FFFFFF"; // White
        ctx.beginPath();
        ctx.ellipse(
            drawX + currentWidth * 0.85, drawY + currentHeight * 0.5, 
            currentWidth * 0.12, currentHeight * 0.45,       
            0, 0, 2 * Math.PI
        );
        ctx.fill();
    }
    
    ctx.fillStyle = customLegColor || color;
    ctx.fillRect(drawX + currentWidth * 0.42, drawY + currentHeight - 5 * (currentHeight/BASE_CHAR_HEIGHT), 5 * (currentWidth/BASE_CHAR_WIDTH), 8 * (currentHeight/BASE_CHAR_HEIGHT)); 
    ctx.fillRect(drawX + currentWidth * 0.42, drawY + 5 * (currentHeight/BASE_CHAR_HEIGHT) - 8 * (currentHeight/BASE_CHAR_HEIGHT), 5 * (currentWidth/BASE_CHAR_WIDTH), 8 * (currentHeight/BASE_CHAR_HEIGHT)); 
    ctx.fillRect(drawX + currentWidth * 0.7, drawY + currentHeight - 5 * (currentHeight/BASE_CHAR_HEIGHT), 5 * (currentWidth/BASE_CHAR_WIDTH), 8 * (currentHeight/BASE_CHAR_HEIGHT)); 
    ctx.fillRect(drawX + currentWidth * 0.7, drawY + 5 * (currentHeight/BASE_CHAR_HEIGHT) - 8 * (currentHeight/BASE_CHAR_HEIGHT), 5 * (currentWidth/BASE_CHAR_WIDTH), 8 * (currentHeight/BASE_CHAR_HEIGHT)); 
    
    // Pass correct width/height to spot drawer
    drawSpots(drawX, drawY, spotColor, currentWidth, currentHeight); 
    drawSuitAndHat(drawX, drawY, hasHat); 
    drawSpidermanSuit(drawX, drawY, hasSpiderman); 

    ctx.fillStyle = "black";
    // Adjust eye size/position for scale
    ctx.beginPath(); ctx.arc(drawX + currentWidth * 0.85, drawY + currentHeight * 0.25, 2.5 * (currentWidth/BASE_CHAR_WIDTH), 0, 2 * Math.PI); ctx.fill();
    ctx.beginPath(); ctx.arc(drawX + currentWidth * 0.85, drawY + currentHeight * 0.75, 2.5 * (currentWidth/BASE_CHAR_WIDTH), 0, 2 * Math.PI); ctx.fill();
    ctx.restore();
  }

  function gameLoop() {
    drawEnvironment(); 
    
    // --- Update Player ---
    if (!isDiscoMode) { 
        player.dx = 0; player.dy = 0;
        if (keys.up) player.dy -= speed; if (keys.down) player.dy += speed; if (keys.left) player.dx -= speed; if (keys.right) player.dx += speed;
        if (player.dx !== 0 || player.dy !== 0) { player.angle = Math.atan2(player.dy, player.dx); }
    } else { player.angle += player.spinSpeed; }
    player.x += player.dx; player.y += player.dy;
    // Use dynamic characterWidth/Height for player
    player.x = Math.max(characterWidth / 2, Math.min(canvas.width - characterWidth / 2, player.x));
    player.y = Math.max(characterHeight / 2, Math.min(canvas.height - characterHeight / 2, player.y));
    
    drawSalamander(player.x, player.y, player.angle, playerBodyColor, playerSpotColor, playerLegColor, playerHasHat, playerHasSpidermanSuit, characterWidth, characterHeight);

    // --- Update and Draw Spawned Salamanders ---
    for (const salamander of spawnedSalamanders) {
        if (!isDiscoMode) {
            salamander.x += salamander.dx; salamander.y += salamander.dy;
            // Use clone's specific dimensions for collision
            if (salamander.x <= salamander.width / 2 || salamander.x >= canvas.width - salamander.width / 2) { salamander.dx *= -1; }
            if (salamander.y <= salamander.height / 2 || salamander.y >= canvas.height - salamander.height / 2) { salamander.dy *= -1; }
            salamander.angle = Math.atan2(salamander.dy, salamander.dx);
        } else { salamander.angle += salamander.spinSpeed; }
        // Pass clone properties
        drawSalamander(salamander.x, salamander.y, salamander.angle, salamander.color, salamander.spotColor, salamander.legColor, salamander.hasHat, salamander.hasSpiderman, salamander.width, salamander.height);
    }
    requestAnimationFrame(gameLoop);
  }
</script>

</body>
</html>
