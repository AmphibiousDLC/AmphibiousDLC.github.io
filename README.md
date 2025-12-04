<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Custom Square Jumper Game</title>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #333;
            margin: 0;
            flex-direction: column;
            color: white;
            font-family: sans-serif;
        }

        #main-layout-container {
            display: flex;
            gap: 15px;
        }

        canvas {
            border: 2px solid black;
            display: block;
        }
        #game-container {
            position: relative;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        /* --- Start Screen Styles (same as before) --- */
        #start-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            padding: 20px;
            box-sizing: border-box;
        }
        #start-screen h1, #start-screen label { margin-bottom: 10px; }
        #start-screen input { padding: 10px; font-size: 1em; margin-bottom: 20px; }
        .color-option { width: 40px; height: 40px; cursor: pointer; border: 2px solid transparent; margin: 5px; display: inline-block; }
        .color-option.selected { border-color: white; }
        #start-button { padding: 10px 20px; font-size: 1.2em; margin-top: 20px; }

        /* --- Controls & Info Styles --- */

        #side-controls-container {
            display: flex;
            flex-direction: column;
            gap: 10px;
            align-items: stretch;
        }

        #movement-controls-container {
            margin-top: 15px;
            display: flex;
            gap: 10px;
            justify-content: center;
        }

        #movement-controls-container button, #side-controls-container button {
            padding: 10px 15px;
            font-size: 1em;
            cursor: pointer;
            background-color: #f0f0f0;
            border: 2px solid #ccc;
            border-radius: 8px;
            user-select: none;
        }

        #left-btn, #right-btn {
            font-size: 2em;
            width: 80px;
            height: 80px;
        }
        #jump-btn {
            font-size: 1.5em;
            width: 120px;
            height: 80px;
        }

        #delete-btn.active { background-color: #e74c3c; color: white; border-color: #c0392b; }
        #portal-btn.active { background-color: #9b59b6; color: white; border-color: #8e44ad; }
        #death-btn.active { background-color: #a00; color: white; border-color: #700; }
        #checkpoint-btn.active { background-color: #f1c40f; color: black; border-color: #d35400; }

        #info-message {
            margin-top: 10px;
            font-size: 0.8em;
            color: #ccc;
            text-align: center;
        }
        #game-message-overlay {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: red;
            font-size: 4em;
            font-weight: bold;
            display: none;
            pointer-events: none;
            z-index: 10;
        }
    </style>
