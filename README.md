<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Dungeon Architect 2026: The Final Edition</title>
    <style>
        body { background: #0a0a0c; color: #d4af37; font-family: 'Garamond', serif; display: flex; flex-direction: column; align-items: center; margin: 0; padding: 5px; overflow: hidden; }
        #ui { background: #1a1a1c; border: 2px solid #444; padding: 10px; border-radius: 5px; display: flex; gap: 6px; flex-wrap: wrap; justify-content: center; max-width: 950px; }
        canvas { background: #050505; border: 3px solid #333; image-rendering: pixelated; touch-action: none; max-width: 100%; }
        .btn { padding: 8px 10px; border: 1px solid #d4af37; background: #222; color: #d4af37; cursor: pointer; font-size: 0.75em; font-weight: bold; }
        .selected { background: #d4af37; color: #111; box-shadow: 0 0 10px #d4af37; }
        .gold { font-size: 1.2em; color: #ffd700; width: 100%; text-align: center; margin-bottom: 2px; }
        #upgrade-menu { position: absolute; background: #111; border: 2px solid #ffd700; padding: 12px; display: none; flex-direction: column; gap: 8px; z-index: 2000; box-shadow: 0 0 20px #000; color: #fff; border-radius: 4px; }
    </style>
</head>
<body>
    <div class="gold">TREASURY: <span id="gold-val">1500</span>g</div>
    
    <div id="upgrade-menu">
        <div id="menu-title" style="font-weight:bold; color:#ffd700; border-bottom:1px solid #444;">TRAP INFO</div>
        <div id="lvl-info">Level: 1</div>
        <button class="btn" id="up-btn" onclick="applyUpgrade()">UPGRADE (75g)</button>
        <button class="btn" onclick="closeMenu()" style="border-color:#555">BACK</button>
    </div>

    <div id="ui">
        <button class="btn selected" onclick="setBrush('wall', 0, event)">WALL (FREE)</button>
        <button class="btn" onclick="setBrush('barrier', 40, event)">BARRIER (40g)</button>
        <button class="btn" onclick="setBrush('lava', 35, event)">LAVA (35g)</button>
        <button class="btn" onclick="setBrush('acid', 35, event)">ACID (35g)</button>
        <button class="btn" onclick="setBrush('guard', 150, event)">GUARD (150g)</button>
        <button class="btn" onclick="setBrush('arrow', 100, event)">ARROW (100g)</button>
        <button class="btn" onclick="setBrush('demon', 200, event)">DEMON (200g)</button>
        <button class="btn" onclick="setBrush('rotate', 0, event)" style="border-color: #0088ff;">ROTATE</button>
        <button class="btn" onclick="setBrush('sell', 0, event)" style="border-color: #f00;">DELETE/SELL</button>
        <button class="btn" id="auto-btn" onclick="toggleAuto()">AUTO-SPAWN: OFF</button>
        <button class="btn" onclick="rollAdventurer()" style="border-color: #0f0; width: 100%;">SUMMON HERO</button>
    </div>
    <canvas id="dungeon" width="800" height="400"></canvas>

<script>
    const canvas = document.getElementById('dungeon');
    const ctx = canvas.getContext('2d');
    const sz = 40, rows = 10, cols = 20;
    let gold = 1500, brush = 'wall', cost = 0, prestige = 0, autoSpawn = false;
    let menuTarget = null;
    const spawn = {x: 20, y: 220}, treasure = {x: 760, y: 220};
    let grid = Array.from({length: rows}, () => Array(cols).fill(null));
    let arrows = [], demons = [], activeHero = null;

    const Classes = [
        { name: "Rogue", color: "#8a2be2", hp: 60, immune: "spike" },
        { name: "Wizard", color: "#4b0082", hp: 70, immune: "arrow" },
        { name: "Minotaur", color: "#5d4037", hp: 250, immune: "none" }
    ];

    function setBrush(t, c, e) { brush = t; cost = c; document.querySelectorAll('.btn').forEach(b => b.classList.remove('selected')); if(e) e.target.classList.add('selected'); closeMenu(); }
    function toggleAuto() { autoSpawn = !autoSpawn; const b = document.getElementById('auto-btn'); b.classList.toggle('selected'); b.innerText = `AUTO-SPAWN: ${autoSpawn ? 'ON' : 'OFF'}`; if(autoSpawn && !activeHero) rollAdventurer(); }
    function closeMenu() { document.getElementById('upgrade-menu').style.display = 'none'; menuTarget = null; }
    function applyUpgrade() { if(menuTarget && menuTarget.lvl < 3 && gold >= 75) { gold -= 75; menuTarget.lvl++; closeMenu(); } }

    canvas.addEventListener('pointerdown', (e) => {
        const r = canvas.getBoundingClientRect();
        const x = Math.floor((e.clientX - r.left) / (r.width / cols)), y = Math.floor((e.clientY - r.top) / (r.height / rows));
        if (y < 0 || y >= rows || x < 0 || x >= cols) return;
        const ex = grid[y][x];

        if (brush === 'sell') { if (ex) { if (ex.type !== 'wall') gold += 20; grid[y][x] = null; } return; }
        if (ex) {
            if (brush === 'rotate') { ex.rot += Math.PI/2; return; }
            if (ex.type !== 'wall') {
                menuTarget = ex; const menu = document.getElementById('upgrade-menu');
                menu.style.display = 'flex'; menu.style.left = Math.min(e.clientX, window.innerWidth - 150) + 'px'; menu.style.top = Math.min(e.clientY, window.innerHeight - 150) + 'px';
                document.getElementById('lvl-info').innerText = "Level: " + ex.lvl; document.getElementById('up-btn').style.display = ex.lvl >= 3 ? 'none' : 'block';
                return;
            }
        } else if (gold >= cost) { grid[y][x] = { type: brush, rot: 0, lastFire: Date.now(), hp: 100, lvl: 1 }; gold -= cost; }
    });

    function rollAdventurer() { if (activeHero) return; const h = Classes[Math.floor(Math.random()*Classes.length)]; activeHero = { ...h, px: spawn.x, py: spawn.y, hp: h.hp + (prestige * 10), maxHp: h.hp + (prestige * 10) }; }

    function gameLoop() {
        const now = Date.now();
        for(let r=0; r<rows; r++) for(let c=0; c<cols; c++) {
            let t = grid[r][c]; if (!t || t.type === 'wall') continue;
            if (t.type === 'arrow' && now - t.lastFire > (1300 - t.lvl*350)) { arrows.push({x: c*sz+20, y: r*sz+20, vx: Math.cos(t.rot)*7, vy: Math.sin(t.rot)*7, dmg: 10 + t.lvl*10}); t.lastFire = now; }
            if (t.type === 'demon' && now - t.lastFire > (15000 / t.lvl)) { demons.push({px: c*sz+20, py: r*sz+20, isKing: Math.random()<0.01}); t.lastFire = now; }
        }

        if (activeHero) {
            let a = Math.atan2(treasure.y - activeHero.py, treasure.x - activeHero.px);
            let vx = Math.cos(a) * 2.2, vy = Math.sin(a) * 2.2;
            let nx = activeHero.px + vx, ny = activeHero.py + vy;
            let gx = Math.floor(nx/sz), gy = Math.floor(ny/sz);
            let block = grid[gy]?.[gx];

            if (block && block.type === 'wall') {
                let canUp = !(grid[gy-1]?.[Math.floor(activeHero.px/sz)]?.type === 'wall');
                let canDown = !(grid[gy+1]?.[Math.floor(activeHero.px/sz)]?.type === 'wall');
                let canBack = !(grid[gy]?.[Math.floor((activeHero.px - vx)/sz)]?.type === 'wall');
                if (canUp) activeHero.py -= 2.2; 
                else if (canDown) activeHero.py += 2.2; 
                else if (!canBack) { activeHero.px = treasure.x; activeHero.py = treasure.y; }
                else activeHero.px -= 2.2;
            } else if (block && block.type === 'barrier') {
                if (activeHero.name === "Minotaur") grid[gy][gx] = null;
                else { block.hp -= 2; if (block.hp <= 0) grid[gy][gx] = null; }
            } else { activeHero.px = nx; activeHero.py = ny; }

            let t = grid[Math.floor(activeHero.py/sz)]?.[Math.floor(activeHero.px/sz)];
            if (t && t.type !== activeHero.immune) activeHero.hp -= {lava:0.8*t.lvl, acid:1.2*t.lvl, guard:1.5*t.lvl}[t.type] || 0.1;
            if (activeHero.hp <= 0 || activeHero.px >= treasure.x - 5) { gold += (activeHero.hp <= 0 ? 200 : -200); prestige++; activeHero = null; demons = demons.filter(d => d.isKing); if(autoSpawn) setTimeout(rollAdventurer, 1000); }
        }

        demons.forEach(d => { if (activeHero) { let a = Math.atan2(activeHero.py-d.py, activeHero.px-d.px); d.px += Math.cos(a)*1.4; d.py += Math.sin(a)*1.4; if (Math.hypot(d.px-activeHero.px, d.py-activeHero.py) < 20) activeHero.hp -= 0.4; } });
        arrows.forEach(a => { a.x += a.vx; a.y += a.vy; if(activeHero && Math.hypot(a.x-activeHero.px, a.y-activeHero.py) < 18) { activeHero.hp -= a.dmg; a.dead = true; } });
        arrows = arrows.filter(a => !a.dead && a.x > 0 && a.x < 800 && a.y > 0 && a.y < 400);
        render(); requestAnimationFrame(gameLoop);
    }

    function render() {
        ctx.clearRect(0,0,800,400); document.getElementById('gold-val').innerText = gold;
        ctx.fillStyle = '#0f0'; ctx.fillRect(0, 200, 20, 40); ctx.fillStyle = '#fd0'; ctx.fillRect(780, 200, 20, 40);
        for(let r=0; r<rows; r++) for(let c=0; c<cols; c++) {
            let t = grid[r][c]; if (!t) continue;
            ctx.save(); ctx.translate(c*sz+20, r*sz+20);
            if (t.type === 'wall') { ctx.fillStyle = '#333'; ctx.fillRect(-18,-18,36,36); }
            else if (t.type === 'acid') { ctx.fillStyle = '#32cd32'; ctx.globalAlpha = 0.6; ctx.fillRect(-20,-20,40,40); ctx.globalAlpha = 1; }
            else if (t.type === 'guard') { 
                if (t.lvl < 3) { ctx.fillStyle = '#24a'; ctx.beginPath(); ctx.moveTo(0,-18); ctx.lineTo(15,15); ctx.lineTo(-15,15); ctx.fill(); }
                else { ctx.fillStyle = '#d00'; ctx.beginPath(); ctx.moveTo(0,-20); ctx.lineTo(18,10); ctx.lineTo(-18,10); ctx.fill(); ctx.fillStyle = '#fa0'; ctx.fillRect(-25,-5,15,5); ctx.fillRect(10,-5,15,5); }
            } else { ctx.rotate(t.rot); ctx.fillStyle = t.type === 'lava' ? (t.lvl===3?'#ff0':'#f40') : (t.type === 'barrier' ? '#543' : '#fff'); ctx.fillRect(-15,-15,30,30); }
            if(t.type !== 'wall') { ctx.fillStyle = "#fff"; ctx.font = "bold 9px Arial"; ctx.fillText("L"+t.lvl, -10, 15); }
            ctx.restore();
        }
        if (activeHero) {
            ctx.fillStyle = activeHero.color; ctx.fillRect(activeHero.px-12, activeHero.py-12, 24, 24);
            if (activeHero.name === "Minotaur") { ctx.fillStyle="#eee"; ctx.fillRect(activeHero.px-18, activeHero.py-15, 8, 4); ctx.fillRect(activeHero.px+10, activeHero.py-15, 8, 4); }
            if (activeHero.name === "Wizard") { ctx.fillStyle="#00f"; ctx.beginPath(); ctx.moveTo(activeHero.px-12,activeHero.py-12); ctx.lineTo(activeHero.px,activeHero.py-28); ctx.lineTo(activeHero.px+12,activeHero.py-12); ctx.fill(); }
        }
        demons.forEach(d => { ctx.fillStyle = d.isKing ? '#f0f' : '#909'; ctx.fillRect(d.px-8, d.py-8, 16, 16); });
        ctx.fillStyle = '#fff'; arrows.forEach(a => ctx.fillRect(a.x-2, a.y-2, 4, 4));
    }
    gameLoop();
</script>
</body>
</html>
