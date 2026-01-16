<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport"
content="width=device-width, height=device-height, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Wooly Warfare</title>

<style>
html, body {
  margin: 0;
  padding: 0;
  overflow: hidden;
  touch-action: none;
  font-family: sans-serif;
}
#ui {
  position: absolute;
  top: 10px;
  left: 10px;
  background: rgba(255,255,255,0.9);
  padding: 10px;
  border-radius: 8px;
  z-index: 10;
}
#menu {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  flex-direction: column;
  gap: 6px;
  z-index: 10;
}
.btn {
  background: #fff;
  border: 2px solid #888;
  border-radius: 8px;
  padding: 6px;
  width: 180px;
}
.btn.selected {
  border-color: orange;
}
</style>
</head>

<body>

<div id="ui">
🧶 Wool: <span id="wool">500</span>
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
renderer.setPixelRatio(window.devicePixelRatio);
document.body.appendChild(renderer.domElement);

scene.add(new THREE.AmbientLight(0xffffff, 1));

/* ================= MATERIALS ================= */
const mats = {
  grass: new THREE.MeshBasicMaterial({ color: 0x2a6a32 }),
  path: new THREE.MeshBasicMaterial({ color: 0x8b6b3e }),
  wool: new THREE.MeshBasicMaterial({ color: 0xdddddd }),
  enemy: new THREE.MeshBasicMaterial({ color: 0xcc3333 }),
  dark: new THREE.MeshBasicMaterial({ color: 0x444444 }),
  wood: new THREE.MeshBasicMaterial({ color: 0x8b5a2b })
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
  new THREE.Vector3(-70, 0, -40),
  new THREE.Vector3(-70, 0, 20),
  new THREE.Vector3(40, 0, 20),
  new THREE.Vector3(40, 0, 60)
];

// visual tiles
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

/* ================= SPAWN PEN ================= */
for (let x = -80; x <= -60; x += 4) {
  for (let z of [-50, -30]) {
    const post = new THREE.Mesh(
      new THREE.BoxGeometry(2, 4, 2),
      mats.wood
    );
    post.position.set(x, 2, z);
    scene.add(post);
  }
}
for (let z = -50; z <= -30; z += 4) {
  for (let x of [-80, -60]) {
    const post = new THREE.Mesh(
      new THREE.BoxGeometry(2, 4, 2),
      mats.wood
    );
    post.position.set(x, 2, z);
    scene.add(post);
  }
}

/* ================= MODELS ================= */
function makeEnemy() {
  const g = new THREE.Group();

  const body = new THREE.Mesh(
    new THREE.SphereGeometry(1.2, 8, 8),
    mats.enemy
  );
  body.position.y = 1.4;
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

  // health bar
  const bar = new THREE.Mesh(
    new THREE.PlaneGeometry(2.5, 0.3),
    new THREE.MeshBasicMaterial({ color: 0x00ff00 })
  );
  bar.position.y = 3;
  g.add(bar);

  g.bar = bar;
  g.hp = 20;
  g.max = 20;
  g.pathIndex = 0;

  return g;
}

/* ================= ENEMIES ================= */
const enemies = [];
let spawnTimer = 0;

function spawnEnemy() {
  const e = makeEnemy();
  e.position.copy(path[0]);
  enemies.push(e);
  scene.add(e);
}

/* ================= TOWERS (PLACEMENT ONLY) ================= */
const TOWERS = { Cube: 10 };
let wool = 500;
let selected = "Cube";

const menu = document.getElementById("menu");
const b = document.createElement("div");
b.className = "btn selected";
b.textContent = "Cube (10)";
menu.appendChild(b);

/* ================= PLACEMENT ================= */
const ray = new THREE.Raycaster();
const pointer = new THREE.Vector2();
let taps = 0, tapTimer;

renderer.domElement.addEventListener("pointerdown", e => {
  taps++;
  if (taps === 1) {
    tapTimer = setTimeout(() => taps = 0, 300);
  } else {
    clearTimeout(tapTimer);
    taps = 0;

    const p = e.touches ? e.touches[0] : e;
    const rect = renderer.domElement.getBoundingClientRect();

    pointer.x = ((p.clientX - rect.left) / rect.width) * 2 - 1;
    pointer.y = -((p.clientY - rect.top) / rect.height) * 2 + 1;

    ray.setFromCamera(pointer, camera);
    const hit = ray.intersectObject(ground);
    if (hit.length) {
      const sheep = new THREE.Mesh(
        new THREE.SphereGeometry(1, 8, 8),
        mats.wool
      );
      sheep.position.set(Math.round(hit[0].point.x), 1, Math.round(hit[0].point.z));
      scene.add(sheep);
    }
  }
});

/* ================= LOOP ================= */
function animate() {
  requestAnimationFrame(animate);

  spawnTimer++;
  if (spawnTimer > 120) {
    spawnTimer = 0;
    spawnEnemy();
  }

  enemies.forEach(e => {
    const next = path[e.pathIndex + 1];
    if (!next) return;
    const dir = next.clone().sub(e.position);
    e.lookAt(next.x, 1.4, next.z);
    if (dir.length() < 0.5) e.pathIndex++;
    e.position.add(dir.normalize().multiplyScalar(0.05));
    e.bar.scale.x = e.hp / e.max;
  });

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
