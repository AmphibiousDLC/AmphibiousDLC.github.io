<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cryptographic</title>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #000;
            color: yellow;
            font-family: monospace, sans-serif;
            flex-direction: column;
        }
        #game-container {
            display: flex;
            gap: 20px;
            width: 850px;
        }
        canvas {
            background-color: #000;
            border: 2px solid yellow;
            display: none;
        }
        #sidebar {
            width: 200px;
            padding: 15px;
            background-color: #111;
            border-radius: 8px;
            border: 1px solid yellow;
        }
        .tabs {
            list-style-type: none;
            margin: 0 0 10px 0;
            padding: 0;
            display: flex;
            border-bottom: 2px solid yellow;
        }
        .tab-button {
            padding: 10px 20px;
            cursor: pointer;
            background-color: #333;
            border: 1px solid yellow;
            border-bottom: none;
            color: yellow;
            font-family: monospace, sans-serif;
        }
        .tab-button.active {
            background-color: #000;
            color: white;
        }
        .tab-content {
            display: none;
            padding: 10px;
            border: 1px solid yellow;
            width: 600px;
            box-sizing: border-box;
        }
        .tab-content.active {
            display: block;
        }
        .shop-item {
            padding: 10px 0;
            border-bottom: 1px solid #333;
        }
        button {
            padding: 10px;
            width: 100%;
            margin-top: 10px;
            cursor: pointer;
            background-color: #333;
            color: yellow;
            border: 1px solid yellow;
            font-weight: bold;
            font-family: monospace, sans-serif;
            margin-bottom: 5px;
        }
        button:disabled {
            background-color: #000;
            color: #555;
            cursor: not-allowed;
            border-color: #555;
        }
        .stats p {
            margin: 5px 0;
            display: flex;
            align-items: center;
        }
        .icon {
            margin-right: 8px;
        }
    </style>
