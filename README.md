<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ChronoBoulder - The Rock Game</title>
    
    <!-- Combined CSS Styles -->
    <style>
        body {
            margin: 0;
            padding: 0;
            font-family: sans-serif;
            display: flex;
            flex-direction: column;
            height: 100vh;
            background-color: #333;
        }

        /* --- Timer Styles --- */
        #timer-container {
            position: absolute;
            top: 20px;
            width: 100%;
            text-align: center;
            z-index: 10;
        }

        #timer-display {
            font-size: 48px;
            font-weight: bold;
            color: #fff;
            background-color: rgba(0, 0, 0, 0.6);
            padding: 10px 20px;
            border-radius: 10px;
        }

        /* --- Game Field & Rock Styles --- */
        #game-field {
            flex-grow: 1;
            /* Placeholder image URL for grass field */
            background-image: url('i.imgur.com'); 
            background-size: cover; 
            background-position: center;
            display: flex;
            justify-content: center;
            align-items: center;
            position: relative;
            overflow: hidden;
        }

        #player-rock {
            width: 150px;
            height: 150px;
            background-color: #607D8B; 
            border: 5px solid #455A64;
            clip-path: polygon(
                50% 0%, 
                100% 30%, 
                90% 90%, 
                60% 100%, 
                0% 80%, 
                10% 20%
            );
            transform: rotate(15deg); 
            box-shadow: 
                10px 10px 40px rgba(0, 0, 0, 0.5),
                -5px -5px 20px rgba(255, 255, 255, 0.1) inset;
            
            position: relative; 
            display: flex;
            justify-content: center;
            align-items: center;
        }

        /* --- The Badly Drawn Smiley Face Container (Dark Grey) --- */
        .rock-face {
            position: absolute;
            top: 30px; 
            left: 30px;
            width: 90px;
            height: 90px;
            transform: rotate(-10deg); 
            color: #333; /* Dark Grey Color for the face */
            /* Using CSS gradients/shadows to draw eyes and mouth imperfectly */
            background-image: 
                radial-gradient(circle at 30% 30%, #333 8px, transparent 8px), /* Left Eye */
                radial-gradient(circle at 70% 35%, #333 6px, transparent 6px); /* Right Eye */
        }
        
        .rock-face::after {
            content: '';
            position: absolute;
            bottom: 20px;
            left: 15px;
            width: 60px;
            height: 20px;
            border-bottom: 5px solid #333; /* Dark Grey Color for the smile */
            border-radius: 50% 50% 100% 100%; 
            transform: rotate(5deg); 
        }


        /* --- Achievement Tab Styles --- */
        #achievement-tab {
            position: fixed;
            right: -240px;
            top: 50%;
            transform: translateY(-50%);
            width: 250px;
            background-color: #f1f1f1;
            border: 1px solid #ccc;
            border-right: none;
            padding: 15px;
            transition: right 0.3s;
            cursor: pointer;
            z-index: 20;
            border-top-left-radius: 8px;
            border-bottom-left-radius: 8px;
        }

        #achievement-tab.open {
            right: 0;
        }

        #achievement-tab h3 {
            margin-top: 0;
            font-size: 16px;
        }

        .achievement-item {
            padding: 10px;
            border: 1px solid #ddd;
            margin-bottom: 5px;
            background-color: #eee;
            font-size: 14px;
            opacity: 0.5;
            transition: opacity 0.5s;
        }

        .achievement-item.unlocked {
            opacity: 1;
            background-color: #d4edda;
            border-color: #c3e6cb;
            color: #155724;
            font-weight: bold;
        }

        #tab-toggle {
            position: absolute;
            left: -40px;
            top: 0;
            background-color: #f1f1f1;
            border: 1px solid #ccc;
            border-right: none;
            padding: 5px 10px;
            cursor: pointer;
            border-top-left-radius: 8px;
            border-bottom-left-radius: 8px;
            font-size: 12px;
        }
    </style>
</head>
<body>

    <div id="timer-container">
        <span id="timer-display">00:00</span>
    </div>

    <div id="game-field">
        <div id="player-rock">
            <div class="rock-face"></div>
        </div>
    </div>

    <div id="achievement-tab">
        <div id="tab-toggle" onclick="toggleAchievements()">🏆</div>
        <h3>Achievements</h3>
        <div class="achievement-item" id="achievement-dino">
            🦕ANCIENT🦖: Endured for 10 minutes.
        </div>
    </div>

    <!-- Combined JavaScript Logic -->
    <script>
        const timerDisplay = document.getElementById('timer-display');
        const achievementTab = document.getElementById('achievement-tab');
        const achievementDino = document.getElementById('achievement-dino');
        const SECONDS_TO_UNLOCK = 10 * 60; // 10 minutes

        let timeElapsed = 0;
        let gameInterval;
        let isDinoUnlocked = false;

        function updateTimer() {
            timeElapsed += 1;
            const minutes = Math.floor(timeElapsed / 60);
            const seconds = timeElapsed % 60;
            const formattedMinutes = minutes.toString().padStart(2, '0');
            const formattedSeconds = seconds.toString().padStart(2, '0');
            timerDisplay.textContent = `${formattedMinutes}:${formattedSeconds}`;

            if (!isDinoUnlocked && timeElapsed >= SECONDS_TO_UNLOCK) {
                unlockDinoAchievement();
            }
        }

        function unlockDinoAchievement() {
            isDinoUnlocked = true;
            achievementDino.classList.add('unlocked');
            achievementTab.classList.add('open'); 
            console.log("ACHIEVEMENT UNLOCKED: 🦕ANCIENT🦖");
        }

        function toggleAchievements() {
            achievementTab.classList.toggle('open');
        }

        window.onload = () => {
            gameInterval = setInterval(updateTimer, 1000);
            console.log("ChronoBoulder game started. The rock endures.");
        };

    </script>

</body>
</html>

