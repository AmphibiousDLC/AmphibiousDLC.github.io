<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Salamander Clicker - Final Layout</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #4CAF50;
            color: white;
            font-family: sans-serif;
            margin: 0;
            text-align: center;
            font-size: 0.9em;
        }

        /* The new wrapper acting as the 'dark gray box' sandwich */
        #game-wrapper {
            background-color: #333; /* Dark gray for the top/bottom effect */
            padding: 10px;
            border-radius: 10px;
            max-width: 980px; /* Fixed maximum width for the whole game area */
            width: 95vw; /* Responsive width up to max-width */
        }

        /* Container for the three main columns to sit side-by-side */
        #main-content-row {
            display: flex;
            flex-direction: row;
            justify-content: center;
            gap: 10px; /* Space between the three columns */
        }


        #main-clicker-area {
            background-color: rgba(0, 0, 0, 0.7);
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 0 15px rgba(0, 0, 0, 0.5);
            width: 330px; 
            height: 650px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
        }

        #salamander-display { font-size: 120px; cursor: pointer; transition: transform 0.1s ease; display: block; margin: 15px auto; }
        #salamander-display:hover { transform: scale(1.1); }
        #salamander-display.cooldown-active { cursor: not-allowed; opacity: 0.6; }

        #message-area { padding: 8px; font-size: 0.9em; }
        #stats-bar { font-size: 1em; margin-bottom: 10px; }

        .upgrade-btn { padding: 8px 12px; font-size: 0.9em; margin: 5px 0; width: 100%; box-sizing: border-box; }
        #shrimp-bait-container, #trout-bait-container { display: none; width: 100%; }

        #upgrades {
            width: 100%;
        }

        /* --- Pond Area Styles --- */
        #pond-area-container {
            width: 250px;
            height: 650px;
            background-color: rgba(0, 0, 0, 0.7);
            padding: 15px;
            border-radius: 8px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        
        #your-pond-grid {
            display: grid;
            grid-template-columns: repeat(5, 40px);
            grid-template-rows: repeat(5, 40px);
            gap: 3px;
            background-color: #4da6ff;
            border: 3px solid #333;
            padding: 5px;
        }

        .pond-cell { width: 40px; height: 40px; background-color: rgba(255, 255, 255, 0.3); border: 1px solid rgba(0, 0, 0, 0.2); display: flex; align-items: center; justify-content: center; font-size: 20px; cursor: pointer; }
        .pond-cell:hover { background-color: rgba(255, 255, 255, 0.5); }
        .pond-cell.weed-active { background-color: rgba(76, 175, 80, 0.7); }


        #trap-controls {
            width: 100%;
            margin-top: 10px;
            margin-bottom: 20px;
        }

        #place-trap-btn { width: 100%; padding: 10px; font-size: 0.9em; background-color: orange; color: black; border: none; border-radius: 5px; cursor: pointer; }
        #place-trap-btn:disabled { background-color: #7f8c8d; cursor: not-allowed; }
        #place-weed-btn { width: 100%; padding: 10px; font-size: 0.9em; background-color: #4CAF50; color: white; border: none; border-radius: 5px; cursor: pointer; margin-top: 10px;}
        #place-weed-btn:disabled { background-color: #7f8c8d; cursor: not-allowed; }


        /* --- Encyclopedia & Inventory Tab Styles --- */
        #info-container {
            width: 300px;
            height: 650px;
            background-color: rgba(0, 0, 0, 0.7);
            padding: 15px;
            border-radius: 8px;
            text-align: left;
            display: flex;
            flex-direction: column;
        }
        #tab-buttons {
            display: flex; 
            justify-content: space-between; /* Ensures even space between */
            margin-bottom: 10px;
            align-items: center;
        }
        .tab-btn {
            padding: 8px 12px;
            cursor: pointer;
            background-color: #555;
            border: none;
            color: white;
            width: 48%; /* Wider width */
            box-sizing: border-box; 
        }
        .tab-btn.active {
            background-color: #4CAF50;
        }
        .tab-content {
            display: none;
            overflow-y: auto;
            flex-grow: 1;
        }
        .tab-content.active {
            display: block;
        }
        .encyclopedia-item {
            padding: 4px 0;
            border-bottom: 1px solid rgba(255, 255, 255, 0.2);
            font-size: 0.9em;
        }
        .item-locked {
            color: #aaa;
            font-style: italic;
        }
        .item-count {
            float: right;
            font-weight: bold;
        }
        #sell-controls {
            margin-top: 15px;
            padding-top: 10px;
            border-top: 1px solid #ddd;
        }
        #sell-controls select, #sell-controls input, #sell-controls button {
            width: 100%;
            padding: 8px;
            margin-bottom: 5px;
            box-sizing: border-box;
        }
    </style>