</head>
<body>

    <ul class="tabs">
        <li class="tab-button active" onclick="openTab(event, 'MiningArea')">Mining Area</li>
        <li class="tab-button" onclick="openTab(event, 'ShopArea')">Market</li>
    </ul>

    <div id="game-container">
        <div id="sidebar">
            <h2>Cryptographic</h2>
            <div class="stats">
                <p><canvas id="icon-tokedillion" class="icon" width="16" height="16"></canvas> Tokedillions: <span id="goldCount">0</span></p>
                <p><canvas id="icon-merchant" class="icon" width="16" height="16"></canvas> Merchants: <span id="merchantCount">0</span>/100</p>
                <p><canvas id="icon-accountant" class="icon" width="16" height="16"></canvas> Accountants: <span id="accountantCount">0</span>/5</p>
                <p><canvas id="icon-ore" class="icon" width="16" height="16"></canvas> Ores: <span id="oreCount">0</span>/50</p>
                <p>Exchanges: <span id="exchangeCount">1</span>/8</p>
            </div>
        </div>

        <div id="MiningArea" class="tab-content active">
            <canvas id="gameCanvas" width="600" height="400"></canvas>
        </div>

        <div id="ShopArea" class="tab-content">
            <h3>Hire/Upgrades</h3>
            <div class="shop-item">
                <p>Hire a Merchant to mine for 10 Tokedillions. (Max 100)</p>
                <button id="spawnMerchantButton" onclick="spawnMerchant()">Hire Merchant (10 Tokedillions)</button>
            </div>
            <div class="shop-item">
                <p>Hire an Accountant. Each doubles your ore income. (Max 5)</p>
                <button id="spawnAccountantButton" onclick="spawnAccountant()">Hire Accountant (300 Tokedillions)</button>
            </div>
            <div class="shop-item">
                <p>Build an additional Exchange Booth. (Max 8)</p>
                <button id="spawnExchangeButton" onclick="spawnExchange()">Build Exchange (1000 Tokedillions)</button>
            </div>
            <div class="shop-item">
                <p>Upgrade Merchant Pickaxes: Level <span id="pickaxeLevel">1</span></p>
                <p>Cost: <span id="pickaxeUpgradeCost">?</span> Tokedillions. Effect: +1 Mine Speed.</p>
                <button id="upgradePickaxeButton" onclick="upgradePickaxe()">Upgrade Pickaxe</button>
            </div>
            <div class="shop-item">
                <p>Deploy an Excavator. Instantly mines ores.</p>
                <button id="spawnExcavatorButton" onclick="spawnExcavator()">Deploy Excavator (20000 Tokedillions)</button>
            </div>
        </div>
    </div>

    <script>
        // --- Tab System Logic ---
        function openTab(evt, tabName) {
            var i, tabcontent, tablinks;
            tabcontent = document.getElementsByClassName("tab-content");
            for (i = 0; i < tabcontent.length; i++) {
                tabcontent[i].style.display = "none";
            }
            tablinks = document.getElementsByClassName("tab-button");
            for (i = 0; i < tablinks.length; i++) {
                tablinks[i].classList.remove("active");
            }
            document.getElementById(tabName).style.display = "block";
            evt.currentTarget.classList.add("active");

            if (tabName === 'MiningArea') {
                document.getElementById('gameCanvas').style.display = 'block';
            } else {
                document.getElementById('gameCanvas').style.display = 'none';
            }
        }

        // --- Game Variables and Setup ---
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const goldCountElement = document.getElementById('goldCount');
        const merchantCountElement = document.getElementById('merchantCount');
        const accountantCountElement = document.getElementById('accountantCount');
        const oreCountElement = document.getElementById('oreCount');
        const maxOreCountElement = document.getElementById('maxOreCount');
        const exchangeCountElement = document.getElementById('exchangeCount');
        const pickaxeLevelElement = document.getElementById('pickaxeLevel');
        const pickaxeUpgradeCostElement = document.getElementById('pickaxeUpgradeCost');
        const spawnMerchantButton = document.getElementById('spawnMerchantButton');
        const spawnAccountantButton = document.getElementById('spawnAccountantButton');
        const upgradePickaxeButton = document.getElementById('upgradePickaxeButton');
        const spawnExchangeButton = document.getElementById('spawnExchangeButton');
        const spawnExcavatorButton = document.getElementById('spawnExcavatorButton');

        let gold = 50;
        let units = []; // Combined array for merchants and excavators
        let ores = [];
        let bases = [];
        let accountants = [];

        const merchantCost = 10;
        const accountantCost = 300;
        const exchangeCost = 1000;
        const excavatorCost = 20000;
        
        const FILL_COLOR = '#000000';
        const STROKE_COLOR = '#FFFF00';
        const MAX_ORES = 50; 
        const MAX_MERCHANTS_PER_ORE = 3;
        const MAX_ACCOUNTANTS = 5;
        const MAX_BASES = 8;
        const MAX_MERCHANTS = 100;
        const BASE_ORE_VALUE = 15;
        let pickaxeLevel = 1;
        let pickaxeUpgradeBaseCost = 50;


        function distanceBetween(obj1, obj2) {
            const dx = obj1.x - obj2.x;
            const dy = obj2.y - obj1.y;
            return Math.sqrt(dx * dx + dy * dy);
        }

        // --- Game Object Classes ---
        class HomeBase {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.width = 25;
                this.height = 40;
            }
            draw() {
                ctx.fillStyle = FILL_COLOR;
                ctx.strokeStyle = STROKE_COLOR;
                ctx.lineWidth = 1;
                ctx.fillRect(this.x - this.width / 2, this.y - this.height / 2, this.width, this.height);
                ctx.strokeRect(this.x - this.width / 2, this.y - this.height / 2, this.width, this.height);
                ctx.beginPath();
                ctx.moveTo(this.x - this.width / 2, this.y - this.height / 2);
                ctx.lineTo(this.x, this.y - this.height / 2 - 10);
                ctx.lineTo(this.x + this.width / 2, this.y - this.height / 2);
                ctx.closePath();
                ctx.fill();
                ctx.stroke();
                ctx.fillStyle = STROKE_COLOR;
                ctx.textAlign = 'center';
                ctx.font = '6px monospace';
                ctx.fillText("Exch", this.x, this.y + 3);
                ctx.font = '10px monospace';
            }
        }

        class Ore {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.radius = 5;
                this.health = 20;
            }
            draw() {
                ctx.beginPath();
                ctx.moveTo(this.x + Math.cos(0) * this.radius, this.y + Math.sin(0) * this.radius);
                for (let i = 1; i <= 6; i++) {
                    const angle = (i / 6) * Math.PI * 2;
                    const variedRadius = this.radius * (0.8 + Math.random() * 0.4);
                    ctx.lineTo(this.x + Math.cos(angle) * variedRadius, this.y + Math.sin(angle) * variedRadius);
                }
                ctx.closePath();
                ctx.fillStyle = FILL_COLOR;
                ctx.strokeStyle = STROKE_COLOR;
                ctx.fill();
                ctx.stroke();
                ctx.fillStyle = STROKE_COLOR;
                ctx.textAlign = 'center';
                ctx.font = '6px monospace';
                ctx.fillText(this.health, this.x, this.y + 2 + this.radius);
                ctx.font = '10px monospace';
            }
        }

        class Merchant {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.radius = 3;
                this.speed = 0.8; 
                this.target = null;
                this.carrying = false;
                this.mineSpeed = pickaxeLevel; 
                this.mineCooldown = 0;
                this.returnTarget = null;
                this.type = 'merchant';
            }
            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
                ctx.fillStyle = FILL_COLOR;
                ctx.strokeStyle = STROKE_COLOR;
                ctx.fill();
                ctx.stroke();
                ctx.closePath();
                ctx.strokeRect(this.x - 2, this.y - 4, 4, 3); 
                ctx.beginPath();
                ctx.moveTo(this.x + 2, this.y + 2);
                ctx.lineTo(this.x + 5, this.y - 1);
                ctx.strokeStyle = STROKE_COLOR;
                ctx.lineWidth = 1;
                ctx.stroke();
            }
            update() {
                this.mineSpeed = pickaxeLevel;
                
                if (!this.carrying) {
                    this.returnTarget = null; 
                    if (!this.target || this.target.health <= 0) {
                        this.findNearestOre();
                    }
                    if (this.target) {
                        if (distanceBetween(this, this.target) < this.target.radius + this.radius + 2) {
                            if (this.mineCooldown <= 0) {
                                this.target.health -= this.mineSpeed;
                                this.mineCooldown = 1000;
                                if (this.target.health <= 0) {
                                    this.carrying = true;
                                    this.target = null;
                                }
                            }
                        } else {
                            this.moveToTarget(this.target, true);
                        }
                    }
                } else {
                    if (!this.returnTarget) {
                        this.findNearestBase(); 
                    }

                    if (this.returnTarget) {
                         this.moveToTarget(this.returnTarget, false);
                        if (distanceBetween(this, this.returnTarget) < 10) {
                            const earnings = BASE_ORE_VALUE * Math.pow(2, accountants.length);
                            gold += earnings;
                            this.carrying = false; 
                            this.returnTarget = null;
                        }
                    }
                }

                if (this.mineCooldown > 0) {
                    this.mineCooldown -= 16;
                }
            }
            findNearestOre() {
               let closestOre = null;
                let minDistance = Infinity;
                // Use units.filter now to count merchants
                const activeMerchants = units.filter(u => u.type === 'merchant' && !u.carrying); 

                for (const ore of ores) {
                    const minersOnOre = activeMerchants.filter(m => m.target === ore).length;
                    if (ore.health > 0 && minersOnOre < MAX_MERCHANTS_PER_ORE) {
                        const distance = distanceBetween(this, ore);
                        if (distance < minDistance) {
                            minDistance = distance;
                            closestOre = ore;
                        }
                    }
                }
                this.target = closestOre;
            }
            findNearestBase() {
                let closestBase = null;
                let minDistance = Infinity;
                for (const base of bases) {
                    const distance = distanceBetween(this, base);
                    if (distance < minDistance) {
                        minDistance = distance;
                        closestBase = base;
                    }
                }
                this.returnTarget = closestBase;
            }
            moveToTarget(target, avoidObstacles = false) {
                let dx = target.x - this.x;
                let dy = target.y - this.y;
                let distance = Math.sqrt(dx * dx + dy * dy);
                if (distance > 0) {
                    dx /= distance;
                    dy /= distance;
                    if (avoidObstacles) {
                        for (const obstacle of ores) {
                            if (obstacle === target || obstacle === this) continue;
                            if (distanceBetween(this, obstacle) < obstacle.radius + this.radius + 2) {
                                const odx = this.x - obstacle.x;
                                const ody = this.y - obstacle.y;
                                dx += odx / distanceBetween(this, obstacle);
                                dy += ody / distanceBetween(this, obstacle);
                                const avoidanceLength = Math.sqrt(dx * dx + dy * dy);
                                dx /= avoidanceLength;
                                dy /= avoidanceLength;
                            }
                        }
                    }
                    this.x += dx * this.speed;
                    this.y += dy * this.speed;
                }
            }
        }

        class Accountant {
             // ... (Same as previous version) ...
            constructor(x, y) {
                this.x = x;
                this.y = y;
            }
            draw() {
                ctx.fillStyle = FILL_COLOR;
                ctx.strokeStyle = STROKE_COLOR;
                ctx.lineWidth = 1;
                ctx.strokeRect(this.x - 6, this.y, 12, 6); 
                ctx.beginPath();
                ctx.arc(this.x, this.y - 3, 3, 0, Math.PI * 2);
                ctx.stroke();
                ctx.beginPath();
                ctx.arc(this.x, this.y - 8, 1.5, 0, Math.PI * 2);
                ctx.stroke();
                ctx.fillStyle = STROKE_COLOR;
                ctx.textAlign = 'center';
                ctx.font = '6px monospace';
                ctx.fillText("ACC", this.x, this.y + 14);
                ctx.font = '10px monospace';
            }
            update() {}
        }
        
        class Excavator {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.width = 15;
                this.height = 10;
                this.speed = 1.2;
                this.target = null;
                this.type = 'excavator';
            }
            draw() {
                ctx.fillStyle = FILL_COLOR;
                ctx.strokeStyle = STROKE_COLOR;
                ctx.lineWidth = 1;
                // Main body (rectangle)
                ctx.strokeRect(this.x - this.width / 2, this.y - this.height / 2, this.width, this.height);
                ctx.fillRect(this.x - this.width / 2, this.y - this.height / 2, this.width, this.height);

                // Slanted triangle on front (bucket/blade)
                ctx.beginPath();
                ctx.moveTo(this.x + this.width / 2, this.y - this.height / 2);
                ctx.lineTo(this.x + this.width / 2 + 5, this.y); // Slanted point
                ctx.lineTo(this.x + this.width / 2, this.y + this.height / 2);
                ctx.closePath();
                ctx.fill();
                ctx.stroke();
            }
            update() {
                if (!this.target || this.target.health <= 0) {
                    this.findNearestOre();
                }
                
                if (this.target) {
                    // Excavator instantly mines on collision
                    if (distanceBetween(this, this.target) < this.width / 2 + this.target.radius + 5) {
                        // Instant mine
                        this.target.health = 0; 
                        const earnings = BASE_ORE_VALUE * Math.pow(2, accountants.length);
                        gold += earnings;
                        this.target = null; // Find new target next frame
                    } else {
                        this.moveToTarget(this.target);
                    }
                }
            }
            findNearestOre() {
                let closestOre = null;
                let minDistance = Infinity;
                for (const ore of ores) {
                    if (ore.health > 0) {
                        const distance = distanceBetween(this, ore);
                        if (distance < minDistance) {
                            minDistance = distance;
                            closestOre = ore;
                        }
                    }
                }
                this.target = closestOre;
            }
            moveToTarget(target) {
                // Excavators ignore obstacles (too powerful!)
                let dx = target.x - this.x;
                let dy = target.y - this.y;
                let distance = Math.sqrt(dx * dx + dy * dy);
                if (distance > 0) {
                    dx /= distance;
                    dy /= distance;
                    this.x += dx * this.speed;
                    this.y += dy * this.speed;
                }
            }
        }


        // --- Game Setup and Loop ---
        
        function initializeGameWorld() {
             // Start with one base in the center
             bases.push(new HomeBase(canvas.width / 2, canvas.height / 2));
             while (ores.length < MAX_ORES) {
                spawnNewOre();
            }
        }

        function spawnNewOre() {
            let x, y, validPlacement = false;
            const minDistance = 40; 
            while (ores.length < MAX_ORES && !validPlacement) {
                x = Math.random() * canvas.width;
                y = Math.random() * canvas.height;
                validPlacement = true;
                for(const base of bases) {
                    if (distanceBetween({x,y}, base) < minDistance) {
                        validPlacement = false;
                        break;
                    }
                }
            }
            if(validPlacement) {
                 ores.push(new Ore(x, y));
            }
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            bases.forEach(base => base.draw());
            ores.forEach(ore => ore.draw());
            // Draw all units
            units.forEach(unit => unit.draw());
            accountants.forEach(accountant => accountant.draw());
        }

        function update() {
            // Update all units
            units.forEach(unit => unit.update());
            accountants.forEach(accountant => accountant.update());

            const minedOresCount = ores.filter(ore => ore.health <= 0).length;
            ores = ores.filter(ore => ore.health > 0);

            for (let i = 0; i < minedOresCount; i++) {
                if(ores.length < MAX_ORES) {
                   spawnNewOre();
                }
            }
            
            // Filter out dead ores from excavator targets
            units.forEach(unit => {
                if (unit.target && unit.target.health <= 0) {
                    unit.target = null;
                }
            });


            // Update UI and Market UI elements
            goldCountElement.textContent = gold.toFixed(0);
            merchantCountElement.textContent = units.filter(u => u.type === 'merchant').length;
            accountantCountElement.textContent = accountants.length;
            oreCountElement.textContent = ores.length;
            exchangeCountElement.textContent = bases.length;
            pickaxeLevelElement.textContent = pickaxeLevel;
            const currentUpgradeCost = pickaxeUpgradeBaseCost * Math.pow(2, pickaxeLevel - 1);
            pickaxeUpgradeCostElement.textContent = currentUpgradeCost;

            // Update button disabled states
            spawnMerchantButton.disabled = gold < merchantCost || units.filter(u => u.type === 'merchant').length >= MAX_MERCHANTS;
            spawnAccountantButton.disabled = gold < accountantCost || accountants.length >= MAX_ACCOUNTANTS;
            upgradePickaxeButton.disabled = gold < currentUpgradeCost;
            spawnExchangeButton.disabled = gold < exchangeCost || bases.length >= MAX_BASES;
            spawnExcavatorButton.disabled = gold < excavatorCost;
            
            drawSidebarIcons();
        }

        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // --- User Interaction Functions ---
        function spawnMerchant() {
            if (gold >= merchantCost && units.filter(u => u.type === 'merchant').length < MAX_MERCHANTS) {
                gold -= merchantCost;
                units.push(new Merchant(canvas.width / 2, canvas.height / 2));
            }
        }

        function spawnAccountant() {
            if (gold >= accountantCost && accountants.length < MAX_ACCOUNTANTS) {
                gold -= accountantCost;
                const angle = (accountants.length / MAX_ACCOUNTANTS) * Math.PI * 2;
                const radiusOffset = 25; 
                const xPos = canvas.width / 2 + Math.cos(angle) * radiusOffset;
                const yPos = canvas.height / 2 + Math.sin(angle) * radiusOffset;
                accountants.push(new Accountant(xPos, yPos));
            }
        }
        
        function spawnExcavator() {
             if (gold >= excavatorCost) {
                gold -= excavatorCost;
                 // Spawn near the center base
                units.push(new Excavator(canvas.width / 2, canvas.height / 2));
            }
        }


        function upgradePickaxe() {
            const currentUpgradeCost = pickaxeUpgradeBaseCost * Math.pow(2, pickaxeLevel - 1);
            if (gold >= currentUpgradeCost) {
                gold -= currentUpgradeCost;
                pickaxeLevel += 1;
            }
        }

        function spawnExchange() {
            if (gold >= exchangeCost && bases.length < MAX_BASES) {
                gold -= exchangeCost;
                let x, y, validPlacement = false;
                const minDistance = 50;
                while(!validPlacement) {
                    x = Math.random() * canvas.width;
                    y = Math.random() * canvas.height;
                    validPlacement = true;
                    for(const base of bases) {
                        if (distanceBetween({x,y}, base) < minDistance) {
                            validPlacement = false;
                            break;
                        }
                    }
                    for(const ore of ores) {
                         if (distanceBetween({x,y}, ore) < minDistance) {
                            validPlacement = false;
                            break;
                        }
                    }
                }
                bases.push(new HomeBase(x, y));
            }
        }

        function drawSidebarIcons() {
            const ctxT = document.getElementById('icon-tokedillion').getContext('2d');
            ctxT.clearRect(0, 0, 16, 16);
            ctxT.beginPath();
            ctxT.arc(8, 8, 7, 0, Math.PI * 2);
            ctxT.fillStyle = FILL_COLOR;
            ctxT.strokeStyle = STROKE_COLOR;
            ctxT.fill();
            ctxT.stroke();
            
            const ctxM = document.getElementById('icon-merchant').getContext('2d');
            ctxM.clearRect(0, 0, 16, 16);
            ctxM.beginPath();
            ctxM.arc(8, 8, 6, 0, Math.PI * 2);
            ctxM.fillStyle = FILL_COLOR;
            ctxM.strokeStyle = STROKE_COLOR;
            ctxM.fill();
            ctxM.stroke();

            const ctxA = document.getElementById('icon-accountant').getContext('2d');
            ctxA.clearRect(0, 0, 16, 16);
            ctxA.strokeStyle = STROKE_COLOR;
            ctxA.strokeRect(3, 10, 10, 4); 
            ctxA.beginPath(); 
            ctxA.arc(8, 7, 4, 0, Math.PI * 2);
            ctxA.stroke();

            const ctxO = document.getElementById('icon-ore').getContext('2d');
            ctxO.clearRect(0, 0, 16, 16);
            ctxO.beginPath();
            ctxO.moveTo(8, 1);
            ctxO.lineTo(15, 8);
            ctxO.lineTo(12, 15);
            ctxO.lineTo(4, 15);
            ctxO.lineTo(1, 8);
            ctxO.closePath();
            ctxO.fillStyle = FILL_COLOR;
            ctxO.strokeStyle = STROKE_COLOR;
            ctxO.fill();
            ctxO.stroke();
        }

        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // Initialize and Start Game
        initializeGameWorld();
        gameLoop();
        
        document.getElementById('MiningArea').style.display = 'block';
        document.getElementById('gameCanvas').style.display = 'block';
        drawSidebarIcons();
    </script>
</body>
</html>
