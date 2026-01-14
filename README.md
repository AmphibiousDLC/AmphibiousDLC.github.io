<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Dungeon Architect 2026: Slower Prey</title>
    <style>
        body { background: #0a0a0c; color: #d4af37; font-family: 'Garamond', serif; display: flex; flex-direction: column; align-items: center; margin: 0; padding: 5px; overflow: hidden; }
        #ui { background: #1a1a1c; border: 2px solid #444; padding: 10px; border-radius: 5px; display: flex; gap: 4px; flex-wrap: wrap; justify-content: center; max-width: 950px; }
        canvas { background: #050505; border: 3px solid #333; image-rendering: pixelated; touch-action: none; max-width: 100%; cursor: crosshair; }
        .btn { padding: 6px 8px; border: 1px solid #d4af37; background: #222; color: #d4af37; cursor: pointer; font-size: 0.7em; font-weight: bold; }
        .selected { background: #d4af37; color: #111; box-shadow: 0 0 10px #d4af37; }
        .gold { font-size: 1.2em; color: #ffd700; width: 100%; text-align: center; margin-bottom: 2px; }
        #merchant-ui { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); background: #1a1a1c; border: 3px solid #ffd700; padding: 20px; display: none; flex-direction: column; gap: 10px; z-index: 3000; box-shadow: 0 0 50px #000; border-radius: 8px; text-align: center; }
    </style>
</head>
<body>
    <div class="gold">TREASURY: <span id="gold-val">1500</span>g | REPUTATION: <span id="prestige-val">0</span></div>
    
    <div id="merchant-ui">
        <h2 style="margin:0; color:#ffd700;">MERCHANT'S BOUNTY</h2>
        <button class="btn" onclick="buyBoost('strength')">STRENGTH BOOST (+20% Dmg)</button>
        <button class="btn" onclick="buyBoost('treasure')">TREASURE BOOST (+20% Gold)</button>
        <button class="btn" onclick="buyBoost('discount')">DISCOUNT BOOST (-10g Traps)</button>
        <button class="btn" id="king-btn" style="display:none; border-color:#0ff;" onclick="buyBoost('king')">SKELETON KING (1000g)</button>
    </div>

    <div id="ui">
        <button class="btn selected" onclick="setBrush('wall', 0, event)">WALL (0g)</button>
        <button class="btn" onclick="setBrush('spike', 50, event)">SPIKE (50g)</button>
        <button class="btn" onclick="setBrush('lava', 35, event)">LAVA (35g)</button>
        <button class="btn" onclick="setBrush('acid', 35, event)">ACID (35g)</button>
        <button class="btn" onclick="setBrush('guard', 150, event)">GUARD/DRAGON (150g)</button>
        <button class="btn" onclick="setBrush('arrow', 100, event)">ARROW (100g)</button>
        <button class="btn" onclick="setBrush('demon', 200, event)">DEMON (200g)</button>
        <button class="btn" id="btn-sking" style="display:none; border-color:#0ff;" onclick="setBrush('sking', 1000, event)">SKELETON KING</button>
        <button class="btn" onclick="setBrush('rotate', 0, event)" style="border-color: #0088ff;">ROTATE</button>
        <button class="btn" onclick="setBrush('upgrade', 75, event)" style="border-color: #ff0;">UPGRADE (75g)</button>
        <button class="btn" onclick="setBrush('sell', 0, event)" style="border-color: #f00;">DELETE</button>
        <button class="btn" id="auto-btn" onclick="toggleAuto()">AUTO-SPAWN: OFF</button>
        <button class="btn" onclick="rollAdventurer()" style="border-color: #0f0; width: 100%;">SUMMON HERO</button>
    </div>
    <canvas id="dungeon" width="800" height="400"></canvas>

<script>
    const canvas = document.getElementById('dungeon'), ctx = canvas.getContext('2d');
    const sz = 25, rows = 16, cols = 32;
    let gold = 1500, brush = 'wall', cost = 0, prestige = 0, autoSpawn = false;
    let trapDiscount = 0, treasureMult = 1, strengthMult = 1;
    let activeHeroes = [], demons = [], skeletons = [], projectiles = [];

    let grid = Array.from({length: rows}, (_, r) => Array.from({length: cols}, (_, c) => {
        if (r === 0 || r === rows-1 || c === 0 || c === cols-1) return {type:'wall', permanent:true, lvl:1, rot:0};
        return null;
    }));

    function setBrush(t, c, e) { brush = t; cost = Math.max(0, c - trapDiscount); document.querySelectorAll('.btn').forEach(b => b.classList.remove('selected')); if(e) e.target.classList.add('selected'); }
    function toggleAuto() { autoSpawn = !autoSpawn; document.getElementById('auto-btn').innerText = `AUTO-SPAWN: ${autoSpawn ? 'ON' : 'OFF'}`; if(autoSpawn && activeHeroes.length === 0) rollAdventurer(); }

    function buyBoost(type) {
        if (type === 'strength') strengthMult += 0.2;
        if (type === 'treasure') treasureMult += 0.2;
        if (type === 'discount') trapDiscount += 10;
        if (type === 'king' && gold >= 1000) { gold -= 1000; document.getElementById('btn-sking').style.display = 'block'; }
        document.getElementById('merchant-ui').style.display = 'none';
    }

    canvas.addEventListener('pointerdown', (e) => {
        const r = canvas.getBoundingClientRect();
        const x = Math.floor((e.clientX - r.left) / (r.width / cols)), y = Math.floor((e.clientY - r.top) / (r.height / rows));
        if (brush === 'upgrade' && grid[y][x] && !grid[y][x].permanent) { if(grid[y][x].lvl < 3 && gold >= 75) { grid[y][x].lvl++; gold -= 75; } return; }
        if (brush === 'rotate' && grid[y][x] && !grid[y][x].permanent) { grid[y][x].rot += Math.PI/2; return; }
        if (brush === 'sell' && grid[y][x] && !grid[y][x].permanent) { gold += 20; grid[y][x] = null; return; }
        if (!grid[y][x] && gold >= cost) { grid[y][x] = { type: brush, lastFire: Date.now(), lvl: 1, rot: 0 }; gold -= cost; }
    });

    function rollAdventurer() {
        if (activeHeroes.length > 0) return;
        const r = Math.random();
        if (r < 0.10) spawnHero("Merchant", 0);
        else { const c = ["Rogue","Knight","Priest","Minotaur","Wizard"]; spawnHero(c[Math.floor(Math.random()*c.length)], 0); }
    }

    function spawnHero(n, off) {
        const stats = { 
            Rogue:   {c:"#8a2be2",h:65,im:["spike"],s:2.5}, 
            Knight:  {c:"#c0c0c0",h:220,im:[],s:1.4}, 
            Priest:  {c:"#fffacd",h:90,im:["demon"],s:1.4}, 
            Wizard:  {c:"#4b0082",h:75,im:["arrow"],s:1.4}, 
            Minotaur:{c:"#5d4037",h:300,im:[],s:1.4}, 
            Merchant:{c:"#ffd700",h:350,im:[],s:1.4} 
        };
        const s = stats[n]; activeHeroes.push({ name:n, color:s.c, px:50+off, py:200, hp:s.h+(prestige*10), maxHp:s.h+(prestige*10), im:s.im, speed:s.s });
    }

    function gameLoop() {
        const now = Date.now();
        for(let r=0; r<rows; r++) for(let c=0; c<cols; c++) {
            let t = grid[r][c]; if (!t) continue;
            if (t.type === 'guard' && t.lvl === 3 && activeHeroes.length > 0 && now - t.lastFire > 2000) {
                let h = activeHeroes[0];
                let a = Math.atan2(h.py - (r*sz+sz/2), h.px - (c*sz+sz/2));
                projectiles.push({x:c*sz+sz/2, y:r*sz+sz/2, vx:Math.cos(a)*4, vy:Math.sin(a)*4, type:'fireball'}); t.lastFire = now;
            }
            if (t.type === 'arrow' && now - t.lastFire > 1200) { projectiles.push({x:c*sz+sz/2, y:r*sz+sz/2, vx:Math.cos(t.rot)*6, vy:Math.sin(t.rot)*6, type:'arrow'}); t.lastFire = now; }
            if (t.type === 'demon' && now - t.lastFire > 8000) { demons.push({px:c*sz+sz/2, py:r*sz+sz/2, isKing: Math.random()<0.01}); t.lastFire = now; }
            if (t.type === 'sking' && now - t.lastFire > 4000) { skeletons.push({px:c*sz+sz/2, py:r*sz+sz/2}); t.lastFire = now; }
        }

        activeHeroes.forEach(h => {
            let a = Math.atan2(200-h.py, 750-h.px), nx = h.px+Math.cos(a)*h.speed, ny = h.py+Math.sin(a)*h.speed;
            let gx = Math.floor(nx/sz), gy = Math.floor(ny/sz);
            if (grid[gy]?.[gx]?.type === 'wall') {
                if (!(grid[gy-1]?.[Math.floor(h.px/sz)]?.type === 'wall')) h.py -= h.speed; else if (!(grid[gy+1]?.[Math.floor(h.px/sz)]?.type === 'wall')) h.py += h.speed; else h.px = 750;
            } else { h.px = nx; h.py = ny; }

            let t = grid[Math.floor(h.py/sz)]?.[Math.floor(h.px/sz)];
            if (t) {
                let d = {lava:0.8, acid:1.2, guard:1.5, spike:1.0}[t.type] || 0.1;
                if (h.name === "Knight" && t.type === "guard" && t.lvl < 3) d = 0;
                if (h.im.includes(t.type)) d = 0;
                h.hp -= d * (t.lvl || 1) * strengthMult;
            }
            if (h.hp <= 0 || h.px >= 745) {
                if (h.hp <= 0) {
                    gold += 150*treasureMult; prestige++; demons = demons.filter(d => d.isKing);
                    if (h.name === "Merchant") { document.getElementById('merchant-ui').style.display='flex'; document.getElementById('king-btn').style.display = (Math.random()<0.05)?'block':'none'; }
                } else { gold -= 350; demons = []; }
                activeHeroes = activeHeroes.filter(hero => hero !== h);
            }
        });

        projectiles.forEach(p => { p.x += p.vx; p.y += p.vy; 
            if(activeHeroes.length > 0 && Math.hypot(p.x-activeHeroes[0].px, p.y-activeHeroes[0].py) < 15) { 
                activeHeroes[0].hp -= p.type === 'fireball' ? 30 : 15; p.dead = true; 
            } 
        });
        projectiles = projectiles.filter(p => !p.dead && p.x > 0 && p.x < 800 && p.y > 0 && p.y < 400);

        [...demons, ...skeletons].forEach(m => {
            if (activeHeroes.length > 0) {
                let h = activeHeroes[0], a = Math.atan2(h.py-m.py, h.px-m.px); m.px += Math.cos(a)*1.8; m.py += Math.sin(a)*1.8;
                if (Math.hypot(m.px-h.px, m.py-h.py) < 18) { if (!(h.name === "Priest" && !m.isKing)) h.hp -= 0.5 * strengthMult; }
            }
        });

        if (autoSpawn && activeHeroes.length === 0) rollAdventurer();
        render(); requestAnimationFrame(gameLoop);
    }

    function render() {
        ctx.clearRect(0,0,800,400); document.getElementById('gold-val').innerText = Math.floor(gold);
        ctx.fillStyle = '#0f0'; ctx.fillRect(10,180,15,40); ctx.fillStyle = '#fd0'; ctx.fillRect(775,180,15,40);
        for(let r=0; r<rows; r++) for(let c=0; c<cols; c++) {
            let t = grid[r][c]; if (!t) continue;
            ctx.save(); ctx.translate(c*sz+sz/2, r*sz+sz/2); ctx.rotate(t.rot);
            if (t.type === 'wall') { ctx.fillStyle = t.permanent ? '#111' : '#222'; ctx.fillRect(-sz/2+1, -sz/2+1, sz-2, sz-2); }
            else if (t.type === 'guard') {
                if (t.lvl === 3) { ctx.fillStyle = '#f00'; ctx.beginPath(); ctx.moveTo(0,-12); ctx.lineTo(12,12); ctx.lineTo(-12,12); ctx.fill(); ctx.fillStyle='#ff0'; ctx.fillRect(-4,2,8,4); }
                else { ctx.fillStyle = t.lvl === 2 ? '#d00' : '#24a'; ctx.beginPath(); ctx.moveTo(0,-10); ctx.lineTo(10,10); ctx.lineTo(-10,10); ctx.fill(); }
            } else if (t.type === 'lava') { ctx.fillStyle = '#f40'; ctx.fillRect(-sz/2, -sz/2, sz, sz); }
            else if (t.type === 'spike') { ctx.fillStyle = '#777'; ctx.beginPath(); ctx.moveTo(0,-10); ctx.lineTo(8,10); ctx.lineTo(-8,10); ctx.fill(); }
            else if (t.type === 'acid') { ctx.fillStyle = '#141'; ctx.fillRect(-sz/2, -sz/2, sz, sz); ctx.fillStyle = '#32cd32'; ctx.globalAlpha = 0.4; ctx.beginPath(); ctx.arc(0,0,8,0,Math.PI*2); ctx.fill(); ctx.globalAlpha = 1; }
            else if (t.type === 'arrow') { ctx.fillStyle='#444'; ctx.fillRect(-8,-3,16,6); ctx.fillStyle='#fff'; ctx.beginPath(); ctx.moveTo(8,0); ctx.lineTo(0,-4); ctx.lineTo(0,4); ctx.fill(); }
            else if (t.type === 'demon') { ctx.strokeStyle='#f00'; ctx.lineWidth=1; ctx.strokeRect(-10,-10,20,20); }
            else if (t.type === 'sking') { ctx.fillStyle='#fff'; ctx.fillRect(-12,-12,24,24); ctx.fillStyle='#0ff'; ctx.fillRect(-8,-6,4,4); ctx.fillStyle='#0f0'; ctx.fillRect(4,-6,4,4); }
            ctx.restore();
        }
        activeHeroes.forEach(h => {
            ctx.fillStyle = h.color; ctx.fillRect(h.px-8, h.py-8, 16, 16);
            ctx.fillStyle='#f00'; ctx.fillRect(h.px-12, h.py-18, 24, 3);
            ctx.fillStyle='#0f0'; ctx.fillRect(h.px-12, h.py-18, 24*(h.hp/h.maxHp), 3);
            ctx.fillStyle="#fff"; ctx.font="7px Arial"; ctx.fillText(h.name, h.px-10, h.py-22);
        });
        projectiles.forEach(p => { ctx.fillStyle = p.type === 'fireball' ? '#f50' : '#fff'; ctx.beginPath(); ctx.arc(p.x, p.y, p.type === 'fireball' ? 5 : 2, 0, Math.PI*2); ctx.fill(); });
        skeletons.forEach(s => { ctx.fillStyle='#eee'; ctx.fillRect(s.px-4, s.py-4, 8, 8); });
        demons.forEach(d => { ctx.fillStyle=d.isKing?'#f0f':'#909'; ctx.beginPath(); ctx.arc(d.px,d.py,6,0,Math.PI*2); ctx.fill(); });
    }
    gameLoop();
</script>
</body>
</html>
