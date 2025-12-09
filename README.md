<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Salamander Friends Game (Themed Names 2)</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #333;
            user-select: none;
            color: white;
            font-family: sans-serif;
        }

        canvas {
            background-color: #eee;
            border: 2px solid black;
            margin-bottom: 10px;
            cursor: default;
        }

        /* Controls positioned in a cross shape */
        .controls {
            display: grid;
            grid-template-areas: 
                ". up ."
                "left middle right"
                ". down .";
            gap: 8px;
            width: 180px;
        }

        .control-btn {
            background-color: #4CAF50;
            border: none;
            color: white;
            padding: 18px; 
            font-size: 20px; 
            cursor: pointer;
            border-radius: 5px;
            touch-action: manipulation;
            text-align: center;
        }

        #gameSetup, #titleScreenContainer {
            position: absolute;
            background-color: rgba(0, 0, 0, 0.8);
            padding: 40px;
            border-radius: 10px;
            text-align: center;
            z-index: 10;
        }
        #titleScreenContainer h1 {
            font-size: 48px; color: yellow;
            text-shadow: -2px -2px 0 #006400, 2px -2px 0 #006400, -2px 2px 0 #006400, 2px 2px 0 #006400;
            margin: 0 0 10px 0;
        }
        #titleScreenContainer h2 { font-size: 18px; color: white; margin: 0 0 20px 0; }
        
        #gameControlsContainer {
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .action-controls {
            display: flex;
            gap: 10px;
            margin-top: 10px;
            align-items: center;
            flex-wrap: wrap;
            justify-content: center;
        }
        #spawnNameInput { padding: 8px; width: 120px; }
        #spawnBtn, .mode-btn {
            padding: 8px 15px; background-color: #008CBA; cursor: pointer; border: none; border-radius: 5px; color: white;
        }
        .mode-btn.active { background-color: #005f8a; border: 2px solid white; }

    </style>
</head>
<body>
    <div id="titleScreenContainer">
        <h1>SALAMANDER FRIENDS</h1>
        <h2>BY AMPHIBIOUSDLC</h2>
        <canvas id="titleCanvas" width="200" height="200"></canvas>
        <p>Click 'Start Game' below to begin!</p>
    </div>

    <div id="gameSetup">
        <label for="usernameInput">Enter your salamander's name:</label>
        <input type="text" id="usernameInput" maxlength="15" placeholder="Username">
        <button id="startButton">Start Game</button>
    </div>

    <canvas id="gameCanvas" style="display:none;"></canvas>

    <div style="display:none;" id="gameControlsContainer">
        <div class="controls">
            <button id="up" class="control-btn">↑</button>
            <button id="left" class="control-btn">←</button>
            <button id="right" class="control-btn">→</button>
            <button id="down" class="control-btn">↓</button>
        </div>
        <div class="action-controls">
            <button id="toggleWaterBtn" class="mode-btn">Place Water</button>
            <button id="toggleDeleteModeBtn" class="mode-btn">Delete Blocks</button>
            <button id="toggleLilyPadModeBtn" class="mode-btn">Place Lily Pad</button>
            <input type="text" id="spawnNameInput" maxlength="15" placeholder="Name (optional)">
            <button id="spawnBtn">Spawn Salamander</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const titleCanvas = document.getElementById('titleCanvas');
        const titleCtx = titleCanvas.getContext('2d');
        const gameSetupScreen = document.getElementById('gameSetup');
        const titleScreenContainer = document.getElementById('titleScreenContainer');
        const startButton = document.getElementById('startButton');
        const usernameInput = document.getElementById('usernameInput');
        const controlsContainer = document.getElementById('gameControlsContainer');
        const spawnBtn = document.getElementById('spawnBtn');
        const spawnNameInput = document.getElementById('spawnNameInput');
        const toggleWaterBtn = document.getElementById('toggleWaterBtn');
        const toggleDeleteModeBtn = document.getElementById('toggleDeleteModeBtn');
        const toggleLilyPadModeBtn = document.getElementById('toggleLilyPadModeBtn');

        canvas.width = 600; 
        canvas.height = 400;
        
        const SALAMANDER_SCALE = 0.7; 
        const TUBE_PADDING = 60;
        const AVOIDANCE_THRESHOLD = 80;
        const TILE_SIZE = 40;

        let gameRunning = false;
        let currentMode = 'none'; 
        let salamanders = [];
        let waterBlocks = []; 
        let lilyPads = []; 
        let keys = {'ArrowUp': false, 'up': false, 'ArrowDown': false, 'down': false, 'ArrowLeft': false, 'left': false, 'ArrowRight': false, 'right': false};

        // Input Handlers
        document.addEventListener('keydown', (e) => { if (keys.hasOwnProperty(e.key)) keys[e.key] = true; });
        document.addEventListener('keyup', (e) => { if (keys.hasOwnProperty(e.key)) keys[e.key] = false; });
        document.querySelectorAll('.control-btn').forEach(button => {
            const direction = button.id;
            const startHandler = () => { keys[direction] = true; };
            const endHandler = () => { keys[direction] = false; };
            button.addEventListener('mousedown', startHandler);
            button.addEventListener('mouseup', endHandler);
            button.addEventListener('mouseleave', endHandler);
            button.addEventListener('touchstart', (e) => { e.preventDefault(); startHandler(); }, { passive: false });
            button.addEventListener('touchend', endHandler);
            button.addEventListener('touchcancel', endHandler);
        });

        function setMode(mode) {
            currentMode = (currentMode === mode) ? 'none' : mode;
            document.querySelectorAll('.mode-btn').forEach(btn => btn.classList.remove('active'));
            if (currentMode !== 'none') {
                 if(mode === 'water') toggleWaterBtn.classList.add('active');
                 if(mode === 'delete') toggleDeleteModeBtn.classList.add('active');
                 if(mode === 'lilypad') toggleLilyPadModeBtn.classList.add('active');
                canvas.style.cursor = 'crosshair';
            } else {
                canvas.style.cursor = 'default';
            }
            toggleWaterBtn.textContent = currentMode === 'water' ? 'Place Water: ON' : 'Place Water';
            toggleDeleteModeBtn.textContent = currentMode === 'delete' ? 'Delete Blocks: ON' : 'Delete Blocks';
            toggleLilyPadModeBtn.textContent = currentMode === 'lilypad' ? 'Place Lily Pad: ON' : 'Place Lily Pad';
        }

        toggleWaterBtn.addEventListener('click', () => setMode('water'));
        toggleDeleteModeBtn.addEventListener('click', () => setMode('delete'));
        toggleLilyPadModeBtn.addEventListener('click', () => setMode('lilypad'));

        canvas.addEventListener('click', (event) => {
            if (!gameRunning || currentMode === 'none') return;
            const rect = canvas.getBoundingClientRect();
            const x = event.clientX - rect.left;
            const y = event.clientY - rect.top;
            
            const tileX = Math.floor(x / TILE_SIZE) * TILE_SIZE;
            const tileY = Math.floor(y / TILE_SIZE) * TILE_SIZE;

            const inBounds = (tileX > TUBE_PADDING && tileX < canvas.width - TUBE_PADDING - TILE_SIZE &&
                              tileY > TUBE_PADDING && tileY < canvas.height - TUBE_PADDING - TILE_SIZE);

            if (currentMode === 'water' && inBounds) {
                const exists = waterBlocks.some(block => block.x === tileX && block.y === tileY);
                if (!exists) { waterBlocks.push({ x: tileX, y: tileY }); }
            } else if (currentMode === 'delete') {
                const waterIndex = waterBlocks.findIndex(block => block.x === tileX && block.y === tileY);
                if (waterIndex > -1) { waterBlocks.splice(waterIndex, 1); }
                const lilyIndex = lilyPads.findIndex(pad => pad.x === tileX && pad.y === tileY);
                if (lilyIndex > -1) { lilyPads.splice(lilyIndex, 1); }
            } else if (currentMode === 'lilypad' && inBounds) {
                const hasWater = waterBlocks.some(block => block.x === tileX && block.y === tileY);
                const hasLily = lilyPads.some(pad => pad.x === tileX && pad.y === tileY);
                if (hasWater && !hasLily) { lilyPads.push({ x: tileX, y: tileY }); }
            }
        });

        // Helper functions
        function getRandomColor() { const letters = '0123456789ABCDEF'; let color = '#'; for (let i = 0; i < 6; i++) { color += letters[Math.floor(Math.random() * 16)]; } return color; }
        const randomNames = ['Spot', 'Slimy', 'Wiggles', 'TubeDude', 'Glimmer', 'Rex', 'Larry', 'Sally', 'Sammy'];
        function getRandomName() { return randomNames[Math.floor(Math.random() * randomNames.length)]; }
        function isInWater(x, y) { for (const block of waterBlocks) { if (x >= block.x && x < block.x + TILE_SIZE && y >= block.y && y < block.y + TILE_SIZE) { return true; } } return false; }

        // Drawing function for a crown (now takes a scale and an optional array of colors for custom patterns)
        function drawCrown(ctxObj, scale, colors = ['#FFD700']) {
            ctxObj.save();
            const crownXBase = 15 * scale;
            const crownYBase = -15 * scale; 
            
            // Draw main crown base shape
            ctxObj.beginPath();
            ctxObj.moveTo(crownXBase - 3 * scale, crownYBase); 
            ctxObj.lineTo(crownXBase, crownYBase - 7 * scale); 
            ctxObj.lineTo(crownXBase + 3 * scale, crownYBase); 
            ctxObj.lineTo(crownXBase + 6 * scale, crownYBase - 7 * scale); 
            ctxObj.lineTo(crownXBase + 9 * scale, crownYBase); 
            ctxObj.lineTo(crownXBase - 3 * scale, crownYBase); 
            ctxObj.closePath();
            
            // Fill with the first color (or default)
            ctxObj.fillStyle = colors[0];
            ctxObj.fill();

            // If we have a second color (e.g., for Toad's dots)
            if (colors.length > 1) {
                ctxObj.fillStyle = colors[1];
                // Draw dots on the crown peaks
                ctxObj.beginPath();
                ctxObj.arc(crownXBase, crownYBase - 7 * scale, 1.5 * scale, 0, Math.PI * 2);
                ctxObj.arc(crownXBase + 6 * scale, crownYBase - 7 * scale, 1.5 * scale, 0, Math.PI * 2);
                ctxObj.fill();
            }

            ctxObj.restore();
        }

        // Salamander Class Definition (Side profile view)
        class Salamander {
            constructor(x, y, name, isPlayer = false, color = '#808080', crownColors = ['#FFD700'], crownScale = 1) {
                this.x = x; this.y = y; this.name = name; this.isPlayer = isPlayer;
                this.baseColor = color;
                this.crownColors = crownColors; // Store crown colors
                this.crownScale = crownScale; // Store crown scale multiplier
                this.color = this.baseColor;
                this.speed = isPlayer ? 3 : 2; this.rotationAngle = 0; 
                this.vx = 0; this.vy = 0;
                this.lastRandomDirChange = Date.now(); this.randomDirInterval = Math.random() * 2000 + 1000;
                this.isStopped = false; this.stopDuration = 0; this.isSwimming = false;
            }
            update() {
                this.isSwimming = isInWater(this.x, this.y); this.color = this.isSwimming ? '#4a7c59' : this.baseColor; 
                if (this.isPlayer) {
                    this.vx = 0; this.vy = 0;
                    if (keys['ArrowUp'] || keys['up']) this.vy -= 1;
                    if (keys['ArrowDown'] || keys['down']) this.vy += 1;
                    if (keys['ArrowLeft'] || keys['left']) this.vx -= 1;
                    if (keys['ArrowRight'] || keys['right']) this.vx += 1;
                    this.speed = this.isSwimming ? 1.5 : 3;
                } else {
                    if (this.isStopped) { if (Date.now() - this.stopStart > this.stopDuration) { this.isStopped = false; this.lastRandomDirChange = Date.now(); } } 
                    else { if (Date.now() - this.lastRandomDirChange > this.randomDirInterval) { if (Math.random() < 0.3) { this.isStopped = true; this.stopStart = Date.now(); this.stopDuration = Math.random() * 1500 + 500; this.vx = 0; this.vy = 0; } else { this.vx = (Math.random() - 0.5) * 2; this.vy = (Math.random() - 0.5) * 2; } this.lastRandomDirChange = Date.now(); this.randomDirInterval = Math.random() * 2000 + 1000; } 
                        if (this.x < TUBE_PADDING + AVOIDANCE_THRESHOLD) this.vx = 1; if (this.x > canvas.width - TUBE_PADDING - AVOIDANCE_THRESHOLD) this.vx = -1;
                        if (this.y < TUBE_PADDING + AVOIDANCE_THRESHOLD) this.vy = 1; if (this.y > canvas.height - TUBE_PADDING - AVOIDANCE_THRESHOLD) this.vy = -1;
                    }
                    this.speed = this.isSwimming ? 1 : 2;
                }
                if ((this.vx !== 0 || this.vy !== 0) && !this.isStopped) { this.rotationAngle = Math.atan2(this.vy, this.vx); }
                this.x += this.vx * this.speed; this.y += this.vy * this.speed;
                this.x = Math.max(TUBE_PADDING, Math.min(this.x, canvas.width - TUBE_PADDING)); this.y = Math.max(TUBE_PADDING, Math.min(this.y, canvas.height - TUBE_PADDING));
            }
            draw() {
                ctx.save(); ctx.translate(this.x, this.y); ctx.rotate(this.rotationAngle); const scale = SALAMANDER_SCALE; 
                ctx.fillStyle = this.color;
                ctx.beginPath(); ctx.ellipse(-20 * scale, 0, 25 * scale, 6 * scale, 0, 0, Math.PI * 2); ctx.fill();
                ctx.beginPath(); ctx.ellipse(0, 0, 15 * scale, 8 * scale, 0, 0, Math.PI * 2); ctx.fill();
                ctx.beginPath(); ctx.ellipse(15 * scale, 0, 8 * scale, 7 * scale, 0, 0, Math.PI * 2); ctx.fill();
                ctx.fillStyle = 'black';
                ctx.beginPath(); ctx.arc(18 * scale, -4 * scale, 2 * scale, 0, Math.PI * 2); ctx.arc(18 * scale, 4 * scale, 2 * scale, 0, Math.PI * 2); ctx.fill();
                ctx.strokeStyle = this.color; ctx.lineWidth = 2 * scale; ctx.beginPath();
                ctx.moveTo(0, 8 * scale); ctx.lineTo(5 * scale, 15 * scale); ctx.moveTo(5 * scale, 15 * scale); ctx.lineTo(6 * scale, 17 * scale); ctx.moveTo(5 * scale, 15 * scale); ctx.lineTo(4 * scale, 17 * scale);
                ctx.moveTo(0, -8 * scale); ctx.lineTo(5 * scale, -15 * scale); ctx.moveTo(5 * scale, -15 * scale); ctx.lineTo(6 * scale, -17 * scale); ctx.moveTo(5 * scale, -15 * scale); ctx.lineTo(4 * scale, -17 * scale);
                ctx.moveTo(-20 * scale, 8 * scale); ctx.lineTo(-25 * scale, 15 * scale); ctx.moveTo(-25 * scale, 15 * scale); ctx.lineTo(-26 * scale, 17 * scale); ctx.moveTo(-25 * scale, 15 * scale); ctx.lineTo(-24 * scale, 17 * scale);
                ctx.moveTo(-20 * scale, -8 * scale); ctx.lineTo(-25 * scale, -15 * scale); ctx.moveTo(-25 * scale, -15 * scale); ctx.lineTo(-26 * scale, -17 * scale); ctx.moveTo(-25 * scale, -15 * scale); ctx.lineTo(-24 * scale, -17 * scale);
                ctx.stroke();
                if (this.isPlayer) {
                    // Pass the scale multiplier to drawCrown
                    drawCrown(ctx, scale * this.crownScale, this.crownColors);
                }
                ctx.restore();
                ctx.fillStyle = 'black'; ctx.textAlign = 'center'; ctx.font = '12px sans-serif';
                ctx.fillText(this.name, this.x, this.y - 30 * scale * this.crownScale); // Adjust name height dynamically
            }
        }

        // Game Initialization & Spawning
        startButton.addEventListener('click', () => {
            const inputName = usernameInput.value.trim();
            if (inputName) {
                gameRunning = true;
                gameSetupScreen.style.display = 'none';
                titleScreenContainer.style.display = 'none'; 
                canvas.style.display = 'block';
                controlsContainer.style.display = 'block';
                
                // Determine color and crown color/scale based on name
                let color = '#808080';
                let crownColors = ['#FFD700'];
                let crownScale = 1;

                if (inputName.toLowerCase() === 'midas') {
                    color = '#FFD700'; // Yellow/Gold body
                } else if (inputName.toLowerCase() === 'poseidon') {
                    color = '#0000FF'; // Blue body
                    crownColors = ['#008000']; // Green crown
                } else if (inputName.toLowerCase() === 'technoblade') {
                    color = '#FFC0CB'; // Pink body
                    crownScale = 1.5; // Bigger crown
                } else if (inputName.toLowerCase() === 'toad') {
                    color = '#F5F5DC'; // Beige body
                    crownColors = ['#FF0000', '#FFFFFF']; // Red with white dots
                    crownScale = 1.1; // Slightly larger crown for mushroom top feel
                }
                
                salamanders.push(new Salamander(canvas.width / 2, canvas.height / 2, inputName, true, color, crownColors, crownScale));
                gameLoop();
            }
        });

        spawnBtn.addEventListener('click', () => {
            const randomX = Math.random() * (canvas.width - TUBE_PADDING * 2) + TUBE_PADDING;
            const randomY = Math.random() * (canvas.height - TUBE_PADDING * 2) + TUBE_PADDING;
            const name = spawnNameInput.value.trim() || getRandomName();
            // Spawn NPCs with default colors/crowns
            salamanders.push(new Salamander(randomX, randomY, name, false, getRandomColor())); 
            spawnNameInput.value = ''; 
        });

        function updateGame() {
            if (!gameRunning) return;
            salamanders.forEach(s => s.update());
        }

        function drawGame() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.strokeStyle = '#555'; ctx.lineWidth = 40; ctx.strokeRect(50, 50, canvas.width - 100, canvas.height - 100);
            for (const block of waterBlocks) { ctx.fillStyle = 'rgba(0, 100, 255, 0.5)'; ctx.fillRect(block.x, block.y, TILE_SIZE, TILE_SIZE); }
            for (const pad of lilyPads) {
                ctx.fillStyle = '#4CAF50'; ctx.beginPath(); ctx.arc(pad.x + TILE_SIZE / 2, pad.y + TILE_SIZE / 2, TILE_SIZE / 2.2, 0, Math.PI * 2); ctx.fill();
                ctx.fillStyle = '#FFC0CB'; ctx.beginPath(); ctx.arc(pad.x + TILE_SIZE / 2, pad.y + TILE_SIZE / 2, TILE_SIZE / 8, 0, Math.PI * 2); ctx.fill();
            }
            salamanders.forEach(s => s.draw());
        }

        // Function to draw the large salamander on the title screen canvas (FRONT View)
        function drawTitleSalamanderFrontView() {
            const scale = 2; 
            const x = titleCanvas.width / 2;
            const y = titleCanvas.height / 2 + 10;
            const color = '#FFC0CB'; // Pink body for the default title character (Technoblade feel)
            
            titleCtx.save(); titleCtx.translate(x, y); 
            titleCtx.fillStyle = color;
            titleCtx.beginPath(); titleCtx.ellipse(-20 * scale, 0, 25 * scale, 6 * scale, 0, 0, Math.PI * 2); titleCtx.fill();
            titleCtx.beginPath(); titleCtx.ellipse(0, 0, 15 * scale, 8 * scale, 0, 0, Math.PI * 2); titleCtx.fill();
            titleCtx.beginPath(); titleCtx.ellipse(15 * scale, 0, 8 * scale, 7 * scale, 0, 0, Math.PI * 2); titleCtx.fill();
            titleCtx.fillStyle = 'black';
            titleCtx.beginPath(); titleCtx.arc(18 * scale, -4 * scale, 2 * scale, 0, Math.PI * 2); titleCtx.arc(18 * scale, 4 * scale, 2 * scale, 0, Math.PI * 2); titleCtx.fill();
            titleCtx.strokeStyle = color; titleCtx.lineWidth = 2 * scale; titleCtx.beginPath();
            titleCtx.moveTo(0, 8 * scale); titleCtx.lineTo(5 * scale, 15 * scale); titleCtx.moveTo(5 * scale, 15 * scale); titleCtx.lineTo(6 * scale, 17 * scale); titleCtx.moveTo(5 * scale, 15 * scale); titleCtx.lineTo(4 * scale, 17 * scale);
            titleCtx.moveTo(0, -8 * scale); titleCtx.lineTo(5 * scale, -15 * scale); titleCtx.moveTo(5 * scale, -15 * scale); titleCtx.lineTo(6 * scale, -17 * scale); titleCtx.moveTo(5 * scale, -15 * scale); titleCtx.lineTo(4 * scale, -17 * scale);
            titleCtx.moveTo(-20 * scale, 8 * scale); titleCtx.lineTo(-25 * scale, 15 * scale); titleCtx.moveTo(-25 * scale, 15 * scale); titleCtx.lineTo(-26 * scale, 17 * scale); titleCtx.moveTo(-25 * scale, 15 * scale); titleCtx.lineTo(-24 * scale, 17 * scale);
            titleCtx.moveTo(-20 * scale, -8 * scale); titleCtx.lineTo(-25 * scale, -15 * scale); titleCtx.moveTo(-25 * scale, -15 * scale); titleCtx.lineTo(-26 * scale, -17 * scale); titleCtx.moveTo(-25 * scale, -15 * scale); titleCtx.lineTo(-24 * scale, -17 * scale);
            titleCtx.stroke();
            
            // Draw default crown (Technoblade style for intro)
            drawCrown(titleCtx, scale * 1.5, ['#FFD700']); 
            titleCtx.restore();
        }

        // Draw the title salamander immediately when the script runs
        drawTitleSalamanderFrontView();

        // Game loop
        function gameLoop() {
            updateGame();
            drawGame();
            requestAnimationFrame(gameLoop);
        }
    </script>
</body>
</html>
