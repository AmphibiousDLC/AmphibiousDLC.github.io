<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Fixed Interactive Physics Ball Game</title>
    <style>
        /* Ensures the body takes up the full mobile viewport height */
        html, body {
            height: 100%;
            margin: 0;
            padding: 0;
        }
        body {
            font-family: sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 10px; 
            background-color: #f0f0f0;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
            touch-action: none; 
        }
        #controls {
            margin-bottom: 15px; 
            text-align: center;
            width: 100%;
            display: flex;
            justify-content: center;
        }
        input[type="text"] {
            padding: 10px;
            font-size: 16px;
            border: 2px solid #ccc;
            border-radius: 5px;
            flex-grow: 1; 
            max-width: 200px;
        }
        button {
            padding: 10px 15px;
            font-size: 16px;
            cursor: pointer;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 5px;
            margin-left: 5px;
        }
        #game-area {
            width: 100%;
            flex-grow: 1; 
            background-color: #fff;
            border: 2px solid #333;
            position: relative;
            overflow: hidden;
            cursor: crosshair;
            box-sizing: border-box; 
            touch-action: none;
        }
        .ball {
            position: absolute;
            width: 60px;
            height: 60px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 12px;
            font-weight: bold;
            text-align: center;
            box-shadow: 0 4px 8px rgba(0,0,0,0.4);
            transform: translate(0px, -60px); 
            cursor: pointer; 
            transition: transform 0.01s linear; 
            touch-action: none; 
        }
    </style>
</head>
<body>

    <div id="controls">
        <input type="text" id="nameInput" placeholder="Enter name">
        <button onclick="addBallFromInput()">Drop</button>
    </div>

    <div id="game-area">
        <!-- Balls will be injected here by JavaScript -->
    </div>

    <script>
        const nameInput = document.getElementById('nameInput');
        const gameArea = document.getElementById('game-area');
        const BALL_SIZE = 60;
        const GRAVITY = 0.5;
        const DAMPING = 0.7; 
        const FRICTION = 0.95;
        const THROW_FORCE_MULTIPLIER = 0.5;
        let balls = []; 
        let draggedBall = null;
        let interactionPos = { x: 0, y: 0 };
        let lastInteractionPos = { x: 0, y: 0 };

        function getRandomColor() {
            const letters = '0123456789ABCDEF';
            let color = '#';
            for (let i = 0; i < 6; i++) {
                color += letters[Math.floor(Math.random() * 16)];
            }
            return color;
        }

        function createBall(name) {
            const ballElement = document.createElement('div');
            ballElement.classList.add('ball');
            ballElement.textContent = name;
            ballElement.style.backgroundColor = getRandomColor();
            
            ballElement.addEventListener('mousedown', onBallInteractionStart);
            ballElement.addEventListener('touchstart', onBallInteractionStart);

            const maxLeft = gameArea.clientWidth - BALL_SIZE;
            const startX = Math.floor(Math.random() * maxLeft);

            const ballPhysics = {
                element: ballElement,
                x: startX,
                y: -BALL_SIZE,
                vx: (Math.random() - 0.5) * 5,
                vy: 0,
                isDragging: false
            };

            gameArea.appendChild(ballElement);
            balls.push(ballPhysics);
        }

        function addBallFromInput() {
            const name = nameInput.value.trim();
            if (name === '') {
                nameInput.focus(); 
                return;
            }
            createBall(name);
            nameInput.value = '';
            nameInput.focus();
        }

        // --- Interaction Handlers (Unified for Mouse and Touch) ---

        function getCoordsInGameArea(e) {
            const clientX = e.touches ? e.touches.clientX : e.clientX;
            const clientY = e.touches ? e.touches.clientY : e.clientY;
            
            const rect = gameArea.getBoundingClientRect();
            return {
                x: clientX - rect.left,
                y: clientY - rect.top
            };
        }

        function onBallInteractionStart(e) {
            draggedBall = balls.find(b => b.element === e.target);
            if (draggedBall) {
                draggedBall.isDragging = true;
                draggedBall.vy = 0; 
                draggedBall.vx = 0;
                lastInteractionPos = getCoordsInGameArea(e);

                document.addEventListener('mousemove', onInteractionMove);
                document.addEventListener('mouseup', onInteractionEnd);
                document.addEventListener('touchmove', onInteractionMove, { passive: false });
                document.addEventListener('touchend', onInteractionEnd);
                
                e.preventDefault(); 
            }
        }

        function onInteractionMove(e) {
            if (draggedBall) {
                interactionPos = getCoordsInGameArea(e);
                
                const dx = interactionPos.x - lastInteractionPos.x;
                const dy = interactionPos.y - lastInteractionPos.y;

                draggedBall.x += dx;
                draggedBall.y += dy;

                draggedBall.x = Math.max(0, Math.min(gameArea.clientWidth - BALL_SIZE, draggedBall.x));
                draggedBall.y = Math.max(0, Math.min(gameArea.clientHeight - BALL_SIZE, draggedBall.y));
                
                draggedBall.vx = dx * THROW_FORCE_MULTIPLIER;
                draggedBall.vy = dy * THROW_FORCE_MULTIPLIER;
                
                lastInteractionPos = interactionPos;
                
                e.preventDefault();
            }
        }

        function onInteractionEnd(e) {
            if (draggedBall) {
                draggedBall.isDragging = false;

                draggedBall = null;
                document.removeEventListener('mousemove', onInteractionMove);
                document.removeEventListener('mouseup', onInteractionEnd);
                document.removeEventListener('touchmove', onInteractionMove);
                document.removeEventListener('touchend', onInteractionEnd);
            }
        }


        // --- Physics Update Loop (FIXED CALCULATION) ---
        function updatePhysics() {
            const gameAreaHeight = gameArea.clientHeight;
            const gameAreaWidth = gameArea.clientWidth;

            for (const ball of balls) {

                if (!ball.isDragging) {
                    ball.vy += GRAVITY;
                    // CORRECT CALCULATION:
                    ball.y += ball.vy; 
                    ball.x += ball.vx;

                    // Wall collision (left/right bounds)
                    if (ball.x + BALL_SIZE > gameAreaWidth) {
                        ball.x = gameAreaWidth - BALL_SIZE;
                        ball.vx *= -DAMPING;
                    } else if (ball.x < 0) {
                        ball.x = 0;
                        ball.vx *= -DAMPING;
                    }

                    // Floor collision (bottom bounds)
                    if (ball.y + BALL_SIZE > gameAreaHeight) {
                        ball.y = gameAreaHeight - BALL_SIZE;
                        ball.vy *= -DAMPING; 
                        
                        if (Math.abs(ball.vy) < 1 && Math.abs(ball.vx) < 0.1) {
                            ball.vy = 0;
                            ball.vx = 0;
                        } else if (ball.vy === 0) {
                            ball.vx *= FRICTION;
                        }
                    }
                }

                // Update the DOM element position
                ball.element.style.transform = `translate(${ball.x}px, ${ball.y}px)`;
            }

            requestAnimationFrame(updatePhysics);
        }

        // Start the physics loop
        updatePhysics();
    </script>

</body>
</html>
