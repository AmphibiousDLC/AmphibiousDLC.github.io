<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Square Jumper Game with Delete</title>
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
            background-color: #e74c3c; /* Red color when active */
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
    <canvas id="gameCanvas" width="600" height="400"></canvas>
    
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

        const GRID_SIZE = 40;
        const PLAYER_SIZE = 38;
        const GRAVITY = 0.5;
        const JUMP_FORCE = -10;
        const MOVE_SPEED = 5;

        let player = {
            x: GRID_SIZE, y: canvas.height - GRID_SIZE * 2, width: PLAYER_SIZE, height: PLAYER_SIZE,
            velX: 0, velY: 0, isJumping: false, groundY: canvas.height - GRID_SIZE
        };
        let keys = { left: false, right: false, jump: false };
        let map = []; 
        let deleteMode = false; // New state variable

        for (let i = 0; i < canvas.width / GRID_SIZE; i++) {
            map.push({ x: i * GRID_SIZE, y: canvas.height - GRID_SIZE, width: GRID_SIZE, height: GRID_SIZE });
        }

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

        // Toggle function for the delete button
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

        // Main canvas click handler
        canvas.addEventListener('click', function(event) {
            handleBlockInteraction(event, 'place');
        });
        canvas.addEventListener('contextmenu', function(event) {
            event.preventDefault(); // Prevent right-click menu
            handleBlockInteraction(event, 'delete');
        });

        function handleBlockInteraction(event, mode) {
            const rect = canvas.getBoundingClientRect();
            const mouseX = event.clientX - rect.left;
            const mouseY = event.clientY - rect.top;
            const gridX = Math.floor(mouseX / GRID_SIZE) * GRID_SIZE;
            const gridY = Math.floor(mouseY / GRID_SIZE) * GRID_SIZE;

            // Ensure we don't delete/place in the player's initial spawn floor row
            if (gridY === player.groundY) return;

            const existingIndex = map.findIndex(block => block.x === gridX && block.y === gridY);

            if (mode === 'delete' || deleteMode) {
                if (existingIndex !== -1) {
                    map.splice(existingIndex, 1); // Remove the block
                }
            } else { // mode === 'place'
                if (existingIndex === -1) {
                    map.push({ x: gridX, y: gridY, width: GRID_SIZE, height: GRID_SIZE });
                }
            }
        }

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
                    if (player.velY > 0) { // Landed on a block
                        player.y = block.y - player.height;
                        player.velY = 0;
                        player.isJumping = false;
                    } else if (player.velY < 0) { // Hit head on a block
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

            ctx.fillStyle = 'red';
            ctx.fillRect(player.x, player.y, player.width, player.height);
        }

        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        gameLoop();
    </script>
</body>
</html>
