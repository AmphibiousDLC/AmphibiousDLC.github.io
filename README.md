<script>
        // --- DOM Element References ---
        const messageArea = document.getElementById('message-area');
        const tokenCountSpan = document.getElementById('token-count');
        const clickCountSpan = document.getElementById('click-count');
        const betterBaitBtn = document.getElementById('better-bait-btn');
        const shrimpBaitContainer = document.getElementById('shrimp-bait-container');
        const shrimpBaitBtn = document.getElementById('shrimp-bait-btn');
        const troutBaitContainer = document.getElementById('trout-bait-container');
        const troutBaitBtn = document = document.getElementById('trout-bait-btn');
        const placeTrapBtn = document.getElementById('place-trap-btn');
        const placeWeedBtn = document.getElementById('place-weed-btn');
        const pondGrid = document.getElementById('your-pond-grid');
        const inventoryContent = document.getElementById('inventory-content');
        const encyclopediaContent = document.getElementById('encyclopedia-content');
        const salamanderDisplay = document.getElementById('salamander-display');
        const sellItemSelect = document.getElementById('sell-item-select');
        const sellQuantityInput = document.getElementById('sell-quantity-input');
        const redeemCodeInput = document.getElementById('redeem-code-input');

        // --- Game Constants and State Variables ---
        const POND_SIZE = 5;
        const TRAP_DURABILITY = 10; // Durability of a trap once placed
        const POND_WEED_DURATION_MS = 20000; // 20 seconds
        const POND_WEED_COST = 20;
        const TRAP_COST = 15;
        const COOLDOWN_TIME_MS = 2000; // Set to 2 seconds
        const TRAP_EMOJI = '🧺';
        const WEED_EMOJI = '🌿';
        
        let totalClicks = 0;
        let betterBaitPurchased = false;
        let shrimpBaitPurchased = false;
        let troutBaitPurchased = false;
        let placingType = null; // Can be 'trap', 'weed', or null
        let canClick = true;
        let dlcCodeRedeemed = false;
        let hrgCodeRedeemed = false; // New state variable for HRG
        
        let pondCellsData = []; 


        // >>>>> INVENTORY DEFINITIONS <<<<<
        const inventory = {
            common: 0, swamp: 0, stream: 0, rocky: 0, dragonic: 0, baitedHuman: 0, trout: 0, eel: 0,
            token: 0, toad: 0, frog: 0, crayfish: 0, shrimp: 0, whitebait: 0,
            dlcSalamander: 0,
            hrgSalamander: 0,
        };
        const itemDetails = {
            // Common Items
            common: { name: 'Common Salamander', emoji: '🦎', type: 'salamander', sellPrice: 1, rarity: 'common' },
            swamp: { name: 'Swamp Salamander', emoji: '🦎', type: 'salamander', sellPrice: 1, rarity: 'common' },
            stream: { name: 'Stream/River Salamander', emoji: '💧', type: 'salamander', sellPrice: 1, rarity: 'common' },
            whitebait: { name: 'Whitebait', emoji: '🐟', type: 'bait', sellPrice: 1, rarity: 'common' },
            shrimp: { name: 'Shrimp', emoji: '🦐', type: 'bait', sellPrice: 0.5, rarity: 'common' },
            
            // Rare Items
            rocky: { name: 'Rocky Bay Salamander', emoji: '🪨', type: 'salamander', sellPrice: 1, rarity: 'rare' },
            dragonic: { name: 'Dragonic Salamander', emoji: '🐉', type: 'salamander', sellPrice: 2, rarity: 'rare' },
            crayfish: { name: 'Crayfish', emoji: '🦞', type: 'bait', sellPrice: 2, rarity: 'rare' },
            toad: { name: 'Toad', emoji: '🐸', type: 'bait', sellPrice: 1, rarity: 'rare' },
            frog: { name: 'Frog', emoji: '🐸', type: 'bait', sellPrice: 2, rarity: 'rare' },

            // Epic Items
            trout: { name: 'Trout', emoji: '🐟', type: 'misc', sellPrice: 2, rarity: 'epic' },
            eel: { name: 'Eel', emoji: '🐍', type: 'misc', sellPrice: 2, rarity: 'epic' },
            
            // Secret Items
            dlcSalamander: { name: 'DLC Salamander', emoji: '🏆', type: 'salamander', sellPrice: 50, rarity: 'secret' },
            hrgSalamander: { name: 'HOG RIDERRRRR Salamander', emoji: '🐖', type: 'salamander', sellPrice: 50, rarity: 'secret' },
            baitedHuman: { name: 'Baited Human', emoji: '👤', type: 'misc', sellPrice: 0, rarity: 'secret' }, 
            
            // Currency (No rarity needed)
            token: { name: 'Token', emoji: '🟡', type: 'currency', sellPrice: 0 }
        };
        // >>>>> END INVENTORY DEFINITIONS <<<<<


        // Function to determine drop table for main clicks (FIXED logic)
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

            // Use independent 'if' statements so all purchased baits can add their drops
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
             // Clear previous entries except for sell controls and code area
            const itemsList = Array.from(inventoryContent.children).filter(child => child.id !== 'sell-controls');
            itemsList.forEach(item => item.remove());

            for (const key in inventory) {
                const count = inventory[key];
                if (count > 0 && key !== 'token') {
                    const details = itemDetails[key];
                    const itemDiv = document.createElement('div');
                    itemDiv.classList.add('encyclopedia-item');
                    // Add the rarity class here
                    itemDiv.classList.add(`rarity-${details.rarity}`); 
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

                // If the code is redeemed, the item is always visible in encyclopedia
                if (count > 0 || (key === 'dlcSalamander' && dlcCodeRedeemed) || (key === 'hrgSalamander' && hrgCodeRedeemed)) {
                    itemDiv.innerHTML = `${details.emoji} ${details.name}`;
                    // Add the rarity class here for discovered items
                    itemDiv.classList.add(`rarity-${details.rarity}`);
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

        function sellAllItems() {
            let totalTokensEarned = 0;
            let itemsSoldCount = 0;

            for (const itemKey in inventory) {
                // Check if the item is sellable (price > 0) and not a token
                if (itemKey !== 'token' && itemDetails[itemKey].sellPrice > 0) {
                    const quantity = inventory[itemKey];
                    if (quantity > 0) {
                        const details = itemDetails[itemKey];
                        totalTokensEarned += quantity * details.sellPrice;
                        itemsSoldCount += quantity;
                        inventory[itemKey] = 0; // Set inventory count to zero
                    }
                }
            }

            if (itemsSoldCount > 0) {
                inventory.token += totalTokensEarned;
                messageArea.textContent = `Sold all ${itemsSoldCount} sellable items for a total of ${totalTokensEarned} tokens!`;
            } else {
                messageArea.textContent = "You have no sellable items to sell.";
            }
            
            updateDisplay();
        }

        function redeemCode() {
            const code = redeemCodeInput.value.trim().toUpperCase(); // Convert to uppercase for case-insensitivity

            if (code === 'AMPHIBIOUSDLC') {
                if (!dlcCodeRedeemed) {
                    inventory.dlcSalamander += 1;
                    dlcCodeRedeemed = true; // Prevents multiple uses
                    messageArea.textContent = "Code accepted! You received one DLC Salamander 🏆!";
                } else {
                    messageArea.textContent = "This code has already been redeemed.";
                }
            } else if (code === 'HRG') {
                if (!hrgCodeRedeemed) {
                    inventory.hrgSalamander += 1;
                    hrgCodeRedeemed = true; // Prevents multiple uses
                    messageArea.textContent = "HOG RIDERRRRR! Code accepted! You received one HRG Salamander 🐖!";
                } else {
                    messageArea.textContent = "This code has already been redeemed.";
                }
            } else {
                messageArea.textContent = "Invalid code.";
            }
            updateDisplay();
            redeemCodeInput.value = "";
        }
        
        // --- Corrected Click Salamander Function ---
        function clickSalamander() {
            if (!canClick) {
                messageArea.textContent = "Cooldown active! Please wait 2 seconds.";
                return;
            }
            canClick = false;
            salamanderDisplay.classList.add('cooldown-active');

            // Set a timer to end the cooldown
            setTimeout(() => {
                canClick = true;
                salamanderDisplay.classList.remove('cooldown-active');
                messageArea.textContent = "Click the salamander to search!"; // Reset message
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
        salamanderDisplay.addEventListener('click', clickSalamander); // Attach the event listener

    </script>
</body>
</html>
