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
        canvas {
            background-color: #eee;
            border: 2px solid black;
            display: block;
        }
        #game-container {
            position: relative;
            /* Flex container for the game screen and input screen overlay */
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        /* --- Start Screen Styles --- */
        #start-screen {
            position: absolute; /* Overlay the canvas area */
            top: 0;
            left: 0;
            width: 100%;
            height: 100%; /* Spans the canvas height */
            background-color: rgba(0, 0, 0, 0.8);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            padding: 20px;
            box-sizing: border-box;
        }
        #start-screen h1, #start-screen label {
            margin-bottom: 10px;
        }
        #start-screen input {
            padding: 10px;
            font-size: 1em;
            margin-bottom: 20px;
        }
        .color-option {
            width: 40px;
            height: 40px;
            cursor: pointer;
            border: 2px solid transparent;
            margin: 5px;
            display: inline-block;
        }
        .color-option.selected {
            border-color: white;
        }
        #start-button {
            padding: 10px 20px;
            font-size: 1.2em;
            margin-top: 20px;
        }

        /* --- Controls & Info Styles (Below Canvas) --- */
        #controls-container {
            margin-top: 15px;
            display: flex;
            gap: 10px;
        }
        #controls-container button {
            padding: 15px 25px;
            font-size: 1.2em;
            cursor: pointer;
            background-color: #f0f0f0;
            border: 2px solid #ccc;
            border-radius: 8px;
            user-select: none;
        }
        #delete-btn.active {
            background-color: #e74c3c;
            color: white;
            border-color: #c0392b;
        }
        #info-message {
            margin-top: 10px;
            font-size: 0.8em;
            color: #ccc;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <canvas id="gameCanvas" width="600" height="400"></canvas>
        
        <!-- Start Screen Overlay -->
        <div id="start-screen">
            <h1>Square Jumper Setup</h1>
            <label for="username-input">Enter Username:</label>
            <input type="text" id="username-input" maxlength="15" placeholder="Player Name">
            
            <p>Choose your square color:</p>
            <div id="color-picker">
                <!-- Color options with data attributes -->
                <span class="color-option" style="background-color: red;" data-color="red" onclick="selectColor('red')"></span>
                <span class="color-option" style="background-color: yellow;" data-color="yellow" onclick="selectColor('yellow')"></span>
                <span class="color-option" style="background-color: blue;" data-color="blue" onclick="selectColor('blue')"></span>
                <span class="color-option" style="background-color: green;" data-color="green" onclick="selectColor('green')"></span>
                <span class="color-option" style="background-color: black;" data-color="black" onclick="selectColor('black')"></span>
            </div>
            
            <button id="start-button" onclick="startGame()">Start Game</button>
        </div>
    </div>
    
    <div id="controls-container">
        <button id="left-btn" onmousedown="setKey(event, 'left', true)" onmouseup="setKey(event, 'left', false)" ontouchstart="setKey(event, 'left', true)" ontouchend="setKey(event, 'left', false)">&#8592;</button>
        <button id="jump-btn" onmousedown="setKey(event, 'jump', true)" onmouseup="setKey(event, 'jump', false)" ontouchstart="setKey(event, 'jump', true)" ontouchend="setKey(event, 'jump', false)">Jump</button>
        <button id="right-btn" onmousedown="setKey(event, 'right', true)" onmouseup="setKey(event, 'right', false)" ontouchstart="setKey(event, 'right', true)" ontouchend="setKey(event, 'right', false)">&#8594;</button>
        <button id="delete-btn" onclick="toggleDeleteMode()">Delete Mode: OFF</button>
    </div>
    <div id="info-message">Left-click the grid to place blocks. Right-click or use the button to delete.</div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        const deleteBtn = document.getElementById("delete-btn");
        const startScreen = document.getElementById("start-screen");
        const usernameInput = document.getElementById("username-input");

        const GRID_SIZE = 40;
        const PLAYER_SIZE = 38;
        const GRAVITY = 0.5;
        const JUMP_FORCE = -10;
        const MOVE_SPEED = 5;

        let player = {
            x: GRID_SIZE, y: canvas.height - GRID_SIZE * 2, width: PLAYER_SIZE, height: PLAYER_SIZE,
            velX: 0, velY: 0, isJumping: false, groundY: canvas.height - GRID_SIZE,
            color: 'red', // Default color
            username: 'Player'
        };
        let keys = { left: false, right: false, jump: false };
        let map = []; 
        let deleteMode = false;

        // Start screen initialization
        let selectedColor = 'red'; // Default selection
        document.querySelector('.color-option[data-color="red"]').classList.add('selected');


        for (let i = 0; i < canvas.width / GRID_SIZE; i++) {
            map.push({ x: i * GRID_SIZE, y: canvas.height - GRID_SIZE, width: GRID_SIZE, height: GRID_SIZE });
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
            startScreen.style.display = 'none'; // Hide the start screen
            gameLoop(); // Start the main game loop
        }
        // End Game Setup Functions

        // --- Event Listeners ---
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
        function toggleDeleteMode() {
            deleteMode = !deleteMode;
            if (deleteMode) {
                deleteBtn.textContent = "Delete Mode: ON";
                deleteBtn.classList.add('active');
                canvas.style.cursor = 'crosshair';
            } else {
                deleteBtn.textContent = "Delete Mode: OFF";
                deleteBtn.classList.remove('active');
                canvas.style.cursor = 'default';
            }
        }
        canvas.addEventListener('click', function(event) {
            handleBlockInteraction(event, 'place');
        });
        canvas.addEventListener('contextmenu', function(event) {
            event.preventDefault();
            handleBlockInteraction(event, 'delete');
        });

        function handleBlockInteraction(event, mode) {
            const rect = canvas.getBoundingClientRect();
            const mouseX = event.clientX - rect.left;
            const mouseY = event.clientY - rect.top;
            const gridX = Math.floor(mouseX / GRID_SIZE) * GRID_SIZE;
            const gridY = Math.floor(mouseY / GRID_SIZE) * GRID_SIZE;
            if (gridY === player.groundY) return;
            const existingIndex = map.findIndex(block => block.x === gridX && block.y === gridY);
            if (mode === 'delete' || deleteMode) {
                if (existingIndex !== -1) {
                    map.splice(existingIndex, 1);
                }
            } else {
                if (existingIndex === -1) {
                    map.push({ x: gridX, y: gridY, width: GRID_SIZE, height: GRID_SIZE });
                }
            }
        }
        // End Event Listeners

        function isColliding(rect1, rect2) {
            return rect1.x < rect2.x + rect2.width && rect1.x + rect1.width > rect2.x &&
                   rect1.y < rect2.y + rect2.height && rect1.y + rect1.height > rect2.y;
        }

        function update() {
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
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.strokeStyle = '#ccc';
            for (let i = 0; i < canvas.width / GRID_SIZE; i++) {
                ctx.beginPath(); ctx.moveTo(i * GRID_SIZE, 0); ctx.lineTo(i * GRID_SIZE, canvas.height); ctx.stroke();
            }
            for (let i = 0; i < canvas.height / GRID_SIZE; i++) {
                ctx.beginPath(); ctx.moveTo(0, i * GRID_SIZE); ctx.lineTo(canvas.width, i * GRID_SIZE); ctx.stroke();
            }

            ctx.fillStyle = 'gray';
            map.forEach(block => {
                ctx.fillRect(block.x, block.y, block.width, block.height);
            });

            // Draw Player Square
            ctx.fillStyle = player.color;
            ctx.fillRect(player.x, player.y, player.width, player.height);

            // Draw Username above the player
            ctx.fillStyle = 'black';
            ctx.font = '12px sans-serif';
            ctx.textAlign = 'center';
            // Position the text slightly above the square
            ctx.fillText(player.username, player.x + player.width / 2, player.y - 5);
        }

        // The game loop is initially NOT running. It starts when the Start Game button is clicked.
        // function gameLoop() is defined but not called yet.
        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

    </script>
</body>
</html>