</head>
<body>
    <!-- Main Sandwich Wrapper -->
    <div id="game-wrapper">
        <div id="main-content-row">

            <div id="main-clicker-area">
                <div>
                    <h1>Salamander Clicker</h1>
                    <p style="font-size: 0.8em; margin-top: -15px; margin-bottom: 15px;">Made by AmphibiousDLC</p>
                    
                    <div id="stats-bar">
                        <div>Clicks: <span id="click-count">0</span></div>
                        <div>Tokens: 🟡 <span id="token-count">0</span></div>
                    </div>

                    <!-- --- Upgrades Area --- -->
                    <div id="upgrades">
                        <button id="better-bait-btn" class="upgrade-btn" onclick="buyBetterBait()">Buy "Better Bait" (Cost: 10 Tokens)</button>
                        
                        <div id="shrimp-bait-container">
                            <button id="shrimp-bait-btn" class="upgrade-btn" onclick="buyShrimpBait()">Buy "Shrimp Bait" (Cost: 50 Tokens)</button>
                        </div>

                        <div id="trout-bait-container">
                            <button id="trout-bait-btn" class="upgrade-btn" onclick="buyTroutBait()">Buy "Trout Bait" (Cost: 100 Tokens)</button>
                        </div>
                    </div>

                    <div id="message-area">Click the salamander to search!</div>

                    <span id="salamander-display" onclick="clickSalamander()">🦎</span>
                </div>
            </div>
            
            
            <!-- --- New Pond Area --- -->
            <div id="pond-area-container">
                <div id="pond-label">Your Pond</div>
                <div id="your-pond-grid">
                    <!-- Pond cells generated by JS -->
                </div>
                <div id="trap-controls">
                    <button id="place-trap-btn" onclick="toggleTrapPlacement('trap')">🛒 Place Bait Trap (Cost: 15 Tokens)</button>
                    <button id="place-weed-btn" onclick="toggleTrapPlacement('weed')">🌿 Place Pond Weed (Cost: 20 Tokens)</button>
                </div>
            </div>

            <!-- --- New Info Area (Tabs) --- -->
            <div id="info-container">
                <div id="tab-buttons">
                    <button id="tab-inventory-btn" class="tab-btn active" onclick="switchTab('inventory')">Inventory</button>
                    <button id="tab-encyclopedia-btn" class="tab-btn" onclick="switchTab('encyclopedia')">Encyclopedia</button>
                </div>

                <div id="inventory-content" class="tab-content active">
                     <!-- Inventory items listed here dynamically -->
                    <div id="sell-controls">
                        <h3>Sell Catches</h3>
                        <select id="sell-item-select">
                            <option value="">Select an item to sell</option>
                        </select>
                        <input type="number" id="sell-quantity-input" placeholder="Quantity (e.g., 1 or 99)" min="1" value="1">
                        <button onclick="sellItems()">Sell Selected Items</button>
                    </div>
                </div>

                <div id="encyclopedia-content" class="tab-content">
                    <!-- Encyclopedia items listed here dynamically -->
                </div>
            </div>
        </div> <!-- End of main-content-row -->
    </div> <!-- End of game-wrapper -->


    <script>
        // --- DOM Element References ---
        const messageArea = document.getElementById('message-area');
        const tokenCountSpan = document.getElementById('token-count');
        const clickCountSpan = document.getElementById('click-count');
        const betterBaitBtn = document.getElementById('better-bait-btn');
        const shrimpBaitContainer = document.getElementById('shrimp-bait-container');
        const shrimpBaitBtn = document.getElementById('shrimp-bait-btn');
        const troutBaitContainer = document.getElementById('trout-bait-container');
        const troutBaitBtn = document.getElementById('trout-bait-btn');
        const placeTrapBtn = document.getElementById('place-trap-btn');
        const placeWeedBtn = document.getElementById('place-weed-btn');
        const pondGrid = document.getElementById('your-pond-grid');
        const inventoryContent = document.getElementById('inventory-content');
        const encyclopediaContent = document.getElementById('encyclopedia-content');
        const salamanderDisplay = document.getElementById('salamander-display');
        const sellItemSelect = document.getElementById('sell-item-select');
        const sellQuantityInput = document.getElementById('sell-quantity-input');


        // --- Game Constants and State Variables ---
        const POND_SIZE = 5;
        const TRAP_DURABILITY = 10; // Durability of a trap once placed
        const POND_WEED_DURATION_MS = 20000; // 20 seconds
        const POND_WEED_COST = 20;
        const TRAP_COST = 15;
        const COOLDOWN_TIME_MS = 5000;
        const TRAP_EMOJI = '🧺';
        const WEED_EMOJI = '🌿';
        
        let totalClicks = 0;
        let betterBaitPurchased = false;
        let shrimpBaitPurchased = false;
        let troutBaitPurchased = false;
        let placingType = null; // Can be 'trap', 'weed', or null
        let canClick = true;
        
        let pondCellsData = []; 


        // >>>>> INVENTORY DEFINITIONS <<<<<
        const inventory = {
            common: 0, swamp: 0, stream: 0, rocky: 0, dragonic: 0, baitedHuman: 0, trout: 0, eel: 0,
            token: 0, toad: 0, frog: 0, crayfish: 0, shrimp: 0, whitebait: 0,
        };
        const itemDetails = {
            common: { name: 'Common Salamander', emoji: '🦎', type: 'salamander', sellPrice: 1 },
            swamp: { name: 'Swamp Salamander', emoji: '🦎', type: 'salamander', sellPrice: 1 },
            stream: { name: 'Stream/River Salamander', emoji: '💧', type: 'salamander', sellPrice: 1 },
            rocky: { name: 'Rocky Bay Salamander', emoji: '🪨', type: 'salamander', sellPrice: 1 },
            dragonic: { name: 'Dragonic Salamander', emoji: '🐉', type: 'salamander', sellPrice: 2 },
            baitedHuman: { name: 'Baited Human', emoji: '👤', type: 'misc', sellPrice: 0 }, 
            trout: { name: 'Trout', emoji: '🐟', type: 'misc', sellPrice: 2 },
            eel: { name: 'Eel', emoji: '🐍', type: 'misc', sellPrice: 2 },
            token: { name: 'Token', emoji: '🟡', type: 'currency', sellPrice: 0 }, 
            toad: { name: 'Toad', emoji: '🐸', type: 'bait', sellPrice: 1 },
            frog: { name: 'Frog', emoji: '🐸', type: 'bait', sellPrice: 2 },
            crayfish: { name: 'Crayfish', emoji: '🦞', type: 'bait', sellPrice: 2 },
            shrimp: { name: 'Shrimp', emoji: '🦐', type: 'bait', sellPrice: 0.5 },
            whitebait: { name: 'Whitebait', emoji: '🐟', type: 'bait', sellPrice: 1 },
        };
        // >>>>> END INVENTORY DEFINITIONS <<<<<


        // Function to determine drop table for main clicks (UPDATED with user logic)
        function getDropsTable() {
            // Base drops available with no bait (and tokens from all)
            let currentDrops = [
                { nameKey: 'common', chance: 40, message: 'You found a Common Salamander.' },
                { nameKey: 'swamp', chance: 25, message: 'You found a Swamp Salamander!' },
                { nameKey: 'stream', chance: 15, message: 'A Stream/River Salamander appeared!' },
                { nameKey: 'rocky', chance: 8, message: 'A Rocky Bay Salamander scampers out!' },
                { nameKey: 'dragonic', chance: 2, message: 'Incredible! A Dragonic Salamander!' },
                { nameKey: 'baitedHuman', chance: 0.5, message: 'A confused human wanders by, looking baited!' },
                { nameKey: 'token', chance: 9.5, message: 'You found a token! 🟡' }
            ];
            
            if (betterBaitPurchased) { 
                currentDrops.push(
                    { nameKey: 'toad', chance: 5, message: 'You caught a common toad! 🐸' }, 
                    { nameKey: 'frog', chance: 3, message: 'You caught a small frog! 🐸' }
                ); 
            }
            if (shrimpBaitPurchased) { 
                currentDrops.push(
                    { nameKey: 'crayfish', chance: 4, message: 'You found a crayfish! 🦞' }, 
                    { nameKey: 'shrimp', chance: 2, message: 'Wow! A rare shrimp! 🦐' }, 
                    { nameKey: 'whitebait', chance: 2, message: 'A school of whitebait swam by! 🐟' }
                ); 
            }
            if (troutBaitPurchased) { 
                currentDrops.push(
                    { nameKey: 'trout', chance: 4, message: 'You caught a large Trout! What a catch!' }, 
                    { nameKey: 'eel', chance: 2, message: 'A slippery eel appeared!' }
                ); 
            }

            return currentDrops;
        }

        // Function to determine drop table for TRAPS
        function getTrapDrops() {
            const weedIsActive = pondCellsData.some(cell => cell && cell.itemType === 'weed' && cell.activeUntil > Date.now());

            if (weedIsActive) {
                return [ 
                    { nameKey: 'whitebait', chance: 30 }, 
                    { nameKey: 'crayfish', chance: 20 }, 
                    { nameKey: 'token', chance: 20 },
                    { nameKey: 'shrimp', chance: 30 }, 
                ];
            } else {
                return [ 
                    { nameKey: 'whitebait', chance: 60 }, 
                    { nameKey: 'crayfish', chance: 30 }, 
                    { nameKey: 'token', chance: 10 } 
                ];
            }
        }


        function updateDisplay() {
            tokenCountSpan.textContent = inventory.token;
            clickCountSpan.textContent = totalClicks;
            betterBaitBtn.disabled = (inventory.token < 10 || betterBaitPurchased);
            
            if (betterBaitPurchased) {
                shrimpBaitContainer.style.display = 'block';
                shrimpBaitBtn.disabled = (inventory.token < 50 || shrimpBaitPurchased);
            } else {
                 shrimpBaitContainer.style.display = 'none';
            }

            if (shrimpBaitPurchased) {
                troutBaitContainer.style.display = 'block';
                troutBaitBtn.disabled = (inventory.token < 100 || troutBaitPurchased);
            } else {
                troutBaitContainer.style.display = 'none';
            }
            
            placeTrapBtn.disabled = (inventory.token < TRAP_COST);
            placeWeedBtn.disabled = (inventory.token < POND_WEED_COST);

            updateInventoryList();
            updateEncyclopediaList();
        }

        function switchTab(tabName) {
            document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
            document.querySelectorAll('.tab-content').forEach(content => content.classList.remove('active'));
            document.getElementById(`tab-${tabName}-btn`).classList.add('active');
            document.getElementById(`${tabName}-content`).classList.add('active');
        }

        function updateInventoryList() {
             // Clear previous entries except for sell controls
            const itemsList = Array.from(inventoryContent.children).filter(child => child.id !== 'sell-controls');
            itemsList.forEach(item => item.remove());

            for (const key in inventory) {
                const count = inventory[key];
                if (count > 0 && key !== 'token') {
                    const details = itemDetails[key];
                    const itemDiv = document.createElement('div');
                    itemDiv.classList.add('encyclopedia-item');
                    itemDiv.innerHTML = `${details.emoji} ${details.name} <span class="item-count">${count}</span>`;
                    inventoryContent.insertBefore(itemDiv, document.getElementById('sell-controls'));
                }
            }

            sellItemSelect.innerHTML = '<option value="">Select an item to sell</option>';
            for (const key in inventory) {
                if (inventory[key] > 0 && key !== 'token' && itemDetails[key].sellPrice > 0) {
                    const details = itemDetails[key];
                    const option = document.createElement('option');
                    option.value = key;
                    option.textContent = `${details.name} (${inventory[key]} in stock) - Sells for ${details.sellPrice} Tokens`;
                    sellItemSelect.appendChild(option);
                }
            }
        }
        
        function updateEncyclopediaList() {
            encyclopediaContent.innerHTML = '';
            for (const key in itemDetails) {
                if (key === 'token') continue; 
                const details = itemDetails[key];
                const count = inventory[key];
                const itemDiv = document.createElement('div');
                itemDiv.classList.add('encyclopedia-item');

                if (count > 0) {
                    itemDiv.innerHTML = `${details.emoji} ${details.name}`;
                } else {
                    itemDiv.innerHTML = `??? (Locked)`;
                    itemDiv.classList.add('item-locked');
                }
                encyclopediaContent.appendChild(itemDiv);
            }
        }

        function sellItems() {
            const itemKey = sellItemSelect.value;
            const quantityString = sellQuantityInput.value;
            const quantity = parseInt(quantityString, 10);

            if (!itemKey) {
                messageArea.textContent = "Please select an item to sell.";
                return;
            }

            if (isNaN(quantity) || quantity < 1) {
                messageArea.textContent = "Please enter a valid quantity (1 or more).";
                return;
            }

            if (inventory[itemKey] >= quantity) {
                const details = itemDetails[itemKey];
                const valuePerItem = details.sellPrice;
                const totalTokensEarned = valuePerItem * quantity;

                inventory[itemKey] -= quantity;
                inventory.token += totalTokensEarned;
                messageArea.textContent = `Sold ${quantity} ${details.name}(s) for ${totalTokensEarned} tokens!`;
                
                sellItemSelect.value = "";
                sellQuantityInput.value = "1";

                updateDisplay();
            } else {
                messageArea.textContent = `You do not have ${quantity} ${itemDetails[itemKey].name}s to sell.`;
            }
        }
        
        function clickSalamander() {
            if (!canClick) {
                messageArea.textContent = "Cooldown active! Please wait 5 seconds.";
                return;
            }
            canClick = false;
            salamanderDisplay.classList.add('cooldown-active');
            let timeLeft = COOLDOWN_TIME_MS / 1000;
            const countdownInterval = setInterval(() => {
                timeLeft--;
                if (timeLeft > 0) { messageArea.textContent = `Next click in ${timeLeft}s...`; } else { clearInterval(countdownInterval); }
            }, 1000);
            setTimeout(() => {
                canClick = true;
                salamanderDisplay.classList.remove('cooldown-active');
            }, COOLDOWN_TIME_MS);

            totalClicks++;
            const currentDrops = getDropsTable();
            const roll = Math.random() * 100;
            let cumulativeChance = 0;
            let foundKey = null;
            let foundMessage = '';
            for (const drop of currentDrops) {
                cumulativeChance += drop.chance;
                if (roll < cumulativeChance) {
                    foundKey = drop.nameKey;
                    foundMessage = drop.message;
                    break;
                }
            }
            if (foundKey) {
                 inventory[foundKey]++; 
                 messageArea.textContent = foundMessage;
            }
            updateDisplay();
        }
        function buyBetterBait() {
            if (inventory.token >= 10 && !betterBaitPurchased) {
                inventory.token -= 10; betterBaitPurchased = true; betterBaitBtn.textContent = "Better Bait Purchased!"; messageArea.textContent = "Better Bait is active! New creatures and better odds are available!"; updateDisplay();
            }
        }
        function buyShrimpBait() {
             if (inventory.token >= 50 && betterBaitPurchased && !shrimpBaitPurchased) {
                inventory.token -= 50; shrimpBaitPurchased = true; shrimpBaitBtn.textContent = "Shrimp Bait Purchased!"; messageArea.textContent = "Shrimp Bait active! Percentages boosted +5%, new items unlocked!"; updateDisplay();
            }
        }
        function buyTroutBait() {
            if (inventory.token >= 100 && shrimpBaitPurchased && !troutBaitPurchased) {
                inventory.token -= 100; troutBaitPurchased = true; troutBaitBtn.textContent = "Trout Bait Purchased! (MAX BAIT TIER)"; messageArea.textContent = "TROUT BAIT ACTIVE! Prepare for ultimate catches!"; updateDisplay();
            }
        }

        function initializePond() {
            for (let i = 0; i < POND_SIZE * POND_SIZE; i++) {
                pondCellsData.push({ itemType: null, durability: 0, activeUntil: 0 }); // Added activeUntil
                const cell = document.createElement('div');
                cell.classList.add('pond-cell');
                cell.dataset.index = i;
                cell.addEventListener('click', handlePondClick);
                pondGrid.appendChild(cell);
            }
            setInterval(processTraps, 3000); // Renamed from runTrapCollection
            setInterval(updatePondVisuals, 500); // Add a small loop to update weed visuals
        }

        function toggleTrapPlacement(type) {
            // Stop current placement if toggling the same type or switching
            if (placingType === type) {
                placingType = null;
                messageArea.textContent = "Placement canceled.";
            } else {
                placingType = type;
                if (type === 'trap' && inventory.token < TRAP_COST) { placingType = null; alert("Not enough tokens!"); return; }
                if (type === 'weed' && inventory.token < POND_WEED_COST) { placingType = null; alert("Not enough tokens!"); return; }
                messageArea.textContent = `Click a pond square to place ${type}.`;
            }

            // Update button visual states
            placeTrapBtn.style.backgroundColor = (placingType === 'trap') ? 'green' : 'orange';
            placeWeedBtn.style.backgroundColor = (placingType === 'weed') ? 'green' : '#4CAF50';
        }

        function handlePondClick(event) {
            const cellIndex = event.target.dataset.index;
            const cellData = pondCellsData[cellIndex];

            if (placingType === 'trap') {
                if (cellData.itemType === null) {
                    inventory.token -= TRAP_COST;
                    cellData.itemType = 'trap';
                    cellData.durability = TRAP_DURABILITY; 
                    event.target.textContent = TRAP_EMOJI;
                    toggleTrapPlacement(null); // End placement mode
                    updateDisplay();
                } else { messageArea.textContent = "That spot is already used!"; }
            } else if (placingType === 'weed') {
                if (cellData.itemType === null) {
                    inventory.token -= POND_WEED_COST;
                    cellData.itemType = 'weed';
                    cellData.activeUntil = Date.now() + POND_WEED_DURATION_MS;
                    event.target.textContent = WEED_EMOJI;
                    event.target.classList.add('weed-active'); // Add a class for visual effect
                    toggleTrapPlacement(null); // End placement mode
                    updateDisplay();
                } else { messageArea.textContent = "That spot is already used!"; }
            }
        }
        
        function processTraps() {
            const trapDrops = getTrapDrops(); // Get the correct drop table (potentially boosted)
            
            pondCellsData.forEach((cellData, index) => {
                if (cellData.itemType === 'trap' && cellData.durability > 0) {
                    if (Math.random() < 0.1) { 
                        const roll = Math.random() * 100;
                        let cumulativeChance = 0;
                        let foundKey = null;

                        for (const drop of trapDrops) {
                            cumulativeChance += drop.chance;
                            if (roll < cumulativeChance) {
                                foundKey = drop.nameKey;
                                break;
                            }
                        }

                        if (foundKey) {
                            if (foundKey === 'token') inventory.token++; else inventory[foundKey]++;
                            
                            cellData.durability--;
                            
                            const caughtItemName = itemDetails[foundKey] ? itemDetails[foundKey].name : 'an unknown item';
                            messageArea.textContent = `A trap caught ${caughtItemName}!`;

                            const cellElement = pondGrid.children[index];
                            if (cellData.durability <= 0) {
                                cellData.itemType = null; // Mark as empty
                                cellElement.textContent = ''; 
                                messageArea.textContent += " (The trap broke!)";
                            }
                        }
                    }
                }
            });
            updateDisplay();
        }

        function updatePondVisuals() {
            const now = Date.now();
            let needUpdate = false;
            pondCellsData.forEach((cellData, index) => {
                if (cellData.itemType === 'weed' && cellData.activeUntil <= now) {
                    cellData.itemType = null;
                    cellData.activeUntil = 0;
                    const cellElement = pondGrid.children[index];
                    cellElement.textContent = '';
                    cellElement.classList.remove('weed-active');
                    needUpdate = true;
                }
            });
            if (needUpdate) {
                updateDisplay();
            }
        }


        // --- Initialization ---
        initializePond(); 
        updateDisplay(); 
        switchTab('inventory'); // Start on inventory tab
    </script>
</body>
</html>
