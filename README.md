<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Wooly Warfare</title>
<style>
body {
  margin:0;
  overflow:hidden;
  font-family:sans-serif;
  touch-action:none;
}
#ui {
  position:absolute;
  top:10px;
  left:10px;
  background:rgba(255,255,255,0.9);
  padding:10px;
  border-radius:8px;
}
#menu {
  position:absolute;
  right:10px;
  top:50%;
  transform:translateY(-50%);
  display:flex;
  flex-direction:column;
  gap:6px;
}
.btn {
  background:#fff;
  border:2px solid #888;
  border-radius:8px;
  padding:6px;
  width:170px;
  font-size:13px;
}
.btn.selected {
  border-color:#ff9900;
  background:#fff3dd;
}
#startBtn {
  margin-top:6px;
  width:100%;
}
</style>
</head>
<body>

<div id="ui">
🧶 Wool: <span id="wool">50</span><br>
🔁 Round: <span id="round">1</span><br>
<button id="startBtn">Start Round</button>
</div>

<div id="menu"></div>

<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
<script>
/* ================= SCENE ================= */
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x87ceeb);

const camera = new THREE.PerspectiveCamera(60, innerWidth / innerHeight, 0.1, 500);
camera.position.set(45, 55, 65);
camera.lookAt(0, 0, 0);

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(window.devicePixelRatio); // ✅ MOBILE FIX
document.body.appendChild(renderer.domElement);

scene.add(new THREE.AmbientLight(0xffffff, 1));

/* ================= MATERIALS ================= */
const mats = {
  grass: new THREE.MeshBasicMaterial({ color: 0x2b6f34 }),
  path: new THREE.MeshBasicMaterial({ color: 0x8b6b3e }),
  wool: new THREE.MeshBasicMaterial({ color: 0xdddddd }),
  goat: new THREE.MeshBasicMaterial({ color: 0xc9b27c }),
  enemy: new THREE.MeshBasicMaterial({ color: 0xcc3333 }),
  fast: new THREE.MeshBasicMaterial({ color: 0x3366cc }),
  tank: new THREE.MeshBasicMaterial({ color: 0x770000 }),
  boss: new THREE.MeshBasicMaterial({ color: 0x990000 }),
  dark: new THREE.MeshBasicMaterial({ color: 0x444444 }),
  horn: new THREE.MeshBasicMaterial({ color: 0x8b5a2b }),
  crown: new THREE.MeshBasicMaterial({ color: 0xffd700 }),
  shot: new THREE.MeshBasicMaterial({ color: 0x44cc44 }),
  metal: new THREE.MeshBasicMaterial({ color: 0xcccccc })
};

/* ================= GROUND ================= */
const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(200, 200),
  mats.grass
);
ground.rotation.x = -Math.PI / 2;
scene.add(ground);

/* ================= PATH ================= */
const path = [
  new THREE.Vector3(-50, 0, -40),
  new THREE.Vector3(-50, 0, 20),
  new THREE.Vector3(40, 0, 20)
];
for (let i = 0; i < path.length - 1; i++) {
  const a = path[i], b = path[i + 1];
  const len = a.distanceTo(b);
  for (let j = 0; j <= len / 4; j++) {
    const tile = new THREE.Mesh(
      new THREE.BoxGeometry(4, 0.2, 4),
      mats.path
    );
    tile.position.copy(a.clone().lerp(b, j / (len / 4)));
    tile.position.y = 0.1;
    scene.add(tile);
  }
}

/* ================= MODELS ================= */
function makeSheep(mat, scale = 1) {
  const g = new THREE.Group();

  const body = new THREE.Mesh(
    new THREE.SphereGeometry(1.2, 8, 8),
    mat
  );
  body.position.y = 1.4;
  body.scale.set(scale, scale, scale);
  g.add(body);

  const face = new THREE.Mesh(
    new THREE.BoxGeometry(0.6, 0.6, 0.6),
    mats.dark
  );
  face.position.set(0, 1.4, 1.1);
  g.add(face);

  [-0.4, 0.4].forEach(x => {
    [-0.4, 0.4].forEach(z => {
      const leg = new THREE.Mesh(
        new THREE.CylinderGeometry(0.12, 0.12, 0.8),
        mats.dark
      );
      leg.position.set(x, 0.4, z);
      g.add(leg);
    });
  });

  g.body = body;
  return g;
}

/* ================= TOWERS ================= */
const TOWERS = {
  cube: { cost: 10 },
  ram: { cost: 25 },
  grazer: { cost: 100 },
  goat: { cost: 500 },
  shearer: { cost: 250 }
};

let wool = 50;
let selected = "cube";
const towers = [];

/* ================= MENU ================= */
const menu = document.getElementById("menu");
for (const k in TOWERS) {
  const b = document.createElement("div");
  b.className = "btn" + (k === selected ? " selected" : "");
  b.textContent = k.toUpperCase() + " (" + TOWERS[k].cost + ")";
  b.onclick = () => {
    selected = k;
    document.querySelectorAll(".btn").forEach(x => x.classList.remove("selected"));
    b.classList.add("selected");
  };
  menu.appendChild(b);
}

/* ================= DOUBLE TAP (FIXED) ================= */
let taps = 0, tapTimer = null;
const ray = new THREE.Raycaster();
const mouse = new THREE.Vector2();

renderer.domElement.addEventListener("pointerdown", e => {
  taps++;
  if (taps === 1) {
    tapTimer = setTimeout(() => taps = 0, 300);
  } else {
    clearTimeout(tapTimer);
    taps = 0;

    const rect = renderer.domElement.getBoundingClientRect(); // ✅ KEY FIX

    mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
    mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;

    ray.setFromCamera(mouse, camera);
    const hit = ray.intersectObject(ground);
    if (hit.length) placeTower(hit[0].point);
  }
});

function placeTower(p) {
  const cost = TOWERS[selected].cost;
  if (wool < cost) return;
  wool -= cost;
  document.getElementById("wool").textContent = wool;

  const u = makeSheep(
    selected === "goat" ? mats.goat : mats.wool,
    selected === "goat" ? 1.3 : 1
  );
  u.position.set(Math.round(p.x), 0, Math.round(p.z));
  scene.add(u);
  towers.push(u);
}

/* ================= ROUND BUTTON (STUB) ================= */
document.getElementById("startBtn").onclick = () => {
  alert("Rounds work — enemies omitted here for placement test build");
};

/* ================= LOOP ================= */
function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}
animate();

window.addEventListener("resize", () => {
  camera.aspect = innerWidth / innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth, innerHeight);
});
</script>
</body>
</html>