</head>
<body>
    <div id="main-layout-container">
        <!-- Left Side: Canvas and Movement Controls -->
        <div>
            <div id="game-container">
                <!-- CANVAS SIZE ADJUSTED TO 512x400 (32x25 grid @ 16px) -->
                <canvas id="gameCanvas" width="512" height="400"></canvas>
                <div id="game-message-overlay">YOU DED NOW</div>
                
                <!-- Start Screen Overlay -->
                <div id="start-screen">
                    <h1>Square Jumper Setup</h1>
                    <label for="username-input">Enter Username:</label>
                    <input type="text" id="username-input" maxlength="15" placeholder="Player Name">
                    
                    <p>Choose your square color:</p>
                    <div id="color-picker">
                        <span class="color-option" style="background-color: red;" data-color="red" onclick="selectColor('red')"></span>
                        <span class="color-option" style="background-color: yellow;" data-color="yellow" onclick="selectColor('yellow')"></span>
                        <span class="color-option" style="background-color: blue;" data-color="blue" onclick="selectColor('blue')"></span>
                        <span class="color-option" style="background-color: green;" data-color="green" onclick="selectColor('green')"></span>
                        <span class="color-option" style="background-color: black;" data-color="black" onclick="selectColor('black')"></span>
                    </div>
                    
                    <button id="start-button" onclick="startGame()">Start Game</button>
                </div>
            </div>
            
            <!-- Movement Buttons Below Canvas -->
            <div id="movement-controls-container">
                <button id="left-btn" onmousedown="setKey(event, 'left', true)" onmouseup="setKey(event, 'left', false)" ontouchstart="setKey(event, 'left', true)" ontouchend="setKey(event, 'left', false)">←</button>
                <button id="jump-btn" onmousedown="setKey(event, 'jump', true)" onmouseup="setKey(event, 'jump', false)" ontouchstart="setKey(event, 'jump', true)" ontouchend="setKey(event, 'jump', false)">Jump</button>
                <button id="right-btn" onmousedown="setKey(event, 'right', true)" onmouseup="setKey(event, 'right', false)" ontouchstart="setKey(event, 'right', true)" ontouchend="setKey(event, 'right', false)">→</button>
            </div>
            <div id="info-message">Left-click the grid to place blocks/items in the selected mode.</div>

        </div>

        <!-- Right Side: Mode/Build Buttons -->
        <div id="side-controls-container">
            <button id="delete-btn" onclick="toggleDeleteMode()">Delete Mode: OFF</button>
            <button id="portal-btn" onclick="togglePortalMode()">Portal Mode: OFF</button>
            <button id="death-btn" onclick="toggleDeathMode()">Death Mode: OFF</button>
            <button id="checkpoint-btn" onclick="toggleCheckpointMode()">Checkpoint Mode: OFF</button>
        </div>
    </div>
    
    <script>
        // GRID SIZE CHANGED HERE TO 16
        const GRID_SIZE = 16; 

        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        const deleteBtn = document.getElementById("delete-btn");
        const portalBtn = document.getElementById("portal-btn");
        const deathBtn = document.getElementById("death-btn");
        const checkpointBtn = document.getElementById("checkpoint-btn");
        const startScreen = document.getElementById("start-screen");
        const gameMessageOverlay = document.getElementById("game-message-overlay");
        const usernameInput = document.getElementById("username-input");

        const PLAYER_SIZE = GRID_SIZE - 2; // Keep player slightly smaller than grid for visibility
        const GRAVITY = 0.5;
        const JUMP_FORCE = -8; // Adjusted jump force for smaller scale
        const MOVE_SPEED = 4; // Adjusted move speed for smaller scale
        const initialSpawnX = GRID_SIZE;
        const initialSpawnY = canvas.height - GRID_SIZE * 2;

        let player = {
            x: initialSpawnX, y: initialSpawnY, width: PLAYER_SIZE, height: PLAYER_SIZE,
            velX: 0, velY: 0, isJumping: false, groundY: canvas.height - GRID_SIZE,
            color: 'red',
            username: 'Player',
            spawnX: initialSpawnX,
            spawnY: initialSpawnY
        };
        let keys = { left: false, right: false, jump: false };
        let map = [];
        let portals = [];
        let deathBlocks = [];
        let checkpoints = [];

        let deleteMode = false;
        let portalMode = false;
        let deathMode = false;
        let checkpointMode = false;
        let selectedColor = 'red';
        let canTeleport = true;
        let isDead = false;

        // Create the floor
        for (let i = 0; i < canvas.width / GRID_SIZE; i++) {
            map.push({ x: i * GRID_SIZE, y: canvas.height - GRID_SIZE, width: GRID_SIZE, height: GRID_SIZE, type: 'normal' });
        }

        // --- Game Setup Functions ---
        function selectColor(color) {
            selectedColor = color;
            document.querySelectorAll('.color-option').forEach(span => {
                span.classList.remove('selected');
            });
            document.querySelector(`.color-option[data-color="${color}"]`).classList.add('selected');
        }

        function startGame() {
            const username = usernameInput.value.trim();
            if (username) {
                player.username = username;
            }
            player.color = selectedColor;
            startScreen.style.display = 'none';
            gameLoop();
        }

        // --- Event Listeners and Modes ---
        document.addEventListener("keydown", function(event) {
            if (event.key === "ArrowLeft") keys.left = true;
            if (event.key === "ArrowRight") keys.right = true;
            if (event.key === " ") keys.jump = true;
        });
        document.addEventListener("keyup", function(event) {
            if (event.key === "ArrowLeft") keys.left = false;
            if (event.key === "ArrowRight") keys.right = false;
            if (event.key === " ") keys.jump = false;
        });
        function setKey(event, keyName, isPressed) {
            event.preventDefault();
            keys[keyName] = isPressed;
        }
        function toggleMode(modeName) {
            deleteMode = false; portalMode = false; deathMode = false; checkpointMode = false;
            canvas.style.cursor = 'default';
            if (modeName === 'delete') deleteMode = true;
            else if (modeName === 'portal') portalMode = true;
            else if (modeName === 'death') deathMode = true;
            else if (modeName === 'checkpoint') checkpointMode = true;
            deleteBtn.classList.toggle('active', deleteMode);
            portalBtn.classList.toggle('active', portalMode);
            deathBtn.classList.toggle('active', deathMode);
            checkpointBtn.classList.toggle('active', checkpointMode);
            if (deleteMode || portalMode || deathMode || checkpointMode) {
                canvas.style.cursor = 'crosshair';
            }
        }
        function toggleDeleteMode() { toggleMode(deleteMode ? 'none' : 'delete'); }
        function togglePortalMode() { toggleMode(portalMode ? 'none' : 'portal'); }
        function toggleDeathMode() { toggleMode(deathMode ? 'none' : 'death'); }
        function toggleCheckpointMode() { toggleMode(checkpointMode ? 'none' : 'checkpoint'); }


        canvas.addEventListener('click', function(event) {
            if (startScreen.style.display !== 'none' || isDead) return;
            if (deleteMode) {
                handleBlockInteraction(event, 'delete');
            } else if (portalMode) {
                handleBlockInteraction(event, 'portal');
            } else if (deathMode) {
                handleBlockInteraction(event, 'death');
            } else if (checkpointMode) {
                handleBlockInteraction(event, 'checkpoint');
            } else {
                handleBlockInteraction(event, 'place');
            }
        });
        canvas.addEventListener('contextmenu', function(event) {
            event.preventDefault();
            if (startScreen.style.display !== 'none' || isDead) return;
            handleBlockInteraction(event, 'delete');
        });

        function handleBlockInteraction(event, mode) {
            const rect = canvas.getBoundingClientRect();
            const mouseX = event.clientX - rect.left;
            const mouseY = event.clientY - rect.top;
            const gridX = Math.floor(mouseX / GRID_SIZE) * GRID_SIZE;
            const gridY = Math.floor(mouseY / GRID_SIZE) * GRID_SIZE;
            if (gridY === player.groundY) return;

            const isOccupied = map.some(b => b.x === gridX && b.y === gridY) ||
                               portals.some(b => b.x === gridX && b.y === gridY) ||
                               deathBlocks.some(b => b.x === gridX && b.y === gridY) ||
                               checkpoints.some(b => b.x === gridX && b.y === gridY);

            if (mode === 'delete') {
                if (gridX === player.spawnX && gridY === player.spawnY) {
                    player.spawnX = initialSpawnX;
                    player.spawnY = initialSpawnY;
                    console.log("Current spawn point deleted. Spawn reset to start.");
                }
                map = map.filter(b => !(b.x === gridX && b.y === gridY));
                portals = portals.filter(b => !(b.x === gridX && b.y === gridY));
                deathBlocks = deathBlocks.filter(b => !(b.x === gridX && b.y === gridY));
                checkpoints = checkpoints.filter(b => !(b.x === gridX && b.y === gridY));
            } else if (mode === 'portal' && !isOccupied) {
                if (portals.length >= 2) portals.shift(); 
                portals.push({ x: gridX, y: gridY, width: GRID_SIZE, height: GRID_SIZE });
            } else if (mode === 'death' && !isOccupied) {
                deathBlocks.push({ x: gridX, y: gridY, width: GRID_SIZE, height: GRID_SIZE });
            } else if (mode === 'checkpoint' && !isOccupied) {
                checkpoints.push({ x: gridX, y: gridY, width: GRID_SIZE, height: GRID_SIZE });
            } else if (mode === 'place' && !isOccupied) {
                map.push({ x: gridX, y: gridY, width: GRID_SIZE, height: GRID_SIZE, type: 'normal' });
            }
        }
        // End EventListeners

        function isColliding(rect1, rect2) {
            return rect1.x < rect2.x + rect2.width && rect1.x + rect1.width > rect2.x &&
                   rect1.y < rect2.y + rect2.height && rect1.y + rect1.height > rect2.y;
        }
        
        function checkPortalCollision() {
            if (portals.length !== 2 || !canTeleport) return;
            portals.forEach((portal, index) => {
                if (isColliding(player, portal)) {
                    canTeleport = false;
                    const destinationPortal = portals[1 - index];
                    player.x = destinationPortal.x + destinationPortal.width + 2; // Small buffer for new grid size
                    player.y = destinationPortal.y; 
                    setTimeout(() => { canTeleport = true; }, 300);
                }
            });
        }

        function setCheckpoint(checkpoint) {
            player.spawnX = checkpoint.x;
            player.spawnY = checkpoint.y;
        }

        function handleDeath() {
            const atInitialSpawn = (player.spawnX === initialSpawnX && player.spawnY === initialSpawnY);
            
            if (atInitialSpawn) {
                isDead = true; 
                gameMessageOverlay.style.display = 'block'; 
                
                map = map.filter(block => block.y === player.groundY);
                portals = [];
                deathBlocks = [];
                checkpoints = [];

                setTimeout(() => {
                    gameMessageOverlay.style.display = 'none';
                    isDead = false; 
                }, 1500); 
            }
            
            player.x = player.spawnX;
            player.y = player.spawnY;
            player.velX = 0;
            player.velY = 0;
            player.isJumping = false;
        }

        function update() {
            if (isDead) return;

            checkPortalCollision(); 

            deathBlocks.forEach(block => {
                if (isColliding(player, block)) {
                    handleDeath();
                }
            });

            checkpoints.forEach(block => {
                if (isColliding(player, block)) {
                    setCheckpoint(block);
                }
            });

            if (keys.left) player.velX = -MOVE_SPEED;
            if (keys.right) player.velX = MOVE_SPEED;
            if (!keys.left && !keys.right) player.velX = 0;
            player.x += player.velX;
            map.forEach(block => {
                if (isColliding(player, block)) {
                    if (player.velX > 0) player.x = block.x - player.width;
                    else if (player.velX < 0) player.x = block.x + block.width;
                }
            });
            if (player.x < 0) player.x = 0;
            if (player.x + player.width > canvas.width) player.x = canvas.width - player.width;
            if (keys.jump && !player.isJumping) {
                player.velY = JUMP_FORCE;
            }
            player.velY += GRAVITY;
            player.y += player.velY;
            player.isJumping = true; 
            map.forEach(block => {
                if (isColliding(player, block)) {
                    if (player.velY > 0) {
                        player.y = block.y - player.height;
                        player.velY = 0;
                        player.isJumping = false;
                    } else if (player.velY < 0) {
                        player.y = block.y + block.height;
                        player.velY = 0;
                    }
                }
            });
        }

        function draw() {
            if (isDead) {
                ctx.fillStyle = 'black';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
            } else {
                ctx.fillStyle = '#eee';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
            }
            
            if (isDead) {
                // Skip drawing game elements during the "dead" blackout screen
            } else {
                ctx.strokeStyle = '#ccc';
                // Line width needs to be adjusted for the smaller grid size to look good
                ctx.lineWidth = 0.5; 
                for (let i = 0; i < canvas.width / GRID_SIZE; i++) {
                    ctx.beginPath(); ctx.moveTo(i * GRID_SIZE, 0); ctx.lineTo(i * GRID_SIZE, canvas.height); ctx.stroke();
                }
                for (let i = 0; i < canvas.height / GRID_SIZE; i++) {
                    ctx.beginPath(); ctx.moveTo(0, i * GRID_SIZE); ctx.lineTo(canvas.width, i * GRID_SIZE); ctx.stroke();
                }
                ctx.lineWidth = 1; // Reset default line width


                ctx.fillStyle = 'gray';
                map.forEach(block => { ctx.fillRect(block.x, block.y, block.width, block.height); });
                portals.forEach(block => {
                    ctx.fillStyle = 'purple';
                    ctx.fillRect(block.x, block.y, block.width, block.height);
                    ctx.beginPath();
                    ctx.arc(block.x + block.width / 2, block.y + block.height / 2, block.width / 4, 0, 2 * Math.PI);
                    ctx.fillStyle = 'white';
                    ctx.fill();
                });
                deathBlocks.forEach(block => {
                    ctx.fillStyle = 'red';
                    ctx.fillRect(block.x, block.y, block.width, block.height);
                    ctx.strokeStyle = 'black';
                    ctx.lineWidth = 1;
                    ctx.beginPath();
                    ctx.moveTo(block.x, block.y); ctx.lineTo(block.x + block.width, block.y + block.height);
                    ctx.moveTo(block.x + block.width, block.y); ctx.lineTo(block.x, block.y + block.height);
                    ctx.stroke();
                });
                checkpoints.forEach(block => {
                    ctx.fillStyle = 'yellow';
                    ctx.fillRect(block.x, block.y, block.width, block.height);
                    ctx.beginPath();
                    ctx.arc(block.x + block.width / 2, block.y + block.height / 2, block.width / 4, 0, 2 * Math.PI);
                    ctx.fillStyle = 'green';
                    ctx.fill();
                });

                ctx.fillStyle = player.color;
                ctx.fillRect(player.x, player.y, player.width, player.height);

                ctx.fillStyle = 'black';
                ctx.font = '12px sans-serif';
                ctx.textAlign = 'center';
                ctx.fillText(player.username, player.x + player.width / 2, player.y - 5);
            }
        }

        function gameLoop() {
            if (startScreen.style.display === 'none') {
                update();
                draw();
            }
            requestAnimationFrame(gameLoop);
        }

    </script>
</body>
</html>
